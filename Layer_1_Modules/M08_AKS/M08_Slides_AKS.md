# Module 8: Azure Kubernetes Service (Hybrid Web + AI Edition) - Slide Deck

## Slide 1: Title Slide
**Title:** Module 8: Azure Kubernetes Service (AKS)
**Exam Domains:** AZ-204 (Web App Deployment) + AI-200 (AI Infrastructure)
**Weight:** 15-20%

**Learning Objectives:**
- Understand core Kubernetes architecture (Pods, Nodes, Deployments, Services)
- Differentiate managed vs unmanaged components in AKS
- Expose web applications using Kubernetes networking
- Run and scale AI inference workloads using GPU pools and KEDA

**Speaker Notes:**
Welcome to Module 8. Today we are diving into Azure Kubernetes Service, commonly known as AKS. This is a massive module because Kubernetes is the industry standard for running modern, scalable applications. For the AZ-204 exam, you need to understand how to deploy web apps here. For the AI-200 exam, you need to understand how to use AKS to run heavy AI models. Let's get started.

---

## Slide 2: The Problem This Service Solves
**Visual:** Diagram showing a single server scaling to dozens of servers, with a stressed IT admin trying to manage cables and error alerts manually.
**Text:** 
- Containers are great. Managing hundreds of them is not.
- What happens if a server crashes at 2 AM?
- How do you route traffic evenly?
- How do you update without downtime?

**Speaker Notes:**
Before we talk about the solution, let's understand the problem. You've learned how to package your app into a Docker container. That's great for one app on your laptop. But in production, you might have hundreds of containers spread across dozens of servers. If a server dies at 2 AM, who restarts the container on a healthy server? If traffic spikes, who spins up more copies? Doing this manually is impossible. We need an automated manager.

---

## Slide 3: What is Kubernetes?
**Visual:** Analogy diagram: A conductor leading a symphony orchestra.
**Text:** 
- Kubernetes (K8s) is an open-source container orchestration platform.
- It is the **Conductor** of the orchestra.
- It tells containers where to run, when to scale, and when to restart.

**Speaker Notes:**
Enter Kubernetes, often abbreviated as K8s. Think of Kubernetes as the conductor of an orchestra. Your containers are the musicians. The conductor doesn't play the instruments, but they ensure everyone plays in time, at the right volume. If a violinist drops their bow, the conductor signals another to take over so the music never stops. Kubernetes orchestrates your containers.

---

## Slide 4: Kubernetes Architecture Overview
**Visual:** High-level diagram showing a Control Plane communicating with multiple Worker Nodes.
**Text:**
Two main parts of a K8s Cluster:
1. **Control Plane:** The brains (API server, scheduler).
2. **Data Plane / Nodes:** The muscle (Where your apps actually run).

**Speaker Notes:**
Every Kubernetes cluster is split into two halves. First, you have the Control Plane. This is the brain of the operation. It makes the decisions. Second, you have the Data Plane, which consists of Worker Nodes. This is the muscle. This is where your actual application code runs. 

---

## Slide 5: Nodes (The Stage)
**Visual:** Icon of a physical server / VM.
**Text:**
- **Node:** A physical or virtual machine.
- Provides CPU, Memory, and Storage.
- A group of Nodes is called a Cluster.

**Speaker Notes:**
Let's break down the vocabulary. A Node is just a virtual machine or a physical server. In our orchestra analogy, the Nodes are the stage and the chairs where the musicians sit. Without nodes, you have nowhere to run your application.

---

## Slide 6: Pods (The Musicians)
**Visual:** A Pod wrapping around a Docker container icon.
**Text:**
- **Pod:** The smallest deployable unit in Kubernetes.
- Usually contains 1 container (e.g., your Web API).
- Ephemeral (temporary): They live, they die, they get replaced.

**Speaker Notes:**
You actually don't deploy containers directly into Kubernetes. You deploy Pods. A Pod is a wrapper around your container. If the Node is the chair, the Pod is the musician sitting in it holding their instrument. Pods are ephemeral. This means they are temporary. If a Pod crashes, Kubernetes doesn't try to fix it; it throws it away and creates a brand new one.

---

## Slide 7: Deployments (The Sheet Music)
**Visual:** A document icon pointing to 3 identical Pods.
**Text:**
- **Deployment:** Instructions for the desired state.
- "I always want exactly 3 replicas of my web app running."
- Auto-heals: If 1 dies, it creates 1 to replace it.

