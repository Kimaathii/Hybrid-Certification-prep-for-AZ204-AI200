# M04 — Concept: Azure Blob Storage

---

| | |
|---|---|
| **Module** | M04 — Azure Blob Storage |
| **Exam Domain** | Develop for Azure Storage (15–20%) |
| **Exam Weight** | 🔴 HIGH |
| **Study Time** | 3–4 hours |
| **Prerequisites** | F01–F08 (especially F08 — Storage Account Basics) |
| **Key SDK Package** | `Azure.Storage.Blobs` |

---

## Section 1: What Is Blob Storage? 🔴 HIGH EXAM WEIGHT

### The Problem

Imagine you are building a web application. Users upload profile photos, documents, and videos. Where do you store these files?

You cannot put them inside your web app's file system — when the app scales to multiple instances, each instance has its own file system. File uploaded to instance A would not be visible to instance B. You need a **centralised, scalable file storage system** that all instances can access.

### The Solution: Azure Blob Storage

**Azure Blob Storage** is Azure's object storage solution for the cloud. Think of it as **Google Drive for your applications** — a massive, scalable storage system where your code can upload, download, list, and manage files programmatically.

> 💡 **KEY CONCEPT**
>
> **Blob** stands for **Binary Large Object** — it is just a fancy name for "file." A blob can be any type of file: an image, a video, a PDF, a log file, a JSON data export, a database backup. Azure Blob Storage stores these files and gives your code an API to manage them.

### Three Types of Blobs

Azure has three blob types, each optimised for different scenarios:

| Blob Type | What It Stores | How It Works | Example |
|---|---|---|---|
| **Block Blob** | Files and media (most common) | Made of blocks that can be uploaded in parallel. Optimised for upload/download of large files. | Profile photos, videos, documents, application data |
| **Append Blob** | Log data | Only allows appending new data at the end. Cannot modify existing content. Optimised for write-heavy, sequential operations. | Application logs, audit trails, telemetry data |
| **Page Blob** | VM disks | Allows random read/write access at any position. Optimised for frequent random access. | Azure VM OS disks, data disks |

> 🚨 **EXAM ALERT**
>
> **Block Blobs** are the default and most common type — used in 95% of AZ-204 scenarios. **Append Blobs** are for logs (write-only, append-only). **Page Blobs** are for VM disks (random access). If the exam mentions "log files that are continuously written to" → the answer is Append Blob.

---

## Section 2: The Blob Storage Hierarchy 🔴 HIGH EXAM WEIGHT

Blob Storage has a three-level hierarchy. Think of it like a filing system:

```
Storage Account          ← The filing cabinet (one per project)
  └── Container          ← A drawer in the cabinet (groups related files)
       └── Blob          ← A file in the drawer (the actual data)
```

| Level | What It Is | Analogy | Naming Rules |
|---|---|---|---|
| **Storage Account** | Top-level Azure resource. Must be globally unique. | The filing cabinet | 3–24 chars, lowercase + numbers only |
| **Container** | A folder-like grouping for blobs. Lives inside a storage account. | A drawer | 3–63 chars, lowercase + numbers + hyphens |
| **Blob** | The actual file. Lives inside a container. | A document | Up to 1,024 chars, case-sensitive |

**URL Pattern:**

```
https://{storage-account}.blob.core.windows.net/{container}/{blob}
```

**Example:**

```
https://az204storage.blob.core.windows.net/profile-photos/user123/avatar.jpg
```

> 📝 **NOTE**
>
> Containers are **flat** — there are no real subfolders. But you can simulate folders using `/` in blob names. `user123/avatar.jpg` looks like a folder structure, but it is actually a single blob with a name that contains a slash.

> ✅ **CHECKPOINT 1**
>
> 1. What are the three levels of the Blob Storage hierarchy?
> 2. What blob type would you use for application log files?
> 3. What does the URL `https://myaccount.blob.core.windows.net/images/photo.jpg` tell you about the storage structure?
>
> **Answers:** 1) Storage Account → Container → Blob. 2) Append Blob. 3) Storage account = `myaccount`, Container = `images`, Blob = `photo.jpg`.

---

## Section 3: Access Tiers — Where Cost Meets Performance 🔴 HIGH EXAM WEIGHT

### The Problem

Not all data is accessed equally. Your users' profile photos are viewed daily (hot data). Last year's financial reports are rarely opened (cool data). Seven-year-old compliance records must be kept but are almost never accessed (cold/archive data).

Storing everything at the same performance level wastes money.

### The Solution: Access Tiers

Azure Blob Storage offers four access tiers that balance **storage cost** vs **access cost**:

