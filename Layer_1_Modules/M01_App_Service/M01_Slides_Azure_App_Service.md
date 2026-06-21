# Module 1 Slide Deck: Azure App Service Web Apps

---

| | |
|---|---|
| **Module** | M01 — Azure App Service Web Apps |
| **Total Slides** | 22 |
| **Colour Scheme** | Azure Blue #0078D4, White, Light Grey |
| **Exam Alert Slides** | Red Background #D13438, White Text |
| **Format** | Each slide includes exact on-slide content + complete speaker notes |

---

## Slide 1: Title Slide

### On-Slide Content

# Azure App Service Web Apps
## Module 1

**Exam Domain:** Develop Azure Compute Solutions (25–30%)

**Learning Objectives:**
- Explain what Azure App Service is and the problem it solves
- Select the correct App Service Plan tier for a given scenario
- Deploy code using multiple methods including ZIP deploy
- Configure deployment slots for zero-downtime deployments
- Design autoscale rules for variable traffic
- Configure key App Service settings (Always On, ARR Affinity, HTTPS, TLS)
- Enable Easy Auth for built-in authentication
- Choose the correct networking feature for inbound vs outbound scenarios

**Prerequisites:** F01–F05

### Speaker Notes

Welcome to Module 1 — Azure App Service Web Apps. This is part of the Develop Azure Compute Solutions domain, which makes up 25 to 30 percent of the AZ-204 exam. That means roughly one quarter of all exam questions come from the compute domain, and App Service is the flagship service in this domain.

By the end of this module, you will be able to create, deploy, configure, and scale web applications on Azure App Service. You will understand deployment slots, autoscaling, networking, and Easy Auth — all of which are heavily tested on the exam.

This module assumes you have completed Foundation modules F01 through F05. You should already have an Azure account, be comfortable with the portal, know how to use the Azure CLI, and have completed Lab Zero. If any of those are unfamiliar, please go back and complete them first.

---

## Slide 2: The Problem This Service Solves

### On-Slide Content

# The Problem: Running a Web App Is Hard

**Without App Service, YOU manage:**

| You Must Handle | Effort |
|---|---|
| Physical/virtual servers | Provision, configure, maintain |
| Operating system | Install, patch monthly, upgrade |
| Networking | DNS, firewalls, load balancers |
| Security | SSL certificates, vulnerability patches |
| Monitoring | Alerts, log aggregation, uptime checks |
| Scaling | Buy more servers, configure load balancing |

**The developer's real job:** Writing application code

**The developer's unwanted job:** Everything above

### Speaker Notes

Before we talk about what App Service IS, let us understand the problem it solves. Imagine you wrote a web application — maybe a restaurant menu website. To make it available on the internet, you need a server running 24/7. You need to install and patch the operating system. You need to configure networking — DNS records, firewalls, maybe a load balancer. You need SSL certificates for HTTPS. You need monitoring so you know when something breaks at 3 AM.

All of that work has nothing to do with your restaurant menu. You are a developer — your job is writing the application, not managing infrastructure.

This is the exact problem that Azure App Service solves. It takes all of the infrastructure management off your plate and lets you focus on code.

---

## Slide 3: The Hotel Analogy

### On-Slide Content

# App Service = A Managed Hotel

```
YOU (the guest)                 AZURE (hotel management)
─────────────                   ──────────────────────
✅ Bring your luggage (code)    ✅ Building maintenance (servers)
✅ Arrange your room (config)   ✅ Plumbing & electricity (OS, patches)
✅ Come and go (deploy)         ✅ Security guards (firewalls, monitoring)
                                ✅ Room cleaning (scaling, health checks)
```

**You check in your code. Azure handles the building.**

### Speaker Notes

Here is the analogy I want you to remember throughout this module. Think of Azure App Service as a managed hotel for your application.

You are the guest. You arrive at the hotel with your luggage — that is your application code. You check in, arrange your room the way you like it — that is your configuration. You come and go as you please — that is deployments.

Azure is the hotel management. They built the building and maintain it. They handle plumbing and electricity — that is the operating system and patches. They have security guards — that is firewalls and monitoring. They clean rooms and manage capacity — that is scaling and health checks.

You never fix the plumbing. You never replace the roof. If a pipe bursts, hotel management handles it — not you. This is PaaS in action.

---

## Slide 4: PaaS — Platform as a Service

### On-Slide Content

# App Service Is PaaS

