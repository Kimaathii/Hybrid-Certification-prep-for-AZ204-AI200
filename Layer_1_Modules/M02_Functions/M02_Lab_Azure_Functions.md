# Module 2 — Azure Functions: Lab Guide

---

| | |
|---|---|
| **Module** | M02 — Azure Functions |
| **Lab Title** | Build, Test, and Deploy Azure Functions with HTTP and Queue Triggers |
| **Prerequisites** | F01–F06 complete, Azure CLI installed, .NET 8 SDK installed, VS Code installed |
| **Estimated Time** | 90–120 minutes |
| **Cost Estimate** | ≈ $0.00–$0.01 (see cost callout below) |
| **What You Build** | A Function App with an HTTP-triggered function that writes to a Queue, and a Queue-triggered function that consumes messages. You test locally, deploy to Azure, and verify in the portal. |
| **What You Learn** | Function project structure, triggers, output bindings, local development with `func start`, Azure deployment with `func azure functionapp publish`, queue message flow, and log viewing. |

> 💰 **COST WARNING**
>
> This lab uses the **Consumption plan**, which includes **1 million free executions per month** and **400,000 GB-seconds of compute per month**. You will use a tiny fraction of this during the lab. The only billable resource is a **Storage Account** (≈ $0.01/month for the small amount of data stored). **Total estimated cost: $0.00–$0.01** if you clean up within 2 hours. Always run the cleanup commands at the end.

---

## Part 1: Install Azure Functions Core Tools

Azure Functions Core Tools is the command-line tool that lets you create, run, and test Azure Functions on your local computer — without deploying to Azure. Think of it as a mini Azure Functions runtime that runs on your machine.

### Step 1.1 — Install Functions Core Tools

**On Windows (using npm — Node Package Manager):**

```bash
npm install -g azure-functions-core-tools@4 --unsafe-perm true
# npm              ← Node Package Manager (installs JavaScript/Node packages)
# install -g       ← install globally (available everywhere, not just this folder)
# @4               ← version 4 (latest, supports .NET 8, Node 20, Python 3.11)
# --unsafe-perm    ← allows installation scripts to run (required on some systems)
```

**Alternative — using winget (Windows Package Manager):**

```bash
winget install Microsoft.AzureFunctionsCoreTools
# winget           ← Windows built-in package manager
# install          ← install the package
```

**Alternative — using Chocolatey:**

```bash
choco install azure-functions-core-tools-4
# choco            ← Chocolatey package manager for Windows
```

📝 **NOTE** — Choose whichever installation method you have available. All three install the same tool.

### Step 1.2 — Verify Installation

```bash
func --version
# func             ← the Azure Functions Core Tools command
# --version        ← print the installed version number
```

**Expected output:**

```
4.x.x
```

You should see a version number starting with `4`. If you see `command not found` or an error, the installation did not succeed — try a different installation method from Step 1.1.

### Step 1.3 — Verify Azure CLI

```bash
az --version
# az               ← Azure CLI command
# --version        ← print the version and all installed extensions
```

**Expected output** (first few lines):

```
azure-cli    2.x.x
...
```

If Azure CLI is not installed, go back to Foundation F05 and install it.

### Step 1.4 — Log In to Azure

```bash
az login
# az login         ← opens a browser window to sign in to your Azure account
```

A browser window will open. Sign in with your Azure account. After successful login, you will see a JSON list of your subscriptions in the terminal.

> ✅ **CHECKPOINT 1**
>
> Verify:
> - [ ] `func --version` shows version 4.x.x
> - [ ] `az --version` shows azure-cli 2.x.x
> - [ ] `az login` completed successfully and shows your subscription

---

## Part 2: Create a Function App Project

A **Function App project** is the folder that contains all your functions, configuration, and dependencies. Think of it as a workspace — like a Visual Studio project or a Node.js project with `package.json`.

### Step 2.1 — Create a Project Directory

```bash
mkdir az204-func-lab
# mkdir            ← make directory (create a new folder)
# az204-func-lab   ← the name of your project folder
```

### Step 2.2 — Initialize the Function App Project

```bash
cd az204-func-lab

func init --worker-runtime dotnet-isolated --target-framework net8.0
# func init        ← initialize a new Azure Functions project in the current folder
# --worker-runtime ← which programming language to use
# dotnet-isolated  ← C# running in an isolated (out-of-process) worker
# --target-framework ← which .NET version to target
# net8.0           ← .NET 8 (Long Term Support)
```

