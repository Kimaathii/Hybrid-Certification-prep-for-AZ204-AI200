# Module 1 Quiz: Azure App Service Web Apps

---

| | |
|---|---|
| **Module** | M01 — Azure App Service Web Apps |
| **Total Questions** | 15 |
| **Question Breakdown** | 5 Foundation + 5 Applied + 5 Exam-Level |
| **Format** | Microsoft AZ-204 exam style — scenario → question → options |
| **SELECT TWO Questions** | Questions 11 and 14 |
| **Estimated Time** | 30–40 minutes |
| **Passing Suggestion** | 12/15 correct (80%) |

---

## How to Use This Quiz

1. Read each scenario carefully before looking at the options
2. Select your answer before revealing the explanation
3. Read ALL explanations — even for questions you got right
4. Pay special attention to 🚨 EXAM ALERT markers — these highlight real exam traps

---

## Part 1: Foundation Questions (1–5)

These questions test your understanding of core App Service concepts.

---

### Question 1: App Service Plan Tiers

**Scenario:** A developer creates a new Azure App Service web app for a personal blog. The blog receives very little traffic and the developer wants to minimize costs. The developer does not need custom domains, SSL certificates, or deployment slots.

**Question:** Which App Service Plan tier should the developer choose?

- A. Free (F1)
- B. Shared (D1)
- C. Basic (B1)
- D. Standard (S1)

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: A. Free (F1)**

**Why A is correct:** The Free tier provides a shared compute environment at no cost. For a personal blog with minimal traffic and no requirements for custom domains, SSL, or slots, Free is the most cost-effective choice. It includes 1 GB of storage and 60 CPU minutes per day, which is sufficient for a low-traffic blog.

**Why B is wrong:** The Shared (D1) tier supports custom domains, but the developer does not need custom domains. D1 has a monthly cost, so it is not the cheapest option when the extra feature is not needed.

**Why C is wrong:** Basic (B1) provides dedicated VMs and SSL support. This is overkill for a personal blog with minimal traffic and no need for these features. It costs significantly more than Free.

**Why D is wrong:** Standard (S1) includes autoscale, deployment slots, and VNet Integration. None of these are needed for a personal blog. This tier has the highest cost among the options and is unnecessarily expensive for this scenario.

</details>

---

### Question 2: Scale Up vs Scale Out

**Scenario:** A web application running on an App Service Plan S1 tier is experiencing slow response times. The application is CPU-bound and uses 95% of the available CPU during peak hours.

**Question:** What is the difference between scaling up and scaling out in Azure App Service?

- A. Scaling up adds more VM instances; scaling out moves to a higher plan tier
- B. Scaling up moves to a higher plan tier with more CPU/RAM; scaling out adds more VM instances
- C. Scaling up increases storage; scaling out increases network bandwidth
- D. Scaling up and scaling out are the same thing — both add more VM instances

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B. Scaling up moves to a higher plan tier with more CPU/RAM; scaling out adds more VM instances**

**Why B is correct:** Scaling up (vertical scaling) means moving to a higher tier — for example, from S1 to S3, which gives more CPU cores and RAM per machine. Scaling out (horizontal scaling) means adding more instances of the same tier — for example, running 3 copies of S1 instead of 1. Autoscale performs scale-out operations.

**Why A is wrong:** This reverses the definitions. Scaling UP is about tier changes (bigger machine). Scaling OUT is about adding instances (more machines).

**Why C is wrong:** Scaling up does increase CPU and RAM (not just storage). Scaling out adds instances (not just bandwidth). Neither description is accurate.

**Why D is wrong:** They are fundamentally different operations. Scaling up changes the machine size. Scaling out changes the number of machines.

</details>

---

### Question 3: Always On Configuration

**Scenario:** A developer has deployed a web application with a background WebJob that processes orders from a queue. The WebJob runs continuously. After deploying to a Basic B1 App Service Plan, the developer notices the WebJob stops running after periods of inactivity.

