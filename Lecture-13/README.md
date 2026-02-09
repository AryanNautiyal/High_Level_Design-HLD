


*** Design Youtube / Streaming Service ***

    -- We don't cover every usecase in HLD interview (we tell interviewer which usecase we will focus on and the usecase should be 
        important to catch interviewer's attention)


# Requirements 

    Functional Requirements :

        (1) User (creator) can upload a video

        (2) Users can view / stream a video

    
    Non-functional Requirements :

        (1) Availability is more important in this case than consistency 

    
    Requirements that are not covered :

        -- Likes / Dislikes

        -- Channel Management

        -- Comments


# Question and Answers


    Q : CAP ? Consistency vs Availability

    Ans : We can ask this question also to our interviewer or can just ask interviewer the Non-functional Requirements but for now
            let's assume interviewer says that availability is given priority 


    Q : Any restriction on video size ?

    Ans : As in youtube we can upload any size video and it will just compress it internally so we can ask interviewer here whether
            there's any limit to video size so let's assume the interviewer says that there are no restrictions on video size the 
            file can be in GBs


    Q : DAU ?

    Ans : Let's assume interviewer says 100M DAU for now 

            1 million videos are uploaded / day

            100 million videos are watched / day 

            So on average 1 user --> watches 1 video / day


    Q : Are there compressions internally

    Ans : Let's assume the interviewer says yes


# Back of the Envelope Calculations

    Assumptions :

        -- DAU = 100M

        -- 1 million videos are uploaded / day

        -- 100 million videos are watched / day

        -- Average video size after compression is 1 GB


    QPS :

        So there are 100 million users that are daily active on our platform 

        Write operations / day = 1 Million write operations / day

        Write operations / sec = 1 Million / 86400 = (1 * 2^20) / (86.4 * 10^3) = (1 * 2^20) / (86.4 * 2^10) = 2^10 / 86.4 

        Write operations / sec = 10^3 / 86.4 = 1000 / 86.4 = ~ 11.57 query / sec

        Read operations / day = 100 Million read operations / day

        Read operations / sec = 100 Million / 86400 = 1 Million / 864 = 1 Million / 864 = ~ 1157.4 query / sec


    Storage Unit :

        Calculating storage unit for 5 yrs

        Let's assume average video size after compression is 1 GB

        Storage unit = 1 Million * 1 GB * 365 * 5 = 1 * 2^20 * 1 * 2^30 * 365 * 5 = 1825 * 2^50 = 1.825 * 10^3 * 2^50

        Storage unit = 1.825 * 2^10 * 2^50 = 1.825 * 2^60 = ~ 1.825 EB


# Basic Flow

    (1) Posting a video

            -- So a client will send request to LB which will then send request to the Video Service server 

            -- So client will send POST request to the LB to post a video on youtube

            -- Then from video service server the video will be stored in DB but we don't store video in DB as storing huge videos 
                in DB will make the retrieval slow so cannot store them in DB

                {
                    Although using BLOB (Binary Large Object) in SQL we can store videos, photos in DB but there's a limit and all 
                    for the DB which is why we don't
                }

            -- Reasons for not storing videos in DB : 

                    -- DBs are expensive and it's memory will become full

                    -- DB will become slow 

                    -- Cannot query on videos

                    -- Migration (SQL --> NoSQL) won't be possible 


            -- So we will update this basic design so our Video Service server will have 2 DBs

            -- So now our actual video will go to S3 Bucket and our metadata of the video will go to the DB

            -- Video metadata can be like videoID, videoName, description, etc

            -- So to store video metadata in DB we will be needed to choose a DB so for this we will see a sample metadata and 
                determine the DB we should use for this usecase

                Sample Metadata :

                        {
                            VideoID : 123,
                            VideoName : "something",
                            videoDesc : "Anything",
                            timestamps : [

                            ]
                        }

            -- So as we can see that they are in key-value pair so we can use NoSQL DB like MongoDB, Cassandra (as they are better
                than Amazon Dynamo DB)

            -- So we choose NoSQL here as we assume that we can have complex documents also as value so in SQL will need to form 
                tables for it, foreign key, etc so we just used NoSQL DB


