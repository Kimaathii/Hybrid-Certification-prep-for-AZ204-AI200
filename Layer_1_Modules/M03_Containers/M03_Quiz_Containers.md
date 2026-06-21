# M03 — Module Quiz: Containerised Solutions (ACR, ACI, ACA)

---

| | |
|---|---|
| **Module** | M03 — Containerised Solutions |
| **Questions** | 15 (5 Foundation + 5 Applied + 5 Exam-Level) |
| **Time Limit** | 25 minutes |
| **Passing Score** | 11/15 (73%) |

---

## FOUNDATION QUESTIONS (1–5)

### Question 1

**What is the difference between a Docker image and a Docker container?**

- A) An image is a running application; a container is a stored template
- B) An image is a read-only template; a container is a running instance of that image ✅
- C) An image runs on Linux; a container runs on Windows
- D) An image contains only code; a container contains the operating system

**Correct Answer: B**

**Explanation:** A Docker **image** is a read-only blueprint that contains the application code, dependencies, and configuration. A **container** is a running instance created from an image. You can create many containers from one image, just like making many dishes from one recipe.

- **A is wrong:** It has the definitions reversed.
- **C is wrong:** Both images and containers can run on Linux or Windows.
- **D is wrong:** An image contains code AND dependencies, runtime, and configuration — not just code.

---

### Question 2

**What does a Dockerfile define?**

- A) The runtime configuration for an Azure virtual machine
- B) The network security rules for a container
- C) Step-by-step instructions for building a container image ✅
- D) The scaling rules for Azure Container Apps

**Correct Answer: C**

**Explanation:** A Dockerfile is a text file containing step-by-step instructions (like `FROM`, `COPY`, `RUN`, `CMD`) that Docker uses to build a container image. Each instruction creates a layer in the image.

---

### Question 3

**Which Azure service is a private container image registry?**

- A) Azure Container Instances (ACI)
- B) Azure Container Apps (ACA)
- C) Azure Container Registry (ACR) ✅
- D) Azure Kubernetes Service (AKS)

**Correct Answer: C**

**Explanation:** **ACR** is Azure's private container image registry — it stores and manages container images. ACI runs containers. ACA is a serverless container platform. AKS is managed Kubernetes (not on AZ-204).

---

### Question 4

**What are the three ACR tiers in order of increasing capability?**

- A) Free, Standard, Premium
- B) Basic, Standard, Premium ✅
- C) Developer, Professional, Enterprise
- D) Consumption, Dedicated, Isolated

**Correct Answer: B**

**Explanation:** ACR has three tiers: **Basic** (10 GB, dev/test), **Standard** (100 GB, production), and **Premium** (500 GB, enterprise with geo-replication). For AZ-204 labs, always use Basic.

---

### Question 5

**Which of the following is NOT a valid ACI restart policy?**

- A) Always
- B) OnFailure
- C) Never
- D) OnSuccess ✅

**Correct Answer: D**

**Explanation:** ACI has three restart policies: **Always** (always restart), **OnFailure** (restart only on error exit code), and **Never** (run once and stop). There is no `OnSuccess` policy.

---

## APPLIED QUESTIONS (6–10)

### Question 6

**A developer needs to build a container image but cannot install Docker on their local machine. Which Azure command should they use?**

- A) `az container build --image myapp:v1`
- B) `az acr build --registry myacr --image myapp:v1 .` ✅
- C) `docker build -t myapp:v1 .`
- D) `az containerapp build --image myapp:v1`

**Correct Answer: B**

**Explanation:** `az acr build` uses **ACR Tasks** to build container images in the cloud. The source code and Dockerfile are uploaded to Azure, where the image is built and stored in ACR — no local Docker installation required.

- **A is wrong:** `az container build` is not a valid command.
- **C is wrong:** `docker build` requires Docker installed locally — the exact problem we're solving.
- **D is wrong:** `az containerapp build` is not the standard approach for building images.

---

### Question 7

**A company has a data processing job that runs for 30 minutes, processes a batch of records, and should retry automatically if it fails. Which service and restart policy should they use?**

