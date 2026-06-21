# How the Cloud Works — From Concept to Practice

**Document:** F01 — Foundation Pack  
**Certification:** AZ-204: Developing Solutions for Microsoft Azure  
**Pages:** 10–14 | **Study Time:** 1.5 hours  
**Prerequisites:** AZ-900 or equivalent cloud awareness  
**Exam Relevance:** 🔴 HIGH — this mental model underpins every AZ-204 module  

---

## Section 1: Quick AZ-900 Recap

> 💡 **KEY CONCEPT**  
> You already know this from AZ-900 — now we go DEEPER. This section is a 2-minute confidence boost, not a re-teach. If anything feels unfamiliar, revisit your AZ-900 notes before continuing.

### What Is the Cloud? (30-Second Reminder)

You learned in AZ-900 that **cloud computing** means using someone else's computers — servers, storage, and networking — over the internet, instead of buying and managing your own hardware. Microsoft owns the data centres. You rent what you need, when you need it.

The key benefits you already know:

| Benefit | One-Line Reminder |
|---|---|
| **High Availability** | Your app stays online even if one server fails — Azure uses backup servers automatically |
| **Scalability** | Add more power (or more servers) when demand grows, shrink when it drops |
| **Elasticity** | Scalability that happens automatically based on real-time demand |
| **Reliability** | Azure's global infrastructure can recover from failures across regions |
| **Pay-as-you-go** | No upfront hardware costs — you pay only for what you use |

### The Shared Responsibility Model (Quick Refresh)

Think of renting a flat versus owning a house:

- **Owning a house (On-premises):** You maintain everything — the roof, the plumbing, the locks, the furniture.
- **Renting a flat (Cloud):** The landlord maintains the building structure, but you take care of your own belongings inside.

In Azure, **Microsoft is the landlord**. They are responsible for the physical data centres, the networking hardware, and the host operating systems. **You, the developer**, are responsible for your data, your application code, and how you configure access to your resources. The exact split depends on whether you use IaaS, PaaS, or SaaS.

### IaaS, PaaS, SaaS — The Pizza Analogy

Imagine you want pizza for dinner. You have four options:

| Option | Pizza Analogy | Cloud Model | What YOU Manage | What the PROVIDER Manages |
|---|---|---|---|---|
| **Make at home** | Buy flour, sauce, cheese. Make dough, bake it yourself | **On-premises** | Everything | Nothing |
| **Take and bake** | Buy a ready-made pizza, bake it in your oven | **IaaS** (Infrastructure as a Service) | OS, runtime, app, data | Servers, storage, networking |
| **Pizza delivery** | Order a pizza, it arrives cooked. You just eat it on your own plates | **PaaS** (Platform as a Service) | App and data only | Everything else |
| **Eat at restaurant** | Walk in, sit down, eat. They even wash the dishes | **SaaS** (Software as a Service) | Just your data (what you type/upload) | Everything |

**Where AZ-204 lives:** Almost entirely in the **PaaS** and **serverless** (a flavour of PaaS) rows. You will rarely touch IaaS in this certification. That's the key shift from AZ-900 awareness to AZ-204 practice.

> ✅ **CHECKPOINT**  
> Can you explain to a friend, using the pizza analogy, why a developer would prefer PaaS over IaaS? If yes, move on. If not, re-read the table above.

---

## Section 2: Why Developers Care About Cloud Differently Than IT Admins

### The Problem: Two People, Same Cloud, Different Worlds

When a company moves to the cloud, two groups of people interact with it — but they think about it in completely different ways. Understanding the *developer* perspective is the single most important mindset shift for AZ-204.

### IT Admin Perspective vs Developer Perspective

Imagine a hotel. The **IT Admin** is the building manager — they care about the structure of the building, the plumbing, the electrical wiring, the security cameras, and the fire alarms. The **Developer** is the guest — they care about their room, their schedule, their luggage, and what they want to accomplish during their stay.

Both use the same building. But they ask very different questions:

| Question | IT Admin Asks | Developer Asks |
|---|---|---|
| **Compute** | "How many VMs do we need? What size? Which OS?" | "Where does my code run? How do I deploy it?" |
| **Networking** | "What's the VNet topology? NSG rules? Firewall?" | "Can my app reach the database? Is my endpoint public?" |
| **Storage** | "What redundancy level? How much capacity?" | "How do I upload a file from my code? What SDK do I use?" |
| **Security** | "Who has access to the subscription? What policies?" | "How does my app authenticate to Key Vault without a password?" |
| **Monitoring** | "Are the VMs healthy? CPU and memory usage?" | "Is my app throwing errors? How long do API calls take?" |
| **Costs** | "What's the monthly bill for the infrastructure?" | "Does this code trigger charges every time it runs?" |

### AZ-204 Is the DEVELOPER Perspective on Azure

The AZ-900 exam gave you a panoramic view of Azure — a little bit of everything, from a high altitude. The **AZ-204 exam zooms in** on what a **developer** needs to know:

- How to **write code** that talks to Azure services
- How to **deploy applications** to Azure
- How to **secure applications** using Azure's identity and secrets services
- How to **monitor applications** once they are running
- How to **connect services** together using events and messages

> 📝 **NOTE**  
> AZ-204 does NOT expect you to configure firewalls, design VNet topologies, manage Active Directory forests, or set up disaster recovery for VMs. Those belong to AZ-104 (Administrator) or AZ-700 (Network Engineer).

### The Developer Mental Model: The Lifecycle Loop

Every developer — whether building for Azure, AWS, or a laptop — follows the same fundamental loop. Think of it as a conveyor belt in a factory that never stops:

```
code → build → package → deploy → monitor → repeat
```

Let's define each stage in plain English:

| Stage | What It Means | Real-World Analogy |
|---|---|---|
| **Code** | Write the application logic — the instructions that make the app do what it does | Writing the recipe for a dish |
| **Build** | Compile and check the code for errors — turn human-readable code into something the computer can execute | Proofreading the recipe and checking you have all ingredients |
| **Package** | Bundle the built code with its dependencies into a deployable unit (a ZIP file, a Docker container image, a NuGet package) | Packing the dish and its container for delivery |
| **Deploy** | Send the package to Azure so it runs on real servers and users can access it | Delivering the packed dish to the restaurant so customers can order it |
| **Monitor** | Watch the live app for errors, slow performance, and unexpected behaviour | Checking customer reviews and kitchen feedback after the dish is served |
| **Repeat** | Fix bugs, add features, and go back to the Code stage | Tweaking the recipe based on feedback and serving the improved version |

Every AZ-204 module maps to one or more stages of this loop. When you study Azure App Service, you are learning **deploy**. When you study Application Insights, you are learning **monitor**. When you study Azure Functions, you are learning how to **code** serverless logic. This loop is your compass.

> 💡 **KEY CONCEPT**  
> The developer lifecycle — code → build → package → deploy → monitor → repeat — is the mental model for the entire AZ-204 certification. Every service you learn fits somewhere in this loop.

---

## Section 3: The Azure Service Landscape for Developers

### The Problem: Azure Has Hundreds of Services — Which Ones Matter?

Azure offers over 200 services. That's overwhelming. The good news: **AZ-204 only tests about 20 of them**. This section gives you a map of every service you will encounter in this course, grouped by what it does.

Think of this map like a shopping mall directory — you don't need to visit every store today, but knowing where each one is helps you navigate confidently.

### Your AZ-204 Course Roadmap