# API Endpoints 

    1 -- User can upload a video 

            POST : /upload

            {
                video,
                videoMetaData
            }

            Response : 201 (Resource created successfully)

            -- So user will send request at /upload path and will send video and video metadata through body in JSON format and 
                server will send response with status code 201 


    2 -- User can watch a video 

            GET : /video/{videoID}

            Response : 

            {
                Video,
                VideoMetaData
            }

    
    -- So now our flow will be client sends request to the LB then LB sends this request to the Video Service server then it will 
        store the video in S3 bucket and video metadata in NoSQL DB

    -- So in our design up till now we know that the video uploading responsibility is given to video service server but it will 
        make the process synchronous and also if upload speed is slow then the load on server will be more 

    -- So first let's understand how our S3 Bucket works 

            So first our video service server will ask URL from our S3 Bucket where the video is to be uploaded and then S3 bucket
            gives the URL to the server and then on this URL the server uploads a video

            We can think of S3 Bucket like a Google Drive where each document has a different URL


# Problems 

        1 -- Server will become extremely busy

        2 -- Even if you introduce Messaging queues & workers then also it won't help as MQ are used to store small size data 
                like in KB but if we store GBs of video in MQ then the storage of MQ will be full as it has limited storage

    
    -- So in chunks video will be transferred from our server to the S3 Bucket like first 1 GB of video was sent then another 1 GB 
        like this

    -- So then to overcome the problems above instead of giving server the responsibility to upload the video we will give client
        this responsibility due to which our server won't be extremely busy 

    -- So now our flow will be like this the client will send POST /upload request to the LB and then LB will send this request to 
        the video service server so the video service server will store the video metadata in DB and will call getURL of the S3 
        Bucket 

    -- So then this URL will be sent to the client and then client will upload the video on this URL meaning client will directly 
        communicate with the S3 Bucket and directly from the client's browser the video will be uploaded 

    -- Now the problems in our current design are these :


        1 -- No compression is used 

        2 -- Video is not divided into chunks


    -- So now in our streaming flow will be like this the client sends GET request at /video/{videoID} path which is sent to LB and 
        then LB will send this request to the S3 Bucket and the DB and returns the data to the client  

    -- So there is a problem in this flow also as if we send video to client as whole then that just means that our video service
        server will have to first retrieve video from S3 bucket which will again make the server busy and slow 

    -- So since currently in our design there's no streaming service so we can only tell client to download the video

    -- So therefore instead of sending whole video to the client, the server asks S3 Bucket for the URL where the video is stored
        then from there the server will just send the URL to the client and from this URL client can download the video 
        (downloading the video will also be slow)

    -- So as we know in YouTube we can stream videos so we will be needed to stream the videos also

    -- So before learning the implementation of streaming service we need to know some concepts  


# Concepts 

    (1) Video Codec 

        -- Codec : Encoder-Decoder 

        -- So to show video in streaming we will be needed to divide the video in chunks but we weren't dividing videos in chunks
            as our approach only sent whole video so if in our flow we also divide videos in chunk while sending video to the user
            for streaming it will then also take more time as first it will take time to retrieve video from S3 Bucket and then it
            will divide the video in chunks and send to user one by one so this will lead to very bad UX

        -- So we need to improve our design by dividing the video in chunks while storing it in S3 Bucket i.e. when the user sends
            POST request we need to divide the video in chunks but for that we will need a better design 

        -- So video codec is responsible to compress and decompress a video

        -- So video codec internally compresses the video and then the video is stored and when the video is again required then
            it can decompress also 

        -- So video codec will reduce the size of the video and will try to keep the quality of the video same so we can say that it
            will try for lossless compression 

        -- It can also do lossy compression but considering our usecase we want lossless compression only 

        -- So we can see that there will be a trade-off in which high compression will lead to higher chances for lossy compression
            whereas lower compression will lead to higher chances for lossless compression

        -- Examples : MPEG-2, MPEG-4, H.265

        { H.265 (HEVC) is supported in OBS Studio for high-efficiency recording and, with newer updates, for streaming, offering 
        better quality at lower bitrates than H.264 }


    (2) Bitrate

        -- It is the number of bits transmitted at a given time (we use like Kbps, Mbps)

            So 'B' = Bytes and 'b' = bits

        -- Like assuming the user has a high-speed uplink with a throughput of 200 Mbps, we need to estimate the time required to 
            upload a 100 GB video asset to our S3 storage 

                So, 

                100 GB = ~ 100000 MB

                Since 1 Byte = 8 bits

                100000 * 8 = 800000 Mb

                So the upload speed is 200 Mbps

                Time = 800000 / 200 = 8000 / 2 = 4000 sec 

                Time (in mins) = 4000 / 60 = 400 / 6 = ~ 66 mins = ~ 1 hr 6 mins

                If we take upload speed in MBps then 

                100 GB = ~ 100000 MB

                So the upload speed is 200 Mbps

                Time = 100000 / 200 = 1000 / 2 = 500 sec 

                Time (in mins) = 500 / 60 = 50 / 6 = ~ 8 mins


