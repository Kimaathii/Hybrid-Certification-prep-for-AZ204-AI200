# M03 — Lab Guide: Containerised Solutions (ACR, ACI, ACA)

---

| | |
|---|---|
| **Module** | M03 — Containerised Solutions |
| **Lab Title** | Build, Push, and Deploy Containers with ACR, ACI, and ACA |
| **Prerequisites** | F01–F06, M01–M02, Azure CLI installed, active Azure subscription |
| **Estimated Time** | 90 minutes |
| **What You Will Build** | A Node.js web app containerised and deployed to both ACI and ACA with traffic splitting |
| **What You Will Learn** | ✅ Build images with ACR Tasks · ✅ Deploy to ACI · ✅ Deploy to ACA · ✅ Traffic splitting · ✅ Cleanup |

---

> 💰 **COST WARNING**
>
> This lab uses the following resources:
> - **ACR Basic:** ~$0.17/day (~$5/month)
> - **ACI:** ~$0.0025/vCPU/hr (pennies for a short lab)
> - **ACA:** Free tier available (first 180,000 vCPU-seconds/month free)
>
> **Estimated total cost if cleaned up within 3 hours: under $0.10**
>
> ⚠️ **Delete all resources at the end of this lab.** The cleanup section is at the end — do NOT skip it.

---

## Part 1: Set Up the Environment

### Step 1: Create a Resource Group

**Why:** Every lab starts by creating a dedicated resource group. When you delete it at the end, everything inside gets deleted too.

```bash
az group create \
  --name az204-containers-rg \       # ← Resource group name
  --location eastus                  # ← Azure region
```

**Expected output:**
```json
{
  "location": "eastus",
  "name": "az204-containers-rg",
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

### Step 2: Create an Azure Container Registry

**Why:** You need a private registry to store your container images before deploying them.

```bash
# Save a unique name for your registry (lowercase, alphanumeric only)
ACR_NAME=az204acr$RANDOM

az acr create \
  --name $ACR_NAME \                 # ← Globally unique registry name
  --resource-group az204-containers-rg \
  --sku Basic \                      # ← Basic tier ($0.17/day) — cheapest option
  --admin-enabled true               # ← Enable admin credentials for lab simplicity
```

> 📝 **NOTE — Windows PowerShell Users**
>
> In PowerShell, variables use `$` prefix just like Bash but `$RANDOM` doesn't exist. Use:
> ```powershell
> $ACR_NAME = "az204acr$(Get-Random)"
> ```

**Save the registry login server:**

```bash
az acr show \
  --name $ACR_NAME \
  --query loginServer \              # ← Extract just the login server URL
  --output tsv
```

**Expected output:** `az204acrXXXXX.azurecr.io`

**Write this down — you'll need it throughout the lab.**

### Step 3: Create a Simple Node.js Application

**Why:** You need application code to containerise.

```bash
# Create a project directory
mkdir az204-container-app && cd az204-container-app

# Create a simple Node.js server
cat > server.js << 'EOF'
const http = require('http');

const version = process.env.APP_VERSION || 'v1.0';
const port = process.env.PORT || 3000;

const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
        message: `Hello from AZ-204 Container Lab!`,
        version: version,
        hostname: require('os').hostname(),
        timestamp: new Date().toISOString()
    }));
});

server.listen(port, () => {
    console.log(`Server running on port ${port} — Version ${version}`);
});
EOF

# Create package.json
cat > package.json << 'EOF'
{
    "name": "az204-container-app",
    "version": "1.0.0",
    "description": "AZ-204 Container Lab Application",
    "main": "server.js",
    "scripts": {
        "start": "node server.js"
    }
}
EOF
```

> ✅ **CHECKPOINT 1 — Steps 1–3**
>
> - ✅ Resource group `az204-containers-rg` created
> - ✅ ACR created and login server noted
> - ✅ Node.js app files created (`server.js` and `package.json`)

---

## Part 2: Build and Push an Image with ACR Tasks

### Step 4: Create a Dockerfile

**Why:** The Dockerfile tells Docker (and ACR Tasks) how to build your application into a container image.

```bash
cat > Dockerfile << 'EOF'
# Use the official Node.js 18 Alpine image as the base
# Alpine is a minimal Linux distro — keeps the image small
FROM node:18-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and install dependencies first
# This layer is cached — only re-runs if package.json changes
COPY package*.json ./
RUN npm install --production