> 💡 **KEY CONCEPT**  
> This map is your course roadmap. Bookmark this page. You will return to it at the start of every module to see where the new service fits in the bigger picture.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AZ-204 SERVICE MAP FOR DEVELOPERS                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🖥️  COMPUTE — Where Your Code Runs                                │
│  ├── Azure App Service         Managed hosting for web apps/APIs    │
│  ├── Azure Functions           Serverless, event-driven code        │
│  ├── Azure Container Registry  Private storage for container images │
│  ├── Azure Container Instances Run containers without servers       │
│  └── Azure Container Apps      Serverless containers at scale       │
│                                                                     │
│  💾  STORAGE — Where Your Data Lives                                │
│  ├── Azure Blob Storage        Files, images, videos, backups       │
│  ├── Azure Cosmos DB           Globally distributed NoSQL database  │
│  └── Azure Queue Storage       Simple message queue                 │
│                                                                     │
│  🔐  SECURITY — How Your App Proves Identity and Protects Secrets   │
│  ├── Azure Key Vault           Store secrets, keys, certificates    │
│  ├── Managed Identity          Password-free auth for Azure apps    │
│  ├── Microsoft Entra ID        Identity provider (users and apps)   │
│  └── MSAL                      Library to add sign-in to your code  │
│                                                                     │
│  🔗  INTEGRATION — How Services Talk to Each Other                  │
│  ├── Azure Event Grid          Lightweight event routing            │
│  ├── Azure Event Hubs          High-volume event streaming          │
│  ├── Azure Service Bus         Enterprise message broker            │
│  └── Azure API Management      API gateway, policies, analytics     │
│                                                                     │
│  📊  MONITORING — How You Watch Your Live App                       │
│  ├── Application Insights      App performance and error tracking   │
│  ├── Azure Monitor             Platform-level metrics and alerts    │
│  └── Log Analytics             Query and analyse log data           │
│                                                                     │
│  ⚡  CACHING — How You Make Things Faster                           │
│  ├── Azure Cache for Redis     In-memory data cache                 │
│  └── Azure CDN                 Cached content at the network edge   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### What Each Category Means

| Category | Developer Question It Answers | Example |
|---|---|---|
| **Compute** | "Where does my code actually run?" | You write a web API — App Service hosts it |
| **Storage** | "Where do I put data my app needs?" | Users upload photos — Blob Storage stores them |
| **Security** | "How does my app prove who it is and protect secrets?" | Your app needs a database password — Key Vault stores it |
| **Integration** | "How do I make two services react to each other?" | A new order is placed — Event Grid notifies the billing service |
| **Monitoring** | "How do I know my live app is healthy?" | Users report errors — Application Insights shows you the stack trace |
| **Caching** | "How do I make frequently accessed data faster?" | A product catalogue is loaded 10,000 times/sec — Redis caches it |

> ✅ **CHECKPOINT**  
> Look at the service map above. Pick any three services and, without looking at the descriptions, try to say in one sentence what each one does. If you can get at least two right, you're on track.

---

## Section 4: IaaS vs PaaS — The Developer Decision

### The Problem: "I Have Code — Where Should It Run?"

This is the first real decision a developer faces when moving to Azure. You've written your application. Now you need to put it somewhere people can use it. Azure gives you two main philosophies for hosting:

- **IaaS (Infrastructure as a Service):** Azure gives you a virtual machine. You install the operating system, the runtime, the web server, and your code. You manage updates, patches, and security.
- **PaaS (Platform as a Service):** Azure gives you a ready-to-use platform. You upload your code. Azure handles the operating system, the runtime, scaling, and patches.

### The Apartment Analogy (Deeper)

Think of deploying your application as moving into a new living space:

- **IaaS = Renting an empty flat.** You get four walls and a roof. You buy the furniture, install the kitchen, and fix the plumbing when it breaks. Maximum control, maximum responsibility.
- **PaaS = Renting a fully furnished flat.** The furniture is there, the kitchen works, and the landlord handles maintenance. You just move in your personal belongings (your code). Less control over the furniture arrangement, but you can focus on living your life (building your app).

### When Does a Developer Choose Each?

| Factor | Choose IaaS | Choose PaaS |
|---|---|---|
| **Operating system control** | Need a specific OS version or custom OS configuration | Don't care — any supported version is fine |
| **Software installation** | Need to install custom software (e.g., legacy middleware, specific drivers) | App runs on standard runtimes (.NET, Node.js, Python, Java) |
| **Migration strategy** | "Lift and shift" — moving an existing on-premises server to the cloud with minimal changes | Building new cloud-native applications or willing to adapt existing ones |
| **Maintenance tolerance** | Team has dedicated IT staff to patch and update servers | Want Azure to handle all infrastructure maintenance |
| **Scaling** | Willing to configure auto-scaling rules for VMs | Want managed autoscale or serverless scale-to-zero |
| **Cost model** | Willing to pay for VMs running 24/7 | Prefer pay-per-use or plan-based pricing |

