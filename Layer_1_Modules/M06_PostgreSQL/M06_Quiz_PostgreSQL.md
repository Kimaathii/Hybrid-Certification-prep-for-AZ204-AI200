# Module 6 Quiz: Azure Database for PostgreSQL & pgvector

This quiz contains 15 questions designed to test your knowledge of both standard Azure database administration (AZ-204) and AI vector data management (AI-200). 

---

## Part 1: Foundation Questions

**1. A developer is new to Azure and needs to deploy a relational database without managing the underlying operating system. Which service should they choose?**
A) SQL Server on Azure Virtual Machines
B) Azure Cosmos DB
C) Azure Database for PostgreSQL - Flexible Server
D) Azure Blob Storage

**Answer: C**
- **A is incorrect** because running SQL Server on a Virtual Machine (IaaS) requires you to manage and patch the underlying Windows or Linux OS.
- **B is incorrect** because Cosmos DB is primarily a NoSQL database, not a relational database.
- **C is correct** because Flexible Server is a fully managed PaaS relational database where Azure handles OS maintenance.
- **D is incorrect** because Blob Storage is for unstructured files (like images), not relational tables.

**2. What is the primary purpose of enabling High Availability (HA) on an Azure PostgreSQL server?**
A) To backup data to prevent accidental deletion by developers.
B) To automatically increase disk storage when the database is full.
C) To keep the database online during a hardware or datacenter failure.
D) To make SELECT queries run faster.

**Answer: C**
- **A is incorrect.** HA is not a backup. Accidental deletions will replicate immediately to the HA standby server.
- **B is incorrect.** That is the purpose of Storage Auto-Growth, not HA.
- **C is correct.** HA provides a standby server that automatically takes over if the primary infrastructure fails.
- **D is incorrect.** HA does not improve query performance; Read Replicas do.

**3. Which SQL command is required to turn a standard PostgreSQL database into a vector database capable of AI semantic search?**
A) `INSTALL PLUGIN ai_search;`
B) `CREATE EXTENSION vector;`
C) `ALTER DATABASE ENABLE AI;`
D) `CREATE TABLE vectors;`

**Answer: B**
- **A is incorrect.** This is not valid PostgreSQL syntax.
- **B is correct.** pgvector is an open-source extension, and `CREATE EXTENSION vector;` is the standard command to enable it.
- **C is incorrect.** This command does not exist.
- **D is incorrect.** Creating a table named "vectors" does not add the mathematical capabilities needed to process embeddings.

**4. When comparing two AI embeddings for text semantic similarity, which mathematical operator is considered the industry standard in pgvector?**
A) `<->` (Euclidean distance)
B) `<=>` (Cosine similarity)
C) `<#>` (Inner product)
D) `==` (Exact match)

**Answer: B**
- **A is incorrect.** Euclidean measures straight-line distance, which is better for spatial data, not text meaning.
- **B is correct.** Cosine similarity measures the angle between vectors, making it highly effective for text embeddings from models like OpenAI.
- **C is incorrect.** Inner product is useful for certain recommendation systems but is not the standard for general text similarity unless vectors are strictly normalized.
- **D is incorrect.** `==` checks if two vectors are perfectly identical, which defeats the purpose of "similar" AI search.

**5. (Select All That Apply) Which of the following are characteristics of PostgreSQL compared to Cosmos DB (NoSQL)?**
A) Uses rigid schemas
B) Stores data in JSON documents
C) Highly optimized for complex JOIN operations
D) Designed primarily for schema-less data

**Answer: A, C**
- **A is correct.** Relational databases require you to define exactly what columns exist before inserting data.
- **B is incorrect.** Cosmos DB NoSQL stores JSON documents; PostgreSQL stores rows and columns.
- **C is correct.** PostgreSQL excels at connecting different tables together using JOINs.
- **D is incorrect.** Cosmos DB is designed for schema-less, unstructured data.

---

## Part 2: Applied Questions

**6. Your company deployed an Azure PostgreSQL Flexible server. Users are reporting that the database crashed with a "Disk Full" error. How can you prevent this from happening again with zero downtime?**
A) Upgrade the compute tier from Burstable to General Purpose.
B) Deploy a Read Replica.
C) Enable Storage Auto-Growth.
D) Configure Zone-Redundant High Availability.

