


*** NoSQL DB (Amazon Dynamo Db & Cassandra DB & MongoDB) ***

    -- SQL is a relational DB and follows ACID 

    -- So SQL DB helps in attaining highest form of consistency

    -- So NoSQL DB was made to store non-relational data

        {

            Personal Note :

                In a Relational database (SQL), "Relational" doesn't mean "organized." It means that data is split into different tables that relate to each other via Keys (Foreign Keys).

                SQL Approach: You have a Users table and a separate Sessions table. You use a JOIN to connect them.

                NoSQL Approach: You store that entire JSON blob as a single unit. It doesn't "relate" to anything else to be understood; it is self-contained.

        }

    -- So in NoSQL DB we store data in key-value pairs 

    -- It satisfies BASE (Basically Available, Soft state and Eventual consistent) property

    -- As NoSQL means Not only SQL so we can just make NoSQL DB to achieve the highest form of consistency

    -- So data is stored in key-value pair in NoSQL DB in which key can be string (plain string or hash string) and in value 
        any datatype can come (like string, object, list, etc)

    -- So mostly used NoSQL DB are :
    
                -- Amazon Dynamo DB (Key-value Based Store)

                -- Cassandra DB (Column Based Store)

                -- MongoDB (Document Based Store)


# Creating our own Key-value Store

    -- So if we think in single server architecture the client will sends request to server to fetch the data

    -- We cannot store all the data in server as RAM has a limited capacity so we need a DB tier to fetch data from so we need to
        implement the hash map that we were implementing on the server on the DB so we can search fast using key in key-value
        based store

    -- So server calls the DB to get this data and then DB returns the data to the server and the server returns the data to the 
        user

    -- So this is our single server architecture but in real world applications this is not preferable as single server 
        architecture is not scalable 

    -- We will be needed to scale the server and also the DBs in the real world applications so whenever we say horizontal scaling
        in DB we mean sharding 

    -- So as we have seen before in " Lecture-4 " about how we started from modulo operator for deciding which data goes to which 
        DB shard

    -- So we discussed how modulo approach had problems so then we moved to consistent hashing 

    -- So now in our key-value store these methods will be there :

            (1) POST

            (2) GET


    -- So we need something to put the data in the DB so we will use POST, like we need a method like put(key, value) to store data
        in our key-value store 

    -- Similarly we need a GET method also using which we can easily get data from DB like get(key) which gets the value

    -- So our put(key, value) method should insert key and value in DB and our get(key) method should retrieve value corresponding 
        to the key from our DB

    -- So we need to make it so our put method puts the data in one of the DB shards and our get method should fetch data from 
        a DB shard in which the data we need is present which will make it consistent as if we put data in DB1 and get data from DB2
        we won't find the data we are looking for as it is present in DB1 hence showing inconsistent behaviour which we need to 
        avoid (so need good hash function)

    -- So we have seen how we will do sharding 

    -- So now we need to do replication so our DB must be replicated so that in case of any natural disaster or anything we can just
        take data from other replica which helps to recover data and our data won't be lost 

    -- So our DBs should be in different data centers as if something happens to one data center, our data will be safe in another 
        data center

    -- So make our replication more strong we move in clockwise direction like we do in consistent hashing and then we take a value
        like let's say we take N = 3 which means we will store our key-value pair in 3 DBs that will come along the way when the 
        key moves in clockwise direction in hash ring (ignoring virtual nodes as they are logical only)

    -- We will ignore virtual nodes in this and will store it in concrete DBs as virtual node doesn't really exist, they are just 
        logical

    -- So replicas of the concrete DB will be stored in different geographical areas as in case of disasters the data won't be lost
        if a data center is compromised 

    -- The N value in this method means the number of DBs so we will store our data in 3 concrete DBs so key will move in clockwise 

    {
        So mainly as manually backup process will be needed so we use N value so that if DB1 doesn't work then user can be 
        redirected to DB4 so that they can find their data and UX is smooth as at frontend they won't feel anything
    }

    -- Mainly done all this for data protection using N value (in real world cases there are many DBs in hash ring so N=3 or 4 is 
        very small value as compared to large numbers)

    -- So MongoDB, Cassandra DB and Dynamo DB uses this 


