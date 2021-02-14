# Designing a URL Shortening service like TinyURL

## Questions to ask from the interviewer:

1. Can CX pick any of his own custom tiny URLs?
2. What should be length of the tiny URL hash? (For this discussion lets take an assumption of 6 character long)
3. Will there be any expiration for thr tiny URL? If yes, Can customer set that? what will be default time of expiration?
4. Ask about traffic estimates? Approximately how many **save/write calls** (tinyURL) and **read** **calls** (redirect) calls are we expecting?
5. Is it mandatory to have CX account to make tinyURL or anyone can use the service?
6. Can CX delete any tiny URL whenever he wants?

## Requirements at one place:

### Functional:

1. CX enter a large URL, need to return a tiny unique URL.
2. CX access the tiny URL, redirect CX to the original large URL endpoint.
3. CX can pick costume tiny URL hash.
4. CX can add expiration time of URL (Optional), else default expiration date would set. 
5. CX should be able to delete his tinyURL stored in system.

### Non-Functional:

1. System should be highly available.
2. System should be highly scalable.
3. Tiny URL redirect should have minimal latency.
4. Need to check for CX abuse (Ton of tiny URL generation requests by same use in short period of time)

## Start the design part from dev end:

Understand the read-write paradigm of system : Our system will be read-heavy. There will be lots of redirection requests compared to new URL shortenings. **Let’s assume a 100:1 ratio between read and write.**

**Traffic estimates:** Assuming, we will have 500M new URL shortenings per month, with 100:1 read/write ratio, we can expect 50B redirections during the same period:

100 * 500M => 50B

What would be Queries Per Second (QPS) for our system? New URLs shortenings per second:

500 million / (30 days * 24 hours * 3600 seconds) = ~200 URLs/s

Considering 100:1 read/write ratio, URLs redirections per second will be:

100 * 200 URLs/s = 20K/s

Since we have 20K requests per second, we will be getting 1.7 billion requests per day:

20K * 3600 seconds * 24 hours = ~1.7 billion

### DataBase Design:

No relation between any long URL with any other long URL, or any user to user relation. As we need to scale rapidly, NoSQL can be a better choice over SQL. Can use DynamoDB.

Read:

