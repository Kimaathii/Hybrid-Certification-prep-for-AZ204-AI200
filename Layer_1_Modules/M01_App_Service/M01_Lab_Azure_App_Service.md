# M01 — Lab Guide: Azure App Service Web Apps

---

| | |
|---|---|
| **Module** | M01 — Azure App Service Web Apps |
| **Lab Title** | Deploy, Configure, and Scale a Web App with Deployment Slots |
| **Prerequisites** | F01–F05, Azure CLI installed, active Azure subscription |
| **Estimated Time** | 90 minutes |
| **What You Will Build** | A live Node.js web app with staging slot, autoscale rule, and app settings |
| **What You Will Learn** | ✅ Create App Service Plan + Web App · ✅ Deploy from GitHub · ✅ Configure app settings · ✅ Use deployment slots · ✅ Swap and rollback · ✅ Autoscale rules |

---

> 💰 **COST WARNING**
>
> This lab uses an **S1 (Standard)** App Service Plan — required for deployment slots and autoscale.
> - **Cost:** ~$0.10/hour (~$73/month if left running)
> - **If cleaned up within 2 hours: under $0.25**
>
> ⚠️ **You MUST complete the cleanup section at the end.** Do NOT leave the S1 plan running overnight — it will cost real money.

---

## Part 1: Create the Infrastructure

### Step 1: Create a Resource Group

```bash
az group create \
  --name az204-appservice-rg \       # ← Resource group name
  --location eastus                  # ← Azure region
```

**Expected output:** `"provisioningState": "Succeeded"`

### Step 2: Create an S1 Linux App Service Plan

**Why S1?** The Standard tier (S1) is the minimum tier that supports **deployment slots** and **autoscale** — both of which you need in this lab and both of which are heavily tested on the exam.

```bash
az appservice plan create \
  --name az204-appservice-plan \     # ← Plan name
  --resource-group az204-appservice-rg \
  --sku S1 \                         # ← Standard tier ($0.10/hr) — required for slots & autoscale
  --is-linux                         # ← Linux-based plan
```

| Flag | What It Does |
|---|---|
| `--sku S1` | Standard tier. Supports: deployment slots, autoscale, custom domains, VNet Integration. The **minimum tier for exam-tested features.** |
| `--is-linux` | Uses Linux as the operating system. All AZ-204 labs use Linux. |

> 🚨 **EXAM ALERT**
>
> Deployment slots require **Standard (S1) tier or higher**. Autoscale also requires **Standard or higher**. The Free (F1) and Basic (B1) tiers do NOT support slots or autoscale. If an exam question asks about the minimum tier for slots → answer is **Standard**.

### Step 3: Create a Node.js Web App

```bash
# Generate a unique name for the web app
APP_NAME=az204-webapp-$RANDOM

az webapp create \
  --name $APP_NAME \                 # ← Globally unique web app name
  --resource-group az204-appservice-rg \
  --plan az204-appservice-plan \     # ← The S1 plan from Step 2
  --runtime "NODE:18-lts"            # ← Node.js 18 LTS runtime
```

**Save your app name — you'll need it throughout the lab.** Write down the value of `$APP_NAME`.

**What success looks like:** A large JSON response with `"state": "Running"`.

> ✅ **CHECKPOINT 1 — Steps 1–3**
>
> - ✅ Resource group `az204-appservice-rg` created
> - ✅ S1 Linux App Service Plan created
> - ✅ Web app created and in `Running` state
>
> Verify: `az webapp show --name $APP_NAME --resource-group az204-appservice-rg --query state --output tsv` → should show `Running`

---

## Part 2: Deploy from GitHub

### Step 4: Deploy a Sample Application from GitHub

**Why:** You need actual application code running in your web app. Microsoft provides sample apps for AZ-204 labs.

