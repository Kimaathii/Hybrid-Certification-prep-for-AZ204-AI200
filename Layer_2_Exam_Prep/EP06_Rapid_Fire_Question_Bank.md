# EP06 — Rapid-Fire Flashcard Bank (Hybrid AZ-204 + AI-200)

**Format:** 150 short Q&A pairs designed for rote memorization. Read the question, answer in under 5 seconds, then check.

---

## Domain 1: Core Compute and Storage

Q1: What is the minimum App Service tier for deployment slots?
A: Standard (S1).

Q2: What is the minimum App Service tier for autoscaling?
A: Standard (S1).

Q3: Do slot settings (sticky settings) travel with the app during a swap?
A: No, they stay attached to the specific slot.

Q4: What feature prevents an App Service from going idle after 20 minutes?
A: Always On.

Q5: Does App Service VNet Integration allow inbound traffic from the VNet?
A: No, it is outbound only.

Q6: What is the App Service feature that provides zero-code Entra ID authentication?
A: Easy Auth (Authentication / Authorization).

Q7: What is the URL format for an App Service slot named 'staging' on an app named 'myapp'?
A: myapp-staging.azurewebsites.net.

Q8: What does horizontal scaling (scale out) do?
A: Adds more VM instances of the same size.

Q9: What does ARR Affinity do in App Service?
A: Enables sticky sessions (routes a user to the same instance).

Q10: Should ARR Affinity be enabled for stateless APIs?
A: No, turn it off for better load balancing.

Q11: How many triggers can a single Azure Function have?
A: Exactly one.

Q12: What is the maximum timeout for a Function on the Consumption plan?
A: 10 minutes (default is 5 minutes).

Q13: Which Function hosting plan eliminates cold starts?
A: Premium (pre-warmed instances) or Dedicated.

Q14: Which Function plan supports VNet Integration?
A: Premium, Dedicated, or Flex Consumption (not standard Consumption).

Q15: What is the first field in an Azure Functions CRON expression?
A: Seconds.

Q16: Which Durable Functions pattern runs multiple tasks in parallel and aggregates the results?
A: Fan-out / Fan-in.

Q17: Which Durable Functions pattern executes tasks sequentially?
A: Function Chaining.

Q18: What local file stores connection strings for Azure Functions but should never be committed to Git?
A: local.settings.json.

Q19: What does the `AzureWebJobsStorage` setting do?
A: Points to the storage account required by the Function runtime for internal state.

Q20: What command deploys an Azure Function from the local CLI?
A: func azure functionapp publish <appname>.

Q21: What are the three blob types in Azure Storage?
A: Block, Append, and Page.

Q22: Which blob type is best for storing virtual machine disks (VHDs)?
A: Page blob.

Q23: What is the default access tier for a new Blob Storage account?
A: Hot.

Q24: What is the minimum storage duration for the Cool tier to avoid an early deletion penalty?
A: 30 days.

Q25: What is the minimum storage duration for the Archive tier?
A: 180 days.

Q26: Can you read a blob directly from the Archive tier?
A: No, it must be rehydrated to Hot or Cool first.

Q27: How long does standard priority Archive rehydration take?
A: Up to 15 hours.

Q28: How often do Blob Storage lifecycle management policies run?
A: Once a day.

Q29: What is the most secure type of Shared Access Signature (SAS)?
A: User Delegation SAS (signed with Entra ID).

Q30: Which storage redundancy option protects against a datacenter failure but not a regional failure?
A: Zone-Redundant Storage (ZRS).

Q31: Is the secondary region in GRS readable by default?
A: No, you must enable RA-GRS (Read-Access Geo-Redundant Storage).

Q32: Which SDK client is used to manage individual blobs?
A: BlobClient.

Q33: What feature automatically serves an `index.html` file from a `$web` container?
A: Static Website Hosting.

Q34: What CLI tool is recommended for massive bulk data transfers to Blob Storage?
A: AzCopy.

