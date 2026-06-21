# Module 2 — Azure Functions

---

| | |
|---|---|
| **Module** | M02 — Azure Functions |
| **Exam Domain** | Develop Azure Compute Solutions (25–30%) |
| **Weight** | Part of 25–30% domain weighting |
| **Estimated Study Time** | 3–4 hours |
| **Prerequisites** | F01–F06 (Foundations), M01 (App Service) |
| **Learning Path** | `implement-azure-functions` |

---

## Section 1: What Is Serverless?

### The Light Switch Analogy

Think about the light switch in your room. When you flip it on, electricity flows. When you flip it off, electricity stops. You do not own the power plant. You do not manage the wires inside the walls. You do not pay an electrician to sit in your house 24 hours a day waiting for you to need light. You only pay for the electricity you actually use.

**Serverless computing works exactly the same way.** You write a small piece of code — a *function* — and you tell Azure *when* to run it (the "trigger"). Azure takes care of everything else: the servers, the operating system, the networking, the scaling. When your function is not running, you pay nothing. When a request comes in, Azure spins up your function, runs it, and shuts it down again.

### The Technical Definition

**Serverless** is a cloud execution model where:

1. **You write code** — small, focused pieces of logic called *functions*
2. **You declare a trigger** — the event that causes your code to run (an HTTP request, a timer, a new file uploaded, a new message in a queue)
3. **Azure handles everything else** — provisioning servers, patching the operating system, scaling up when traffic increases, scaling down to zero when traffic stops

The word "serverless" does not mean there are no servers. Servers absolutely exist — Azure runs your code on real machines in real data centres. The word means *you* never see, manage, or worry about those servers. They are invisible to you.

**Azure Functions** is Microsoft's serverless compute service. It is the Azure implementation of the serverless model.

> 💡 **KEY CONCEPT**
>
> Serverless = you write code + declare a trigger. Azure handles servers, scaling, and infrastructure. You pay only when your code runs.

### How Is This Different from App Service?

In Module 1, you learned about Azure App Service. App Service runs your *entire application* on a dedicated plan that you choose and pay for continuously. Even if nobody visits your website at 3am, your App Service plan is still running and costing money.

Azure Functions flips this model:

| Feature | App Service (M01) | Azure Functions (M02) |
|---|---|---|
| **What you deploy** | A full web application | Small, focused functions |
| **Runs when** | Always (or on schedule) | Only when triggered |
| **Billing** | Per hour/month for the plan | Per execution (Consumption plan) |
| **Scaling** | You configure autoscale rules | Automatic, event-driven |
| **Best for** | Web apps, APIs, full applications | Event-driven tasks, integrations, automation |

📝 **NOTE** — Azure Functions *can* run on an App Service plan (called the "Dedicated" plan). In that case, billing works just like App Service. We will cover hosting plans in Section 4.

---

## Section 2: Function Anatomy — Triggers and Bindings

### The Conveyor Belt Analogy

Imagine a factory worker standing at a workstation on a conveyor belt. Three things happen:

1. **A bell rings** (the *trigger*) — this tells the worker "start working now!" The bell might ring because a new item arrived on the belt, because a timer went off, or because someone pressed a button. There is exactly **one bell per workstation** — not two, not three, just one.
2. **Ingredients arrive on the belt** (the *input binding*) — raw materials are automatically delivered to the worker's station. The worker did not go to the warehouse to fetch them — the conveyor belt brought them.
3. **Finished products leave on the belt** (the *output binding*) — when the worker finishes, the product is automatically sent to the next station or to the shipping dock. The worker did not carry it there — the belt took it.

The worker only focuses on one thing: **the actual work** (the logic). Everything else — the bell, the delivery of ingredients, the shipping of finished products — is handled by the factory infrastructure.

### The Technical Definition

Every Azure Function has exactly this structure:

- **ONE trigger** (required) — the event that causes the function to execute. Every function must have exactly one trigger. No more, no less.
- **Zero or more input bindings** (optional) — data sources that Azure automatically reads and passes into your function as parameters. You do not write SDK code to fetch this data — Azure does it for you.
- **Zero or more output bindings** (optional) — destinations where Azure automatically sends data that your function produces. You do not write SDK code to send this data — Azure does it for you.

> 🚨 **EXAM ALERT**
>
> Every function has exactly **ONE trigger**. This is a hard rule. If you need to respond to two different event types, you create two separate functions. The exam will test this — never select an answer that puts two triggers on one function.

### Complete Trigger Reference Table

A **trigger** is what causes your function to run. Think of it as the "wake-up call." Here is every trigger type you need to know for the AZ-204 exam:

| Trigger | When It Fires | Real-World Example | Exam Scenario |
|---|---|---|---|
| **HTTP** | An incoming HTTP request hits the function's URL | A user clicks "Submit" on a web form | Building a simple REST API or webhook receiver |
| **Timer** | A scheduled time arrives (cron expression) | Every night at midnight | Nightly database cleanup, daily report generation |
| **Blob Storage** | A new or updated file (blob) appears in a storage container | A customer uploads a profile photo | Automatically resize or process uploaded images |
| **Queue Storage** | A new message arrives in an Azure Queue Storage queue | An order is placed and a message is added to the queue | Asynchronous task processing |
| **Service Bus** | A new message arrives in a Service Bus queue or topic subscription | An enterprise order message arrives | Enterprise messaging with guaranteed delivery |
| **Cosmos DB** | A document is inserted or updated (via change feed) | A new user record is added to the database | Syncing data to another system when records change |
| **Event Grid** | An event is published to an Event Grid topic | An Azure resource is created or deleted | Reacting to Azure resource lifecycle events |
| **Event Hub** | New events arrive in an Event Hub stream | IoT sensors send temperature readings every second | Processing high-volume telemetry or streaming data |