| Tier | Access Frequency | Min. Duration | Storage Cost | Access Cost | Retrieval Time |
|---|---|---|---|---|---|
| **Hot** | Frequently accessed | None | 💰💰💰 Highest | 💰 Lowest | Instant |
| **Cool** | Infrequently (30+ days) | 30 days | 💰💰 Medium | 💰💰 Medium | Instant |
| **Cold** | Rarely (90+ days) | 90 days | 💰 Low | 💰💰💰 Higher | Instant |
| **Archive** | Almost never (180+ days) | 180 days | 💰 Lowest | 💰💰💰💰 Highest | ⏱ 1–15 hours |

Think of it like parking:
- **Hot** = valet parking (expensive to park, instant to retrieve your car)
- **Cool** = nearby parking garage (cheaper to park, you walk a bit)
- **Cold** = remote lot (very cheap to park, longer walk)
- **Archive** = deep warehouse storage (cheapest to store, takes hours to retrieve)

> 🚨 **EXAM ALERT — Archive Tier Rehydration**
>
> Archive blobs are **offline** — they cannot be read directly. To access an archived blob, you must **rehydrate** it (move it back to Hot, Cool, or Cold first):
> - **Standard priority:** Up to **15 hours**
> - **High priority:** Up to **1 hour** (more expensive)
>
> This is a heavily tested concept. If the exam asks "How long does it take to access an archived blob?" → the answer is NOT instant — it requires rehydration.

> 🚨 **EXAM ALERT — Early Deletion Penalty**
>
> If you delete or move a blob before its minimum retention period:
> - Cool: charged for the remaining days of the 30-day minimum
> - Cold: charged for the remaining days of the 90-day minimum
> - Archive: charged for the remaining days of the 180-day minimum
>
> Moving a blob from Cool to Hot after only 5 days incurs an early deletion charge for the remaining 25 days.

---

## Section 4: Lifecycle Management Policies 🟡 MEDIUM EXAM WEIGHT

### The Problem

You have millions of blobs. Manually moving data between tiers is impossible. You need automation.

### The Solution: Lifecycle Management Policies

Lifecycle management lets you define **rules** that automatically transition blobs between tiers or delete them based on age.

Think of it like a **library return policy** — books that haven't been checked out in 30 days go to the back shelves (Cool). Books untouched for 90 days go to the warehouse (Cold). Books untouched for a year are recycled (deleted).

**Example policy (JSON):**

```json
{
  "rules": [
    {
      "name": "move-to-cool-after-30-days",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 180
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        }
      }
    }
  ]
}
```

This rule says: for all block blobs in the `logs/` prefix → move to Cool after 30 days → move to Archive after 180 days → delete after 365 days.

> 🚨 **EXAM ALERT**
>
> Lifecycle management policies run **once per day**. They are NOT real-time. If you create a rule to move blobs to Cool after 30 days, the blob might not move until the next daily run — up to 24 hours after the 30-day mark.

---

## Section 5: Access Control 🔴 HIGH EXAM WEIGHT

### Container Public Access Levels

Each container has a public access level that controls whether anonymous (unauthenticated) users can access blobs:

| Level | What Anonymous Users Can Do | Use Case |
|---|---|---|
| **Private** (default) | ❌ Nothing — all requests require authentication | Sensitive data, application files |
| **Blob** | ✅ Read individual blobs if they know the exact URL | Public images with known URLs |
| **Container** | ✅ Read blobs AND list all blobs in the container | Public file hosting |

> 🚨 **EXAM ALERT**
>
> The default is **Private**. Anonymous access requires explicit configuration. Most production scenarios use Private access with authentication (SAS tokens or RBAC).

### RBAC (Role-Based Access Control)

RBAC assigns roles to users, groups, or managed identities at the storage account or container level:

| Role | Permissions |
|---|---|
| **Storage Blob Data Owner** | Full access (read, write, delete, manage permissions) |
| **Storage Blob Data Contributor** | Read, write, delete blobs (no permission management) |
| **Storage Blob Data Reader** | Read-only access to blobs |

### SAS Tokens (Shared Access Signatures)

A **SAS token** is like a **visitor pass** — it grants limited access to specific resources for a limited time.

| SAS Type | What It Is | Security Level |
|---|---|---|
| **Service SAS** | Signed with storage account key. Grants access to one service (Blob, Queue, Table, or File). | 🟡 Medium — depends on key security |
| **Account SAS** | Signed with storage account key. Grants access across multiple services. | 🟡 Medium — broader scope than Service SAS |
| **User Delegation SAS** | Signed with Microsoft Entra ID credentials (no storage key needed). Grants access to Blob and Data Lake Storage only. | 🟢 Most secure — recommended |

> 🚨 **EXAM ALERT**
>
> **User Delegation SAS** is the **most secure** type because it does NOT use storage account keys. It is signed with Microsoft Entra ID (Azure AD) credentials and requires RBAC. The exam frequently asks which SAS type is "most secure" or "recommended for production" → answer is User Delegation SAS.

