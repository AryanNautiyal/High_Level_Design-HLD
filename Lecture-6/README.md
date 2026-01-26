

*** Designing a URL Shortener (Like Bit.Ly) ***

    -- So URL shortener is nothing but we just give a long url as an input then it returns a very short url

    -- So when we will click on this short url we will just go to the website that long url opened


    Long URL ------------> Bit.Ly Clone ------------------> Short URL


    So this is how it will work if we see from above


    So when we click on short URL

    Short URL ------------------> Browser ------------------> Long URL 


    Q : What is the rate of service or how many people we are targeting ?

    Ans : So let's assume interviewer says that we want it to short 20 million URLs in a day (might ask for more but we are 
            assuming this here)

    
    Q : Ask interviewer for an example 

    Ans : Let's assume interviewer gives an example 

                    www.example.com/api/v1/userId=1&userName=Aditya

                                    |
                                    |
                                    |
                                    V

                        www.bit.ly/aQw12


    Q : How small the short URL should be?

    Ans : Interviewer can say any number but our best answer is to make it as short as possible 


    Q : What characters I can use to create short URL?

    Ans : Let's assume interviewer says use 0-9, a-z, A-Z

    ## Flow ##

        1. User comes to our platform and gives us a long URL 

        2. We will give him back a short URL

        3. User put this short URL in browser and is redirected to the long URL

    
    ## Tasks ##

        1. Generate Short URL

        2. User redirection

    
    Back of the envelope calculations 

        -- QPS

            Write operations

                -- 20 Million URLs are generated per day

                -- For each URL generation 1 query will be done to DB

                So 20 Million write operations are performed in DB in 1 day

                Write Operations : 20 Million / day

                Write operations /sec = 20 Million / (24 * 60 * 60) = 20 * 2^20 / 86400 

                Write operations / sec = 0.0002314814815 * 2 ^ 20 = 231.48 * 10^(-6) * 2^20

                Write operations / sec = 231.48 * 2^(-20) * 2^20 = ~ 231.48 operations/sec


            Read Operations :

                -- So our application will be read intensive as for every short url generated the user will atleast visit it 1 time

                -- So we assume a ratio that 1 : 10 meaning for every 1 write operation there will be 10 read operations 

                -- Or can say that our read operations will be 10 times more as that of write operations 

                Read operations / sec = 231.48 * 10 = ~ 2314.8 operations/sec

            
        -- Storage Unit 

            -- Assume for 5 years 

            20 Million URLs are generated daily 

            So 20 * 2^20 * 365 * 5 = 36500 * 2^20

            36500 records are stored in 5 years

            36500 * 2^20 = 36.5 * 10^3 * 2^20 = 36.5 * 2^30 = 36.5 Billions records are stored in 5 yrs

            Let's assume 1 record takes 100 bytes of memory (assuming 1 char = 1 byte so 100 bytes stored for each URL)

            1 record = 100 bytes

            36.5 Billion * 100 = 36.5 * 2^30 * 0.1 * 10^3 = 3.65 * 2^40 = 3.65 TB

            ~ 3.65 TB of memory is required to store records for 5 yrs for an application that stores 20 Million urls in a day 


    Now HLD part will be made

        -- So there will be one client and one server and they will interact with each other with API end points

        -- So 

                Task 1 : Generate a Short URL

                        POST : www.bit.ly/api/v1/generate

                        So we can either give link in path or through JSON body or it can give in query parameter like longURL : 
                        value can say object

                        Query parameters are made of Key-Value Pairs (key=value)
                        
                        The Separator (?): Marks the start of the query string

                        The Assignment (=): Connects the key to its value
                        
                        The Multiplier (&): Separates different pairs.

                        In JSON it will be like 

                        Body : {
                            longURL : value
                        }

                        So mentioning the long URL in the path itself makes the URL long and browser has a specific capacity for 
                        URLs

                        So we choose body JSON can also like ask interviewer which method to take 

                            So Generate Short URL

                                POST : www.bit.ly/aoi/v1/generate

                                Body {longURL}

                                Return : short URL

                Task 2 : Redirect to long URL

                        GET : www.bit.ly/{{shortURL}}

                        Response : Redirect to Long URL

    How to Redirect ?

        www.bit.ly/QW123

        So when short URL is given it will first bring user to our website so then we will just ask DB if they have long URL for the
        given short URL

        If DB has it will redirect the user to long URL 

        We have 2 status code here for redirection 

        301 : Redirection ==> 301 is of permanent redirection 

        302 : Redirection ==> 302 is of temporary redirection 

        So whenever we give status code 301 to browser the browser understands that it's permanent redirection so it will store this in it's cache so that everytime user enters short URL is redirects to long URL

        Whereas in status code 302 it won't save in cache as it's temporary redirection

    So how do we decide which one to use ?

        With status code 301 the user won't come to our server but this is only good in cases if our server is small or we want 
        minimum DB query as our application is small so we want to save cost

        But disadvantage of 301 code is that we won't be able to determine how much the user uses our website 

        With 302 we will know that how many users are using our website (user Metrics {we need this for calculations and to show 
        data also like how many users visited and how many users are using it daily and to find peak hours and where users get 
        stuck and all that we need to determine so we use metrics}) and all that  

        But it is only used if our application can handle many users i.e server is scalable and DB read query is optimized

        So let's assume we asked interviewer this so he said to use 302 for our URL shortener

    
    Design an Initial Flow 

        1. URL shortening (POST)

        2. URL Redirect (GET)


    1. URL Shortening 

            -- So Long URL -------> Mapped To ------------> Short URL 
            
            -- So first thing that comes to mind is Hash Map/ Hash Table 

            -- We will use short URL as the key and long URL as the value

            -- How to generate short URL from Long URL 

                So we have different hash functions to convert long url to short url

                So what hash function does is that it uses hash function to short the long url 

                So which hash function should we use SHA-1, SHA-256, MD5 so short url obtained from this will be unique as we 
                assume that hash function will always generate unique hash values 

                So we will store this in hashmap where our key will be short url and value will be long url


        Flow for URL Shortening : 

            -- Users gives us a long URL

            -- We generate a small URL for the same using hash function

            -- Store the short URL as key and long URL as value in our hashmap

            -- Return the short URL


    2. URL Redirect 

        Flow for URL Redirect : 

            -- User comes with a short URL 

            -- We find the equivalent long URL by doing map.get(shortURL)

            -- Redirect the user to long URL (using 302 redirect)

    
    -- As Hashmap is just a program type and it's only inside our application or can say inside server only so it's not like a 
        separate entity like a DB 

    -- So we will use a DB and as we have heard in Lecture-2 about Key-Value Based Store DB in NoSQL like Amazon Dynamo DB

    -- As if our RAM is restarted or can say our application is restarted so our data will be gone so we need a DB

    -- HashMap equivalent DB is Amazon Dynamo DB


    In HLD we have multiple servers and multiple DB servers so DB replication will be there and there will also be scaling 

    So in our DB we call horizontal scaling as Sharding 

    So we will tell interviewer that when our application will go big we will scale servers and scale our DBs and replicate our DB

    So our DB will be horizontally scaled so we will use consistent hashing 

    So now we have 2 hash functions :

            One for finding out shards in hash rings (like for example we use SHA-1)

            Other hash function will be used to get short URL from long URL (can also use SHA-1 here or just use any)

    
    ** Better Design **

        Moves from NoSQL DB -----------> SQL DB

        Why ??

            -- Let's assume a user gives the same long URL for which we have already saved the short URL in the DB as it was 
                calculated before

            -- So we cannot query on long URL as in Dynamo DB we can query on keys only to search in O(1) but to search with value
                it will be a O(n) process which will be very bad for 20 Million URL per day 

            -- So to overcome this problem to not calculate hash again and again for same long URL we will have to make another copy
                of Dynamo DB with long URL as key and short URL as value and we will also be needed to keep them consistent 

            -- So it is very bad to keep copy of records that are generated 20 Million per day and keeping them consistent also is 
                more work

            -- As for first we will have to query to find if short URL exists for the corresponding long URL provided by the user

            -- Then if not present then we will have to query in other one to write the short URL for it after calculating hash 
                value for URL and also DB 

            -- So for this reason it's better for us to shift to SQL DB 

            -- Another reason for shifting to SQL DB is that we can perform complex queries on that to get user metrics and all that

            -- So if in SQL we can find how many URLs are generated till now for a particular user 

            -- Third reason is that write operations in SQL DB are faster than NoSQL DB (creates little difference) {Careful here
                NoSQL DB have fast write operations rather than SQL DB}

            -- In our SQL table we will have attributes like ID (Primary Key), Short URL and Long URL so we can use indexing in this
                to do queries in this 

        
        Hash functions ??

            -- Converts Long URL to Short URL like SHA-1, SHA-256, MD5, etc

            -- Hash value length should be as short as possible

            -- Possible symbols : 0-9, a-z, A-Z

                Total = 10 + 26 + 26 = 62 characters 

            -- So if we see that the hash algorithms mentioned like SHA-1, SHA-256, MD5, etc they generate very large hash value 

            -- So it was like if somebody gives short URL to short it more it would give long URL 

            -- So we need to keep it as small as possible 

            -- So how small is the question so like we will use permutations 

                    So if we take only one character then how many short URLs are possible 

                        So 62 Short URLs will be possible 

                    So if we take just 2 characters then we can make 62^2 short URLs

                    If we take 3 then 62^3

                    So if we take n characters then we can make 62^n short URLs

                    But if we see our back of the envelope calculations in storage unit calculations we need to store 36.5 Billion
                    records in our DB

                    So we need to take n in such a way so that 62^n >= 36.5 Billion

                    So 62^6 >= 36.5 Billion

                    As 62^6 will give 56 Billion short URL

                    So at max now our link can have 6 characters 

                        www.bit.ly/_ _ _ _ _ _

                    So this is our smallest possible link we can make as if we take below this then number of short URL we can 
                    generate will be smaller than the records we will store in 5 yrs

                    n = 6 now 

                    So SHA-1, MD5 and SHA-256 were generating very large values so we need to reduce the size of their values 

                    So what we can do is we can just take first 6 values from the hash value generated by these algorithms or can
                    take last 6 values also 

                    But this method is not good as our chances of collision will increase as hashing using existing algorithms will
                    lead to too much collisions 

                    So we will be needed to use hashing + collision removal technique 

                    So now what we will do is that we will generate short URL from hashing algorithms then we will check if this
                    URL already exists in the DB or not 

                    If it exists then we will add some temp string to the long URL and again use hashing algorithm to generate
                    small URL from it and we will do it recursively until we get a URL that doesn't exist in DB 

                    Then if it does not exists in DB then we will simply just save it in DB

                    Disadvantage :

                        -- Our DB calls have increased as for just generating short URL we see DB everytime and there are 20 
                            Million short URLs generated per day 

                        -- It will be slow as each time DB call is done and DB is slow 

                    So we use BASE X approach 

                        Converting Base 10 ------> Base 2

                        Decimal        Binary

                        2 ------------> 10

                        3 ------------> 11

                    So as there are 62 possible characters we will use 

                        Number (Base 10) ----------> Base 62

                    So our Base 62 will be like this 

                        Decimal             Base 62

                        0           -       0
                        1           -       1
                        2           -       2
                        3           -       3
                        4           -       4
                        5           -       5
                        6           -       6
                        7           -       7
                        8           -       8
                        9           -       9
                        a           -       10
                        |                   |
                        |                   |
                        z           -       35
                        A           -       36
                        |                   |
                        |                   |
                        Z           -       61

                    So to convert 14320 into Base 62 

                            62   |    14320    |    60    ( 14320 / 62 = 60 and 14320 % 62 = 60 ) {Like we do to convert to base 2}
                            62   |    230      |    44
                            62   |    3        |    3
                                      0

                    So 14320 ---------> 03(44)(60) = 03IY = 3IY

                    So now we will use ID in our SQL DB and do it's base 62 calculations to get the short URL

                    So like if ID is 14320 then short URL will be 3IY for the given long URL

                    Advantage of this approach :

                            -- Collision is impossible (as ID will be unique everytime)

                    Disadvantage of this approach :

                            -- Need a unique ID generator 

                            -- URL length is not fixed 

                            -- It is easy to figure out the next generated short URL (if ID is incremented by 1)    [like if user 
                                14321 has short URL then it will be 3IZ]


                    So we can easily make unique ID generator and the URL length will be less than 6 only (will tell reason later)

                    And to overcome the problem of predictive nature of next short URL we can just use random ID to make it 
                    randomized

                    So it's disadvantage can be easily overcome so we can ignore them for now but it's advantage is very useful for
                    us as no collision will mean that each short URL will be different for each long URL

        So now our current flow is like this

                User : https://www.youtube.com

                            |
                            |
                            |           POST
                            |
                            |
                            V

                    Unique ID : 14320
                    Short URL : Base62(14320) = 3IY

                    Stored it in DB ID, Short URL, Long URL


                So why we need unique ID generator and cannot use SQL default one 

                So as our application will be scaled so we will have distributed system so there will be multiple DB Shards

                So shards can give same ID as each will be different so we need a unique ID generator so that each link will be 
                unique (each shard will generate their own IDs)

                So we will use a centralized ID generator 


                Q : Why our Base62 will only generate 6 digit max and not go over it

                Ans : So we can only store 36.5 Billion unique records in our DB so the unique ID given to the very last long URL
                        will be 36.5 Billion so base 62 of it will be in 6 digits only as in 6 digits we can store only 56 Billion 
                        records

                

                36500000000         26     
                588709677           23
                9495317             17
                153150              10
                2470                52
                39                  39
                0

                So our 36.5 Billion record will be this in base62

                    0(39)(52)(10)(17)(23)(26) = 0DSahnq = DSahnq

                    So this will be the short URL for 36.5 Billion record

                        









     
            







