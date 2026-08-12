# M10 — Lab Guide: Key Vault, Managed Identity & App Configuration (Enterprise Edition)

---

| | |
|---|---|
| **Module** | M10 — Secrets & Configuration |
| **Lab Title** | Build a Zero-Trust Secret Management Architecture with Key Vault and Managed Identity |
| **Prerequisites** | M01 Lab (App Service + Managed Identity), active Azure subscription, Azure Cloud Shell (Bash) |
| **Estimated Time** | 90 minutes |
| **What You Will Build** | A complete zero-trust connection: App Service → Managed Identity → Key Vault Reference, with no passwords in code or config |
| **What You Will Learn** | ✅ Create Key Vault (RBAC mode) · ✅ Store secrets · ✅ Grant MI permissions · ✅ Configure Key Vault Reference · ✅ Verify via portal Green Checkmark · ✅ Soft Delete & Purge |
| **Tool** | Azure Cloud Shell (Bash) |

---

> ⚠️ **IMPORTANT: Use Azure Cloud Shell for the entire lab.**
> Open [portal.azure.com](https://portal.azure.com) → Click the `>_` icon → Select **Bash**.

---

> 💰 **COST WARNING**
>
> - **Key Vault:** ~$0.03 per 10,000 operations (essentially free for labs)
> - **App Configuration Free tier:** 1,000 requests/day, no charge
> - **Estimated total cost: under $0.05 if cleaned up same day**

---

## ⚡ Fast-Forward Prerequisites Check

This lab continues directly from the M01 Lab. Your `az204-appservice-rg` resource group and App Service should already exist with Managed Identity enabled.

**Run this check first:**

```bash
# Check if the M01 App Service resource group still exists
az group show --name az204-appservice-rg --query properties.provisioningState --output tsv
```

**If it shows `Succeeded`** → Great, proceed to Part 1.

**If it shows an error** → Your M01 resources were cleaned up. Run this Fast-Forward script to recreate them:

```bash
# FAST-FORWARD SCRIPT — Only run if your M01 lab was cleaned up

APP_NAME=az204webapp$RANDOM
echo "New app name: $APP_NAME"

az group create --name az204-appservice-rg --location eastus
az appservice plan create --name az204-appservice-plan --resource-group az204-appservice-rg --sku S1 --is-linux

az webapp create \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --plan az204-appservice-plan \
  --runtime "DOTNETCORE|9.0"

# Enable Managed Identity immediately
az webapp identity assign \
  --name $APP_NAME \
  --resource-group az204-appservice-rg

echo "Fast-forward complete. App: $APP_NAME"
```

**If you just finished the M01 lab in this session**, your variables may still be set. Verify:

```bash
echo "APP_NAME: $APP_NAME"
echo "PRINCIPAL_ID: $PRINCIPAL_ID"
```

If they are empty, retrieve them:

```bash
# Retrieve your App Service name (list all apps in the resource group)
APP_NAME=$(az webapp list \
  --resource-group az204-appservice-rg \
  --query "[0].name" \
  --output tsv)
echo "App Service: $APP_NAME"

# Retrieve the Managed Identity Principal ID
PRINCIPAL_ID=$(az webapp identity show \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --query principalId \
  --output tsv)
echo "Principal ID: $PRINCIPAL_ID"
```

---

## Part 1: Create the Key Vault (Enterprise Configuration)

### Step 1: Create a Dedicated Resource Group for Secrets

**Why a separate resource group?** In enterprise environments, the Key Vault is often in a separate resource group with separate RBAC controls. This ensures that even if a developer accidentally deletes the application resource group, the secrets remain protected.

```bash
KV_NAME=kv-az204-$RANDOM
echo "Key Vault name: $KV_NAME"   # ← WRITE THIS DOWN

az group create \
  --name az204-secrets-rg \
  --location eastus \
  --tags Project=AZ204 Module=M10 Sensitivity=High
```

### Step 2: Create the Key Vault with Production-Grade Settings

```bash
az keyvault create \
  --name $KV_NAME \
  --resource-group az204-secrets-rg \
  --location eastus \
  --enable-rbac-authorization true \    # ← Modern RBAC mode (NOT legacy Access Policies)
  --retention-days 7 \                  # ← Soft delete retention (7 days for labs, 90 for production)
  --sku standard
```

| Flag | What It Does | Why It Matters |
|---|---|---|
| `--enable-rbac-authorization true` | Uses Azure RBAC for authorization | Modern, recommended approach. Legacy Access Policies are being deprecated. |
| `--retention-days 7` | Soft-deleted secrets survive for 7 days | Protects against accidental deletion. Set to 90 in production. |

> 📝 **NOTE:** We deliberately did NOT set `--enable-purge-protection true` in this lab. Purge Protection cannot be disabled after creation. If you enable it in a lab, you cannot reuse the vault name for 7–90 days after deletion. This is the trap the Concept module warned you about.

> 👀 **UI Check:** Portal → **Key vaults** → click `$KV_NAME` → Left menu → **Settings** → **Properties** → verify "Vault access policy" is **Azure role-based access control (RBAC)**.

### Step 3: Store a Simulated Azure OpenAI API Key as a Secret

```bash
az keyvault secret set \
  --vault-name $KV_NAME \
  --name "OpenAIApiKey" \
  --value "sk-fake-openai-key-AZ204-lab-do-not-use-in-production"
```

**Expected output:**
```json
{
  "id": "https://kv-az204-XXXXX.vault.azure.net/secrets/OpenAIApiKey/...",
  "name": "OpenAIApiKey",
  "value": "sk-fake-openai-key-AZ204-lab-do-not-use-in-production"
}
```

```bash
# Store the Secret URI — you'll need this to create the Key Vault Reference
SECRET_URI=$(az keyvault secret show \
  --vault-name $KV_NAME \
  --name "OpenAIApiKey" \
  --query id \
  --output tsv)
echo "Secret URI: $SECRET_URI"
```

> 👀 **UI Check:** Portal → Key vault → **Objects** → **Secrets** → click `OpenAIApiKey` → click the current version → you'll see the **Secret Identifier URI** and a **Show Secret Value** button (the value is masked by default).

> ✅ **CHECKPOINT 1**
>
> - ✅ Key Vault created in RBAC authorization mode
> - ✅ Secret stored — value not visible in code or config files
> - ✅ Secret URI captured for use in Key Vault Reference

---

## Part 2: Grant the App Service Access to Key Vault

This is the critical step. Without this, the App Service has no permission to read the secret, even though it has an identity.

### Step 4: Grant the Managed Identity the Key Vault Secrets User Role

```bash
# Get the Key Vault resource ID
KV_ID=$(az keyvault show \
  --name $KV_NAME \
  --resource-group az204-secrets-rg \
  --query id \
  --output tsv)
echo "Key Vault ID: $KV_ID"

# Grant the App Service Managed Identity the "Key Vault Secrets User" role
az role assignment create \
  --assignee $PRINCIPAL_ID \            # ← The App Service's Managed Identity Principal ID
  --role "Key Vault Secrets User" \     # ← Read-only access to secrets (cannot create or delete)
  --scope $KV_ID                        # ← Scoped to THIS vault only (least privilege)
```

**Breaking down the RBAC principle of least privilege:**

| What We Assigned | What It Can Do | What It CANNOT Do |
|---|---|---|
| `Key Vault Secrets User` | Read (Get) secrets, List secret versions | Create, Delete, or Update secrets |
| Scoped to `$KV_ID` | Access THIS Key Vault only | Access any other Key Vault in the subscription |

> 🚨 **EXAM ALERT:** `Key Vault Secrets User` = read secrets. `Key Vault Secrets Officer` = create/edit/delete secrets. The exam will give you a scenario and ask which role is needed. A web app only needs to *read* secrets → `Secrets User`. A CI/CD pipeline that *writes* secrets during deployment → `Secrets Officer`.

> 📝 **NOTE:** RBAC role assignments propagate in **1–5 minutes**. If the next steps fail with `Forbidden`, wait 3 minutes and retry.

> 👀 **UI Check:** Portal → Key vault → Left menu → **Access control (IAM)** → **Role assignments** tab → filter by role `Key Vault Secrets User` → you'll see your App Service name listed as the assignee.

> ✅ **CHECKPOINT 2**
>
> - ✅ RBAC role assigned: App Service identity → Key Vault Secrets User
> - ✅ Scoped to the specific Key Vault only
> - ✅ Assignment visible in the Key Vault IAM blade

---

## Part 3: Configure the Key Vault Reference (Zero Code Pattern)

### Step 5: What Is a Key Vault Reference?

> 💡 **KEY CONCEPT: The Magic Sticky Note**
>
> Normally, to read a secret from Key Vault in your app, you have to write SDK code with `SecretClient`, install NuGet packages, handle exceptions, etc.
>
> A **Key Vault Reference** lets you skip ALL of that. You put a special string into your App Service's App Settings. App Service reads the string, goes to Key Vault, gets the secret, and injects the **actual secret value** into your app as a normal environment variable.
>
> Your application code sees `"sk-fake-openai-key-..."` as a regular environment variable. It has no idea a Key Vault was involved.

**The syntax format:**
```
@Microsoft.KeyVault(SecretUri=<your-secret-uri>)
```

### Step 6: Set the Key Vault Reference in App Service

```bash
# Construct the Key Vault Reference string
KV_REFERENCE="@Microsoft.KeyVault(SecretUri=$SECRET_URI)"
echo "Reference string: $KV_REFERENCE"

# Set it as an App Service App Setting
az webapp config appsettings set \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --settings OPENAI_API_KEY="$KV_REFERENCE"
```

**What just happened:**
- You set an App Setting called `OPENAI_API_KEY`
- Its value is the Key Vault Reference string, NOT the actual API key
- App Service will automatically resolve this reference at runtime using the Managed Identity
- Your `.NET` code reads `Environment.GetEnvironmentVariable("OPENAI_API_KEY")` and gets the real key value

### Step 7: Verify the Key Vault Reference (The Green Checkmark)

```bash
# Check the status of the Key Vault reference
az webapp config appsettings list \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --query "[?name=='OPENAI_API_KEY']"
```

> 👀 **UI Check (THE KEY VERIFICATION STEP):**
> Portal → App Service → Left menu → **Settings** → **Environment variables** → find `OPENAI_API_KEY`.
>
> - If the reference is working: you'll see a **🟢 Green circle with a checkmark** next to the value. This means App Service successfully connected to Key Vault using the Managed Identity and resolved the secret.
> - If the reference is failing: you'll see a **🔴 Red circle with an X** and a status message. The most common causes are: Managed Identity not enabled, RBAC role not assigned yet, or incorrect Secret URI.

> 🚨 **EXAM ALERT:** The Green Checkmark status in the portal is a real exam question target. Knowing what causes a Red X (missing identity, missing RBAC, wrong URI) is a key exam topic.

> ✅ **CHECKPOINT 3**
>
> - ✅ Key Vault Reference set in App Service App Settings
> - ✅ Green checkmark visible in the portal (or RBAC assignment propagating — wait 3 min)
> - ✅ App service reads the secret with zero code changes

---

## Part 4: Create an Azure App Configuration Store

### Step 8: Create the App Configuration Store

```bash
APPCONFIG_NAME=appcs-az204-$RANDOM
echo "App Config name: $APPCONFIG_NAME"

az appconfig create \
  --name $APPCONFIG_NAME \
  --resource-group az204-secrets-rg \
  --location eastus \
  --sku free
```

### Step 9: Store Non-Sensitive Configuration Values

> 💡 **KEY CONCEPT: What Goes in Key Vault vs App Configuration?**
>
> | Item | Where It Goes | Why |
> |---|---|---|
> | Azure OpenAI API Key | **Key Vault** (Secret) | Highly sensitive — financial liability if leaked |
> | Azure OpenAI Endpoint URL | **App Configuration** | Not sensitive — it's a URL, not a credential |
> | Feature Flag: EnableAIChat | **App Configuration** | Non-sensitive boolean toggle |
> | Database Password | **Key Vault** (Secret) | Sensitive |
> | Database Server Hostname | **App Configuration** | Not sensitive |

```bash
# Store the AI endpoint (non-sensitive config, NOT a secret)
az appconfig kv set \
  --name $APPCONFIG_NAME \
  --key "AI:Endpoint" \
  --value "https://placeholder.openai.azure.com/" \
  --yes

# Create a Feature Flag for the AI Chat feature
az appconfig feature set \
  --name $APPCONFIG_NAME \
  --feature "EnableAIChat" \
  --yes

# Set the Sentinel key (used for Dynamic Refresh)
az appconfig kv set \
  --name $APPCONFIG_NAME \
  --key "App:Sentinel" \
  --value "1" \
  --yes
```

> 👀 **UI Check:** Portal → App Configuration → **Operations** → **Configuration explorer** → you'll see `AI:Endpoint` and `App:Sentinel` listed. Click **Feature manager** to see the `EnableAIChat` feature flag.

### Step 10: Test Dynamic Refresh — Toggle the Feature Flag Live

```bash
# Check the current state of the feature flag
az appconfig feature show \
  --name $APPCONFIG_NAME \
  --feature "EnableAIChat" \
  --query state \
  --output tsv
```

**Expected output:** `on`

```bash
# Disable the feature (simulating: AI model is hallucinating, kill switch activated)
az appconfig feature disable \
  --name $APPCONFIG_NAME \
  --feature "EnableAIChat" \
  --yes

echo "Feature disabled without touching any code or redeploying the app"

# Update the sentinel key to signal the app to refresh
az appconfig kv set \
  --name $APPCONFIG_NAME \
  --key "App:Sentinel" \
  --value "2" \
  --yes
```

> 👀 **UI Check:** Portal → App Configuration → **Feature manager** → toggle `EnableAIChat` back on using the portal UI. This simulates your operations team using the portal to control app behaviour.

> ✅ **CHECKPOINT 4**
>
> - ✅ App Configuration store created (free tier)
> - ✅ Non-sensitive settings and feature flag stored
> - ✅ Sentinel key configured for Dynamic Refresh
> - ✅ Feature flag disabled via CLI (no code change, no redeployment)

---

## Part 5: Demonstrate Soft Delete

### Step 11: Accidentally Delete a Secret

```bash
# Simulate an accidental secret deletion
az keyvault secret delete \
  --vault-name $KV_NAME \
  --name "OpenAIApiKey"

echo "Secret 'deleted'..."

# Try to read the secret — it should fail
az keyvault secret show \
  --vault-name $KV_NAME \
  --name "OpenAIApiKey"
```

**Expected:** Error — `SecretNotFound`

### Step 12: Recover the Soft-Deleted Secret

```bash
# List soft-deleted secrets in the vault
az keyvault secret list-deleted \
  --vault-name $KV_NAME \
  --query "[].{name:name, deletedDate:attributes.deleted}" \
  --output table

# Recover the secret (bring it back from soft-delete)
az keyvault secret recover \
  --vault-name $KV_NAME \
  --name "OpenAIApiKey"

echo "Secret recovered successfully"

# Verify it is back
az keyvault secret show \
  --vault-name $KV_NAME \
  --name "OpenAIApiKey" \
  --query value \
  --output tsv
```

**Expected:** The secret value is restored: `sk-fake-openai-key-AZ204-lab-do-not-use-in-production`

> 💡 **KEY CONCEPT:** This is why Soft Delete exists. A real-world scenario: a developer runs the wrong script and deletes the production OpenAI key. Without Soft Delete, the application goes down and the key is gone permanently. With Soft Delete, the operations team recovers it in seconds.

> ✅ **CHECKPOINT 5**
>
> - ✅ Secret "accidentally" deleted
> - ✅ Secret successfully recovered from soft-delete state
> - ✅ Key Vault Reference in App Service automatically reconnects after recovery

---

## Part 6: Clean Up 🧹

> 💰 **COST WARNING:** Key Vault and App Configuration are low-cost but should still be cleaned up.

### Step 13: Delete the Resource Groups

```bash
# Delete secrets resource group
az group delete \
  --name az204-secrets-rg \
  --yes \
  --no-wait

# Delete App Service resource group (if no longer needed)
az group delete \
  --name az204-appservice-rg \
  --yes \
  --no-wait
```

### Step 14: Purge the Soft-Deleted Key Vault

> 🚨 **CRITICAL:** Because Soft Delete is enabled (default), the Key Vault still exists in a deleted state. If you try to create a new vault with the same name, it will fail. You MUST purge it.

```bash
# List soft-deleted vaults
az keyvault list-deleted --query "[].{name:name}" --output table

# Purge the vault (permanently destroy, cannot be recovered)
az keyvault purge --name $KV_NAME --no-wait

echo "Key Vault purged. Name is now available for reuse."
```

### Step 15: Verify Cleanup

```bash
# Wait 2–3 minutes, then verify
az group list --output table
az keyvault list-deleted --output table
```

Both resource groups and the deleted vault should be gone.

---

## Troubleshooting

### Error: Red X next to Key Vault Reference in Portal
**Causes (in order of likelihood):**
1. Managed Identity not enabled on the App Service → Go to App Service → Identity → turn System Assigned to On
2. RBAC role not yet propagated → Wait 5 minutes after `az role assignment create`
3. Wrong Secret URI format → Verify `$SECRET_URI` does not have an extra `https://` or trailing character
4. Key Vault RBAC mode disabled → Check `az keyvault show --name $KV_NAME --query properties.enableRbacAuthorization`

### Error: `Forbidden` when granting RBAC role
**Cause:** You don't have permission to assign roles in this subscription.
**Fix:** Contact your subscription administrator to add you as `User Access Administrator`.

### Error: `VaultAlreadyExists` when recreating the vault
**Cause:** The vault is in soft-delete state.
**Fix:** Run `az keyvault purge --name $KV_NAME` then recreate.

### Error: `SecretNotFound` after recovering
**Cause:** Recovery is asynchronous and may take 1–2 minutes.
**Fix:** Wait 2 minutes then retry the `show` command.

---

## What We Built ✅

- [x] Key Vault created with **RBAC authorization mode** (modern) and **Soft Delete**
- [x] Azure OpenAI API key stored as a Secret (no hardcoding anywhere)
- [x] Managed Identity from M01 App Service granted `Key Vault Secrets User` role (least privilege)
- [x] **Key Vault Reference** configured in App Service App Settings (`@Microsoft.KeyVault(...)`)
- [x] **Green Checkmark** verified in the portal — zero-trust connection confirmed
- [x] App Configuration store created (free tier)
- [x] Non-sensitive config (`AI:Endpoint`) stored separately from secrets
- [x] Feature Flag (`EnableAIChat`) created and toggled off via CLI (no redeploy)
- [x] **Sentinel key** configured for Dynamic Refresh pattern
- [x] **Soft Delete demonstrated** — secret "accidentally" deleted and recovered in seconds
- [x] Resources cleaned up — vault **purged** to free the name