# Note

    Keep the theory and practical implementation of consistent hashing different as it depends on company

    As in Lecture-4 it is the theory of consistent hashing as can implement that way in theory

    But in practical implementation like by MongoDB, Cassandra DB and Amazon Dynamo DB it is like this :

        -- Virtual Nodes are logical replicas of Original Nodes 

        -- Virtual nodes are there only to distribute data evenly among all the DBs (they do not physically exist they just
            like show same DB at different locations in hash ring so that data is smoothly distributed)

            {

                Virtual Nodes are logical shadows used for balancing.

                Cannot use real nodes as there are chances that they might end up close to other replicas so high chances
                of uneven data distribution

                Like if we take hash ring of 1-100 and 2 servers s2 and s3 end up too close to each other and s2 far from s1 then
                load on s2 will be more and load on s3 will be less due to which there will be uneven distribution of keys



                If the Concrete DB (Physical Server) fails, all the data it was responsible for—across all its 256 "logical" 
                VNodes—becomes unavailable on that specific machine.

                So if one DB fails then all virtual nodes and all are removed from hash ring and then data from it's replica is
                hashed again and moved and replicas are then removed

                So even if we take 256 virtual nodes then also we aren't resharding the whole DB so work is saved

                Locations are determined by like if we use IP hashing then adding some suffix like #1 or something to make 
                unqiue identifiers of it

                Although it doesn't guarantee that the virtual nodes will be located far but it isn't a problem in real world
                as even if they are they would just seen as 1 virtual nodes responsible for storing data only (in real world
                many virtual nodes are made)

                Instead of re-calculating the hash values of the keys we can just keep their hash values stored or something so 
                it will save CPU cycles

            }


        -- So now the ways of replications for each of them are different

            {
                Cassandra & DynamoDB: The "N-Value" WayThese two are the "Consistent Hashing" brothers. You are 100% correct about 
                                        them.No "Backup" Servers: Every server is a peer. There are no "original" or "replica" 
                                        servers in the traditional sense.The Rule: You set $N=3$. For every piece of data, the 
                                        system simply picks the next 2 physical neighbors on the ring to be the replicas.The 
                                        
                Result: Any given server is a "Primary" for some data and a "Replica" for other data at the same time.


                So in short Cassandra and Dynamo DB use N-value to implement replication 

                Whereas in MongoDB

                MongoDB (usually) does not use the N-value ring logic. Instead, it uses a Primary-Secondary model within "Replica 
                Sets."

                The Group: You create a group (a Replica Set) of, say, 3 servers.

                The Hierarchy: One is elected the Primary (handles all writes), and the other two are Secondaries (replicate data 
                                from the primary).

                Sharding: If you want to scale your platform, you create multiple Replica Sets. Shard A is one group of 3; Shard B 
                            is another group of 3.

                The Replica Set (The Safety Group): Each "Shard" is not just one computer; it is a group of 3 (or more) computers.

                The Primary: The only one that the hash ring (or the balancer) "talks" to for writes.

                The Secondaries: They just follow the Primary. They are not represented on the global shard map or the hash ring.

                So in short in MongoDB instead of using the N-value for replication they use the replication technique by making 
                replicas of each shard in hash ring 

            }


        -- So in theory part of consistent hashing we use concrete nodes for data sharding and virtual nodes for data replication