📝 **NOTE** — A **blob** is a file stored in Azure Blob Storage (covered in detail in M04). A **queue** is a line of messages waiting to be processed (covered in M10). **Cosmos DB** is Azure's globally distributed database (M05). **Event Grid** and **Event Hub** are event routing services (M09). You do not need to master these services yet — just know that Functions can be triggered by them.

> 💡 **KEY CONCEPT**
>
> A function = ONE trigger + optional bindings. The trigger is the "what wakes me up." Bindings are the "automatic conveyor belts" that bring data in and send data out without you writing boilerplate code.

### What Are Bindings?

**Bindings** are declarative connections between your function and other Azure services. Instead of writing code to connect to a queue, read a message, send a response, and handle errors, you simply *declare* the binding and Azure handles the plumbing.

There are two types:

| Binding Type | Direction | What It Does | Example |
|---|---|---|---|
| **Input binding** | Into the function | Reads data from a source and passes it as a parameter | Read a document from Cosmos DB |
| **Output binding** | Out of the function | Sends data to a destination | Write a message to a Queue |

A trigger is technically a special type of input binding — it both *activates* the function and *delivers* data. But in Azure terminology, triggers and bindings are discussed separately.

> ✅ **CHECKPOINT**
>
> Before moving on, make sure you can answer these:
> 1. How many triggers can a single function have? *(Answer: Exactly one)*
> 2. What is the difference between a trigger and an input binding? *(Answer: A trigger causes the function to execute AND delivers data. An input binding only delivers data — it does not cause execution.)*
> 3. If you need to react to both HTTP requests and timer events, how many functions do you need? *(Answer: Two — one with an HTTP trigger, one with a Timer trigger)*

---

## Section 3: Timer Trigger and Cron Syntax

### What Is a Timer Trigger?

A **Timer trigger** makes your function run on a schedule — like setting an alarm clock. Instead of waiting for an external event (like an HTTP request or a new file), the function wakes up at specific times you define.

Common use cases:
- Run a database cleanup every night at midnight
- Generate a sales report every Monday at 9am
- Check for expired accounts every 5 minutes

### The NCRONTAB Expression (6-Part Cron)

To define the schedule, Azure Functions uses a format called **NCRONTAB** — a cron expression with **six parts**. Each part represents a unit of time:

```
{second} {minute} {hour} {day-of-month} {month} {day-of-week}
```

| Position | Field | Allowed Values |
|---|---|---|
| 1st | Second | 0–59 |
| 2nd | Minute | 0–59 |
| 3rd | Hour | 0–23 (24-hour clock) |
| 4th | Day of month | 1–31 |
| 5th | Month | 1–12 |
| 6th | Day of week | 0–6 (0 = Sunday, 6 = Saturday) |

### Special Characters

| Character | Meaning | Example |
|---|---|---|
| `*` | Every value | `* * * * * *` = every second |
| `*/n` | Every n units | `*/5` in minute field = every 5 minutes |
| `-` | Range | `1-5` in day-of-week = Monday through Friday |
| `,` | List | `1,3,5` in day-of-week = Monday, Wednesday, Friday |

### Cron Examples You Must Know

| Cron Expression | What It Means | Explanation |
|---|---|---|
| `0 */5 * * * *` | Every 5 minutes | Second=0, every 5th minute, every hour/day/month/weekday |
| `0 0 * * * *` | Every hour (on the hour) | Second=0, minute=0, every hour |
| `0 0 9 * * *` | Every day at 9:00 AM | Second=0, minute=0, hour=9, every day |
| `0 30 8 * * *` | Every day at 8:30 AM | Second=0, minute=30, hour=8, every day |
| `0 0 0 * * *` | Every day at midnight | Second=0, minute=0, hour=0 (midnight) |
| `0 0 9 * * 1-5` | 9:00 AM Monday–Friday | Second=0, minute=0, hour=9, weekdays only |
| `0 0 */2 * * *` | Every 2 hours | Second=0, minute=0, every 2nd hour |
| `0 0 9 1 * *` | 9:00 AM on the 1st of every month | Second=0, minute=0, hour=9, day=1 |

### How to Read Any Cron Expression

Follow this three-step process:

1. **Read right to left** — start with day-of-week, then month, then day, then hour, then minute, then second
2. **Replace `*` with "every"** — `*` in the month field means "every month"
3. **Read `*/n` as "every nth"** — `*/5` in the minute field means "every 5th minute"

**Example:** Read `0 0 9 * * 1-5`
- Day-of-week: `1-5` → Monday through Friday
- Month: `*` → every month
- Day: `*` → every day (that falls on Mon–Fri)
- Hour: `9` → at 9am
- Minute: `0` → at minute 0
- Second: `0` → at second 0
- **Result:** "At 9:00:00 AM, Monday through Friday, every month"

