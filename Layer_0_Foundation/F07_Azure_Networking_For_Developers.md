# F07 — Azure Networking for Developers

> **Foundation Document 7 of 8** | **Pages:** 14–18 | **Study Time:** 2 hours
> **Purpose:** Cover exactly and only the networking concepts that appear in AZ-204 labs and exam questions.
> **Prerequisites:** F01–F06 completed. Comfortable with Azure Portal, CLI, and basic resource creation.

---

> ⚠️ **IMPORTANT — What This Document Is and Is NOT**
>
> This document is **NOT** a networking certification guide. AZ-204 is a **developer** exam.
> You will never be asked to design a hub-and-spoke network topology or configure BGP routing.
>
> **What this document WILL cover:**
> - Why your App Service can't reach a private database (and how to fix it)
> - Why your Key Vault CLI command returns "network access denied" (and how to fix it)
> - VNet Integration, Private Endpoints, and Service Endpoints — the three networking features that appear on the AZ-204 exam
> - Key Vault firewall settings — the single most common networking scenario in AZ-204 labs
>
> **What this document will NOT cover:**
> - VPN Gateways, ExpressRoute, Azure Firewall, Traffic Manager, Load Balancer, Application Gateway
> - Routing tables, BGP, DNS zone configuration, hub-and-spoke architecture
> - Anything tested on AZ-700 (Azure Network Engineer) but not on AZ-204

---

## Section 1: Why Developers Need to Know Networking

**Exam Relevance:** 🟡 MEDIUM — You won't get pure networking questions, but networking concepts are embedded in compute, security, and storage questions.

You might be thinking: *"I'm a developer, not a network engineer. Why do I need to know about virtual networks?"*

Fair question. Here are two real scenarios from AZ-204 labs that will answer it immediately.

---

### Scenario A: App Service Can't Reach a Private Database

You're working on Module 5 (Cosmos DB). You've deployed an App Service that needs to read data from a Cosmos DB account. Your Cosmos DB has been configured to **deny public network access** for security — it only accepts connections from within a Virtual Network.

You deploy your app. You test it. You get this error:

```text
System.Net.Http.HttpRequestException: Connection refused
Unable to connect to the remote server
```

Your code is correct. Your connection string is correct. Your credentials are correct. **But your App Service is sitting on the public internet, and Cosmos DB only accepts traffic from inside a private network.**

The fix? **VNet Integration** (Section 4) + **Private Endpoint** (Section 5). You'll learn both in this document.

---

### Scenario B: Key Vault CLI Command Returns "Network Access Denied"

You're working on Module 7 (Key Vault). You created a Key Vault, stored a secret, and now you run this command from your laptop:

```bash
az keyvault secret show \
  --vault-name myKeyVault \    # ← the Key Vault name
  --name DatabasePassword      # ← the secret name
```

You expect to see the secret value. Instead, you get:

```text
(Forbidden) Client address is not authorized and caller is not a trusted service.
Code: Forbidden
```

Your RBAC permissions are correct — you have the **Key Vault Secrets User** role. **But the Key Vault firewall is blocking your IP address.** You have the right key to the door, but the bouncer won't let you close enough to use it.

The fix? Add your IP to the Key Vault firewall (Section 8). This is the **single most common networking error** in AZ-204 labs.

---

### The Developer's Networking Mental Model

You don't need to understand networking deeply. You need to understand **three things**:

1. **Where is my app running?** (Public internet? Inside a VNet?)
2. **Where is the service I'm trying to reach?** (Public endpoint? Private endpoint?)
3. **Is there a firewall between them?** (NSG? Service firewall? Key Vault networking rules?)

That's it. Every networking problem in AZ-204 comes down to these three questions.

---

## Section 2: Virtual Networks (VNet) — The Private Space in Azure

**Exam Relevance:** 🟡 MEDIUM — VNets themselves aren't tested directly, but every networking feature in AZ-204 builds on them.

### The Problem

When you create Azure resources, they exist in Microsoft's datacentres. But what if you need some resources to communicate privately — like a database server that should **never** be accessible from the public internet?

You need a way to create a **private, isolated network** inside Azure where only authorised resources can talk to each other.

### The Analogy: A Private Office Building

Think of a Virtual Network as a **private office building**:

- The **building** itself is the VNet — it has walls, a front door, and security
- Inside the building, there are **floors** — these are **subnets** (subdivisions of the network)
- Each office on each floor has a **room number** — these are **private IP addresses**
- People inside the building can walk to each other's offices freely — resources in the same VNet can communicate directly
- To get in from outside, you must go through **reception** — this is a gateway, endpoint, or firewall rule
- The building has an **address** — this is the **address space**

### Technical Definition

A **Virtual Network (VNet)** is a logically isolated section of the Azure cloud where you can launch Azure resources in a private network that you define.

### Key Components

| Component | What It Is | Analogy | Example |
|---|---|---|---|
| **Address Space** | The range of private IP addresses available in the VNet | The total number of rooms in the building | `10.0.0.0/16` (65,536 addresses) |
| **Subnet** | A subdivision of the address space | A floor in the building | `10.0.1.0/24` (256 addresses) |
| **Private IP** | An address assigned to a resource inside the VNet | A specific room number | `10.0.1.5` |
| **Region** | VNets exist in a single Azure region | The building's physical location | East US |

### Address Space — Don't Worry About the Math

You'll see notation like `10.0.0.0/16`. Here's what you need to know for AZ-204:

- The `/16` part tells Azure how many IP addresses are available (about 65,000)
- The `/24` means about 256 addresses — commonly used for subnets
- **Smaller number after the slash = more addresses** (`/16` > `/24` > `/28`)
- You do NOT need to calculate CIDR ranges for AZ-204. Just know that VNets have address spaces and subnets divide them

### What's Inside a VNet vs. What's Outside

This is the critical distinction for AZ-204:

```
┌─────────────────────────────────────────────────────┐
│                    INSIDE a VNet                     │
│                                                     │
│   Virtual Machines (VMs)                            │
│   Azure SQL Managed Instance                        │
│   App Service Environment (Isolated tier)           │
│   Private Endpoints (bringing services in)          │
│   Azure Kubernetes Service (AKS) nodes              │
│                                                     │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│                   OUTSIDE a VNet                     │
│              (on the public internet)                │
│                                                     │
│   App Service (Free, Basic, Standard, Premium)      │
│   Azure Functions (Consumption plan)                │
│   Azure Key Vault (by default)                      │
│   Azure Storage (by default)                        │
│   Azure Cosmos DB (by default)                      │
│   Azure SQL Database (by default)                   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

> 🚨 **EXAM ALERT**
>
> **App Service and Azure Functions on the Consumption plan are NOT inside a VNet by default.** They sit on the public internet. This is why you need VNet Integration (Section 4) to let them reach private resources, and Private Endpoints (Section 5) to bring services into the VNet. The exam tests whether you understand this distinction.

💡 **KEY CONCEPT**

A VNet is a private network in Azure. Most PaaS services (App Service, Functions, Key Vault, Storage) are **outside** the VNet by default. Getting them to communicate privately requires VNet Integration or Private Endpoints — the two features tested on AZ-204.

---

## Section 3: Network Security Groups (NSG)

**Exam Relevance:** ⚪ CONTEXT ONLY — NSGs are rarely tested directly on AZ-204, but understanding them helps you troubleshoot lab connectivity issues.

### The Problem

You have resources inside a VNet (say, a virtual machine running a database). You want to control **who can talk to whom** — allow your app server to reach the database on port 1433, but block everything else.

### The Analogy: A Bouncer at Every Door

An NSG is like a **bouncer** stationed at the entrance to a subnet or a specific resource:

- The bouncer has a **list of rules** (the guest list)
- Each rule says: *"If someone matching THIS description tries to go IN or OUT, ALLOW or DENY them"*
- The bouncer checks rules in **priority order** — lowest number first
- First matching rule wins — the bouncer stops checking after a match

### Technical Definition

A **Network Security Group (NSG)** is a set of firewall rules that filter network traffic to and from Azure resources inside a VNet.

### Rule Components

Every NSG rule has these parts:

| Component | What It Means | Example |
|---|---|---|
| **Priority** | Order of evaluation (100–4096). Lower = checked first | `100` (checked before `200`) |
| **Direction** | Is traffic coming IN or going OUT? | `Inbound` or `Outbound` |
| **Source** | Where is the traffic coming FROM? | `10.0.1.0/24` or `*` (anywhere) |
| **Destination** | Where is the traffic going TO? | `10.0.2.0/24` or `*` (anywhere) |
| **Port** | Which port number? | `1433` (SQL Server), `443` (HTTPS) |
| **Protocol** | TCP, UDP, or both? | `TCP` |
| **Action** | Allow or block? | `Allow` or `Deny` |

### Developer Context: The Port 1433 Check

If your application can't connect to an Azure SQL database running on a VM inside a VNet, here's the first thing to check:

**Is there an NSG blocking port 1433 (the default SQL Server port)?**

```
┌──────────────┐         NSG Rule Check          ┌──────────────┐
│  App Server  │ ──── Port 1433, TCP ────────▶   │  Database VM │
│  10.0.1.5    │                                  │  10.0.2.10   │
└──────────────┘                                  └──────────────┘
                         ▲
                         │
                   Is there an NSG rule
                   that ALLOWs TCP 1433
                   from 10.0.1.0/24 to
                   10.0.2.0/24?
                         │
                   ┌─────┴─────┐
                   │ YES → ✅   │
                   │ NO  → ❌   │
                   └───────────┘
```

📝 **NOTE**

For AZ-204, you won't be asked to create NSG rules. But if a lab exercise involves a database VM that your app can't reach, an NSG blocking the port is a common cause. Now you know what to check.

---

✅ **CHECKPOINT — Sections 1–3**

Before continuing, make sure you can answer:

1. What are the two most common networking errors in AZ-204 labs? *(App can't reach private DB; Key Vault blocks your IP)*
2. Are App Service and Functions (Consumption) inside a VNet by default? *(No — they're on the public internet)*
3. What does an NSG do? *(Filters traffic in/out of a VNet using priority-ordered rules)*
4. What port does SQL Server use by default? *(1433)*

---

## Section 4: VNet Integration for App Service and Functions

**Exam Relevance:** 🔴 HIGH — This is one of the most frequently tested networking concepts on AZ-204.

### The Problem

Remember Scenario A from Section 1? Let's draw it:

```
┌──────────────────┐                    ┌─────────────────────────┐
│                  │                    │       VNet              │
│   App Service    │ ───── ❌ ─────▶   │   ┌──────────────┐     │
│  (public internet)│  Can't reach!     │   │   Database    │     │
│                  │                    │   │  (private)    │     │
└──────────────────┘                    │   └──────────────┘     │
                                        └─────────────────────────┘
