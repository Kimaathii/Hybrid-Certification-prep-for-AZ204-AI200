# EP04 — Practice Exam 1: Foundation to Intermediate (Hybrid AZ-204 + AI-200)

**Questions:** 40 | **Difficulty:** Foundation 40% + Intermediate 60% | **Time:** 80 minutes

**Domains Covered:**
- Core Compute/Storage (Web compute, Blob storage)
- AI Data Management (Cosmos DB, pgvector, Redis)
- Advanced Containers/Messaging (AKS, Service Bus, Events)
- Security/Observability (Entra ID, Key Vault, OpenTelemetry)

---

## Part 1: Core Compute and Storage

**1. A developer is deploying a web application to Azure App Service. The application needs to support zero-downtime deployments with instant rollback capabilities. What is the minimum App Service plan tier required?**
A. Free (F1)
B. Basic (B1)
C. Standard (S1)
D. Premium V3 (P1v3)

**2. You are configuring an Azure Function to process uploaded images. The function should only execute when a new image is added to a specific Blob Storage container. Which of the following should you use?**
A. A Timer trigger running every 5 minutes
B. An Event Grid trigger responding to `Microsoft.Storage.BlobCreated` events
C. An HTTP trigger invoked by a Logic App
D. A Queue trigger checking for manual processing requests

**3. A company is migrating 50 TB of archive data to Azure Blob Storage. The data will be accessed at most once a year, but when requested, it must be available within 1 hour. Which storage tier and rehydration priority should be configured?**
A. Cool tier with standard priority rehydration
B. Archive tier with standard priority rehydration
C. Archive tier with high priority rehydration
D. Hot tier, because Archive takes too long

**4. You are writing a C# application using `BlobServiceClient`. You need to ensure the application retrieves the attributes of a blob without downloading the entire blob content. Which method should you use?**
A. `DownloadContentAsync`
B. `GetPropertiesAsync`
C. `GetBlobClient`
D. `SetHttpHeadersAsync`

**5. You need to ensure that an Azure App Service application scales out automatically when CPU utilization exceeds 70% over a 5-minute period. What must you configure?**
A. Vertical scaling
B. An autoscale rule with a metric-based condition
C. Deployment slots
D. Application Insights Smart Detection

**6. An Azure Function running on the Consumption plan needs to access a database that resides inside a private Azure Virtual Network (VNet). What is the appropriate solution?**
A. Use VNet Integration on the Consumption plan
B. Upgrade the Function to the Premium or Dedicated plan and enable VNet Integration
C. Create a Service Endpoint on the Consumption plan
D. Enable public access on the database firewall

**7. A developer wants to execute a long-running batch processing task in a container. The task runs once a night and takes about 2 hours, and the container should be terminated immediately after. What is the most cost-effective compute service?**
A. Azure Kubernetes Service (AKS)
B. Azure App Service for Containers
C. Azure Container Instances (ACI) with restart policy `Never`
D. Azure Functions Premium plan

**8. You are configuring a storage account lifecycle management policy. You want to move blobs to the Cool tier if they haven't been modified in 30 days. When will the policy evaluation occur?**
A. Immediately when a blob reaches 30 days old
B. Once daily during a scheduled run
C. Every 15 minutes
D. Whenever a new blob is uploaded

**9. A team wants to deploy a multi-container microservices architecture. They want Kubernetes-style features, such as KEDA for scaling and Dapr integration, but they do NOT want to manage the underlying Kubernetes nodes or control plane. Which service should they choose?**
A. Azure Container Instances (ACI)
B. Azure Container Apps (ACA)
C. Azure Kubernetes Service (AKS)
D. Azure Service Fabric

**10. What is the primary difference between Azure Container Registry (ACR) and Azure Container Instances (ACI)?**
A. ACR stores container images; ACI runs containers.
B. ACR runs containers; ACI stores container images.
C. ACR is for Windows containers; ACI is for Linux containers.
D. ACR is free; ACI requires a subscription.

---

## Part 2: AI Data Management (Cosmos DB, pgvector, Redis)

**11. You are designing an Azure Cosmos DB container for a highly transactional e-commerce application. Which of the following is the most important factor when choosing a partition key?**
A. A property with low cardinality (few distinct values)
B. A property with high cardinality that evenly distributes data and requests
C. A property that frequently changes
D. The `_ts` (timestamp) system property

