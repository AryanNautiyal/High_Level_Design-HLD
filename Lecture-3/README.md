


** CAP Theorem **

    Scalability : Distributed System 

    Centralized (wrong) [discussed in last class] {as SPOF and cannot scale centralized server}


    C - Consistency 
    A - Availability 
    P - Partition Tolerance

    C says that data should remain consistent in all the DBs
    
    A says that for every request a response should be given (data should be available at all times)

    P says that our application should always be up or it should always operate 


    CAP theorem says that no matter what we cannot get 3 in CAP, the max we can get is 2

    Partition Tolerance is mandatory

    ** Why all 3 are not possible **

        -- A user sends POST request to DB to update his profile 

        -- The DBs will make this data consistent 

        -- If a server or node is down then the request is redirected to different node 

        -- So data is also available to everyone 

        -- So if one of our node goes down then partition tolerance is supported so other DBs will just do the work 

        -- So now if a user tries to get data from the DB that is down then it will receive stale data or old data

        -- So we should like not support write operations on other DB when one DB is down 

        -- So if a user now comes to update his profile then he cannot update it so availability is compromised 

        -- So to achieve the Consistency we lost Availability 

        -- So if we don't want Consistency for now but we get availability

        -- So if user updates data then in user asks from DB that is down then it will get stale data 

        -- So consistency is lost but availability is there 

        Hence we cannot achieve C and A both together so max is 2 only (in this DB or nodes are data centers)

    ** Consistency vs Availability **

        -- If response is mandatory to be up to date then C (Consistency) is preferred

        -- If giving response is mandatory and response to be up to date is optional then A (Availability) is preferred

    ** Availability Numbers **

        -- In an application there is a contract between the application and the user 

        -- Contract tells that how many times can this application be down in an year

        -- If application doesn't go down then it's 100% available application

        -- If it's downtime is 99% then in an year it will go down for 3.65 days

        -- If it's downtime is 99.9% then in an year it will be down for 8.75 hrs

        [ Downtime : time during which a machine, especially a computer, is out of action or unavailable for use ]

        -- So 99% is the base standard nowadays as anything below it might be considered as bad application

        -- So Google, Amazon, Microsoft they all lie in 99.9% and 99.99%

        -- So as 99% is the base so we define our application in number of nines

        -- So Nmuber of 9s = Availability numbers 

        -- As more number of 9's mean that more available our application is in a year