**Answer: C**
- **A is incorrect.** Upgrading compute adds CPU and RAM, but does not add storage space, and it requires a server restart.
- **B is incorrect.** Read replicas help with CPU load from read queries, not storage limits.
- **C is correct.** Storage Auto-Growth automatically expands disk space behind the scenes before it fills up, preventing crashes with no downtime.
- **D is incorrect.** HA protects against hardware failure, but if your disk is full, the standby server's disk will also be full.

**7. A financial application generates heavy end-of-day reports using massive SQL queries. During this time, the database CPU hits 100%, and normal users cannot input new transactions. What architectural change should you apply?**
A) Route the reporting application to a newly created Read Replica.
B) Route the reporting application to a Same-Zone HA Standby server.
C) Enable VNet injection.
D) Change the vector index from HNSW to IVFFlat.

**Answer: A**
- **A is correct.** A Read Replica is an asynchronous copy used specifically to offload heavy read/reporting workloads from the primary server, freeing it up for user transactions.
- **B is incorrect.** HA standby servers cannot be queried directly; they only sit idle waiting for a failover.
- **C is incorrect.** VNet injection is a security feature, not a performance scaling feature.
- **D is incorrect.** Vector indexes apply to AI searches, not standard financial SQL queries.

**8. You need to ensure your PostgreSQL database is completely isolated from the public internet. No public IP address should exist for the server. What must you configure?**
A) IP Firewall rules
B) SSL/TLS enforcement
C) VNet Injection
D) Microsoft Entra ID Authentication

**Answer: C**
- **A is incorrect.** IP Firewall rules block unauthorized public IPs, but the server still technically has a public endpoint exposed to the internet.
- **B is incorrect.** SSL encrypts data in transit but does not hide the server from the public internet.
- **C is correct.** VNet Injection places the database inside an Azure Virtual Network with a private IP, completely removing public internet access.
- **D is incorrect.** Entra ID manages who can log in, not network visibility.

**9. Your web application uses OpenAI's `text-embedding-ada-002` model to generate embeddings for user reviews. You are creating the table in PostgreSQL to store these embeddings. Which data type definition is correct?**
A) `vector(100)`
B) `varchar(max)`
C) `vector(1536)`
D) `jsonb`

**Answer: C**
- **A is incorrect.** 100 dimensions will not fit the output of the ada-002 model.
- **B is incorrect.** Storing vectors as plain text prevents you from using mathematical operators like cosine similarity.
- **C is correct.** The ada-002 model outputs exactly 1536 dimensions. Therefore, the column must be defined as `vector(1536)`.
- **D is incorrect.** While JSON can store arrays, the `jsonb` type does not support pgvector's AI indexing and distance operators.

**10. (Select All That Apply) What are the benefits of using Microsoft Entra ID Authentication with PostgreSQL instead of standard SQL passwords?**
A) It eliminates the need to store passwords in application code.
B) It makes database queries run faster.
C) You can use Managed Identities to securely authorize Azure App Services.
D) It automatically enables Zone-Redundant HA.

**Answer: A, C**
- **A is correct.** Entra ID relies on identity tokens, meaning no passwords exist to be hardcoded or leaked.
- **B is incorrect.** Authentication methods do not impact query performance.
- **C is correct.** Managed Identities act as an electronic badge for Azure resources, seamlessly integrating with Entra ID.
- **D is incorrect.** Authentication has nothing to do with infrastructure availability.

---

## Part 3: Exam-Level Scenarios

**11. Scenario:** You are managing a product catalog database with 10 million vectors using pgvector. The application's search feature is too slow. You need to implement an index. Your primary requirement is the highest possible query speed and recall (accuracy). Your database runs on the Memory Optimized tier, so RAM usage is not a concern.
**Question:** Which index type should you create?
A) B-Tree
B) IVFFlat
C) HNSW
D) Hash