```

Your App Service lives on the **public internet**. Your database is inside a **VNet** (private network). The database doesn't accept connections from the public internet — that's the whole point of putting it in a VNet.

**How do you let your App Service reach the database without exposing the database to the internet?**

### The Solution: VNet Integration

**VNet Integration** creates a secure tunnel from your App Service **outward** into a VNet. Think of it as giving your App Service a **back door** that opens directly into the private office building (VNet).

```
┌──────────────────┐                    ┌──────────────────────────┐
│                  │    VNet            │        VNet              │
│   App Service    │  Integration       │   ┌──────────────┐      │
│  (public internet)│ ═══════════▶      │   │   Database    │      │
│                  │   (outbound        │   │  (private)    │      │
│                  │    tunnel)         │   └──────────────┘      │
└──────────────────┘                    └──────────────────────────┘

                    ✅ App Service can NOW reach the database
                       through the VNet Integration tunnel
```

### The Analogy: A Private Hallway

Imagine your App Service is in a public shopping mall (the internet). The database is in a private office building next door (the VNet). VNet Integration builds a **private hallway** from the mall into the office building. Your app can walk through the hallway to reach the database — but people in the office building still can't walk through the hallway into the mall. The hallway is **one-way outbound only**.

> 🚨 **EXAM ALERT — THE MOST IMPORTANT NETWORKING FACT FOR AZ-204**
>
> **VNet Integration is OUTBOUND ONLY.**
>
> This means three things:
> 1. ✅ Your app CAN reach resources **inside** the VNet (databases, VMs, private endpoints)
> 2. ❌ VNet Integration does NOT place your app inside the VNet
> 3. ❌ VNet Integration does NOT restrict or control inbound traffic to your app
>
> **The exam trap:** A question describes wanting to prevent public users from reaching an App Service. VNet Integration is offered as an option. **It is WRONG.** VNet Integration only controls outbound traffic. To restrict inbound traffic, you need **Access Restrictions** (Section 7) or an **App Service Environment** (Section 7).

### Requirements — What Tiers Support VNet Integration?

| Service | Minimum Tier for VNet Integration | Tiers That Do NOT Support It |
|---|---|---|
| **App Service** | Standard (S1+) | Free (F1), Shared (D1), Basic (B1–B3) |
| **Azure Functions** | Premium or Dedicated (App Service Plan) | ❌ **Consumption plan** — no VNet Integration |

> 🚨 **EXAM ALERT**
>
> **Azure Functions on the Consumption plan cannot use VNet Integration.** If a question describes a Function app that needs to reach resources in a VNet, the answer is to use the **Premium plan** or a **Dedicated (App Service) plan** — never Consumption.

🔗 **MODULE LINK:** You'll see VNet Integration again in Module 1 (App Service — Section 8: Networking) and Module 2 (Azure Functions — Section 4: Hosting Plans). The Premium plan requirement for Functions is tested frequently.

### How VNet Integration Works — The Complete Picture

```
                        PUBLIC INTERNET
    ┌──────────────────────────────────────────────────┐
    │                                                  │
    │   Users browse to: https://myapp.azurewebsites.net
    │         │                                        │
    │         ▼                                        │
    │   ┌──────────────────┐                           │
    │   │   App Service    │                           │
    │   │   (Standard+)    │                           │
    │   └────────┬─────────┘                           │
    │            │                                     │
    └────────────┼─────────────────────────────────────┘
                 │  VNet Integration
                 │  (outbound only)
                 ▼
    ┌────────────────────────────────────────────────────┐
    │                    VNet (10.0.0.0/16)              │
    │                                                    │
    │   ┌─────────────────┐    ┌──────────────────┐     │
    │   │  Integration    │    │  Database Subnet  │     │
    │   │  Subnet         │───▶│  10.0.2.0/24     │     │
    │   │  10.0.1.0/24    │    │  ┌────────────┐  │     │
    │   │  (delegated to  │    │  │ SQL Server │  │     │
    │   │   App Service)  │    │  │ 10.0.2.10  │  │     │
    │   └─────────────────┘    │  └────────────┘  │     │
    │                          └──────────────────┘     │
    └────────────────────────────────────────────────────┘
```

**Key points from this diagram:**

1. Users still access the App Service via its **public URL** — VNet Integration doesn't change that
2. The App Service sends outbound traffic through a **dedicated integration subnet**
3. That subnet is **delegated** to Microsoft.Web/serverFarms — meaning only App Service can use it
4. From the integration subnet, traffic can reach other subnets in the VNet (like the database subnet)

💡 **KEY CONCEPT**

VNet Integration = **your app reaches OUT into the VNet**. It's a one-way outbound tunnel. It does not protect your app from inbound traffic, and it does not place your app inside the VNet.

---

## Section 5: Private Endpoints — Bringing Services Into Your VNet

**Exam Relevance:** 🔴 HIGH — Private Endpoints are tested alongside VNet Integration. The exam tests whether you know which direction each one works.

### The Problem

Azure PaaS services like Key Vault, Storage Accounts, and Cosmos DB are accessible via **public URLs** by default:

```
https://mykeyvault.vault.azure.net         ← public URL
https://mystorage.blob.core.windows.net    ← public URL  
https://mycosmosdb.documents.azure.com     ← public URL
```

Anyone on the internet can **attempt** to reach these URLs. Authentication stops unauthorised access, but the **network path** goes over the public internet. For compliance and security, some organisations require that traffic to these services **never touches the public internet**.

**How do you make an Azure PaaS service accessible only from inside your private network?**

### The Solution: Private Endpoint

A **Private Endpoint** creates a **private IP address inside your VNet** that maps to an Azure PaaS service. Instead of reaching Key Vault via its public URL over the internet, your resources reach it via a private IP inside the VNet.

### The Analogy: Bringing the Post Office Inside Your Building

Normally, to mail a letter (access Key Vault), you walk outside your private office building (VNet), cross the public street (internet), and go to the post office (Key Vault's public endpoint).

A Private Endpoint is like **opening a post office branch inside your building**. Now you never leave the building. The mail still gets to the same place, but your path is entirely private.

### Before and After

**Before Private Endpoint:**
```
┌───────────────────┐          PUBLIC           ┌──────────────┐
│    VNet           │         INTERNET           │              │
│  ┌─────────┐     │    ┌─────────────────┐     │  Key Vault   │
│  │   VM    │─────┼───▶│  Public route   │────▶│  (public IP) │
│  └─────────┘     │    └─────────────────┘     │              │
└───────────────────┘     Traffic goes over      └──────────────┘
                          the public internet
