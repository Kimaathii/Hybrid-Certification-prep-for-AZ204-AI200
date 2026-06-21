# M04 — Module Quiz: Azure Blob Storage

---

| | |
|---|---|
| **Module** | M04 — Azure Blob Storage |
| **Questions** | 15 (5 Foundation + 5 Applied + 5 Exam-Level) |
| **Time Limit** | 25 minutes |
| **Passing Score** | 11/15 (73%) |

---

## FOUNDATION QUESTIONS (1–5)

### Question 1

**What does "Blob" stand for in Azure Blob Storage?**

- A) Block Level Object Binary
- B) Binary Large Object ✅
- C) Basic Linked Object Bundle
- D) Binary Logical Object Base

**Correct Answer: B**

**Explanation:** Blob stands for **Binary Large Object** — a term for any binary file. In Azure, a blob is simply a file stored in the cloud: images, videos, documents, logs, backups, or any other binary data.

---

### Question 2

**What is the correct three-level hierarchy of Azure Blob Storage?**

- A) Subscription → Storage Account → Blob
- B) Resource Group → Container → Blob
- C) Storage Account → Container → Blob ✅
- D) Storage Account → Folder → File

**Correct Answer: C**

**Explanation:** The hierarchy is: **Storage Account** (top level, globally unique name) → **Container** (groups related blobs, like a folder) → **Blob** (the actual file). Containers are flat — there are no real subfolders, though `/` in blob names simulates folder structures.

---

### Question 3

**Which blob type is optimised for log files that are continuously written to?**

- A) Block Blob
- B) Append Blob ✅
- C) Page Blob
- D) Archive Blob

**Correct Answer: B**

**Explanation:** **Append Blobs** are optimised for append operations — new data is added only at the end, and existing content cannot be modified. This makes them ideal for log files, audit trails, and telemetry data where you write sequentially.

- **A is wrong:** Block Blobs support upload/download of complete files — they allow modification but aren't optimised for sequential append.
- **C is wrong:** Page Blobs are for random read/write access (VM disks).
- **D is wrong:** Archive is an access tier, not a blob type.

---

### Question 4

**What is the default public access level for a new blob container?**

- A) Blob (read access by exact URL)
- B) Container (read and list access)
- C) Private (no anonymous access) ✅
- D) Public (full anonymous access)

**Correct Answer: C**

**Explanation:** New containers default to **Private** — all requests require authentication. Anonymous public access must be explicitly enabled by changing the access level to Blob or Container.

---

### Question 5

**Which SDK client class do you use to upload or download a specific blob?**

- A) BlobServiceClient
- B) BlobContainerClient
- C) BlobClient ✅
- D) StorageClient

**Correct Answer: C**

**Explanation:** The SDK hierarchy maps to the storage hierarchy: `BlobServiceClient` (account level) → `BlobContainerClient` (container level) → **`BlobClient`** (individual blob level). `BlobClient` has `UploadAsync()`, `DownloadAsync()`, `DeleteAsync()`, and metadata methods.

---

## APPLIED QUESTIONS (6–10)

### Question 6

**A company stores transaction records that are accessed frequently for the first month, occasionally for the next 5 months, and almost never after that. They must be retained for 2 years. Which lifecycle management strategy minimises cost?**

- A) Keep in Hot tier for 2 years
- B) Hot → Cool after 30 days → Archive after 180 days → Delete after 730 days ✅
- C) Archive from day 1 → Delete after 730 days
- D) Cool from day 1 → Delete after 730 days

**Correct Answer: B**

**Explanation:** This follows the access pattern: Hot for frequent first-month access, Cool for occasional 2–6 month access, Archive for long-term retention at minimal cost, then auto-delete at 2 years (730 days).

- **A is wrong:** Keeping everything in Hot wastes money — Hot storage costs are 10x higher than Archive.
- **C is wrong:** Archive from day 1 means no instant access during the first month when records are frequently needed.
- **D is wrong:** Cool is better than Hot for months 2–6, but records are accessed frequently in month 1 (needs Hot).

