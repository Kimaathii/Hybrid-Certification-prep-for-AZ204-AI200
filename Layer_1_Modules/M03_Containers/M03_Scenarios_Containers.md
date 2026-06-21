# M03 — Scenario Cards: Containerised Solutions (ACR, ACI, ACA)

---

## SCENARIO 1 — Beginner

**TITLE:** The Data Migration Script

### BUSINESS CONTEXT:
A financial services company has a Python script that migrates data from an old database to a new one. The script takes about 45 minutes to run, processes all records, and then exits.

### THE PROBLEM:
The DevOps team needs to run this migration script in Azure. They do not want to set up a VM or maintain any infrastructure. The script should run once, complete its work, and then the resources should stop consuming costs.

### CONSTRAINTS:
- The script must run once and stop — no restarts
- No infrastructure management
- Must pull the container image from their private Azure registry
- Minimal cost

### YOUR TASK:
Which Azure container service should they use, and what restart policy should they configure?

### HINTS:
- Think about which service is designed for quick, one-off container runs
- Which restart policy means "run once and never restart"?

### SOLUTION:
**Azure Container Instances (ACI)** with restart policy set to **`Never`**.

ACI is the fastest and simplest way to run a container in Azure with zero infrastructure management. The `Never` restart policy ensures the container runs once, completes the migration, and then stops — no wasted compute after completion. ACI bills per-second, so the company pays only for the 45 minutes of actual execution.

They would pull the image from their ACR using admin credentials (or Managed Identity) and deploy with:
```bash
az container create --restart-policy Never --image myacr.azurecr.io/migration:v1.0
```

### EXAM CONNECTION:
The exam frequently tests the mapping between restart policies and use cases. "One-time task that should stop" → `Never`. "Batch job with retry" → `OnFailure`. "Always-running service" → `Always`.

---

## SCENARIO 2 — Intermediate

**TITLE:** The Image Processing Pipeline

### BUSINESS CONTEXT:
A media company receives thousands of user-uploaded photos daily. Each photo needs to be resized into 5 different sizes for web, mobile, and thumbnail use. Currently, a developer manually runs the processing on their laptop.

### THE PROBLEM:
The company needs to automate this processing in Azure. They want a container that:
- Picks up new images from a queue
- Processes them
- Uploads the results to blob storage
- Scales automatically based on queue depth
- Costs nothing when there are no images to process

### CONSTRAINTS:
- Must scale based on queue message count
- Must scale to zero when no work is available
- Must use containers (their processing code is containerised)
- Cost-efficient — no paying for idle time

### YOUR TASK:
Which Azure container service should they use? Justify why the other options are not the best fit.

### HINTS:
- Which service can scale to zero?
- Which service supports queue-based autoscaling?

### SOLUTION:
**Azure Container Apps (ACA)** with **KEDA queue-based scaling**.

ACA is the correct choice because it:
1. **Scales to zero** when the queue is empty (no cost when idle)
2. **Scales automatically** based on queue depth using KEDA
3. Supports **container workloads** natively
4. Requires **no infrastructure management**

Why NOT ACI: ACI does not support auto-scaling based on queue depth. You would need to manually create/destroy containers.

Why NOT a VM: VMs require manual scaling and infrastructure management, and you pay even when idle.

### EXAM CONNECTION:
This tests the combination of "scale to zero" + "event-driven scaling" which points directly to ACA with KEDA. The exam may describe a queue-based processing scenario — always consider ACA when scaling to zero is a requirement.

---

## SCENARIO 3 — Intermediate

**TITLE:** The Multi-Service Web Application

### BUSINESS CONTEXT:
An e-commerce startup has built their application as 6 microservices (user auth, product catalog, shopping cart, order processing, payment gateway, notification service). Each service is containerised.

### THE PROBLEM:
They need to deploy all 6 services to Azure with:
- Auto-scaling for each service independently
- Internal communication between services
- External HTTPS access only for the product catalog and user auth services
- Blue/green deployment capability for safe updates

### CONSTRAINTS:
- All services containerised
- Independent scaling per service
- Both internal and external ingress needed
- Must support gradual traffic shifting for deployments
- Team has no Kubernetes experience

### YOUR TASK:
Which Azure container service should they use? Design the high-level architecture.

### HINTS:
- Which service supports multiple containers with independent scaling?
- Which service offers traffic splitting without Kubernetes knowledge?

### SOLUTION:
**Azure Container Apps (ACA)** — deploy all 6 services within a single **ACA Environment**.

Architecture:
- **1 ACA Environment** — shared VNet and logging boundary
- **6 Container Apps** — one per microservice
- **External ingress** on user auth and product catalog
- **Internal ingress** on the other 4 services (not publicly accessible)
- **KEDA scaling** configured per service based on their specific needs
- **Traffic splitting** via revisions for blue/green deployments

