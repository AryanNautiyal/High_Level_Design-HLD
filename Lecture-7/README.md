

*** Notification System ***

    Q : What are the type of notifications ? 

    Ans : 
            -- PUSH Notification (Mobile Push)

            -- SMS Notification

            -- Email Notification


    Q : Real Time System vs Soft Real Time System 

    Ans : Real time system are those system that immediately send notifications to the user

          They are used in like In-App notification (easier to implement in In-App) or in like stock market

          Soft Real time system is a system where we try to send notification as soon as possible (there can be exceptions)

          Exceptions can be like system down, server is busy, etc

          So companies make soft real time system as in case of real time system the notification can get lost so we don't want
          notification to get lost 

          So like if we don't get notification that someone has followed us on Instagram in real time then no harm is done so 
          soft real time system are usually used 

          So let's assume interviewer asks us that make soft real time system 

          OTPs as we have seen also come after 2-3 mins so they are also made using soft real time system

    
    Q : Which device types are supported ?

    Ans : Let's say interviewer says that every type of device should be supported

          Like IOS, Android, Web (Windows, MacOS)

    
    *** Core Entities ***

        (*) Client (client can be anything like in microservices we see that other microservice can act as client to get it's 
            request fulfilled)

        (*) Notification Service (so our notification service will create notifications, set the payload as it will send API call 
            to send notification and there will be some templates in which it will just fill details and create notifications)

        (*) Executor (it actually sends notifications to the users and knows the logic for sending notifications to client through 
            SMS, Email and PUSH notifications)

        

    *** Back of the Envelope Calculations *** 

        Assumptions : 

            Daily Active Users (DAU) : 1 Million

            How many notifications are to be sent : 1 user ----> 10 notifications

            (Due to 1 user 10 notifications are created)

            So total notifications : 1 Million * 10 = 10 Million 





        QPS :

            Create notifications operations (Write operations) = 10 Million 

            Create notification operations (Write operations) / sec = 10 Million / 86400 = ~ 116 operations/sec

            Peak QPS = 2 * QPS (Write operations/sec) = ~ 232 operations/sec

            Let's assume 1 notification is dispatched to or read by 10 user so 1 : 10 ratio

            Dispatch notification operations (Read operations) / sec = (10 * 10 Million) / 86400 = ~ 1160 operations/sec

            Peak QPS = 2 * QPS (Read operations/sec) =  ~ 2320 operations/sec


        Storage Unit :

            Let's assume 1 notification takes 200 bytes (content, timestamp, userID, notificationId, etc)

            Total = 10 Million * 200 byte = 10 * 2^20 * 200 = 10^3 * 2^20 * 2 = 2 * 2^30

            Total = 2 * 2^30 = ~ 2GB per day storage required

            -- Notification store for 1 month as we don't need much history about notifications so we won't store notifications 
                beyond 1 month 

            For 1 month :

                Total = 2GB * 30 = ~ 60 GB 


        Bandwidth Calculations (Important in Notification systems) :

            (#) Definition : Maximum amount of data that can be transmitted through the network in a given time 

            -- So it tells how busy our server will be 

            -----------------                                               -------------------------------------
            |    Service    |                     HTTP                      |           Notification            |
            |               |---------------------------------------------->|           Service                 |
            |               |                                               |                                   |
            -----------------                                               -------------------------------------

            Calculations : 

                Notification Payload : 1KB  {for both upload and download}  
                
                    (components in payload can be headers, body, request param etc)

                DAU = 1 Million

                Total notifications / day = 10 Million 

                So 1 KB * 10 Million = 1 * 2^10 * 10 * 2^20 = 10 * 2^30 = 10 GB/day

                Bandwidth = (10 Million * 1 KB) / 24 * 60 * 60 = (10 Million * 1 KB) / 86400 = 10 * 2^20 * 1 * 2^10 / 86.4 * 2^10

                          = 0.115 * 2^20

                          = 115 * 10^(-3) * 2^20 = 115 * 2^10

                          = ~ 115 KB / sec

                So if our server is not capable of transmitting 115 KB / sec then we will be needed to scale the server

    
    *** API Interactions ***

        So we will assume that tables are created in SQL DB for this so userID, Name, mobile, etc will be stored

        1. To retrieve user information

            -- So service will tell notification service to send a notification so notification service will first need user data
                to whom the notification will be sent like his email if email notification or his phone number if sms

            GET : /fetchuser/{{userID}}

            Response : 200 (ok)

            {
                "userID" : 101,
                "userName" : "Aditya",
                "notificationPermissions" : {
                    "PUSH" : true,
                    "Email" : true,
                    "SMS" : false
                }
            }

        
        2. Send Notification

            -- After getting details of user the notification service will send notification to the user

            -- from field can be null if the system is sending notifications

            POST : /sendNotification/{{userID}}

            Body : {
                "from" : null,
                "to" : 202,
                "subject" : "",
                "content" : "",
                "type" : "push/email/sms"
            }

            Response : 201 (Resource created successfully)

            {
                "status" : "sent",
                "notificationID" : 301
            }

        3. To fetch notification history (optional)

            GET : /fetchAllNotifications

            Queryparam : filters :
            
            date : from to

            Type : email/sms etc.

            Response : 200 (ok)

            [
                {
                    "status" : "sent",
                    "content" : ""
                },
                {
                    "status" : "sent",
                    "content" : ""
                }
            ]


    *** Basic Design ***

        -- So in our push notification it goes to mobile only so 

            Push Notifications --> Mobile
        
                    -- IOS (Apple) ----> APNs (Apple Push Notifications)           
                    
                        {It is a 3rd party system that apple provides to send notifications to Apple's Push Notification System}

                    -- Same for Android also they have FCM (Firebase Cloud Messaging)

                    -- For SMS there are Twilio, nexmo, etc 

            So 3rd party systems are made to send messages to the devices 

            For Apple, Android the APNs and FCM are official whereas for SMS we can use Twilio, nexmo, etc

            Same for Email notifications we can use MailChimp, Sendgrid, etc

            [Interviewer won't ask us to create this ourselves as 3rd party systems will be used here]


        So in our notification system there will be many services that will tell notification system to send notifications to other
        person 

        These services can be microservices like in Amazon payment gateway (to send notification that order has been placed after 
        payment), user cart (might give notifications like complete your order by paying for the items type) or can be of discount 
        also, etc 

        These services can also be cron jobs (Cron jobs are scheduled APIs like we can schedule a method to be called everyday at 
        11am)

        So now our notification server will communicate with 3rd party systems to send notifications to the people 

        So simply our cron jobs or microservices will communicate with notification server which will then communicate with 3rd
        party systems

    
    *** Flow ***

        1 -- Any service (microservice, cron jobs) trigger notifications and send it to our notification server

        2 -- The notification server has APIs that builds up the payload for 3rd party services (APIs) to send the requested
                notifications

        3 -- Third party services actually send the notifications to the client 

    
    *** Improvements in Basic Design of our Notification System ***

        -- Current problems in our basic design are :

                (-) SPOF is there as single Notification Server 

                (-) Hard to scale 

                (-) Retry mechanism missing (as in case a notification is not sent to client due to server crash so retry mechanism
                    should be there)

                (-) Tight coupling (synchronous) 

                    {So like if our service1 sends request to notification server for IOS notification then server will do that 
                    work but at that time also if our service2 also sends request to notification server then server will put it on
                    hold as APNs will send response 200 then server will know that notification is successfully sent hence 
                    synchronous or tightly coupled and this leads to our system being slow}

            # So we will assume that microservices to notification server all the things are done (like scaling and all) as our main
              task is to design notification server so assume this 

        -- So we will introduce cache and DB in our notification system and between notification server and 3rd party systems we 
            will introduce messaging queues

        -- So our notification server will be publisher which will publish all the notifications in their specific queues like 
            in SMS queues SMS notifications will be there later given to Twilio, etc and same like this for others 

        -- All the 3rd party systems are now subscribers with each one having it's own messaging queue

        -- So we will also introduce workers which will give notifications from messaging queue to 3rd party systems whenever they
            are free (cannot make 3rd party systems as workers as we don't have internal access to their codes and all) 
            {Solved tight coupling problem}

        -- Messaging queues removed dependency between 3rd party APIs and our notification server and service is now asynchronous 

        -- So now we will scale our notification server so we will horizontally scale our notification server which will solve our
            SPOF problem

        -- So we are assuming our Notification servers, cache and DB are internally scaled horizontally as our main focus is 
            notification system

        -- So now service tells notification server to send notification then our notification server picks template for 
            notification from DB and creates notification then sends it to messaging queue then workers pick up the notification 
            then send it to 3rd party system to send to user 

        -- So what if our worker picked up notification from messaging queue and then gave it to 3rd party system but due to some
            reason there was some internal error due to which notification wasn't sent to the user to retry mechanism will be 
            needed here as notification is removed from the queue so the notification is lost 

        -- So our notification system should be reliable (no notification should be lost)

        -- For now we have created soft real time system for our notification system as notification server puts the notification 
            in messaging queue which is later sent according to server

        -- So to make this notification system reliable we use worker to send the data back to the messaging queue if in case
            some error occurs so that afterwards that notification is again sent to the 3rd party system by worker 

        -- So now our retry mechanism is implemented


        ** What all is stored and where **

            DB & Cache : user information, notification templates, etc 

            Messaging queue : stores notifications 

    
    *** Final Improvements ***

        -- Improvements that can be done in our current updated design :

                -- Notification templates

                -- Notification settings on user level : user can turn on/off a particular type of notification 

                -- Rate Limiting

                -- API authentication (so that noone can use our API except our services)

                -- Monitoring Logic (fetching notification history from DB but need to store notifications also)

                    {Can be used to determine like how many SMS notifications, etc are done in a day, how many retry notifications
                     are there, latency, etc}

                -- Notification Log  


        -- So in notification server we will implement the rate limiting and API authentication to protect our server 

        -- Then in our workers we will give each worker it's own DB to store Notification Log 

        -- Then for monitoring logic we will create a service named like let's say Analytics service that talks with Notification
            server, Messaging queues, 3rd party systems and Notification Log DB

            {
                Like Analytics will talk to notification server to determine how many notifications are sent and created in a day

                It talks with messaging queues to determine how many notifications go to retry 

                It talks to notification log to get logs of notifications to see when and where our system got faulty

                And it talks to 3rd party system to determine how many Email, SMS notifications are sent 

                It also determines like let's say our Email notification 3rd party service was failing too much so we can partner
                up with another 3rd party which provides this service and works fine so this is how analytics helps
                
            }



        




             