# Copy the rest of the application code
COPY . .

# Set the default port
ENV PORT=3000

# Expose port 3000 (documentation for other developers)
EXPOSE 3000

# Command to run when the container starts
CMD ["npm", "start"]
EOF
```

### Step 5: Build the Image Using ACR Tasks (Cloud Build)

**Why:** This builds the image **in the cloud** — no Docker needed on your local machine. This is the `az acr build` command that is exam-critical.

```bash
az acr build \
  --registry $ACR_NAME \             # ← Your ACR name
  --image az204-app:v1.0 \           # ← Image name:tag
  .                                  # ← Build context (current directory)
```

**What happens:** Azure uploads your source code to ACR, builds the Docker image using your Dockerfile, and stores the resulting image in your registry. You will see build logs streaming in your terminal.

**Expected output (last lines):**
```
...
Step 7/7 : CMD ["npm", "start"]
 ---> Running in abc123
 ---> def456
Successfully built def456
Successfully tagged az204acrXXXXX.azurecr.io/az204-app:v1.0
```

### Step 6: Verify the Image Exists in ACR

```bash
az acr repository list \
  --name $ACR_NAME \                 # ← Your ACR name
  --output table                     # ← Clean table output
```

**Expected output:**
```
Result
----------
az204-app
```

```bash
# Check the tags for this image
az acr repository show-tags \
  --name $ACR_NAME \
  --repository az204-app \
  --output table
```

**Expected output:**
```
Result
------
v1.0
```

> ✅ **CHECKPOINT 2 — Steps 4–6**
>
> - ✅ Dockerfile created
> - ✅ Image built via `az acr build` (cloud build, no local Docker)
> - ✅ Image `az204-app:v1.0` visible in ACR repository list

---

## Part 3: Deploy to Azure Container Instances (ACI)

### Step 7: Get ACR Credentials

**Why:** ACI needs credentials to pull the image from your private ACR.

```bash
# Get the ACR admin password
ACR_PASSWORD=$(az acr credential show \
  --name $ACR_NAME \
  --query "passwords[0].value" \     # ← Extract the first password
  --output tsv)

# Get the ACR login server
ACR_LOGIN=$(az acr show \
  --name $ACR_NAME \
  --query loginServer \
  --output tsv)

echo "Login Server: $ACR_LOGIN"
echo "Password saved to variable"
```

### Step 8: Deploy the Container to ACI

**Why:** This runs your container image as a live application on the internet using ACI.

```bash
az container create \
  --name az204-aci-app \             # ← Container instance name
  --resource-group az204-containers-rg \
  --image $ACR_LOGIN/az204-app:v1.0 \  # ← Full image path in ACR
  --registry-login-server $ACR_LOGIN \  # ← ACR login server
  --registry-username $ACR_NAME \    # ← ACR admin username (same as registry name)
  --registry-password $ACR_PASSWORD \  # ← ACR admin password
  --dns-name-label az204-aci-$RANDOM \  # ← Public DNS label
  --ports 3000 \                     # ← Port to expose
  --environment-variables APP_VERSION=v1.0-ACI \  # ← Set environment variable
  --restart-policy Always \          # ← Always restart if container stops
  --cpu 1 \                          # ← 1 CPU core
  --memory 1                         # ← 1 GB RAM
```

### Step 9: Get the ACI Public URL and Test

```bash
az container show \
  --name az204-aci-app \
  --resource-group az204-containers-rg \
  --query ipAddress.fqdn \           # ← Get the full DNS name
  --output tsv