- A) ACI with restart policy `Always`
- B) ACI with restart policy `OnFailure` ✅
- C) ACI with restart policy `Never`
- D) ACA with scale-to-zero

**Correct Answer: B**

**Explanation:** ACI with `OnFailure` is ideal for batch jobs: it runs the container, and if it fails (non-zero exit code), it retries automatically. When the job succeeds (exit code 0), the container stops. This saves costs compared to `Always` (which would restart even after success).

- **A is wrong:** `Always` would restart the container even after successful completion, wasting resources.
- **C is wrong:** `Never` would not retry on failure.
- **D is wrong:** While ACA could work, ACI is simpler and more appropriate for a single batch job.

---

### Question 8

**In an ACI Container Group, how do containers communicate with each other?**

- A) Through Azure Service Bus messages
- B) Via the public internet using their DNS names
- C) Through localhost, as they share the same network namespace ✅
- D) Using Azure Event Grid events

**Correct Answer: C**

**Explanation:** Containers in a Container Group share the same network namespace, which means they can communicate via **localhost**. If container A listens on port 80 and container B needs to reach it, B simply calls `localhost:80`. They also share storage volumes and have the same lifecycle.

---

### Question 9

**A company deploys version 2.0 of their API to ACA and wants to send only 10% of traffic to the new version while keeping 90% on version 1.0. What ACA feature enables this?**

- A) Deployment slots
- B) Revision traffic splitting ✅
- C) Blue/green containers
- D) Load balancer rules

**Correct Answer: B**

**Explanation:** ACA creates a new **revision** each time you update a container app. You can then use **traffic splitting** to route a percentage of traffic to each revision — for example, 90% to revision 1 (v1.0) and 10% to revision 2 (v2.0). This enables canary releases.

- **A is wrong:** Deployment slots are an App Service feature (M01), not ACA.
- **C is wrong:** "Blue/green containers" is not an ACA feature name — the feature is called revision traffic splitting.
- **D is wrong:** You don't configure load balancer rules directly — ACA handles traffic splitting via revision weights.

---

### Question 10

**Which ACA component is responsible for event-driven autoscaling based on metrics like HTTP traffic and queue depth?**

- A) Dapr
- B) Ingress controller
- C) KEDA ✅
- D) Container Environment

**Correct Answer: C**

**Explanation:** **KEDA** (Kubernetes Event-Driven Autoscaling) is the component in ACA that handles autoscaling based on various triggers — HTTP concurrent requests, Azure Queue messages, CPU usage, and many others. It enables scale-to-zero.

- **A is wrong:** Dapr is for microservice communication (service-to-service calls, state management, pub/sub) — not scaling.
- **B is wrong:** The ingress controller handles HTTP routing, not scaling decisions.
- **D is wrong:** The environment is the shared infrastructure boundary, not a scaling component.

---

## EXAM-LEVEL QUESTIONS (11–15)

### Question 11

**A company needs to deploy a containerised application. The application processes messages from an Azure Queue. When there are no messages, the application should consume zero resources. When 1,000 messages arrive, the application should automatically scale up. The team has no Kubernetes experience. Which solution should they use?**

- A) Azure Container Instances with restart policy `Always`
- B) Azure Kubernetes Service with Horizontal Pod Autoscaler
- C) Azure Container Apps with KEDA queue-based scaling ✅
- D) Azure App Service with containers and autoscale rules

**Correct Answer: C**

**Explanation:** ACA with KEDA provides: (1) scale-to-zero when no messages exist, (2) automatic scaling based on queue depth, (3) no Kubernetes management needed. This perfectly matches all requirements.

- **A is wrong:** ACI does not support auto-scaling based on queue depth.
- **B is wrong:** AKS requires Kubernetes experience — the team explicitly doesn't have this.
- **D is wrong:** App Service cannot scale to zero (even the lowest tier uses resources).

---

### Question 12 — SELECT TWO

**Which TWO of the following statements about Azure Container Registry are correct? (SELECT TWO)**

- A) ACR can build images in the cloud without Docker installed locally ✅
- B) ACR can run containers directly without ACI or ACA
- C) ACR Premium tier supports geo-replication ✅
- D) ACR requires Docker Desktop on the developer's machine
- E) ACR only supports Linux container images