---

### Question 7

**A law firm needs to share a confidential document with a client for exactly 24 hours using the most secure method. The client has no Azure account. Which approach should they use?**

- A) Make the container public temporarily
- B) Generate a Service SAS with 24-hour expiry
- C) Generate a User Delegation SAS with 24-hour expiry and read-only permission ✅
- D) Share the storage account access key

**Correct Answer: C**

**Explanation:** **User Delegation SAS** is the most secure because it's signed with Microsoft Entra ID credentials (no storage account keys involved). With read-only permission and a 24-hour expiry, the client can download the document but the link stops working after 24 hours.

- **A is wrong:** Making the container public exposes ALL blobs, not just one document.
- **B is wrong:** Service SAS works but is less secure — it uses the storage account key for signing.
- **D is wrong:** Sharing the access key gives the client full access to everything in the storage account — a massive security risk.

---

### Question 8

**A developer runs the following code and gets an error: "Blob not found." What is the most likely cause?**

```csharp
BlobClient blob = containerClient.GetBlobClient("Reports/quarterly.pdf");
```

- A) Container names are case-sensitive
- B) Blob names are case-sensitive — the actual blob might be named "reports/quarterly.pdf" ✅
- C) The forward slash is not allowed in blob names
- D) GetBlobClient requires the full URL, not just the name

**Correct Answer: B**

**Explanation:** Blob names are **case-sensitive**. `Reports/quarterly.pdf` is a different blob than `reports/quarterly.pdf`. The developer used an uppercase `R` but the actual blob may have been created with a lowercase `r`.

---

### Question 9

**How often do lifecycle management policies evaluate and execute rules?**

- A) Every hour
- B) In real-time as blobs are modified
- C) Once per day ✅
- D) Once per week

**Correct Answer: C**

**Explanation:** Lifecycle management policies run **once per day**. They are NOT real-time. After a blob passes a rule's age threshold, it may take up to 24 hours for the policy to execute the tier transition or deletion.

---

### Question 10

**Which of the following is a valid use case for static website hosting in Azure Blob Storage?**

- A) Hosting a C# MVC web application
- B) Hosting a single-page React application ✅
- C) Hosting a Node.js Express API
- D) Hosting a .NET Web API with database access

**Correct Answer: B**

**Explanation:** Blob static website hosting supports **static content only** — HTML, CSS, JavaScript, images. A React SPA (single-page application) is pure client-side JavaScript that runs in the browser. Server-side applications (C# MVC, Node.js Express, .NET Web API) require a compute service like App Service (M01).

---

## EXAM-LEVEL QUESTIONS (11–15)

### Question 11

**A company uses GRS (Geo-Redundant Storage) for critical data. During a disaster recovery drill, they attempt to read data from the secondary region but receive an error. What must they change to enable secondary region reads?**

- A) Enable ZRS instead of GRS
- B) Upgrade to RA-GRS (Read-Access Geo-Redundant Storage) ✅
- C) Configure a private endpoint in the secondary region
- D) Enable public access on the containers

**Correct Answer: B**

**Explanation:** GRS replicates data to a secondary region, but the secondary is **NOT readable** unless you enable **RA-GRS** (Read-Access GRS). RA-GRS provides a secondary read endpoint: `https://{account}-secondary.blob.core.windows.net`.

- **A is wrong:** ZRS replicates across zones within ONE region — it does not have a secondary region.
- **C is wrong:** Private endpoints control network path, not read access to a secondary region.
- **D is wrong:** Public access controls anonymous access, not secondary region availability.

---

### Question 12 — SELECT TWO

**Which TWO of the following statements about the Archive access tier are correct? (SELECT TWO)**

- A) Archive blobs can be read directly without any additional steps
- B) Archive blobs must be rehydrated before they can be read ✅
- C) Archive has the lowest storage cost of all tiers ✅
- D) Standard rehydration completes in 1 hour
- E) Archive has no minimum retention period

