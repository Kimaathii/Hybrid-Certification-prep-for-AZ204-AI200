# M03 — Slide Deck: Containerised Solutions (ACR, ACI, ACA)

---

## Slide 1: Title Slide

**ON SLIDE:**
- **Module 3: Containerised Solutions — ACR, ACI, ACA**
- Exam Domain: Develop Azure Compute Solutions (25–30%)
- Learning Objectives:
  - Understand containers, images, Dockerfiles, and registries
  - Use ACR to build and store container images
  - Deploy containers to ACI for simple workloads
  - Deploy containers to ACA for production microservices
  - Configure traffic splitting between ACA revisions

**SPEAKER NOTES:**
Welcome to Module 3. This module covers Azure's three container services — ACR for storing images, ACI for running simple containers, and ACA for production microservices. Containers are a significant part of the Compute domain on the exam, and the key skill is knowing which service fits which scenario.

---

## Slide 2: The Problem — Why Containers Exist

**ON SLIDE:**
- "It works on my machine!" — the classic developer excuse
- Problem: different environments have different configurations
- Your laptop ≠ test server ≠ production server
- Solution: package EVERYTHING your app needs into one standardised box

**SPEAKER NOTES:**
Before containers, deploying software was unreliable. An app might work perfectly on a developer's laptop but crash on the production server because of different library versions, missing dependencies, or configuration differences. Containers solve this by packaging the application with all its dependencies into a standardised unit that runs identically everywhere.

---

## Slide 3: What Is a Container?

**ON SLIDE:**
- Analogy: **Shipping container** — standardised box, runs on any ship/truck/train
- Container = your code + dependencies + runtime, packaged together
- **Image** = blueprint (read-only template)
- **Container** = running instance of an image
- **Dockerfile** = recipe to build the image
- **Registry** = warehouse that stores images

**SPEAKER NOTES:**
Think of containers like shipping containers. Before standardised shipping containers, every port handled cargo differently. The shipping container standardised everything — same shape, any ship. Software containers do the same thing. An image is the blueprint, a container is the running copy, a Dockerfile is the recipe to build it, and a registry is the warehouse.

---

## Slide 4: Container vs VM

**ON SLIDE:**

| | VM | Container |
|---|---|---|
| Includes | Full OS | App + dependencies only |
| Size | Gigabytes | Megabytes |
| Startup | Minutes | Seconds |
| Isolation | Hardware-level | Process-level |
| AZ-204 Focus | ⚪ No | 🔴 Yes |

**SPEAKER NOTES:**
VMs include a complete operating system — that's why they're gigabytes and take minutes to start. Containers share the host OS kernel, so they only include the app and its specific dependencies. They're megabytes in size and start in seconds. For AZ-204, containers are a core topic. VMs are not.

---

## Slide 5: Azure Container Registry (ACR)

**ON SLIDE:**
- **ACR = Azure's private Docker image warehouse**
- Tiers: Basic ($5/mo) | Standard ($10/mo) | Premium ($50/mo)
- Labs: Always use **Basic**
- Key command: `az acr create --name myacr --sku Basic`
- URL format: `myacr.azurecr.io`

**SPEAKER NOTES:**
ACR is where you store your container images in Azure. It's private — only authorised users and services can pull images. For labs, always use Basic tier. The registry name becomes part of the URL — myacr.azurecr.io.

---

## Slide 6: ⚠️ EXAM ALERT — ACR Tasks (Cloud Build)

**ON SLIDE (RED BACKGROUND):**
- `az acr build --registry myacr --image myapp:v1.0 .`
- **Builds the image IN THE CLOUD**
- ❌ Does NOT require Docker installed locally
- If the exam asks: "build image without local Docker" → **ACR Tasks**

**SPEAKER NOTES:**
This is a frequently tested concept. ACR Tasks lets you build container images entirely in the cloud. You send your source code and Dockerfile to Azure, and Azure builds the image for you. You do not need Docker installed on your machine. If the exam asks about building images without Docker locally, the answer is always az acr build with ACR Tasks.

---

## Slide 7: Azure Container Instances (ACI)

**ON SLIDE:**
- **ACI = vending machine for containers**
- Fastest way to run a container in Azure
- No VMs, no cluster, no orchestration
- Best for: quick tests, batch jobs, one-off tasks
- ❌ NOT for production microservices at scale

**SPEAKER NOTES:**
ACI is the simplest container option. You point it at an image, and it runs immediately. Think of it like a vending machine — insert an image, get a running container. It's perfect for quick tasks but not designed for large-scale production workloads with many microservices.

---

## Slide 8: ACI Restart Policies

**ON SLIDE:**

