

Every application has client-server architecture


*** HTTP Components ***

HTTP calls --> POST, GET, PUT, etc

URL : www.example.com

HEADERS : 

BODY : 

Example of Header can be anything like Client Credentials, Request JSON

Example of Body is like JSON returned like
{
    "userID" : 1,
    "name" : "Aditya"
}

This is the whole request that is sent to the server



# Steps to improve the application

-- Introduced a DB tier to separate DB from server (Can now scale server and DB independently)

        ** Which DB to use **

            --> Relational DB : 

                        -- SQL Based : MYSQL, POSTGRESQL, ORACLE SQL

            --> Non-Relational DB : It is of 4 types

                        -- Key-Value Store : AMAZON DYNAMO DB

                        -- Column Based Store : Cassandra DB 

                        -- Document Based Store : MongoDB

                        -- Graph Store : GraphQL (used in cases like follow, unfollow, like, unlike, etc)


        ** Select a DB we will use depending on usecase **

            -- We select DB by seeing if the data we will be dealing with is it structured or unstructured 

               If the data is structured then we can easily use SQL based DB otherwise we use Non-Relational DB

            -- Second Criteria to judge is response time 

               So for less response time we use non-relational DB 


-- Vertically or Horizontally scaled the Web Server

        ** Need of scaling **

            -- Our 1 web server can handle only a limited number of clients so what if many users come

            -- Another reason is that what if our 1 web server crashes this will lead to whole application down

        
        ** Vertical Scaling **

            -- Vertically scaling a server has a limit beyond which we cannot scale the system 

            -- In this SPOF (Single Point Of Failure) is there

        ** Horizontal Scaling **

            -- Scaling horizontally is preferable as there will be no limit to which we can scale our application

            -- Problem here will be how will we direct the traffic from one server to two server now as some user will send request
               to server1 while other to server2 (DNS has their IP address also stored)


-- Addition of Load Balancer 

        ** Need of Load Balancer **

            It is used to distribute the load or traffic of users to multiple servers 

            So now the Client sends request to Load Balancer and Load Balancer decides which server to send this request to 

            So if Load Balancer sees that more load is on server1 then it will redirect users to server2 that will send request

        ** Flow **

            -- Client sends request 

            -- DNS sees the url and converts to IP address

            -- This IP address is of Load Balancer 

            -- Request is finally sent to Load Balancer 

            -- Load Balancer sees the load on servers and direct the request to the server with less load 

            -- Now due to introduction of Load Balancer the servers have private IP as we don't need anyone to direclty contact web 
               server

            -- So now our web servers are protected as no one can really contact them as they have private IP

            -- Then usual flow gives the response and all to user according to the request


-- DB Scaling

        ** Need of DB Scaling **

            -- Similar to how it was in web server, DB is also a SPOF (Single Point of Failure)

            -- So we will be needed to scale the DB also 

            -- So in case our primary DB fails we will need a backup DB

        ** -- Master Slave Architecture -- **

            -- It consist of a Master DB and some Slave DB

            -- All write queries like insert, update, delete, etc will be done on Master DB

            -- All the read queries will be done on Slave DB

            -- Master DB will replicate the written data to it's slaves

            -- The updation in the slaves can be done in 2 ways :

                        == Instantly updating as the write operation is performed on Master DB

                        == Timely updating like after 5-6 minutes so that all write operations can be done at once to slave's DB

            -- So the Slave's work will be to send the data to web server as requested 

            -- Slave DB will also have their own Load Balancer as only one master is there and many slaves are there

            -- So if by chance some DB corrupts or crashes then we will have it's multiple slaves

            -- Now what will happen if our Master DB crashes 

            -- So if we do a write operation on Master DB then due to this write operation Master DB crashes 

            -- So the slave DB aren't updated now as the crash happened before it could perform updation in slave DB

            -- So now we will select a Master DB from one of the Slave DB (different algos are there to select the new master)

            -- The new master is temporary as master DB is one so it will have good specs so therefore will up the master again and 
               then all good


