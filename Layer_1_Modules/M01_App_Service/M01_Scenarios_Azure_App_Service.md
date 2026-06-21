# Module 1 Scenario Cards: Azure App Service Web Apps

---

| | |
|---|---|
| **Module** | M01 — Azure App Service Web Apps |
| **Total Scenarios** | 5 |
| **Difficulty Distribution** | 1 Beginner + 2 Intermediate + 2 Advanced |
| **Format** | Business Context → Problem → Constraints → Task → Hints → Solution → Exam Connection |
| **Purpose** | Apply App Service concepts to realistic business scenarios that mirror AZ-204 exam question patterns |

---

## How to Use These Scenario Cards

1. Read the **Business Context** and **Problem** carefully
2. Note the **Constraints** — your solution must satisfy ALL of them
3. Write your answer before looking at the **Solution**
4. Read the **Exam Connection** to understand how this scenario appears on the actual exam

---

## Scenario 1: Custom Domain on a Budget (Beginner)

### Business Context

**FreshBites** is a small restaurant startup with 3 employees. They have built a simple website that displays their menu, hours, and contact information. The site is a static HTML page with some JavaScript — it receives about 200 visitors per day.

The founder wants the website at `www.freshbites.com` instead of the default Azure URL. They are bootstrapping (self-funding) and need to keep costs as low as possible. They have no need for deployment slots, autoscale, or any advanced features.

### The Problem

The founder initially deployed the site on the Free (F1) App Service Plan because it costs nothing. However, when they tried to configure their custom domain `www.freshbites.com`, they received an error message stating that custom domains are not supported on their current plan.

They need to find the cheapest App Service Plan tier that supports custom domains.

### Constraints

1. Must support custom domain mapping (`www.freshbites.com`)
2. Must be the **lowest cost** option available
3. Does NOT need deployment slots, autoscale, or VNet Integration
4. SSL certificate binding is NOT required (they will add this later)

### Your Task

Select the appropriate App Service Plan tier for FreshBites. Explain what features are included and what limitations the chosen tier has.

### Hints

1. Look at the tier table — which is the first tier that supports custom domains?
2. The question says SSL is NOT required yet. Does that change your answer?

### Solution

**Recommended Tier: Shared (D1)**

The Shared (D1) tier is the cheapest option that supports custom domain mapping. Here is why:

| Tier | Custom Domain | SSL Binding | Cost | Verdict |
|---|---|---|---|---|
| Free (F1) | ❌ No | ❌ No | $0/month | Cannot map custom domain |
| **Shared (D1)** | ✅ Yes | ❌ No | ~$9.49/month | ✅ Cheapest with custom domain |
| Basic (B1) | ✅ Yes | ✅ Yes | ~$54.75/month | Overkill — SSL not needed yet |

**Why Shared (D1) is correct:**
- It supports custom domain mapping, which is the primary requirement
- It is the cheapest tier that supports custom domains
- The site gets only 200 visitors/day, well within Shared tier capacity

**Limitations of Shared (D1):**
- No SSL certificate binding (the founder will need to upgrade to Basic when they want HTTPS with their custom domain)
- No deployment slots
- No autoscale
- No Always On (the app may unload during idle periods)
- Shared compute — the app shares CPU with other tenants

**When to upgrade:** When FreshBites needs SSL (HTTPS with their custom domain), they will need to move to Basic (B1) minimum. For deployment slots or autoscale, they would need Standard (S1).

### Exam Connection

🚨 **How this appears on the exam:**

> "A company needs to host a web app with a custom domain at the lowest possible cost. They do not need SSL, deployment slots, or autoscale. Which App Service Plan tier should they choose?"
> 
> - A. Free (F1)
> - B. Shared (D1) ✅
> - C. Basic (B1)
> - D. Standard (S1)

The exam tests whether you know that Free does NOT support custom domains and that Shared is cheaper than Basic. The trap is choosing Free (because it is cheapest) or Basic (because it sounds like the minimum paid tier).

---

## Scenario 2: Zero-Downtime Weekly Deployments (Intermediate)

### Business Context

**ShopStream** is a mid-sized e-commerce company with 50,000 daily active users. They deploy updates to their web application every Wednesday at 2 PM. Currently, each deployment causes 3–5 minutes of downtime. During this window:

- Users see error pages
- Shopping carts are lost
- The company estimates $2,000 in lost revenue per deployment

The engineering team wants to eliminate downtime during deployments and have the ability to instantly roll back if a new release introduces bugs.

### The Problem

ShopStream currently deploys directly to their production App Service using ZIP deploy. When the deployment runs, the app restarts, and users experience errors until the new version fully loads.

They need a deployment strategy that:
- Eliminates user-facing downtime during deployments
- Allows testing the new version before users see it
- Provides instant rollback if something goes wrong

### Constraints

1. **Zero downtime** during deployments — users must not see errors
2. **Pre-production testing** — the team must be able to test the new version on a live Azure environment before it goes to production
3. **Instant rollback** — if the new version has a bug, they must be able to revert within seconds
4. The production app connects to a production database; the staging environment must connect to a separate test database
5. The solution must work within the existing Standard S1 plan

### Your Task

Design the deployment strategy using App Service features. Specify:
1. What to create (slots, settings)
2. How to configure connection strings
3. The step-by-step deployment workflow
4. How rollback works

### Hints

1. Which App Service feature enables zero-downtime deployments?
2. How do you ensure production always connects to the production database — even after a swap?

### Solution

**Strategy: Deployment Slots with Sticky Connection Strings**

**Step 1: Create a staging slot**

```bash
# Create a staging deployment slot
az webapp deployment slot create \
  --resource-group shopstream-rg \     # ← Resource group
  --name shopstream-app \              # ← App name
  --slot staging                       # ← Slot name
```

This creates a staging environment at `https://shopstream-app-staging.azurewebsites.net`.

**Step 2: Configure connection strings as slot settings (sticky)**

For the **production slot:**
```bash
az webapp config connection-string set \
  --resource-group shopstream-rg \
  --name shopstream-app \
  --settings MainDB="Server=prod-server;Database=shopstream-prod" \
  --connection-string-type SQLServer \
  --slot-setting                        # ← Makes it STICKY
```

For the **staging slot:**
```bash
az webapp config connection-string set \
  --resource-group shopstream-rg \
  --name shopstream-app \
  --slot staging \
  --settings MainDB="Server=test-server;Database=shopstream-test" \
  --connection-string-type SQLServer \
  --slot-setting                        # ← Makes it STICKY
```

**Why slot settings?** By marking connection strings as slot settings (sticky), production always connects to the production database and staging always connects to the test database — even after swaps. The code moves, but the connection strings stay.

**Step 3: Weekly deployment workflow**

1. **Deploy** new version to staging (not production):
   ```bash
   az webapp deployment source config-zip \
     --resource-group shopstream-rg \
     --name shopstream-app \
     --slot staging \                     # ← Deploy to STAGING, not production
     --src release-v2.5.zip
   ```

2. **Test** the staging slot: Browse `https://shopstream-app-staging.azurewebsites.net`, run automated tests, verify functionality against the test database.

3. **Swap** staging to production:
   ```bash
   az webapp deployment slot swap \
     --resource-group shopstream-rg \
     --name shopstream-app \
     --slot staging \
     --target-slot production
   ```
   Azure warms up the staging slot, then swaps routing. Users see the new version. Zero downtime.

4. **Verify** production is working correctly.

**Step 4: Rollback (if needed)**

If the new version has a bug, swap back immediately:
```bash
az webapp deployment slot swap \
  --resource-group shopstream-rg \
  --name shopstream-app \
  --slot staging \
  --target-slot production
```

The old version is still in the staging slot (it was swapped there in step 3). Swapping again puts it back in production. This takes seconds — not minutes.

### Exam Connection

🚨 **How this appears on the exam:**

> "An e-commerce company deploys weekly and needs zero-downtime deployments with instant rollback. Connection strings must remain with their respective environments. What should you configure?"
>
> The answer involves: (1) deployment slots, (2) connection strings marked as slot settings, and (3) the swap operation.
>
> The exam may ask: "After swapping, which database does production connect to?" — Answer: the production database, because the connection string is a slot setting (sticky).

---

## Scenario 3: Weekend Traffic Spikes (Intermediate)

### Business Context

