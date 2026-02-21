

*** Google Docs HLD / Collaborative Editing ***


    Q : What user can do ?

    Ans : Let's assume the interviewer says single user document editing and multi user editing same document (collaborative 
            editing)

    Q : How many DAU ?

    Ans : Let's assume the interviewer says 5 Million DAU

    Q : How many concurrent user can edit the same document at a time (Max limit of people that can edit the same document 
        concurrently) ?

    Ans : Let's assume the interviewer says 100 users


# Requirements

    -- Functional Requirements

        -- Single user editing : Single user can create/read/update/delete a document 

        -- Multi user editing : Multiple user can edit same document at a time 

    
    -- Non-Functional Requirements 

        -- Availability over Consistency

        -- Eventual Consistency is accepted 

        -- Read your own writes : User can see his/her own edits immediately (Important Functionality)


    -- Non-Focused area :

        -- Authentication and Authorization 

        -- Access writes 

    
# Back of the Envelope Calculations 

    Assumptions :

        -- DAU = 5 Million 

        -- One document size = 100 KB (on average)

        -- Read : Write ratio = 10 : 1 (considering 1 document is created by user once in a day and a user reads 10 documents in a 
                                day)

    
    QPS :

        Read operations = 5 Million * 10 = 5 * 2^20 * 10 = 50 Million 

        Read operations / sec = 50 Million / 86400 = 0.00057870 * 2^20 = 0.00057870 * 10^6 = ~ 578.70 queries/sec

        Peak QPS = 2 * QPS = 2 * 578.70 = ~ 1157.4 queries/sec

        Write operations = 5 Million * 1 = 5 Million

        Write operations / sec = 5 Million / 86400 = ~ 57.870 queries/sec

        Peak QPS = 2 * QPS = 2 * 57.870 = ~ 115.74 queries/sec


    Storage Unit : 

        So each document is of size 100 KB on average 

        So,

            5 Million * 1 * 100 KB (5 Million users daily create 1 document of size on average 100 KB)  {Storage Unit for 1 day}

        For 5 yrs

            Storage Unit = 5 Million * 1 * 100 KB * 365 * 5 = 9125 * 2^20 * 100 * 2^10 = 912.5 * 2^30 * 10^3 = 912.5 * 2^40

            Storage Unit = 912.5 * 2^40 = ~ 912.5 TB