Q35: What HTTP header is used to acquire a lease on a blob?
A: x-ms-lease-id.

Q36: What feature prevents blobs from being deleted or modified for a specific interval?
A: Immutable Storage (Time-based retention policies).

Q37: What is an Append Blob best used for?
A: Logging and auditing (appending data to the end).

Q38: Can a Block Blob be converted to a Page Blob?
A: No, you must copy the data to a new blob of the desired type.

---

## Domain 2: AI Data Management (Cosmos/pgvector/Redis)

Q39: What is the default consistency level for Azure Cosmos DB?
A: Session consistency.

Q40: Which consistency level guarantees the latest read but has the highest latency?
A: Strong consistency.

Q41: Which consistency level provides the lowest latency but may return stale data?
A: Eventual consistency.

Q42: Can a Cosmos DB partition key be changed after the container is created?
A: No.

Q43: What defines a good Cosmos DB partition key?
A: High cardinality and even distribution of requests.

Q44: What happens if a partition key has low cardinality?
A: It creates "hot partitions" (bottlenecks).

Q45: Does the Cosmos DB Change Feed capture item deletions?
A: No, only inserts and updates.

Q46: What is the scope of a Cosmos DB stored procedure?
A: A single logical partition key value.

Q47: Are all properties in Cosmos DB indexed by default?
A: Yes.

Q48: What is the billing and performance metric in Cosmos DB?
A: Request Units (RUs).

Q49: Which Cosmos DB capacity mode is best for unpredictable, sporadic traffic?
A: Serverless.

Q50: What exception is thrown when Cosmos DB provisioned throughput is exceeded?
A: 429 Too Many Requests (Throttling).

Q51: What PostgreSQL extension is used to store and query AI vector embeddings?
A: pgvector.

Q52: What data type does pgvector introduce?
A: `vector`.

Q53: What operator calculates Cosine Distance in pgvector?
A: `<=>`.

Q54: What operator calculates Euclidean Distance (L2) in pgvector?
A: `<->`.

Q55: What operator calculates Inner Product in pgvector?
A: `<#>`.

Q56: Which pgvector index type provides the fastest Approximate Nearest Neighbor (ANN) search?
A: HNSW (Hierarchical Navigable Small World).

Q57: What is the older, alternative pgvector index type to HNSW?
A: IVFFlat.

Q58: Does HNSW require a training step before adding data?
A: No (unlike IVFFlat, which does).

Q59: What is the purpose of an embedding in AI data management?
A: To represent text, images, or audio as an array of floating-point numbers.

Q60: In RAG architectures, what is semantic search?
A: Searching by conceptual meaning (using vectors) rather than exact keyword matches.

Q61: What PostgreSQL command creates the pgvector extension?
A: `CREATE EXTENSION vector;`

Q62: Should you index a vector column before or after inserting a massive amount of initial data?
A: After (to speed up bulk inserts and build a better index).

Q63: What Azure service hosts fully managed PostgreSQL with pgvector?
A: Azure Database for PostgreSQL - Flexible Server.