---

## Section 6: SDK Operations — The Code You Will Write 🔴 HIGH EXAM WEIGHT

### The Three-Client Hierarchy

The Azure Blob Storage SDK has three client classes that match the three-level hierarchy:

```
BlobServiceClient       ← Connects to the Storage Account level
  └── BlobContainerClient  ← Works with a specific Container
       └── BlobClient      ← Works with a specific Blob
```

### Install the SDK

```bash
dotnet add package Azure.Storage.Blobs    # ← The main Blob Storage SDK package
dotnet add package Azure.Identity          # ← For DefaultAzureCredential auth
```

### Complete SDK Example — Upload, List, Download, Delete

```csharp
using Azure.Identity;                      // ← For DefaultAzureCredential
using Azure.Storage.Blobs;                 // ← Blob Storage SDK
using Azure.Storage.Blobs.Models;          // ← Model classes (BlobItem, etc.)

// STEP 1: CREATE CLIENT — connect to the storage account
var serviceClient = new BlobServiceClient(
    new Uri("https://az204storage.blob.core.windows.net"),  // ← Storage account URL
    new DefaultAzureCredential());                           // ← Secure auth (no keys in code)

// STEP 2: GET OR CREATE A CONTAINER
BlobContainerClient containerClient = serviceClient.GetBlobContainerClient("documents");
await containerClient.CreateIfNotExistsAsync();  // ← Creates the container if it doesn't exist

// STEP 3: UPLOAD A BLOB
BlobClient blobClient = containerClient.GetBlobClient("reports/quarterly-q1.pdf");
await blobClient.UploadAsync(
    "C:/data/quarterly-q1.pdf",    // ← Local file path to upload
    overwrite: true);               // ← Overwrite if blob already exists

// STEP 4: SET METADATA (key-value pairs attached to the blob)
var metadata = new Dictionary<string, string>
{
    { "department", "finance" },
    { "quarter", "Q1-2024" },
    { "author", "Jane Smith" }
};
await blobClient.SetMetadataAsync(metadata);

// STEP 5: LIST ALL BLOBS IN THE CONTAINER
await foreach (BlobItem blob in containerClient.GetBlobsAsync())
{
    Console.WriteLine($"Name: {blob.Name}");
    Console.WriteLine($"  Tier: {blob.Properties.AccessTier}");
    Console.WriteLine($"  Size: {blob.Properties.ContentLength} bytes");
    Console.WriteLine($"  Last Modified: {blob.Properties.LastModified}");
}

// STEP 6: DOWNLOAD A BLOB
BlobDownloadInfo download = await blobClient.DownloadAsync();
using (FileStream file = File.OpenWrite("C:/downloads/quarterly-q1.pdf"))
{
    await download.Content.CopyToAsync(file);  // ← Stream blob content to local file
}

// STEP 7: DELETE A BLOB
await blobClient.DeleteIfExistsAsync();  // ← Deletes the blob (no error if it doesn't exist)
```

> 💡 **KEY CONCEPT — The Three Clients**
>
> | Client | What It Does | How You Get It |
> |---|---|---|
> | `BlobServiceClient` | Connects to the account, lists containers | `new BlobServiceClient(uri, credential)` |
> | `BlobContainerClient` | Creates/deletes containers, lists blobs | `serviceClient.GetBlobContainerClient("name")` |
> | `BlobClient` | Uploads/downloads/deletes a single blob | `containerClient.GetBlobClient("name")` |

> 🔗 **MODULE LINK**
>
> This three-client pattern (Service → Container → Item) mirrors the pattern in Cosmos DB (M05): `CosmosClient → Database → Container`. Learning it here makes M05 easier.

---

## Section 7: Blob Properties and Metadata 🟡 MEDIUM EXAM WEIGHT

Every blob has two types of associated data:

| Type | What It Is | Examples | Mutable? |
|---|---|---|---|
| **System Properties** | Set by Azure automatically. Some can be modified. | `Content-Type`, `Content-Length`, `Last-Modified`, `ETag` | Some (like Content-Type) |
| **User Metadata** | Key-value pairs YOU set. For your own organisational purposes. | `department=finance`, `author=Jane`, `status=reviewed` | ✅ Yes |

```csharp
// Set the Content-Type system property
var headers = new BlobHttpHeaders
{
    ContentType = "application/pdf"        // ← Tells browsers how to display this blob
};
await blobClient.SetHttpHeadersAsync(headers);

// Read properties
BlobProperties properties = await blobClient.GetPropertiesAsync();
Console.WriteLine($"Content Type: {properties.ContentType}");
Console.WriteLine($"Size: {properties.ContentLength}");
Console.WriteLine($"Last Modified: {properties.LastModified}");

// Read metadata
foreach (var kvp in properties.Metadata)
{
    Console.WriteLine($"  {kvp.Key} = {kvp.Value}");
}
```

