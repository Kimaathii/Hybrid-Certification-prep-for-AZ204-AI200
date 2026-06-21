# F02 — Azure Hierarchy, ARM, and Infrastructure as Code

> **Foundation Document 02** | **Pages:** 14–18 | **Study Time:** 2 hours
> **Prerequisites:** F01 — How the Cloud Works: From Concept to Practice
> **What you will learn:** How Azure organises resources, how every Azure request is processed, how to define infrastructure in code, and how to govern and control costs in your subscription.

---

## How to Use This Document

This is not a lecture to skim — it is a working reference you will return to throughout every AZ-204 module. Every lab you complete in this course begins with the concepts in this document: creating resource groups, deploying resources, tagging them for cost tracking, and cleaning them up when you are done.

Read each section in order. When you see a code block, read every line and its annotation. When you see a checkpoint, pause and answer before continuing. By the end, you will deploy a real Azure resource using an ARM template from the command line.

---

## Section 1: The Azure Hierarchy in Practice

### The Problem

Imagine you are running a company with hundreds of employees, dozens of departments, and thousands of projects. Without an organisational chart, nobody knows who reports to whom, which budget pays for what, or how to find a specific project. Azure faces the same problem — millions of customers creating millions of resources. It needs an organisational hierarchy to keep everything structured, governed, and billable.

### The Four Levels

You saw this briefly in AZ-900 — now we go deeper because you will **use** this hierarchy in every single lab.

Think of Azure's hierarchy like a filing system in a large office:

- **Management Group** = the filing cabinet (holds multiple drawers together, sets policies that apply to everything inside)
- **Subscription** = a drawer in the cabinet (a billing boundary — everything inside gets charged to one credit card)
- **Resource Group** = a folder in the drawer (a logical container where you group related resources for a single project or environment)
- **Resource** = an individual document in the folder (one actual Azure service — a web app, a database, a storage account)

```
Management Group          ← Policy and access control boundary
  └── Subscription        ← Billing boundary (one invoice)
        └── Resource Group    ← Logical container (your daily workspace)
              └── Resource        ← An actual Azure service (web app, database, etc.)
```

💡 **KEY CONCEPT**
> Every Azure resource must live inside a Resource Group. Every Resource Group must live inside a Subscription. You cannot create a resource without these two layers.

### How You Actually Use This in Labs

Here is the practical reality for your AZ-204 journey:

1. **You will almost never touch Management Groups.** They are for enterprise administrators managing multiple subscriptions. You have one subscription.
2. **You have one Subscription.** It came with your Azure account. It is your billing boundary — every resource you create is charged to this subscription.
3. **You will create a Resource Group at the start of every single lab.** This is always Step 1.
4. **You will delete that Resource Group at the end of every lab.** This is always the last step.

Why? Because **deleting a Resource Group deletes everything inside it** — every web app, every database, every storage account, every function app. One command cleans up your entire lab. No orphaned resources. No surprise charges.

🚨 **EXAM ALERT**
> Deleting a Resource Group deletes ALL resources inside it — permanently and irreversibly. There is no "undo." This is both a powerful cleanup tool and a dangerous action if applied to the wrong group.

### Naming Conventions

In a professional environment, resources follow a naming pattern so anyone on the team can look at a resource name and immediately understand what it is. The pattern used throughout this course is:

```
{project}-{service}-{environment}
```

Examples:

| Resource Name | Project | Service | Environment |
|---|---|---|---|
| `az204-webapp-dev` | AZ-204 labs | Web App | Development |
| `az204-storage-prod` | AZ-204 labs | Storage Account | Production |
| `az204-cosmos-test` | AZ-204 labs | Cosmos DB | Testing |
| `az204-func-dev` | AZ-204 labs | Function App | Development |

For Resource Groups, we keep it simple:

```
az204-lab01-rg
az204-lab02-rg
az204-functions-rg
```

The `-rg` suffix tells anyone looking at it that this is a Resource Group, not a resource.

### Why `$RANDOM` Appears in CLI Commands

Many Azure resources need a **globally unique name** — meaning no other Azure customer in the world can have the same name. This is because the name becomes part of a public URL. For example, a storage account named `mystorage` gets the URL `mystorage.blob.core.windows.net`. If someone else already took that name, your command fails.

The solution is `$RANDOM` — a built-in shell variable (a placeholder that automatically generates a random number between 0 and 32,767 every time you use it). When you write `az204storage$RANDOM` in a Bash terminal, the shell replaces `$RANDOM` with a random number like `14582`, giving you `az204storage14582` — almost certainly unique.

📝 **NOTE**
> `$RANDOM` works in Bash (Linux, macOS, Azure Cloud Shell). In PowerShell on Windows, the equivalent is `$(Get-Random)`. All labs in this course use Bash syntax, which works in Azure Cloud Shell regardless of your local operating system.

### Creating a Resource Group — Your First CLI Command

Here is the exact command you will run at the start of nearly every lab:

```bash
az group create \
  --name az204-lab-rg \
  --location eastus \
  --tags Environment=dev Project=az204
```

Let us break down every piece of this command:

| Part | What It Does |
|---|---|
| `az` | The Azure CLI program — every Azure command starts with this |
| `group` | The service you are working with — resource groups |
| `create` | The action you want to perform — create a new one |
| `--name az204-lab-rg` | The name for your new resource group |
| `--location eastus` | The Azure region (physical datacenter location) where the group's metadata is stored. Resources inside can be in different regions, but best practice is to keep them in the same one. |
| `--tags Environment=dev Project=az204` | Key-value metadata labels attached to the resource group (more on tags in Section 5) |

