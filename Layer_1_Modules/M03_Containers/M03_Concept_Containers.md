# M03 — Containerised Solutions: ACR, ACI, and ACA

---

| | |
|---|---|
| **Module** | M03 — Containerised Solutions |
| **Exam Domain** | Develop Azure Compute Solutions (25–30%) |
| **Exam Weight** | 🔴 HIGH |
| **Study Time** | 3–4 hours |
| **Prerequisites** | F01–F06, M01, M02 |
| **Services Covered** | Azure Container Registry (ACR), Azure Container Instances (ACI), Azure Container Apps (ACA) |

---

## Section 1: What Is a Container? 🔴 HIGH EXAM WEIGHT

### The Analogy: Shipping Containers Changed the World

Before standardised shipping containers existed, moving goods internationally was chaos. Every ship loaded cargo differently — barrels, crates, bags, loose items. Every port needed different equipment. Every transfer risked damage, delays, and confusion.

Then someone invented the **shipping container** — a standardised metal box. It did not matter what was inside (electronics, food, furniture). The container was always the same shape, the same size, and could be loaded onto any ship, any truck, any train. The transport system no longer cared about the contents — it only needed to handle containers.

**Software containers work the same way.** A container is a standardised package that contains your application code, all its dependencies (libraries, runtimes, configuration files), and everything it needs to run. It does not matter what is inside — a Node.js web app, a .NET API, a Python data processor. The container is always the same shape, and it runs identically on any machine that supports containers.

> 💡 **KEY CONCEPT**
>
> A **container** is a lightweight, standalone, executable package that includes everything needed to run a piece of software: the code, runtime, system tools, libraries, and settings. It runs the same way on your laptop, your colleague's laptop, a test server, and Azure.

### Docker: The Container Platform

**Docker** is the most popular tool for creating and running containers. Think of Docker as the container shipping company — it provides the tools to build containers, run them, and manage them.

Here are the key Docker concepts you need for AZ-204:

| Term | What It Is | Analogy |
|---|---|---|
| **Image** | A read-only blueprint/template for a container. Contains all the code, dependencies, and configuration. | A recipe — you can make many dishes (containers) from one recipe (image) |
| **Container** | A running instance of an image. The actual executing application. | The dish — made from the recipe, actually being served |
| **Dockerfile** | A text file with step-by-step instructions for building an image. | The recipe card — step-by-step instructions to create the dish |
| **Registry** | A storage location for images. Like a library of recipes. | A cookbook library — stores recipes so anyone can find and use them |
| **Tag** | A version label on an image (e.g., `myapp:v1.0`, `myapp:latest`). | Edition number on a book |

### Dockerfile: The Recipe Card

A **Dockerfile** is a plain text file that tells Docker how to build an image. Here is a simple example for a Node.js application:

```dockerfile
# Start from an official Node.js base image (like starting with a pre-made cake base)
FROM node:18-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package.json first (for efficient caching of dependencies)
COPY package*.json ./

# Install dependencies listed in package.json
RUN npm install

# Copy the rest of the application code
COPY . .

# Tell Docker which port the app listens on (documentation, not enforcement)
EXPOSE 3000

# The command that runs when the container starts
CMD ["node", "server.js"]
```

Each line is a step. Docker reads them top to bottom to build the image.

### Container vs Virtual Machine (VM)

You know from AZ-900 that virtual machines give you a full computer in the cloud. Containers are lighter:

| Feature | Virtual Machine (VM) | Container |
|---|---|---|
| **What it includes** | Full operating system + your app | Just your app + its dependencies |
| **Size** | Gigabytes (2–50 GB) | Megabytes (50–500 MB) |
| **Startup time** | Minutes | Seconds |
| **Isolation** | Full hardware-level isolation | Process-level isolation (shares host OS kernel) |
| **Resource usage** | Heavy — each VM runs its own OS | Light — many containers share one OS |
| **AZ-204 relevance** | ⚪ Not a focus | 🔴 Core exam topic |

> 📝 **NOTE**
>
> **Kernel** is the core of an operating system — it manages memory, processes, and hardware. Containers share the host machine's kernel, which is why they are so lightweight. VMs each have their own kernel, which is why they are heavier but more isolated.

> ✅ **CHECKPOINT 1**
>
> Before continuing, can you answer these questions?
> 1. What is the difference between a Docker image and a Docker container?
> 2. What does a Dockerfile do?
> 3. Why are containers lighter than VMs?
>
> **Answers:** 1) Image = blueprint/template (read-only). Container = running instance of an image. 2) A Dockerfile contains step-by-step instructions to build an image. 3) Containers share the host OS kernel instead of running their own full OS.

