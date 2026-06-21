# EP03: 50 Common Exam Traps

🔴 HIGH EXAM RELEVANCE

## Introduction

Microsoft exams are designed to test not just what you know, but whether you can distinguish the *best* answer from a *technically possible* answer. This document catalogs the 50 most common "traps" found across the AZ-204 and AI-200 exams. 

**Analogy:** A trap is a fake door painted on a wall. It looks like an exit, but if you run into it, you fail. Knowing the traps means you can navigate the maze safely.

---

## Domain 1: Compute (Traps 1-15)

### 1. The "Lift and Shift" Trap
- **The Trap Phrase:** "You must migrate an existing legacy web application. You cannot make any code changes."
- **The Distractor:** Azure App Service Web App.
- **The Correct Answer:** Azure Container Instances (ACI) or Azure Virtual Machines.
- **Why?** App Service often requires minor configuration changes or SDK updates for logging and state management. True zero-code "lift and shift" of legacy dependencies usually requires a container or a VM.

### 2. The "Minimum Tier for Slots" Trap
- **The Trap Phrase:** "You need to configure a staging environment for your web app to ensure zero-downtime deployments."
- **The Distractor:** Basic (B1) Tier.
- **The Correct Answer:** Standard (S1) Tier.
- **Why?** Basic tier allows custom domains and SSL, but it **does not** support deployment slots. Standard is the absolute minimum.

### 3. The "Sticky Slot Settings" Trap
- **The Trap Phrase:** "You swap your Staging and Production slots. The application crashes because it connects to the test database."
- **The Distractor:** Update the connection string in the application code.
- **The Correct Answer:** Mark the connection string as a "Slot Setting" (Sticky).
- **Why?** If a setting is marked as a "Slot Setting," it stays glued to the slot during a swap. Production keeps its Production DB string, and Staging keeps its Test DB string.

### 4. The "Autoscale Out vs Up" Trap
- **The Trap Phrase:** "Your web app is experiencing high CPU load. You need to handle the traffic dynamically."
- **The Distractor:** Scale Up (change to a larger VM size).
- **The Correct Answer:** Scale Out (add more instances via an Autoscale rule).
- **Why?** Scaling up requires a manual change and causes brief downtime. Scaling out (adding identical clones of your app) is dynamic and automatic.

### 5. The "Function VNet Integration" Trap
- **The Trap Phrase:** "Your Azure Function must securely access a database inside a Virtual Network (VNet)."
- **The Distractor:** Consumption Plan.
- **The Correct Answer:** Premium or Dedicated Plan.
- **Why?** The standard serverless Consumption plan **does not** support VNet integration. You must upgrade to Premium.

### 6. The "Function Timeout" Trap
- **The Trap Phrase:** "Your Azure Function processes video files and takes 15 minutes to run. It keeps failing."
- **The Distractor:** Increase the timeout setting in `host.json` to 20 minutes.
- **The Correct Answer:** Upgrade to Premium Plan or use Durable Functions.
- **Why?** The absolute maximum timeout for a Consumption plan function is 10 minutes. You cannot override it past 10 minutes.

### 7. The "Multiple Triggers" Trap
- **The Trap Phrase:** "Your Function must run every night at midnight AND when an HTTP request is received."
- **The Distractor:** Add both a Timer Trigger and an HTTP Trigger to the same function.
- **The Correct Answer:** Create two separate functions.
- **Why?** A function can only ever have **one** trigger. 

### 8. The "Durable Function Fan-Out" Trap
- **The Trap Phrase:** "You need to process 100 images simultaneously and then send an email when ALL of them are finished."
- **The Distractor:** Function Chaining (A -> B -> C).
- **The Correct Answer:** Fan-Out / Fan-In pattern.
- **Why?** Chaining is sequential. Fan-out executes them in parallel and waits for all to finish before proceeding.

### 9. The "ACR Build" Trap
- **The Trap Phrase:** "You need to compile a Docker image and push it to Azure, but your corporate laptop does not allow Docker to be installed."
- **The Distractor:** Provision a Virtual Machine, install Docker, and build it there.
- **The Correct Answer:** Use Azure Container Registry (ACR) Tasks (`az acr build`).
- **Why?** ACR Tasks build the image entirely in the Azure cloud. No local Docker installation is needed.