The `\` (backslash) at the end of each line is just a line-continuation character — it tells the terminal "this command continues on the next line." It makes long commands easier to read. You could also write the entire command on one line without any backslashes.

After running this command, you will see output like:

```json
{
  "id": "/subscriptions/abc-123-def/resourceGroups/az204-lab-rg",
  "location": "eastus",
  "name": "az204-lab-rg",
  "tags": {
    "Environment": "dev",
    "Project": "az204"
  },
  "type": "Microsoft.Resources/resourceGroups"
}
```

This JSON output (JSON stands for JavaScript Object Notation — a structured text format that computers use to exchange data) confirms the resource group was created and shows its properties.

To verify your resource group exists:

```bash
az group list --output table
```

| Part | What It Does |
|---|---|
| `az group list` | List all resource groups in your subscription |
| `--output table` | Display the results as a human-readable table instead of raw JSON |

To delete a resource group when your lab is finished:

```bash
az group delete \
  --name az204-lab-rg \
  --yes \
  --no-wait
```

| Part | What It Does |
|---|---|
| `--yes` | Skip the "Are you sure?" confirmation prompt — useful in scripts |
| `--no-wait` | Do not wait for the deletion to finish before returning control to you. Deletion happens in the background. It may take 1–5 minutes. |

✅ **CHECKPOINT**
> 1. What are the four levels of the Azure hierarchy, from top to bottom?
> 2. What happens when you delete a resource group?
> 3. Why do lab commands use `$RANDOM` in resource names?
>
> **Answers:** 1. Management Group → Subscription → Resource Group → Resource. 2. All resources inside it are permanently deleted. 3. Because many Azure resources require globally unique names, and `$RANDOM` generates a random number to avoid name collisions.

---

## Section 2: Azure Resource Manager (ARM) — What It Really Is

### The Problem

You can create Azure resources using the portal (clicking buttons), the CLI (typing commands), PowerShell (scripting), the REST API (sending HTTP requests), or the SDKs (writing code). That is five different tools — but they all need to behave consistently. If you create a storage account via the portal, the CLI should see it. If you delete something via the SDK, the portal should reflect that instantly. How does Azure keep all of these tools in sync?

### The Receptionist Analogy

Imagine a large hospital with one reception desk at the front entrance. No matter how you arrive — by walking in, by calling, by using the hospital's mobile app, or by sending an email — every request goes through that same reception desk. The receptionist:

1. **Checks your identity** (are you a registered patient?)
2. **Validates your request** (does this appointment make sense?)
3. **Routes your request** to the right department (cardiology, radiology, etc.)
4. **Records everything** (the visit is logged in the system)
5. **Returns the result** (your appointment confirmation)

**Azure Resource Manager (ARM) is that reception desk.** ARM is a management layer — a single gateway that sits between you and all Azure services. Every tool you use (portal, CLI, PowerShell, SDKs, REST API) sends its request to ARM. ARM then authenticates you, validates the request, routes it to the right service, and returns the result.

```
  Portal ──┐
  CLI ─────┤
  SDKs ────┼──▶  ARM (Azure Resource Manager)  ──▶  Azure Services
  PowerShell─┤       ↑ Authenticates                  (Storage, Web Apps,
  REST API ──┘       ↑ Validates                       Functions, etc.)
                     ↑ Routes
                     ↑ Tracks state
```

💡 **KEY CONCEPT**
> ARM is not a tool you install or run. It is an invisible layer that every Azure interaction passes through. You never "use" ARM directly — but understanding it explains why Azure behaves consistently no matter which tool you choose.

### What ARM Does

| ARM Responsibility | What It Means |
|---|---|
| **Authenticates** | Checks your identity via Microsoft Entra ID (Azure's identity service). Are you who you say you are? |
| **Authorises** | Checks your permissions via RBAC (Role-Based Access Control). Are you allowed to do this action? |
| **Validates** | Checks if your request makes sense. Does this region exist? Is this SKU valid? Is the name available? |
| **Routes** | Sends the validated request to the correct Azure resource provider (e.g., `Microsoft.Storage` for storage accounts, `Microsoft.Web` for web apps) |
| **Tracks State** | Records the current state of every resource so Azure knows what exists, where, and with what configuration |
| **Returns Result** | Sends the response back to whatever tool you used |

### ARM Templates — Infrastructure Defined in a File

ARM does not only process one-off commands. You can hand ARM a complete **blueprint** of everything you want to create — written in a JSON file called an ARM template. ARM reads the blueprint, checks it, and creates everything described in it.

Think of it this way: instead of calling a builder and saying "first do the foundation, then the walls, then the roof" (giving individual commands), you hand the builder a complete architectural drawing and say "build exactly this." The drawing is the ARM template.

An ARM template (a JSON file that describes one or more Azure resources and their configuration) has six possible sections:

```json
{
  "$schema": "...",
  "contentVersion": "...",
  "parameters": { },
  "variables": { },
  "resources": [ ],
  "outputs": { }
}
```

| Section | Required? | Purpose |
|---|---|---|
| `$schema` | ✅ Yes | Tells ARM which version of the template language to use. You copy-paste this — never change it. |
| `contentVersion` | ✅ Yes | Your own version number for tracking changes. Convention: `"1.0.0.0"`. |
| `parameters` | ❌ Optional | Values the user provides at deployment time (like filling in blanks on a form). Example: resource name, location. |
| `variables` | ❌ Optional | Values calculated from parameters — defined once, reused throughout the template. Reduces repetition. |
| `resources` | ✅ Yes | The actual Azure resources to create. This is the heart of the template. |
| `outputs` | ❌ Optional | Values to display after deployment completes. Example: the URL of the created web app. |

### A Complete Working ARM Template

Here is a complete ARM template that creates a single storage account. Every line is annotated:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "The name for the storage account. Must be globally unique."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Azure region. Defaults to the resource group's region."
      }
    }
  },
  "variables": {
    "storageSku": "Standard_LRS"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2023-01-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[variables('storageSku')]"
      },
      "kind": "StorageV2",
      "properties": {
        "supportsHttpsTrafficOnly": true,
        "minimumTlsVersion": "TLS1_2"
      }
    }
  ],
  "outputs": {
    "storageEndpoint": {
      "type": "string",
      "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
    }
  }
}
```

Let us walk through every section in detail:

**Line-by-line breakdown:**