| Model | You Manage | Azure Manages | Analogy |
|---|---|---|---|
| **IaaS** | OS, runtime, app, data | Hardware, network | Empty apartment |
| **PaaS** ← App Service | **App + data only** | Hardware, OS, runtime, patches | **Hotel** |
| **SaaS** | Nothing | Everything | Gmail |

**PaaS = You deploy code. Azure handles everything else.**

**Supported Runtimes:** .NET · Node.js · Python · Java · PHP · Ruby

**Hosting:** Windows or Linux (chosen at creation, cannot change)

### Speaker Notes

Let us put this in formal terms. App Service is a Platform as a Service — PaaS. On the exam, you need to understand the three cloud service models and know that App Service is PaaS.

With IaaS — Infrastructure as a Service — you get a virtual machine. Azure gives you the hardware, but you install the operating system, configure the runtime, deploy your app, and manage everything on top. It is like renting an empty apartment — you furnish and maintain it yourself.

With PaaS — like App Service — you only manage your application and data. Azure handles everything else: hardware, operating system, runtime environment, security patches, and networking. It is our hotel analogy.

With SaaS — like Gmail or Microsoft 365 — you manage nothing. You just use the software. Azure handles everything.

App Service supports six runtimes: .NET, Node.js, Python, Java, PHP, and Ruby. You choose Windows or Linux when you create the App Service, and this choice cannot be changed after creation.

---

## Slide 5: App Service Plans — The Foundation

### On-Slide Content

# App Service Plans

## The Plan = Hardware. The App = Software.

```
┌─────────────────────────────────┐
│      App Service Plan (S1)      │  ← The building
│  ┌─────────┐  ┌─────────┐      │
│  │  App A   │  │  App B   │     │  ← Tenants
│  │ (website)│  │  (API)   │     │
│  └─────────┘  └─────────┘      │
│                                 │
│  CPU: 1 core  RAM: 1.75 GB     │
│  Storage: 50 GB                │
└─────────────────────────────────┘
```

- Multiple apps share one plan (and its resources)
- The **plan** determines available features
- Higher tier = more features + more power + higher cost

### Speaker Notes

The App Service Plan is one of the most important concepts for the exam. Think of the plan as the building, and your app as a tenant living in that building.

The plan defines the hardware: how much CPU, how much RAM, how much disk space. It also determines which features are available — things like deployment slots, autoscale, and VNet Integration.

You can run multiple apps on a single plan. They share CPU and memory. This is cost-effective for small apps, but if one app consumes too much CPU, it affects the others.

The key takeaway is: the plan controls features. If you need deployment slots, the PLAN must support slots — not just the app. This is a common exam trap.

---

## Slide 6: Plan Tier Table

### On-Slide Content

# App Service Plan Tiers — Memorise This Table

| Tier | SKUs | Compute | Slots | Autoscale | VNet | Max Instances |
|---|---|---|---|---|---|---|
| **Free** | F1 | Shared | ❌ | ❌ | ❌ | — |
| **Shared** | D1 | Shared | ❌ | ❌ | ❌ | — |
| **Basic** | B1–B3 | Dedicated | ❌ | ❌ | ❌ | 3 |
| **Standard** | S1–S3 | Dedicated | **5** ✅ | ✅ | ✅ | 10 |
| **Premium** | P1v3–P3v3 | Dedicated | **20** ✅ | ✅ | ✅ | 30 |
| **Isolated** | I1v2–I3v2 | Dedicated (ASE) | **20** ✅ | ✅ | ✅ | 100 |

**The magic line is between Basic and Standard.**

### Speaker Notes

This is the tier table. You must memorise this for the exam.

There are six tiers. Free and Shared are for testing only — they use shared compute, meaning your app shares CPU with other customers' apps. No custom SSL, no slots, no autoscale.

Basic gives you dedicated virtual machines. You get custom domains and SSL. But — and this is critical — Basic does NOT support deployment slots or autoscale. Many students assume Basic supports these because it is a paid tier. It does not.

Standard is where the serious features begin. Deployment slots — up to 5. Autoscale. VNet Integration. This is the minimum production tier for most apps.

Premium gives you faster hardware, up to 20 slots, and up to 30 instances.

Isolated is the highest tier. It runs in a dedicated App Service Environment — your app is fully inside a Virtual Network. Maximum isolation, maximum cost.