### AZ-204 Focus: Mostly PaaS and Serverless

Here's the critical mental model for this certification:

```
AZ-204 Service Positioning:

  ◄── IaaS ──────────────── PaaS ──────────────── Serverless ──►
       │                      │                         │
    Virtual Machines     App Service              Azure Functions
    (NOT AZ-204)         Container Apps           Event Grid
                         Cosmos DB                Queue-triggered logic
                         Key Vault
                         API Management
```

Almost every service in AZ-204 sits in the **PaaS** or **serverless** column. You will *not* be asked to create, configure, or manage virtual machines on the AZ-204 exam.

> 🚨 **EXAM ALERT**  
> AZ-204 is NOT about virtual machines. If an exam question offers both a VM-based solution and a PaaS-based solution, the PaaS answer is almost always correct. VMs belong to the AZ-104 (Administrator) exam. AZ-204 is the developer exam — developers use managed platforms, not raw infrastructure.

### The Developer Decision Tree

When deciding where to host your code, use this simplified decision tree:

```
Do you need full OS-level control?
├── YES → Use a VM (IaaS) — but this is NOT an AZ-204 topic
└── NO → Does your code run continuously (web app, API)?
         ├── YES → Is it containerised?
         │         ├── YES → Azure Container Apps or ACI
         │         └── NO  → Azure App Service
         └── NO → Does it run in response to events?
                   ├── YES → Azure Functions (serverless)
                   └── NO  → Re-evaluate — most apps fit above
```

> 💡 **KEY CONCEPT**  
> As an AZ-204 developer, your default answer is PaaS. You choose App Service for web apps, Azure Functions for event-driven code, and Container Apps for containerised microservices. VMs are the last resort, and they are not part of this exam.

---

## Section 5: How Azure Charges You — Developer Perspective

### The Problem: "Will I Get a Surprise Bill?"

One of the biggest fears for new cloud developers is accidentally leaving a resource running and getting charged hundreds of dollars. Understanding Azure's pricing models removes that fear and helps you make smart decisions in labs and on the exam.

Think of Azure pricing like three different restaurants:

- **Pay-per-bite restaurant** — you pay only for each bite you eat. If you eat nothing, you pay nothing.
- **Buffet restaurant** — you pay a fixed price for unlimited access to a table. Whether you eat a little or a lot, the price is the same.
- **Per-dish restaurant** — each dish you order has a small individual price. The more dishes you order, the more you pay.

Azure services use one (or a combination) of these three models.

### Model 1: Consumption (Pay-Per-Use)

**Analogy:** The pay-per-bite restaurant. You are charged only when something actually happens.

| Service | What You Pay For | Free Tier |
|---|---|---|
| **Azure Functions** (Consumption plan) | Each execution + execution time (GB-seconds) | 1,000,000 executions/month free |
| **Azure Event Grid** | Each event delivered | 100,000 operations/month free |

**Developer impact:** These services can cost literally **$0.00** during labs if you stay within the free tier. They scale to zero — if nobody calls your function, you pay nothing.

**When to use:** Sporadic workloads, event-driven architectures, prototyping, labs.

### Model 2: Reserved / Allocated (Pay-for-Plan)

**Analogy:** The buffet restaurant. You choose a tier (table), and you pay for that tier every hour — whether your app handles 1 request or 1 million requests.

| Service | What You Pay For | Example |
|---|---|---|
| **Azure App Service Plan** | The plan tier (F1, B1, S1, P1v3), billed per hour | S1 plan ≈ $0.10/hour even if no traffic |
| **Azure Cosmos DB** (provisioned) | Reserved Request Units per second (RU/s) | 400 RU/s ≈ $23/month |
| **Azure Cache for Redis** | Cache tier and size, billed per hour | Basic C0 ≈ $0.022/hour |

**Developer impact:** These services cost money **as long as they exist**, even if nobody is using your app. This is why cleanup after labs is critical.

**When to use:** Predictable workloads, production applications, services that need to be "always on."

