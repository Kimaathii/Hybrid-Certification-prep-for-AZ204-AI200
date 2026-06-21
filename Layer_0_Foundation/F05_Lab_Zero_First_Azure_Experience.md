# F05 — Lab Zero: Your First Complete Azure Experience

---

| | |
|---|---|
| **Lab Title** | Lab Zero — Your First Complete Azure Experience |
| **Prerequisites** | F01–F04 completed, Azure account with active subscription |
| **Estimated Time** | 90 minutes |
| **Cost** | 💰 **$0** — Uses only free-tier resources |
| **What You Will Build** | A simple web app on Azure's free tier, with an application setting, viewed live in your browser |
| **What You Will Learn** | ✅ Navigate the Azure Portal confidently · ✅ Use Cloud Shell and local CLI · ✅ Create and delete resource groups · ✅ Deploy a web app on the free tier · ✅ Configure application settings · ✅ Set up a spending budget alert |

---

> 💰 **COST WARNING**
>
> This entire lab uses **free-tier resources only**. You will pay $0 if you follow every step and complete the cleanup section at the end. However, as a best practice, **always set a spending budget alert** (Step 21) so you are never surprised by charges in future labs.

---

## Introduction

Welcome to your very first hands-on Azure lab. This is the moment where everything you have read in F01–F04 becomes **real**.

**Why does this lab exist?** Because reading about Azure and *doing* Azure are completely different experiences. Before you tackle any real AZ-204 module lab, you need to be confident that you can:

- Log into Azure from both the portal and the command line
- Create and delete resources without hesitation
- Read command output and know whether it worked or failed
- Navigate the portal to find what you just created
- Clean up after yourself so you never get an unexpected bill

**What you will have achieved by the end:**
- You will have created a live web app on the internet, seen it in your browser, configured it, and then deleted everything — all from the command line
- You will have a spending budget alert protecting your subscription
- You will feel **confident** and ready for Module 1

> 📝 **NOTE**
>
> Every step in this lab has a checkpoint. **Do NOT skip checkpoints.** They exist to catch problems early. If a checkpoint fails, stop and fix it before continuing. It is much easier to fix Step 3 than to debug Step 15 when the real problem was back at Step 3.

---

## Part 1: Verify Your Environment (Steps 1–7)

### Step 1: Verify Azure Account Access

**What you are doing:** Confirming you can sign into the Azure portal.

**Action:** Open your browser and navigate to:

```
https://portal.azure.com
```

Sign in with the Microsoft account that has your Azure subscription.

**What success looks like:** You see the Azure portal home page with the search bar at the top, your account avatar in the top-right corner, and either a list of recent resources or a welcome screen.

**What to do if it fails:**
- If you see "No subscriptions found" → You need to create a free Azure account at https://azure.microsoft.com/free
- If you see a sign-in error → Confirm you are using the correct Microsoft account (check the email address)

---

### Step 2: Navigate the Portal and Locate Cloud Shell

**What you are doing:** Finding the Cloud Shell button so you know where it is for every future lab.

**Action:** Look at the **top bar** of the portal. Find the icon that looks like this: **`>_`** (a greater-than symbol followed by an underscore). It is located to the right of the search bar, near the notifications bell 🔔.

**Do not click it yet** — just confirm you can see it.

**What success looks like:** You can point to the `>_` icon in the top bar.

---

### Step 3: Open Cloud Shell (Bash) for the First Time

**What you are doing:** Opening a terminal inside your browser that is already connected to your Azure account.

**Action:** Click the **`>_`** Cloud Shell icon in the top bar.

If this is your **first time**, Azure will show a prompt:

> "You have no storage mounted. Select a subscription to create a storage account and file share."

**Select your subscription** from the dropdown and click **Create storage**.

Azure will create a small storage account (costs pennies per month) to save your Cloud Shell files.

When prompted to choose between **Bash** and **PowerShell**, select **Bash**. This is what all AZ-204 labs use.

**What success looks like:** A black terminal panel appears at the bottom of your browser with a prompt that looks like:

```
user@Azure:~$
```

**What to do if it fails:**
- If you see "No subscription found" → Make sure you selected a subscription in the dropdown
- If the panel says "Requesting a Cloud Shell" for more than 2 minutes → Refresh the browser and try again

> ✅ **CHECKPOINT 1 — Steps 1–3**
>
> At this point you should have:
> - ✅ Successfully signed into portal.azure.com
> - ✅ Located the Cloud Shell icon in the top bar
> - ✅ Opened Cloud Shell with a Bash prompt visible
>
> **If any of these failed, STOP.** Fix them before continuing.

---

### Step 4: Verify Azure CLI Works in Cloud Shell

**What you are doing:** Confirming that the Azure CLI (the tool you use to command Azure from the terminal) is available in Cloud Shell.

**Action:** In the Cloud Shell terminal, type:

```bash
az --version
```

**What success looks like:** You see output listing the Azure CLI version and installed extensions. The first line will look something like:

```
azure-cli                         2.xx.x
```

The exact version number does not matter — what matters is that the command runs without an error.

> 📝 **NOTE**
>
> Cloud Shell comes with the Azure CLI **pre-installed and already logged in**. You do not need to run `az login` in Cloud Shell — it already knows who you are because you signed into the portal.

---

### Step 5: Verify Azure CLI on Your Local Machine

**What you are doing:** Confirming that the Azure CLI is installed on your own computer (not just in Cloud Shell). You will use your local terminal for most labs because it is faster and you can work offline on your code.

**Action:** Open your **local terminal**:
- **Windows:** Open **Windows Terminal** or **PowerShell** (press `Win + X` → select "Terminal" or "PowerShell")
- **Mac:** Open **Terminal** (press `Cmd + Space`, type "Terminal", press Enter)
- **Linux:** Open your terminal application

Type:

```bash
az --version
```

**What success looks like:** Same as Step 4 — you see the Azure CLI version number.

**What to do if it fails:**
If you see `'az' is not recognized` or `az: command not found`:
- The Azure CLI is **not installed** on your local machine
- **Windows:** Run `winget install -e --id Microsoft.AzureCLI` in PowerShell
- **Mac:** Run `brew update && brew install azure-cli` in Terminal
- **Linux:** Run `curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash`
- After installing, **close and reopen your terminal**, then try `az --version` again

---

### Step 6: Log In Locally with `az login`

**What you are doing:** Connecting your local terminal to your Azure account. Unlike Cloud Shell (which is already logged in), your local machine needs to be told who you are.

**Action:** In your **local terminal**, type:

```bash
az login
```

**What happens:** Your default web browser opens to a Microsoft sign-in page. Sign in with the same account you used for the portal. After signing in, you will see a message in the browser saying "You have logged in" and your terminal will show a JSON array of your subscriptions.

**What success looks like:**

```json
[
  {
    "cloudName": "AzureCloud",
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "isDefault": true,
    "name": "Your Subscription Name",
    "state": "Enabled",
    "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  }
]
```

**What to do if it fails:**
- If the browser does not open → Try `az login --use-device-code` instead, which gives you a code to enter at https://microsoft.com/devicelogin
- If you see "No subscriptions found" → You may be signed into the wrong Microsoft account

---

### Step 7: View Your Subscriptions

**What you are doing:** Listing all Azure subscriptions available to your account, displayed in a readable table format.

**Action:**

```bash
az account list \
  --output table        # ← Display results as a clean table instead of JSON
```