### 10. The "Container Apps vs Instances" Trap
- **The Trap Phrase:** "You need to deploy 5 microservices that scale to zero and communicate with each other securely using Dapr."
- **The Distractor:** Azure Container Instances (ACI).
- **The Correct Answer:** Azure Container Apps (ACA).
- **Why?** ACI is for simple, isolated containers. ACA provides a managed environment, KEDA scaling, and Dapr integration for microservices.

### 11. The "Long-Running Batch Container" Trap
- **The Trap Phrase:** "You need to run a containerized data processing job that runs once a week and takes 4 hours."
- **The Distractor:** Azure Kubernetes Service (AKS).
- **The Correct Answer:** Azure Container Instances (ACI).
- **Why?** AKS is massive overkill and costs too much for a single weekly batch job. ACI is perfect for short-lived, isolated tasks.

### 12. The "AKS Image Pull" Trap
- **The Trap Phrase:** "Your newly deployed AKS pod fails with an `ImagePullBackOff` error when trying to get an image from ACR."
- **The Distractor:** Make the ACR repository public.
- **The Correct Answer:** Assign the `AcrPull` role to the AKS cluster's managed identity.
- **Why?** By default, AKS cannot read from a private ACR. It needs explicit RBAC permissions.

### 13. The "App Service File System" Trap
- **The Trap Phrase:** "Your App Service saves user-uploaded images to the local `C:\` drive. They disappear when the app scales out."
- **The Distractor:** Mount an Azure Disk.
- **The Correct Answer:** Rewrite the app to save to Azure Blob Storage.
- **Why?** App Service file systems are ephemeral and not shared between scale-out instances. 

### 14. The "Linux vs Windows Plan" Trap
- **The Trap Phrase:** "You are deploying a Python web application."
- **The Distractor:** Deploy to an App Service Windows Plan.
- **The Correct Answer:** Deploy to an App Service Linux Plan.
- **Why?** Python is only supported natively on Linux App Service plans.

### 15. The "WebJob vs Function" Trap
- **The Trap Phrase:** "You already have an App Service Plan and want to run a background script without incurring extra compute costs."
- **The Distractor:** Azure Functions Consumption Plan.
- **The Correct Answer:** Azure WebJobs.
- **Why?** WebJobs run on the same VM as your App Service. If you already paid for the App Service, the WebJob runs for free using spare CPU cycles.

---

## Domain 2: Storage (Traps 16-25)

### 16. The "Read from Secondary" Trap
- **The Trap Phrase:** "You need disaster recovery. If the primary region goes down, the application must immediately read data from the secondary region."
- **The Distractor:** Geo-Redundant Storage (GRS).
- **The Correct Answer:** Read-Access Geo-Redundant Storage (RA-GRS).
- **Why?** Standard GRS replicates data, but the secondary region is strictly *offline* until Microsoft declares a disaster. RA-GRS lets you read from it at any time.

### 17. The "Archive Rehydration" Trap
- **The Trap Phrase:** "You store legal documents in the Archive tier. You need to download a document instantly for a court case."
- **The Distractor:** Use the Blob REST API to download it directly.
- **The Correct Answer:** Rehydrate the blob to Hot or Cool tier first.
- **Why?** You cannot read data directly from Archive. It must be moved (rehydrated) first, which can take up to 15 hours.

### 18. The "Storage Account Key Sharing" Trap
- **The Trap Phrase:** "A third-party vendor needs to upload a single file to your container. You must maximize security."
- **The Distractor:** Give them the Storage Account Access Key.
- **The Correct Answer:** Generate a User Delegation SAS token valid for 1 hour.
- **Why?** The Access Key gives them god-level access to the entire account. SAS tokens grant granular, time-limited access.

### 19. The "Static Website Hosting" Trap
- **The Trap Phrase:** "You want to host a simple HTML/CSS/JS website as cheaply as possible."
- **The Distractor:** Azure App Service Free Tier.
- **The Correct Answer:** Azure Storage Static Website Hosting.
- **Why?** App Service is overkill for plain HTML. Storage accounts have a built-in feature to host static files from a `$web` container for pennies.

### 20. The "Early Deletion Penalty" Trap
- **The Trap Phrase:** "You move files to the Cool tier to save money, but you delete them 5 days later. Your bill goes up."
- **The Distractor:** The files were too large.
- **The Correct Answer:** You incurred an early deletion penalty.
- **Why?** Cool tier requires files to stay for a minimum of 30 days. Archive requires 180 days. If you delete them early, you pay for the remaining days anyway!

