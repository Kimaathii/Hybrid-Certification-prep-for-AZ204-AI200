# F08 — Azure Storage Account and SDK Basics

**Foundation Pack — Document 8 of 8**
**Pages:** 14–16 | **Study Time:** 2 hours
**Prerequisites:** F01–F07 completed

---

> **What you will learn:** How Azure Storage Accounts work, the four services inside every account, how to create one via CLI, redundancy options the exam tests, access keys, SAS tokens, Azure Storage Explorer, and AzCopy. By the end, you will have created a storage account, connected to it with Storage Explorer, uploaded a file, and verified it via CLI.

---

## Section 1: What Is an Azure Storage Account?

### The Problem

Imagine you are building an application. That application needs to store uploaded photos somewhere. It also needs a queue to process background tasks. It might also need a simple table to track user sessions. And your team wants a shared file system for configuration files.

Without Azure, you would need to set up a file server for photos, install message queue software, configure a database for session data, and provision a network file share for config files. Four separate systems to install, maintain, patch, and back up.

That is a lot of infrastructure just to store things.

### The Analogy: A Building with Multiple Storage Rooms

Think of an Azure Storage Account as **a building with four different types of storage rooms inside**:

| Room | What It Stores | Real-World Comparison |
|------|---------------|----------------------|
| **Blob Storage** | Files, images, videos, documents, backups — any unstructured data | A warehouse with shelves for boxes of any size |
| **Queue Storage** | Short text messages waiting to be processed | A mailroom where letters wait in line to be delivered |
| **Table Storage** | Simple structured data in rows and columns (NoSQL) | A filing cabinet with labelled folders |
| **File Storage** | Shared files accessible via network protocol (SMB) | A shared network drive in an office |

You do not create these four services separately. When you create **one** Storage Account, all four rooms are available inside it. You choose which ones to use.

### The Technical Definition

💡 **KEY CONCEPT**
> An Azure Storage Account is a single Azure resource that provides access to four storage services: **Blob**, **Queue**, **Table**, and **File**. One account, multiple services. Every service shares the same account name, access keys, and billing.

### Why This Matters for AZ-204

Storage accounts appear in virtually every AZ-204 module:

- **Azure Functions** (Module 2): Every Function App **requires** a storage account for internal state management. When you create a Function App, Azure asks for a storage account — it stores trigger tracking, queue processing state, and Durable Functions history.
- **Blob Storage** (Module 4): The entire module is about one of the four services inside a storage account.
- **Cosmos DB** (Module 5): Labs often use blob storage alongside Cosmos DB.
- **Event Grid / Service Bus** (Modules 9–10): Messages frequently reference blobs stored in storage accounts.

🚨 **EXAM ALERT**
> The exam assumes you know that a **Function App cannot exist without a storage account**. If a question asks about Function App prerequisites, a storage account is always required — even if the function does not explicitly use Blob, Queue, Table, or File storage.

### The Storage Account URL Pattern

When you create a storage account, the **account name becomes part of every URL** used to access its services. The name must be globally unique across all of Azure — no two storage accounts in the world can share the same name.

Here is the URL pattern for each service:

```
Blob:   https://<account-name>.blob.core.windows.net
Queue:  https://<account-name>.queue.core.windows.net
Table:  https://<account-name>.table.core.windows.net
File:   https://<account-name>.file.core.windows.net
```

For example, if your storage account is named `az204labstore42`, the blob endpoint would be:

```
https://az204labstore42.blob.core.windows.net
```

📝 **NOTE**
> Storage account names must be **3–24 characters**, **lowercase letters and numbers only** — no hyphens, no underscores, no uppercase. This is one of the strictest naming rules in Azure.

---

✅ **CHECKPOINT**
Before moving on, make sure you can answer these:
1. How many storage services does a single storage account provide? *(Four: Blob, Queue, Table, File)*
2. Why does every Azure Function App need a storage account? *(For internal state management — trigger tracking, queue state, Durable Functions history)*
3. What part of the URL is unique to your storage account? *(The account name — it becomes the subdomain)*

---

## Section 2: Storage Account Types

### Why Different Types Exist

Not all workloads have the same performance requirements. A photo gallery serving 10 uploads per day has very different needs from a real-time analytics pipeline processing millions of events per second. Azure offers different storage account types to match different performance and cost profiles.

### The Complete Storage Account Types Table

| Type | Also Called | Supported Services | Performance | When to Use | Exam Relevance |
|------|-----------|-------------------|-------------|-------------|----------------|
| **General Purpose v2 (GPv2)** | StorageV2 | Blob, Queue, Table, File | Standard (HDD) or Premium (SSD) | **Default choice for everything** | 🔴 HIGH |
| **Blob Storage** | BlobStorage | Blob only | Standard only | Legacy — do not create new ones | 🟡 MEDIUM |
| **Premium Block Blobs** | BlockBlobStorage | Block blobs and append blobs only | Premium (SSD) | High-transaction workloads, low latency | ⚪ CONTEXT ONLY |
| **Premium File Shares** | FileStorage | File shares only | Premium (SSD) | Enterprise file shares needing high IOPS | ⚪ CONTEXT ONLY |
| **Premium Page Blobs** | StorageV2 with premium | Page blobs only | Premium (SSD) | VM disks (unmanaged) | ⚪ CONTEXT ONLY |

💡 **KEY CONCEPT**
> **Always use General Purpose v2 (GPv2) for labs.** It supports all four services, works with all features (lifecycle policies, access tiers, etc.), and is the default when creating a storage account. Unless a lab or exam question specifically says otherwise, GPv2 is your answer.

