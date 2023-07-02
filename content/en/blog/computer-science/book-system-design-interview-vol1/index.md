---
title: '[Book] System Design Interview Vol. 1'
date: 2023-07-02T22:00:00+09:00
weight: 16
categories:
  - Book
  - System Design
image: preview.png
---
## Overview of the book

**★★★★★ - Highly recommended for students who have wondered the architecture of large scaled systems or who are preparing for the first job as a software engineer.
Engineers who are currently working in related fields can also review overall concepts about system architectures!**

## Personal impressions of the book

- I was able to review the basic concepts of computer science. But, at the same time, it was a bit boring.
- The same systems and concepts like notification systems, caching, sharding and message queses were introduced and referred repeatedly, so I was able to understand easily and reuse the same system when imagining another case in this book.
    - If I were reading a random article in the internet, it might be somewhat ambiguous what the systems and concepts exactly are. For example, I might have a question like "What is the notification service like mentioned in this article exactly?"
    - However, in the book, the ambiguity was removed by reusing the similar notification system.
- I didn't understand why the virtual interviewee asked certain questions, such as the following:
  e.g., inquiring "Is internationalization required?" after learning that uploading and watching videos are key features.
  
  e.g., inquiring "How many friends can a user have?" even though the number is not the main focus.
    
- It was new to me to summarize the range of typical cases for metrics related to latency and data size in the field of computer science.

## Summaries of each chapter

### Chapter 1: Scale From Zero To Millions Of Users

- Covered basic computer science concepts:
    - Database, scalability, cache, CDN, stateless vs stateful, datacenter, message queue, log and metric

### Chapter 2: Back-of-the-envelope Estimation

- Fostered an intuition the metric related to latency and data size in computer science
    - Which tasks should be handled in nanoseconds
    - Which tasks should be handled in milliseconds

### Chapter 3: A Framework For System Design Interviews

- Covered interviews themselves
- Explained the 4 steps
    1. Understanding the problems and define the range of architecture
    2. Suggesting a rough architecture and negotiating
    3. Detailed architecture
    4. Wrapping up

### Chapter 4: Design A Rate Limiter

- Covered rate limiting algorithms: Token buckets, limiting buckets, fixed window counter, moving window log, moving window counter
- Was able to review the concepts even though I forgot them after a few days.
- Learned about the use of `X-Ratelmit-*` headers, which was new to me.

### Chapter 5: Design Consistent Hashing

- Explained the concepts of consistent hashing
- The explanation in the book seemed challenging. I recommend other articles in the internet.
    - I referenced https://dev.to/karanpratapsingh/system-design-consistent-hashing-1dbl
      ```
      R = K / N
      ```
      Where,
      
      `R`: Data that would require re-distribution.
      
      `K`: Number of partition keys.
      
      `N`: Number of nodes.
      
      In short, with consistent hashing with virtual nodes, data needs to be re-distributed regresses to  `total data / the number of nodes`
      
      - e.g., Let's say there is 300 data and 3 nodes(i.e., 1(1~100), 2(101~200), 3(201~300)) and we'll add 1 node.
          - with consistent hashing → 1(1~75), 2(101~175), 3(201~275), 4(76~100, 176~200, 276~300)
              - 1 → 4: 25
              - 2 → 4: 25
              - 3 → 4: 25
              - Only `75 (= 300 / 4)` data is re-distributed!  
          - without consistent hashing,  → 1(1~75), 2(76~150), 3(151~225), 4(226~300)
              - 1 → 2: 25
              - 2 → 3: 50
              - 3 → 4: 75
              - Half data(`150/300`) is re-distributed

### Chapter 6: Design A Key-value Store

- Covered CAP theorem - There are 2 kinds of distributed systems., CP and AP
    - CP - On network partition, forgo availability for consistency
    - AP - On network partition, forgo compatibility and run the system, there can be data collisions.
    - CA - Network partitioning is an inevitable being. so we must tolerate network partition.
    - Trade-offs between replication for availability and consistency
    - Introduced consistency models: strong consistency, weak consistency, eventual consistency
    - A solution to inconsistency: versioning data
    - Heartbeats and gossip protocol
    - Merkle tree for rebalancing when there is lots of replicated data.
- Too many concepts and techniques were introduced here.

### Chapter 7: Design A Unique Id Generator In Distributed Systems

- From this part, the book got more interesting. The following parts are relevant to my previous experiences and curiosities.
- To generate unique ids
    - Multi-master replication (SQL)
        - Ensures an increase within each shard
        - Orders across all shards are not guaranteed.
    - UUID
        - Too long(128bit)
        - Cannot sort by time
        - ID can be non-numeric
    - A ticket server to issue unique IDs
        - A ticket server is a dedicated server to issue unique IDs but this is a SPOF and which is hard to be replicated due to the synchronization and inconsistency.
    - Twitter snowflakes
        - A kind of the method of "divide and conquer". This method divided an ID into several parts
        - <timestamp, datacenter id, server id, sequence>
            - timestamp - A timestamp after start time defined by twitter
            - sequence - An incremental number within the current millisecond.
        - Can sort by timestamp within same data center
        - Within the same datacenter and the same server, even though the timestamps between multiple data are same, we can differentiate each data by the sequences.