Why NOT ACI: ACI doesn't support independent scaling of multiple services, traffic splitting, or service discovery between containers.

Why NOT AKS: The team has no Kubernetes experience. ACA provides the same microservice features without requiring Kubernetes knowledge.

### EXAM CONNECTION:
When the exam describes multiple microservices needing independent scaling, service discovery, and traffic splitting without Kubernetes management → the answer is ACA. The phrase "without managing infrastructure/Kubernetes" is the key signal.

---

## SCENARIO 4 — Advanced

**TITLE:** The CI/CD Pipeline Without Local Docker

### BUSINESS CONTEXT:
A consulting firm has developers who work on lightweight laptops. They cannot install Docker Desktop due to licensing costs and company policy. However, they need to build container images as part of their CI/CD pipeline.

### THE PROBLEM:
The development team needs to:
- Build container images from their source code
- Store images in a private registry
- Trigger builds automatically when code is pushed to Git
- Never require Docker Desktop on any developer's machine

### CONSTRAINTS:
- Docker Desktop cannot be installed on developer machines
- Images must be stored privately (not on public Docker Hub)
- Builds should be automated from Git push
- Must integrate with their Azure-based deployment pipeline

### YOUR TASK:
Design the image build and storage solution using Azure services. Which ACR features would you use?

### HINTS:
- Which ACR feature builds images in the cloud?
- Can ACR Tasks be triggered automatically?

### SOLUTION:
Use **Azure Container Registry (ACR)** with **ACR Tasks** for cloud-based builds.

Setup:
1. **Create ACR** (Standard or Premium tier for automated tasks)
2. **Use `az acr build`** for manual cloud builds — sends source code to Azure, builds the image there, stores it in ACR
3. **Configure ACR Tasks with Git trigger** — automatically builds a new image when code is pushed to a branch:
   ```bash
   az acr task create \
     --name auto-build \
     --registry myacr \
     --image myapp:{{.Run.ID}} \
     --context https://github.com/company/repo.git \
     --file Dockerfile \
     --git-access-token $GIT_TOKEN
   ```
4. Images are stored privately in ACR — only authorised users and services can access them

This eliminates Docker Desktop entirely from the developer workflow. Developers write code and Dockerfiles, push to Git, and ACR Tasks handles the building.

### EXAM CONNECTION:
This tests knowledge of ACR Tasks beyond basic `az acr build`. The exam may ask about automated builds triggered by source code changes — the answer is ACR Tasks with Git triggers.

---

## SCENARIO 5 — Advanced

**TITLE:** The Canary Release Strategy

### BUSINESS CONTEXT:
A SaaS company runs their API backend as a container app serving 50,000 requests per minute. They deploy updates weekly but have experienced outages when a new version had a bug that wasn't caught in testing.

### THE PROBLEM:
The company needs a deployment strategy that:
- Tests new versions with real production traffic before full rollout
- Allows instant rollback if the new version has issues
- Does not require a separate test environment
- Minimises risk to the 50,000 requests/minute workload

### CONSTRAINTS:
- Cannot afford downtime (SLA requirement)
- Must use real production traffic for testing, not synthetic
- Must be able to roll back in seconds, not minutes
- Currently running on Azure Container Apps

### YOUR TASK:
Design the deployment strategy using ACA features. Specify the traffic percentages and rollback procedure.

### HINTS:
- What ACA feature allows percentage-based traffic routing?
- How are revisions used in ACA deployments?

### SOLUTION:
Use **ACA Revisions with progressive traffic splitting** (canary release):

**Deployment procedure:**
1. Deploy new version → ACA creates **Revision 2** automatically
2. Set traffic split: **95% Revision 1** (current stable) / **5% Revision 2** (new)
3. Monitor Revision 2 for errors, latency, and success rate (5 minutes)
4. If healthy: shift to **80/20**, monitor again (10 minutes)
5. If still healthy: shift to **50/50**, monitor (10 minutes)
6. If still healthy: shift to **0/100** (full rollout to Revision 2)
7. Deactivate Revision 1

**Rollback procedure (if Revision 2 has issues at any stage):**
1. Immediately shift traffic: **100% Revision 1** / **0% Revision 2**
2. This takes effect in seconds — instant rollback
3. Investigate and fix Revision 2 before retrying

```bash
# Rollback command — instant
az containerapp ingress traffic set \
  --name api-backend \
  --resource-group prod-rg \
  --revision-weight revision1=100 revision2=0
```

### EXAM CONNECTION:
The exam tests understanding of ACA revisions and traffic splitting. Key points: revisions are immutable snapshots, traffic splitting enables canary releases, and rollback is instant by shifting traffic percentages. This is similar to App Service deployment slots (M01) but uses percentage-based routing instead of binary swap.