**Question:** What App Service configuration setting should the developer enable to resolve this issue?

- A. ARR Affinity
- B. HTTPS Only
- C. Always On
- D. Minimum TLS Version 1.2

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: C. Always On**

**Why C is correct:** Always On keeps the app loaded in memory even when there are no incoming requests. Without Always On, App Service unloads the app after a period of inactivity to conserve resources. When the app is unloaded, WebJobs stop running. Enabling Always On ensures the WebJob runs continuously. Always On is available on Basic tier and above.

**Why A is wrong:** ARR Affinity (Application Request Routing) controls whether user requests are routed to the same instance (sticky sessions). It has nothing to do with keeping the app loaded or WebJob execution.

**Why B is wrong:** HTTPS Only redirects HTTP requests to HTTPS for secure communication. It does not affect whether the app stays loaded in memory.

**Why D is wrong:** Minimum TLS version controls the encryption protocol version. Setting it to 1.2 is a security best practice, but it has nothing to do with keeping the app running during idle periods.

</details>

---

### Question 4: Easy Auth Provider

**Scenario:** A company builds an internal web application that only company employees should access. All employees have Microsoft 365 accounts managed through Microsoft Entra ID (formerly Azure Active Directory).

**Question:** Where is App Service Easy Auth (built-in authentication) configured?

- A. In the application's source code by adding authentication middleware
- B. At the App Service level through Azure portal configuration
- C. In the App Service Plan settings
- D. In the Azure subscription's security centre

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B. At the App Service level through Azure portal configuration**

🚨 **EXAM ALERT:** This is a commonly tested concept. Easy Auth is configured at the App Service resource level — not in code.

**Why B is correct:** Easy Auth (built-in authentication/authorization) is configured at the App Service level. In the Azure portal, you navigate to your App Service → Authentication and configure the identity providers. No changes to application code are needed. App Service intercepts requests and validates tokens before they reach your code.

**Why A is wrong:** Easy Auth does NOT require code changes. The entire point of "built-in" authentication is that App Service handles it at the platform level. While you CAN add authentication middleware in code (using libraries like MSAL), that is a separate approach — not Easy Auth.

**Why C is wrong:** The App Service Plan defines compute resources (tier, instances, features). Authentication is configured on the App Service (the app), not the Plan (the hardware).

**Why D is wrong:** Azure Security Centre (Microsoft Defender for Cloud) provides security monitoring and recommendations. It does not configure App Service authentication.

</details>

---

### Question 5: VNet Integration Direction

**Scenario:** A developer deploys a web app to Azure App Service on a Standard S1 plan. The app needs to access an Azure SQL Database that is inside a Virtual Network with no public endpoint.

**Question:** What does VNet Integration enable for the App Service?

- A. It allows inbound traffic from the VNet to reach the App Service
- B. It allows the App Service to make outbound calls to resources inside the VNet
- C. It places the App Service fully inside the VNet, replacing its public URL
- D. It encrypts all traffic between the App Service and the internet

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B. It allows the App Service to make outbound calls to resources inside the VNet**

🚨 **EXAM ALERT:** VNet Integration is OUTBOUND ONLY. This is tested frequently.

**Why B is correct:** VNet Integration allows the App Service to make outbound connections to resources inside a Virtual Network. In this scenario, the app can reach the SQL Database through its private IP inside the VNet. Traffic flows FROM the app INTO the VNet.

**Why A is wrong:** VNet Integration does not control inbound traffic. To control who can reach the App Service from the internet, use Access Restrictions (inbound IP firewall).

**Why C is wrong:** VNet Integration does NOT place the App Service inside the VNet. The app still has its public URL (`https://appname.azurewebsites.net`). To fully place an app inside a VNet, you need an App Service Environment (Isolated tier).

**Why D is wrong:** VNet Integration is about network routing (connecting the app to a VNet), not about encryption. HTTPS and TLS settings handle encryption.

</details>

---

## Part 2: Applied Questions (6–10)

