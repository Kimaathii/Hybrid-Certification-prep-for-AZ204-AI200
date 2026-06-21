# Module 2 — Azure Functions: Slide Deck

---

## Slide 1: Title Slide

### Azure Functions — Serverless Compute

| | |
|---|---|
| **Module** | M02 — Azure Functions |
| **Exam Domain** | Develop Azure Compute Solutions (25–30%) |
| **Prerequisites** | F01–F06, M01 (App Service) |

**Learning Objectives:**
- Explain what serverless computing is and when to use it
- Describe Azure Functions anatomy: triggers, bindings, and execution model
- Write and read 6-part NCRONTAB cron expressions
- Choose the correct hosting plan for a given scenario
- Identify the four Durable Functions patterns from business descriptions
- Configure functions locally and understand authorization levels

> **Speaker Notes:**
> Welcome to Module 2 — Azure Functions. In Module 1, you learned about App Service, which hosts full web applications. Now we are moving to a more focused model — serverless computing with Azure Functions. By the end of this module, you will understand how to write, trigger, configure, and deploy small pieces of event-driven code, and you will be able to answer exam questions about hosting plans, Durable Functions patterns, and cron syntax. This module is part of the Develop Azure Compute Solutions domain, which accounts for 25–30% of the AZ-204 exam — one of the heaviest-weighted domains.

---

## Slide 2: The Problem This Service Solves

### You Don't Always Need a Full Application

**Traditional approach problems:**
- You need a small piece of code that runs when a file is uploaded
- You deploy an entire web application just for this one task
- The application runs 24/7, even when no files are being uploaded
- You pay for idle servers all month
- You manage OS patches, runtime updates, and infrastructure

**What if you could just write the code and let Azure handle everything else?**

> **Speaker Notes:**
> Before we talk about what Azure Functions IS, let's talk about the PROBLEM it solves. Imagine your company needs a small piece of logic — say, whenever a customer uploads a photo, you need to resize it. With traditional hosting, you'd spin up a full web application, configure a server, set up the runtime, deploy your code, and keep it running 24/7. But 95% of the time, nobody is uploading photos. You're paying for an idle server. Azure Functions eliminates this waste. You write ONLY the resize logic, you tell Azure "run this when a new file appears," and Azure handles absolutely everything else — servers, scaling, monitoring, patching. When no files are uploaded, the cost is literally zero.

---

## Slide 3: The Light Switch Analogy

### Serverless = Pay Only When the Switch Is On

```
🔌 Traditional Server              💡 Serverless (Functions)
━━━━━━━━━━━━━━━━━━━━              ━━━━━━━━━━━━━━━━━━━━━━━━
Power plant running 24/7    vs.    Electricity flows only when switch is ON
You pay monthly flat rate   vs.    You pay per kilowatt-hour used
You manage the wiring       vs.    Power company manages everything
Always consuming power      vs.    Zero cost when switch is OFF
```

**The three-step serverless model:**
1. ✍️ **Write code** — small, focused functions
2. 📢 **Declare a trigger** — what event wakes it up
3. ☁️ **Azure handles the rest** — servers, scaling, patching, billing

> **Speaker Notes:**
> Here's the analogy that makes serverless click. Think about the light switch in your room. When you flip it on, electricity flows. When you flip it off, it stops. You don't own the power plant. You don't manage the wires. You don't pay an electrician to sit in your house all day. You just flip the switch and pay for what you use. Azure Functions works the same way. You write a small piece of code — a function. You declare what triggers it — an HTTP request, a timer, a new file. Azure handles the rest — the servers, the operating system, the scaling. When your function isn't running, you pay nothing. "Serverless" doesn't mean no servers — servers exist, but they're invisible to you.

---

## Slide 4: Function Anatomy

### Every Function = ONE Trigger + Optional Bindings

