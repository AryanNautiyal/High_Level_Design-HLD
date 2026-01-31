

*** Kafka : Continued ***

    -- So now we come to the producer side and understand the process how the producer decides in which partition the message 
        should go in a topic 

    -- So whenever a producer wants to send a message to the queue it sends 4 things which are the following :

                (.) Key

                (.) Value

                (.) Topic

                (.) Partition 

    -- So value is the actual message that the consumer consumes 

    -- Key is the unique identifier (unique ID which can be hashed)

    -- Topic specifies the topic in which the message will go to

    -- Partition specifies the partition in which the message will go to 

    -- Our key and partition are optional in kafka so even if we send value and topic only then it will also work

    -- So without sending key and partition the broker can itself decide in which partition will this message will go to

    -- So broker does the following thing to find out the partition in which the message will go to 

                    ==> It first hashes the key and then finds the modulo of it 

                                hash % (number of partitions in that topic)

                    ==> So if we have like 3 partitions p0, p1 and p2 then the % 3 will always give values as 0, 1 and 2 so 0 means
                        store in p0, 1 means store in p1 and 2 means store in p2

    -- In this first priority is given to partition that is given by the producer and second priority is for key so that it hashes
        the key and % number of partition to find the partition

    -- But as we have said that key and partition are also optional so now it will determine the partition in which the message 
        will be stored by using Round Robin Algorithm by one by one distributing message to each partition like message m1 to p0 
        then m2 to p1 then m3 to p2 then again m4 to p0 like this


    *** Dead Letter Queue ***

            -- So all the messages that due to some bug or error are not consumed by the consumer are kept in dead letter queue

            -- Then messages from this queue are sent back to the partition then consumed by consumer 

            -- A retry is fixed so that after the retry limit reached and message is still not conusmed by consumer then the message
                is dropped like retry is fixed to 3 times so each message will be sent back to partition 3 times if the 4th time it
                comes back to dead letter queue then that message is dropped 

            -- Hence with this retry mechanism is also implemented so that no message is lost 


*** Rabbit MQ ***

    -- So in this also we have a producer and a consumer and there is 2 things now here exchange and there are queues (which were 
        called as partitions in kafka)

    -- So in this producer gives message to the exchange and exchange decides in which queue the data will be sent to 

    -- No consumer group concept is here and consumer can read from multiple queues

    -- Offset and all is same, only different thing is exchange concept 

    -- So like how in kafka key, value, topic and partition where given by producer or found by broker (key and partition not given)
        so similarly here this work is done by exchange

    -- Exchange can be thought of as some rules that decide the queue in which the message will go 

    -- There are 3 types of exchange :


                    (i) Fan-out

                            ==> Fan-out says to broadcast the message sent by the producer
                            
                            ==> So in this when producer produces the message and send it to exchange the exchange will just send 
                                this message to each queue (kind of like broadcasting the message)


                    (ii) Direct Exchange

                            ==> In this it maps the message key to the routing key

                            ==> So let's assume that exchange has routing key as 123qwe so if any message with key 123qwe comes
                                it will just send it to queue 0 similar if any message has key 345rty then those messages will be 
                                sent to the queue 1 and like this messages are mapped to the queues

                            ==> So in this manner by matching routing key and message key the messages are sent to dedicated queues

                    (iii) Topic Exchange

                            ==> Topic exchange is similar to direct exchange but in this wildcards are allowed 

                            ==> So in this let's assume the example in direct exchange only so in this instead of whole key matching
                                it will have like key 123* meaning that any key that starts with 123 will be sent to queue 0 then 
                                345* meaning any message key with starting 345 will go to queue 1 so if a message comes with key 
                                123rty it will be sent to queue 0


    -- In this there are 2 types of keys :

                (.) Message Keys

                    -- It is the key that comes with the message from producer (each message is allocated a key and then sent to 
                        exchange)


                (.) Routing Keys

                    -- It is the key that is given to the exchange (it is a set of keys that exchange has)


    -- In this also there can be a dead letter queue or a re-queue mechanism

    -- So we already know about dead letter queue so it can either put the message in dead letter queue or can implement re-queue 
        mechanism in which either the consumer or the dead letter queue puts the message back in the queue to get it consumed by 
        consumer again