> 📝 **NOTE — Windows PowerShell Users**
>
> The `\` character at the end of a line means "this command continues on the next line" in Bash. In **Windows PowerShell**, use the backtick `` ` `` instead, or type the entire command on one line. In this lab, you can always type any multi-line command as a single line by removing the `\` and joining the parts.

**What success looks like:**

```
Name                    CloudName    SubscriptionId                        TenantId                              State    IsDefault
----------------------  -----------  ------------------------------------  ------------------------------------  -------  -----------
Your Subscription Name  AzureCloud   xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  Enabled  True
```

You should see at least one subscription with `State` = `Enabled`.

> ✅ **CHECKPOINT 2 — Steps 4–7**
>
> At this point you should have:
> - ✅ Verified Azure CLI works in Cloud Shell
> - ✅ Verified Azure CLI works on your local machine (installed if needed)
> - ✅ Successfully ran `az login` on your local machine
> - ✅ Seen your subscription(s) listed in a table
>
> **If `az login` failed, STOP.** Everything from here uses the CLI.

---

## Part 2: Create and Manage Resource Groups (Steps 8–10)

### Step 8: Create a Resource Group via the Portal

**What you are doing:** Creating a **resource group** (a container that holds related Azure resources) using the portal's graphical interface. Think of a resource group as a labelled folder — when you delete the folder, everything inside it gets deleted too.

**Action — follow this exact click path:**

1. **Portal** → Click **+ Create a resource** (big blue button, top-left)
2. In the search bar, type **Resource group** and press Enter
3. Click **Resource group** in the results
4. Click **Create**
5. Fill in the form:
   - **Subscription:** Select your subscription
   - **Resource group name:** `az204-lab0-portal-rg`
   - **Region:** `(US) East US` (or the region closest to you)
6. Click **Review + create**
7. Click **Create**

**What success looks like:** A notification appears saying "Resource group created successfully." You can click **Go to resource group** to see it.

---

### Step 9: Create a Resource Group via CLI

**What you are doing:** Creating a second resource group, this time using the command line. This is the method you will use in every single AZ-204 lab — it is faster and repeatable.

**Action:** In your **local terminal**:

```bash
az group create \
  --name az204-lab0-cli-rg \        # ← The name of the resource group
  --location eastus                  # ← The Azure region where it is created
```

**What each flag means:**

| Flag | What It Does |
|---|---|
| `--name` | The name you choose for this resource group. Must be unique within your subscription. |
| `--location` | The Azure region (data centre location). `eastus` is cheap and commonly used for labs. Other options: `westus2`, `westeurope`, `uksouth`. |

**What success looks like:**

```json
{
  "id": "/subscriptions/.../resourceGroups/az204-lab0-cli-rg",
  "location": "eastus",
  "name": "az204-lab0-cli-rg",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.Resources/resourceGroups"
}
```

The key line is `"provisioningState": "Succeeded"` — this means it worked.

---

### Step 10: List All Resource Groups

**What you are doing:** Verifying that both resource groups (the one from portal and the one from CLI) exist.

**Action:**

```bash
az group list \
  --output table        # ← Show results as a table for readability
```

**What success looks like:**

```
Name                    Location    Status
----------------------  ----------  ---------
az204-lab0-portal-rg    eastus      Succeeded
az204-lab0-cli-rg       eastus      Succeeded
```

You should see **both** resource groups listed.

> ✅ **CHECKPOINT 3 — Steps 8–10**
>
> At this point you should have:
> - ✅ Created a resource group using the portal (`az204-lab0-portal-rg`)
> - ✅ Created a resource group using the CLI (`az204-lab0-cli-rg`)
> - ✅ Verified both appear in `az group list --output table`
>
> **If either is missing, STOP.** Re-read the step and try again.

---

## Part 3: Deploy a Web App (Steps 11–16)

### Step 11: Create a Free F1 App Service Plan via CLI

**What you are doing:** An **App Service Plan** is the "hardware" that runs your web app. Think of it as renting a desk in an office — the plan is the desk, and the web app is the work you do at that desk. The **F1 (Free) tier** costs $0 — perfect for this lab.

**Action:**

```bash
az appservice plan create \
  --name az204-lab0-plan \           # ← Name of the App Service Plan
  --resource-group az204-lab0-cli-rg \  # ← Which resource group to put it in
  --sku F1 \                         # ← F1 = Free tier ($0/month)
  --is-linux                         # ← Use Linux as the operating system