Q64: How many dimensions can a pgvector index currently support by default in Azure?
A: Up to 2000 dimensions (perfect for OpenAI's `text-embedding-ada-002` which is 1536).

Q65: What pattern checks the cache first, and if missed, queries the database?
A: Cache-Aside (Lazy Loading).

Q66: Does Azure Cache for Redis Basic tier have an SLA?
A: No, it is for dev/test only.

Q67: What is the minimum Redis tier for production workloads?
A: Standard (replicated).

Q68: What Redis tier supports VNet Integration and clustering?
A: Premium.

Q69: Which .NET class should be implemented as a singleton for Redis connections?
A: `ConnectionMultiplexer`.

Q70: What eviction policy removes the least recently used keys when the cache is full?
A: `allkeys-lru`.

Q71: What is TTL in caching?
A: Time-To-Live (expiry time for a cached item).

Q72: What Redis data structure is best for leaderboards?
A: Sorted Sets.

Q73: Can Redis Cache store ASP.NET Core session state?
A: Yes.

Q74: What caching pattern writes to the database and cache simultaneously?
A: Write-Through.

Q75: What is a Redis cache miss?
A: When requested data is not found in the cache.

Q76: How do you force a cache item to expire immediately?
A: Delete the key (e.g., `KeyDeleteAsync`).

---

## Domain 3: Advanced Containers and Messaging

Q77: Which Azure service is a private registry for Docker images?
A: Azure Container Registry (ACR).

Q78: What command builds an image in the cloud using ACR Tasks?
A: `az acr build`.

Q79: Which service runs a single container or batch job quickly without orchestration?
A: Azure Container Instances (ACI).

Q80: Which service provides serverless microservices based on Kubernetes?
A: Azure Container Apps (ACA).

Q81: Which service provides full control over Kubernetes nodes and control plane?
A: Azure Kubernetes Service (AKS).

Q82: In ACA, what is an immutable snapshot of an application called?
A: A Revision.

Q83: What underlying component does ACA use to scale based on external events (like queue length)?
A: KEDA (Kubernetes Event-Driven Autoscaling).

Q84: Can ACA route a percentage of traffic (e.g., 20%) to a new revision?
A: Yes (traffic splitting).

Q85: What restart policy should an ACI batch job use?
A: `Never` or `OnFailure`.

Q86: Is Azure Event Grid push-based or pull-based?
A: Push-based.

Q87: What is the delivery guarantee of Event Grid?
A: At-least-once.

Q88: Because Event Grid guarantees at-least-once delivery, what must the receiving handler be?
A: Idempotent (safe to process the same event twice).

Q89: Which messaging service is designed for millions of telemetry events per second?
A: Azure Event Hubs.

Q90: What feature automatically writes Event Hubs streams to Blob Storage?
A: Event Hubs Capture.

Q91: Does Event Hubs delete an event once a consumer reads it?
A: No, events stay until the retention period expires.

Q92: How do multiple independent applications read the same Event Hubs stream?
A: By using separate Consumer Groups.

Q93: What does an Event Hubs consumer use to track its reading position?
A: An offset.

Q94: What is the primary difference between a Service Bus Queue and Topic?
A: Queues have one receiver; Topics have multiple subscribers (pub/sub).

Q95: Which Service Bus feature guarantees First-In-First-Out (FIFO) ordered processing?
A: Sessions.

Q96: Where do Service Bus messages go if they exceed the max delivery count?
A: The Dead-Letter Queue (DLQ).

Q97: Which Service Bus receive mode provides at-least-once processing?
A: Peek-Lock.

Q98: What must a Peek-Lock receiver call after successfully processing a message?
A: `CompleteAsync()`.

Q99: What happens if a Peek-Lock receiver crashes before completing the message?
A: The lock expires, and the message reappears on the queue.

Q100: Which Service Bus receive mode deletes the message immediately upon reading (at-most-once)?
A: Receive-and-Delete.

Q101: Which service is best for a massive volume of simple messages with no ordering requirement?
A: Azure Queue Storage.

Q102: What is the maximum message size for Queue Storage?
A: 64 KB.

Q103: What is the maximum message size for Service Bus Standard tier?
A: 256 KB.

Q104: What makes a message temporarily invisible to other Queue Storage readers?
A: Visibility timeout.

Q105: Can Queue Storage support sessions?
A: No.

Q106: Can Service Bus support transactions?
A: Yes.

Q107: Can Event Grid filter events based on the subject or event type?
A: Yes.

Q108: What are Azure service-generated events in Event Grid called?
A: System Topics.

Q109: What is the payload size limit for Event Grid?
A: 1 MB.

Q110: In Event Hubs, what splits the stream into parallel lanes?
A: Partitions.

Q111: If you need to send an alert when an Azure resource is created, which service do you use?
A: Event Grid.

Q112: If you need to process financial transactions exactly once in order, which service do you use?
A: Service Bus.

Q113: If you need to stream logs from 10,000 servers, which service do you use?
A: Event Hubs.

---

## Domain 4: Security and Observability

Q114: Which OAuth 2.0 flow is used for a background daemon app (no user)?
A: Client Credentials flow.

Q115: Which OAuth flow is used for a CLI tool or IoT device with no browser?
A: Device Code flow.

Q116: Which OAuth flow is used for Single Page Applications (SPAs)?
A: Authorization Code flow with PKCE.

Q117: What type of token proves the identity of the user?
A: ID Token.

Q118: What type of token proves what permissions the application has?
A: Access Token.

Q119: What is the recommended SDK for integrating Entra ID auth in custom code?
A: MSAL (Microsoft Authentication Library).

Q120: What is the central API endpoint to access Microsoft 365 data?
A: Microsoft Graph API.

Q121: Do System-Assigned Managed Identities survive if the parent resource is deleted?
A: No.

Q122: Do User-Assigned Managed Identities survive if the parent resource is deleted?
A: Yes.

Q123: What class automatically negotiates Managed Identity credentials in the Azure SDK?
A: `DefaultAzureCredential`.

Q124: What three types of objects does Azure Key Vault store?
A: Secrets, Keys, and Certificates.

Q125: What is the recommended, modern access control method for Key Vault?
A: Azure RBAC (Role-Based Access Control).

Q126: What role is required just to read a secret from Key Vault?
A: Key Vault Secrets User.

Q127: What Key Vault feature retains deleted objects for 7-90 days?
A: Soft Delete.

Q128: What feature prevents soft-deleted objects from being permanently deleted?
A: Purge Protection.

Q129: What is the App Settings syntax to reference a Key Vault secret?
A: `@Microsoft.KeyVault(SecretUri=...)`.

Q130: Does Azure App Configuration store sensitive secrets?
A: No, use Key Vault for secrets.

Q131: What feature in App Configuration lets you toggle app behavior without redeploying?
A: Feature Flags.

Q132: What are the three components of API Management (APIM)?
A: Gateway, Management Plane, Developer Portal.

Q133: In APIM, where must a `<mock-response>` policy be placed?
A: `<inbound>`.

Q134: What is the difference between a rate-limit and a quota in APIM?
A: Rate-limit is per small time window (e.g., minute). Quota is a hard cap over a long period (e.g., month).

Q135: What APIM policy validates an Entra ID token?
A: `<validate-jwt>`.

Q136: In APIM, what is a Named Value?
A: A key-value pair used to store variables (like environment variables) for policies.

Q137: Do APIM revisions represent breaking changes?
A: No, revisions are non-breaking. Versions are breaking.

Q138: What are the three pillars of observability?
A: Logs, Metrics, and Traces.

Q139: Which App Insights feature provides a visual topology of components and failure rates?
A: Application Map.

Q140: Which App Insights feature shows telemetry in real-time with zero delay?
A: Live Metrics Stream.

Q141: What App Insights feature pings your web app from global locations?
A: Availability Tests.

Q142: In KQL, what operator filters records?
A: `where`.

Q143: In KQL, what function returns the time 24 hours ago?
A: `ago(24h)`.

Q144: What AI-powered feature automatically detects anomalies in App Insights?
A: Smart Detection.

Q145: What method logs a custom event in Application Insights SDK?
A: `TrackEvent()`.

Q146: What open standard is Azure Monitor heavily adopting for instrumentation?
A: OpenTelemetry.

Q147: Can App Insights auto-instrument an App Service without changing code?
A: Yes (codeless attach).

Q148: What Azure Monitor feature is triggered when a metric crosses a threshold?
A: An Alert Rule.

Q149: What dictates the notification method (e.g., email, SMS) when an alert fires?
A: An Action Group.

Q150: What is distributed tracing?
A: Tracking a single request as it passes through multiple microservices to identify bottlenecks.