---

## Section 2: Azure Container Registry (ACR) 🔴 HIGH EXAM WEIGHT

### The Problem

You have built a Docker image on your laptop. Now you need to:
- Store it somewhere safe (not just on your laptop)
- Share it with your team
- Deploy it to Azure services (ACI, ACA, App Service)

You could use **Docker Hub** (the public registry), but for business applications you want a **private** registry — one that only your team and your Azure resources can access.

### The Solution: Azure Container Registry

**Azure Container Registry (ACR)** is Azure's private container image registry. Think of it as a **private warehouse** for your container images — only people and services you authorise can access the images stored there.

> 💡 **KEY CONCEPT**
>
> **ACR** = a private Docker image warehouse in Azure. You push images to it, and Azure services (ACI, ACA, App Service) pull images from it to run your containers.

### ACR Tiers

| Tier | Storage | Throughput | Geo-Replication | Best For |
|---|---|---|---|---|
| **Basic** | 10 GB | Low | ❌ | Dev/test, learning, AZ-204 labs |
| **Standard** | 100 GB | Medium | ❌ | Production, small teams |
| **Premium** | 500 GB | High | ✅ | Enterprise, global distribution |

For all AZ-204 labs, use **Basic** — it is the cheapest and has everything you need.

### Key Operations

**Create an ACR:**

```bash
az acr create \
  --name az204acr$RANDOM \          # ← Registry name (must be globally unique, lowercase, alphanumeric only)
  --resource-group az204-containers-rg \
  --sku Basic \                      # ← Basic tier for labs
  --admin-enabled true               # ← Enable simple username/password auth (for labs only)
```

| Flag | What It Does |
|---|---|
| `--name` | Registry name. Becomes part of the URL: `az204acrXXXXX.azurecr.io`. Must be globally unique. Only lowercase letters and numbers allowed. |
| `--sku` | Pricing tier. `Basic` for labs. |
| `--admin-enabled` | Enables a simple username/password for pulling images. For labs only — production should use Managed Identity. |

### ACR Tasks: Build Images in the Cloud ☁️

Here is the most important ACR concept for the exam:

```bash
az acr build \
  --registry az204acrXXXXX \        # ← Your ACR name
  --image myapp:v1.0 \              # ← Image name and tag
  .                                  # ← Build context (current directory, where Dockerfile is)
```

This command sends your code to Azure, builds the Docker image **in the cloud**, and stores it directly in your ACR — all without Docker installed on your local machine.

> 🚨 **EXAM ALERT**
>
> `az acr build` builds container images **in the cloud** using ACR Tasks. You do **NOT** need Docker installed locally to build images this way. If an exam question asks "How can a developer build a container image without installing Docker locally?" → the answer is `az acr build` (ACR Tasks).

### Authentication Methods

| Method | How It Works | When to Use |
|---|---|---|
| **Admin account** | Username + password stored in ACR | Labs and quick testing only |
| **Service Principal** | App registration with RBAC role | CI/CD pipelines |
| **Managed Identity** | Automatic credential from Azure service | ✅ Production — recommended |
| **az acr login** | Uses your Azure CLI credentials | Local development |

> 🚨 **EXAM ALERT**
>
> For **production** deployments pulling from ACR, use **Managed Identity** — not admin credentials. Admin credentials are a shared password (security risk). Managed Identity is automatic, per-resource, and requires no stored secrets.

---

## Section 3: Azure Container Instances (ACI) 🔴 HIGH EXAM WEIGHT

### The Problem

You have a container image in ACR. You want to run it in Azure **right now** — quickly, with no infrastructure setup, no cluster management, no orchestration platform. You just want: "Run this container."

### The Solution: Azure Container Instances

**Azure Container Instances (ACI)** is the fastest and simplest way to run a container in Azure. No VMs to manage, no orchestration to configure. You give Azure a container image, and it runs it.

Think of ACI as a **vending machine for containers** — insert your image, get a running container immediately.

> 💡 **KEY CONCEPT**
>
> **ACI** = run a single container (or small group) in Azure with zero infrastructure management. Best for quick tasks, batch jobs, and simple workloads. NOT for large-scale production microservices.

### Creating a Container Instance