# Better Approach 

    -- So we will introduce a video processing service 

    Flow for posting a video (updated)

        -- So the client will send a request at /upload path which is sent to LB

        -- LB sends the request to the Video Service server with less load 

        -- Video Service server gets URL from S3 Bucket 

        -- Video Service server sends this URL to the client 

        -- Client then directly uploads this video in the S3 Bucket

    
    -- So now in this flow if client uploads 100 GB video then it will be as it is stored in S3 Bucket 

    -- So to maintain this video we have introduced Video Processing Service server

    -- So when client finishes uploading the video in S3 Bucket it triggers an event for which the video processing service server 
        was listening for then the server will use codecs to compress the video that is being uploaded by the client at any bitrate

    -- Then the original raw video will be deleted from the bucket and compressed one will be stored 

    -- So video compression is done so now any video size can be uploaded

    -- So now we want our Video Processing server to do 2 tasks i.e. compression and dividing video into chunks 

    -- So now we compress the video and then divide it into chunks

    -- So videoID and chunkID will be there as for each videoID there will be chunks which will have chunkID so now we will map
        these videoID with chunkID in DB 

            videoID : [chunk1ID, chunk2ID, ....]

            {
                By compressing the stream first, the encoder creates "Keyframes" (I-frames). These are perfect "clean break" points 
                where a new chunk can safely start without causing a visual glitch or "flicker" for the user.

                (Both operations occurs simultaneously i.e. compression and chunks)
            }

    -- So now these chunks are stored in S3 Buckets instead of the compressed file (each chunk has unique URL in S3 Bucket)

        [So there is one manifest file which is like an index catalog in which all chunks URL are there so we give this file to 
            user and from this file user one by one get each chunk]

        {
            If we store each chunk URL in DB then our DB can get full fast like if we assume a 100 GB video which is divided into
            1400 chunks then we will be needed to store all those 1400 URL in our DB which will take considerable space whereas in
            S3 Bucket that much space costs less than the DB 
        }

        
    -- So now both of our problems are solved 

    -- So we divided the video into chunks for streaming purpose as if client asks for video now the S3 Bucket will send chunks of 
        video 

    -- Download doesn't always mean the file must exist in your device like in YouTube also we see a red line which shows the 
        portion of video we have watched and there's light white line also showing how much more chunks are downloaded of the video
        so that even if there's a power cut the user can watch the downloaded chunks and can rewind video to watch starting part 
        again as user has downloaded those chunks (chunks are downloaded in RAM and cache)

    -- So like if we skip some chunk like user says I want to skip intro of this video so user puts the red line in middle so it 
        will start downloading chunks from the middle and if user again goes back to watch the intro part then it will download 
        those chunks

    -- So we store the minute-to-chunk mapping in our DB (as it's metadata) [minute to chunk mapping is mapping of chunks like from
        this time to this time this chunk is there in a video then this chunk]

        {
            Ideally it is stored in manifest file in S3 Bucket
        }


    -- So user will send SFTP (Secure File Transfer Protocol) request instead of HTTP request

    {
        Personal Note :

            When a user records a video on their iPhone or Android and uploads it to the platform, it isn't "raw" uncompressed 
            data. If it were truly uncompressed (raw), a 1-minute video would be over 10 GB, and the user’s data plan would vanish 
            instantly!

            The Phone's Job: The phone uses its own built-in codec (usually H.264 or HEVC) to compress the video so it can be 
                                stored on the device.

            The Problem: Every user has a different phone, a different quality setting, and a different file format (.mov, .mp4, .
                            avi). This is "Unstructured" and "Non-standard" for your system.

            Why we "Re-Encode" (Transcoding)

                    -- Even though the video arrives at the S3 bucket already compressed by the phone, we still need our Video 
                        Processing Service to do its job. We call this Transcoding (transfer-encoding).

                    -- We re-encode for three reasons:

                        (1) Standardization: To make sure every video on platform is in the exact same format so our web player can 
                                                play it.

                        (2) Bitrate Control: The user might have uploaded a video with a massive bitrate (e.g., 50 Mbps). We want 
                                                to compress it down to a "Streaming Friendly" bitrate (e.g., 5 Mbps) to save our 
                                                bandwidth costs.

                        (3) Multiple Qualities: The user only sends one version (e.g., 1080p). We need to encode the 720p, 480p, 
                                                and 360p versions ourselves.

    }


    -- S3 Bucket has many functions (lambda functions) that can trigger the event for Video Processing Service server


