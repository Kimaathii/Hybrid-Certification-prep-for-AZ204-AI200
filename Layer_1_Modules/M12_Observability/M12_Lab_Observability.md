# Module 12 Lab: Observability (Hybrid Web + AI Edition)

| Lab Header | Details |
| :--- | :--- |
| **Module** | 12 - Observability |
| **Lab Title** | Instrumenting .NET Apps with OpenTelemetry & KQL |
| **Prerequisites** | Azure CLI, .NET 8 SDK, an active Azure Subscription |
| **Estimated Time** | 45 minutes |
| **Cost Estimate** | < $1.00 (Standard Tier Log Analytics, Pay-As-You-Go) |
| **What You Build** | A Log Analytics Workspace, App Insights, and a .NET Console App instrumented with OpenTelemetry. |
| **What You Learn** | How to provision monitoring infrastructure, export OpenTelemetry data, simulate AI failures, and query logs using KQL. |

> 💰 **COST WARNING**
> This lab uses Log Analytics Workspace on the Pay-As-You-Go tier. You are billed per GB of data ingested. The few kilobytes generated in this lab will cost pennies. Estimated cost: < $1.00 if cleaned up within 24 hours.

---

## Part 1: Provisioning the Monitoring Infrastructure

First, we need to create the "Command Center" and the "Filing Cabinet" for our telemetry.

1. **Log in to Azure and create a Resource Group:**

```bash
# Login to Azure
az login

# Create a resource group
# -n: The name of the resource group
# -l: The geographic location
az group create -n rg-observability-lab -l eastus
```

2. **Create the Log Analytics Workspace (The Filing Cabinet):**

```bash
# Create the workspace
# --resource-group: Target resource group
# --workspace-name: Globally unique name for the workspace
az monitor log-analytics workspace create \
  --resource-group rg-observability-lab \
  --workspace-name law-observability-$RANDOM 
```
*Expected Output:* JSON describing the new workspace, including its `id`.

3. **Create the Application Insights Resource (The Detective):**

*Note: You will need the workspace ID from the previous step. Alternatively, you can use the Azure Portal for this step to easily link them.*

```bash
# For this lab, we will use a simplified CLI command 
# that automatically provisions a backing workspace if needed.
# --name: Name of the App Insights instance
# --resource-group: Target resource group
# --location: Region
# --application-type: web (Standard type for .NET apps)
az monitor app-insights component create \
  --app ai-observability-lab \
  --location eastus \
  --kind web \
  -g rg-observability-lab \
  --application-type web
```

4. **Retrieve the Connection String:**
We need this connection string so our .NET app knows where to send the telemetry.

```bash
# Get the connection string
# --app: Name of the app insights component
# -g: Resource group name
az monitor app-insights component show \
  --app ai-observability-lab \
  -g rg-observability-lab \
  --query connectionString \
  --output tsv
```
*Expected Output:* `InstrumentationKey=xxxx...;IngestionEndpoint=https://eastus...`
**Copy this connection string to Notepad. You will need it in Part 3.**

> ✅ **CHECKPOINT**
> Do you have your Application Insights Connection String saved? Without this, your .NET application will be tracking data, but it will have nowhere to send it!

---

## Part 2: Create a .NET Console App

Let's build a simple console application to simulate our workload.

1. **Initialize the .NET Project:**

```bash
# Create a new console application named AiObservabilityApp
dotnet new console -n AiObservabilityApp

# Navigate into the project directory
cd AiObservabilityApp
```

2. **Add OpenTelemetry and Azure Monitor Packages:**

```bash
# Add the OpenTelemetry Core API
dotnet add package OpenTelemetry

# Add the specific exporter that translates OTel to Azure Monitor format
dotnet add package Azure.Monitor.OpenTelemetry.Exporter
```

---

## Part 3: Instrument the App with OpenTelemetry

1. Open `Program.cs` in your preferred editor (e.g., VS Code).
2. Replace the contents of `Program.cs` with the following code. **Make sure to replace the placeholder connection string with your actual connection string.**