**12. You are using Azure Database for PostgreSQL to build a Retrieval-Augmented Generation (RAG) AI application. Which PostgreSQL extension is required to store and query high-dimensional vector embeddings efficiently?**
A. PostGIS
B. pg_stat_statements
C. pgvector
D. TimescaleDB

**13. In a pgvector implementation, you want to perform a nearest-neighbor search to find the most semantically similar text to a user's prompt. Which mathematical operation is typically used in the SQL query to calculate cosine distance?**
A. `<->`
B. `<=>`
C. `<#>`
D. `===`

**14. An application uses Azure Cache for Redis to store user session data. You want to ensure that if the cache is full, the least recently used keys are removed first. Which eviction policy should you configure?**
A. noeviction
B. allkeys-lru
C. volatile-ttl
D. volatile-random

**15. A developer is writing a C# app that queries Cosmos DB. They want to ensure they always read the latest committed write. Which consistency level must they select?**
A. Eventual
B. Consistent Prefix
C. Session
D. Strong

**16. What is the recommended pattern for instantiating the `ConnectionMultiplexer` object when connecting to Azure Cache for Redis in a .NET application?**
A. Create a new instance per HTTP request
B. Create a new instance per user session
C. Create a single static instance (singleton) and reuse it
D. Create an instance for every cache read/write operation

**17. You are implementing the Cache-Aside pattern for an AI model's prediction results. What is the correct sequence of operations?**
A. Check Cache -> If miss, run AI model -> Store result in DB -> Return result
B. Run AI model -> Store result in Cache -> Return result
C. Check Cache -> If miss, run AI model -> Store result in Cache -> Return result
D. Write to DB -> Async flush to Cache -> Return result

**18. Azure Cosmos DB exposes a feature that allows a serverless Azure Function to be triggered automatically whenever an item is inserted or updated in a container. What is this feature called?**
A. Cosmos DB Triggers
B. Cosmos DB Change Feed
C. Cosmos DB Materialized Views
D. Cosmos DB Event Grid Integration

**19. You are creating an Azure Cosmos DB account for a web app that expects unpredictable, sporadic traffic with long periods of zero activity. Cost optimization is a priority. Which capacity mode should you choose?**
A. Provisioned throughput (manual)
B. Provisioned throughput (autoscale)
C. Serverless
D. Dedicated gateway

**20. When querying a pgvector column in PostgreSQL, which type of index should you create to speed up approximate nearest neighbor (ANN) searches?**
A. B-Tree
B. Hash
C. HNSW (Hierarchical Navigable Small World)
D. BRIN (Block Range INdex)

---

## Part 3: Advanced Containers and Messaging

**21. An e-commerce system needs to process customer orders. It is critical that orders are processed in the exact sequence they were submitted by each individual customer. Which Azure messaging service and feature should you use?**
A. Azure Queue Storage with visibility timeout
B. Azure Service Bus Queues with Sessions enabled
C. Azure Event Grid with custom topics
D. Azure Event Hubs with standard partitions

**22. An Azure Service Bus receiver application uses the Peek-Lock receive mode. The application successfully processes a message but crashes before calling `CompleteAsync()`. What happens to the message?**
A. It is immediately moved to the Dead-Letter Queue (DLQ).
B. It is permanently deleted.
C. It becomes visible again to other receivers once the lock duration expires.
D. It remains locked forever.

**23. You are designing an architecture to ingest 2 million telemetry events per second from IoT devices. Which Azure service is specifically designed for this high-throughput streaming scenario?**
A. Azure Service Bus
B. Azure Queue Storage
C. Azure Event Grid
D. Azure Event Hubs

**24. What is the primary difference between an Azure Service Bus Queue and an Azure Service Bus Topic?**
A. Queues have a maximum message size of 64KB; Topics allow 256KB.
B. Queues are for one-to-one communication; Topics are for one-to-many publish/subscribe communication.
C. Queues support sessions; Topics do not.
D. Queues guarantee delivery; Topics do not.

**25. A developer wants to send a notification to multiple microservices whenever a new user registers. They want an event-driven, push-based routing system with at-least-once delivery. Which service should they choose?**
A. Azure Event Grid
B. Azure Queue Storage
C. Azure Relay
D. Azure Cache for Redis

**26. In Azure Event Hubs, what is the role of a Consumer Group?**
A. To automatically scale the number of partitions based on traffic.
B. To allow multiple consuming applications to each have a separate view (offset) of the event stream.
C. To route specific events to specific consumers based on message properties.
D. To encrypt data at rest for specific groups of users.

