# EP05 — Practice Exam 2: Exam-Realistic Difficulty (Hybrid AZ-204 + AI-200)

**Questions:** 50 | **Difficulty:** Intermediate 30% + Exam-level 70% | **Time:** 120 minutes

**Characteristics:**
- Scenario-based architecture questions
- Multiple constraints per question
- Focus on AZ-204 Core + AI-200 Infrastructure (pgvector, Cosmos DB, Azure OpenAI integration)

---

## Part 1: Scenario-Based Architecture

**1. A company needs to deploy a microservices application that consists of a frontend web app and a backend API. The application requires:**
- **Zero downtime deployments**
- **Autoscaling based on HTTP queue length**
- **Outbound access to a private SQL database in an Azure VNet**
**Which Azure App Service plan is the most cost-effective tier that meets ALL requirements?**
A. Basic (B1)
B. Standard (S1)
C. Premium (P1v3)
D. Isolated (I1v2)

**2. You are developing a generative AI application that uses Azure OpenAI. You need to store embeddings for customer documents and perform rapid semantic similarity searches. The solution must support fully managed relational data, strict ACID transactions, and native vector search. Which database service should you choose?**
A. Azure Cosmos DB for NoSQL
B. Azure Database for PostgreSQL with pgvector
C. Azure Cache for Redis Enterprise
D. Azure Blob Storage

**3. A financial organization is building a transaction processing system using Azure Functions. The system must process files uploaded to Blob Storage. Requirements:**
- **Processing takes up to 20 minutes per file.**
- **The function must be able to securely access a Key Vault that has public access disabled.**
- **The function should scale to zero when no files are being processed.**
**Which hosting plan should you use?**
A. Consumption Plan
B. Premium Plan
C. Dedicated (App Service) Plan
D. Azure Container Instances

**4. A developer is designing a messaging architecture. Order validation messages are sent by a web app and must be processed by two independent backend services: Inventory and Billing. Both services must receive a copy of every message. The messages must be processed in the exact order they were generated. Which solution meets these requirements?**
A. Azure Queue Storage
B. Azure Service Bus Queues with Sessions
C. Azure Service Bus Topics with Subscriptions and Sessions
D. Azure Event Grid

**5. You need to implement an API Gateway using Azure API Management (APIM). You have a requirement to validate an OAuth 2.0 JSON Web Token (JWT) provided by clients. If the token is invalid, the request must be rejected before it reaches the backend API. In which APIM policy section must you place the `<validate-jwt>` policy?**
A. `<inbound>`
B. `<backend>`
C. `<outbound>`
D. `<on-error>`

**6. An AI application generates massive volumes of user interaction telemetry (3 million events per second). You need to stream this data into Azure Databricks for real-time model training, while simultaneously saving a raw copy of the events to Azure Data Lake Storage. Which service should you implement?**
A. Azure Event Grid
B. Azure Event Hubs with Capture enabled
C. Azure Service Bus Premium
D. Azure Cosmos DB Change Feed

**7. You are building an intelligent chatbot. The bot must query a Cosmos DB container that stores user profiles. The partition key is `/country`. The bot frequently runs a query to find all users over the age of 30, regardless of their country. What happens when this query is executed?**
A. The query will fail because the partition key is not included.
B. The query will execute a cross-partition query, which consumes more RUs and has higher latency.
C. The query will automatically create a materialized view to optimize the search.
D. Cosmos DB will block the query to prevent hot partitions.

**8. An Azure Function uses the `TimerTrigger` with the cron expression `0 0 2 * * *`. When does the function execute?**
A. Every 2 hours
B. Every 2 minutes
C. Once daily at 2:00 AM
D. Every second at the 2-minute mark

**9. You are building an AI orchestrator that delegates tasks to multiple subagents. The orchestrator must start 5 subagents in parallel, wait for all 5 to complete their reasoning tasks, and then summarize the aggregated results. Which Durable Functions pattern must you use?**
A. Function Chaining
B. Monitor
C. Fan-out/Fan-in
D. Async HTTP API

**10. A developer configures an App Service to use a System-Assigned Managed Identity. The App Service needs to read a secret named `DbPassword` from Azure Key Vault. What are the exact steps required?**
A. Create an Access Policy or Role Assignment granting 'Key Vault Secrets User' to the App Service identity.
B. Pass the Tenant ID and Client Secret of the App Service to Key Vault.
C. Add the App Service's public IP address to the Key Vault firewall.
D. Store the Key Vault access key in the App Service application settings.

---

## Part 2: Compute, Storage, and Observability