> 💰 **COST WARNING**  
> App Service Plans, Cosmos DB (provisioned), and Redis Cache charge you **every hour they exist** — even with zero traffic. If you forget to delete them after a lab, you will see charges on your next bill. Always delete lab resources after every session.

### Model 3: Per-Transaction (Pay-Per-Operation)

**Analogy:** The per-dish restaurant. Each individual operation (read, write, list, delete) has a tiny price.

| Service | What You Pay For | Cost Example |
|---|---|---|
| **Azure Blob Storage** | Per read/write/list/delete operation + storage per GB | $0.004 per 10,000 read operations |
| **Azure Queue Storage** | Per message operation (add, peek, get, delete) | $0.004 per 10,000 operations |
| **Azure Key Vault** | Per secret/key/certificate operation | $0.03 per 10,000 operations |

**Developer impact:** Individual operations are extremely cheap. You'd need millions of operations to see meaningful charges. During labs, these costs are typically fractions of a cent.

**When to use:** Data storage, message passing, secret management — services where the per-operation cost is negligible but scales linearly with usage.

### Free Tier Services for AZ-204 Students

These free tiers are generous enough to cover **all** your lab exercises without any charges:

| Service | Free Tier Allowance | Enough for Labs? |
|---|---|---|
| **Azure Functions** (Consumption) | 1,000,000 executions + 400,000 GB-seconds/month | ✅ More than enough |
| **Azure Cosmos DB** (serverless) | 1,000 RU/s + 25 GB storage (free account) | ✅ Plenty for lab data |
| **Azure Key Vault** | 10,000 operations/month (standard vault) | ✅ Labs use a few dozen operations |
| **Azure Event Grid** | 100,000 operations/month | ✅ Labs use a few hundred events |
| **Azure App Service** (F1 tier) | 1 GB storage, 60 CPU-minutes/day, shared infrastructure | ✅ For testing only (no custom domain, no SSL) |

> 📝 **NOTE**  
> The free tier limits above apply to the **Azure free account** (first 12 months) and the **Always Free** services. Some limits vary. Always check the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/) before creating resources outside of labs.

### The Golden Rule for Lab Cost Management

> 💰 **COST WARNING**  
> **Always delete lab resources after every session.** The fastest way is to delete the entire Resource Group (**resource group** = a container that holds related Azure resources together). One command removes everything inside it:
>
> ```bash
> az group delete --name my-lab-rg --yes --no-wait
> #  --name my-lab-rg    ← the name of the resource group to delete
> #  --yes               ← skip the "Are you sure?" confirmation prompt
> #  --no-wait           ← don't wait for deletion to finish; return immediately
> ```
>
> Verify it's gone:
> ```bash
> az group list --output table
> #  --output table      ← display results in a human-readable table format
> ```
>
> If your lab resource group no longer appears in the list, you're safe.

> ✅ **CHECKPOINT**  
> Without looking at the tables above, can you name the three Azure pricing models and give one service example for each? If you can, you've got this section down.

---

## Section 6: Management Groups — Completing the Hierarchy

### The Problem: "How Is Everything Organised?"

When you log into Azure and create resources, they don't float in a void. Every resource you create has an address — a specific place in Azure's organisational tree. Understanding this tree helps you navigate the portal, understand permissions, and avoid "access denied" errors in labs.

### The Filing Cabinet Analogy

Think of Azure's organisation like a company's filing system:

- **Management Group** = The filing cabinet itself — the highest-level container. A large company might have one cabinet for "Production" and another for "Development."
- **Subscription** = A drawer in the cabinet — a billing boundary. Each drawer gets its own bill. Your Azure free account is one subscription.
- **Resource Group** = A folder inside the drawer — a logical grouping of related resources. You might have a folder called "my-web-app-lab" that contains everything for one project.
- **Resource** = A single document inside the folder — an actual Azure service like an App Service, a Storage Account, or a Function App.

### The Four Levels — Visual Diagram