### 21. The "Lifecycle Management" Trap
- **The Trap Phrase:** "You want to move blobs to the Cool tier immediately when they are marked as 'processed'."
- **The Distractor:** Use a Storage Lifecycle Management policy.
- **The Correct Answer:** Use an Azure Function triggered by Blob creation to move them.
- **Why?** Lifecycle Management policies only run *once a day*. They are not real-time. 

### 22. The "Blob vs File vs Disk" Trap
- **The Trap Phrase:** "Your legacy app requires the SMB protocol to mount a network drive."
- **The Distractor:** Azure Blob Storage.
- **The Correct Answer:** Azure Files.
- **Why?** Blob is for object storage (REST APIs). Azure Files provides traditional SMB network shares.

### 23. The "AzCopy vs Storage Explorer" Trap
- **The Trap Phrase:** "You need to automate the nightly migration of 5 TB of data from an on-premises server to Azure."
- **The Distractor:** Azure Storage Explorer.
- **The Correct Answer:** AzCopy.
- **Why?** Storage Explorer is a GUI tool for humans. AzCopy is a command-line tool built for high-performance scripting and automation.

### 24. The "Page Blob" Trap
- **The Trap Phrase:** "You need to store Virtual Machine disks that require frequent, random read/write operations."
- **The Distractor:** Block Blobs.
- **The Correct Answer:** Page Blobs.
- **Why?** Block blobs are for sequential data (images, videos). Page blobs are optimized for random-access reads and writes (VHD files).

### 25. The "Immutability Policy" Trap
- **The Trap Phrase:** "You are legally required to ensure that financial records cannot be modified or deleted for 7 years, even by the administrator."
- **The Distractor:** Set Read-Only RBAC permissions.
- **The Correct Answer:** Apply a Time-Based Retention Policy (WORM - Write Once, Read Many).
- **Why?** RBAC can be bypassed by someone with higher privileges. Immutability policies physically lock the data at the storage layer.

---

## Domain 3: AI Data - Cosmos & Vector (Traps 26-35)

### 26. The "Postgres vs Cosmos" Trap
- **The Trap Phrase:** "You have an existing highly relational database and want to add Semantic Search for AI."
- **The Distractor:** Migrate everything to Cosmos DB.
- **The Correct Answer:** Use Azure Database for PostgreSQL with the `pgvector` extension.
- **Why?** If the data is already relational (SQL), keep it relational. `pgvector` adds vector search directly to Postgres.

### 27. The "Changing the Partition Key" Trap
- **The Trap Phrase:** "Your Cosmos DB container is experiencing throttling. You realize your partition key is bad and want to change it."
- **The Distractor:** Update the container settings in the Azure Portal.
- **The Correct Answer:** Create a new container with the new key and migrate the data.
- **Why?** A partition key **cannot be changed** once the container is created. Ever.

### 28. The "Hot Partition" Trap
- **The Trap Phrase:** "You choose `Date` as your partition key for a logging application."
- **The Distractor:** This is a good choice because queries usually filter by date.
- **The Correct Answer:** This is a terrible choice because all writes for today hit a single partition (a "Hot Partition").
- **Why?** A good partition key spreads traffic evenly across all partitions (e.g., `DeviceId` or `UserId`).

### 29. The "Strong Consistency Latency" Trap
- **The Trap Phrase:** "You set Cosmos DB to Strong Consistency across 5 global regions. Your users complain the app is very slow."
- **The Distractor:** Increase the Request Units (RUs).
- **The Correct Answer:** Lower the consistency level to Session or Eventual.
- **Why?** Strong Consistency means a write must be fully replicated to *all 5 regions* before returning success to the user. This causes massive latency.

### 30. The "Default Consistency" Trap
- **The Trap Phrase:** "You create a new Cosmos DB account. What is the default consistency level?"
- **The Distractor:** Strong.
- **The Correct Answer:** Session.
- **Why?** Session consistency guarantees that a user will always read their own writes, which is perfect for 90% of web applications.

### 31. The "Vector Dimension" Trap
- **The Trap Phrase:** "You generate embeddings using OpenAI `text-embedding-ada-002`. The output has 1,536 dimensions. You create a Postgres vector column with 1,000 dimensions."
- **The Distractor:** The database will truncate the remaining dimensions.
- **The Correct Answer:** The insert will fail.
- **Why?** The dimension size of the database column must *exactly match* the dimension size of the embedding model.