1. [Why are noSQL databases more scalable than SQL?](https://softwareengineering.stackexchange.com/questions/194340/why-are-nosql-databases-more-scalable-than-sql)
2. [Why NoSQL is better at “scaling out” than RDBMS?](https://stackoverflow.com/questions/8729779/why-nosql-is-better-at-scaling-out-than-rdbms)

Remember NoSQL will give High availability  & Easy Scaling at the cost of Eventual Consistency. (CAP Theorem)

Note : Our System is READ heavy.

Table : User

- UserId (Primary Key)
- Name
- Email
- CreatedDate
- LastLogin

Table : URL

- tinyURL [Hash] (Primary Key)
- Original Large URL
- Created Date
- Expiration Date
- UserId (Can created GSI over this, to get/access all tiny URL made by this CX)(Secondary Key)

### System APIs:

**createUser(userId, name, email)**

- CX Account  creation. All field are mandatory.

**createLongToTinyURL(userId, longURL, customHash, expirationDate)**

- If anyone can make the URL userId will be optional field.
- customHash & expirationDate will be always optional fields.

**deleteURL(userId, tinyURL)**

- userId ca be optional field.

**getLongURL(tinyURL)**

To prevent CX abuse we can mandate in our system that an UserId can generate an **x** amount of TinyURL in last 60 seconds. 

### LongURL to TinyURL Logic:

We can compute a unique hash (e.g., [MD5](https://en.wikipedia.org/wiki/MD5) or [SHA256](https://en.wikipedia.org/wiki/SHA-2), etc.) of the given URL. The hash can then be encoded for display. This encoding could be base36 ([a-z ,0-9]) or base62 ([A-Z, a-z, 0-9]) and if we add ‘+’ and ‘/’ we can use [Base64](https://en.wikipedia.org/wiki/Base64#Base64_table) encoding.

Using base64 encoding, a 6 letters long key would result in 64^6 = ~68.7 billion possible strings.

Using base64 encoding, an 8 letters long key would result in 64^8 = ~281 trillion possible strings.

**Why base62 or base64 been used, why not base10?**

Base62 uses : [A - Z] 26, [a - z] 26, [0 - 9] 10 = 62 characters ~ 62^7 = 3.5 Trillions combinations 

Base10 uses : 10^7 = 10 Million combinations

Read about Base 62

- [Base 62 text encoding/decoding](https://medium.com/analytics-vidhya/base-62-text-encoding-decoding-b43921c7a954)
- [Base62.java](https://gist.github.com/jdcrensh/4670128)

[How Base64 works?](https://youtu.be/aUdKd0IFl34)

Interviewer might ask about 2 use case:

1. CX1 is logged-In and with same LongURL had called **createLongToTinyURL()** multiple times, what to do in that case?
2. Multiple CX users trying same LongURL to converted to TinyURL. Will we have the same TinyURL for all the CX requests ?? what id one then called the **deleteURL()**?

**Workaround for the issues:** We can append an increasing sequence number to each input URL to make it unique and then generate its hash. We don’t need to store this sequence number in the databases, though. That sequence number can be current milliseconds time in epoch.

Example : 

1. Long URL : [https://www.linkedin.com/in/amit2197kumar/](https://www.linkedin.com/in/amit2197kumar/)
2. Increasing sequence number: 1613284115000
3. Use base64 encoding for : https://www.linkedin.com/in/amit2197kumar/1613284115000

If due to high traffic we are not able to find unique Increasing sequence number, we will retry that again. Though it's very corner case to happen. 

![](/Images/TinyURL01.png)

We can have a standalone **Key Generation Service (KGS)**

**Isn’t KGS a single point of failure?** 

Yes, it is. To solve this, we can have a standby replica of KGS. Whenever the primary server dies, the standby server can take over to generate and provide keys.

**How would we perform a key lookup?** 

We can look up the key in our database to get the full URL. If it’s present in the DB, issue an “HTTP 302 Redirect” status back to the browser, passing the stored URL in the “Location” field of the request. If that key is not present in our system, issue an “HTTP 404 Not Found” status or redirect the user back to the homepage.

![](/Images/TinyURL02.png)

### Cache:

We can cache URLs that are frequently accessed. Before hitting backend storage, the application servers can quickly check if the cache has the desired URL.

**Memory estimates:** If we want to cache some of the hot URLs that are frequently accessed, how much memory will we need to store them? If we follow the 80-20 rule, meaning 20% of URLs generate 80% of traffic, we would like to cache these 20% hot URLs.

**Which cache eviction policy would best fit our needs?** 

When the cache is full, and we want to replace a link with a newer/hotter URL, how would we choose? Least Recently Used (LRU) can be a reasonable policy for our system.

To further increase the efficiency, we can replicate our caching servers to distribute the load between them.

**How can each cache replica be updated?** 

Whenever there is a cache miss, our servers would be hitting a backend database. Whenever this happens, we can update the cache and pass the new entry to all the cache replicas. Each replica can update its cache by adding the new entry. If a replica already has that entry, it can simply ignore it.

![](/Images/TinyURL03.png)

### Load Balancer (LB):

We can add a Load balancing layer at three places in our system:

1. Between Clients and Application servers
2. Between Application Servers and database servers
3. Between Application Servers and Cache servers

We could use a simple Round Robin approach that distributes incoming requests equally among backend servers.

### Purging or DB cleanup:

Should entries stick around forever, or should they be purged? If a user-specified expiration time is reached, what should happen to the link?

**If we chose to actively search for expired links to remove them, it would put a lot of pressure on our database**. Instead, we can slowly remove expired links and do a lazy cleanup.

What all option we have:

1. [Expiring Items By Using DynamoDB Time to Live (TTL)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)
2. Whenever a user tries to access an expired link, we can delete the link and return an error to the user.
3. A separate Cleanup service can run periodically to remove expired links from our storage and cache. This service should be very lightweight and can be scheduled to run only when the user traffic is expected to be low.

![](/Images/TinyURL04.png)

![](/Images/TinyURL05.jpeg)

## Reference:

1. [URL shortener system design | tinyurl system design](https://www.youtube.com/watch?v=JQDHz72OA3c)
2. [Designing a URL Shortening service like TinyURL](https://www.educative.io/courses/grokking-the-system-design-interview/m2ygV4E81AR)
3. [Base64 Encoding](https://youtu.be/aUdKd0IFl34)
4. [Why are noSQL databases more scalable than SQL?](https://softwareengineering.stackexchange.com/questions/194340/why-are-nosql-databases-more-scalable-than-sql)
5. [Why NoSQL is better at “scaling out” than RDBMS?](https://stackoverflow.com/questions/8729779/why-nosql-is-better-at-scaling-out-than-rdbms)
6. [Coding for base 62 in tinyURL](https://www.geeksforgeeks.org/how-to-design-a-tiny-url-or-url-shortener/)
7. [Expiring Items By Using DynamoDB Time to Live (TTL)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)
8. [About Redis](https://aws.amazon.com/redis/)
9. [Redis: What and Why?](https://codeburst.io/redis-what-and-why-d52b6829813)