**Expected output:**

```
Writing .gitignore
Writing host.json
Writing local.settings.json
Writing az204-func-lab.csproj
```

### Step 2.3 — Explore the Project Structure

```bash
dir
# dir              ← list files in the current directory (Windows)
# (use 'ls' on Mac/Linux)
```

**Expected output:**

```
    Directory: C:\...\az204-func-lab

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         ...                              .gitignore
-a----         ...                              az204-func-lab.csproj
-a----         ...                              host.json
-a----         ...                              local.settings.json
-a----         ...                              Program.cs
```

| File | Purpose |
|---|---|
| `.gitignore` | Tells Git which files to ignore (includes `local.settings.json`!) |
| `az204-func-lab.csproj` | C# project file — lists dependencies (NuGet packages) |
| `host.json` | Azure Functions runtime configuration (logging, extensions, timeouts) |
| `local.settings.json` | Local configuration — connection strings, app settings. **Never commit to Git.** |
| `Program.cs` | Entry point for the isolated worker process |

### Step 2.4 — View local.settings.json

```bash
type local.settings.json
# type             ← display file contents (Windows)
# (use 'cat' on Mac/Linux)
```

**Expected output:**

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated"
  }
}
```

Note that `AzureWebJobsStorage` is set to `"UseDevelopmentStorage=true"` — this means the functions will use the local storage emulator (Azurite) instead of a real Azure Storage account.

> ✅ **CHECKPOINT 2**
>
> Verify:
> - [ ] `az204-func-lab` directory exists with the files listed above
> - [ ] `local.settings.json` contains `AzureWebJobsStorage` and `FUNCTIONS_WORKER_RUNTIME`
> - [ ] `.gitignore` exists (it automatically excludes `local.settings.json`)

---

## Part 3: Create an HTTP Trigger Function

### Step 3.1 — Generate the HTTP Trigger

```bash
func new --name PlaceOrder --template "HTTP trigger" --authlevel function
# func new          ← create a new function inside the current project
# --name            ← the name of the function
# PlaceOrder        ← we're calling it "PlaceOrder" (handles incoming orders)
# --template        ← which function template to use
# "HTTP trigger"    ← creates a function triggered by HTTP requests
# --authlevel       ← authorization level for the HTTP endpoint
# function          ← requires a function key to call (default, recommended)
```

**Expected output:**

```
The function "PlaceOrder" was created successfully from the "HTTP trigger" template.
```

### Step 3.2 — View the Generated Code

```bash
type PlaceOrder.cs
```

You will see auto-generated code with an HTTP-triggered function. We will replace this code in the next part to add a Queue output binding.

> ✅ **CHECKPOINT 3**
>
> Verify:
> - [ ] `PlaceOrder.cs` file exists in the project directory
> - [ ] The file contains `[Function("PlaceOrder")]` and `HttpTrigger` attribute

---

## Part 4: Add Queue Output Binding to HTTP Function

Now we will modify the HTTP function to write incoming orders to an Azure Queue Storage queue. This demonstrates the **output binding** concept — instead of writing SDK code to connect to a queue, we declare a binding attribute and Azure handles the plumbing.

### Step 4.1 — Install the Queue Storage Extension Package

```bash
dotnet add package Microsoft.Azure.Functions.Worker.Extensions.Storage.Queues
# dotnet add package  ← add a NuGet package (library) to the project
# Microsoft.Azure.Functions.Worker.Extensions.Storage.Queues
#                     ← the Azure Functions extension that enables Queue bindings
```

**Expected output:**

```
info : Package 'Microsoft.Azure.Functions.Worker.Extensions.Storage.Queues' ... successfully installed.
```

### Step 4.2 — Replace the PlaceOrder Function Code

Open `PlaceOrder.cs` in your text editor (VS Code recommended) and replace the entire content with:

```csharp
using System.Net;                                          // For HttpStatusCode
using Microsoft.Azure.Functions.Worker;                    // For Function attribute, FunctionContext
using Microsoft.Azure.Functions.Worker.Http;                // For HttpRequestData, HttpResponseData
using Microsoft.Extensions.Logging;                        // For ILogger
using System.Text.Json;                                    // For JsonSerializer

