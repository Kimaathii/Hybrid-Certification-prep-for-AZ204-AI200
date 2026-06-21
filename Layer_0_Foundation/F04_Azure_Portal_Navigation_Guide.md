# F04 — Azure Portal Navigation Guide

> **Foundation Document 4 of 8** | **Pages:** 12–15 (screenshot-heavy) | **Study Time:** 1 hour
> **Purpose:** Know the Azure Portal inside-out so you never waste time looking for things during labs.
> **Prerequisites:** F01 (Cloud Concepts), F02 (Azure Hierarchy & ARM), F03 (Developer Basics)

---

## How to Use This Guide

This guide is designed to be your **visual map** of the Azure Portal. Every section contains **[SCREENSHOT]** placeholders describing exactly what you should see on screen, followed by detailed explanations. As you read, **open [portal.azure.com](https://portal.azure.com)** in your browser and follow along — this guide becomes ten times more useful when you're clicking through in real time.

> 📝 **NOTE:** The Azure Portal interface updates frequently. Button positions and colours may shift slightly, but the core layout — top bar, left sidebar, main content area — remains consistent. If something looks slightly different from what's described, don't panic — the functionality is the same.

---

## Section 1: Portal at a Glance

### The Problem

Imagine walking into a massive office building on your first day at a new job. There are hundreds of rooms, dozens of floors, and nobody gave you a map. You'd waste hours just finding the cafeteria. The Azure Portal is that office building — it contains **every Azure service** Microsoft offers (hundreds of them), and without knowing the layout, you'll spend more time navigating than actually building things.

This section gives you the map.

### Your First Look at the Azure Portal

**[SCREENSHOT: Full portal.azure.com home page — freshly signed in, showing the default dashboard view. The entire browser window should be visible, with annotations pointing to three major regions: (1) the top navigation bar highlighted with a red box, (2) the left sidebar/hamburger menu highlighted with a blue box, and (3) the main content area highlighted with a green box.]**

When you sign in to **portal.azure.com**, you see a page divided into **three major areas**. Think of them like this:

| Area | Real-World Analogy | What It Contains |
|---|---|---|
| **Top Navigation Bar** | The reception desk at a building entrance | Search bar, Cloud Shell button, notifications bell, settings gear, your account profile |
| **Left Sidebar** | The building directory (floor-by-floor listing) | Quick links to favourite services, "Create a resource" button, All Services |
| **Main Content Area** | The actual office rooms you work in | Dashboard tiles, resource details, configuration forms — whatever you're currently working on |

### The Top Navigation Bar — Your Command Centre

**[SCREENSHOT: Close-up of the portal top bar, zoomed in. Annotate each element from left to right: (1) the hamburger menu icon (three horizontal lines), (2) "Microsoft Azure" text/logo, (3) the search bar in the centre, (4) the Cloud Shell icon (looks like >_ ), (5) the settings gear icon, (6) the notifications bell icon, (7) the help question mark icon, (8) the user account circle on the far right.]**

Let's walk through each element from left to right:

| Icon / Element | What It Does | How Often You'll Use It |
|---|---|---|
| **☰ Hamburger Menu** | Expands or collapses the left sidebar | Occasionally — most people leave it expanded |
| **"Microsoft Azure" Logo** | Clicks back to the portal home page from anywhere | Frequently — your "home" button |
| **🔍 Search Bar** | Searches for ANY Azure service, resource, or documentation | **Constantly** — this is the fastest way to find anything |
| **>_ Cloud Shell** | Opens a browser-based terminal (Bash or PowerShell) | Every lab — your in-browser command line |
| **⚙️ Settings** | Portal appearance, language, default directory, timeout settings | Once during setup |
| **🔔 Notifications Bell** | Shows deployment status, errors, and alerts | After every resource creation |
| **❓ Help** | Documentation links, support tickets, "What's New" | When stuck |
| **👤 Account** | Switch directories/tenants, sign out, view permissions | Occasionally |

💡 **KEY CONCEPT:** The **Search Bar** is the single most important element on this page. You will use it hundreds of times throughout this course. Learning to type a service name and press Enter is faster than clicking through any menu.

### The Left Sidebar — Your Quick Access Panel

**[SCREENSHOT: The left sidebar expanded, showing the default items: "+ Create a resource" button at the top, then "Home", "Dashboard", "All services", and below that any favourited/starred services like "Resource groups", "App Services", "Storage accounts", etc. Annotate the "+ Create a resource" button and the star/favourite icons next to each service.]**

The left sidebar has two key sections:

1. **Fixed Items at the Top:**
   - **+ Create a resource** — The starting point for building anything in Azure
   - **Home** — Returns to the portal home page
   - **Dashboard** — Your customisable overview page
   - **All services** — The complete catalogue of every Azure service

2. **Favourited Services Below:**
   - Services you've "starred" appear here as quick-access links
   - By default, you'll see common ones like Resource Groups, App Services, and Storage Accounts
   - You can add or remove favourites (we'll cover this in Section 2)

### The Main Content Area — Your Workspace

**[SCREENSHOT: The default dashboard view in the main content area, showing tiles like "Azure services" quick links (Virtual machines, Storage accounts, App Services, etc.), "Recent resources" list, "Navigate" section with links to subscriptions and resource groups, and any "Getting started" cards.]**

The main content area changes based on what you're doing:

- **On the Home page:** You see quick links, recent resources, and getting-started guides
- **Inside a resource:** You see that resource's configuration, monitoring, and settings
- **Creating a resource:** You see the creation form (wizard)

This area is where you'll spend 90% of your time.

### Customising Your Dashboard

The dashboard is like your office desk — you arrange it to show what matters most to you.

**[SCREENSHOT: The dashboard page with the "Edit" button visible in the top-left of the main content area (pencil icon or "Edit" text). Show the dashboard in edit mode with the tile gallery panel open on the left, showing available tiles like "Resource groups", "Clock", "Markdown", "Metrics chart", and drag handles visible on existing tiles.]**

**To customise your dashboard:**

1. Navigate to **Dashboard** from the left sidebar
2. Click **Edit** (pencil icon) at the top of the dashboard
3. The **Tile Gallery** opens on the left — drag tiles onto your dashboard
4. **Resize tiles** by dragging their corners
5. **Remove tiles** by hovering and clicking the "..." menu → Remove
6. Click **Save** when finished
7. Click **+ New Dashboard** to create multiple dashboards for different purposes

**Recommended dashboard for AZ-204 students:**

| Tile | Why |
|---|---|
| **Resource Groups** | See all your lab resource groups at a glance |
| **Cost Management** | Monitor spending in real-time |
| **Recent Resources** | Quick access to whatever you're currently working on |
| **Service Health** | Know if Azure itself is having issues (so you don't blame your code) |

> 📝 **NOTE:** You can create multiple dashboards. Consider one for "Labs in Progress" and one for "Cost Monitoring." Switch between them using the dropdown at the top of the dashboard page.

---

## Section 2: Finding Any Service Instantly

### The Problem

Azure has **over 200 services**. Even experienced developers don't memorise where each one lives in the menus. You need a way to find any service in under 3 seconds — because during labs, you'll be jumping between services constantly.

### Method 1: The Search Bar (Fastest — Use This)

**[SCREENSHOT: The search bar at the top of the portal, with "App Service" typed into it. The dropdown results should show: (1) a "Services" section with "App Services" as the top result, (2) a "Resources" section showing any existing App Service resources, and (3) a "Documentation" section with links to App Service docs. Annotate each section of the dropdown.]**

The search bar is a **universal finder**. It searches across three categories simultaneously:

| Search Category | What It Shows | Example |
|---|---|---|
| **Services** | Azure service pages (the "home page" for that service type) | Searching "Key Vault" shows the Key Vault service |
| **Resources** | Specific resources you've already created | Searching "my-webapp" shows your existing web app |
| **Documentation** | Microsoft Learn docs and guides | Shows relevant help articles |

**How to use it:**
1. Click the search bar (or press **G + /** as a keyboard shortcut)
2. Start typing the service name — results appear instantly
3. Click the result you want

💡 **KEY CONCEPT:** You don't need to type the full name. Typing "func" will find "Function App." Typing "cosmos" will find "Azure Cosmos DB." The search is fuzzy and forgiving.

🚨 **EXAM ALERT:** The exam won't test you on portal navigation directly, but knowing WHERE to find settings (e.g., "Where do you configure deployment slots?") helps you answer scenario questions about which blade or section contains a specific setting.

### Method 2: All Services (Browse the Full Catalogue)

**[SCREENSHOT: The "All services" page showing the categorised list of every Azure service. Show the category filter on the left (Compute, Networking, Storage, Databases, etc.) and the alphabetical service list on the right. Annotate the search/filter bar at the top of the All Services page and the category tabs.]**

If you want to **browse** rather than search:

1. Click **All services** in the left sidebar
2. Browse by **category** (Compute, Storage, Networking, Databases, etc.)
3. Or use the **filter bar** at the top of the All Services page to narrow results
4. Each service shows a brief description when you hover over it

This is useful when you're exploring — "What storage services does Azure offer?" — rather than looking for something specific.

### Method 3: Favouriting/Starring Services

**[SCREENSHOT: The "All services" page with the star icon visible next to each service name. Show one service (e.g., "Function App") with the star outlined (not favourited) and another service (e.g., "Resource groups") with the star filled/solid (already favourited). An arrow should point from a filled star to the left sidebar, showing that service appearing in the sidebar.]**

Services you use frequently should be **favourited** so they appear in your left sidebar:

**To add a favourite:**
1. Go to **All services**
2. Find the service you want
3. Click the **star icon** ☆ next to its name — it turns solid ★
4. The service now appears in your left sidebar for one-click access

**To remove a favourite:**
1. Right-click the service in the left sidebar
2. Select **Remove from favorites**

**Recommended favourites for AZ-204 students:**

| Service | Why You'll Need It |
|---|---|
| **Resource groups** | Every lab starts and ends here |
| **App Services** | Module 1 — Web Apps |
| **Function App** | Module 2 — Azure Functions |
| **Storage accounts** | Module 4 + used by Functions internally |
| **Azure Cosmos DB** | Module 5 — NoSQL database |
| **Key vaults** | Module 7 — Secrets management |
| **API Management services** | Module 8 — API gateway |
| **Cost Management + Billing** | Always — monitor your spending |

✅ **CHECKPOINT:** Open the Azure Portal right now. Can you identify the three main areas (top bar, left sidebar, main content)? Can you find "Function App" using the search bar? Try starring it as a favourite and confirming it appears in your left sidebar.

---

## Section 3: Resource Groups as Your Daily Work Area

### The Problem

Imagine you're cooking five different meals at the same time. If you throw all the ingredients into one giant pile, you'll never find anything and you'll accidentally put pasta sauce in the cake. You need separate bowls and trays for each meal.

**Resource Groups** are those bowls and trays. Every Azure resource (web app, database, storage account, etc.) must live inside a Resource Group. When you're done with a project, you delete the Resource Group and everything inside it disappears — no orphaned resources, no surprise bills.

> 🔗 **MODULE LINK:** You first learned about Resource Groups in F01 and F02. Here we focus on the **portal experience** — how to create, browse, and manage them visually.

### Creating a Resource Group

**[SCREENSHOT: The "Create a resource group" form showing the fields: (1) Subscription dropdown (with a subscription selected), (2) Resource group name text field (showing "az204-module01-rg" as an example), (3) Region dropdown (showing "(US) East US" selected). The "Review + create" button should be visible at the bottom. Annotate each field.]**

**Steps to create a Resource Group:**

1. Click **+ Create a resource** in the left sidebar (or search for "Resource group")
2. Click **Resource group** from the results
3. Click **Create**
4. Fill in the form:

| Field | What to Enter | Why |
|---|---|---|
| **Subscription** | Select your subscription | Determines who pays — you'll usually have only one |
| **Resource group name** | A descriptive name, e.g., `az204-module01-rg` | Naming convention: `{course}-{module}-rg` |
| **Region** | Choose a region close to you, e.g., `East US` | Metadata only — resources inside can be in any region |

5. Click **Review + create**
6. Click **Create**

> 📝 **NOTE:** The Resource Group's region is just where the **metadata** (the group's information) is stored. Resources inside the group can be in *any* Azure region. However, it's good practice to keep your resources in the same region as the group for simplicity.

### Browsing a Resource Group

**[SCREENSHOT: Inside a resource group (e.g., "az204-module01-rg"), showing the Overview page. The left blade menu should be visible with sections like "Overview", "Activity log", "Access control (IAM)", "Tags", "Events", "Settings" heading with "Deployments", "Policies", "Properties", "Locks". The main area should show a list of resources (e.g., an App Service Plan, a Web App, a Storage Account) with columns: Name, Type, Location, Status.]**

When you click into a Resource Group, you see:

- **Overview** — A list of every resource in this group, with type, location, and status
- **A left-side blade menu** — Links to Activity Log, Deployments, Access Control, Tags, and more

This is your "project folder" view. Everything related to a specific lab or project is here in one place.

### The Activity Log — What Happened and When

**[SCREENSHOT: The Activity Log tab within a resource group, showing a list of operations with columns: Operation name (e.g., "Create or Update Web App", "Delete Storage Account"), Status (Succeeded/Failed), Time, Initiated by. Show the filter bar at the top with options for Timespan, Subscription, Event severity, and Resource type.]**

The Activity Log is like a **security camera recording** for your Azure resources. It shows:

- **What** operation was performed (Create, Update, Delete)
- **Who** performed it (your email or a service principal)
- **When** it happened (timestamp)
- **Whether it succeeded or failed** (with error details if failed)

**When you'll use this:**
- A lab step fails and you need to know *what went wrong*
- You want to confirm a deployment actually completed
- You need to troubleshoot "I'm sure I created that resource — where did it go?"

### The Deployments Tab — Your Deployment History

**[SCREENSHOT: The "Deployments" section under Settings in the resource group blade. Show a list of deployments with columns: Deployment name, Status (Succeeded/Failed), Timestamp, Duration. One deployment should be expanded or clicked to show deployment details including the template used and outputs.]**

Every time you create resources — whether through the portal, CLI, or an ARM template — Azure records it as a **deployment**. The Deployments tab shows:

- Every deployment that targeted this Resource Group
- Whether each deployment succeeded or failed
- The duration of each deployment
- Click any deployment to see the full details, including the template used

💡 **KEY CONCEPT:** When a CLI command like `az webapp create` seems to succeed but nothing appears, check the **Deployments** tab. If the deployment is listed as "Failed," click it to see the error message. This is your first debugging step.

### Deleting a Resource Group (With Everything Inside)

**[SCREENSHOT: The "Delete resource group" confirmation dialog. Show the resource group overview page with the "Delete resource group" button highlighted in the top toolbar. Then show the confirmation popup that says "Are you sure you want to delete resource group 'az204-module01-rg'?" with a text field requiring you to type the resource group name to confirm, and the "Delete" button greyed out until the name is typed.]**

**This is the most important cleanup operation you'll perform.** After every lab:

1. Navigate to the Resource Group
2. Click **Delete resource group** in the top toolbar
3. A confirmation dialog appears — **you must type the resource group name** to confirm
4. Click **Delete**
5. Wait for the notification confirming deletion (check the 🔔 bell)

🚨 **EXAM ALERT:** Deleting a Resource Group deletes **ALL resources inside it** — web apps, databases, storage accounts, everything. There is no undo. This is by design — it's the clean-up mechanism. Always double-check the resource group name before typing it in the confirmation box.

💰 **COST WARNING:** Every lab in this course ends with "delete the Resource Group." If you skip this step, resources continue running and **you continue being charged**. Make deleting your Resource Group a habit — the very last thing you do after every lab.

---

## Section 4: Creating Resources — The Universal Pattern

### The Problem

You're going to create dozens of Azure resources throughout this course — web apps, function apps, storage accounts, databases, key vaults. Each one has a slightly different creation form. But here's the good news: **every single resource follows the same creation pattern.** Learn this pattern once, and you can create anything.

### The Universal 6-Step Pattern

Think of this like ordering at a restaurant. Every restaurant has slightly different menus, but the process is always: **sit down → read menu → choose items → waiter confirms order → food arrives → enjoy.**

In Azure, the process is always:

```
Step 1: Click "+ Create a resource"
Step 2: Search for the service you want
Step 3: Click "Create"
Step 4: Fill in the creation form (Basics, Networking, Monitoring, Tags, etc.)
Step 5: Click "Review + Create" — Azure validates your settings
Step 6: Click "Create" — Azure deploys the resource
```

Let's walk through each step visually.

### Step 1: Start the Creation

**[SCREENSHOT: The left sidebar with the "+ Create a resource" button highlighted. Alternatively, show the portal home page with the "+ Create a resource" button visible in the top-left area.]**

Click **+ Create a resource** — this is always the starting point. You can find this button in:
- The left sidebar (always visible)
- The portal home page (prominent button)

### Step 2: Search for the Service

**[SCREENSHOT: The "Create a resource" marketplace page showing the search bar at the top with "Web App" typed in. Below, search results display showing "Web App" by Microsoft with a "Create" button. Also visible: popular categories like "AI + Machine Learning", "Compute", "Databases", etc.]**

The Azure Marketplace opens. You have two options:
- **Search** by typing the service name (recommended)
- **Browse** by category (Compute, Databases, etc.)

Type the name and press Enter. Click the service you want.

### Step 3: Click "Create"

**[SCREENSHOT: The service page for "Web App" (or any service) showing the description, pricing information, and a prominent blue "Create" button.]**

Each service has an overview page. Click **Create** to open the creation form.

### Step 4: Fill in the Creation Form

**[SCREENSHOT: The "Create Web App" form showing the tabbed interface: Basics | Runtime | Deployment | Networking | Monitoring | Tags | Review + Create. The "Basics" tab should be active, showing fields like Subscription, Resource Group, Name, Publish (Code/Docker), Runtime stack, Region, App Service Plan. Annotate the tabs at the top and the key fields.]**

Every creation form has **tabs across the top**:

| Tab | What You Configure | Required? |
|---|---|---|
| **Basics** | Name, Resource Group, Region, Pricing tier | ✅ Always |
| **Networking** | VNet integration, private endpoints | Sometimes |
| **Monitoring** | Application Insights, diagnostics | Usually optional |
| **Tags** | Key-value metadata (e.g., Environment=dev) | Optional but recommended |
| **Review + Create** | Summary of all settings + validation | ✅ Always |

> 📝 **NOTE:** You don't have to fill in every tab. Most labs only require the **Basics** tab. The other tabs have sensible defaults. When a lab requires a non-default setting, the lab guide will tell you exactly which tab and field to change.

### Step 5: Review + Create (Validation)

**[SCREENSHOT: The "Review + Create" tab showing a green checkmark with "Validation passed" at the top. Below, a summary of all settings chosen: Subscription, Resource Group, Name, Region, SKU, Runtime stack, etc. The "Create" button is visible at the bottom, now clickable (not greyed out).]**

When you click **Review + Create**, Azure runs **validation checks**:

- Is the resource name available? (Names must be globally unique for many services)
- Is the selected region available for this service?
- Does your subscription have quota for this resource?
- Are all required fields filled in?

**If validation passes:** You see a green ✅ "Validation passed" message and the **Create** button becomes clickable.

**[SCREENSHOT: A failed validation example, showing a red "Validation failed" message at the top with specific error details like "The name 'myapp' is already taken" or "The subscription is not registered for the resource type."]**

**If validation fails:** You see a red ❌ message explaining exactly what's wrong. Common failures:

| Error | What It Means | Fix |
|---|---|---|
| "Name already taken" | Someone else is using that globally unique name | Add random characters, e.g., `myapp-742` |
| "Subscription not registered" | Your subscription hasn't enabled this resource provider | Register via CLI: `az provider register --namespace Microsoft.Web` |
| "Quota exceeded" | You've hit a limit for this resource type | Delete unused resources or request a quota increase |
| "Region not available" | This service isn't available in your chosen region | Choose a different region (East US and West Europe are safest) |

### Step 6: Create and Monitor

**[SCREENSHOT: The deployment page showing "Your deployment is underway" with a progress animation. Show fields: Deployment name, Subscription, Resource group, and a status indicator. Below, a "Go to resource" button (greyed out while deploying, blue when complete).]**

After clicking **Create**:
1. Azure begins deploying the resource
2. You see a **"Deployment in progress"** page
3. Deployment time varies: seconds (Resource Groups) to minutes (Cosmos DB, App Service)
4. When complete, you see **"Your deployment is complete"**

### The Notifications Bell — Your Deployment Tracker

**[SCREENSHOT: The notifications bell icon in the top bar, clicked/expanded to show a dropdown list of recent notifications. Show items like: "Deployment succeeded — mywebapp" (with a green checkmark), "Deployment in progress — mycosmosdb" (with a spinning indicator), and possibly a "Deployment failed — myfunctionapp" (with a red X). Annotate the "Go to resource" link within a successful notification.]**

The **🔔 notifications bell** in the top bar is your real-time status tracker:

- **Blue spinner** = deployment in progress
- **Green checkmark** = deployment succeeded → click **"Go to resource"** to jump directly to it
- **Red X** = deployment failed → click to see error details

💡 **KEY CONCEPT:** After clicking "Create," you don't need to stay on the deployment page. You can navigate elsewhere and check the **🔔 bell** later. The notification will be waiting for you with a "Go to resource" link.

✅ **CHECKPOINT:** Here's a drill. Without looking back, can you recite the 6-step universal creation pattern? (1) Click + Create a resource → (2) Search for service → (3) Click Create → (4) Fill in form → (5) Review + Create → (6) Create. If you can remember this, you can create ANY Azure resource.

---

## Section 5: Cloud Shell — The Browser Terminal

### The Problem

In F03, you installed the Azure CLI on your local machine. But what if you're on a school computer, a friend's laptop, or a tablet? You can't install software everywhere. Azure solves this by giving you a **full terminal running inside your browser** — no installation needed.

Think of Cloud Shell as a **virtual office** you can access from any computer with a web browser. All your tools (Azure CLI, PowerShell, Git, Python, Node.js, .NET SDK, `vim`, `nano`) are pre-installed and always up-to-date.

### Opening Cloud Shell

**[SCREENSHOT: The top navigation bar with an arrow pointing to the Cloud Shell icon (>_). Then show the Cloud Shell panel that opens at the bottom half of the portal, taking up the lower portion of the screen. The panel should show a terminal prompt with a blinking cursor.]**

1. Click the **>_** icon in the top navigation bar
2. Cloud Shell opens as a **panel at the bottom of your portal window**
3. You now have a fully functional terminal — right inside your browser

### First-Time Setup

**[SCREENSHOT: The Cloud Shell first-time setup dialog. Show the prompt asking you to choose between "Bash" and "PowerShell", with Bash highlighted/selected. Below that, show the message about creating a storage account: "You have no storage mounted. We will create a storage account for you." with a "Create storage" button.]**

The first time you open Cloud Shell, Azure asks you two things:

**1. Choose Your Shell: Bash or PowerShell**

| Shell | Best For | Recommendation |
|---|---|---|
| **Bash** | Linux-style commands, most Azure CLI examples online | ✅ **Choose this** for AZ-204 |
| **PowerShell** | Windows-style commands, PowerShell modules | Also works, but most tutorials use Bash |

> 📝 **NOTE:** You can switch between Bash and PowerShell at any time using the dropdown in the top-left of the Cloud Shell panel. You're not locked into your initial choice.

**2. Storage Account Creation**

Cloud Shell needs a small **storage account** to persist your files between sessions (scripts, configuration files, etc.). Azure offers to create one for you automatically.

- Click **Create storage** to let Azure create one in a new Resource Group called `cloud-shell-storage-*`
- This storage account is tiny and costs **pennies per month** (usually under $0.05/month)
- It stores your Cloud Shell home directory (`~/`) so your files survive between sessions

💰 **COST WARNING:** The Cloud Shell storage account is small and cheap, but it does exist. If you want to be thorough about cleanup at the end of the course, you can delete the `cloud-shell-storage-*` resource group. Note: this wipes your Cloud Shell files and the next time you open Cloud Shell, it will ask you to set up storage again.

### You're Already Logged In

**[SCREENSHOT: Cloud Shell with the command `az account show --output table` typed and executed, showing the output with columns: EnvironmentName (AzureCloud), HomeTenantId, IsDefault (True), Name (subscription name), State (Enabled), TenantId. Annotate the output showing the logged-in user's subscription details.]**

Here's the magic: **Cloud Shell is already authenticated.** You don't need to run `az login` — you're already signed in as the same user who's logged into the portal.

```bash
# Verify you're logged in — this works immediately in Cloud Shell
az account show --output table
# ↑ Shows your subscription name, tenant ID, and state
```

Compare this to your local terminal, where you have to:
1. Install Azure CLI
2. Run `az login`
3. Open a browser window
4. Sign in
5. Select a subscription

In Cloud Shell, all of that is done for you automatically.

💡 **KEY CONCEPT:** Cloud Shell is pre-authenticated. Any `az` command you run uses your portal login automatically. This makes it perfect for quick tasks — no setup, no login, just type and go.

### Uploading and Downloading Files

**[SCREENSHOT: The Cloud Shell toolbar showing the upload/download buttons. Show the upload dialog that appears when clicking the upload button — a file picker dialog. Also show the file system navigation with `ls` showing the home directory contents, including any uploaded files.]**

You can transfer files between your local computer and Cloud Shell:

**Upload a file:**
1. Click the **Upload/Download** button in the Cloud Shell toolbar (looks like a page with an up arrow)
2. Select **Upload** 
3. Choose a file from your computer
4. The file appears in your Cloud Shell home directory (`~/`)

**Download a file:**
1. Click the **Upload/Download** button
2. Select **Download**
3. Type the file path (e.g., `output.json`)
4. The file downloads to your local machine

**Alternatively, use the built-in file editor:**

```bash
# List files in your Cloud Shell home directory
ls ~/

# Open the built-in code editor (Monaco-based, like VS Code)
code myfile.json
# ↑ Opens a visual editor right inside Cloud Shell
```

### Cloud Shell vs Local Terminal — When to Use Which

| Situation | Use Cloud Shell | Use Local Terminal |
|---|---|---|
| Quick one-off command | ✅ | |
| On someone else's computer | ✅ | |
| Lab that requires local file editing | | ✅ |
| Building and running .NET projects | | ✅ (faster) |
| Checking resource status quickly | ✅ | |
| Long-running scripts | | ✅ (Cloud Shell times out after 20 min idle) |

> 📝 **NOTE:** Cloud Shell has an **idle timeout of 20 minutes**. If you don't type anything for 20 minutes, the session disconnects. Your files are saved (in the storage account), but any running processes are terminated. For long-running tasks, use your local terminal.

✅ **CHECKPOINT:** Open Cloud Shell in the portal right now. Run `az group list --output table`. Do you see any resource groups? If you created one in a previous lab, it should appear. If the list is empty, that's fine — it means you've cleaned up properly!

---

## Section 6: Key Portal Sections for Every Module

### The Problem

Throughout this course, you'll work with many different Azure services. Each service has its own section in the portal with its own layout. If you know the **navigation path** — how to get from the portal home page to the exact setting you need — you'll save enormous amounts of time during labs.

This section gives you a **pre-flight briefing** for every major service you'll encounter.

### App Service (Module 1)

**[SCREENSHOT: Inside an App Service resource, showing the left blade menu with these sections highlighted: (1) Overview (showing URL, Status, App Service Plan, Runtime), (2) Under "Deployment" section: Deployment Center, Deployment slots, (3) Under "Settings" section: Configuration (Application settings, Connection strings), Environment variables, (4) Under "Scale" section: Scale up (App Service plan), Scale out (manual/auto). Annotate each section with arrows.]**

**Navigation Path:** Search bar → "App Services" → Click your app name

Key areas you'll visit:

| Section | Where to Find It | What You Do There |
|---|---|---|
| **Overview** | Main page when you click the app | See URL, status, resource group, restart/stop the app |
| **Configuration** | Settings → Configuration | Add Application Settings (environment variables), Connection Strings |
| **Deployment slots** | Deployment → Deployment slots | Create staging slots, swap to production |
| **Deployment Center** | Deployment → Deployment Center | Connect to GitHub, configure CI/CD |
| **Scale up** | Scale → Scale up | Change the App Service Plan tier (vertical scaling) |
| **Scale out** | Scale → Scale out | Add/remove instances, configure autoscale rules |
| **Log stream** | Monitoring → Log stream | Watch live application logs |

🚨 **EXAM ALERT:** "Application settings" in App Service are **environment variables** injected into your app at runtime. This is where you put connection strings and secrets (or better yet, Key Vault references). Know this mapping: App Setting in portal = Environment Variable in code.

### Function App (Module 2)

**[SCREENSHOT: Inside a Function App resource, showing the left blade menu with these sections highlighted: (1) Overview, (2) Functions list (showing individual functions like "HttpTrigger1", "QueueProcessor"), (3) Under Settings: Configuration, (4) App keys section. Click into a specific function to show the "Code + Test" view with the function code editor, "Get function URL" button, and Test/Run panel.]**

**Navigation Path:** Search bar → "Function App" → Click your function app name

Key areas you'll visit:

| Section | Where to Find It | What You Do There |
|---|---|---|
| **Functions** | Left blade → Functions | See list of all functions in this app |
| **Code + Test** | Click a function → Code + Test | View/edit code, test with built-in HTTP tester |
| **Function Keys** | Click a function → Function Keys | Get the `code=` parameter for authenticated calls |
| **App keys** | Left blade → App keys | Get host-level keys (master key — admin access) |
| **Configuration** | Settings → Configuration | App Settings, Connection Strings — same as App Service |
| **Monitor** | Click a function → Monitor | See invocation logs, success/failure counts |

💡 **KEY CONCEPT:** To get a Function's URL for testing, click the function → **Get function URL** → copy the URL (which includes the `?code=` key). This is the URL you'll use in `curl` commands during labs.

### Key Vault (Module 7)

**[SCREENSHOT: Inside a Key Vault resource, showing the left blade menu with these sections highlighted: (1) Overview, (2) Under "Objects" section: Keys, Secrets, Certificates, (3) Under "Settings" section: Access configuration, Networking. Show the Secrets list view with a few secret names like "DatabasePassword", "ApiKey" visible (values hidden). Annotate the "Generate/Import" button at the top of the Secrets list.]**

**Navigation Path:** Search bar → "Key vaults" → Click your vault name

Key areas you'll visit:

| Section | Where to Find It | What You Do There |
|---|---|---|
| **Secrets** | Objects → Secrets | Store, view, and manage secret values |
| **Keys** | Objects → Keys | Cryptographic keys for encryption |
| **Certificates** | Objects → Certificates | SSL/TLS certificates |
| **Access configuration** | Settings → Access configuration | Choose between Vault Access Policy and Azure RBAC |
| **Networking** | Settings → Networking | Firewall rules, Private Endpoints, allowed IPs |
| **Access control (IAM)** | Left blade → Access control (IAM) | Assign RBAC roles (Key Vault Secrets User, etc.) |

🚨 **EXAM ALERT:** Key Vault has **two layers of access control**: (1) **Network access** — configured under Networking (firewall/VNet rules), and (2) **Permission access** — configured either as Vault Access Policies or Azure RBAC. Both must allow access for a request to succeed. A common exam scenario: "User has correct RBAC role but cannot read secrets" — answer: network firewall is blocking them.

### Storage Account (Module 4)

**[SCREENSHOT: Inside a Storage Account resource, showing the left blade menu with these sections highlighted: (1) Overview, (2) Under "Data storage" section: Containers, File shares, Queues, Tables, (3) Under "Security + networking" section: Access keys, Shared access signature, Networking, Encryption. Show the Containers view with a container named "images" and inside it some blob files listed.]**

**Navigation Path:** Search bar → "Storage accounts" → Click your account name

Key areas you'll visit:

| Section | Where to Find It | What You Do There |
|---|---|---|
| **Containers** | Data storage → Containers | Create/browse blob containers, upload/download files |
| **Access keys** | Security + networking → Access keys | Find connection strings and account keys |
| **Shared access signature** | Security + networking → Shared access signature | Generate SAS tokens for limited-time access |
| **Networking** | Security + networking → Networking | Firewall rules, VNet access |
| **Lifecycle management** | Data management → Lifecycle management | Create rules to automatically move blobs between tiers |
| **Static website** | Data management → Static website | Enable web hosting from the `$web` container |

### Cosmos DB (Module 5)

**[SCREENSHOT: Inside a Cosmos DB account, showing the left blade menu with these sections highlighted: (1) Overview (showing the account URI and global map), (2) Data Explorer (showing databases and containers in a tree view, with a query editor panel), (3) Keys (showing URI, Primary Key, Connection String). Show the Data Explorer with a database expanded → a container expanded → Items showing JSON documents.]**

**Navigation Path:** Search bar → "Azure Cosmos DB" → Click your account name

Key areas you'll visit:

| Section | Where to Find It | What You Do There |
|---|---|---|
| **Data Explorer** | Left blade → Data Explorer | Browse databases, containers, run SQL queries, view/edit items |
| **Keys** | Settings → Keys | Get the URI and Primary Key for SDK connection |
| **Replicate data globally** | Settings → Replicate data globally | Add/remove regions for global distribution |
| **Default consistency** | Settings → Default consistency | Set consistency level (Strong, Bounded Staleness, Session, Consistent Prefix, Eventual) |
| **Scale & Settings** | Inside a container → Scale & Settings | Configure RU throughput |

💡 **KEY CONCEPT:** The **Data Explorer** is your best friend for verifying lab work. After your code writes data to Cosmos DB, you can open Data Explorer, navigate to your container, and run a SQL query like `SELECT * FROM c` to see if the data is actually there.

### API Management (Module 8)

**[SCREENSHOT: Inside an API Management resource, showing the left blade menu with these sections highlighted: (1) Overview (showing Gateway URL, Developer portal URL), (2) Under "APIs" section: APIs list showing imported APIs, (3) Under "Policies" section: showing the XML policy editor. Show the APIs view with a sample API listed and the "Add API" button visible.]**

**Navigation Path:** Search bar → "API Management services" → Click your APIM instance

Key areas you'll visit:

| Section | Where to Find It | What You Do There |
|---|---|---|
| **APIs** | Left blade → APIs | Import, define, and manage API definitions |
| **Products** | Left blade → Products | Group APIs and manage access (Starter, Unlimited) |
| **Policies** | Click an API → Design → Inbound/Outbound Processing | Apply XML policies for rate limiting, caching, transformation |
| **Subscriptions** | Left blade → Subscriptions | Manage API keys for consumers |
| **Developer portal** | Overview → Developer portal URL | The self-service portal API consumers see |

> 📝 **NOTE:** API Management has a **lengthy deployment time** — up to 30–45 minutes for the Developer tier. When you create an APIM instance in a lab, start the creation first, then work on other tasks while it deploys.

✅ **CHECKPOINT:** For each service above, can you describe the **navigation path** from the portal home page? Practice: "To find my App Service's deployment slots, I would search for 'App Services', click my app, then look under 'Deployment' → 'Deployment slots'." Try describing the path to a Key Vault's secrets and a Storage Account's access keys.

---

## Section 7: Cost Management and Billing

### The Problem

Cloud services bill by the hour (or by the second, or by the request). Unlike buying a physical server once, Azure is a **running meter** — and if you forget to turn off resources, the meter keeps running. Students have been surprised by bills of $50, $100, or more because they left lab resources running over a weekend.

This section ensures you always know what you're spending and can set alerts before costs get out of hand.

> 🔗 **MODULE LINK:** You learned about Azure's billing models in F01 (consumption, reserved, per-transaction). Here we focus on the **portal tools** for monitoring and controlling costs.

### Finding Cost Management

**[SCREENSHOT: The search bar with "Cost Management" typed, showing "Cost Management + Billing" in the results. Then show the Cost Management overview page with the left blade showing: Overview, Cost analysis, Budgets, Cost alerts, Advisor recommendations.]**

**Navigation Path:** Search bar → "Cost Management" → Click "Cost Management + Billing"

You can also access cost information per resource group:
**Navigation Path:** Resource Groups → Click a resource group → Left blade → "Cost analysis"

### Cost Analysis — See What You're Spending

**[SCREENSHOT: The Cost Analysis page showing a bar chart of daily costs over the past 30 days. Show the filter bar at the top with filters set to: (1) Scope: a specific subscription, (2) Resource group: "az204-module01-rg" selected. The chart should show cost breakdown by service (e.g., App Service, Storage, Cosmos DB) in different colours. Below the chart, show a table listing each resource with its cost. Annotate the filter dropdowns and the chart.]**

Cost Analysis gives you a **visual breakdown** of your spending. Here's how to use it effectively:

**Step 1: Open Cost Analysis**
- Search for "Cost Management" → Click "Cost analysis" in the left blade

**Step 2: Filter by Resource Group**
- Click the **Scope** dropdown → Select your subscription
- Click **Add filter** → Choose "Resource group" → Select your lab's resource group
- This isolates costs to just one lab's resources

**Step 3: Read the Charts**

| View | What It Shows | Best For |
|---|---|---|
| **Accumulated costs** | Running total over time | "How much have I spent this month?" |
| **Daily costs** | Cost per day as a bar chart | "Which day had the spike?" |
| **Cost by service** | Breakdown by Azure service | "Is it Cosmos DB or App Service costing me?" |
| **Cost by resource** | Breakdown by specific resource | "Which specific resource is expensive?" |

**Step 4: Change the Date Range**
- Use the date picker at the top to narrow the range
- For lab work, set it to "Last 7 days" to see recent activity

### Setting Up a Budget Alert — Step by Step

A budget alert is like a **smoke detector for your wallet**. It doesn't stop the spending (just like a smoke detector doesn't put out the fire), but it warns you before things get out of control.

**[SCREENSHOT: The "Budgets" page within Cost Management, showing the "+ Add" button to create a new budget. Then show the budget creation form with fields: (1) Name: "az204-monthly-budget", (2) Reset period: Monthly, (3) Amount: $10.00. Show the "Alert conditions" section with a threshold set to 80% of budget ($8.00) and an email recipient field.]**

**Step-by-step budget creation:**

1. Navigate to **Cost Management** → **Budgets** in the left blade
2. Click **+ Add** to create a new budget
3. Fill in the budget form:

| Field | What to Enter | Explanation |
|---|---|---|
| **Name** | `az204-monthly-budget` | A descriptive name for this budget |
| **Reset period** | Monthly | Budget resets at the start of each month |
| **Creation date** | (auto-filled with today) | When this budget starts tracking |
| **Expiration date** | Set to a few months from now | When this budget stops tracking |
| **Budget amount** | `10` (or your comfort level) | The dollar amount you don't want to exceed |

4. Click **Next** to set alert conditions:

| Field | What to Enter | Explanation |
|---|---|---|
| **Alert condition type** | Actual | Alerts based on real spending (not forecasted) |
| **% of budget** | `80` | Alert when you've used 80% of your budget |
| **Action group recipients** | Your email address | Where the alert email goes |

5. Click **Create**

💰 **COST WARNING:** A budget alert **does NOT automatically stop your resources**. It only sends you an email. If you get an alert, you must manually go to Azure and delete or stop the expensive resources. Think of it as a warning light on your car's dashboard — the car keeps driving, but you should probably pull over.

> 📝 **NOTE:** For this course, a monthly budget of **$10–$20** is reasonable if you clean up after every lab. If you delete Resource Groups promptly, most labs cost under $0.50. The biggest cost risks are leaving App Service Plans (S1 = ~$2.40/day) or Cosmos DB accounts running overnight.

### Quick Cost-Saving Tips for Students

| Tip | How |
|---|---|
| **Delete Resource Groups immediately after labs** | `az group delete --name myRG --yes --no-wait` |
| **Use Free tiers when available** | App Service F1, Functions Consumption, Cosmos DB free tier |
| **Stop (don't delete) if you need to resume later** | `az webapp stop --name myapp --resource-group myRG` |
| **Check costs daily during active lab weeks** | Cost Management → Cost Analysis → Last 7 days |
| **Set a budget alert before starting any labs** | Follow the steps above — do this NOW |

✅ **CHECKPOINT:** Go to Cost Management right now and check your current spending. Is it $0? If you've been cleaning up labs, it should be close to zero. If there's a surprise charge, investigate which resource is causing it by filtering Cost Analysis by resource group.

---

## Section 8: Microsoft Entra ID

### The Problem

Imagine you're building an app that needs to know who the user is. Or your app needs permission to read secrets from Key Vault. Or an Azure service needs to call another Azure service on your behalf. All of these involve **identity** — proving who (or what) is making a request.

**Microsoft Entra ID** (formerly called **Azure Active Directory** or **Azure AD**) is Azure's identity service. It's the bouncer at the door — it verifies who you are, checks if you're allowed in, and gives you a wristband (token) that proves it.

> 🔗 **MODULE LINK:** You'll dive deep into Entra ID in Module 6 (Implement User Authentication and Authorization) and Module 7 (Secure Cloud Solutions). This section covers just the **portal navigation** — where to find things and what key terms mean.

### Finding Microsoft Entra ID

**[SCREENSHOT: The search bar with "Entra" typed, showing "Microsoft Entra ID" in the results. Then show the Microsoft Entra ID overview page with the left blade menu showing sections: Overview, Users, Groups, App registrations, Enterprise applications, Roles and administrators. The Overview page should show the Tenant name, Tenant ID (a GUID), and Primary domain.]**

**Navigation Path:** Search bar → "Microsoft Entra ID" → Click the result

> 📝 **NOTE:** In older documentation, blog posts, and some exam questions, you may still see the name "Azure Active Directory" or "Azure AD." These are the **same service** — Microsoft renamed it to "Microsoft Entra ID" in 2023. If you see "Azure AD" anywhere, mentally replace it with "Microsoft Entra ID."

### The Tenant ID — Note This for Labs

**[SCREENSHOT: The Microsoft Entra ID Overview page with the Tenant ID highlighted (a GUID like "72f988bf-86f1-41af-91ab-2d7cd011db47"). Show an arrow pointing to the Tenant ID with a callout saying "Copy this — you'll need it in labs."]**

Your **Tenant ID** is a unique identifier (a GUID) for your organisation's Entra ID directory. Think of it as your organisation's postal code — it doesn't change, and you reference it whenever Azure needs to know *which* directory you're working with.

**Where you'll need the Tenant ID:**
- Configuring authentication in App Service (Module 6)
- Setting up app registrations (below)
- Connecting to Key Vault with specific identity configurations
- Any time a lab asks for `--tenant-id` or `AZURE_TENANT_ID`

**How to copy it:**
1. Navigate to Microsoft Entra ID
2. The Tenant ID is right on the Overview page
3. Click the **copy icon** next to it
4. Paste it somewhere safe (a notes file, your lab workspace)

💡 **KEY CONCEPT:** You have three IDs you'll use repeatedly throughout this course. Keep all three written down somewhere accessible:

| ID | What It Is | Where to Find It |
|---|---|---|
| **Tenant ID** | Your directory's unique ID | Entra ID → Overview |
| **Subscription ID** | Your billing account's ID | Subscriptions → Click subscription → Overview |
| **Object ID** | Your user account's unique ID | Entra ID → Users → Click your name |

### App Registrations — Giving Your App an Identity

**[SCREENSHOT: The "App registrations" page within Microsoft Entra ID, showing the "+ New registration" button at the top. Below, show a list of registered applications (if any exist) with columns: Display name, Application (client) ID, Supported account types, Created date. If empty, show the "No app registrations" empty state with a prompt to create one.]**

**Navigation Path:** Microsoft Entra ID → Left blade → App registrations

An **App Registration** is like creating an **employee badge** for your application. Just like people need a badge to access the office building, your application needs an identity to access Azure resources.

When you register an app, Entra ID gives it:

| What It Gets | What It's Called | Analogy |
|---|---|---|
| **A unique ID** | Application (client) ID | Employee badge number |
| **A secret password** | Client Secret (or Certificate) | Badge PIN code |
| **A directory it belongs to** | Directory (tenant) ID | Which office building it works in |

**When you'll create App Registrations:**
- Module 6: Configuring authentication for a web app
- Module 7: Creating a service identity that can access Key Vault
- Any time the exam asks about "registering an application with Entra ID"

**[SCREENSHOT: Inside an App Registration, showing the Overview page with the Application (client) ID and Directory (tenant) ID highlighted. Also show the left blade with sections: Overview, Authentication, Certificates & secrets, API permissions, Expose an API.]**

After registering an app, the key sections are:

| Section | What You Configure |
|---|---|
| **Overview** | Find the Application (client) ID and Directory (tenant) ID |
| **Authentication** | Set redirect URIs (where users go after login), platform configurations |
| **Certificates & secrets** | Create client secrets (passwords) or upload certificates |
| **API permissions** | Grant your app permission to call other APIs (Microsoft Graph, custom APIs) |
| **Expose an API** | Define your own API scopes that other apps can request |

🚨 **EXAM ALERT:** When the exam says "register the application," it means create an App Registration in Entra ID. When it says "grant the application permission to read secrets from Key Vault," it means: (1) register the app in Entra ID, (2) assign the app a RBAC role like "Key Vault Secrets User" on the Key Vault resource. These are two separate steps — registration alone does NOT grant access to anything.

### Enterprise Applications — Apps People Actually Use

**[SCREENSHOT: The "Enterprise applications" page within Microsoft Entra ID, showing a list of applications with columns: Name, Application ID, Object Type, Homepage URL. Show both Microsoft first-party apps and any custom apps. Annotate the difference: "App registrations = apps you build. Enterprise applications = apps people (and services) use."]**

**Navigation Path:** Microsoft Entra ID → Left blade → Enterprise applications

While **App Registrations** are apps you *create*, **Enterprise Applications** are apps that *exist in your directory* — including ones from Microsoft, third-party SaaS apps (Salesforce, Slack), and your own registered apps.

Think of it this way:
- **App Registration** = the blueprint for a badge
- **Enterprise Application** = an actual badge that's been issued and is in use

For AZ-204, you'll mostly work with **App Registrations**. Enterprise Applications become more relevant in administrator roles.

✅ **CHECKPOINT:** Navigate to Microsoft Entra ID right now. Can you find your Tenant ID? Can you navigate to App registrations? You don't need to create one yet — just confirm you know where the page is. Write down your Tenant ID somewhere you can find it easily during labs.

---

## Navigation Speed Drill — 10 Timed Tasks

Now it's time to put everything together. The goal is to navigate the portal confidently and quickly. **Time yourself** on each task — aim to complete all 10 in under 15 minutes. As you get faster, you'll spend less time fumbling through menus during labs and more time actually learning.

### Instructions

1. Open **portal.azure.com** in your browser
2. Start a timer (use your phone or a stopwatch)
3. Complete each task below
4. Record your time next to each task
5. Your goal: complete all 10 in under 15 minutes

---

### The Tasks

| # | Task | What You Should Do | Target Time |
|---|---|---|---|
| **1** | **Find your Subscription ID** | Navigate to Subscriptions, click your subscription, copy the Subscription ID from the Overview page | 30 seconds |
| **2** | **Find your Tenant ID** | Navigate to Microsoft Entra ID, copy the Tenant ID from the Overview page | 30 seconds |
| **3** | **Create a Resource Group** | Create a resource group named `speed-drill-rg` in East US | 1 minute |
| **4** | **Open Cloud Shell and verify login** | Click the >_ icon, run `az account show --output table`, confirm your subscription is listed | 1 minute |
| **5** | **Star "Function App" as a favourite** | Go to All services, find Function App, click the star, verify it appears in the left sidebar | 45 seconds |
| **6** | **Find where deployment slots are configured** | Navigate to App Services (if no app exists, just navigate to the App Services page and note the path: inside an app → Deployment → Deployment slots) | 1 minute |
| **7** | **Navigate to Cost Analysis, filtered by your resource group** | Open Cost Management → Cost Analysis → Add filter → Resource group → select `speed-drill-rg` | 1.5 minutes |
| **8** | **Find where you'd create a Key Vault secret** | Navigate to Key Vaults (if no vault exists, just navigate to the Key Vaults page and note the path: inside a vault → Objects → Secrets → + Generate/Import) | 1 minute |
| **9** | **Find the App Registrations page** | Navigate to Microsoft Entra ID → App registrations | 30 seconds |
| **10** | **Delete the Resource Group you created** | Navigate to `speed-drill-rg`, click Delete resource group, type the name, confirm deletion | 1.5 minutes |

---

### Scoring

| Total Time | Rating | Comment |
|---|---|---|
| **Under 10 minutes** | ⭐⭐⭐ Expert | You'll fly through labs |
| **10–15 minutes** | ⭐⭐ Comfortable | You're ready for Module 1 |
| **15–20 minutes** | ⭐ Getting There | Run through the drill one more time tomorrow |
| **Over 20 minutes** | Keep Practising | Re-read the sections for the tasks you struggled with, then try again |

> 📝 **NOTE:** If you scored ⭐ or below, don't worry — the portal becomes second nature after a few labs. The purpose of this drill is to identify which areas feel unfamiliar so you can focus on those sections. By the time you finish Module 3, you'll be navigating the portal without thinking.

---

## 🔗 Module Connections

| Connection | Details |
|---|---|
| **F01 – How the Cloud Works** | You learned about Azure's service landscape — now you know how to find and navigate to each service in the portal |
| **F02 – Azure Hierarchy, ARM, and IaC** | Resource Groups, deployments, and tags — you now know where to find these in the portal |
| **F03 – Developer Basics** | Cloud Shell in this guide connects to the terminal and CLI skills from F03 |
| **F05 – Lab Zero** | Your first hands-on lab — you'll use every portal skill from this guide |
| **Every Module (M01–M13)** | You'll navigate to App Service, Functions, Key Vault, Storage, Cosmos DB, and APIM blades constantly |

---

## What We Covered — Checklist

Use this checklist to confirm you're ready to move on. Check off each item you can do confidently:

- [ ] **Identify the three main portal areas** — top navigation bar, left sidebar, main content area
- [ ] **Use the search bar** to find any Azure service in under 5 seconds
- [ ] **Star/favourite services** to add them to the left sidebar
- [ ] **Create a Resource Group** using the portal form
- [ ] **Browse a Resource Group** and find the Activity Log and Deployments tab
- [ ] **Delete a Resource Group** and understand that it deletes all resources inside
- [ ] **Follow the universal 6-step resource creation pattern** — Create a resource → Search → Create → Fill form → Review + Create → Create
- [ ] **Interpret validation results** — understand green "Validation passed" and red "Validation failed" messages
- [ ] **Use the notifications bell** to track deployments and click "Go to resource"
- [ ] **Open Cloud Shell** and run a command without needing to log in
- [ ] **Upload/download files** to/from Cloud Shell
- [ ] **Navigate to App Service** configuration, deployment slots, and scale settings
- [ ] **Navigate to Function App** functions list, function keys, and Get function URL
- [ ] **Navigate to Key Vault** secrets, networking, and access configuration
- [ ] **Navigate to Storage Account** containers, access keys, and lifecycle management
- [ ] **Navigate to Cosmos DB** Data Explorer and Keys
- [ ] **Navigate to API Management** APIs list and policies editor
- [ ] **Open Cost Analysis** and filter by resource group
- [ ] **Create a budget alert** with an 80% threshold and email notification
- [ ] **Find Microsoft Entra ID** and locate the Tenant ID
- [ ] **Navigate to App Registrations** and understand what they are (identity for your app)
- [ ] **Complete the 10-task speed drill** in under 15 minutes

> **Next up:** [F05 – Lab Zero: Your First Complete Azure Experience](../Layer_0_Foundation/F05_Lab_Zero_First_Azure_Experience.md) — where you'll put ALL of these portal skills into practice by creating your first Azure resource, deploying code, and cleaning everything up.