**Answer: C**
- **A is incorrect.** B-Tree is for standard relational data (like numbers and text), not multidimensional vectors.
- **B is incorrect.** IVFFlat uses less memory but sacrifices recall accuracy and query speed at massive scale.
- **C is correct.** HNSW (Hierarchical Navigable Small World) provides the best query performance and recall for vector searches, though it consumes more memory (which the scenario explicitly stated was not a concern).
- **D is incorrect.** Hash indexes check for exact equality, which is useless for semantic similarity.

**12. Scenario:** An e-commerce app experiences a 500% increase in traffic every November. Currently, the PostgreSQL Flexible server is on the Burstable compute tier. You anticipate heavy read AND write traffic (users purchasing items).
**Question:** What is the best scaling strategy to prepare for this traffic spike?
A) Add a Read Replica.
B) Scale up the compute tier to General Purpose before November begins.
C) Enable Storage Auto-Growth.
D) Implement an HNSW index on all text columns.

**Answer: B**
- **A is incorrect.** A Read Replica only helps with read traffic. It cannot handle the increase in writes (purchases).
- **B is correct.** Scaling up (Vertical Scaling) adds more CPU and RAM to the primary server, allowing it to handle higher volumes of both read and write operations.
- **C is incorrect.** Auto-Growth helps with disk space, not compute bottlenecks caused by high traffic.
- **D is incorrect.** HNSW is for AI vector data, not standard text columns.

**13. Scenario:** A junior database administrator accidentally runs a `DROP TABLE customers;` command, deleting all user data. The database is configured with Zone-Redundant High Availability.
**Question:** What will happen, and how should you recover the data?
A) The standby server will reject the `DROP` command, protecting the data. You don't need to do anything.
B) The standby server automatically restores the table from a backup.
C) The `DROP` command is replicated to the standby server instantly. You must use Point-in-Time Restore from Automated Backups to recover the data.
D) The data is permanently lost because HA overwrites backups.

**Answer: C**
- **A is incorrect.** HA standby servers faithfully execute whatever commands the primary server executes, including destructive ones.
- **B is incorrect.** HA servers do not auto-restore backups.
- **C is correct.** HA does not protect against human error. The deletion immediately syncs to the standby. You must use Azure's automated backups to restore the database to the exact minute before the command was run.
- **D is incorrect.** Azure automatically keeps backups for up to 35 days precisely for this reason.

**14. Scenario:** You are migrating an on-premises web application to Azure. The application's data is highly structured, relies on strictly enforced schemas, and utilizes complex JOIN operations across dozens of tables. The application does not require global distribution.
**Question:** Which Azure service is the most appropriate target for this migration?
A) Azure Table Storage
B) Azure Cosmos DB
C) Azure Database for PostgreSQL - Flexible Server
D) Azure Cache for Redis

**Answer: C**
- **A is incorrect.** Table storage is a simple key-value store, not suited for complex JOINs.
- **B is incorrect.** Cosmos DB is a NoSQL database designed for unstructured data and global scale. Migrating a strictly relational app with complex JOINs to Cosmos DB would require a total rewrite of the app.
- **C is correct.** PostgreSQL is a powerful relational engine designed exactly for rigid schemas and complex JOINs.
- **D is incorrect.** Redis is an in-memory cache, not a persistent primary database for an application.

**15. (Select All That Apply) Scenario:** You are configuring a brand new PostgreSQL Flexible Server to handle a semantic search workload for an AI application. What three steps are mandatory to successfully return AI search results?
A) Execute `CREATE EXTENSION vector;`
B) Define a table column with the `vector` data type.
C) Execute `CREATE EXTENSION ai_search;`
D) Order the SELECT query using a distance operator like `<=>`.
E) Deploy the server in a Zone-Redundant configuration.

**Answer: A, B, D**
- **A is correct.** The pgvector extension must be explicitly enabled in the database.
- **B is correct.** You must create a column designed to hold mathematical arrays (e.g., `vector(1536)`).
- **C is incorrect.** This is a made-up command.
- **D is correct.** To perform semantic search, you must calculate the distance between the query vector and the stored vectors using an operator like `<=>`.
- **E is incorrect.** While Zone-Redundant HA is a good practice for reliability, it has nothing to do with enabling AI search capabilities.