### Chapter 8: Design A Url Shortener

- From this part, the book mentioned Bloom filter a lot.
- I think I call this a microservice. This was fun.
- The table structure of database was so simple. but the service demanded high performance.
    - Table structure: \<ID\>, \<Short URL\>, \<long URL\>

### Chapter 9: Design A Web Crawler

- Nothing especially interesting
- Mentioned Bloom filter repeatedly
- Politeness was fun
- Checksums to filter duplicated contents

### Chapter 10: Design A Notification System

- Interested a lot because I was in a notification system team when I worked for Karrot as a Go server engineer internship.
- There was a notification platform like this system, I had some questions like ‘is this right..? Do we really need a system like this?". But the similar system is introduced in this book!
- Message queues, worker patterns, queue monitoring and rate limiting
- Using templates for contents of notifications (e.g, `New products for {{ USERNAME }}: {{[product.name for product in PRODUCTS]}}!`)
- Logging and retrying in case of loss (in case of errors, we can go over the logs)
- Security(Auth) between clients and the notification system. Only authorized clients can send requests to the notification system using api key, app secret, jwt token or mTLS.

### Chapter 11: Design A News Feed System

- Which I didn't understand when I first came across the same pattern(push on write) in Redis in Action.
    
    When someone writes a twit, the timelines of the followers will be updated by pushing the information about the new twit(fan out to followers) to the ZSETs of each of them. So their timelines kept up-to-date. This is called as "push on write". 
    
- Initially, I didn't fully resonate with the system because of the inefficiency from muted followers and dormant users. But this book let me know about the techniques to filter them beforehand. They were filtering and "pulling on write"
- Comparison between "push on write" and "pull on read" based on pros and cons
- Caching was also actively used here.

### Chapter 12: Design A Chat System

- Chat services have a high demand for writing and servers should be able to notify clients about new messages
- Preserving connections is efficient in case of high demand for writing. It reduces the overhead of creating a new connection every time.
- Polling: after preserving a connection, we can poll the new data periodically from the servers.
- Websocket: Furthermore, we can also utilize WebSocket.
    - Once a connection is formed, the protocol is upgraded from simple http to websocket.
    - Services to maintain connections alive tend to be stateful services.
- The book said the table structures of the database differ slightly depending on their purpose, which means they **vary** based on
  whether they're designed for 1 on 1 chat or group chat. In the case of group chat, the group id column is added    
  
  ⬆️ I didn't fully agree with this part, because we can just treat 1 on 1 chat and group chat the same way in terms of the table structures.
    
  According to the pattern described in this book for handling a case with fewer users in the same chat room, there could be a substantial number of message queues, as it creates a message queue for each user or device.
    
  At this part, I was concerned about the possibility of having an excessive number of queues when using Kafka or any other message queues.
    

### Chapter 13: Design A Search Autocomplete System

- Mainly focused on utilizing trie which is a tree-based data structure.
- It was impressive how important a data structure became when there was a significant amount of data.
- I couldn't imagine the magnitude of data stored in huge IT services like Google, Instagram, Slack and so on.
- How to shard data
    - From starting sharding from \<a~z\>
    - To sharding from \<aa~zz\>
- Browers(client-side) caching

### Chapter 14: Design Youtube

- Glad to come across a topic that I had worked on when I was a university student and server engineer working for a small strartup.
- Video transcoding - adaptive streaming like HLS (HTTP Live Streaming), Bitrates
    - Managed service examples on AWS I know: AWS Elemental MediaConvert, AWS Elastic Trasncoder
- Presigned URL: For users to upload their video to the cloud storages directly and for servers not to proxy the large objects to the cloud storage
- As we did in SNS, we can apply the same way to handle popular videos and other videos differently
    - If it's a popular video, we'd better make use of CDN.
    - If it's an unpopular video, serving the video through the server itself would be rather efficient.

### Chapter 15: Design Google Drive

- Liked the sequence diagram illustrating the sequence of uploading new file and notifying the change of states. My company uses similar sequence diagram when we illustrate some sequences
- This book also unquestionably advocated the use of S3 as a vast storage system.
- Compressing, archiving and caching were mentioned as well.

### Chapter 16: The Learning Continues

- Nothing special

## Conclusions

This book has been one of the most popular books between Korean software engineers. I've wanted to read this book and I finally read it 🙂.

At the beginning of this book, honestly, I felt like boring. However, from the middle of the book, I was able to be more intrigued because
there were contents relevant to my previous experiences and curiosities which I've had since I was in university.

I agree that this book is not the only option to retrieve the contents like architecting large-scaled systems in this book. We can find them in the internet as well.
But I liked that I could relate the same knowledge I just learned at the previous chapter at the right next chapter. Thanks to it, I was able to take in the knowledge more easily.  