```bash
az container create \
  --name my-container \              # ← Name for this container instance
  --resource-group az204-containers-rg \
  --image az204acrXXXXX.azurecr.io/myapp:v1.0 \  # ← Full image reference from ACR
  --cpu 1 \                          # ← Number of CPU cores allocated
  --memory 1.5 \                     # ← GB of memory allocated
  --registry-login-server az204acrXXXXX.azurecr.io \  # ← ACR login server
  --registry-username <admin-user> \  # ← ACR admin username
  --registry-password <admin-pass> \  # ← ACR admin password
  --dns-name-label my-container-app \ # ← Public DNS label
  --ports 80                         # ← Port(s) to expose
```

### Restart Policies

When the process inside a container finishes or crashes, what should Azure do? The **restart policy** controls this:

| Policy | Behaviour | Use Case |
|---|---|---|
| **Always** | Restarts the container whenever it stops, regardless of exit code | Web servers, APIs — things that should always be running |
| **OnFailure** | Restarts only if the container exits with a non-zero (error) exit code | Batch jobs that should retry on failure |
| **Never** | Never restarts — runs once and stops | One-time data processing, migrations, scripts |

```bash
az container create \
  --restart-policy Never \            # ← Run once, then stop
  # ... other flags
```

> 🚨 **EXAM ALERT**
>
> When the exam describes a **one-time task that must run once and stop**, the restart policy is `Never`. When it describes a **batch job that should retry on failure**, use `OnFailure`. For **always-running services**, use `Always`.

### Container Groups: The Sidecar Pattern

A **Container Group** is multiple containers that share the same host, network, storage, and lifecycle. They are deployed together and run together.

Think of it like a **motorcycle with a sidecar** — the main motorcycle (your application container) has a sidecar (a helper container) attached. They travel together, share the same road (network), but each has its own job.

```
┌─────────────────────────────────────────┐
│           Container Group               │
│                                         │
│  ┌──────────────┐  ┌──────────────────┐ │
│  │  App Container│  │ Logging Sidecar  │ │
│  │  (your code)  │  │ (collects logs)  │ │
│  │  Port 80      │  │ Port 8080        │ │
│  └──────────────┘  └──────────────────┘ │
│                                         │
│  Shared: Network (localhost), Storage   │
│  Public IP: assigned to the group       │
└─────────────────────────────────────────┘
```

Common sidecar examples:
- **Logging sidecar:** Collects and forwards logs from the main container
- **Proxy sidecar:** Handles SSL termination or request routing
- **Monitoring sidecar:** Collects performance metrics

> 🚨 **EXAM ALERT**
>
> **ACI is NOT for production microservices at scale.** If an exam question describes a company running 10+ microservices that need auto-scaling, service discovery, and traffic management → the answer is **ACA** (or AKS), not ACI. ACI is for **simple, quick, small-scale** container workloads.

---

## Section 4: Azure Container Apps (ACA) 🔴 HIGH EXAM WEIGHT

### The Problem

ACI is great for running one or two containers quickly. But what if you have a real application with:
- Multiple microservices that need to communicate
- Auto-scaling based on HTTP traffic or queue depth
- Blue/green deployments with traffic splitting
- Need for Kubernetes-level features without managing Kubernetes

### The Solution: Azure Container Apps

**Azure Container Apps (ACA)** gives you the power of Kubernetes without the complexity of managing Kubernetes. Think of it as a **luxury apartment building** — you get all the amenities (scaling, networking, service discovery, traffic management) without having to maintain the plumbing, wiring, and elevators yourself.

> 💡 **KEY CONCEPT**
>
> **ACA** = serverless containers on Kubernetes. You get Kubernetes features (scaling, revisions, traffic splitting, service discovery) without ever touching kubectl, nodes, pods, or cluster configuration. Azure manages all of that for you.

### Key ACA Concepts

| Concept | What It Is | Analogy |
|---|---|---|
| **Environment** | A shared boundary for a group of container apps. Provides a common VNet and Log Analytics workspace. | The apartment building — all apps inside share the same building infrastructure |
| **Container App** | A single application deployed within an environment. | One apartment in the building |
| **Revision** | An immutable snapshot of a container app version. When you update your app, a new revision is created. | A version number — v1.0, v1.1, v2.0. Each is a complete snapshot. |
| **Ingress** | The HTTP traffic configuration — controls whether the app is accessible externally or only internally. | The front door — is it open to the public, or only to other tenants? |
| **KEDA** | Kubernetes Event-Driven Autoscaling. Scales based on HTTP requests, queue messages, CPU, custom metrics. | An automatic thermostat — adjusts capacity based on demand |
| **Dapr** | Distributed Application Runtime. Provides building blocks for microservice communication. | A shared phone system for all apartments — service-to-service calls made easy |