namespace az204_func_lab                                    // Namespace matches project name
{
    // This class holds the data returned by the function:
    // both the HTTP response AND the queue message
    public class PlaceOrderOutput                           // Multi-output class
    {
        [QueueOutput("orders",                              // Queue output binding:
            Connection = "AzureWebJobsStorage")]            //   "orders" = queue name
        public string? QueueMessage { get; set; }           //   Message to put on the queue

        public HttpResponseData? HttpResponse { get; set; } // HTTP response to return to caller
    }

    // Simple class representing an order
    public class OrderRequest                               // Data transfer object
    {
        public string? ProductName { get; set; }            // What product is being ordered
        public int Quantity { get; set; }                   // How many units
        public string? CustomerEmail { get; set; }          // Customer's email for confirmation
    }

    public class PlaceOrderFunction                         // Class containing the function
    {
        private readonly ILogger _logger;                   // Logger instance

        public PlaceOrderFunction(ILoggerFactory loggerFactory) // Constructor injection
        {
            _logger = loggerFactory.CreateLogger<PlaceOrderFunction>();
        }

        [Function("PlaceOrder")]                            // Function name Azure uses
        public async Task<PlaceOrderOutput> Run(            // Returns multi-output object
            [HttpTrigger(AuthorizationLevel.Function,       // Trigger: HTTP, requires function key
                "post")]                                    // Only accepts POST method
            HttpRequestData req)                            // The incoming HTTP request
        {
            _logger.LogInformation("PlaceOrder function triggered.");

            // Step 1: Read and parse the JSON body from the HTTP request
            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            var order = JsonSerializer.Deserialize<OrderRequest>(requestBody,
                new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
            //  ↑ PropertyNameCaseInsensitive = true means "productName" matches "ProductName"

            // Step 2: Validate the order
            if (order == null || string.IsNullOrEmpty(order.ProductName))
            {
                var badResponse = req.CreateResponse(HttpStatusCode.BadRequest);
                await badResponse.WriteStringAsync("Please provide a valid order with ProductName.");
                return new PlaceOrderOutput { HttpResponse = badResponse };
                //     ↑ Return only HTTP response, no queue message
            }

            // Step 3: Create success response
            var response = req.CreateResponse(HttpStatusCode.OK);
            await response.WriteStringAsync(
                $"Order received: {order.Quantity}x {order.ProductName}. " +
                $"Confirmation will be sent to {order.CustomerEmail}.");

            // Step 4: Return BOTH the HTTP response AND the queue message
            _logger.LogInformation($"Order for {order.ProductName} queued successfully.");

            return new PlaceOrderOutput
            {
                HttpResponse = response,                    // Sent back to the caller as HTTP response
                QueueMessage = requestBody                  // Automatically placed on "orders" queue!
                //            ↑ The binding handles everything: connection, serialization, sending
            };
        }
    }
}
```

**What just happened?**
- We created a **multi-output** class (`PlaceOrderOutput`) that holds both the HTTP response and the queue message
- The `[QueueOutput("orders")]` attribute is the **output binding** — it tells Azure to automatically send the `QueueMessage` property to a queue named "orders"
- We did NOT write any queue connection code — no `QueueClient`, no connection string handling, no message serialization

---

## Part 5: Create a Queue Trigger Consumer Function

Now we will create a second function that is triggered when a message arrives in the "orders" queue. This demonstrates the **event-driven chain**: HTTP request → Queue message → Queue-triggered function.

### Step 5.1 — Generate the Queue Trigger Function

```bash
func new --name ProcessOrder --template "Queue trigger"
# func new          ← create a new function
# --name            ← function name
# ProcessOrder      ← this function processes orders from the queue
# --template        ← template type
# "Queue trigger"   ← triggered when a new message appears in a queue
```

### Step 5.2 — Replace the ProcessOrder Function Code

Open `ProcessOrder.cs` and replace the entire content with:

```csharp
using Microsoft.Azure.Functions.Worker;                    // For Function attribute
using Microsoft.Extensions.Logging;                        // For ILogger

namespace az204_func_lab
{
    public class ProcessOrderFunction
    {
        private readonly ILogger _logger;