**11. You are migrating an ASP.NET Core application to Azure App Service. The application relies heavily on in-memory session state. You plan to scale out the App Service to 5 instances. What is the most appropriate architecture change to ensure session state is maintained across instances?**
A. Enable ARR Affinity (Sticky Sessions) on the App Service.
B. Move the session state to Azure Cache for Redis.
C. Store the session state in a local file on the App Service.
D. Use a single App Service instance instead of scaling out.

**12. A static web app is hosted in Azure Blob Storage. The marketing team wants to deliver the website globally with the lowest possible latency and wants to attach a custom domain with HTTPS. Which Azure service should you place in front of the Blob Storage account?**
A. Azure Application Gateway
B. Azure Load Balancer
C. Azure CDN (Content Delivery Network)
D. Azure API Management

**13. A company is using Application Insights to monitor a distributed application. You need to investigate a sudden spike in failed HTTP requests to a backend dependency. Which Application Insights feature provides an interactive visual map showing response times and failure rates between components?**
A. Live Metrics Stream
B. Smart Detection
C. Application Map
D. Availability Tests

**14. You are writing a Kusto (KQL) query to find the top 10 slowest requests in Application Insights over the last hour. Which combination of operators is correct?**
A. `where timestamp > ago(1h) | top 10 by duration desc`
B. `filter timestamp > 1h | order by duration asc limit 10`
C. `where timestamp > 1h | summarize max(duration) top 10`
D. `where timestamp > ago(1h) | top 10 by duration asc`

**15. You are designing a Docker container deployment. The container runs a background worker that processes messages from a queue. The workload is continuous and runs 24/7. Which service is the most cost-effective for running this single container?**
A. Azure Kubernetes Service (AKS)
B. Azure Container Apps (ACA)
C. Azure Container Instances (ACI)
D. Azure App Service for Linux

**16. An application uses the `BlobServiceClient` to generate a Shared Access Signature (SAS) token. You want to generate a token that delegates access using Microsoft Entra ID credentials rather than the storage account key. What type of SAS must you generate?**
A. Account SAS
B. Service SAS
C. User Delegation SAS
D. System SAS

**17. You deploy a background worker application to Azure. The application must connect to Microsoft Graph API to read user profiles. No human user interacts with the application. Which OAuth 2.0 grant type must the application use to acquire an access token?**
A. Authorization Code
B. Client Credentials
C. Device Code
D. Implicit Grant

**18. You are configuring an Azure Cosmos DB account. Your primary requirement is to ensure the lowest possible latency for both reads and writes, even if it means users might temporarily read slightly stale data. Which consistency level should you choose?**
A. Strong
B. Bounded Staleness
C. Session
D. Eventual

**19. An AI application uses a vector database to perform semantic search. You notice that search latency is too high when querying millions of vectors. You are using `pgvector`. What is the primary method to optimize the search speed?**
A. Increase the vector dimensionality.
B. Create an HNSW or IVFFlat index on the vector column.
C. Switch the distance metric from Cosine to Euclidean.
D. Store the vectors as JSON strings instead of the `vector` type.

**20. A Logic App and an Azure Function both need to react when a user uploads a new video file to Blob Storage. The video upload triggers an event. How should you route this event to ensure both services receive it reliably?**
A. Create an Event Hub and have both services read from it.
B. Create an Event Grid System Topic for the storage account and configure two separate Event Subscriptions.
C. Use a single Service Bus Queue and have both services poll it.
D. Configure the Blob Storage container to call an HTTP webhook that triggers both services.

---

## Part 3: Deep Dives and Edge Cases

**21. You are managing an API in Azure API Management. A backend API is slow and frequently times out. To improve performance for clients, you want to return a cached response for identical requests. In which policy section must you place the `<cache-lookup>` policy?**
A. `<inbound>`
B. `<backend>`
C. `<outbound>`
D. `<on-error>`

**22. You are deploying an Azure Function that uses a Blob Trigger. The storage account is secured behind a VNet and has public access disabled. Which hosting plan and configuration are required for the Blob Trigger to work?**
A. Consumption plan with VNet Integration.
B. Premium plan with VNet Integration and VNet Triggers enabled.
C. Any plan, because Blob Triggers run outside the VNet.
D. App Service plan with an active VPN gateway.

**23. You have an Azure App Service application running on the Standard (S1) tier. You create a deployment slot named `staging`. You configure a database connection string and mark it as a "deployment slot setting" (sticky). You swap `staging` into `production`. What happens to the connection string?**
A. The `staging` connection string replaces the `production` connection string.
B. The `production` slot retains its original connection string.
C. The connection strings are merged.
D. The App Service restarts and prompts for a new connection string.