### Traffic Splitting Between Revisions

One of ACA's most powerful features is **traffic splitting** — sending a percentage of traffic to different revisions of your app:

```
                    ┌──────────────────┐
                    │   Incoming       │
                    │   HTTP Traffic   │
                    └────────┬─────────┘
                             │
                    ┌────────┴─────────┐
                    │  ACA Ingress     │
                    │  Traffic Split   │
                    └──┬──────────┬────┘
                       │          │
                  80%  │          │  20%
                       ▼          ▼
              ┌────────────┐ ┌────────────┐
              │ Revision 1 │ │ Revision 2 │
              │ (v1.0)     │ │ (v2.0)     │
              │ Current    │ │ New/Test   │
              └────────────┘ └────────────┘
```

This enables **blue/green deployments** and **canary releases**:
- Deploy a new version as a new revision
- Send 10% of traffic to the new revision
- Monitor for errors
- Gradually increase to 100% if everything looks good
- Roll back instantly by shifting traffic back to the old revision

```bash
# Update traffic split: 80% to revision1, 20% to revision2
az containerapp ingress traffic set \
  --name my-app \
  --resource-group az204-containers-rg \
  --revision-weight revision1=80 revision2=20
```

### Scaling in ACA

ACA can **scale to zero** — meaning if no requests are coming in, Azure removes all instances and you pay nothing. When requests arrive, instances are automatically created.

Scaling triggers (via KEDA):
- **HTTP traffic:** More requests → more instances
- **Queue depth:** More messages in a queue → more instances to process them
- **CPU/Memory utilization:** High resource usage → more instances
- **Custom:** Any KEDA-supported scaler (Cosmos DB change feed, Kafka, etc.)

```bash
# Set scaling rules
az containerapp update \
  --name my-app \
  --resource-group az204-containers-rg \
  --min-replicas 0 \                 # ← Scale to zero when idle
  --max-replicas 10                  # ← Maximum 10 instances
```

> 🚨 **EXAM ALERT**
>
> **ACA hides Kubernetes completely.** You do NOT manage nodes, pods, deployments, services, or any Kubernetes objects. If an exam question mentions "without managing Kubernetes infrastructure" or "serverless containers" → the answer is ACA. If the question mentions needing full Kubernetes control (custom operators, specific Kubernetes versions) → the answer is AKS (Azure Kubernetes Service), which is NOT covered in AZ-204.

> ✅ **CHECKPOINT 2**
>
> 1. What is the difference between ACI and ACA?
> 2. What is a revision in ACA?
> 3. Can ACA scale to zero?
>
> **Answers:** 1) ACI = simple, quick, small-scale container runs. ACA = full microservice platform with scaling, traffic splitting, and Kubernetes features. 2) A revision is an immutable snapshot of a container app version — like a deployment slot. 3) Yes — ACA can scale to zero replicas when there is no traffic, meaning you pay nothing when idle.

---

## Section 5: The Decision Table — Which Service When? 🔴 HIGH EXAM WEIGHT

This is the table you need to memorise for the exam. Every container question essentially asks: "Which service is the right choice for this scenario?"

| Scenario | Use This Service | Why |
|---|---|---|
| Store and manage container images privately | **ACR** | Azure's private Docker registry, integrated with other Azure services |
| Build a container image without local Docker | **ACR** (ACR Tasks / `az acr build`) | Cloud-based build — no local Docker installation needed |
| Run a container quickly for a one-time task | **ACI** | Fastest way to run a container. No setup. Restart policy: `Never` |
| Run a batch job that retries on failure | **ACI** | Restart policy: `OnFailure` handles retries automatically |
| Simple container with a sidecar (logging, proxy) | **ACI** (Container Groups) | Multiple containers sharing network and storage |
| Production microservices with auto-scaling | **ACA** | Serverless containers with KEDA scaling, traffic splitting, revisions |
| Serverless containers (scale to zero) | **ACA** | Only ACA supports scaling to zero replicas |
| Kubernetes without managing Kubernetes | **ACA** | Kubernetes power, zero Kubernetes management |
| Traffic splitting between app versions | **ACA** (Revisions) | Percentage-based traffic routing between revisions |
| Full Kubernetes control (custom operators) | **AKS** (not AZ-204) | AKS gives you full Kubernetes — but it is NOT on the AZ-204 exam |