> 🚨 **EXAM ALERT**
>
> Azure Functions uses a **6-part** NCRONTAB expression that starts with **seconds**. Standard Linux cron uses a **5-part** expression that starts with minutes (no seconds field). The exam will show you cron expressions — if you count 6 fields starting with seconds, it is the Azure format. If you accidentally read it as 5-part Linux cron, every field shifts and your answer will be wrong.
>
> Quick check: `0 0 9 * * 1-5`
> - ✅ Azure 6-part: second=0, minute=0, hour=9, day=*, month=*, weekday=Mon–Fri → **9am weekdays**
> - ❌ Linux 5-part (wrong): minute=0, hour=0, day=9, month=*, weekday=Mon–Fri → **midnight on the 9th, Mon–Fri only** ← completely different!

### Timer Trigger in Code

Here is how a timer trigger looks in C#:

```csharp
// This function runs every 5 minutes
[FunctionName("CleanupFunction")]                    // Name of the function
public static void Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo timer, // Timer trigger: runs at second 0 of every 5th minute
    ILogger log)                                      // Logger to write messages
{
    log.LogInformation($"Cleanup ran at: {DateTime.Now}");
    // Your cleanup logic goes here
}
```

Every line explained:
- `[FunctionName("CleanupFunction")]` — gives this function a name that Azure uses to identify it
- `[TimerTrigger("0 */5 * * * *")]` — the **trigger attribute** that says "run this every 5 minutes"
- `TimerInfo timer` — an object Azure passes in with schedule info (like whether the timer missed a run)
- `ILogger log` — a logging object you use to write messages that appear in Azure's monitoring

> ✅ **CHECKPOINT**
>
> Test yourself:
> 1. How many parts does an Azure Functions cron expression have? *(Answer: 6 — starting with seconds)*
> 2. What does `0 0 0 * * *` mean? *(Answer: Every day at midnight)*
> 3. What does `0 30 9 * * 1` mean? *(Answer: Every Monday at 9:30 AM — day-of-week 1 = Monday)*

---

## Section 4: Hosting Plans — Where Your Functions Run

### The Problem

You have written your function and declared its trigger. But Azure needs to run your code *somewhere* — on real servers with real CPUs and real memory. The **hosting plan** determines:

- **Where** your functions run (what kind of infrastructure)
- **How** they scale (automatically? manually?)
- **How long** they can run (timeout limits)
- **How much** you pay (per-execution? per-hour?)
- **What features** you get (networking, always-on, etc.)

Choosing the wrong hosting plan is like choosing the wrong vehicle for a trip. A bicycle is great for a short commute (cheap, efficient) but terrible for hauling furniture across the country. A semi-truck hauls anything but is expensive for a grocery run.

### The Five Hosting Plans

Azure Functions offers five hosting plans. Here is the complete comparison:

| Feature | Consumption | Flex Consumption | Premium | Dedicated (App Service) | Container Apps |
|---|---|---|---|---|---|
| **Scaling** | Automatic, scales to zero | Automatic, faster scale-up | Pre-warmed instances, auto-scale | Manual or auto-scale rules (like M01) | Automatic (KEDA-based) |
| **Cold Start** | ✅ Yes (delay on first request) | Minimal | ❌ No (pre-warmed) | ❌ No (always running) | Minimal |
| **Max Timeout** | **10 minutes** | 30 minutes | Unlimited (default 30 min) | Unlimited | Unlimited |
| **VNet Integration** | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| **Billing** | Per execution + execution time | Per execution, tiered | Per second (pre-warmed instances) | Same as App Service plan | Per vCPU/memory per second |
| **Min Instances** | 0 (scales to zero) | 0 (configurable) | 1+ (always warm) | 1+ (always running) | 0 (scales to zero) |
| **Best For** | Cost-sensitive, sporadic/unpredictable workloads | Better cold start needs at lower cost | Production APIs needing instant response | Existing App Service plan, predictable load | Microservices alongside containers |

### Understanding Cold Start

**Cold start** is the delay that occurs when Azure needs to allocate a new server instance for your function. Think of it like a restaurant kitchen that shuts down between lunch and dinner. When a dinner customer arrives, the kitchen needs time to turn on the stoves, prep the ingredients, and get ready. This delay — maybe 1–10 seconds — is the cold start.

- **Consumption plan** — the kitchen shuts down completely between orders. Every new order after a quiet period requires a cold start.
- **Premium plan** — the kitchen keeps at least one chef ready at all times (a "pre-warmed instance"). No cold start, but you pay for that always-ready chef.
- **Dedicated plan** — the kitchen is always open with a full staff. No cold start, but highest ongoing cost.

### Understanding Timeouts

The **timeout** is the maximum time a single function execution can run before Azure forcibly stops it.

- **Consumption plan: 10 minutes maximum** — if your function takes longer than 10 minutes, Azure kills it. This is a hard limit.
- **Premium and Dedicated: unlimited** — your function can run for hours if needed (though this is unusual for serverless).

> 🚨 **EXAM ALERT**
>
> Three critical hosting plan facts the exam tests:
>
> 1. **Consumption plan max timeout = 10 minutes.** If a scenario describes a process that takes 20 minutes, Consumption plan will NOT work. The answer is Premium or Dedicated.
> 2. **VNet Integration requires Premium or Dedicated.** Consumption plan CANNOT connect to resources inside a Virtual Network. If the scenario requires VNet access, eliminate Consumption.
> 3. **No cold start = Premium plan.** If a scenario requires instant response with no startup delay, the answer is Premium (pre-warmed instances). Dedicated also has no cold start, but Premium is the serverless answer with auto-scaling.