*** Difference between Kafka and RabbitMQ ***

    Kafka uses Pull based technique (in which the consumer polls meaning asks kafka again and again if a message has come and if a 
        message is there then it consumes that message) 

    RabbitMQ uses Push based technique (in which the queue in RabbitMQ itself pushes the message to the consumer or tells the 
        consumer that a message is there)

    Kafka has 2 types of polling :

        (=) Short Polling

                -- Short polling is the simplest form of pulling. The consumer sends a request to the Kafka broker, and the broker 
                    responds immediately.

                -- If there is data: The broker sends the messages, and the consumer processes them.

                -- If there is NO data: The broker returns an empty response immediately.

                -- The Problem: If the topic is quiet, the consumer ends up in a "tight loop," constantly asking for data and 
                                getting nothing back. This wastes CPU cycles and creates unnecessary network noise.

        (=) Long Polling 

                -- Kafka actually uses a sophisticated version of Long Polling to stay efficient. Instead of an immediate empty 
                    response, the broker "holds" the request open for a specific amount of time.

                -- When a Kafka consumer calls poll(), it includes two important parameters:

                            (.) max_wait_ms: "If you have no data, keep this request open for X milliseconds."

                            (.) min_bytes: "Don't bother replying until you have at least X amount of data accumulated."

                -- How it works in reality:

                        1. The consumer asks for data.

                        2. If the partition is empty, the broker waits.

                        3. The moment a message arrives (or the timer runs out), the broker sends the response.


*** Caching ***

    -- So we introduce cache between server and DB to give server data faster than DB (data given from cache is frequently accessed 
        data)

    -- As time taken to get data from DB will be more than getting data from cache 

    -- So as we know cache stores data temporarily so each data in cache has TTL (time to live)

    -- Cache is expensive as it's fast and has small storage so we need to use cache efficiently 

        [ CDN also uses type of caching ]

        [ Load Balancer also uses caching techniques (meaning load balancer can also cache some web pages) ]

        [ Server side caching like Redis ]

        [ Proxies can also use caching ]
    
    -- Generally when we talk about caching in our real world application we are talking about distributed caching as we have many
        servers distributed across the network so we need distributed caching technique also as our cache will also be distributed 
        across the network so we will be needed to ensure data consistency

    -- Two main reasons why local caching fails :

            (-) The Inconsistency Problem : If Server A updates a user's profile and saves it in its local cache, Server B won't 
                know about it. A user might refresh their page and see their old data (if the request hits Server B) and then see 
                new data (if it hits Server A). This "flickering" is a terrible user experience.
            
            (-) Wasted Resources : If you have 10 servers, each one is caching the same "Top 10 Cars" list. You are using $10 
                \times$ the memory for the exact same data.

    -- Internally our distributed caching uses consistent hashing 
    

*** Algorithms for reading and writing data in Cache ***


# Reading Algorithm

    1. Cache Aside technique (for retrieving data from cache)

        -- So in this the server first asks cache for the data if the data is found in cache then cache HIT but in case of cache 
            MISS the data is then retrieved from DB and then this data is first put in cache and then sended to server

        -- So if we say it in steps :

                (1) Check cache for data

                (2) If data found in cache then return the data to server (cache HIT) 

                (3) If cache MISS then fetch the data from the DB

                (4) Put the data fetched from DB in cache and then return the data to the server

        # Pros

            -- Simple to implement

            -- DB document structure and cache document structure can be different (like empID in DB can be string but in cache we 
                can make it integer so can keep different data type or can change the whole document structure itself) [data type
                can be changed but all values in string must be numbers then only can convert]

        # Cons 

            -- For every new data there will be always cache MISS (as when we start our computer the cache is empty so there will
                be many misses)

                (+) To overcome this con we pre-heat the cache meaning we put data in cache that we know user will always try to 
                    fetch or can say data that is used in high frequency (it depends from application to application that which 
                    data to pre-heat) 

    2. Read Through Cache 

        -- In this server first asks cache for the data if the data is found in cache then the cache returns the data, if data is 
            not found in cache then cache itself retrieves data from DB and then returns it to server (in before one server was 
            asking DB for data)

        -- So if we say it in steps :

                (1) Check cache for data

                (2) If data found in cache then return the data to the server (cache HIT)

                (3) If cache MISS then cache internal library will automatically fetch the data from DB, put the data in cache and
                    then return it to the server

        # Pros

            -- Client don't need to bother about DB logic 

        # Cons

            -- Cache document structure and DB document structure should be same (as then only cache can fetch data from DB)

            -- For every new data there will be always cache MISS (as when we start our computer the cache is empty so there will
                be many misses) [same as above]


