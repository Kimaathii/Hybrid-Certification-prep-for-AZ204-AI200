# Module 11 Lab: Passwordless AI Authentication

| Meta | Details |
|------|---------|
| **Module** | 11: Identity & Authorization |
| **Lab Title** | Implementing Passwordless Authentication for AI Services |
| **Prerequisites** | Azure CLI installed, .NET 8 SDK installed, active Azure Subscription |
| **Estimated Time** | 45 minutes |
| **Cost Estimate** | Free ($0.00) using F1 Free Tier for App Service and Azure OpenAI Mock |
| **What You Build** | An App Service that connects to an AI endpoint using a Managed Identity (no API keys) |
| **What You Learn** | Entra ID App Registrations, System-Assigned Identities, Azure RBAC, `DefaultAzureCredential` |

💰 **COST CALLOUT:** This lab uses Azure App Service (F1 Free Tier) and Azure RBAC role assignments. Estimated cost: $0.00 if cleaned up within 24 hours. *Note: If you use a real Azure OpenAI instance instead of a mock endpoint, standard token generation costs apply.*

---

## Part 1: Initial Setup and App Registration

First, we will create our resource group and register a dummy application in Microsoft Entra ID to understand the process.

**Step 1:** Log in to Azure and set variables.
```bash
# Log in to your Azure account
az login

# Set variables to make copying/pasting easier
export RG_NAME="rg-authlab-dev"
export LOCATION="eastus"

# Create the resource group
az group create \
  --name $RG_NAME \
  --location $LOCATION
```
*Expected Output:* JSON confirming `provisioningState: Succeeded`.

**Step 2:** Create an App Registration in Entra ID.
Even though we will use Managed Identities later, it is crucial to know how to create a manual App Registration.

```bash
# Create an App Registration (Service Principal will be created automatically in the background)
az ad app create \
  --display-name "MyAuthLabApp" \
  --sign-in-audience "AzureADMyOrg" # Restricts login to only users in your specific Entra ID tenant
```

✅ **CHECKPOINT:** We just created an App Registration. Is this the global blueprint or the local badge? *(Answer: The global blueprint. Azure automatically created the local Service Principal behind the scenes for you).*

---

## Part 2: Create App Service and System-Assigned Identity

We will create a free Azure Web App and turn on its System-Assigned Managed Identity.

**Step 3:** Create an App Service Plan (Free Tier).
```bash
export APP_PLAN="plan-authlab-free"

# Create the hosting plan
az appservice plan create \
  --name $APP_PLAN \
  --resource-group $RG_NAME \
  --sku F1 \
  --is-linux # Deploys a Linux-based host instead of Windows
```

**Step 4:** Create the Web App and enable Identity.
```bash
export APP_NAME="app-authlab-$RANDOM"

# Create the web app AND assign a system identity in one command
az webapp create \
  --name $APP_NAME \
  --resource-group $RG_NAME \
  --plan $APP_PLAN \
  --assign-identity '[system]' # This flag tells Azure to create the System-Assigned Managed Identity
```
*Expected Output:* A large JSON block. Look for the `"identity"` section which will contain a `"principalId"`. This is the ID of your new security badge!

**Step 5:** Retrieve the Principal ID for later use.
```bash
# Extract just the Principal ID of the web app's identity using a JMESPath query
export PRINCIPAL_ID=$(az webapp identity show --name $APP_NAME --resource-group $RG_NAME --query principalId -o tsv)
echo "My App's Identity ID is: $PRINCIPAL_ID"
```

✅ **CHECKPOINT:** If we delete this Web App, what happens to the identity whose ID we just printed? *(Answer: It is permanently deleted because it is System-Assigned).*

---

## Part 3: Assign RBAC Roles

Our Web App has an identity, but it has no permissions. We need to Authorize it (AuthZ) to use Cognitive Services.

**Step 6:** Get your subscription ID.
```bash
export SUB_ID=$(az account show --query id -o tsv)
```

**Step 7:** Assign the "Cognitive Services User" role.
*Note: We are assigning this role at the resource group scope for the lab. In production, apply it strictly to the specific AI resource.*

```bash
# Assign the RBAC role
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Cognitive Services User" \
  --scope "/subscriptions/$SUB_ID/resourceGroups/$RG_NAME" # Grants permission to any AI service inside this Resource Group
```
*Expected Output:* JSON block confirming the `roleDefinitionName` is "Cognitive Services User".

---

## Part 4: The Passwordless Code

Now we write the C# code that leverages this identity.

**Step 8:** Create a new .NET console app locally.
```bash
dotnet new console -n PasswordlessAILab
cd PasswordlessAILab
```

**Step 9:** Install the required Azure SDK packages.
```bash
# Install the Identity library containing DefaultAzureCredential
dotnet add package Azure.Identity

# Install the OpenAI library
dotnet add package Azure.AI.OpenAI --prerelease
```

**Step 10:** Replace `Program.cs` with the passwordless code.
Open `Program.cs` and paste the following:

```csharp
using System;
using Azure.Identity;
using Azure.AI.OpenAI;

Console.WriteLine("Starting Passwordless Connection...");

// 1. Initialize DefaultAzureCredential. 
// No secrets, passwords, or API keys are written here!
var credential = new DefaultAzureCredential();

// 2. Define the endpoint. (Replace with a real endpoint if you have one)
var aiEndpoint = new Uri("https://my-mock-ai-resource.openai.azure.com/");

try 
{
    // 3. Connect using the identity credential instead of an API key
    var client = new OpenAIClient(aiEndpoint, credential);
    Console.WriteLine("Successfully authenticated using Entra ID!");
    
    // Note: If using a real endpoint, you would make an API call here.
}
catch (Exception ex)
{
    Console.WriteLine($"Authorization Failed: {ex.Message}");
}
```

✅ **CHECKPOINT:** Run `dotnet run`. If it succeeds locally, what identity did `DefaultAzureCredential` use? *(Answer: Your personal developer credentials from Azure CLI/Visual Studio. When deployed to Azure, it will seamlessly switch to the Web App's System-Assigned Identity).*

---

## TROUBLESHOOTING

Here are the top 5 errors you might encounter in this lab and how to fix them:

1. **Error: "AuthenticationFailedException: DefaultAzureCredential failed to retrieve a token"**
   *Fix:* You are not logged in locally. Run `az login` in your terminal before running `dotnet run`.
2. **Error: "AuthorizationFailed: The client does not have authorization to perform action"**
   *Fix:* AuthZ failure. The RBAC role assignment hasn't propagated yet. Wait 2-3 minutes and try again.
3. **Error: "ResourceNotFound" when calling OpenAI**
   *Fix:* The identity works, but the AI endpoint URL is fake. Update the `Uri` in C# to point to a deployed Azure OpenAI instance.
4. **Error: Cannot create App Registration (Insufficient privileges)**
   *Fix:* Your Entra ID tenant restricts users from registering apps. Ask your Azure Administrator to grant you the "Application Developer" directory role.
5. **Error: Free tier App Service Plan fails to create**
   *Fix:* You already have an F1 plan in this region. You can only have one per region. Change the `$LOCATION` variable to a different region (e.g., `westus`).

---

## CLEANUP

Always clean up your resources to prevent unexpected billing or clutter.

```bash
# Delete the entire resource group and everything inside it
az group delete \
  --name $RG_NAME \
  --yes \
  --no-wait # Returns control to the terminal immediately while deletion happens in the background

# Verify deletion has started
az group list -o table
```
