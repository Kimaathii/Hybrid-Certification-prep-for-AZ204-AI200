# Module 7: Azure Managed Redis Quiz

## Section 1: Foundation Questions (Concepts & Definitions)

**1. You are explaining the purpose of Azure Cache for Redis to a new developer. Which statement best describes the primary function of Redis in a web application architecture?**
A) It is a relational database designed to replace Azure SQL.
B) It is an in-memory data store used to improve application performance by caching frequently accessed data.
C) It is a durable, long-term storage solution for archive data.
D) It is an identity provider used to authenticate web users.

**Answers:**
- **A is incorrect:** Redis is a NoSQL, key-value store, not a relational database, and it sits in front of SQL, not replacing it.
- **B is CORRECT:** Redis stores data in RAM (in-memory) for sub-millisecond retrieval, dramatically speeding up read-heavy applications.
- **C is incorrect:** Because RAM is volatile and expensive, Redis is terrible for long-term archival storage.
- **D is incorrect:** Azure Entra ID (Active Directory) is an identity provider; Redis does not handle authentication.

---

**2. Your application is adding data to Redis, but the server's RAM is 100% full. You have configured the eviction policy to `allkeys-lru`. What will Redis do?**
A) Reject the new data and throw an out-of-memory error.
B) Delete the oldest data that was inserted into the cache.
C) Delete the data that has not been read or accessed in the longest amount of time.
D) Automatically upgrade your pricing tier to add more RAM.

**Answers:**
- **A is incorrect:** This describes the `noeviction` policy.
- **B is incorrect:** LRU stands for "Least Recently Used," not "First In First Out" (FIFO).
- **C is CORRECT:** `allkeys-lru` throws away the data that hasn't been accessed recently, making room for the new data.
- **D is incorrect:** Azure Redis never automatically upgrades your pricing tier.

---

**3. You need to ensure that cached data automatically deletes itself after 30 minutes so that users do not see stale prices. Which Redis feature should you use?**
A) Eviction Policy
B) Time-To-Live (TTL)
C) Data Persistence
D) Cache-Aside

**Answers:**
- **A is incorrect:** Eviction policies only trigger when the cache is 100% full; they don't operate on a countdown timer.
- **B is CORRECT:** Setting a TTL (Time-To-Live) attaches a countdown timer to a key, automatically deleting it when time expires.
- **C is incorrect:** Data Persistence saves data to a hard drive; it doesn't delete it.
- **D is incorrect:** Cache-Aside is an application code pattern, not a Redis feature for expiring data.

---

**4. You want to implement real-time chat in your application. One user sends a message, and it needs to instantly broadcast to 50 other users. Which Redis capability is designed for this?**
A) Distributed Locking
B) Pub/Sub
C) Session State
D) RediSearch

**Answers:**
- **A is incorrect:** Distributed locks prevent multiple servers from doing the same task.
- **B is CORRECT:** Publish/Subscribe (Pub/Sub) allows one client to publish a message to a channel, and all clients subscribed to that channel receive it instantly.
- **C is incorrect:** Session state stores individual user profiles across web requests.
- **D is incorrect:** RediSearch is for AI vector querying.

---

**5. You are writing a .NET application that needs to connect to Azure Redis. Which standard NuGet package should you install?**
A) Microsoft.Data.SqlClient
B) Azure.Storage.Blobs
C) StackExchange.Redis
D) EntityFrameworkCore

**Answers:**
- **A is incorrect:** This is for connecting to Azure SQL.
- **B is incorrect:** This is for Azure Blob Storage.
- **C is CORRECT:** `StackExchange.Redis` is the industry-standard and Microsoft-recommended client library for .NET.
- **D is incorrect:** Entity Framework is an ORM for relational databases.

---

## Section 2: Applied Questions (Implementation & Scenarios)

**6. You are implementing the Cache-Aside pattern in C#. A user requests the price of "Product A". The data is NOT found in Redis (a Cache Miss). What is the very next step your application code must take?**
A) Return an error to the user.
B) Query the primary database (e.g., Azure SQL) for the price.
C) Write a blank value to Redis to prevent future errors.
D) Wait for Redis to automatically sync with the primary database.