```
              ┌──────────────────────────┐
              │      YOUR FUNCTION       │
              │                          │
TRIGGER ─────►│   📥 Input Binding(s)    │
(wake-up)     │         │                │
              │    ┌────▼────┐           │
              │    │  YOUR   │           │
              │    │  CODE   │           │
              │    │ (logic) │           │
              │    └────┬────┘           │
              │         │                │
              │   📤 Output Binding(s)───►── To Queue, Blob, etc.
              │                          │
              └──────────────────────────┘
```

- **Trigger** = the "wake-up call" (exactly ONE per function)
- **Input Binding** = "ingredients delivered to you" (optional, zero or more)
- **Output Binding** = "finished products shipped out" (optional, zero or more)

**Conveyor Belt Analogy:**
Bindings are automatic conveyor belts. They bring data in and send data out without you writing connection code.

> **Speaker Notes:**
> Every Azure Function has three components. First, exactly ONE trigger — this is the event that wakes the function up. It could be an HTTP request, a timer, a new message in a queue, a new file in blob storage. You can only have ONE trigger per function — that's a hard rule. Second, optional input bindings — these are like automatic conveyor belts that bring data INTO your function. Instead of writing code to connect to Cosmos DB and query a document, you declare an input binding and Azure delivers the document to you as a parameter. Third, optional output bindings — these send data OUT of your function automatically. Instead of writing code to connect to a queue and send a message, you declare a Queue output binding and just add items to a collector. Azure handles the connection, serialization, and error handling.

---

## Slide 5: Complete Trigger Reference Table

### 8 Trigger Types You Must Know

| Trigger | When It Fires | Exam Scenario |
|---|---|---|
| **HTTP** | Incoming HTTP request | REST API, webhook |
| **Timer** | Cron schedule fires | Nightly cleanup, daily report |
| **Blob Storage** | New/updated file in container | Process uploaded images |
| **Queue Storage** | New message in queue | Async task processing |
| **Service Bus** | Message in SB queue/topic | Enterprise messaging |
| **Cosmos DB** | Document insert/update (change feed) | Sync data downstream |
| **Event Grid** | Event published to topic | React to Azure events |
| **Event Hub** | Stream events arrive | IoT telemetry, high-volume |

> **Speaker Notes:**
> Here are all eight trigger types you need to know for the exam. HTTP is the most common — it turns your function into a simple API endpoint. Timer runs your function on a schedule using cron expressions, which we'll cover in detail. Blob Storage fires when someone uploads or updates a file — perfect for image processing. Queue Storage and Service Bus fire when messages arrive — we'll cover the difference in Modules 9 and 10. Cosmos DB fires when documents change via the change feed. Event Grid fires when events are published. Event Hub fires when streaming data arrives. For the exam, you need to match the scenario to the correct trigger type. "Process uploaded images" = Blob trigger. "Run every night at midnight" = Timer trigger. "React to new orders in a queue" = Queue trigger.

---

## Slide 6: Timer Trigger — 6-Part NCRONTAB

### Azure Cron = 6 Parts (NOT 5!)

```
 ┌──────────── second (0-59)
 │ ┌────────── minute (0-59)
 │ │ ┌──────── hour (0-23)
 │ │ │ ┌────── day of month (1-31)
 │ │ │ │ ┌──── month (1-12)
 │ │ │ │ │ ┌── day of week (0-6, Sun=0)
 │ │ │ │ │ │
 0 */5 * * * *     ← Every 5 minutes
 0 0 9 * * *       ← Daily at 9:00 AM
 0 0 0 * * *       ← Daily at midnight
 0 0 9 * * 1-5     ← 9 AM Mon-Fri
 0 30 8 * * *      ← Daily at 8:30 AM
```

**Standard Linux cron = 5 parts (minute first, no seconds)**
**Azure Functions cron = 6 parts (second first!)**

