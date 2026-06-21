# M02 — Scenario Cards: Azure Functions

---

## SCENARIO 1 — Beginner

**TITLE:** The Welcome Email

### BUSINESS CONTEXT:
An e-commerce company wants to send a welcome email to every new customer who creates an account. Currently, the web application team has to write email-sending code inside the main web app, which slows down the registration process.

### THE PROBLEM:
The registration API takes 3 seconds to respond because it waits for the email to send before returning a success response. The team wants to decouple the email sending from the registration flow.

### CONSTRAINTS:
- Email sending should not slow down the registration API
- Must trigger automatically when a new user record is created
- Cost should be near-zero (only ~50 registrations per day)
- No servers to manage

### YOUR TASK:
Design a solution using Azure Functions. What trigger type would you use?

### HINTS:
- Think about what event signals "a new user was created"
- Could a message queue decouple the registration from the email?

### SOLUTION:
Use an **Azure Function with a Queue trigger**.

The architecture:
1. **Registration API** adds a message to an **Azure Queue** containing the new user's email address and name
2. **Azure Function** with a **Queue trigger** automatically picks up the message
3. Function sends the welcome email
4. Registration API returns immediately (no waiting for email)

The function runs on the **Consumption plan** — with only 50 executions per day, this falls well within the free tier (1 million executions/month free).

```csharp
[Function("SendWelcomeEmail")]
public void Run(
    [QueueTrigger("new-registrations")] string message)
{
    var user = JsonSerializer.Deserialize<UserInfo>(message);
    // Send welcome email using email service
    _emailService.SendWelcome(user.Email, user.Name);
}
```

### EXAM CONNECTION:
This tests understanding of the Queue trigger and the event-driven pattern. The exam may describe a scenario where work needs to be "decoupled" from a main application — Queue trigger is the classic answer.

---

## SCENARIO 2 — Beginner

**TITLE:** The Midnight Cleanup

### BUSINESS CONTEXT:
A SaaS company stores temporary files in Azure Blob Storage. These files are generated during user sessions and are no longer needed after 24 hours. Currently, a developer manually deletes old files every few days.

### THE PROBLEM:
The company needs an automated process that runs every night at midnight to delete temporary files older than 24 hours.

### CONSTRAINTS:
- Must run exactly at midnight UTC every day
- No manual intervention required
- Should log how many files were deleted
- Minimal cost

### YOUR TASK:
Design a solution using Azure Functions. What trigger type and cron expression would you use?

### HINTS:
- Which trigger runs a function on a schedule?
- What is the NCRONTAB expression for "midnight every day"?

### SOLUTION:
Use an **Azure Function with a Timer trigger** and cron expression `0 0 0 * * *`.

```csharp
[Function("MidnightCleanup")]
public async Task Run(
    [TimerTrigger("0 0 0 * * *")] TimerInfo timer)  // ← 6-part: sec min hr day month dayOfWeek
{
    _logger.LogInformation($"Cleanup started at: {DateTime.UtcNow}");
    
    int deletedCount = 0;
    // List blobs, check LastModified, delete if > 24 hours old
    await foreach (var blob in _containerClient.GetBlobsAsync())
    {
        if (blob.Properties.LastModified < DateTimeOffset.UtcNow.AddHours(-24))
        {
            await _containerClient.DeleteBlobAsync(blob.Name);
            deletedCount++;
        }
    }
    
    _logger.LogInformation($"Cleanup complete. Deleted {deletedCount} files.");
}
```

> 🚨 **EXAM ALERT:** Azure Functions uses **6-part** NCRONTAB: `{second} {minute} {hour} {day} {month} {day-of-week}`. Standard Linux/Unix cron uses 5 parts (no seconds). The leading `0` for seconds is critical.

### EXAM CONNECTION:
Timer trigger cron syntax is a favourite exam topic. Know: `0 0 0 * * *` = midnight daily. `0 */5 * * * *` = every 5 minutes. `0 30 9 * * 1-5` = 9:30 AM weekdays. Always 6 parts.