> 🚨 **EXAM ALERT**
>
> The exam loves to test these distinctions:
> - **"Quick, one-time task in a container"** → ACI
> - **"Production microservices with auto-scaling"** → ACA
> - **"Build image without Docker installed"** → ACR Tasks (`az acr build`)
> - **"Serverless containers, scale to zero"** → ACA
> - **"Traffic splitting between versions"** → ACA revisions
>
> If you get these five right, you will handle most container questions correctly.

### Three Services Working Together

In a real AZ-204 scenario, all three services work together:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Developer's Machine                                        │
│  ┌──────────────────┐                                       │
│  │ Write Dockerfile  │                                      │
│  │ Write app code    │                                      │
│  └────────┬─────────┘                                       │
│           │                                                 │
│           │  az acr build                                   │
│           ▼                                                 │
│  ┌──────────────────┐                                       │
│  │ ACR (Registry)    │  ← Stores the built image            │
│  │ myapp:v1.0        │                                      │
│  └────────┬─────────┘                                       │
│           │                                                 │
│           │  Pull image                                     │
│           ├──────────────────┐                               │
│           ▼                  ▼                               │
│  ┌──────────────┐   ┌──────────────────┐                    │
│  │ ACI           │   │ ACA              │                   │
│  │ Quick test    │   │ Production       │                   │
│  │ One-off task  │   │ Microservices    │                   │
│  └──────────────┘   │ Auto-scaling     │                   │
│                      │ Traffic splitting│                   │
│                      └──────────────────┘                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔗 Module Connections

| Connected Module | How It Connects |
|---|---|
| **M01 — App Service** | App Service can also run containers (Docker deployment method). Compare with ACI/ACA for container hosting. |
| **M02 — Azure Functions** | Functions can run in containers on ACA (Container Apps hosting plan). Functions on ACA = serverless containers. |
| **M04 — Blob Storage** | ACR stores images internally using blob storage. ACI/ACA can mount Azure Files or Blob Storage volumes. |
| **M07 — Secure Solutions** | ACR authentication with Managed Identity. ACA uses Managed Identity for secure access to other Azure services. |
| **F06 — C# Basics** | Lab code uses .NET SDK for container-related operations. Dockerfiles for .NET apps follow specific patterns. |

---

## What We Covered ✅

- [ ] What containers are and how they differ from VMs (shipping container analogy)
- [ ] Docker concepts: image, container, Dockerfile, registry, tag
- [ ] How to read a Dockerfile and understand each instruction
- [ ] Azure Container Registry (ACR): private image storage in Azure
- [ ] ACR tiers: Basic (labs) / Standard / Premium
- [ ] ACR Tasks: `az acr build` to build images in the cloud without local Docker
- [ ] ACR authentication: admin account (labs) vs Managed Identity (production)
- [ ] Azure Container Instances (ACI): fastest way to run a container
- [ ] ACI restart policies: Always / OnFailure / Never — when to use each
- [ ] ACI Container Groups: sidecar pattern for multi-container workloads
- [ ] Why ACI is NOT for production microservices at scale
- [ ] Azure Container Apps (ACA): serverless containers on Kubernetes
- [ ] ACA concepts: Environment, Container App, Revision, Ingress, KEDA, Dapr
- [ ] ACA traffic splitting between revisions (blue/green, canary)
- [ ] ACA scaling: scale to zero, HTTP/queue/CPU triggers
- [ ] The decision table: when to use ACR vs ACI vs ACA
- [ ] How all three services work together in a real workflow

---

## Self-Check Questions

1. **A developer needs to build a Docker image but does not have Docker installed on their machine. What Azure service and command should they use?**

2. **A company has a batch processing job that runs once per night, processes data, and then should stop. The job should retry if it fails. Which Azure container service and restart policy should they use?**

3. **A company is deploying 8 microservices that need auto-scaling, traffic splitting between versions, and the ability to scale to zero when idle. They do not want to manage Kubernetes. Which service should they use?**

<details>
<summary>Answers</summary>

1. **ACR Tasks** — use `az acr build --registry myregistry --image myapp:v1.0 .` to build the image in the cloud without local Docker.

2. **ACI with restart policy `OnFailure`** — ACI runs the container quickly with no infrastructure management. `OnFailure` restarts the job if it fails but stops when it succeeds.

3. **Azure Container Apps (ACA)** — provides serverless containers with auto-scaling (KEDA), traffic splitting between revisions, scale-to-zero capability, and hides all Kubernetes complexity.

</details>