| Line / Section | What It Does |
|---|---|
| `"$schema": "https://..."` | Points to the ARM template schema definition. This tells ARM "I am using the 2019-04-01 template format." You always copy this verbatim. |
| `"contentVersion": "1.0.0.0"` | Your version label for this template. ARM does not enforce this — it is for your own tracking. Start at `1.0.0.0` and increment when you make changes. |
| `"parameters"` section | Defines inputs that can change between deployments. Here we have two: the storage account name (no default — the user must provide it) and the location (has a default — the resource group's own region). |
| `"type": "string"` | The data type of the parameter. Can be `string`, `int`, `bool`, `object`, `array`, or `secureString` (for passwords — never shown in logs). |
| `"defaultValue": "[resourceGroup().location]"` | A template function (code that runs inside the template). `resourceGroup()` returns information about the resource group you are deploying into. `.location` extracts its region. Square brackets `[ ]` mean "evaluate this expression." |
| `"metadata": { "description": "..." }` | A human-readable explanation. Shown in the Azure portal when deploying via the UI. |
| `"variables"` section | Values calculated once and reused. Here we store the SKU (pricing/redundancy tier) as a variable so we can reference it in multiple places without repeating it. |
| `"resources"` section | An array (a list) of resources to create. Each resource is a JSON object with specific required properties. |
| `"type": "Microsoft.Storage/storageAccounts"` | The resource provider and resource type. This tells ARM "I want a storage account." Every Azure resource has a type in the format `Provider/ResourceType`. |
| `"apiVersion": "2023-01-01"` | Which version of the resource provider API to use. Different versions support different properties. Use the latest stable version. |
| `"name": "[parameters('storageAccountName')]"` | The resource's name, pulled from the parameter the user provides. The `parameters()` function retrieves the value. |
| `"location": "[parameters('location')]"` | Where to create the resource, pulled from the location parameter. |
| `"sku": { "name": "[variables('storageSku')]" }` | The pricing tier, pulled from the variable we defined. `Standard_LRS` = Standard performance, Locally Redundant Storage (3 copies in one datacenter). |
| `"kind": "StorageV2"` | General Purpose v2 — the recommended storage account type for all scenarios. |
| `"supportsHttpsTrafficOnly": true` | Only allow encrypted connections. Security best practice. |
| `"minimumTlsVersion": "TLS1_2"` | Require TLS 1.2 (a security protocol for encrypted communication) as the minimum version. Older versions have known vulnerabilities. |
| `"outputs"` section | Values to display after deployment. Here we output the blob storage endpoint URL so the user knows where to access their storage. |
| `"reference(parameters('storageAccountName'))"` | A template function that retrieves the runtime properties of the created resource. `.primaryEndpoints.blob` navigates to the blob endpoint URL. |

🚨 **EXAM ALERT**
> ARM templates use **square brackets** `[ ]` to indicate expressions that ARM should evaluate. `"[parameters('name')]"` means "look up the parameter called 'name' and insert its value here." If you see square brackets in an ARM template, something is being calculated — it is not a literal string.

### Deploying an ARM Template

To deploy this template, you use a single CLI command:

```bash
az deployment group create \
  --resource-group az204-lab-rg \
  --template-file template.json \
  --parameters storageAccountName=az204store14582
```

| Part | What It Does |
|---|---|
| `az deployment group create` | Create a deployment at the resource group level (you can also deploy at subscription or management group level, but resource group is the most common) |
| `--resource-group az204-lab-rg` | Which resource group to deploy into. The resource group must already exist. |
| `--template-file template.json` | Path to your ARM template file on your local machine |
| `--parameters storageAccountName=az204store14582` | Provide values for template parameters. For parameters without a `defaultValue`, you must provide them here. |

After running this command, ARM:
1. Reads your template file
2. Authenticates you (are you logged in?)
3. Authorises you (do you have permission to create storage accounts in this resource group?)
4. Validates the template (is the JSON correct? Does the storage account name meet requirements?)
5. Creates the storage account
6. Returns the result, including any outputs you defined

### Viewing the Deployment in the Azure Portal

Every ARM deployment is recorded in the portal. To see it:

1. Open the **Azure Portal** (portal.azure.com)
2. Navigate to **Resource Groups**
3. Click on your resource group (e.g., `az204-lab-rg`)
4. In the left menu, click **Deployments**
5. You will see a list of all deployments — each one with a name, status (Succeeded / Failed), timestamp, and duration

Clicking on a specific deployment shows you:
- **Inputs** — the parameter values you provided
- **Outputs** — the values the template returned (like the storage endpoint URL)
- **Template** — the full ARM template that was deployed
- **Errors** — if the deployment failed, the detailed error message

This is invaluable for troubleshooting. If a lab deployment fails, the error details in the portal are often more informative than the CLI error message.

✅ **CHECKPOINT**
> 1. What is ARM's role in Azure?
> 2. Name the six sections of an ARM template.
> 3. What does `[parameters('storageAccountName')]` mean in a template?
>
> **Answers:** 1. ARM is the central management layer that every Azure request passes through — it authenticates, validates, routes, and tracks all operations. 2. `$schema`, `contentVersion`, `parameters`, `variables`, `resources`, `outputs`. 3. It is a template expression that retrieves the value of the parameter named "storageAccountName" and inserts it at that position.

---

## Section 3: Bicep — The Modern Alternative

### The Problem with ARM JSON

Look at the ARM template we just wrote. It is 35+ lines of JSON to create a single storage account. JSON was not designed for humans to write by hand — it was designed for machines to read. The curly braces, the square brackets, the commas, the quotation marks around every key — all of it is error-prone and hard to read. Imagine writing a template with 20 resources. The JSON file could easily be 500+ lines of deeply nested brackets.

### The Word Processor Analogy

Think of it this way:

- **ARM JSON** is like writing a letter using raw HTML code. You can do it, but you have to type `<p>` for every paragraph, `<b>` for every bold word, and `</body></html>` at the end. One missing tag and the whole page breaks.
- **Bicep** is like writing the same letter in a word processor (like Microsoft Word). You just type your content naturally, and the word processor generates the HTML behind the scenes.

Bicep is a **domain-specific language** (a programming language designed for one specific purpose — in this case, defining Azure resources) that compiles down to ARM JSON. When you deploy a Bicep file, Azure converts it to an ARM template behind the scenes and deploys it through the same ARM pipeline. You get the same result with cleaner, more readable code.

### The Same Storage Account in Bicep

Here is the exact same storage account — the one we created with 35+ lines of ARM JSON — written in Bicep:

```bicep
// Parameter: storage account name (must be provided at deployment time)
param storageAccountName string

// Parameter: location, defaults to the resource group's region
param location string = resourceGroup().location

// Variable: the SKU for the storage account
var storageSku = 'Standard_LRS'

// Resource: create a storage account
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName          // Use the parameter value
  location: location                // Use the parameter value
  sku: {
    name: storageSku                // Use the variable value
  }
  kind: 'StorageV2'                 // General Purpose v2
  properties: {
    supportsHttpsTrafficOnly: true  // HTTPS only
    minimumTlsVersion: 'TLS1_2'    // Minimum TLS 1.2
  }
}

// Output: the blob endpoint URL
output storageEndpoint string = storageAccount.properties.primaryEndpoints.blob
```

Notice the differences:
- No curly braces around every key-value pair
- No quotation marks around property names
- No `$schema` or `contentVersion` boilerplate
- Comments use `//` instead of not being supported at all (JSON does not support comments)
- The resource type and API version are combined in one line: `'Microsoft.Storage/storageAccounts@2023-01-01'`
- Referencing a parameter is just `storageAccountName` instead of `[parameters('storageAccountName')]`

### Side-by-Side Comparison

| Feature | ARM JSON | Bicep |
|---|---|---|
| **File extension** | `.json` | `.bicep` |
| **Syntax style** | JSON (verbose, curly braces everywhere) | Clean, concise, purpose-built |
| **Comments** | ❌ Not supported in JSON | ✅ `//` for single line, `/* */` for multi-line |
| **Parameter reference** | `[parameters('name')]` | `name` (just use the variable directly) |
| **Variable reference** | `[variables('sku')]` | `sku` (just use the variable directly) |
| **Resource type + API version** | Two separate properties | Combined: `'Type@Version'` |
| **Lines for one storage account** | ~35 lines | ~20 lines |
| **Schema boilerplate** | Required (`$schema`, `contentVersion`) | Not needed — Bicep handles it |
| **Deployed by** | ARM (directly) | Converted to ARM JSON, then deployed by ARM |
| **Learning curve** | Higher (JSON complexity) | Lower (cleaner syntax) |
| **What actually runs** | ARM processes the JSON | Bicep compiles to JSON → ARM processes it |

### Deploying a Bicep File

The deployment command is **exactly the same** — you just change the filename:

```bash
az deployment group create \
  --resource-group az204-lab-rg \
  --template-file main.bicep \
  --parameters storageAccountName=az204store14582
```

| Part | What It Does |
|---|---|
| `--template-file main.bicep` | Path to your Bicep file. The Azure CLI detects the `.bicep` extension and automatically compiles it to ARM JSON before deploying. |

Everything else is identical. ARM receives the same JSON either way.

📝 **NOTE**
> **AZ-204 exam context:** The exam does not test Bicep syntax deeply. You will not be asked to write Bicep code. However, you may see Bicep snippets in exam questions and need to recognise what they do. The key takeaway: Bicep and ARM JSON are two ways to express the same thing. Bicep is the newer, recommended approach, but ARM JSON templates remain fully supported and frequently tested.

✅ **CHECKPOINT**
> 1. What is the relationship between Bicep and ARM JSON?
> 2. Can you deploy a Bicep file using the same `az deployment group create` command?
>
> **Answers:** 1. Bicep compiles down to ARM JSON — they produce the same result. Bicep is a cleaner syntax that converts to ARM JSON behind the scenes. 2. Yes, the same command works for both. The CLI auto-detects the `.bicep` extension.

---

## Section 4: Infrastructure as Code — Why It Matters

### The Problem Without IaC

Imagine your team has a production environment that took three days to set up manually — clicking through the portal, configuring settings, creating resources one by one. Six months later, you need an identical environment for testing. What happens?

- **Nobody remembers all the steps.** Was it Standard_LRS or Standard_GRS? Did we enable HTTPS-only? What was the firewall rule?
- **The new environment is slightly different.** This is called **environment drift** — when environments that should be identical gradually become different because of manual, undocumented changes.
- **A disaster destroys your production environment.** You need to rebuild it from scratch. But the person who set it up originally left the company three months ago. No documentation exists.

This is the problem **Infrastructure as Code (IaC)** solves. IaC means defining your infrastructure — servers, databases, networks, storage accounts, everything — in text files (code) that can be version-controlled, reviewed, tested, and reused.

💡 **KEY CONCEPT**
> Infrastructure as Code (IaC) = defining your cloud infrastructure in files instead of clicking through a portal. The files become the single source of truth. You can recreate any environment by running the file. Nothing is forgotten, nothing drifts, nothing depends on one person's memory.

### Imperative vs Declarative — Two Approaches to IaC

There are two fundamentally different ways to tell a computer what you want. This distinction appears on the AZ-204 exam.

**Analogy: Ordering food**

- **Imperative** (how to do it): "Go to the fridge. Take out the bread. Put it in the toaster. Set it to medium. Wait 3 minutes. Take the toast out. Put butter on it. Bring it to me."
- **Declarative** (what you want): "I want buttered toast." The kitchen figures out how to make it.

**In Azure terms:**

| Approach | Style | Example Tool | How It Works |
|---|---|---|---|
| **Imperative** | "Do these steps in this order" | Azure CLI, PowerShell | You write step-by-step commands. You are responsible for the order, error handling, and knowing what already exists. |
| **Declarative** | "Here is the desired end state" | ARM templates, Bicep, Terraform | You describe what you want to exist. The tool figures out what needs to be created, updated, or left alone. |

**Imperative example — Azure CLI:**

```bash
# Step 1: Create a resource group
az group create --name az204-lab-rg --location eastus

# Step 2: Create a storage account inside it
az storage account create \
  --name az204store14582 \
  --resource-group az204-lab-rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2
```

You gave explicit, ordered instructions. If the storage account already exists, this command will fail (or update it, depending on the command). You have to manage the logic.

**Declarative example — ARM template:**

```json
{
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "az204store14582",
      "location": "eastus",
      "sku": { "name": "Standard_LRS" },
      "kind": "StorageV2"
    }
  ]
}
```

You described the desired state. ARM figures out the rest:
- If the storage account does not exist → create it
- If it exists but the configuration is different → update it
- If it already exists and matches → do nothing

This is called **idempotent** deployment — a word meaning "you can run it multiple times and get the same result." Run the ARM template ten times and you still have one storage account with the correct configuration.

🚨 **EXAM ALERT**
> **ARM templates and Bicep = declarative.** You describe the desired state; ARM figures out the steps.
> **CLI commands and PowerShell scripts = imperative.** You write the steps yourself.
> The exam will describe a scenario and ask which approach is being used. If the question mentions "desired state" or "template," it is declarative. If it mentions "step-by-step commands" or "script that runs commands in order," it is imperative.

### Why Declarative Usually Wins for Infrastructure

| Benefit | Imperative (CLI) | Declarative (ARM/Bicep) |
|---|---|---|
| **Repeatability** | Must handle "already exists" errors | Idempotent — safe to rerun |
| **Documentation** | Script IS the documentation, but can be unclear | Template IS the documentation, very explicit |
| **Version control** | Can be version-controlled | Can be version-controlled |
| **Environment consistency** | Depends on execution order and state | Guaranteed same result every time |
| **Complexity at scale** | Gets hard with 50+ resources and dependencies | Handles dependencies automatically |

📝 **NOTE**
> Both approaches have their place. CLI is great for quick one-off tasks (creating a resource group, deleting a test resource). ARM/Bicep is better for reproducible, multi-resource deployments. In this course, we use CLI for simple tasks and ARM/Bicep when we need infrastructure defined in a file.

---

## Section 5: Azure Tags — Governance in Practice

### The Problem

After a month of labs, your subscription has dozens of resources scattered across multiple resource groups. Your company has multiple teams sharing one subscription. The monthly Azure bill arrives: $247. But who spent what? Which resources belong to which project? Which ones are for production and which are just test environments someone forgot to delete?

### What Tags Are

Tags are **key-value metadata** (descriptive labels) that you attach to Azure resources. They are like sticky notes on items in your office — the sticky note does not change what the item does, but it tells you important information about it.

Think of tags as labels on boxes in a warehouse:

- Box 1: `Department: Marketing`, `Project: Campaign2024`, `Owner: Sarah`
- Box 2: `Department: Engineering`, `Project: AZ204`, `Owner: You`

The labels do not change what is inside the boxes. But they let you find, sort, filter, and report on boxes by any label.

💡 **KEY CONCEPT**
> Tags are metadata only. They do not affect how a resource works, how it performs, or what it can do. They exist purely for organisation, cost tracking, automation, and governance.

### Common Tags Used in Practice

| Tag Key | Example Value | Purpose |
|---|---|---|
| `Environment` | `dev`, `test`, `staging`, `prod` | Identify which environment a resource belongs to |
| `Project` | `az204`, `website-redesign` | Track costs per project |
| `Owner` | `sarah@company.com` | Know who to contact about a resource |
| `CostCenter` | `CC-1234` | Map resources to accounting cost centres |
| `ManagedBy` | `terraform`, `manual` | Track how the resource was created |
| `ExpiryDate` | `2024-12-31` | Flag resources for cleanup after a date |

### Adding Tags via CLI

To add tags to an existing resource:

```bash
az resource tag \
  --tags Environment=dev Project=az204 Owner=student \
  --resource-group az204-lab-rg \
  --name az204store14582 \
  --resource-type Microsoft.Storage/storageAccounts
```

| Part | What It Does |
|---|---|
| `az resource tag` | The command to add or update tags on a resource |
| `--tags Environment=dev Project=az204 Owner=student` | The key-value pairs to set. Multiple tags separated by spaces. No quotes needed around simple values. |
| `--resource-group az204-lab-rg` | Which resource group the resource is in |
| `--name az204store14582` | The name of the specific resource to tag |
| `--resource-type Microsoft.Storage/storageAccounts` | The type of resource (required because names alone are not unique across resource types) |

To add tags when creating a resource (most `az create` commands support `--tags`):

```bash
az storage account create \
  --name az204store14582 \
  --resource-group az204-lab-rg \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --tags Environment=dev Project=az204
```

### Filtering by Tag

**In the Azure Portal:**
1. Go to **All Resources** in the left menu
2. Click **Add filter**
3. Select **Tag** as the filter type
4. Choose a tag key (e.g., `Environment`) and value (e.g., `dev`)
5. The list now shows only resources with that tag

**Using the CLI:**

```bash
az resource list \
  --tag Environment=dev \
  --output table
```

| Part | What It Does |
|---|---|
| `az resource list` | List all resources in the subscription |
| `--tag Environment=dev` | Filter to only resources tagged with `Environment=dev` |
| `--output table` | Display results as a readable table |

🚨 **EXAM ALERT**
> Tags do **NOT** affect resource functionality. They are metadata only. A storage account tagged `Environment=prod` behaves identically to one tagged `Environment=dev`. Tags are for humans and automation tools, not for the resource itself. The exam may include a distractor option suggesting that tags control resource behaviour — they do not.

📝 **NOTE**
> Tags are not inherited. Tagging a resource group does NOT automatically tag the resources inside it. Each resource must be tagged individually (or via a script/policy). Azure Policy can enforce tagging rules — see the next section.

✅ **CHECKPOINT**
> 1. What are tags in Azure?
> 2. Do tags affect how a resource behaves?
> 3. If you tag a resource group, are the resources inside it automatically tagged?
>
> **Answers:** 1. Key-value metadata labels attached to resources for organisation, cost tracking, and governance. 2. No — tags are metadata only and have zero effect on functionality. 3. No — tags are not inherited. Each resource must be tagged individually.

---

## Section 6: Azure Policy and Resource Locks

### Azure Policy — Enforcing Standards

#### The Problem

Your team agrees on standards: "Every resource must have an Environment tag," "Nobody should create expensive GPU virtual machines in the dev subscription," "All storage accounts must use HTTPS only." But agreements are just words. People forget. New team members do not know the rules. How do you enforce these standards automatically?

#### What Azure Policy Is

Azure Policy is like a building inspector. Before any construction (resource creation or modification) is finalised, the inspector checks it against the building code (your policies). If the construction violates the code, the inspector can:

- **Block it** (deny the creation entirely)
- **Flag it** (allow the creation but mark it as non-compliant in a report)
- **Fix it automatically** (add the missing tag, enable the missing setting)

Azure Policy works at the ARM level — every resource creation or modification passes through ARM, and policies are evaluated at that point.

#### Common Policy Examples

| Policy | Effect | What It Does |
|---|---|---|
| "Require Environment tag on all resources" | Deny | Blocks resource creation if the `Environment` tag is missing |
| "Allowed resource types" | Deny | Only allows specific resource types (e.g., storage accounts, web apps) — blocks everything else |
| "Allowed locations" | Deny | Restricts resource creation to specific regions (e.g., East US, West Europe) |
| "Storage accounts must use HTTPS" | Deny or Audit | Blocks or flags storage accounts not configured for HTTPS-only |
| "Inherit Environment tag from resource group" | Modify | Automatically copies the resource group's `Environment` tag to new resources |

#### Developer Context: When Policy Blocks Your Lab

Here is a scenario you may encounter: you run a lab command to create a virtual machine, and it fails with an error like:

```
Resource 'myvm' was disallowed by policy. Policy: 'Allowed resource types'.
```

This means an Azure Policy is in effect on your subscription that prevents creating that resource type. This is not a bug — it is governance working as designed. If this happens:

1. Check with your subscription administrator (or instructor) about which resource types are allowed
2. If you are using a free trial or personal subscription, you probably do not have restrictive policies — the error is more likely a different issue
3. Check the exact error message — it always names the policy that blocked the action

### Resource Locks — Preventing Accidental Damage

#### The Problem

You have a production database that took weeks to configure. A new team member, unfamiliar with the subscription, sees the resource group and runs `az group delete` thinking it is a test environment. Everything is gone. Permanently. Resource locks prevent this.

#### What Resource Locks Are

A resource lock is like a safety latch on a window. The window still works (you can look through it, open it slightly), but the latch prevents it from being opened all the way or being removed.

There are two types of locks:

| Lock Type | What It Prevents | What It Allows |
|---|---|---|
| **CanNotDelete** | Deleting the resource | Reading and modifying the resource. You can change settings, update configuration — but you cannot delete it. |
| **ReadOnly** | Deleting the resource AND modifying it | Reading the resource only. No changes of any kind — not even updating tags or settings. |

💡 **KEY CONCEPT**
> **CanNotDelete** = "You can change it but not destroy it."
> **ReadOnly** = "You can look but not touch."

#### Applying a Lock via CLI

```bash
az lock create \
  --name DoNotDelete \
  --resource-group az204-lab-rg \
  --lock-type CanNotDelete \
  --notes "Protect lab resources during exam prep"
```

| Part | What It Does |
|---|---|
| `az lock create` | Create a new resource lock |
| `--name DoNotDelete` | A name for the lock (your choice — make it descriptive) |
| `--resource-group az204-lab-rg` | The resource group to lock. You can also lock individual resources. |
| `--lock-type CanNotDelete` | The type of lock. Options: `CanNotDelete` or `ReadOnly`. |
| `--notes "..."` | Optional description explaining why the lock exists |

To remove a lock (you must remove it before you can delete the locked resource):

```bash
az lock delete \
  --name DoNotDelete \
  --resource-group az204-lab-rg
```

🚨 **EXAM ALERT**
> A **ReadOnly** lock on a storage account prevents you from listing the access keys — because listing keys is considered a modification operation (it generates new values). This trips up many exam candidates. If you cannot retrieve connection strings for a resource, check if a ReadOnly lock is in place.

📝 **NOTE**
> Locks are applied at the resource or resource group level and are inherited by resources within the group. A lock on a resource group applies to all resources inside it. To delete a locked resource, you must first remove the lock — having Owner or Contributor permissions is not enough to bypass a lock.

---

## Section 7: Cost Management in Practice

### Why This Matters for AZ-204 Students

Every lab in this course creates real Azure resources that cost real money. Most services have free tiers or free amounts that cover lab work, but if you forget to delete resources after a lab, charges accumulate. A forgotten App Service Plan running on Standard tier costs about $73/month. A forgotten Cosmos DB instance can cost $24/month even with no data.

The single most important habit in this course: **always delete your resource group when you finish a lab.**

### Setting Up a Budget Alert — Step by Step

A budget alert sends you an email when your spending approaches a threshold you set. Here is how to configure one:

1. Open the **Azure Portal** (portal.azure.com)
2. Search for **"Cost Management"** in the top search bar
3. Click **Cost Management** in the results
4. In the left menu, click **Budgets**
5. Click **+ Add**
6. Configure the budget:
   - **Name:** `az204-monthly-limit`
   - **Reset period:** Monthly (resets each calendar month)
   - **Amount:** `10` (ten dollars — appropriate for lab work with proper cleanup)
7. Click **Next** to configure alerts:
   - **Alert condition 1:** Actual spending reaches **50%** of budget ($5) → sends email
   - **Alert condition 2:** Actual spending reaches **80%** of budget ($8) → sends email
   - **Alert condition 3:** Actual spending reaches **100%** of budget ($10) → sends email
   - **Alert recipients:** Enter your email address
8. Click **Create**

💰 **COST WARNING**
> Set a $10/month budget alert now, before starting any labs. This is your safety net. If you consistently clean up resources after each lab, you should stay well under $10/month. If you receive a 50% alert, check which resources are still running and delete any you are not actively using.

### Cost Analysis — Finding Where Your Money Goes

To see which resources are costing you money:

1. Open **Cost Management** in the portal
2. Click **Cost Analysis** in the left menu
3. The default view shows total spending over time
4. To filter by resource group: click **Add filter** → select **Resource group** → choose the group you want to examine
5. To see per-resource costs: change the **Group by** dropdown to **Resource**
6. To see daily costs: change the time range at the top to **Last 7 days** and the granularity to **Daily**

This tells you exactly which resources are consuming your budget and helps you identify anything you forgot to clean up.

### Free Tier Limits for AZ-204 Services

These are the free amounts included in an Azure subscription. As long as your lab usage stays within these limits, you will not be charged for these specific services:

| Service | Free Tier / Free Amount | What It Means |
|---|---|---|
| **Azure Functions (Consumption)** | 1,000,000 executions/month + 400,000 GB-seconds | Your Function App runs free for up to 1 million triggers per month. Lab work will use a tiny fraction of this. |
| **Cosmos DB** | 1,000 RU/s + 25 GB storage (free tier — must be enabled at creation) | One free Cosmos DB account per subscription. Must select "Apply Free Tier Discount" when creating. |
| **Key Vault** | 10,000 transactions/month (Standard tier) | Reading and writing secrets, keys, and certificates. Labs will use a few dozen transactions at most. |
| **Event Grid** | 100,000 operations/month | Publishing and delivering events. Labs will use a few hundred at most. |
| **App Service** | F1 (Free) tier — 60 minutes CPU/day, 1 GB RAM, 1 GB storage | Very limited but enough for testing. Some labs require Standard tier (S1) — these have cost callouts. |
| **Azure Container Registry** | Basic tier ≈ $0.17/day | No true free tier. Use Basic and delete after labs. |
| **Azure Container Instances** | ≈ $0.0025/vCPU-second | Pay per second while running. Stop or delete after labs. |
| **Blob Storage** | First 5 GB free on LRS Hot tier | Labs typically use a few MB. Well within free limits. |
| **Service Bus** | Basic tier: $0.05/million operations | Near-free for lab volumes. |
| **API Management** | Consumption tier: first 1,000,000 calls/month free | Sufficient for all lab work. |

📝 **NOTE**
> "Free tier" and "free trial credits" are different things. Free tiers are permanent limits that never expire. Free trial credits ($200 for 30 days) are one-time credits that expire. After your trial credits expire, free tier services continue to work at no cost — but paid services will stop unless you convert to a pay-as-you-go subscription.

🚨 **EXAM ALERT**
> The exam does not test cost management details, but understanding pricing models (consumption vs reserved vs per-transaction) can help eliminate wrong answers. If a question asks about reducing costs for a service that charges per-execution, "reducing the number of executions" or "using a reserved plan" are typical correct answers.

✅ **CHECKPOINT**
> 1. What is the recommended monthly budget alert amount for AZ-204 lab work?
> 2. How do you view costs filtered by resource group in the portal?
> 3. How many Azure Functions executions per month are free on the Consumption plan?
>
> **Answers:** 1. $10/month. 2. Cost Management → Cost Analysis → Add filter → Resource group. 3. 1,000,000 executions per month.

---

## 🔬 Hands-On Exercise: Deploy a Storage Account Using an ARM Template

### What You Will Do

In this exercise, you will:
1. Create a resource group
2. Create an ARM template file
3. Deploy the template using the Azure CLI
4. View the deployment in the Azure portal
5. Clean up by deleting the resource group

**Estimated time:** 20 minutes
**Cost:** $0 (storage account with no data, deleted within the hour)

### Prerequisites

- Azure CLI installed and logged in (`az login`)
- A text editor (Visual Studio Code recommended, but Notepad works)
- An active Azure subscription

### Step 1: Create a Resource Group

Open your terminal (Azure Cloud Shell, or your local terminal with Azure CLI installed) and run:

```bash
az group create \
  --name az204-arm-lab-rg \
  --location eastus
```

**Expected output:** A JSON object with `"provisioningState": "Succeeded"`.

### Step 2: Create the ARM Template File

Create a new file named `storage-template.json` with the following content. You can use any text editor — Visual Studio Code, Notepad, or the Cloud Shell's built-in editor (`code storage-template.json`).

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 24,
      "metadata": {
        "description": "The name for the storage account. Must be globally unique, 3-24 characters, lowercase letters and numbers only."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Azure region for the storage account. Defaults to the resource group location."
      }
    },
    "environment": {
      "type": "string",
      "defaultValue": "dev",
      "allowedValues": [
        "dev",
        "test",
        "prod"
      ],
      "metadata": {
        "description": "The environment tag value. Restricted to dev, test, or prod."
      }
    }
  },
  "variables": {
    "storageSku": "Standard_LRS",
    "storageKind": "StorageV2"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2023-01-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "tags": {
        "Environment": "[parameters('environment')]",
        "Project": "az204",
        "ManagedBy": "arm-template"
      },
      "sku": {
        "name": "[variables('storageSku')]"
      },
      "kind": "[variables('storageKind')]",
      "properties": {
        "supportsHttpsTrafficOnly": true,
        "minimumTlsVersion": "TLS1_2",
        "allowBlobPublicAccess": false
      }
    }
  ],
  "outputs": {
    "storageAccountName": {
      "type": "string",
      "value": "[parameters('storageAccountName')]"
    },
    "blobEndpoint": {
      "type": "string",
      "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
    },
    "deploymentResourceGroup": {
      "type": "string",
      "value": "[resourceGroup().name]"
    }
  }
}
```

**What is new in this template compared to the earlier example:**

| Addition | Purpose |
|---|---|
| `"minLength": 3, "maxLength": 24` | Parameter validation — ARM rejects names shorter than 3 or longer than 24 characters before even trying to create the resource |
| `"allowedValues": ["dev", "test", "prod"]` | Parameter validation — the environment parameter only accepts these three values |
| `"tags"` on the resource | Applies metadata tags so we can identify this resource later |
| `"allowBlobPublicAccess": false` | Security best practice — prevents anonymous public access to blob containers |
| Multiple outputs | Returns the storage account name, blob endpoint URL, and resource group name after deployment |

### Step 3: Deploy the Template

Run the following command. Replace `YOURNAME` with a unique lowercase string (your initials and a few numbers work well):

```bash
az deployment group create \
  --resource-group az204-arm-lab-rg \
  --template-file storage-template.json \
  --parameters storageAccountName=az204labYOURNAME