---

## SCENARIO 3 — Intermediate

**TITLE:** The Image Resize Pipeline

### BUSINESS CONTEXT:
A real estate platform allows agents to upload property photos. Each photo needs to be resized into 3 versions: thumbnail (150x150), medium (800x600), and large (1920x1080). Currently, the web app does this synchronously, causing uploads to take 15 seconds.

### THE PROBLEM:
Resize the images asynchronously. When a photo is uploaded to Blob Storage, automatically generate the 3 sizes and save them to separate containers.

### CONSTRAINTS:
- Must trigger automatically when a new blob is uploaded
- The 3 resize operations are independent of each other
- Must complete within 5 minutes
- Should scale automatically during busy hours (agents upload most photos between 9 AM–5 PM)

### YOUR TASK:
Design a solution using Azure Functions. What trigger type would you use? Should you use Durable Functions?

### HINTS:
- Which trigger fires when a new blob appears?
- The 3 resizes are independent — which Durable Functions pattern handles independent parallel work?

### SOLUTION:
Use **Durable Functions** with the **Fan-out/Fan-in pattern**.

Architecture:
1. **Blob trigger** function detects new photo upload
2. Calls an **Orchestrator function** that starts 3 **Activity functions** in parallel
3. Each Activity resizes to one size (thumbnail, medium, large)
4. Orchestrator waits for all 3 to complete (fan-in)
5. Orchestrator logs completion

```csharp
// Orchestrator — fans out to 3 parallel activities
[Function("ResizeOrchestrator")]
public async Task RunOrchestrator(
    [OrchestrationTrigger] TaskOrchestrationContext context)
{
    var imageUrl = context.GetInput<string>();
    
    // Fan-out: start all 3 resizes in parallel
    var tasks = new List<Task>
    {
        context.CallActivityAsync("ResizeImage", new ResizeRequest(imageUrl, 150, 150, "thumbnails")),
        context.CallActivityAsync("ResizeImage", new ResizeRequest(imageUrl, 800, 600, "medium")),
        context.CallActivityAsync("ResizeImage", new ResizeRequest(imageUrl, 1920, 1080, "large"))
    };
    
    // Fan-in: wait for all 3 to complete
    await Task.WhenAll(tasks);
}
```

### EXAM CONNECTION:
This tests the **Fan-out/Fan-in** Durable Functions pattern. The exam describes scenarios where multiple independent tasks run in parallel and you wait for all to finish. Key identifier: "multiple independent tasks" + "wait for all results."

---

## SCENARIO 4 — Advanced

**TITLE:** The Video Encoding Timeout

### BUSINESS CONTEXT:
A media company has an Azure Function that encodes uploaded videos into web-friendly formats. The encoding process takes 15–45 minutes per video, depending on file size.

### THE PROBLEM:
The function keeps timing out. Videos larger than 100MB consistently fail because the processing takes longer than the timeout allows.

### CONSTRAINTS:
- Current plan: Consumption (max 10-minute timeout)
- The company does not want to manage VMs or infrastructure
- Must support files that take up to 45 minutes to process
- Needs to connect to a private VNet to access internal storage

### YOUR TASK:
Which Azure Functions hosting plan should the company switch to? Justify your answer.

### HINTS:
- Which plan has no timeout limit (or a much higher one)?
- Which plan supports VNet Integration?

### SOLUTION:
Switch to the **Premium plan** (Elastic Premium / EP1).

Why Premium:
1. **No timeout limit** — Consumption plan has a 10-minute max. Premium has no default limit (configurable, default unlimited).
2. **VNet Integration** — Premium supports VNet Integration for connecting to private resources. Consumption does NOT.
3. **No cold starts** — Premium keeps instances warm, so there's no delay when a video arrives.
4. **Still serverless** — Azure manages the infrastructure; the company doesn't manage VMs.

Why NOT Dedicated (App Service plan):
- Dedicated works but doesn't auto-scale as effectively as Premium
- Premium is the best balance of serverless convenience and advanced features