**Speaker Notes:**
How do we tell the system what we want? We use a Deployment. A Deployment is like the sheet music for the conductor. You write a rule that says, "I always want 3 copies of my web app running." If a server catches fire and takes down one of your Pods, the Deployment notices you only have 2, and immediately spins up a 3rd one on a healthy server. 

---

## Slide 8: Services (The Sound System)
**Visual:** A stable IP address routing traffic to 3 changing Pods.
**Text:**
- **Service:** A stable, permanent IP address for your Pods.
- Since Pod IPs constantly change, Services provide a reliable entry point.

**Speaker Notes:**
Because Pods die and are replaced constantly, their IP addresses change all the time. You can't point your frontend website to a backend Pod's IP, because tomorrow it will be gone. A Service is like the orchestra's sound system. It provides a permanent IP address. You send traffic to the Service, and the Service routes it to whichever Pods happen to be alive at that exact millisecond.

---

## Slide 9: What is Azure Kubernetes Service (AKS)?
**Visual:** Azure logo + Kubernetes logo.
**Text:**
- **AKS:** Microsoft's managed Kubernetes platform.
- Removes the headache of installing and maintaining the Control Plane.
- Secure, integrated with Azure AD, and ready for production.

**Speaker Notes:**
Running Kubernetes yourself is incredibly hard. You have to configure the control plane, manage the network, and handle security certificates. Azure Kubernetes Service, or AKS, takes that pain away. It is a managed service where Microsoft sets up and maintains the entire cluster foundation for you, so you can just focus on deploying your apps.

---

## Slide 10: Managed Control Plane vs Worker Nodes
**Visual:** Split screen. Left: Control Plane (Microsoft Logo). Right: Worker Nodes (Customer Logo).
**Text:**
- **Microsoft Manages:** The Control Plane (API, Scheduler, Etcd database).
- **You Manage:** The Worker Nodes (The VMs running your pods).

**Speaker Notes:**
In AKS, the responsibility is split. Microsoft completely manages the Control Plane. You don't even see the VMs running it. They patch it and keep it online. You only manage the Worker Nodes. You decide how many VMs you need and how powerful they should be.

---