| Policy | Behaviour | Use Case |
|---|---|---|
| **Always** | Always restart | Web servers, APIs |
| **OnFailure** | Restart only on error | Batch jobs with retry |
| **Never** | Run once, stop | One-time tasks |

**SPEAKER NOTES:**
Restart policies control what happens when the container process finishes. Always is for long-running services. OnFailure retries if something goes wrong — perfect for batch jobs. Never runs once and stops — perfect for one-time data migrations or scripts. The exam will describe a scenario and you need to pick the correct policy.

---

## Slide 9: ACI Container Groups

**ON SLIDE:**
- Multiple containers sharing network + storage
- **Sidecar pattern**: main app + helper containers
- Example: App container + Logging sidecar
- Deployed and managed together as one unit

```
┌─────────────────────────────┐
│     Container Group         │
│ ┌─────────┐ ┌─────────────┐│
│ │ App     │ │ Logging     ││
│ │ :80     │ │ Sidecar     ││
│ └─────────┘ └─────────────┘│
│    Shared: Network, Storage │
└─────────────────────────────┘
```

**SPEAKER NOTES:**
Container Groups let you run multiple containers together, sharing network and storage. The most common pattern is the sidecar — a main container with a helper. For example, your app container on port 80 with a logging sidecar that collects and forwards logs. They share localhost so they can communicate directly.

---

## Slide 10: Azure Container Apps (ACA)

**ON SLIDE:**
- **ACA = Kubernetes power WITHOUT Kubernetes management**
- Serverless containers — scale to zero
- Automatic HTTPS, service discovery, load balancing
- Key concepts: Environment, Revision, Ingress, KEDA

**SPEAKER NOTES:**
ACA is the big one for production container workloads. It gives you Kubernetes features — auto-scaling, traffic management, service discovery — without you ever touching kubectl or managing nodes. It's serverless, meaning it can scale to zero replicas when idle, and you only pay for what you use.

---

## Slide 11: ACA Key Concepts

**ON SLIDE:**

| Concept | What It Is |
|---|---|
| **Environment** | Shared boundary — VNet + logging |
| **Container App** | Your application in the environment |
| **Revision** | Immutable snapshot of a version |
| **Ingress** | HTTP traffic control (external/internal) |
| **KEDA** | Event-driven autoscaling |

**SPEAKER NOTES:**
An Environment is the building that holds your apps. A Container App is one apartment. A Revision is a version snapshot — like v1.0, v2.0. Ingress controls whether the app is publicly accessible. KEDA handles auto-scaling based on HTTP traffic, queue depth, or custom metrics.

---

## Slide 12: ACA Traffic Splitting

**ON SLIDE:**

```
           Traffic
          ┌──┴──┐
     80%  │     │  20%
     ▼    │     │   ▼
  Rev 1   │     │ Rev 2
  (v1.0)  │     │ (v2.0)
```

- Deploy new version → creates new revision
- Split traffic: 80% old, 20% new
- Monitor → increase gradually → or roll back instantly

**SPEAKER NOTES:**
Traffic splitting is one of ACA's most powerful features. When you deploy a new version, ACA creates a new revision. You can then split traffic — say 80% to the stable version and 20% to the new version. If the new version works well, gradually shift to 100%. If it fails, shift back to 0% instantly. This is called a canary release.

---

## Slide 13: ⚠️ EXAM ALERT — ACA Hides Kubernetes

**ON SLIDE (RED BACKGROUND):**
- ACA uses Kubernetes under the hood
- You **NEVER** manage: nodes, pods, kubectl, cluster config
- "Serverless containers without managing Kubernetes" → **ACA**
- "Full Kubernetes control needed" → **AKS** (not on AZ-204)

**SPEAKER NOTES:**
This is critical for the exam. ACA is built on Kubernetes but completely abstracts it away. You don't manage any Kubernetes objects. If the exam says "without managing infrastructure" or "serverless containers," think ACA. If a question asks about needing full Kubernetes control with custom operators or specific Kubernetes versions, that's AKS — but AKS is not on the AZ-204 exam.

---

## Slide 14: The Decision Table

**ON SLIDE:**

| Scenario | Service |
|---|---|
| Store/manage images | **ACR** |
| Build image without local Docker | **ACR Tasks** |
| Quick one-time task | **ACI** |
| Batch job with retry | **ACI** (OnFailure) |
| Production microservices | **ACA** |
| Scale to zero | **ACA** |
| Traffic splitting | **ACA** |

**SPEAKER NOTES:**
This decision table is the most important slide for exam preparation. Memorise these mappings. The exam will describe a scenario, and you need to pick the right service. ACR for storage. ACI for simple, quick runs. ACA for anything production-scale with scaling and traffic management.

---

## Slide 15: All Three Working Together

**ON SLIDE:**