> **Speaker Notes:**
> This is a critical exam topic. Azure Functions uses NCRONTAB — a cron expression with SIX parts. The first field is SECONDS, which standard Linux cron does not have. Linux cron has five parts starting with minutes. If you see a cron expression on the exam with six fields, read it as Azure format — second, minute, hour, day, month, day-of-week. Let's read some examples. "0 */5 * * * *" — second 0, every 5th minute, every hour, every day. That's "every 5 minutes." "0 0 9 * * 1-5" — second 0, minute 0, hour 9, every day, every month, days 1 through 5 (Monday through Friday). That's "9 AM on weekdays." If you accidentally read these as 5-part Linux cron, every field shifts by one position and your answer will be completely wrong.

---

## Slide 7: ⚠️ EXAM ALERT — Cron and Triggers

### 🔴 Two Critical Exam Traps

**Trap 1: 6-Part vs 5-Part Cron**

| | Azure Functions | Linux Standard |
|---|---|---|
| **Parts** | 6 | 5 |
| **First field** | Second | Minute |
| **Example** | `0 0 9 * * *` = 9am daily | `0 9 * * *` = 9am daily |

If you see `0 0 9 * * 1-5` on the exam → it has 6 fields → Azure format → **9 AM weekdays**

**Trap 2: ONE Trigger Per Function**

❌ WRONG: A function with both an HTTP trigger AND a Timer trigger
✅ RIGHT: Two separate functions — one with HTTP trigger, one with Timer trigger

> **Speaker Notes:**
> Let me highlight two exam traps that catch many students. First, the cron format. The exam may show you a 6-part expression and ask what schedule it represents. If you miscount the fields or read it as standard 5-part cron, you'll get the wrong answer. Always count the fields first — if there are 6, the first one is seconds. Practice: "0 0 9 * * 1-5" has six fields. First is seconds=0, minute=0, hour=9, day=any, month=any, weekday=1-5. That's 9 AM Monday through Friday. Second trap: one trigger per function. The exam may present an answer option that puts two triggers on one function. That's always wrong. If you need to react to two event types, you create two separate functions.

---

## Slide 8: Hosting Plans Overview

### Five Plans — One Decision Table

| Plan | Scaling | Cold Start | Max Timeout | VNet |
|---|---|---|---|---|
| **Consumption** | Auto → zero | ✅ Yes | **10 min** | ❌ |
| **Flex Consumption** | Auto, faster | Minimal | 30 min | ✅ |
| **Premium** | Pre-warmed | ❌ No | Unlimited | ✅ |
| **Dedicated** | Manual/auto | ❌ No | Unlimited | ✅ |
| **Container Apps** | Auto (KEDA) | Minimal | Unlimited | ✅ |

**The tradeoff:** Cost ↔ Performance ↔ Features

> **Speaker Notes:**
> Azure Functions offers five hosting plans. The Consumption plan is the cheapest — you only pay per execution, and it scales automatically, even down to zero instances when there's no traffic. But it has two major limitations: a 10-minute timeout and no VNet integration. The Flex Consumption plan is newer — it offers faster scaling and VNet support with a 30-minute timeout. The Premium plan eliminates cold starts with pre-warmed instances and has unlimited timeout. The Dedicated plan runs on an App Service plan — just like Module 1. Container Apps is for when you want to run functions alongside other containers. For the exam, the critical decision is between Consumption and Premium. Consumption is the default for cost-sensitive workloads. Premium is required when you need instant response (no cold start), long-running functions (beyond 10 minutes), or VNet access.

---

## Slide 9: ⚠️ EXAM ALERT — Hosting Plan Facts

### 🔴 Three Facts the Exam WILL Test

**1. Consumption Plan Max Timeout = 10 Minutes**
> Scenario says "function takes 20 minutes to process video"
> → Consumption plan FAILS → Answer: Premium or Dedicated

**2. VNet Integration = Premium or Dedicated ONLY**
> Scenario says "function must access a database inside a VNet"
> → Consumption CANNOT connect → Answer: Premium

**3. No Cold Start = Premium Plan**
> Scenario says "API must respond instantly, no startup delay"
> → Consumption has cold starts → Answer: Premium (pre-warmed instances)

