# Module 1: Azure App Service Web Apps

---

| | |
|---|---|
| **Module** | M01 — Azure App Service Web Apps |
| **Exam Domain** | Develop Azure Compute Solutions (25–30%) |
| **Exam Weight** | Part of the highest-weighted domain on the AZ-204 exam |
| **Estimated Study Time** | 3–4 hours |
| **Prerequisites** | F01 (Cloud Concepts), F02 (Azure Account Setup), F03 (Developer Toolkit), F04 (Portal Navigation), F05 (Lab Zero) |
| **What You Will Learn** | How to create, configure, deploy, and scale web applications using Azure App Service — the most fundamental compute service on the AZ-204 exam |

---

## Section 1: What Problem Does App Service Solve?

### 🔴 Exam Relevance: HIGH

### The Problem: Managing Servers Is Not Your Job

Imagine you wrote a web application — a simple website that shows a restaurant menu. You want anyone in the world to access it from their browser. But to make that happen, you need more than just code. You need:

- A **physical server** (or virtual machine) running 24/7
- An **operating system** (Windows or Linux) installed and kept up to date
- **Security patches** applied regularly so hackers cannot break in
- **Networking** configured so the internet can reach your server
- **Monitoring** to know when something breaks at 3 AM
- **Backup plans** in case the server hardware fails

That is an enormous amount of work that has nothing to do with your restaurant menu application. You are a developer — your job is writing code, not babysitting servers.

This is the exact problem that **Azure App Service** solves.

### The Analogy: A Managed Hotel for Your Application

Think of Azure App Service as a **managed hotel** for your application:

| Hotel Concept | App Service Equivalent |
|---|---|
| The hotel building (walls, roof, plumbing, electricity) | The server hardware, operating system, networking |
| Hotel management (cleaning, maintenance, security guards) | Azure manages patching, monitoring, backups, scaling |
| Your hotel room (your personal space) | Your application — your code, your configuration |
| You check in with your luggage | You deploy your code to App Service |
| You do NOT fix the plumbing or replace the roof | You do NOT patch the OS or replace server hardware |

You bring your application (your luggage), check it into the hotel (deploy to App Service), and Azure handles everything about the building itself. If a pipe bursts (server fails), Azure fixes it — not you.

### The Technical Definition: Platform as a Service (PaaS)

Azure App Service is a **fully managed Platform as a Service (PaaS)** for hosting web applications, REST APIs, and mobile backends.

Let us break down what "Platform as a Service" means:

**PaaS (Platform as a Service)** is a cloud model where the cloud provider manages the infrastructure (servers, operating system, networking, storage) and you only manage your application and its data. It sits between two other models:

| Model | What YOU Manage | What AZURE Manages | Analogy |
|---|---|---|---|
| **IaaS** (Infrastructure as a Service) | OS, runtime, app, data | Hardware, networking, storage | Renting an empty apartment — you furnish and maintain it |
| **PaaS** (Platform as a Service) | App and data only | Hardware, OS, runtime, patches, networking | Staying in a hotel — you bring your luggage, they handle the building |
| **SaaS** (Software as a Service) | Nothing — just use it | Everything | Using Gmail — Google handles everything |

App Service is PaaS. You deploy code. Azure handles servers.

### Supported Runtimes

A **runtime** is the software environment that executes your application code. Different programming languages need different runtimes. App Service supports:

| Runtime | Language | Example Use Case |
|---|---|---|
| .NET 8 | C# | Enterprise web applications |
| Node.js 20 | JavaScript | REST APIs, lightweight apps |
| Python 3.12 | Python | Data-driven apps, Flask/Django |
| Java 17 | Java | Corporate applications |
| PHP 8.x | PHP | Content management systems |
| Ruby | Ruby | Ruby on Rails applications |

App Service supports both **Windows** and **Linux** hosting. When you create an App Service, you choose the operating system. This choice cannot be changed after creation — you would need to create a new App Service.

💡 **KEY CONCEPT**
> Azure App Service is a fully managed PaaS that lets developers deploy web apps without managing servers. You bring code. Azure handles infrastructure. It supports .NET, Node.js, Python, Java, PHP, and Ruby on both Windows and Linux.

---

## Section 2: App Service Plans — The Foundation

### 🔴 Exam Relevance: HIGH

### The Problem: How Does Azure Know How Much Hardware to Give Your App?

When you deploy an application to App Service, Azure needs to know: How powerful should the server be? How much memory? How many CPU cores? Should your app share resources with others, or get its own dedicated machine?

This is what an **App Service Plan** answers.

### The Analogy: The Plan Is the Hardware, the App Is the Software

Think of it this way:

- The **App Service Plan** is the **apartment building** — it defines the size of the building, the number of floors, the quality of the facilities
- The **App Service (your app)** is a **tenant** living in that building

Multiple tenants (apps) can live in the same building (plan). They share the same facilities (CPU, memory, disk). If the building is small (Free tier), everyone is cramped. If the building is large (Premium tier), everyone has room.

**Key rule:** The App Service Plan determines what features are available to ALL apps running on it. If you need deployment slots, the PLAN must support slots — not just the app.

### The Complete Tier Table

This table is one of the most important things to memorise for the AZ-204 exam:

| Tier | SKU Names | Compute Type | Key Features | Limitations |
|---|---|---|---|---|
| **Free** | F1 | Shared (multi-tenant) | 1 GB storage, 60 CPU min/day | No custom domain, no SSL, no scale, no slots, no Always On |
| **Shared** | D1 | Shared (multi-tenant) | Custom domains | No SSL binding, no scale, no slots, no Always On |
| **Basic** | B1, B2, B3 | Dedicated VMs | Custom domains, SSL, manual scale (up to 3 instances) | No autoscale, NO deployment slots, no VNet Integration |
| **Standard** | S1, S2, S3 | Dedicated VMs | Autoscale (up to 10 instances), **5 deployment slots**, VNet Integration, daily backups | — |
| **Premium v3** | P1v3, P2v3, P3v3 | Dedicated VMs (faster hardware) | Autoscale (up to 30 instances), **20 deployment slots**, VNet Integration, enhanced performance | Higher cost |
| **Isolated v2** | I1v2, I2v2, I3v2 | Dedicated VMs in a private App Service Environment (ASE) | Full VNet isolation, maximum scale (up to 100 instances) | Highest cost, most complex setup |

Let us read this table carefully. The tiers go from cheapest (Free) to most expensive (Isolated). As you move up:
- You get **more features** (slots, autoscale, VNet)
- You get **more instances** (more copies of your app for traffic)
- You get **dedicated resources** instead of shared ones
- You pay **more money**

🚨 **EXAM ALERT**
> **Deployment slots require Standard tier or higher.** If an exam question asks about zero-downtime deployments using slots, the answer MUST use at least Standard (S1). Basic does NOT support slots.
>
> **Autoscale requires Standard tier or higher.** If a question asks about automatically adding instances when CPU goes above 70%, the plan must be Standard or above.
>
> **VNet Integration requires Standard tier or higher.** If an app needs to connect to resources inside a Virtual Network, it needs at least Standard.

### Multiple Apps on One Plan

You can run multiple App Service apps on a single App Service Plan. They share the same compute resources (CPU and memory). This is cost-effective if you have several small apps that do not individually need much power.

However, if one app consumes too much CPU, it affects all other apps on the same plan. For production workloads, consider separate plans or higher tiers.

✅ **CHECKPOINT**
> Before moving on, make sure you can answer:
> 1. What is the minimum tier needed for deployment slots? *(Answer: Standard)*
> 2. What is the difference between scaling up and the tier table? *(Answer: The tier table shows the available tiers. Scaling up means moving to a higher tier.)*
> 3. Can you run 3 different web apps on 1 Standard plan? *(Answer: Yes — multiple apps can share one plan.)*

---

## Section 3: Deployment Methods

### 🟡 Exam Relevance: MEDIUM

### The Problem: How Do I Get My Code Onto App Service?

You have written an application on your local computer. The App Service is running in Azure. How do you get your code from your computer to Azure?

Azure App Service supports several deployment methods. Think of these as different ways to ship a package to a hotel — you can mail it, deliver it in person, or have someone bring it for you.

### Deployment Method Reference

| Method | How It Works | Best For | CLI Command or Tool |
|---|---|---|---|
| **ZIP Deploy** | Package your app as a .zip file and upload it | Quick deployments from CLI or CI/CD | `az webapp deployment source config-zip --resource-group myRG --name myApp --src app.zip` |
| **Local Git** | Push from a local Git repository directly to App Service | Solo developers who use Git | `git push azure main` (after configuring App Service as a remote) |
| **GitHub Actions** | Automated deployment triggered by pushing code to GitHub | Teams using GitHub for source control | Configured via `.github/workflows/` YAML file |
| **Azure DevOps** | Automated pipeline in Azure DevOps | Enterprise teams using Azure DevOps | Configured via Azure Pipelines YAML |
| **Docker Container** | Deploy a Docker image from Azure Container Registry (ACR) or Docker Hub | Containerised applications | `az webapp create --deployment-container-image-name myregistry.azurecr.io/myapp:latest` |
| **FTP/FTPS** | Upload files via FTP (File Transfer Protocol) | Legacy systems — NOT recommended for new apps | FTP client (e.g., FileZilla) |

### ZIP Deploy — The One to Know for the Exam

ZIP deploy is the most commonly tested CLI deployment method. Here is the command broken down:

```bash
# Deploy a ZIP file to an App Service
az webapp deployment source config-zip \
  --resource-group myResourceGroup \    # ← The resource group containing your App Service
  --name myWebApp \                     # ← The name of your App Service
  --src ./myapp.zip                     # ← Path to the ZIP file on your local machine
```

Each flag explained:
- `--resource-group`: Tells Azure which resource group (a named container that groups related Azure resources) to look in
- `--name`: The name of the specific App Service to deploy to
- `--src`: The path to the ZIP file containing your application code

💡 **KEY CONCEPT**
> For the AZ-204 exam, know that ZIP deploy is the most common CLI-based deployment method. The command is `az webapp deployment source config-zip`. GitHub Actions and Azure DevOps are used for CI/CD (Continuous Integration/Continuous Deployment) — automated pipelines that deploy code every time you push changes to your repository.

---

## Section 4: Deployment Slots

### 🔴 Exam Relevance: HIGH

### The Problem: Deploying Without Breaking Your Live Application

Imagine your website is live, serving thousands of users. You have a new version ready. If you deploy the new version directly to your live app:

- Users might see a broken page during the deployment
- If the new version has a bug, ALL users are affected
- Rolling back to the old version takes time — during which users see errors

This is called **downtime** — the period when your application is unavailable or broken. For business-critical applications, even a few minutes of downtime can mean lost revenue and unhappy customers.

### The Analogy: A Rehearsal Stage and a Live Stage

Think of a theatre with two stages:

- **The live stage** (production slot): This is where the audience is watching. The show is running.
- **The rehearsal stage** (staging slot): This is behind the curtain. Actors can practice the new version of the show without the audience seeing anything.

When the rehearsal is perfect, you **swap** the stages — the rehearsal stage becomes the live stage, and the old live stage becomes the new rehearsal stage. The audience never sees the transition.

If something goes wrong after the swap, you can **swap back** immediately. The old show is still on the rehearsal stage, ready to go.

This is exactly how **deployment slots** work in Azure App Service.

### How Deployment Slots Work

Every App Service has a **production slot** by default — this is your live application. When you create additional deployment slots, you get separate instances of your app with their own hostnames:

| Slot | URL | Purpose |
|---|---|---|
| Production (default) | `https://myapp.azurewebsites.net` | Live application — users see this |
| Staging | `https://myapp-staging.azurewebsites.net` | Testing new version before going live |
| Dev | `https://myapp-dev.azurewebsites.net` | Development and debugging |

**The deployment workflow:**
1. Deploy new code to the **staging slot**
2. Test the staging slot to verify everything works
3. **Swap** the staging and production slots
4. Users now see the new version — with zero downtime
5. If something is wrong, **swap back** — the old version returns instantly

### The Warm-Up Process

When you initiate a swap, App Service does not simply switch URLs. It performs a **warm-up**:

1. Azure sends HTTP requests to the staging slot
2. It waits until the staging slot responds successfully
3. Only then does it complete the swap

This ensures the new version is fully loaded and ready before any users see it. No cold start, no loading screens.

### Sticky vs Non-Sticky Settings — Critical Exam Topic

This is one of the most tested App Service concepts on the AZ-204 exam.

When you swap slots, some settings **travel with the code** (they swap) and some settings **stay with the slot** (they do NOT swap). Settings that stay with the slot are called **sticky** or **slot settings**.

| Setting Type | What Happens During Swap | Why |
|---|---|---|
| **Sticky (slot settings)** | Stays with the slot — does NOT move | Production needs its own database connection, staging needs its own |
| **Non-sticky (regular settings)** | Swaps with the code — moves to the other slot | Application-level settings that should follow the code |

**What is typically sticky (stays with slot):**
- Connection strings to databases (production connects to production DB, staging connects to staging DB)
- App settings marked as "Slot setting"
- Authentication settings
- Custom domain bindings
- SSL certificates

**What is typically non-sticky (swaps with code):**
- Application code itself
- General app settings NOT marked as "Slot setting"
- Most configuration values

To make a setting sticky, you check the **"Deployment slot setting"** checkbox next to it in the Azure portal (or use the `slotSetting` flag in CLI).

🚨 **EXAM ALERT**
> **Sticky settings stay with the SLOT, not the app.** When you swap staging to production, the production slot KEEPS its own connection string. The staging code moves to production, but the production connection string stays in place. If the exam asks "After swapping staging to production, which database does the production slot connect to?" — the answer is the production database, because the connection string is sticky and stays with the production slot.

### Slot Requirements

Remember from Section 2:
- **Free, Shared, and Basic** tiers: NO deployment slots available
- **Standard** tier: Up to **5** deployment slots
- **Premium** tier: Up to **20** deployment slots
- **Isolated** tier: Up to **20** deployment slots

✅ **CHECKPOINT**
> Before moving on, make sure you can answer:
> 1. What is the URL format for a staging slot? *(Answer: `https://appname-slotname.azurewebsites.net`)*
> 2. If a connection string is marked as a slot setting, what happens to it during a swap? *(Answer: It stays with the slot — it does NOT swap.)*
> 3. What tier minimum is required for deployment slots? *(Answer: Standard — S1 or above.)*

---

## Section 5: Autoscaling

### 🔴 Exam Relevance: HIGH

### The Problem: Handling Traffic That Changes Over Time

Imagine you run an online store. On a normal Tuesday, you get 500 visitors per hour. On Black Friday, you get 50,000 visitors per hour. If your app is sized for 500 visitors, it will crash on Black Friday. If it is sized for 50,000 visitors, you are wasting money on 364 other days.

You need your app to **automatically grow** when traffic increases and **automatically shrink** when traffic decreases. This is **autoscaling**.

### Two Types of Scaling: Up vs Out

There are two completely different ways to make your application handle more load:

**Scale Up (Vertical Scaling)** — Make the machine bigger:
- Move from a Standard S1 plan to a Standard S3 plan
- Same number of machines, but each machine has more CPU and RAM
- Manual process — you change the tier in Azure
- Requires a brief restart
- Analogy: Replacing your sedan with a truck — same number of vehicles, each one is bigger

**Scale Out (Horizontal Scaling)** — Add more machines:
- Stay on the same plan tier, but run multiple copies (instances) of your app
- Azure distributes traffic across all instances using a load balancer
- This IS autoscaling — it happens automatically based on rules you define
- No restart required — new instances are added alongside existing ones
- Analogy: Hiring more cashiers during the lunch rush — same-sized registers, just more of them

| | Scale Up (Vertical) | Scale Out (Horizontal) |
|---|---|---|
| What changes | Machine size (CPU, RAM) | Number of machines (instances) |
| How it works | Change plan tier | Add/remove identical instances |
| Automatic? | No — manual | Yes — autoscale rules |
| Restart? | Brief restart | No restart |
| Analogy | Bigger truck | More trucks |
| Exam term | Scale up | Autoscale / scale out |