**Answers:**
- **A is incorrect:** A cache miss is normal; the app should seamlessly handle it without the user knowing.
- **B is CORRECT:** In Cache-Aside, the application is responsible for falling back to the primary database when the cache is empty.
- **C is incorrect:** Writing a blank value (cache poisoning) doesn't solve the user's request.
- **D is incorrect:** Redis does not automatically talk to SQL; the application handles all communication.

---

**7. Your security team requires that your Azure Cache for Redis instance must be deployed completely inside a private Virtual Network (VNet) and cannot have a public IP address. Which pricing tier is the MINIMUM you can select?**
A) Basic
B) Standard
C) Premium
D) Enterprise

**Answers:**
- **A is incorrect:** Basic is on shared infrastructure and cannot be injected into a VNet.
- **B is incorrect:** Standard is also on shared infrastructure.
- **C is CORRECT:** Premium is the lowest tier that supports VNet integration (placing the cache in a private subnet).
- **D is incorrect:** While Enterprise supports this, it is not the *minimum* required tier.

---

**8. (Select all that apply) You have a fleet of 10 stateless web servers behind a load balancer. Users are complaining that their shopping carts randomly empty while browsing. You decide to use Redis for Session State. What are the benefits of this approach?**
A) The shopping cart data is persisted permanently even if the user deletes their account.
B) Any web server can instantly access the user's cart.
C) The web servers do not need to use their own local RAM to store user sessions.
D) Redis automatically encrypts the shopping cart data using Azure Key Vault.

**Answers:**
- **A is incorrect:** Session state is temporary (usually tied to a TTL); it is not permanent storage.
- **B is CORRECT:** Because Redis is centralized, the load balancer can send the user to any server, and that server can fetch the cart from Redis.
- **C is CORRECT:** Offloading session state frees up local RAM on the web servers, allowing them to handle more traffic.
- **D is incorrect:** Redis does not automatically encrypt data via Key Vault out of the box.

---

**9. You are building an AI chatbot using Azure OpenAI. The chatbot needs to search through 10,000 corporate documents by "meaning" rather than exact keywords. You plan to use Redis. Which specific module must you use?**
A) RedisJSON
B) RediSearch
C) RedisBloom
D) RedisTimeSeries

**Answers:**
- **A is incorrect:** RedisJSON stores JSON documents but cannot perform mathematical vector searches on its own.
- **B is CORRECT:** RediSearch is the module that adds Vector Database capabilities (like K-Nearest Neighbor searches) to Redis.
- **C is incorrect:** RedisBloom is for probabilistic data structures.
- **D is incorrect:** RedisTimeSeries is for IoT metric tracking.

---

**10. Following up on the previous question regarding the AI Chatbot and the RediSearch module. Which Azure Redis pricing tier MUST you select to use this capability?**
A) Basic
B) Standard
C) Premium
D) Enterprise

**Answers:**
- **A is incorrect:** Basic does not support modules.
- **B is incorrect:** Standard does not support modules.
- **C is incorrect:** Premium supports persistence and VNets, but not the RediSearch module.
- **D is CORRECT:** Advanced Redis modules (like RediSearch, RedisJSON) are only available on the Enterprise tier, which runs specialized Redis Labs software.

---

## Section 3: Exam-Level Questions (Complex Scenarios)

**11. A legacy application uses the Write-Through caching pattern. You are tasked with migrating it to the Cache-Aside pattern. What is the primary operational difference the application will experience after the migration?**
A) Data writes to the database will take longer to complete.
B) The very first time a specific piece of data is requested, the read operation will be slower than subsequent reads.
C) The Redis cache will fill up much faster and require more RAM.
D) The Redis cache will automatically update itself whenever the SQL database changes.

**Answers:**
- **A is incorrect:** Writes actually become faster in Cache-Aside because you only write to the DB, not both DB and Cache simultaneously.
- **B is CORRECT:** In Cache-Aside (Lazy Loading), data is only cached *after* it is requested. The first request causes a Cache Miss (slow), which populates the cache for future requests (fast).
- **C is incorrect:** Cache-Aside uses *less* RAM because it only caches data that users actually ask for.
- **D is incorrect:** Redis never automatically syncs with SQL.

