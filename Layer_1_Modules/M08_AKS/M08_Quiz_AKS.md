# Module 8: Azure Kubernetes Service (AKS) - Quiz

**Instructions:** This quiz contains 15 questions divided into Foundation, Applied, and Exam-level difficulty. Read the scenario carefully before selecting your answer. 

---

## Part 1: Foundation (Concepts & Terminology)

**Question 1:**
You are explaining the basic architecture of Kubernetes to a new team member using the "orchestra" analogy. They ask you what represents the individual musician playing an instrument.
Which Kubernetes component is the smallest deployable unit that wraps around a container?
A) Node
B) Deployment
C) Pod
D) Service

**Answer: C**
- **A is incorrect:** A Node is the virtual machine (the stage), not the container wrapper.
- **B is incorrect:** A Deployment is the instruction set (the sheet music) that manages the Pods.
- **C is correct:** A Pod is the smallest deployable unit in Kubernetes and typically contains one container.
- **D is incorrect:** A Service provides the network IP address (the sound system), not the container itself.

**Question 2:**
Your company is migrating from self-hosted Kubernetes to Azure Kubernetes Service (AKS). The CFO asks what infrastructure components your team will no longer need to manage or pay for.
In standard AKS, which component is fully managed by Microsoft and provided for free?
A) The Worker Nodes
B) The Control Plane
C) The Azure Load Balancer
D) The Container Registry

**Answer: B**
- **A is incorrect:** You are responsible for managing and paying for the Worker Nodes (the VMs).
- **B is correct:** Microsoft fully manages the Control Plane (API server, scheduler) for free in AKS.
- **C is incorrect:** If you provision an Azure Load Balancer, you pay for the public IP and data processing.
- **D is incorrect:** Azure Container Registry is a separate service that incurs its own storage costs.

**Question 3:**
You need to deploy a new web application to AKS. Instead of running a sequence of terminal commands to start containers one by one, you write a single file that states: "I want 3 replicas of the frontend running." 
What language/format is primarily used to write these declarative Kubernetes manifests?
A) XML
B) JSON
C) YAML
D) PowerShell

**Answer: C**
- **A is incorrect:** XML is not the standard format for Kubernetes manifests.
- **B is incorrect:** While Kubernetes APIs accept JSON, humans write manifests in YAML.
- **C is correct:** YAML (YAML Ain't Markup Language) is the declarative, human-readable standard for K8s manifests.
- **D is incorrect:** PowerShell is a scripting language, whereas Kubernetes requires declarative manifests.

**Question 4:**
You are deploying a database to your AKS cluster. This database must NOT be accessible from the internet. It should only be reachable by other applications running inside the exact same cluster.
Which Kubernetes Service type should you use?
A) LoadBalancer
B) Ingress
C) NodePort
D) ClusterIP

**Answer: D**
- **A is incorrect:** LoadBalancer exposes the service to the internet via an external IP.
- **B is incorrect:** Ingress provides external HTTP/HTTPS routing.
- **C is incorrect:** NodePort opens a specific port on every node's IP, which is not as secure or internal as ClusterIP.
- **D is correct:** ClusterIP is the default service type and exposes the service only on an internal IP inside the cluster.

**Question 5:**
Your data science team wants to run a heavy machine learning inference model on AKS. They insist they need GPU hardware.
What is the recommended way to provision GPUs in AKS?
A) Upgrade the AKS Control Plane to the GPU tier.
B) Create a dedicated GPU Node Pool.
C) Install a GPU container image inside a standard Pod.
D) Attach an external Azure GPU disk to a standard Node.

**Answer: B**
- **A is incorrect:** The Control Plane does not run your workloads and cannot be upgraded to a GPU tier.
- **B is correct:** You create a separate Node Pool consisting of N-Series (GPU) virtual machines specifically for heavy workloads.
- **C is incorrect:** Software (the container image) cannot magically create GPU hardware if the underlying Node doesn't have it.
- **D is incorrect:** GPUs are compute processors (VM types), not attachable storage disks.

---

## Part 2: Applied (Configuration & Architecture)

**Question 6:**
You have written a deployment manifest for a React.js frontend. The pods are running successfully. Now, you need to allow customers on the internet to reach the frontend via a single Public IP address.
Which of the following must you create?
A) A Service manifest with `type: ClusterIP`
B) A Service manifest with `type: LoadBalancer`
C) A Deployment manifest with `replicas: 1`
D) A Persistent Volume Claim