# Problem with current design 

    -- Slow internet connection of user can lead to too much buffering which will lead to bad UX as up until now the video will be
        stored in the resolution that the user uploaded it in like 4k will be 4k, 1080p will be 1080p so there's no difference so
        user will slow internet connection won't be able to smoothly watch these 1080p chunks 

        {
            Imagine a user has a 3 Mbps internet connection in a rural area.

            1080p Chunk: Requires a bitrate of roughly 5–8 Mbps to play smoothly.

            The Conflict: The user’s "pipe" is 3 Mbps, but the video "water flow" is 8 Mbps.

            The Result: The player will play 1 second of video, then wait 2–3 seconds to download the next chunk. This is 
            "Buffering Hell."
        }

        Solution :

            -- So we will store chunks in different resolutions by transcoding

                {
                    By transcoding the original video into multiple resolutions, you create a "ladder" of chunks. Instead of one 
                    set of chunks, you now have several sets, allowing the video player to pick the best one based on the user's 
                    current internet speed.
                }

            -- So now each set of chunks is stored in S3 Bucket to offer different resolutions to user so they can easily watch the
                video

                {
                    Transcoding: Changes the actual video data (the pixels). It requires huge CPU power because you are 
                                re-calculating the math for every frame.
                }

                {

                    Personal Note :

                        Video bitrate is the amount of data processed or transferred per second of video, usually measured in Mbps 
                        (megabits per second). Think of it as the "density" of information: a higher bitrate means better picture 
                        quality, finer details, and smoother motion, but results in a larger file size and requires faster internet 
                        for streaming. 

                        Like if we divide a video of 100GB to chunks of 10GBs each and compress each to 50MB and each chunk 
                        represents only 10 sec of the whole video so the user will need a bitrate of 

                            Bitrate = (Average Size of compressed chunk) / Time = 50MB / 10 = 5 MBps = 5 * 8 = 40 Mbps
                        
                        to watch this video smoothly

                        (We also take a peak bitrate to see the bitrate of the largest chunk out of all the chunks requires)

                }


            -- So now for a videoID ---> multiple formats ---> multiple chunks

                {
                    So now master manifest file will be there with URL of manifest file of each format and in those file will be
                    the chunks URL and miutes-to-chunks mapping also and master manifest file URL will be stored in DB
                }

            -- So if a video is originally in 1080p then to degrade quality to 480p, etc we will use lossy compression (lossy as 
                quality is being degraded)

    
    -- Now we need auto option so that according to user's internet speed it just select the appropriate resolution


        {
            Personal Note :

                The "Auto" Decision Loop

                        The video player (like HLS.js or the native player on an iPhone) is constantly running a background loop 
                        that acts like a speed test:

                            Monitor: While the 360p chunk is downloading, the player measures the speed: "I just downloaded a 1MB 
                            chunk in 4 seconds. My speed is roughly 2 Mbps."

                            Compare: The player looks at the Master Manifest (the "Menu"). It sees:

                            360p needs 0.8 Mbps.

                            240p needs 0.4 Mbps.

                        The Drop: If the speed suddenly drops to 0.5 Mbps (maybe the user moved to a room with bad Wi-Fi), the 
                                    player realizes 360p will cause a buffer stall.

                        The Switch: The player finishes the current 360p chunk and immediately requests the next chunk URL from the 
                                        240p Media Playlist.
        }


        Solution :

            -- So this automatic format selection we call it as *** Adaptive Bitrate Streaming (ABR) *** 

            -- So the video processing service server is responsible for checking the bitrate of the user if the user has 
                established a SFTP connection with the S3 Bucket

            -- Bitrate here can be downloading speed as user is asking for chunks here

            -- So if bitrate is high of user then will show user the video in 1080p and if user moves to room with poor internet 
                connection then will just change the video resolution to 360p as user bitrate will be low

            -- So we can see that according to bitrate the system is adapting so Adaptive Bitrate Streaming

            -- To handle information like this can use Zookeeper also 


# Optimize more

    -- So if interviewer says that the chunks still take too much space and we need to optimize space so that we can easily store
        many videos

    -- So what we can do is we can just do lossy compression and then while sending chunks to user we will perform decompression 
        first so that there will be no losses so in short just making them too small so that we can store them easily but opening 
        them back up while giving it to user so that they get good video

    -- Else without decompression our system will work fine 

    [Upload = Upstream & Download = Downstream]


# Final Design 

    -- The diagrams at the end of the pdf are the final design of the system for both usecases or functional requirement





    
    


    











        






