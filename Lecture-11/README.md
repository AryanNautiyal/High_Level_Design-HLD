

*** Continuing : Authentication and Authorization (Lecture-10) ***

# OAuth 2.0

    -- So user can register basically either by entering his details like username, password, email, mobile number, etc

    -- Or the user will either sign in with Google or facebook account with the options given in the website on the bottom 

    -- So sign in with Google or Facebook works by using OAuth 2.0

    -- So what happens in this case is that the 3rd party application requests Google or Facebook to give it the access token which
        contains all the details of the user with user's permission 

    -- So in access token only those information will be present that is required by the 3rd party application and no extra 
        information is shared in the access token

    -- So like if we cannot trust 3rd party application that much we can trust google to only share some details and we have also 
        seen that google asks us for confirmation also telling what all information the 3rd party application is asking 

    -- So we give confirmation to google to send those details to 3rd party application which also improves the UX as user doesn't 
        need to manually fill all the details instead he can just do it quickly by letting google handle it 

    -- So even if the 3rd party application gets hacked or goes down then our information won't get leaked much as only access to 
        those information will be given that the 3rd party application asked (as we gave the permission to share those details with
        them) 


        {
            Personal Thoughts : I think it's much safer that once I get information about user I store it in DB and create my own 
                                token and give to user for accessing the application as using google's will complicate things so 
                                like then if user logs out and then login with google account again I can just match google ID and 
                                just give it new access token 

                        [Gemini Response]

            1. The "First Time" Flow (Registration)
            
                (1) User logs in via Google.Google sends your backend the Google ID and email.
                
                (2) Your backend checks the DB: "Does this Google ID exist?" --> No.
                
                (3) Your backend creates a new user record in MongoDB with that Google ID.
                
                (4) Your Step: You generate your own JWT (JSON Web Token) or Session Cookie.
                
                (5) The Result: You send your token to the user’s browser.
                
            2. The "Returning" Flow (The "Match" Logic)
            
                This is the part you just described, and you're exactly right :
                
                        (1) User clicks "Login with Google" again.
                        
                        (2) Google verifies them and sends you the Google ID.
                        
                        (3) Your backend checks the DB: "Does this Google ID exist?" --> Yes!
                        
                        (4) The Shortcut: You don't ask for a password. You don't care about Google's access tokens anymore. You 
                                        simply generate a brand new internal token for them.
                                        
                        (5) The Result: The user is logged in instantly.


            Why your idea is "Much Safer"

                -- Isolation: If Google’s servers go down for 10 minutes, users who are already logged into Zelce can keep working 
                                because your server is checking your tokens, not Google's.

                -- Database Control: You control when your token expires (e.g., 24 hours or 7 days), regardless of Google's 1-hour 
                                        limit.

                -- Simplicity: You don't have to manage Refresh Tokens or complex Google API calls every time the user clicks a 
                                button on your site.

        }



*** Autocomplete Engine ***

        -- Other name for this problem is *** Design top k most searched queries ***

        -- So like in Google we search like 'sw' so it autocompletes and gives suggestions like swan, swing, etc like this 

        -- So it almost gives results in O(1) time as it doesn't take any time for loading instead it just instantly shows us the 
            result

        -- So today we will design this autocomplete engine 


    Q : How many autocomplete suggestion is needed ?

    Ans : So let's assume that interviewer says that 6 autocomplete suggestions should be there


    Q : How to decide the search result ?

    Ans : Based on popularity (frequency)

          So like for any word or letter we will have many autocomplete results so we only need 6 from all of them so we will select
          those 6 based on the popularity or in other words can say that those that are accessed very frequently or again and 
          again so like this we will show the top 6 results in autocomplete engine  


    Q : Search Key Length ?

    Ans : So in search box user can type how many characters till which we can search for top result and give 6 frequently accessed 
          results 

          So let's assume interviewer says max length let's assume 100 so user can give 100 characters max in search box for which
          it will autocomplete 


    Q : Which languages are supported ? 

    Ans : So let's assume interviewer says that for now keep it in English later we will expand it to cover other languages also 


    Q : Should we need to sort the result ?

    Ans : So let's assume interviewer says we need to sort results that we get from autocomplete like for 'sw' we get some results 
          with frequency like let's say 1000, 1200, 3000, etc so we will just sort them in order like the most popular result will
          be shown first then second then third so in frequencies like 3000, then 1200, then 1000, etc like this

    
    Q : Is search result case sensitive ?

    Ans : So let's assume interviewer says no that if user types 'sw' or 'sW' the results should be same 


    Q : How many daily active users (DAU) ?

    Ans : Let's assume interviewer says that our DAU is 5 Million