```

| Flag | What It Does |
|---|---|
| `--name` | Name for this plan. You choose it. |
| `--resource-group` | The resource group that will contain this plan. We use the CLI one from Step 9. |
| `--sku` | The pricing tier. `F1` = Free. Other options: `B1` (Basic), `S1` (Standard), `P1v3` (Premium). |
| `--is-linux` | Run on Linux instead of Windows. Linux is commonly used in AZ-204 labs. |

**What success looks like:** A JSON response with `"provisioningState": "Succeeded"` and `"sku": { "name": "F1" }`.

---

### Step 12: Create a Simple Web App on the F1 Plan via CLI

**What you are doing:** Creating a **web app** — an actual application that will be live on the internet. The web app runs on the App Service Plan you just created.

**Action:**

```bash
az webapp create \
  --name az204-lab0-app-$RANDOM \    # ← Globally unique name ($RANDOM adds a random number)
  --resource-group az204-lab0-cli-rg \  # ← Same resource group as the plan
  --plan az204-lab0-plan \           # ← The plan from Step 11
  --runtime "NODE:18-lts"            # ← The programming language runtime
```

> 💡 **KEY CONCEPT — Why `$RANDOM`?**
>
> Every web app in Azure gets a public URL like `yourname.azurewebsites.net`. This URL must be **globally unique** across ALL of Azure — not just your account. Adding `$RANDOM` (which generates a random number between 0 and 32767) helps ensure your name does not clash with someone else's. If you still get a "name already taken" error, just run the command again — you will get a different random number.
>
> **Windows PowerShell users:** `$RANDOM` does not work in PowerShell. Instead, use: `az204-lab0-app-$(Get-Random)`

**What success looks like:** A large JSON response. Look for:
- `"state": "Running"` — the app is alive
- `"defaultHostName": "az204-lab0-app-XXXXX.azurewebsites.net"` — this is your URL

**Save the app name!** You will need it in the next steps. Note down the value after `--name` (including the random number).

---

### Step 13: Get the Web App URL

**What you are doing:** Retrieving the URL of the web app you just created so you can open it in your browser.

**Action:**

```bash
az webapp show \
  --name <YOUR-APP-NAME> \           # ← Replace with your actual app name from Step 12
  --resource-group az204-lab0-cli-rg \
  --query defaultHostName \          # ← Extract only the URL field from the response
  --output tsv                       # ← Plain text output (no quotes, no JSON)
```

| Flag | What It Does |
|---|---|
| `--query` | A JMESPath expression that extracts a specific field from the JSON response. `defaultHostName` is the URL field. |
| `--output tsv` | Tab-separated values — gives you plain text without JSON formatting or quotes. |

**What success looks like:**

```
az204-lab0-app-12345.azurewebsites.net
```

---

### Step 14: Open the URL in Your Browser — Verify It Is Running

**What you are doing:** Seeing your web app live on the internet!

**Action:** Open your browser and navigate to:

```
https://az204-lab0-app-XXXXX.azurewebsites.net
```

(Replace `XXXXX` with your actual random number from Step 12.)

**What success looks like:** You see a default Azure page that says something like:

> **"Your web app is running and waiting for your content"**

or a similar default landing page.

🎉 **Congratulations!** You just deployed something to Azure. It is live on the internet. Anyone with the URL can see it.

**What to do if it fails:**
- If you see a "404 Not Found" → Wait 30 seconds and refresh. New apps sometimes take a moment to start.
- If you see "Error 403" → The app might still be starting. Wait 1 minute and try again.

> ✅ **CHECKPOINT 4 — Steps 11–14**
>
> At this point you should have:
> - ✅ Created a free F1 App Service Plan
> - ✅ Created a web app running on that plan
> - ✅ Retrieved the web app URL
> - ✅ Seen the default Azure page in your browser
>
> **If you cannot see the page in your browser, STOP.** Check the troubleshooting section.

---

### Step 15: Add an Application Setting to the Web App via CLI

**What you are doing:** Adding a configuration value to your web app. **Application Settings** in Azure are how you pass configuration values (like database connection strings, API keys, or feature flags) to your running application — without putting them in your code.

Think of it like a sticky note on your desk at work: the note has a name ("WiFi Password") and a value ("Azure123!"). Your app reads the note by its name.

**Action:**

```bash
az webapp config appsettings set \
  --name <YOUR-APP-NAME> \           # ← Your app name from Step 12
  --resource-group az204-lab0-cli-rg \
  --settings GREETING="Hello from Lab Zero!"  # ← The setting: NAME="VALUE"
