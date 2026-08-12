# M04 — Lab Guide: Azure Blob Storage (Enterprise Edition)

---

| | |
|---|---|
| **Module** | M04 — Azure Blob Storage |
| **Lab Title** | Upload, Secure, and Govern Blobs with User Delegation SAS, RBAC, and Immutability |
| **Prerequisites** | F01–F08, active Azure subscription, Azure Cloud Shell (Bash) |
| **Estimated Time** | 90 minutes |
| **What You Will Build** | A governed blob storage environment with RBAC access, enterprise-grade User Delegation SAS tokens, lifecycle policies, and a compliance immutability lock |
| **What You Will Learn** | ✅ Create Storage Account · ✅ Assign RBAC roles · ✅ Generate User Delegation SAS · ✅ Apply lifecycle policies · ✅ Apply Legal Hold · ✅ Host static website |
| **Tool** | Azure Cloud Shell (Bash) |

---

> ⚠️ **IMPORTANT: Use Azure Cloud Shell for the entire lab.**
> Open [portal.azure.com](https://portal.azure.com) → Click the `>_` icon → Select **Bash**.

---

> 💰 **COST WARNING**
>
> - **GPv2 Storage Account (LRS):** ~$0.018/GB/month for Hot tier
> - **Data operations:** ~$0.004 per 10,000 read operations
> - **Estimated total cost for this lab: under $0.05**
>
> ⚠️ Delete all resources at the end.

---

## Part 1: Create the Storage Infrastructure

### Step 1: Create a Resource Group

```bash
az group create \
  --name az204-blob-rg \
  --location eastus \
  --tags Project=AZ204 Module=M04
```

> 👀 **UI Check:** Portal → **Resource Groups** → `az204-blob-rg` appears with your tags.

### Step 2: Create a Storage Account

```bash
STORAGE_NAME=az204blob$RANDOM
echo "Storage account name: $STORAGE_NAME"   # ← WRITE THIS DOWN

az storage account create \
  --name $STORAGE_NAME \                      # ← Globally unique, 3–24 lowercase chars
  --resource-group az204-blob-rg \
  --location eastus \
  --sku Standard_LRS \                        # ← Locally Redundant Storage (cheapest for labs)
  --kind StorageV2 \                          # ← General Purpose v2 (always use this)
  --allow-blob-public-access false \          # ← Enterprise best practice: disable anonymous access
  --min-tls-version TLS1_2                    # ← Enforce TLS 1.2 minimum (same as App Service)
```

| Flag | Why It Matters |
|---|---|
| `--allow-blob-public-access false` | Prevents anonymous users from ever accessing blobs, even if someone sets a container to "Blob" or "Container" access. Enterprise standard. |
| `--min-tls-version TLS1_2` | Rejects any connection using older, insecure TLS versions. Security hardening. |

> 👀 **UI Check:** Portal → **Storage accounts** → click `$STORAGE_NAME` → **Security + networking** → **Configuration** → verify "Allow Blob public access" is **Disabled** and "Minimum TLS version" is **TLS 1.2**.

### Step 3: Create Blob Containers

```bash
# documents container — general file uploads
az storage container create \
  --account-name $STORAGE_NAME \
  --name documents \
  --auth-mode login                           # ← Use your Entra ID identity (not the account key)

# compliance-records container — will receive an immutability policy later
az storage container create \
  --account-name $STORAGE_NAME \
  --name compliance-records \
  --auth-mode login
```

> 📝 **NOTE:** We use `--auth-mode login` throughout this lab instead of `--auth-mode key`. This instructs the CLI to use your logged-in Entra ID identity rather than the storage account key. This is the enterprise standard — it means you never need to extract or handle the account key directly.

> 👀 **UI Check:** Portal → your Storage account → **Data storage** → **Containers** → both containers appear.

> ✅ **CHECKPOINT 1**
>
> - ✅ Resource group created with tags
> - ✅ Storage account created with public access disabled and TLS 1.2 enforced
> - ✅ Two containers created using Entra ID authentication

---

## Part 2: Upload Real Data

### Step 4: Create Test Files and Upload Them

```bash
# Create realistic test files (simulating a document management system)
cat > invoice-2024-001.txt << 'EOF'
INVOICE #2024-001
Date: 2024-01-15
Client: AlphaAI Ltd
Amount: $15,000.00
Status: PAID
EOF

cat > contract-2024-001.txt << 'EOF'
SERVICE AGREEMENT
Between: AlphaAI Ltd and CloudProvider Inc
Start Date: 2024-01-01
Duration: 12 months
Value: $180,000/year
Status: ACTIVE
EOF

cat > report-q1-2024.txt << 'EOF'
Q1 2024 FINANCIAL REPORT
Revenue: $1.2M
Operating Costs: $800K
Net Profit: $400K
Status: AUDITED AND APPROVED
EOF

# Upload files to the documents container using Entra ID auth
az storage blob upload \
  --account-name $STORAGE_NAME \
  --container-name documents \
  --name "invoices/invoice-2024-001.txt" \
  --file invoice-2024-001.txt \
  --auth-mode login

az storage blob upload \
  --account-name $STORAGE_NAME \
  --container-name compliance-records \
  --name "contracts/contract-2024-001.txt" \
  --file contract-2024-001.txt \
  --auth-mode login

az storage blob upload \
  --account-name $STORAGE_NAME \
  --container-name compliance-records \
  --name "reports/report-q1-2024.txt" \
  --file report-q1-2024.txt \
  --auth-mode login

echo "Files uploaded successfully"
```

### Step 5: Verify Uploads

```bash
# List blobs in the documents container
az storage blob list \
  --account-name $STORAGE_NAME \
  --container-name documents \
  --output table \
  --auth-mode login

# List blobs in the compliance-records container
az storage blob list \
  --account-name $STORAGE_NAME \
  --container-name compliance-records \
  --output table \
  --auth-mode login
```

> 👀 **UI Check:** Portal → Storage account → **Containers** → click `documents` → you'll see the `invoices/` virtual folder. Click into it to see the blob.

> ✅ **CHECKPOINT 2**
>
> - ✅ Three test files created
> - ✅ Files uploaded to correct containers using `--auth-mode login`
> - ✅ Upload verified via CLI and portal

---

## Part 3: RBAC — Grant Secure Access Without Keys

### Step 6: Understand Why We Use RBAC Instead of Keys

> 💡 **KEY CONCEPT: The Problem with Storage Account Keys**
>
> Every storage account has two **Account Keys**. These keys are like a master key to the entire building — anyone with a key has full read/write/delete access to EVERY container and EVERY blob, forever, with no audit trail of who did what.
>
> In enterprise environments, you never hand out storage account keys. Instead, you use **RBAC roles** scoped to the minimum level necessary.

### Step 7: Assign Your Own Identity the Storage Blob Data Contributor Role

In this step, you assign your own logged-in identity a role on the storage account. This is required to generate a User Delegation SAS in the next part.

```bash
# Get your own account's identity (object ID)
USER_ID=$(az ad signed-in-user show --query id --output tsv)
echo "Your user Object ID: $USER_ID"

# Get the storage account resource ID
STORAGE_ID=$(az storage account show \
  --name $STORAGE_NAME \
  --resource-group az204-blob-rg \
  --query id \
  --output tsv)

# Assign yourself the Storage Blob Data Contributor role
az role assignment create \
  --assignee $USER_ID \
  --role "Storage Blob Data Contributor" \
  --scope $STORAGE_ID
```

| Flag | What It Does |
|---|---|
| `--assignee` | The identity receiving the role (your own account here) |
| `--role "Storage Blob Data Contributor"` | Grants read, write, and delete on blobs. NOT admin rights. |
| `--scope $STORAGE_ID` | Scopes the role to this specific storage account only |

> 📝 **NOTE:** RBAC role assignments can take **1–5 minutes** to propagate through Azure's identity system. If the next steps fail with a permission error, wait 2 minutes and try again.

> 👀 **UI Check:** Portal → Storage account → **Access control (IAM)** → **Role assignments** tab → filter by your email → you'll see your "Storage Blob Data Contributor" assignment listed.

> ✅ **CHECKPOINT 3**
>
> - ✅ RBAC role assigned without touching account keys
> - ✅ Assignment visible in the portal IAM blade

---

## Part 4: User Delegation SAS — The Enterprise-Grade Secure Token

### Step 8: Why User Delegation SAS Instead of Service or Account SAS?

The existing lab used `--auth-mode key` to generate a SAS token. **This is the wrong approach for production.** Here is why:

| | Account/Service SAS | User Delegation SAS |
|---|---|---|
| **Signed with** | Storage account key | Microsoft Entra ID credentials |
| **Revocation** | Must rotate the storage key (breaks ALL apps using that key) | Revoke via Entra ID — scoped, precise |
| **Audit trail** | None | Full Entra ID audit log |
| **RBAC required** | No | Yes — generating identity must have `Storage Blob Data Contributor` or higher |
| **Enterprise use** | ❌ Not recommended | ✅ Recommended |

> 🚨 **EXAM ALERT:** The User Delegation SAS requires the generating identity to have the `Storage Blob Data Delegator` or `Storage Blob Data Contributor` RBAC role. Without an RBAC role, you cannot generate a User Delegation SAS. This is by design — it forces you to use proper identity-based access.

### Step 9: Generate a User Delegation SAS Token

```bash
# Set token expiry to 2 hours from now
EXPIRY=$(date -u -d "+2 hours" '+%Y-%m-%dT%H:%MZ')
echo "Token will expire at: $EXPIRY"

# Generate the User Delegation SAS (notice: --auth-mode login, NOT key)
USER_DELEGATION_SAS=$(az storage blob generate-sas \
  --account-name $STORAGE_NAME \
  --container-name documents \
  --name "invoices/invoice-2024-001.txt" \
  --permissions r \                           # ← Read-only permission on THIS blob only
  --expiry $EXPIRY \
  --auth-mode login \                         # ← Signed with YOUR Entra ID identity (not the key)
  --as-user \                                 # ← This flag is what makes it a User Delegation SAS
  --https-only \                              # ← Only allow HTTPS access (never plain HTTP)
  --output tsv)

echo "User Delegation SAS Token: $USER_DELEGATION_SAS"
```

**Every flag explained:**
| Flag | Why It Matters |
|---|---|
| `--permissions r` | Read-only. The token cannot be used to write, delete, or list. Least-privilege. |
| `--auth-mode login` | Signs the SAS with your Entra ID credential, not the storage key |
| `--as-user` | **Critical flag** — this is what upgrades it from a Service SAS to a User Delegation SAS |
| `--https-only` | The SAS URL will only work over HTTPS. If someone uses plain HTTP, it fails. |

### Step 10: Prove the Token Works

```bash
# Construct the full SAS URL
BLOB_URL="https://$STORAGE_NAME.blob.core.windows.net/documents/invoices/invoice-2024-001.txt?$USER_DELEGATION_SAS"

echo "Testing SAS URL..."
curl -s "$BLOB_URL"
```

**Expected output:** The contents of your invoice file:
```
INVOICE #2024-001
Date: 2024-01-15
Client: AlphaAI Ltd
Amount: $15,000.00
Status: PAID
```

### Step 11: Prove the Token Cannot Write (Least Privilege Verification)

```bash
# Attempt to delete the blob using the read-only SAS — this MUST fail
echo "Testing that read-only SAS cannot delete..."
curl -s -X DELETE "$BLOB_URL"
```

**Expected output:** An XML error message containing `AuthorizationPermissionMismatch` — the token is read-only and correctly rejects the delete attempt.

This is enterprise security in action: the partner gets a URL that expires, cannot be used to delete data, and is traceable to your identity in the Entra ID audit logs.

> ✅ **CHECKPOINT 4**
>
> - ✅ User Delegation SAS generated (signed with Entra ID, not account key)
> - ✅ SAS URL successfully downloads the file
> - ✅ SAS URL correctly REJECTS the delete attempt
> - ✅ No storage account key was used in this entire section

---

## Part 5: Lifecycle Management Policy

### Step 12: Apply an Automated Tiering Policy

**Why:** In production, financial and compliance documents follow a predictable access pattern — they're accessed frequently right after creation, then rarely. Automating the tiering saves significant cost.

```bash
# Create the policy JSON
cat > lifecycle-policy.json << 'EOF'
{
  "rules": [
    {
      "name": "tier-and-delete-old-documents",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["invoices/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          }
        }
      }
    }
  ]
}
EOF

# Apply the policy to the storage account
az storage account management-policy create \
  --account-name $STORAGE_NAME \
  --resource-group az204-blob-rg \
  --policy @lifecycle-policy.json
```

> 📝 **NOTE:** Lifecycle policies execute **once per day**. You won't see tier changes in this lab — the policy is deployed and will begin managing costs automatically going forward.

> 👀 **UI Check:** Portal → Storage account → **Data management** → **Lifecycle management** → your rule `tier-and-delete-old-documents` appears with the three actions (Cool/Archive/Delete) and their day thresholds.

> ✅ **CHECKPOINT 5**
>
> - ✅ Lifecycle policy JSON created (Cool at 30 days, Archive at 90, Delete at 2555)
> - ✅ Policy applied and visible in the portal

---

## Part 6: Immutability — Protecting Compliance Data

### Step 13: Apply a Legal Hold to the Compliance Container

**The scenario:** The `compliance-records` container holds financial contracts and Q1 reports. Your legal team has notified you that these documents may be needed for regulatory review. They must not be deleted under any circumstances until further notice.

```bash
# Apply a Legal Hold to the compliance-records container
az storage container legal-hold set \
  --account-name $STORAGE_NAME \
  --resource-group az204-blob-rg \
  --container-name compliance-records \
  --tags "RegReview2024"            # ← A descriptive tag identifying this hold
```

**Expected output:** `"hasLegalHold": true`

### Step 14: Prove the Legal Hold Prevents Deletion

```bash
# Attempt to delete a protected blob — this MUST fail
echo "Attempting to delete legally protected blob..."
az storage blob delete \
  --account-name $STORAGE_NAME \
  --container-name compliance-records \
  --name "contracts/contract-2024-001.txt" \
  --auth-mode login
```

**Expected output:** An error containing `BlobImmutableDueToPolicy` or `ContainerHasLegalHold`.

The blob cannot be deleted. Not by you. Not by the subscription administrator. Not by Azure support. The hold must be released first.

> 👀 **UI Check:** Portal → Storage account → **Containers** → click `compliance-records` → Left menu → **Access policy** → scroll to **Legal holds** → you'll see `RegReview2024` listed.

### Step 15: Release the Legal Hold (So We Can Clean Up)

```bash
# Release the legal hold (simulating: legal team confirmed the case is closed)
az storage container legal-hold clear \
  --account-name $STORAGE_NAME \
  --resource-group az204-blob-rg \
  --container-name compliance-records \
  --tags "RegReview2024"
```

> ✅ **CHECKPOINT 6**
>
> - ✅ Legal Hold applied to compliance container
> - ✅ Blob deletion correctly blocked by the hold
> - ✅ Legal Hold released (required for cleanup)

---

## Part 7: Static Website Hosting

### Step 16: Enable and Deploy a Static Website

```bash
# Enable static website hosting
az storage blob service-properties update \
  --account-name $STORAGE_NAME \
  --static-website \
  --index-document index.html \
  --404-document 404.html

# Create the index page
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>AlphaAI Document Portal</title>
    <style>
        body { font-family: 'Segoe UI', Arial, sans-serif; background: #0078d4; color: white; display: flex; align-items: center; justify-content: center; height: 100vh; margin: 0; flex-direction: column; }
        h1 { font-size: 2.5em; margin-bottom: 10px; }
        p { font-size: 1.2em; opacity: 0.9; }
        .badge { background: rgba(255,255,255,0.2); padding: 8px 20px; border-radius: 20px; margin-top: 20px; }
    </style>
</head>
<body>
    <h1>📁 AlphaAI Document Portal</h1>
    <p>Enterprise document storage powered by Azure Blob Storage</p>
    <div class="badge">🔒 Secured with User Delegation SAS</div>
</body>
</html>
EOF

# Upload to $web container
az storage blob upload \
  --account-name $STORAGE_NAME \
  --container-name '$web' \
  --name index.html \
  --file index.html \
  --auth-mode login

# Get the static website URL
WEBSITE_URL=$(az storage account show \
  --name $STORAGE_NAME \
  --resource-group az204-blob-rg \
  --query "primaryEndpoints.web" \
  --output tsv)

echo "Static website live at: $WEBSITE_URL"
```

Open the URL in your browser to see your enterprise document portal.

> 👀 **UI Check:** Portal → Storage account → **Data management** → **Static website** → you'll see the **Primary endpoint** URL and the index/error document settings.

> ✅ **CHECKPOINT 7**
>
> - ✅ Static website hosting enabled
> - ✅ Custom HTML deployed to `$web` container
> - ✅ Website accessible in browser

---

## Part 8: Clean Up 🧹

> 💰 **COST WARNING:** Storage accounts accumulate charges even at rest. Delete everything now.

```bash
az group delete \
  --name az204-blob-rg \
  --yes \
  --no-wait
```

Verify after 2 minutes:
```bash
az group list --output table
```

---

## Troubleshooting

### Error: `AuthorizationPermissionMismatch` when generating User Delegation SAS
**Cause:** Your RBAC role has not propagated yet.
**Fix:** Wait 2–5 minutes after running the `az role assignment create` command. RBAC assignments are not instant.

### Error: `az storage container legal-hold set: error`
**Cause:** The storage account may need versioning enabled for some immutability features.
**Fix:** Run `az storage account blob-service-properties update --account-name $STORAGE_NAME --resource-group az204-blob-rg --enable-versioning true` then retry.

### Error: `BlobNotFound` when accessing via SAS URL
**Cause:** The blob path in the SAS URL doesn't match.
**Fix:** Ensure the `--name` in the SAS generation command exactly matches the blob's full path including virtual folders (e.g., `invoices/invoice-2024-001.txt`).

### SAS Token works but file is empty
**Cause:** The file was not created before upload.
**Fix:** Verify the file exists using `ls -la` before uploading.

---

## What We Built ✅

- [x] GPv2 Storage Account with public access **disabled** and TLS 1.2 enforced (enterprise security defaults)
- [x] Blob containers created using `--auth-mode login` (no account key exposure)
- [x] Realistic test files uploaded to simulate a document management system
- [x] RBAC role (`Storage Blob Data Contributor`) assigned via CLI, not via account keys
- [x] **User Delegation SAS** generated (signed with Entra ID, `--auth-mode login --as-user`)
- [x] SAS token proven to work (file download via `curl`)
- [x] SAS token proven to fail on write operations (least-privilege verified)
- [x] Lifecycle management policy deployed (Cool at 30d, Archive at 90d, Delete at 2555d)
- [x] Legal Hold applied to compliance container
- [x] Blob deletion blocked by Legal Hold (compliance enforced)
- [x] Legal Hold released for cleanup
- [x] Static website hosted in `$web` container
- [x] All resources deleted to prevent ongoing charges