The magic line to remember is between Basic and Standard. Everything above Standard gets slots, autoscale, and VNet Integration. Everything below does not.

---

## Slide 7: 🚨 EXAM ALERT — Tier Requirements

### On-Slide Content

# 🚨 EXAM ALERT

## Features That Require Standard Tier or Higher

| Feature | Free | Shared | Basic | Standard+ |
|---|---|---|---|---|
| Deployment Slots | ❌ | ❌ | ❌ | ✅ |
| Autoscale | ❌ | ❌ | ❌ | ✅ |
| VNet Integration | ❌ | ❌ | ❌ | ✅ |
| Custom Domains | ❌ | ✅ | ✅ | ✅ |
| SSL Certificates | ❌ | ❌ | ✅ | ✅ |

## Common Exam Trap:
**Basic is a paid, dedicated tier — but it does NOT support slots, autoscale, or VNet Integration.**

### Speaker Notes

This is one of the highest-probability exam questions. When you see a question asking "what is the minimum tier for deployment slots" — the answer is Standard. When you see "what tier supports autoscale" — Standard.

The most common trap is Basic. Students see that Basic gives you dedicated VMs and SSL certificates and assume it supports all features. It does not. Basic does NOT support slots. Basic does NOT support autoscale. Basic does NOT support VNet Integration.

Remember: Standard is the minimum for the three big features — Slots, Autoscale, VNet. Write this down. Put it on a sticky note. It will be on your exam.

---

## Slide 8: Deployment Methods

### On-Slide Content

# How to Get Code to App Service

```
Your Code ──→ ? ──→ App Service
```

| Method | Type | Best For |
|---|---|---|
| **ZIP Deploy** | CLI push | Quick deployments |
| **Local Git** | Git push to App Service remote | Solo developers |
| **GitHub Actions** | CI/CD pipeline from GitHub | Teams using GitHub |
| **Azure DevOps** | CI/CD pipeline from Azure DevOps | Enterprise teams |
| **Docker Container** | Deploy container image | Containerised apps |
| **FTP** | File upload | Legacy only ⚠️ |

**Key CLI command:**
```bash
az webapp deployment source config-zip \
  --resource-group myRG --name myApp --src app.zip
```

### Speaker Notes

Once you have your App Service Plan and web app created, you need to get your code onto it. Azure supports six deployment methods.

ZIP deploy is the simplest and most commonly tested CLI method. You package your app as a ZIP file and upload it using the command shown here. Know this command for the exam.

Local Git lets you configure App Service as a Git remote. You push code directly with `git push azure main`. Simple for solo developers but does not support CI/CD.

GitHub Actions and Azure DevOps are CI/CD solutions — Continuous Integration, Continuous Deployment. Every time you push code to your repository, an automated pipeline builds, tests, and deploys your app. GitHub Actions is configured with a YAML file in your repository.

Docker lets you deploy container images from Azure Container Registry or Docker Hub. We will cover containers in detail in Module 3.

FTP is the legacy method. Do not use it for new applications, but know it exists.

---

## Slide 9: Deployment Slots — The Stage Analogy

### On-Slide Content

# Deployment Slots

## Analogy: Rehearsal Stage vs Live Stage

```
┌──────────────────────────────────────────────┐
│                   Theatre                     │
│                                              │
│  ┌──────────────┐      ┌──────────────┐      │
│  │  LIVE STAGE   │      │  REHEARSAL   │      │
│  │  (Production) │ ←──→ │  (Staging)   │      │
│  │              │  SWAP │              │      │
│  │  v1.0        │      │  v2.0        │      │
│  │  Audience    │      │  No audience │      │
│  └──────────────┘      └──────────────┘      │
│                                              │
└──────────────────────────────────────────────┘
```

- Deploy v2.0 to staging → test → **swap** → zero downtime
- Swap back = instant rollback (v1.0 is still on old stage)

### Speaker Notes

Deployment slots are one of the most important App Service features for the exam. Let me explain them with a theatre analogy.

Imagine a theatre with two stages. The live stage is where the audience is watching — that is your production slot. The rehearsal stage is behind the curtain — that is your staging slot.

You deploy version 2.0 of your app to the rehearsal stage (staging slot). You test it thoroughly. The audience sees nothing — they are still watching version 1.0 on the live stage.

When you are confident v2.0 works, you perform a swap. The stages trade places. The rehearsal stage becomes the live stage, and vice versa. The audience transition is instant — zero downtime.