# Continue 

    -- So now we will try to achieve consistency in our current DB

    -- So N = Number of Replications 

    -- So now if client uses put(key,value) method then how will our key-value will be stored in N-1 DBs after being stored in the 
        DB while moving clockwise and if our client uses get(key) method then from out of which DB will it fetch data like if we 
        have N=3 then how will it store data in 2 more DBs after storing in the first one and then when get method is used from 
        which DB it will fetch data 

    -- So what MongoDB, Cassandra DB and Amazon Dynamo DB does is that they make a coordinator and coordinator is a proxy

    -- So now client will talk to the coordinator and the coordinator will talk to the DBs

    -- So let's take an example that the client sends the put(k,v) request to the coordinator so now coordinator will be responsible
        for storing this data in N value DBs

    -- So there are chances that due to some errors or something the data wasn't stored in any one of the DBs so to keep track of it
        we use *** Quorum census ***

    -- Quorum are of 2 types :

            -- R : Read Quorum 

            -- W : Write Quorum 

    -- So we have let's say W = 3 meaning we need to put(k,v) in 3 DBs as W = 3 so Write Quorum will tell coordinator from how 
        many DBs it should recieve acknowledgement that the data is inserted in the DB 

    -- So the three DBs will send acknowledgement when the coordinator sends data either parallelly to all of them or sequential 
        (first to DB1 then DB2, ..)  

    -- So after it recieves acknowledgement from all the 3 DBs then only it will send status code 201 to the client 

    -- So coordinator will wait until it recieves acknowledgement from all the 3 DBs for W = 3 

        {
            Real World: Most people set W=2 and N=3 so that the system can still work even if one server dies.

            So our W value can be <= N value
        }

    -- So if we have N = 3 and W = 2 then coordinator will send data to 3 DBs only but will wait for acknowledgement from 2 DBs 
        only so that write operations can be performed fast and won't get time out if one server dies

    -- Similarly we can do W = 1 also so in short W or write quorum is the number of acknowledgement the coordinator must recieve 
        or can say how much acknowledgement is needed from the servers

    -- So now for R = 3 it will asks all the 3 DBs to get the data corresponding to the key and then when all 3 DBs return the value
        then it matches them all if all of them matches then only it sends the data to the client and if any one of them don't match
        then the coordinator won't send any response to the client (as a collision has occurred here)

    -- So if we take R = 2 then it will only asks 2 DB for the data and if the response from any 2 DB that comes first matches then 
        it will send the data as response to the client it won't even care if the 3rd response from DB didn't come as it only needs
        2 responses and needs to just match them 

    -- Similarly for R = 1 it will ask all 3 DBs only but if one of them replies first then it will just send that data to the 
        client 

    -- So we take value of W and R not equal to 3 for N = 3 so these N, R, W values makes trade-off between consistency and latency

    -- So if we take R and W value to 3 only for N = 3 then consistency will be there but latency will be more as in write it will
        wait for acknowledgement from all 3 DBs and similarly in read it will wait for responses from all 3 DBs

    -- So if we take R = 1 then it will sacrifice consistency for availability as if some key k has value v1 stored in 2 out of 3 
        DBs and in some DB out of the 3 it has stored old result for it only and it's value v2 so if DB3 responds faster than 
        DB1 and DB2 then coordinator will just return the v2 value for the key k

    -- So, 

            if R = 1 and W = N then we can say that this system is optimized for reads (as R = 1)

                -- So in applications that has read intensive work will use this technique as in this way by keeping W = N 
                    consistency is achieved and for faster read operations we have made R = 1 only as data will be consistent in 
                    the DBs

            if R = N and W = 1 then we can say that this system is optimized for writes (as W = 1)

                -- So it is useful in applications where there is write intensive work so even if there is old data in some other DB
                    like if for a key k DB1 has value RAM, DB2 has value RAM and DB3 has value Shyam as value was later updated to
                    Shyam only but due to some error the process wasn't completed in DB1 and DB2 as W = 1 so coordinator got 
                    acknowledgement from DB3 first so it didn't care about others and so then also as we have R = N so even if data 
                    is not consistent in the DB so using the read quorum it will compare values from each DB and if mismatch then 
                    it won't return data to the client (collision is present)

                -- So coordinator will first remove collision and then will respond to the client 

            
        So we can analyze a pattern that if R + W value is greater than N then we can say that there is strong consistency

                R + W > N

        So we can also determine that if R + W value is less than or equal to N then there is weak consistency

                R + W <= N

        So in short

                R + W > N  {system has strong consistency}


                R + W <= N  {system has weak consistency}


    -- So coordinator is needed to make sure that acknowledgement will be recieved by it and same for data comparision and fetching 
        data

    -- So as discussed in CAP theorem that there is a trade-off between C & A but there is a way to achieve both C & A and in most
        real world applications which prefer availability this method is used and it is called *** Eventual Consistency ***