```

**What success looks like:**

```json
[
  {
    "name": "GREETING",
    "slotSetting": false,
    "value": "Hello from Lab Zero!"
  }
]
```

> 💡 **KEY CONCEPT**
>
> In Azure, **Application Settings automatically become environment variables** inside your running application. Your code reads them with `Environment.GetEnvironmentVariable("GREETING")` in C# or `process.env.GREETING` in Node.js. You will use this pattern in **every single module lab**.

---

### Step 16: View the Application Setting in the Portal

**What you are doing:** Verifying that the setting you just added via CLI is visible in the portal. This teaches you how to check configuration visually.

**Action — follow this exact click path:**

1. **Portal** → Search bar → type your app name → click on the **App Service** result
2. In the left menu, scroll down to **Settings**
3. Click **Environment variables**
4. Click the **App settings** tab

**What success looks like:** You see a row with:
- **Name:** `GREETING`
- **Value:** (click the eye icon 👁 to reveal) `Hello from Lab Zero!`

> ✅ **CHECKPOINT 5 — Steps 15–16**
>
> At this point you should have:
> - ✅ Added an application setting via CLI
> - ✅ Verified the setting is visible in the portal
>
> You have now used BOTH the CLI and the portal to manage the same resource. This is a critical skill.

---

## Part 4: Clean Up Everything (Steps 17–20)

> 💰 **COST WARNING**
>
> Cleaning up is **not optional**. Even though we used free-tier resources in this lab, building the habit of cleaning up after every lab is essential. In future labs, resources left running can cost real money. **Always delete your resource groups at the end of every lab.**

### Step 17: Delete the Web App via CLI

**What you are doing:** Removing the web app. After this, the URL will stop working.

**Action:**

```bash
az webapp delete \
  --name <YOUR-APP-NAME> \           # ← Your app name from Step 12
  --resource-group az204-lab0-cli-rg \
  --keep-empty-plan                  # ← Don't delete the plan yet (we delete it next)
```

**What success looks like:** The command returns with no output (silence = success for delete operations).

---

### Step 18: Delete the App Service Plan via CLI

**What you are doing:** Removing the App Service Plan (the "desk" the web app was sitting on).

**Action:**

```bash
az appservice plan delete \
  --name az204-lab0-plan \
  --resource-group az204-lab0-cli-rg \
  --yes                              # ← Skip the confirmation prompt
```

**What success looks like:** No output (silence = success).

---

### Step 19: Delete Both Resource Groups via CLI

**What you are doing:** Deleting both resource groups. This is the **nuclear cleanup** — deleting a resource group deletes **everything inside it**. This is why we put lab resources in their own resource group.

**Action:**

```bash
# Delete the CLI resource group (and everything inside it)
az group delete \
  --name az204-lab0-cli-rg \
  --yes \                            # ← Skip the "are you sure?" prompt
  --no-wait                          # ← Don't wait for it to finish (runs in background)

# Delete the portal resource group
az group delete \
  --name az204-lab0-portal-rg \
  --yes \
  --no-wait