Here is the beautiful part: if something goes wrong with v2.0 in production, you can swap back immediately. Version 1.0 is still sitting on the rehearsal stage, ready to go. Instant rollback.

Every App Service has a production slot by default. You create additional slots like staging and dev. Standard tier gives you up to 5 slots, Premium gives up to 20.

---

## Slide 10: The Swap Process

### On-Slide Content

# How a Swap Works

```
Step 1: You initiate swap (staging → production)

Step 2: Azure applies production's STICKY settings to staging
        (staging now has production's connection strings)

Step 3: Azure WARMS UP staging
        (sends HTTP requests, waits for responses)

Step 4: Azure SWAPS the routing
        (staging URL → production traffic, production URL → staging traffic)

Step 5: Swap complete
        Zero downtime. Users see new code.
```

**Warm-up ensures no cold start for users.**

### Speaker Notes

Let me walk you through what actually happens during a swap, because this helps you understand sticky settings.

Step 1: You initiate the swap, either from the portal or CLI.

Step 2: Azure takes the production slot's sticky settings — like connection strings marked as slot settings — and applies them to the staging slot. Why? Because staging is about to BECOME production, so it needs production's connection strings.

Step 3: Azure warms up the staging slot. It sends HTTP requests to the staging app and waits until the app responds successfully. This ensures the new version is fully loaded in memory. Users will not experience a cold start.

Step 4: Azure swaps the routing rules. Traffic that was going to the production slot now goes to what was the staging slot, and vice versa. This is a DNS-level swap — it is instant.

Step 5: Done. Users see the new version. If you need to rollback, just swap again.

---

## Slide 11: Sticky vs Non-Sticky Settings

### On-Slide Content

# Sticky vs Non-Sticky Settings

| Setting Type | During Swap | Example |
|---|---|---|
| **Sticky (Slot Setting)** | Stays with the slot | Production DB connection string |
| **Non-Sticky** | Swaps with the code | API keys, feature flags |

## Before Swap:
| | Production Slot | Staging Slot |
|---|---|---|
| Code | v1.0 | v2.0 |
| DB Connection (sticky) | ProdDB | TestDB |

## After Swap:
| | Production Slot | Staging Slot |
|---|---|---|
| Code | **v2.0** ← moved | **v1.0** ← moved |
| DB Connection (sticky) | **ProdDB** ← stayed | **TestDB** ← stayed |

### Speaker Notes

This is the most tested App Service concept on the AZ-204 exam. You MUST understand sticky versus non-sticky settings.

A sticky setting — also called a slot setting — is a setting that STAYS WITH THE SLOT. It does not move during a swap.

Look at the before-and-after tables on this slide. Before the swap, production has v1.0 code and a ProdDB connection string. Staging has v2.0 code and a TestDB connection string. The connection strings are marked as slot settings — sticky.

After the swap, the CODE moves. v2.0 is now in production, v1.0 is now in staging. But the CONNECTION STRINGS stay put. Production still points to ProdDB. Staging still points to TestDB.

This is by design. You want production to always connect to the production database, regardless of which version of code is running. You do not want a swap to accidentally point production at the test database.

If a connection string is NOT marked as a slot setting, it WOULD swap with the code. That is usually not what you want for database connections.

---

## Slide 12: 🚨 EXAM ALERT — Sticky Settings

### On-Slide Content

# 🚨 EXAM ALERT

## "After swapping staging to production, which database does production connect to?"

### If the connection string is a SLOT SETTING (sticky):
**Answer: Production database** — the connection string stays with the production slot.

### If the connection string is NOT a slot setting:
**Answer: Test database** — the connection string swaps with the code.

## The Rule:
**Sticky settings stay with the SLOT, not the code.**

Mark a setting as sticky: ✅ Check "Deployment slot setting" in the portal

### Speaker Notes

This is the exam question. I can almost guarantee you will see some version of this on the AZ-204 exam.

The question will describe a scenario where someone swaps staging to production. It will ask: which database does production connect to after the swap?

The answer depends on one thing: is the connection string marked as a slot setting?

If yes — it is sticky — the connection string stays with the production slot. Production connects to the production database. The code moved, but the connection string did not.

If no — it is not sticky — the connection string swaps with the code. Production now connects to whatever database staging was using, which is typically the test database. This is usually a mistake in a real scenario.