# Consistency Models

    (1) Strong Consistency 

            -- Client never sees wrong data in this 

            -- So as we know there's a trade-off between consistency and latency so hence we risk our availability to gain 
                consistency 

            -- So if we have like R = N then our coordinator will wait from responses from all the DBs so if in case 1 DB was taking
                time to response and then after retry (imagine retry mechanism is implemented) also DB didn't respond and after 
                sometime it was seen that the DB was down so in this case the coordinator won't send any response to the client 

            -- So now the data is not available so hence our availability is at risk 

            -- So in this model we will always risk our availability as consistency is strong 

    
    (2) Weak Consistency 

            -- Client may see wrong data in this 

            -- So in this model we prefer availability over consistency 

            -- So in same example as above if we make R = 1 then even if the data is old in the DB that responses first we will send
                this data only to the client

            -- So can say that in this model consistency is at risk

    
    (3) Eventual Consistency

            -- So this is the practical model and in real world applications this is used 

            -- So in this we prefer availability over consistency 

            -- Client may see wrong data in this but given enough time the consistency will be achieved

            -- So if we take for N = 3 we take W = 2 and R = 1 so if one DB has wrong data and that DB responds faster when get() 
                method is called then it will return wrong value but if user gives enough time then eventually the data in DB will 
                be consistent 

            -- So we will later see algorithms for collision detection, collision avoidance and wrong value in DB detection 

            -- So this is called as eventual consistency as given enough time the data will be consistent in the DB if no more 
                updates were made

            -- So in all 3 MongoDB, Cassandra DB and Amazon Dynamo DB they all follow eventual consistency but there are 
                configurations that let's us pick strong consistency in these 

            -- So by default they prefer eventual consistency but by changing configurations we can make them strong consistency 
                model

            -- So we need to select the amount of time after which the system will be consistent as in case of banking applications
                we keep this time very less

            
# Resolving Inconsistencies in Data (Versioning)

    -- So to resolve inconsistencies in the data in DBs we use *** Versioning ***

    -- So we use versioning so along with the data the version of the data will be stored so if we do put(name, Ram) so in DB 
        3 things will be stored key = name , value = Ram and version = 1 so if later on we change this name to Shyam then 
        key = name , value = Shyam and version = 2 so with this we can determine that higher the version value the more latest or 
        updated data it is 

    -- So if a data has version = 3 we can easily determine that this data has been updated twice 

    -- So we do versioning in our NoSQL DB 

    -- So there are vectors in this in format [ server, version ] 

    -- So like if our client sends put(name, Aditya) so then the coordinator let's say only sends this to store in server s0 so s0
        will store the value and along with the value it will store a vector [s0, v1]

        {
            Storing server as a UUID as if one server crashes and we name the new server same as the old one then it will be problem
            for coordinator as it will consider it the as the same old server 
        }

    -- So now let's assume put(name, Aditya) is called and coordinator stores data in 3 DBs i.e. DB0, DB1 and DB2 so each DB will 
        have it's vectore [s0, v1], [s1, v1], [s2, v1] so now if client again sends the request put(name, Abhay) then let's assume
        that our request didn't reach DB2 and only reached DB0 and DB1 so now DB0 and DB1 will update their vectors as [s0, v2],
        [s1, v2] and in DB2 vector will be [s2, v1] (as we have W = 2 so it didn't wait for acknowledgement for DB2 so hence 
        inconsistency rose) so now if client sends read request to the coordinator and let's say for now R = 3 so coordinator will
        fetch data from all 3 DBs and it will then compare and will see that there is mismatch in data so now coordinator will see
        the version of data and will see that the latest version of this data is v2 so it will know that value Abhay is correct so now it will also update the data in DB2 as it knows that it doesn't have the latest version

    -- So hence eventual consistency is obtained

    -- So in above example only if we take R = 1 or 2 then if client sends GET request then coordinator will fetch data from 1 or 2 
        DB and will perform comparison but what if for R = 2 the response came from those two DBs that had v2 of the data whereas 
        the 3rd DB had v3 then data might be provided wrong and similarly for R = 1

    -- So we have 2 cases that we can do :

            -- In first case we leave it for client to again fetch the data as if client again and again sends request to get the   
                data at one time the response will come from that DB which has older version of data so the version and data can 
                then be updated

            -- Or in other case we just send the wrong data 

            {
                So in real world applications algos are used that update the stale data in DB eventually hence attaining eventual
                consistency
            }

    -- So how will coordinator know that it has sent the wrong data to the client or how can it be sure that the data that it is 
        sending is correct

    -- So when server will send the wrong data in response to the client request it will internally have a log file and coordinator
        will visit this log file and will search for all the versions of the data like in above case put(name, Abhay)

    -- So coordinator will look in the log file 

            Let's say in log file it's like this 

                Aditya --> v1

                Abhay --> v2

    -- So with this the coordinator will know that the response it had given to the client was of the old version and it had the 
        new version available for it so it will send the put(name, Abhay) request to all the DBs again 

        {
            If some DB contain correct version then it won't get overwritten and it's version won't be updated as DB will see that
            it's value is same and won't change 
        }

    -- So if again one DB didn't update then this will go on repeatedly and eventually consistency will be attained 

    -- Then if correct data is sent to client then also coordinator will look into the log file and will confirm that it has sent 
        the correct data

    -- So in log file data ID is stored and their corresponding version to save space as storing whole data might lead to less 
        storage

        {
            Quorum consensus is a mechanism in distributed systems where a minimum number of nodes (a quorum) must agree on a 
            transaction or data update to ensure consistency and fault tolerance
        }

    -- So we attain eventual consistency by versioning

        {
            Each DB will have it's own log file to avoid SPOF

            So it will fetch log file from different DBs and will compare and the one with the highest version of the data will be
            considered as the current truth
        }