🚨 **EXAM ALERT**
> **Blob Storage accounts are legacy.** If an exam question mentions creating a new storage account, the answer is almost always GPv2. Blob Storage accounts do not support Queue, Table, or File services. If a question requires any of those, Blob Storage is automatically wrong.

📝 **NOTE**
> The "kind" parameter in CLI commands maps to these types: `StorageV2` = GPv2, `BlobStorage` = Blob Storage, `BlockBlobStorage` = Premium Block Blobs, `FileStorage` = Premium File Shares. You will see `--kind StorageV2` in every lab command.

---

## Section 3: Creating a Storage Account via CLI

### The Problem Before the Solution

You could create a storage account through the Azure Portal by clicking through several form pages. But in AZ-204 labs, you will create storage accounts repeatedly — sometimes multiple times in a single lab. Clicking through forms every time is slow and error-prone. The CLI lets you create a storage account with a single command that you can copy, paste, and reuse.

### Prerequisites

Before running this command, you need:
1. Azure CLI installed (covered in F03)
2. Logged in: `az login` (covered in F03)
3. A resource group already created (covered in F05)

If you do not have a resource group yet, create one first:

```bash
# Create a resource group to hold the storage account
# --name       → the name you choose for the resource group
# --location   → the Azure region (datacenter location) to create it in
az group create \
  --name az204-storage-rg \
  --location eastus
```

### The Complete Storage Account Create Command

Here is the full command with every flag explained:

```bash
# Create an Azure Storage Account
az storage account create \
  --name az204store$RANDOM \
  --resource-group az204-storage-rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2
```

Let's break down every piece:

| Flag | Value | What It Means |
|------|-------|---------------|
| `az storage account create` | *(command)* | Tells Azure CLI: "Create a new storage account" |
| `--name az204store$RANDOM` | `az204store` + random number | The globally unique name for your account. `$RANDOM` is a built-in shell variable that generates a random number (e.g., `az204store18472`). This prevents "name already taken" errors. |
| `--resource-group az204-storage-rg` | `az204-storage-rg` | Which resource group to place this account in. The resource group must already exist. |
| `--location eastus` | `eastus` | The Azure region (datacenter) where your storage account will physically live. `eastus` is commonly used in labs because it has the widest service availability. |
| `--sku Standard_LRS` | `Standard_LRS` | The pricing tier and redundancy level. `Standard` = HDD-based storage (cheaper). `LRS` = Locally Redundant Storage (3 copies in one datacenter). Cheapest option — perfect for labs. |
| `--kind StorageV2` | `StorageV2` | The storage account type. `StorageV2` = General Purpose v2 (GPv2), the default and recommended type. |

### What `$RANDOM` Does

`$RANDOM` is a special variable available in Bash (the default shell in Azure Cloud Shell and most Linux/Mac terminals). Every time you reference it, it generates a random integer between 0 and 32767.

```bash
# Example: each time you run this, you get a different number
echo $RANDOM    # Output: 18472
echo $RANDOM    # Output: 7291
echo $RANDOM    # Output: 30154
```

We append it to the storage account name so you get a unique name like `az204store18472` instead of conflicting with someone else's `az204store`.

📝 **NOTE**
> In **PowerShell** on Windows, `$RANDOM` does not exist by default. Use `$(Get-Random)` instead, or use Azure Cloud Shell (which runs Bash). All lab commands in this course assume Bash or Azure Cloud Shell.

### What `Standard_LRS` Means

The `--sku` flag combines two pieces of information:

```
Standard_LRS
────────  ───
   │       │
   │       └── Redundancy: LRS (Locally Redundant Storage)
   │                       3 copies in ONE datacenter
   │
   └────────── Performance: Standard (HDD-based)
                           Cheaper, sufficient for most workloads
```

Other SKU combinations you might see:

| SKU | Performance | Redundancy |
|-----|-------------|------------|
| `Standard_LRS` | Standard (HDD) | 3 copies, 1 datacenter |
| `Standard_ZRS` | Standard (HDD) | 3 copies, 3 availability zones |
| `Standard_GRS` | Standard (HDD) | 6 copies, 2 regions |
| `Standard_GZRS` | Standard (HDD) | 6 copies, 3 zones + secondary region |
| `Premium_LRS` | Premium (SSD) | 3 copies, 1 datacenter |

💰 **COST WARNING**
> For all AZ-204 labs, use `Standard_LRS`. It is the cheapest option. Premium and geo-redundant options cost significantly more and provide no benefit for practice labs. Always delete lab storage accounts when finished.

### Expected Output

After running the create command, Azure CLI returns a JSON object with details about the newly created storage account. Here is a simplified version of what you will see:

```json
{
  "id": "/subscriptions/.../resourceGroups/az204-storage-rg/providers/Microsoft.Storage/storageAccounts/az204store18472",
  "kind": "StorageV2",
  "location": "eastus",
  "name": "az204store18472",
  "primaryEndpoints": {
    "blob": "https://az204store18472.blob.core.windows.net/",
    "file": "https://az204store18472.file.core.windows.net/",
    "queue": "https://az204store18472.queue.core.windows.net/",
    "table": "https://az204store18472.table.core.windows.net/"
  },
  "provisioningState": "Succeeded",
  "sku": {
    "name": "Standard_LRS"
  }
}
```

Notice the `primaryEndpoints` section — those are the four URLs for the four services inside your account. This confirms that one storage account gives you access to all four services.

### Saving the Account Name for Later

Since `$RANDOM` generates a different number each time, you should save the account name in a variable so you can reference it in later commands:

```bash
# Save the account name in a variable for reuse
# This creates the account AND stores the name in STORAGE_NAME
STORAGE_NAME="az204store$RANDOM"

# Now use the variable in the create command
az storage account create \
  --name $STORAGE_NAME \
  --resource-group az204-storage-rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2

# Verify the variable holds your account name
echo "Storage account created: $STORAGE_NAME"
```

---

✅ **CHECKPOINT**
Before moving on, make sure you can answer these:
1. Why do we append `$RANDOM` to the storage account name? *(To ensure global uniqueness and avoid "name already taken" errors)*
2. What does `Standard_LRS` mean? *(Standard performance tier with Locally Redundant Storage — 3 copies in one datacenter)*
3. What does `--kind StorageV2` specify? *(General Purpose v2, the default and recommended account type)*

---

## Section 4: Redundancy Options — The Exam Table

### The Problem

Hardware fails. Disks crash. Datacenters lose power. Entire regions can be disrupted by natural disasters. If your data only exists in one place, any failure means data loss. Azure Storage solves this by automatically copying your data — the question is: **how many copies, and where?**

### The Analogy: Copies of Your Important Documents

Think of redundancy like making copies of your passport:

- **LRS** = You make 3 photocopies and keep all of them in the same drawer at home. If your house is fine, you are safe. If there is a house fire, all copies are gone.
- **ZRS** = You make 3 photocopies and keep one at home, one at your office, and one at a friend's house across town. A house fire does not destroy all copies.
- **GRS** = You make 6 photocopies — 3 in your home city and 3 in a different city. Even a city-wide disaster does not destroy all copies.
- **GZRS** = You make 6 photocopies — 3 spread across different buildings in your city, and 3 more in a different city. Maximum protection.

### The Full Redundancy Table

This table is **exam-critical**. You need to know every row.

| SKU | Full Name | How It Works | Number of Copies | Protects Against | Cost | Exam Relevance |
|-----|-----------|-------------|-----------------|-----------------|------|----------------|
| **LRS** | Locally Redundant Storage | 3 copies in **one datacenter** in one region | 3 | Single disk or server failure | Lowest | 🔴 HIGH |
| **ZRS** | Zone-Redundant Storage | 3 copies across **3 availability zones** in one region | 3 | Entire datacenter failure (one zone goes down) | Medium | 🔴 HIGH |
| **GRS** | Geo-Redundant Storage | 3 copies in primary region + 3 copies in a **secondary region** (hundreds of miles away) | 6 | Regional disaster (earthquake, flood affecting entire region) | Higher | 🔴 HIGH |
| **GZRS** | Geo-Zone-Redundant Storage | 3 copies across **3 zones in primary** + 3 copies in **secondary region** | 6 | Zone failure + regional disaster (maximum protection) | Highest | 🔴 HIGH |

### The Secondary Region Read Access Problem

Here is where the exam loves to test you:

When you choose **GRS** or **GZRS**, Azure copies your data to a secondary region. But here is the critical detail — **you cannot read from the secondary region unless you explicitly enable read access**.

| SKU | Secondary Region Copies? | Can You Read from Secondary? |
|-----|-------------------------|------------------------------|
| LRS | No | N/A |
| ZRS | No | N/A |
| GRS | Yes | ❌ **NO** — secondary is write-only (for failover) |
| RA-GRS | Yes | ✅ **YES** — read-only access at `*-secondary` URL |
| GZRS | Yes | ❌ **NO** — secondary is write-only (for failover) |
| RA-GZRS | Yes | ✅ **YES** — read-only access at `*-secondary` URL |

🚨 **EXAM ALERT**
> **GRS replicates to a secondary region, but the secondary is NOT readable by default.** You must enable **RA-GRS** (Read-Access Geo-Redundant Storage) to read from the secondary. The exam tests this distinction frequently. If a question says "read data from the secondary region during an outage," the answer requires RA-GRS or RA-GZRS — plain GRS is not enough.

When RA-GRS is enabled, the secondary endpoint URL changes slightly:

```
Primary:   https://az204store18472.blob.core.windows.net
Secondary: https://az204store18472-secondary.blob.core.windows.net
                                   ^^^^^^^^^^
                                   "-secondary" is appended
```

💡 **KEY CONCEPT**
> **Use LRS for all labs.** It is the cheapest option and provides sufficient protection for practice data. You would never use LRS for production data that matters, but for labs that you will delete within hours, it saves money. The exam expects you to know all four tiers, but your labs should always use `Standard_LRS`.

### Choosing Redundancy — Decision Guide

| Your Situation | Recommended Redundancy |
|---------------|----------------------|
| Lab or development environment | LRS |
| Production, single region, data is replaceable | ZRS |
| Production, data must survive regional disaster | GRS or GZRS |
| Production, must read data during regional outage | RA-GRS or RA-GZRS |
| Compliance requires geographic data replication | GRS, GZRS, RA-GRS, or RA-GZRS |

---

## Section 5: Access Keys and Connection Strings

### The Problem

You have created a storage account. It is sitting in Azure, ready to store data. But how does your application actually connect to it? How does Azure know that YOUR application is allowed to access YOUR storage account, and not someone else's?

You need credentials — a way to prove identity.

### The Analogy: Master Keys to Your Building

Remember the building analogy from Section 1? Your storage account is a building with four rooms. **Access keys are the master keys to that building.** Anyone who has a copy of the key can open every room and do anything — read, write, delete, everything.

Azure gives you **two** master keys, not one.

### Why Two Keys?

Imagine you gave copies of your office key to 50 employees. Now the lock needs to be changed (maybe a key was compromised). If you only had one lock, you would need to:
1. Change the lock
2. Distribute 50 new keys
3. During the changeover, some people cannot get in