The rule is simple: sticky settings stay with the SLOT, not the code. Memorise this sentence.

---

## Slide 13: Scale Up vs Scale Out

### On-Slide Content

# Scaling: Two Different Concepts

```
SCALE UP (Vertical)                 SCALE OUT (Horizontal)
═══════════════════                 ════════════════════════

┌───────────┐                      ┌─────┐ ┌─────┐ ┌─────┐
│           │                      │ App │ │ App │ │ App │
│  BIGGER   │                      │  ①  │ │  ②  │ │  ③  │
│  MACHINE  │                      │     │ │     │ │     │
│           │                      └─────┘ └─────┘ └─────┘
│  S1 → S3  │                          ↑       ↑       ↑
│           │                      ┌───────────────────────┐
└───────────┘                      │    Load Balancer      │
                                   └───────────────────────┘
Manual                              Automatic (Autoscale)
Brief restart                       No restart
Change tier                         Add identical instances
```

### Speaker Notes

There are two types of scaling in App Service, and the exam tests whether you know the difference.

Scale up — also called vertical scaling — means making the machine bigger. You move from an S1 plan to an S3 plan. Same number of machines, but each machine has more CPU and RAM. This is a manual process and requires a brief restart. Think of it as replacing your car with a bigger car.

Scale out — also called horizontal scaling — means adding more machines. You stay on the same plan tier but run multiple copies of your app. Azure's load balancer distributes traffic across all copies. This is autoscaling — it happens automatically based on rules. No restart needed. Think of it as hiring more cashiers during the lunch rush.

On the exam, "autoscale" always means scale OUT, not scale up. Scale up is manual. Scale out is automatic.

---

## Slide 14: Autoscale Rule Components

### On-Slide Content

# Autoscale Rule — 5 Components

| Component | What It Defines | Example |
|---|---|---|
| **Metric** | What to measure | CPU Percentage |
| **Threshold** | Trigger value | > 70% |
| **Duration** | How long to wait | 5 minutes |
| **Action** | What to do | Add 1 instance |
| **Cooldown** | Wait before re-evaluating | 5 minutes |

## Complete Rule Example:
> "When **CPU** is **above 70%** for **5 minutes**, **add 1 instance**. Wait **5 minutes** before checking again."

## Two Types:
- **Metric-based:** React to CPU, memory, HTTP queue
- **Schedule-based:** Set instances for known patterns (e.g., weekends)

### Speaker Notes

An autoscale rule has five components. Let me walk through each one.

The metric is what you measure — CPU percentage is the most common, but you can also use HTTP queue length, memory percentage, or custom metrics.

The threshold is the trigger value — when the metric exceeds this value, the rule is triggered. For example, CPU greater than 70%.

The duration is how long the metric must exceed the threshold before the action fires. This prevents reacting to momentary spikes. If CPU jumps to 80% for 30 seconds and then drops, a 5-minute duration rule would NOT trigger.

The action is what happens when the rule fires — typically "increase instance count by 1" or "increase to 5 instances."

The cooldown is the waiting period after an action. After adding an instance, Azure waits the cooldown period before evaluating the rule again. This prevents rapid scaling oscillation.

You should always define matching scale-in rules as well. For example, when CPU drops below 30% for 5 minutes, remove 1 instance. Otherwise, you will keep paying for instances you no longer need.

---

## Slide 15: Configuration Key Settings

### On-Slide Content

# Key App Service Configuration Settings

| Setting | What It Does | Default | Exam Relevance |
|---|---|---|---|
| **Always On** | Keeps app in memory | OFF (Free/Shared) | Required for WebJobs |
| **ARR Affinity** | Sticky sessions (same instance) | ON | Disable for stateless apps |
| **HTTPS Only** | Redirects HTTP → HTTPS | OFF | Enable for security |
| **Min TLS Version** | Minimum encryption version | 1.2 | Set to 1.2 for security |

## App Settings & Connection Strings
- Key-value pairs → injected as **environment variables**
- Override values in config files (appsettings.json)
- **Slot settings** checkbox = sticky (stays with slot)

### Speaker Notes

Let me cover the four key configuration settings you need to know.

Always On keeps your app loaded in memory even when nobody is visiting. Without it, App Service unloads idle apps to save resources, and the next visitor gets a slow cold start. Always On is required for WebJobs — continuous background jobs. It is available on Basic tier and above but not on Free or Shared.

