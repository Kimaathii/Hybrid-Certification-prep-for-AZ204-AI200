# Lab Guide: M09 - Event-Driven Architecture (Hybrid Web + AI Edition)

| **Module** | 09 |
|------------|----|
| **Lab Title** | Decoupling AI Workloads with Azure Service Bus |
| **Prerequisites** | Azure Subscription, Azure CLI, .NET 8 SDK |
| **Estimated Time** | 45 Minutes |
| **Cost Estimate** | 💰 $0.05 if cleaned up within 24 hours (Service Bus Basic Tier) |
| **What you build** | A Service Bus Queue, a .NET Sender (simulating a Web App), and a .NET Receiver (simulating an AI Worker). |
| **What you learn** | How to provision namespaces, send messages, and implement Peek-Lock message retrieval. |

💰 **COST CALLOUT:** This lab uses Azure Service Bus on the Basic tier. Estimated cost: $0.05 if cleaned up within 24 hours.

---

## Step 1: Provision Azure Infrastructure

First, we will create the resource group, a Service Bus Namespace (the container), and a Queue.

```bash
# 1. Create a Resource Group
az group create \
  --name "rg-az204-events-lab" \
  --location "eastus"

# 2. Create the Service Bus Namespace
# Note: The namespace name MUST be globally unique. Add random numbers to the end.
az servicebus namespace create \
  --resource-group "rg-az204-events-lab" \
  --name "sb-az204-lab-namespace-9988" \
  --location "eastus" \
  --sku Basic  # <-- Basic tier is the cheapest, supports Queues but not Topics

# 3. Create the Queue inside the Namespace
az servicebus queue create \
  --resource-group "rg-az204-events-lab" \
  --namespace-name "sb-az204-lab-namespace-9988" \
  --name "ai-processing-queue"
```

✅ **CHECKPOINT:** Run `az servicebus queue list --resource-group rg-az204-events-lab --namespace-name sb-az204-lab-namespace-9988 --output table` to confirm the queue exists.

### Get the Connection String
We need the connection string to allow our .NET apps to securely communicate with the Queue.

```bash
az servicebus namespace authorization-rule keys list \
  --resource-group "rg-az204-events-lab" \
  --namespace-name "sb-az204-lab-namespace-9988" \
  --name "RootManageSharedAccessKey" \
  --query "primaryConnectionString" \
  --output tsv
```
📝 **NOTE:** Copy the output of this command to a safe place. You will need it for the code.

---

## Step 2: Create the Sender App (Simulating the Web Frontend)

We will build a .NET Console App that pretends to be a Web API accepting requests to analyze documents.

```bash
# Create the project
dotnet new console -n AiWebSender
cd AiWebSender

# Install the Service Bus SDK package
dotnet add package Azure.Messaging.ServiceBus
```

Replace the contents of `Program.cs` with the following code. (Update the `connectionString` variable with the one you copied earlier).

```csharp
using System;
using System.Threading.Tasks;
using Azure.Messaging.ServiceBus;

class Program
{
    // PASTE YOUR CONNECTION STRING HERE
    static string connectionString = "Endpoint=sb://sb-az204-lab-namespace-9988...";
    static string queueName = "ai-processing-queue";

    static async Task Main()
    {
        // 1. Create the client
        await using var client = new ServiceBusClient(connectionString);
        
        // 2. Create the sender tied to our specific queue
        ServiceBusSender sender = client.CreateSender(queueName);

        // 3. Create a batch of messages
        using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();

        for (int i = 1; i <= 3; i++)
        {
            // Simulate an AI Document processing request
            string messageBody = $"Analyze Document #{i} - Extract text and summarize.";
            var message = new ServiceBusMessage(messageBody);
            messageBatch.TryAddMessage(message);
        }

        // 4. Send the batch to the Queue
        await sender.SendMessagesAsync(messageBatch);
        Console.WriteLine($"A batch of 3 AI requests has been published to the queue.");
    }
}
```

Run the application:
```bash
dotnet run
```
*Expected Output:* `A batch of 3 AI requests has been published to the queue.`

---

## Step 3: Create the Receiver App (Simulating the AI Worker)

Now we will create the backend worker that pulls the messages using **Peek-Lock**.

```bash
cd ..
dotnet new console -n AiBackendWorker
cd AiBackendWorker
dotnet add package Azure.Messaging.ServiceBus
```

Replace the contents of `Program.cs`:

```csharp
using System;
using System.Threading.Tasks;
using Azure.Messaging.ServiceBus;

class Program
{
    // PASTE YOUR CONNECTION STRING HERE
    static string connectionString = "Endpoint=sb://sb-az204-lab-namespace-9988...";
    static string queueName = "ai-processing-queue";

    static async Task Main()
    {
        await using var client = new ServiceBusClient(connectionString);
        
        // Create a processor to continuously listen to the queue
        ServiceBusProcessor processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions());

        // Attach event handlers
        processor.ProcessMessageAsync += MessageHandler;
        processor.ProcessErrorAsync += ErrorHandler;

        // Start processing
        await processor.StartProcessingAsync();
        Console.WriteLine("AI Worker started. Press any key to stop...");
        Console.ReadKey();

        // Stop processing gracefully
        await processor.StopProcessingAsync();
    }

    // This handles incoming messages (Peek-Lock is default)
    static async Task MessageHandler(ProcessMessageEventArgs args)
    {
        string body = args.Message.Body.ToString();
        Console.WriteLine($"[AI Worker] Received request: {body}");
        
        // Simulate heavy AI processing time
        await Task.Delay(2000); 

        // ✅ Complete the message. This tells Service Bus to delete it from the queue.
        await args.CompleteMessageAsync(args.Message);
        Console.WriteLine($"[AI Worker] Finished analyzing document.");
    }

    static Task ErrorHandler(ProcessErrorEventArgs args)
    {
        Console.WriteLine($"Error occurred: {args.Exception.Message}");
        return Task.CompletedTask;
    }
}
```

---

## Step 4: Run and Test the Decoupled System

1. Run the Backend Worker:
```bash
dotnet run
```
2. You should immediately see it pull the 3 messages you sent earlier:
*Expected Output:*
```text
AI Worker started. Press any key to stop...
[AI Worker] Received request: Analyze Document #1 - Extract text and summarize.
[AI Worker] Finished analyzing document.
[AI Worker] Received request: Analyze Document #2 - Extract text and summarize.
...
```

🔬 **HANDS-ON:** Notice how the Web Sender (Step 2) finished instantly, but the AI Worker takes its time (simulated 2 seconds per document). This is the power of Event-Driven Decoupling!

---

## Troubleshooting

1. **Error: NamespaceNameInvalid** 
   - *Fix:* The namespace must be globally unique. Change the numbers in `sb-az204-lab-namespace-9988`.
2. **Error: UnauthorizedAccess**
   - *Fix:* Double-check that your connection string includes the `SharedAccessKey` portion and was copied entirely.
3. **Error: QueueNotFound**
   - *Fix:* Ensure the `queueName` variable in your code exactly matches `ai-processing-queue`.
4. **App runs but receives no messages**
   - *Fix:* You might have run the receiver earlier and drained the queue. Go to the Sender folder and run `dotnet run` again to put new messages on the queue.
5. **NuGet package not found**
   - *Fix:* Ensure you ran `dotnet add package Azure.Messaging.ServiceBus` in the correct directory.

---

## Step 5: Clean Up

Always clean up your resources to avoid unexpected charges.

```bash
az group delete \
  --name "rg-az204-events-lab" \
  --yes \
  --no-wait
```
✅ Run `az group list --output table` to confirm the status is "Deleting".