**27. You need to capture Azure Event Hubs streaming data automatically to an Azure Blob Storage account in Avro format. What feature should you enable?**
A. Event Hubs Archive
B. Event Hubs Capture
C. Stream Analytics integration
D. Event Grid routing

**28. A microservice needs to send a simple status message to a background worker. The volume is expected to be massive, but message ordering is not required, and there is no need for a dead-letter queue. Which service is the most cost-effective?**
A. Azure Service Bus Premium
B. Azure Service Bus Standard
C. Azure Queue Storage
D. Azure Event Hubs

**29. An AKS (Azure Kubernetes Service) cluster needs to pull images from a private Azure Container Registry (ACR). What is the recommended way to grant AKS access to ACR?**
A. Use the ACR admin username and password in a Kubernetes secret.
B. Assign the `AcrPull` role to the AKS managed identity.
C. Enable anonymous pull access on the ACR.
D. Create a Service Endpoint between AKS and ACR.

**30. Which Azure service is best suited for implementing a complex orchestration of serverless tasks using the Fan-out/Fan-in pattern?**
A. Azure Logic Apps
B. Azure Functions (Durable Functions extension)
C. Azure Event Grid
D. Azure Service Bus

---

## Part 4: Security and Observability

**31. You are developing a .NET API that needs to retrieve a database connection string from Azure Key Vault. What is the most secure and recommended way to authenticate the application to Key Vault without storing credentials in code?**
A. Use a Shared Access Signature (SAS) token.
B. Store a client secret in the `appsettings.json` file.
C. Enable a Managed Identity for the application and use `DefaultAzureCredential`.
D. Use the Key Vault access key.

**32. An API hosted on Azure App Service requires users to sign in with their Microsoft Entra ID credentials. You want to implement this with zero code changes to the application. Which feature should you use?**
A. Microsoft Authentication Library (MSAL)
B. App Service Easy Auth (Authentication / Authorization)
C. Azure API Management validate-jwt policy
D. Microsoft Graph API

**33. You are configuring OpenTelemetry in your cloud-native application. You want to correlate a specific user's HTTP request across multiple microservices to identify performance bottlenecks. Which observability pillar does this represent?**
A. Metrics
B. Logs
C. Distributed Tracing
D. Dashboards

**34. An Azure App Service is configured to use an App Setting that references a secret in Azure Key Vault using the `@Microsoft.KeyVault(...)` syntax. However, the app cannot retrieve the secret. What is the most likely cause?**
A. The App Service does not have a managed identity with Key Vault Secrets User permissions.
B. The Key Vault is in a different region than the App Service.
C. The secret is stored as a base64 string.
D. Key Vault references are only supported in Azure Functions.

**35. A background worker application runs on an on-premises server with no browser available. It needs to authenticate to Azure via Microsoft Entra ID. Which OAuth 2.0 flow is most appropriate?**
A. Authorization Code flow
B. Implicit flow
C. Device Code flow
D. On-Behalf-Of flow

**36. You want to monitor an Azure Function and receive an email alert immediately if the failure rate exceeds 5% over a 5-minute period. What should you create?**
A. An Application Insights Smart Detection rule
B. An Azure Monitor metric alert and an Action Group
C. A Log Analytics workspace query
D. A Kusto query scheduled via a Timer function

**37. Which Kusto Query Language (KQL) command is used to filter records to only include those from the last 24 hours?**
A. `| filter timestamp > 24h`
B. `| where timestamp > ago(24h)`
C. `| select timestamp within 24 hours`
D. `| limit 24h`

**38. You want to centrally manage application settings and feature flags for a suite of microservices. You do NOT want to store sensitive secrets here. Which Azure service should you use?**
A. Azure Key Vault
B. Azure App Configuration
C. Azure Cosmos DB
D. Azure Cache for Redis

**39. You are implementing a daemon application (service-to-service communication with no human user involved). Which Microsoft Entra ID OAuth flow is required?**
A. Client Credentials flow
B. Authorization Code flow
C. PKCE flow
D. Device Code flow

**40. To securely configure an Azure App Service to communicate with Azure Key Vault, what must be configured at the network level if the Key Vault has public access disabled?**
A. VNet Integration on the App Service and a Private Endpoint on the Key Vault
B. VNet Integration on both the App Service and Key Vault
C. A Service Endpoint on the App Service
D. An Application Gateway in front of Key Vault

---
---

## Answer Key and Explanations

