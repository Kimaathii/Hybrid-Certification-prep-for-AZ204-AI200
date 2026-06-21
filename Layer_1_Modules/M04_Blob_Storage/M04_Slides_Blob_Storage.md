# M04 — Slide Deck: Azure Blob Storage

---

## Slide 1: Title Slide

**ON SLIDE:**
- **Module 4: Azure Blob Storage**
- Exam Domain: Develop for Azure Storage (15–20%)
- Learning Objectives: Blob types, access tiers, lifecycle policies, SAS tokens, SDK operations, static websites

**SPEAKER NOTES:**
Welcome to Module 4 — Azure Blob Storage. This is the first of two storage modules (the second is Cosmos DB in M05). Blob Storage is one of the most heavily tested topics because it has many features with subtle differences — especially access tiers and SAS tokens.

---

## Slide 2: What Is Blob Storage?

**ON SLIDE:**
- **Blob** = Binary Large Object = a file
- Azure's object storage — like Google Drive for your applications
- Stores any file type: images, videos, PDFs, logs, backups, JSON exports
- Accessed via REST API or SDK from your application code

**SPEAKER NOTES:**
Blob just means file. Azure Blob Storage is where your applications store files — profile photos, uploaded documents, log files, database backups. Unlike a regular file system, it's infinitely scalable, globally accessible, and has built-in redundancy.

---

## Slide 3: Three Blob Types

**ON SLIDE:**

| Type | Purpose | Key Feature |
|---|---|---|
| **Block Blob** | Files, media (95% of use) | Parallel upload of blocks |
| **Append Blob** | Log files | Append-only, no modify |
| **Page Blob** | VM disks | Random read/write |

🚨 "Continuously written log files" → **Append Blob**

**SPEAKER NOTES:**
Block blobs are the default — use them for everything unless you have a specific reason for the other types. Append blobs are optimised for log files where you only ever add new data at the end. Page blobs are for VM disks — random access at any position. The exam loves to test this: if they mention log files being continuously written to, the answer is append blob.

---

## Slide 4: Storage Hierarchy

**ON SLIDE:**
```
Storage Account    ← az204storage
  └── Container    ← profile-photos
       └── Blob    ← user123/avatar.jpg
```

**URL:** `https://az204storage.blob.core.windows.net/profile-photos/user123/avatar.jpg`

Containers are flat — `/` in blob names simulates folders

**SPEAKER NOTES:**
Three levels: storage account at the top, containers inside (like folders), and blobs inside containers (the actual files). The URL always follows this pattern. Important: there are no real subfolders. A blob named user123/avatar.jpg is just a blob with a slash in its name.

---

## Slide 5: Access Tiers

**ON SLIDE:**

| Tier | Frequency | Min Duration | Storage Cost | Access Cost |
|---|---|---|---|---|
| **Hot** | Frequent | None | $$$ | $ |
| **Cool** | 30+ days | 30 days | $$ | $$ |
| **Cold** | 90+ days | 90 days | $ | $$$ |
| **Archive** | 180+ days | 180 days | ¢ | $$$$ + wait |

Parking analogy: Hot=valet, Cool=garage, Cold=remote lot, Archive=warehouse

**SPEAKER NOTES:**
This is one of the most exam-tested topics in the entire certification. Four tiers — as you go from Hot to Archive, storage gets cheaper but access gets more expensive. Hot is instant access, high storage cost. Archive is pennies for storage but takes hours to retrieve. The exam will present scenarios and ask you to pick the right tier.

---

## Slide 6: ⚠️ EXAM ALERT — Archive Rehydration

**ON SLIDE (RED BACKGROUND):**
- Archive blobs are **OFFLINE** — cannot be read directly
- Must **rehydrate** to Hot/Cool/Cold first
- **Standard priority:** Up to 15 hours
- **High priority:** Up to 1 hour
- Early deletion penalty applies if moved before minimum retention

**SPEAKER NOTES:**
This catches many students. Archive is not just slow — the data is completely offline. You must rehydrate it, which means copying it to a readable tier. Standard takes up to 15 hours. High priority up to 1 hour but costs more. And if you delete or move a blob before its minimum retention period, you pay an early deletion penalty.

---

## Slide 7: Lifecycle Management Policies