**GameHub** is an online gaming community platform. Their web application runs on Azure App Service Standard S2 with 2 instances. Traffic follows a predictable pattern:

| Day | Time | Users | CPU % |
|---|---|---|---|
| Monday–Thursday | All day | 5,000 | 25% |
| Friday | 6 PM – midnight | 15,000 | 60% |
| Saturday–Sunday | All day | 30,000 | 90% |
| Monday | 6 AM | Back to 5,000 | 25% |

On weekends, the app becomes sluggish and occasionally returns timeout errors (HTTP 504). The team tried manually adding instances on Friday evenings, but sometimes they forget, and the app suffers.

### The Problem

GameHub needs automatic scaling that handles the predictable weekend spike without manual intervention. They also need protection against unexpected traffic surges during the week (e.g., a viral post driving 20,000 users on a Tuesday).

### Constraints

1. The solution must **automatically** handle weekend traffic — no manual intervention
2. Weekday spikes (unpredictable) must also be handled
3. Maximum budget allows up to **8 instances** at peak
4. Minimum **2 instances** must always run (for redundancy)
5. Must avoid "autoscale oscillation" (rapidly adding and removing instances)

### Your Task

Design an autoscale configuration that handles both the predictable weekend pattern and unpredictable weekday spikes. Specify:
1. Schedule-based rules for weekends
2. Metric-based rules for unexpected spikes
3. Instance limits (min/max/default)
4. Cooldown periods

### Hints

1. Azure autoscale supports both schedule-based AND metric-based rules. Can you combine them?
2. What cooldown period prevents oscillation?

### Solution

**Strategy: Combined Schedule-Based + Metric-Based Autoscale**

**Instance Limits:**

| Setting | Value | Reason |
|---|---|---|
| Minimum instances | 2 | Always maintain redundancy |
| Maximum instances | 8 | Budget constraint |
| Default instances | 2 | Used when metrics are unavailable |

**Schedule-Based Rules (for predictable weekend traffic):**

Rule 1 — Scale up for weekends:
> **When:** Friday at 5:00 PM
> **Action:** Set minimum instances to 5
> **Why:** Pre-scales before the Friday evening rush. Instances are ready before traffic hits.

Rule 2 — Scale down after weekends:
> **When:** Monday at 6:00 AM
> **Action:** Set minimum instances back to 2
> **Why:** Weekend traffic drops by Monday morning. Reduces cost.

**Metric-Based Rules (for unpredictable spikes):**

Rule 3 — Scale out on high CPU:
> **When:** Average CPU > 70% for 5 minutes across all instances
> **Action:** Increase instance count by 1
> **Cooldown:** 5 minutes
> **Why:** Handles unexpected spikes (e.g., viral post on Tuesday). The 5-minute duration prevents reacting to brief momentary spikes. The 5-minute cooldown prevents adding too many instances too quickly.

Rule 4 — Scale in on low CPU:
> **When:** Average CPU < 30% for 10 minutes across all instances
> **Action:** Decrease instance count by 1
> **Cooldown:** 10 minutes
> **Why:** Removes unnecessary instances when traffic drops. The longer duration (10 minutes) and cooldown (10 minutes) for scale-in prevents oscillation — we are more cautious about removing instances than adding them.

**How the rules work together:**

- **Normal Tuesday:** 2 instances running. CPU at 25%. No scaling needed.
- **Viral Tuesday:** CPU spikes to 80%. After 5 minutes, Rule 3 adds 1 instance (now 3). If still high, adds another after 5 minutes (now 4). Cap at 8.
- **Friday 5 PM:** Rule 1 fires. Minimum set to 5. If currently at 2, Azure immediately adds 3 instances to meet the minimum.
- **Saturday:** 5+ instances running. If CPU goes above 70%, Rule 3 adds more (up to max 8).
- **Monday 6 AM:** Rule 2 fires. Minimum set to 2. Rule 4 gradually removes instances as CPU drops below 30%.

### Exam Connection

🚨 **How this appears on the exam:**

> "A gaming platform experiences 3x traffic on weekends. Traffic is unpredictable during weekdays. Which autoscale configuration correctly handles both patterns?"
>
> The correct answer combines schedule-based rules (for predictable weekend patterns) with metric-based rules (for unpredictable spikes). The exam may test whether you know:
> - Autoscale = scale OUT (horizontal), not scale UP (vertical)
> - Autoscale adds IDENTICAL instances
> - Both scale-out AND scale-in rules should be defined
> - Cooldown periods prevent oscillation

