

*** Authentication and Authorization ***

    Authentication : Authentication asks us : Who you are ? What is your Identity ?

    Authorization : Authorization asks us : What you can do ? (Access control / Access policies)

    -- Authenticate means like the client has to login (username and password)

    -- Flow :

            -- User comes to our platform 

            -- User go to register himself / herself (Sign In)

            -- User go to Login (Authentication)

            -- Now you can access application server based on your policies / rules (Authorization)

# Requirements

    (1) User Registration : User can register on our platform (basic details --> name, mobileno., etc)

    (2) User Login : Username and password (Authentication)

    (3) 1 FA vs MFA (Multi Factor Authentication) [MFA used by Secure Applications --> banking, Gmail, etc]

    (4) Password Recovery : (Forgot Password) [Notification --> mobile / email (Already registered)]

    (5) Session Management : Stateless vs Stateful Architecture 

    (6) Access control (Authorization) [Leetcode clone --> premium feature (accessed by only those users who paid)]


*** Questions ***


    Q : MFA or 1 FA ?

    Ans : Let's assume the interviewer says MFA (using Notification --> Email)


    Q : Applications DAU or monthly active users 

    Ans : Let's assume DAU = 100k


    Q : Session management ? Stateful or Stateless ?

    Ans : Let's assume interviewer says Stateless 


*** Back of the Envelope Calculations ***

    Assumptions :

        -- DAU = 100k/day

        -- 1 user --> 5KB (To store 1 user details it takes 5KB of memory)


    QPS :

        So we have 100k user per day

        So 100000 / 86400 = 1.15 req/sec 

        (Didn't make it complicated here and just found out the Average queries per second) [So above QPS is for both Signup/SignIn]

    
    Storage Unit : 

        Register --> (username, email, mobile number, etc)

        These details will be collected from the user when they register on our platform 

        So with this we can determine that each user information can take how much memory so let's assume 1 user --> 5KB (User 
        Information) [Can ask interviewer also this]

        Estimating for 5 years 

            100000 * 5 * 2^10 * 365 * 5 = 10^6 * 0.1 * 9125 * 2^10 = 912.5 * 2^30 = ~ 912.5 GB = ~ 913 GB


*** 1 FA vs MFA ***

    -- So let's assume we have an application like Instagram where there are billions of user interacting with the server so we try
        to make a separate server for our authentication system as it will have it's own authentication DB (Auth DB) 

    -- So now our user will give his username and password to the authentication server and after verifying or can say 
        authenticating the user it can then interact with our application 

    -- (We are assuming that all the horizontal scaling of servers and all is done)

    -- So this is our 1 FA as user gives username and password and authentication system return 200 (ok) to user then the user uses
        the application 

    -- So now in our 1 FA these are the components : user, authentication server, Auth DB, application server 

    -- In our MFA a notification server is added to the 1 FA system so now the user gives the username and password to 
        authentication server, the authentication server verifies the user and if the username and password are correct then it 
        will request the notification server to send OTP to the user on his mail ID and then user will enter this OTP to verify him
        and then the user can access the application server 

    -- So the notification server will store this OTP in Redis or some cache so that when user gives authentication server the OTP 
        it can match the OTP and verify the user

        [Used Mail ID as in Q & A we assumed interviewer said this]