🚨 **EXAM ALERT**
> **Autoscale adds IDENTICAL instances.** It does NOT create different-sized machines. It does NOT load balance different apps. Every new instance is an exact copy of your app running on the same tier hardware. A load balancer (a service that distributes incoming traffic across multiple instances) automatically distributes requests across all instances.

### Autoscale Rule Components

An autoscale rule has five components. You need to know all five for the exam:

| Component | What It Defines | Example |
|---|---|---|
| **Metric** | What measurement triggers the rule | CPU Percentage, HTTP Queue Length, Memory Percentage |
| **Threshold** | The value that triggers the action | CPU > 70% |
| **Duration** | How long the metric must exceed the threshold before acting | 5 minutes (prevents reacting to brief spikes) |
| **Action** | What to do when triggered | Increase instance count by 1 |
| **Cooldown** | How long to wait after an action before evaluating again | 5 minutes (prevents rapid scaling back and forth) |

Here is a concrete example of a complete autoscale rule:

> "When the **average CPU** across all instances is **above 70%** for **5 minutes**, **add 1 instance**. After adding, **wait 5 minutes** before evaluating again."

And the corresponding scale-in rule:

> "When the **average CPU** across all instances is **below 30%** for **5 minutes**, **remove 1 instance**. After removing, **wait 5 minutes** before evaluating again."

### Metric-Based vs Schedule-Based Rules

You can use two types of autoscale rules — or combine them:

**Metric-based rules:** React to real-time measurements
- CPU Percentage > 70% → add instance
- HTTP Queue Length > 100 → add instance
- Memory > 80% → add instance

**Schedule-based rules:** Set instance counts for known patterns
- Every Friday at 6 PM → set minimum instances to 5 (for weekend traffic)
- Every Monday at 6 AM → set minimum instances to 2 (weekday baseline)

You can combine both: "Always have at least 3 instances on weekends, AND add more if CPU goes above 70%."

### Autoscale Requirements

- **Minimum tier:** Standard (S1) or higher — Free, Shared, and Basic do NOT support autoscale
- **Instance limits:** You set minimum, maximum, and default instance counts
  - Minimum: The fewest instances that must always run (e.g., 2)
  - Maximum: The most instances autoscale will ever create (e.g., 10)
  - Default: Used when metrics are unavailable (e.g., 2)

💡 **KEY CONCEPT**
> Scale up = bigger machine (manual, vertical). Scale out = more machines (autoscale, horizontal). Autoscale adds identical instances based on metric or schedule rules. Requires Standard tier or higher. Always define both scale-out AND scale-in rules to avoid running unnecessary instances.

---

## Section 6: App Service Configuration

### 🟡 Exam Relevance: MEDIUM

### The Problem: Controlling How Your App Behaves in Azure

Your app is deployed and running. But there are dozens of settings that control its behaviour — should it stay running even when nobody is visiting? Should it enforce HTTPS? What database should it connect to? These settings are configured in the **App Service Configuration**.

### Key Configuration Settings

#### Always On

**The problem:** When no one visits your web app for a period of time, Azure unloads it from memory to save resources. The next visitor experiences a slow load (called a **cold start**) because Azure has to reload the app.

**The solution:** **Always On** keeps your app loaded in memory at all times, even when no one is visiting.

| Setting | Default | When to Enable | Requirement |
|---|---|---|---|
| Always On | OFF (for Free/Shared), ON (for Basic+) | WebJobs, apps that must respond instantly | Basic tier or higher (not available on Free/Shared) |

📝 **NOTE**
> Always On is required for **WebJobs** (background tasks in App Service). If Always On is off, WebJobs may stop running when the app is unloaded.

#### ARR Affinity (Application Request Routing)

**The problem:** When you have multiple instances (from scaling out), a user's requests might go to different instances each time. If your app stores user data in memory (like a shopping cart), the data could be lost when the request goes to a different instance.

**The solution:** **ARR Affinity** uses a cookie to route all requests from the same user to the same instance (called **sticky sessions**).

| Setting | Default | When to Enable | When to Disable |
|---|---|---|---|
| ARR Affinity | ON | Stateful apps that store data in memory | Stateless apps (recommended for most modern apps) |

**Best practice:** Modern applications should be **stateless** (they store session data in a database or cache like Redis, not in memory). For stateless apps, turn ARR Affinity **OFF** — it allows Azure to distribute traffic evenly across all instances.

#### HTTPS Only

| Setting | Default | What It Does |
|---|---|---|
| HTTPS Only | OFF | When enabled, all HTTP requests are automatically redirected to HTTPS |

**HTTPS** (HyperText Transfer Protocol Secure) encrypts data between the user's browser and your app. You should always enable this in production.

#### Minimum TLS Version

**TLS** (Transport Layer Security) is the encryption protocol that makes HTTPS work. Older versions of TLS have security vulnerabilities.

| Setting | Recommended Value | Why |
|---|---|---|
| Minimum TLS Version | **1.2** | TLS 1.0 and 1.1 are deprecated and have known security flaws. Azure defaults to 1.2. |

🚨 **EXAM ALERT**
> If an exam question mentions security hardening, the answer often includes setting the minimum TLS version to 1.2 and enabling HTTPS Only. These are standard security configurations.

### App Settings and Connection Strings

**App Settings** are key-value pairs that your application reads at runtime. In Azure App Service, app settings are injected as **environment variables** (named values that the operating system makes available to running applications).

**Why use App Settings instead of hard-coding values?**
- Different values for different environments (staging database vs production database)
- You can change values without redeploying code
- Secrets are not stored in source code