---

## Scenario 4: Connecting to an On-Premises Database (Advanced)

### Business Context

**HealthFirst** is a healthcare company that runs a patient portal on Azure App Service (Standard S2). They are migrating to the cloud in phases. Phase 1 deployed the web application to Azure. However, the patient records database is still in their on-premises data centre — it cannot be moved to Azure yet due to regulatory compliance requirements.

The on-premises database server:
- IP address: `192.168.1.100`
- Port: `1433` (SQL Server)
- Is behind a corporate firewall with no public IP
- Cannot be exposed to the internet (compliance requirement)

### The Problem

The web application in Azure needs to query patient records from the on-premises SQL Server database. But the database has no public IP address and is behind a corporate firewall. A direct connection from Azure App Service to `192.168.1.100:1433` will fail — the internet cannot reach private IP addresses behind a firewall.

The networking team asks: "How do we securely connect our cloud-hosted app to our on-premises database without exposing the database to the internet?"

### Constraints

1. The database **cannot have a public IP** address (compliance)
2. The database **cannot be moved to Azure** yet (regulatory timeline)
3. The connection must be **secure** — data must not travel over the public internet unencrypted
4. The solution must work with the existing **Standard S2** plan
5. The app needs TCP connectivity on port 1433 to the database server

### Your Task

Design the networking architecture. Specify:
1. Which Azure networking feature to use
2. What needs to be installed on-premises
3. How the connection flows from App Service to the database
4. Any prerequisites or limitations

### Hints

1. VNet Integration connects to Azure VNet resources. Does the on-premises database live in an Azure VNet?
2. There is a specific App Service feature designed for connecting to on-premises resources.

### Solution

**Strategy: Hybrid Connections**

**Why Hybrid Connections?**

| Feature | Suitable? | Reason |
|---|---|---|
| VNet Integration | ❌ No | Connects to Azure VNet resources. The database is on-premises, not in a VNet. |
| VPN Gateway | ⚠️ Possible but complex | Requires VPN infrastructure. Overkill for a single host:port connection. |
| **Hybrid Connections** | ✅ Yes | Designed for connecting App Service to on-premises resources via TCP. No public IP needed. |
| ExpressRoute | ⚠️ Possible but expensive | Dedicated private connection. Far too expensive for a single database connection. |

**Architecture:**

```
┌─────────────────┐                              ┌──────────────────────┐
│   Azure          │                              │   On-Premises        │
│                  │                              │   Data Centre        │
│   ┌───────────┐  │      Secure Relay            │                      │
│   │ App       │  │      (Azure Service Bus)     │  ┌────────────────┐  │
│   │ Service   │──┼──────────────────────────────┼──│ Hybrid         │  │
│   │ (web app) │  │      Outbound HTTPS (443)    │  │ Connection     │  │
│   └───────────┘  │      from HCM to Azure       │  │ Manager (HCM)  │  │
│                  │                              │  └───────┬────────┘  │
│                  │                              │          │           │
│                  │                              │  ┌───────▼────────┐  │
│                  │                              │  │ SQL Server     │  │
│                  │                              │  │ 192.168.1.100  │  │
│                  │                              │  │ Port 1433      │  │
│                  │                              │  └────────────────┘  │
└─────────────────┘                              └──────────────────────┘
```

**Step 1: Create Hybrid Connection in Azure Portal**

Navigate to App Service → Networking → Hybrid Connections → Add Hybrid Connection.

Configure:
- **Endpoint host:** `192.168.1.100` (the on-premises server IP)
- **Endpoint port:** `1433` (SQL Server port)
- **Service Bus namespace:** Create new or use existing

**Step 2: Install Hybrid Connection Manager (HCM) on-premises**

Install the HCM agent on a Windows server in the on-premises network that can reach the database server on port 1433. The HCM:
- Makes an OUTBOUND HTTPS connection (port 443) to Azure Service Bus
- No inbound firewall rules needed — the connection goes outward
- Creates a secure relay through Azure Service Bus

**Step 3: Configure the connection string in App Service**

