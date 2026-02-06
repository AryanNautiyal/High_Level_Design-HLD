

*** Design a Instagram feed ***

    Q : What are the features that we need to design ?

    Ans : Let's assume interviewer says to create post, follow/unfollow and see his/her own feed so these are the features that
            we need to design 

    
    Q : How many users can we follow ?

    Ans : Like in Facebook we can make maximum 5000 friends so here let's assume interviewer says that unlimited as in Instagram
            we can see that many people can follow you and you can also follow unlimited users


    Q : How many DAU ?

    Ans : Let's assume interviewer says 20 Million


# Back of the Envelope Calculations

    Assumptions :

        -- DAU : 20 Million

        -- So let's assume that user when create post will send 1 write request to the server and when he will see his own feed then
            1 read request will be sent to the server

        -- So let's assume that the user opens or refreshes his own feed 10 times a day and posts 1 picture a day 

        -- Each post let's say is 1 MB on average


    QPS : 

        Read operations per day = 20 Million * 1 * 10 = 200 Million 

        Read operations per sec = 200 Million / 86400 = 2 Million / 864 = (2 * 2^20) / 864 = (2 * 10^6) / 864 = ~ 2314.8 query/sec

        Peak QPS = QPS * 2 = 2314.8 * 2 = ~ 4629.6 query/sec

        Write operations per day = 20 Million * 1 = 20 Million

        Write operations per sec = 20 Million / 86400 = (20 * 2^20) / 86400 = (20 * 10^6) / 86400 = (2 * 10^5) / 864 
        
        Write operations per sec = ~ 231.5 query/sec

        Peak QPS = QPS * 2 = 231.5 * 2 = ~ 463 query/sec

    
    Storage Unit :

        -- Let's calculate it for 5 years

        -- So 20 Million users perform write queries once a day and store their posts in DB which is 1 MB on average

        Storage Unit = 20 Million * 1MB * 1 * 365 * 5 = 20 * 2^20 * 1 * 2^20 * 365 * 5 = 20 * 365 * 5 * 2^40 = 36500 * 2^40

        Storage Unit = 36.5 * 10^3 * 2^40 = 36.5 * 2^10 * 2^40 = 36.5 * 2^50 = 36.5 PB

        So 36.5 PB of storage is required to store all the user data for 5 years


# Requirements 

    -- Functional Requirements

        (1) User can create a post (Image/Video/Reel)

        (2) User can follow another user

        (3) User can see his/her own feed

    -- Non-Functional Requirements 

        (1) Availability is more important in this case than consistency 

        (2) Scalable (It could handle billions of users)


-- So as we have seen in Facebook, Instagram, LinkedIn that user can follow another user so like user 1 can follow user 2 and user 2
    can follow user 3 so for this we use Graph DB as we already know (most commonly used Graph DB is Neo4J)

-- So to represent follow/unfollow feature we use Graph DB as it can clearly show the relations by using users as the vertex and 
    their relationships as the edges

-- So like in Facebook as we have seen that is a user let's say user 1 sends friend request to user 2 and user 2 accepts this 
    request then this automatically means that user 1 is friend of user 2 and user 2 is also friend of user 1 so this is 
    bidirectional relationship which is shown by undirected graphs (User 1 <---> User 2)

-- Whereas in Instagram as we have seen that if user 1 follows user 2 then it doesn't necessarily mean that user 2 also follows 
    user 1 so this relationship is unidirectional which is shown by directed graph (User 1 ---> User 2) 

-- So in Instagram case if user 2 uploads something then it will be shown in user 1 feed and if user 1 uploads something then it 
    won't be shown in user 2 feed

-- Nowadays we have seen in LinkedIn we can use both bidirectional and unidirectional connections like for bidirectional there is 
    connection and for unidirectional there is follow 

    (So we can show unidirectional and bidirectional relationship by directed graph only)