```csharp
using System;
using System.Diagnostics;
using System.Threading.Tasks;
using OpenTelemetry;
using OpenTelemetry.Trace;
using Azure.Monitor.OpenTelemetry.Exporter;

class Program
{
    // Define an ActivitySource (The OpenTelemetry name for a Tracer)
    private static readonly ActivitySource MyActivitySource = new ActivitySource("AiObservabilityApp");

    static async Task Main(string[] args)
    {
        // 1. Configure OpenTelemetry
        using var tracerProvider = Sdk.CreateTracerProviderBuilder()
            .AddSource("AiObservabilityApp") // Listen to our ActivitySource
            .AddAzureMonitorTraceExporter(o =>
            {
                // REPLACE THIS WITH YOUR CONNECTION STRING
                o.ConnectionString = "InstrumentationKey=xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx;IngestionEndpoint=https://eastus-0.in.applicationinsights.azure.com/;LiveEndpoint=https://eastus.livediagnostics.monitor.azure.com/";
            })
            .Build();

        Console.WriteLine("Tracing initialized. Starting operations...");

        // 2. Simulate standard workload
        using (var activity = MyActivitySource.StartActivity("StandardDatabaseCall"))
        {
            activity?.SetTag("db.system", "sqlserver");
            await Task.Delay(100); // Simulate 100ms latency
        }

        // 3. Move to Part 4: Simulating an AI API Call
        await SimulateAiApiCallAsync();

        // 4. Important: Flush the telemetry before the app exits!
        tracerProvider.ForceFlush();
        Console.WriteLine("Telemetry flushed. Exiting.");
    }
    
    // Placeholder for Part 4
    static async Task SimulateAiApiCallAsync() { }
}
```

> ✅ **CHECKPOINT**
> Notice the `ForceFlush()` method. Because telemetry is sent in batches to improve performance, if a short-lived console app exits before the batch is sent, telemetry is lost. Always flush before exit in console apps!

---

## Part 4: Simulate a Failed AI API Call

We will now write the logic to simulate a call to an AI endpoint that fails, generating an exception and attaching token counts.

1. Update the `SimulateAiApiCallAsync` method in `Program.cs`:

```csharp
    static async Task SimulateAiApiCallAsync()
    {
        // Create a new Span (Activity) for the AI Call
        using var activity = MyActivitySource.StartActivity("AzureOpenAICall");
        
        try
        {
            // Attach AI specific metadata to the span
            activity?.SetTag("ai.model", "gpt-4o");
            activity?.SetTag("ai.prompt.tokens", 150);
            
            Console.WriteLine("Calling AI Model...");
            await Task.Delay(2000); // Simulate high latency

            // Simulate an HTTP 429 Too Many Requests Exception
            throw new Exception("HTTP 429: AI Rate Limit Exceeded. Please try again in 10 seconds.");
        }
        catch (Exception ex)
        {
            // Record the exception in the telemetry
            activity?.SetStatus(ActivityStatusCode.Error, ex.Message);
            activity?.AddEvent(new ActivityEvent("Exception", tags: new ActivityTagsCollection
            {
                { "exception.message", ex.Message },
                { "exception.stacktrace", ex.StackTrace }
            }));
            
            Console.WriteLine($"Error recorded: {ex.Message}");
        }
    }
```

2. **Run the Application:**

```bash
dotnet run
```
*Expected Output:*
```
Tracing initialized. Starting operations...
Calling AI Model...
Error recorded: HTTP 429: AI Rate Limit Exceeded. Please try again in 10 seconds.
Telemetry flushed. Exiting.
```

---

## Part 5: Run a KQL Query in the Portal

It takes about 3-5 minutes for telemetry to appear in the Azure Portal.

1. Go to the Azure Portal (portal.azure.com).
2. Navigate to your Application Insights resource (`ai-observability-lab`).
3. On the left menu, under **Monitoring**, click **Logs**.
4. Close the pre-built query popup.
5. In the query editor, type the following KQL to find the slow operations:

```kql
// Look in the dependencies table (where OpenTelemetry Spans are mapped)
dependencies
// Filter for our AI call
| where name == "AzureOpenAICall"
// Select specific columns to view
| project timestamp, name, duration, success, customDimensions
```
6. Click **Run**.
7. Expand the `customDimensions` JSON bag in the results. You will see your `"ai.model": "gpt-4o"` and `"ai.prompt.tokens": "150"` tags stored there!

---

## Troubleshooting

- **Error: No data appearing in Application Insights Logs.**
  - *Fix:* Ensure you replaced the placeholder connection string in `Program.cs`. Did you wait 5 minutes? Did you include `tracerProvider.ForceFlush();`?
- **Error: Package 'Azure.Monitor.OpenTelemetry.Exporter' not found.**
  - *Fix:* Ensure you are connected to the internet and run `dotnet restore`. Check for typos in the package name.
- **Error: az command not recognized.**
  - *Fix:* Ensure the Azure CLI is installed and added to your system PATH.

---

## Part 6: Cleanup

To stop incurring charges, delete the resource group.

```bash
# Delete the resource group and all contents without prompting
# -n: Name of the resource group
# -y: Yes (do not prompt for confirmation)
# --no-wait: Return to the command prompt immediately
az group delete -n rg-observability-lab -y --no-wait

# Confirm deletion is in progress
az group list -o table
```
*Expected Output:* The `rg-observability-lab` will either be missing or show a Provisioning State of `Deleting`.