### When to Use Each Plan

| Scenario | Recommended Plan | Why |
|---|---|---|
| Startup with unpredictable traffic, minimal budget | Consumption | Free tier covers 1M executions, scales to zero |
| API that must respond instantly, no delays | Premium | Pre-warmed instances eliminate cold start |
| Function that processes video files for 30+ minutes | Premium or Dedicated | Consumption's 10-min timeout is too short |
| Function that needs to access a database inside a VNet | Premium or Dedicated | Consumption has no VNet integration |
| Already running an App Service plan with spare capacity | Dedicated | No additional cost — share the existing plan |
| Microservice architecture with other containers | Container Apps | Integrates with broader container ecosystem |

> 💡 **KEY CONCEPT**
>
> The hosting plan is a **tradeoff triangle**: cost vs performance vs features. Consumption is cheapest but has cold starts and a 10-minute timeout. Premium eliminates cold starts and timeout limits but costs more. Dedicated is like App Service — always running, always costing, but fully featured.

> ✅ **CHECKPOINT**
>
> Answer these:
> 1. What is the maximum execution timeout on the Consumption plan? *(Answer: 10 minutes)*
> 2. Which plan eliminates cold starts? *(Answer: Premium — uses pre-warmed instances)*
> 3. Can a function on the Consumption plan connect to a VNet? *(Answer: No — VNet integration requires Premium or Dedicated)*

---

## Section 5: Durable Functions — Orchestrating Long-Running Workflows

### The Problem Durable Functions Solve

Regular Azure Functions are **stateless** — each execution is independent with no memory of previous runs. This works perfectly for simple tasks like "receive a request, process it, return a response."

But what about complex workflows?

- **An order process:** validate order → charge payment → send receipt → update inventory. Each step depends on the previous one.
- **Image processing:** resize one image into 5 different sizes simultaneously, then combine all results.
- **A long-running job:** a video encoding task takes 45 minutes. You cannot use a regular Consumption function (10-minute limit), and even if you could, how does the client know when it is done?

These scenarios need **state** (remembering what happened), **orchestration** (coordinating multiple steps), and **long-running execution** (beyond timeout limits).

**Durable Functions** is an extension of Azure Functions that lets you write stateful, long-running workflows in serverless code.

### The Three Roles

Durable Functions use three types of functions working together, like a project team:

| Role | Analogy | What It Does |
|---|---|---|
| **Client Function** | The receptionist | Receives the initial request and **starts** the orchestration. It gets the ball rolling. |
| **Orchestrator Function** | The project manager | **Coordinates** the workflow. Decides what to do, in what order, and what to do with results. Does NOT do the actual work — it delegates. |
| **Activity Function** | The team members | **Does the actual work.** Each activity function performs one specific task (charge payment, send email, resize image). |

How they work together:
1. A **Client** function receives a trigger (e.g., an HTTP request)
2. The Client starts an **Orchestrator**
3. The Orchestrator calls one or more **Activity** functions in a defined pattern
4. The Orchestrator manages state automatically — if the process is paused, it remembers where it stopped

> 💡 **KEY CONCEPT**
>
> Orchestrator functions NEVER do the actual work. They only coordinate. Think of them as a conductor who waves the baton but never plays an instrument. The actual work is done by Activity functions.

### Pattern 1 — Function Chaining 🔴 HIGH

**What it looks like:** A → B → C → D (sequential pipeline)

**Analogy:** An assembly line in a factory. Station 1 builds the frame. Station 2 paints it. Station 3 adds wheels. Station 4 inspects it. Each station must finish before the next one starts, and each station uses the output from the previous one.

**When to use:** When you have a sequence of steps where each step depends on the result of the previous step.

**Example scenario:** Process a customer order:
1. **Validate** the order data → passes validated order to…
2. **Charge** the customer's credit card → passes payment confirmation to…
3. **Send** receipt email → passes email confirmation to…
4. **Update** inventory counts

```
[Client] → [Orchestrator] → Activity1 → Activity2 → Activity3 → Activity4 → [Result]
                                  ↓           ↓           ↓           ↓
                            Validate    →  Charge   →  Receipt  →  Inventory
```

