# Module 8: Azure Kubernetes Service (Hybrid Web + AI Edition)

**Exam Domain:** AZ-204 (Web App Deployment) + AI-200 (AI Infrastructure)  
**Weight:** 15-20%  
**Estimated Study Time:** 3-4 hours  
**Prerequisites:** Basic understanding of containers (Docker), basic networking concepts.

---

## 1. What is Kubernetes (K8s)?

### The Problem This Solves
Imagine you are running a successful web application. Initially, it lived on one server. As your traffic grew, you added more servers. But managing these servers became a nightmare:
- If a server crashes, you have to manually restart the application.
- If traffic spikes, you have to manually spin up new servers and configure them.
- When you release a new version, you have to carefully update each server without causing downtime.

Managing dozens or hundreds of containers manually is like trying to play every instrument in a symphony orchestra yourself. You need a conductor.

### The Analogy: The Orchestra Conductor
Think of your containers as musicians. Each musician plays an instrument (runs a specific part of your application). 
**Kubernetes (K8s)** is the conductor of the orchestra. The conductor doesn't play the instruments. Instead, the conductor tells the musicians when to start, how loud to play, and what to do if someone falls out of sync. If a violinist drops their bow, the conductor ensures another violinist steps in so the music never stops.

### Technical Definition
**Kubernetes (often abbreviated as K8s)** is an open-source container orchestration platform. It automates the deployment, scaling, and management of containerized applications. 

Let's break down the key players in our orchestra (Kubernetes Architecture):

#### Nodes (The Chairs and Stages)
- **Analogy:** The chairs where the musicians sit. You need a physical place for them to perform.
- **Technical Definition:** A **Node** is a physical or virtual machine (VM) that runs your application workloads. A group of nodes is called a **Cluster**.

#### Pods (The Musicians)
- **Analogy:** The individual musicians holding their instruments.
- **Technical Definition:** A **Pod** is the smallest deployable unit in Kubernetes. It typically contains one container (like a Docker container), though it can contain multiple tightly-coupled containers. You don't deploy containers directly in K8s; you deploy Pods.

#### Deployments (The Sheet Music)
- **Analogy:** The sheet music that tells the orchestra exactly how the piece should be played (e.g., "We need 3 violins for this section").
- **Technical Definition:** A **Deployment** is an instruction set that tells Kubernetes how many copies (replicas) of a Pod should be running at all times. If a Pod crashes, the Deployment controller automatically starts a new one to maintain the desired number.

#### Services (The Microphones and Speakers)
- **Analogy:** The sound system. The audience doesn't need to know exactly which violinist is playing; they just listen to the output from the speakers.
- **Technical Definition:** A **Service** provides a stable, permanent network address (IP) for a set of Pods. Because Pods are constantly being created and destroyed, their individual IP addresses change. A Service gives your application a reliable way to talk to them.

💡 **KEY CONCEPT**
Pods are ephemeral (temporary). They die and are reborn with new IP addresses. Services are permanent. Always use a Service to communicate with Pods.

🚨 **EXAM ALERT**
On the AZ-204 exam, if a question asks how to ensure a specific number of instances are always running even if a node fails, the answer is to use a **Deployment** (not a Pod directly).

✅ **CHECKPOINT**
Can you explain the difference between a Pod and a Node? (Answer: A Node is the virtual machine hardware; a Pod is the wrapper around your container that runs *on* the Node).

---

## 2. What is AKS?

### The Problem This Solves
Running Kubernetes yourself is incredibly difficult. You have to install the software, manage the network, secure the control plane, and patch the servers. It's like having to build the concert hall and hire the conductor before you can even play music.

### The Analogy: The Managed Conductor
If running your own Kubernetes is building the concert hall, **Azure Kubernetes Service (AKS)** is renting a world-class concert hall where the conductor is already hired, trained, and managed for you. You just bring your musicians (your code).

### Technical Definition
**Azure Kubernetes Service (AKS)** is Microsoft's managed Kubernetes service. It simplifies deploying a managed Kubernetes cluster in Azure by offloading the operational overhead to Azure.

In any Kubernetes cluster, there are two main parts:
1. **The Control Plane (The Conductor):** Makes decisions about the cluster (scheduling, scaling, etc.).
2. **The Data Plane / Nodes (The Musicians):** Where your actual application runs.

In AKS:
- **Microsoft manages the Control Plane.** It is completely free, highly available, and patched automatically. You don't even see the VMs running it.
- **You manage the Nodes.** You pay for the virtual machines that run your Pods. 

🚨 **EXAM ALERT**
You are ONLY billed for the agent nodes (the VMs) you use in AKS. The AKS control plane is free (unless you opt into the Uptime SLA tier for production).

