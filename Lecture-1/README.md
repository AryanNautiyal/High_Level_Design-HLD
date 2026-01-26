
*** Introduction to HLD ***

System Design : 

    -- LLD

    -- HLD (Focuses on architecture of an application)

So we need to see that which tech stack are we using as our tech stack can be MERN, springboot, Django, etc 

Let's take an example of multithreading environment like in bookmyshow to select same seat multiple users can select it at once so we won't use python there as it's not much good for multithreading 

We also see Cost Optimization of an application in HLD (cost of servers and other resources and our profit should be greater)

We also see which DB we will use (SQL, NoSQL, GraphQL, Column, etc)

thousands --> millions of user scaling of application to handle more users

We find estimate of how many users can our application handle so this estimate is called as **Back of the envelope estimate**


# Software / Application

    -- Website

    -- Mobile Application 

    -- Software 

So our primary focus is on website and mobile application in HLD

Software like photoshop we have in our computer so it's user's choice whether they want to update the software or not so in software cases HLD is less and LLD is more as it's an offline software that just uses the computer resources 


*** Request-Response model or Client-Server model ***

So we have a client and a server in this 

The client has a web browser through which it sends HTTP/HTTPS request to server and the server process this request and give the appropriate response to the client

So client can be anything not just web browser as in case of mobile application our mobile application acts as a client

So any client that may be a web browser or mobile application (any frontend) that sends an HTTP/HTTPS request to the server and the server process the request and give the appropriate response

This is client-server model

So HLD is purely based on a procedure called distributed system 

As if we have let's say 1000 clients and our server has limitations obviously so let's say it has 32 GB RAM so if 1000 users come and they use up all of our 32 GB RAM then how will we serve clients that come after 

So even if we increase the specs of the computer there's still a limit to how much we can upgrade 


# Horizontal Scaling vs Vertical Scaling 

Vertical scaling means upgrading the components of computer like making RAM from 32 GB to 64 GB 

Whereas the horizontal scaling means to just buy new servers

We only use vertical scaling when we have like kind of fixed users like 10000 only or something 

If we want to make application that can serve millions or billions of people then horizontal scaling is must

Later in this course we will study how these individual servers interact so that the data is common in each of them or can say DB is consistent 

So in HLD we call servers as **nodes** (represented by circles)

These nodes need to communicate with each other to maintain data integrity 


*** Network Communication ***

Network communication are rules that both client and server follow to communicate 

Network Protocols (Protocols == Rules)

Whole application is based on iSO_OSI model (7 layer model)

So application and Transport layer protocols are important for HLD

Protocols we use in Application Layer --> HTTP, HTTPS, FTP, SFTP, SMTP, WebRTC, WebSockets

Protocols we use in Transport Layer --> TCP, UDP

When a client calls a server over the internet it follows HTTP/HTTPS protocols 

So whenever client calls like an authentication or a critical resource it uses HTTPS call else it uses HTTP 

FTP (File Transfer Protocol) and SFTP (Secure FTP) so if client wants to send any document to the server or server wants to send then it will use these protocols 

SMTP (Simple Mail Transfer Protocol) is used to send the mail over the internet and POP3/IMAP is used to recieve mail over the internet 

HTTP says that client can only first send request to server then only server can respond 

So if we want like real time communication where server can also send messages we use WebRTC 

For P2P connection we use WebSockets

TCP (Transfer Control Protocol) and UDP (User Datagram Protocol)

When we want a reliable connection that ensures that our message reaches the server we use TCP connection 

Client sends message to server and we want to ensure that this message reaches the server so we use TCP connection here

In TCP it divides the message into packets and then sends packets one by one 

When first packet is received by the server it sends an acknowledgement to the client that it has received the first packet then only the client sends the next packet 

So hence as acknowledgement is needed for every packet so therefore it is slow

UDP is fast and non-reliable as here our client doesn't wait for acknowledgement and sends packets 

So UDP is favourable in real time video streaming and all as all the packets we receive we just show it even if some packets are lost then also we just see the video pause or voice break or something but if with other packets it makes sense then using those packets only the video is shown

UDP is connectionless where TCP needs connection 

Client doesn't directly talk with the server there is a DNS (Domain Name System) in between

So client talks to DNS and DNS brings the IP address (Internet Protocol Address) of the server

So client uses the URL (Uniform Resource Locator) https://www.youtube.com but this isn't the IP address of the server so DNS translates this URL to IP address of the server