```

For example, if your name is Alex:

```bash
az deployment group create \
  --resource-group az204-arm-lab-rg \
  --template-file storage-template.json \
  --parameters storageAccountName=az204labalex2024
```

**Expected output:** A large JSON object. Look for:
- `"provisioningState": "Succeeded"` — the deployment worked
- In the `"outputs"` section, you will see:
  - `blobEndpoint`: the URL where blobs would be stored (e.g., `https://az204labalex2024.blob.core.windows.net/`)
  - `deploymentResourceGroup`: `az204-arm-lab-rg`

If you get an error like `"StorageAccountAlreadyTaken"`, the name is not unique — try adding more numbers.

### Step 4: Verify in the Azure Portal

1. Open **portal.azure.com**
2. Navigate to **Resource Groups** → click **az204-arm-lab-rg**
3. You should see your storage account listed as a resource
4. In the left menu of the resource group, click **Deployments**
5. You will see a deployment entry (named something like `storage-template`). Click it.
6. Explore the tabs:
   - **Overview** — status, duration, timestamp
   - **Inputs** — the parameter values you provided
   - **Outputs** — the values the template returned (blob endpoint, resource group name)
   - **Template** — the full ARM JSON that was deployed

✅ **CHECKPOINT**
> Confirm you can see:
> - [ ] The storage account in the resource group
> - [ ] The deployment entry under "Deployments"
> - [ ] The outputs showing the blob endpoint URL
> - [ ] The tags (Environment=dev, Project=az204, ManagedBy=arm-template) on the storage account