These questions test your ability to apply App Service concepts to real-world scenarios.

---

### Question 6: Deployment Slot Swap — Sticky Settings

**Scenario:** A company has a production web app at `https://contoso.azurewebsites.net` and a staging slot at `https://contoso-staging.azurewebsites.net`. The production slot has a connection string marked as a **slot setting** pointing to the production database (`ProdDB`). The staging slot has a connection string marked as a slot setting pointing to the test database (`TestDB`). A developer deploys new code to the staging slot and then performs a swap.

**Question:** After the swap completes, which database does the production slot connect to?

- A. TestDB — the connection string moves with the code from staging
- B. ProdDB — the connection string stays with the production slot
- C. Both TestDB and ProdDB — the slot has two connection strings after swap
- D. Neither — connection strings are deleted during a swap

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B. ProdDB — the connection string stays with the production slot**

🚨 **EXAM ALERT:** Sticky settings (slot settings) stay with the SLOT, not the code. This is the #1 most tested App Service concept.

**Why B is correct:** When a connection string is marked as a "slot setting" (sticky), it stays with the slot during a swap. The production slot's connection string pointing to ProdDB stays with production. The staging code moves to production, but the connection string does NOT move. Production continues connecting to ProdDB.

**Why A is wrong:** This would be true if the connection string was NOT marked as a slot setting (non-sticky). Non-sticky settings travel with the code during a swap. But the scenario explicitly states the connection string is marked as a slot setting, so it stays with the slot.

**Why C is wrong:** Swap does not duplicate settings. Each slot maintains exactly one value for each connection string name.

**Why D is wrong:** Swap never deletes settings. It either moves them (non-sticky) or keeps them in place (sticky).

</details>

---

### Question 7: Autoscale Rule Configuration

**Scenario:** An e-commerce application runs on Azure App Service Standard S2 with 2 instances. During a flash sale, CPU usage spikes to 90% for extended periods, causing slow response times. The operations team wants to automatically add instances when load is high and remove them when load returns to normal.

**Question:** Which autoscale rule configuration correctly addresses this scenario?

- A. Scale out: When average CPU > 70% for 10 minutes, add 1 instance (cooldown 5 min). Scale in: When average CPU < 30% for 10 minutes, remove 1 instance (cooldown 5 min).
- B. Scale up: When average CPU > 70% for 10 minutes, change from S2 to S3. Scale down: When average CPU < 30%, change from S3 to S2.
- C. Scale out: When average CPU > 70% for 10 minutes, add 1 instance (cooldown 5 min). No scale-in rule defined.
- D. Scale out: When average CPU > 70% for 1 second, add 5 instances (no cooldown).

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: A. Scale out with matching scale in rules**

**Why A is correct:** This follows autoscale best practices. The scale-out rule adds instances when CPU exceeds 70% for a sustained period (10 minutes — prevents reacting to momentary spikes). The cooldown of 5 minutes prevents rapid scaling oscillation. The scale-in rule removes instances when CPU drops below 30%, ensuring you do not pay for unnecessary instances. Both rules work together for a balanced autoscale configuration.

**Why B is wrong:** Scale UP (changing tier from S2 to S3) is a manual operation that changes the machine size — it is NOT autoscale. Autoscale is about scaling OUT (adding/removing instances of the SAME tier). Additionally, scale-up requires a brief restart, which would impact users during a flash sale.

**Why C is wrong:** While the scale-out rule is correct, having no scale-in rule is a poor practice. After the flash sale ends and CPU drops, you would continue paying for extra instances indefinitely. Always define matching scale-out AND scale-in rules.

**Why D is wrong:** A 1-second duration with no cooldown would cause aggressive, unstable scaling. A momentary CPU spike lasting 1 second would trigger adding 5 instances, which is excessive. Without cooldown, scaling actions could fire continuously, creating an oscillation pattern.

</details>

---

### Question 8: Deployment Method Selection