With **two locks** (two keys), you can:
1. Tell everyone to start using Key 2 instead of Key 1
2. Change Key 1 (regenerate it) — nobody is affected because they are using Key 2
3. Later, migrate everyone back to Key 1 and regenerate Key 2

This is called **key rotation**, and it is a security best practice.

### The Key Rotation Process

```
Step 1: Your app uses Key 1
Step 2: Update your app's configuration to use Key 2
Step 3: Regenerate Key 1 (old Key 1 is now invalid)
Step 4: Later, update your app to use the new Key 1
Step 5: Regenerate Key 2
Result: Both keys have been rotated, zero downtime
```

### Finding Your Access Keys

To view your storage account's access keys:

**Portal path:** **Storage Account** → **Security + networking** → **Access keys**

Or via CLI:

```bash
# List access keys for your storage account
# --account-name → the name of your storage account
# --resource-group → the resource group containing the account
# --output table → display results in an easy-to-read table format
az storage account keys list \
  --account-name $STORAGE_NAME \
  --resource-group az204-storage-rg \
  --output table
```

Expected output:

```
KeyName    Permissions    Value
---------  -------------  ----------------------------------------------------------------
key1       Full           a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6A7B8C9==
key2       Full           z9y8x7w6v5u4t3s2r1q0p9o8n7m6l5k4j3i2h1g0f9e8d7c6b5a4Z3Y2X1==
```

### The Connection String

An access key by itself is just a long random string. Your application also needs to know the account name and which endpoints to connect to. A **connection string** bundles all of this information into a single string that SDKs can parse.

Here is the full format of a storage account connection string, with each part labelled:

```
DefaultEndpointsProtocol=https;AccountName=az204store18472;AccountKey=a1b2c3...X1==;EndpointSuffix=core.windows.net
│                              │                            │                      │
│                              │                            │                      └── Domain suffix for Azure endpoints
│                              │                            └── The actual access key (master password)
│                              └── Your storage account name
└── Protocol: always use HTTPS for security
```

To retrieve the connection string via CLI:

```bash
# Get the full connection string for your storage account
# --name → the storage account name
# --resource-group → the resource group
# --query → JMESPath query to extract just the connection string value
#           "connectionStrings[0].connectionString" navigates the JSON:
#           → "connectionStrings" = array of connection string objects
#           → "[0]" = first item in the array
#           → ".connectionString" = the actual string value
# --output tsv → output as plain text (no quotes), easy to copy-paste
az storage account show-connection-string \
  --name $STORAGE_NAME \
  --resource-group az204-storage-rg \
  --query "connectionString" \
  --output tsv
```

### Storing the Connection String Safely

```bash
# Save the connection string in an environment variable
# This keeps it out of your code files
STORAGE_CONNECTION=$(az storage account show-connection-string \
  --name $STORAGE_NAME \
  --resource-group az204-storage-rg \
  --query "connectionString" \
  --output tsv)

# Verify it was saved
echo $STORAGE_CONNECTION
```

🚨 **SECURITY WARNING**
> **Never put access keys or connection strings directly in your code.** If your code is pushed to GitHub, anyone who finds it has full access to your storage account — they can read, modify, or delete everything. Instead:
>
> - **In local development:** Store in environment variables or `local.settings.json` (which is in `.gitignore`)
> - **In Azure:** Store in App Settings (which become environment variables) or Key Vault references
> - **In production:** Use **Managed Identity** with `DefaultAzureCredential` — no keys needed at all
>
> 🔗 **MODULE LINK:** Key Vault (Module 7) covers secure secret storage. Managed Identity (Module 6) covers keyless authentication.

💡 **KEY CONCEPT**
> Access keys provide **full, unrestricted access** to your entire storage account — all four services, all operations. They are the most powerful (and most dangerous) credential. For finer-grained access control, use SAS tokens (next section) or RBAC roles.

---

✅ **CHECKPOINT**
Before moving on, make sure you can answer these:
1. Why does Azure give you TWO access keys? *(To enable key rotation without downtime)*
2. What four pieces of information does a connection string contain? *(Protocol, account name, account key, endpoint suffix)*
3. Where should you NEVER store a connection string? *(Directly in your source code)*

---

## Section 6: SAS Tokens at a Glance

### The Problem

Access keys give **full access** to everything in your storage account. But what if you want to give someone limited access? For example:

- A web application that should **only read** blobs from one container
- A partner company that needs **upload-only** access for 24 hours
- A mobile app that needs to download a **specific file** for 1 hour

Handing out your access key for any of these scenarios is like giving someone a master key to your building when they only need to enter one room for one hour.

### The Analogy: Visitor Passes

A SAS token (Shared Access Signature) is like a **visitor pass** to your building:

| Feature | Access Key (Master Key) | SAS Token (Visitor Pass) |
|---------|------------------------|-------------------------|
| Access level | Everything, all rooms, all actions | Specific rooms, specific actions |
| Expiration | Never expires (until rotated) | Has a set expiry time |
| Revocation | Regenerate key (breaks ALL users) | Let it expire, or revoke stored access policy |
| Who uses it | Your own applications | External users, limited-access apps |

💡 **KEY CONCEPT**
> A **SAS token is a limited version of an access key.** It grants specific permissions to specific resources for a specific time period. Think of it as a scoped, time-limited credential.

### Three Types of SAS