**Code sketch (C#):**

```csharp
[FunctionName("OrderOrchestrator")]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)  // Orchestrator trigger
{
    var order = context.GetInput<Order>();                        // Get input passed by Client

    var validated = await context.CallActivityAsync<Order>(       // Step 1: call Activity function
        "ValidateOrder", order);                                  // Function name + input

    var payment = await context.CallActivityAsync<PaymentResult>( // Step 2: uses Step 1's result
        "ChargePayment", validated);

    var receipt = await context.CallActivityAsync<string>(        // Step 3: uses Step 2's result
        "SendReceipt", payment);

    await context.CallActivityAsync(                              // Step 4: final step
        "UpdateInventory", validated);

    return receipt;                                               // Return final result
}
```

Each `await` pauses the orchestrator until the activity finishes. The orchestrator automatically saves its state, so even if the server restarts, it resumes from where it left off.

### Pattern 2 — Fan-Out/Fan-In 🔴 HIGH

**What it looks like:** One task splits into many parallel tasks, then results are aggregated.

**Analogy:** A teacher gives the same exam to 30 students. All students take the exam *at the same time* (fan-out). When everyone finishes, the teacher collects all papers and calculates the class average (fan-in).

**When to use:** When you can break a task into independent pieces that can run simultaneously, and you need all results before proceeding.

**Example scenario:** Resize a single uploaded image into 4 different sizes (thumbnail, small, medium, large) at the same time:

```
                    ┌→ Resize(thumbnail) ─┐
[Client] → [Orch] ─┼→ Resize(small)     ─┼→ [Aggregate results]
                    ├→ Resize(medium)    ─┤
                    └→ Resize(large)     ─┘
```

**Code sketch (C#):**

```csharp
[FunctionName("ImageResizeOrchestrator")]
public static async Task<string[]> RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var imageUrl = context.GetInput<string>();          // URL of uploaded image

    var tasks = new List<Task<string>>();               // List to hold parallel tasks

    // Fan-out: start all resize tasks simultaneously
    tasks.Add(context.CallActivityAsync<string>("ResizeImage", new { imageUrl, size = "thumbnail" }));
    tasks.Add(context.CallActivityAsync<string>("ResizeImage", new { imageUrl, size = "small" }));
    tasks.Add(context.CallActivityAsync<string>("ResizeImage", new { imageUrl, size = "medium" }));
    tasks.Add(context.CallActivityAsync<string>("ResizeImage", new { imageUrl, size = "large" }));

    // Fan-in: wait for ALL tasks to complete
    var results = await Task.WhenAll(tasks);            // WhenAll = wait for every task

    return results;                                     // Return all resized image URLs
}
```

The key line is `Task.WhenAll(tasks)` — this waits for every parallel task to finish before continuing.

### Pattern 3 — Async HTTP API 🔴 HIGH

**What it looks like:** Client starts a long job → immediately gets a status URL → polls the URL until the job is done → retrieves the result.

**Analogy:** Ordering a custom birthday cake from a bakery. You place the order (start the job), the bakery gives you a receipt with a phone number (status URL), you call periodically to check if it is ready (polling), and when it is done, you pick it up (retrieve the result). You do not stand in the bakery waiting — you go about your day and check back.

**When to use:** When a job takes too long for the client to wait (e.g., video encoding, report generation) and the client needs a way to check on progress.

**How it works:**

```
Client → POST /start        → 202 Accepted + status URL
Client → GET  /status/{id}  → 202 Running...
Client → GET  /status/{id}  → 202 Running...
Client → GET  /status/{id}  → 200 Completed + result
```

Durable Functions provides this status-checking infrastructure **automatically**. When you start a durable orchestration via HTTP, Azure Functions returns a set of URLs the client can use to check status, without you writing any polling logic.

### Pattern 4 — Monitor 🟡 MEDIUM

**What it looks like:** A loop that checks a condition repeatedly, waits between checks, and exits when the condition is met (or a timeout expires).

**Analogy:** Checking your oven while baking cookies. You set a timer, check if the cookies are golden, and if not, close the oven door and wait before checking again. You repeat until they are done or you decide they are burnt.

**When to use:** When you need to poll an external system until a condition is true (e.g., check if a website comes back online, check if a file appears in a folder, check if an approval is granted).

**How it works:**

```
[Start] → Check condition → Not met → Wait (e.g., 30 sec) → Check again
                          → Met! → Exit with result
                          → Timeout → Exit with failure
```

**Code sketch (C#):**

```csharp
[FunctionName("MonitorOrchestrator")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var endpoint = context.GetInput<string>();               // URL to monitor
    var expiryTime = context.CurrentUtcDateTime.AddHours(2); // Give up after 2 hours

    while (context.CurrentUtcDateTime < expiryTime)          // Loop until timeout
    {
        var isHealthy = await context.CallActivityAsync<bool>(
            "CheckHealth", endpoint);                         // Activity checks the endpoint

        if (isHealthy)                                        // Condition met!
        {
            return;                                           // Exit — we're done
        }

        // Not ready yet — wait 30 seconds before checking again
        var nextCheck = context.CurrentUtcDateTime.AddSeconds(30);
        await context.CreateTimer(nextCheck, CancellationToken.None);  // Durable timer (free, no resources used while waiting)
    }

    // If we get here, the timeout expired
    throw new TimeoutException("Endpoint did not become healthy in time.");
}
```

### Pattern Summary Table

| Pattern | Structure | Key Signal in Exam Question | Example Scenario |
|---|---|---|---|
| **Chaining** | A → B → C → D (sequential) | "Each step depends on the previous result" | Order processing pipeline |
| **Fan-out/Fan-in** | Split → parallel tasks → aggregate | "Process multiple items simultaneously" | Resize image into multiple sizes |
| **Async HTTP API** | Start → poll for status → get result | "Long-running job, client checks progress" | Video encoding, report generation |
| **Monitor** | Loop → check → wait → repeat | "Poll until condition is met" | Check if external API is back online |

> 🚨 **EXAM ALERT**
>
> The exam describes a business scenario and asks you to identify the correct Durable Function pattern. It will NOT say "Which pattern is this?" — it will describe the behaviour and expect you to recognise it.
>
> **Pattern recognition shortcuts:**
> - Sequential steps where output feeds the next? → **Chaining**
> - Multiple independent tasks running at the same time? → **Fan-out/Fan-in**
> - Client needs to check on a long-running job's progress? → **Async HTTP API**
> - Repeatedly checking until something is true? → **Monitor**

> ✅ **CHECKPOINT**
>
> Test yourself:
> 1. "A function needs to validate data, then charge a credit card, then send an email — each using the previous step's result." Which pattern? *(Answer: Chaining)*
> 2. "A function resizes an image into 5 sizes at the same time, then returns all URLs." Which pattern? *(Answer: Fan-out/Fan-in)*
> 3. "A video upload triggers a 30-minute encoding job. The mobile app checks every 10 seconds for completion." Which pattern? *(Answer: Async HTTP API)*

---

## Section 6: Function Configuration

### local.settings.json — Your Local Development Configuration

When you develop Azure Functions on your own computer, you need a way to store configuration values like connection strings, API keys, and runtime settings. This file is called `local.settings.json`.

**Analogy:** Think of `local.settings.json` as a sticky note on your desk with your personal passwords and notes. It is useful when you are working at your desk, but you would never pin it to the office bulletin board for everyone to see.

Here is what a typical `local.settings.json` file looks like:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=mystore;AccountKey=abc123...",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "MyApiKey": "sk-12345-abcde",
    "QueueConnectionString": "DefaultEndpointsProtocol=https;AccountName=..."
  },
  "ConnectionStrings": {
    "SqlConnection": "Server=tcp:myserver.database.windows.net..."
  }
}
```

Every property explained:

| Property | What It Does | Required? |
|---|---|---|
| `IsEncrypted` | Whether the values are encrypted. Usually `false` during local development. | Yes |
| `Values` | Key-value pairs for app settings and connection strings. This is where most configuration goes. | Yes |
| `AzureWebJobsStorage` | Connection string to the Azure Storage account that Azure Functions uses internally for state management, triggers, and logging. **Every function app needs this.** | Yes (except HTTP-only on Consumption) |
| `FUNCTIONS_WORKER_RUNTIME` | Tells Azure which language runtime your functions use: `dotnet`, `node`, `python`, `java`, `powershell`, `custom`. | Yes |
| `ConnectionStrings` | Additional connection strings (like SQL databases). | No |

> 🚨 **EXAM ALERT**
>
> `local.settings.json` is for **local development only**. It must **never** be committed to source control (Git). The `.gitignore` file generated by `func init` automatically excludes it. When deployed to Azure, these values must be configured as **Application Settings** in the Azure portal or via CLI.

### AzureWebJobsStorage

This is the most important configuration value. Azure Functions uses a Storage Account internally to:

- Manage timer trigger schedules (so the timer knows when it last ran)
- Handle Durable Functions state (storing orchestration history)
- Manage lease ownership for scaling (deciding which instance handles which events)

Without `AzureWebJobsStorage`, most triggers (except HTTP on Consumption) will fail to start.

For local development, you can use the **Azure Storage Emulator** (Windows) or **Azurite** (cross-platform) instead of a real Azure Storage account:

```json
{
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true"
  }
}
```

The value `"UseDevelopmentStorage=true"` tells the Functions runtime to use the local emulator instead of connecting to Azure.

### FUNCTIONS_WORKER_RUNTIME

This setting tells Azure which language your functions are written in. It determines which language worker process Azure starts.

| Value | Language |
|---|---|
| `dotnet` | C# (in-process) |
| `dotnet-isolated` | C# (isolated worker — recommended for new projects) |
| `node` | JavaScript / TypeScript |
| `python` | Python |
| `java` | Java |
| `powershell` | PowerShell |
| `custom` | Custom handler (any language) |

### AuthorizationLevel — Who Can Call Your HTTP Function?

When you create an HTTP-triggered function, you must decide who is allowed to call it. Azure Functions provides three built-in authorization levels:

| Level | Who Can Call | How | Use Case |
|---|---|---|---|
| **Anonymous** | Anyone on the internet | No key required — just the URL | Public APIs, webhooks |
| **Function** | Anyone with the function key | Include `?code=<function-key>` in URL or `x-functions-key` header | Default. Most common for external callers. |
| **Admin** | Anyone with the host (master) key | Same as Function but requires the master key | Administrative endpoints — rarely used |

**Function keys** are auto-generated tokens that Azure creates for each function. They act as simple API keys. You can find them in the Azure portal under your function → **Function Keys**.

**Host key (master key)** is a single key that grants access to ALL functions in the Function App plus admin endpoints. Treat it like a root password.

```csharp
// Setting the authorization level in code:
[FunctionName("MyApi")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequest req,
    //           ↑ Authorization level                ↑ Allowed HTTP methods
    ILogger log)
{
    // Function code here
}
```

- `AuthorizationLevel.Anonymous` — no key required. Anyone can call this function.
- `AuthorizationLevel.Function` — caller must include the function-specific key.
- `AuthorizationLevel.Admin` — caller must include the host master key.

> 💡 **KEY CONCEPT**
>
> Configuration summary:
> - `local.settings.json` = local-only config file, never committed to Git
> - `AzureWebJobsStorage` = required storage account for internal function state
> - `FUNCTIONS_WORKER_RUNTIME` = language runtime (dotnet, node, python, java)
> - `AuthorizationLevel` = who can call HTTP functions (Anonymous, Function, Admin)

---

## Section 7: Bindings in Code — A Complete Annotated Example

### The Scenario

You are building an e-commerce API. When a customer places an order via an HTTP POST request, your function should:

1. **Receive** the order data from the HTTP request (trigger)
2. **Save** the order to a queue for asynchronous processing (output binding)
3. **Return** a confirmation response to the customer (HTTP response)

Instead of writing code to connect to the Azure Queue Storage SDK, create a queue client, serialize the message, send it, and handle errors, you simply *declare* a Queue output binding. Azure handles everything.

### The Complete Annotated Code

```csharp
using System.IO;                                     // Required for StreamReader
using System.Threading.Tasks;                        // Required for Task (async)
using Microsoft.AspNetCore.Mvc;                      // Required for IActionResult, OkObjectResult
using Microsoft.Azure.WebJobs;                       // Required for FunctionName, Queue attributes
using Microsoft.Azure.WebJobs.Extensions.Http;       // Required for HttpTrigger attribute
using Microsoft.AspNetCore.Http;                     // Required for HttpRequest
using Microsoft.Extensions.Logging;                  // Required for ILogger
using Newtonsoft.Json;                               // Required for JsonConvert (JSON parsing)

