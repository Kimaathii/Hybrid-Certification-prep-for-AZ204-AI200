# Module 10 Lab: Securely Managing AI Keys and Feature Flags

| Module | Lab Title | Prerequisites | Estimated Time | Cost Estimate | What You Build | What You Learn |
|--------|-----------|---------------|----------------|---------------|----------------|----------------|
| 10 | Securely Managing AI Keys & Feature Flags | Active Azure Subscription, Azure CLI installed, .NET 8 SDK | 45 Minutes | ~$0.10 | A .NET Console App that securely fetches an AI API key and toggles an AI feature using remote configuration. | Azure Key Vault, App Configuration, `DefaultAzureCredential`, Feature Flags. |

💰 **COST WARNING**
> This lab uses Azure Key Vault and Azure App Configuration on standard/free tiers. Estimated cost: < $0.10 if cleaned up within 24 hours.

---

## Part 1: Create a Key Vault and App Configuration Store

First, we need to create the Azure resources to hold our secrets and settings.

1. Open your terminal and log in to Azure:
```bash
az login
```

2. Create a Resource Group to hold our lab resources:
```bash
az group create \
  --name rg-az204-secrets-lab \
  --location eastus
```
* `--name` The name of the resource group.
* `--location` The datacenter region.

3. Create the Azure Key Vault. (Note: Key Vault names must be globally unique, so add some random numbers at the end).
```bash
az keyvault create \
  --name kv-az204-ai-12345 \
  --resource-group rg-az204-secrets-lab \
  --location eastus \
  --enable-rbac-authorization false
```
* `--name` The globally unique name of your vault. Change `12345` to your own numbers.
* `--enable-rbac-authorization false` For this lab, we will use legacy Access Policies because they are faster to set up locally for beginners.

4. Create the Azure App Configuration store (also needs a globally unique name).
```bash
az appconfig create \
  --name appcs-az204-ai-12345 \
  --resource-group rg-az204-secrets-lab \
  --location eastus \
  --sku free
```
* `--sku free` The pricing tier. The free tier allows 1,000 requests per day.

---

## Part 2: Store a Fake Azure OpenAI API Key

Now we will simulate having an expensive AI API key and store it securely.

1. Add a secret to your Key Vault:
```bash
az keyvault secret set \
  --vault-name kv-az204-ai-12345 \
  --name "OpenAIApiKey" \
  --value "sk-fake-ai-key-do-not-share-999"
```
* `--vault-name` The name of the vault you just created.
* `--name` The identifier for the secret.
* `--value` The actual secret string we want to protect.

**Expected Output:**
```json
{
  "attributes": {
    "created": "2023-10-25T12:00:00+00:00",
    "enabled": true
  },
  "id": "https://kv-az204-ai-12345.vault.azure.net/secrets/OpenAIApiKey/xyz123",
  "name": "OpenAIApiKey",
  "value": "sk-fake-ai-key-do-not-share-999"
}
```

✅ **CHECKPOINT**
> Your fake API key is now stored in the cloud. It is completely missing from your local computer files.

---

## Part 3: Create a .NET Console App

Let's build an app to fetch this secret.

1. Create a new .NET console application:
```bash
dotnet new console -n SecureAIApp
cd SecureAIApp
```

2. Install the required Azure SDK packages:
```bash
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Secrets
dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
dotnet add package Microsoft.FeatureManagement
```
* `Azure.Identity` Handles logging in to Azure securely.
* `Azure.Security.KeyVault.Secrets` Allows code to read Key Vault.
* `Microsoft.Extensions...` Allows code to read App Configuration.
* `Microsoft.FeatureManagement` Adds Feature Flag capabilities.

---

## Part 4: Fetch the Key Securely using DefaultAzureCredential

We will use `DefaultAzureCredential`. This magical tool automatically checks if you are logged in via `az login` locally, and uses those credentials to access the vault!

1. Open `Program.cs` and replace the contents with the following code:

```csharp
using System;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

class Program
{
    static void Main(string[] args)
    {
        // Replace with your actual Key Vault URL
        string keyVaultUrl = "https://kv-az204-ai-12345.vault.azure.net/";

        // 1. Authenticate using the local developer's Azure CLI credentials
        var credential = new DefaultAzureCredential();

        // 2. Connect to the Key Vault
        var client = new SecretClient(new Uri(keyVaultUrl), credential);

        // 3. Fetch the secret securely at runtime
        Console.WriteLine("Fetching AI API Key from Vault...");
        KeyVaultSecret secret = client.GetSecret("OpenAIApiKey");

        Console.WriteLine($"Success! The API Key is: {secret.Value}");
        Console.WriteLine("Notice how this key is nowhere in our source code!");
    }
}
```