### Step 5: Clean Up

Delete the resource group and everything inside it:

```bash
az group delete \
  --name az204-arm-lab-rg \
  --yes \
  --no-wait
```

Verify it is being deleted (it may take 1–2 minutes to fully remove):

```bash
az group list --output table
```

The resource group should no longer appear in the list (or show as "Deleting").

🎉 **Congratulations!** You just deployed your first Azure resource using Infrastructure as Code. Every lab in this course follows this same pattern: create a resource group, deploy resources (via CLI commands or ARM templates), verify in the portal, and clean up.

---

## 🔗 Module Connections

| Connection | Where It Leads |
|---|---|
| **Resource Groups** | Every module lab (M01–M13) begins with creating a resource group |
| **ARM Templates** | Module 01 (App Service), Module 02 (Functions), Module 03 (Containers) — all have ARM template deployment options |
| **Tags** | Module 07 (Key Vault & App Configuration) — tags used for cost tracking across secure resources |
| **Cost Management** | Every module lab includes a cost callout with estimated charges |
| **IaC concepts** | Module 01 (App Service deployment methods include ARM/Bicep) |
| **Azure Policy** | Module 07 (Key Vault policies), Module 06 (Managed Identity RBAC) |
| **Naming conventions** | Used consistently across all 13 module labs |