```

| Flag | What It Does |
|---|---|
| `--yes` | Skips the confirmation prompt. Without this, Azure asks "Are you sure?" and waits for you to type `y`. |
| `--no-wait` | Returns immediately instead of waiting for the deletion to complete. Deletion continues in the background and may take 1–2 minutes. |

**What success looks like:** Both commands return immediately with no output.

---

### Step 20: Verify Everything Is Deleted

**What you are doing:** Confirming that both resource groups (and all resources inside them) have been fully deleted.

**Action:** Wait about 2 minutes (deletions happen in the background), then run:

```bash
az group list --output table
```

**What success looks like:** The `az204-lab0-portal-rg` and `az204-lab0-cli-rg` groups should **NOT** appear in the list. If you have no other resource groups, the output will be empty.

**What to do if they still appear:**
- If `provisioningState` shows `Deleting` → Wait another minute and try again
- If they are still `Succeeded` after 5 minutes → Run the delete commands from Step 19 again without `--no-wait`

> ✅ **CHECKPOINT 6 — Steps 17–20**
>
> At this point you should have:
> - ✅ Deleted the web app
> - ✅ Deleted the App Service Plan
> - ✅ Deleted both resource groups
> - ✅ Verified they no longer appear in `az group list`
>
> **Your subscription is clean.** No resources are running. No charges are accumulating.

---

## Part 5: Protect Your Wallet (Step 21)

### Step 21: Set a Spending Budget Alert in the Portal

**What you are doing:** Creating a budget alert that will email you if your Azure spending approaches a threshold you set. This is your safety net for all future labs.

**Action — follow this exact click path:**

1. **Portal** → Search bar → type **Cost Management** → click **Cost Management + Billing**
2. In the left menu, click **Cost Management**
3. Click **Budgets**
4. Click **+ Add**
5. Fill in the form:
   - **Name:** `az204-lab-budget`
   - **Reset period:** Monthly
   - **Creation date:** (leave as today)
   - **Expiration date:** Set to 3 months from now
   - **Budget amount:** `10` (this means $10/month)
6. Click **Next**
7. Set an **alert condition:**
   - **Type:** Actual
   - **% of budget:** `80` (alert at $8)
   - **Action group:** Click **Manage action groups** → create a new one:
     - **Name:** `az204-email-alert`
     - **Notification type:** Email
     - **Email:** Enter your email address
     - Save the action group
8. Click **Create**

**What success looks like:** You see your budget listed with a green bar showing $0 spent out of $10.

> 💡 **KEY CONCEPT**
>
> A $10/month budget is generous for AZ-204 labs if you clean up after every session. Most individual labs cost less than $0.25. The budget alert is a safety net — if you accidentally leave resources running overnight, the email will warn you before the bill gets significant.

> ✅ **CHECKPOINT 7 — Step 21**
>
> - ✅ Budget alert created with $10/month limit
> - ✅ Email notification configured at 80% threshold
>
> **Your subscription is now protected.**

---

## Troubleshooting Section

These are the 5 most common errors students encounter in this lab. If you hit a problem not listed here, copy the **exact error message** and search for it on [Microsoft Learn](https://learn.microsoft.com).

---

### Error 1: `'az' is not recognized` or `az: command not found`

**What it means:** The Azure CLI is not installed on your machine, or it was installed but your terminal cannot find it.

**How to fix it:**
1. Install the Azure CLI:
   - **Windows:** `winget install -e --id Microsoft.AzureCLI`
   - **Mac:** `brew update && brew install azure-cli`
   - **Linux:** `curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash`
2. **Close your terminal completely** and open a new one
3. Run `az --version` to verify

> 📝 **NOTE:** This error **never happens in Cloud Shell** because Cloud Shell has the CLI pre-installed. If you see this error, you are in your local terminal and need to install the CLI.

---

### Error 2: `Please run 'az login' to setup account`

**What it means:** Your local terminal is not logged into Azure. The CLI does not know who you are.

**How to fix it:**
1. Run `az login`
2. Complete the sign-in in your browser
3. Verify with `az account list --output table`

> 📝 **NOTE:** This error **never happens in Cloud Shell** because Cloud Shell is already logged in via the portal session.

---

### Error 3: `The resource name 'X' is already taken`

**What it means:** The name you chose for a resource (like a web app) is already being used by someone else in the world. Azure web app names must be **globally unique** because they become part of a public URL.

**How to fix it:**
1. Choose a different name by adding a random suffix:
   - **Bash:** Append `$RANDOM` to the name (e.g., `myapp-$RANDOM`)
   - **PowerShell:** Append `$(Get-Random)` (e.g., `myapp-$(Get-Random)`)
2. Run the command again with the new name

---

### Error 4: `AuthorizationFailed` — The client does not have authorization to perform action

**What it means:** Your Azure account does not have permission to create resources in this subscription. This usually happens when:
- You are using a company or school subscription that restricts resource creation
- Your free trial has expired
- You are logged into the wrong account

**How to fix it:**
1. Run `az account list --output table` and check which subscription is marked `IsDefault = True`
2. If you have multiple subscriptions, switch to the correct one:
   ```bash
   az account set --subscription "Your Subscription Name"
   ```
3. If your free trial expired, check https://portal.azure.com → **Subscriptions** to see the status
4. If using a company/school subscription, contact your IT admin — they may have a Policy restricting resource creation

---

### Error 5: `ResourceGroupNotFound` — Resource group 'X' could not be found

**What it means:** You typed the resource group name incorrectly, or it does not exist (perhaps it was already deleted or never created).

**How to fix it:**
1. List your existing resource groups to see the exact names:
   ```bash
   az group list --output table
   ```
2. Check for typos — names are **case-sensitive**
3. If the resource group does not exist, create it:
   ```bash
   az group create --name az204-lab0-cli-rg --location eastus
   ```

---

## ✅ Lab Complete Checklist

Before you close this lab, verify all of the following:

- [ ] ✅ I can sign into portal.azure.com
- [ ] ✅ I can open Cloud Shell and run Azure CLI commands
- [ ] ✅ I can run `az login` on my local machine and see my subscriptions
- [ ] ✅ I created a resource group via the portal AND via the CLI
- [ ] ✅ I deployed a web app and saw it running in my browser
- [ ] ✅ I added an application setting and verified it in the portal
- [ ] ✅ I deleted all resources and verified with `az group list --output table`
- [ ] ✅ I set up a $10/month budget alert on my subscription

**If all 8 boxes are checked: you are ready for Module 1.** 🎉

---

## What Comes Next

You have completed the Foundation Pack. You are now ready to begin **Layer 1 — Module Packs**.

Your next document is: **M01 — Azure App Service Web Apps**

In Module 1, you will:
- Create a real web application on Azure App Service
- Deploy code from GitHub
- Use deployment slots for zero-downtime updates
- Configure autoscaling rules
- Learn the exam-critical facts about App Service tiers and features

Everything you practised in this lab — creating resource groups, running CLI commands, navigating the portal, cleaning up resources — you will use in **every single module lab from here on**.

> 🔗 **MODULE LINK**
>
> Lab Zero connects to every module in the course. The skills you practised here (CLI commands, resource groups, portal navigation, cleanup) are the foundation for all 13 module labs.

---

## What We Covered ✅

- [ ] How to sign into the Azure portal and navigate the home page
- [ ] How to open and use Cloud Shell (Bash) in the browser
- [ ] How to install and verify the Azure CLI on your local machine
- [ ] How to log in to Azure from your local terminal with `az login`
- [ ] How to list subscriptions with `az account list --output table`
- [ ] How to create a resource group via the portal (graphical method)
- [ ] How to create a resource group via the CLI (command-line method)
- [ ] How to create an App Service Plan on the free F1 tier
- [ ] How to create and deploy a web app using the CLI
- [ ] How to retrieve a web app URL with `az webapp show --query`
- [ ] How to add application settings with `az webapp config appsettings set`
- [ ] How to view application settings in the portal
- [ ] How to delete individual resources (web app, plan) via CLI
- [ ] How to delete resource groups via CLI (the recommended cleanup method)
- [ ] How to verify cleanup with `az group list --output table`
- [ ] How to set up a spending budget alert in Cost Management
- [ ] The troubleshooting process for 5 common Azure CLI errors