> **Speaker Notes:**
> These three facts appear in almost every AZ-204 exam. Let me drill them in. First: Consumption plan has a hard 10-minute timeout. If the exam describes a process that takes longer than 10 minutes — video encoding, large file processing, complex data migration — and asks which plan to use, Consumption is wrong. The answer is Premium or Dedicated. Second: if the scenario involves connecting to resources inside a Virtual Network — a private database, an internal service — Consumption plan cannot do this. You need Premium or Dedicated for VNet integration. Third: if the scenario requires instant response with no startup delay — a customer-facing API where every millisecond matters — Consumption has cold starts that can add 1-10 seconds of delay. Premium plan uses pre-warmed instances that are always ready, eliminating cold starts. Know these three facts cold.

---

## Slide 10: Durable Functions Introduction

### Regular Functions Are Stateless — Durable Functions Add State

**The Problem:**
- Order processing: validate → charge → email → update inventory (sequential steps)
- Image resize: generate 5 sizes simultaneously (parallel tasks)
- Video encoding: takes 45 minutes (too long, client needs status updates)
- Health check: poll every 30 seconds until service is healthy (recurring check)

**Regular functions can't do this — they have no memory between executions.**

**Durable Functions** = an extension that adds orchestration and state management.

**Three roles:**
| Role | Job | Analogy |
|---|---|---|
| **Client** | Starts the workflow | Receptionist |
| **Orchestrator** | Coordinates steps | Project manager |
| **Activity** | Does actual work | Team members |

> **Speaker Notes:**
> Regular Azure Functions are stateless — each execution is independent with no memory of what happened before. This works great for simple tasks like "receive request, process it, return response." But real-world workflows are often more complex. What if you need to run steps in sequence, where each step uses the previous step's output? What if you need to run multiple tasks in parallel and wait for all of them? What if a job takes 45 minutes and the client needs to check on its progress? Durable Functions is an extension that adds orchestration and state management to Azure Functions. It uses three roles: the Client function starts the process, the Orchestrator function coordinates the steps like a project manager, and Activity functions do the actual work. The Orchestrator NEVER does work itself — it only delegates and coordinates.

---

## Slide 11: Pattern 1 — Function Chaining

### Sequential Pipeline: A → B → C → D

```
[Client] → [Orchestrator] → Validate → Charge → Receipt → Inventory
                               │          │         │          │
                               └──output──┘──output─┘──output──┘
                                 Each step feeds the next
```

**Key signal:** "Each step depends on the previous step's result"

**Example:** Process customer order:
1. Validate order data ✓ → pass to
2. Charge credit card ✓ → pass to
3. Send receipt email ✓ → pass to
4. Update inventory ✓

> **Speaker Notes:**
> Pattern 1 is Function Chaining — a sequential pipeline where each step's output becomes the next step's input. Think of an assembly line: station 1 builds the frame, station 2 paints it, station 3 adds wheels, station 4 inspects it. Each station must finish before the next starts. In exam questions, look for language like "then," "after that," "using the result from the previous step." A classic scenario is order processing: validate the order, charge the payment using the validated order, send a receipt using the payment confirmation, update inventory. Each step depends on the previous step completing successfully.

---

## Slide 12: Pattern 2 — Fan-Out/Fan-In

### Parallel Tasks → Aggregate Results

```
                    ┌→ Resize(thumbnail) ─┐
[Orchestrator] ─────┼→ Resize(small)     ─┼→ Aggregate all URLs
                    ├→ Resize(medium)    ─┤
                    └→ Resize(large)     ─┘
```

**Key signal:** "Process multiple items simultaneously / in parallel"

**Example:** Resize one image into 4 sizes at the same time, then save all URLs.

**Key code:** `await Task.WhenAll(tasks)` — waits for ALL parallel tasks.