```

**After Private Endpoint:**
```
┌────────────────────────────────────────────────┐
│                     VNet                       │
│                                                │
│  ┌─────────┐         ┌───────────────────┐    │
│  │   VM    │────────▶ │  Private Endpoint │    │
│  └─────────┘         │  10.0.3.5         │    │
│                      │  (maps to         │    │
│                      │   Key Vault)      │    │
│                      └───────────────────┘    │
│                                                │
│   Traffic stays ENTIRELY inside the VNet       │
│   Key Vault's public endpoint can be DISABLED  │
└────────────────────────────────────────────────┘
```

### The Critical Contrast: VNet Integration vs. Private Endpoint

This is the **single most important networking distinction** for AZ-204. They solve **opposite problems**:

| Feature | Direction | What It Does | Analogy |
|---|---|---|---|
| **VNet Integration** | App → VNet (OUTBOUND) | Lets your app reach **out** into a VNet | Building a hallway from the mall into the office building |
| **Private Endpoint** | Service → VNet (INBOUND) | Brings a service **into** your VNet | Opening a branch office inside the building |

### Using Both Together

In practice, you often need **both** — VNet Integration to let your app reach into the VNet, and a Private Endpoint to bring the target service into the VNet:

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   PUBLIC INTERNET                                       │
│   ┌──────────────────┐                                  │
│   │   App Service    │                                  │
│   │   (Standard+)    │                                  │
│   └────────┬─────────┘                                  │
│            │                                            │
└────────────┼────────────────────────────────────────────┘
             │  VNet Integration (outbound)
             ▼
┌────────────────────────────────────────────────────────┐
│                     VNet                               │
│                                                        │
│  ┌────────────────┐         ┌───────────────────┐     │
│  │  Integration   │         │  Private Endpoint  │     │
│  │  Subnet        │────────▶│  for Key Vault    │     │
│  │  (App Service  │         │  10.0.3.5         │     │
│  │   traffic      │         │                   │     │
│  │   enters here) │         │  (maps to         │     │
│  └────────────────┘         │   mykeyvault      │     │
│                             │   .vault.azure.   │     │
│                             │   net)            │     │
│                             └───────────────────┘     │
│                                                        │
│  App Service reaches Key Vault entirely over           │
│  the private network. No public internet involved.     │
└────────────────────────────────────────────────────────┘
```

> 🚨 **EXAM ALERT — KNOW THE DIRECTION**
>
> **Traffic FROM your app INTO the VNet = VNet Integration.**
> **A SERVICE brought INTO your VNet = Private Endpoint.**
>
> The exam will describe a scenario and ask which feature to use. The trick is identifying the **direction**:
> - *"App Service needs to connect to a database in a VNet"* → **VNet Integration** (app reaching out)
> - *"Key Vault should only be accessible from within the VNet"* → **Private Endpoint** (service brought in)
> - *"App Service needs to connect to Key Vault privately"* → **Both** (VNet Integration for the app + Private Endpoint for Key Vault)

💡 **KEY CONCEPT**

VNet Integration and Private Endpoints work in **opposite directions**. VNet Integration = app reaches out. Private Endpoint = service comes in. For full private connectivity, you typically need both.

---

## Section 6: Service Endpoints — The Lighter Alternative

**Exam Relevance:** 🟡 MEDIUM — The exam occasionally asks you to compare Service Endpoints and Private Endpoints.

### The Problem

Private Endpoints provide maximum security, but they require creating a private IP address, managing DNS, and paying per endpoint. What if you want **better security than the default public access** but don't need the full Private Endpoint setup?

### The Solution: Service Endpoints