| SAS Type | Signed With | Scope | Security Level | Exam Relevance |
|----------|------------|-------|----------------|----------------|
| **Service SAS** | Storage account key | One service (Blob, Queue, Table, or File) | Medium — relies on account key | 🟡 MEDIUM |
| **Account SAS** | Storage account key | One or more services, with additional operations | Medium — relies on account key | 🟡 MEDIUM |
| **User Delegation SAS** | Entra ID credentials (OAuth) | Blob and Data Lake only | **Highest** — no account key involved | 🔴 HIGH |

🚨 **EXAM ALERT**
> **User Delegation SAS is the most secure type** because it is signed with Entra ID credentials (your identity), not the storage account key. If the account key is compromised, User Delegation SAS tokens are unaffected. The exam frequently asks which SAS type is "most secure" or "recommended" — the answer is always User Delegation SAS.

### SAS Token Parameters

Every SAS token includes these parameters that control exactly what the holder can do:

| Parameter | What It Controls | Example |
|-----------|-----------------|---------|
| **Permissions** | Which operations are allowed | `r` = read, `w` = write, `d` = delete, `l` = list |
| **Start time** | When the token becomes valid | `2025-01-15T08:00:00Z` |
| **Expiry time** | When the token stops working | `2025-01-15T20:00:00Z` |
| **IP address** | Which IP addresses can use the token | `168.1.5.65` or `168.1.5.60-168.1.5.70` |
| **Protocol** | HTTP or HTTPS required | `https` (always use HTTPS) |
| **Services** | Which services (for Account SAS) | `b` = blob, `q` = queue, `t` = table, `f` = file |
| **Resource types** | Service, container, or object level | `s` = service, `c` = container, `o` = object |

### What a SAS Token Looks Like

A SAS token is a query string appended to a storage URL. Here is an example with each parameter labelled:

```
https://az204store18472.blob.core.windows.net/photos/sunset.jpg?
  sv=2022-11-02        ← Signed version (API version)
  &ss=b                ← Signed services: b = blob only
  &srt=o               ← Signed resource type: o = object (individual blob)
  &sp=r                ← Signed permissions: r = read only
  &se=2025-01-15T20:00:00Z  ← Signed expiry: token expires at this time
  &st=2025-01-15T08:00:00Z  ← Signed start: token valid from this time
  &sip=168.1.5.65      ← Signed IP: only this IP can use the token
  &spr=https           ← Signed protocol: HTTPS required
  &sig=abc123...       ← Signature: cryptographic proof of authenticity
```

Anyone who has this URL can read the `sunset.jpg` file — but **only** read, **only** from IP `168.1.5.65`, **only** via HTTPS, and **only** until 8:00 PM on January 15, 2025.

### Generating a SAS Token via CLI

```bash
# Generate a SAS token for the entire storage account (Account SAS)
# --account-name → which storage account
# --account-key → the access key used to sign the token
# --services b → blob service only
# --resource-types co → container + object level access
# --permissions rl → read + list
# --expiry → when the token stops working (ISO 8601 format)
# --https-only → require HTTPS connections
az storage account generate-sas \
  --account-name $STORAGE_NAME \
  --account-key $STORAGE_KEY \
  --services b \
  --resource-types co \
  --permissions rl \
  --expiry 2025-12-31T23:59:59Z \
  --https-only \
  --output tsv
```

🔗 **MODULE LINK**
> SAS tokens are covered in full detail in **Module 4 (Blob Storage)**. This section gives you the overview needed to understand what SAS tokens are and when to use them. Module 4 will include hands-on labs for generating and testing SAS tokens.

---

## Section 7: Azure Storage Explorer

### The Problem

You have created a storage account, but how do you actually see what is inside it? The CLI can list blobs, but reading JSON output is not intuitive when you just want to browse, upload a file, or check whether your code created a container correctly.

### The Analogy: File Explorer for the Cloud

Azure Storage Explorer is like **Windows File Explorer (or Mac Finder), but for your cloud storage**. Instead of browsing folders on your local computer, you browse containers, blobs, queues, and tables in your Azure storage accounts — with a familiar drag-and-drop interface.

### What It Is

Azure Storage Explorer is a **free, cross-platform desktop application** (Windows, Mac, Linux) that lets you visually manage Azure Storage resources. You can:

- Browse containers and blobs (like browsing folders and files)
- Upload and download files (drag and drop or click)
- Create and delete containers, queues, and tables
- View and edit blob properties and metadata
- Generate SAS tokens through a visual interface
- Connect to multiple storage accounts across multiple subscriptions

### Download and Install

1. Go to **https://azure.microsoft.com/products/storage/storage-explorer**
2. Click **Download** for your operating system
3. Run the installer — accept defaults
4. Launch Azure Storage Explorer

📝 **NOTE**
> Storage Explorer requires **.NET** to be installed. On Windows, the installer handles this automatically. On Mac/Linux, you may need to install it separately if prompted.

### How to Connect — Three Methods

Storage Explorer supports three connection methods. Here they are from simplest to most secure:

| Method | How | Best For | Security |
|--------|-----|----------|----------|
| **Azure account sign-in** | Click "Add an account" → sign in with your Azure credentials | Your own accounts during labs | Good — uses your Azure RBAC permissions |
| **Connection string** | Click "Connect" → select "Storage account" → paste connection string | Quick access or shared lab environments | Medium — connection string has full access |
| **SAS URI** | Click "Connect" → select "SAS URI" → paste the SAS URL | External access or limited permissions | Best — limited, time-scoped access |

### Step-by-Step: Connecting with Your Azure Account

This is the recommended method for AZ-204 labs:

1. Open Azure Storage Explorer
2. Look at the left panel — click the **plug icon** (🔌) labelled "Connect to Azure Storage"
3. Select **"Subscription"**
4. Choose **"Azure"** as the environment
5. Click **"Next"** — your browser opens for sign-in
6. Sign in with the same email and password you use for the Azure Portal
7. After signing in, close the browser tab and return to Storage Explorer
8. You will see your subscriptions listed — check the ones you want to browse
9. Click **"Open Explorer"**
10. In the left panel, expand your subscription → expand **Storage Accounts** → expand your account → you will see **Blob Containers**, **Queues**, **Tables**, and **File Shares**

### When to Use Storage Explorer in AZ-204 Labs

| Scenario | Use Storage Explorer? |
|----------|---------------------|
| Verify that your code created a container correctly | ✅ Yes — browse and confirm |
| Check whether a blob was uploaded by your SDK code | ✅ Yes — find and preview the file |
| View queue messages sent by your application | ✅ Yes — inspect message content |
| Upload a test file before running download code | ✅ Yes — drag and drop |
| Create storage accounts or configure redundancy | ❌ No — use CLI or Portal |
| Automate bulk file operations | ❌ No — use AzCopy (next section) |

💡 **KEY CONCEPT**
> Storage Explorer is your **verification tool**. After running lab code that creates containers, uploads blobs, or sends queue messages, switch to Storage Explorer to visually confirm that everything worked. It is much faster than running CLI list commands for quick checks.

---

✅ **CHECKPOINT**
Before moving on, make sure you can answer these:
1. What is Azure Storage Explorer? *(A free desktop app that lets you visually browse and manage Azure Storage resources)*
2. What are three ways to connect to a storage account in Storage Explorer? *(Azure account sign-in, connection string, SAS URI)*
3. When should you use Storage Explorer in AZ-204 labs? *(To verify that code created containers, uploaded blobs, or sent queue messages correctly)*

---

## Section 8: AzCopy — Moving Files at Scale

### The Problem

You have a folder with 10,000 log files totalling 50 GB. You need to upload all of them to Azure Blob Storage. Using Storage Explorer's drag-and-drop for 10,000 files would be painfully slow. Writing SDK code to upload each file individually works, but it is unnecessary for a bulk transfer operation. You need a tool designed specifically for moving large amounts of data quickly.

### The Analogy: Hiring a Moving Company

If Storage Explorer is like carrying boxes to your car yourself (fine for a few items), AzCopy is like **hiring a professional moving company**. It is optimized for speed, handles thousands of files efficiently, supports resume-on-failure, and can transfer between Azure accounts directly.

### What Is AzCopy?

AzCopy is a **command-line tool** designed for high-performance data transfers to and from Azure Storage. It is NOT part of the Azure CLI — it is a separate tool.

Key features:
- Copies data to/from Blob and File storage
- Supports local-to-Azure, Azure-to-local, and Azure-to-Azure transfers
- Parallelizes transfers automatically for speed
- Resumes interrupted transfers
- Supports SAS tokens and Entra ID authentication

### Installing AzCopy

AzCopy is a standalone download — it does not install through `npm` or `dotnet`:

1. Go to **https://learn.microsoft.com/azure/storage/common/storage-use-azcopy-v10**
2. Download the version for your operating system
3. Extract the archive
4. Add the `azcopy` (or `azcopy.exe`) location to your system PATH

Verify the installation:

```bash
# Check that AzCopy is installed and see the version
azcopy --version
# Expected output example: azcopy version 10.22.1
```

### Authenticating AzCopy

Before copying files, AzCopy needs permission to access your storage account. Two methods:

```bash
# Method 1: Login with Entra ID (recommended for security)
# This opens a browser for interactive sign-in
azcopy login

# Method 2: Use SAS tokens appended to the URL (no login needed)
# The SAS token is included directly in the destination URL
# See examples below
```

### Key AzCopy Commands

#### Upload a Single File

```bash
# Copy a single file from local machine to a blob container
# azcopy copy → the command for all copy operations
# "local-file.txt" → the source: a file on your local machine
# "https://...?SAS" → the destination: blob container URL with SAS token appended
azcopy copy \
  "local-file.txt" \
  "https://az204store18472.blob.core.windows.net/uploads/local-file.txt?sv=2022-11-02&ss=b&sp=rcw&se=2025-12-31..."
```

#### Upload an Entire Folder

```bash
# Copy an entire local folder to a blob container
# --recursive → include all subfolders and their contents
#               without this flag, only files in the top-level folder are copied
azcopy copy \
  "/path/to/local-folder" \
  "https://az204store18472.blob.core.windows.net/backups?SAS_TOKEN" \
  --recursive
```

#### Download Blobs to Local Machine

```bash
# Download all blobs from a container to a local folder
# Source and destination are reversed compared to upload
# --recursive → download contents of all virtual subdirectories
azcopy copy \
  "https://az204store18472.blob.core.windows.net/logs?SAS_TOKEN" \
  "/path/to/local-folder" \
  --recursive
```

#### Copy Between Azure Storage Accounts (Server-Side)

```bash
# Copy blobs between two Azure storage accounts
# This is SERVER-SIDE — data goes directly between accounts
# Your local machine does NOT download and re-upload the data
# Both source and destination need SAS tokens (or use azcopy login)
azcopy copy \
  "https://sourceaccount.blob.core.windows.net/data?SOURCE_SAS" \
  "https://destaccount.blob.core.windows.net/data?DEST_SAS" \
  --recursive
```

#### Sync a Local Folder with a Container

```bash
# Sync local folder with blob container
# --delete-destination=true → deletes blobs in container that
#                             don't exist locally (mirror sync)
# Unlike 'copy', 'sync' only transfers files that are new or changed
azcopy sync \
  "/path/to/local-folder" \
  "https://az204store18472.blob.core.windows.net/website?SAS_TOKEN" \
  --delete-destination=true
```

