# M04 — Lab Guide: Azure Blob Storage

---

| | |
|---|---|
| **Module** | M04 — Azure Blob Storage |
| **Lab Title** | Upload, Manage, and Secure Blobs with SDK, SAS, Lifecycle, and Static Sites |
| **Prerequisites** | F01–F08, M01–M03, .NET SDK installed, Azure CLI |
| **Estimated Time** | 75 minutes |
| **What You Will Build** | A .NET console app that uploads/downloads/lists blobs, plus a static website hosted in Blob Storage |
| **What You Will Learn** | ✅ SDK operations · ✅ Metadata · ✅ SAS tokens · ✅ Lifecycle policy · ✅ Static website hosting |

---

> 💰 **COST WARNING**
>
> - **GPv2 Storage Account (LRS):** ~$0.018/GB/month for Hot tier
> - **Data operations:** ~$0.004 per 10,000 read operations
> - **Estimated total cost for this lab: under $0.01**
>
> ⚠️ Delete all resources at the end.

---

## Part 1: Create the Storage Account

### Step 1: Create a Resource Group

```bash
az group create \
  --name az204-blob-rg \
  --location eastus
```

### Step 2: Create a Storage Account

```bash
STORAGE_NAME=az204blob$RANDOM

az storage account create \
  --name $STORAGE_NAME \             # ← Globally unique, lowercase + numbers only
  --resource-group az204-blob-rg \
  --location eastus \
  --sku Standard_LRS \               # ← Standard performance, LRS redundancy (cheapest)
  --kind StorageV2                   # ← General-purpose v2 (always use this)
```

### Step 3: Get the Connection String

```bash
CONNECTION_STRING=$(az storage account show-connection-string \
  --name $STORAGE_NAME \
  --resource-group az204-blob-rg \
  --output tsv)

echo "Connection string saved to variable"
```

> ✅ **CHECKPOINT 1**
> - ✅ Resource group created
> - ✅ Storage account created
> - ✅ Connection string saved

---

## Part 2: Create a .NET Console App with Blob SDK

### Step 4: Create the Project

```bash
mkdir az204-blob-lab && cd az204-blob-lab

dotnet new console --name BlobLab    # ← Create a new .NET console project
cd BlobLab

dotnet add package Azure.Storage.Blobs   # ← Blob Storage SDK
```

### Step 5: Write the Blob Operations Code

Replace the contents of `Program.cs` with:

```csharp
using Azure.Storage.Blobs;                 // ← Main Blob SDK namespace
using Azure.Storage.Blobs.Models;          // ← Model classes (BlobItem, BlobProperties)

// --- CONFIGURATION ---
// In production, use DefaultAzureCredential. For this lab, using connection string.
string connectionString = Environment.GetEnvironmentVariable("AZURE_STORAGE_CONNECTION")
    ?? throw new InvalidOperationException("Set AZURE_STORAGE_CONNECTION env var");

string containerName = "lab-documents";     // ← Container name

// --- STEP 1: CREATE SERVICE CLIENT ---
var serviceClient = new BlobServiceClient(connectionString);
Console.WriteLine($"Connected to: {serviceClient.AccountName}");

// --- STEP 2: CREATE CONTAINER ---
BlobContainerClient containerClient = serviceClient.GetBlobContainerClient(containerName);
await containerClient.CreateIfNotExistsAsync(PublicAccessType.None);  // ← Private access
Console.WriteLine($"Container '{containerName}' ready.");

// --- STEP 3: UPLOAD BLOBS ---
// Create sample files to upload
File.WriteAllText("report-q1.txt", "Q1 Financial Report\nRevenue: $1.2M\nProfit: $300K");
File.WriteAllText("report-q2.txt", "Q2 Financial Report\nRevenue: $1.5M\nProfit: $400K");
File.WriteAllText("notes.txt", "Meeting notes from project kickoff.");

string[] files = { "report-q1.txt", "report-q2.txt", "notes.txt" };

foreach (string file in files)
{
    BlobClient blobClient = containerClient.GetBlobClient($"reports/{file}");
    await blobClient.UploadAsync(file, overwrite: true);   // ← Upload with overwrite
    Console.WriteLine($"  Uploaded: reports/{file}");
}

// --- STEP 4: SET METADATA ---
BlobClient q1Blob = containerClient.GetBlobClient("reports/report-q1.txt");
var metadata = new Dictionary<string, string>
{
    { "department", "finance" },
    { "quarter", "Q1" },
    { "status", "reviewed" }
};
await q1Blob.SetMetadataAsync(metadata);
Console.WriteLine("\nMetadata set on report-q1.txt");

// --- STEP 5: LIST ALL BLOBS ---
Console.WriteLine($"\nBlobs in '{containerName}':");
await foreach (BlobItem blob in containerClient.GetBlobsAsync())
{
    Console.WriteLine($"  Name: {blob.Name}");
    Console.WriteLine($"    Size: {blob.Properties.ContentLength} bytes");
    Console.WriteLine($"    Tier: {blob.Properties.AccessTier}");
    Console.WriteLine($"    Modified: {blob.Properties.LastModified}");
}

// --- STEP 6: READ METADATA ---
BlobProperties properties = await q1Blob.GetPropertiesAsync();
Console.WriteLine($"\nMetadata for report-q1.txt:");
foreach (var kvp in properties.Metadata)
{
    Console.WriteLine($"  {kvp.Key} = {kvp.Value}");
}

// --- STEP 7: DOWNLOAD A BLOB ---
BlobDownloadInfo download = await q1Blob.DownloadAsync();
using (StreamReader reader = new StreamReader(download.Content))
{
    string content = await reader.ReadToEndAsync();
    Console.WriteLine($"\nDownloaded content:\n{content}");
}

// --- STEP 8: DELETE A BLOB ---
BlobClient notesBlob = containerClient.GetBlobClient("reports/notes.txt");
await notesBlob.DeleteIfExistsAsync();
Console.WriteLine("\nDeleted: reports/notes.txt");

// Final blob count
int count = 0;
await foreach (BlobItem blob in containerClient.GetBlobsAsync()) { count++; }
Console.WriteLine($"\nRemaining blobs: {count}");
Console.WriteLine("\n✅ All blob operations complete!");
```

### Step 6: Run the Application

```bash
# Set the connection string as an environment variable
# Linux/Mac:
export AZURE_STORAGE_CONNECTION="$CONNECTION_STRING"

# Windows PowerShell:
$env:AZURE_STORAGE_CONNECTION = $CONNECTION_STRING

# Run the app
dotnet run
```

**Expected output:**
```
Connected to: az204blobXXXXX
Container 'lab-documents' ready.
  Uploaded: reports/report-q1.txt
  Uploaded: reports/report-q2.txt
  Uploaded: reports/notes.txt

Metadata set on report-q1.txt

Blobs in 'lab-documents':
  Name: reports/notes.txt
    Size: 44 bytes
    Tier: Hot
  Name: reports/report-q1.txt
    Size: 54 bytes
    Tier: Hot
  Name: reports/report-q2.txt
    Size: 54 bytes
    Tier: Hot

Metadata for report-q1.txt:
  department = finance
  quarter = Q1
  status = reviewed

Downloaded content:
Q1 Financial Report
Revenue: $1.2M
Profit: $300K

Deleted: reports/notes.txt

Remaining blobs: 2

✅ All blob operations complete!
```

> ✅ **CHECKPOINT 2**
> - ✅ .NET project created with Azure.Storage.Blobs package
> - ✅ Container created programmatically
> - ✅ 3 blobs uploaded
> - ✅ Metadata set and read
> - ✅ Blob listed, downloaded, and deleted

---

## Part 3: Generate a SAS Token via CLI

### Step 7: Generate a SAS Token

```bash
# Generate a SAS token for the container (valid for 1 hour, read-only)
END_DATE=$(date -u -d "+1 hour" '+%Y-%m-%dT%H:%MZ')  # ← 1 hour from now

SAS_TOKEN=$(az storage container generate-sas \
  --account-name $STORAGE_NAME \
  --name lab-documents \             # ← Container name
  --permissions r \                  # ← Read-only permission
  --expiry $END_DATE \               # ← Token expires in 1 hour
  --auth-mode key \                  # ← Sign with storage account key
  --output tsv)

echo "SAS Token: $SAS_TOKEN"
```

### Step 8: Access a Blob Using the SAS URL

```bash
# Construct the full SAS URL
BLOB_URL="https://$STORAGE_NAME.blob.core.windows.net/lab-documents/reports/report-q1.txt?$SAS_TOKEN"

# Access the blob using curl
curl "$BLOB_URL"
```