# Handling Failures

    -- So what will we do if one of our DBs go down due to some errors so failure will occur

    -- So there can be two types of failure in this :

                (1) Temporary failure

                        -- So in this we can handle this type of failure by just assigning next server in hash ring for temporary 
                            data push/pull

                        -- When our old DB is again up then we need to perform migration as the new data that came when the old DB 
                            was down must be stored in the next DB in the hash ring so migration will be performed

                
                (2) Permanent Failure 

                        -- So in this case we create a new server and migrate old data to it 

                        -- So now our new server will be the replacement to the old server


# How to detect Failure (Gossip Protocol)

    -- We use Gossip Protocol to detect failure

    -- So let's assume we have N = 5 so how will we detect if a failure has occurred as we have already seen if a failure occurs 
        then how we will handle it so now we need to detect failure also

    -- So we will give each server a table in which there will be 3 entries : Server Name, Heartbeat and Timestamp

    -- So at a fixed time interval each server will update it's heartbeat 

    -- Heartbeat is nothing but just like in client-server if we need to check if our server is up so what we do is that we make an
        API call at let's say /healthy path and server return 200 (ok) so we will know that the server is up and if server won't 
        respond to our API call then we can say that the server is down

    -- So in this after each time interval each server will update it's own heartbeat and send this table to other server (1 or 2 
        servers)

        Then from this table the other server updates their own table and then update their own heartbeat and send this to other 
        server

        This happens again and again and then all servers will have updated heartbeat 

        So in this way they can easily detect whose heartbeat isn't updated in a while 

        If a heartbeat is not updated in a while then the other servers will know that a failure has occurred at this server

        So hence by gossiping between servers they determine that a failure has occured 

        If the Current_Time - Timestamp > T (a pre-set limit), the nodes mark the server as "Down" in their local tables

    -- So in this theirs no central authority that initiates gossiping which is why it is better than API calls as if there's some
        network problem in API then we will assume that our server is dead but it might be alive whereas in this it will be easily
        detected by servers that some server is down 

    -- So by gossiping with each other server 1 can tell to everyone that he's alive

    -- Then if we have detected a failure then we know now how to handle it 


