# M01 — Lab Guide: Azure App Service Web Apps (Enterprise Edition)

---

| | |
|---|---|
| **Module** | M01 — Azure App Service Web Apps |
| **Lab Title** | Build, Deploy, Scale, and Secure a .NET Web API on App Service |
| **Prerequisites** | F01–F05, active Azure subscription, Azure Cloud Shell (Bash) |
| **Estimated Time** | 120 minutes |
| **What You Will Build** | A live .NET 9 Minimal API deployed to Azure App Service with deployment slots, autoscale, and a Managed Identity |
| **What You Will Learn** | ✅ Create App Service Plan + Web App · ✅ Build and deploy a real .NET API · ✅ Configure app settings · ✅ Use deployment slots · ✅ Swap and rollback · ✅ Autoscale rules · ✅ Enable Managed Identity |
| **Tool** | Azure Cloud Shell (Bash) — everything runs in the browser, nothing to install |

---

> ⚠️ **IMPORTANT: Use Azure Cloud Shell for the entire lab.**
> Open [portal.azure.com](https://portal.azure.com) → Click the `>_` icon in the top toolbar → Select **Bash**.
> Cloud Shell has .NET 9 SDK, Azure CLI, and `curl` pre-installed. You do not need to install anything on your local machine.

---

> 💰 **COST WARNING**
>
> This lab uses an **S1 (Standard)** App Service Plan — required for deployment slots and autoscale.
> - **Cost:** ~$0.10/hour (~$73/month if left running)
> - **If cleaned up within 2 hours: under $0.30**
>
> ⚠️ **You MUST complete the cleanup section at the end.** The M10 lab will re-create what it needs.

---

## Part 1: Create the Infrastructure

### Step 1: Create a Resource Group

**Why:** A resource group is a logical folder for your project. Deleting it deletes everything inside — your clean-up mechanism.

```bash
az group create \
  --name az204-appservice-rg \       # ← Resource group name
  --location eastus \                 # ← Azure region
  --tags Project=AZ204 Module=M01    # ← Tags for cost tracking
```

**Expected output:** `"provisioningState": "Succeeded"`

> 👀 **UI Check:** Portal → search **"Resource Groups"** → find `az204-appservice-rg`. Notice the tags you applied are visible under the **Tags** column.

### Step 2: Create an S1 Linux App Service Plan

**Why S1?** The Standard tier is the minimum tier that supports **deployment slots**, **autoscale**, and **VNet Integration** — all heavily tested on AZ-204.

```bash
az appservice plan create \
  --name az204-appservice-plan \
  --resource-group az204-appservice-rg \
  --sku S1 \                         # ← Standard tier ($0.10/hr)
  --is-linux                         # ← Linux OS
```

| Flag | What It Does |
|---|---|
| `--sku S1` | Standard tier. Supports: deployment slots, autoscale, VNet Integration. |
| `--is-linux` | Linux OS. All AZ-204 labs use Linux. |

> 🚨 **EXAM ALERT**
> Deployment slots, autoscale, and VNet Integration all require **Standard (S1) tier or higher**. Free (F1) and Basic (B1) do NOT support any of these features.

> 👀 **UI Check:** Portal → **App Service Plans** → click `az204-appservice-plan` → Left menu → **Scale up (App Service plan)** → you'll see all the tiers. Notice which features are marked as requiring Standard or higher.

### Step 3: Create the Web App Resource

```bash
# Generate a globally unique name (storage accounts and app names must be unique)
APP_NAME=az204webapp$RANDOM
echo "Your app name is: $APP_NAME"    # ← WRITE THIS DOWN. You'll need it throughout.

az webapp create \
  --name $APP_NAME \                  # ← Globally unique app name
  --resource-group az204-appservice-rg \
  --plan az204-appservice-plan \      # ← The S1 plan from Step 2
  --runtime "DOTNETCORE|9.0"         # ← .NET 9 runtime (Linux uses | separator)
```

**Expected output:** A JSON object with `"state": "Running"`.

> 👀 **UI Check:** Portal → **App Services** → click your app → the **Overview** blade shows the default domain URL (e.g., `az204webapp12345.azurewebsites.net`). The app exists but shows a placeholder page because we haven't deployed code yet.

> ✅ **CHECKPOINT 1**
>
> - ✅ Resource group created with tags
> - ✅ S1 Linux App Service Plan created
> - ✅ Web App resource created and in `Running` state
>
> Verify: `az webapp show --name $APP_NAME --resource-group az204-appservice-rg --query state --output tsv` → should show `Running`

---

## Part 2: Build and Deploy a Real .NET 9 Web API

**Why this approach matters:** In real enterprise work, you never pull code from someone else's GitHub. You build your own code and deploy it. This section teaches the exact pattern used in CI/CD pipelines.

### Step 4: Create a .NET 9 Minimal Web API in Cloud Shell

```bash
# Create a project directory
mkdir ~/az204-api && cd ~/az204-api

# Create a new .NET 9 Minimal API project
dotnet new web --name AlphaApi --framework net9.0
cd AlphaApi
```

**What `dotnet new web` creates:**
- `Program.cs` — the entire application in one file (Minimal API pattern)
- `AlphaApi.csproj` — the project configuration file
- `appsettings.json` — default configuration file

### Step 5: Customise the API to Prove It's Yours

Open the built-in editor in Cloud Shell:

```bash
code Program.cs
```

Replace the entire contents with this code:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Root endpoint — proves the app is running
app.MapGet("/", () => new {
    message = "AlphaAI API is live!",
    version = "1.0.0",
    environment = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Unknown",
    // In a real app, this would come from Key Vault via Managed Identity
    // We demonstrate this in M10
    aiKeyConfigured = !string.IsNullOrEmpty(
        Environment.GetEnvironmentVariable("OPENAI_API_ENDPOINT")
    )
});

// Health check endpoint — standard practice for all production APIs
app.MapGet("/health", () => Results.Ok(new {
    status = "Healthy",
    timestamp = DateTime.UtcNow
}));

// Info endpoint — shows environment variables set by Azure App Service
app.MapGet("/info", () => new {
    appName = Environment.GetEnvironmentVariable("WEBSITE_SITE_NAME") ?? "local",
    instanceId = Environment.GetEnvironmentVariable("WEBSITE_INSTANCE_ID") ?? "local",
    region = Environment.GetEnvironmentVariable("REGION_NAME") ?? "local"
});

app.Run();
```

Click `...` (top right of editor) → **Save** → **Close Editor**

> 📝 **NOTE**
> Notice the `/info` endpoint reads `WEBSITE_SITE_NAME` and `WEBSITE_INSTANCE_ID`. Azure App Service automatically injects these as environment variables into every running instance. This is how you can tell which instance served a request — critical for debugging autoscale issues.

### Step 6: Build and Package the API

```bash
# Build the project in Release mode
dotnet publish -c Release -o ./publish

# Package as a ZIP file for deployment
cd publish
zip -r ../app.zip .
cd ..
```

| Command | What It Does |
|---|---|
| `dotnet publish -c Release` | Compiles the code optimised for production (Release mode) |
| `-o ./publish` | Output the compiled files into a `publish` folder |
| `zip -r ../app.zip .` | Creates a ZIP of the publish output. This is what Azure receives. |

### Step 7: Deploy to App Service via ZIP Deploy

```bash
az webapp deployment source config-zip \
  --resource-group az204-appservice-rg \
  --name $APP_NAME \
  --src ./app.zip
```

| Flag | What It Does |
|---|---|
| `--resource-group` | Which resource group the App Service lives in |
| `--name` | The target App Service name |
| `--src` | Path to the ZIP file to deploy |

This takes **1–2 minutes**. Azure unpacks the ZIP, configures the .NET 9 runtime, and starts the app.

### Step 8: Verify the Live API

```bash
# Get the live URL
APP_URL=$(az webapp show \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --query defaultHostName \
  --output tsv)

echo "API is live at: https://$APP_URL"

# Test the root endpoint
curl -s https://$APP_URL | python3 -m json.tool

# Test the health check (standard enterprise pattern)
curl -s https://$APP_URL/health | python3 -m json.tool

# Test the info endpoint (shows App Service environment variables)
curl -s https://$APP_URL/info | python3 -m json.tool
```

**Expected output from root endpoint:**
```json
{
    "message": "AlphaAI API is live!",
    "version": "1.0.0",
    "environment": "Production",
    "aiKeyConfigured": false
}
```

> ✅ **CHECKPOINT 2**
>
> - ✅ .NET Minimal API created in Cloud Shell
> - ✅ API compiled, packaged, and deployed via ZIP Deploy
> - ✅ Three endpoints respond correctly: `/`, `/health`, `/info`

---

## Part 3: Configure Application Settings

### Step 9: Add App Settings (Environment Variables)

**Why:** Instead of hardcoding configuration inside your code, you inject it from outside. This is how the same compiled code runs differently in different environments.

```bash
az webapp config appsettings set \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --settings \
    ASPNETCORE_ENVIRONMENT="Production" \
    APP_VERSION="1.0.0" \
    OPENAI_API_ENDPOINT="https://placeholder.openai.azure.com/"
```

**Now test the root endpoint again:**
```bash
curl -s https://$APP_URL | python3 -m json.tool
```

**Expected output — notice `aiKeyConfigured` is now `true`:**
```json
{
    "message": "AlphaAI API is live!",
    "version": "1.0.0",
    "environment": "Production",
    "aiKeyConfigured": true
}
```

This demonstrates a critical principle: **the Azure App Setting overrode the app's default behaviour at runtime without redeploying the code.**

> 👀 **UI Check:** Portal → your App Service → Left menu → **Settings** → **Environment variables** → **App settings** tab → verify `OPENAI_API_ENDPOINT` appears. Click on it — notice the value is masked (a security feature).

> 💡 **KEY CONCEPT**
> App Settings in Azure always override values in `appsettings.json`. If your code reads `Environment.GetEnvironmentVariable("OPENAI_API_ENDPOINT")`, it gets the Azure-injected value. In M10, we will replace this placeholder value with a secure reference to Azure Key Vault.

> ✅ **CHECKPOINT 3**
>
> - ✅ App settings configured via CLI
> - ✅ Settings visible in the portal
> - ✅ App behaviour changed at runtime without redeployment

---

## Part 4: Deployment Slots — Zero-Downtime Deployments

### Step 10: Create a Staging Deployment Slot

**Why:** This is the industry-standard deployment pattern. You never deploy new code directly to production. You deploy to staging, test it, and then swap.

```bash
az webapp deployment slot create \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --slot staging
```

The staging slot gets its own URL: `https://$APP_NAME-staging.azurewebsites.net`

> 👀 **UI Check:** Portal → your App Service → Left menu → **Deployment** → **Deployment slots** → you'll see both **production** and **staging** listed with their own URLs.

### Step 11: Deploy a New Version to Staging

```bash
# Modify the API version number to simulate a new release
cat > ~/az204-api/AlphaApi/Program.cs << 'EOF'
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => new {
    message = "AlphaAI API is live!",
    version = "2.0.0",                    // ← Changed: simulates a new release
    environment = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Unknown",
    aiKeyConfigured = !string.IsNullOrEmpty(
        Environment.GetEnvironmentVariable("OPENAI_API_ENDPOINT")
    )
});

app.MapGet("/health", () => Results.Ok(new {
    status = "Healthy",
    timestamp = DateTime.UtcNow
}));

app.MapGet("/info", () => new {
    appName = Environment.GetEnvironmentVariable("WEBSITE_SITE_NAME") ?? "local",
    instanceId = Environment.GetEnvironmentVariable("WEBSITE_INSTANCE_ID") ?? "local",
    region = Environment.GetEnvironmentVariable("REGION_NAME") ?? "local"
});

app.Run();
EOF

# Rebuild and repackage
cd ~/az204-api/AlphaApi
dotnet publish -c Release -o ./publish
cd publish && zip -r ../app-v2.zip . && cd ..

# Deploy v2.0.0 to staging ONLY — production is untouched
az webapp deployment source config-zip \
  --resource-group az204-appservice-rg \
  --name $APP_NAME \
  --slot staging \                         # ← Deploy to staging, NOT production
  --src ./app-v2.zip
```

### Step 12: Configure a Sticky (Slot-Specific) Setting

**Why:** A sticky setting stays with the slot and does NOT move during a swap. Use this for environment-specific config like database connection strings.

```bash
az webapp config appsettings set \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --slot staging \
  --slot-settings \
    SLOT_NAME="staging"              # ← Sticky: stays with the staging slot forever
```

**Verify both slots independently:**
```bash
# Production should show version 1.0.0
curl -s https://$APP_NAME.azurewebsites.net | python3 -m json.tool

# Staging should show version 2.0.0
curl -s https://$APP_NAME-staging.azurewebsites.net | python3 -m json.tool
```

> 🚨 **EXAM ALERT**
> **Sticky settings stay with the SLOT, not the code.** During a swap, sticky settings do NOT travel to the other slot. This is why production can have a production database connection string and staging can have a staging database connection string — they will never accidentally cross over.

> ✅ **CHECKPOINT 4**
>
> - ✅ Staging slot created
> - ✅ v2.0.0 deployed to staging (production still on v1.0.0)
> - ✅ Sticky setting configured on staging

---

## Part 5: Swap and Rollback

### Step 13: Swap Staging → Production (Zero Downtime)

```bash
az webapp deployment slot swap \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --slot staging \                   # ← Source slot (what you're promoting)
  --target-slot production           # ← Destination (what it replaces)
```

This takes **30–60 seconds**. During this time, Azure:
1. Sends warm-up requests to the staging slot
2. Only when staging responds successfully, swaps the virtual IPs
3. Users experience zero downtime — the switch is atomic

**Verify the swap:**
```bash
# Production should now show version 2.0.0
curl -s https://$APP_NAME.azurewebsites.net | python3 -m json.tool
```

### Step 14: Rollback — Swap Back Instantly

If v2.0.0 has a bug in production, roll back in seconds:

```bash
az webapp deployment slot swap \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --slot staging \
  --target-slot production
```

The old production version (now sitting in staging after the first swap) is promoted back. **Instant rollback.** No re-deployment needed.

> 💡 **KEY CONCEPT**
> After a swap, the previous production version lives in staging. This is not waste — it is your instant rollback button. Never delete the staging slot content immediately after a swap.

> ✅ **CHECKPOINT 5**
>
> - ✅ Swapped staging → production, verified version changed
> - ✅ Rolled back by swapping again
> - ✅ Zero downtime maintained throughout

---

## Part 6: Autoscale — Handling Traffic Spikes

### Step 15: Configure Metric-Based Autoscale

**Why:** In production, you never manually adjust instance counts. You define rules and let Azure scale automatically based on real traffic.

```bash
# Step 1: Get the App Service Plan resource ID
PLAN_ID=$(az appservice plan show \
  --name az204-appservice-plan \
  --resource-group az204-appservice-rg \
  --query id --output tsv)

# Step 2: Create the autoscale profile (min/max/default instance bounds)
az monitor autoscale create \
  --resource-group az204-appservice-rg \
  --resource $PLAN_ID \              # ← Target the PLAN, not the web app
  --resource-type Microsoft.Web/serverfarms \
  --name az204-autoscale \
  --min-count 1 \                    # ← Never go below 1 instance
  --max-count 3 \                    # ← Never exceed 3 instances
  --count 1                          # ← Start with 1 instance
```

> 📝 **NOTE:** Autoscale targets the **App Service Plan** (the hardware), not the individual web app. The Plan scales, and all apps on that plan benefit.

```bash
# Step 3: Add scale-OUT rule — add an instance when CPU > 70% for 5 minutes
az monitor autoscale rule create \
  --resource-group az204-appservice-rg \
  --autoscale-name az204-autoscale \
  --condition "CpuPercentage > 70 avg 5m" \
  --scale out 1
```

```bash
# Step 4: Add scale-IN rule — remove an instance when CPU < 30% for 5 minutes
az monitor autoscale rule create \
  --resource-group az204-appservice-rg \
  --autoscale-name az204-autoscale \
  --condition "CpuPercentage < 30 avg 5m" \
  --scale in 1
```

> 🚨 **EXAM ALERT**
> **Always define BOTH scale-out AND scale-in rules.** A scale-out rule without a scale-in rule means Azure will keep adding instances but never remove them, running up your bill indefinitely. In production, this is an expensive mistake.

> 👀 **UI Check:** Portal → your App Service → Left menu → **Scale out (App Service plan)** → click **Custom autoscale** → you'll see your rules listed. Notice the min (1), max (3), and default (1) values.

> ✅ **CHECKPOINT 6**
>
> - ✅ Autoscale profile created with min=1, max=3
> - ✅ Scale-out rule: CPU > 70% → add 1 instance
> - ✅ Scale-in rule: CPU < 30% → remove 1 instance

---

## Part 7: Enable Managed Identity (The App's Identity)

> 💡 **Why this matters:** In M10 (Key Vault), your app needs to retrieve secrets without using passwords. This is how you set that up. Even if you're not doing M10 immediately, you should enable the identity now — it's a zero-cost, zero-risk operation.

### Step 16: Enable System-Assigned Managed Identity

```bash
az webapp identity assign \
  --name $APP_NAME \
  --resource-group az204-appservice-rg
```

**Expected output — note the `principalId`:**
```json
{
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy",
  "type": "SystemAssigned",
  "userAssignedIdentities": null
}
```

```bash
# Save the Principal ID for use in M10
PRINCIPAL_ID=$(az webapp identity show \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --query principalId \
  --output tsv)

echo "App Service Principal ID: $PRINCIPAL_ID"
echo "App Service Name: $APP_NAME"
echo ">>> SAVE BOTH OF THESE VALUES FOR THE M10 LAB <<<"
```

> 👀 **UI Check:** Portal → your App Service → Left menu → **Settings** → **Identity** → **System assigned** tab → Status is **On** → the **Object (principal) ID** matches your `$PRINCIPAL_ID`. This is now your app's identity in Entra ID.

> 💡 **KEY CONCEPT**
> Your App Service now has an Entra ID identity. It costs nothing. It has no permissions yet. In M10, you will grant this identity the `Key Vault Secrets User` role on your Key Vault, completing the zero-trust authentication chain.

> ✅ **CHECKPOINT 7**
>
> - ✅ System-Assigned Managed Identity enabled
> - ✅ Principal ID captured and saved
> - ✅ Identity visible in the Portal under Settings → Identity
> - 📌 **Note the values — required for M10 Lab**

---

## Part 8: Security Hardening

### Step 17: Enforce HTTPS and Minimum TLS 1.2

**Why:** In production, all traffic must be encrypted. Allowing plain HTTP is a security failure.

```bash
az webapp update \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --https-only true                  # ← Redirect all HTTP to HTTPS automatically
```

```bash
az webapp config set \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --min-tls-version 1.2             # ← Reject any TLS 1.0 or 1.1 connections
```

**Verify HTTPS redirect works:**
```bash
# This HTTP request should be redirected to HTTPS (301 response)
curl -I http://$APP_URL
```

You should see a `301 Moved Permanently` with a `Location: https://...` header.

> 👀 **UI Check:** Portal → your App Service → Left menu → **Settings** → **Configuration** → **General settings** tab → **HTTPS Only** is **On** and **Minimum Inbound TLS Version** is **1.2**.

---

## Part 9: Save Lab State for M10

Before cleanup, record the values you need for the M10 lab:

```bash
echo "============================================"
echo "M01 LAB OUTPUTS — REQUIRED FOR M10"
echo "============================================"
echo "App Name:       $APP_NAME"
echo "Resource Group: az204-appservice-rg"
echo "Principal ID:   $PRINCIPAL_ID"
echo "App URL:        https://$APP_URL"
echo "============================================"
```

> 📝 **NOTE: If you are continuing directly to M10, DO NOT run the cleanup steps below.** Proceed to M10 now. If you are stopping here, run the cleanup, and M10 will give you a "Fast-Forward" script to recreate the App Service quickly.

---

## Part 10: Clean Up 🧹

> 💰 **COST WARNING: S1 costs $0.10/hr. Delete now if you're not continuing to M10.**

### Step 18: Delete Everything

```bash
az group delete \
  --name az204-appservice-rg \
  --yes \
  --no-wait
```

### Step 19: Verify Cleanup

```bash
# Wait 2–3 minutes then run:
az group list --output table
```

`az204-appservice-rg` should no longer appear.

---

## Troubleshooting

### Error: `"Cannot create deployment slot for Free/Basic tier"`
**Cause:** Your plan is not Standard or higher.
**Fix:** Ensure you created the plan with `--sku S1`.

### Error: `"dotnet: command not found"`
**Cause:** You are running the command in PowerShell (Windows), not Bash Cloud Shell.
**Fix:** In Cloud Shell, click the **Bash** dropdown at the top and select Bash.

### Error: `"Conflict: Cannot modify this site because it is being swapped"`
**Cause:** A swap is in progress.
**Fix:** Wait 1–2 minutes for the swap to complete, then retry.

### Error: `"The resource name is already in use"`
**Cause:** Web app names are globally unique.
**Fix:** Re-run `APP_NAME=az204webapp$RANDOM` to generate a new unique name and recreate the app.

### Error: `"Autoscale setting failed — resource not found"`
**Cause:** The resource parameter is not the Plan ID.
**Fix:** Ensure you used the `PLAN_ID` variable (not the app name) as the `--resource`.

---

## What We Built ✅

- [x] Resource group with project tags
- [x] S1 Linux App Service Plan (Standard tier)
- [x] .NET 9 Minimal Web API created in Cloud Shell (no local install)
- [x] API deployed via ZIP Deploy (the enterprise CLI pattern)
- [x] App Settings configured and verified as live environment variables
- [x] Staging deployment slot created
- [x] New version deployed to staging while production runs v1.0.0
- [x] Sticky settings configured on staging slot
- [x] Zero-downtime slot swap performed (staging → production)
- [x] Instant rollback demonstrated
- [x] CPU-based autoscale rules configured (scale out at 70%, scale in at 30%)
- [x] System-Assigned Managed Identity enabled (Principal ID saved for M10)
- [x] HTTPS-only enforced and minimum TLS 1.2 set
- [x] Lab state captured for M10 continuity