**Scenario:** A development team uses GitHub as their source control repository. They want every push to the `main` branch to automatically deploy to their Azure App Service staging slot. The deployment should include automated tests before deployment completes.

**Question:** Which deployment method is most appropriate?

- A. FTP upload of application files
- B. ZIP deploy using Azure CLI
- C. GitHub Actions with a CI/CD workflow
- D. Local Git deployment

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: C. GitHub Actions with a CI/CD workflow**

**Why C is correct:** GitHub Actions provides a CI/CD (Continuous Integration/Continuous Deployment) pipeline that automatically triggers when code is pushed to a specific branch. You define a workflow YAML file in `.github/workflows/` that can run automated tests, build the application, and deploy to App Service. This matches the requirement of automatic deployment on push to `main` with automated testing.

**Why A is wrong:** FTP is a manual file transfer method. It does not support automatic deployment triggered by code pushes, and it does not support running automated tests. FTP is considered legacy and not recommended for new applications.

**Why B is wrong:** ZIP deploy via CLI is a manual command-line operation (`az webapp deployment source config-zip`). Someone must run the command manually each time. It does not trigger automatically when code is pushed to GitHub, and it does not include automated testing as part of the deployment.

**Why D is wrong:** Local Git allows pushing code directly to App Service via `git push azure main`, but it does not integrate with GitHub workflows. It pushes directly from a developer's machine to App Service, bypassing GitHub. It also does not include automated testing.

</details>

---

### Question 9: ARR Affinity for Stateless Apps

**Scenario:** A web application is deployed on Azure App Service with 4 instances. The application is stateless — it stores all session data in Azure Cache for Redis, not in memory. A developer notices that traffic is unevenly distributed across the 4 instances.

**Question:** Which configuration change would help distribute traffic more evenly?

- A. Enable Always On
- B. Disable ARR Affinity
- C. Enable HTTPS Only
- D. Increase the minimum TLS version to 1.3

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B. Disable ARR Affinity**

**Why B is correct:** ARR Affinity (Application Request Routing Affinity) uses cookies to route all requests from the same user to the same instance (sticky sessions). When enabled, traffic can become unevenly distributed because some instances receive more returning users than others. Since the app is stateless (session data is in Redis, not in memory), there is no reason to pin users to specific instances. Disabling ARR Affinity allows the load balancer to distribute traffic evenly across all 4 instances.

**Why A is wrong:** Always On keeps the app loaded in memory during periods of inactivity. It does not affect traffic distribution across instances.

**Why C is wrong:** HTTPS Only redirects HTTP requests to HTTPS. It affects the protocol used for communication, not how traffic is distributed across instances.

**Why D is wrong:** Minimum TLS version controls encryption protocol security. It has no effect on traffic distribution across instances. Additionally, TLS 1.3 is not currently a standard setting in App Service — the recommended setting is 1.2.

</details>

---

### Question 10: Slot URL Format

**Scenario:** A developer creates an App Service named `inventoryapp` on a Standard S1 plan. The developer creates a deployment slot named `staging`.

**Question:** What is the URL of the staging slot?

- A. `https://staging.inventoryapp.azurewebsites.net`
- B. `https://inventoryapp-staging.azurewebsites.net`
- C. `https://inventoryapp.azurewebsites.net/staging`
- D. `https://inventoryapp.staging.azurewebsites.net`

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B. `https://inventoryapp-staging.azurewebsites.net`**

**Why B is correct:** The URL format for a deployment slot is `https://{appname}-{slotname}.azurewebsites.net`. The app name and slot name are separated by a hyphen. For the production slot, the URL is just `https://inventoryapp.azurewebsites.net` (no suffix). For any other slot, it is `https://inventoryapp-staging.azurewebsites.net`.

**Why A is wrong:** The slot name does not go before the app name as a subdomain. `staging.inventoryapp` would imply `staging` is a subdomain of `inventoryapp`, which is not how App Service slot URLs are structured.

**Why C is wrong:** The slot is not a path segment after the app URL. `/staging` would be a page or route within the application, not a deployment slot.