### 32. The "Cross-Partition Query" Trap
- **The Trap Phrase:** "Your query filters on `Status` but your partition key is `UserId`. The query consumes massive RUs."
- **The Distractor:** Add an index to the `Status` field.
- **The Correct Answer:** You are executing a "Fan-out" cross-partition query.
- **Why?** Because the query does not include the partition key (`UserId`), Cosmos DB must wake up every single partition and check them all. This is highly inefficient.

### 33. The "Change Feed" Trap
- **The Trap Phrase:** "You need to trigger an Azure Function exactly when a new item is inserted into Cosmos DB."
- **The Distractor:** Poll the database every 5 minutes using a Timer Trigger.
- **The Correct Answer:** Use a Cosmos DB Trigger (which listens to the Change Feed).
- **Why?** Polling wastes RUs. The Change Feed is a push-based mechanism that reacts instantly to inserts and updates.

### 34. The "Change Feed Deletion" Trap
- **The Trap Phrase:** "An item is deleted from Cosmos DB. Your Change Feed function does not fire."
- **The Distractor:** There is a bug in your function code.
- **The Correct Answer:** The Change Feed does not capture hard deletes.
- **Why?** By default, the Change Feed only logs Inserts and Updates. For deletes, you must implement a "Soft Delete" pattern (setting a flag `IsDeleted = true`).

### 35. The "Postgres Flexible vs Single" Trap
- **The Trap Phrase:** "You want to deploy an AI application using PostgreSQL with pgvector."
- **The Distractor:** Azure Database for PostgreSQL Single Server.
- **The Correct Answer:** Azure Database for PostgreSQL Flexible Server.
- **Why?** Single Server is deprecated and does not support many modern AI extensions. Flexible Server is the required modern standard.

---

## Domain 4: AI Messaging (Traps 36-42)

### 36. The "Queue vs Topic" Trap
- **The Trap Phrase:** "When an order is placed, the Billing app and the Shipping app BOTH need a copy of the order."
- **The Distractor:** Service Bus Queue.
- **The Correct Answer:** Service Bus Topic.
- **Why?** A Queue delivers a message to *one* receiver. A Topic broadcasts the message to *multiple* subscribing receivers.

### 37. The "Storage Queue vs Service Bus" Trap
- **The Trap Phrase:** "You need a simple queue. The messages are 50 KB, and you don't care about strict ordering. You have zero budget."
- **The Distractor:** Service Bus Standard.
- **The Correct Answer:** Azure Storage Queues.
- **Why?** Storage Queues are practically free and great for simple tasks. Service Bus is for complex enterprise messaging (ordering, transactions).

### 38. The "Receive-and-Delete" Trap
- **The Trap Phrase:** "Your app uses Receive-and-Delete mode. The app crashes while processing a message. The message is missing."
- **The Distractor:** Check the Dead Letter Queue.
- **The Correct Answer:** The message is permanently lost.
- **Why?** Receive-and-Delete removes the message from the queue the moment it is fetched. Always use Peek-Lock mode for safety.

### 39. The "Dead Letter Queue (DLQ)" Trap
- **The Trap Phrase:** "A message has an invalid format and causes an exception in your code every time you try to read it."
- **The Distractor:** Write code to delete it.
- **The Correct Answer:** After `MaxDeliveryCount` is reached, Service Bus automatically moves it to the Dead Letter Queue.
- **Why?** The DLQ is a sub-queue for "poison messages" that fail repeatedly, allowing administrators to inspect them later without blocking the main queue.

### 40. The "Event Grid vs Event Hub" Trap
- **The Trap Phrase:** "You are collecting telemetry from 10,000 IoT devices sending 50,000 messages per second."
- **The Distractor:** Event Grid.
- **The Correct Answer:** Event Hubs.
- **Why?** Event Grid is for discrete events (e.g., "A file was uploaded"). Event Hubs is a massive pipeline designed for high-velocity streaming data (telemetry, logs).

### 41. The "Message Size" Trap
- **The Trap Phrase:** "You need to send a 5 MB video file through a Service Bus Queue."
- **The Distractor:** Upgrade to Service Bus Premium.
- **The Correct Answer:** Implement the Claim-Check pattern.
- **Why?** Even Premium has limits. The Claim-Check pattern means saving the massive video to Blob Storage, and sending a tiny message through the queue containing the URL to the video.