**24. Which of the following is TRUE regarding Azure Managed Identities?**
A. A User-Assigned Managed Identity is deleted automatically when the resource it is attached to is deleted.
B. A System-Assigned Managed Identity can be shared across multiple Azure resources.
C. A User-Assigned Managed Identity has an independent lifecycle and can be assigned to multiple resources.
D. Managed Identities require you to rotate credentials manually every 90 days.

**25. An application uses Azure Service Bus. Messages must be processed exactly once. To achieve this, the receiver uses `PeekLock` mode. If the receiver successfully processes the message, what method must it call to permanently remove the message from the queue?**
A. `AbandonAsync()`
B. `DeadLetterAsync()`
C. `CompleteAsync()`
D. `DeferAsync()`

**26. You are configuring an App Configuration store. You want to change the logging level of your application from "Error" to "Debug" without redeploying or restarting the application. What feature should you use?**
A. Key Vault references
B. Feature Flags
C. Dynamic configuration with a sentinel key
D. Azure Monitor log ingestion rules

**27. An AI model deployment requires caching prediction results to reduce compute costs. The results are stored as JSON strings. You want the cache entries to automatically expire 24 hours after they are created. Using the `StackExchange.Redis` library, how do you set the expiry?**
A. Pass a `TimeSpan` parameter to the `StringSetAsync` method.
B. Call `KeyDeleteAsync` with a 24-hour delay.
C. Set the `TTL` configuration property on the `ConnectionMultiplexer`.
D. Use the `allkeys-lru` eviction policy.

**28. You are designing a Cosmos DB SQL API solution. You have a container partitioned by `/tenantId`. You need to execute a stored procedure that updates multiple documents. Which of the following constraints applies?**
A. The stored procedure can update documents across multiple tenants simultaneously.
B. The stored procedure execution is scoped to a single logical partition key value.
C. Stored procedures cannot update documents, they can only read them.
D. The stored procedure must be written in C#.

**29. A developer writes an Azure Function that connects to an external third-party API. The API requires a secret API key. The key is stored in Azure Key Vault. What is the recommended way to make this key available to the Azure Function code?**
A. Hardcode the key in the Function code.
B. Store the key in `local.settings.json` and push it to source control.
C. Create an App Setting in the Function App that uses the `@Microsoft.KeyVault(...)` syntax.
D. Query the Key Vault using the REST API in every function execution.

**30. You deploy an Azure Container App (ACA) with an HTTP ingress. You want to route 80% of traffic to revision `v1` and 20% of traffic to a new revision `v2`. How is this achieved in ACA?**
A. By deploying an Azure Traffic Manager profile in front of ACA.
B. By configuring traffic splitting weights on the ACA revisions.
C. By modifying the Kubernetes ingress controller manually.
D. By using an APIM policy.

---
## Part 4: 20 Rapid-Fire Scenarios

**31. Service selection: High-volume message queuing without ordering requirements.**
A. Service Bus
B. Queue Storage

**32. Container service: Running a production serverless microservice architecture with Dapr.**
A. ACI
B. ACA

**33. Identity selection: Authenticating a background daemon app.**
A. Client Credentials
B. Authorization Code

**34. App Service Tier: Minimum tier for Autoscaling.**
A. Basic
B. Standard

**35. Blob Storage: Rehydrating an Archive blob in 1 hour.**
A. Standard priority
B. High priority

**36. Cosmos DB: Partition key design rule.**
A. High cardinality
B. Low cardinality

**37. Redis Cache: Best practice for `ConnectionMultiplexer`.**
A. Singleton
B. Transient (per request)

**38. APIM Policy: Modifying a request URL before it hits the backend.**
A. `<inbound>`
B. `<outbound>`

**39. Event Hubs: Retaining a raw copy of events.**
A. Capture
B. Dead Letter Queue

**40. Key Vault: Modern access control method.**
A. Access Policies
B. RBAC

**41. Durable Functions: Sequential execution.**
A. Function Chaining
B. Fan-out

**42. App Configuration: Toggling UI elements without redeploy.**
A. Sentinel keys
B. Feature Flags

**43. Service Bus: Ensuring ordered processing.**
A. Sessions
B. Duplicate Detection

**44. Cosmos DB consistency: Read your own writes.**
A. Strong
B. Session

**45. Azure Functions: Max timeout on Consumption.**
A. 5 minutes (default, max 10)
B. Unlimited

**46. `pgvector`: Distance metric for NLP embeddings.**
A. Euclidean
B. Cosine

