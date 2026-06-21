# EP02: Master Cheat Sheet (AZ-204 + AI-200)

🔴 HIGH EXAM RELEVANCE

## Introduction

This is your master reference guide for the AZ-204 and AI-200 Hybrid Certification Course. It summarizes the critical limits, tiers, CLI commands, and decision criteria for all 12 modules. 

> 💡 **KEY CONCEPT**
> You do not need to memorize every single CLI flag, but you **must** memorize the specific commands that create resources, deploy code, and handle security. You must also know the differences between pricing tiers!

---

## Module 1: Azure App Service

**Analogy:** A managed hotel for your web application. You bring the code (the guest), Azure provides the building, security, and maintenance (the infrastructure).

### Tiers & Limits
| Tier | Key Features | Exam Focus |
|---|---|---|
| **Free (F1) / Shared (D1)** | 1 GB RAM, Shared compute | **NO** custom domains (Free), **NO** SSL, **NO** scaling. For dev/test only. |
| **Basic (B1-B3)** | Dedicated compute, Custom domains, SSL | **NO** deployment slots, **NO** auto-scaling. Manual scale only. |
| **Standard (S1-S3)** | Auto-scaling, 5 Deployment Slots, VNet Integration | The minimum tier for production. **Must know** for VNet and Slots! |
| **Premium (P1v3-P3v3)** | Up to 20 slots, faster processors | Use when heavy compute or max slots are required. |
| **Isolated (I1-I3)** | App Service Environment (ASE), fully private | Use when maximum network isolation (inside a VNet) is required. |

### Essential CLI Commands
```bash
# Create an App Service Plan (The server farm)
az appservice plan create --name myPlan --resource-group myRG --sku S1

# Create the Web App inside the plan
az webapp create --name myApp --resource-group myRG --plan myPlan

# Deploy code via ZIP
az webapp deployment source config-zip --resource-group myRG --name myApp --src app.zip

# Create a deployment slot
az webapp deployment slot create --name myApp --resource-group myRG --slot staging

# Swap slots
az webapp deployment slot swap --resource-group myRG --name myApp --slot staging --target-slot production
```

### When to Use What?
- Need to host a simple web app or API? **App Service**
- Need background processing? **WebJobs** (runs inside App Service)
- Need to route traffic to specific slots? **Traffic Routing (Preview)**

> 🚨 **EXAM ALERT**
> **Sticky Settings:** When you swap deployment slots, connection strings and app settings marked as "slot setting" DO NOT swap. They stick to the slot. If Production has a Prod DB string, and Staging has a Test DB string, they stay where they are after the code swaps!

---

## Module 2: Azure Functions

**Analogy:** A light switch. The electricity (billing) only flows when someone flips the switch (the trigger). It runs tiny pieces of code in response to events.

### Tiers & Limits
| Plan | Billing | Max Timeout | VNet Support | Cold Starts |
|---|---|---|---|---|
| **Consumption** | Per execution (1M free/mo) | 10 minutes | **NO** | Yes |
| **Flex Consumption** | Per execution | 30 minutes | **YES** | Minimal |
| **Premium** | Per vCPU/RAM | Unlimited | **YES** | **NO** (Pre-warmed) |
| **Dedicated** | Runs on App Service Plan | Unlimited | **YES** | **NO** |

### Essential CLI Commands
```bash
# Create a Function App project locally
func init MyProject --worker-runtime dotnet

# Create a new function inside the project
func new --template "HTTP trigger" --name MyHttpTrigger

# Run locally for testing
func start

# Publish to Azure
func azure functionapp publish myAwesomeFunctionApp
```

### When to Use What?
- Respond to an HTTP request briefly? **HTTP Trigger**
- Run a cleanup script every night at 2 AM? **Timer Trigger** (uses CRON syntax)
- Process a file the moment it uploads to Storage? **Blob Trigger**
- Need to coordinate multiple functions, or wait for human approval? **Durable Functions**

> 🚨 **EXAM ALERT**
> A Function can have **many** input bindings and **many** output bindings, but it can **ONLY HAVE ONE TRIGGER**.