### 42. The "Event Hub Capture" Trap
- **The Trap Phrase:** "You need to permanently save all the streaming telemetry from Event Hubs to a data lake for later analysis."
- **The Distractor:** Write an Azure Function to read events and save them.
- **The Correct Answer:** Enable Event Hub Capture.
- **Why?** Capture is a built-in feature that automatically batches and saves the incoming data stream to Blob Storage or Data Lake without writing any code.

---

## Domain 5: Security (Traps 43-50)

### 43. The "Hardcoded Connection String" Trap
- **The Trap Phrase:** "You need to securely provide a database password to your App Service."
- **The Distractor:** Save it in the App Service Configuration as an App Setting.
- **The Correct Answer:** Save it in Key Vault, and put a Key Vault Reference in the App Service Configuration.
- **Why?** App Settings are encrypted at rest, but anyone with portal access can click "Show Value". Key Vault protects the actual value from everyone.

### 44. The "Key Vault Network" Trap
- **The Trap Phrase:** "Your Azure Function uses a Managed Identity and has Key Vault Data Plane RBAC permissions, but it still gets an 'Access Denied' error when fetching a secret."
- **The Distractor:** Give the Function 'Owner' role.
- **The Correct Answer:** Whitelist the Function's IP address in the Key Vault Firewall (or use a Private Endpoint).
- **Why?** Security is two layers. Identity (RBAC) and Networking (Firewall). If the firewall blocks the traffic, RBAC doesn't matter.

### 45. The "System vs User Identity" Trap
- **The Trap Phrase:** "You have a fleet of 5 identical Virtual Machines that all need access to the same storage account."
- **The Distractor:** Enable a System-Assigned Managed Identity on all 5 VMs.
- **The Correct Answer:** Create one User-Assigned Managed Identity and attach it to all 5 VMs.
- **Why?** System-Assigned means managing 5 separate identities and 5 separate role assignments. User-Assigned lets you manage one identity and attach it to many resources.

### 46. The "App Registration" Trap
- **The Trap Phrase:** "You are building a custom web app that lets users log in with their Microsoft 365 accounts."
- **The Distractor:** Create a Service Principal.
- **The Correct Answer:** Create an App Registration in Microsoft Entra ID.
- **Why?** App Registration is the process of telling Entra "My app exists, here is its callback URL, please give me a Client ID." 

### 47. The "Client Secret vs Certificate" Trap
- **The Trap Phrase:** "Your daemon background service needs to authenticate to an API. You must use the highest possible security."
- **The Distractor:** Generate a Client Secret (password).
- **The Correct Answer:** Use Certificate-based authentication.
- **Why?** Passwords can be leaked or guessed. Certificates use asymmetric cryptography and are considered much more secure for service-to-service auth.

### 48. The "RBAC Scope" Trap
- **The Trap Phrase:** "You want a developer to be able to restart Virtual Machines, but only inside the `Dev-RG` resource group."
- **The Distractor:** Assign the 'Virtual Machine Contributor' role at the Subscription level.
- **The Correct Answer:** Assign the role at the Resource Group scope (`Dev-RG`).
- **Why?** Principle of Least Privilege. Assigning at the subscription level would let them restart Production VMs too!

### 49. The "App Configuration vs Key Vault" Trap
- **The Trap Phrase:** "You need to store the background color setting (`BackgroundColor=Blue`) for your web app centrally."
- **The Distractor:** Store it as a Secret in Azure Key Vault.
- **The Correct Answer:** Store it in Azure App Configuration.
- **Why?** Key Vault is expensive and rate-limited; it is for *secrets* only. App Configuration is built for fast, non-secret application settings.

### 50. The "Key Vault Soft Delete" Trap
- **The Trap Phrase:** "A disgruntled admin deletes a critical Key Vault containing all production passwords."
- **The Distractor:** Restore from a manual backup.
- **The Correct Answer:** Recover it using the Soft Delete and Purge Protection features.
- **Why?** Soft Delete is now mandatory and on by default. When a vault is deleted, it is held in a "recycle bin" for 90 days. Purge Protection ensures that no one (not even an Owner) can force-delete it before the 90 days are up.

---

## What We Covered
- [x] Exam Traps related to Compute services (Containers vs PaaS, scaling, networking).
- [x] Exam Traps related to Storage (tiers, redundancy, security).
- [x] Exam Traps related to AI Data (Cosmos DB partitioning, Postgres vector limits).
- [x] Exam Traps related to Messaging (Service Bus locks, Queues vs Topics).
- [x] Exam Traps related to Security (Key Vault references, Managed Identities).
