# Azure Foundation Practical Assessment (F01 & F02)

**Course:** AZ-204 / AI-200 Hybrid Masterclass  
**Sprint Name:** Foundation Setup & Governance  
**Estimated Time:** 60 minutes  
**Tools Required:** Azure Portal, Azure Cloud Shell (Bash)  

---

## 🏢 Scenario: Welcome to the Team
Imagine you are the Lead Cloud Engineer at a tech startup building a new AI platform called **AlphaAI**. Before your developers can write a single line of code, you must build the foundation. 

If you use the Azure Portal (the graphical interface) to click around and build this, your infrastructure cannot be replicated, tracked, or automated. As a professional, you will use the **Azure CLI (Command Line Interface)** and **Infrastructure as Code (IaC)**. 

### 📝 Submission Requirements for Google Classroom
To pass this sprint, you must submit proof of your work. For each ticket below, take a **screenshot** of your successful outcome (usually the JSON output in the terminal) and compile them into a single document to upload. **Do not skip Ticket 5, or you will incur costs!**

---

## 🎫 TICKET-F01: Establish Financial Guardrails

**The Problem:** The AlphaAI project is starting. If developers accidentally create expensive resources or leave them running, the company's Azure bill will skyrocket. We need a tripwire.

**Acceptance Criteria:**
- [ ] Create a $10 monthly budget alert in the Azure Portal.

💡 **Student Guide (How to complete):**
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **Cost Management** in the top search bar.
3. In the left menu, click **Budgets** -> **+ Add**.
4. Set the name to `AlphaAI-Monthly-Limit` and the amount to `10`.
5. Set an alert condition: When spending reaches 50% ($5), send an email to your address.
6. **📸 Take a screenshot of your active budget in the portal.**

---

## 🎫 TICKET-F02: Provision the Safe Workspace (The Blast Radius)

**The Problem:** We need a logical folder to hold all AlphaAI resources. If we mix development and production resources, someone will eventually delete the wrong database. 

**Acceptance Criteria:**
- [ ] Use the Azure CLI to create a Resource Group named `AlphaAI-Dev-RG` in the `eastus` region.
- [ ] Apply two tags: `Project=AlphaAI` and `Environment=Development`.

💡 **Student Guide (How to complete):**
1. Open the **Azure Cloud Shell** (the `>_` icon at the top of the portal) and select **Bash**.
2. Run the following command. The `\` allows you to write long commands across multiple lines for readability.
```bash
az group create \
  --name AlphaAI-Dev-RG \
  --location eastus \
  --tags Project=AlphaAI Environment=Development
```
3. **📸 Take a screenshot of the terminal showing the successful JSON output.**

---

## 🎫 TICKET-F03: Declarative Infrastructure Deployment

**The Problem:** We need a storage account to hold our AI training data. The CTO insists we do not use the portal to create it. We must use a Bicep file so we have a permanent, version-controlled blueprint of our infrastructure.

**Acceptance Criteria:**
- [ ] Save the provided Bicep code into a file.
- [ ] Deploy the Bicep file to your Resource Group using the Azure CLI.
- [ ] Ensure the storage account name is globally unique by appending `$RANDOM`.

💡 **Student Guide (How to complete):**
1. In the Cloud Shell, type `code main.bicep` to open the built-in code editor.
2. Paste the following declarative code into the editor:
```bicep
param storageAccountName string
param location string = resourceGroup().location

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```
3. Click the `...` menu in the top right of the editor and select **Save**, then **Close Editor**.
4. Run the deployment command. Notice how we pass `$RANDOM` to guarantee the name isn't already taken by someone else on the internet:
```bash
az deployment group create \
  --resource-group AlphaAI-Dev-RG \
  --template-file main.bicep \
  --parameters storageAccountName=alphaai$RANDOM
```
5. **📸 Take a screenshot of the terminal showing the successful deployment output.**

---

## 🎫 TICKET-F04: Secure the Infrastructure

**The Problem:** The storage account you just deployed is going to contain sensitive training data. We cannot risk a junior developer accidentally running a delete command on this resource group.

**Acceptance Criteria:**
- [ ] Apply a `CanNotDelete` Resource Lock to the resource group via the Azure CLI.
- [ ] Attempt to delete the resource group to prove the lock works.

💡 **Student Guide (How to complete):**
1. Apply the protective lock to the entire Resource Group:
```bash
az lock create \
  --name SaveMyJob \
  --resource-group AlphaAI-Dev-RG \
  --lock-type CanNotDelete \
  --notes "Protects AlphaAI from accidental destruction"
```
2. Test the lock! Run the delete command:
```bash
az group delete --name AlphaAI-Dev-RG --yes --no-wait
```
3. **📸 Take a screenshot of the red ERROR message in the terminal. The error proves you successfully secured the environment!**

---

## 🎫 TICKET-F05: Sprint Cleanup (CRITICAL)

**The Problem:** The sprint is over. If you leave this environment running, the company continues to pay for it. A core skill of a cloud engineer is knowing how to clean up when testing is done.

**Acceptance Criteria:**
- [ ] Remove the Resource Lock.
- [ ] Delete the Resource Group.
- [ ] Verify the Resource Group is completely gone.

💡 **Student Guide (How to complete):**
1. You cannot delete the group until you remove the lock:
```bash
az lock delete --name SaveMyJob --resource-group AlphaAI-Dev-RG
```
2. Now, execute the Golden Cleanup Command. This destroys the group and everything inside it permanently:
```bash
az group delete --name AlphaAI-Dev-RG --yes --no-wait
```
*(Note: Deletion happens in the background and takes 2-3 minutes).*

3. Verify it is gone. Run this command until `AlphaAI-Dev-RG` no longer appears in the list:
```bash
az group list --output table
```
4. **📸 Take a final screenshot of the `az group list` output, proving your lab is clean and you will not incur costs.**

---
✅ **Congratulations! You have successfully completed the Foundation Sprint. Upload your 5 screenshots to Google Classroom.**