## Slide 11: ⚠️ EXAM ALERT: AKS Pricing
**Visual:** [Red Background #D13438, White Text]
**Text:**
- 🚨 **EXAM ALERT**
- The AKS Control Plane is **FREE**.
- You only pay for the **Agent Nodes** (the VMs you spin up).
- (Exception: Uptime SLA tier adds a small fee for production).

**Speaker Notes:**
Pay close attention to this for the exam. How are you billed for AKS? The control plane is completely free. You only pay for the underlying virtual machines, the disks, and the networking used by your worker nodes. There is an optional "Uptime SLA" tier for strict production environments that costs money, but the standard cluster management is free.

---

## Slide 12: Deploying Apps: YAML Manifests
**Visual:** Snippet of YAML code showing `kind: Deployment` and `replicas: 3`.
**Text:**
- K8s is **Declarative**.
- You write YAML files to declare what you want.
- You don't write scripts telling it *how* to do it.

**Speaker Notes:**
To deploy your app, you write a YAML file. This is called a manifest. Kubernetes is declarative. You don't write a script that says "start container A, then if it works, start container B." You just declare "I want 3 containers." Kubernetes reads the YAML and figures out all the complex steps required to make reality match your file.

---

## Slide 13: Networking: Exposing Your App
**Visual:** 3 icons: ClusterIP (internal), LoadBalancer (external), Ingress (smart router).
**Text:**
1. **ClusterIP:** Internal only. 
2. **LoadBalancer:** Public IP for a single service.
3. **Ingress:** Smart HTTP/HTTPS routing.

**Speaker Notes:**
Once your app is running, how do users reach it? There are three Service types to know. ClusterIP is the default; it is strictly internal. Use it for backend databases. LoadBalancer provisions an Azure Load Balancer and gives you a Public IP. Ingress is the smartest; it sits at the edge and routes traffic based on the URL path.

---

## Slide 14: ⚠️ EXAM ALERT: Ingress vs LoadBalancer
**Visual:** [Red Background #D13438, White Text]
**Text:**
- 🚨 **EXAM ALERT**
- **Scenario:** Route `website.com/api` to Backend A, and `website.com/web` to Backend B using ONE Public IP.
- **Answer:** Use an **Ingress Controller**.

**Speaker Notes:**
Exam trap! If you have multiple web services in one cluster and you want to use a single public IP address, routing traffic based on the URL path—like /api versus /web—you must use an Ingress Controller. A standard LoadBalancer service cannot do path-based routing.

---

## Slide 15: Storage in AKS: Stateful vs Stateless
**Visual:** Hotel Room (Stateless) vs Storage Unit (Stateful).
**Text:**
- **Stateless:** Local Pod storage is deleted when the Pod dies.
- **Stateful (Persistent Volumes):** Independent storage attached to the Pod.
- **Azure Disks:** Attach to ONE node.
- **Azure Files:** Attach to MULTIPLE nodes.

**Speaker Notes:**
Because Pods die, any data saved directly inside them is lost. This is stateless. If you are running a database on AKS, you need a Persistent Volume. This is like renting a storage unit. If your Pod dies and moves to a new Node, it just reattaches the volume. Remember, Azure Disks can only attach to one node at a time. Azure Files can attach to many concurrently.

---

## Slide 16: AI on AKS: Why use AKS for AI?
**Visual:** Brain icon integrated with a K8s wheel.
**Text:**
- AI Inference requires massive compute (GPUs).
- GPUs are very expensive.
- AKS allows fine-grained scaling to minimize costs.

**Speaker Notes:**
Switching gears to AI. Why put AI models on Kubernetes? Because AI inference—which is running the model to get predictions—requires GPUs. GPUs are incredibly expensive. AKS gives you the ultimate control to scale these resources precisely, ensuring you only pay for heavy compute when you actually need it.

---

## Slide 17: GPU Node Pools & Isolation
**Visual:** Two boxes: System Node Pool (Cheap CPUs) and User Node Pool (Expensive GPUs).
**Text:**
- Keep regular web apps on cheap CPU nodes.
- Put AI models on specialized GPU node pools.
- Use **Taints and Tolerations** to keep normal apps off expensive hardware.

**Speaker Notes:**
In AKS, you can have different "pools" of nodes. You should put your standard web frontend on cheap CPU nodes. You create a separate pool of expensive GPU nodes just for your AI models. To ensure the web app doesn't accidentally run on the expensive hardware, Kubernetes uses a feature called Taints and Tolerations. You "taint" the GPU node, and only the AI pod has the "toleration" to run there.

---

## Slide 18: The Problem with Standard Scaling for AI
**Visual:** Graph showing CPU at 10% but a queue backlog of 10,000 messages.
**Text:**
- Standard K8s scales based on **CPU/Memory** (HPA).
- AI workloads might have low CPU, but a massive backlog of messages in a queue.
- HPA won't scale up. Traffic bottlenecks.

**Speaker Notes:**
Scaling AI is tricky. Standard Kubernetes scaling looks at CPU usage. But AI often works off a queue—like an Azure Service Bus queue full of images to process. The CPU might look low while it waits for the next image, but there could be 10,000 images waiting in line! Standard scaling won't add more pods, and your system will grind to a halt.

---

## Slide 19: KEDA: Event-Driven Scaling
**Visual:** KEDA logo scaling Pods based on Queue Length, scaling all the way down to 0.
**Text:**
- **KEDA (Kubernetes Event-driven Autoscaling).**
- Scales based on **Queue Length** (events).
- Can scale GPU pods down to **ZERO** when the queue is empty.

**Speaker Notes:**
The solution is KEDA. KEDA is an add-on that lets Kubernetes scale based on events, like the length of a queue. If 500 messages hit the queue, KEDA instantly spins up more AI pods. Even better, when the queue is completely empty, KEDA scales the expensive GPU pods down to zero. Standard K8s scaling can only go down to 1. Scaling to zero saves massive amounts of money.

---

## Slide 20: Module Summary Table
**Visual:** A clean summary table.
**Text:**
| Concept | Definition | Exam Focus |
|---------|------------|------------|
| **Pod** | Runs your container | Never deploy directly, use Deployments |
| **Deployment**| Desired state / Replicas | Auto-heals failed pods |
| **Service** | Permanent Network IP | LoadBalancer (public), Ingress (path-based) |
| **Node Pools**| Groups of VMs | Isolate AI to GPU pools via Taints |
| **KEDA** | Event-driven scaling | Scale to zero based on Queue length |

**Speaker Notes:**
Here is your cheat sheet for this module. Remember that Deployments manage Pods. Services provide networking, and you must know the difference between LoadBalancer and Ingress. For AI workloads, isolate them on GPU node pools, and use KEDA to scale based on queue length down to zero. That concludes Module 8!
