


*** DB Sharding ***

    -- Here will deeply study about DB sharding, how to do DB sharding, limitations of modulo function for sharding and consistent
        hashing 

    -- So let's say we have billions of user so we scale our DB horizontally for fast responses hence we do sharding 

    -- So we have studied a hash method to implement DB sharding in which we do ' % ' with number of DB we have to store all data
        in DBs and also get data faster

    -- So as discussed before we take a hash function that converts string, integer and all to hash value, we can take any string, 
        integer, etc and find it's hash value then we modulo the hash value to find the DB in which data will be stored of this user

    -- Limitation with this hashing was that when a single DB crashed or something happened then we will be needed to do resharding 
        of the DB as now instead of % 3 (assumption) now one DB down so %2 now which will mean that many data will now be moved 
        from crashed DB to their appropriate DB and all other data will also be moved

    -- Similarly if we introduce one more DB then also it will lead to problems as %3 will change to %4 so imagine if we have 
        billions of data then it will be costly and time taking to reshard the whole DB

    -- So using % as hash function has these limitations as we need to migrate data each time a DB is down or new one is added 

    -- So migration is very costly so we try to avoid it 

    -- So to overcome this limitation consistent hashing was used 


    *** Consistent Hashing ***

        -- So consistent hashing means a hashing that is consistent meaning the migrations will be less when scaling DB

        -- Main problem was due to modulo operator due to which resharding or migration is difficult

        -- So ** Hash Ring ** is introduced in which there are many values like here let's assume there is 1-100

        -- So consistent hashing says that to hash the DBs before hashing the keys so to find hash value of the DBs let's say we 
            use their IP to find the hash value 

        -- So let's assume DB1 comes at 1, DB2 at 25, DB3 at 50, DB4 at 75 

        -- So now our DBs are distributed in a hash ring (hash ring is ring like here 1-100 number ring after 100 again 1 comes)

        -- So first we have introduced Hash Ring in this then we have introduced DB hashing and now we will do key hashing 

        -- So now let's assume some hash values for keys and let's say 4 keys 0, 1, 2, 3 and their respective hash values 11, 41, 
            60 and 80

        -- So now we will locate the keys in hash ring also 

        -- Now our DBs are at 1, 25, 50, 75 and keys are at 11, 41, 60, 80 

        -- Now from each key we will move in clockwise direction until and unless we encounter a DB, when we encounter a DB then we 
            will store the key in that DB

        -- So while moving clockwise key0 will go in DB2, key1 in DB3, key2 in DB4, key3 in DB1 

        -- So like this keys will be assigned to DBs and data will be stored in DBs

        -- So now let's see how consistent hashing overcome the limitation of modulo operator 

        -- So now in above example if we introduce a new DB let's say DB5 then if we find hash of DB5 let's say it comes as 90 

        -- So now we will only be needed to migrate our key3 only to DB5 as for rest of the keys if we move clockwise same DB will 
            come

        -- This greatly reduces the amount of resharding we are needed to do 

        -- So now even if we introduce more DBs let's say we introduce DB6 whose hash value comes as 35 then no keys will be needed 
            to resharded as no key is there that will move clockwise and comes to DB6 so no resharding needed

        -- So now let's say due to some reasons our DB5, DB6 and DB2 are down then we will be needed to reshard our key0 as it was
            stored in DB2 so now when we again move clockwise then we see that it reaches DB3 so it is stored in DB3 and also due 
            to DB5 going down now key3 will be again stored in DB1

        -- We can simply implement this by using circular array and hash functions

        -- Remember : At one number in hash ring only one DB can exist 

        -- Same for data retrieval we find keys hash value and move clockwise to find the DB in which the key is stored

        -- So we assume that the hash collision will be negligible here as hash values will be unqiue as IP is unique 

        -- So now migration is greatly reduced

        -- If hash value of key comes as the same as that of a DB then we will just store that key in that DB and there is no need 
            to move clockwise 

        -- Uneven distribution of keys so we need a way so that the keys will be evenly distributed among all the DBs in the hash 
            ring 

        -- So for this we try to *** minimize the standard deviation ***

        -- In statistics, Standard Deviation measures how much your data points vary from the average. In the context of a Hash 
            Ring, it tells you how much the "load" (the number of keys) on each database varies from the "ideal equal share."

        -- So in cryptography taking hash function we often use prime number as they uniformly distribute data and if we take close 
            to odd or even number then it will try to go more towards factors of 2 

        -- As without distributing evenly we can think of a scenario where all the data is stored in a single DB and rest of the
            DBs are just idle doing nothing so this will be a problem as we are not fully utilizing our consistent hashing technique
            
        -- Like for example all our keys hash are in between 1-25 so all are keys key0, key1, key2 and key3 will all be stored in 
            DB2 increasing load on DB2 whereas rest of the DBs will be empty

        -- So choosing a good hash function is important as in case of modulo operation also if our hash function was bad then data 
            was not evenly distributed

        -- So having good hash function will always help in uniform distribution of data but introducion of virtual nodes is also 
            necessary to make sure that data always distribute uniformly 

        
        ## Virtual Nodes ##

            -- Virtual Nodes are replicas of Original Nodes 

            -- Like in Master Slave Architecture, the master DB has it's replicas called as slave DB so hence the slave DBs are 
                here called as virtual nodes

            -- So if we take above example in which hash ring is of 1-100 numbers and there are 4 DBs DB1 at 1, DB2 at 25, DB3 at 50
                and DB4 at 75 so let's assume each of these DB has 2 replicas 

            -- So each replica will have different IP address or can say different hash value due to IP so now we will also map the
                replicas also in the hash ring

            -- We assume that cases of collision are negligible so distribution will be good in case of DB hash (no two DB will get 
                same hash value)

            -- With introduction of replicas now data will be distributed uniformly as there will be uniform distribution of DBs 
                also

            -- Like let's say in above example replicas are named as DB1_1 and DB1_2 like this so let's say hash value for 
                DB1_1 is 35 , DB1_2 is 65, DB2_1 is 60 , DB2_2 is 85, DB3_1 is 95 , DB3_2 is 10, DB4_1 is 20 , DB4_2 is 40

            -- So now even if our key0, key1, key2, key3 all are in between 1-25 still they will be evenly distributed as they 
                between 1-25 there is DB1, DB3_2, DB4_1 and DB2 so key will be stored in these DB which will be uniform

            -- As all DB 1, 2, 3, 4 are coming in between 1-25 (even a hash value of a key comes to a virtual node then the data is 
                written to all the replicas and original node of that DB like if key0 is stored in DB3_2 then it will also be 
                stored in DB3 and DB3_1 as DB3_1 and DB3_2 are replicas of DB3 only)

            -- So with introduction of virtual nodes we have ensured that data will be distributed evenly among the DBs

            -- There was another limitation that is also overcome by using virtual nodes i.e. *** Celebrity problem (hotspot key 
                problem *** )

            -- So to understand this problem let's take example of instagram so in instagram there are some celebrity people 
                accounts so let's assume key4 : SRK, key5 : Virat Kohli, key6 : Ronaldo, key7 : Messi so with our consistent 
                hashing all these keys are in DB3 or let's say at 50 value in hash ring so all these accounts are of celebrity
                so many people will fetch their account data so can say they are hotspot keys so as all 4 are in DB3 only so 
                load of DB3 will be increased due to many requests

            -- So this is the celebrity problem that occurs so this problem is resolved by virtual nodes also as now with replicas
                also in hash ring the data is distributed uniformly so no single server will have heavy load 

            -- So without virtual nodes there were 2 limitations of consistent hashing : 1. uneven distribution of keys
                                                                                         2. celebrity problem (hotspot key problem)