2. Run the application:
```bash
dotnet run
```

**Expected Output:**
```
Fetching AI API Key from Vault...
Success! The API Key is: sk-fake-ai-key-do-not-share-999
Notice how this key is nowhere in our source code!
```

---

## Part 5: Implement a Feature Flag

Let's imagine we are building a new "AI Chat" feature, but we want to be able to turn it off instantly if the AI starts hallucinating. 

1. Go back to your terminal and create a Feature Flag in App Configuration:
```bash
az appconfig feature set \
  --name appcs-az204-ai-12345 \
  --feature "EnableAIChat" \
  --yes
```
* `--feature` The name of the flag.
* `--yes` Automatically sets the flag to 'On' without prompting.

2. Get the connection string for your App Configuration (you need this to connect the app):
```bash
az appconfig credential list \
  --name appcs-az204-ai-12345 \
  --query "[?name=='Primary'].connectionString" \
  --output tsv
```
*(Copy the output string. It looks like: `Endpoint=https://...;Id=...;Secret=...`)*

3. Update `Program.cs` to include Feature Management. (Replace your entire `Program.cs` with this):

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Extensions.Configuration;
using Microsoft.FeatureManagement;
using Microsoft.Extensions.DependencyInjection;

class Program
{
    static async Task Main(string[] args)
    {
        // Replace with your App Config Connection String
        string appConfigConnectionString = "Endpoint=https://appcs-az204-ai-12345.azconfig.io;Id=...;Secret=...";

        // 1. Build configuration and connect to Azure App Configuration
        var builder = new ConfigurationBuilder();
        builder.AddAzureAppConfiguration(options =>
        {
            options.Connect(appConfigConnectionString)
                   .UseFeatureFlags(); // Tell it to load feature flags
        });
        var config = builder.Build();

        // 2. Setup Dependency Injection for Feature Management
        var services = new ServiceCollection();
        services.AddSingleton<IConfiguration>(config)
                .AddFeatureManagement();
        
        using var serviceProvider = services.BuildServiceProvider();
        var featureManager = serviceProvider.GetRequiredService<IFeatureManager>();

        // 3. Check if the AI feature is turned on
        Console.WriteLine("Checking AI Feature Status...");
        
        if (await featureManager.IsEnabledAsync("EnableAIChat"))
        {
            Console.WriteLine("🤖 AI Chat is ENABLED! Connecting to Azure OpenAI...");
        }
        else
        {
            Console.WriteLine("🛑 AI Chat is DISABLED. Showing standard UI.");
        }
    }
}
```

4. Run the app:
```bash
dotnet run
```
**Expected Output:** `🤖 AI Chat is ENABLED!...`

5. **The Magic Step:** Turn off the feature in Azure without touching the code.
```bash
az appconfig feature disable \
  --name appcs-az204-ai-12345 \
  --feature "EnableAIChat" \
  --yes
```

6. Run the app again immediately:
```bash
dotnet run
```
**Expected Output:** `🛑 AI Chat is DISABLED...`

✅ **CHECKPOINT**
> You just changed the behavior of your application remotely without recompiling or redeploying your code!

---

## Troubleshooting

1. **Error: "DefaultAzureCredential failed to retrieve a token"**
   - *Fix:* You are not logged into the Azure CLI. Run `az login` in your terminal.
2. **Error: "Access Denied / Forbidden" when accessing Key Vault**
   - *Fix:* Your CLI user doesn't have an Access Policy on the vault. You may need to run `az keyvault set-policy --name kv-az204-ai-12345 --upn your-email@domain.com --secret-permissions get list`.
3. **Error: "ResourceGroupNameInvalid"**
   - *Fix:* Ensure you typed the resource group name correctly in all commands.

---

## Cleanup

Always clean up to prevent recurring costs.

1. Delete the resource group:
```bash
az group delete \
  --name rg-az204-secrets-lab \
  --yes \
  --no-wait
```
* `--yes` Skips the confirmation prompt.
* `--no-wait` Returns you to the terminal immediately while Azure deletes things in the background.

2. **Purge the Key Vault (Important)**
Key Vaults have "soft delete" enabled by default. You cannot recreate a vault with the same name unless you purge the old one.
```bash
az keyvault purge --name kv-az204-ai-12345
```

3. Confirm deletion:
```bash
az group list -o table
```