**Expected output:** The contents of `report-q1.txt` displayed in your terminal.

> ✅ **CHECKPOINT 3**
> - ✅ SAS token generated with read-only permissions and 1-hour expiry
> - ✅ Blob accessed via SAS URL without authentication credentials

---

## Part 4: Create a Lifecycle Management Policy

### Step 9: Create a Lifecycle Policy

```bash
# Create a lifecycle policy JSON file
cat > lifecycle-policy.json << 'EOF'
{
  "rules": [
    {
      "name": "move-old-reports-to-cool",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["reports/"]
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
EOF

# Apply the policy
az storage account management-policy create \
  --account-name $STORAGE_NAME \
  --resource-group az204-blob-rg \
  --policy @lifecycle-policy.json    # ← @ prefix reads from file
```

> 📝 **NOTE:** Lifecycle policies run **once per day**. You won't see immediate tier changes in this lab — the policy is configured for future automated management.

> ✅ **CHECKPOINT 4**
> - ✅ Lifecycle policy JSON created
> - ✅ Policy applied to storage account

---

## Part 5: Static Website Hosting

### Step 10: Enable Static Website

```bash
az storage blob service-properties update \
  --account-name $STORAGE_NAME \
  --static-website \                 # ← Enable static website hosting
  --index-document index.html \      # ← Default page
  --404-document 404.html            # ← Error page
```

### Step 11: Upload HTML Files

```bash
# Create a simple index page
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>AZ-204 Blob Storage Lab</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 50px; background: #0078d4; color: white; }
        h1 { font-size: 2.5em; }
        p { font-size: 1.2em; }
    </style>
</head>
<body>
    <h1>🎉 Hello from Azure Blob Storage!</h1>
    <p>This static website is hosted directly from Azure Blob Storage.</p>
    <p>No web server required — just HTML, CSS, and JavaScript.</p>
</body>
</html>
EOF

cat > 404.html << 'EOF'
<!DOCTYPE html>
<html><head><title>404</title></head>
<body><h1>404 — Page Not Found</h1><p>This page doesn't exist.</p></body>
</html>
EOF

# Upload to the $web container
az storage blob upload \
  --account-name $STORAGE_NAME \
  --container-name '$web' \          # ← Special static website container
  --name index.html \
  --file index.html \
  --auth-mode key

az storage blob upload \
  --account-name $STORAGE_NAME \
  --container-name '$web' \
  --name 404.html \
  --file 404.html \
  --auth-mode key
```

### Step 12: Access the Static Website

```bash
# Get the website URL
az storage account show \
  --name $STORAGE_NAME \
  --resource-group az204-blob-rg \
  --query "primaryEndpoints.web" \
  --output tsv
```

Open the URL in your browser. You should see your "Hello from Azure Blob Storage!" page.

🎉 **You just hosted a website with zero web servers!**

> ✅ **CHECKPOINT 5**
> - ✅ Static website hosting enabled
> - ✅ HTML files uploaded to `$web` container
> - ✅ Website accessible in browser

---

## Part 6: Clean Up 🧹

> 💰 **COST WARNING:** Storage accounts accrue costs even when idle (for stored data). Delete everything now.

```bash
az group delete \
  --name az204-blob-rg \
  --yes --no-wait
```

Verify after 2 minutes: `az group list --output table`

---

## What We Covered ✅

- [ ] Created a GPv2 storage account with LRS redundancy
- [ ] Built a .NET console app with `Azure.Storage.Blobs` SDK
- [ ] Created a blob container programmatically
- [ ] Uploaded blobs using `BlobClient.UploadAsync()`
- [ ] Set and read user metadata on blobs
- [ ] Listed all blobs in a container with properties
- [ ] Downloaded blob content with `BlobClient.DownloadAsync()`
- [ ] Deleted a blob with `BlobClient.DeleteIfExistsAsync()`
- [ ] Generated a SAS token via CLI with read-only permissions
- [ ] Accessed a blob via SAS URL
- [ ] Created a lifecycle management policy (JSON format)
- [ ] Enabled static website hosting
- [ ] Uploaded HTML to the `$web` container
- [ ] Accessed the static website in a browser
- [ ] Cleaned up all resources