**1. Answer: C**
- **Explanation:** To use deployment slots, the App Service Plan must be on the Standard tier (S1) or higher. Basic (B1) does not support slots. Free (F1) does not support slots or custom domains. Premium V3 (P1v3) supports slots but is not the *minimum* required. 
- 🚨 **EXAM ALERT:** Standard tier is the threshold for production features like slots, autoscale, and VNet integration.

**2. Answer: B**
- **Explanation:** Azure Event Grid provides reactive, push-based event routing. When a blob is created, Event Grid immediately pushes a `Microsoft.Storage.BlobCreated` event to trigger the Function. This is serverless and instant. A timer trigger would require inefficient polling.

**3. Answer: C**
- **Explanation:** The Archive tier is for data accessed rarely (at most once a year). However, standard rehydration takes up to 15 hours. Because the scenario demands availability within 1 hour, high priority rehydration is required.

**4. Answer: B**
- **Explanation:** `GetPropertiesAsync` retrieves only the metadata and system properties (like content length, content type, and ETag) without downloading the actual payload. `DownloadContentAsync` downloads the whole blob.

**5. Answer: B**
- **Explanation:** Autoscaling (scaling out/in automatically based on load) requires an autoscale rule with a metric condition (e.g., CPU > 70%). Vertical scaling is changing the hardware size manually.

**6. Answer: B**
- **Explanation:** The standard Consumption plan does *not* support VNet Integration. To reach private resources inside a VNet, the Function must be hosted on the Premium (Elastic Premium), Dedicated (App Service), or Flex Consumption plan, which support VNet Integration.

**7. Answer: C**
- **Explanation:** Azure Container Instances (ACI) is perfect for short-lived, one-off, or batch jobs without the overhead of cluster management. Setting the restart policy to `Never` ensures the container terminates when the batch job finishes, minimizing costs.

**8. Answer: B**
- **Explanation:** Azure Blob Storage lifecycle management policies are evaluated once daily. They do not run in real-time or immediately when the threshold is crossed.

**9. Answer: B**
- **Explanation:** Azure Container Apps (ACA) provides serverless microservices with built-in Kubernetes features (KEDA, Dapr, Envoy) but abstracts away the control plane and nodes. AKS requires you to manage the nodes and cluster.

**10. Answer: A**
- **Explanation:** ACR (Azure Container Registry) is a private warehouse/registry where Docker images are stored. ACI (Azure Container Instances) is the compute engine that actually pulls the image and runs the container.

**11. Answer: B**
- **Explanation:** A Cosmos DB partition key must have high cardinality (many distinct values) to ensure data and requests are distributed evenly across physical partitions. Low cardinality causes "hot partitions." 

**12. Answer: C**
- **Explanation:** `pgvector` is the standard open-source PostgreSQL extension that enables the storage and querying of high-dimensional vector embeddings, which is critical for vector search and RAG architectures in AI workloads.

**13. Answer: B**
- **Explanation:** In `pgvector`, `<=>` is the operator used to calculate cosine distance. `<->` is used for Euclidean distance (L2), and `<#>` is used for inner product. Cosine distance is the standard for NLP and text embeddings.

**14. Answer: B**
- **Explanation:** The `allkeys-lru` eviction policy evicts the least recently used (LRU) keys out of all keys in the cache to make room for new data. `noeviction` would cause the cache to return an error when full.

**15. Answer: D**
- **Explanation:** Strong consistency guarantees that a read operation always returns the value of the most recent committed write. Session consistency (the default) guarantees read-your-own-writes within a session, but not globally instantly. Eventual consistency offers the lowest latency but data may be stale.

**16. Answer: C**
- **Explanation:** `ConnectionMultiplexer` in the `StackExchange.Redis` library is designed to be shared and reused across the application. Creating a new instance per request causes connection exhaustion and massive performance degradation.
- 🚨 **EXAM ALERT:** Always implement the Redis multiplexer as a singleton.

**17. Answer: C**
- **Explanation:** In the Cache-Aside pattern, the application checks the cache first. If the data isn't there (miss), it queries the underlying source (DB or AI model), stores the result in the cache, and then returns it.

**18. Answer: B**
- **Explanation:** The Cosmos DB Change Feed outputs a continuous, sorted record of changes (inserts and updates) to a container. Azure Functions has a native trigger to process this feed. Note: Change feed does *not* capture deletes.

**19. Answer: C**
- **Explanation:** For unpredictable, sporadic traffic with long periods of zero activity, the Serverless capacity mode is most cost-effective because you only pay for the RUs consumed per operation. Provisioned throughput charges hourly regardless of usage.