**Correct Answers: B, C**

**Explanation:**
- **B is correct:** Archive blobs are offline and must be rehydrated (moved to Hot, Cool, or Cold) before reading.
- **C is correct:** Archive has the lowest per-GB storage cost of all four tiers.
- **A is wrong:** Archive blobs are offline — they cannot be read directly.
- **D is wrong:** Standard priority rehydration takes up to **15 hours**, not 1 hour. High priority takes up to 1 hour.
- **E is wrong:** Archive has a **180-day** minimum retention period. Early deletion incurs a penalty.

---

### Question 13

**A developer creates a lifecycle management policy to move blobs from Hot to Cool after 30 days. A blob is modified on January 1st. When is the earliest the blob could be moved to Cool?**

- A) January 31st at midnight
- B) January 31st or February 1st (within 24 hours of day 30) ✅
- C) Immediately after 30 days (real-time)
- D) February 15th (next monthly run)

**Correct Answer: B**

**Explanation:** Lifecycle policies run **once per day**. After the 30-day threshold is reached on January 31st, the policy will process the transition during its next daily run — which could be on January 31st or February 1st, depending on when the daily cycle executes. It is NOT real-time.

---

### Question 14

**A developer writes the following code to set metadata on a blob:**

```csharp
var metadata = new Dictionary<string, string>
{
    { "Author", "Jane Smith" },
    { "AUTHOR", "John Doe" }
};
await blobClient.SetMetadataAsync(metadata);
```

**What happens when this code runs?**

- A) Both entries are set — "Author" = "Jane Smith" and "AUTHOR" = "John Doe"
- B) An error occurs because metadata keys are case-insensitive and "Author" conflicts with "AUTHOR" ✅
- C) Only the last entry is set — "AUTHOR" = "John Doe"
- D) Both entries are set with different casing treated as different keys

**Correct Answer: B**

**Explanation:** Blob metadata keys are **case-insensitive**. "Author" and "AUTHOR" are considered the same key, which creates a duplicate key conflict. The `Dictionary<string, string>` in C# is case-sensitive by default, so it allows both entries, but Azure's API rejects duplicate keys when they are case-insensitively equal.

---

### Question 15 — SELECT TWO

**A company needs to migrate 10 TB of data from an on-premises file server to Azure Blob Storage. They also need the transferred data to be in the Cool tier. Which TWO tools/approaches should they use? (SELECT TWO)**

- A) Use AzCopy to upload files in bulk ✅
- B) Use Azure Storage Explorer for drag-and-drop upload
- C) Use the Blob Storage SDK in a custom application
- D) Create a lifecycle management policy to move to Cool after upload ✅
- E) Set the storage account default tier to Cool before upload

**Correct Answers: A, D**

**Explanation:**
- **A is correct:** **AzCopy** is the recommended tool for bulk data migration. It supports parallel transfers, resume on failure, and server-side copy.
- **D is correct:** Upload to Hot first (fastest), then use a lifecycle policy to move to Cool. Alternatively, you could set the blob tier during upload, but a lifecycle policy automates this for millions of files.
- **B is wrong:** Storage Explorer works but is impractical for 10 TB — it's designed for interactive browsing, not bulk migration.
- **C is wrong:** Writing a custom application is unnecessary when AzCopy handles bulk uploads natively.
- **E is wrong:** Setting the default tier to Cool works but option E says "before upload" — AzCopy can set the tier per-blob during upload, and lifecycle policy is more flexible for ongoing management.

---

## Score Interpretation

| Score | Assessment | Recommendation |
|---|---|---|
| 13–15 | 🟢 Excellent — Exam ready for Blob Storage | Move to M05 |
| 11–12 | 🟡 Good — Review the questions you missed | Re-read specific sections |
| 8–10 | 🟠 Needs work — Review the Concept document | Re-read M04 Concept, redo scenarios |
| Below 8 | 🔴 Not ready — Start from the beginning | Re-read M04 Concept thoroughly |