```

**Expected output:** `az204-aci-XXXXX.eastus.azurecontainer.io`

**Test it:**

```bash
curl http://az204-aci-XXXXX.eastus.azurecontainer.io:3000
```

**Expected output:**
```json
{
    "message": "Hello from AZ-204 Container Lab!",
    "version": "v1.0-ACI",
    "hostname": "abc123",
    "timestamp": "2024-01-15T10:30:00.000Z"
}
```

🎉 Your container is running live on Azure via ACI!

> ✅ **CHECKPOINT 3 — Steps 7–9**
>
> - ✅ ACR credentials retrieved
> - ✅ Container deployed to ACI
> - ✅ Accessed the running container via its public URL

---

## Part 4: Deploy to Azure Container Apps (ACA)

### Step 10: Create an ACA Environment

**Why:** An ACA Environment is the shared infrastructure boundary for your container apps. All apps in the same environment share a VNet and logging.

```bash
az containerapp env create \
  --name az204-aca-env \             # ← Environment name
  --resource-group az204-containers-rg \
  --location eastus
```

This may take 1–2 minutes.

### Step 11: Deploy to ACA (Revision 1)

```bash
az containerapp create \
  --name az204-aca-app \             # ← Container App name
  --resource-group az204-containers-rg \
  --environment az204-aca-env \      # ← The environment from Step 10
  --image $ACR_LOGIN/az204-app:v1.0 \  # ← Image from ACR
  --registry-server $ACR_LOGIN \     # ← ACR login server
  --registry-username $ACR_NAME \    # ← ACR username
  --registry-password $ACR_PASSWORD \  # ← ACR password
  --target-port 3000 \               # ← Port the app listens on inside the container
  --ingress external \               # ← Make it publicly accessible
  --env-vars APP_VERSION=v1.0-ACA \  # ← Environment variable
  --min-replicas 0 \                 # ← Scale to zero when idle
  --max-replicas 3                   # ← Maximum 3 instances
```

### Step 12: Get the ACA URL and Test

```bash
az containerapp show \
  --name az204-aca-app \
  --resource-group az204-containers-rg \
  --query properties.configuration.ingress.fqdn \
  --output tsv
```

**Expected output:** `az204-aca-app.niceriver-abc123.eastus.azurecontainerapps.io`

```bash
curl https://az204-aca-app.niceriver-abc123.eastus.azurecontainerapps.io
```

> 📝 **NOTE:** ACA uses HTTPS by default (unlike ACI which uses HTTP).

> ✅ **CHECKPOINT 4 — Steps 10–12**
>
> - ✅ ACA environment created
> - ✅ Container app deployed as Revision 1
> - ✅ Accessible via HTTPS URL

---

## Part 5: Traffic Splitting with ACA Revisions

### Step 13: Build and Push Version 2 of the App

**Why:** To demonstrate traffic splitting, you need two different versions of your app.

```bash
# Update the app to show v2.0
sed -i "s/v1.0/v2.0/g" server.js

# Build version 2
az acr build \
  --registry $ACR_NAME \
  --image az204-app:v2.0 \           # ← New tag: v2.0
  .
```

### Step 14: Deploy Version 2 as a New Revision

```bash
az containerapp update \
  --name az204-aca-app \
  --resource-group az204-containers-rg \
  --image $ACR_LOGIN/az204-app:v2.0 \  # ← Updated image
  --set-env-vars APP_VERSION=v2.0-ACA   # ← Updated env var
```

This creates **Revision 2** automatically.

### Step 15: Configure 80/20 Traffic Split

**Why:** Send 80% of traffic to v1 (stable) and 20% to v2 (new). This is a **canary release**.

```bash
# List revisions to get their names
az containerapp revision list \
  --name az204-aca-app \
  --resource-group az204-containers-rg \
  --output table