*** Load Balancing ***

         -- Load balancer distributes traffic to different servers from a server pool (traffic is distributed among all the servers
            that are active)

        # Applications of Load Balancer 

                -- Scalability : Auto Scaling => Helps in scaling servers {horizontal scaling} 

                -- Availability : If a server is down redirect all the requests to another server that is working 

                -- Security : Protects from DOS (Denial Of Service) and DDOS (Distributed Denial Of Service) attacks 

                                (<>) So it protects from DOS and DDOS attacks by distributing requests to various servers to 
                                    prevent overload in a server

                                (<>) It also makes every request go through a firewall to detect if any request is malicious or not

                -- Responsiveness

                                (<>) Response Time : Time taken by server to give response back to the client after processing the 
                                                    request sent by the client (another name for it is TTFB {Time To First Bit})

                                (<>) Server can be called busy when server is processing one request and many other request also 
                                    come (thread pool {to handle multiple requests parallelly concept of thread pool is done})


        ## Algorithms of Load Balancer ##

                -- These are the algorithms by which the load balancer balances the load on each server 

                -- The algorithms can be divided into 2 parts : 

                                        (.) Static Load Balancer

                                        (.) Dynamic Load Balancer 

                        
                        *** Static Load Balancer Algorithms ***

                            -- These Load Balancer have a fixed algorithm with which they work (they do not see load on each 
                                server) 

                                1. Round Robin Algorithm 

                                        -- Similar to the one in Operating System (OS)

                                        -- It doesn't think about whether the load on servers is more or less 

                                        -- It just one by one distribute requests to each server

                                        -- Let's understand this with an example like lets say we have 6 requests r1, r2, r3, r4, r5
                                            and r6 whereas we have 3 servers s1, s2 and s3

                                        -- So in this when r1 comes Load Balancer distributes the request to s1 then when r2 comes 
                                            it distributes it to s2 then when r3 comes it distributes it to s3 then again r4 is 
                                            sent to s1 then r5 to s2 and r6 to s3 



                                                            s1-------
                                                            ^       |
                                                            |       |
                                                            |       |
                                                            |       |
                                                            |       V
                                                            s3<-----s2

                                        
                                        -- Kinda like this distributed to each server one by one

                                        -- First one request is given to s1 then to s2 then to s3 and this cycle is repeated


                                        # Limitations :

                                            (-) Do not consider load on any server 

                                
                                2. Weighted Round Robin Algorithm (Advance Version of Round Robin Algorithm)

                                        -- So in this it assigns weight to the server based on hard disk, RAM, CPU processing 
                                            power, etc

                                        -- So let's say we have 3 servers s1 has weight 1 and s2 has weight 5 and s3 has weight 10

                                        -- So these weights are alloted based on how much request they can handle or computer specs

                                        -- So when request r1 comes it allocates it to s1 then when r2 comes it allocates it to s2
                                            but when r3 comes it sees that s2 has weight 5 so it means that it can handle 4 more 
                                            requests so r3, r4, r5, r6 are all allocated to s2 so r2-r6 are allocated to server s2
                                            then similarly for the s3 requests r7-r16 are allocated to it 

                                        -- So weigths are given like let's say that weight are given according to RAM so now s1 with
                                            weight 1 will have RAM let's say 32 GB then s2 server will have 5 times more RAM than s1
                                            and s3 will have 10 times RAM than s1

                                        -- (Gemini says requests distributed evenly over time they are arranged in a way so one by 
                                            one request is assigned to each server)

                                        -- After r16 this is again repeated (can say according to capacity the requests are given)


                                        # Limitations :

                                            (-) Do not consider load on any server (like if r3 and r4 were heavy requests then load
                                                balancer won't consider anything will just give more request to server until weight
                                                or capacity reached)    [Does not manage load on server]


                                3. IP Hashing 

                                        -- So when requests comes it hash the IP address of the client and then based on hash value 
                                            it assigns a server to the request 

                                        
                                        # Limitation :

                                            (-) Each user will always get same server (tightly coupled to the server {stateful 
                                                architecture})

                                            (-) Proxy (VPN) : In companies for protection each request sent is from same IP so all
                                                              the requests will be coming from same IP to the same server only so
                                                              server will be overloaded 
                        

                        *** Dynamic Load Balancer Algorithms ***

                            -- These Load Balancer Algorithms focus on load on a server (or can say they also consider load on 
                                server)



                                1. Least Connection Method

                                        -- Connection : So when a client sends HTTP request to the server and server processes that
                                                        request till the time server processes the request a connection with the 
                                                        client is maintained

                                        -- In this it allocates requests according to the number of connections made by each server,
                                            it allocates request to the server that has less number of connection

                                        -- Let's assume s1 has 2 connections, s2 has 4 connections and s3 has 6 connections so when
                                            r1 comes it is allocated to server s1 so now s1 has 3 connections similarly for r2 but 
                                            when r3 comes then both s1 and s2 have same number of connections so it allocates to 
                                            the first one then this is repeated

                                        -- Static LB algos are better used when all servers have same capability 

                                        -- (Better than Round Robin because it naturally stops sending requests to a server that is 
                                            "struggling" and keeping connections open for a long time {Source : Gemini})


                                2. Weighted Least Connection Method

                                        -- So in this also weigths are assigned to each server

                                        -- It focuses on both number of connections and weight of each server by taking ratio of it

                                        -- Whichever server has least number of active connection / weight ratio is allocated the 
                                            request

                                        -- Let's assume s1, s2 and s3 have weights 1, 5 and 10 respectively so now when r1 comes it 
                                            calculates let's say s1, s2 and s3 have number of connections 3, 5 and 4 respectively 
                                            So for s1 : 3/1 = 3
                                               for s2 : 5/5 = 1
                                               for s3 : 4/10 = 0.4

                                            So s3 has the least number of connection / weight ratio so hence r1 is allocated to 
                                            server s3 

                                            So now connection of s3 are 5 now 

                                        -- Can use this if servers have different powers or capacity (as allocating weight)

                                
                                3. Least Response Time Method

                                        -- So in this response time of each server is seen

                                        -- Server which has the lowest response time * number of active connection is allocated the 
                                            request

                                        -- Let's say our server s1, s2 and s3 has response time 10ms, 11ms and 15ms respectively 
                                            and 3, 4 and 5 active connections respectively so when our request r1 comes we see 
                                            number of active connection * response time for it 
                                            So for s1 = 10 * 3 = 30
                                               for s2 = 11 * 4 = 44
                                               for s3 = 15 * 5 = 75

                                            So hence our server s1 has least number of active connection * response time so hence 
                                            r1 is allocated to it 

                                            So now s1 has 4 active connection and due to 4 request at server s1 it's response time 
                                            might increase from 10ms to 12ms

                                        -- So response time and number of active connection varies after each request is allocated

                                        -- Response time is average response time for request here (but we can take response 
                                            time for each request also but will be complex)

                                    
                                4. Resources Method 

                                        -- So in above methods our Load Balancer does all the calculations but in this agents are 
                                            there (used in modern applications)

                                        -- Agents are nothing but programs that run on each server in background to perform health 
                                            checks on the server

                                        -- So it just sends empty HTTP request to check if server responses 200 (ok) or not

                                        -- The agents helps to determine which servers are up and which servers are down

                                        -- Agents also helps to determine the capacity or load on the server at a particular moment

                                        -- So when request comes the Load Balancer asks the agents of each server about the health 
                                            of the server

                                        -- Agents tells the load balancer whether the server is up or not

                                        -- If not server is not up agent tells Load Balancer to not send requests on this server
                                            and when the server is up it tells the LB to send request here

                                        -- Capacity/Load can depend on many factors :
                                                            => Number of Active Connections 
                                                            => Response Time
                                                            => Latency 
                                                            => Health Check


        # Types of Load Balancers

            -- So we have discussed the layers of ISO_OSI model so in HLD we have said we will focus on application layer, network
                layer and transport layer

            -- Load Balancers can be divided into 4 types : 

                        => Application Level Load Balancer  

                                    -- It does Load Balancing at application level 

                                    -- It load balances the HTTP calls and HTTPS calls 

                                    -- It doesn't know anyone's IP as IP comes at the time of Network Layer


                        => Network Level Load Balancer

                                    -- It does Load Balancing at network level

                                    -- It load balances the TCP and UDP calls

                                    -- It knows IP addresses so distributes traffic based on IP addresses

                        => Global Server Load Balancing 

                                    -- Geo Enabled Load Balancer

                                    -- It is also called as Geo DNS (Geo Domain Name System)

                                    -- Its job is to distribute traffic across different Geographical Regions

                                    -- Unlike simple Geo-DNS, GSLB is smart enough to know if a whole data center is down and will 
                                        reroute traffic to the next closest region

                        => DNS Load Balancer 

                                    -- So in our application we can have sub domains

                                    -- Like for example we have 3 sub domains www.example.com/home, www.example.com/contact and 
                                        www.example.com/email so we have these 3 sub domains that handle separate work in an 
                                        application

                                    -- So when our request hits www.example.com/home it will go to home one load balancer, if it 
                                        hits www.example.com/contact then it will go to contact load balancer

                                    -- Here our /home, /contact and /email do not consider them as paths like in react routes they 
                                        are hosted in other servers

                                    
        # Types of Load Balancers

            -- It is of 2 types :

                        => Hardware Load Balancer 

                                    -- These are hardware devices that are used to balance load on servers

                                    -- They cost money upfront but they are used by very large applications where software LBs won't
                                        work that good

                                    -- They are LBs that are installed in data centers to distribute traffic among our servers

                                    -- They can easily distribute millions of request with ease

                                    -- If more requests are coming than what the hardware can handle then they are vertically or 
                                        horizontally scaled


                        => Software Load Balancer

                                    -- In this comes our Cloud Based Load Balancer (example of this is AWS ELB {Amazon Elastic Load 
                                        Balancer}) [AWS ELB provides serverless architecture]

                                    -- It costs money to use these Load Balancer per month 

                                    -- It handles all the load balancing by itself
                                    
                                    -- It is limited by the hardware of the server it is installed on 

                                    -- Small and medium applications use this 

                                    



                                        

                                            

                                            





                                

                                

        