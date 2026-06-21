# Module 8: Azure Kubernetes Service (AKS) - Scenario Cards

## SCENARIO 1: The Dual-Layer Architecture (Exposing APIs)
**Difficulty:** Beginner

**Business Context:** "TechStore" is building a new e-commerce platform on AKS. The application has two components: a React.js frontend (UI) and a Python backend (Payment processing).
**The Problem:** The frontend needs to be accessible to customers on the internet. However, the payment processing backend contains sensitive logic and must never be exposed directly to the public internet. It should only accept traffic from the frontend pods.
**Constraints:** 
1. The frontend must have a public IP address.
2. The backend must NOT have a public IP address.
3. Both must run in the same AKS cluster.
**Your Task:** Which Kubernetes Service types should you assign to the frontend and the backend?
**Hints:** 
- Think about the "Office Building" analogy. Who needs to talk to the street, and who just uses the internal intercom?
**Solution:** 
You should deploy a **LoadBalancer** service for the React frontend, and a **ClusterIP** service for the Python backend. 
A LoadBalancer service hooks into an Azure Load Balancer to provision a public IP. A ClusterIP service assigns an internal-only IP that can only be reached by other pods within the identical K8s cluster.
**Exam Connection:** The AZ-204 exam frequently asks you to choose the correct service type. If the scenario mentions "internal only" or "backend microservice," always choose ClusterIP.

---

## SCENARIO 2: Handling Black Friday Traffic (HPA)
**Difficulty:** Intermediate

**Business Context:** "GiftBox" is an online retailer. They normally have steady traffic, but on Black Friday, their traffic increases by 1000%. 
**The Problem:** Last year, their single web server crashed due to CPU exhaustion. They have moved to AKS, but currently, their Deployment is hardcoded to `replicas: 2`. 
**Constraints:** 
1. The system must automatically add more Pods when CPU usage exceeds 75%.
2. The system must automatically remove Pods when the sale ends to save money.
3. No manual intervention from IT administrators is allowed on the day.
**Your Task:** What native Kubernetes feature must you configure to handle this?
**Hints:** 
- You want to scale horizontally (adding more pods), not vertically (making the pods bigger).
**Solution:** 
You must configure a **Horizontal Pod Autoscaler (HPA)**. 
HPA continuously monitors the resource metrics (like CPU and memory) of your pods. You can set a rule saying: "Maintain average CPU usage across all pods at 75%. Minimum 2 pods, maximum 20 pods." When traffic spikes and CPU hits 80%, HPA automatically updates the Deployment replica count, spinning up new pods. When traffic drops, it scales back down to 2.
**Exam Connection:** For AZ-204, know the difference between HPA (scaling the number of pods) and Cluster Autoscaler (scaling the underlying Azure VMs when the cluster runs out of room for new pods).

---

## SCENARIO 3: The Forgotten User Data (Stateful vs Stateless)
**Difficulty:** Intermediate

**Business Context:** "NoteTaker App" allows users to write markdown notes. The developers deployed a MongoDB database container inside AKS alongside their web app.
**The Problem:** Every time the AKS node undergoes maintenance, the MongoDB pod restarts. When the database comes back online, all the users' notes are completely gone. 
**Constraints:** 
1. Database data must survive pod restarts.
2. Database data must survive node crashes.
3. The database pod only requires access to the disk from one node at a time.
**Your Task:** How do you modify the deployment to ensure data is permanently saved?
**Hints:** 
- By default, pods are like hotel rooms. Everything is thrown away when you leave. How do you rent a permanent storage unit?
**Solution:** 
The database is currently relying on ephemeral pod storage (stateless). You need to implement **Persistent Volumes (PV)** and **Persistent Volume Claims (PVC)** using **Azure Disks**.
You create a PVC in your YAML manifest requesting (e.g.) 50GB of storage. Azure provisions an actual managed disk. The pod mounts this disk. When the pod dies, the disk is detached, safely retaining the data. When the new pod spins up, Kubernetes reattaches the disk to the new pod. Since it's a database accessed by one pod, Azure Disks (ReadWriteOnce) is the correct choice over Azure Files.
**Exam Connection:** If a question asks how to persist state in AKS, look for Persistent Volumes or Azure Disks/Files.

---

## SCENARIO 4: AI Inference Bottlenecks (GPU Isolation)
**Difficulty:** Advanced (AI-200 Focus)

**Business Context:** "MedicalVision" runs a web platform where doctors upload X-rays. A heavy AI model analyzes the X-ray for anomalies. 
**The Problem:** To make the AI run fast enough, they upgraded all 10 of their AKS nodes to expensive N-Series GPU virtual machines. Now, their monthly Azure bill is $15,000. 80% of the time, the GPU nodes are just serving static HTML web pages.
**Constraints:** 
1. The AI model must run on GPUs.
2. The static web pages must run on cheap CPUs.
3. Both must run in the same AKS cluster.
**Your Task:** How do you restructure the AKS node architecture to slash costs while meeting performance needs?
**Hints:** 
- Think about creating separate groups of VMs. 
- How do you prevent the web app from sneaking onto the expensive VMs?
**Solution:** 
You need to implement multiple **Node Pools** with **Taints and Tolerations**.
1. Create a "System" node pool using cheap CPU VMs (e.g., D-Series) for the frontend web app.
2. Create a "User" node pool using expensive GPU VMs (N-Series) exclusively for the AI model.
3. Apply a **Taint** to the GPU node pool (e.g., `hardware=gpu:NoSchedule`). This acts as a forcefield. Standard pods will be repelled.
4. Add a **Toleration** to the AI model's deployment YAML. This acts as the VIP pass, allowing only the AI pod to bypass the taint and run on the GPU nodes.
**Exam Connection:** Taints and tolerations are critical exam topics for isolating workloads, especially expensive AI or memory-intensive database workloads.

---

## SCENARIO 5: The Midnight AI Queue (KEDA)
**Difficulty:** Advanced (AI-200 Focus)

**Business Context:** "AutoSubtitle" processes massive batches of videos overnight using an AI transcription model on AKS. Users upload videos, which drop a message into an Azure Service Bus Queue.
**The Problem:** Because the transcription pod just pulls one message at a time, its CPU usage is always a steady 50%. The Horizontal Pod Autoscaler (HPA) sees 50% CPU and decides NOT to scale up. Meanwhile, there are 5,000 videos waiting in the queue, and customers are angry about the delay.
**Constraints:** 
1. The system must scale up pods based on the number of waiting videos, NOT CPU usage.
2. When the queue is empty at 4 AM, the number of AI pods must scale down to exactly zero to stop all GPU billing.
**Your Task:** What tool must you install and configure to solve this?
**Hints:** 
- The scaling needs to be *driven by events*.
**Solution:** 
You must use **KEDA (Kubernetes Event-driven Autoscaling)**.
Standard HPA cannot see inside an Azure Service Bus Queue; it only sees the pod's CPU/Memory. KEDA acts as a bridge. You configure KEDA to monitor the Service Bus Queue length. If the queue length hits 100, KEDA overrides standard scaling and forces AKS to spin up dozens of pods to crush the backlog. More importantly, while HPA can only scale down to 1 minimum pod, KEDA supports **scale-to-zero**, completely shutting down the GPU workloads when the queue is empty.
**Exam Connection:** Any question mentioning scaling based on "queue length," "events," "Service Bus," or "scaling down to 0" points directly to KEDA.