namespace OrderApi                                    // Namespace: a named container for your code
{
    // A simple class to represent an order
    public class Order                                // Class definition
    {
        public string ProductName { get; set; }       // Property: the product being ordered
        public int Quantity { get; set; }             // Property: how many units
        public string CustomerEmail { get; set; }     // Property: customer's email address
    }

    public static class PlaceOrderFunction            // The class that holds our function
    {
        [FunctionName("PlaceOrder")]                  // Azure uses this name to identify the function
        public static async Task<IActionResult> Run(  // Returns an HTTP response
            // ===== TRIGGER: HTTP (the "wake-up call") =====
            [HttpTrigger(
                AuthorizationLevel.Function,          // Requires function key to call
                "post")]                              // Only accepts POST requests
            HttpRequest req,                          // The incoming HTTP request object

            // ===== OUTPUT BINDING: Queue (the "outgoing conveyor belt") =====
            [Queue("orders",                          // Queue name: "orders"
                   Connection = "AzureWebJobsStorage")] // Which storage account connection to use
            IAsyncCollector<Order> orderQueue,         // Collector: lets us add messages to the queue

            // ===== LOGGING =====
            ILogger log)                              // Logger for writing diagnostic messages
        {
            log.LogInformation("PlaceOrder function received a request.");

            // Step 1: Read the HTTP request body
            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            //     ↑ Read the entire body of the POST request as a string

            // Step 2: Convert JSON string to an Order object
            var order = JsonConvert.DeserializeObject<Order>(requestBody);
            //         ↑ Parse JSON like {"ProductName":"Laptop","Quantity":1,"CustomerEmail":"a@b.com"}
            //           into an Order object with typed properties

            // Step 3: Validate the order
            if (order == null || string.IsNullOrEmpty(order.ProductName))
            {
                return new BadRequestObjectResult(    // Return HTTP 400 Bad Request
                    "Please provide a valid order with a ProductName.");
            }

            // Step 4: Add the order to the queue (OUTPUT BINDING does the work!)
            await orderQueue.AddAsync(order);
            //    ↑ This single line puts a message on the "orders" queue.
            //      No QueueClient, no connection code, no serialization — the binding handles it all.

            log.LogInformation($"Order for {order.ProductName} added to queue.");

            // Step 5: Return success response to the customer
            return new OkObjectResult(                // Return HTTP 200 OK
                $"Order received: {order.Quantity}x {order.ProductName}. " +
                $"Confirmation will be sent to {order.CustomerEmail}.");
        }
    }
}
```

### What the Binding Did For You

Without the Queue output binding, you would need to write all of this yourself:

```csharp
// WITHOUT binding — manual queue code (you DON'T need to write this!)
var connectionString = Environment.GetEnvironmentVariable("AzureWebJobsStorage");
var queueClient = new QueueClient(connectionString, "orders");
await queueClient.CreateIfNotExistsAsync();
var messageBytes = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(order));
var base64Message = Convert.ToBase64String(messageBytes);
await queueClient.SendMessageAsync(base64Message);
```

With the binding, you replaced all 6 lines with:

```csharp
// WITH binding — one line!
await orderQueue.AddAsync(order);
```

> 💡 **KEY CONCEPT**
>
> Bindings eliminate boilerplate code. Instead of writing connection logic, client creation, serialization, and error handling, you declare the binding as an attribute and Azure handles the plumbing. One attribute replaces many lines of SDK code.

### How the Pieces Connect

```
Customer sends POST /api/PlaceOrder
        │
        ▼