# API Endpoints 

    -- So we will have a client and a document service server to handle client requests 

    (1) Create a new document 

        POST : /add/document 

            Request Body :
            
            {
                "title" : "First Document",
            }

            Response :

            {
                "documentID" : "123",
                "title" : "First Document"
            }

            -- So first we send the create document request in which we give the title of the document and then after getting 
                document ID we write the document and store the written content 

    
    (2) Read an existing document 

        GET : /fetch/document/{documentID}

        Response :

        {
            "documentID" : "123",
            "title" : "First Document"
            "content" : "",
            "created_at" : "",
            "updated_at" : ""
        }


    (3) Edit an existing document 

        PUT : /edit/document/{documentID}

        Request Body :

        {
            "title" : "",
            "content" : ""
        }

        Response :

        {
            "documentID" : "",
            "title" : "",
            "content" : ""
        }

    
    (4) Delete an existing document 

        DELETE : /remove/document/{documentID}

        Response : 200 (ok)

        {
            "documentID" : ""
        }


    (5) Editing within a document 

        -- /edit is used to change the whole content of the document (or title) whereas this will be used to edit some part of 
            document like updating normal text to bold and all 

        POST : /edit/document/{documentID}/operation

        Request Body :

        {
            operations : [
                {
                    "operationsID" : "",
                    "type" : "insert",
                    "position" : "20",
                    "text" : "Hello"
                },
                {
                    "operationsID" : "",
                    "type" : "",
                    "position" : "",
                    "text" : ""
                }
            ]
        } 

        Response : 200 (ok)

        {
            "documentID" : "",
            "status" : "success"
        }


    (6) Collaborative Editing

        -- So let's assume we have 3 clients c1, c2 and c3 so now our c1 is the creator of the document and he has created a 
            document and named it first doc then c1 didn't write any content in the document instead it gave edit access to c2 and
            c3 so now using /edit/document/{documentID} the c2 and c3 client can edit the document created by client c1

        -- So now c1, c2 and c3 all can edit the document (collaborative editing) so now if c2 changes the title from first doc to 
            collab doc then the changes will be either directly or eventually reflected to c1 and c3 also 

        -- So ways to do collaborative editing :

            [1] HTTP Polling 

                -- So if c1 makes changes in a document then c1 will send a HTTP request to the server to make the changes and the 
                    server will respond to the c1 with status code 200 (ok)

                -- But our server won't be able to tell c2 and c3 as HTTP call is based on request-response model so our client c2 
                    and c3 haven't requested for the data so our server cannot give response to them without a request and c1 cannot
                    also tell them that it has made a change in the document 

                -- So we use HTTP polling 

                -- In HTTP polling our client c2 and c3 will repeatedly request the server asking if there is a change in the 
                    document again and again so now if c1 makes any changes in the document then server can send the changes to the 
                    c2 and c3 client also as they are requesting for the data now 

                -- So now we have changed our request-response architecture to polling

                -- Pros :

                    (+) Easy to implement 

                -- Cons :

                    (-) Server will be bombarded with too many requests 

                    (-) Latency will increase (slow)

                    (-) It will consume a lot of bandwidth


            [2] Long Polling

                -- It is same as HTTP polling but instead of repeatedly requesting server again and again and server again and again
                    instantly replying to client request in this the server will hold the client request for some specific amount 
                    of time and will return response after some time 

                -- Like if c2 sends request to the server to fetch the new data then server will hold the request for like 1 min and
                    after 1 min the server will reply and then after 200 ms the client again requests for the data (200ms is like a
                    time we have set so that after each 200ms client will send request as there is no point in sending request 
                    every 1ms)

                -- So now there is a high probability like after 1 min there will be changes in the document

                -- Pros :

                    -- Server load is slightly reduced 

            
            [3] WebSockets (Best Approach)

                -- So in WebSockets it provides a bidirectional communication

                -- So client will establish a connection with the server and then now both can communicate with each other 

                -- So our (REST APIs or) HTTP calls (POST, GET, PUT, PATCH, DELETE) works on request-response model whereas our 
                    WebSocket works on 2 way communication

                -- So now our communication can be real time using WebSockets as now the server doesn't have to wait for client's 
                    request to send the data


                Why do we need WebSockets in Google Docs ?

                    -- So now our c1, c2 and c3 will form a WebSocket connection with our server so now if c1 changes content in 
                        a document then c2 and c3 will get the changed data from server without requesting for the data 

        
        -- API Endpoint for establishing connection 

            WS : /api/document/{documentID}/collaborate 

            Response : 200 (ok)

            {
                "message" : "Connection Established"
            }

        -- API Endpoint for editing document (in collaboration)

            WS : /edit/document/{documentID}

            Request Body :

            {
                "type" : "operation",
                "operation" : {
                    "type" : "insert",
                    "position" : 20,
                    "text" : "Hello"
                }
            }

            Response :  (This response will be broadcasted to every user connected as they all will be needed to change the data)

            {
                "userID" : "user-1",
                "type" : "operation",
                "operation" : {
                    "type" : "insert",
                    "position" : 20,
                    "text" : "Hello"
                },
                "timestamp" : ""
            }

        
        -- Pros :

            -- Latency will be low 

            -- Server is not bombarded with request 


# High Level Architecture

    -- So we will have a client and a document service server and for our DB we will use S3 Bucket to store the document and to 
        store metadata about the document we will use SQL DB

    -- So instead of Amazon S3 Bucket we can also use Google Cloud Storage (Google won't use Amazon S3 Bucket as they have their 
        own)

    -- Document can be file, image, document, etc

    -- So we are using SQL DB for metadata as we know that there are relations between data in metadata 

    -- Refer to diagram in pdf for diagram of the system 