*** # Requirements ***

    (1) Fast response time --> almost O(1)

    (2) Sorted by popularity (frequency)

    (3) Scalable and Availability over consistency 


*** Back of the Envelope calculations ***

    Assumptions :

        -- DAU = 5 Million 

        -- A person does 10 search result per day (average)

        -- Each search query has 30 bytes of data on an average like we know 1 character = 1 byte so 30 characters = 30 bytes so we
            assume that for each search query we get 30 bytes of data 

    QPS :
    
        -- So internally we are expecting like when user types 1 letter it sends 1 API GET request then when user types another     
            letter then it just sends one more API call and so on 

        -- So like this user sends APIs call so we assumed that on an average user searches 30 character long data which means 30 
            bytes of data

        -- So for 30 characters we will be needed to hit 30 queries on DB per search result 

        -- So for calculating QPS we will calculate by as 5 millions people are daily active users and each user from this does 10 
            searches in a day and each search on an average is 30 characters long which hits DB 30 times

            So,

            QPS = (5 Million * 10 * 30) / 86400 = (5 * 2^20 * 10 * 30) / 86400 = (1500 * 2^20) / 86400 
            
            QPS = (1.5 * 10^3 * 2^20) / (86.4 * 10^3) = (1.5 * 2^20) / 86.4 = 0.01736 * 2^20 = ~ 0.01736 million request / sec

            QPS = 0.01736 * 2^10 thousand req/sec = 0.01736 * 10^3 thousand req/sec = ~ 17.36 thousand req/sec = ~ 18000 req/sec

            or can say ~ 17360 req/sec

        Peak QPS = 2 * QPS = 2 * 18000 = ~ 36000 req/sec

    
    Storage Unit :

        Assume for 5 yrs

            So 5 million user daily search something and each user performs 10 searches in a day and each search takes 30 bytes of 
            data on an average 

            So,

            Storage Unit = 5 * 2^20 * 10 * 30 * 365 * 5 = 2737500 * 2^20 = 2.7375 * 10^6 * 2^20 = 2.74 * 2^20 * 2^20 = 2.74 * 2^40

            Storage Unit = ~ 2.74 TB = ~ 3 TB

            So 3 TB of storage will be required to store the query in our DB for 5 yrs


*** High Level Overview ***

    -- So basically in our autocomplete search engine there will be 3 components one is the autocomplete search engine and the other
        two are data gathering service and data query service

    -- Data gathering service will gather all the data that relates to the user search query and then data query service will query
        on this data to get the top 6 to show to the user 


    Data Gathering Service 

        -- Responsible for collecting information (like for example gathers all the data about things that start from 's' from data
            pool) 

        -- It basically deals with how we are actually storing all the data

    
    Data Query Service

        -- Given a search query or prefix, return top 6 most searched result 

        -- It basically deals with queries on the data obtained from data gathering service


    -- So we will first go in depth of data gathering service

    
    *** Data Gathering Service *** 

        -- Let's assume we are choosing SQL DB to store our data and basic frequency table (which will be initially empty)

        -- So let's assume in frequency table we have 2 columns query and frequency 

        -- So in frequency table what will happen is whatever user searches for will be stored in query column and for that we will
            initialize the frequency to 1 if it unique in that table and if it's not unique then the frequency will be incremented 
            like if someone searches swing first then it will be inserted in DB with frequency 1 and when another user searches the 
            same word the frequency of the swing will be incremented meaning it will be 2 now 

    
    *** Data Query Service *** 

        -- From all the data gathered in the DB by data gathering service, the data query service performs operations on that data 
            to obtain the top 6 popular words to give to user

        -- So SQL query for it will look like this

                    SELECT * FROM Frequency WHERE Query LIKE 'prefix%' ORDER BY frequency DESC LIMIT 6;

            So like if we are searching for top 6 most searched result starting with letters 'sw' then query will be like this

                    SELECT * FROM Frequency WHERE Query LIKE 'sw%' ORDER BY frequency DESC LIMIT 6;
            
        -- So now if after letters 'sw' users gives one more letter let's say 'i' then another API call will be done to backend 
            and now the query will be looking like 

                    SELECT * FROM Frequency WHERE Query LIKE 'swi%' ORDER BY frequency DESC LIMIT 6;