ARR Affinity — Application Request Routing Affinity — uses cookies to send all requests from the same user to the same instance. This is called sticky sessions. Modern stateless apps should disable ARR Affinity so traffic distributes evenly. If your app stores session data in Redis or a database, you do not need sticky sessions.

HTTPS Only redirects all HTTP requests to HTTPS. Always enable this in production.

Minimum TLS version should be set to 1.2. TLS 1.0 and 1.1 have known vulnerabilities.

For App Settings and Connection Strings: these are key-value pairs that App Service injects as environment variables into your running application. They override values in your application's configuration files. If you mark a setting as a "slot setting," it becomes sticky during swaps.

---

## Slide 16: Easy Auth — Built-In Authentication

### On-Slide Content

# Easy Auth — Authentication Without Code

```
User Request
     │
     ▼
┌──────────────────────┐
│   App Service        │
│   ┌──────────────┐   │
│   │  Easy Auth   │   │  ← Intercepts EVERY request
│   │  (platform)  │   │     Validates token
│   └──────┬───────┘   │     If valid → pass through
│          │           │     If invalid → 401 / redirect
│          ▼           │
│   ┌──────────────┐   │
│   │  Your App    │   │  ← Your code runs here
│   │  (code)      │   │     No auth code needed
│   └──────────────┘   │
└──────────────────────┘
```

**Providers:** Entra ID · Facebook · Google · Twitter · GitHub · OpenID Connect

**Token store:** Automatic, encrypted cache of auth tokens

### Speaker Notes

Easy Auth is App Service's built-in authentication feature. It is like a security guard at the hotel entrance.

Here is how it works: Every HTTP request to your app first hits Easy Auth, which runs at the App Service platform level — outside your application code. Easy Auth checks whether the request includes a valid authentication token.

If the token is valid, the request passes through to your application code. Easy Auth injects the user's identity information into HTTP headers that your code can read.

If the token is missing or invalid, Easy Auth either returns a 401 Unauthorized response or redirects the user to a login page.

The key point is that your application code does NOT need any authentication logic. You do not need to validate tokens, check signatures, or manage sessions. App Service handles all of that.

Easy Auth supports six identity providers: Microsoft Entra ID for corporate/enterprise apps, Facebook, Google, Twitter, and GitHub for consumer apps, and any OpenID Connect provider for custom identity solutions.

When a user authenticates, Easy Auth stores the token in a built-in, encrypted token store. Your app can use these tokens to make API calls on behalf of the user.

---

## Slide 17: 🚨 EXAM ALERT — Easy Auth

### On-Slide Content

# 🚨 EXAM ALERT

## Easy Auth Is Configured at the APP SERVICE Level

**NOT in your application code.**

| Question Pattern | Answer |
|---|---|
| "Where do you configure built-in authentication?" | **App Service → Authentication** (portal) |
| "Does Easy Auth require code changes?" | **No** — it is a platform feature |
| "When does Easy Auth validate tokens?" | **Before** your code runs |

## Common Trap:
The exam may list MSAL code as an option. MSAL is for custom authentication — **NOT** Easy Auth.

Easy Auth = Azure configures it.
MSAL = You write code for it.

### Speaker Notes

Here is the exam alert for Easy Auth. The exam will ask where authentication is configured, and the trap is that they include options like "add authentication middleware to the code" or "use MSAL in the application."

If the question is about Easy Auth — App Service's BUILT-IN authentication — the answer is always at the App Service level. You configure it in the portal under App Service → Authentication. No code changes.

MSAL — the Microsoft Authentication Library — is a completely different approach. With MSAL, you write code to handle authentication flows yourself. MSAL gives you more control, but Easy Auth gives you zero-code simplicity.

If the exam says "built-in authentication" or "Easy Auth," the answer is App Service configuration, not code. If the exam says "MSAL" or "custom authentication," then you write code.

---

## Slide 18: Networking — VNet Integration

### On-Slide Content

# Networking: VNet Integration

```
┌─────────────────┐                    ┌─────────────────┐
│   App Service   │    VNet            │   Virtual        │
│   (public)      │    Integration     │   Network        │
│                 │ ──────────────→    │                  │
│  https://myapp  │   OUTBOUND ONLY   │  ┌────────────┐  │
│  .azurewebsites │                    │  │ Database   │  │
│  .net           │                    │  │(private IP)│  │
│                 │                    │  └────────────┘  │
└─────────────────┘                    └─────────────────┘
```