> **Speaker Notes:**
> Pattern 2 is Fan-out/Fan-in. The orchestrator starts multiple activity functions in PARALLEL — they all run at the same time — and then waits for ALL of them to complete before continuing. Think of a teacher giving an exam to 30 students simultaneously. All students take the exam at the same time — that's the fan-out. When everyone finishes, the teacher collects all papers and calculates the average — that's the fan-in. The key code is "Task.WhenAll" — this waits until every parallel task has completed. In exam questions, look for words like "simultaneously," "in parallel," "at the same time," or "aggregate results from multiple tasks."

---

## Slide 13: Pattern 3 — Async HTTP API

### Long Job → Status Polling → Result

```
Client                              Azure Functions
  │                                       │
  │── POST /start ──────────────────────►│ Start job
  │◄── 202 Accepted + status URL ────────│
  │                                       │ ⏳ Processing...
  │── GET /status/abc123 ──────────────►│
  │◄── 202 Running ─────────────────────│
  │                                       │ ⏳ Still processing...
  │── GET /status/abc123 ──────────────►│
  │◄── 200 Completed + result ──────────│ ✅ Done!
```

**Key signal:** "Long-running job, client needs to check progress"

**Example:** Video encoding — client submits video, polls for completion.

> **Speaker Notes:**
> Pattern 3 is the Async HTTP API pattern. This is for long-running jobs where the client can't wait for the result. Instead of blocking the HTTP connection for 45 minutes, the client gets back a status URL immediately. It then polls that URL periodically to check if the job is done. When it's done, the status URL returns the result. Think of ordering a custom birthday cake — you don't stand in the bakery waiting. You get a receipt with a phone number, go about your day, and call back to check if it's ready. The beauty of Durable Functions is that this entire polling infrastructure is provided automatically — you don't write the status endpoint yourself.

---

## Slide 14: Pattern 4 — Monitor

### Poll Until Condition Is Met

```
┌─────────────┐
│    Start     │
└──────┬──────┘
       ▼
┌─────────────┐     No
│  Check      ├────────► Wait (30s) ──┐
│  condition  │                        │
└──────┬──────┘                        │
       │ Yes                           │
       ▼                    ┌──────────┘
┌─────────────┐             │
│    Done!    │     ◄───────┘
└─────────────┘      (loop back to check)
```

**Key signal:** "Repeatedly check / poll until something is true"

**Example:** Check every 30 seconds if an external API is back online.

> **Speaker Notes:**
> Pattern 4 is the Monitor pattern. It's a loop that repeatedly checks a condition, waits between checks, and exits when the condition is met or a timeout expires. Think of checking your oven while baking cookies — you look, they're not done yet, you wait, check again, still not done, wait, check again, done! In the code, the orchestrator uses a durable timer to wait between checks — this timer is "free" because the function isn't consuming resources while waiting. The orchestrator automatically saves its state, so even if the server restarts during a wait period, it picks up right where it left off. In exam questions, look for "poll," "check periodically," "wait until," or "retry until condition is met."

---

## Slide 15: ⚠️ EXAM ALERT — Pattern Identification

### 🔴 The Exam Describes Scenarios — You Identify the Pattern

| If the question says... | The pattern is... |
|---|---|
| "Steps run in sequence, each using previous result" | **Chaining** |
| "Multiple tasks run at the same time, then aggregate" | **Fan-out/Fan-in** |
| "Long job, client polls for status" | **Async HTTP API** |
| "Repeatedly check until condition is true" | **Monitor** |

**Practice:** "A company processes insurance claims. Each claim must be validated, then reviewed by an adjuster, then approved by a manager, then payment is issued. Each step requires the output of the previous step."

**Answer:** Function Chaining ✅