** Back of the Envelope Calculations ** 

    -- We do some calculations before or after making the application so we call these calculations as back of the envelope 
        calculations

    -- We calculate how many users at once our application can handle 

    -- So we calculate this by *** QPS (Query Per Second) and Storage Unit ***

    -- So QPS tells us how many queries at once will our application get or can say how many read/write operations will be done in 
        a second 

    -- If this we can estimate the RAM of the DB, the number of caches we will need, In-memory cache needed or not, how many 
        replications of DB to be done 

    -- Like we are having multiple or can say many queries per second then we will make replicas of DB to perform parallel read and
        all that

    -- Storage unit tells that how much data the DB will store throughout the time (memory needed)

    -- So storage unit is calculated for a specific time period like let's say 5 yrs

    [ Back of the Envelope Calculations for Instagram ]

        -- Power of 2 is very useful for any calculation (in computer science)

        -- Power of 2 := 10 : Thousands : KILOBYTES (KB)
                         20 : Millions : MEGABYTES (MB)
                         30 : Billions : GIGABYTES (GB)
                         40 : Trillions : TERABYTES (TB)
                         50 : Quadrillions : PETABYTES (PB)
                         60 : Quintillions : EXABYTES (EB)

        -- Assumptions :

                    1. Monthly Active Users : 2 Billion

                    2. 60% of these users use Insta daily : ~1.2 Billion {this is called as DAU (Daily Active Users )}

                    We use Insta to only see reels and posts from others so here we are considering 2 usecases only 

                    -- User see feed

                    -- User posts photo/video/reel

                    So whenever user see feed, 1 feed request will be sent -- 1 query 

                    And also when user posts photo/video/reel, 1 post request will be sent -- 1 query

                    3. User check feed 30 times a day (Average)
                    
                    4. User publish one picture/reel a day (Average)

                    5. Estimate for 5 yrs

                    Feed View QPS : 

                        Daily Feed Request = DAU * 30 = 1.2 Billion * 30 {Multiplied 30 as on average 30 times feed request is sent}

                        Daily Feed Request = (1.2 Billion * 30) / (24 * 60 * 60) {Divided by 24*60*60 to convert to seconds as QPS} 

                                / 24 gives in hrs then / 24*60 gives in min and /24*60*60 gives in seconds (note it's in req/sec)

                                Hence divided not multiplied 

                        Daily Feed request = (1.2 Billion * 30) / 86400 = ~ 416667 or ~ 420k req/sec or ~ 417k req/sec

                        QPS = 417k req/sec

                            So we will also find QPS for peak hrs as in peak the QPS will be high

                    Peak QPS = 2 * QPS              { Formula for peak QPS }

                    Peak QPS = 2 * 420k = 840k req/sec

                    Upload QPS :

                        Upload QPS = 1.2 Billion * 1 = 1.2 Billion req/day

                        Upload QPS = 1.2 Billion / 24 * 60 * 60 = ~14k req/sec

                        Peak QPS = 14k * 2 = 28k req/sec

                    
                    Storage Unit :

                        Assumptions :

                            1. 20% of users upload video and 80% of user uploads photo

                            2. Average size of video uploaded by user is 50MB and for photo it's 1MB

                        
                        Photos : 

                            number of users uploading photo (or photos/day) = 1.2 Billion * 80% = ~1 Billion

                            1 Billion * 1MB = 2 ^ 30 * 1 * 2 ^ 20       { Converted Billion and MB in power of 2 for easier calc }

                            2 ^ 30 * 2 ^ 20 = 2 ^ 50 = ~1PB         { This amount of storage is required to store photos every day }

                        Videos :

                            videos/day = 1.2 Billion * 20% = ~0.25 Billion

                            0.25 Billion * 50MB = 0.25 * 2 ^ 30 * 50 * 2 ^ 20 = 2 ^ 50 * 12 

                            2 ^ 50 * 12 = 12PB                      { This amount of storage is required to store videos every day }

                        Total storage required : 1PB + 12PB = 13PB 

                        Calculate for 5 yrs = 13PB /day * 365 * 5 = ~24k PB 

                        24000 PB = 24 * 10 ^ 3 = 24 * 2 ^ 10 * 2 ^ 50 = 24 * 2 ^ 60 = 24EB      { EB = Exabyte }

                        So 24EB is the amount of storage required for 5 yrs


** Monolith vs Microservice Architecture **

    # Monolith 

        -- Single Code Repository (Monorepo)

        -- Like for example if we take Zomato then it's order, restaurants, payment and users handling programs are all in one place

        -- These are legacy application

        -- In monolith the process is faster as if someone wants to pay for the order then both are at same place so fast response

        -- Disadvantage of monolith architecture is scalability as it cannot scale much 

        -- Like if orders are coming in bulk then we will be needed to increase the server leading to increased cost and extra 
            amount is being paid just to scale orders as all are in one server so scaling as a whole is done 

        -- Deployment is also a disadvantage as we have all in one server making it bulky so if we want to add additional features
            then we will be again needed to deploy the whole thing 

        -- Debugging and testing will also be difficult as we will be needed to check the whole application so that it doesn't lead 
            to problems anywhere else 

        -- So to overcome disadvantages of monolith like main problem was of scalability so microservices was introduced 

    # Microservices

        -- So in this it says that instead of making one big service we can just divide that big service in small parts for 
            specific purpose like order is separated to handle only order related matter, etc

        -- So now if we only want to increase orders server then we can just do that as all are separated 

        -- So now the interaction between these services is not method call anymore as they all are in different servers, now it's 
            an http call (internet call)

        --  We can automatically scale each component independently, there is no more a single heavy service but now there are many
            lightweight services and debugging and testing will be easier in this

        -- In this like if we add new features in order then we will only be needed to redeploy the order server rest will be same

        -- Now in microservices as for interaction with each component a network call is required hence the network call will take 
            time resulting in latency (slow)

        -- Another disadvantage of this is ** transaction management ** and ** multiple microservice join **

        -- So like in ACID property as we know Atmoicity (A) states that either a transaction should be complete or rollbacked,
            it means that any transaction shouldn't be partially executed 

        -- So if our transaction hits like let's say 3 services s1, s2 and s3 so the transaction has performed operations in s1 DB
            then it does to s2 service but at s2 service the transaction failed so s2 cannot tell s1 to rollback hence it's a 
            problem here (although the disadvantages of microservices are overcome by the different design patterns)

        -- So now as for the other disadvantage if our services s1, s2 and s3 are dependent on each other in an order like 
            s1 --> s2 --> s3, so let this be the flow of transaction from s1 it goes to s2 then to s3 so if we update or change 
            code in s2 then we will be needed to make sure that s1 is also ok as we might change parameters that s2 expects so
            we will make sure that s1 gives the expected parameters so hence it forms a chain of dependency

        -- So we can remove the transaction management and multiple microservice join disadvantages by using different algorithms 
            but it will still be slow 


** Different phases of a microservice **        { Important Interview Question }

    1. Decomposition 

            -- Decomposition tells how to break down monolith application into microservices 

            -- It has it's own patterns to break down monolith application into microservices 

    2. Database

            -- Database tells which type of DB we will use i.e shared DB or unique DB

            -- Shared DB means all the services share a common DB

            -- Unique DB means that each service has it's own DB

            -- In microservices unique DB is preffered as shared DB has it's own disadvantages

    3. Communication 

            -- So we all know that services will make HTTP calls or can say internet calls to communicate with each other 

            -- But here we are also talking about whether the services will communicate using * APIs * or ** Event Driven System **

    4. Deployment 

            -- In this phase we decide how will we deploy our application (or microservices)

            -- Basically how will our CI/CD pipeline will work 

    5. Observability 

            -- In this phase we decide how will we monitor our application (or microservices)

    So 4 and 5 are handled in DevOPs i.e Deployment and Observability are both handled in DevOPs so the rest 3 are focused mainly 
    in HLD 

    Now diving deep into each phase

    ## Decomposition ##

        -- If you have a monolith then how will you break it to a microservice architecture 

        ** Design patterns of Decomposition **

            => Decomposition by Business Logic 

                -- So let's again take example of Zomato here so in Zomato monolith architecture we have orders, payments, users, 
                    restaurants, etc so here we will identify entities that communicate with each other or can say business logic

                -- So here orders, users, payments, restaurants can be made microservices as each of them are entities that 
                    communicate with each other or can say are the business logic 

                -- Disadvantage of this is that developer should be aware of the entire business logic (in teams can do this but 
                    still developers working should know the entire business logic)

            => Decomposition by Sub Domains 

                -- Better than Decomposition by Business Logic 

                -- So this includes breaking down of monolith application by domains

                -- Like we have www.zomato.com/orders/ so this is one microservice then www.zomato.com/payments/ is another 
                    microservice 

                -- So like this we will decompose our monolith application into microservices by decomposition by sub domains

            =>  Strangler Pattern 

                -- It says how will you do it 

                -- So when converting our monolith application to microservices we will have our monolith application deployed so
                    while building microservices to convert monolith application to microservices we cannot leave our application
                    unattended till we have made the microservices, we also cannot take down our application make it into 
                    microservices then deploy it (also it won't be tested and all also)

                -- So strangler pattern says that we should convert our application from monolith to microservices step by step 

                -- So we can strangle the monolith to convert it to microservices like let's say currently we have monolith 
                    application so 100% of our APIs calls from client are being handled by monolith application so now let's say
                    we have completely built our orders microservice then we will just redirect all order APIs to the microservice
                    so let's say now 90% of our APIs are handled by monolith application and 10% by our order microservice

                -- Now we will similarly create payments microservice let's say it handles 40% API calls so rest 50% API calls are 
                    handled by our monolith application now so in conclusion like this will slowly strangle the monolith application
                    so that the API calls are handled by microservices 100%

                -- So now all our API calls will go towards microservices rather than monolith 

                -- In this we can do this either by APIs or by users also 

                -- Like if we have built our microservices so we redirect some users to the microservices to ensure that everything 
                    is working properly, if everything is working properly then step by step we can increase the number of users 
                    gradually redirecting all to microservices to eliminate monolith but if some bugs happen then we can fix bug 
                    and while fixing can redirect them all to monolith only 


    ## Database ##

        -- Each microservice can have any one of the two :

                        ==> Shared DB

                                    Advantages :

                                        (+) Simple to carry operations (JOIN in SQL)

                                        (+) Transactions management is simple (ACID)

                                    Disadvantage : 

                                        (-) Cannot be scaled properly as if order is getting hit every time so instead of scaling
                                            order in DB we will be needed to scale everything 

                                        (-) Cannot use different DBMS for each microservice as if we think user data to be kept in 
                                            NoSQL and order data in SQL then we won't be able to as DB is shared


                        ==> Unique DB

                                Advantage : 

                                        (+) Can be scaled properly as if order is getting hit every time so can scale
                                            order only

                                        (+) Can use different DBMS for each microservice as if we think user data to be kept in 
                                            NoSQL and order data in SQL 

                                Disadvantages :

                                        (-) Not simple to carry operations (JOIN in SQL) {To solve this problem we use CQRS design 
                                            pattern}

                                        (-) Transactions management is not simple (ACID) like if user creates an order then payment 
                                            processes the order payment then it goes to restaurant microservice but due to some 
                                            error transaction was rollbacked in restaurant hence restaurant needs to tell payment 
                                            and order to rollback also       {To solve this problem we use SAGA pattern} 


                            So by using CQRS and SAGA pattern we can eliminate the disadvantages of our unique DB hence 
                            making it perfect for use


                            ## SAGA Pattern ##

                                    -- SAGA pattern solves the transaction management problem by using Event Driven System 

                                    -- So if we take three microservices order, payment and restaurant then in SAGA pattern
                                        after order completes the updation it publishes an event which then payment and restaurant
                                        listens to then after seeing event the payment executes it's process then it also publishes
                                        an event then after this restaurant also publishes an event 

                                    -- So if in case let's say due to some error the restaurant process is rollbacked then it will
                                        publish an event rollback event which the order and payment will listen to then payment 
                                        will rollback and publish an event rollback which then order listens to and rollbacks 


                                    -- If this might seem complex then SAGA pattern says that it also has another method to manage
                                        transactions

                                    -- So in another method in SAGA pattern a ** Orchestrator ** class is created which manages all
                                        the work 

                                    -- So Orchestrator will tell order to do work then after completion of order work it will send
                                        response to orchestrator then orchestrator will tell payment to do work and after completion
                                        of work payment will send response to orchestrator and then same for restaurant 
                                        (All the calls here between orchestrator and microservices will be HTTP calls as it's not an event driven system now)

                                    -- So now if in case restaurant fails due to some reason so it will send error instead of ok as 
                                        response to the orchestrator which will lead to orchestrator sending HTTP calls to both 
                                        payment and order to rollback

                                    -- Other name for event driven system is ** Choreography ** 

                                    -- Event driven system is kind of complex to make hence orchestrator is used

                                    -- In orchestration it is kind of tightly coupled as orchestrator cannot be free unless the 
                                        whole transaction is complete but in case of event driven system it can work 
                                        asynchronously as it is loosely coupled 


                            ## CQRS (Command Query Request Separation) ## 

                                    -- Command is any write operation like update, delete, insert

                                    -- Query is any read operation 

                                    -- So CQRS states that keep our read and write separate 

                                    -- So write will be performed in unique DB of each microservice whereas all read operations will
                                        be performed on a single View DB which will have all the DBs data 

                                    -- So now as in View DB all the different data that were in each microservice were there are in 
                                        view DB hence now join operations can be performed as all data is present in view DB

                                    -- So hence in this way we have separated command and query requests to different DBs

                                    -- Our Master Slave model supports this only as in that also master is used to do write 
                                        operations and slaves were used to do read operations





                    