**Why D is wrong:** The slot name is not inserted between the app name and `azurewebsites.net` as a separate subdomain. It is appended to the app name with a hyphen.

</details>

---

## Part 3: Exam-Level Questions (11–15)

These questions mirror the difficulty and format of actual AZ-204 exam questions.

---

### Question 11: SELECT TWO — Deployment Slot Requirements

**Scenario:** A company is planning to implement deployment slots for their Azure App Service web application to achieve zero-downtime deployments. They currently run on a Basic B2 plan.

**Question:** Which TWO actions must the company take to use deployment slots? (Select two.)

- A. Upgrade the App Service Plan to Standard S1 or higher
- B. Enable VNet Integration on the App Service
- C. Create at least one additional deployment slot (e.g., staging)
- D. Enable ARR Affinity on the App Service
- E. Move the App Service to a Linux hosting environment

<details>
<summary>Click to reveal answer</summary>

**Correct Answers: A and C**

**Why A is correct:** Deployment slots require Standard tier or higher. The company is currently on Basic B2, which does NOT support slots. They must upgrade to at least Standard S1 to enable the deployment slots feature.

**Why C is correct:** Every App Service has a production slot by default, but to perform slot swaps, you need at least one additional slot (such as a staging slot). You must explicitly create this additional slot.

**Why B is wrong:** VNet Integration allows outbound connections to Virtual Network resources. It is not required for deployment slots. Slots work independently of networking features.

**Why D is wrong:** ARR Affinity controls sticky sessions (routing users to the same instance). It is not required for deployment slots. Slots work regardless of the ARR Affinity setting.

**Why E is wrong:** Deployment slots work on both Windows and Linux App Service plans. There is no requirement to use a specific operating system. The choice between Windows and Linux is based on your application's runtime needs, not slot requirements.

</details>

---

### Question 12: Autoscale Instance Behaviour

**Scenario:** An Azure App Service is configured with autoscale rules on a Standard S2 plan. The autoscale configuration has a minimum of 2 instances, a maximum of 8 instances, and a default of 2 instances. The scale-out rule specifies: "When average CPU > 75% for 10 minutes, increase instance count by 2." Currently, 4 instances are running and the average CPU across all instances has been above 75% for 12 minutes.

**Question:** How many instances will be running after the autoscale action completes?

- A. 5 instances
- B. 6 instances
- C. 8 instances
- D. 10 instances

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B. 6 instances**

**Why B is correct:** The autoscale rule says "increase instance count by 2." Currently there are 4 instances running. 4 + 2 = 6 instances. This is within the maximum limit of 8 instances, so the action proceeds. All 6 instances are identical copies of the application on S2 hardware.

**Why A is wrong:** The rule adds 2 instances, not 1. If the rule said "increase by 1," the answer would be 5.

**Why C is wrong:** 8 is the maximum limit, but the current scale-out action only adds 2 (from 4 to 6). The maximum only prevents scaling beyond 8 — it does not automatically scale to the maximum.

**Why D is wrong:** 4 + 2 = 6, not 10. Even if it were 10, the maximum limit of 8 would cap it at 8. Autoscale never exceeds the maximum instance count.

</details>

---

### Question 13: Easy Auth Token Handling

**Scenario:** A developer configures Easy Auth on an Azure App Service web application using Microsoft Entra ID as the identity provider. The application needs to call Microsoft Graph API to read the signed-in user's profile. The developer wants to access the authentication tokens that Easy Auth obtains.

**Question:** How does Easy Auth make authentication tokens available to the application?

- A. Easy Auth writes tokens to Azure Blob Storage, and the app reads them from the container
- B. Easy Auth injects tokens into HTTP request headers that the application code can read
- C. Easy Auth stores tokens in the App Service Plan's configuration settings
- D. Easy Auth requires the developer to implement MSAL library in the application code to obtain tokens

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B. Easy Auth injects tokens into HTTP request headers that the application code can read**