        public ProcessOrderFunction(ILoggerFactory loggerFactory)
        {
            _logger = loggerFactory.CreateLogger<ProcessOrderFunction>();
        }

        [Function("ProcessOrder")]                          // Function name
        public void Run(
            [QueueTrigger("orders",                         // Trigger: fires when message arrives
                Connection = "AzureWebJobsStorage")]        //   in queue named "orders"
            string orderMessage)                            // The message content as a string
        {
            _logger.LogInformation(
                $"ProcessOrder triggered! Order received from queue: {orderMessage}");

            // In a real application, you would:
            // 1. Parse the order JSON
            // 2. Validate the order details
            // 3. Process payment
            // 4. Send confirmation email
            // 5. Update inventory

            _logger.LogInformation("Order processed successfully.");
        }
    }
}
```

**What this function does:**
- It watches the "orders" queue using the `[QueueTrigger("orders")]` attribute
- When a new message appears (put there by our `PlaceOrder` function), this function automatically wakes up
- It reads the message content as a string and logs it
- In a real application, this is where you would process the order

> ✅ **CHECKPOINT 4**
>
> Verify:
> - [ ] `PlaceOrder.cs` contains `[QueueOutput("orders")]` attribute
> - [ ] `ProcessOrder.cs` contains `[QueueTrigger("orders")]` attribute
> - [ ] Both functions use `Connection = "AzureWebJobsStorage"`
> - [ ] The queue name "orders" matches in both functions

---

## Part 6: Run Locally and Test with curl

### Step 6.1 — Start the Local Azure Storage Emulator (Azurite)

Before running functions locally, you need a local storage emulator. Azurite simulates Azure Storage on your machine.

**If you have VS Code with Azurite extension:**
1. Press `Ctrl+Shift+P` → type "Azurite: Start" → press Enter

**If you have Azurite installed via npm:**

```bash
# In a SEPARATE terminal window (keep it running):
azurite --silent --location .azurite --debug .azurite/debug.log
# azurite          ← local Azure Storage emulator
# --silent         ← minimal console output
# --location       ← where to store emulator data
# .azurite         ← store data in a .azurite subfolder
# --debug          ← write debug logs to a file
```

If Azurite is not installed:

```bash
npm install -g azurite
# Install Azurite globally
```

### Step 6.2 — Start the Functions Runtime

```bash
func start
# func start       ← start the Azure Functions runtime locally
#                     reads host.json and local.settings.json
#                     discovers all functions in the project
#                     starts listening for triggers
```

**Expected output (key lines):**

```
Azure Functions Core Tools
...

Functions:

        PlaceOrder: [POST] http://localhost:7071/api/PlaceOrder