**Answer: B**
- **A is incorrect:** ClusterIP keeps the application private to the cluster.
- **B is correct:** `type: LoadBalancer` tells Azure to spin up a public-facing Azure Load Balancer and assign a Public IP.
- **C is incorrect:** A Deployment only ensures the pods are running; it does not provide network ingress.
- **D is incorrect:** Persistent Volume Claims are for storage, not networking.

**Question 7:**
You are migrating three different APIs to AKS: a User API, a Products API, and an Orders API. To save money, your boss wants all three APIs to be accessible via the exact same public IP address, routing traffic based on the URL path (e.g., `/users`, `/products`).
Which Kubernetes feature is required to achieve this path-based routing?
A) Ingress Controller
B) Horizontal Pod Autoscaler
C) Azure Files Volume
D) ClusterIP

**Answer: A**
- **A is correct:** An Ingress Controller sits at the edge of the cluster and acts as a smart router, routing traffic to different backend services based on the URL path.
- **B is incorrect:** HPA is for scaling pod counts, not routing traffic.
- **C is incorrect:** Azure Files is for storage.
- **D is incorrect:** ClusterIP is for internal networking, not public path-based routing.

**Question 8:**
You are running a stateful application in AKS that requires reading and writing data to a file directory. The application is highly available, meaning 5 replicas (Pods) of the app are running across 3 different Nodes. All 5 Pods must be able to read and write to the exact same storage directory simultaneously.
Which storage solution must you implement?
A) Local Pod Storage
B) Azure Disks (ReadWriteOnce)
C) Azure Files (ReadWriteMany)
D) Azure Blob Storage

**Answer: C**
- **A is incorrect:** Local storage is wiped when a pod dies and cannot be shared across pods.
- **B is incorrect:** Azure Disks can only be attached to ONE node at a time (ReadWriteOnce).
- **C is correct:** Azure Files creates an SMB file share that supports ReadWriteMany, allowing pods across multiple nodes to access it simultaneously.
- **D is incorrect:** While Blob is storage, Kubernetes Persistent Volumes natively map block/file storage via Azure Disks or Azure Files.

**Question 9:**
Your e-commerce API is experiencing random spikes in traffic. When a spike hits, CPU utilization on the Pods reaches 100% and requests fail. You want Kubernetes to automatically spin up more Pods when CPU usage goes above 80%, and terminate them when traffic subsides.
Which resource should you configure?
A) Cluster Autoscaler
B) Horizontal Pod Autoscaler (HPA)
C) Azure Load Balancer
D) KEDA

**Answer: B**
- **A is incorrect:** Cluster Autoscaler adds new VMs (Nodes) to the cluster, not Pods.
- **B is correct:** HPA automatically scales the number of Pods in a Deployment based on observed CPU/Memory utilization.
- **C is incorrect:** A Load Balancer distributes traffic but does not create new pods.
- **D is incorrect:** KEDA is for event-driven scaling (like queue length), while HPA handles standard CPU/Memory scaling perfectly.

**Question 10: (Select all that apply)**
You have created a new Node Pool using expensive GPU virtual machines for an AI workload. You notice that standard, lightweight web application Pods are randomly being scheduled onto these expensive GPU nodes.
Which two Kubernetes features must you use together to ensure ONLY the AI Pods run on the GPU nodes?
A) Taints (applied to the GPU Node)
B) ClusterIP (applied to the Web Pods)
C) Tolerations (applied to the AI Pods)
D) Ingress (applied to the GPU Node)

**Answers: A and C**
- **A is correct:** You apply a Taint to the Node. This acts like a forcefield, repelling all pods.
- **B is incorrect:** ClusterIP is a networking feature.
- **C is correct:** You add a Toleration to the AI Pod's deployment manifest. This acts as a VIP pass, allowing it to bypass the Node's taint.
- **D is incorrect:** Ingress is for HTTP routing.

---

## Part 3: Exam-Level Scenarios

**Question 11:**
You are developing an AI inference service on AKS. Users upload images, which generate messages in an Azure Service Bus queue. An AI Pod pulls messages from the queue and processes them. You notice that when 10,000 messages hit the queue, the pod's CPU stays at a steady 60%, so the Horizontal Pod Autoscaler (HPA) refuses to scale up, causing a massive processing delay.
What is the best solution to scale the AI pods based on the backlog of queue messages?
A) Lower the HPA CPU threshold to 10%.
B) Install KEDA and configure an Azure Service Bus trigger.
C) Write a custom PowerShell script that runs inside the container to restart it.
D) Change the Service type from ClusterIP to LoadBalancer.