> 🚨 **EXAM ALERT**
>
> **Metadata keys** are case-insensitive. **Metadata values** are strings only. Maximum 8 KB total metadata per blob.

---

## Section 8: Static Website Hosting 🟡 MEDIUM EXAM WEIGHT

Azure Blob Storage can host static websites (HTML, CSS, JS) directly — no web server needed.

### How It Works

1. Enable static website hosting on the storage account
2. Azure creates a special container called **`$web`**
3. Upload your HTML/CSS/JS files to `$web`
4. Azure serves them at `https://{account}.z{xx}.web.core.windows.net`

```bash
# Enable static website hosting
az storage blob service-properties update \
  --account-name az204storage \
  --static-website \                 # ← Enable the feature
  --index-document index.html \      # ← Default page
  --404-document 404.html            # ← Error page
```

> 📝 **NOTE**
>
> Static website hosting is for **static content only** — HTML, CSS, JavaScript, images. It cannot run server-side code (no C#, no Node.js on the server). For server-side rendering, use App Service (M01).

---

## Section 9: Redundancy — Protecting Your Data 🟡 MEDIUM EXAM WEIGHT

Covered in detail in F08, here is the exam-critical summary:

| Option | Copies | Protection Against | Use Case |
|---|---|---|---|
| **LRS** | 3 copies in 1 data centre | Hardware failure | Labs, non-critical data |
| **ZRS** | 3 copies across 3 zones in 1 region | Zone failure | Production (regional) |
| **GRS** | 6 copies (3 primary + 3 secondary region) | Region failure | Critical data, compliance |
| **GZRS** | 6 copies (3 zones primary + 3 secondary) | Zone + region failure | Maximum durability |

> 🚨 **EXAM ALERT**
>
> GRS secondary region is **NOT readable** by default. You need **RA-GRS** (Read-Access GRS) or **RA-GZRS** to read from the secondary. If the exam asks "How can an application read data from the secondary region during a primary region outage?" → the answer is enable **RA-GRS** or **RA-GZRS**.

---

## 🔗 Module Connections

| Connected Module | How It Connects |
|---|---|
| **F08 — Storage Basics** | Storage account creation, access keys, SAS tokens fundamentals |
| **M02 — Functions** | Blob trigger fires when a new blob is uploaded. Blob input/output bindings. |
| **M05 — Cosmos DB** | Same three-client SDK pattern (Service → Container → Item) |
| **M07 — Security** | Key Vault stores storage access keys. Managed Identity for SDK authentication. |
| **M09 — Events** | Event Grid fires events on blob creation/deletion for event-driven processing |

---

## What We Covered ✅

- [ ] What Blob Storage is and what problem it solves
- [ ] Three blob types: Block (files), Append (logs), Page (VM disks)
- [ ] Three-level hierarchy: Storage Account → Container → Blob
- [ ] URL pattern for blob access
- [ ] Four access tiers: Hot, Cool, Cold, Archive
- [ ] Archive tier rehydration (1–15 hours) and early deletion penalties
- [ ] Lifecycle management policies (daily execution, JSON format)
- [ ] Container public access levels: Private, Blob, Container
- [ ] RBAC roles for blob storage
- [ ] Three SAS types: Service, Account, User Delegation (most secure)
- [ ] SDK three-client hierarchy: BlobServiceClient → BlobContainerClient → BlobClient
- [ ] SDK operations: upload, download, list, delete, metadata
- [ ] System properties vs user metadata
- [ ] Static website hosting with `$web` container
- [ ] Redundancy options: LRS, ZRS, GRS, GZRS, RA-GRS

---

## Self-Check Questions

1. **A company stores financial records that must be retained for 7 years but are never accessed after the first month. Which access tier strategy minimises cost?**

2. **A developer needs to generate a time-limited URL that allows a partner company to download a specific blob for 24 hours. Which SAS type is most secure?**

3. **In the Blob Storage SDK, what is the correct order of client classes from broadest to most specific?**

<details>
<summary>Answers</summary>

1. Store in **Hot** tier for the first month, then use a **lifecycle management policy** to automatically move to **Archive** after 30 days. Keep in Archive until the 7-year retention period, then delete. This minimises storage cost while data is rarely accessed.

2. **User Delegation SAS** — signed with Microsoft Entra ID credentials, no storage account key needed. Set the expiry to 24 hours and the permission to read-only on that specific blob.

3. `BlobServiceClient` (account level) → `BlobContainerClient` (container level) → `BlobClient` (individual blob level).

</details>