**ON SLIDE:**
- Automate tier transitions based on blob age
- JSON policy format with rules, filters, actions
- Example: Hot → Cool after 30 days → Archive after 180 → Delete after 365
- 🚨 Runs **once per day** (not real-time)

**SPEAKER NOTES:**
Lifecycle policies automate what would otherwise be a manual nightmare. Imagine having millions of blobs — you can't manually move them between tiers. You define rules in JSON that say "move blobs with this prefix to Cool after 30 days, to Archive after 180 days, delete after a year." But remember: these policies run once per day, not in real-time.

---

## Slide 8: SAS Tokens — The Three Types

**ON SLIDE:**

| Type | Signed With | Scope | Security |
|---|---|---|---|
| **Service SAS** | Account key | One service | 🟡 Medium |
| **Account SAS** | Account key | Multiple services | 🟡 Medium |
| **User Delegation SAS** | Entra ID | Blob/ADLS only | 🟢 Best |

🚨 **User Delegation SAS = most secure** (no storage keys involved)

**SPEAKER NOTES:**
SAS tokens are visitor passes — limited permissions for limited time. The exam focuses on knowing which type is most secure. User Delegation SAS is signed with Azure AD credentials, not storage account keys. This means if the SAS is compromised, you can revoke it via Azure AD — you can't do that with key-based SAS tokens. Always answer User Delegation when asked about the most secure option.

---

## Slide 9: SDK Client Hierarchy

**ON SLIDE:**
```
BlobServiceClient         ← Storage Account level
  └── BlobContainerClient ← Container level
       └── BlobClient     ← Individual Blob level
```

Same pattern as Cosmos DB (M05): Service → Container → Item

**SPEAKER NOTES:**
The SDK mirrors the storage hierarchy. BlobServiceClient connects to the account, BlobContainerClient works with a specific container, BlobClient works with a specific blob. Notice this is the same pattern Cosmos DB uses — learn it once, apply it twice.

---

## Slide 10: SDK Code — Upload

**ON SLIDE:**
```csharp
var serviceClient = new BlobServiceClient(connectionString);
var containerClient = serviceClient.GetBlobContainerClient("photos");
await containerClient.CreateIfNotExistsAsync();

var blobClient = containerClient.GetBlobClient("avatar.jpg");
await blobClient.UploadAsync("local-file.jpg", overwrite: true);
```

3 lines: Get client → Get container → Upload

**SPEAKER NOTES:**
Uploading a blob takes just a few lines. Create a service client with your connection string, get a container client, then get a blob client for the specific file. UploadAsync sends the local file to Azure. The overwrite parameter controls whether to replace an existing blob with the same name.

---

## Slide 11: SDK Code — Download & List

**ON SLIDE:**
```csharp
// Download
BlobDownloadInfo download = await blobClient.DownloadAsync();
using var stream = File.OpenWrite("downloaded.jpg");
await download.Content.CopyToAsync(stream);

// List all blobs
await foreach (BlobItem blob in containerClient.GetBlobsAsync())
{
    Console.WriteLine($"{blob.Name} — {blob.Properties.ContentLength} bytes");
}
```

**SPEAKER NOTES:**
Downloading streams the blob content to a local file. Listing uses async foreach to iterate through all blobs in a container. Each BlobItem gives you the name, size, tier, and last modified date. These are the operations you'll use most in labs and that the exam tests.

---

## Slide 12: Static Website Hosting

**ON SLIDE:**
- Host HTML/CSS/JS directly from Blob Storage
- No web server needed
- Special container: **`$web`**
- URL: `https://{account}.z{xx}.web.core.windows.net`
- Static content only — no server-side code

**SPEAKER NOTES:**
You can turn a storage account into a basic web host for static sites. Enable static website hosting, and Azure creates a special dollar-sign-web container. Upload your HTML and it's instantly available at a public URL. This is great for single-page apps, documentation sites, or landing pages. But remember — static only, no server-side processing.

---

## Slide 13: Container Access Levels

**ON SLIDE:**

| Level | Anonymous Access |
|---|---|
| **Private** (default) | ❌ None |
| **Blob** | ✅ Read blobs by exact URL |
| **Container** | ✅ Read + list all blobs |

🚨 Default = **Private**. Production = usually Private + SAS/RBAC.

**SPEAKER NOTES:**
By default, containers are private — you need credentials to access anything. You can enable anonymous access at the blob level (read if you know the URL) or container level (read and list everything). In production, you almost always use Private with SAS tokens or RBAC for controlled access.