```bash
az webapp deployment source config \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --repo-url "https://github.com/Azure-Samples/nodejs-docs-hello-world" \
  --branch master \                  # ← Branch to deploy from
  --manual-integration               # ← Don't set up webhooks (simpler for labs)
```

| Flag | What It Does |
|---|---|
| `--repo-url` | The GitHub repository containing the application code |
| `--branch` | Which branch to deploy |
| `--manual-integration` | Deploy once from the repo without setting up continuous deployment webhooks |

**This takes 1–2 minutes.** Azure clones the repo, installs dependencies, and starts the application.

### Step 5: Verify the App Is Live

```bash
# Get the app URL
APP_URL=$(az webapp show \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --query defaultHostName \
  --output tsv)

echo "Your app is live at: https://$APP_URL"
```

Open the URL in your browser. You should see: **"Hello World!"**

```bash
# Or test with curl
curl https://$APP_URL
```

> ✅ **CHECKPOINT 2 — Steps 4–5**
>
> - ✅ Sample app deployed from GitHub
> - ✅ App is accessible via browser — shows "Hello World!"

---

## Part 3: Configure Application Settings

### Step 6: Add Application Settings

**Why:** Application settings in Azure become **environment variables** inside your running app. This is how you configure apps without changing code — a pattern used in every AZ-204 module.

```bash
az webapp config appsettings set \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --settings \
    ENVIRONMENT="Production" \
    APP_VERSION="1.0.0" \
    FEATURE_FLAG_DARK_MODE="true"
```

**Expected output:** A JSON array showing all three settings with their names and values.

### Step 7: Add a Connection String

**Why:** Connection strings are a special type of app setting used for database connections. Azure treats them differently from regular settings — they have a **type** (SQLServer, MySQL, Custom, etc.).

```bash
az webapp config connection-string set \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --connection-string-type Custom \  # ← Type of connection (Custom, SQLServer, MySQL, etc.)
  --settings \
    MyDatabase="Server=myserver.database.windows.net;Database=mydb;"
```

### Step 8: Verify Settings in the Portal

**Action:**
1. **Portal** → Search for your app name → click the **App Service**
2. Left menu → **Settings** → **Environment variables**
3. **App settings** tab → verify `ENVIRONMENT`, `APP_VERSION`, `FEATURE_FLAG_DARK_MODE`
4. **Connection strings** tab → verify `MyDatabase`

> 💡 **KEY CONCEPT**
>
> Application settings are **encrypted at rest** and transmitted over an **encrypted channel**. They override values in your application's configuration files (like `appsettings.json` in .NET). The value in Azure always wins over the value in code.

> ✅ **CHECKPOINT 3 — Steps 6–8**
>
> - ✅ Three application settings added
> - ✅ One connection string added
> - ✅ All visible in the portal

---

## Part 4: Deployment Slots — Zero-Downtime Deployments

### Step 9: Create a Staging Deployment Slot

**Why:** Deployment slots let you deploy a new version to a **staging environment**, test it, and then **swap** it into production with zero downtime.

Think of it like a **rehearsal stage** — the actors practise on the side stage while the main stage is performing. When rehearsal is perfect, you swap the stages instantly.

```bash
az webapp deployment slot create \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --slot staging                     # ← Name of the slot
```

**What success looks like:** A JSON response showing the staging slot with `"state": "Running"`.

**The staging slot URL is:** `https://<APP_NAME>-staging.azurewebsites.net`

> 📝 **NOTE**
>
> The slot URL follows the pattern: `https://{app-name}-{slot-name}.azurewebsites.net`. The production slot has no suffix — it's just `https://{app-name}.azurewebsites.net`.

### Step 10: Deploy a Different Version to Staging

```bash
az webapp deployment source config \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --slot staging \                   # ← Deploy to staging, not production
  --repo-url "https://github.com/Azure-Samples/nodejs-docs-hello-world" \
  --branch master \
  --manual-integration
```

Add a setting to the staging slot to distinguish it:

```bash
az webapp config appsettings set \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --slot staging \
  --settings APP_VERSION="2.0.0-staging"
```

### Step 11: Configure a Sticky (Slot-Specific) Setting

**Why:** **Sticky settings** stay with the slot — they do NOT move during a swap. This is critical for settings that are environment-specific (like database connection strings that point to different databases for staging vs production).

```bash
az webapp config appsettings set \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --slot-settings \
    SLOT_IDENTIFIER="staging"        # ← This setting is STICKY — stays with the slot
```

> 🚨 **EXAM ALERT**
>
> **Sticky settings stay with the SLOT, not with the app code.** During a swap:
> - ❌ Sticky settings do NOT move — they stay where they are
> - ✅ Non-sticky settings DO move — they travel with the app code
>
> If the exam asks "After swapping staging to production, which slot has the production database connection string?" → The answer is: the production slot KEEPS its production connection string (because it's sticky).

### Step 12: Verify Both Slots Are Running

```bash
# Check production
curl -s https://$APP_NAME.azurewebsites.net

# Check staging
curl -s https://$APP_NAME-staging.azurewebsites.net
```

Both should return "Hello World!" but with different app settings.

> ✅ **CHECKPOINT 4 — Steps 9–12**
>
> - ✅ Staging slot created
> - ✅ Code deployed to staging
> - ✅ Sticky setting configured
> - ✅ Both slots accessible via their URLs

---

## Part 5: Swap and Rollback

### Step 13: Swap Staging to Production

**Why:** This is the core deployment pattern — swap the tested staging version into production with **zero downtime**.

```bash
az webapp deployment slot swap \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --slot staging \                   # ← The source slot (what is being promoted)
  --target-slot production           # ← The destination (what it replaces)
```

**What happens during a swap:**
1. Azure warms up the staging slot (sends it real traffic patterns)
2. Azure swaps the virtual IPs — staging becomes production and vice versa
3. Users experience **zero downtime** — the swap is atomic

**This takes 30–60 seconds.**

### Step 14: Verify the Swap Worked

```bash
# Production should now have the staging version
az webapp config appsettings list \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --query "[?name=='APP_VERSION'].value" \
  --output tsv
```

**Expected output:** `2.0.0-staging` (the staging version is now in production)

```bash
# The sticky setting should NOT have moved
az webapp config appsettings list \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --query "[?name=='SLOT_IDENTIFIER'].value" \
  --output tsv
```

**Expected output:** Empty or the production value — the sticky setting stayed with production.

### Step 15: Rollback — Swap Back

**Why:** If the new version has a bug, you can instantly roll back by swapping again.

```bash
az webapp deployment slot swap \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --slot staging \
  --target-slot production
```

**What happens:** The previous production version (now in staging) swaps back to production. Instant rollback.

> 💡 **KEY CONCEPT**
>
> After a swap, the **old production version is sitting in staging**. If you need to roll back, just swap again. This is why deployment slots are so powerful — you always have the previous version ready to go.

> ✅ **CHECKPOINT 5 — Steps 13–15**
>
> - ✅ Swapped staging → production successfully
> - ✅ Verified the app version changed in production
> - ✅ Verified sticky settings stayed with their slot
> - ✅ Performed a rollback (swap-back)

---

## Part 6: Autoscale

### Step 16: Enable Autoscale with a CPU-Based Rule

**Why:** Autoscale automatically adds or removes instances based on metrics. This is how production apps handle traffic spikes without manual intervention.

**What you are configuring:** When CPU usage exceeds 70% for 5 minutes, add 1 instance. When CPU drops below 30% for 5 minutes, remove 1 instance. Maximum 3 instances.

```bash
# Create an autoscale setting
az monitor autoscale create \
  --resource-group az204-appservice-rg \
  --name az204-autoscale \
  --resource $APP_NAME \             # ← The web app to scale
  --resource-type "Microsoft.Web/serverfarms" \
  --resource-namespace "Microsoft.Web" \
  --min-count 1 \                    # ← Minimum 1 instance (never go below this)
  --max-count 3 \                    # ← Maximum 3 instances (never exceed this)
  --count 1                          # ← Default (starting) instance count
```

```bash
# Add scale-OUT rule (add instance when CPU > 70%)
az monitor autoscale rule create \
  --resource-group az204-appservice-rg \
  --autoscale-name az204-autoscale \
  --condition "CpuPercentage > 70 avg 5m" \  # ← Metric > threshold, averaged over 5 min
  --scale out 1                      # ← Add 1 instance
```

```bash
# Add scale-IN rule (remove instance when CPU < 30%)
az monitor autoscale rule create \
  --resource-group az204-appservice-rg \
  --autoscale-name az204-autoscale \
  --condition "CpuPercentage < 30 avg 5m" \
  --scale in 1                       # ← Remove 1 instance
```

> 🚨 **EXAM ALERT**
>
> Autoscale **adds identical instances** (scale OUT). It does NOT increase the size of existing instances. Scaling from 1 to 3 instances means you now have 3 identical copies of your app behind a load balancer. This is **horizontal scaling** (scale out), not **vertical scaling** (scale up).
>
> Also: autoscale requires **Standard tier or higher**. Free and Basic tiers cannot autoscale.

> ✅ **CHECKPOINT 6 — Step 16**
>
> - ✅ Autoscale setting created with min=1, max=3
> - ✅ Scale-out rule: CPU > 70% → add 1 instance
> - ✅ Scale-in rule: CPU < 30% → remove 1 instance

---

## Part 7: Clean Up 🧹

> 💰 **COST WARNING**
>
> The S1 plan costs **$0.10/hour**. If you leave it running for 24 hours, that's $2.40. For a month, it's $73. **Delete everything now.**

### Step 17: Delete Everything

```bash
# Delete the entire resource group (deletes everything inside)
az group delete \
  --name az204-appservice-rg \
  --yes \                            # ← Skip confirmation
  --no-wait                          # ← Run in background
```

### Step 18: Verify Cleanup

Wait 2 minutes, then:

```bash
az group list --output table
```

`az204-appservice-rg` should no longer appear.

---

## Troubleshooting

### Error: `"Conflict: Cannot modify this site because it is being swapped"`
**Cause:** A swap is in progress. **Fix:** Wait 1–2 minutes for the swap to complete.

### Error: `"Cannot create deployment slot for Free/Basic tier"`
**Cause:** Your plan is not Standard or higher. **Fix:** Ensure you created an S1 plan (`--sku S1`).

### Error: `"The resource name is already in use"`
**Cause:** Web app names are globally unique. **Fix:** Use `$RANDOM` in the name.

### Error: `"Deployment source config failed"`
**Cause:** GitHub URL is incorrect or the repo is private. **Fix:** Verify the repo URL is correct and publicly accessible.

### Error: `"Autoscale setting failed — resource not found"`
**Cause:** The resource parameter doesn't match. **Fix:** Use the App Service Plan name (not the web app name) as the `--resource`.

---

## What We Covered ✅

- [ ] Created a resource group for App Service resources
- [ ] Created an S1 (Standard) Linux App Service Plan
- [ ] Created a web app with Node.js runtime
- [ ] Deployed sample code from GitHub
- [ ] Added application settings and connection strings
- [ ] Verified settings in the Azure portal
- [ ] Created a staging deployment slot
- [ ] Configured sticky (slot-specific) settings
- [ ] Performed a slot swap (staging → production)
- [ ] Verified swap behaviour (sticky vs non-sticky settings)
- [ ] Performed a rollback (swap back)
- [ ] Configured autoscale rules (scale out on CPU > 70%, scale in on CPU < 30%)
- [ ] Understood the difference between scale out and scale up
- [ ] Cleaned up all resources
