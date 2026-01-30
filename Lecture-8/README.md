


*** Messaging Queue ***

    -- So in our messaging queue there is a producer which produces messages and put in the messaging queue and there is a consumer
        that takes messages from messaging queue and consumes the message

    -- Main benefit is that the producer and consumer can work on their own pace

    -- So let's say we have 2 microservices named ms1 and ms2, so ms1 will produce message and give it to ms2 for consumption
        then ms1 will stay idle or can say wait for the response from ms2 so ms1 is sitting idle and this process is showing 
        synchronous operation

    -- With introduction of messaging queue now the process is asynchronous as now ms1 will not sit idle or can say wait for ms2
        response instead it will do some other work 

    -- So if our ms1 is a busy service and ms2 is a normal service, so if our ms1 bombards ms2 with many requests then we need to 
        see if our ms2 is able to handle all those requests or not or can say if our ms2 has high throughput or not

    -- So if our ms2 has high throughput then there will be no problem as ms1 also will be having high throughput and ms2 also will
        have high throughput 

    -- But if our ms2 has low throughput then our ms2 might go down sometime

        # Throughput (in HLD) : OPS (Operations Per Second)   

    -- So let's understand this with an example of any app like Ola or Uber or Rapido

            So in this we see driver's live location so this will be implement by driver vehicle sending GPS to server and then 
            server sending to client's device

            Let's assume the frequency is 1 signal/sec so if we assume that 1 lakh users request the server for live locations per 
            second then the throughput of live location system should be high or it might crash 

            Even if we do sharding and replication still if in a particular region there are many users still might crash 

            And also it will store it in DB also to perform some calculations like time taken for driver to reach customer, journey 
            time, journey fare, etc so without DB having throughput it might crash

            Even if we use best of the best DB still it might crash due to too many requests coming towards it

            So ola doing this across India and as for Uber it works worldwide so there will be too much load on their servers 

            So every DB that exists will not have such a high throughput 

    -- So these problems are solved by our messaging queues



*** Kafka ***

    -- How Kafka solves the above problem ?

            -- So to solve above problem we just need a high throughput device so that device is Kafka

            -- Kafka is a messaging queue (MQ), that have a very high throughput 

            -- So now between Ola or Uber server and DB we will put kafka so that Ola or Uber server hits kafka now

            -- Since kafka has a very high throughput so it can process many requests from the server 

            -- So there will be workers introduced between kafka and DB now so workers pick data from kafka and give it to DB

            -- So Ola or Uber server is a producer as Ola or Uber server is trying to push many requests to Kafka and kafka can 
                handle that amount of requests

            -- Workers are the consumers, they have low throughput so workers take requests from Kafka and then when they have 
                reached their limit they will just in bulk give all the requests to the DB

            -- Worker is just another server or can say microservice whose task is to take data from kafka to DB

            -- So kafka acts as an intermediate between high throughput and low throughput devices


    -- Why Kafka is useful ?

        1   -- Pace Matching : Producer might produce at a very high speed whereas consumer might work slow so it helps in matching 
                                their speeds as producer will just produce at very high speeds and puts data in kafka and consumer
                                will consume data from kafka at it's own pace

        2   -- Asynchronous Communication : It can also be used to implement asynchronous communication between different systems
                                            (Like used in Lecture-7 for Notification System)