> **Speaker Notes:**
> This is how the exam tests Durable Functions — through scenario descriptions. The exam will NOT say "which pattern is this?" — it will describe a business workflow and expect you to recognize the pattern. Here's the key: look for the distinguishing keywords. Sequential with dependencies? Chaining. Parallel independent tasks? Fan-out/Fan-in. Client needs to poll for status on a long job? Async HTTP API. Repeated checking until something is true? Monitor. Let me give you a practice question: "A company processes insurance claims. Each claim must be validated, then reviewed, then approved, then payment issued. Each step requires the previous step's output." What pattern? Chaining — because the steps are sequential and each depends on the previous result.

---

## Slide 16: Bindings in Code

### HTTP Trigger + Queue Output Binding (C#)

```csharp
[FunctionName("PlaceOrder")]
public static async Task<IActionResult> Run(
    // TRIGGER: HTTP POST request
    [HttpTrigger(AuthorizationLevel.Function, "post")]
    HttpRequest req,

    // OUTPUT BINDING: automatically send to queue
    [Queue("orders", Connection = "AzureWebJobsStorage")]
    IAsyncCollector<Order> orderQueue,

    ILogger log)
{
    var order = JsonConvert.DeserializeObject<Order>(
        await new StreamReader(req.Body).ReadToEndAsync());

    await orderQueue.AddAsync(order);  // ← This sends to the queue!
    // No QueueClient, no connection code, no serialization needed

    return new OkObjectResult("Order received!");
}
```

**Without binding:** 6+ lines of SDK boilerplate
**With binding:** 1 line — `await orderQueue.AddAsync(order)`

> **Speaker Notes:**
> Let me show you bindings in action. This function is an HTTP-triggered API that receives an order and puts it on a queue. Look at the function signature — two attributes. First, the HttpTrigger attribute — this is the trigger. It requires a function key and accepts POST requests. Second, the Queue attribute — this is the OUTPUT binding. It says "connect to a queue named 'orders' using the storage connection string." Inside the function, we just call "orderQueue.AddAsync(order)" — one line. Azure handles creating the queue client, serializing the order to JSON, sending the message, and handling errors. Without the binding, you'd need at least 6 lines of SDK code to do the same thing. Bindings are declarative — you describe WHAT you want, not HOW to do it.

---

## Slide 17: Authorization Levels

### Who Can Call Your HTTP Function?

| Level | Key Required? | Use Case |
|---|---|---|
| **Anonymous** | ❌ No key needed | Public APIs, webhooks |
| **Function** | ✅ Function-specific key | Default. External callers. |
| **Admin** | ✅ Host master key | Admin endpoints (rare) |

**How callers send the key:**
```
GET https://myapp.azurewebsites.net/api/PlaceOrder?code=abc123def...
                                                    ↑ function key
```
OR via header: `x-functions-key: abc123def...`

**Where to find keys:** Azure Portal → Function App → Functions → [function name] → Function Keys

> **Speaker Notes:**
> When you create an HTTP-triggered function, you choose an authorization level. Anonymous means anyone can call the function — no key required. This is for public APIs or webhook receivers where the calling service can't send custom headers. Function level — the default — requires a function-specific key. The caller must include the key either as a "code" query parameter or in an "x-functions-key" header. Each function gets its own key. Admin level requires the host master key — a single key that grants access to ALL functions in the app plus admin endpoints. Treat the master key like a root password. For the exam, know that Function is the default and that Anonymous is required for webhook scenarios where the external service can't pass a key.

---

## Slide 18: Configuration Summary

### Key Settings Every Developer Must Know

| Setting | What It Does | Where |
|---|---|---|
| `local.settings.json` | Local dev config (NEVER commit to Git!) | Project root |
| `AzureWebJobsStorage` | Storage account for internal function state | App Settings |
| `FUNCTIONS_WORKER_RUNTIME` | Language: dotnet/node/python/java | App Settings |
| `AuthorizationLevel` | Who can call HTTP functions | In code (attribute) |

**local.settings.json is local ONLY:**
- ✅ Used during `func start` (local development)
- ❌ NOT deployed to Azure
- ❌ NOT committed to Git (auto-excluded by `.gitignore`)
- 🔄 In Azure, use **Application Settings** instead