```bash
# Set an app setting via CLI
az webapp config appsettings set \
  --resource-group myRG \              # ← Resource group containing the app
  --name myWebApp \                    # ← App Service name
  --settings DB_HOST=myserver.database  # ← KEY=VALUE pair
```

**Connection Strings** are a special type of app setting specifically designed for database connections. They have a type field (SQLServer, MySQL, PostgreSQL, Custom) that ensures proper handling.

```bash
# Set a connection string via CLI
az webapp config connection-string set \
  --resource-group myRG \                       # ← Resource group
  --name myWebApp \                             # ← App Service name
  --settings MyDB="Server=myserver;Database=mydb" \  # ← Connection string value
  --connection-string-type SQLServer            # ← Type of database
```

**How your code reads these values:**

```csharp
// In C#, App Settings become environment variables
string dbHost = Environment.GetEnvironmentVariable("DB_HOST");
// Returns: "myserver.database"
```

```javascript
// In Node.js, App Settings become environment variables
const dbHost = process.env.DB_HOST;
// Returns: "myserver.database"
```

💡 **KEY CONCEPT**
> App Settings in Azure App Service override values in your application's configuration files. If your code has a `appsettings.json` with `DB_HOST=localhost` and Azure has an App Setting `DB_HOST=production-server`, the Azure value wins at runtime. This is how you run the same code in different environments with different configurations.

✅ **CHECKPOINT**
> Before moving on, make sure you can answer:
> 1. What does Always On do and when is it required? *(Answer: Keeps the app loaded in memory. Required for WebJobs.)*
> 2. When should you disable ARR Affinity? *(Answer: For stateless apps — which is most modern apps.)*
> 3. What is the recommended minimum TLS version? *(Answer: 1.2)*

---

## Section 7: Easy Auth (Built-In Authentication)

### 🔴 Exam Relevance: HIGH

### The Problem: Adding Login Without Writing Authentication Code

You want users to sign in to your web app using their Microsoft, Google, or Facebook account. Writing authentication code from scratch is complex, error-prone, and a security risk. You need to handle tokens, validate signatures, manage sessions, protect against attacks — and one mistake could expose user data.

### The Analogy: A Security Guard at the Hotel Entrance

Remember our hotel analogy? Easy Auth is like a **security guard at the hotel entrance**:

- Every guest (HTTP request) must show ID (authentication token) to the guard
- The guard checks the ID before the guest enters the building
- If the ID is invalid, the guard turns the guest away (401 Unauthorized)
- Your app (the hotel staff inside) never needs to check IDs — the guard already did it

The key insight: the security guard works **outside** your app. Your code never handles authentication logic. App Service handles it at the platform level.

### How Easy Auth Works

1. **You enable Easy Auth** in App Service configuration (not in your application code)
2. **You choose identity providers** — the services users can log in with
3. **App Service intercepts every HTTP request** before it reaches your code
4. **App Service validates the authentication token**
5. If valid → the request proceeds to your code (with user identity information available)
6. If invalid or missing → App Service returns a 401 (Unauthorized) or redirects to the login page

### Supported Identity Providers

| Provider | Description | Common Use |
|---|---|---|
| **Microsoft Entra ID** (formerly Azure AD) | Microsoft's identity service | Corporate/enterprise apps |
| **Facebook** | Facebook Login | Consumer-facing apps |
| **Google** | Google Sign-In | Consumer-facing apps |
| **Twitter** | Twitter OAuth | Social apps |
| **GitHub** | GitHub OAuth | Developer tools |
| **OpenID Connect** | Any provider supporting OpenID Connect standard | Custom identity providers |

### Token Store

When a user successfully authenticates, App Service stores the authentication token in a **token store** — a built-in, encrypted cache associated with your app. Your application code can access these tokens to:

- Read user profile information (name, email)
- Make API calls on behalf of the user (e.g., call Microsoft Graph)
- Determine the user's identity without any authentication libraries

The token store is enabled automatically when you enable Easy Auth.

🚨 **EXAM ALERT**
> **Easy Auth is configured at the App Service level, NOT inside your application code.** If an exam question asks where authentication is configured for a web app using App Service's built-in authentication, the answer is the App Service configuration (in the portal: App Service → Authentication), NOT in the application's source code. No code changes are needed to enable Easy Auth.

💡 **KEY CONCEPT**
> Easy Auth provides built-in authentication/authorization without writing code. It intercepts requests at the platform level, validates tokens, and passes authenticated requests to your app. It supports Microsoft Entra ID, Facebook, Google, Twitter, GitHub, and any OpenID Connect provider. Configured entirely in App Service — no code changes needed.

---

## Section 8: Networking

### 🔴 Exam Relevance: HIGH

### The Problem: Connecting Your App to Private Resources

By default, an App Service is accessible from the public internet. Any browser can reach `https://myapp.azurewebsites.net`. But what about the reverse direction — what if your app needs to reach a database that is inside a private network? Or what if you need to restrict who can access your app from the internet?

App Service networking features solve three problems:
1. **Outbound connections** — your app needs to reach private resources
2. **On-premises connections** — your app needs to reach servers in your company's data centre
3. **Inbound restrictions** — you need to control who can access your app

### VNet Integration (Outbound Only)

**VNet (Virtual Network)** is a logically isolated network in Azure — like a private office building where only authorised resources can communicate. Resources inside a VNet have private IP addresses that the public internet cannot reach.