-- Introducing Cache

        ** Need of Cache **

            -- Performs faster read, write operations than DB

            -- Located between Web Server and DB 

            -- First Web Server will ask cache if it has that data that the client requested if not then from DB taken 

            -- Cache Miss : Web Server asked cache for data but data was not present in cache then requested data from DB will come 
               in cache and then cache will send the data to Web Server to further send it to client

            -- Cache Eviction Algorithm (Page Replacement Algorithms I think is in this)

            -- So cache tier is introduced so now we can also scale number of cache 


-- Introduction of CDN (Content Delivery Network)

        ** Need of CDN **

            -- To load the static content of the web page faster we use CDN

            -- As without CDN client will request the load balancer then it will forward the request to one of the server then it 
               will ask DB then DB will return then web server will return the web page and then load balancer will return the data 
               to client 

            -- So to avoid this whole process CDN is used so that it will just take static data faster from CDN rather than asking 
               web server

            -- We take CDN from 3rd parties we don't make our own CDN

            -- CDN acts as a proxy  (CDN acts as a web server) [A proxy is a server that acts as an intermediary for requests 
               between a client and a server]

            -- Load Balancer is a reverse proxy [A reverse proxy is a server that sits in front of web servers, intercepting client 
               requests (like from your browser) and forwarding them to the correct backend server, then sending the server's 
               response back to the client]

            -- AMAZON : CLOUDFRONT || AKAMAI (AKAMAI is not of AMAZON just to be clear)

            -- As we know depending on geographical location we make multiple CDN

            -- So client will request data from closest CDN only 

            -- If data requested by client is not in closest CDN then it will go to other CDN

            -- If data is not found then it will go to web server

            -- CDN has the Load Balancer IP address if in case data is not found in CDN 

        ** Effects from adding CDN **

            -- Need to change DNS mapping table

            -- So in DNS mapping table we will add URL with the IP address of the CDN 

            -- If data not found in closest CDN then this CDN will ask data from other CDN and if not found in all then web server

            -- CDN has its own cache 


-- Introduction of Auto Scaling of Servers

        ** Need of Auto Scaling **

            -- So when the traffic on our website increases or decreases we manually cannot everytime know that and decrease or 
               increase the number of servers
            
            -- So auto scaling is necessary to automatically scale the servers when the traffic is high or low on web servers

            -- CDN is also costly so we don't use CDN for small application

            -- If we don't use auto scaling and leave number of server as it is then it will only increase the cost 


-- Introduction of Stateless Architecture 

        ** Need of Stateless Architecture **

            -- Client Sessions are also a problem 

            -- Like if a client does signup in one server then the client should be able to send request to the web server 

            -- But after the signup request, the second request was gone to another server 

            -- So another server doesn't recognize the client as he signed up on another server 

            -- But the other server in which the client signed up has stored the client session id

            -- HTTP requests are stateless so server doesn't know from where the request is coming 

            -- So we call this ** sticky sessions ** or ** Stateful Architecture **

            -- As the sessions of client are in one server and it depends on state so in this we will be needed to send all the 
               request from the client to one web server in which the client just signed up

            -- So we will also be needed to give information to load balancer also to only send client1 request to web server1 as 
               client1 has signed up in that server

        ** Proposed Solution **

            -- We will use stateless architecture 

            -- So we will use a Shared DB which will store the client data that has signed up 

            -- This will help client send request in any server as all servers will refer to the same DB to verify if that client 
               exists

            -- So we use either Redis or cache for this purpose

            -- Now this will help in auto scaling as our load balancer doesn't need to know about which request to be sent where

            -- So there is a server pool in which some servers are working and some servers are down so auto scaling will just up 
               the servers from server pool if needed and if not needed then will down the working servers so auto scaling and auto 
               downscaling


