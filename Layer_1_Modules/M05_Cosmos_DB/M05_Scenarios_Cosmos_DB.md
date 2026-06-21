# Scenario Cards: Cosmos DB (Web + AI)

## SCENARIO 1: The Social Media Feed
**Difficulty:** Beginner

**Business Context:** "PicShare" is a global social media platform where users upload photos and like other users' photos.
**The Problem:** The database costs are skyrocketing because they provisioned 100,000 RU/s to handle unpredictable weekend traffic spikes. Their data read speeds are also suffering globally.
**Constraints:** 
1. The solution must reduce costs for unpredictable traffic.
2. Users must immediately see the "Like" button turn blue when they click it.
3. It is acceptable if other users don't see the new "Like" count for a few seconds.
**Your Task:** Choose the correct Cosmos DB billing model and the correct Consistency Level.
**Hints:** 
- What billing model is best for traffic that spikes wildly and drops to zero?
- What consistency level ensures "read-your-own-writes"?
**Solution:** You should configure the Cosmos DB account to use **Serverless** billing. This means they only pay for the exact RUs consumed during the weekend spikes, eliminating the high fixed hourly cost. For consistency, you should choose **Session Consistency**. This is the default level and guarantees that the user who clicked "Like" will immediately see their action reflected (read-your-own-writes), while providing fast eventual consistency for the rest of the world.
**Exam Connection:** (AZ-204) You will be given a scenario requiring a balance of low latency, read-your-own-writes, and high availability. The answer is always Session Consistency.

---

## SCENARIO 2: The E-Commerce Hot Partition
**Difficulty:** Intermediate

**Business Context:** "GlobalShop" is an e-commerce platform hosting storefronts for multiple different companies (Tenants). 
**The Problem:** During Black Friday, the application crashed with a `429 Too Many Requests` error. They have a massive container storing all orders. The partition key is set to `/tenantId`. 
**Constraints:**
1. You must resolve the `429` errors.
2. The solution must allow the database to scale horizontally infinitely.
3. You cannot increase the total Provisioned RU/s for the entire database.
**Your Task:** Identify why the database is failing and propose a new partition key design.
**Hints:**
- What happens if 90% of the Black Friday traffic is for one specific tenant?
- How many physical servers is that traffic hitting?
**Solution:** The database is experiencing a **Hot Partition**. Because the partition key is `/tenantId`, all orders for the most popular tenant are being routed to a single logical partition, which maps to a single physical server. That server maxes out its RU capacity and returns `429` errors, while other servers are idle. You must choose a partition key with higher cardinality. Changing the partition key to `/orderId` (or a synthetic key like `/tenantId_date`) will evenly distribute the writes across all physical servers, preventing the bottleneck.
**Exam Connection:** (AZ-204) Hot partition questions are highly common. The fix always involves choosing a property with a wider range of distinct values (high cardinality).

---

## SCENARIO 3: The Receipt Emailer
**Difficulty:** Intermediate

**Business Context:** "TicketMastery" sells concert tickets globally.
**The Problem:** Every time a new ticket is written to the Cosmos DB `Tickets` container, an email receipt must be sent to the user. Currently, they have a background worker polling the database every 10 seconds asking `SELECT * FROM Tickets WHERE emailSent = false`. This is consuming massive amounts of RUs and slowing down the main app.
**Constraints:**
1. The solution must not poll the database.
2. The email must be sent almost instantly after the document is created.
3. The solution must be fully serverless.
**Your Task:** Design an event-driven architecture to send the emails.
**Hints:**
- Does Cosmos DB have a built-in feature that acts like a news ticker for document changes?
- What serverless compute service can listen to that feature?
**Solution:** You should implement the **Cosmos DB Change Feed** and connect it to an **Azure Function**. The Change Feed acts as a persistent log of all inserts and updates to the container. By creating an Azure Function with a Cosmos DB Trigger, the function will be invoked automatically the millisecond a new ticket is inserted. The function will receive the ticket JSON, send the email via SendGrid, and complete. This eliminates polling, saves RUs, and is entirely serverless.
**Exam Connection:** (AZ-204) Whenever you need to trigger a downstream action (cache update, email, event hub) based on a Cosmos DB insert/update, the answer is always the Change Feed.

---

## SCENARIO 4: The AI Corporate Chatbot (RAG Pattern)
**Difficulty:** Intermediate

**Business Context:** "TechCorp" has 10,000 internal HR documents (PDFs, Word docs). They want an AI chatbot that answers employee questions using strictly these documents.
**The Problem:** They are using Azure OpenAI, but the AI doesn't know about the private HR documents. They need a place to store the documents so the AI can search them "by meaning" (e.g., matching the question "How do I take time off?" to the document "PTO Policy").
**Constraints:**
1. The solution must use Cosmos DB for NoSQL.
2. You must search by conceptual meaning, not exact keyword matches.
**Your Task:** Describe the architecture to store and query the HR data for the AI.
**Hints:**
- What do we convert text into so that AI can understand it mathematically?
- What specific function in Cosmos DB compares these mathematical representations?
**Solution:** This is the Retrieval-Augmented Generation (RAG) pattern. First, TechCorp must convert their HR documents into **Vector Embeddings** (arrays of floats) using an AI embedding model. These embeddings are stored in Cosmos DB JSON documents alongside the text. When an employee asks a question, the app converts the question into a vector. It then queries Cosmos DB using the `VectorDistance()` SQL function to find the HR documents with the vectors most mathematically similar to the question's vector. The app then sends those matched documents to Azure OpenAI to formulate the final answer.
**Exam Connection:** (AI-200) Understanding the flow of Vector Search in a RAG pattern (Embed -> Store -> Embed Query -> VectorDistance -> Generate) is critical.

---

## SCENARIO 5: Optimizing Massive Vector Searches
**Difficulty:** Advanced

**Business Context:** "MediSearch" provides an AI search engine across 50 million medical research papers. 
**The Problem:** They have implemented Cosmos DB Vector Search, but queries are taking 5-10 seconds. They are currently using a Flat (Brute Force) vector indexing policy, which guarantees perfect accuracy but is too slow.
**Constraints:**
1. Query latency must drop below 100 milliseconds.
2. The dataset will grow to 100 million documents soon.
3. A slight reduction in absolute perfect accuracy (recall) is acceptable in exchange for massive speed gains.
**Your Task:** Reconfigure the Cosmos DB Vector Index.
**Hints:**
- What Microsoft-developed vector index is optimized for massive scale and low latency?
**Solution:** You must change the vector indexing policy on the Cosmos DB container from Flat (exact match) to **DiskANN**. The Flat index performs a mathematically exact comparison of the search vector against all 50 million documents (k-nearest neighbors), which is why it is slow. DiskANN is an Approximate Nearest Neighbor (ANN) algorithm developed by Microsoft Research. It creates a highly optimized graph structure that allows Cosmos DB to search tens of millions of vectors in milliseconds, trading a microscopic amount of accuracy for incredible speed.
**Exam Connection:** (AI-200) You must know the difference between vector indexes. Flat = slow/perfect accuracy. IVF = fast/clusters. DiskANN = Microsoft's highly optimized index for massive scale.