```

**Expected output (note the revision names):**
```
Name                            Active    TrafficWeight
------------------------------  --------  -------------
az204-aca-app--xxxxxxx          True      0
az204-aca-app--yyyyyyy          True      100
```

```bash
# Set 80/20 traffic split
az containerapp ingress traffic set \
  --name az204-aca-app \
  --resource-group az204-containers-rg \
  --revision-weight \
    az204-aca-app--xxxxxxx=80 \      # ← 80% to revision 1 (v1.0)
    az204-aca-app--yyyyyyy=20        # ← 20% to revision 2 (v2.0)
```

### Step 16: Test the Traffic Split

**Why:** Verify that requests are being distributed across both revisions.

```bash
# Make 10 requests and see the version in each response
for i in {1..10}; do
  curl -s https://az204-aca-app.niceriver-abc123.eastus.azurecontainerapps.io | grep version
done
```

**Expected output (approximately 8 v1.0 and 2 v2.0 responses):**
```
"version": "v1.0-ACA"
"version": "v1.0-ACA"
"version": "v2.0-ACA"
"version": "v1.0-ACA"
...
```

> ✅ **CHECKPOINT 5 — Steps 13–16**
>
> - ✅ Version 2 of the app built and pushed to ACR
> - ✅ New revision created in ACA
> - ✅ 80/20 traffic split configured
> - ✅ Both versions responding to requests

---

## Part 6: Clean Up 🧹

> 💰 **COST WARNING**
>
> **Do NOT skip this section.** ACR Basic costs ~$5/month if left running. Delete everything now.

### Step 17: Delete All Resources

```bash
# Delete the entire resource group (deletes everything inside)
az group delete \
  --name az204-containers-rg \
  --yes \                            # ← Skip confirmation prompt
  --no-wait                          # ← Return immediately, deletion runs in background
```

### Step 18: Verify Cleanup

Wait 2 minutes, then:

```bash
az group list --output table
```

**If `az204-containers-rg` no longer appears, you are fully cleaned up.** ✅

---

## Troubleshooting

### Error 1: `"az acr build" failed with "unable to prepare context"`

**Cause:** You ran `az acr build` from a directory without a Dockerfile.
**Fix:** Make sure you are in the directory containing the Dockerfile: `ls Dockerfile`

### Error 2: `"unauthorized: authentication required"` when deploying to ACI

**Cause:** ACR admin credentials are wrong or admin is not enabled.
**Fix:** Enable admin: `az acr update --name $ACR_NAME --admin-enabled true`, then re-fetch credentials.

### Error 3: ACI container shows status `Failed`

**Cause:** The container crashed on startup.
**Fix:** Check logs: `az container logs --name az204-aci-app --resource-group az204-containers-rg`

### Error 4: ACA environment creation fails

**Cause:** The region may not support ACA yet.
**Fix:** Try a different region: `--location westus2` or `--location westeurope`

### Error 5: Traffic split percentages don't add up to 100

**Cause:** ACA requires all traffic weights to sum to exactly 100.
**Fix:** Ensure your `--revision-weight` values total 100 (e.g., 80 + 20 = 100).

---

## What We Covered ✅

- [ ] Created a resource group for container resources
- [ ] Created an Azure Container Registry (Basic tier)
- [ ] Wrote a Dockerfile for a Node.js application
- [ ] Built a container image using ACR Tasks (`az acr build`) — no local Docker
- [ ] Verified the image exists in ACR
- [ ] Deployed the image to Azure Container Instances (ACI)
- [ ] Accessed the running ACI container via its public URL
- [ ] Created an Azure Container Apps (ACA) environment
- [ ] Deployed the container to ACA with external ingress
- [ ] Built a second version of the app (v2.0)
- [ ] Created a new ACA revision with the updated image
- [ ] Configured 80/20 traffic splitting between revisions
- [ ] Verified traffic distribution across both versions
- [ ] Cleaned up all resources
