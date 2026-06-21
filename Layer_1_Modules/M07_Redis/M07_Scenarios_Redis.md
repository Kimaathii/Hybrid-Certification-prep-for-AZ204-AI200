# Module 7: Scenario Cards

## Scenario 1: The Crashing Catalog (Difficulty: Beginner)

**Business Context:** "ShoeUniverse" is a rapidly growing e-commerce store. They have a massive product catalog stored in an Azure SQL database.

**The Problem:** During a recent Black Friday sale, thousands of users hit the website simultaneously. Every user's page load triggered a query to the Azure SQL database to get the price of popular shoes. The database hit 100% CPU utilization and the website crashed.

**Constraints:**
1. You must reduce the CPU load on the primary Azure SQL Database.
2. The solution must keep costs as low as possible.
3. If the solution fails temporarily, the website must still function, just slower.

**Your Task:** Recommend an architecture addition and a specific code pattern to solve this problem.

**Hints:**
- Think about the "whiteboard" analogy.
- Which code pattern ensures the application can still fetch data if the whiteboard is erased?

**Solution:** You should deploy **Azure Cache for Redis (Standard Tier)** and implement the **Cache-Aside pattern**. 
By putting Redis in front of Azure SQL, the first user who views a shoe pulls the data from SQL and writes it to Redis. The next 9,999 users pull the price instantly from Redis, completely bypassing the SQL database and saving CPU. Because we use Cache-Aside, if Redis goes offline, the code will just experience a "cache miss" and fall back to querying the SQL database directly, ensuring the website stays online.

**Exam Connection:** AZ-204 often tests your knowledge of the Cache-Aside pattern as a fallback mechanism. The application *always* queries the cache first, and falls back to the database on a miss.

---

## Scenario 2: The Amnesic Servers (Difficulty: Intermediate)

**Business Context:** "StreamFlix" is a video streaming platform. They have a web application running on a cluster of 5 virtual machines behind an Azure Load Balancer.

**The Problem:** Users are reporting a frustrating bug. They log in, add three movies to their "Watchlist," and then click "Checkout." Suddenly, the website says they are logged out, and their Watchlist is empty. 

**Constraints:**
1. The web application must remain stateless so virtual machines can be added or removed easily.
2. The user's Watchlist must be remembered regardless of which virtual machine they communicate with.
3. The solution must provide sub-millisecond response times so page loads aren't delayed.

**Your Task:** Identify why this is happening and recommend a solution.

**Hints:**
- Look up what a Load Balancer does to user traffic.
- Where should you store data that belongs to a single user's current session?

**Solution:** The problem is that the user's "Session State" is being stored in the local RAM of individual virtual machines. The Load Balancer sends the user to VM 1 (where they log in), then VM 3 (where they add a movie), then VM 2 (where the server has no idea who they are).
The solution is to use **Azure Cache for Redis for Session State Provider**. Instead of storing the Watchlist in the local VM memory, all 5 VMs are configured to read and write Session State to a single, centralized Redis Cache. Now, no matter which VM the user hits, the VM instantly pulls their state from Redis.

**Exam Connection:** AZ-204 tests your ability to choose Redis as a "Distributed Session State" provider when using stateless web servers.

---

## Scenario 3: Secure the Cache (Difficulty: Intermediate)

**Business Context:** "FinBank" is a financial institution migrating their monolithic application to Azure. They process sensitive, real-time stock pricing data.

**The Problem:** FinBank wants to use Azure Redis to cache the stock prices to improve dashboard load times. However, the security team has blocked the deployment.

**Constraints:**
1. The Redis Cache must *not* be accessible over the public internet.
2. The Redis Cache must be injected directly into FinBank's private Azure Virtual Network (VNet).
3. The cache must survive a complete server reboot (Data Persistence).

**Your Task:** Select the appropriate Azure Redis pricing tier and explain your choice.

**Hints:**
- Basic and Standard tiers share public infrastructure.
- Which tier acts like a dedicated, enterprise-grade server?

**Solution:** You must select the **Premium Tier** of Azure Cache for Redis. 
The Basic and Standard tiers are multi-tenant and do not support VNet injection or Data Persistence. The Premium tier allows you to deploy the cache entirely within a private subnet (VNet integration), satisfying the security team's requirement that no public IP exists. Furthermore, Premium supports Data Persistence (RDB or AOF), which periodically writes the contents of RAM to an Azure Storage account, ensuring data survives a reboot.

**Exam Connection:** Knowing exactly which features unlock at the **Premium** tier (VNet, Persistence, Geo-replication) is a guaranteed exam question.

---

## Scenario 4: The Chatty Dashboard (Difficulty: Intermediate)

**Business Context:** "LogisticsPro" tracks thousands of delivery trucks across the country. 

**The Problem:** Their dispatchers look at a live dashboard in their web browser. Currently, the dashboard asks the server "Are there any new updates?" every 5 seconds (polling). With thousands of trucks and hundreds of dispatchers, this polling is overwhelming the server with useless requests when trucks haven't moved.

**Constraints:**
1. Polling must be eliminated.
2. When a truck moves, the server must push the update instantly to all connected dispatchers.
3. The system must be able to handle thousands of messages per second with minimal latency.

**Your Task:** Which specific feature of Redis should be used to solve this?

**Hints:**
- You need a way to broadcast a message.
- Think of a radio station.

**Solution:** You should use the **Redis Pub/Sub (Publish/Subscribe)** feature. 
Instead of the dashboard polling the server, the server simply "Subscribes" to a Redis channel called `TruckUpdates`. When a truck's GPS changes, the backend system "Publishes" a tiny message to the `TruckUpdates` channel. Redis instantly routes that message to all subscribers, allowing the dashboard to update in real-time without constant polling.

**Exam Connection:** If an exam scenario mentions "real-time chat," "live dashboards," or "broadcasting messages instantly," the correct Redis feature is Pub/Sub.

---

## Scenario 5: Semantic Product Search (Difficulty: Advanced / AI-200)

**Business Context:** "TechGadgets" has thousands of product manuals. They are building an AI chatbot to help customers troubleshoot issues.

**The Problem:** Traditional keyword search is failing. A customer searches for "screen cracked," but the manual uses the phrase "display fractured." The chatbot returns zero results. They have used Azure OpenAI to generate vector embeddings (arrays of numbers) for all their manuals to enable semantic search, but they need a place to store and query these vectors instantly.

**Constraints:**
1. You must store vector embeddings of the product manuals.
2. You must be able to perform K-Nearest Neighbor (KNN) searches in single-digit milliseconds to keep the chatbot responsive.
3. You must use Azure Managed Redis.

**Your Task:** Specify the exact Redis Tier and the Redis Module required to accomplish this.

**Hints:**
- AI vectors are an advanced data type.
- Standard Redis only knows text and numbers, not vector math.

**Solution:** You must deploy Azure Cache for Redis on the **Enterprise Tier** and enable the **RediSearch** module.
Vector search requires complex mathematical calculations to find similarity (Cosine distance). The RediSearch module adds these capabilities to Redis. Because RediSearch is a specialized module, Azure only allows it to be installed on the Enterprise tier. Once deployed, the AI embeddings can be stored as Vector fields inside Redis Hashes, allowing the chatbot to instantly find "display fractured" when the user searches "screen cracked."

**Exam Connection:** For AI-200 and modern architectures, knowing that Vector capabilities require **RediSearch + Enterprise Tier** is critical. You will be tested on the difference between standard caching and vector storage.