        ProcessOrder: queueTrigger

For detailed output, run func with --verbose flag.
...
```

You should see both functions listed:
- `PlaceOrder` — an HTTP function with a URL
- `ProcessOrder` — a queue trigger function (no URL — it watches the queue)

📝 **NOTE** — Leave this terminal running. Open a **new terminal window** for the next step.

### Step 6.3 — Test with curl

In a **new terminal window**, send a POST request to the PlaceOrder function:

```bash
curl -X POST http://localhost:7071/api/PlaceOrder -H "Content-Type: application/json" -d "{\"ProductName\":\"Laptop\",\"Quantity\":1,\"CustomerEmail\":\"test@example.com\"}"
# curl             ← command-line HTTP client
# -X POST          ← use HTTP POST method
# http://...       ← the local function URL (from func start output)
# -H "Content-Type: application/json"  ← tell the server we're sending JSON
# -d "{...}"       ← the JSON body (the order data)
```

**Expected response:**

```
Order received: 1x Laptop. Confirmation will be sent to test@example.com.
```

### Step 6.4 — Observe the Queue Consumption

Switch back to the terminal where `func start` is running. You should see log output from BOTH functions:

```
[2026-06-03T22:00:00.000Z] PlaceOrder function triggered.
[2026-06-03T22:00:00.001Z] Order for Laptop queued successfully.
[2026-06-03T22:00:00.050Z] ProcessOrder triggered! Order received from queue: {"ProductName":"Laptop","Quantity":1,"CustomerEmail":"test@example.com"}
[2026-06-03T22:00:00.051Z] Order processed successfully.
```

**What happened:**
1. Your `curl` command sent an HTTP POST to `PlaceOrder`
2. `PlaceOrder` read the order, put it on the "orders" queue via the output binding
3. `ProcessOrder` detected the new message and automatically woke up
4. `ProcessOrder` logged the order details and "processed" it

This is the **decoupling pattern** in action — the HTTP function does not process the order itself. It drops a message on a queue and returns immediately. A separate function picks up the message asynchronously.

> ✅ **CHECKPOINT 5**
>
> Verify:
> - [ ] `curl` returned "Order received: 1x Laptop..."
> - [ ] The `func start` terminal shows logs from BOTH PlaceOrder and ProcessOrder
> - [ ] ProcessOrder received the same order data that was sent via curl

---

## Part 7: Observe Queue Consumption (Optional Exploration)

### Step 7.1 — Send Multiple Orders

Send several orders rapidly to see the queue trigger in action:

```bash
curl -X POST http://localhost:7071/api/PlaceOrder -H "Content-Type: application/json" -d "{\"ProductName\":\"Mouse\",\"Quantity\":3,\"CustomerEmail\":\"alice@example.com\"}"

curl -X POST http://localhost:7071/api/PlaceOrder -H "Content-Type: application/json" -d "{\"ProductName\":\"Keyboard\",\"Quantity\":1,\"CustomerEmail\":\"bob@example.com\"}"

curl -X POST http://localhost:7071/api/PlaceOrder -H "Content-Type: application/json" -d "{\"ProductName\":\"Monitor\",\"Quantity\":2,\"CustomerEmail\":\"carol@example.com\"}"
```

### Step 7.2 — Watch the Logs

In the `func start` terminal, you should see each order being placed and then processed:

```
PlaceOrder function triggered.
Order for Mouse queued successfully.
PlaceOrder function triggered.
Order for Keyboard queued successfully.
PlaceOrder function triggered.
Order for Monitor queued successfully.
ProcessOrder triggered! Order received from queue: {"ProductName":"Mouse",...}
Order processed successfully.
ProcessOrder triggered! Order received from queue: {"ProductName":"Keyboard",...}
Order processed successfully.
ProcessOrder triggered! Order received from queue: {"ProductName":"Monitor",...}
Order processed successfully.
```

Notice how `ProcessOrder` picks up each message one by one from the queue. If you had multiple instances (in Azure with the Consumption plan), they could process messages in parallel.

### Step 7.3 — Stop the Local Runtime

Press `Ctrl+C` in the `func start` terminal to stop the runtime.

---

## Part 8: Create Azure Resources for Deployment

Now we will create the Azure resources needed to deploy your functions to the cloud.

### Step 8.1 — Set Variables

```bash
$RANDOM_SUFFIX = Get-Random -Minimum 10000 -Maximum 99999
# Generate a random number to make resource names unique

$RG_NAME = "az204-func-rg"
# Resource group name

$LOCATION = "eastus"
# Azure region (data center location)

$STORAGE_NAME = "az204funcstore$RANDOM_SUFFIX"
# Storage account name (must be globally unique, lowercase, no hyphens)

$FUNC_APP_NAME = "az204-func-app-$RANDOM_SUFFIX"
# Function App name (must be globally unique)
```

### Step 8.2 — Create Resource Group

```bash
az group create --name $RG_NAME --location $LOCATION
# az group create   ← create a new resource group
# --name            ← name of the resource group
# --location        ← Azure region to create it in
```

**Expected output** (key line):

```json
{
  "id": "/subscriptions/.../resourceGroups/az204-func-rg",
  "location": "eastus",
  "name": "az204-func-rg",
  "provisioningState": "Succeeded"
}
```

### Step 8.3 — Create Storage Account

```bash
az storage account create --name $STORAGE_NAME --resource-group $RG_NAME --location $LOCATION --sku Standard_LRS --kind StorageV2
# az storage account create  ← create a new storage account
# --name                     ← globally unique name for the storage account
# --sku Standard_LRS         ← Locally Redundant Storage (cheapest, fine for labs)
# --kind StorageV2            ← General Purpose v2 (default, recommended type)
```

This may take 30–60 seconds.

### Step 8.4 — Create the Function App

```bash
az functionapp create --name $FUNC_APP_NAME --resource-group $RG_NAME --storage-account $STORAGE_NAME --consumption-plan-location $LOCATION --runtime dotnet-isolated --runtime-version 8 --functions-version 4 --os-type Windows
# az functionapp create           ← create a new Function App in Azure
# --name                          ← globally unique name for the Function App
# --storage-account               ← the storage account for AzureWebJobsStorage
# --consumption-plan-location     ← region for the Consumption plan (auto-created)
# --runtime dotnet-isolated        ← C# isolated worker runtime
# --runtime-version 8              ← .NET 8
# --functions-version 4            ← Azure Functions v4
# --os-type Windows                ← Windows operating system
```

This may take 1–2 minutes.

**Expected output** (key lines):

```json
{
  "defaultHostName": "az204-func-app-12345.azurewebsites.net",
  "name": "az204-func-app-12345",
  "state": "Running"
}
```

> ✅ **CHECKPOINT 6**
>
> Verify:
> - [ ] Resource group `az204-func-rg` was created successfully
> - [ ] Storage account was created (note the name)
> - [ ] Function App was created and is in "Running" state
> - [ ] The `defaultHostName` URL is displayed (you'll need this for testing)

---

## Part 9: Deploy to Azure

### Step 9.1 — Publish Your Functions

```bash
func azure functionapp publish $FUNC_APP_NAME
# func azure functionapp publish  ← build the project and deploy it to Azure
# $FUNC_APP_NAME                  ← the name of your Function App in Azure
```

**Expected output:**

```
Getting site publishing info...
Preparing archive...
Uploading ...
Upload completed successfully.
Deployment completed successfully.
Syncing triggers...
Functions in az204-func-app-12345:
    PlaceOrder - [httpTrigger]
        Invoke url: https://az204-func-app-12345.azurewebsites.net/api/placeorder?code=abc123...