```
┌──────────────────────────────────────────────────────┐
│  MANAGEMENT GROUP                                     │
│  "The filing cabinet"                                 │
│  Organises multiple subscriptions.                    │
│  Policies and access applied here flow DOWN.          │
│                                                       │
│  ┌──────────────────────────────────────────────────┐ │
│  │  SUBSCRIPTION                                     │ │
│  │  "A drawer in the cabinet"                        │ │
│  │  Billing boundary — gets its own invoice.         │ │
│  │  Every resource belongs to exactly one.           │ │
│  │                                                    │ │
│  │  ┌──────────────────────────────────────────────┐ │ │
│  │  │  RESOURCE GROUP                               │ │ │
│  │  │  "A folder inside the drawer"                 │ │ │
│  │  │  Logical container for related resources.     │ │ │
│  │  │  Deleting the group deletes EVERYTHING in it. │ │ │
│  │  │                                                │ │ │
│  │  │  ┌──────────────────────────────────────────┐ │ │ │
│  │  │  │  RESOURCE                                │ │ │ │
│  │  │  │  "A document inside the folder"          │ │ │ │
│  │  │  │  The actual Azure service instance.      │ │ │ │
│  │  │  │  Examples: App Service, Storage Account, │ │ │ │
│  │  │  │  Function App, Key Vault, Cosmos DB      │ │ │ │
│  │  │  └──────────────────────────────────────────┘ │ │ │
│  │  └──────────────────────────────────────────────┘ │ │
│  └──────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

### What Each Level Does

| Level | Purpose | Key Fact |
|---|---|---|
| **Management Group** | Organise subscriptions into a governance hierarchy | Can be nested up to 6 levels deep |
| **Subscription** | Billing boundary and access control boundary | You get one with a free Azure account |
| **Resource Group** | Group related resources for a project, lab, or application | Deleting the group deletes **all** resources inside it |
| **Resource** | An individual Azure service instance | Must belong to exactly one resource group |

### RBAC: Permissions Flow Downward

**RBAC** (**Role-Based Access Control**) is Azure's permission system. It answers the question: "Who can do what to which resources?"

RBAC can be assigned at **any** of the four levels, and permissions **inherit downward**:

```
Management Group  ──  RBAC assigned here applies to ALL subscriptions below
    │
    ▼
Subscription      ──  RBAC assigned here applies to ALL resource groups below
    │
    ▼
Resource Group    ──  RBAC assigned here applies to ALL resources in the group
    │
    ▼