*** Problem in above solution and next solution ***

    -- If dataset is small then above solution will work fine but our dataset will be large in this case as we have 5 million DAU
        and our SQL query will also work slow as first it will see each data in DB then will select the ones that match the 
        conditions and then it will take time to sort them according to the frequencies so overall it will take some time whereas
        in our requirements we need almost O(1) time 

    -- So now we move towards our next solution in which we want data storing and data retrieving to be fast

    -- So we will use Tries Data Structure (as we already know about tries data structure)

    -- So we are choosing tries as in this we can search a data in O(p) where p is the length of the prefix to be searched so it is 
        faster than our SQL query in terms of searching 

        So it will look like this 

                    Tries 
                    {
                        Tries * Children[26]

                        bool eow;

                    }
    
        eow = end of word and it is marked as true only when a word ends at a node in tries 

    
    -- So instead of using normal Tries data structure we are modifying it according to our usecase 

*** Problems with normal Tries data structure ***

        So to implement autocomplete we will need to follow these steps :

                p = length of the prefix to be searched 

                n = total number of nodes in tries

                c = number of children of a given node

            So to find Top k most frequent search we will follow these step :

                1. Find the prefix : To find the top k results we will first be needed to identify the prefix in the tries so
                                        like if we need to search for words related to 'sw' then we will be needed to locate 'sw'
                                        in the tries first 

                                    It will take O(p) time

                2. Traverse the subtree from this node to get all the valid children : So after finding the prefix we will be 
                        needed to search all the words that start from that prefix in tries so the time this will take will 
                        depend upon the number of childs the prefix last letter has in tries as if in 'sw' the letter 'w' has 
                        childs 'i', 'a' then it will have to search in these 2 paths for all the possible words that can be 
                        there (will have to go inside those path one by one)

                        So it will take O(c) time 

                3. Sort the result on the basis of frequency : So after finding all the words we will sort them according to 
                                                                their frequencies to obtain the top 6 most searched words 

                                                                So sorting will take O(clogc) time

                                                                We will just store frequency in each node as if eow = 1 then it
                                                                means a word end is there and we will just put frequency of 
                                                                that word in that node itself

                                                                Tries 
                                                                    {
                                                                        Tries * Children[26]

                                                                        bool eow;

                                                                        int freq;

                                                                    }



            So the total time complexity will be --> O(p) + O(c) + O(clogc) 

            So this is also very time taking so we need to make it fast 

            So in worst case we will just traverse the whole tries only which is not good 


