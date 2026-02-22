

Refer to Lecture-16 pdf for Lecture-17 also


# Whatsapp

    -- So in our peer-to-peer connection we are storing IP addresses of all the other clients that this client wants to chat with 
        which is not a good approach as client will have to store all the IP addresses of his friends in order to chat with them

    -- So we use our 2nd approach which is client-server architecture 

    -- So in this if client 1 sends message to client 2 the message will first go to the server but since client will be using HTTP 
        call like client 1 will send [message,receiverPhoneNo.] so server cannot broadcast the message so it can only reply back to 
        the client 1 only which is not good

    -- So our 3rd approach will be to use polling (HTTP polling or Long polling)

    -- So client 2 can poll for messages to get the messages for him so as already we discussed similar things in Lecture-16 so we 
        know that this approach is also not much good

    -- So we use our 4th approach which is WebSockets

    -- So now 2 way communication is enabled so now client 1 can send message to server and server will just send the message to 
        client 2 

    -- So similar to our Google Docs only we need to make similar to collaborate like anyone can send anyone messages, group chats

    -- So as we know that our Whatsapp service server will be auto scaled so if client 1 sends message to server 1 and if we assume 
        that client 2 is far from server 1 and closer to server 3 so the response should be given from server 3 to the client 2

    -- So we will make the system eventually consistent and will use a DB to make the system data consistent so now let's assume for
        now we are using SQL DB and we will have a table named Communicate with attributes as sender, receiver, message, timestamp,
        etc

    -- In our case 1 if both users are online then if user 1 sends message to server 1 and then server 1 will just store the 
        message in DB and then server 3 will fetch the message and send it to the user 2 

    -- In our case 2 if user 2 is offline then user 1 will just send message to server 1 and server 1 will store the message in DB
        so when user 2 comes online it will establish a WebSocket connection with server 3 and then server 3 will check for all the
        messages that are there for the user 2

    -- So we are using DB which is slow so we will use Zookeeper here 

    -- Zookeeper will store the client-server mapping as it is necessary to know which client has established WebSocket connection 
        to which server so that server can communicate with each other and user can receive data 

    -- So now we can either store messages in Zookeeper to make process fast or can use cache to get messages faster than DB

    -- So now to implement group chat functionality 

    -- So for group chat we will introduce 1 more mapping in Zookeeper which will map a specific group to all the client and then
        server can just look up server to which the client is connected and send messages to all those servers

    -- So our final design will consist of client, LB, Whatsapp service server, SQL DB, cache (like Redis), Zookeeper (refer to 
        Lecture-16 pdf for the diagram)


# API Gateway

    -- So API Gateway is a gateway for APIs which helps API calls to go to their desired microservice

    -- In a microservices architecture, the API Gateway acts as the "Single Point of Entry" for all clients

    -- When a request hits the Gateway, it performs Routing.

            If the request is /users, it routes it to the User Service.

            If the request is /orders, it routes it to the Order Service.

    -- The Gateway isn't just a simple "pass-through." It also handles the "boring" stuff so your microservices don't have to:

        -- Authentication/Authorization: It checks if the user is logged in before letting the request reach the microservice.

        -- Rate Limiting: It makes sure one user doesn't spam the API and crash the system.

        -- Protocol Translation: It can take a modern HTTP/2 request from a phone and convert it to gRPC for internal communication 
                                    between services.

        -- Load Balancing: It can decide which instance of a microservice is the least busy.

    -- LB distributes load to multiple server of same instances like if we have a microservice named Order then auto scaling will 
        be implemented to handle large number of users so multiple servers will be there for Order microservice only

    -- So in short LB distributes traffic to multiple instances of the same server 

    -- But API Gateway decide which microservice to call 

    -- API Gateway can do these things :

            -- Transfer API calls to correct microservice

            -- It can be used for Authentication (it will send request to Auth server to check if a client is authenticated or not, 
                if the client is not authenticated then it just rejects the request)

            -- We can implement Rate Limiting logic to API Gateway 


# How API Gateway decide which Microservice to call

    -- So if we have like 3 microservice named ms1, ms2 and ms3 then how will our API Gateway will decide which microservice to call

    -- So to decide this our API Gateway uses a concept called *** Service Discovery ***

    -- So for this we use Zookeeper so API Gateway asks IP address of the microservice to which this API call should be redirected

    -- So this request is then sent to that IP address of the microservice (LB of that microservice as the IP will be of LB as 
        microservice will have multiple instance)

    -- And Zookeeper gets the IP address information from the DNS (we can also directly connect DNS to API Gateway but as Zookeeper 
        has great information storing capacity so we use it)

    -- Can use Eureka also instead of Zookeeper 

    -- So this process is called as Service Discovery 


# Final Design using API Gateway 

    -- So in our final design the client will send API call which will first go to DNS to find the IP address of the servers

    -- DNS will have IP address of the servers based on Regions then the IP will be given to client of the server nearest to the 
        client

    -- Then after this (the IP given to client was of API Gateway) API Gateway determines from service discovery where to send the 
        request to 

    -- The request will go to LB of that microservice which will then give it to microservice 

    -- So after API Gateway we can further divide it into Availability zones like if we take Region 0 as Delhi then we can make 
        availability zones which will also consist of API Gateway, LBs, Microservices

    -- This will help us reduce traffic to one zone 

    -- If due to a disaster or something Availability zone goes down then all the requests coming to the availability zone that is 
        down are redirected to the other availability zone

        {
            Can also put LB before API Gateway so that we can use multiple instances of the API Gateway also so that load is not
            put on a single API Gateway
        }