*** In-depth Architecture of Kafka ***

    Terms :

            (=) Cluster

            (=) Broker

            (=) Topic

            (=) Partition

            (=) Offset

            (=) Zookeeper

            (=) Consumer

            (=) Consumer Group

            (=) Producer

    
    -- Basic architecture will be same i.e. there will be a producer and a consumer and kafka

    -- Broker is nothing but a kafka server 

    -- Inside kafka there are multiple topics (topics can be thought of like whatsapp groups)

    -- There can be many types of notifications like in Ola it can be of rider locations, notifications, etc so they are just 
        divided into topics like rider locations will go to topic 0, notifications go to topic 1 and so on

        If understand with another example like notification system even in notification system if we send notification to IOS 
        devices we use APNs so before we discussed we introduced messaging queues here so on IOS device notifications can be 
        of types like discount deals, hot products, sales, etc so they are kept in different topics to differentiate between 
        them hence they can be done like in topic 0 discount deals, topic 1 hot products, topic 2 sales like this

    -- Inside topic there are partitions like partition 0, partition 1, etc, each topic has indexing starting from 0 for partitions

    -- Partition is like an actual queue inside which the message is stored so the producer puts message inside partitions 

    -- Inside partitions there are offsets (offset can be considered like indexes in an array like how we can element is there at
        index 0 similarly in this case we can say message is at offset 0)

    -- So inside offset the message is really stored 


    *** Now we will introduce Producer-Consumer Interaction ***

        -- So producer will put the message inside a topic in a partition 

        -- So if we say that in topic 0 the messages are of rider location and topic 1 has notifications so based on this messages
            will be distributed among topics and then inside a topic the message is put inside a partition and the way partition is 
            selected will be discussed later 

        -- So let's assume for now that a producer produced a message m1 which is stored inside offset 0 of partition 0 of topic 0
            similarly producer produces message m2 which is stored inside offset 1 of partition 0 of topic 0

            So consumer will first consume m1 then will consume m2 
        
    
    -- No consumer can exist without consumer group, a consumer can exist only if they are part of the consumer group and there can
        be many consumer groups 

    -- So now let's for now assume that there is one broker and one topic and one partition so when producer produces message m1
        it goes in partition of the topic so now what broker does is that it does *** Auto-scaling *** so it sees that there is only
        one consumer and one partition so it assigns that partition to that consumer only so now the consumer will consume all the 
        message in that partition

    -- So now if we increase number of partitions in the topic then also all will be assigned to same consumer only as there is 
        only one consumer for now in the consumer group so broker assigns all the partitions to the consumer 

    -- So now if we say there are 2 consumers instead of one consumer then the broker's auto-scaling will just divide the partitions
        so that almost equal amount of partitions are allocated to each consumer 

        Let's assume there is one topic inside a broker topic 0 and there are p0, p1, p2, p3 partitions in the topic and there are 
        two consumer c1 and c2 that belong to the same consumer group 

        So broker's auto-scaling will divide i.e. data in p0 and p1 will be consumed by c1 and data in p2 and p3 will be consumed by
        c2

        So now if we take above case and introduce one more partition p4 then how will the partitions will be distributed among c1
        and c2 so data in p4 will be either consumed by c1 or c2 it depends on broker whom he is giving priority so let's assume
        data in p4 is consumed by c2

        So now if we introduce one more partition p5 then it will be allocated to c1 as to make the number of partitions allocated
        to each consumer equal 

    -- So if we again redefine our problem and assume that for topic 0 there is only one partition p0 but now there are 2 consumers 
        c1 and c2 then the broker's auto-scaling will allocate p0 to any one of the consumer only, in other words it will never 
        allocate same partition to 2 or more consumers (*** Important Point ***)

        So from this point we can conclude that if number of partitions are less than number of consumers then some consumers will 
        remain idle 

    -- So 1 consumer can consume multiple partitions but one partition can be consumed by only one consumer 

    -- So above conclusion is incomplete so the completed version of above conclusion is that 1 consumer can consume multiple 
        partitions but one partition can be consumed by only one consumer from same consumer group 

        So if we take above case again with 2 consumer groups this time, consumer group 1 has 2 consumers c1 and c2 and consumer 
        group 2 also has 2 consumers c1 and c2 so now the partition p0 will be allocated to 1 consumer only but from each consumer
        group

        So let's assume data in p0 is consumed by c1 in consumer group 1 and c2 in consumer group 2 so in each group one consumer is
        sitting idle 

        So now if we introduce partition p1 then it will be allocated to c2 in consumer group 1 and c1 in consumer group 2


    *** Why kafka does all this (why they made complex system just for using it as messaging queue) ? ***

            -- So this consumer group concept is fairly new in kafka as before there were no consumer group concept in kafka or can
                say there were only 1 consumer group and a partition can be consumed by only 1 consumer

            -- So concept of consumer groups were introduced as messaging queues serve 2 purpose :

                            (i) Queue (FIFO)

                                    -- So before when messaging queues concept was there so it was like there will be a producer and
                                        a consumer and there will be a normal queue between them (Dequeue)

                                    -- So producer produces messages and puts it in queue and consumer consumes data from queue 

                                    -- So in normal queue only one producer and one consumer were allowed

                            
                            (ii) Pub/Sub Model (Publisher/Subscriber Model)

                                    -- So in this there was one publisher allowed but now in this multiple subscribers (consumers) 
                                        were allowed to consume data from special type of queue 

                                    -- So now in this publisher will put data in queue and we call this queue as broadcasting queue
                                        from which multiple subscriber consume data 


            -- So before messaging queues were made from the concept of queues but then they were later made from the pub/sub model

            -- So it was done as if we assume a youtube channel so the creator of youtube channel is the publisher and all of us 
                are the subscriber that see his content so one publisher can broadcast a single message to many subscriber like when
                they release new video so notification goes to each subscriber that they have posted new video

            -- So in this case we will be needed to move towards the pub/sub model as we have many consumer that need to consume 
                same data published by the publisher 

            -- So the queue (FIFO) is also used in cases like in Ola ride booking the rider's location will be shared to only one
                consumer so normal queue will be used there 

            -- So basically for 1:1 mapping we use normal queue model whereas for 1:n mapping we use pub/sub model

            -- So now as we know Kafka was invented by LinkedIn so before kafka was invented as a producer-consumer problem (queue 
                [FIFO]) but later it was discovered that pub-sub model was also needed so this model was also introduced 

            -- So using only one consumer group the queue (FIFO) usecase was satisfied but they needed to also integrate the pub/sub
                model also so they introduced the concept of consumer group

            -- So kafka also uses another thing called Zookeeper, zookeeper is a 3rd party service (zookeeper exists independently)

            -- Kafka uses zookeeper as zookeeper keeps all the information about the consumer as to which consumer group it belongs,
                which topic is it listening to, which partition is it listening to and currently which offset the consumer is 
                consuming so it keeps tracks of all this and updates data

            -- So why is there a need for zookeeper just to store this information, zookeeper is needed as in cases if let's say 
                our consumer c1 was consuming offset 0 from partition 0 from topic 0 so if our c1 goes down then the broker's 
                auto-scaling will give these partitions to other consumer let's say consumer c2 got this partition so now c2
                won't know that from where it needs to continue consuming offset from so either the broker shares this information
                with c2 by asking zookeeper or c2 asks zookeeper himself 

            -- So zookeeper will keep track of all the topics, offset, partition and consumers

            -- So now if one of our broker goes down as broker is nothing but a server then the data kept in the broker won't be 
                lost as we have multiple brokers 

            -- Cluster is a group of brokers 

            -- So in the end kafka is a service and to provide service to users it will need multiple server (called as brokers)

            -- So one kafka service is deployed on multiple brokers so that in case one broker is down then there will be multiple
                brokers for data replication 

            -- Similarly to how we scale server using load balancer and many server same is the case with kafka

            -- So if one broker goes down now then multiple broker will be available in which data will be replicated

            -- So now data is replicated by let's assume that there is topic 0 in broker b1, b2, b3 so partition p0 for topic 0 will
                be stored in b1 then for p1 for topic 0 it will be stored in b2 and then p2 for topic 0 will be stored in b3

            -- So now load is distributed among 3 brokers b1, b2 and b3

            -- So topic is same across all brokers but partitions of same topic are distributed among brokers but there is still a 
                problem that if b1 goes down then p0 will be lost for topic 0

            -- So to solve this problem we just need to do replication now (similar to how DB partioning and DB replication was 
                done)

            -- So in this there's a leader-follower concept (same as master-slave) so now b1's p0 will be leader and it's copy will
                be in b2 also which will be follower similarly b2's p1 will be leader and it's copy will be stored in b3 also which
                will be follower and so on 

            -- So now let's understand in this architecture how read and write operations are being performed :

                    -- So in this leader is responsible for both read and write operations whereas followers will just follow 
                        (listen to the leader)

                    -- So we can think of followers as synchronized back up copies so that in case if they lost leader then it can 
                        act as leader 

            -- Before zookeeper service was used but now kraft service is used instead of zookeeper to keep track 
                