*** How to make this fast ***

    (1) Limit the max length of query (100 length max)

            -- So this will make our tries faster as we have already discussed above the max limit we have set is to 100 
                characters

            -- So our p will be at max 100

            -- So our system can easily search 100 letters in O(1) time in tries so hence our O(p) --> O(1)

    (2) Cache top 6 results for each node 

            -- Since our k value is very small i.e. we only need to show top 6 results so we can just cache it  

            -- So at each node we will cache our top 6 results so that after finding the prefix in that node itself there will be 
                the result i.e. top 6 result so our searching will take only O(1) time as there was no need to traverse the subtree
                from this node to get all the valid children and there was also no need to sort the result to obtain the top 6 
                result

            -- So now our only step will be to find the prefix 

            -- Since we will cache the top 6 results in ordered format only so our total time complexity will be O(1) + O(1) = O(1)
                hence we now know why time complexity was O(1)

            -- So here we did a time-space tradeoff as we took more space to do the work in less time

            -- So now we just need to know how will we cache all this data

            -- So as we have already discussed before in Notification service design the difference between real time systems and 
                soft real time systems 

            -- So making a real time system in this case is almost impossible as if we keep on updating tries again and again as 
                many people will be just using this so our throughput will be increased and this can lead to DB failure (as each
                user will search something and increase frequency of one word and millions of user will do it so in real time 
                system millions of request will be there in milliseconds to update the tries which might crash the DB as making 
                tries, calculating each word frequency and caching top 6 result is time taking)

                {


                    -- Soft real-time and almost/near real-time systems are closely related but not identical. Both prioritize 
                        speed, but soft real-time systems have defined deadlines where missing them lowers service quality (e.g., 
                        video streaming), whereas almost/near real-time systems often mean data is processed immediately but with, 
                        for example, a few seconds' delay, such as a stock ticker. 

                    Key Differences and Overlaps:

                        -- Soft Real-Time Systems (SRT): These systems have explicit, albeit flexible, deadlines. Missing a 
                                                        deadline results in degraded service (e.g., dropped frames in video, audio 
                                                        stutter) but does not cause system failure.

                        -- Almost/Near Real-Time (NRT) Systems: These usually refer to systems where the latency is within human 
                                                                perception constraints (e.g., a website updating). While they often 
                                                                operate in a "soft" manner, they may not always have strictly 
                                                                defined, hard-coded temporal deadlines for every task like an SRT 
                                                                system does.
                }


            -- Also as we might have seen in Google's autocomplete it is not always necessary that our top 6 result will always 
                change like if our word 'sword' has 1000 frequency and 'swan' has 800 frequency so if 1 user searches 'swan' then also the top 6 result will be same as gap is huge between them so it's not practical to update tries for each and
                every request

            -- So in this we will build a almost real time system as we need to update our tries in some interval instead of every 
                request 

            *** Real time vs Timely updates ***

                -- So if we take example of X, in X we see that a hashtag is trending and all that so they require a much more real 
                    time system to show the trending hashtag so they will use different DB and other ways to make it more real time
                    but as in our case to make a autocomplete engine we need to use Tries data structure

                -- So in our case we need to do timely updates as making it real time is not useful for our usecase as for the 
                    reasons discussed above

            
            -- So we will make a Analytics logs in which there will be SQL table only in which we will just append data so append 
                only table so in this we will store the query searched by the user and the timestamp when that query was done

                        Like this  

                                Query       Timestamp
                                
                                swing       Feb 2 2026 22:48:03

                                swan        Feb 2 2026 22:48:05

                (Obviously our timestamp will be in unix epochs { epoch is a fixed reference point in time from which a computer 
                system measures time })

            -- So analytics logs will just keep on storing data so even if same word like 'swan' is searched again then also it will
                make the entry for it 

            -- So now we will make a aggregator service which will pick raw data in analytics logs and will understand that data 
                and convert it into aggregated data

            -- So for non-real time system we can just execute aggregator service once a week or we can adjust it to once a day to 
                make it more real time system or every hour also but in our case there won't be much problem even if we run it every
                week so we can consider it as a cron job which runs every week

            -- So our aggregated data will look like this

                    Query       Time        Freq

                    swing       -           2

                    swam        -           4


                So as our aggregator service will run weekly so our aggregator data will be made weekly

                (Time stored in this will be the time of updation {time will be of one week as it's run every week} as every week 
                    the frequency only will be updated in this so time will be noted of updation)


                {

                    1. The "Last Updated" Timestamp (Most Common)
                    
                            If your aggregator runs weekly, the timestamp for "swing" will be updated to Feb 9, 2026 (one week 
                            after your last run).

                            Why? This tells the system that the frequency of 2 is the most current data we have.

                            Use Case: This is great for Cache Invalidation. If the Trie DB sees a record that hasn't been updated 
                            in 3 weeks, it might decide that "swing" is no longer popular and delete it to save space.

                    2. The "Window" Timestamp

                            Sometimes, instead of just the "last update," we store the Time Range the frequency belongs to (e.g., "Week 5 of 2026").

                            Why? This allows you to track Trends. If "swing" had a frequency of 10 last week and 10,000 this week, the system knows this is a "Trending Topic" because it can compare the two timestamps.

                            Use Case: Essential for showing "What's Hot" or "Trending Now" in your search suggestions.

                }


            -- So now we have introduced workers which will internally have messaging queues in which aggregated data will be fed 
                as it is ready

            -- So it will keep feeding data in messaging queue and from these queues workers will asynchronously perform operations
                and will get data from messaging queues and will build tries from that data and will store it in trie DB  


                {
                    When your messaging queue feeds the data to the Workers to build the Trie, the Worker looks at the timestamp to 
                    make a decision:

                        -- If the query is new: It creates a new node in the Trie.

                        -- If the query exists: It checks the timestamp. If the timestamp is newer than the one currently in the 
                            Trie, it overwrites the frequency with the new aggregated count.

                    -- Worker can asynchronously work like by allocating work to each like A-D one worker, E-I second and so on

                }

                
            -- Then from trie DB the trie will be stored in trie cache so that faster read operations can be performed 

            -- As DB are slow so everytime we take data from DB it will be time taking so we chose cache for it 

            -- So both cache and DB will hold the weekly updated data 

            -- So now we need to choose a DB that will work very well for our usecase as trie DB is not a DB 

            -- So our trie DB will be persistent storage whereas our cache will be temporary storage

            -- So to store tries in our DB we need to convert it to key-value pair in any way then we can use MongoDB, Cassandra DB
                , etc

            -- So our trie will be stored periodically 

            -- So we can directly store our trie in cache as cache is in-memory so we can make trie data structure and directly 
                implement it in cache but we cannot do same in DB as there is no DB that directly supports tries 

            -- So we will use an existing DB to store our trie (so will need to convert it to key and value pair to store it in DB)

                {Used NoSQL DB as it was better for scaling reasons}

            -- So we need a way now to serialize trie data in key-value pair 

                    Trie --> serialize --> Key, value pair

            -- So if we talk in terms of data structures then key-value pair is just HashMap in which there's a key and a value

            -- So here our key will be every prefix in the trie and value will be data in each trie node (top 6 results)

            -- Our workers will be responsible for filling the cache 

            -- Workers will asynchronously work in background to build the new tries and after completion of building the new tries,
                the old trie in DB and cache will be replaced by the new one

        
*** Flow ***

    -- A search query is sent to the LB

    -- The LB route this to a server with less or equal load as other servers

    -- Server tries to fetch data from cache 

            If exists then data will be sent fast to the client

            If it doesn't exist then this data will be fetched from the DB (although whole trie is in cache but just in case cache 
            miss occurs {and other scenario can be when trie is updated in DB but not in cache and user searches for new term that 
            is in trie in DB so then might help} or cache full or cache down)