A **Service Endpoint** extends your VNet identity to an Azure service. The service still has its public IP address, but traffic from your VNet travels over the **Azure backbone network** (Microsoft's private network) instead of the public internet.

### The Analogy: A VIP Express Lane

If Private Endpoints are like opening a branch office inside your building (the service moves to you), then Service Endpoints are like having a **VIP express lane** on the public road. You still go outside your building to reach the post office, but you take a private highway that the public can't access. The post office is still in its public location — but your trip is faster and more secure.

### Comparison Table: Service Endpoint vs. Private Endpoint

| Feature | Service Endpoint | Private Endpoint |
|---|---|---|
| **How it works** | Extends VNet identity to the service | Creates a private IP inside VNet for the service |
| **Service IP** | Still uses **public** IP | Gets a **private** IP in your VNet |
| **Traffic path** | Azure backbone (not public internet) | Entirely within VNet |
| **DNS changes** | None needed | Requires private DNS zone |
| **Cost** | **Free** | Per-hour charge + data processing charge |
| **On-premises access** | ❌ Cannot be used from on-prem | ✅ Accessible from on-prem via VPN/ExpressRoute |
| **Security level** | Good — traffic on Azure backbone | Maximum — service has private IP only |
| **Setup complexity** | Low — enable on subnet | Higher — create endpoint, configure DNS |
| **When to use** | Good security at no extra cost | Compliance requires no public exposure |

### Quick Decision Guide

```
Do you need the service to have NO public IP at all?
├── YES → Private Endpoint
└── NO
    └── Do you need on-premises access to the service?
        ├── YES → Private Endpoint
        └── NO → Service Endpoint (simpler, free)
```

📝 **NOTE**

For AZ-204, you need to know the **difference** between Service Endpoints and Private Endpoints. The exam won't ask you to configure either from scratch, but it will present a scenario and ask you to choose the right one. Use the table above to decide.

---

✅ **CHECKPOINT — Sections 4–6**

Before continuing, make sure you can answer:

1. What direction does VNet Integration work? *(Outbound only — app reaches out into VNet)*
2. What direction does a Private Endpoint work? *(Brings a service into the VNet — service gets a private IP)*
3. What is the minimum App Service tier for VNet Integration? *(Standard S1+)*
4. Can Azure Functions on the Consumption plan use VNet Integration? *(No — requires Premium or Dedicated)*
5. What is the main difference between a Service Endpoint and a Private Endpoint? *(Service Endpoint: service keeps public IP, traffic uses Azure backbone. Private Endpoint: service gets a private IP inside VNet)*

---

## Section 7: Public and Private Access to App Service

**Exam Relevance:** 🟡 MEDIUM — Access restrictions and App Service Environment appear in scenario questions.

### The Default: Publicly Accessible

When you create an App Service, it gets a public URL like `https://myapp.azurewebsites.net`. **Anyone on the internet** can reach this URL. Authentication may stop them from using the app, but the network connection itself is open.

For many applications, this is fine — you want users to access your web app from anywhere. But what if you need to **restrict who can even reach the app** at the network level?

### Option 1: Access Restrictions (IP-Based Firewall)

**Access Restrictions** let you create an **allow/deny list** of IP addresses or VNet subnets that can reach your App Service. Think of it as a bouncer at the front door of your app.

**How it works:**

```
┌───────────────────────────────────────────────────────┐
│                    App Service                        │
│                  Access Restrictions                  │
│                                                       │
│  Rule 1: ALLOW  10.0.0.0/16     (office VNet)  ✅    │
│  Rule 2: ALLOW  203.0.113.50    (your home IP)  ✅    │
│  Rule 3: DENY   *               (everything else) ❌  │
│                                                       │
└───────────────────────────────────────────────────────┘

   Office VNet    →  ✅ Allowed in
   Your home IP   →  ✅ Allowed in
   Random hacker  →  ❌ Blocked at network level
```

**Key facts:**
- Found in: **App Service → Networking → Access Restrictions**
- Rules are evaluated by **priority** (lowest number first) — just like NSG rules
- You can allow specific **IP addresses**, **IP ranges**, or **VNet subnets**
- Works on **inbound** traffic — this is how you restrict who can reach your app

📝 **NOTE**

Remember: **VNet Integration** controls outbound traffic (your app reaching out). **Access Restrictions** control inbound traffic (who can reach your app). They solve different problems and are often used together.

### Option 2: App Service Environment (ASE) — Maximum Isolation

For organisations that need the **highest level of network isolation**, there's the App Service Environment (ASE).

**What makes ASE different:**

| Feature | Regular App Service | App Service Environment (ASE) |
|---|---|---|
| **Network location** | Public internet | **Fully inside a VNet** |
| **Tier** | Free through Premium v3 | **Isolated v2 only** |
| **Isolation** | Shared infrastructure | Dedicated infrastructure |
| **Inbound control** | Access Restrictions | Full VNet control (NSGs, etc.) |
| **Cost** | $ to $$$ | $$$$ (significantly higher) |
| **Use case** | Most applications | High-compliance, maximum security |

**The Analogy:**

- Regular App Service with Access Restrictions = putting a bouncer at a restaurant in a public shopping mall
- App Service Environment = building your own **private restaurant** inside a **gated compound**

```
Regular App Service:                App Service Environment:
┌──────────────────────┐           ┌──────────────────────────┐
│   PUBLIC INTERNET    │           │         VNet             │
│                      │           │                          │
│  ┌────────────────┐  │           │   ┌────────────────┐    │
│  │  App Service   │  │           │   │  App Service   │    │
│  │  + Access      │  │           │   │  Environment   │    │
│  │  Restrictions  │  │           │   │  (Isolated)    │    │
│  └────────────────┘  │           │   └────────────────┘    │
│                      │           │                          │
└──────────────────────┘           └──────────────────────────┘
     Bouncer at the                    Private compound
     public door                       with its own walls
```

> 🚨 **EXAM ALERT**
>
> **App Service Environment uses the Isolated tier.** If a question asks about running an App Service *inside* a VNet (not just connecting outbound via VNet Integration), the answer is **App Service Environment (Isolated tier)**. Don't confuse VNet Integration (outbound tunnel, Standard+) with ASE (app actually lives inside the VNet, Isolated tier).

💡 **KEY CONCEPT**

There's a spectrum of App Service network isolation. From least to most secure:

1. **Default** — Public URL, no restrictions
2. **Access Restrictions** — IP/subnet allow list on inbound traffic
3. **VNet Integration** — Outbound traffic goes through VNet (does NOT affect inbound)
4. **App Service Environment** — App lives entirely inside a VNet (Isolated tier, highest cost)

---

## Section 8: Key Vault Networking — The Most Common AZ-204 Networking Scenario

**Exam Relevance:** 🔴 HIGH — Key Vault networking questions combine security (RBAC) with networking (firewall). The exam tests both layers.

### Why Key Vault Networking Matters Most

Of all the networking scenarios in AZ-204, **Key Vault** is the one you'll encounter most often — both in labs and on the exam. This is because:

1. Key Vault is used in almost every module (storing connection strings, API keys, certificates)
2. Key Vault has **two separate security layers** that both must allow access
3. The error messages are confusing if you don't know about both layers

### Key Vault's Two Security Layers

Think of Key Vault as a **bank vault** with two separate security checkpoints:

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│   CHECKPOINT 1: NETWORK LAYER (Firewall)               │
│   "Can your network connection reach Key Vault?"       │
│                                                        │
│   ┌──────────────────────────────────────────────┐     │
│   │  Options:                                    │     │
│   │  • Allow all networks (public access)        │     │
│   │  • Allow specific VNets and IPs only         │     │
│   │  • Disable public access (Private Endpoint   │     │
│   │    only)                                     │     │
│   └──────────────────────────────────────────────┘     │
│                         │                              │
│                    Pass? ▼                              │
│                                                        │
│   CHECKPOINT 2: ACCESS CONTROL (RBAC)                  │
│   "Do you have permission to perform this action?"     │
│                                                        │
│   ┌──────────────────────────────────────────────┐     │
│   │  RBAC Roles:                                 │     │
│   │  • Key Vault Secrets User (read secrets)     │     │
│   │  • Key Vault Secrets Officer (manage secrets)│     │
│   │  • Key Vault Administrator (full control)    │     │
│   │  • Key Vault Crypto User (key operations)    │     │
│   └──────────────────────────────────────────────┘     │
│                         │                              │
│                    Pass? ▼                              │
│                                                        │
│              ✅ ACCESS GRANTED — read your secret       │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**You must pass BOTH checkpoints.** If either one denies you, you're blocked.

> 🚨 **EXAM ALERT — TWO LAYERS, BOTH REQUIRED**
>
> **Key Vault has TWO separate access control layers:**
> 1. **Network layer** (firewall) — controls which **network locations** can reach Key Vault
> 2. **RBAC layer** (permissions) — controls which **identities** can perform which **actions**
>
> **The exam trap:** A scenario says a developer has the correct RBAC role but still gets "Forbidden". The correct answer involves the **network firewall** blocking their IP. Or vice versa: their IP is allowed but they lack the correct RBAC role. **Always check both layers.**

### Key Vault Firewall Settings

Key Vault has three firewall configurations. You find them in **Key Vault → Networking**:

| Setting | What It Means | When to Use |
|---|---|---|
| **Allow public access from all networks** | Anyone on the internet can attempt to connect (RBAC still required) | Development, quick labs |
| **Allow public access from specific virtual networks and IP addresses** | Only connections from listed VNets/IPs are allowed | Production with known consumers |
| **Disable public access** | Only Private Endpoints can reach Key Vault | Maximum security, compliance |

### The Most Common Lab Error — And How to Fix It

When you create a Key Vault in some Azure subscriptions, the default firewall setting may restrict access. You run a command like:

```bash
az keyvault secret set \
  --vault-name myKeyVault \    # ← your Key Vault name
  --name MySecret \            # ← name for the secret
  --value "SuperSecretPassword123"  # ← the secret value
```

And you get:

```text
(Forbidden) Client address is not authorized and caller is not a trusted service.
Code: Forbidden
Message: Client address is not authorized and caller is not a trusted service.
```

**This error means:** Your RBAC permissions might be fine, but the Key Vault **firewall** is blocking your IP address.

### Fix: Add Your IP to the Firewall

**Option A: Via Azure Portal**

1. Navigate to your Key Vault in the Azure Portal
2. Click **Networking** in the left menu (under Settings)
3. Select **"Allow public access from specific virtual networks and IP addresses"**
4. Under **Firewall**, check **"Add your client IP address"**
5. Click **Apply**
6. Wait 30 seconds, then retry your command

**Option B: Via Azure CLI**

```bash
# Step 1: Find your public IP address
curl ifconfig.me
# Returns something like: 203.0.113.50

# Step 2: Add your IP to the Key Vault firewall
az keyvault network-rule add \
  --name myKeyVault \              # ← your Key Vault name
  --ip-address 203.0.113.50/32    # ← your IP with /32 (single IP)
```

**Option C: Allow All Networks (Quick Lab Fix)**

```bash
# Allow all networks — use for labs only, not production
az keyvault update \
  --name myKeyVault \                       # ← your Key Vault name
  --resource-group myResourceGroup \        # ← your resource group
  --default-action Allow                    # ← allow connections from any network
```

💰 **COST WARNING**

Private Endpoints for Key Vault incur a per-hour charge (approximately $0.01/hour = ~$7.30/month). For AZ-204 labs, use the "allow specific IPs" or "allow all networks" options instead. Only use Private Endpoints when the lab specifically requires them. Always clean up Private Endpoints after the lab.

### Key Vault Networking + App Service — The Complete Scenario

Here's the most common AZ-204 networking scenario, bringing everything together:

```
SCENARIO: App Service needs to read secrets from Key Vault.
          Key Vault has public access disabled.
          What do you need?

Step 1: VNet Integration on App Service (Standard+ tier)
        → App Service can reach into the VNet

Step 2: Private Endpoint for Key Vault in the VNet
        → Key Vault gets a private IP inside the VNet

Step 3: Managed Identity on App Service
        → App Service has an identity for RBAC

Step 4: RBAC role assignment: "Key Vault Secrets User"
        → The identity has permission to read secrets

RESULT: App Service → VNet Integration → VNet → Private Endpoint → Key Vault
        Network layer ✅ (Private Endpoint in VNet)
        RBAC layer ✅ (Key Vault Secrets User role)
```

```
┌────────────────┐         ┌─────────────────────────────────────┐
│  PUBLIC        │         │              VNet                   │
│  INTERNET      │         │                                     │
│                │  VNet   │  ┌─────────┐    ┌───────────────┐  │
│  ┌──────────┐  │ Integr. │  │Integr.  │    │ Private       │  │
│  │App       │──┼────────▶│  │Subnet   │───▶│ Endpoint      │  │
│  │Service   │  │         │  │         │    │ for Key Vault │  │
│  │+Managed  │  │         │  └─────────┘    │ 10.0.3.5      │  │
│  │ Identity │  │         │                 └───────────────┘  │
│  └──────────┘  │         │                                     │
│                │         │  Key Vault is only accessible       │
└────────────────┘         │  via this Private Endpoint          │
                           └─────────────────────────────────────┘
```

💡 **KEY CONCEPT**

Key Vault access requires passing TWO checkpoints: the network firewall AND RBAC permissions. If either one blocks you, access is denied. The most common lab error is having correct RBAC but a firewall that blocks your IP.

🔗 **MODULE LINK:** Key Vault is covered in depth in Module 7 (Implement Secure Azure Solutions). The networking concepts from this section will be essential for that module's labs.

---

✅ **CHECKPOINT — Sections 7–8**

Before continuing, make sure you can answer:

1. What are the three options for Key Vault firewall settings? *(Allow all, allow specific VNets/IPs, disable public access)*
2. If you have the correct RBAC role but Key Vault returns "Forbidden", what's the likely cause? *(Firewall is blocking your IP)*
3. What tier of App Service puts the app fully inside a VNet? *(Isolated — App Service Environment)*
4. How is Access Restrictions different from VNet Integration? *(Access Restrictions = inbound IP filtering; VNet Integration = outbound VNet tunnel)*

---

## Diagram Exercise — Test Your Understanding

### Instructions

For each scenario below, **identify which networking features are needed** and **draw the correct network diagram** using the text-diagram style from this document. Then compare your answer with the solution provided.

---

### Scenario 1: Basic Private Database Access

> **Situation:** You have an App Service (Standard S1 tier) that needs to connect to an Azure SQL Database. The SQL Database has been configured with a Private Endpoint in a VNet. The App Service must reach the database privately.
>
> **Question:** What networking feature(s) does the App Service need?

**Think about it before scrolling down...**

<details>

**Answer: VNet Integration**

The App Service needs **VNet Integration** to reach outbound into the VNet where the SQL Database's Private Endpoint lives.

```
┌─────────────────┐        ┌──────────────────────────────────┐
│  PUBLIC INTERNET │        │              VNet                │
│                  │        │                                  │
│  ┌────────────┐  │  VNet  │  ┌──────────┐  ┌────────────┐  │
│  │App Service │──┼──Integ.┼─▶│ Integr.  │─▶│ Private    │  │
│  │ (S1 tier)  │  │        │  │ Subnet   │  │ Endpoint   │  │
│  └────────────┘  │        │  └──────────┘  │ for SQL DB │  │
│                  │        │                └────────────┘  │
└─────────────────┘        └──────────────────────────────────┘

Features used:
✅ VNet Integration on App Service (outbound into VNet)
✅ Private Endpoint on SQL Database (service brought into VNet)
```

</details>

---

### Scenario 2: Restricted App Service + Private Key Vault

> **Situation:** You have an App Service that reads secrets from Key Vault. Security requirements:
> - Key Vault must NOT be accessible from the public internet
> - The App Service should only be accessible from your corporate network (IP range: 10.50.0.0/16)
> - The App Service uses Managed Identity for authentication
>
> **Question:** What networking features do you need? List all of them.

**Think about it before scrolling down...**

<details>

**Answer: Four features needed**

```
CORPORATE NETWORK (10.50.0.0/16)
         │
         │  Access Restriction:
         │  ALLOW 10.50.0.0/16
         ▼
┌─────────────────┐        ┌──────────────────────────────────┐
│  PUBLIC INTERNET │        │              VNet                │
│                  │        │                                  │
│  ┌────────────┐  │  VNet  │  ┌──────────┐  ┌────────────┐  │
│  │App Service │──┼──Integ.┼─▶│ Integr.  │─▶│ Private    │  │
│  │(Standard+) │  │        │  │ Subnet   │  │ Endpoint   │  │
│  │+Managed ID │  │        │  └──────────┘  │ for KV     │  │
│  │+Access     │  │        │                │ (10.0.3.5) │  │
│  │ Restrict.  │  │        │                └────────────┘  │
│  └────────────┘  │        │                                  │
└─────────────────┘        └──────────────────────────────────┘

Features used:
1. ✅ Access Restrictions — restrict INBOUND to corporate IPs only
2. ✅ VNet Integration — allow OUTBOUND into VNet (Standard+ required)
3. ✅ Private Endpoint for Key Vault — bring Key Vault into VNet
4. ✅ Managed Identity + RBAC — "Key Vault Secrets User" role
     (Network layer ✅ + RBAC layer ✅ = access granted)

Key Vault firewall: "Disable public access" (Private Endpoint only)
```

</details>

---

### Scenario 3: Azure Function with Consumption Plan

> **Situation:** Your team has an Azure Function on the **Consumption plan**. The Function needs to read secrets from a Key Vault. The Key Vault firewall is set to "Allow public access from all networks." The Function has a Managed Identity with the "Key Vault Secrets User" role. But the Function is getting "Forbidden" errors.
>
> **Question:** What's wrong? How do you fix it?

**Think about it before scrolling down...**

<details>

**Answer: This is a trick question — the networking SHOULD work**

```
Analysis:
- Key Vault firewall: "Allow all networks" ✅
  (The Function can reach Key Vault from the public internet)
- RBAC: "Key Vault Secrets User" role ✅
  (The Function has permission to read secrets)
- Consumption plan: No VNet Integration, but NOT NEEDED here
  (Key Vault allows public access, so no VNet route is required)

If the Function is STILL getting "Forbidden" errors, the issue
is NOT networking. Check:

1. Is the Managed Identity ENABLED on the Function App?
2. Is the RBAC role assigned to the CORRECT identity?
   (System-assigned vs user-assigned?)
3. Is there a delay? RBAC assignments can take up to 10 minutes
   to propagate.
4. Is the Managed Identity assigned at the correct SCOPE?
   (Key Vault level, not subscription or resource group)

HOWEVER — if the question asks how to make this work
PRIVATELY (not over the public internet), THEN you need:
- Upgrade to Premium plan (Consumption can't do VNet Integration)
- Add VNet Integration
- Create a Private Endpoint for Key Vault
- Disable public access on Key Vault

This changes the diagram to:

┌────────────────┐         ┌──────────────────────────────┐
│  ┌──────────┐  │  VNet   │           VNet               │
│  │ Function │──┼──Integ.─┼─▶ ┌─────────────────┐       │
│  │ (Premium)│  │         │   │Private Endpoint │       │
│  └──────────┘  │         │   │ for Key Vault   │       │
└────────────────┘         │   └─────────────────┘       │
                           └──────────────────────────────┘
```

**The lesson:** Always check BOTH layers (network + RBAC). And remember: Consumption plan Functions cannot use VNet Integration — if private access is needed, you must upgrade to Premium.

</details>

---

## What We Covered — Checklist

Use this checklist to verify you understand every concept in this document. If any item is unclear, re-read the corresponding section.

### Section 1: Why Developers Need Networking
- [ ] I can describe two real AZ-204 lab scenarios where networking causes failures
- [ ] I know the three questions to ask when troubleshooting networking: Where is my app? Where is the service? Is there a firewall?

### Section 2: Virtual Networks (VNet)
- [ ] I can explain what a VNet is using the private office building analogy
- [ ] I know that VNets have address spaces divided into subnets
- [ ] I know that App Service and Functions (Consumption) are NOT inside a VNet by default

### Section 3: Network Security Groups (NSG)
- [ ] I can explain what an NSG does using the bouncer analogy
- [ ] I know the components of an NSG rule: priority, direction, source, destination, port, protocol, action
- [ ] I know to check port 1433 if an app can't reach a SQL database

### Section 4: VNet Integration
- [ ] I know that VNet Integration is **OUTBOUND ONLY**
- [ ] I know it does NOT place the app inside the VNet
- [ ] I know it does NOT restrict inbound traffic
- [ ] I know the minimum tier: Standard+ for App Service, Premium/Dedicated for Functions
- [ ] I know that Functions on the Consumption plan CANNOT use VNet Integration

### Section 5: Private Endpoints
- [ ] I can explain what a Private Endpoint does: creates a private IP for a service inside a VNet
- [ ] I can clearly distinguish VNet Integration (app reaches out) from Private Endpoint (service comes in)
- [ ] I know when to use both together

### Section 6: Service Endpoints
- [ ] I know the difference: Service Endpoint = VNet identity to service, still public IP. Private Endpoint = private IP inside VNet
- [ ] I can choose between them based on requirements (compliance = Private Endpoint, simple + free = Service Endpoint)

### Section 7: App Service Networking
- [ ] I know the four levels of App Service network isolation: Default → Access Restrictions → VNet Integration → ASE
- [ ] I know that ASE uses the Isolated tier and places the app fully inside a VNet

### Section 8: Key Vault Networking
- [ ] I know Key Vault has TWO security layers: Network (firewall) + RBAC (permissions)
- [ ] I know the three Key Vault firewall settings: allow all, allow specific, disable public
- [ ] I can fix the "Client address is not authorized" error by adding my IP to the firewall
- [ ] I can describe the complete scenario: App Service + VNet Integration + Private Endpoint + Managed Identity + RBAC → Key Vault

---

> 🔗 **MODULE LINKS — Where These Concepts Appear Next**
>
> | Concept | Where You'll See It Again |
> |---|---|
> | VNet Integration | Module 1 (App Service — Networking), Module 2 (Functions — Hosting Plans) |
> | Private Endpoints | Module 4 (Blob Storage), Module 5 (Cosmos DB), Module 7 (Key Vault) |
> | Key Vault Networking | Module 7 (Implement Secure Azure Solutions) — the primary module for Key Vault |
> | Access Restrictions | Module 1 (App Service — Configuration) |
> | App Service Environment | Module 1 (App Service — App Service Plans) |
> | Managed Identity + RBAC | Module 6 (Microsoft Identity Platform), Module 7 (Key Vault) |

---

*End of F07 — Azure Networking for Developers*