# API Endpoints 

    -- So client will send HTTP request to our Instagram Feed Application server 

    -- So the client will interact with server with these APIs :

            1 -- Posting something (User ----> img/video/reel)

                        POST : /create/post/{userId}

                        -- So user will send this JSON body with the request to create a post

                        Body : {
                            "captions" : "",
                            "content" : "",                             { content can be img/video/reel }
                            "timestamp" : ""
                        }

                        Response : 201 (Resource created successfully)


            2 -- Following a user (Directed Graph)

                *** Idempotent APIs ***

                    -- Dedupe (double request)

                    -- These are APIs that even if double request comes then our API does not do anything wrong 

                    -- Meaning if client sends same request twice (dedupe) then the server treat them as only one API call 

                    -- So this ensures that if our client sends same request multiple time then the server can handle it 

                    {
                        Example :

                                Imagine a user on LeetCode is buying a "Premium Coding Course" for $100.

                                (1) The user clicks "Pay Now."

                                (2) The internet lags. The user thinks, "Did it work?" and clicks "Pay Now" again.

                                (3) Without Idempotency: The user is charged $200 (two separate transactions).

                                (4) With Idempotency: The server recognizes the second click as a duplicate and says, "I already 
                                                        processed this," and does nothing. The user is only charged $100.

                    }

                    (To make Idempotent APIs)

                        -- Whenever client sends HTTP request to the server we just need to add an extra header to the request in 
                            which we give UUID (Universally Unique Identifier) or Idempotent ID (any unique ID we only need which 
                            is unique for each request) 

                        -- UUID is 128 bit number used to uniquely identify information without needing a central authority to 
                            coordinate it. It is often represented as a 36-character string (32 hexadecimal digits and 4 hyphens)

                        -- So we give a unique ID to each different request so like if a user gives POST request and UUID to it was
                            given let's say 1234 then server will process this request so if user again sends the same POST request
                            then the UUID given to it will be same like here 1234 so server will see that the UUID is same as that 
                            of the request that came before so server will just return status 200 (ok) without processing the 
                            request again (UUID is same as the client just sended 2 request just by clicking like post button 2
                            times so no new UUID was generated for it)

                        -- So the server can store UUID of the request in Redis and then work on the request so that if new request 
                            that comes has the same UUID it can just match it in Redis and just return either processing request or 
                            just 200 (ok) without processing the same request

                        {
                            // Modern Browser way

                            [ const idempotencyKey = crypto.randomUUID();  ]

                            // Returns something like: "f47ac10b-58cc-4372-a567-0e02b2c3d479"

                            Like this we can create unique UUID at client side 

                            If an interviewer asks, "What would you use for the Idempotency Key?", give this specific answer:

                                "I would use a client-generated UUID v4. It’s globally unique and decentralized, meaning the 
                                frontend can generate it without talking to the backend. I would pass this in a custom HTTP header 
                                like X-Idempotency-Key."


                            In case the user get same UUID and by coincidence have same request then

                                In your Redis cache, we don't store the UUID alone. We store it as a Composite Key: idempotency_key 
                                = {user_id}:{uuid}

                                    -- User A sends UUID 1234. Redis stores: user_A:1234

                                    -- User B (by a miracle) sends the same UUID 1234. Redis stores: user_B:1234

                                    -- Result: Because the user_id is different, the keys in Redis are different. There is no 
                                                collision. A user can only "collide" with themselves, which is exactly what we want 
                                                (Idempotency!). 

                            
                            The industry standard for storing idempotency keys in Redis is typically 24 to 48 hours.

                            48 hrs mostly used in cases of critical payments where we don't want user to pay twice for a thing he 
                            should pay once only 

                            So our GET, PUT, PATCH, DELETE are by default idempotent but we need to make our POST idempotent

                            -- Like if we send same GET request twice then server will just make 2 DB call to get data and respond 
                                to client (there's no impact on DB)

                            -- If we use PUT twice instead of GET then also server will just make 2 DB call and update data

                            -- If we use DELETE also twice then in 1st the user account will be deleted and then in second when it 
                                will look for user account to delete account it won't find the account

                            -- POST is the only one not idempotent by default as POST always pushes data into the DB so same data 
                                will be inserted in DB 2 times which leads to redundancy in data 


                        }

                -- So when someone follows other user we need to make it idempotent by default so that it won't affect anything when
                    user presses follow button 2-3 times in a short period of time so we can either use PUT or if we are using POST
                    then we need to make it idempotent 

                PUT : /user/{userID}

                Body : {

                    }

                    Response : 200 (ok)

                
                -- So we are using Neo4J for this purpose which is a Graph DB 


            3 -- View our feed

                GET : /fetch/feed/{userID}

                Response : {
                    [
                        "postID" : "",
                        "postName" : "",
                        "content" : "",
                        "timestamp" : ""
                    ],
                    [
                        "postID" : "",
                        "postName" : "",
                        "content" : "",
                        "timestamp" : ""
                    ],
                    [
                        "postID" : "",
                        "postName" : "",
                        "content" : "",
                        "timestamp" : ""
                    ]
                }

                -- So when we try to view our feed what instagram must do is that they see the people we follow take all of their 
                    posts and order them in reverse chronological order and display it 


# High Level Overview 

    1 -- User posting something (img/video/reel)

            -- So whenever user posts something a HTTP request is sent to the Post Service server and then it stores all the 
                metadata like postID, type, timestamp, etc related to the img/video/reel in Post DB

            -- And it will store the actual img/video/reel in a Amazon S3 Bucket 

            -- So whenever user posts a img/video/reel then a call will be made to both the DBs

            -- So for Post DB we can use both SQL or NoSQL but here we choose NoSQL as we can see a key-value pairs so we don't want
                to waste time to setup tables and all that so we use NoSQL

            -- So we can use Amazon Dynamo DB here as it is key-value based store

    
    2 -- User following other users

            -- So there will be Follow Service server which will call Graph DB everytime a user sends follow request 

            -- In this there can be many to many relationships between users and also it can be easily model by graph hence we use
                Graph DB

        
    3 -- User viewing his/her own feed

            -- So whenever we view feed we see posts of all the people we follow arranged in reverse chronological order

            # Flow

                (1) Fetch the user's following list

                (2) Fetch all the new post from this list (metadata)

                (3) Finally sort the list in reverse chronological order 

        
            -- So the Feed Service server will grab user's list from Graph DB 

            -- Then from the user's list will grab all the post IDs of the users (metadata) from Post DB

            -- Then it will post all this data in Feed DB and Feed Cache (will use write back technique)

            -- As our cache is small so stored posts IDs only as img/video/reel are very large 

            -- So now our Feed cache will communicate with the Amazon S3 Bucket to fetch the img/video/reels


*** Problems in our current approach ***

    -- As we already know that in Instagram the read operations are far more than write operations 

    -- So our current design to view the feed is very slow as read operations are far more so hence we need a system that can be
        much faster than this to show feed

    -- So what we can do is that we can just make write operations slower in this but should make read faster as read operations
        are far more in numbers (case specific approach)

    -- So there are 2 types of models :

            (1) Pull Based

                    -- So the design we have made above is the Pull Based model as we have to send fetch request to fetch the feed

                    -- So in this whenever we refresh or pull the feed then only we bring data

                    -- In this read operation is slow 

                    # Pros

                        -- This is better for users that are inactive (as we shouldn't waste resources to calculate feed data for a 
                            user that comes online in a month or so)

                    # Cons

                        -- It is slow 


            (2) Push Based

                    -- In this read operation is fast

                    -- In this write operation is slow 

                    -- In this approach what we will do is that when a user posts a img/video/reel we just fetch all the followers 
                        of the user and put this post in their feed 

                            User --> POST ---> Pre-compute follower's feed cache

                    -- So it is pre-calculation / pre-compute model 

                    # Pros

                        -- Read is faster

                    # Cons

                        -- Bad for inactive user (as will update feed of a user also who is just logging in every year or month)

                        -- Hot-key problem 

                            So let's understand it by taking example of Virat Kohli Insta account which has let's say 250M followers

                            So if Virat Kohli uploads a img/video/reel then this will be put in the feed of 250M people 

                            So we will bring list of 250M people then this will be uploaded to each of their feeds which will result
                            in very slow write operation or in some cases it might fail

                            So this is celebrity problem or hot-key problem 

    
    -- For now let's say we are using push based approach 

    -- So now in push based approach :

                (1) User post something 

                        Flow :

                            1. We will fetch the followers list of the current user

                            2. Then we will add the postID to the feed cache

                            3. We will store a list of pair of <userID, postID> where userID will be of the user for whom the post 
                                is for and postID will the ID of the user which posted the content

                        
                        -- So our Post Service server will call Graph DB to obtain the list of all the followers with which it will
                            post this data (<userID, postID>) in feed cache for users and then will store in feed DB 


                (2) When user see his/her own feed

                    {In this I am seeing my feed meaning all the posts that other users posted for me should be shown to me}

                        -- So in this the Feed Service server calls the Feed cache to get all the postID of the user (in feed cache
                            stored in this format : <userID, postID> in which userID will be that of user posting the post and it's 
                            postID)

                        -- Then from this information it will fetch all the metadata from Post DB about posts and then user 
                            information from Graph DB

                            {Keep data in feed DB for like 30-90 days}


    -- So everything here is pre-computed 


# Problems 

    (1) Both approach (pull & push) has some disadvantages 

        Solution :

                -- We will use hybrid approach (pull & push both)

                -- Every user has a boolean variable which is named isPreComputed so we separated by this logic

                        if user has many followers (like million, thousands, etc) then for them pull base approach will be used

                        if user has less followers then push based approach

                -- So if isPreComputed = 1 then push based approach will be followed and then if isPreComputed = 0 then pull based
                    approach 

                -- So if a user follows many people and all the people that user follows uploaded a img/video/reel then it won't be 
                    possible to load all the content of the people to show to the user so we limit the number of posts that appear 
                    in feed cache to like let's say 200 so only 200 posts will appear in feed cache and if user scrolls down after 
                    seeing all 200 posts then another 200 posts we will fetch (basically if someone with like 100k following or 
                    something posts a img/video/reel his/her posts will only go to 200 people only)

                -- So with the use of pull & push based hybrid approach we solved the follower issue and with the other method we 
                    solved the following method, if following is less for a person then all the posts will be loaded for him but 
                    still max is 200

    (2) Still after using the hybrid approach the process is slow

        Solution :

                -- So if someone has like 500k followers for which it will just use push based approach as let's assume for million
                    followers only the pull based approach is used so it will take 500k users list from Graph DB and then will post 
                    it to their feed cache which will be slow as posts is to be sent to 500k users 


                    {

                        Strategy: Pull.

                        When the celebrity posts, the post goes only to the celebrity's own User Timeline in the Post DB. Nothing is pushed to the 10 million followers' caches.

                        How do followers see it? When a follower (like you) opens your feed, the system does two things:

                            1. Fetches your pre-computed feed from your Feed Cache (your friends' posts).

                            2. Fetches the latest posts from the Celebrities you follow directly from the Post DB.

                            3. Merges them together in real-time.

                        
                        The "Celebrity List" is Cached

                            We don't go to the Graph DB to calculate the celebrity list every time.

                            The Setup: For every user (like Aman), we keep a small list in Redis of just the Celebrity IDs they follow.

                            The Key: user:celebrities:Aman

                            The Value: [Celebrity_ID_1, Celebrity_ID_2, ...]

                            Why this is cheap: Most people follow fewer than 50–100 "celebrities." Fetching a list of 50 IDs from Redis takes less than 2 milliseconds.

                    }


                -- So we will use messaging queue here as the throughput of cache and DB is less so pushing posts data to 500k 
                    users might crash cache or DB due to write intensive load

                -- So we use messaging queue to match the throughput of them and to also make our post service more faster for push 
                    based approach 

                -- So our push operation post service is faster due to messaging queues as now our Post Service Server is not 
                    directly uploading data to cache which was synchronous instead now our Post Service server will just upload 
                    data in messaging queues and will do other work (as messaging queue will handle the rest) so then the workers 
                    just one by one pick data and insert them in the feed cache and then by write back all this is written in feed 
                    DB (so now our API calls are asynchronous)



*** Final Design of the system ***

        -- Final design is provided in the pdf (in design shown the push based approach can draw another design for pull based and 
            use them together)

        -- So when a client uploads img/video/reel it communicates to Post Service server which calls Post DB to store metadata 
            about post and then from Graph DB it grabs the list of users that follows the client then makes <userID, postID> combo 
            and then give it to Kafka (messaging queue) from where one by one the workers take data and store it in feed cache

        -- So the design is of push based approach if a user gets more than some fixed amount of followers then we can just shift 
            them to pull based design

                







                



                        

                        