    ProcessOrder - [queueTrigger]
```

📝 **NOTE** — The output shows the **Invoke URL** for your HTTP function, complete with the function key (`?code=abc123...`). Save this URL — you will need it for testing.

---

## Part 10: Test the Live Endpoint

### Step 10.1 — Copy the Invoke URL

Copy the full invoke URL from the deployment output. It looks like:

```
https://az204-func-app-12345.azurewebsites.net/api/placeorder?code=abc123def...
```

### Step 10.2 — Send a Test Request

```bash
curl -X POST "https://az204-func-app-12345.azurewebsites.net/api/placeorder?code=YOUR_FUNCTION_KEY_HERE" -H "Content-Type: application/json" -d "{\"ProductName\":\"Cloud Laptop\",\"Quantity\":1,\"CustomerEmail\":\"azure@example.com\"}"
# Replace the URL with YOUR actual invoke URL from Step 9.1
```

**Expected response:**

```
Order received: 1x Cloud Laptop. Confirmation will be sent to azure@example.com.
```

If you get this response, your function is running successfully in Azure!

📝 **NOTE** — Notice the `?code=abc123...` in the URL. This is the **function key** required because we set `AuthorizationLevel.Function`. Without this key, the request would return HTTP 401 Unauthorized.

> ✅ **CHECKPOINT 7**
>
> Verify:
> - [ ] Deployment completed successfully
> - [ ] The invoke URL is displayed with a function key
> - [ ] The curl test returned a successful response
> - [ ] The authorization key was required (try without `?code=` and observe the 401 error)

---

## Part 11: View Logs in the Azure Portal

### Step 11.1 — Open the Azure Portal

1. Go to [https://portal.azure.com](https://portal.azure.com)
2. In the search bar at the top, type your Function App name (e.g., `az204-func-app-12345`)
3. Click on the Function App resource

### Step 11.2 — Navigate to Functions

1. In the left menu, click **Functions**
2. You should see both `PlaceOrder` and `ProcessOrder` listed

### Step 11.3 — View Invocation Logs

1. Click on **PlaceOrder**
2. Click **Monitor** in the left menu
3. Click the **Invocations** tab
4. You should see your recent test invocations with timestamps, success/failure status, and duration

### Step 11.4 — View Live Logs (Optional)

1. Click the **Logs** tab (under Monitor)
2. Click **Connect** to start the log stream
3. Send another curl request from your terminal
4. Watch the log output appear in real-time

📝 **NOTE** — It may take a few minutes for Application Insights data to appear in the Monitor tab. Live logs stream instantly.

---

## Part 12: Clean Up Resources

> ⚠️ **IMPORTANT** — Always clean up lab resources to avoid unexpected charges. Even though the Consumption plan is essentially free, the storage account incurs a small ongoing cost.

### Step 12.1 — Delete the Resource Group

```bash
az group delete --name $RG_NAME --yes --no-wait
# az group delete   ← delete the resource group and ALL resources inside it
# --name            ← name of the resource group to delete
# --yes             ← skip the confirmation prompt
# --no-wait         ← don't wait for deletion to complete (runs in background)
```

### Step 12.2 — Verify Deletion

Wait 1–2 minutes, then verify the resource group is gone:

```bash
az group list --output table
# az group list     ← list all resource groups in your subscription
# --output table    ← display in a readable table format
```

**Expected:** The `az204-func-rg` resource group should NOT appear in the list. If it still shows, wait another minute and check again — deletion takes time.

### Step 12.3 — Clean Up Local Files (Optional)

```bash
cd ..
Remove-Item -Recurse -Force az204-func-lab
# Remove-Item      ← delete files/folders (PowerShell)
# -Recurse         ← delete all contents including subfolders
# -Force           ← don't prompt for confirmation
```

> ✅ **FINAL CHECKPOINT**
>
> Verify:
> - [ ] Resource group `az204-func-rg` is deleted
> - [ ] `az group list` does NOT show the resource group
> - [ ] No Azure resources from this lab remain

---

## Troubleshooting — Top 5 Errors and Fixes

### Error 1: "Azurite is not running" / "Connection refused" when running `func start`

```
Microsoft.Azure.Storage.Common: No connection could be made because the target machine
actively refused it. 127.0.0.1:10000
```

**Cause:** The local storage emulator (Azurite) is not running.

**Fix:** Start Azurite before running `func start`:
- VS Code: `Ctrl+Shift+P` → "Azurite: Start"
- Terminal: `azurite --silent --location .azurite --debug .azurite/debug.log`

---

### Error 2: "func: command not found"

```
func : The term 'func' is not recognized as the name of a cmdlet
```

**Cause:** Azure Functions Core Tools is not installed or not in your system PATH.

**Fix:** Install using one of the methods in Part 1 (npm, winget, or choco). Close and reopen your terminal after installation.

---

### Error 3: "The function app name is already in use"

```
The function app name 'az204-func-app-xxxxx' is not available.
```

**Cause:** Function App names must be globally unique across all Azure customers.

**Fix:** Use a different random suffix or add your initials:

```bash
$FUNC_APP_NAME = "az204-func-$(Get-Random -Minimum 10000 -Maximum 99999)"
```

---

### Error 4: "401 Unauthorized" when calling the deployed function

```
HTTP/1.1 401 Unauthorized
```

**Cause:** You did not include the function key in the request.

**Fix:** Include the function key as a query parameter (`?code=YOUR_KEY`) or as an `x-functions-key` header. Get the key from the deployment output or the Azure portal: Function App → Functions → PlaceOrder → Function Keys.

---

### Error 5: "ProcessOrder never fires" / Queue messages not being consumed

**Cause:** The `QueueTrigger` function is not finding messages because the queue name does not match, or Azurite is not running.

**Fix:**
1. Verify the queue name is "orders" in BOTH `PlaceOrder.cs` (QueueOutput) and `ProcessOrder.cs` (QueueTrigger)
2. Verify `Connection = "AzureWebJobsStorage"` matches in both files
3. Restart Azurite and then restart `func start`

---

## What You Learned

- [ ] Installed and used Azure Functions Core Tools (`func init`, `func new`, `func start`)
- [ ] Created a Function App project with `dotnet-isolated` runtime
- [ ] Created an HTTP-triggered function with a Queue output binding
- [ ] Created a Queue-triggered function as a message consumer
- [ ] Tested functions locally using `func start` and `curl`
- [ ] Observed the decoupling pattern: HTTP → Queue → Queue consumer
- [ ] Created Azure resources (resource group, storage account, Function App)
- [ ] Deployed to Azure using `func azure functionapp publish`
- [ ] Tested the live endpoint with a function key
- [ ] Viewed logs in the Azure portal
- [ ] Cleaned up all resources