```
Developer → Dockerfile → az acr build → ACR (stores image)
                                           │
                              ┌────────────┼────────────┐
                              ▼            ▼            ▼
                           ACI          ACA          App Service
                        (quick test)  (production)  (also supports
                                      (scale, split)  containers)
```

**SPEAKER NOTES:**
In practice, all three services work together. You write a Dockerfile, build the image with ACR Tasks, store it in ACR, then deploy to either ACI for quick testing or ACA for production. App Service can also run containers, which you learned in Module 1.

---

## Slide 16: Lab Preview

**ON SLIDE:**
- Create ACR (Basic tier)
- Write a simple Node.js app + Dockerfile
- Build with `az acr build` (no local Docker)
- Deploy to ACI → access via public IP
- Deploy to ACA → configure HTTPS ingress
- Create v2 → traffic split 80/20
- 💰 Cost: Under $0.10 if cleaned up within 3 hours

**SPEAKER NOTES:**
In the lab, you'll go through the complete workflow — building an image with ACR Tasks, deploying it to both ACI and ACA, and setting up traffic splitting. Everything will cost less than 10 cents if you clean up promptly.

---

## Slide 17: Common Exam Traps

**ON SLIDE:**
1. ❌ Using ACI for production microservices → Use ACA
2. ❌ Confusing ACR (storage) with ACI (runtime) → ACR stores, ACI runs
3. ❌ Thinking ACA manages Kubernetes nodes → ACA hides everything
4. ❌ Forgetting `az acr build` works without Docker → It does!
5. ❌ Using admin credentials for production ACR → Use Managed Identity

**SPEAKER NOTES:**
Let's review the most common mistakes students make. ACI is not for production microservices. ACR and ACI are different services — one stores images, the other runs them. ACA hides Kubernetes completely. ACR Tasks builds without Docker. And admin credentials should only be used in labs, not production.

---

## Slide 18: Module Summary Table

**ON SLIDE:**

| Service | Purpose | Best For | Key Command |
|---|---|---|---|
| **ACR** | Private image registry | Storing and building images | `az acr build` |
| **ACI** | Run containers quickly | One-off tasks, batch jobs | `az container create` |
| **ACA** | Serverless container platform | Production microservices | `az containerapp create` |

**SPEAKER NOTES:**
Here's the summary table. ACR stores and builds images. ACI runs containers quickly for simple tasks. ACA is the full serverless container platform for production workloads. Know these three services, their purposes, and when to use each.

---

## Slide 19: Checkpoint Questions

**ON SLIDE:**
1. A developer needs to build a Docker image without having Docker installed locally. What should they use?
2. A company has a nightly batch job that should retry on failure and stop when successful. Which service and restart policy?
3. A company needs serverless containers that scale to zero, with traffic splitting between versions. Which service?

**SPEAKER NOTES:**
Let's test your understanding before we wrap up. Take 30 seconds to think about each question before I reveal the answers.

---

## Slide 20: Checkpoint Answers

**ON SLIDE:**
1. **ACR Tasks** — `az acr build` builds images in the cloud. No local Docker needed.
2. **ACI with restart policy `OnFailure`** — runs the batch job, retries on failure, stops on success.
3. **Azure Container Apps (ACA)** — serverless, scales to zero, traffic splitting via revisions.

**SPEAKER NOTES:**
If you got all three right, you're ready for the container questions on the exam. If any surprised you, go back to the concept document and review that section.

---

## Slide 21: Key Takeaways

**ON SLIDE:**
- 🏗️ ACR = **Build and store** container images
- 🚀 ACI = **Run quickly**, simple tasks
- 🌐 ACA = **Production microservices**, scale to zero, traffic splitting
- 🔑 `az acr build` = No local Docker needed
- ⚠️ ACI ≠ production microservices. ACA ≠ managing Kubernetes.

**SPEAKER NOTES:**
These five bullets are what you need to remember from this module. ACR for building and storing. ACI for quick simple runs. ACA for production with scaling and traffic splitting. az acr build doesn't need local Docker. And know the anti-patterns — ACI is not for production microservices, and ACA doesn't expose Kubernetes management.

---

## Slide 22: Module Connections

**ON SLIDE:**
- **M01 App Service** — Also runs containers (Docker deploy)
- **M02 Functions** — Functions can run on ACA hosting plan
- **M04 Blob Storage** — ACR uses blob storage internally
- **M07 Security** — Managed Identity for ACR authentication
- **Next: M04 — Azure Blob Storage**

**SPEAKER NOTES:**
Container concepts connect to several other modules. App Service can also run containers. Functions can be hosted on Container Apps. ACR uses blob storage internally. And you'll learn about Managed Identity in Module 7, which is the recommended way to authenticate with ACR in production. Next up is Module 4 on Azure Blob Storage.