# Merkle Tree

    Cons of versioning :

        -- Our system becomes complex 

        -- So if one server goes permanently down then we need to migrate data to new server that we will create but we will also 
            need consistency in other DBs also so that data migration will be easier so coordinator will have to check if data's 
            latest version is matching in all and if not then will have to update data and version in each one so this is obviously
            time taking as they will play with themself all along until they themselves have the consistent data

    
    -- So here we introduced Merkle Tree to overcome the complexity of data migration in above case

    -- So it will make it easy to migrate data and we won't be needed to visit the log file again and again 

    -- So in this we will find hash values of each of the data stored in DB

    -- So we will find hash at each level like in level 1 we will find hash of each value and then in next level we will make pairs
        of 2 from level 1 and will hash them together (so hash values of values found in level 1 will be combined and hashed and 
        stored in level 2)

        Similarly for level 3 we will combine and hash again values in level 2 and so on

        So at last we will have a single hash for a group of values that were stored in DB

    
    -- So this will be root and from this only we will compare whether we want to change or update values in DB or not

    -- So in diagram we can see that at level 3 DB0 and DB1 hash doesn't match so we will go deep in its left and right child

    -- Left child will be the first hash in level 1 in both DBs and right child will be the 2nd hash in both the DBs

    -- So after comparing left child in DB1 and DB0 we see that they are equal so we won't go further in left child direction as 
        hash are equal so values will be same 

    -- So now we will compare right child and as their hashes are not equal so we will again go deep in the right child and will 
        again compare the hashes in level 1 so we will compare left and right child (left child is first one in both DBs and right
        child is 2nd one in both DBs after excluding the first 2 as hash 1 in level 2 was their hash)

    -- So we can easily see that hash of e and d is different so we can say that the data is different here so we will just update 
        it

    -- So we can use any hash function in this case like SHA-256

    -- So we do this in O(log N) instead of O(N)

    {
        Engineers use three "tricks" to make this process much faster than it sounds:

            1. Incremental Hashing (The "Pre-calculation" Trick)
                    
                    -- The database doesn't wait for a failure to start calculating hashes.

                    -- Every time a write happens (like a user updating their profile on platform), the DB updates the leaf hash 
                        and "bubbles up" the change to the root immediately or in small batches.

                    The Result: When it’s time to sync or migrate, the tree is already 99% built. The DB just has to "read" the 
                    existing hashes rather than calculating them from scratch.

            2. Physical Range "Bucketing"
                    
                    Instead of one giant tree for the whole database, NoSQL systems like Cassandra create separate Merkle Trees for 
                    different Token Ranges (or VNodes) on the hash ring.

                    If only one small part of the ring is failing, the system only compares the Merkle Tree for that specific range.

                    This follows the "Divide and Conquer" principle. You don't scan the whole 1 TB; you only scan the 1 GB section 
                    that is suspicious.

            3. Background "Off-Peak" Scheduling
                    
                    Merkle Tree comparisons (Anti-Entropy) are usually not part of the critical path for a user's request.

                    Read Repair (comparing versions during a read) is the "Fast/First Response" fix.

                    Merkle Tree Sync is the "Deep Clean" fix. It usually runs in the background during low-traffic hours.

                    This ensures that while the CPUs are busy hashing, your platform users aren't experiencing lag.
    }

    {
        So when mismatch is found the DBs directly exchange their version and compare with it's own and write the latest version 
        data in their own DB

        (DB is not just DB as it will have a computer so can do it's own talking)

        So hence coordinator won't have much load on it 

        So merkle tree is only seen in background processes and not by the coordinator as it is used to keep data consistent in the 
        DBs and used in cases of scehduled maintenance and data migration
    }


    -- So internally our MongoDB, Cassandra DB, Dynamo DB uses these things but they have more complex process there in them 

    -- Internally they have Bloom filters, Memory cache, SS Tables (Sorted String Tables) 

    {

        *** Extra ***
        

        1. Memory Cache (MemTable)
        
                -- The MemTable is the "Fast Desk" of the database. It is a data structure stored entirely in RAM.
                
                    -- How it works: When a write request arrives, the DB writes it to the local Commit Log (for safety) and then immediately puts it into the MemTable.
                    
                    -- The Sorting: Unlike the log file, the MemTable keeps data sorted by key. This makes searching in RAM incredibly fast (O(log N)).
                    
                    -- The Flush: RAM is expensive and limited. Once the MemTable reaches a certain size (e.g., 128MB), the DB freezes it and "flushes" it to the disk as a permanent file.
                    
        2. SSTables (Sorted String Tables)
        
                -- Once a MemTable is written to the disk, it becomes an SSTable. This is the permanent "Filing Cabinet."
                
                    -- Immutable: Once an SSTable is written to disk, it never changes. If you update a value, the DB just writes a 
                                    new version in a new SSTable.
                    
                    -- Sorted: Because the data was sorted in the MemTable before being flushed, the SSTable on disk is also 
                                perfectly sorted.
                    
                    -- Efficiency: Because it is sorted, the DB doesn't have to scan the whole file to find a key. it can use an 
                                    index or Binary Search to find your data in milliseconds.

        3. Bloom Filters
        
                -- This is the "Security Guard" that stands in front of the SSTables.

                        -- The Problem: A NoSQL DB might have hundreds of SSTables on disk. If a user asks for User_ID_99, the DB 
                                        doesn't want to check every single file on the slow hard drive to see if that ID exists.

                        -- The Solution: A Bloom Filter is a tiny, super-fast structure in RAM that can tell the DB: "That ID is 
                                        definitely NOT in this file" or "It MIGHT be in this file."

                        -- How it helps: It allows the DB to skip 90% of the files on disk instantly. If the Bloom Filter says "No,
                                            " the DB doesn't even touch the hard drive.



        When our old SSTables are deleted from DB?

            -- To prevent your hard drive from filling up with "ghost" data, the database performs a background process called 
                Compaction.

            -- The Merge: The DB takes two or more small SSTables and merges them into one large, new SSTable.

            -- The Cleanup: During this merge, the DB looks at the versions. If it sees Version 2 and Version 1 for the same user, 
                            it only writes Version 2 into the new file.

            -- The Deletion: Only after the new merged SSTable is successfully saved to disk, the DB finally deletes the old, 
                            original SSTables.



        

        When does Compaction happen?

            -- It isn't always once a week; it usually depends on size and frequency. Most NoSQL databases use one of two 
                strategies:

                        -- Size-Tiered Compaction: When the DB notices you have, say, four SSTables that are roughly the same size 
                                                    (e.g., all 100MB), it triggers a merge to turn them into one 400MB file.

                        -- Leveled Compaction: The DB organizes files into "Levels" (L0, L1, L2). Once L0 gets too many files, they 
                                                are merged down into L1.

            -- So compaction is not done between large SSTables and small SSTables as it would be like if you merge a tiny "post-it 
                note" file with a "giant encyclopedia" file every time, the CPU would be working way too hard for very little gain.

    }


*** Extra : Simple Flow ***

        -- Unlike a B+ Tree, which goes directly to a specific spot on the hard disk to update data (In-place update), the LSM Tree 
            uses a multi-layered approach :

            
                Step A: The MemTable (In-Memory)

                    -- When you write data (like a new car listing for Mercauto), it is first written to a MemTable in your RAM.

                    -- Writing to RAM is incredibly fast because there is no mechanical movement or "disk seeking."

                    -- The MemTable keeps the data sorted (usually using a Skip List or a balanced tree).

                Step B: The SSTable (On-Disk)

                    -- Once the MemTable gets full, it is "flushed" to the hard disk as a single, large block of data called an 
                        SSTable (Sorted String Table).

                    -- Crucially, this SSTable is Immutable (it never changes).

                    -- If you update a record, the LSM Tree doesn't find the old version to change it; it simply writes a new 
                        version in a new SSTable.

                Step C: Compaction (The Cleanup)

                    -- Over time, you end up with many SSTable files on your disk. To keep things fast, the DBMS runs a background 
                        process called Compaction.

                    -- It merges multiple SSTables into one, keeping only the most recent version of each record and deleting the   
                        old ones.





    

                




