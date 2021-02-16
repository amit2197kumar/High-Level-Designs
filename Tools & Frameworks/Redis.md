# Redis

## What is Redis?

Redis, which stands for Remote Dictionary Server, is a fast, open-source, in-memory key-value data store for use as a database, cache, message broker, and queue. Redis delivers sub-millisecond response times enabling millions of requests per second for real-time applications in Gaming, Ad-Tech, Financial Services, Healthcare, and IoT. Redis is a popular choice for caching, session management, gaming, leaderboards, real-time analytics, geospatial, ride-hailing, chat/messaging, media streaming, and pub/sub apps.

## How does Redis work?

All Redis data resides in-memory, in contrast to databases that store data on disk or SSDs. By eliminating the need to access disks, in-memory data stores such as Redis avoid seek time delays and can access data in microseconds. Redis features on-disk persistence in batch, in in certain periods.

## Benefits of Redis:

1. In-memory data store
2. Flexible data structures
3. Simplicity and ease-of-use
4. Replication and persistence
5. High availability and scalability
6. Extensibility

### 1. In-memory data store

All Redis data resides in the server’s main memory. In comparison to traditional disk based databases where most operations require a roundtrip to disk, in-memory data stores such as Redis don’t suffer the same penalty. The result is – blazing fast performance with average read or write operations taking less than a millisecond and support for millions of operations per second.

### 2. Flexible data structures

Unlike simplistic key-value data stores that offer limited data structures, Redis has a vast variety of data structures to meet your application needs. Redis data types include:

- Strings – text or binary data up to 512MB in size
- Lists – a collection of Strings in the order they were added
- Sets – an unordered collection of strings with the ability to intersect, union, and diff other Set types
- Sorted Sets – Sets ordered by a value
- Hashes – a data structure for storing a list of fields and values
- Bitmaps – a data type that offers bit level operations
- HyperLogLogs – a probabilistic data structure to estimate the unique items in a data set

### 3. Simplicity and ease-of-use

Redis simplifies your code by enabling you to write fewer lines of code to store, access, and use data in your applications. For example, if your application has data stored in a hashmap, and you want to store that data in a data store – you can simply use the Redis hash data structure to store the data. A similar task on a data store with no hash data structures would require many lines of code to convert from one format to another. Redis comes with native data structures and many options to manipulate and interact with your data. Over a hundred open source clients are available for Redis developers. Supported languages include Java, Python, PHP, C, C++, C#, JavaScript, Node.js, Ruby, R, Go and many others.

### 4. Replication and persistence

Redis employs a primary-replica architecture and supports asynchronous replication where data can be replicated to multiple replica servers. This provides improved read performance (as requests can be split among the servers) and faster recovery when the primary server experiences an outage. For persistence, Redis supports point-in-time backups (copying the Redis data set to disk).

### 5. High availability and scalability

Redis offers a primary-replica architecture in a single node primary or a clustered topology. This allows you to build highly available solutions providing consistent performance and reliability. When you need to adjust your cluster size, various options to scale up and scale in or out are also available. This allows your cluster to grow with your demands.

### 6. Extensibility

Redis is an open source project supported by a vibrant community. There’s no vendor or technology lock in as Redis is open standards based, supports open data formats, and features a rich set of clients.

## Popular Redis Use Cases

### **Caching**

Redis is a great choice for implementing a highly available in-memory cache to decrease data access latency, increase throughput, and ease the load off your relational or NoSQL database and application. Redis can serve frequently requested items at sub-millisecond response times, and enables you to easily scale for higher loads without growing the costlier backend. Database query results caching, persistent session caching, web page caching, and caching of frequently used objects such as images, files, and metadata are all popular examples of caching with Redis.

Read More here: [Popular Redis Use Cases](https://aws.amazon.com/redis/#Popular-Redis-Use-Cases)

# Reference:

1. [Introduction to Redis - redis.io](https://redis.io/topics/introduction)
2. [Redis - AWS](https://aws.amazon.com/redis/)
3. [Sample Redis Commands](https://codeburst.io/redis-what-and-why-d52b6829813) 
4. [15 Interesting Facts About Redis](https://dltlabs.medium.com/15-interesting-facts-about-redis-45e7e47e1a3b)
5. [Why Redis?](https://youtu.be/OG610oe_kxs)
6. [Redis system design | Distributed cache System design](https://youtu.be/DUbEgNw-F9c)