---

**12. (Select all that apply) You are deploying an Azure Cache for Redis instance (Premium Tier). You want to ensure that if the entire Azure region experiences a power failure and all servers reboot, the data in your cache is not lost. Which two configurations are valid ways to achieve this?**
A) Enable RDB (Redis Database) persistence.
B) Enable AOF (Append Only File) persistence.
C) Set the eviction policy to `noeviction`.
D) Use a `TimeSpan.MaxValue` for all TTLs.

**Answers:**
- **A is CORRECT:** RDB persistence takes a periodic snapshot of the RAM and saves it to an Azure Storage account.
- **B is CORRECT:** AOF persistence logs every single write operation to an Azure Storage account, allowing the cache to rebuild itself perfectly on reboot.
- **C is incorrect:** Eviction policies handle full memory; they do not save data to a hard drive.
- **D is incorrect:** TTLs control when data expires in RAM; they do not protect against server reboots.

---

**13. You have an AI application that generates an OpenAI vector embedding representing a user's search query. You are using the `NRedisStack` SDK in C# to query an Enterprise Redis cache. Which syntax conceptually represents how you execute a vector search?**
A) `db.StringGetAsync("vector:searchQuery")`
B) `db.FT().Search("myIndex", "*=>[KNN 5 @vector_field $query]")`
C) `db.Execute("SELECT TOP 5 FROM Redis WHERE Vector = query")`
D) `db.ListRightPopAsync("vectorList")`

**Answers:**
- **A is incorrect:** `StringGet` is for simple key-value lookups, not mathematical vector comparisons.
- **B is CORRECT:** `FT.SEARCH` is the specific RediSearch command used via `NRedisStack` to perform a K-Nearest Neighbor (KNN) semantic search.
- **C is incorrect:** Redis is NoSQL and does not use SQL SELECT syntax.
- **D is incorrect:** Popping from a list removes an item from a queue; it does not perform a search.

---

**14. You have a background worker process that reads orders from an Azure Service Bus and updates an inventory database. You want to scale this out to 5 worker VMs. However, you must ensure that two VMs do not process the exact same order simultaneously. How can Azure Redis solve this?**
A) Use the `StackExchange.Redis` Pub/Sub feature to announce when an order is finished.
B) Store the Session State of the worker VMs in a Premium tier cache.
C) Implement a Distributed Lock in Redis, where a VM must acquire the lock for a specific Order ID before processing it.
D) Use the RediSearch module to find matching orders across the 5 VMs.

**Answers:**
- **A is incorrect:** Pub/Sub is for broadcasting; it doesn't prevent two listeners from acting on the broadcast simultaneously.
- **B is incorrect:** Session State is for web users, not for controlling background job execution.
- **C is CORRECT:** A Distributed Lock ensures that only one process (out of many) can hold the "token" (the lock) at any given time, preventing duplicate processing.
- **D is incorrect:** RediSearch is for AI and text searching, not concurrency control.

---

**15. Your team deployed a Standard tier Azure Cache for Redis. You notice the application occasionally throws a `RedisConnectionException` stating "Timeout performing GET". What is the most common architectural cause for this timeout when the cache is under heavy load?**
A) The RediSearch module is using too much CPU.
B) The ThreadPool in your .NET application is exhausted and cannot handle the asynchronous Redis responses.
C) The Redis cache has automatically switched to a Basic tier to save money.
D) The Azure SQL database is taking too long to respond to Redis.

**Answers:**
- **A is incorrect:** RediSearch cannot be installed on a Standard tier.
- **B is CORRECT:** In high-traffic .NET applications, if the ThreadPool isn't configured correctly, it bottlenecks and causes timeouts waiting for the network response from Redis. This is a very common real-world and exam scenario.
- **C is incorrect:** Tiers never change automatically.
- **D is incorrect:** Azure SQL does not communicate with Redis directly.