| Feature | Direction | Purpose | Min Tier |
|---|---|---|---|
| **VNet Integration** | Outbound → | App reaches VNet resources | Standard |
| **Hybrid Connections** | Outbound → | App reaches on-premises | Standard |
| **Access Restrictions** | ← Inbound | IP firewall for the app | All tiers |
| **ASE (Isolated)** | ← Both → | Full VNet isolation | Isolated |

### Speaker Notes

Networking is a critical exam topic. There are three main networking features for App Service, and you need to know the direction of each one.

VNet Integration allows your App Service to make outbound connections to resources inside a Virtual Network. The arrow goes FROM your app INTO the VNet. If you have a database with a private IP inside a VNet, VNet Integration lets your app reach it.

Hybrid Connections let your App Service reach on-premises resources — servers in your company's data centre that are not in Azure. It uses an agent called the Hybrid Connection Manager installed on a server in your on-premises network.

Access Restrictions are an inbound firewall. They control who can access your App Service from the internet. You create rules that allow or deny traffic based on IP addresses or VNet subnets.

The direction is the key differentiator. VNet Integration and Hybrid Connections are outbound — your app reaching out. Access Restrictions are inbound — controlling who reaches your app. App Service Environment on the Isolated tier gives you full isolation in both directions.

---

## Slide 19: 🚨 EXAM ALERT — VNet Integration Is Outbound Only

### On-Slide Content

# 🚨 EXAM ALERT

## VNet Integration Is OUTBOUND ONLY

| Statement | True or False? |
|---|---|
| "VNet Integration lets my app reach a private database in a VNet" | ✅ TRUE |
| "VNet Integration restricts who can access my app" | ❌ FALSE |
| "VNet Integration places my app inside the VNet" | ❌ FALSE |
| "VNet Integration replaces my app's public URL" | ❌ FALSE |

## If you need to restrict INBOUND traffic:
Use **Access Restrictions** (IP-based firewall rules)

## If you need full VNet isolation (inbound + outbound):
Use **App Service Environment** (Isolated tier)

### Speaker Notes

This is another high-probability exam question. VNet Integration is OUTBOUND ONLY.

Let me go through the true/false statements on this slide.

VNet Integration lets your app reach a private database in a VNet — TRUE. This is exactly what VNet Integration does. Outbound traffic from your app can route through the VNet.

VNet Integration restricts who can access your app — FALSE. VNet Integration has absolutely no effect on inbound traffic. Your app's public URL is still accessible from the internet. To restrict inbound access, use Access Restrictions.

VNet Integration places your app inside the VNet — FALSE. Your app still runs on the public App Service infrastructure. It just has a connection INTO the VNet for outbound traffic. To truly place your app inside a VNet, you need an App Service Environment on the Isolated tier.

VNet Integration replaces your app's public URL — FALSE. Your app still has `https://myapp.azurewebsites.net` as its public URL. VNet Integration does not change the app's URL or address.

---

## Slide 20: Top 3 Exam Traps — Summary

### On-Slide Content

# Top 3 App Service Exam Traps

## Trap 1: Basic Tier Has Everything
❌ **Wrong.** Basic does NOT have slots, autoscale, or VNet Integration.
✅ **Minimum for slots/autoscale/VNet = Standard.**

## Trap 2: Sticky Settings Move During Swap
❌ **Wrong.** Sticky settings STAY with the slot.
✅ **Code swaps. Sticky connection strings stay.**

## Trap 3: VNet Integration Restricts Inbound Traffic
❌ **Wrong.** VNet Integration is outbound only.
✅ **Use Access Restrictions for inbound control.**

### Speaker Notes

Let me summarise the three most dangerous exam traps for App Service.

Trap number 1: Students assume that because Basic is a paid tier with dedicated VMs, it must support all features. It does not. Slots, autoscale, and VNet Integration all require Standard or higher. If a question asks about any of these features, the answer is never Basic.

Trap number 2: Students think sticky settings travel with the code during a swap. They do not. Sticky settings — connection strings and app settings marked as slot settings — stay with the slot. The code moves, but the settings stay. Production keeps its own database connection.

Trap number 3: Students think VNet Integration protects their app from the internet. It does not. VNet Integration is outbound only — it lets your app reach into a VNet. To control who can access your app, use Access Restrictions, which is an inbound IP firewall.

---

## Slide 21: Module Summary Table