**Answer: B**
- **A is incorrect:** Lowering the CPU threshold won't help if the pod never naturally spikes CPU while waiting for queue operations.
- **B is correct:** KEDA (Kubernetes Event-driven Autoscaling) allows you to autoscale pods based on external events, such as the length of an Azure Service Bus queue.
- **C is incorrect:** Custom scripts inside containers are an anti-pattern for orchestration.
- **D is incorrect:** Networking changes do not affect autoscaling logic.

**Question 12:**
Your company processes financial reports using an AI model on AKS GPU nodes. This process only happens for exactly one hour at the end of every month. For the other 29 days, the queue is completely empty. Management wants to ensure they are billed $0 for GPU compute during those 29 days.
Which combination of technologies allows the pods to scale completely down to zero, triggering the Cluster Autoscaler to shut down the GPU VMs?
A) Standard HPA and Azure Disks
B) KEDA and Cluster Autoscaler
C) Ingress Controller and Node Taints
D) LoadBalancer and HPA

**Answer: B**
- **A is incorrect:** Standard HPA can only scale down to a minimum of 1 pod; it cannot scale to zero.
- **B is correct:** KEDA can scale deployments to exactly zero when the queue is empty. Once zero pods are running, the Cluster Autoscaler realizes the GPU nodes are empty and shuts the VMs down, stopping the billing.
- **C is incorrect:** Ingress and Taints do not control autoscaling.
- **D is incorrect:** HPA cannot scale to zero.

**Question 13:**
You are deploying a mission-critical web application to AKS. The exam question states: "You must ensure that during an update to the application's container image, the application remains highly available and experiences zero downtime."
What native Kubernetes Deployment behavior guarantees this by default?
A) Rolling Updates
B) Node Eviction
C) Taints and Tolerations
D) Persistent Volume Attachment

**Answer: A**
- **A is correct:** Deployments use a "Rolling Update" strategy by default. They spin up a new version of the Pod, wait for it to be healthy, and then terminate an old version of the Pod, ensuring zero downtime.
- **B is incorrect:** Node eviction is what happens when a node fails or runs out of resources.
- **C is incorrect:** Taints and Tolerations control pod placement, not update strategies.
- **D is incorrect:** Volume attachment handles storage, not zero-downtime application updates.

**Question 14:**
You are reviewing the Azure monthly invoice. You see a charge for "Virtual Machine Scale Sets - Standard_DS2_v2" associated with your AKS cluster. Your developer claims they only use the "Free Tier" of AKS. 
Why are you being charged?
A) The Free Tier only covers 10 hours of usage per month.
B) The developer accidentally upgraded the Control Plane to the Paid tier.
C) The Control Plane is free, but you must pay for the Worker Nodes that run your Pods.
D) You are being charged for the YAML manifest storage.

**Answer: C**
- **A is incorrect:** There is no hourly time limit on the free control plane.
- **B is incorrect:** While a paid Uptime SLA exists, the VM charges described are for the compute nodes.
- **C is correct:** In AKS, the control plane is managed by Microsoft for free, but you are billed for the underlying Virtual Machines (Worker Nodes) that actually execute your containers.
- **D is incorrect:** Manifests are stored in the control plane's Etcd database, which is covered by the free tier.

**Question 15: (Select all that apply)**
You are designing the architecture for a new internal HR portal. The frontend needs to be highly available but should NEVER be exposed to the public internet. The frontend needs to talk to a backend database, which also must never be exposed to the internet.
Which of the following configurations are required to meet these security constraints?
A) Expose the Frontend using a LoadBalancer Service.
B) Expose the Frontend using an Internal LoadBalancer Service (annotated for internal use).
C) Expose the Database using an Ingress Controller.
D) Expose the Database using a ClusterIP Service.

**Answers: B and D**
- **A is incorrect:** A standard LoadBalancer provisions a Public IP.
- **B is correct:** By using a LoadBalancer service with an internal Azure annotation, Azure provisions an Internal Load Balancer with a private IP, keeping the frontend off the internet but highly available on the corporate network.
- **C is incorrect:** Ingress is typically used for public HTTP routing.
- **D is correct:** The Database should only be accessed by the frontend within the cluster, so ClusterIP (which provides a purely internal cluster IP) is the perfect choice.
