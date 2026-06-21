# Module 8 Lab: Deploying a Web API to Azure Kubernetes Service

| Meta | Details |
|------|---------|
| **Module** | 8: Azure Kubernetes Service (AKS) |
| **Lab Title** | Deploying a Web API to AKS |
| **Prerequisites** | Azure CLI installed, valid Azure subscription |
| **Estimated Time** | 45 minutes |
| **Cost Estimate** | ~$2.00 (if cleaned up immediately) |
| **What you build** | A managed Kubernetes cluster running a Python AI API exposed to the internet. |
| **What you learn** | `az aks create`, `kubectl`, writing YAML manifests, scaling Pods. |

💰 **COST WARNING**
This lab uses the Standard Azure Kubernetes Service tier and creates 2 Virtual Machines (Standard_B2s). Estimated cost: $2.00 if cleaned up within 2 hours. You MUST run the cleanup script at the end of this lab to prevent ongoing charges.

---

## Step 1: Create the AKS Cluster via Azure CLI

First, we need to create the Azure resource. We will use the Azure CLI to spin up the managed control plane and two worker nodes.

1. Open your terminal and log in to Azure:
```bash /
az login
```

2. Create a Resource Group to hold our cluster:
```bash /
# --name: The name of the resource group
# --location: The Azure region to deploy into
az group create \
  --name rg-aks-lab \
  --location eastus
```

3. Create the AKS cluster. (This will take 3-5 minutes).
```bash /
# --resource-group: Where to put the cluster
# --name: The name of your cluster
# --node-count: How many Worker Nodes (VMs) to create
# --generate-ssh-keys: Automatically creates SSH keys for node access
az aks create \
  --resource-group rg-aks-lab \
  --name my-aks-cluster \
  --node-count 2 \
  --generate-ssh-keys
```

**Expected Output:**
A massive block of JSON confirming the cluster was created successfully.

---

## Step 2: Connect via `kubectl`

AKS is running in Azure, but you need to manage it from your laptop using the Kubernetes command-line tool, `kubectl`.

1. Download the credentials for your new cluster:
```bash /
# --resource-group: Your resource group
# --name: Your cluster name
# This command merges the cluster keys into your local ~/.kube/config file
az aks get-credentials \
  --resource-group rg-aks-lab \
  --name my-aks-cluster
```

2. Verify you can talk to the cluster:
```bash /
kubectl get nodes
```

**Expected Output:**
```text
NAME                                STATUS   ROLES   AGE     VERSION
aks-nodepool1-12345678-vmss000000   Ready    agent   5m      v1.27.3
aks-nodepool1-12345678-vmss000001   Ready    agent   5m      v1.27.3
```

✅ **CHECKPOINT**
Look at the output above. Why are there two nodes? Because we passed `--node-count 2` in Step 1. Where is the Control Plane? It's managed by Microsoft and completely hidden from this list!

---

## Step 3: Create a Deployment YAML for a Python AI API

We will now deploy a pre-built Docker image of a simple Python API that simulates an AI workload. We do this by writing a **Deployment** YAML manifest.

1. Create a new file named `deployment.yaml` and paste the following code:

```yaml /
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-api-deployment
spec:
  # We want 1 pod running initially
  replicas: 1
  selector:
    matchLabels:
      app: ai-api
  template:
    metadata:
      labels:
        app: ai-api
    spec:
      containers:
      - name: python-api
        # A sample public image we are using for this lab
        image: mcr.microsoft.com/azuredocs/aci-helloworld:latest
        ports:
        # The port the container listens on internally
        - containerPort: 80
```

2. Apply the manifest to your cluster:
```bash /
# -f: The file path to the YAML manifest
kubectl apply -f deployment.yaml
```

3. Check if your Pod is running:
```bash /
kubectl get pods
```

**Expected Output:**
```text
NAME                                 READY   STATUS    AGE
ai-api-deployment-7f89b4c5d-x8z9w    1/1     Running   15s
```

---

## Step 4: Create a Service YAML to Expose It

Your Python API is running, but it's trapped inside the cluster. We need a **Service** of type `LoadBalancer` to give it a public IP address.

1. Create a new file named `service.yaml` and paste the following code:

```yaml /
apiVersion: v1
kind: Service
metadata:
  name: ai-api-service
spec:
  # This requests a Public IP from Azure
  type: LoadBalancer
  ports:
  # The public port exposed to the internet
  - port: 80
    # The internal port the container is listening on
    targetPort: 80
  selector:
    # This must perfectly match the label in our deployment.yaml
    app: ai-api
```

2. Apply the service manifest:
```bash /
kubectl apply -f service.yaml
```

3. Find your Public IP:
```bash /
# -w: "Watch" mode. It will update live as Azure provisions the IP.
kubectl get services -w
```

**Expected Output:**
```text
NAME             TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
ai-api-service   LoadBalancer   10.0.150.12    <pending>       80:30123/TCP   10s
ai-api-service   LoadBalancer   10.0.150.12    20.123.45.67    80:30123/TCP   45s
```
*(Wait until `<pending>` changes to a real IP address. Press `Ctrl+C` to exit watch mode).*

4. Open your web browser and navigate to the `EXTERNAL-IP`. You should see the welcome screen of the application!

✅ **CHECKPOINT**
If you deleted `service.yaml` and recreated it, would the Pods die? No! Services and Deployments are independent. The pods would keep running, they just wouldn't be reachable from the internet until you restored the Service.

---

## Step 5: Scale the Deployment

Your AI API has gone viral. One pod is not enough. Let's scale it to 3. 

1. You can scale dynamically via the CLI:
```bash /
# deployments/ai-api-deployment: The resource we are modifying
# --replicas: The new total number of pods we want
kubectl scale deployments/ai-api-deployment --replicas=3
```

2. Verify the new pods are spinning up:
```bash /
kubectl get pods
```

**Expected Output:**
```text
NAME                                 READY   STATUS              AGE
ai-api-deployment-7f89b4c5d-x8z9w    1/1     Running             10m
ai-api-deployment-7f89b4c5d-a1b2c    0/1     ContainerCreating   5s
ai-api-deployment-7f89b4c5d-d3e4f    0/1     ContainerCreating   5s
```

---

## Troubleshooting Top 5 Errors

1. **Error:** `az: command not found`
   - **Fix:** You need to install the Azure CLI on your machine.
2. **Error:** `ResourceGroupNotFound` when running `az aks create`
   - **Fix:** You missed Step 1.2. Ensure the resource group `rg-aks-lab` was created successfully.
3. **Error:** `kubectl: command not found`
   - **Fix:** Run `az aks install-cli` to install kubectl, then restart your terminal.
4. **Error:** Pod status shows `ImagePullBackOff`
   - **Fix:** You have a typo in the `image:` line of your `deployment.yaml`. Kubernetes cannot find the Docker image.
5. **Error:** External-IP stays `<pending>` forever.
   - **Fix:** This occasionally happens if your Azure region is out of Public IPs. Delete the service, wait 5 minutes, and try `kubectl apply -f service.yaml` again.

---

## Step 6: CLEANUP (Mandatory)

AKS node pools cost money every minute they run. You must destroy the cluster now.

1. Delete the entire resource group (this automatically deletes the AKS cluster, the Load Balancer, and the VMs):
```bash /
# --name: The resource group to delete
# --yes: Don't prompt for confirmation
# --no-wait: Run in the background so you get your terminal prompt back immediately
az group delete \
  --name rg-aks-lab \
  --yes \
  --no-wait
```

2. Verify deletion has started:
```bash /
az group list -o table
```
*(The `rg-aks-lab` should either be gone, or show a status of "Deleting").*