**Why B is correct:** When Easy Auth validates a request, it injects the authentication token and user claims into special HTTP request headers (such as `X-MS-TOKEN-AAD-ACCESS-TOKEN` for Entra ID). Your application code can read these headers to access the user's identity information and tokens. Additionally, tokens are cached in the built-in token store. The app can call `/.auth/me` to retrieve token information.

**Why A is wrong:** Easy Auth does not use Blob Storage for tokens. Tokens are stored in the App Service's built-in token store (an encrypted cache associated with the App Service), not in an external storage service.

**Why C is wrong:** Tokens are not stored in the App Service Plan's configuration. The App Service Plan defines compute resources (tier, instances). Tokens are managed per-request and cached in the App Service's token store.

**Why D is wrong:** Easy Auth is specifically designed so that developers do NOT need to implement MSAL or any authentication library. While you CAN use MSAL for more advanced scenarios, Easy Auth's purpose is to handle authentication without code changes. If the question specifically mentions Easy Auth, the answer should not involve writing authentication code.

</details>

---

### Question 14: SELECT TWO — Networking Features

**Scenario:** A company has an Azure App Service web application on a Standard S1 plan. They have two requirements:
1. The app must be able to connect to an Azure SQL Database that has a private endpoint inside a Virtual Network (no public access).
2. Only requests from the company's office IP address (198.51.100.10) should be able to access the web app.

**Question:** Which TWO features should the developer configure? (Select two.)

- A. VNet Integration to enable the app to reach the SQL Database through the VNet
- B. Hybrid Connections to connect to the SQL Database
- C. Access Restrictions to allow only the office IP address
- D. Easy Auth to authenticate requests from the office IP
- E. App Service Environment (Isolated tier) to place the app inside the VNet

<details>
<summary>Click to reveal answer</summary>

**Correct Answers: A and C**

**Why A is correct:** VNet Integration allows the App Service to make outbound connections to resources inside a Virtual Network. Since the SQL Database has a private endpoint inside the VNet (no public access), the App Service needs VNet Integration to route outbound traffic through the VNet and reach the database via its private IP. Standard S1 supports VNet Integration.

**Why C is correct:** Access Restrictions provide an inbound IP firewall. By creating an allow rule for `198.51.100.10/32` (the office IP) and a default deny rule, only requests from the office can reach the web app. This satisfies the second requirement of restricting access to the office IP.