---

## Module 3: Containers (ACR, ACI, ACA)

**Analogy:** 
- **ACR (Registry):** The secure parking garage where you store your container images.
- **ACI (Instances):** A valet parking service. You hand them the keys (image), they park it instantly. No management.
- **ACA (Apps):** A luxury resort for containers. Handles load balancing, scaling to zero, and microservice communication.

### Tiers & Limits
| Service | Best For | Scaling | Exam Focus |
|---|---|---|---|
| **ACR** | Storing private Docker images | N/A | Use **Premium** tier for Geo-replication. |
| **ACI** | Short-lived tasks, batch jobs | Manual / None | Fast boot, no orchestration, billed per second. |
| **ACA** | Serverless microservices | Auto (to zero via KEDA) | Hides Kubernetes complexity. Built-in Dapr. |

### Essential CLI Commands
```bash
# Build an image in the cloud WITHOUT local Docker installed!
az acr build --registry myRegistry --image myapp:v1 .

# Run a container quickly in ACI
az container create --resource-group myRG --name myapp --image myRegistry.azurecr.io/myapp:v1 --dns-name-label myapp

# Deploy to Container Apps
az containerapp create --name myApp --resource-group myRG --environment myEnv --image myRegistry.azurecr.io/myapp:v1
```

> 🚨 **EXAM ALERT**
> If a question says "You do not have Docker installed locally, but you need to build an image from a Dockerfile and push it to Azure", the answer is **ACR Tasks** (`az acr build`).

---

## Module 4: Azure Blob Storage

**Analogy:** A massive, bottomless warehouse for unstructured data (images, videos, backups). 

### Tiers & Limits
| Access Tier | Cost to Store | Cost to Read | Minimum Storage Duration |
|---|---|---|---|
| **Hot** | Highest | Lowest | None |
| **Cool** | Lower | Higher | 30 days (early deletion penalty) |
| **Cold** | Very Low | Very High | 90 days |
| **Archive** | Lowest | Highest | 180 days. Takes up to 15 hours to rehydrate! |

### Redundancy (Must Know!)
- **LRS (Locally Redundant):** 3 copies in 1 data center. Protects against disk failure.
- **ZRS (Zone Redundant):** 3 copies across 3 zones in 1 region. Protects against datacenter fire/flood.
- **GRS (Geo-Redundant):** 6 copies across 2 regions. Protects against regional outage. 
- **RA-GRS (Read-Access GRS):** Same as GRS, but you can *read* from the secondary region during normal operations.

### Essential CLI Commands
```bash
# Generate a SAS (Shared Access Signature) token for a specific blob
az storage blob generate-sas --account-name myAcc --container-name myCont --name myImage.png --permissions r --expiry 2026-01-01T00:00Z
```

> 🚨 **EXAM ALERT**
> If a question asks how to give temporary, restricted access to a single file WITHOUT sharing the main storage account key, the answer is always **SAS Token (Shared Access Signature)**.

---

## Module 5: Azure Cosmos DB

**Analogy:** A globally distributed, infinitely scalable filing cabinet. 

### Tiers & Limits
| Consistency Level | Read Latency | Data Accuracy | Exam Focus |
|---|---|---|---|
| **Strong** | Highest | Perfect | Guarantees you read the latest write. Slowest performance. |
| **Bounded Staleness** | High | Near Perfect | Lags by a configured time or item count. |
| **Session (Default)** | Low | Perfect for your session | You always read your own writes. Best for user profiles. |
| **Consistent Prefix** | Low | Good | Updates always arrive in the correct order. |
| **Eventual** | Lowest | Weakest | Highest performance, lowest cost. You might read stale data. |

### When to Use What API?
- Migrating from MongoDB? **API for MongoDB**
- Migrating from Cassandra? **API for Cassandra**
- Migrating from on-prem Graph DB? **Gremlin API**
- Building a new cloud-native app? **Core (SQL) API**