┌─────────────────────────────┐
│  HTTP Trigger                │  ← Wakes up the function
│  AuthorizationLevel.Function │  ← Checks for function key
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Function Logic              │  ← Your code: parse, validate
│  (PlaceOrder)                │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Queue Output Binding        │  ← Automatically sends order to "orders" queue
│  [Queue("orders")]           │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Azure Queue Storage         │  ← Message sits here waiting for a consumer
│  "orders" queue              │
└─────────────────────────────┘
```

A separate Queue-triggered function (in a different file) could then automatically pick up and process each order from the queue. This is the **decoupling** pattern — the HTTP function does not process the order itself; it drops it on a queue and returns immediately. A background worker processes it later.

> ✅ **CHECKPOINT**
>
> Answer these:
> 1. What does `[Queue("orders")]` do in the function signature? *(Answer: It declares a Queue output binding — any data added to the `orderQueue` collector is automatically sent to a queue named "orders")*
> 2. What does `Connection = "AzureWebJobsStorage"` mean? *(Answer: It tells Azure which connection string in the configuration to use for connecting to the storage account that hosts the queue)*
> 3. What is the benefit of using a binding instead of writing SDK code manually? *(Answer: Less code, no boilerplate — Azure handles connections, serialization, and error handling automatically)*

---

## 🔗 Module Connections

Azure Functions connects to many other modules in this course:

| Connected Module | How It Connects |
|---|---|
| **M01 — App Service** | Functions can run on a Dedicated (App Service) plan. Deployment slots work similarly. |
| **M03 — Containers** | Functions can run in containers on Azure Container Apps hosting plan. |
| **M04 — Blob Storage** | Blob trigger fires when files are uploaded. Blob input/output bindings read/write files. |
| **M05 — Cosmos DB** | Cosmos DB trigger (change feed) fires when documents change. Input/output bindings read/write documents. |
| **M06 — Authentication** | HTTP-triggered functions can use Easy Auth or validate JWT tokens for user authentication. |
| **M07 — Secure Solutions** | Functions use Managed Identity + Key Vault for secure secret access. `AzureWebJobsStorage` can reference Key Vault. |
| **M09 — Event-Based** | Event Grid trigger and Event Hub trigger connect Functions to event-driven architectures. |
| **M10 — Message-Based** | Queue Storage trigger and Service Bus trigger connect Functions to messaging systems. |
| **M11 — Monitoring** | Application Insights integrates with Functions for logging, tracing, and performance monitoring. |

---

## ✅ What We Covered

Use this checklist to verify you understood every key concept in this module:

- [ ] **Serverless concept** — You write code + declare a trigger. Azure handles infrastructure, scaling, and billing.
- [ ] **Function anatomy** — Every function has exactly ONE trigger + optional input/output bindings.
- [ ] **Eight trigger types** — HTTP, Timer, Blob, Queue, Service Bus, Cosmos DB, Event Grid, Event Hub.
- [ ] **One trigger rule** — ONE trigger per function. Always. No exceptions.
- [ ] **Timer cron syntax** — Azure uses 6-part NCRONTAB (seconds first), NOT 5-part Linux cron.
- [ ] **Cron examples** — Can read and write common cron expressions (every 5 min, daily at 9am, weekdays only).
- [ ] **Five hosting plans** — Consumption, Flex Consumption, Premium, Dedicated, Container Apps.
- [ ] **Consumption limits** — 10-minute timeout, no VNet integration, cold starts.
- [ ] **Premium advantages** — No cold start (pre-warmed), VNet integration, unlimited timeout.
- [ ] **Durable Functions** — Extension for stateful, long-running workflows.
- [ ] **Three roles** — Client (starts), Orchestrator (coordinates), Activity (does the work).
- [ ] **Four patterns** — Chaining (sequential), Fan-out/Fan-in (parallel), Async HTTP API (poll for status), Monitor (check until condition met).
- [ ] **Pattern identification** — Can identify patterns from scenario descriptions.
- [ ] **local.settings.json** — Local-only config, never committed to Git.
- [ ] **AzureWebJobsStorage** — Required storage account for internal function state.
- [ ] **FUNCTIONS_WORKER_RUNTIME** — Language setting (dotnet, node, python, java).
- [ ] **AuthorizationLevel** — Anonymous (no key), Function (function key), Admin (master key).
- [ ] **Bindings in code** — Can read and understand C# function code with trigger and output binding attributes.

---

## 🧠 Self-Check Questions

1. **A company needs to run a data processing job that takes 25 minutes. They are currently using the Consumption plan and the function keeps timing out. What should they do?**

   *Answer: Switch to the Premium or Dedicated hosting plan. The Consumption plan has a 10-minute maximum timeout, which is too short for a 25-minute job. Premium plan has an unlimited timeout (default 30 minutes, configurable) and also provides pre-warmed instances to eliminate cold starts.*

2. **An exam question describes this scenario: "When a customer uploads a photo, the system must generate a thumbnail, a medium image, and a high-resolution version simultaneously, then store all three URLs in a database." Which Durable Functions pattern is this?**

   *Answer: Fan-out/Fan-in. The key words are "simultaneously" (parallel processing) and "then store all three" (aggregate results after all parallel tasks complete). The three resize operations are the "fan-out" and storing all URLs is the "fan-in."*

3. **What is the difference between `AuthorizationLevel.Function` and `AuthorizationLevel.Anonymous`?**

   *Answer: `AuthorizationLevel.Function` requires the caller to include a function key (either as a `?code=` query parameter or an `x-functions-key` header). Without the correct key, the request is rejected with HTTP 401 Unauthorized. `AuthorizationLevel.Anonymous` allows anyone to call the function without any key — the URL alone is enough. Use Anonymous for public APIs or webhooks; use Function for APIs that need basic access control.*