**20. Answer: C**
- **Explanation:** HNSW (Hierarchical Navigable Small World) is the recommended index type in pgvector for large datasets because it provides excellent query speed for approximate nearest neighbor (ANN) searches compared to exact searches or simple B-Trees.

**21. Answer: B**
- **Explanation:** Azure Service Bus Queues support Sessions. When sessions are enabled, all messages with the same `SessionId` are guaranteed to be processed in the exact order they were enqueued, and by the same consumer. Queue Storage does not support guaranteed ordering.

**22. Answer: C**
- **Explanation:** Peek-Lock hides the message from other receivers for a set duration (lock duration). If the app crashes before calling `CompleteAsync()`, the lock expires, and the message reappears on the queue to be processed again (at-least-once delivery).

**23. Answer: D**
- **Explanation:** Azure Event Hubs is a big data streaming platform capable of receiving and processing millions of events per second. Service Bus and Queue Storage cannot handle this level of raw throughput.

**24. Answer: B**
- **Explanation:** Queues are point-to-point (one sender, one receiver). Topics are publish/subscribe (one sender, multiple independent receivers via Subscriptions). 

**25. Answer: A**
- **Explanation:** Azure Event Grid is designed for reactive, event-driven routing to multiple subscribers. It is a push-based model with at-least-once delivery guarantees.

**26. Answer: B**
- **Explanation:** Consumer Groups enable multiple consuming applications (e.g., an analytics app and an archiving app) to read the exact same event stream at their own pace, maintaining their own separate offsets.

**27. Answer: B**
- **Explanation:** Event Hubs Capture automatically writes the incoming streaming data to Blob Storage or Azure Data Lake Storage (ADLS) in Avro format based on time or size windows.

**28. Answer: C**
- **Explanation:** Azure Queue Storage is a simple, highly scalable, and very cost-effective queuing service. If ordering, dead-lettering, and sessions are not required, Queue Storage is the preferred and cheaper choice over Service Bus.

**29. Answer: B**
- **Explanation:** The secure and Azure-native way to authorize AKS to pull from ACR is to assign the `AcrPull` RBAC role to the Managed Identity of the AKS cluster. Using admin credentials is a security risk.

**30. Answer: B**
- **Explanation:** Durable Functions (an extension of Azure Functions) allows you to write stateful workflows in a serverless compute environment. The Fan-out/Fan-in pattern executes multiple functions in parallel and then waits for all to finish to aggregate the results.

**31. Answer: C**
- **Explanation:** Using a Managed Identity along with `DefaultAzureCredential` allows the application to authenticate to Azure Key Vault without any credentials or secrets hard-coded in the source code or config files. 

**32. Answer: B**
- **Explanation:** App Service Easy Auth intercepts traffic at the web server layer and handles the Entra ID OAuth flow before requests reach the application code. It requires zero code changes.

**33. Answer: C**
- **Explanation:** Distributed Tracing follows a single request across multiple services and components, allowing developers to see the complete end-to-end journey and identify latency bottlenecks.

**34. Answer: A**
- **Explanation:** To use Key Vault references in App Settings, the App Service must have a System-Assigned or User-Assigned Managed Identity, and that identity must be granted the 'Key Vault Secrets User' RBAC role on the Key Vault.

**35. Answer: C**
- **Explanation:** The Device Code flow is specifically designed for devices or operating systems that do not have a web browser. It provides a URL and a code for the user to authenticate on a different device (like their smartphone or laptop).

**36. Answer: B**
- **Explanation:** To trigger an immediate action (like an email) based on a metric crossing a threshold, you configure an Azure Monitor Metric Alert and link it to an Action Group configured with an email receiver.

**37. Answer: B**
- **Explanation:** In KQL, the `where` operator is used for filtering, and `ago(24h)` calculates the datetime exactly 24 hours prior to execution. `| where timestamp > ago(24h)` is the correct syntax.

**38. Answer: B**
- **Explanation:** Azure App Configuration is a centralized service for managing application settings and feature flags. While it can reference Key Vault, App Config itself is not meant to store sensitive secrets.

**39. Answer: A**
- **Explanation:** The Client Credentials flow is used for service-to-service communication (daemons or background workers) where no user is present. The application authenticates using its own identity (client ID and secret/certificate).

**40. Answer: A**
- **Explanation:** If Key Vault has public access disabled, you must create a Private Endpoint to bring the Key Vault into the VNet. To allow the App Service to reach into that VNet, you must enable VNet Integration on the App Service.