Why NOT Consumption:
- Consumption has a hard 10-minute timeout limit (configurable to max 10 minutes)
- Consumption does NOT support VNet Integration

### EXAM CONNECTION:
This tests the timeout limits and VNet support differences between Consumption and Premium plans:
- **Consumption:** Max 10 minutes, NO VNet, cold starts
- **Premium:** Unlimited timeout, YES VNet, no cold starts
- **Dedicated:** Unlimited timeout, YES VNet, but less elastic scaling

---

## SCENARIO 5 — Advanced

**TITLE:** The Order Processing Chain

### BUSINESS CONTEXT:
An e-commerce company processes orders through a sequence of steps:
1. Validate payment
2. Reserve inventory
3. Generate invoice
4. Send confirmation email
5. Update analytics dashboard

Each step depends on the previous step's result. If payment validation fails, the entire chain should stop.

### THE PROBLEM:
The company needs a reliable way to execute these 5 steps in sequence, with error handling at each step, and the ability to retry failed steps.

### CONSTRAINTS:
- Steps must execute in strict order (step 2 depends on step 1's result)
- If any step fails, subsequent steps should not execute
- Failed steps should be retried up to 3 times before the chain fails
- The entire chain should be tracked as a single workflow with a status endpoint

### YOUR TASK:
Design a solution using Durable Functions. Which pattern applies? What are the three function types needed?

### HINTS:
- Which Durable Functions pattern executes steps one after another?
- The 3 function types are: Client, Orchestrator, Activity

### SOLUTION:
Use **Durable Functions** with the **Function Chaining pattern**.

Three function types:
1. **Client function** (HTTP trigger) — receives the order, starts the orchestration, returns a status URL
2. **Orchestrator function** — calls each Activity in sequence, passes output of one to input of the next
3. **Activity functions** (5 total) — each performs one step

```csharp
// Orchestrator — chains 5 activities in sequence
[Function("OrderOrchestrator")]
public async Task<OrderResult> RunOrchestrator(
    [OrchestrationTrigger] TaskOrchestrationContext context)
{
    var order = context.GetInput<Order>();
    
    // Step 1: Validate payment (output feeds step 2)
    var paymentResult = await context.CallActivityAsync<PaymentResult>(
        "ValidatePayment", order);
    
    // Step 2: Reserve inventory
    var inventoryResult = await context.CallActivityAsync<InventoryResult>(
        "ReserveInventory", new { order, paymentResult });
    
    // Step 3: Generate invoice
    var invoice = await context.CallActivityAsync<Invoice>(
        "GenerateInvoice", new { order, paymentResult });
    
    // Step 4: Send confirmation email
    await context.CallActivityAsync("SendConfirmation", 
        new { order, invoice });
    
    // Step 5: Update analytics
    await context.CallActivityAsync("UpdateAnalytics", order);
    
    return new OrderResult { Success = true, InvoiceId = invoice.Id };
}
```

For retry logic, configure the Activity call with retry options:
```csharp
var retryOptions = new TaskRetryOptions(
    new RetryPolicy(maxNumberOfAttempts: 3, firstRetryInterval: TimeSpan.FromSeconds(5)));

var paymentResult = await context.CallActivityAsync<PaymentResult>(
    "ValidatePayment", order, retryOptions);
```

The **Client function** returns a status endpoint so the caller can check progress:
```
GET /api/status/{instanceId}
→ { "runtimeStatus": "Running", "output": null }
→ { "runtimeStatus": "Completed", "output": { "success": true } }
```

This is also the **Async HTTP API pattern** — start a long-running operation via HTTP and poll for status.

### EXAM CONNECTION:
This tests: (1) **Function Chaining** — sequential steps where each depends on the previous. (2) **Three function types** — Client starts it, Orchestrator coordinates, Activities do the work. (3) **Async HTTP API pattern** — the status endpoint for polling. The exam may describe a multi-step workflow and ask you to identify the correct Durable Functions pattern.