*** Further Optimizations ***

    -- So whenever client types every letter an API call will be made to the server so here we will not use HTTP requests instead we
        will use AJAX requests (AJAX internally uses HTTP only) {in js terms async function and inside it a fetch()}

        [AJAX is not a different protocol; it is just a way of using HTTP]

    -- So whenever we sends requests using AJAX it will not refresh the web page each time an call was made whereas in standard HTTP
        calls it might refresh the page each time the user types a letter which degrades the UX


    -- To decrease load on our server we can also cache results at browser side (client side)

    -- Offensive / abusive words can also come so we will introduce filters between the API servers and the trie cache so filters 
        just consists of the rules  that filter the data coming from trie cache as if the data that is coming is offensive / 
        abusive words then the server returns empty response or removes those words from the response

    -- We can also scale the DB by sharding so we can use sharding key as the first english character but there might be a problem
        like many words having 'a' as starting character whereas very less have 'x' as starting character which can lead to heavy
        load on one DB

    -- So to solve this problem we can use a shard mapping table in which the rules will be defined like in above case we can just 
        use one DB to store 'x' as starting character data and we can use 2 DB to store words starting with 'a' like 'aa' to 'am' 
        in one DB and 'an' to 'az' in another DB

    -- So to provide multiple language support we can use unicode characters of every language in the world 

    -- And to scale our application more then we can just make different servers for different countries and we can also make a 
        country specific top results 

    -- Lastly we can use CDN (cache) for better response 


            








         