*** Session Management *** 

    -- So the http requests are stateless (no information about the sender) {meaning each http request is unique}

    -- So as each http request is unique so server won't know that the user trying to access application resources is authenticated
        or not 

    -- So if we do like using user's IP, we save it in server (like RAM) and give it a session ID so that whenever user sends 
        request to the server using his IP we can determine that the user is authenticated so this is stateful architecture 

    -- But this is not good as we have already discussed in Lecture-2 as if user sends request to another server of the application 
        after authenticating then the server won't recognize the user as user is authenticated in another server so they are tightly
        coupled 

    -- So now instead of saving session ID in the server we save it in a central DB or Cache so that every server can recognize the 
        user 

    -- So we have a better way than this also (Session Based Auth vs Token Based Auth)

            (.) One way is that we use the session IDs in which the authentication server will generate a session ID and will store 
                this session ID in a DB 

            (.) Another way is that instead of storing the session ID in DB we give this session ID to user's browser to store it in
                cookies so we tell user's browser that whenever user sends HTTP / HTTPS request, the header in request must contain
                the session ID

    -- So in case of storing session ID in DB the servers can authenticate user by searching session ID provided by user's browser 
        cookie in DB or cache and will just remove the session details of the user when he logout

    -- It is more secure than token based auth (used in bank websites)

    -- So now in another method i.e. token based auth in which when a user sends login request to the auth server the server 
        internally generates a token which is like an encrypted object which contains user's information, permissions and might
        contain TTL (Time To Live)

    -- So server can decrypt using secret key using which it encrypts the message

    -- So this token in stored in user's HTTP only cookies

    -- Now using this cookie the server can get information about the user

    -- Types of Tokens :

            (=) Access token

                -- After authentication each request of the user will be sent with the access token

                -- Our server decrypts the access token using secret key and then will verify user information, TTL of access token 
                    to see it's not expired and user permissions

            (=) Refresh token

                -- Access token will have short TTL so we give refresh token a long TTL as it is used to generate access token

                -- Now after authentication every request is sent with an access token 

                -- So our server decrypts the access token and if access token is expired then refresh token will generate a new 
                    access token 

                -- As if our access token is expired then we cannot ask user to login again and again as it degrades the UX

                -- So now user will send request to auth server using refresh token so the auth server generates a new access token
                    for user to use

                -- So refresh token is not shared by user until and unless the user wants to generate new access token 

    -- So we use JWT (JSON Web Tokens)

    -- So if hacker gets our access token then the hacker won't have much time to do anything with our access token

    -- So we use refresh token as it's only sent when we need to generate access token and also improves UX

    -- So if we take example of Gmail also if we open Gmail and then leave our browser unattended for like let's say 3 hrs or 
        something then browser auto refreshes when we come back to Gmail window then Gmail silently uses refresh token to get a new 
        access token

    -- And then in Gmail after few days when refresh token is expired then Gmail asks for sign in again (if we select the remember 
        me option then refresh token won't expire)


*** How passwords are stored in DB ***

    -- Our first option is to store the password in our DB as it is i.e. in plain text

    -- So we already know that this is the worst method as if hackers hacked our DB only then all the passwords will be leaked as 
        they all are in plain text

    -- Our second method is to store password hashes in DB

    -- So user gives us his password and then we make his password go through a hash function using which we will get a random 
        string 

                password -----------------> hashFunction(password) ------------------> qwe234f&4@a#23

    -- So now even if our DB is hacked and hacker can see passwords then also he will only see the hash codes for the password, not
        the actual password 

    -- So as we already know that hash codes cannot be reverted back meaning we cannot reverse engineer the random string to get 
        the original password

    -- But as we already know that hackers can use Rainbow Table Attack to find the original password 

    -- As most people just use simple passwords like password123, aditya, etc, so all these common passwords can be easily obtained 
        by using Rainbow Table Attack

    -- Rainbow Table Attack is nothing but just a very large DB stored with many common passwords and their common hashes that users
        use so that using this they can easily find each password using the hashes

    -- So like we can just use all the permutations and combinations of all the a-z, 0-9, A-Z to get all the possible strings then 
        generate their hash codes using common hash functions like SHA-256, SHA-1, Bcrypt, Argon2, etc

    -- So using this table the hacker can just try all the passwords for a username to get the original password of the user 

    -- So using this technique many accounts can be unlocked 

    -- So we call the file or DB as Rainbow Table in which it will be like password, SHA-1 hash code for the password, SHA-256 hash
        code for the password and so on for other algorithms 

    -- So this is very unsafe 

    -- So we use Salt and Pepper technique to store the passwords in our DB

    -- So in this technique what we do is that let's assume we get a password from the user let's say aditya123 so we generate hash
        code for this password by combining it with salt after getting this hash code we also store salt in raw form in the DB so 
        that while comparing original password with the hashed password the system knows the salt (in case of using different salt
        for each password)

    -- So what bcrypt does in step by step :

            (1) First if we use bcrypt.genSalt(10) in this it will just generate a random string and will add $2B$10$ which is
                the bcrypt version and the number of rounds the password + salt will be hashed (it just adds header $2B$10$ with 
                salt so that whenever we make hash password we do not need to specify rounds again)

            (2) Then if we use bcrypt.hash(password, salt) the bcrypt sees in the salt that $2B$10$[salt] is the format it uses 
                so it will know that 2B = bcrypt version 2 is used for hashing and 10 = number of rounds so using this information
                it will combine the salt and password and will use bcrypt version 2 and will hash them for 10 rounds or 1024 times

            (3) Then after doing this the final hash password is obtained but as we are using different salt for each of the 
                password so that it's difficult for the hacker to crack many passwords together as if 200 users use password 123456
                and we add same salt like "s32" then the hacker can easily determine salt as all of their hash codes will look same
                so hacker will need to crack one only to get the salt also and the original password for many accounts so to make
                each password different we use different salt and the bcrypt also will be needed to know about this salt when 
                comparing the passwords so it adds the salt also in front of it so the salt is 22 characters and hashed password
                (mixture of salt and original password) is 31 characters and before salt it adds the bcrypt version and number of 
                rounds as well so that even if hacker knows the salt he will be needed to guess the original password

                    Format : $2B$10$[salt][password + salt <= hashed]

            (4) When we use bcrypt.compare(password, hashpassword), it extracts information like bcrypt version, number of rounds 
                and first 22 characters which is the salt and then it mixes salt with password given by user and hash them for 10
                rounds and compares the final hashcode with 31 characters in the end in the hash password, if they match user 
                successfully logins

    -- Sometimes only salting is not enough so we add pepper to the hashed password 

    -- Pepper is a predefined random string that is same for each password

    -- We don't store pepper in DB unlike salt as pepper is a secret 

    -- So that even if DB is leaked the hacker will not know the pepper

    -- So we store pepper in the .env file so that it stays safe and no one can read it 

    -- So now when pepper is there what we do is that we do Password + Salt + Pepper hashing many times according to the number of 
        rounds given 

    -- So for comparing also the bcrypt or any other system will have to compare by adding salt and pepper to the password given by 
        the user and then do 10 rounds or number of rounds mentioned on it 

    -- So pepper will be stored in very secure region so that no one can gain access to this 

    -- So we cannot just add salt and pepper to hash password simply as hacker if finds salt and pepper then it will be less secure 
        for others so we do plain password + salt + pepper --> hash and store this hash in DB (as discussed above)

    -- Good Hash Functions (asked wherever we use hash function)

            -- Password --> hash(password)

            -- So to store any password we need a good hash function so that we can hash that password and the password cannot be 
                easily obtainable to any hacker

            -- So our next question comes that what we consider a good hash function

            -- So in this case as our priority is security so we need a hash function which is slow 

            -- So we are using slow hash function as if we use basic hash functions like SHA-1, SHA-256, etc they are basic hash
                functions so they can hash billions of records in a second 

            -- So the fast hash functions can generate hash codes in microseconds which makes it feasible for a hacker to attack

            -- As if we think practically that hacker can just generate many passwords in seconds and in some days hacker can crack 
                the password 

            -- So with slow hash functions it makes the attack infeasible for a hacker as in seconds the amount of passwords 
                generated will be less and it will take more than some days to crack the password

            -- So good slow hashing algorithms are Argon2, bcrypt, scrypt, etc so they are slow meaning they take milliseconds to 
                generate hash codes 

            -- UX will not be impacted that much whereas the hacker experience will be impacted 

            -- So based on how much security we want we can add many salts in iterations


*** Authorization ***

    -- It tells us what we can do in an application or the access level or access rights

    -- They are 3 types of authorization :

                (1) Role Based Access Control (RBAC)

                        -- In this based on roles the access rights are given like admin, owner, member, etc

                        -- Like in discord we make owner which can do anything on the server whereas the manager can kick people,
                            change a person server username , etc

                        -- So based on roles the access rights are given 

                (2) Policy Based Access Control (PBAC)

                        -- In this we define dynamic rules or policies or privileges 

                        -- Like we can make a rule or policy like some user can access paid questions so we can decide which user 
                            has these privileges

                        -- In this we can make new policies 

                (3) Access Control List (ACL)    

                        -- It is useful for very small application

                        -- It gives highly granular control 

                        -- It gives individual permissions to user

                        -- Like in a discord server when we open a user's profile we can mute person or can give perms to each  
                            individual person like if a person can see this channel or can talk on this channel


*** API Endpoints *** [Make this after Back of the Envelope Calculations in HLD interview]

        -- POST : www.instagramclone.com/register

                -- Will give some details of user in this to register on the platform

                {
                    "username" : "aditya",
                    "fullName" : "",
                    "password" : "",
                    "mobileNo" : "",
                    "email" : ""
                }

        -- POST : www.instagramclone.com/login

                -- Will give username and password in this to login to the application

                {
                    "username" : "",
                    "password" : ""
                }

        --  POST : www.intagramclone.com/logout

            -- Will send username to logout 

            -- Reasons for choosing POST method 

                    (1) State Change: GET requests are intended for retrieving data and should be "safe" (read-only) and idempotent 
                                        (multiple identical requests have the same effect, no side effects). Logging out changes 
                                        the server-side state by invalidating a session or token, which is a side effect and is not 
                                        an idempotent operation.

                    (2) Security (CSRF Protection): Using GET for logout makes your application vulnerable to Cross-Site Request 
                                                    Forgery (CSRF) attacks. An attacker could trick a user into visiting a page 
                                                    that contains a hidden image or link to your /logout endpoint, automatically 
                                                    logging the user out without their knowledge.

                    (3) Browser Behavior: Modern web browsers and web accelerators might pre-fetch or pre-cache GET links to 
                                            improve performance. This could cause users to be accidentally logged out just by 
                                            hovering over a link or the browser anticipating their next action.

                    (4) Sensitive Data: While not typically an issue for the logout action itself (unlike login), GET requests 
                                        expose all parameters in the URL, which can be stored in browser history, server logs, or 
                                        shared via bookmarks, posing potential security risks for sensitive data. 

        
        -- Other API Endpoints can be

                -- /create/{userID} to create user

                -- /grant/{userID} to grant permission to user

                -- /revoke/{userID} to revoke someone's privilege

                -- /expire/{userID} to expire the session

                -- /validate/{sessionID} to check session 


*** Table Structure ***

        -- Sometimes interviewer might ask to tell table structure to store the data

                User Table : to store basic user information 

                                userID, Email, name, username, role, createdAt, updatedAt, etc


                Credentials Table : credentialID, Email(Foreign Key), access_token, refresh_token


                Password Table : passwordID, Email(Foreign Key), hashed_Password, salt, last_login, etc


                Session Table : Active sessions of user (history)

                                sessionID, Email(Foreign Key), last_logged_in, etc


*** Workflow using diagrams ***

        -- So client will send /register request to the Auth server and all the user information will go to the DB (diagram 1 for 
            it)

        -- So now for login the client will send /login request to the Auth server and the user will give username and password and
            Auth server will match password or can say validate password so if password validated then generate access token and 
            refresh token and store their details in DB and also store in client's cookies (diagram 2 for it)

        --  Client's access token will expire so now client will send request to Auth server with refresh token then with this 
            refresh token the Auth server will generate new access token and update the details in DB and also in client's cookies
            (diagram 3 for it)

        -- So now our combined diagram will look like diagram 4 (or last diagram in pdf)

            [ For MFA used notification server which will have it's own cache and DB ]

    { Stored access tokens and refresh tokens in DB so that we can just change token in case previous one gets hacked or something
        so that we can get voluntary control over the tokens }

        
    