**The problem:** Your App Service is on the public internet. Your database is inside a VNet (private network). By default, they cannot talk to each other.

**The solution:** **VNet Integration** connects your App Service to a VNet, allowing your app to make **outbound** calls to resources inside that VNet.

Think of it like this: VNet Integration gives your app a **badge** to enter the private building (VNet). Your app can walk into the building and talk to resources inside. But the badge does NOT change who can visit your app from outside — that is a separate concern.

```
┌──────────────────┐         VNet Integration          ┌──────────────────┐
│   App Service    │ ─────────────────────────────────→ │   Virtual Network │
│  (public internet)│      OUTBOUND traffic only        │  ┌──────────────┐│
│                  │                                     │  │   Database   ││
│                  │                                     │  │ (private IP) ││
│                  │                                     │  └──────────────┘│
└──────────────────┘                                     └──────────────────┘
```

🚨 **EXAM ALERT**
> **VNet Integration is OUTBOUND ONLY.** It allows your app to reach INTO a VNet (to access a database, for example). It does NOT restrict inbound traffic to your app. It does NOT place your app inside the VNet. If an exam question asks how to restrict inbound traffic, VNet Integration is the WRONG answer — use Access Restrictions instead.

**Requirements:**
- App Service Plan must be **Standard tier or higher**
- The VNet must be in the **same region** as the App Service (for regional VNet Integration)

### Hybrid Connections

**The problem:** Your company has a database server in its own data centre (on-premises — not in Azure). Your Azure App Service needs to connect to it. But the on-premises server is behind a corporate firewall and has no public IP address.

**The solution:** **Hybrid Connections** create a secure tunnel between your App Service and an on-premises resource. It uses an agent called the **Hybrid Connection Manager** installed on a server in your on-premises network.

| Feature | VNet Integration | Hybrid Connections |
|---|---|---|
| Connects to | Azure VNet resources | On-premises resources (or Azure VMs) |
| Protocol | All TCP/UDP | TCP only (specific host:port) |
| Requires | Standard+ plan, VNet in same region | Hybrid Connection Manager agent on-premises |
| Use case | App → private Azure database | App → on-premises SQL Server |

### Access Restrictions (Inbound Firewall)

**The problem:** Your App Service is publicly accessible. You want to restrict access so only specific IP addresses or VNets can reach it.

**The solution:** **Access Restrictions** act as an inbound firewall. You create rules that allow or deny traffic based on:
- IP addresses or IP ranges
- VNet subnets (using Service Endpoints)
- Azure Front Door

```bash
# Add an access restriction rule to allow only a specific IP
az webapp config access-restriction add \
  --resource-group myRG \                # ← Resource group
  --name myWebApp \                      # ← App Service name
  --rule-name "AllowOfficeIP" \          # ← Human-readable name for the rule
  --action Allow \                       # ← Allow or Deny
  --ip-address 203.0.113.50/32 \         # ← IP address (/32 = single IP)
  --priority 100                         # ← Lower number = higher priority
```

### Networking Summary Table

| Feature | Direction | Purpose | Minimum Tier |
|---|---|---|---|
| **VNet Integration** | Outbound | App → private Azure resources | Standard |
| **Hybrid Connections** | Outbound | App → on-premises resources | Standard |
| **Access Restrictions** | Inbound | Control who can access the app | Available on all tiers |
| **App Service Environment (ASE)** | Both | Full VNet isolation (app lives inside VNet) | Isolated |

💡 **KEY CONCEPT**
> VNet Integration = outbound only (your app reaches into a VNet). Hybrid Connections = outbound to on-premises. Access Restrictions = inbound firewall. For full VNet isolation (both inbound and outbound), you need the Isolated tier with App Service Environment. Know the direction for each feature — this is a common exam question.

✅ **CHECKPOINT**
> Before moving on, make sure you can answer:
> 1. What direction is VNet Integration? *(Answer: Outbound only — app reaches into the VNet.)*
> 2. How do you connect App Service to an on-premises database? *(Answer: Hybrid Connections with the Hybrid Connection Manager agent.)*
> 3. How do you restrict who can access your App Service from the internet? *(Answer: Access Restrictions — an inbound IP firewall.)*

---

## Section 9: Managed Identity — The Application's Identity

### 🔴 Exam Relevance: HIGH

### The Problem: How Does Your App Prove Who It Is?

Let us think carefully about what your deployed App Service actually needs to do in the real world. It needs to:

- Read secrets from **Azure Key Vault** (your database passwords, API keys)
- Access blobs in **Azure Blob Storage**
- Write logs to a **Storage Account**
- Call the **Azure OpenAI API** on behalf of your company

Every single one of these actions requires your *application* to prove its identity to another Azure service. The question is: **how?**

The old, dangerous way: hardcode a username and password (or API key) into your `appsettings.json` file. This works but creates massive security problems — secrets stored in source code, rotation is a nightmare, and any compromise of the app means total compromise of every downstream service.

### The Analogy: A Corporate Staff Badge

In a large office building, employees carry a **staff badge**. They don't need to type a password every time they enter a secure room — they just badge in. The building's security system knows who they are because the badge is issued by the company's HR system and is cryptographically tied to that specific employee.

When an employee leaves the company, HR deactivates the badge. No other systems need to change.

**Managed Identity** is the staff badge for your Azure application. Azure issues a cryptographic identity to your App Service, and other Azure services can verify that identity automatically — no passwords, no credentials stored in code.

### What Is a Managed Identity?