> **Speaker Notes:**
> Let's summarize the key configuration settings. local.settings.json is your local development configuration file — it holds connection strings, API keys, and runtime settings. Critically, it is NEVER committed to source control. The gitignore file generated by func init automatically excludes it. When you deploy to Azure, these same values must be configured as Application Settings in the portal or via CLI. AzureWebJobsStorage is the connection string to a storage account that Azure Functions uses internally for managing triggers, timers, and Durable Functions state. FUNCTIONS_WORKER_RUNTIME tells Azure which language runtime to use — dotnet, node, python, or java. AuthorizationLevel is set in code using the HttpTrigger attribute — it controls whether callers need a key.

---

## Slide 19: Module Summary Table

### Everything You Need to Know — One Table

| Topic | Key Fact |
|---|---|
| **Serverless** | Write code + declare trigger. Azure handles rest. Pay per execution. |
| **Triggers** | ONE per function. 8 types: HTTP, Timer, Blob, Queue, Service Bus, Cosmos DB, Event Grid, Event Hub |
| **Cron** | 6-part NCRONTAB (second first). NOT 5-part Linux cron. |
| **Consumption** | Auto-scale, cold starts, 10-min timeout, NO VNet |
| **Premium** | Pre-warmed (no cold start), VNet, unlimited timeout |
| **Chaining** | Sequential: A → B → C → D |
| **Fan-out/Fan-in** | Parallel tasks → aggregate results |
| **Async HTTP API** | Long job → client polls status URL |
| **Monitor** | Loop: check → wait → check → done |
| **Bindings** | Declarative data connections — replace SDK boilerplate |
| **AuthorizationLevel** | Anonymous (no key) / Function (default) / Admin (master key) |
| **local.settings.json** | Local only. Never commit to Git. |

> **Speaker Notes:**
> Here is the complete summary table for Module 2. I recommend taking a screenshot or printing this slide — it contains every key fact you need for the exam. The most tested topics are: one trigger per function, 6-part cron syntax, Consumption plan 10-minute timeout, VNet requiring Premium or Dedicated, Durable Functions pattern identification, and authorization levels. Make sure you can match each Durable Functions pattern to a scenario description — that's how the exam tests this.

---

## Slide 20: Checkpoint Questions and Answers

### Test Your Knowledge

**Q1:** Your function processes video files and takes 25 minutes. You're using the Consumption plan. What happens?

**Q2:** Write a cron expression for "every day at 8:30 AM."

**Q3:** A company needs to resize an image into 5 sizes simultaneously, then store all URLs. Which Durable Functions pattern?

**Q4:** What is the difference between AuthorizationLevel.Function and AuthorizationLevel.Anonymous?

---

### Answers

**A1:** The function times out and fails. Consumption plan has a 10-minute maximum. Switch to Premium or Dedicated.

**A2:** `0 30 8 * * *` — Second=0, Minute=30, Hour=8, every day, every month, every weekday.

**A3:** Fan-out/Fan-in — "simultaneously" means parallel tasks, "then store all" means aggregate results.

**A4:** Function requires a function key (sent via `?code=` or `x-functions-key` header). Anonymous requires no key — anyone can call it.

> **Speaker Notes:**
> Let's test what you've learned. Give students 2-3 minutes to think about each question before revealing answers. For question 1, the key insight is that Consumption plan has a HARD 10-minute limit — the function will be forcibly terminated after 10 minutes, regardless of how close it is to finishing. The solution is to switch to Premium or Dedicated, which have unlimited timeouts. For question 2, remember Azure uses 6-part cron — second, minute, hour, day, month, weekday. 8:30 AM = second 0, minute 30, hour 8, then asterisks for "every day." For question 3, the keywords "simultaneously" and "then store all" point to Fan-out/Fan-in. For question 4, Function requires a key for security — Anonymous allows unrestricted access. In the next module, we'll build on Functions by learning about containerized solutions.