**Correct Answers: A, C**

**Explanation:**
- **A is correct:** ACR Tasks (`az acr build`) builds images in the cloud — no local Docker needed.
- **C is correct:** ACR Premium tier supports geo-replication, distributing images across multiple Azure regions.
- **B is wrong:** ACR is a registry (storage). It cannot run containers — you need ACI, ACA, or App Service to run them.
- **D is wrong:** The entire point of ACR Tasks is that Docker Desktop is NOT required.
- **E is wrong:** ACR supports both Linux and Windows container images.

---

### Question 13

**A developer deploys a container to ACI. The container starts, processes data for 10 minutes, exits with code 0, and then the container restarts and processes the same data again. The developer wants the container to run ONCE and then stop. What should the developer change?**

- A) Set the CPU limit to 0 after execution
- B) Change the restart policy from `Always` to `Never` ✅
- C) Add a STOP command to the Dockerfile
- D) Delete the container group after execution

**Correct Answer: B**

**Explanation:** The container is restarting because the restart policy is `Always` (the default). Changing it to `Never` ensures the container runs once and stops when the process exits, regardless of exit code.

- **A is wrong:** You cannot set CPU to 0 on a running container.
- **C is wrong:** There is no STOP command in Docker. The container exits when its process completes.
- **D is wrong:** While deleting would stop it, the correct solution is using the right restart policy.

---

### Question 14 — SELECT TWO

**A company is choosing between ACI and ACA for a new project. Which TWO capabilities are available in ACA but NOT in ACI? (SELECT TWO)**

- A) Running containers from ACR images
- B) Traffic splitting between application versions ✅
- C) Scaling to zero replicas when idle ✅
- D) Setting environment variables
- E) Exposing containers on a public IP

**Correct Answers: B, C**

**Explanation:**
- **B is correct:** ACA supports traffic splitting between revisions (e.g., 80/20). ACI has no revision or traffic splitting concept.
- **C is correct:** ACA can scale to zero replicas when idle. ACI runs a container until it stops — it doesn't have auto-scaling or scale-to-zero.
- **A is wrong:** Both ACI and ACA can pull images from ACR.
- **D is wrong:** Both ACI and ACA support environment variables.
- **E is wrong:** Both ACI and ACA can expose containers publicly.

---

### Question 15

**A company has the following architecture: developers push code to GitHub, which triggers a cloud-based container image build without Docker Desktop. The image is stored privately. Then the image is deployed to a serverless platform that supports auto-scaling and traffic splitting. Which Azure services should be used for the BUILD/STORE and RUN phases?**

- A) BUILD/STORE: Docker Hub · RUN: Azure Virtual Machines
- B) BUILD/STORE: ACR with ACR Tasks · RUN: Azure Container Apps ✅
- C) BUILD/STORE: Azure DevOps · RUN: Azure Container Instances
- D) BUILD/STORE: ACR with ACR Tasks · RUN: Azure Kubernetes Service

**Correct Answer: B**

**Explanation:** The scenario requires: (1) cloud-based build without Docker Desktop → **ACR Tasks**, (2) private image storage → **ACR**, (3) serverless with auto-scaling and traffic splitting → **ACA**. Together, ACR + ACA is the complete solution.

- **A is wrong:** Docker Hub is public (not private), and VMs are not serverless with auto-scaling.
- **C is wrong:** ACI does not support auto-scaling or traffic splitting.
- **D is wrong:** AKS is managed Kubernetes, not "serverless." AKS requires infrastructure management.

---

## Score Interpretation

| Score | Assessment | Recommendation |
|---|---|---|
| 13–15 | 🟢 Excellent — Exam ready for container topics | Move to M04 |
| 11–12 | 🟡 Good — Review the concepts you missed | Re-read specific sections, then proceed |
| 8–10 | 🟠 Needs work — Review the Concept document | Re-read M03 Concept, redo scenarios, retake quiz |
| Below 8 | 🔴 Not ready — Start from the beginning | Re-read M03 Concept thoroughly, then retake |
