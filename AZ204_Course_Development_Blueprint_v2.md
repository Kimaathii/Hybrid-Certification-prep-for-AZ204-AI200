# AZ-204: Developing Solutions for Microsoft Azure
# Complete Course Content Development Blueprint — Version 2.0

> **Purpose:** This is the master production brief for all AZ-204 certification course materials.
> Every section contains exact specifications: what to create, what to include, how to structure it,
> and what the student outcome must be. Use this as the complete instruction set for Copilot.
>
> **Version 2.0 changes from v1.0:**
> Foundation Pack expanded from 5 → 8 documents based on full AZ-900 → AZ-204 gap analysis.
> Three new foundation documents added: C# Basics, Azure Networking for Developers,
> Storage Account and SDK Basics. F02 and F03 significantly expanded.
> All module labs updated with mandatory cost callouts.
> Exam trap list expanded to 50. Flashcard bank expanded to 150. Total files updated to 79.

---

## MASTER OVERVIEW

| Item | Detail |
|---|---|
| **Certification** | Microsoft Certified: Azure Developer Associate |
| **Exam Code** | AZ-204 |
| **Exam Retirement** | July 31, 2026 |
| **Pass Score** | 700 / 1000 |
| **Exam Duration** | 120 minutes |
| **Question Count** | 40–60 questions |
| **Target Student** | Beginners from AZ-900, little or no development experience |
| **Course Structure** | 3 Layers: 8 Foundation Docs + 13 Module Packs + Exam Prep Pack |
| **Total Deliverables** | 79 documents |

---

## STUDENT PROFILE

**Who this student is:**
- Passed AZ-900 or has equivalent cloud awareness
- Little or no hands-on development experience
- Has never written or run code professionally
- Knows terms like API, JSON, HTTP conceptually but cannot use them confidently
- Has never used a terminal beyond 1–2 commands
- Has never deployed an application to any server or cloud

**What AZ-900 already gave them (acknowledge, do NOT re-teach at length):**
- Cloud computing, shared responsibility model
- IaaS, PaaS, SaaS concepts
- High availability, scalability, reliability
- Azure regions, availability zones, resource groups, subscriptions
- Portal awareness, Azure Monitor, Entra ID, RBAC (all awareness level only)
- ARM templates concept, cost management concept

**What the Foundation Pack must bridge (gaps between AZ-900 and AZ-204):**
- Hands-on CLI proficiency
- C# / programming basics — enough to read, run, debug lab code
- Git and source control for deployment labs
- JSON in depth — reading, writing, understanding
- HTTP and REST APIs in practice
- Environment variables — what and why Azure uses them
- VNet developer concepts — deeper than AZ-900
- ARM / IaC at operational level
- Storage account setup and SDK connection patterns
- .NET and Node.js project structure
- REST API testing (Postman / curl)
- Azure cost awareness per service in labs

**Design Principle:** Analogy before definition. Problem before solution. No undefined jargon. Ever.

---

## CONTENT ARCHITECTURE

```
LAYER 0 — Foundation Pack          8 documents  (Bridge AZ-900 → AZ-204 readiness)
LAYER 1 — 13 Module Packs         65 documents  (Complete AZ-204 syllabus)
LAYER 2 — Exam Preparation Pack    6 documents  (Strategy, tests, cheat sheets)
────────────────────────────────────────────────
TOTAL                              79 documents
```

---

## RECOMMENDED TIMELINE

```
WEEK 1      Layer 0: Foundation Pack
            Goal: comfortable in terminal, CLI, basic C#, Git, JSON, HTTP, Portal

WEEKS 2–6   Layer 1: Module Packs (one module every 2–3 days)
            Goal: understand all AZ-204 services, completed all labs hands-on

WEEK 7      Layer 2: Exam Prep Pack
            Goal: 80%+ on timed 50-question mock exam

WEEK 8      Review weak modules, book and sit exam
```


---
---

# LAYER 0: FOUNDATION PACK
## "Bridging AZ-900 to AZ-204 Readiness"

**Purpose:** Eliminate every hands-on and conceptual gap that blocks progress in Layer 1.
A student who completes all 8 Foundation documents can: open a terminal confidently,
run Azure CLI commands, read basic C# code, use Git, make HTTP requests, understand
environment variables, VNet concepts, and follow every Module lab without confusion.