**47. App Service Slots: Do sticky settings swap?**
A. Yes
B. No

**48. VNet Integration direction on App Service.**
A. Inbound
B. Outbound

**49. Storage Redundancy: Replicate across regions.**
A. ZRS
B. GRS

**50. Application Insights: Visual dependency tree.**
A. Application Map
B. Live Metrics

---
---

## Answer Key and Explanations

1. **B.** Standard (S1) is the minimum tier for deployment slots, autoscaling, and VNet integration. Basic (B1) lacks autoscale and slots.
2. **B.** PostgreSQL with `pgvector` provides a fully managed relational DB with ACID compliance and native high-dimensional vector search. Cosmos DB NoSQL supports vectors but is non-relational.
3. **B.** The Premium plan supports VNet Integration (to reach the secured Key Vault), avoids the 10-minute maximum timeout of the Consumption plan (since it takes 20 min), and scales dynamically. 
4. **C.** Topics allow one-to-many publish/subscribe (Inventory and Billing). Sessions ensure strict ordered processing.
5. **A.** The token must be validated in the `<inbound>` section before the request is routed to the backend.
6. **B.** Event Hubs is designed for massive telemetry streaming. Event Hubs Capture automatically saves the raw stream to Blob/Data Lake.
7. **B.** Because the query filters on age but the partition key is country, Cosmos DB must check every physical partition (a cross-partition query), which is expensive and slow.
8. **C.** The Azure cron format is `{second} {minute} {hour} {day} {month} {day-of-week}`. `0 0 2 * * *` means at second 0, minute 0, hour 2 (2:00 AM) every day.
9. **C.** Fan-out/Fan-in executes multiple functions in parallel and aggregates the results.
10. **A.** The Managed Identity needs an RBAC role assignment (Key Vault Secrets User) to be authorized to read the secret.
11. **B.** Moving state to Azure Cache for Redis externalizes the session state, allowing any app instance to access it (stateless scale-out). Sticky sessions are an anti-pattern for true scale-out.
12. **C.** Azure CDN caches static assets at edge nodes globally, reducing latency, and provides custom domain and HTTPS support.
13. **C.** Application Map provides a visual topology of components, highlighting failures and latency.
14. **A.** `where timestamp > ago(1h) | top 10 by duration desc` correctly filters by time and orders by duration descending.
15. **B.** While ACI can run containers, ACA is best for continuous background workers (like queue processors) because it natively integrates with KEDA for scaling based on queue depth.
16. **C.** User Delegation SAS uses Entra ID OAuth credentials to sign the token, which is more secure than Account or Service SAS (which use the storage account key).
17. **B.** Client Credentials flow is used for machine-to-machine/daemon apps with no human user.
18. **D.** Eventual consistency offers the lowest latency and highest throughput, but reads may lag behind writes.
19. **B.** Creating an ANN index like HNSW or IVFFlat is critical for fast vector search over millions of rows.
20. **B.** Event Grid is the correct routing service. An Event Grid System Topic attached to the storage account can push events to multiple independent Event Subscriptions (one for Logic App, one for Function).
21. **A.** `<cache-lookup>` must be in the `<inbound>` section to intercept the request and return the cached response before calling the backend. (`<cache-store>` goes in the outbound section).
22. **B.** To trigger on a VNet-secured storage account, the Function needs VNet Integration and VNet Triggers (Runtime Scale Monitoring) enabled. Consumption plan cannot trigger on private endpoints.
23. **B.** Slot settings (sticky settings) do not swap. The production slot keeps its own connection string.
24. **C.** User-Assigned Managed Identities are created as standalone Azure resources and can be attached to multiple services.
25. **C.** `CompleteAsync()` tells the broker the message was successfully processed and can be deleted.
26. **C.** Dynamic configuration using a sentinel key allows an application to refresh its configuration automatically when the sentinel key is updated.
27. **A.** `StringSetAsync` accepts an optional `TimeSpan` parameter to set the TTL (expiry).
28. **B.** Cosmos DB stored procedures are strictly scoped to a single logical partition key value. They cannot perform cross-partition transactions.
29. **C.** Key Vault references in App Settings (`@Microsoft.KeyVault`) automatically fetch the secret at runtime using Managed Identity.
30. **B.** ACA revisions natively support traffic splitting based on percentage weights.

**Rapid Fire:**
31. B
32. B
33. A
34. B
35. B
36. A
37. A
38. A
39. A
40. B
41. A
42. B
43. A
44. B
45. A
46. B
47. B
48. B
49. B
50. A