**Why B is wrong:** Hybrid Connections are designed for connecting to on-premises resources (resources outside Azure, in your company's data centre). The SQL Database is in Azure (inside a VNet), not on-premises. VNet Integration is the correct feature for connecting to Azure VNet resources.

**Why D is wrong:** Easy Auth handles user authentication (verifying identity via login). It does not filter traffic by IP address. Even if Easy Auth blocks unauthenticated users, it does not restrict by network location. Access Restrictions is the correct feature for IP-based filtering.

**Why E is wrong:** An App Service Environment (ASE) on the Isolated tier would provide full VNet isolation and solve both problems. However, it requires upgrading to the Isolated tier (significantly more expensive) and is a much more complex solution. VNet Integration + Access Restrictions on the existing Standard plan solves both requirements without the cost and complexity of ASE.

</details>

---

### Question 15: Comprehensive Scenario — Tier, Slots, and Autoscale

**Scenario:** A retail company operates an e-commerce web application on Azure App Service. They have the following requirements:

1. Zero-downtime deployments using staging slots
2. Automatic scaling when CPU exceeds 70% (up to 10 instances)
3. The staging slot's connection string must point to a test database, and the production slot's connection string must point to the production database — even after swaps
4. The application is stateless and stores session data in Azure Cache for Redis

**Question:** Which set of configurations correctly satisfies ALL requirements?

- A. Basic B3 plan, create staging slot, configure autoscale, mark connection strings as slot settings, disable ARR Affinity
- B. Standard S1 plan, create staging slot, configure autoscale, mark connection strings as slot settings, disable ARR Affinity
- C. Standard S1 plan, create staging slot, configure autoscale, leave connection strings as regular settings, enable ARR Affinity
- D. Free F1 plan, create staging slot, configure autoscale, mark connection strings as slot settings, disable ARR Affinity

<details>
<summary>Click to reveal answer</summary>

**Correct Answer: B. Standard S1 plan, create staging slot, configure autoscale, mark connection strings as slot settings, disable ARR Affinity**

**Why B is correct:** Let us check each requirement:
1. ✅ **Zero-downtime deployments:** Standard S1 supports deployment slots (up to 5). A staging slot is created.
2. ✅ **Autoscale up to 10 instances:** Standard supports autoscale with up to 10 instances. Autoscale rules can be configured for CPU > 70%.
3. ✅ **Connection strings stay with slots:** Marking connection strings as "slot settings" makes them sticky — they stay with their respective slots during swaps. Production keeps its production DB connection; staging keeps its test DB connection.
4. ✅ **Stateless app:** Disabling ARR Affinity allows traffic to be evenly distributed across instances since the app does not rely on in-memory sessions (Redis handles session data).

**Why A is wrong:** Basic B3 does NOT support deployment slots and does NOT support autoscale. Both features require Standard tier or higher. This option fails requirements 1 and 2.

**Why C is wrong:** Standard S1 correctly supports slots and autoscale. However, leaving connection strings as regular (non-sticky) settings means they WILL swap with the code. After a swap, production would connect to the test database — which violates requirement 3. Also, enabling ARR Affinity for a stateless app is unnecessary and causes uneven traffic distribution (though it does not break functionality).

**Why D is wrong:** Free F1 does NOT support deployment slots, autoscale, or custom scaling. It fails requirements 1, 2, and effectively 3 (since slots do not exist). Free tier is designed for testing only.

</details>

---

## Quiz Score Tracking

| Question | Topic | Difficulty | Your Answer | Correct? |
|---|---|---|---|---|
| 1 | Plan tier selection | Foundation | | |
| 2 | Scale up vs scale out | Foundation | | |
| 3 | Always On configuration | Foundation | | |
| 4 | Easy Auth configuration | Foundation | | |
| 5 | VNet Integration direction | Foundation | | |
| 6 | Sticky settings during swap | Applied | | |
| 7 | Autoscale rule configuration | Applied | | |
| 8 | Deployment method selection | Applied | | |
| 9 | ARR Affinity for stateless apps | Applied | | |
| 10 | Slot URL format | Applied | | |
| 11 | Slot requirements (SELECT TWO) | Exam-Level | | |
| 12 | Autoscale instance behaviour | Exam-Level | | |
| 13 | Easy Auth token handling | Exam-Level | | |
| 14 | Networking features (SELECT TWO) | Exam-Level | | |
| 15 | Comprehensive scenario | Exam-Level | | |

**Total: ___ / 15**

---

## Key Exam Takeaways from This Quiz

If you missed any questions, review these critical facts:

| Fact | Source Section |
|---|---|
| Deployment slots require **Standard tier or higher** | Section 2 (Plans) and Section 4 (Slots) |
| Autoscale requires **Standard tier or higher** | Section 2 (Plans) and Section 5 (Autoscaling) |
| Sticky settings **stay with the SLOT** — they do NOT swap with the code | Section 4 (Slots) |
| Autoscale adds **identical instances** (scale out ≠ scale up) | Section 5 (Autoscaling) |
| VNet Integration is **OUTBOUND ONLY** | Section 8 (Networking) |
| Easy Auth is configured at the **App Service level**, not in code | Section 7 (Easy Auth) |
| ARR Affinity should be **disabled** for stateless apps | Section 6 (Configuration) |
| Slot URL format: `https://{appname}-{slotname}.azurewebsites.net` | Section 4 (Slots) |
| Always On is required for **WebJobs** | Section 6 (Configuration) |
| Minimum TLS version should be **1.2** | Section 6 (Configuration) |
