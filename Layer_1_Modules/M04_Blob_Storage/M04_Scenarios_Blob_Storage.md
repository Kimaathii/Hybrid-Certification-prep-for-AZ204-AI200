# M04 — Scenario Cards: Azure Blob Storage

---

## SCENARIO 1 — Beginner

**TITLE:** The Profile Photo Storage

### BUSINESS CONTEXT:
A social media startup needs to store user profile photos. Users upload photos through the web app, and these photos are displayed on their profile pages. The app has 10,000 users and growing.

### THE PROBLEM:
Currently, photos are stored in the web app's local file system. When the team added a second web server for load balancing, half the users couldn't see their photos — each server only had the photos uploaded to it.

### CONSTRAINTS:
- Photos must be accessible from all web app instances
- Photos are viewed frequently (multiple times per day per user)
- Storage cost should be reasonable
- Users need to see photos immediately after upload

### YOUR TASK:
Design a Blob Storage solution. Which access tier? Which container access level?

### SOLUTION:
Use **Azure Blob Storage** with **Hot tier** and **Private container access** + application-managed SAS tokens.

- **Hot tier** because photos are viewed frequently — low access cost is more important than low storage cost
- **Private access** because photos should only be viewable by authenticated users (not anonymously)
- The web application generates short-lived **SAS URLs** for each photo when rendering profile pages
- Blob name pattern: `profiles/{userId}/avatar.jpg`

All web app instances access the same storage account — solving the multi-instance problem.

### EXAM CONNECTION:
This tests understanding of access tiers (Hot for frequently accessed data) and access control (Private + SAS for controlled access). If the exam says "frequently accessed" → Hot tier.

---

## SCENARIO 2 — Intermediate

**TITLE:** The Compliance Data Archive

### BUSINESS CONTEXT:
A healthcare company must retain patient records for 7 years by law. After the first year, records are never accessed unless there is a legal audit (happens roughly once per decade). The company stores 50 TB of records.

### THE PROBLEM:
Storing 50 TB in the Hot tier costs ~$900/month. The company needs to minimise storage costs while maintaining the 7-year retention requirement.

### CONSTRAINTS:
- Records must be retained for 7 years
- After the first year, records are almost never accessed
- When accessed for an audit, a 15-hour retrieval delay is acceptable
- Must be automated — manual tier management for millions of files is impossible

### YOUR TASK:
Design a lifecycle management strategy with specific tier transitions and timing.

### SOLUTION:
Create a **lifecycle management policy** with the following transitions:

1. **Days 0–30:** Hot tier (records may be actively referenced)
2. **Day 31–365:** Cool tier (occasional access within the first year)
3. **Day 366+:** Archive tier (almost never accessed, cheapest storage)
4. **Day 2,556 (7 years):** Auto-delete

```json
{
  "rules": [{
    "name": "healthcare-retention",
    "type": "Lifecycle",
    "definition": {
      "filters": { "blobTypes": ["blockBlob"], "prefixMatch": ["patient-records/"] },
      "actions": {
        "baseBlob": {
          "tierToCool": { "daysAfterModificationGreaterThan": 30 },
          "tierToArchive": { "daysAfterModificationGreaterThan": 365 },
          "delete": { "daysAfterModificationGreaterThan": 2556 }
        }
      }
    }
  }]
}
```

Cost savings: Archive tier for 50 TB ≈ **$50/month** (vs $900 in Hot) — 94% reduction.

### EXAM CONNECTION:
Tests lifecycle policies, tier cost optimisation, and Archive rehydration acceptance. The exam may describe "data retained for X years, rarely accessed" — always consider Archive + lifecycle policy.

---

## SCENARIO 3 — Intermediate

**TITLE:** The Secure Document Sharing

### BUSINESS CONTEXT:
A law firm needs to share confidential documents with external clients. Each document should be accessible for exactly 48 hours via a private link, after which access should expire automatically.

### THE PROBLEM:
The firm cannot give clients access to their Azure storage account. They need time-limited, read-only links to specific documents that automatically expire.

### CONSTRAINTS:
- External users have no Azure accounts
- Links must expire after 48 hours
- Read-only access to specific blobs only
- Must be the most secure method possible
- Cannot use storage account keys in the sharing process

### YOUR TASK:
Which SAS type should the firm use? Design the access solution.

### SOLUTION:
Use **User Delegation SAS** tokens with 48-hour expiry and read-only permissions.