# Writing Algorithm

    1. Write Through Cache

        -- In this whenever we need to write data in DB we will write data in cache and DB simultaneously so that both of them are
            always consistent 

        # Pros

            -- Consistency will be high

        # Cons

            -- It is slow

            -- Uses 2 phase commit 

    2. Write Back/Behind Cache

        -- In this whenever we need to write data in DB first we will write data in cache and then it will return a message to the 
            server that data is successfully written in DB and then an asynchronous request will be sent to DB to write data in DB

        -- So in simple words write into cache and asynchronous write to DB

        # Pros

            -- It is fast

        # Cons

            -- Inconsistency can come (as data in cache and DB are different as first written in cache but for DB asynchronous 
                request is sent for write operation of data)

            -- Data loss risk if the cache crashes before updating the database

    3. Write Around Cache 

        -- In this whenever we need to write data in DB we will directly write data in DB and invalidate the data in cache so that
            whenever user asks for that data the cache will see that data is invalid and will fetch updated data from DB 

        -- So in simple words do not write data in cache, just invalidate the data in cache (mark it as dirty)

        -- So like it will keep an extra bit just to tell that whether the data is dirty or not so that if data is dirty then data
            from DB will be brought

        # Pros

            -- Without using any 2 phase locking it can maintain consistency


*** Cache Eviction Policies ***

    -- So as cache has limited capacity so we need to determine which data will be removed from cache and new data will come in that
        place in cache when cache is full

    -- So the data that is to be removed from the cache is decided by Cache Eviction Policies


    1. Least Recently Used (LRU)

        -- In this cache eviction algorithm the data which is least recently used is removed from the cache when cache is full

        -- So if we take 4 data in cache d1, d2, d3 and d4 and cache is full so if new data d5 comes then d1 will be removed but
            if after d4 the d1 data is again accessed so d2 will be removed and d5 will be put there

        -- LRU evict the item that hasn't been accessed in the longest time when the cache is full

        # Pros

            -- Simplicity

        # Cons

            -- Assume past record patterns will repeat in future 

    2. Most Recently Used (MRU)

        -- It is the reverse of LRU

        -- In this the most recently used data is evicted when cache is full

        -- So if we take 4 data in cache d1, d2, d3, d4 and cache is full so if new data d5 comes then d4 will be removed but if 
            after d4 the d1 data is again accessed so d1 will be removed and d5 will be put there

        # Cons 

            -- Do not bother about past records 

    3. Least Frequently Used (LFU)

        -- In this cache eviction algorithm the data which is least frequently used is removed from the cache when cache is full (
            in this frequency means number of times that data is accessed)

        -- So if we take 4 data in cache d1, d2, d3 and d4 and cache is full so each data will have frequency 1 as all were just 
            now accessed and inserted into cache so if new data d5 comes then it will see frequency of each data and will remove 
            the one with lowest frequency but as in this all have frequency 1 so it will remove d1 which came first but if after d4 
            the d1 data is again accessed then frequency of d1 is 2 so from d2, d3, d4 one will be chosen to get removed as they all have same frequency which is least so d2 will be removed and d5 will be put there as d2 came before d3 and d4

    4. First In First Out (FIFO)

        -- In this cache eviction algorithm the data which is first in cache is removed from the cache when cache is full

        -- So if we take 4 data in cache d1, d2, d3, d4 and cache is full so if new data d5 comes then d1 will be removed even if 
            after d4 the d1 data is again accessed still d1 will be removed and d5 will be put there as d1 was the first to come and
            will be first to go out of cache

    5. Random 

        -- In this cache eviction algorithm it randomly picks any data to be removed from the cache and inserts new data when cache
            is full 


    -- In good applications FIFO almost works at same level as that of LRU and LFU (MRU is the worst)
            