> 🚨 **EXAM ALERT**
> The **Partition Key** cannot be changed after a container is created. A good partition key has a wide range of values and spreads requests evenly (e.g., `TenantId` or `DeviceId`). A bad partition key has few values and causes "hot partitions" (e.g., `IsActive` which is only true/false).

---

## Module 6: PostgreSQL and pgvector (AI-200)

**Analogy:** A traditional relational database that went to the gym and learned how to understand the "meaning" of data (vectors).

### Tiers & Limits
| Feature | Explanation |
|---|---|
| **Azure Database for PostgreSQL Flexible Server** | The managed PaaS offering. You manage the DB, Azure manages the hardware. |
| **pgvector extension** | Allows Postgres to store and query high-dimensional embeddings (vectors) generated by AI models like OpenAI. |
| **HNSW vs IVFFlat** | HNSW (Hierarchical Navigable Small World) is faster and more accurate for vector search, but uses more memory. IVFFlat is older and uses less memory. |

### Essential CLI Commands
```bash
# Create a Flexible Server
az postgres flexible-server create --resource-group myRG --name myServer --admin-user admin --admin-password myPassword

# You must enable the pgvector extension via SQL, not CLI:
# CREATE EXTENSION vector;
```

> 🚨 **EXAM ALERT**
> AI-200 focuses heavily on Semantic Search. If you need to find "similar items based on meaning" and you already use relational data, **PostgreSQL with pgvector** is the answer. If the data is purely NoSQL documents, use Cosmos DB's vector search.

---

## Module 7: Azure Cache for Redis

**Analogy:** The short-term memory of your application. It stores frequently accessed data in RAM so you don't have to constantly ask the slow database.

### Tiers & Limits
| Tier | Key Features | Exam Focus |
|---|---|---|
| **Basic** | Single node, no SLA | Dev/Test only. |
| **Standard** | Two nodes (Primary/Replica) | Standard production SLA. |
| **Premium** | Data persistence, Clustering, VNet | Use if you need to survive a crash (persistence) or need massive scale (clustering). |
| **Enterprise** | Redis modules (RediSearch, RedisJSON) | Highest tier. Required for advanced modules. |

> 🚨 **EXAM ALERT**
> **Cache-Aside Pattern:** This is the most common exam pattern. The app checks Redis first. If data is there (Cache Hit), return it. If not (Cache Miss), query the database, save the result in Redis, and return it. Redis *does not* automatically sync with the database in Azure!

---

## Module 8: Azure Kubernetes Service (AKS)

**Analogy:** A massive cruise ship. Docker is the engine, but Kubernetes is the captain, crew, and navigation system making sure the ship runs smoothly and handles storms.

### Tiers & Limits
| Concept | Definition |
|---|---|
| **Control Plane** | The "Captain". Managed by Azure. Free. You cannot SSH into it. |
| **Nodes** | The Virtual Machines (VMs) running your containers. You pay for these. |
| **Pods** | The smallest deployable unit. Usually contains one container. |
| **Deployment** | Instructs K8s to maintain a specific number of Pod replicas. |

### Essential CLI Commands
```bash
# Get credentials to connect kubectl to your AKS cluster
az aks get-credentials --resource-group myRG --name myCluster

# Deploy an application
kubectl apply -f deployment.yaml

# Check running pods
kubectl get pods
```

> 🚨 **EXAM ALERT**
> To allow AKS to pull images from Azure Container Registry (ACR), you must grant the AKS cluster the **AcrPull** RBAC role assignment on the ACR resource. Without this, pods will fail with `ImagePullBackOff`.

---

## Module 9: Azure Service Bus

**Analogy:** A professional post office for your enterprise applications. It guarantees delivery of critical messages, even if the receiving application is offline.

### Tiers & Limits
| Concept | Definition | Exam Focus |
|---|---|---|
| **Queue** | 1 Sender, 1 Receiver (Point-to-Point). | Use for decoupled processing (e.g., Order processing). |
| **Topic** | 1 Sender, Multiple Receivers (Publish/Subscribe). | Use when multiple downstream systems need the same message (e.g., Order -> Shipping AND Billing). |
| **Standard Tier** | Pay per operation | Topics require Standard tier minimum. |
| **Premium Tier** | Dedicated resources, VNet support | Predictable performance, no noisy neighbors. |