---

## 3. Web App Angle: Deploying a Scalable Web API

### The Problem This Solves
How do we actually tell the conductor what to do? We can't just shout commands. We need a written, formal instruction manual.

### The Analogy: The Contract
Imagine handing a written contract to a builder: "Build me a house with exactly 3 bedrooms and 2 bathrooms." The builder reads it and makes it happen.

### Technical Definition
In Kubernetes, we use **YAML Manifests** to declare the desired state of our application. YAML (YAML Ain't Markup Language) is a human-readable data format. We tell Kubernetes *what* we want, and Kubernetes figures out *how* to do it.

#### YAML Manifest Basics
Here is a Deployment YAML for a simple Web API:

```yaml /
# The API version of the Kubernetes object we are creating
apiVersion: apps/v1 
# The type of object (Deployment ensures our app stays running)
kind: Deployment 
metadata:
  # The name of our deployment
  name: my-web-api 
spec:
  # We want exactly 3 copies of our API running at all times
  replicas: 3 
  selector:
    matchLabels:
      # How the deployment knows which Pods belong to it
      app: web-api 
  template:
    metadata:
      labels:
        # The tag given to every Pod created
        app: web-api 
    spec:
      containers:
      # The name of the container inside the Pod
      - name: api-container 
        # The Docker image to pull from an Azure Container Registry
        image: myregistry.azurecr.io/myapi:v1 
        ports:
        # The port our container listens on
        - containerPort: 80 
```

💡 **KEY CONCEPT**
Kubernetes is **declarative**. You don't write a script saying "run a container, then check if it's alive, then run another." You write a YAML file declaring "I want 3 replicas," and Kubernetes continuously works to make reality match your declaration.

---

## 4. Networking: Exposing Your Application

### The Problem This Solves
You have 3 Pods running your Web API. How does a user on the internet actually reach them? 

### The Analogy: The Office Building
- **ClusterIP:** The internal intercom system. Only people inside the building can use it.
- **LoadBalancer:** The front reception desk. Anyone from the outside street can walk up and ask to be connected.
- **Ingress:** The intelligent mailroom. It looks at the envelope (the URL) and routes it to the specific department (e.g., mail for `/sales` goes here, mail for `/support` goes there).

### Technical Definition
Kubernetes **Services** handle networking. There are three main types you must know for the exam:

#### 1. ClusterIP (Default)
- **What it does:** Exposes the Service on a cluster-internal IP. 
- **Use case:** For internal microservices that should NOT be reachable from the internet (e.g., a backend database API).

#### 2. LoadBalancer
- **What it does:** Exposes the Service externally using Azure's native Load Balancer. It assigns a public IP address.
- **Use case:** When you need a simple, direct public entry point to a single application.

#### 3. Ingress Controller
- **What it does:** A smart router that sits at the edge of your cluster. It provides HTTP/HTTPS routing based on URLs.
- **Use case:** When you have multiple web apps in the same cluster and want to use one single Public IP to route traffic based on the path (e.g., `myapp.com/api` goes to one service, `myapp.com/web` goes to another).

🚨 **EXAM ALERT**
If a scenario asks how to route traffic to *different* backend services based on the URL path (Path-based routing), the answer is always **Ingress**.

✅ **CHECKPOINT**
If you deploy a backend payment processor that only the frontend web server should talk to, which Service type do you use? (Answer: ClusterIP).

---

## 5. Storage: Stateful vs Stateless

### The Problem This Solves
Pods are temporary. If a Pod crashes, any data saved inside its local file system is destroyed. This is fine for a stateless web API, but terrible for a database that needs to save user data.

### The Analogy: The Hotel Room vs. The Storage Unit
- **Stateless (Local Pod Storage):** A hotel room. You sleep there, but when you check out, the cleaners throw away anything you left behind.
- **Stateful (Persistent Volumes):** A rented storage unit. You can move to a different apartment (Pod), but your stuff is safely locked in the storage unit (Volume) and you still have the key.

### Technical Definition
To keep data permanently, Kubernetes uses two concepts:

1. **Persistent Volume (PV):** The actual physical storage. In Azure, this is typically an Azure Disk or Azure Files share.
2. **Persistent Volume Claim (PVC):** The request made by the Pod to use the storage. It's like a ticket that says "I need 10GB of fast storage."

When a Pod requests a PVC, Kubernetes finds a matching PV and mounts it. If the Pod dies, the PV remains untouched. When the replacement Pod spins up, it attaches to the exact same PVC and picks up right where the old one left off.

💡 **KEY CONCEPT**
Use **Azure Disks** if the volume only needs to be attached to ONE Pod at a time (ReadWriteOnce). Use **Azure Files** if multiple Pods across different Nodes need to read and write to the same volume simultaneously (ReadWriteMany).

---

## 6. [AI-200 ADDITION] AI Workloads on AKS

### The Problem This Solves
You've trained an incredible Machine Learning model. Now you need to run it (Inference) so users can send it data and get predictions. AI inference requires massive computational power, specifically GPUs (Graphics Processing Units), which are expensive. You only want to pay for GPUs when you are actively processing AI requests.

### The Analogy: The Specialized Workshop
If regular Nodes are standard office desks for office workers (Web APIs), **GPU Nodes** are specialized heavy machinery workshops. You only want to rent the heavy machinery when you have physical metal to cut, and you want to lock the door and stop paying rent the minute the job is done.

### Technical Definition
Why use AKS for AI?
1. **GPU Node Pools:** AKS allows you to create separate "pools" of nodes. You can have a cheap Linux CPU node pool for your Web API frontend, and an expensive GPU node pool (e.g., Azure N-Series VMs) exclusively for your AI models.
2. **Taints and Tolerations:** K8s ensures normal web apps don't accidentally get placed on expensive GPU nodes. You "taint" the GPU node (like putting up a "Hazardous Materials" sign), and only Pods that explicitly "tolerate" that taint can run there.

🚨 **EXAM ALERT**
To run AI workloads cost-effectively, isolate them. Deploy standard web services to a system node pool (CPU), and deploy AI inference models to a specialized user node pool equipped with GPUs. Use taints/tolerations to restrict access.

---

## 7. [AI-200 ADDITION] Scaling AI Inference with KEDA

### The Problem This Solves
Standard Kubernetes scaling (Horizontal Pod Autoscaler - HPA) looks at CPU and Memory. If CPU hits 80%, it adds more Pods. But AI workloads are different. An AI model might be pulling messages from an Azure Service Bus queue (e.g., "Analyze this image"). The CPU might be low, but the queue has 10,000 images waiting! HPA won't scale up because CPU is fine, resulting in massive delays.

### The Analogy: The Supermarket Checkout
Standard scaling (HPA) is like opening a new checkout lane only when the cashiers start sweating and their heart rates go up (CPU). 
**Event-driven scaling (KEDA)** is like looking at the length of the line of customers. If there are 50 people waiting in line (Queue length), open 5 more lanes instantly, regardless of how the cashiers feel.

### Technical Definition
**KEDA (Kubernetes Event-driven Autoscaling)** is an add-on for AKS. It allows Kubernetes to scale Pods based on external events, such as the number of messages in an Azure Storage Queue, Azure Service Bus, or Kafka topic.

**How it works for AI:**
1. Users upload images to a website. The website drops a message in an Azure Service Bus Queue.
2. KEDA monitors the Queue.
3. As the Queue grows, KEDA tells Kubernetes to spin up more AI Inference Pods on the GPU nodes.
4. When the Queue is empty, KEDA scales the Pods down to **ZERO** (saving you huge amounts of money on idle GPUs).

💡 **KEY CONCEPT**
Standard HPA can only scale down to 1. KEDA can scale down to **0**. For expensive AI GPU nodes, scaling to zero when the queue is empty is critical for cost savings.

---

## What We Covered (Checklist)

- [ ] **Kubernetes Architecture:** The orchestra. Nodes (VMs), Pods (containers), Deployments (replicas), Services (networking).
- [ ] **AKS:** Microsoft manages the Control Plane (free); you manage the Nodes (paid).
- [ ] **YAML Manifests:** Declarative instructions for deploying apps.
- [ ] **Networking:** ClusterIP (internal), LoadBalancer (public IP), Ingress (smart URL routing).
- [ ] **Storage:** Persistent Volumes (PV) and Claims (PVC) for saving state. Azure Disks vs Azure Files.
- [ ] **AI Workloads:** Using separate GPU node pools to isolate and run heavy AI inference models.
- [ ] **KEDA:** Scaling Pods based on queue length (events) instead of CPU, including scaling to zero to save costs.

🔗 **MODULE CONNECTIONS**
- **Module 3 (Azure Container Registry):** Where your Docker images are stored before AKS pulls them to create Pods.
- **Module 11 (Azure Service Bus):** The queue system KEDA monitors to scale your AI workloads.

📝 **NOTE**
Kubernetes is a massive topic. For the AZ-204 and AI-200 exams, focus on the *Azure-specific* integrations (Azure Disks, Azure Load Balancer, KEDA) rather than deep, pure open-source Kubernetes administration.