### AzCopy vs Other Tools — When to Use What

| Tool | Best For | Speed | Interface |
|------|----------|-------|-----------|
| **Azure CLI** (`az storage blob upload`) | Single files, scripted operations | Moderate | Command line |
| **Storage Explorer** | Visual browsing, small uploads, verification | Slow for bulk | GUI (graphical) |
| **AzCopy** | Bulk transfers, large files, account-to-account copies | **Fastest** | Command line |
| **SDK code** (C#/Python/Node) | Programmatic uploads within applications | Moderate | Code |

🚨 **EXAM ALERT**
> **AzCopy is the recommended tool for bulk data migration to Azure Storage.** If an exam question describes a scenario involving large-scale file transfers (thousands of files, terabytes of data, migration from on-premises to Azure), the answer is AzCopy. Not Storage Explorer, not the Azure CLI, not the SDK.

🚨 **EXAM ALERT**
> **AzCopy server-side copy (Azure-to-Azure) does NOT route data through your local machine.** Data flows directly between storage accounts on Azure's backbone network. This is critical for migration scenarios where bandwidth is limited.

📝 **NOTE**
> AzCopy is also used internally by Azure Data Box and Azure Storage Migration tools. Understanding AzCopy gives you insight into how Azure handles large-scale data movement under the hood.

---

## 🔬 Hands-On Exercise: Your First Storage Account End-to-End

**Estimated time:** 30 minutes
**Cost:** Under $0.01 (delete resources when finished)
**What you will do:** Create a storage account, find its connection string, install Storage Explorer, connect, create a container, upload a file, verify via CLI, and clean up.

---

### Step 1: Create a Resource Group

Every Azure resource lives inside a resource group. Create one for this exercise:

```bash
# Create a resource group for this exercise
# --name → a descriptive name for the resource group
# --location → Azure region (eastus is widely available)
az group create \
  --name az204-storage-lab-rg \
  --location eastus
```

**Expected output:** JSON with `"provisioningState": "Succeeded"`

---

### Step 2: Create a Storage Account

```bash
# Store the name in a variable so we can reuse it
STORAGE_NAME="az204lab$RANDOM"

# Create the storage account
az storage account create \
  --name $STORAGE_NAME \
  --resource-group az204-storage-lab-rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2

# Print the name so you know what was created
echo "✅ Storage account created: $STORAGE_NAME"
```

**Expected output:** A large JSON object with `"provisioningState": "Succeeded"` and your `primaryEndpoints` listed.

---

### Step 3: Find the Connection String

```bash
# Retrieve and save the connection string
CONN_STRING=$(az storage account show-connection-string \
  --name $STORAGE_NAME \
  --resource-group az204-storage-lab-rg \
  --query "connectionString" \
  --output tsv)

# Print it (you will need this for Storage Explorer)
echo "Connection string: $CONN_STRING"
```

**Expected output:** A long string starting with `DefaultEndpointsProtocol=https;AccountName=...`

📝 **NOTE**
> Copy this connection string and save it somewhere temporarily (a text file or clipboard). You will paste it into Storage Explorer in the next step.

---

### Step 4: Install and Open Azure Storage Explorer

If you have not installed it yet:

1. Go to **https://azure.microsoft.com/products/storage/storage-explorer**
2. Download and install for your operating system
3. Launch the application

If you already have it installed, open it now.

---

### Step 5: Connect Storage Explorer to Your Account

**Method A: Sign in with your Azure account (easiest)**

1. In Storage Explorer, click the **plug icon** (🔌) in the left toolbar
2. Select **"Subscription"**
3. Select **"Azure"** → click **Next**
4. Sign in with your Azure credentials in the browser window
5. Return to Storage Explorer → check your subscription → click **Open Explorer**
6. Expand: **Your Subscription** → **Storage Accounts** → **your account name**

**Method B: Use connection string (if sign-in has issues)**

1. Click the **plug icon** (🔌)
2. Select **"Storage account or service"**
3. Select **"Connection string"** → click **Next**
4. Paste the connection string from Step 3
5. Click **Connect**

---

✅ **CHECKPOINT**
> At this point, you should see your storage account in the left panel of Storage Explorer, with expandable sections for **Blob Containers**, **Queues**, **Tables**, and **File Shares**. If you do not see it, re-check your connection method.

---

### Step 6: Create a Container via CLI

A container is like a folder inside Blob Storage. You must create a container before you can upload blobs.

```bash
# Create a blob container named "lab-files"
# --account-name → which storage account to create it in
# --name → the container name (lowercase, hyphens allowed)
# --auth-mode login → use your Azure CLI login credentials
#                     (instead of passing an access key)
az storage container create \
  --account-name $STORAGE_NAME \
  --name lab-files \
  --auth-mode login
```

**Expected output:**

```json
{
  "created": true
}
```

📝 **NOTE**
> If you get a permissions error with `--auth-mode login`, you may need to assign yourself the "Storage Blob Data Contributor" role, or use `--connection-string $CONN_STRING` instead:
>
> ```bash
> az storage container create \
>   --name lab-files \
>   --connection-string "$CONN_STRING"
> ```

---

### Step 7: Verify Container in Storage Explorer

Switch to Azure Storage Explorer:

1. In the left panel, find your storage account
2. Expand **Blob Containers**
3. You should see the **lab-files** container you just created
4. Click on it — the right panel shows it is empty (no blobs yet)

---

### Step 8: Create a Test File and Upload It

First, create a simple text file to upload:

```bash
# Create a simple text file
echo "Hello from AZ-204 Storage Lab! Created on $(date)" > hello.txt

# Verify the file was created
cat hello.txt
```

Now upload it to your container:

```bash
# Upload the file to the lab-files container
# --container-name → which container to upload into
# --file → path to the local file
# --name → what to name the blob in Azure (can differ from local filename)
# --account-name → which storage account
# --auth-mode login → authenticate with Azure CLI credentials
az storage blob upload \
  --container-name lab-files \
  --file hello.txt \
  --name hello.txt \
  --account-name $STORAGE_NAME \
  --auth-mode login
```

**Expected output:** JSON with blob properties including `"content_length"`, `"last_modified"`, and `"etag"`.

📝 **NOTE**
> If `--auth-mode login` gives a permissions error, use the connection string method:
>
> ```bash
> az storage blob upload \
>   --container-name lab-files \
>   --file hello.txt \
>   --name hello.txt \
>   --connection-string "$CONN_STRING"
> ```

---

### Step 9: Verify the Upload via CLI

```bash
# List all blobs in the lab-files container
# --output table → show results as a readable table
az storage blob list \
  --container-name lab-files \
  --account-name $STORAGE_NAME \
  --auth-mode login \
  --output table
```

**Expected output:**

```
Name        Blob Type    Blob Tier    Length    Content Type              Last Modified
----------  -----------  -----------  --------  ------------------------  -------------------------
hello.txt   BlockBlob    Hot          52        application/octet-stream  2025-01-15T14:30:22+00:00
```

---

### Step 10: Verify the Upload in Storage Explorer

Switch to Azure Storage Explorer:

1. In the left panel, click on your **lab-files** container
2. If needed, click the **Refresh** button (🔄) in the toolbar
3. You should see **hello.txt** listed in the right panel
4. Double-click it to download and view the contents
5. Verify it contains the text "Hello from AZ-204 Storage Lab!"

---

✅ **CHECKPOINT**
> You have now completed the full cycle: created a storage account via CLI, found its connection string, connected with Storage Explorer, created a container, uploaded a file, and verified it both via CLI and Storage Explorer. This is the exact workflow you will use in Module 4 (Blob Storage) labs.

---

### Step 11: Clean Up — Delete All Resources

Always delete lab resources to avoid charges:

```bash
# Delete the entire resource group and everything inside it
# --yes → skip the confirmation prompt
# --no-wait → don't wait for deletion to complete (runs in background)
az group delete \
  --name az204-storage-lab-rg \
  --yes \
  --no-wait
```

Verify the resource group is being deleted:

```bash
# List resource groups to confirm deletion is in progress
# The group should show "Deleting" status or be gone entirely
az group list --output table
```

Also clean up local files:

```bash
# Remove the test file you created
rm hello.txt
```

💰 **COST WARNING**
> Even `Standard_LRS` storage accounts cost money over time. The cost is tiny (fractions of a cent per day for an empty account), but it adds up if you forget about it for months. **Always delete lab resource groups when finished.** If you want to be extra safe, set up a budget alert (covered in F02).

---

## 🔗 Module Connections

This foundation document connects to these AZ-204 modules:

| Module | Connection |
|--------|-----------|
| **Module 2 — Azure Functions** | Every Function App requires a storage account. You now know why and how to create one. |
| **Module 4 — Blob Storage** | The entire module is about one of the four services inside a storage account. Containers, SAS tokens, and access tiers are covered in depth. |
| **Module 5 — Cosmos DB** | Labs often use storage accounts alongside Cosmos DB for file storage. |
| **Module 7 — Key Vault** | Connection strings should be stored in Key Vault, not in code. |
| **Modules 9–10 — Event Grid / Service Bus** | Events often reference blobs in storage accounts. |

---

## ✅ What We Covered

Use this checklist to confirm you understand everything in this document:

- [ ] **Storage Account concept** — One Azure resource that provides four services: Blob, Queue, Table, File
- [ ] **URL pattern** — Account name becomes the subdomain: `https://<name>.blob.core.windows.net`
- [ ] **Function App dependency** — Every Azure Function App requires a storage account
- [ ] **Account types** — GPv2 (StorageV2) is the default for everything; Blob Storage accounts are legacy
- [ ] **CLI creation** — `az storage account create` with `--name`, `--resource-group`, `--location`, `--sku`, `--kind` flags
- [ ] **$RANDOM** — Bash variable that generates a random number for unique naming
- [ ] **Standard_LRS** — Standard performance + Locally Redundant Storage (3 copies, 1 datacenter)
- [ ] **Redundancy tiers** — LRS (1 datacenter), ZRS (3 zones), GRS (2 regions), GZRS (3 zones + 2 regions)
- [ ] **RA-GRS** — Required to READ from the secondary region; plain GRS secondary is NOT readable
- [ ] **Access keys** — Two master keys for rotation without downtime
- [ ] **Connection strings** — Bundle protocol + account name + key + endpoint suffix into one string
- [ ] **Security rule** — Never put keys or connection strings in source code
- [ ] **SAS tokens** — Limited, time-scoped credentials (visitor passes vs. master keys)
- [ ] **Three SAS types** — Service SAS, Account SAS, User Delegation SAS (most secure)
- [ ] **User Delegation SAS** — Signed with Entra ID, not account key — most secure
- [ ] **Azure Storage Explorer** — Free desktop app for visual storage management and verification
- [ ] **AzCopy** — CLI tool for high-performance bulk data transfers
- [ ] **AzCopy exam fact** — Recommended tool for bulk data migration to Azure Storage
- [ ] **Hands-on completed** — Created account, found connection string, connected Explorer, uploaded file, verified, cleaned up