> 🚨 **EXAM ALERT**
> **Peek-Lock vs Receive-and-Delete:**
> - **Peek-Lock (Default):** The receiver locks the message, processes it, and then explicitly completes it. If it crashes, the message unlocks and another receiver can try. (Safe).
> - **Receive-and-Delete:** The message is deleted the moment it is read. If the app crashes during processing, the message is lost forever! (Fast but dangerous).

---

## Module 10: Azure Key Vault

**Analogy:** A maximum-security bank vault. You don't store your money in your mattress (code); you put it in the vault and use an ID card (Managed Identity) to access it.

### Tiers & Limits
| Object Type | What it stores |
|---|---|
| **Secrets** | Passwords, Connection Strings, API Keys (Values smaller than 25KB). |
| **Keys** | Cryptographic keys used to encrypt/decrypt data. |
| **Certificates** | SSL/TLS certificates. Key Vault handles auto-renewal. |

### Essential CLI Commands
```bash
# Create a Key Vault
az keyvault create --name myVault --resource-group myRG

# Add a secret
az keyvault secret set --vault-name myVault --name "DatabasePassword" --value "SuperSecret123!"

# Give a Managed Identity permission to read the secret (RBAC)
az role assignment create --role "Key Vault Secrets User" --assignee <identity-object-id> --scope <key-vault-resource-id>
```

> 🚨 **EXAM ALERT**
> **Never** put connection strings or API keys directly in App Service configuration or GitHub. Store them in Key Vault, and use **Key Vault References** in your App Service configuration: `@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)`

---

## Module 11: Microsoft Entra ID (Formerly Azure AD)

**Analogy:** The bouncer at an exclusive club. It checks your ID (Authentication) and the VIP list to see what rooms you are allowed to enter (Authorization).

### Tiers & Limits
| Concept | Definition | Exam Focus |
|---|---|---|
| **App Registration** | You must register your application in Entra ID so Entra knows it exists and can issue tokens for it. |
| **Service Principal** | The specific identity of your application within a specific tenant. |
| **Managed Identity** | A special Service Principal managed entirely by Azure. No passwords to rotate! |

> 🚨 **EXAM ALERT**
> **System-Assigned vs User-Assigned Managed Identity:**
> - **System-Assigned:** Tied to the lifecycle of the resource (e.g., App Service). If you delete the App Service, the identity is deleted.
> - **User-Assigned:** Created as a standalone resource. Can be attached to multiple VMs or services. Survives if the VM is deleted.

---

## Module 12: Application Insights

**Analogy:** A heart rate monitor and diagnostic tool hooked up to your live application.

### Tiers & Limits
| Concept | What it monitors | Exam Focus |
|---|---|---|
| **Application Map** | Visual map showing how your services connect and which connections are failing. | Great for diagnosing microservice bottlenecks. |
| **Live Metrics** | Real-time 1-second latency views. | Good for watching a deployment go live. |
| **Availability Tests** | "Ping" tests from various global regions to ensure your site is up. | |

### Essential Queries (KQL - Kusto Query Language)
```kql
// Find all exceptions in the last 24 hours
exceptions
| where timestamp > ago(24h)
| order by timestamp desc

// Find the slowest requests
requests
| top 10 by duration desc
```

> 🚨 **EXAM ALERT**
> To enable Application Insights without modifying application code, you use **Auto-Instrumentation**. If you want custom telemetry (e.g., logging a specific business event like `UserAddedItemToCart`), you must use the **Application Insights SDK** in your code.

---

## What We Covered
- [x] All 12 critical hybrid modules.
- [x] Clear comparisons between pricing tiers and limits.
- [x] The difference between App Service, Functions, and Container services.
- [x] The exact CLI commands required for resource creation.
- [x] Crucial exam traps regarding Managed Identities, Key Vault, and Service Bus.