-- Introduction of multiple data centers

        ** Need of Multiple Data Centers **

            -- International users trying to access our site when all our servers are in India only 

            -- This will lead to a huge latency as accessing data from far away server will take time

            -- So to avoid this problem we use multiple data centers so that international users can also access our site fast

        ** What will happen in multiple data center **

            -- Each data center will have it's own servers, own cache and own DB

            -- Only shared DB will be same for all

            -- So for authentication and all we will just use one shared DB 

            -- As a client can also travel to other countries and we still want him to login to his account only so data about 
               client must be stored in shared DB 

            -- Data Center is a geographical location of any server 

            -- So now our Load Balancer will become ** Geo Enabled DNS ** and each data center will have it's own load balancer also

            -- Geo Enabled DNS will redirect user to their closest Data Center 

            -- So now each data center will have it's own caching strategies and all the other things

            -- Now our application can handle international users


-- Introduction of Messaging Queue

         ** Need of Messaging Queue **

            -- So when our server handles many requests from client so now our server is busy so we use messaging queue here 

            -- So if we have web server and say a DB so our web server gives a task to DB but our DB is crashed or under 
               maintenance then the server will give 500 error to the user but we don't want that so the task will be put in a queue
               so when DB comes back online it can just pick task from queue and just start processing it 

            -- Let's assume our DB has a limit of 1000 writes/minute and in a sale or something more than 1000 requests are coming 
               per minute then it will crash the DB as too many operations so here the server will just put tasks in queue and our DB can do tasks at 1000 writes/minute only saving from crashes

            -- As some tasks take long time like processing pdf or something then so if a user sends a file then the server 
               processing will take too much time and user will just see loading screen so it will put this task in a queue so user 
               will just get faster response and in background this task will be done by the worker 

            -- So like if our DB is doing a task from the queue then due to some reason DB crashes or something the task isn't lost
               as in queue the task is only removed when it recieves acknowledgement from the worker that task has been done so if 
               DB crashes then the queue will give this work to other worker 

         ** About Messaging Queue **

            -- So messaging queue works on ** Producer-Consumer Model ** or also called ** PUB-SUB Model ** (Publisher-Subscriber   
               Model)

            -- So Publisher doesn't care how fast the Subscriber works it will only just put tasks in queue according to it's use 
               and the consumer will work at it's own pace to finish tasks in queue 

            -- So a double ended queue is used also called as DeQueue 

            -- World wide these 2 are used for implementing messaging queues : Apache Kafka, RabbitMQ

            -- So in messaging queue we can say that we are implementing asynchronous behaviour 


-- Introduction of Sharding (or DB Partitioning)

         ** About Sharding **

            -- So we have only applied replication to DB in our system so if let's say our DB has max limit 1 million data as of 
               course DB will have it's physical capacity also so if we have more than 1 million data of users then we do sharding 

            -- So we only have Master only till now as Slave's only contain the copy of data in the Master DB so we will need to 
               scale DB so either vertical scaling or horizontal scaling

            -- So as vertical scaling has a capacity so we do horizontal scaling here 

            -- So in sharding the data is distributed in multiple DBs and data is distributed by taking mod value like '% 5' if we 
               have 5 server

            -- So by this we can easily distribute the data between various DBs

            -- So we need to take a sharding key whose mod we will take to determine the DB the data will be stored in 

            -- Resharding is very complex as to reshard billions of data it will be costly (as if we get many users so we add 1 
               more DB)

            -- So to reduce resharding we use algorithms like MacKay Tree Algorithm 

            -- Second problem is that like in our SQL DBs there is JOIN operations so how will we now perform JOIN operations 

            -- In this we call each DB as shard 

            -- Coming back to JOIN operations problem and also there is a problem like if we want multiple user's data at once then 

            -- So answer to these problems is that we De-Normalization the data 

            -- So each shard will have its own slave's as each shard is master 


** Introduction of API Gateway ** 

[Will discuss this later as need to understand API Gateway Structure]

