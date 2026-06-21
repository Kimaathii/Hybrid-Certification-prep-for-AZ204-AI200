# Scenario Cards: Azure Database for PostgreSQL & pgvector

These scenarios simulate real-world consulting problems and mimic the format of AZ-204 and AI-200 exam case studies.

---

### SCENARIO 1: The E-Commerce Migration (Beginner)
**Business Context:** "BookBazaar" is an independent online bookstore currently running on a physical server in the owner's basement.
**The Problem:** During holiday sales, the server frequently crashes. The owner spends hours every week applying Linux patches and manually copying backups to a USB drive. 
**Constraints:** 
1. The new solution must use their existing relational database engine (PostgreSQL).
2. The owner must not manage any underlying operating systems.
3. The database must automatically backup data for up to 14 days.
**Your Task:** Recommend the appropriate Azure database service to solve this problem.
**Hints:** 
- Think about the difference between IaaS (Virtual Machines) and PaaS (Managed Services).
- Since it's a relational database, NoSQL (Cosmos DB) is not a good fit.
**Solution:** Recommend **Azure Database for PostgreSQL - Flexible Server**. It is a fully managed platform-as-a-service (PaaS). Microsoft handles all OS patching, hardware maintenance, and provides automated point-in-time backups (which can be configured for 14 days of retention). This allows the bookstore owner to focus entirely on selling books rather than IT administration.
**Exam Connection:** AZ-204 will ask you to choose between Cosmos DB, SQL VM, and PostgreSQL Flexible Server based on whether the data is relational and whether the customer wants to manage the OS.

---

### SCENARIO 2: The Black Friday Reporting Crisis (Intermediate)
**Business Context:** "ElectroWorld" is a massive electronics retailer running their web application on PostgreSQL Flexible Server.
**The Problem:** Every hour, their internal analytics team runs a complex SQL query to generate sales reports. While this query is running, the main customer-facing website becomes incredibly slow, causing customers to abandon their shopping carts. 
**Constraints:** 
1. You cannot change the structure of the database or rewrite the queries.
2. The solution must ensure the customer-facing website remains fast.
3. The reporting team must still have access to near real-time data.
**Your Task:** Design an architectural change to handle the reporting workload without impacting customer sales.
**Hints:** 
- Scaling up (adding more CPU) might help, but it doesn't isolate the problem.
- Is the reporting team inserting data, or just reading it?
**Solution:** Implement a **Read Replica**. A read replica creates a synchronized, read-only clone of the primary database. You will then instruct the internal analytics team to point their reporting application's connection string exclusively to the Read Replica. This offloads the intensive read queries, freeing up 100% of the primary server's compute power to handle customer purchases (writes).
**Exam Connection:** Offloading read-heavy workloads (like reporting or dashboarding) to Read Replicas is a highly tested concept on AZ-204.

---

### SCENARIO 3: Securing the Vault (Intermediate)
**Business Context:** "MediCare App" stores highly sensitive patient health records.
**The Problem:** The Chief Security Officer (CSO) is terrified of credentials leaking. Currently, the Azure App Service (the web server) connects to the PostgreSQL database using a username and password stored in the application's configuration settings.
**Constraints:** 
1. The web application must continue to read and write to the database.
2. There must be zero passwords or secrets stored anywhere in the application code or configuration.
3. The database should only allow connections from the web app.
**Your Task:** Configure the authentication mechanism between the App Service and PostgreSQL to satisfy the CSO.
**Hints:** 
- What Azure service acts like an "electronic ID badge" for applications?
- Think beyond Key Vault, as the constraint says *no* secrets stored *anywhere*.
**Solution:** Enable **Microsoft Entra ID Authentication** on the PostgreSQL Flexible Server. Then, assign a **System-Assigned Managed Identity** to the Azure App Service. Configure PostgreSQL to map that specific Managed Identity to a database user role. The App Service will now securely request temporary access tokens from Azure automatically, completely eliminating the need for hardcoded passwords.
**Exam Connection:** AZ-204 loves asking for the "most secure" way to authenticate between Azure resources. Managed Identity + Entra ID is almost always the correct answer.

---

### SCENARIO 4: The Smart Real Estate Agent (Intermediate / AI-200)
**Business Context:** "PropTech" has a web portal with 50,000 real estate listings stored in a PostgreSQL database. 
**The Problem:** Users currently search by exact terms (e.g., "3 bedroom pool"). PropTech wants to allow users to search using natural language (e.g., "A cozy place for a large family that loves swimming").
**Constraints:** 
1. They want to use OpenAI to generate text embeddings for every property description.
2. They do not have the budget to purchase and manage a separate standalone Vector Database.
3. The search must be integrated directly into their existing database.
**Your Task:** Detail the steps required to implement this semantic search within their current infrastructure.
**Hints:** 
- How can PostgreSQL understand AI embeddings?
- What specific operator compares the "meaning" of two vectors?
**Solution:** First, run `CREATE EXTENSION vector;` on their existing Flexible Server database. Second, add a new column to the property table using the `vector(1536)` data type (the dimension size for OpenAI ada-002). Third, use an Azure OpenAI API to convert all descriptions into vectors and save them in the new column. Finally, when a user searches, convert the query into a vector and use the **Cosine Similarity operator (`<=>`)** in a SQL `ORDER BY` clause to find the closest semantic matches.
**Exam Connection:** AI-200 tests the integration of vector stores into existing relational systems. Knowing `CREATE EXTENSION vector` and the `<=>` operator is critical.

---

### SCENARIO 5: The Million-Vector Bottleneck (Advanced / AI-200)
**Business Context:** "GlobalNews" is an international media conglomerate that has successfully implemented pgvector. They have 15 million news articles stored as vectors.
**The Problem:** The semantic search feature was fast when they had 10,000 articles, but now that they have 15 million, every single search takes over 10 seconds. The database compute is maxing out trying to calculate cosine similarity across millions of rows.
**Constraints:** 
1. Search query times must drop below 100 milliseconds.
2. They are willing to sacrifice a tiny percentage of perfect accuracy (recall) to achieve this speed.
3. They have plenty of RAM (Memory-Optimized tier) and prioritize speed over memory consumption.
**Your Task:** Select and implement the correct vector index to solve the performance issue.
**Hints:** 
- You need an Approximate Nearest Neighbor (ANN) index.
- Which index type is best when memory is not an issue but speed and accuracy are paramount?
**Solution:** Implement an **HNSW (Hierarchical Navigable Small World)** index on the vector column. Without an index, PostgreSQL does an "exact nearest neighbor" search—calculating the math against all 15 million rows. HNSW creates a graph structure that allows the database to skip irrelevant vectors and quickly navigate to the closest matches. Because they have plenty of RAM and prioritize high recall/speed, HNSW is the superior choice over the cluster-based IVFFlat index.
**Exam Connection:** On AI-200, if a scenario asks to optimize pgvector search speed at massive scale while maximizing recall (and memory is available), HNSW is the answer. If they specify strict memory limitations, the answer shifts to IVFFlat.