### On-Slide Content

# Module 1 Summary — All Key Facts

| Topic | Key Fact | Exam Relevance |
|---|---|---|
| App Service | Fully managed PaaS for web apps | 🔴 HIGH |
| Plan tiers | Standard+ for slots/autoscale/VNet | 🔴 HIGH |
| Deployment | ZIP deploy CLI: `az webapp deployment source config-zip` | 🟡 MEDIUM |
| Slots | Zero-downtime swap; warm-up before swap | 🔴 HIGH |
| Sticky settings | Stay with SLOT, not code | 🔴 HIGH |
| Scale up | Bigger machine (manual, vertical) | 🟡 MEDIUM |
| Scale out | More instances (autoscale, horizontal) | 🔴 HIGH |
| Autoscale | Metric + Threshold + Duration + Action + Cooldown | 🔴 HIGH |
| Always On | Required for WebJobs | 🟡 MEDIUM |
| ARR Affinity | Disable for stateless apps | 🟡 MEDIUM |
| TLS | Minimum 1.2 | 🟡 MEDIUM |
| Easy Auth | Platform-level auth, no code needed | 🔴 HIGH |
| VNet Integration | OUTBOUND ONLY | 🔴 HIGH |
| Hybrid Connections | On-premises TCP tunnel | 🟡 MEDIUM |
| Access Restrictions | Inbound IP firewall | 🟡 MEDIUM |

### Speaker Notes

Here is your complete summary table for Module 1. I recommend printing this or saving it as a reference card.

Everything marked HIGH in the exam relevance column has appeared in multiple practice exams and is very likely to be on your real exam. Focus your review on: plan tiers and the Standard threshold, deployment slots with sticky settings, autoscale rules, Easy Auth configuration, and VNet Integration direction.

The MEDIUM items are still testable but appear less frequently. You should still know them, but the HIGH items are your priority for exam preparation.

---

## Slide 22: Checkpoint — Test Your Knowledge

### On-Slide Content

# ✅ Checkpoint: Test Yourself

**Answer these before proceeding to the lab:**

1. What is the minimum App Service Plan tier for deployment slots?

2. After a swap, where does a sticky connection string end up?

3. What is the direction of VNet Integration — inbound, outbound, or both?

4. What are the 5 components of an autoscale rule?

5. Where is Easy Auth configured — in your code or at the App Service level?

*(Answers on next slide — try to answer first!)*

### Speaker Notes

Before you move on to the lab, test yourself with these five questions. Try to answer them from memory before looking at the answers.

Question 1: What tier do you need for slots? If you said Standard, you are correct.

Question 2: After a swap, where does a sticky connection string end up? It stays with the slot. Production's connection string stays in the production slot.

Question 3: VNet Integration is outbound only. It lets your app reach into a VNet but does not restrict inbound traffic.

Question 4: The five components are Metric, Threshold, Duration, Action, and Cooldown.

Question 5: Easy Auth is configured at the App Service level — in the Azure portal, not in your application code.

If you got all five right, you are well prepared for the lab. If you missed any, go back to that section and review before continuing.

---

## Slide 22 (continued): Answers

### On-Slide Content

# Answers

| # | Question | Answer |
|---|---|---|
| 1 | Minimum tier for slots | **Standard (S1+)** |
| 2 | Sticky connection string after swap | **Stays with the slot** — does not move |
| 3 | VNet Integration direction | **Outbound only** |
| 4 | 5 autoscale components | **Metric, Threshold, Duration, Action, Cooldown** |
| 5 | Easy Auth configured where | **App Service level** — no code changes |

**Next step:** Complete the M01 Lab to build hands-on experience.

### Speaker Notes

Here are the answers. Let me quickly reinforce each one.

Standard S1 is the minimum for deployment slots. Not Basic, not Free — Standard.

Sticky connection strings stay with the slot during a swap. The code swaps, but sticky settings stay put.

VNet Integration is outbound only. Use Access Restrictions for inbound control.

Autoscale rules have five parts: what to measure (metric), when to trigger (threshold), how long to wait (duration), what to do (action), and how long to wait after acting (cooldown).

Easy Auth is a platform feature configured at the App Service level. No code changes needed.

You are now ready for the Module 1 Lab, where you will create an App Service, deploy code, configure settings, set up slots, perform a swap, and configure autoscale. Everything you learned in this slide deck, you will now do hands-on.