```bash
az webapp config connection-string set \
  --resource-group healthfirst-rg \
  --name healthfirst-portal \
  --settings PatientDB="Server=192.168.1.100;Database=PatientRecords;User=app_user;Password=..." \
  --connection-string-type SQLServer
```

The App Service resolves `192.168.1.100` through the Hybrid Connection relay instead of trying to reach it over the public internet.

**How data flows:**
1. App Service sends a TCP request to `192.168.1.100:1433`
2. The request is routed through the Hybrid Connection to Azure Service Bus
3. The HCM agent on-premises picks up the request from Service Bus
4. The HCM agent forwards the request to the local SQL Server at `192.168.1.100:1433`
5. The response travels back through the same relay

**Key facts:**
- Data is encrypted in transit (TLS over Service Bus relay)
- The database server never has a public IP
- No inbound firewall rules are needed on the corporate firewall (HCM connects outbound)
- Standard S2 supports Hybrid Connections

### Exam Connection

🚨 **How this appears on the exam:**

> "An App Service needs to connect to an on-premises SQL Server that has no public IP. Which networking feature should you use?"
>
> - VNet Integration — wrong (connects to Azure VNet, not on-premises)
> - Hybrid Connections — ✅ correct
> - Access Restrictions — wrong (controls inbound traffic, not outbound connectivity)
> - App Service Environment — wrong (provides VNet isolation, does not create on-premises connectivity)
>
> The exam tests whether you know that Hybrid Connections are specifically designed for connecting App Service to on-premises resources.

---

## Scenario 5: Multi-Region Application Architecture (Advanced)

### Business Context

**GlobalTrade** is an international trading platform with users in three regions: North America, Europe, and Asia-Pacific. Their web application currently runs on a single App Service in the East US region. Users in Europe and Asia experience response times of 400–600ms, while North American users see 50–100ms.

The company's SLA (Service Level Agreement) requires:
- Response time under 200ms for all regions
- 99.95% uptime
- Automatic failover if a region goes down

### The Problem

A single App Service in one region cannot satisfy the latency requirements for users worldwide. The speed of light imposes a fundamental limit: data travelling from East US to Asia takes ~200ms round trip just for network transit, before any processing.

Additionally, a single region creates a single point of failure. If the East US region experiences an outage, the entire application goes down for all users worldwide.

### Constraints

1. Response time under **200ms** for users in all three regions
2. **99.95% uptime** — the app must survive a full regional outage
3. All regions must serve the **same version** of the application
4. Database is Azure Cosmos DB with **multi-region writes** enabled
5. Budget allows for Premium v3 plans in each region

### Your Task

Design a multi-region App Service architecture. Specify:
1. Which App Service features to use in each region
2. How traffic is routed to the nearest region
3. How deployments work across multiple regions
4. What happens during a regional failure

### Hints

1. App Service can be deployed in multiple regions, but it does not have a built-in global load balancer. Which Azure service distributes traffic globally?
2. How can deployment slots help ensure all regions run the same version?

### Solution

**Strategy: Multi-Region App Service + Azure Front Door + Deployment Slots**

**Architecture Overview:**

```
                        ┌─────────────────┐
                        │  Azure Front     │
                        │  Door (Global    │
                        │  Load Balancer)  │
                        └───┬─────┬───┬───┘
                            │     │   │
              ┌─────────────┘     │   └─────────────┐
              ▼                   ▼                   ▼
    ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
    │  App Service      │ │  App Service      │ │  App Service      │
    │  East US          │ │  West Europe      │ │  Southeast Asia   │
    │  (P1v3)          │ │  (P1v3)          │ │  (P1v3)          │
    │                  │ │                  │ │                  │
    │  Prod + Staging  │ │  Prod + Staging  │ │  Prod + Staging  │
    └────────┬─────────┘ └────────┬─────────┘ └────────┬─────────┘
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  │
                         ┌────────▼────────┐
                         │  Azure Cosmos DB │
                         │  (Multi-region   │
                         │   writes)        │
                         └─────────────────┘
```

**Component 1: App Service in Each Region**

Create an App Service Plan (P1v3) and web app in three regions:

```bash
# East US
az appservice plan create --name globaltrade-plan-eastus \
  --resource-group globaltrade-rg --location eastus \
  --sku P1V3 --is-linux

az webapp create --name globaltrade-eastus \
  --resource-group globaltrade-rg \
  --plan globaltrade-plan-eastus --runtime "NODE:20-lts"

# West Europe
az appservice plan create --name globaltrade-plan-westeu \
  --resource-group globaltrade-rg --location westeurope \
  --sku P1V3 --is-linux

az webapp create --name globaltrade-westeu \
  --resource-group globaltrade-rg \
  --plan globaltrade-plan-westeu --runtime "NODE:20-lts"

# Southeast Asia
az appservice plan create --name globaltrade-plan-seasia \
  --resource-group globaltrade-rg --location southeastasia \
  --sku P1V3 --is-linux

az webapp create --name globaltrade-seasia \
  --resource-group globaltrade-rg \
  --plan globaltrade-plan-seasia --runtime "NODE:20-lts"
```

**Component 2: Azure Front Door for Global Traffic Routing**

Azure Front Door is a global load balancer and CDN (Content Delivery Network). It routes each user's request to the **nearest** healthy App Service instance:

- User in New York → East US App Service (~50ms)
- User in London → West Europe App Service (~30ms)
- User in Singapore → Southeast Asia App Service (~20ms)

Front Door also handles failover: if the East US App Service goes down, North American users are automatically routed to West Europe (next closest).

**Component 3: Deployment Strategy Using Slots**

To ensure all regions run the same version, use deployment slots as a staged rollout:

1. Deploy new version to **staging slot** in East US
2. Test the staging slot
3. **Swap** staging → production in East US
4. If East US production is healthy for 30 minutes:
   - Deploy same version to staging slots in West Europe and Southeast Asia
   - Swap staging → production in both regions

This approach ensures:
- Every region is tested before users see the new version
- Swaps are zero-downtime
- If the new version has issues, only one region is affected initially

**Component 4: App Service Features Used**

| Feature | Purpose | Applied How |
|---|---|---|
| **Deployment Slots** | Zero-downtime regional deployments | Each region has a staging slot |
| **Autoscale** | Handle varying regional traffic | Metric-based rules (CPU > 70%) per region |
| **Sticky Connection Strings** | Region-specific Cosmos DB endpoints | Each region's connection string is a slot setting |
| **Always On** | Keep apps warm in all regions | Enabled on all Premium plans |
| **HTTPS Only + TLS 1.2** | Security hardening | Enabled on all apps |
| **Access Restrictions** | Only allow traffic from Front Door | Restrict inbound to Front Door's IP ranges |

**How Regional Failure is Handled:**

1. Azure Front Door continuously health-checks all three App Service instances
2. If East US fails health check → Front Door removes it from routing
3. North American users are automatically routed to West Europe (next closest)
4. When East US recovers → Front Door adds it back to routing
5. Users experience minimal impact — automatic failover

### Exam Connection

🚨 **How this appears on the exam:**

> "A company needs to serve users in multiple regions with low latency and automatic failover. Which combination of services should they use?"
>
> The correct answer includes:
> - Multiple App Service instances in different regions
> - Azure Front Door (or Traffic Manager) for global routing
> - Deployment slots for zero-downtime deployments in each region
>
> The exam tests whether you know that:
> - App Service is a single-region service — you need multiple instances for multi-region
> - Azure Front Door (or Traffic Manager) provides global traffic routing and failover
> - Deployment slots work independently in each region
> - Access Restrictions should limit inbound traffic to Front Door only

📝 **NOTE:** Azure Front Door and Traffic Manager are covered in more detail in other modules. For the AZ-204 exam, know that App Service needs a global load balancer for multi-region architectures.

---

## Summary of Scenario Difficulty and Topics

| # | Scenario | Difficulty | Primary Topics |
|---|---|---|---|
| 1 | Custom domain on a budget | Beginner | Tier selection, feature limitations |
| 2 | Zero-downtime deployments | Intermediate | Deployment slots, sticky settings, swap |
| 3 | Weekend traffic spikes | Intermediate | Autoscale rules, schedule + metric rules |
| 4 | On-premises database connection | Advanced | Hybrid Connections, networking direction |
| 5 | Multi-region architecture | Advanced | Multi-region deployment, Front Door, slots |