A **Managed Identity** (MI) is an identity in Microsoft Entra ID that Azure creates, manages, and rotates automatically on behalf of your resource. It works like a service account, but you never manage the password — Azure does.

💡 **KEY CONCEPT**
> A Managed Identity gives your App Service an Entra ID identity. Other Azure services use RBAC to grant permissions to that identity. Your code authenticates using `DefaultAzureCredential` — no passwords, no connection strings, no secrets in code.

### System-Assigned vs User-Assigned Managed Identity

There are two types. The distinction is tested on the AZ-204 exam:

| Feature | System-Assigned MI | User-Assigned MI |
|---|---|---|
| **Created by** | Azure, automatically, when you enable it on a resource | You create it manually as a standalone Azure resource |
| **Tied to** | One specific resource (e.g., one App Service) | Can be assigned to *multiple* resources |
| **Lifecycle** | Deleted when the resource is deleted | Independent — you manage when it is deleted |
| **Best for** | Single-resource access — simplest setup | Shared identity across multiple apps or services |
| **Analogy** | Badge printed for one specific employee | A master passcard that multiple contractors can use |

🚨 **EXAM ALERT**
> **System-Assigned MI** is tied to one resource — deleting the App Service automatically deletes its identity. **User-Assigned MI** is a standalone resource that can be shared across multiple services and has an independent lifecycle. The exam will describe a scenario and ask which type to use.
>
> Scenario: "Multiple Azure Functions and one App Service all need to read from the same Key Vault" → Use **User-Assigned MI** (one identity, assigned to all three resources).
>
> Scenario: "One App Service needs to read from a Key Vault and nothing else" → Use **System-Assigned MI** (simpler, automatic lifecycle).

### The Zero-Trust Pattern: How It Works End-to-End

Here is the complete flow of how a Managed Identity replaces a password when your App Service reads a secret from Key Vault:

```
1. You enable System-Assigned MI on your App Service
2. Azure creates an identity in Entra ID for this specific App Service
3. You grant that identity the "Key Vault Secrets User" role on your Key Vault (via RBAC)
4. Your app code uses DefaultAzureCredential to authenticate:
   - In Cloud Shell / local dev: uses your `az login` credentials
   - In App Service: uses the Managed Identity automatically
5. Your code calls Key Vault to get the secret value
6. Key Vault checks: "Does this identity have the Secrets User role?" → YES → returns the secret
7. Your code receives the secret value at runtime — it was never stored in any config file
```

The critical point: **your code never sees, stores, or handles a credential**. The identity is managed entirely by Azure.

### Enabling Managed Identity via CLI

```bash
# Enable a System-Assigned Managed Identity on your App Service
az webapp identity assign \
  --name $APP_NAME \                # ← Your App Service name
  --resource-group az204-appservice-rg
```

This command returns the **Principal ID** — the unique identifier for this App Service's identity in Entra ID. **Save this value** — you need it to grant RBAC roles.

```json
{
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",  ← Save this
  "tenantId": "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy",
  "type": "SystemAssigned"
}
```

### Granting the Identity Permission to Key Vault

```bash
# Save the Principal ID from the output above
PRINCIPAL_ID=$(az webapp identity show \
  --name $APP_NAME \
  --resource-group az204-appservice-rg \
  --query principalId \
  --output tsv)

# Get your Key Vault resource ID
KV_ID=$(az keyvault show \
  --name $YOUR_KEYVAULT_NAME \
  --query id \
  --output tsv)

# Grant the App Service identity the "Secrets User" role
az role assignment create \
  --assignee $PRINCIPAL_ID \         # ← The identity we just created
  --role "Key Vault Secrets User" \  # ← Read-only access to secrets
  --scope $KV_ID                     # ← Scope it to this specific vault only
```

| Flag | What It Does |
|---|---|
| `--assignee` | The Principal ID of the Managed Identity |
| `--role` | The RBAC role to assign. `Key Vault Secrets User` allows reading secrets but NOT creating or deleting them |
| `--scope` | Limits where this role applies. Scoping to the vault (not the subscription) follows least-privilege |

### Reading the Code Side: DefaultAzureCredential

In your .NET application, you use the `DefaultAzureCredential` class from the Azure Identity SDK. This one class handles authentication for *both* local development and cloud deployment automatically:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

// DefaultAzureCredential tries multiple authentication methods in order:
// 1. Environment variables (CI/CD pipelines)
// 2. Managed Identity (when running in App Service, Functions, AKS)
// 3. Visual Studio credentials (local development)
// 4. Azure CLI credentials (az login — your Cloud Shell session)
// 5. ... and more fallbacks
var credential = new DefaultAzureCredential();

var client = new SecretClient(
    new Uri("https://your-vault.vault.azure.net/"),
    credential   // ← No password, no API key. Just the identity.
);