---

## What We Covered — Checklist

Use this checklist to confirm you understand every concept in this document. If any item is unclear, re-read that section before moving on.

- [ ] **Azure Hierarchy:** I can name the four levels (Management Group → Subscription → Resource Group → Resource) and explain the purpose of each
- [ ] **Resource Groups in Practice:** I understand that deleting a resource group deletes all resources inside it, and that every lab starts with creating one
- [ ] **Naming Conventions:** I can follow the `{project}-{service}-{environment}` pattern and explain why `$RANDOM` is used for globally unique names
- [ ] **ARM (Azure Resource Manager):** I can explain that ARM is the central management layer that every Azure tool communicates through, and I can describe what ARM does (authenticate, validate, route, track)
- [ ] **ARM Templates:** I can identify the six sections of an ARM template (`$schema`, `contentVersion`, `parameters`, `variables`, `resources`, `outputs`) and explain what each does
- [ ] **ARM Template Expressions:** I understand that square brackets `[ ]` indicate expressions (e.g., `[parameters('name')]`) and can read basic template functions
- [ ] **Deploying ARM Templates:** I can use `az deployment group create` with `--resource-group`, `--template-file`, and `--parameters` flags
- [ ] **Bicep:** I understand that Bicep is a cleaner syntax that compiles to ARM JSON, and I can recognise Bicep code in exam questions
- [ ] **Infrastructure as Code:** I can explain why IaC matters (repeatability, no drift, no forgotten steps) and I know the difference between imperative (CLI) and declarative (ARM/Bicep)
- [ ] **Imperative vs Declarative:** I know that ARM templates = declarative, CLI commands = imperative, and I can identify which is which in exam scenarios
- [ ] **Azure Tags:** I know that tags are key-value metadata that do NOT affect resource functionality, and I can add and filter by tags using the CLI
- [ ] **Azure Policy:** I understand that policies enforce standards (e.g., require tags, restrict resource types) and that a policy can block resource creation in labs
- [ ] **Resource Locks:** I can distinguish between CanNotDelete (prevents deletion, allows modification) and ReadOnly (prevents all changes), and I know that a ReadOnly lock on a storage account prevents listing access keys
- [ ] **Cost Management:** I have set up a $10/month budget alert and I know how to use Cost Analysis to filter costs by resource group
- [ ] **Free Tier Limits:** I know the free amounts for Functions (1M executions), Cosmos DB (1000 RU/s + 25 GB), Key Vault (10K ops), and Event Grid (100K ops)
- [ ] **Hands-On Deployment:** I have successfully deployed a storage account using an ARM template and viewed the deployment in the Azure portal