# Link Sharing in Collaborative Editing 

    -- So our first approach will be to just send the S3 Bucket link for the data to the user 

    -- So our S3 Bucket link will look like this 

            https://www.amazons3.com/bucket/documentID/.....

    -- So our S3 Bucket uses hierarchical file system like we have in our Windows and MacOS (root folder --> sub folder --> sub 
        folders like this)

    -- Problem associated with this is as we have seen in Google Docs that we can share the document link only and not the whole 
        folder structure along with it like for example if we have Home ---> Coding ---> HLD ---> HLD_Lec_1.pdf so in S3 bucket
        the link will be like https://www.amazons3.com/bucket/Home/Coding/HLD/HLD_Lec_1.pdf 

    -- So in this if the client to whom the link was sent remove HLD_Lec_1.pdf then it can just see the whole HLD folder only so our
        basic approach will be to just apply access rights but this is just too much work for it so instead of all this work we 
        just need to send link of the pdf only so that way they cannot access the HLD folder and link will look like /HLD_Lec_1.pdf
        only

    -- So we can conclude that the hierarchical folder structure is not a good approach for file sharing 

    -- So problems we faced in hierarchical folder structure are :

            -- Too long URL (as if the document is inside a folder which is inside a folder and this chain is long then the URL 
                will be long too)

            -- We don't want to expose entire folder to all users (even if we apply access constraints then also the user can see 
                the folder structure which he does not wish to see as user only wants the document)


    -- So to solve these problems we will be needed to 

            -- Flatten up the URL

            -- Generate a unique URL (unique ID) for each file shared

    -- So our unique ID should be unique and as short as possible 

    -- So Google Docs generate this unique ID by just using alphanumeric string whereas Notion uses MD5 Algorithm which generates 
        MD5 hash of document name 

    -- To shorten URL we have also seen the approach in one of our Lecture in which we used the Base X approach 


# Final Design 

    -- So in our final design we have made a separate service for our collaborative editing as this service will work on WebSockets

    -- So all our WebSockets call will go through this server 

    -- So now our client 1 will send the edit request to the Collaborative Editing service and then this will go to client 2 also 
        which is collaborating on the document and then this is also sent to the Document service server to store the document and
        it's metadata 

    -- But this will be valid for our max 100 users only but what if we change max limit to handle 1000 people max in collaborating 
        so then as both the servers are tightly coupled so they will be needed to be loosely coupled to scale it as Collaborative 
        editing server might have high throughput as 1000 people might be just editing document together which will send many 
        requests to the server and all this request will be sent to the document service server 

    -- So if document service server has low throughput then server might go down so we will introduce Messaging Queues to loosely
        couple both the server and prevent low throughput server from falling 

    -- So now with the introduction of messaging queues now our system will be fine


# How to handle conflict

    -- So in a collaborated document if both client c1 and c2 edit the document at the same time and at the same line then how to 
        avoid this collision 

    -- So let's assume both of their requests looks like this 

            c1 : insert "hello" at position 0

            c2 : insert "Bye" at position 0

    -- So to solve this problem our basic approach will be timestamp approach 

    -- So in this we our client sends above example request then their timestamps will be seen and timestamps will be taken till 
        milliseconds or can say granular level so that we can see which client edited latest and will show the content edited by 
        the latest client

    -- Another approach is *** Operational Transformation (OT) *** which is used by Google Docs 

    -- The timestamp approach makes the UX bad as text inserted by c1 will be gone and user will think that he typed something and 
        it's not in the document so we use OT to make the UX good

    -- So now in OT if we take the same example as above then if c1 edited before then at position 0 the "hello" will be there and
        for c2 it will transform it's position from 0 to 6 as "hello" contains 5 letters and one space too so at position 6 it will
        insert the "bye"

    -- So in OT it will try to keep the changes of all the users in the document 

    -- This is the simple version of OT, in Google Docs more complex version of OT is used so that sometimes it overwrites, sometime
        it just keeps all of them together in document 

    