KeyVaultSecret secret = await client.GetSecretAsync("MyOpenAIKey");
string apiKey = secret.Value;
```

💡 **KEY CONCEPT**
> `DefaultAzureCredential` is the recommended way to authenticate to Azure services in application code. It automatically uses Managed Identity when running in Azure, and your developer credentials when running locally. **You write the same code for both environments** — no environment-specific credential handling.

👀 **UI Check:**
In the Azure Portal, after enabling Managed Identity:
1. Go to your App Service → Left menu → **Identity**
2. Under the **System assigned** tab, you will see **Status: On** and the **Object (principal) ID**
3. This is the identity that needs the RBAC role on Key Vault

---

## Module Connections

🔗 **MODULE LINK**

| Connected Module | How It Relates |
|---|---|
| **M02 — Azure Functions** | Functions can run on an App Service Plan (Dedicated plan). Many configuration concepts (App Settings, deployment) are shared. |
| **M03 — Containers (ACR, ACI, ACA)** | App Service can deploy Docker containers from Azure Container Registry. ACA is an alternative compute platform. |
| **M06 — Authentication** | Easy Auth uses Microsoft Entra ID, which is covered in depth in M06. OAuth flows and tokens are explained there. |
| **M07 — Secure Solutions** | App Settings can reference Key Vault secrets using Key Vault references. Managed Identities (M07) replace connection strings. |
| **M08 — API Management** | App Service APIs can be published through API Management for rate limiting, caching, and security policies. |
| **M11 — Monitoring** | Application Insights (M11) integrates with App Service for performance monitoring, logging, and diagnostics. |
| **F07 — Networking** | VNet Integration, Private Endpoints, and Access Restrictions are explained in depth in the Networking foundation module. |

---

## What We Covered — Summary Checklist

Use this checklist to verify you understand every key concept from this module:

- [ ] **App Service is PaaS** — you deploy code, Azure manages servers, OS, patching, and networking
- [ ] **App Service Plan** defines the hardware (tier, features, instance count); the app is software running on it
- [ ] **Tier table** — Free → Shared → Basic → Standard → Premium → Isolated, with increasing features and cost
- [ ] **Slots require Standard+** — Free, Shared, and Basic do NOT support deployment slots
- [ ] **Autoscale requires Standard+** — only Standard and above support automatic horizontal scaling
- [ ] **VNet Integration requires Standard+** — needed to connect to private Azure resources
- [ ] **Deployment methods** — ZIP deploy (CLI), Local Git, GitHub Actions, Azure DevOps, Docker, FTP
- [ ] **ZIP deploy command** — `az webapp deployment source config-zip`
- [ ] **Deployment slots** enable zero-downtime deployments through staging → swap → production
- [ ] **Sticky settings stay with the SLOT** — connection strings marked as slot settings do NOT swap
- [ ] **Non-sticky settings swap with the code** — they move during a swap operation
- [ ] **Warm-up** — App Service pings the staging slot before completing the swap
- [ ] **Scale up (vertical)** — bigger machine, manual, brief restart
- [ ] **Scale out (horizontal)** — more identical instances, automatic (autoscale), no restart
- [ ] **Autoscale rule components** — Metric, Threshold, Duration, Action, Cooldown
- [ ] **Autoscale adds identical instances** — does not create different-sized machines
- [ ] **Always On** — keeps app in memory, required for WebJobs
- [ ] **ARR Affinity** — sticky sessions, disable for stateless apps
- [ ] **HTTPS Only** — redirects HTTP to HTTPS
- [ ] **Minimum TLS 1.2** — standard security hardening
- [ ] **App Settings** — key-value pairs injected as environment variables, override local config files
- [ ] **Easy Auth** — built-in authentication at App Service level, no code changes needed (User Identity)
- [ ] **Easy Auth providers** — Entra ID, Facebook, Google, Twitter, GitHub, OpenID Connect
- [ ] **Token store** — automatic encrypted cache of authentication tokens
- [ ] **VNet Integration** — OUTBOUND ONLY, connects app to private Azure VNet resources
- [ ] **Hybrid Connections** — connects app to on-premises resources via TCP tunnel
- [ ] **Access Restrictions** — inbound IP firewall to control who can reach the app
- [ ] **Managed Identity (App Identity)** — gives the App Service an Entra ID identity without passwords
- [ ] **System-Assigned MI** — tied to one resource, deleted with the resource, simplest setup
- [ ] **User-Assigned MI** — standalone resource, assignable to multiple services, independent lifecycle
- [ ] **RBAC + MI** — grant the identity a role (e.g., `Key Vault Secrets User`) at a specific scope
- [ ] **DefaultAzureCredential** — authenticates with MI in Azure, developer credentials locally. Same code, both environments

---

## Self-Check Questions

Test your understanding with these three questions before moving to the lab:

**Question 1:** Your company wants to deploy a web application that requires deployment slots for zero-downtime deployments. The application also needs to connect to a database inside a Virtual Network. What is the minimum App Service Plan tier you need?

<details>
<summary>Click to reveal answer</summary>

**Standard (S1).** Standard is the minimum tier that supports both deployment slots (up to 5) AND VNet Integration. Basic supports neither slots nor VNet Integration. Premium and Isolated also work but are more expensive than necessary.

</details>

**Question 2:** You swap your staging slot to production. Your staging slot had a connection string marked as a "slot setting" pointing to a test database. After the swap, which database does the production slot connect to?

<details>
<summary>Click to reveal answer</summary>

**The production database.** Because the connection string is marked as a slot setting (sticky), it stays with the production slot. The code moves from staging to production, but the connection string does not move — it stays where it was. The production slot keeps its own connection string.

</details>

**Question 3:** Your web app needs to access an Azure SQL Database that is inside a Virtual Network. You enable VNet Integration. A colleague says this also prevents public internet users from accessing your web app. Is your colleague correct?

<details>
<summary>Click to reveal answer</summary>

**No, your colleague is incorrect.** VNet Integration is OUTBOUND ONLY — it allows your app to reach into the VNet (to access the database). It does NOT affect inbound traffic. Public internet users can still access your app. To restrict inbound access, you need Access Restrictions (an inbound IP firewall) or an App Service Environment (Isolated tier).

</details>