Why User Delegation SAS (not Service or Account SAS):
1. **Signed with Entra ID credentials** — no storage account key exposed
2. **Revocable** — can revoke via Azure AD if compromised
3. **Most secure** — Microsoft's recommended approach

Implementation:
1. Lawyer uploads document to Private container
2. Application generates User Delegation SAS:
   - Permission: `r` (read only)
   - Expiry: 48 hours from now
   - Scope: specific blob path only
3. SAS URL sent to client via email
4. Client clicks URL → downloads document
5. After 48 hours → URL stops working automatically

### EXAM CONNECTION:
The exam asks "most secure SAS type" — always answer **User Delegation SAS**. Key differentiator: no storage account key involved, signed with Entra ID.

---

## SCENARIO 4 — Advanced

**TITLE:** The Event-Driven Image Pipeline

### BUSINESS CONTEXT:
An e-commerce platform allows sellers to upload product images. When an image is uploaded, the system needs to: (1) generate a thumbnail, (2) scan for inappropriate content, (3) update the product catalog database.

### THE PROBLEM:
Currently, the upload API does all three tasks synchronously, making uploads take 8 seconds. The team wants to make uploads instant and process images in the background.

### CONSTRAINTS:
- Upload must complete in under 1 second
- Thumbnail, content scan, and database update can happen asynchronously
- Must automatically trigger when a new blob appears
- Each processing step is independent

### YOUR TASK:
Design an event-driven architecture combining Blob Storage with Azure Functions.

### SOLUTION:
Use **Blob Storage + Event Grid + Azure Functions** with **fan-out/fan-in** processing:

1. **Seller uploads image** to `product-images` container → returns immediately
2. **Event Grid** fires a `BlobCreated` event (M09)
3. **Three Azure Functions** trigger independently:
   - **Thumbnail Generator:** Reads original, creates thumbnail, saves to `thumbnails` container
   - **Content Scanner:** Calls Azure AI Content Safety API, flags inappropriate content
   - **Catalog Updater:** Adds image URL to product database

Architecture:
```
Upload → Blob Storage → Event Grid → ┌─ Function: Thumbnail
                                      ├─ Function: Content Scan  
                                      └─ Function: Catalog Update
```

Why Event Grid (not Blob Trigger):
- Event Grid delivers events in seconds; Blob Trigger can take up to 10 minutes for non-General Purpose v2 accounts
- Event Grid supports filtering (only trigger on .jpg/.png files)

### EXAM CONNECTION:
Tests integration between Blob Storage and Functions/Event Grid. The exam may contrast Blob Trigger vs Event Grid for blob events — Event Grid is faster and more reliable.

---

## SCENARIO 5 — Advanced

**TITLE:** The Multi-Region Disaster Recovery

### BUSINESS CONTEXT:
A financial services company stores critical trading data in Azure Blob Storage. Regulations require that the data must be accessible even if an entire Azure region goes down. The company needs to read data from a secondary region during a primary region outage.

### THE PROBLEM:
The company currently uses GRS (Geo-Redundant Storage) but discovered during a disaster recovery drill that they could NOT read data from the secondary region.

### CONSTRAINTS:
- Must read data during primary region outage
- Cannot lose more than 15 minutes of data (RPO)
- Must maintain current write performance
- Budget allows for higher storage costs

### YOUR TASK:
What redundancy option should the company use? Why didn't GRS work for reads?

### SOLUTION:
Switch from **GRS** to **RA-GRS** (Read-Access Geo-Redundant Storage).

Why GRS failed:
- GRS replicates data to a secondary region, but the secondary is **NOT readable by default**
- The secondary only becomes accessible during a Microsoft-initiated failover (which the customer cannot trigger on demand)
- GRS = disaster recovery storage, NOT disaster recovery access

Why RA-GRS works:
- **RA-GRS** = GRS + **read access to the secondary at all times**
- Secondary endpoint: `https://{account}-secondary.blob.core.windows.net`
- Application can read from secondary immediately during primary outage
- RPO: ~15 minutes (async replication lag)

Application code must handle two endpoints:
```csharp
string primaryUri = "https://trading.blob.core.windows.net";
string secondaryUri = "https://trading-secondary.blob.core.windows.net";

// Use primary for reads/writes normally
// Fall back to secondary for reads during primary outage
```

### EXAM CONNECTION:
This is a **classic exam trap**: GRS ≠ read access to secondary. You need RA-GRS (or RA-GZRS) for read access. The exam question will describe needing to "read during outage" — always look for the RA- prefix.