---

## Slide 14: Redundancy at a Glance

**ON SLIDE:**

| Option | Copies | Protects Against |
|---|---|---|
| LRS | 3 in 1 DC | Hardware failure |
| ZRS | 3 across zones | Zone failure |
| GRS | 6 (3+3 regions) | Region failure |
| RA-GRS | GRS + read secondary | Region failure + read |

🚨 GRS secondary NOT readable without RA- prefix

**SPEAKER NOTES:**
Quick redundancy recap from F08. LRS is three copies in one data centre — cheapest, use for labs. ZRS spreads across availability zones. GRS copies to a secondary region but you can't read from it unless you enable RA-GRS — the Read-Access variant. The exam tests this: GRS secondary is NOT readable by default.

---

## Slide 15: Lab Preview

**ON SLIDE:**
1. Create storage account (GPv2, LRS)
2. Build .NET console app with SDK
3. Upload, list, download, delete blobs
4. Set and read metadata
5. Generate SAS token via CLI
6. Create lifecycle policy
7. Enable and test static website
- 💰 Total cost: under $0.01

**SPEAKER NOTES:**
In the lab, you'll build a complete .NET application that performs all the core blob operations. You'll also generate a SAS token from the CLI, create a lifecycle policy, and host a static website. Total cost is essentially zero.

---

## Slide 16: Common Exam Traps

**ON SLIDE:**
1. ❌ Archive blobs can be read instantly → **15 hours rehydration**
2. ❌ All SAS types are equally secure → **User Delegation is most secure**
3. ❌ GRS secondary is readable → **Only RA-GRS is readable**
4. ❌ Lifecycle policies run in real-time → **Once per day**
5. ❌ Containers have real subfolders → **Flat, `/` simulates folders**

**SPEAKER NOTES:**
These five traps appear regularly on the exam. Archive requires rehydration. User Delegation SAS is the most secure because it doesn't use storage keys. GRS secondary needs the RA prefix to be readable. Lifecycle policies run daily, not instantly. And blob containers are flat — the slash in names just simulates a folder structure.

---

## Slide 17: Blob Properties vs Metadata

**ON SLIDE:**

| | System Properties | User Metadata |
|---|---|---|
| Set by | Azure (some editable) | You |
| Examples | Content-Type, ETag, Size | department=finance |
| Max size | N/A | 8 KB total |
| Mutable | Some | Yes |

**SPEAKER NOTES:**
Every blob has system properties set by Azure — content type, size, last modified, ETag. You can also set custom metadata as key-value string pairs. Metadata is limited to 8 KB total per blob. Keys are case-insensitive. The exam may test metadata limits or ask you to choose between properties and metadata for a scenario.

---

## Slide 18: Checkpoint Questions

**ON SLIDE:**
1. A company has compliance records accessed once per year. Which tier minimises cost?
2. Which SAS type should be used for production applications?
3. What command generates a SAS token from the CLI?

**SPEAKER NOTES:**
Take a moment to think about these. Answers on the next slide.

---

## Slide 19: Checkpoint Answers

**ON SLIDE:**
1. **Archive** — lowest storage cost, acceptable retrieval time for annual access
2. **User Delegation SAS** — signed with Entra ID, no storage keys involved
3. `az storage container generate-sas` with `--permissions` and `--expiry` flags

**SPEAKER NOTES:**
Archive is the cheapest for data that's almost never accessed. User Delegation SAS is always the most secure answer. And the CLI command includes permissions and expiry as required parameters.

---

## Slide 20: Module Connections & Next Steps

**ON SLIDE:**
- **F08** — Storage account fundamentals (created there, deepened here)
- **M02** — Functions: Blob trigger fires on upload
- **M05** — Cosmos DB: Same SDK pattern (Service → Container → Item)
- **M07** — Security: Key Vault stores access keys, Managed Identity for SDK auth
- **M09** — Events: Event Grid fires on blob create/delete
- **Next: M05 — Azure Cosmos DB**

**SPEAKER NOTES:**
Blob Storage connects to many other modules. Functions can trigger on blob uploads. Cosmos DB uses the same three-level SDK pattern. Key Vault secures your storage keys. Event Grid can react to blob events. Next up is Cosmos DB, where you'll see a very similar SDK pattern applied to a different type of storage.