**Copilot instruction:** AZ-900 topics = acknowledge briefly ("You know X from AZ-900 —
here we go deeper"), not re-taught at length. Focus on PRACTICAL and HANDS-ON dimensions.

---

### F01 — How the Cloud Works: From Concept to Practice
**Filename:** `F01_How_The_Cloud_Works.pdf`
**Pages:** 10–14 | **Study time:** 1.5 hours

**Sections:**
1. Quick AZ-900 Recap (2 pages, confidence-building): cloud, shared responsibility, IaaS/PaaS/SaaS — fast. Callout: "You know this — now going DEEPER."
2. Why Developers Care About Cloud Differently Than IT Admins: infrastructure view vs developer view. AZ-204 = the DEVELOPER perspective. Developer mental model: code → build → package → deploy → monitor → repeat.
3. The Azure Service Landscape for Developers: visual map of ALL services in AZ-204 grouped by category (Compute, Storage, Security, Integration, Monitoring, Caching). This map is the student's course roadmap — reference it in every module.
4. IaaS vs PaaS — The Developer Decision (Deeper): when does a developer CHOOSE each? AZ-204 focus = mostly PaaS and serverless. VMs are NOT a major AZ-204 topic. EXAM ALERT: AZ-204 is not about virtual machines.
5. How Azure Charges You — Developer Perspective: consumption model (Functions, Event Grid), reserved model (App Service Plan, Cosmos DB), per-transaction model (Blob, Queue). Always delete lab resources. Free tier services list relevant to AZ-204.
6. Management Groups — Completing the Hierarchy: Management Group → Subscription → Resource Group → Resource. RBAC roles can be assigned at any level. Diagram with all 4 levels.

**End:** 5-question self-check

---

### F02 — Azure Hierarchy, ARM, and Infrastructure as Code
**Filename:** `F02_Azure_Hierarchy_ARM_IaC.pdf`
**Pages:** 14–18 | **Study time:** 2 hours

**Sections:**
1. The Azure Hierarchy in Practice: not just theory — how you ACTUALLY use it in labs. Every lab: create RG first. Deleting RG deletes everything. Naming conventions: `{project}-{service}-{env}`. Using `$RANDOM` for uniqueness.
2. Azure Resource Manager (ARM) — What It Really Is: ARM is like the receptionist at Azure's front desk. Every request (portal, CLI, SDK, PowerShell) goes through ARM. ARM authenticates, validates, routes, returns result. ARM template structure: schema, contentVersion, parameters, variables, resources, outputs. Full annotated example (create storage account). Deploy command: `az deployment group create --resource-group myRG --template-file template.json`.
3. Bicep — The Modern Alternative: if ARM JSON is writing HTML by hand, Bicep is a word processor that generates it. Side-by-side comparison. Deploy: `az deployment group create --template-file main.bicep`. AZ-204 doesn't test Bicep deeply — just don't be confused by it.
4. Infrastructure as Code — Why It Matters: imperative (CLI commands = "do these steps") vs declarative (ARM/Bicep = "here is desired state"). EXAM ALERT: ARM = declarative, CLI = imperative.
5. Azure Tags — Governance in Practice: key-value metadata on resources. Cost allocation, filtering, automation. `az resource tag --tags Environment=dev`. EXAM ALERT: tags don't affect functionality, they are metadata only.
6. Azure Policy and Resource Locks: Policy = enforce standards (all resources need Environment tag). Locks = prevent deletion (CanNotDelete) or changes (ReadOnly). Developer context: if a lab fails to create a resource type, a Policy might be blocking it.
7. Cost Management in Practice: budget alert setup step-by-step. Cost analysis per resource group. Free service limits for AZ-204 services (Functions 1M/month, Cosmos DB 1000RU/25GB, Key Vault 10K ops, Event Grid 100K ops).

**End:** Hands-on exercise — deploy storage account via ARM template CLI, view deployment in portal under Resource Group → Deployments.

---

### F03 — Developer Basics: Terminal, Code, Git, and APIs
**Filename:** `F03_Developer_Basics_Terminal_Code_Git_APIs.pdf`
**Pages:** 22–28 | **Study time:** 3–4 hours | **MOST IMPORTANT foundation document**

**Section 1: The Terminal — Your New Best Friend**
Analogy: texting your computer. Opening terminal on Windows/Mac/Linux/Cloud Shell. Essential commands:
```
pwd / ls / dir / cd / mkdir / clear / cat / echo
```
Common beginner mistake: running command in wrong directory — always check `pwd` first.
Hands-on: navigate to Desktop, create `az204-labs` folder, create `module-01` subfolder.

**Section 2: The Azure CLI**
Why CLI over portal: faster, repeatable, scriptable. Installation for Windows/Mac/Linux. Verify: `az --version`. Login: `az login`.
Command structure pattern (teach this — every command follows it):
```
az  [service]   [action]    --parameter value   --parameter value
az   group      create      --name myRG          --location eastus
```
Key flags used in ALL AZ-204 labs:
```
--name / --resource-group / --location / --output table / --output json
--yes / --no-wait / --query
```
Common commands across all labs (reference table). Help: `az webapp --help`.
Hands-on: login, list subscriptions, create `az204-practice-rg`, list groups, delete, verify.

**Section 3: JSON — Reading and Writing Azure's Language**
Analogy: a form with labelled fields, structured so computers can read reliably.
Complete JSON anatomy with annotated example (object, array, string, number, boolean, null, nested object).
Reading Azure CLI JSON output. Using `--query` to extract fields. `--output table` for readability.
Common mistake: forgetting quotes around string values.
Hands-on: given a JSON blob, answer 5 questions about its contents.

**Section 4: Environment Variables — The Secret Plumbing**
Analogy: employee ID badge — you refer to it by name, not by value. Why NEVER hard-code secrets: GitHub exposure, change management, environment differences.
Setting locally (Mac/Linux `export`, Windows PowerShell `$env:`).
Reading in C#: `Environment.GetEnvironmentVariable("KEY")`.
Reading in Node.js: `process.env.KEY`.
How Azure App Settings = environment variables injected into your running app.
The `.env` file pattern — NEVER commit to Git.
EXAM ALERT: secrets → Key Vault, referenced from App Settings, read as environment variables.

**Section 5: HTTP and REST APIs — From Theory to Practice**
HTTP = the postal system for the internet. HTTP Methods with analogies:
```
GET    = "Give me something"
POST   = "Here is something new"
PUT    = "Replace this entirely"
PATCH  = "Update part of this"
DELETE = "Remove this"
```
Status codes to memorise: 200, 201, 204, 301, 400, 401, 403, 404, 409, 429, 500, 503.
REST API = standard way to design HTTP APIs. URLs identify resources, HTTP methods define action, JSON responses.
Headers: Content-Type, Authorization, Accept. Request body for POST/PUT.
Why this matters for AZ-204: every Azure SDK call is HTTP under the hood.

**Section 6: REST API Testing Tools — Postman and curl**
Postman (visual, for beginners): installation, interface walkthrough (method dropdown, URL bar, Headers, Body, Send, Response). First GET: `https://jsonplaceholder.typicode.com/users`. First POST with JSON body.
curl (CLI, used in all labs):
```bash
curl https://jsonplaceholder.typicode.com/users
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"Test","body":"Hello","userId":1}'
curl "https://myapp.azurewebsites.net/api/func?code=mykey"
```
Hands-on: GET request in Postman AND curl. Compare results.

**Section 7: Git and Source Control**
Analogy: time machine for code. Every save point (commit) is recoverable. Multiple people can work simultaneously.
Why AZ-204 students need Git: deployment labs deploy FROM GitHub. CI/CD triggered by Git push. All sample code is on GitHub.
Concepts: repository, commit, branch, remote, push, pull, clone.
Essential commands:
```bash
git --version
git config --global user.name "Name"
git config --global user.email "email"
git clone https://github.com/user/repo
git init / git status / git add . / git commit -m "msg"
git push / git pull / git log --oneline
```
GitHub account setup: create account, create repo, push local folder to it.
The `.gitignore` file — NEVER commit: `local.settings.json`, `.env`, `bin/`, `obj/`, `node_modules/`.
CRITICAL RULE: Never commit secrets to Git. Ever.
Hands-on: clone sample repo, make a change, commit, push to own GitHub.

**Section 8: .NET Project Structure**
Install .NET SDK. Verify: `dotnet --version` (8.0+).
Create projects: `dotnet new console`, `dotnet new webapi`, `dotnet new func`.
Project folder anatomy (every file labelled):
```
MyProject/
├── MyProject.csproj   ← project file: dependencies and settings
├── Program.cs         ← entry point: where code starts
├── bin/               ← compiled output (ignore)
└── obj/               ← build intermediates (ignore)
```
The `.csproj` file structure explained. Adding NuGet packages: `dotnet add package Azure.Storage.Blobs`.
Running: `dotnet build` (compile, check errors) vs `dotnet run` (build + execute).
Reading C# — annotated `Program.cs` with every element labelled:
`using` / `namespace` / `class` / `static async Task Main` / variables / objects / `Console.WriteLine` / `$""` / `//` comments.
async/await explained: ordering food analogy — you don't stand at the kitchen blocking everyone.
Reading a stack trace: first line = the actual error. Everything below = the call path.
Node.js alternative: `node --version`, `npm init -y`, `npm install @azure/storage-blob`, `node index.js`.
Hands-on: create .NET console project, add `Azure.Identity` package, print "Hello AZ-204", run.

**End:** 8-question self-check on all sections

---

### F04 — Azure Portal Navigation Guide
**Filename:** `F04_Azure_Portal_Navigation_Guide.pdf`
**Pages:** 12–15, screenshot-heavy | **Study time:** 1 hour

**Sections:**
1. Portal at a Glance: full annotated screenshot — top bar, left sidebar, main content. Customise dashboard.
2. Finding Any Service Instantly: search bar, All Services, Favouriting services.
3. Resource Groups as Your Daily Work Area: create, browse resources, Activity Log, Deployments tab, delete with confirmation.
4. Creating Resources — Universal Pattern: Create a resource → search → fill form → Review + Create → notifications bell → Go to Resource.
5. Cloud Shell: the `>_` icon, first-time storage account creation, Bash vs PowerShell, upload files, auto-logged-in.
6. Key Portal Sections for Every Module: App Service config/slots/scale, Function App keys, Key Vault secrets/networking, Storage containers/access keys, Cosmos DB Data Explorer, APIM APIs/policies.
7. Cost Management: Cost Analysis, filter by RG, budget setup step-by-step.
8. Entra ID: location, App registrations, Tenant ID — note this for labs.

**End:** Navigation speed drill — 10 tasks, students time themselves.

---

### F05 — Lab Zero: Your First Complete Azure Experience
**Filename:** `F05_Lab_Zero_First_Azure_Experience.pdf`
**Pages:** 12–15 | **Time:** 90 minutes | **Cost:** $0

**Objectives:** Create Azure account, navigate to Cloud Shell, log in via local CLI,
run essential commands, create RG via portal AND CLI, deploy simple static web app (F1 free tier),
view live in browser, delete RG and verify, set spending budget alert.

**Every step includes:** exact command or click path, expected output, explanation of what happened, ✅ CHECKPOINT every 3 steps.

**Troubleshooting section:**
- "az: command not found" → Azure CLI not installed
- "Please run az login" → run `az login`
- "Resource name already taken" → use `$RANDOM` suffix
- "AuthorizationFailed" → check subscription role
- "ResourceGroupNotFound" → typo in name, check with `az group list`

---

### F06 — C# Basics for AZ-204 Labs
**Filename:** `F06_CSharp_Basics_For_AZ204_Labs.pdf`
**Pages:** 18–22 | **Study time:** 3–4 hours

**Purpose:** Students do NOT need to be C# developers. They need to READ, UNDERSTAND,
modify, and RUN C# code in labs. This covers exactly and only what AZ-204 labs require.

**Section 1: Why C# for AZ-204?**
Microsoft built Azure AND .NET. Azure SDK is best supported in C#. Goal: not building apps from scratch — reading and running existing lab code.

**Section 2: C# File Anatomy**
Full annotated code file with every syntax element labelled:
`using` (import library), `namespace` (named container), `class` (blueprint),
`static async Task Main` (entry point), variables, object creation (`new`),
method calls, `await`, `Console.WriteLine`, `$""` (string interpolation), `//` (comment).

**Section 3: Data Types**
```csharp
string name = "Hello";    // Text
int count = 42;           // Whole number
bool isEnabled = true;    // True/false
double price = 9.99;      // Decimal
var client = new X();     // var = compiler infers type
```

**Section 4: Methods**
Named blocks of code. Calling methods. Async methods return `Task` or `Task<T>`. Always `await` them.

**Section 5: Async/Await — The Pattern You Will See Everywhere**
Ordering food analogy: you don't stand at the kitchen blocking everyone.
Rules: method using `await` must be marked `async`. Note the `Async` suffix naming pattern.
```csharp
// WRONG (blocks thread)
var container = client.CreateBlobContainer("name");

// RIGHT (async)
var container = await client.CreateBlobContainerAsync("name");
```

**Section 6: Exception Handling**
```csharp
try { var secret = await client.GetSecretAsync("name"); }
catch (Azure.RequestFailedException ex)
{ Console.WriteLine($"Error {ex.Status}: {ex.Message}"); }
```
Reading stack traces: first line = actual error.
Common Azure errors: 403 = permission denied, 404 = not found, CredentialUnavailableException = auth failed.

**Section 7: The Azure SDK Pattern**
Every Azure service SDK follows: (1) Create CLIENT → (2) Perform OPERATIONS → (3) Handle RESPONSE.
Two client creation patterns:
```csharp
// Simple (connection string — early labs)
BlobServiceClient c = new BlobServiceClient(connectionString);

// Production (DefaultAzureCredential — uses Managed Identity in Azure, az login locally)
BlobServiceClient c = new BlobServiceClient(new Uri("https://..."), new DefaultAzureCredential());
```
EXAM ALERT: `DefaultAzureCredential` = recommended for ALL production SDK authentication. Zero code changes between local and Azure.

**Section 8: SDK Packages Reference Table**

| Module | Package | Install Command |
|---|---|---|
| Blob Storage (M04) | Azure.Storage.Blobs | `dotnet add package Azure.Storage.Blobs` |
| Cosmos DB (M05) | Microsoft.Azure.Cosmos | `dotnet add package Microsoft.Azure.Cosmos` |
| Key Vault (M07) | Azure.Security.KeyVault.Secrets | `dotnet add package Azure.Security.KeyVault.Secrets` |
| Managed Identity | Azure.Identity | `dotnet add package Azure.Identity` |
| Service Bus (M10) | Azure.Messaging.ServiceBus | `dotnet add package Azure.Messaging.ServiceBus` |
| Event Hubs (M09) | Azure.Messaging.EventHubs | `dotnet add package Azure.Messaging.EventHubs` |
| App Config (M07) | Azure.Data.AppConfiguration | `dotnet add package Azure.Data.AppConfiguration` |
| Redis (M12) | StackExchange.Redis | `dotnet add package StackExchange.Redis` |

**End:** Given a complete C# lab file, students annotate every line. Then run with `dotnet run`.

---

### F07 — Azure Networking for Developers
**Filename:** `F07_Azure_Networking_For_Developers.pdf`
**Pages:** 14–18 | **Study time:** 2 hours

**Purpose:** AZ-204 is NOT a networking exam. But VNet Integration, Private Endpoints,
and Key Vault networking appear in labs and exam questions. This covers exactly what
developers need — no more, no less.

**Section 1: Why Developers Need Networking Knowledge**
Real scenarios: App Service needs to connect to private DB (connection fails — why?). Key Vault CLI command fails with "network access denied" (why?). These are real AZ-204 lab and exam scenarios.

**Section 2: Virtual Networks (VNet)**
Analogy: private office building. Resources inside talk freely. To get in from outside, go through reception (gateway/endpoint/firewall).
VNet = logically isolated Azure section. Address space (e.g. `10.0.0.0/16`). Subnets (subdivisions). Private IPs inside VNet.
EXAM ALERT: App Service and Functions on Consumption plan are NOT inside a VNet by default. They are on the public internet.

**Section 3: Network Security Groups (NSG)**
Firewall rules for subnet or network interface. Rules: priority, direction, source, destination, port, protocol, allow/deny.
Developer context: if app can't reach database VM, an NSG may be blocking port 1433.

**Section 4: VNet Integration for App Service and Functions**
Problem: App Service is on public internet. Database is in VNet. They can't communicate.
Solution: VNet Integration — allows OUTBOUND calls from app INTO a VNet.
EXAM ALERT (critical): VNet Integration is OUTBOUND ONLY. Does NOT place app inside VNet. Does NOT affect inbound traffic.
Requirements: App Service = Standard+. Functions = Premium or Dedicated (NOT Consumption).
Diagram: App Service (public) → VNet Integration → VNet → Private Database.

**Section 5: Private Endpoints**
Problem: Azure services (Key Vault, Storage, Cosmos DB) accessible on public internet by default. Need private-only access.
Solution: Private Endpoint = a private IP inside VNet that maps to the Azure service.
After creating: service gets a private IP. Traffic stays on private network. No public internet.
EXAM ALERT: Private Endpoint brings the SERVICE into the VNet. VNet Integration lets the APP reach OUT into the VNet. Opposite directions.

**Section 6: Service Endpoints**
Lighter alternative to Private Endpoints. Service still has public IP but traffic from VNet uses Azure backbone.
Use Private Endpoints for maximum security/compliance. Use Service Endpoints for good security at lower cost.

**Section 7: Public and Private Access to App Service**
Default: App Service URL is publicly accessible. Access restrictions: IP-based firewall rules.
App Service Environment (Isolated tier): App Service fully inside a VNet. Maximum isolation, highest cost.

**Section 8: Key Vault Networking — Common AZ-204 Scenario**
Default: Key Vault accessible from public internet (authenticated). Firewall settings: allow all, allow specific VNets/IPs, disable public (Private Endpoint only).
Adding local IP: Key Vault → Networking → Firewalls → Add your client IP.
Common lab error: "This operation is not permitted" = firewall blocking your IP. Add IP or allow all for lab.
EXAM ALERT: Key Vault networking + RBAC = TWO separate layers. Both required. Correct role AND network access.

**End:** Diagram exercise — draw correct network diagram for 3 scenarios, compare with answer diagrams.

---

### F08 — Azure Storage Account and SDK Basics
**Filename:** `F08_Storage_Account_And_SDK_Basics.pdf`
**Pages:** 14–16 | **Study time:** 2 hours

**Purpose:** Storage accounts are used in virtually every AZ-204 module. Functions need one.
Labs store data in them. Blob Storage IS one. Students must understand storage accounts before Module 4.

**Section 1: What Is an Azure Storage Account?**
Analogy: a building with multiple types of storage rooms inside. Rooms: Blob (files/media), Queue (messages), Table (structured data), File (shared file system). One account, multiple services.
Every Azure Function App requires a storage account for internal state.

**Section 2: Storage Account Types**
General Purpose v2 (GPv2) = default choice for all labs. Premium Block Blobs = high-performance. Use GPv2 unless instructed otherwise.

**Section 3: Creating via CLI**
```bash
az storage account create \
  --name az204storage$RANDOM \
  --resource-group az204-rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2
```

**Section 4: Redundancy Options — The Exam Table**

| SKU | Name | Protects Against | Copies |
|---|---|---|---|
| LRS | Locally Redundant | Single disk failure | 3, 1 datacenter |
| ZRS | Zone-Redundant | Datacenter failure | 3, 3 zones same region |
| GRS | Geo-Redundant | Regional disaster | 6, 2 regions |
| GZRS | Geo-Zone-Redundant | Zone + Regional | 6, 3 zones + 2 regions |

EXAM ALERT: GRS replicates to secondary region but secondary is READ-ONLY unless you enable RA-GRS.

**Section 5: Access Keys and Connection Strings**
Two access keys = master passwords. Find in: Storage Account → Security + Networking → Access keys.
Connection string format. Why two keys: rotate one without downtime.
SECURITY: access keys = full account access. NEVER in code. Use env vars or Key Vault references.

**Section 6: SAS Tokens at a Glance (full detail in M04)**
Service SAS / Account SAS / User Delegation SAS. Parameters: permissions, start/expiry time, IP, protocol.
User Delegation SAS = most secure (uses Entra ID credentials, not account key).

**Section 7: Azure Storage Explorer**
Free desktop app: storageexplorer.com. Browse containers, upload/download, create queues.
Use during labs to verify code worked. Sign in with Azure account or connection string.

**Section 8: AzCopy**
CLI tool for fast bulk transfers. Common commands for copy local→blob, folder→container, account→account.
EXAM ALERT: AzCopy = recommended tool for bulk data migration to Azure Storage.

**End:** Hands-on — create storage account, find connection string, connect with Storage Explorer,
create container, upload file, verify via CLI `az storage blob list`.


---
---

# LAYER 1: THE 13 MODULE PACKS
## AZ-204 Official Syllabus — Complete Content Specifications

### EXAM WEIGHT DISTRIBUTION

| Domain | Weight | Modules |
|---|---|---|
| Develop Azure Compute Solutions | 25–30% | M01, M02, M03 |
| Develop for Azure Storage | 15–20% | M04, M05 |
| Implement Azure Security | 15–20% | M06, M07 |
| Monitor, Troubleshoot, Optimize | 10–15% | M11 |
| Connect to and Consume Azure Services | 20–25% | M08, M09, M10, M12, M13 |

---

## UNIVERSAL STANDARDS — APPLY TO ALL 13 MODULE PACKS

### Concept PDF Standards
- Cover page: Module number, title, exam domain, weight %, estimated study time, prerequisites
- Every term defined on first use
- Analogy BEFORE technical definition — always
- 🚨 EXAM ALERT boxes (red): exam traps and critical must-know facts
- 💡 KEY CONCEPT boxes (blue): most important takeaway per section
- ✅ CHECKPOINT boxes (green): mid-document comprehension pause
- 🔗 MODULE CONNECTIONS box at end: which other modules this connects to
- End with: "What We Covered" summary checklist

### Slide Deck Standards (15–25 slides per module)
- Slide 1: title, exam domain, weight, learning objectives
- Slide 2: "The Problem This Service Solves" — why before what
- One concept per slide — diagram preferred over text bullets
- Colours: Azure blue #0078D4, white, light grey
- ⚠️ EXAM ALERT slides: red background #D13438, white text
- Final slide: Module Summary Table (all key facts in one table)
- Speaker notes on EVERY slide: the complete instructor script

### Lab Guide Standards — MANDATORY ELEMENTS
- Header table: Module, Lab title, Prerequisites, Estimated time, Cost estimate, What you build, What you learn
- 💰 COST CALLOUT (required on every lab): "This lab uses [service on X tier]. Estimated cost: $X if cleaned up within Y hours."
- Every command in a fenced code block
- Every flag explained inline: `--sku S1  ← Standard tier, minimum for deployment slots`
- Expected output shown after key commands
- ✅ CHECKPOINT every 3–5 steps
- TROUBLESHOOTING section: top 5 errors for this specific lab + fixes
- CLEANUP section always last: exact commands, confirm with `az group list`

### Scenario Cards Standards (4–6 per module)
```
SCENARIO TITLE
Business Context: who the company is, what they do
The Problem: what they need to solve
Constraints: 2–3 requirements that must ALL be satisfied
Your Task: what the student must decide/design/configure
Hints: 1–2 nudges without giving the answer
Solution: full explanation with reasoning
Exam Connection: how this appears as an exam question
```
Difficulty: 1 Beginner + 2–3 Intermediate + 1–2 Advanced per module

### Quiz Standards (15 questions per module)
- 5 Foundation + 5 Applied + 5 Exam-level
- Every wrong answer: explanation of why it is wrong
- Every right answer: explanation of why it is correct
- Microsoft format: scenario first → question → 4 options
- At least 2 "select all that apply" questions per module

---

## MODULE 1 — Azure App Service Web Apps
**Domain:** Develop Azure Compute Solutions (25–30%)
**Path:** `az-204-create-azure-app-service-web-apps`
**Prerequisites:** F01–F05

### Concept PDF — `M01_Concept_Azure_App_Service.pdf` — 20–25 pages

**Section 1: What Problem Does App Service Solve?**
Managing servers is not a developer's job. Analogy: App Service is a managed hotel for your app. You check in your code. Azure handles the building (servers, OS, patches). You only manage the app.
Definition: fully managed PaaS for web apps, REST APIs, mobile backends. Supported runtimes: .NET, Node.js, Python, Java, PHP, Ruby. Windows and Linux hosting.

**Section 2: App Service Plans — The Foundation**
Analogy: the Plan = the hardware, the App = your software. Multiple apps can share one plan.

| Tier | SKUs | Key Features | Limitations |
|---|---|---|---|
| Free | F1 | Shared CPU, 1GB storage | No custom domain, no SSL, no scale, no slots |
| Shared | D1 | Custom domain | No SSL, no scale, no slots |
| Basic | B1–B3 | Dedicated VMs, custom domain, SSL | No autoscale, NO slots |
| Standard | S1–S3 | Autoscale, 5 deployment slots, VNet | — |
| Premium v3 | P1v3–P3v3 | Faster, 20 slots, VNet | — |
| Isolated v2 | I1v2–I3v2 | Dedicated VNet (ASE) | Highest cost |

🚨 EXAM ALERT: Slots require Standard+. Autoscale requires Standard+. VNet Integration requires Standard+.

**Section 3: Deployment Methods**
ZIP deploy (`az webapp deployment source config-zip`), Local Git, GitHub Actions, Azure DevOps, Docker (ACR), FTP (legacy). Know the CLI command for ZIP deploy.

**Section 4: Deployment Slots**
Analogy: rehearsal stage and live performance stage. Staging slot → test → swap → zero downtime. Swap-back = instant rollback.
Sticky settings (stay with slot, do NOT travel): connection strings and app settings marked as slot setting.
Non-sticky settings (DO swap with code): everything else.
Warm-up: App Service pings staging until it responds before completing swap.
🚨 EXAM ALERT: sticky settings stay with the SLOT, not the app. Production keeps its connection string after swap.

**Section 5: Autoscaling**
Scale up (vertical): change plan tier — more CPU/RAM. Manual. Brief restart.
Scale out (horizontal): add more VM instances. THIS is autoscale.
Rule components: Metric (CPU %, HTTP queue), Threshold (e.g. 70%), Duration (5 min), Action (+1 instance), Cooldown (wait period).
Metric-based OR schedule-based rules (or both).
🚨 EXAM ALERT: autoscale adds IDENTICAL instances. It does not load balance different apps.

**Section 6: App Service Configuration**
Always On (keeps warm, required for WebJobs), ARR Affinity (sticky sessions — turn OFF for stateless apps), HTTPS Only, TLS minimum 1.2, Custom domains, SSL/TLS certificates (App Service managed = free, auto-renews), App Settings, Connection Strings, Slot settings.

**Section 7: Easy Auth**
Built-in auth without writing code. Providers: Entra ID, Facebook, Google, Twitter, OpenID Connect.
App Service intercepts every request, validates token before your code runs. Token store cached automatically.
🚨 EXAM ALERT: Easy Auth is configured at App Service level, not inside application code.

**Section 8: Networking**
VNet Integration: OUTBOUND only, Standard+. Hybrid Connections: on-premises TCP. Access Restrictions: inbound IP firewall.
🚨 EXAM ALERT: VNet Integration is OUTBOUND ONLY. Does not restrict inbound traffic.

### Slide Deck — `M01_Slides_Azure_App_Service.pptx` — 22 slides
Title → Objectives → Hotel analogy → PaaS vs IaaS → Runtimes → Plan tier table → EXAM ALERT (slots/autoscale thresholds) → Deployment methods flow → Slots diagram → Swap steps → Sticky settings table → EXAM ALERT (sticky stays with slot) → Scale up vs out → Autoscale rule components → Config key settings → Easy Auth flow → Networking VNet diagram → EXAM ALERT (outbound only) → Top 3 traps → Summary table → Checkpoint → Answers

### Lab Guide — `M01_Lab_Azure_App_Service.pdf`
💰 COST: S1 Plan ≈ $0.10/hr. Clean up within 2 hours = under $0.25.
Parts: (1) Create RG + S1 Linux Plan (2) Create Node.js web app (3) Deploy from GitHub sample (4) Add App Settings + Connection String (5) Create staging slot (6) Deploy different version to staging (7) Swap staging→production (8) Verify swap (9) Rollback swap-back (10) Enable autoscale rule (CPU>70%→+1 instance) (11) Clean up

### Scenario Cards — `M01_Scenarios_Azure_App_Service.pdf`
(1-Beginner) Startup needs custom domain, low budget — choose tier and explain limitations
(2-Intermediate) E-commerce needs zero-downtime weekly deploys with instant rollback
(3-Intermediate) Retail site spikes 500→10,000 users on weekends — design autoscale
(4-Advanced) Healthcare app needs to connect to on-premises database — design networking
(5-Advanced) Multi-region app — what App Service features apply

### Quiz — `M01_Quiz_Azure_App_Service.pdf`
15 questions: minimum tier for slots, autoscale requirements, sticky settings during swap, VNet Integration direction, Easy Auth provider config, scale out vs up, ARR Affinity, Always On, slot URL format, TLS settings, deployment method selection.

---

## MODULE 2 — Azure Functions
**Domain:** Develop Azure Compute Solutions (25–30%)
**Path:** `implement-azure-functions`
**Prerequisites:** F01–F06, M01

### Concept PDF — `M02_Concept_Azure_Functions.pdf` — 20–25 pages

**Section 1: What Is Serverless?**
Light switch analogy: electricity only flows when you flip it. Functions only run (and charge) when triggered. Write focused code, declare what triggers it, Azure handles everything else.

**Section 2: Function Anatomy**
Every function = exactly ONE trigger + optional input bindings + optional output bindings.
Bindings analogy: automatic conveyor belts that bring ingredients (input) and take finished products away (output). You just write the logic.

**Complete Trigger Reference Table:**
| Trigger | When It Fires | Exam Scenario |
|---|---|---|
| HTTP | Incoming HTTP request | Simple API or webhook |
| Timer | Cron schedule | Nightly cleanup, reports |
| Blob Storage | New/updated blob | Process uploaded files |
| Queue Storage | New queue message | Async task processing |
| Service Bus | Queue/topic message | Enterprise messaging |
| Cosmos DB | Change feed event | Sync data, downstream actions |
| Event Grid | Event published | React to Azure resource events |
| Event Hub | Stream events arrive | IoT, telemetry, high-volume |

🚨 EXAM ALERT: ONE trigger per function. Always.

**Section 3: Timer Trigger Cron Syntax**
6-part NCRONTAB: `{second} {minute} {hour} {day} {month} {day-of-week}`
Examples: `"0 */5 * * * *"` (every 5 min) / `"0 0 9 * * 1-5"` (9am Mon–Fri) / `"0 30 8 * * *"` (8:30am daily)
🚨 EXAM ALERT: Azure uses 6-part (second first). Standard Linux cron is 5-part. Do NOT mix up.

**Section 4: Hosting Plans**
| Plan | Scaling | Cold Start | Max Timeout | VNet | Use When |
|---|---|---|---|---|---|
| Consumption | Auto to zero | Yes | 10 min | ❌ | Cost-sensitive, sporadic |
| Flex Consumption | Auto, faster | Minimal | 30 min | ✅ | Better cold start |
| Premium | Pre-warmed | No | Unlimited | ✅ | No cold starts needed |
| Dedicated | Manual/auto | No | Unlimited | ✅ | Predictable load |
| Container Apps | Auto | Minimal | Unlimited | ✅ | Microservices |

🚨 EXAM ALERTS: Consumption max = 10 min. VNet = Premium or Dedicated only. No cold start = Premium.

**Section 5: Durable Functions — 4 Patterns**
Problem: stateful, long-running orchestrations in serverless. Three roles: Orchestrator, Activity, Client.

Pattern 1 — Function Chaining: sequential, output feeds next. `F1 → F2 → F3 → F4`
Use case: validate order → charge payment → send receipt → update inventory.

Pattern 2 — Fan-out/Fan-in: parallel, aggregate results. `Orchestrator → F2/F3/F4 (parallel) → aggregate`
Use case: resize image into 5 sizes simultaneously.

Pattern 3 — Async HTTP API: long job with status polling. `Client → start → get status URL → poll → result`
Use case: video encoding taking minutes — client polls for completion.

Pattern 4 — Monitor: flexible polling until condition met. `check → not done → wait → check → done → exit`
Use case: poll external API until it comes back online.

🚨 EXAM ALERT: Questions DESCRIBE a scenario — you must identify the pattern by description, not name alone.

**Section 6: Function Configuration**
`local.settings.json`: local secrets/config, NEVER commit to Git. `AzureWebJobsStorage`: required storage account. `FUNCTIONS_WORKER_RUNTIME`: dotnet/node/python/java. AuthorizationLevel: Anonymous / Function / Admin.

**Section 7: Bindings in Code**
Annotated C# example: HTTP trigger + Queue output binding. The `[Queue("orders")]` attribute = zero SDK boilerplate — Azure wires it up.

### Slide Deck — `M02_Slides_Azure_Functions.pptx` — 20 slides
Title → Objectives → Light switch analogy → Function anatomy diagram → Triggers table → Cron examples → EXAM ALERT (6-part, one trigger) → Hosting plans table → EXAM ALERT (Consumption limits, VNet) → Durable intro → Pattern 1 diagram → Pattern 2 diagram → Pattern 3 diagram → Pattern 4 diagram → EXAM ALERT (pattern identification) → Bindings code example → Auth levels → Summary table → Checkpoint → Answers

### Lab Guide — `M02_Lab_Azure_Functions.pdf`
💰 COST: Consumption plan ≈ free (1M executions/month free). Storage ≈ $0.01/month.
Parts: (1) Install Azure Functions Core Tools, verify (2) Create Function App project (`func init`) (3) Create HTTP trigger (`func new`) (4) Add Queue output binding to HTTP function (5) Create Queue trigger function as consumer (6) Run locally `func start`, test with curl (7) Observe queue message consumed by second function (8) Create Azure resources: storage + Function App (9) Deploy: `func azure functionapp publish` (10) Test live endpoint (11) View logs in portal (12) Clean up

### Scenario Cards — `M02_Scenarios_Azure_Functions.pdf`
(1-Beginner) Send welcome email 5 min after signup — which trigger and cron?
(2-Beginner) DB cleanup every night midnight — trigger and cron expression?
(3-Intermediate) Resize image into 4 sizes simultaneously — which Durable pattern?
(4-Intermediate) Video encoding takes 20 min on Consumption — problem and solution?
(5-Advanced) Validate → charge → receipt → inventory (all or nothing) — which Durable pattern and why?

### Quiz — `M02_Quiz_Azure_Functions.pdf`
15 questions: trigger types, cron syntax, Consumption timeout limit, Durable pattern identification, Premium vs Consumption selection, binding direction, AuthorizationLevel, local.settings.json purpose, VNet requirement.

---

## MODULE 3 — Containerised Solutions (ACR, ACI, ACA)
**Domain:** Develop Azure Compute Solutions (25–30%)
**Path:** `az-204-implement-iaas-solutions`
**Prerequisites:** F01–F06, M01–M02

### Concept PDF — `M03_Concept_Containers.pdf` — 22–28 pages

**Section 1: What Is a Container?**
Shipping container analogy: standardised box that runs identically everywhere. Docker: image (blueprint) vs container (running instance). Dockerfile (recipe). Registry (warehouse of images).
Container vs VM: containers share host OS kernel (lighter, faster). VMs have their own OS (heavier, more isolated).

**Section 2: Azure Container Registry (ACR)**
Azure's private Docker registry. Why private: security, compliance, Azure integration.
Tiers: Basic/Standard/Premium (storage and throughput differences).
Key ops: create ACR, build/push image, enable admin user (simple auth), use managed identity (production).
ACR Tasks: `az acr build` — build image in the cloud without Docker installed locally.
🚨 EXAM ALERT: `az acr build` = cloud build. No local Docker needed.

**Section 3: Azure Container Instances (ACI)**
Fastest way to run a container in Azure. No infrastructure management.
Restart policies: Always (web servers), OnFailure (batch with retry), Never (one-time tasks).
Container Groups: multiple containers sharing network and storage (sidecar pattern).
🚨 EXAM ALERT: ACI = single containers or small groups. Not for production microservices at scale.

**Section 4: Azure Container Apps (ACA)**
Serverless containers on Kubernetes — Kubernetes power without managing Kubernetes.
Key concepts: Environment (shared VNet/logging boundary), Revision (immutable snapshot = like a deployment slot), Ingress (HTTP traffic control), KEDA scaling (HTTP/queue/CPU/custom), DAPR integration.
Traffic splitting between revisions: e.g. 80% to v1, 20% to v2 — percentage-based blue/green.
🚨 EXAM ALERT: ACA hides Kubernetes. You don't manage nodes, pods, or cluster config.

**Section 5: Decision Table**
| Scenario | Use |
|---|---|
| Store and manage container images | ACR |
| Run a container quickly, short-lived task | ACI |
| Microservices in production | ACA |
| Kubernetes without managing it | ACA |
| One-off batch job in a container | ACI |
| Private registry integrated with pipelines | ACR |

### Lab Guide — `M03_Lab_Containers.pdf`
💰 COST: ACR Basic ≈ $0.17/day. ACI ≈ $0.0025/vCPU/hr. ACA has free tier. Under $0.10 total within 3 hrs.
Parts: (1) Create ACR (2) Write simple Dockerfile (3) Build image via ACR Tasks `az acr build` (4) Verify image in portal (5) Deploy from ACR to ACI (6) Access via public IP (7) Create ACA environment (8) Deploy to ACA (9) Configure HTTP ingress (10) Create second revision, configure 80/20 traffic split (11) Clean up

---

## MODULE 4 — Blob Storage
**Domain:** Develop for Azure Storage (15–20%)
**Path:** `az-204-develop-solutions-that-use-blob-storage`
**Prerequisites:** F01–F08, M01–M03

### Concept PDF — `M04_Concept_Blob_Storage.pdf` — 18–22 pages

Sections: (1) What is Blob Storage — object storage analogy, three blob types: Block (files/media), Append (logs, write-only append), Page (VMs, random read/write). (2) Hierarchy: Storage Account → Container → Blob. (3) Access tiers — Hot/Cool/Cold/Archive with: access frequency, minimum storage duration, retrieval cost, storage cost. (4) Lifecycle management policies — automate tier transitions, JSON policy format, rules with conditions and actions. Policy runs ONCE DAILY — not real-time. (5) Access control — container public access levels, RBAC, SAS tokens. (6) SAS deep dive — Service/Account/User Delegation SAS, parameters (permissions/start/expiry/IP/protocol). User Delegation SAS = most secure. (7) SDK operations — BlobServiceClient, BlobContainerClient, BlobClient — upload/download/list/delete/metadata. (8) Static website hosting — enable on account, `$web` container. (9) Redundancy — LRS/ZRS/GRS/GZRS, RA-GRS explained.

🚨 EXAM ALERTS: Archive requires rehydration before reading (up to 15 hrs standard, 1 hr high-priority). Early deletion penalty for Cool/Cold/Archive. GRS secondary = read-only unless RA-GRS enabled.

### Lab Guide — `M04_Lab_Blob_Storage.pdf`
💰 COST: GPv2 ≈ $0.018/GB/month + minimal transactions. Lab under $0.01.
Parts: (1) Create GPv2 storage account (2) Create container (private access) (3) Upload blobs via SDK (4) Set blob metadata via SDK (5) List blobs via SDK (6) Download blob via SDK (7) Generate SAS token via CLI (8) Access blob via SAS URL in browser (9) Create lifecycle management policy (10) Configure static website (11) Upload HTML, access via website URL (12) Clean up

---

## MODULE 5 — Azure Cosmos DB
**Domain:** Develop for Azure Storage (15–20%)
**Path:** `az-204-develop-solutions-that-use-azure-cosmos-db`
**Prerequisites:** F01–F08, M01–M04

### Concept PDF — `M05_Concept_Cosmos_DB.pdf` — 20–25 pages

Sections: (1) What is Cosmos DB — globally distributed, multi-model NoSQL. Warehouse analogy. (2) APIs — Core (SQL), MongoDB, Cassandra, Gremlin, Table. AZ-204 focus = Core (SQL). (3) Hierarchy — Account → Database → Container → Item. (4) Partition key — THE most important concept. High cardinality, evenly distributed, commonly queried. Cannot change after creation. (5) Request Units (RUs) — billing/performance currency. Every operation costs RUs. (6) Consistency levels — memorise all 5:

| Level | Description | Latency | Use When |
|---|---|---|---|
| Strong | Always latest | Highest | Financial transactions |
| Bounded Staleness | Lag by K versions or T seconds | High | Near-real-time |
| Session | Consistent per session | Medium | DEFAULT — most apps |
| Consistent Prefix | No out-of-order reads | Low-medium | Social feeds |
| Eventual | May read stale | Lowest | High availability priority |

(7) SDK operations — CosmosClient, Database, Container, Item CRUD + SQL query. (8) Change feed — stream of inserts+updates in order. Trigger Functions, sync data. Does NOT include deletes. (9) Indexing — all properties indexed by default, custom policy to reduce RU cost. (10) Stored procedures — scoped to single partition key value.

🚨 EXAM ALERTS: Session consistency = DEFAULT. Change feed does NOT include deletes. Partition key CANNOT change after creation.

### Lab Guide — `M05_Lab_Cosmos_DB.pdf`
💰 COST: Free tier = 1000 RU/s + 25GB (one account per subscription). Cost = $0.
Parts: (1) Create Cosmos DB account (Core SQL, free tier) (2) Create database + container with partition key (3) Insert items via Data Explorer (4) Insert via .NET SDK (5) Read item by ID + partition key (6) Query with SQL via SDK (7) Update item (8) Delete item (9) Create Function with Cosmos DB change feed trigger (10) Insert item, observe Function trigger (11) Clean up

---

## MODULE 6 — User Authentication and Authorization
**Domain:** Implement Azure Security (15–20%)
**Path:** `az-204-implement-authentication-authorization`
**Prerequisites:** F01–F08, M01–M05

### Concept PDF — `M06_Concept_Auth_AuthZ.pdf` — 20–24 pages

Sections: (1) AuthN vs AuthZ — hotel check-in analogy. (2) Microsoft Identity Platform — Entra ID + MSAL + App Registrations umbrella. (3) OAuth 2.0 flows:

| Flow | Use When | User Present? |
|---|---|---|
| Authorization Code | Web app, browser sign-in | Yes |
| Client Credentials | Service-to-service, no user | No |
| Device Code | No browser (CLI/IoT) | Yes (other device) |
| PKCE | SPA/mobile (prevents code interception) | Yes |

🚨 EXAM ALERT: Client Credentials = no user context. Device Code = no browser. Choose by whether a human + browser are present.

(4) OpenID Connect — built on OAuth 2.0. Adds ID token (who you are) + access token (what you can do). (5) Tokens — ID token (identity claims), Access token (permissions), Refresh token (renew silently). JWT structure: header.payload.signature. (6) App Registration — Application ID, Tenant ID, client secret, redirect URIs, API permissions. Step-by-step in portal. (7) MSAL — PublicClientApplication (interactive), ConfidentialClientApplication (server/service). AcquireTokenInteractive / AcquireTokenSilent (try first) / AcquireTokenForClient. Token caching = automatic. (8) Microsoft Graph API — `https://graph.microsoft.com/v1.0/`. Common: GET /me, GET /users, POST /me/sendMail. Permissions must be declared in App Registration.

### Lab Guide — `M06_Lab_Auth_AuthZ.pdf`
💰 COST: Entra ID basic use = free. No compute resources. Cost = $0.
Parts: (1) Register app in Entra ID (2) Note tenant + client ID (3) Create client secret (4) Configure Graph API permissions (User.Read) (5) Write .NET app with MSAL Device Code flow (6) Call GET /me, print user profile (7) Add User.ReadBasic.All, call GET /users (8) Demonstrate token caching (second call instant) (9) Clean up app registration

---

## MODULE 7 — Secure Cloud Solutions
**Domain:** Implement Azure Security (15–20%)
**Path:** `az-204-implement-secure-cloud-solutions`
**Prerequisites:** F01–F08, M01–M06

### Concept PDF — `M07_Concept_Secure_Solutions.pdf` — 20–24 pages

Sections: (1) The security problem — hard-coded secrets = breaches. Three pillars: Key Vault + Managed Identity + App Configuration. (2) Azure Key Vault — Secrets/Keys/Certificates. RBAC vs Access Policies (RBAC = modern, recommended). Key RBAC roles: Secrets User (read), Secrets Officer (manage), Administrator (full). SDK ops: SetSecretAsync, GetSecretAsync, DeleteSecretAsync. Soft delete (7–90 day retention). Purge protection. Key Vault reference syntax in App Settings: `@Microsoft.KeyVault(SecretUri=https://vault.../secrets/Name/)`. 🚨 EXAM ALERT: RBAC role + network access = TWO separate layers. Both required.

(3) Managed Identities — employee badge analogy. System-assigned (tied to resource, deleted with it) vs User-assigned (standalone, assignable to multiple resources). How to use: enable → assign RBAC role on target → use DefaultAzureCredential. DefaultAzureCredential chain: Managed Identity → Visual Studio → Azure CLI → Env vars → Browser. 🚨 EXAM ALERT: DefaultAzureCredential recommended for ALL production SDK auth.

(4) Azure App Configuration — centralised key-value store for settings + feature flags. NOT for secrets. Feature flags: toggle without redeployment. Labels for environment grouping (dev/staging/prod).

(5) RBAC deep dive — Principal (User/Group/Service Principal/Managed Identity) + Role Definition + Scope. Principle of least privilege. Key built-in roles for AZ-204: Storage Blob Data Contributor, Key Vault Secrets User, Cosmos DB Account Reader, Service Bus Data Sender/Receiver.

### Lab Guide — `M07_Lab_Secure_Solutions.pdf`
💰 COST: Key Vault ≈ $0.03/10K ops. App Configuration = free tier. Lab under $0.05.
Parts: (1) Create Key Vault (2) Add local IP to Key Vault firewall (3) Create secret via CLI (4) Retrieve via .NET SDK + DefaultAzureCredential (az login) (5) Create App Service (6) Enable system-assigned Managed Identity (7) Assign Key Vault Secrets User role to Managed Identity (8) Add Key Vault reference as App Setting (9) Verify app reads secret at runtime (10) Create App Configuration store (11) Add settings + feature flag (12) Connect .NET app to App Configuration (13) Toggle feature flag — observe change without redeployment (14) Clean up

---

## MODULE 8 — API Management (APIM)
**Domain:** Connect to and Consume Azure Services (20–25%)
**Path:** `az-204-implement-api-management`
**Prerequisites:** F01–F08, M01–M07

### Concept PDF — `M08_Concept_API_Management.pdf` — 18–22 pages

Sections: (1) What problem APIM solves — hotel concierge analogy. Without APIM: no security, no rate limiting, no analytics. (2) Architecture — Gateway (processes requests, applies policies, forwards to backend) + Management Plane (config) + Developer Portal (auto-generated docs site). (3) Products and Subscriptions — Product = package of APIs. Subscription key sent with every request. (4) Importing APIs — OpenAPI/Swagger, WSDL, App Service, Function App, manual. (5) Policies — the heart of APIM:

Policy structure: `<inbound>` → `<backend>` → `<outbound>` → `<on-error>`

Key policies to know:
```xml
<!-- Rate limit: 10 calls per 60 seconds per subscription key -->
<rate-limit calls="10" renewal-period="60" />

<!-- Quota: 1000 calls per week total -->
<quota calls="1000" renewal-period="604800" />

<!-- IP filter -->
<ip-filter action="allow"><address-range from="X" to="Y" /></ip-filter>

<!-- Cache response for 5 minutes -->
<cache-lookup /><cache-store duration="300" />

<!-- Mock response WITHOUT calling backend — goes in INBOUND -->
<mock-response status-code="200" content-type="application/json" />

<!-- Validate OAuth token -->
<validate-jwt header-name="Authorization" failed-validation-httpcode="401">
  <openid-config url="..." />
</validate-jwt>

<!-- Set header -->
<set-header name="X-Custom" exists-action="override"><value>@(context.Request.IpAddress)</value></set-header>

<!-- Rewrite URI -->
<rewrite-uri template="/v2/users/{id}" />
```

Policy expressions: C#-like using `context` object (context.Request, context.Response, context.Subscription, context.User).

🚨 EXAM ALERTS: Mock response goes in INBOUND (never reaches backend). Rate-limit resets at renewal-period boundary, not from first call. Named values = key-value store for policies (like env vars).

(6) Versioning — URL path (/v1/, /v2/), query string (?api-version=1.0), header (Api-Version: 1.0).
(7) Revisions — non-breaking changes without new version. Current revision = live.

### Lab Guide — `M08_Lab_API_Management.pdf`
💰 COST: APIM Developer tier ≈ $0.07/hr (~$50/month). DELETE IMMEDIATELY after lab. Under $0.25 if deleted within 3 hours.
Parts: (1) Create APIM instance — Developer tier (takes 30–45 min to provision) (2) Import public REST API via OpenAPI spec (3) Test in developer portal (4) Add rate-limit policy (5 calls/30 sec) (5) Test rate limit with curl loop (6) Add mock response for one operation (7) Add cache policy (60 sec) (8) Create product + subscription (9) Add JWT validation policy (10) CLEAN UP IMMEDIATELY

---

## MODULE 9 — Event-Based Solutions
**Domain:** Connect to and Consume Azure Services (20–25%)
**Path:** `az-204-develop-event-based-solutions`
**Prerequisites:** F01–F08, M01–M08

### Concept PDF — `M09_Concept_Event_Based.pdf` — 18–22 pages

Sections: (1) Events vs Messages — newspaper headline (event: broadcast, readers decide what to do) vs work order (message: receiver must do a specific job). (2) Azure Event Grid — fully managed event routing. System topics (Azure services) vs custom topics (your events). Event handlers: Functions, webhooks, Logic Apps, Service Bus. Delivery guarantee: AT-LEAST-ONCE. Retry policy + dead-letter. 🚨 EXAM ALERT: at-least-once = your handler may receive same event twice. Make handler IDEMPOTENT. (3) Azure Event Hubs — big data streaming. Millions of events/second. Key concepts: Partitions (parallel lanes), Consumer Groups (independent readers — each reads ALL events independently), Retention period (1–7 days standard, up to 90 Premium), Offset (consumer position). Capture: auto-save events to Blob or ADLS. 🚨 EXAM ALERT: Event Hubs is NOT a queue. Events are NOT deleted when consumed. Each consumer group reads independently. (4) Choosing Event Grid vs Event Hubs:

| | Event Grid | Event Hubs |
|---|---|---|
| Type | Discrete events | Continuous stream |
| Throughput | Low-medium | Millions/second |
| Pattern | Reactive/serverless routing | Analytics/data pipelines |
| Message size | Up to 1MB | Up to 1MB |
| Retention | 24 hrs (retry) | 1–90 days |

### Lab Guide — `M09_Lab_Event_Based.pdf`
💰 COST: Event Grid ≈ $0.60/million ops (first 100K/month free). Event Hubs Basic ≈ $0.015/hr. Under $0.10.
Parts: (1) Create Event Grid custom topic (2) Create Function with Event Grid trigger (3) Publish custom event via CLI (4) Observe Function execution in logs (5) Add subject filter to subscription (6) Create Event Hubs namespace + hub (7) .NET producer app sends events (8) .NET consumer app reads events (9) Observe consumer group offset behaviour (10) Enable Capture to Blob Storage (11) Verify Avro files created (12) Clean up

---

## MODULE 10 — Message-Based Solutions
**Domain:** Connect to and Consume Azure Services (20–25%)
**Path:** `az-204-develop-message-based-solutions`
**Prerequisites:** F01–F08, M01–M09

### Concept PDF — `M10_Concept_Message_Based.pdf` — 18–22 pages

Sections: (1) Why queues — decoupling (sender/receiver don't need to run simultaneously), reliability (messages wait when receiver is down), load levelling (smooth traffic spikes). (2) Azure Service Bus — Queues (point-to-point FIFO) + Topics/Subscriptions (pub-sub, each subscription = independent copy). Message properties: Body, MessageId, TTL, SessionId. Peek-lock: lock → process → complete (delete) or abandon (return). Sessions: guaranteed ordered processing for related messages (same SessionId → same consumer). Dead-letter queue (DLQ): messages exceeding max delivery count or TTL. (3) Azure Queue Storage — simple cheap queue. Max 64KB messages. Visibility timeout: message invisible during processing, reappears if not deleted. 7-day TTL. (4) SDK: ServiceBusSender, ServiceBusReceiver, ServiceBusProcessor. (5) Decision table:

| Need | Service Bus | Queue Storage |
|---|---|---|
| FIFO ordering | ✅ (sessions) | ❌ |
| Pub/sub multiple subscribers | ✅ (topics) | ❌ |
| Built-in dead-letter | ✅ | ❌ |
| Messages > 64KB | ✅ (up to 100MB Premium) | ❌ |
| Transactions | ✅ | ❌ |
| Very high volume + simplicity | ❌ (higher cost) | ✅ |

🚨 EXAM ALERTS: Service Bus = ordering/sessions/transactions/DLQ. Queue Storage = simple, cheap, massive volume. Peek-lock = at-least-once processing guarantee.

### Lab Guide — `M10_Lab_Message_Based.pdf`
💰 COST: Service Bus Basic ≈ $0.05/million ops. Queue Storage negligible. Lab under $0.01.
Parts: (1) Create Service Bus namespace (2) Create queue (3) Send messages via SDK (4) Receive + process (peek-lock) (5) Complete messages (6) Demonstrate abandon (message reappears) (7) Observe DLQ after max delivery exceeded (8) Create topic + 2 subscriptions (9) Send to topic — both subscriptions receive copy (10) Create Queue Storage queue (11) Send/receive via QueueClient (12) Observe visibility timeout (13) Clean up

---

## MODULE 11 — Application Insights and Monitoring
**Domain:** Monitor, Troubleshoot, and Optimize (10–15%)
**Path:** `az-204-instrument-solutions-support-monitoring-logging`
**Prerequisites:** F01–F08, M01–M10

### Concept PDF — `M11_Concept_Monitoring.pdf` — 16–20 pages

Sections: (1) Why monitoring matters — you cannot fix what you cannot see. (2) Three pillars of observability — Logs (what happened), Metrics (how is it performing), Traces (where did this request go). (3) Application Insights — Azure's APM service. Auto-collects: requests, dependencies, exceptions, performance counters, custom events. (4) Instrumentation — SDK (add package + one config line), client-side JS snippet, auto-instrumentation (codeless via App Service settings). (5) Application Map — visual diagram of components + dependencies + failure rates. (6) Live Metrics Stream — real-time, zero delay. Used during deployments and incidents. (7) Availability tests — URL ping from global regions. Alerts on fail or slow response. (8) Alerts + Action Groups — metric alert / log alert (KQL query). Action: email/SMS/webhook/Function. (9) KQL basics:

```kql
// Failed requests in last 24 hours
requests | where timestamp > ago(24h) | where success == false
| summarize count() by resultCode

// Top 10 slowest requests
requests | where timestamp > ago(1h)
| top 10 by duration desc
| project timestamp, name, duration, resultCode

// Exception count by type
exceptions | where timestamp > ago(24h)
| summarize count() by type | order by count_ desc

// Avg response time per hour
requests | where timestamp > ago(24h)
| summarize avg(duration) by bin(timestamp, 1h) | render timechart
```

(10) Smart Detection — AI-powered anomaly detection. Automatic. No config needed. (11) Sampling — adaptive/fixed-rate/ingestion to control cost. (12) Custom telemetry — TrackEvent(), TrackMetric(), TrackException(), TrackDependency().

### Lab Guide — `M11_Lab_Monitoring.pdf`
💰 COST: App Insights = 5GB/month free. Lab generates <1MB. Cost = $0.
Parts: (1) Create App Insights resource (2) Create .NET web app (3) Add SDK: `dotnet add package Microsoft.ApplicationInsights.AspNetCore` (4) Configure connection string (5) Run app, make requests including errors (6) View Live Metrics during load (7) Explore Application Map (8) Query exceptions via KQL (9) Query slow requests via KQL (10) Create availability test (11) Create metric alert (response >2 sec) (12) Create action group (email) (13) Trigger alert, confirm email (14) Clean up

---

## MODULE 12 — Azure Cache for Redis
**Domain:** Connect to and Consume Azure Services (20–25%)
**Path:** `az-204-integrate-caching-content-delivery-within-solutions`
**Prerequisites:** F01–F08, M01–M11

### Concept PDF — `M12_Concept_Redis_Cache.pdf` — 14–18 pages

Sections: (1) What is caching — notepad analogy. Without cache: every request hits DB. With cache: most requests return from memory instantly. (2) Azure Cache for Redis — fully managed in-memory key-value store. (3) Tiers: Basic (no SLA, dev/test only), Standard (replicated, SLA, production minimum), Premium (clustering, persistence, VNet, geo-replication). 🚨 EXAM ALERT: Basic has NO SLA. Minimum for production = Standard. (4) Redis data structures: Strings (key-value), Hashes (field-value map), Lists (ordered), Sets (unique unordered), Sorted Sets (scored — leaderboards). (5) Caching patterns:

Cache-aside (Lazy Loading — MOST COMMON): check cache → miss → load from DB → store in cache → return.
Write-through: write to cache AND DB simultaneously. Always fresh. Higher write latency.
Write-behind: write cache first, async flush to DB. Fastest writes. Risk of data loss.
🚨 EXAM ALERT: Cache-aside is the most tested pattern.

(6) StackExchange.Redis SDK:
```csharp
// Create ONCE (singleton — expensive to create)
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(connectionString);
IDatabase db = redis.GetDatabase();

// Set with TTL
await db.StringSetAsync("user:42", JsonSerializer.Serialize(user), TimeSpan.FromMinutes(30));

// Get (cache hit/miss)
string cached = await db.StringGetAsync("user:42");

// Check existence / delete / set expiry
await db.KeyExistsAsync("key");
await db.KeyDeleteAsync("key");
await db.KeyExpireAsync("key", TimeSpan.FromHours(1));
```

🚨 EXAM ALERT: ConnectionMultiplexer = create ONCE as singleton and reuse. Creating per-request = performance issues and connection exhaustion.

(7) TTL — always set it. Never cache indefinitely. (8) Eviction policies: allkeys-lru (recommended), volatile-lru, noeviction. (9) Session state caching — store ASP.NET sessions in Redis for stateless scale-out.

### Lab Guide — `M12_Lab_Redis_Cache.pdf`
💰 COST: Redis Basic C0 ≈ $0.022/hr. Clean up within 3 hrs = under $0.07.
Parts: (1) Create Azure Cache for Redis (Basic C0) (2) Note connection string (3) Create .NET app with StackExchange.Redis (4) Implement cache-aside pattern (5) Run twice — observe cache miss then cache hit (6) Set TTL, wait for expiry, observe miss (7) Use Redis Console in portal to inspect keys (8) Clean up

---

## MODULE 13 — Azure CDN
**Domain:** Connect to and Consume Azure Services (20–25%)
**Path:** `az-204-integrate-caching-content-delivery-within-solutions`
**Prerequisites:** F01–F08, M01–M12

### Concept PDF — `M13_Concept_CDN.pdf` — 12–16 pages

Sections: (1) What is CDN — overseas user analogy. Without CDN: everyone downloads from one origin server. With CDN: content cached at edge servers (PoPs) near users worldwide. (2) Azure CDN products: Microsoft (most tested in AZ-204), Verizon, Akamai. (3) Key concepts: Origin (Blob, App Service, custom URL), Edge node (PoP near user), Cache hit (fast, from edge), Cache miss (fetched from origin then cached), CDN profile (container), CDN endpoint (the azureedge.net URL). (4) Cache behaviour: Cache-Control headers control cache duration. Override with CDN rules. Query string behaviour: ignore / bypass / unique. (5) Purging: force discard of cached content. Portal, CLI (`az cdn endpoint purge --content-paths "/*"`), SDK. 🚨 EXAM ALERT: purge takes MINUTES to propagate to all edge nodes — not instant. (6) HTTPS on CDN: add custom domain + provision SSL certificate (auto-managed by Azure CDN). (7) CDN + Blob static website: add CDN in front of M04 static website hosting. (8) CDN vs Redis Cache:

| | Azure CDN | Redis Cache |
|---|---|---|
| WHERE it caches | At the edge (near users) | At the app layer (your region) |
| WHAT it caches | Static assets (images, CSS, JS, video) | Data (DB results, sessions, computed) |
| WHO benefits | End users (browser) | Your backend (DB, compute) |
| Primary goal | Geographic distribution | Reduce backend load |

🚨 EXAM ALERT: CDN ≠ Redis. They solve different problems. Know which scenario asks for which.

### Lab Guide — `M13_Lab_CDN.pdf`
💰 COST: CDN ≈ $0.087/GB transfer (first 5GB/month free). Lab transfers <1MB. Cost ≈ $0.
Parts: (1) Create CDN profile (Microsoft Standard) (2) Create endpoint with Blob static website as origin (3) Upload assets to static website (4) Access via CDN URL, compare load time (5) Configure caching rule (max-age 1 hour) (6) Purge cache via portal (7) Purge via CLI (8) Configure query string caching (9) Clean up


---
---

# LAYER 2: EXAM PREPARATION PACK

---

## EP01 — How Microsoft Writes AZ-204 Questions
**Filename:** `EP01_How_Microsoft_Writes_Questions.pdf`
**Pages:** 12–15

**Section 1: Question Anatomy**
Every question: scenario → constraints → question → 4 options.
Read the CONSTRAINTS first. The scenario gives context. Constraints tell you exactly what the answer must satisfy.

**Section 2: The 2-Step Elimination**
Eliminate 2 obviously wrong answers immediately. Spend thinking time on the final 2.

**Section 3: Keyword Spotting — Complete Reference Table**

| Keyword / Phrase in Question | Points To |
|---|---|
| "without managing infrastructure" / "serverless" | Functions, Container Apps, App Service |
| "zero downtime deployment" | Deployment slots (Standard tier+) |
| "instant rollback" | Deployment slot swap-back |
| "no cold starts" | Functions Premium plan |
| "VNet integration" for Functions | Premium or Dedicated (NOT Consumption) |
| "function runs longer than 10 minutes" | Durable Functions / Premium plan |
| "parallel tasks, aggregate results" | Durable Functions Fan-out/Fan-in |
| "sequential steps, output feeds next" | Durable Functions Function Chaining |
| "long job, HTTP status polling" | Durable Functions Async HTTP API |
| "flexible polling until condition met" | Durable Functions Monitor |
| "store and manage container images" | Azure Container Registry |
| "run container quickly, one-off task" | Azure Container Instances |
| "microservices, serverless containers" | Azure Container Apps |
| "traffic splitting between app versions" | ACA revisions |
| "no credentials in code" | Managed Identity + DefaultAzureCredential |
| "store secrets centrally and securely" | Azure Key Vault |
| "centralise app config, feature flags" | Azure App Configuration |
| "toggle feature without redeployment" | App Configuration feature flags |
| "App Setting references Key Vault" | @Microsoft.KeyVault syntax |
| "service-to-service, no user" | Client Credentials OAuth flow |
| "device with no browser" | Device Code OAuth flow |
| "call Microsoft 365 data" | Microsoft Graph API |
| "rate limit API calls per subscription" | APIM rate-limit policy |
| "validate OAuth token at gateway" | APIM validate-jwt policy |
| "mock API response for testing" | APIM mock-response policy (inbound) |
| "return fake response, no backend call" | APIM mock-response (inbound section) |
| "cache API response at gateway" | APIM cache-lookup + cache-store |
| "discrete event routing to handlers" | Azure Event Grid |
| "high-volume streaming, IoT, telemetry" | Azure Event Hubs |
| "ordered message delivery" | Service Bus (sessions for FIFO) |
| "pub/sub, multiple independent subscribers" | Service Bus Topics or Event Grid |
| "simple cheap queue, very high volume" | Queue Storage |
| "reduce database load, repeated queries" | Azure Cache for Redis (cache-aside) |
| "serve static content globally, low latency" | Azure CDN |
| "blob uploaded, trigger processing" | Blob trigger (Function) or Event Grid |
| "at-least-once delivery" | Event Grid, Service Bus |
| "idempotent handler required" | Consequence of at-least-once delivery |
| "rehydrate archived blob" | Archive tier → change to Hot or Cool |
| "move blobs to archive after N days" | Blob lifecycle management policy |
| "app needs to reach private VNet resources" | VNet Integration (Standard+, outbound only) |
| "bring Azure service into VNet" | Private Endpoint |
| "read from secondary region" | RA-GRS storage |
| "Cosmos DB — always latest, highest latency" | Strong consistency |
| "Cosmos DB — default, per session" | Session consistency |
| "change feed does not include..." | Deletes |
| "monitor performance, distributed tracing" | Application Insights |
| "query logs" | KQL in Log Analytics |
| "anomaly detection, automatic alerts" | Smart Detection |

**Section 4: "Select All That Apply" Strategy**
Usually exactly 2 correct answers. If you found only 1, look harder. If you found 3, one is a distractor.

**Section 5: Time Management**
120 min for 40–60 questions = ~2 min per question. Flag hard questions, move on, return at end. Never spend >3 min on one question.

**Section 6: The "Most" Trap**
Questions asking for "most secure", "most cost-effective", "most appropriate" — all options might work. Pick the BEST one for the given constraints.

**Section 7: Common Question Structures**
- "Developer wants to do X with requirement Y. What should they use?" → feature selection
- "Company has X and needs Y. What change to make?" → upgrade/configuration
- "Need to ensure X. What to configure?" → specific setting
- "Which code is correct for X?" → SDK syntax recognition

---

## EP02 — Master Cheat Sheet: All 13 Modules
**Filename:** `EP02_Master_Cheat_Sheet.pdf`
**Format:** One A4 landscape page per module — designed for print, final-day revision

**Each page contains:**
- Service name + one-line description
- The single best analogy
- Top 5 exam facts (most likely to appear)
- Top 3 exam traps (wrong answers + why wrong)
- Decision rule: "Choose this when... / Do NOT choose when..."
- One essential CLI command
- Connects to: (linked modules)

---

## EP03 — The 50 Most Common Exam Traps
**Filename:** `EP03_50_Common_Exam_Traps.pdf`
**Format per trap:** Wrong answer | Why students pick it | Right answer | Reasoning

### App Service Traps (1–8)
1. Picking Basic B2 for slot requirement → Standard S1 minimum. "Dedicated" sounds production-ready but Basic has no slots.
2. Thinking VNet Integration allows INBOUND traffic → outbound only. Direction confusion is the #1 networking trap.
3. Thinking sticky settings travel with the app during swap → sticky settings STAY WITH THE SLOT.
4. Enabling autoscale on Basic tier → requires Standard+. Basic = no autoscale.
5. Assuming F1 free tier supports custom domains → requires Basic+ for custom domains.
6. Thinking Easy Auth replaces writing auth code in all cases → Easy Auth is for app-level auth, not fine-grained API authorization.
7. Choosing scale up (vertical) when scenario requires handling more concurrent users → scale OUT (horizontal) for concurrency.
8. Thinking ARR Affinity should be ON for stateless apps → turn OFF for stateless to enable proper load distribution.

### Functions Traps (9–16)
9. Adding two triggers to one function → one trigger per function, always.
10. Using 5-part cron format → Azure uses 6-part NCRONTAB (second is the first field).
11. Choosing Consumption plan when VNet integration is required → Premium or Dedicated only.
12. Expecting Consumption plan to run for 15 minutes → max 10 minutes.
13. Confusing Fan-out/Fan-in with Function Chaining → Fan-out = PARALLEL tasks. Chaining = SEQUENTIAL steps.
14. Confusing Async HTTP API pattern with Monitor pattern → Async HTTP = client polls for one long job. Monitor = flexible recurring check until condition met.
15. Not knowing that `local.settings.json` must never be committed to Git → it contains connection strings and secrets.
16. Assuming Premium plan has the same timeout limit as Consumption → Premium has UNLIMITED timeout.

### Container Traps (17–20)
17. Using ACI for production microservices deployment → ACA or AKS for production microservices.
18. Confusing ACR (image storage) with ACI (running containers) → ACR = registry/warehouse. ACI = runtime.
19. Thinking ACA manages Kubernetes nodes and pods → ACA hides all Kubernetes infrastructure from you.
20. Not knowing ACR Tasks can build images without local Docker → `az acr build` = cloud-based build.

### Storage Traps (21–26)
21. Attempting to read from Archive tier immediately → requires rehydration first (up to 15 hours standard, 1 hour high-priority).
22. Assuming GRS secondary is readable by default → must enable RA-GRS separately for read access.
23. Thinking lifecycle policies execute immediately → policies run ONCE DAILY.
24. Confusing LRS (same datacenter) with ZRS (three zones) → LRS protects against disk failure. ZRS protects against datacenter failure.
25. Using account access key in production code → use Managed Identity + RBAC or User Delegation SAS instead.
26. Thinking all blob types support the same operations → Page blobs = random read/write (VMs). Append blobs = append only (logs). Block blobs = general files.

### Cosmos DB Traps (27–31)
27. Picking Strong consistency for lowest latency → Strong has the HIGHEST latency. Eventual = lowest latency.
28. Expecting change feed to capture deletes → change feed does NOT include deletes.
29. Thinking partition key can be changed after container creation → CANNOT be changed. Choose carefully upfront.
30. Forgetting that stored procedures are scoped to a single partition key value → cross-partition operations in stored procs are not supported.
31. Confusing Request Units as just a cost metric → RUs are BOTH a cost metric AND a performance throttle. Exceed your provisioned RU/s = requests get throttled.

### Security Traps (32–37)
32. Thinking RBAC role alone grants Key Vault access → network access (firewall/private endpoint) is also required. TWO layers.
33. Using Client Credentials flow when a user must sign in → Client Credentials has NO user context.
34. Hard-coding connection strings in application code → use Key Vault references in App Settings with Managed Identity.
35. Confusing system-assigned and user-assigned Managed Identity lifecycle → system-assigned is deleted WITH the resource. User-assigned persists independently.
36. Thinking App Configuration stores secrets → App Configuration = settings and feature flags ONLY. Secrets → Key Vault.
37. Using Access Policies model for new Key Vault deployments → RBAC model is the recommended modern approach.

### APIM Traps (38–42)
38. Placing mock-response in outbound section → mock-response MUST be in inbound section (it never reaches the backend).
39. Confusing rate-limit (per subscription per time window) with quota (total for the subscription period) → rate-limit = per-minute/hour throttle. Quota = total cap over a longer period.
40. Confusing API versions with revisions → versions = BREAKING changes (new URL/header). Revisions = non-breaking changes, same URL.
41. Thinking APIM replaces authentication → APIM validates tokens with validate-jwt but the tokens are still issued by an identity provider (Entra ID).
42. Not knowing Named Values purpose → Named Values are like environment variables for APIM policies. Can reference Key Vault secrets.

### Events and Messages Traps (43–47)
43. Using Event Grid for high-volume IoT telemetry → Event Hubs is designed for high-throughput streams.
44. Using Service Bus when Queue Storage is the better fit → Service Bus is overkill for simple high-volume queuing with no ordering/session/DLQ requirement.
45. Expecting Event Hubs to delete messages after a consumer reads them → messages are retained for the retention period. Consumer groups track their own offset independently.
46. Thinking at-least-once delivery means exactly-once → it means the handler may receive duplicates. Handler must be idempotent.
47. Thinking Service Bus and Queue Storage are interchangeable → they are not. Service Bus = ordering, sessions, DLQ, pub-sub, transactions. Queue Storage = simple, cheap, massive volume.

### Monitoring, Caching and CDN Traps (48–50)
48. Using CDN to cache database query results → CDN = static content at the edge for geographic distribution. Use Redis for application data caching.
49. Creating a new ConnectionMultiplexer on every Redis request → ConnectionMultiplexer is expensive. Create ONCE as a singleton and reuse it across the application lifetime.
50. Thinking Application Insights is a separate monitoring product from Azure Monitor → Application Insights IS a feature of Azure Monitor. They are integrated.

---

## EP04 — Practice Exam 1: Foundation to Intermediate
**Filename:** `EP04_Practice_Exam_1.pdf`
**Questions:** 40 | **Difficulty:** Foundation 40% + Intermediate 60% | **Time:** 80 minutes

**Distribution:**
- Compute (M01+M02+M03): 12 questions (30%)
- Storage (M04+M05): 8 questions (20%)
- Security (M06+M07): 8 questions (20%)
- Monitoring (M11): 4 questions (10%)
- Connectivity (M08+M09+M10+M12+M13): 8 questions (20%)

**After each section:** answer key with full explanation for EVERY question — why correct is correct, why each wrong answer is wrong.

**Sample questions to include:**
- Q: An App Service on Basic B2 tier. Developer wants to create a staging deployment slot. What must happen first? → Upgrade to Standard S1 (Basic has no slots)
- Q: An Azure Function on Consumption plan is timing out processing large files. The processing takes 12 minutes. What are the two valid solutions? → (SELECT TWO) Switch to Premium plan / Use Durable Functions
- Q: Which Cosmos DB consistency level is the default? → Session
- Q: A developer wants app secrets accessible in code without storing them in source control or app config files. What is the recommended approach? → Key Vault + Managed Identity + DefaultAzureCredential

---

## EP05 — Practice Exam 2: Exam-Realistic Difficulty
**Filename:** `EP05_Practice_Exam_2.pdf`
**Questions:** 50 | **Difficulty:** Intermediate 30% + Exam-level 70% | **Time:** 120 minutes (same as real exam)

**Characteristics:**
- Every question is a business scenario with 2–3 explicit constraints
- All 4 options are plausible — distractors carefully designed
- Includes "SELECT TWO" questions
- Mix of services — some questions span multiple modules
- Final 10 questions: complex multi-service architecture scenarios

**Distribution:** Weighted by exam domain (25-30% compute, 20-25% connectivity, 15-20% storage, 15-20% security, 10-15% monitoring)

**Sample exam-level questions to include:**
- A company needs a container-based microservice that: scales to zero when idle, connects to a private database in a VNet, has no cold start requirement, and uses percentage-based traffic splitting between versions. Which service and configuration? → ACA with VNet integration + revisions traffic splitting (Consumption plan Functions eliminated by VNet, ACI eliminated by no traffic splitting, App Service eliminated by container microservices context)
- A developer needs to ensure that an Azure Function reading from Service Bus processes each order EXACTLY ONCE and in the correct sequence per customer. What must be configured? → Peek-lock receive mode + Service Bus sessions (SessionId = customer ID)

**Complete answer key:** Every question with full explanation including why wrong answers are wrong.

---

## EP06 — Rapid-Fire Flashcard Question Bank
**Filename:** `EP06_Rapid_Fire_Question_Bank.pdf`
**Questions:** 150 | **Format:** Print-ready flashcard layout + listed digital format

**Complete 150-question bank (sample — expand to full 150):**

**App Service (Questions 1–20)**
Q1: Minimum tier for deployment slots? → Standard
Q2: Minimum tier for autoscale? → Standard
Q3: What do sticky settings do during a swap? → Stay with the slot, do not travel with the app
Q4: What is Always On used for? → Prevent app unloading after 20 min of inactivity (required for WebJobs)
Q5: Is VNet Integration inbound or outbound? → Outbound only
Q6: What tier is required for VNet Integration? → Standard or above
Q7: What does ARR Affinity do? → Routes same user to same instance (sticky sessions)
Q8: Should ARR Affinity be on or off for stateless apps? → OFF
Q9: Minimum TLS version recommended? → 1.2
Q10: What does HTTPS Only do? → Redirects all HTTP → HTTPS
Q11: Which deployment method deploys from a GitHub repo automatically on push? → GitHub Actions
Q12: What is the URL format for a staging slot named "staging" on app "myapp"? → myapp-staging.azurewebsites.net
Q13: What happens to non-sticky settings during a slot swap? → They travel with the app code to the target slot
Q14: What is the Isolated tier used for? → Deploy App Service into its own dedicated VNet (App Service Environment)
Q15: What is Easy Auth? → Built-in authentication without writing code, configured at App Service level
Q16: Which Easy Auth provider is used for enterprise corporate identity? → Microsoft Entra ID
Q17: What does scale up mean? → Change to a larger VM size (vertical scaling)
Q18: What does scale out mean? → Add more VM instances (horizontal scaling)
Q19: What is the autoscale cooldown period? → Wait time after scaling before evaluating rules again
Q20: Can Free tier have a custom domain? → No. Requires Basic tier minimum.

**Azure Functions (Questions 21–40)**
Q21: How many triggers can one function have? → Exactly one
Q22: Max timeout on Consumption plan? → 10 minutes (default 5 min, configurable up to 10)
Q23: Max timeout on Premium plan? → Unlimited
Q24: Which plan requires VNet integration for Functions? → Premium or Dedicated (not Consumption)
Q25: Which plan eliminates cold starts? → Premium (pre-warmed instances)
Q26: First field in Azure Functions cron expression? → Seconds
Q27: Cron expression for every day at midnight? → 0 0 0 * * *
Q28: Cron expression for every 5 minutes? → 0 */5 * * * *
Q29: Which Durable pattern for parallel tasks with aggregated results? → Fan-out/Fan-in
Q30: Which Durable pattern for sequential steps passing output forward? → Function Chaining
Q31: Which Durable pattern for long job with client polling for status? → Async HTTP API
Q32: Which Durable pattern for flexible polling until a condition is met? → Monitor
Q33: What file stores local Function App configuration? → local.settings.json
Q34: Should local.settings.json be committed to Git? → Never
Q35: What does AzureWebJobsStorage set in Functions? → The storage account connection string for internal Function App state
Q36: What is AuthorizationLevel.Function? → Requires a function-specific key in the request
Q37: What is AuthorizationLevel.Anonymous? → No key required, public access
Q38: What command deploys a Function App from local to Azure? → func azure functionapp publish <appname>
Q39: What command runs Functions locally? → func start
Q40: What NuGet package enables Azure Functions output bindings for Service Bus? → Microsoft.Azure.WebJobs.Extensions.ServiceBus

**Containers (Questions 41–55)**
Q41: What is Azure Container Registry? → Private Docker image registry in Azure
Q42: What is Azure Container Instances? → Run a single container in Azure without managing infrastructure
Q43: What is Azure Container Apps? → Serverless containers on Kubernetes (Kubernetes abstracted away)
Q44: Which service for running a one-time batch job in a container? → ACI
Q45: Which service for production microservices? → ACA
Q46: What does az acr build do? → Builds a Docker image in the cloud using ACR Tasks (no local Docker needed)
Q47: What is an ACA revision? → An immutable snapshot of a container app configuration
Q48: What is ACA traffic splitting? → Routing a percentage of requests to different revisions
Q49: What is KEDA? → Kubernetes Event-Driven Autoscaling — used in ACA for custom scaling rules
Q50: ACI restart policy for a one-time task that should never restart? → Never
Q51: ACI restart policy for a web server that must always be running? → Always
Q52: ACI restart policy for a batch job with retry on failure? → OnFailure
Q53: What is an ACR Basic vs Premium tier difference? → Storage limit and throughput (Premium also has geo-replication and private link)
Q54: What is a Container Group in ACI? → Multiple containers sharing the same network and storage lifecycle
Q55: How do you authenticate ACA to pull from ACR in production? → Managed Identity

**Blob Storage (Questions 56–70)**
Q56: What are the 3 blob types? → Block (files), Append (logs), Page (VMs, random access)
Q57: Minimum storage duration for Archive tier? → 180 days
Q58: Minimum storage duration for Cool tier? → 30 days
Q59: How do you read a blob in Archive tier? → Rehydrate it first (change tier to Hot or Cool)
Q60: Rehydration time for Archive (standard priority)? → Up to 15 hours
Q61: Rehydration time for Archive (high priority)? → Under 1 hour
Q62: When do lifecycle management policies run? → Once daily
Q63: What is a User Delegation SAS? → SAS token signed with Entra ID credentials — most secure SAS type
Q64: What storage redundancy replicates to a second Azure region? → GRS (Geo-Redundant Storage)
Q65: Is the GRS secondary region readable by default? → No — must enable RA-GRS for read access
Q66: Which blob type is for append-only write patterns like logs? → Append blob
Q67: What is the $web container used for? → Azure Blob Storage static website hosting
Q68: Which SDK client manages individual blobs? → BlobClient
Q69: Which SDK client manages containers? → BlobContainerClient
Q70: How do you rotate a storage account access key without downtime? → Use Key2 while rotating Key1, then switch back

**Cosmos DB (Questions 71–85)**
Q71: What is the default Cosmos DB consistency level? → Session
Q72: Which consistency level always returns the latest data? → Strong
Q73: Which consistency level has the lowest latency? → Eventual
Q74: Can you change a Cosmos DB partition key after container creation? → No
Q75: What does change feed NOT include? → Deletes
Q76: What is a Request Unit (RU)? → The unit of throughput in Cosmos DB — every operation costs RUs
Q77: Which Cosmos DB API is most used in AZ-204? → Core (SQL) API
Q78: What is the Cosmos DB free tier limit? → 1000 RU/s + 25GB storage per account
Q79: What is a hot partition in Cosmos DB? → When one partition key receives a disproportionate amount of traffic
Q80: Are all Cosmos DB properties indexed by default? → Yes — all properties indexed by default
Q81: What scope do stored procedures operate within? → Single partition key value
Q82: What .NET SDK class connects to a Cosmos DB account? → CosmosClient
Q83: How do you read an item by ID in Cosmos DB? → You must provide both the item ID AND the partition key value
Q84: What is upsert in Cosmos DB SDK? → Create the item if it doesn't exist, replace it if it does
Q85: What is bounded staleness consistency? → Reads lag behind writes by at most K versions or T seconds (configurable)

**Security (Questions 86–100)**
Q86: What three object types does Key Vault store? → Secrets (passwords/conn strings), Keys (crypto), Certificates (SSL/TLS)
Q87: What is the modern access model for Key Vault? → RBAC (vs legacy Access Policies)
Q88: Key Vault RBAC role to read secrets (not manage)? → Key Vault Secrets User
Q89: What is soft delete in Key Vault? → Deleted objects are retained for 7–90 days and can be recovered
Q90: What does purge protection do? → Prevents permanent deletion during the retention period
Q91: Key Vault reference syntax in App Settings? → @Microsoft.KeyVault(SecretUri=https://vault.../secrets/Name/)
Q92: What are the two separate access control layers for Key Vault? → RBAC (who can access) + Network (where from)
Q93: System-assigned vs user-assigned Managed Identity lifecycle? → System-assigned: deleted with the resource. User-assigned: independent.
Q94: What is DefaultAzureCredential? → SDK class that automatically uses Managed Identity in Azure and az login locally
Q95: What OAuth flow for service-to-service with no user? → Client Credentials
Q96: What OAuth flow for a CLI tool with no browser? → Device Code
Q97: What is an ID token? → JWT containing claims about who the user IS (identity)
Q98: What is an access token? → JWT containing permissions for what the user CAN DO (authorization)
Q99: What is App Configuration used for? → Centralised key-value settings + feature flags (NOT secrets)
Q100: What is MSAL? → Microsoft Authentication Library — SDK for authenticating with Microsoft Identity Platform

**APIM, Events, Messages (Questions 101–130)**
Q101: What are the 3 components of APIM? → Gateway + Management Plane + Developer Portal
Q102: Which APIM policy section does mock-response go in? → Inbound
Q103: Which APIM policy validates OAuth tokens? → validate-jwt
Q104: What is the difference between rate-limit and quota policies? → Rate-limit = per time window throttle. Quota = total cap over a longer period.
Q105: What are Named Values in APIM? → Key-value pairs stored in APIM for use in policies (like env vars for policies)
Q106: Difference between API versions and revisions? → Versions = breaking changes (new URL). Revisions = non-breaking changes (same URL).
Q107: Event Grid delivery guarantee? → At-least-once
Q108: What does at-least-once delivery require of your handler? → Idempotency (same result regardless of how many times it runs)
Q109: Event Grid vs Event Hubs — which for IoT telemetry at 1 million events/second? → Event Hubs
Q110: Event Grid vs Event Hubs — which for routing a user-created event to an Azure Function? → Event Grid
Q111: Does Event Hubs delete events when a consumer reads them? → No — retained for the retention period
Q112: What is a consumer group in Event Hubs? → An independent reader of the full event stream
Q113: What is Event Hubs Capture? → Auto-save all incoming events to Blob Storage or ADLS
Q114: Service Bus queue vs topic — key difference? → Queue = one receiver. Topic = multiple independent subscribers via subscriptions.
Q115: What is a Service Bus session? → Guarantees ordered processing of messages with the same SessionId
Q116: What is the Service Bus dead-letter queue (DLQ)? → Queue for messages that exceed max delivery count or TTL
Q117: Peek-lock vs Receive-and-Delete in Service Bus? → Peek-lock = at-least-once, safe. Receive-and-Delete = at-most-once, risky if processing fails.
Q118: Queue Storage max message size? → 64KB
Q119: Service Bus max message size (Standard tier)? → 256KB
Q120: What is visibility timeout in Queue Storage? → Period during which a received message is invisible to other consumers

**Monitoring, Redis, CDN (Questions 121–150)**
Q121: What are the 3 pillars of observability? → Logs + Metrics + Traces
Q122: What does Application Insights Application Map show? → Visual diagram of app components, dependencies, response times, and failure rates
Q123: What is Live Metrics Stream? → Real-time telemetry with zero delay — used during deployments and incidents
Q124: What is an availability test? → Ping your app URL from multiple Azure regions on a schedule
Q125: What KQL command returns the last 24 hours of data? → where timestamp > ago(24h)
Q126: What is Smart Detection? → AI-powered automatic anomaly detection in Application Insights
Q127: What does TrackEvent() do? → Records a custom named event in Application Insights telemetry
Q128: Redis Cache Basic tier — production suitable? → No — Basic has no SLA and no replication
Q129: Redis Cache minimum tier for production? → Standard (replicated, has SLA)
Q130: What is the cache-aside pattern? → Check cache → miss → load from source → store in cache → return to caller
Q131: What StackExchange.Redis class connects to Redis? → ConnectionMultiplexer
Q132: Should you create ConnectionMultiplexer per request? → Never — create once as a singleton and reuse
Q133: What is TTL in Redis? → Time-to-Live — automatic expiry of a cached key after a set duration
Q134: What eviction policy removes least recently used keys when cache is full? → allkeys-lru
Q135: Can Redis Cache store session state? → Yes — replace in-memory ASP.NET sessions with Redis for stateless scale-out
Q136: What is a Redis sorted set used for? → Leaderboards and ranked data (each member has a score)
Q137: What is a CDN Point of Presence (PoP)? → An edge server close to users that caches and serves content
Q138: What is a CDN cache hit? → Content served directly from the edge node without contacting the origin
Q139: Is a CDN cache purge instant? → No — takes minutes to propagate to all edge nodes
Q140: CDN query string behaviour "ignore"? → Same cached response for all query strings on the same URL
Q141: CDN vs Redis — which caches at the edge near users? → CDN
Q142: CDN vs Redis — which reduces database load? → Redis
Q143: What is the origin in Azure CDN? → Where the original content lives (Blob, App Service, custom URL)
Q144: How do you force CDN to fetch fresh content? → Purge the cache
Q145: Which CDN product is most tested in AZ-204? → Microsoft Standard (Azure Front Door CDN)
Q146: What is ARM? → Azure Resource Manager — processes ALL Azure requests (portal, CLI, SDK) for auth, validation, routing
Q147: Is ARM declarative or imperative? → Declarative (desired state). CLI commands = imperative.
Q148: What command deploys an ARM template? → az deployment group create --resource-group rg --template-file template.json
Q149: What does DefaultAzureCredential use when running in Azure? → Managed Identity (automatically)
Q150: What does DefaultAzureCredential use when running locally? → Azure CLI credentials (via az login)


---
---

# APPENDIX A: COMPLETE FILE INVENTORY — 79 FILES

## Foundation Pack — 8 files
| # | Filename | Title | Pages | Study Time |
|---|---|---|---|---|
| F01 | F01_How_The_Cloud_Works.pdf | How the Cloud Works: From Concept to Practice | 10–14 | 1.5 hrs |
| F02 | F02_Azure_Hierarchy_ARM_IaC.pdf | Azure Hierarchy, ARM, and IaC | 14–18 | 2 hrs |
| F03 | F03_Developer_Basics_Terminal_Code_Git_APIs.pdf | Developer Basics: Terminal, Code, Git, APIs | 22–28 | 3–4 hrs |
| F04 | F04_Azure_Portal_Navigation_Guide.pdf | Azure Portal Navigation Guide | 12–15 | 1 hr |
| F05 | F05_Lab_Zero_First_Azure_Experience.pdf | Lab Zero: Your First Azure Experience | 12–15 | 1.5 hrs |
| F06 | F06_CSharp_Basics_For_AZ204_Labs.pdf | C# Basics for AZ-204 Labs | 18–22 | 3–4 hrs |
| F07 | F07_Azure_Networking_For_Developers.pdf | Azure Networking for Developers | 14–18 | 2 hrs |
| F08 | F08_Storage_Account_And_SDK_Basics.pdf | Storage Account and SDK Basics | 14–16 | 2 hrs |

**Foundation Pack total study time: ~17–19 hours (1 week)**

## Module Packs — 13 modules × 5 files = 65 files

| Module | Concept PDF | Slides | Lab | Scenarios | Quiz |
|---|---|---|---|---|---|
| M01 App Service | M01_Concept_Azure_App_Service.pdf | M01_Slides_Azure_App_Service.pptx | M01_Lab_Azure_App_Service.pdf | M01_Scenarios_Azure_App_Service.pdf | M01_Quiz_Azure_App_Service.pdf |
| M02 Functions | M02_Concept_Azure_Functions.pdf | M02_Slides_Azure_Functions.pptx | M02_Lab_Azure_Functions.pdf | M02_Scenarios_Azure_Functions.pdf | M02_Quiz_Azure_Functions.pdf |
| M03 Containers | M03_Concept_Containers.pdf | M03_Slides_Containers.pptx | M03_Lab_Containers.pdf | M03_Scenarios_Containers.pdf | M03_Quiz_Containers.pdf |
| M04 Blob Storage | M04_Concept_Blob_Storage.pdf | M04_Slides_Blob_Storage.pptx | M04_Lab_Blob_Storage.pdf | M04_Scenarios_Blob_Storage.pdf | M04_Quiz_Blob_Storage.pdf |
| M05 Cosmos DB | M05_Concept_Cosmos_DB.pdf | M05_Slides_Cosmos_DB.pptx | M05_Lab_Cosmos_DB.pdf | M05_Scenarios_Cosmos_DB.pdf | M05_Quiz_Cosmos_DB.pdf |
| M06 Auth | M06_Concept_Auth_AuthZ.pdf | M06_Slides_Auth_AuthZ.pptx | M06_Lab_Auth_AuthZ.pdf | M06_Scenarios_Auth_AuthZ.pdf | M06_Quiz_Auth_AuthZ.pdf |
| M07 Security | M07_Concept_Secure_Solutions.pdf | M07_Slides_Secure_Solutions.pptx | M07_Lab_Secure_Solutions.pdf | M07_Scenarios_Secure_Solutions.pdf | M07_Quiz_Secure_Solutions.pdf |
| M08 APIM | M08_Concept_API_Management.pdf | M08_Slides_API_Management.pptx | M08_Lab_API_Management.pdf | M08_Scenarios_API_Management.pdf | M08_Quiz_API_Management.pdf |
| M09 Events | M09_Concept_Event_Based.pdf | M09_Slides_Event_Based.pptx | M09_Lab_Event_Based.pdf | M09_Scenarios_Event_Based.pdf | M09_Quiz_Event_Based.pdf |
| M10 Messages | M10_Concept_Message_Based.pdf | M10_Slides_Message_Based.pptx | M10_Lab_Message_Based.pdf | M10_Scenarios_Message_Based.pdf | M10_Quiz_Message_Based.pdf |
| M11 Monitoring | M11_Concept_Monitoring.pdf | M11_Slides_Monitoring.pptx | M11_Lab_Monitoring.pdf | M11_Scenarios_Monitoring.pdf | M11_Quiz_Monitoring.pdf |
| M12 Redis | M12_Concept_Redis_Cache.pdf | M12_Slides_Redis_Cache.pptx | M12_Lab_Redis_Cache.pdf | M12_Scenarios_Redis_Cache.pdf | M12_Quiz_Redis_Cache.pdf |
| M13 CDN | M13_Concept_CDN.pdf | M13_Slides_CDN.pptx | M13_Lab_CDN.pdf | M13_Scenarios_CDN.pdf | M13_Quiz_CDN.pdf |

## Exam Prep Pack — 6 files
| # | Filename | Title | Content |
|---|---|---|---|
| EP01 | EP01_How_Microsoft_Writes_Questions.pdf | How Microsoft Writes AZ-204 Questions | Question anatomy, keyword table, strategy |
| EP02 | EP02_Master_Cheat_Sheet.pdf | Master Cheat Sheet: All 13 Modules | 1 page per module, print-ready |
| EP03 | EP03_50_Common_Exam_Traps.pdf | The 50 Most Common Exam Traps | All traps with wrong/right answer + reasoning |
| EP04 | EP04_Practice_Exam_1.pdf | Practice Exam 1 — Foundation Level (40 Qs) | 40 questions + full answer key |
| EP05 | EP05_Practice_Exam_2.pdf | Practice Exam 2 — Exam-Realistic (50 Qs) | 50 questions + full answer key |
| EP06 | EP06_Rapid_Fire_Question_Bank.pdf | Rapid-Fire Flashcard Bank (150 Qs) | 150 Q&A pairs, flashcard format |

**GRAND TOTAL: 79 files**

---

# APPENDIX B: DESIGN STANDARDS

## Colour Palette
| Colour | Hex | Use |
|---|---|---|
| Azure Blue | #0078D4 | Primary headings, key elements, Azure branding |
| Dark Navy | #1B1B2F | Document headings, dark backgrounds |
| Alert Red | #D13438 | EXAM ALERT boxes, critical warnings |
| Success Green | #107C10 | Checkpoints, correct answers, lab success |
| Warning Orange | #FF8C00 | Cost warnings, caution boxes |
| Info Blue | #0058AD | KEY CONCEPT boxes |
| Light Grey | #F5F5F5 | Page backgrounds, table alternating rows |
| Code Background | #1E1E1E | Dark background for all code blocks (VS Code dark) |
| Code Text | #D4D4D4 | Light text on dark code background |
| White | #FFFFFF | Primary content areas |

## Typography
- Headings: Segoe UI Bold, 16–24pt
- Subheadings: Segoe UI SemiBold, 13–15pt
- Body text: Segoe UI Regular, 11–12pt, line spacing 1.4
- Code blocks: Consolas or Courier New, 10pt, dark background, syntax highlighted
- Captions: Segoe UI Regular, 9–10pt, colour #666666
- Page numbers: bottom centre, 9pt

## Callout Box System — Use Consistently Across ALL Documents
| Box | Icon | Border Colour | Background | Purpose |
|---|---|---|---|---|
| EXAM ALERT | 🚨 | #D13438 (red) | #FDE7E9 | Critical traps, must-know exam facts |
| KEY CONCEPT | 💡 | #0078D4 (blue) | #EFF6FF | Most important takeaway per section |
| CHECKPOINT | ✅ | #107C10 (green) | #E6F4EA | Mid-document comprehension pause |
| HANDS-ON | 🔬 | #6B2D8B (purple) | #F3E8FF | Lab activity embedded in concept doc |
| COST WARNING | 💰 | #FF8C00 (orange) | #FFF4E6 | Cost callout at start of every lab |
| NOTE | 📝 | #999999 (grey) | #F5F5F5 | Extra context, not exam-critical |
| MODULE LINK | 🔗 | #007B6E (teal) | #E6F6F4 | End-of-module connections to other modules |

## Code Block Standards
- Language always specified: ```csharp / ```bash / ```json / ```kql / ```xml
- Inline comments explain every non-obvious line
- Syntax highlighting applied
- Expected output shown in separate block labelled "Expected output:"
- Error examples labelled "If you see this error:"

---

# APPENDIX C: PEDAGOGICAL STANDARDS

1. **Analogy before definition (non-negotiable):** New concept = real-world analogy FIRST, then technical definition. Zero exceptions.

2. **Problem before solution:** Always explain the PROBLEM a service solves before introducing the service. "The problem is X. Azure solves it with Y."

3. **No undefined jargon:** Every term defined on first use, same sentence or KEY CONCEPT callout. If defined in previous doc, give a one-line reminder.

4. **AZ-900 bridging:** Start each Foundation doc with: "You already know X from AZ-900. Here we go deeper into the practical application." Don't re-teach AZ-900 at length.

5. **Exam relevance labelling:** Every major section labelled: 🔴 HIGH EXAM WEIGHT | 🟡 MEDIUM EXAM WEIGHT | ⚪ CONTEXT ONLY.

6. **Progressive difficulty:** Foundation concepts before advanced ones. No advanced concept without its prerequisites established first.

7. **Retrieval practice:** Every document ends with self-check section. Students close document and answer from memory first.

8. **Module connections map:** Every module ends with 🔗 MODULE CONNECTIONS showing how it links to other modules. Students see the architecture, not isolated services.

9. **Cost consciousness:** Every lab starts with 💰 COST callout. Students learn to think like a developer responsible for the bill.

10. **Format consistency:** Every document in the same category (all Concept PDFs, all Lab Guides) follows the exact same structure. Students never have to relearn the format.

---

# APPENDIX D: STUDENT LEARNING JOURNEY CHECKLIST
*Hand this to every student on Day 1. They track their own progress.*

## LAYER 0 — Foundation (Week 1)
- [ ] F01: I understand the cloud from a developer perspective (not just IT admin)
- [ ] F01: I can describe IaaS/PaaS/SaaS and know AZ-204 is mostly PaaS/serverless
- [ ] F02: I understand the full hierarchy: Management Group → Subscription → RG → Resource
- [ ] F02: I know what an ARM template is and can deploy one from CLI
- [ ] F02: I have set a spending budget alert on my Azure subscription
- [ ] F03: I can open a terminal and run navigation commands confidently
- [ ] F03: I can run `az login` and all basic Azure CLI resource management commands
- [ ] F03: I can read a JSON object and identify all fields and data types
- [ ] F03: I understand environment variables and why secrets must NEVER be in code
- [ ] F03: I can make GET and POST requests using Postman AND curl
- [ ] F03: I know all HTTP methods and key status codes (200, 201, 401, 403, 404, 500)
- [ ] F03: I can clone a GitHub repo, make a change, commit, and push
- [ ] F03: My .gitignore excludes local.settings.json and .env
- [ ] F04: I can navigate the Azure Portal and find any service instantly
- [ ] F04: I can open Cloud Shell and run CLI commands from the browser
- [ ] F05: I completed Lab Zero end-to-end without help
- [ ] F06: I can read a C# file and explain what every line does
- [ ] F06: I understand async/await and can explain why Azure SDK calls use it
- [ ] F06: I can create a .NET project, add an SDK package, build and run it
- [ ] F06: I know which NuGet package to use for each AZ-204 service
- [ ] F07: I can explain what a VNet is and why apps sometimes need one
- [ ] F07: I understand the difference: VNet Integration (app reaches INTO VNet, outbound) vs Private Endpoint (service is BROUGHT INTO VNet)
- [ ] F07: I know Key Vault has TWO separate access control layers (RBAC + Network)
- [ ] F08: I understand storage account types and redundancy options
- [ ] F08: I can find a storage account connection string in the portal
- [ ] F08: I have connected Azure Storage Explorer to a storage account

## LAYER 1 — Modules (Weeks 2–6)
*Repeat for each module M01 through M13:*
- [ ] Read Concept PDF — can explain every section without notes
- [ ] Reviewed Slide Deck — understand every EXAM ALERT slide
- [ ] Completed Lab — every step, hands-on, no skipping
- [ ] Ran cleanup commands — verified all resources deleted
- [ ] Worked through all Scenario Cards — wrote out reasoning
- [ ] Scored 80%+ on Module Quiz (retake if under 80%)
- [ ] Reviewed every wrong quiz answer — understand why I was wrong
- [ ] Filled in Module Connections — can see how this module links to others

### Module Completion Tracker
- [ ] M01 App Service — done ✓ Quiz score: ___
- [ ] M02 Functions — done ✓ Quiz score: ___
- [ ] M03 Containers — done ✓ Quiz score: ___
- [ ] M04 Blob Storage — done ✓ Quiz score: ___
- [ ] M05 Cosmos DB — done ✓ Quiz score: ___
- [ ] M06 Authentication — done ✓ Quiz score: ___
- [ ] M07 Secure Solutions — done ✓ Quiz score: ___
- [ ] M08 API Management — done ✓ Quiz score: ___
- [ ] M09 Events — done ✓ Quiz score: ___
- [ ] M10 Messages — done ✓ Quiz score: ___
- [ ] M11 Monitoring — done ✓ Quiz score: ___
- [ ] M12 Redis Cache — done ✓ Quiz score: ___
- [ ] M13 CDN — done ✓ Quiz score: ___

## LAYER 2 — Exam Prep (Week 7)
- [ ] EP01: Read the question-writing guide, can identify keyword patterns
- [ ] EP02: Reviewed Master Cheat Sheet for all 13 modules
- [ ] EP03: Studied all 50 exam traps, can explain why each wrong answer is wrong
- [ ] EP04: Completed Practice Exam 1 with 80%+ (retake if under 80%)
- [ ] EP04: Reviewed and understood every wrong answer from Exam 1
- [ ] EP05: Completed Practice Exam 2 under timed conditions (120 minutes)
- [ ] EP05: Scored 75%+ on Practice Exam 2 (retake if under 75%)
- [ ] EP06: Drilled Rapid-Fire Question Bank — answering under 5 seconds per question
- [ ] I can explain any AZ-204 service using only an analogy, no notes
- [ ] Exam is booked

## Ready to Pass ✅
- [ ] Two consecutive practice exam scores of 75%+ ✅
- [ ] Rapid-fire questions answered in under 5 seconds ✅
- [ ] Can explain the correct service for any scenario without hesitation ✅
- [ ] **EXAM BOOKED AND READY 🎯**

---

*End of AZ-204 Course Content Development Blueprint — Version 2.0*
*Gap-validated against official AZ-900 study guide (Jan 2026) and AZ-204 prerequisites*
*Built with Claude by Anthropic | For instructor and Copilot production use*
*Total deliverables: 79 files | Recommended student journey: 7–8 weeks*
*Layer 0: 8 docs | Layer 1: 65 docs (13 modules × 5) | Layer 2: 6 docs*