Resource          ──  RBAC assigned here applies to THIS resource only
```

**What this means for you as a student:**

- Your Azure free account gives you **Owner** role on your subscription. This means you can create, modify, and delete anything inside it.
- In a real company, you might only have **Contributor** access to a specific resource group — meaning you can create resources inside it but cannot change who else has access.
- If a lab tells you "access denied," it could be because RBAC is restricting you at the subscription, resource group, or resource level.

> 🚨 **EXAM ALERT**  
> RBAC is **inherited** — permissions set at a higher level flow down to all levels below. A role assigned at the subscription level applies to every resource group and resource within that subscription. You cannot "block" an inherited role at a lower level (you can only add more specific roles).

### The Three Most Important RBAC Roles for AZ-204

| Role | What It Can Do | When You'll See It |
|---|---|---|
| **Owner** | Full access + can assign roles to others | Your free account subscription |
| **Contributor** | Full access but cannot assign roles | Common in team environments |
| **Reader** | View resources only — no changes | Auditors, read-only dashboards |

> 📝 **NOTE**  
> RBAC is covered at awareness level here. You will encounter it again in Module 6 (Managed Identity) and Module 7 (Key Vault), where you'll learn how to assign specific roles like "Key Vault Secrets User" to your applications.

> 🔗 **MODULE LINK**  
> The management hierarchy you just learned is used in **every single lab**. In Module F02 (Azure Hierarchy, ARM, and Infrastructure as Code), you will practice creating resource groups via CLI and learn how Azure Resource Manager (ARM) processes every request you make.

---

## Self-Check Quiz

Test your understanding with these five questions. Try to answer each one **before** looking at the answers at the end of this document.

**Question 1:**  
A developer needs to host a web API that runs continuously and serves traffic 24/7. They don't need operating system-level access. Which Azure service category is the best fit — IaaS or PaaS — and which specific service would you recommend?

**Question 2:**  
You are studying for the AZ-204 exam. A practice question gives you two options: (A) Deploy the application on a Virtual Machine with a custom Nginx configuration, or (B) Deploy the application to Azure App Service. Which is more likely to be the correct answer, and why?

**Question 3:**  
Your lab created an Azure App Service on the S1 (Standard) tier. You finished the lab two hours ago but forgot to delete the resource group. Are you being charged? Explain why using the correct pricing model name.

**Question 4:**  
You assign the "Contributor" role to a user at the **Subscription** level. Can that user create resources inside a resource group within that subscription? Can they assign roles to other users?

**Question 5:**  
Name the six categories in the AZ-204 service map and give one example service from each category.

---

## Self-Check Quiz — Answers

**Answer 1:**  
**PaaS** is the best fit. The developer doesn't need OS-level access, so IaaS (virtual machines) adds unnecessary management burden. The recommended service is **Azure App Service** — it's a fully managed PaaS for hosting web apps and APIs with continuous uptime.

**Answer 2:**  
**(B) Azure App Service** is almost always the correct answer on AZ-204. The AZ-204 exam is the developer exam and focuses on PaaS and serverless solutions. Virtual machines belong to the AZ-104 (Administrator) exam. Unless the question explicitly requires OS-level control or custom software that only runs on a VM, choose the PaaS option.

**Answer 3:**  
**Yes, you are being charged.** App Service Plans use the **Reserved / Allocated (pay-for-plan)** pricing model. The S1 plan costs approximately $0.10 per hour regardless of whether your app is receiving traffic. Two hours of forgetting costs roughly $0.20 — small, but it adds up. This is why you should always delete lab resource groups when you're done.

**Answer 4:**  
**Yes**, the user can create resources inside resource groups within that subscription — RBAC permissions inherit downward, so Contributor access at the subscription level flows down to all resource groups and resources. **No**, the user cannot assign roles to others — the Contributor role grants full access to manage resources but does not include the permission to manage access (role assignments). Only the Owner role can assign roles.

**Answer 5:**  
The six categories and example services:
1. **Compute** — Azure App Service (or Functions, ACR, ACI, ACA)
2. **Storage** — Azure Blob Storage (or Cosmos DB, Queue Storage)
3. **Security** — Azure Key Vault (or Managed Identity, Entra ID, MSAL)
4. **Integration** — Azure Event Grid (or Event Hubs, Service Bus, API Management)
5. **Monitoring** — Application Insights (or Azure Monitor, Log Analytics)
6. **Caching** — Azure Cache for Redis (or CDN)

---

## What We Covered

Use this checklist to confirm you're ready to move to the next foundation document:

- [ ] **Cloud recap:** I can explain cloud computing, shared responsibility, and IaaS/PaaS/SaaS using the pizza analogy
- [ ] **Developer vs Admin:** I understand that AZ-204 is the developer perspective on Azure, not the infrastructure administrator perspective
- [ ] **Developer lifecycle:** I can recite the loop — code → build → package → deploy → monitor → repeat — and explain each stage
- [ ] **Service map:** I can name the six AZ-204 service categories (Compute, Storage, Security, Integration, Monitoring, Caching) and at least one service in each
- [ ] **IaaS vs PaaS:** I know that AZ-204 focuses on PaaS and serverless, and that VMs are NOT a major exam topic
- [ ] **Pricing models:** I can explain consumption (pay-per-use), reserved (pay-for-plan), and per-transaction pricing with an example service for each
- [ ] **Free tier:** I know the free tier limits for Functions (1M), Cosmos DB (1000 RU/25GB), Key Vault (10K ops), and Event Grid (100K ops)
- [ ] **Cost safety:** I know to always delete lab resource groups after every session using `az group delete`
- [ ] **Management hierarchy:** I can draw the four levels — Management Group → Subscription → Resource Group → Resource
- [ ] **RBAC:** I understand that RBAC can be assigned at any level and permissions inherit downward

---

**Next:** [F02 — Azure Hierarchy, ARM, and Infrastructure as Code →](../Layer_0_Foundation/F02_Azure_Hierarchy_ARM_IaC.md)
