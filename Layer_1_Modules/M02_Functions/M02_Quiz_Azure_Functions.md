# M02 — Module Quiz: Azure Functions

---

| | |
|---|---|
| **Module** | M02 — Azure Functions |
| **Questions** | 15 (5 Foundation + 5 Applied + 5 Exam-Level) |
| **Time Limit** | 25 minutes |
| **Passing Score** | 11/15 (73%) |

---

## FOUNDATION QUESTIONS (1–5)

### Question 1

**What is the maximum timeout for an Azure Function on the Consumption plan?**

- A) 5 minutes
- B) 10 minutes ✅
- C) 30 minutes
- D) Unlimited

**Correct Answer: B**

**Explanation:** The Consumption plan has a **default timeout of 5 minutes**, which can be configured up to a **maximum of 10 minutes**. If your function needs to run longer, you must use the Premium or Dedicated plan.

- **A is wrong:** 5 minutes is the default, not the maximum.
- **C is wrong:** 30 minutes is the max for the Dedicated (App Service) plan.
- **D is wrong:** Only the Premium plan supports unlimited (no default limit).

---

### Question 2

**How many triggers can a single Azure Function have?**

- A) Unlimited
- B) Up to 5
- C) Up to 2
- D) Exactly 1 ✅

**Correct Answer: D**

**Explanation:** Every Azure Function has **exactly one trigger** — the event that causes the function to execute. A function can have multiple input and output bindings, but only ONE trigger.

---

### Question 3

**What is the NCRONTAB cron expression for "every 5 minutes"?**

- A) `*/5 * * * *`
- B) `0 */5 * * * *` ✅
- C) `* 5 * * * *`
- D) `0 5 * * * *`

**Correct Answer: B**

**Explanation:** Azure Functions uses **6-part NCRONTAB**: `{second} {minute} {hour} {day} {month} {day-of-week}`. `0 */5 * * * *` means: at second 0, every 5th minute, every hour, every day. Standard Unix cron uses 5 parts (no seconds), which is why option A is wrong.

- **A is wrong:** This is 5-part Unix cron, not 6-part NCRONTAB.
- **C is wrong:** This means "every second during the 5th minute" — not every 5 minutes.
- **D is wrong:** This means "at exactly the 5th minute of every hour" — not every 5 minutes.

---

### Question 4

**What does the `AzureWebJobsStorage` setting in `local.settings.json` specify?**

- A) The connection string for the function's database
- B) The storage account connection string used by the Functions runtime ✅
- C) The URL of the function's web hosting
- D) The authentication key for the function

**Correct Answer: B**

**Explanation:** `AzureWebJobsStorage` is a **required** connection string pointing to an Azure Storage account. The Functions runtime uses this storage for internal operations: managing triggers, logging function executions, tracking timer triggers, and coordinating Durable Functions state.

---

### Question 5

**Which Azure Functions hosting plan supports VNet Integration?**

- A) Consumption plan only
- B) Premium and Dedicated plans only ✅
- C) All plans
- D) Dedicated plan only

**Correct Answer: B**

**Explanation:** VNet Integration is available on the **Premium** and **Dedicated (App Service)** plans. The **Consumption plan does NOT support VNet Integration**. If your function needs to access resources inside a VNet (like a private database), you must use Premium or Dedicated.

---

## APPLIED QUESTIONS (6–10)

### Question 6

**A developer creates an Azure Function with an HTTP trigger. They want the function to be callable without any authentication key. Which authorization level should they set?**

- A) Function
- B) Admin
- C) Anonymous ✅
- D) System

**Correct Answer: C**

**Explanation:** Authorization levels control who can call an HTTP-triggered function:
- **Anonymous:** No key required — anyone can call it
- **Function:** Requires a function-specific key
- **Admin:** Requires the master (admin) key
- **System:** Used internally by Azure (like Event Grid webhooks)

---

### Question 7

**A developer has a function that reads a message from Azure Queue Storage, processes it, and writes the result to Azure Blob Storage. How should this be designed using bindings?**

- A) Queue trigger + Blob input binding
- B) Queue trigger + Blob output binding ✅
- C) HTTP trigger + Queue input binding + Blob output binding
- D) Blob trigger + Queue output binding

**Correct Answer: B**

**Explanation:** The function is **triggered by a queue message** (Queue trigger) and **writes to Blob Storage** (Blob output binding). The trigger fires when a message arrives in the queue. The output binding automatically saves the result to a blob container without the developer writing storage-specific code.

---

### Question 8

**A company wants to run Azure Functions that connect to a private SQL database inside a VNet. They also want to eliminate cold start delays. Which hosting plan should they choose?**

- A) Consumption
- B) Flex Consumption
- C) Premium ✅
- D) Free tier (F1)

**Correct Answer: C**

**Explanation:** The Premium plan provides: (1) **VNet Integration** to connect to private resources, (2) **pre-warmed instances** that eliminate cold starts, (3) serverless auto-scaling. This matches both requirements.

- **A is wrong:** Consumption doesn't support VNet Integration and has cold starts.
- **B is wrong:** Flex Consumption supports VNet but still has cold start characteristics.
- **D is wrong:** F1 is an App Service Plan tier, not a Functions-specific plan.

---

### Question 9

**Which Durable Functions pattern should be used when 10 images need to be resized in parallel, and the workflow must wait for ALL 10 to complete before proceeding?**

- A) Function Chaining
- B) Fan-out/Fan-in ✅
- C) Async HTTP API
- D) Monitor

**Correct Answer: B**

**Explanation:** **Fan-out/Fan-in** starts multiple activities in parallel (fan-out) and then waits for all of them to complete (fan-in) before continuing. The orchestrator uses `Task.WhenAll()` to wait for all parallel tasks.

- **A is wrong:** Function Chaining executes steps sequentially (one after another), not in parallel.
- **C is wrong:** Async HTTP API is for long-running operations with a status polling endpoint.
- **D is wrong:** Monitor is for periodic polling (checking a condition at intervals until met).

---

### Question 10

**What are the three function types (roles) in Durable Functions?**

- A) Trigger, Binding, Output
- B) Producer, Consumer, Processor
- C) Client, Orchestrator, Activity ✅
- D) Starter, Coordinator, Worker

**Correct Answer: C**

**Explanation:**
- **Client function:** Starts the orchestration (typically HTTP or Queue triggered)
- **Orchestrator function:** Coordinates the workflow, calls Activity functions, manages state
- **Activity function:** Does the actual work (database calls, API calls, computations)

---

## EXAM-LEVEL QUESTIONS (11–15)

### Question 11

**A company has an Azure Function triggered by an HTTP request. The function processes the request and needs to add a message to Azure Queue Storage. The developer wants to use bindings instead of writing SDK code. Which attribute should be applied to the output parameter?**

- A) `[QueueTrigger("output-queue")]`
- B) `[Queue("output-queue")]` ✅
- C) `[QueueInput("output-queue")]`
- D) `[BlobOutput("output-queue")]`

**Correct Answer: B**

**Explanation:** The `[Queue("output-queue")]` attribute on an output parameter (or return value) creates an **output binding** to Azure Queue Storage. When the function returns, the value is automatically added as a message to the specified queue.

- **A is wrong:** `[QueueTrigger]` is for triggering a function FROM a queue — it's an input trigger, not an output binding.
- **C is wrong:** `[QueueInput]` is not a valid attribute name.
- **D is wrong:** `[BlobOutput]` writes to Blob Storage, not Queue Storage.

---

### Question 12 — SELECT TWO

**Which TWO of the following are TRUE about the Azure Functions Consumption plan? (SELECT TWO)**

- A) Functions can run for a maximum of 10 minutes ✅
- B) Functions scale to zero when idle ✅
- C) Functions support VNet Integration
- D) Functions never experience cold starts
- E) Functions can have unlimited concurrent instances

**Correct Answers: A, B**

**Explanation:**
- **A is correct:** Consumption plan has a configurable timeout with a maximum of 10 minutes.
- **B is correct:** When no functions are executing, the Consumption plan scales to zero instances — you pay nothing.
- **C is wrong:** Consumption does NOT support VNet Integration — you need Premium or Dedicated.
- **D is wrong:** Consumption experiences cold starts when scaling from zero — the runtime needs to allocate resources.
- **E is wrong:** Consumption has a default limit of 200 instances (can be increased but not unlimited).

---

### Question 13

**An exam question describes the following scenario: "A function must check an external API every 30 seconds until a specific condition is met, then trigger a notification." Which Durable Functions pattern is this?**

- A) Function Chaining
- B) Fan-out/Fan-in
- C) Async HTTP API
- D) Monitor ✅

**Correct Answer: D**

**Explanation:** The **Monitor pattern** periodically checks a condition at defined intervals until the condition is met. It uses `context.CreateTimer()` to wait between checks. This is ideal for polling external systems.

- **A is wrong:** Chaining runs sequential steps — not periodic polling.
- **B is wrong:** Fan-out/Fan-in runs parallel tasks — not periodic checking.
- **C is wrong:** Async HTTP API exposes a status endpoint for long-running operations — it doesn't poll an external system.

---

### Question 14

**A developer has the following timer trigger cron expression: `0 30 9 * * 1-5`. When does this function execute?**

- A) Every 30 seconds past 9 AM on weekdays
- B) At 9:30 AM every day
- C) At 9:30 AM on weekdays (Monday through Friday) ✅
- D) Every 9 hours and 30 minutes on weekdays

**Correct Answer: C**

**Explanation:** Breaking down the 6-part NCRONTAB `0 30 9 * * 1-5`:
- `0` — second 0
- `30` — minute 30
- `9` — hour 9 (9 AM)
- `*` — every day of month
- `*` — every month
- `1-5` — Monday (1) through Friday (5)

Result: 9:30:00 AM, Monday through Friday.

---

### Question 15 — SELECT TWO

**A company's Azure Function needs to process messages from a Service Bus queue. When a message arrives, the function should: (1) save data to Cosmos DB and (2) send a notification to an Event Grid topic. The developer wants to minimise code. Which TWO binding configurations are correct? (SELECT TWO)**

- A) Service Bus Queue trigger ✅
- B) Service Bus Queue output binding
- C) Cosmos DB output binding ✅
- D) Cosmos DB trigger
- E) Event Grid trigger

**Correct Answers: A, C**

**Explanation:**
- **A is correct:** The function is triggered by messages in a Service Bus queue → **Service Bus Queue trigger**.
- **C is correct:** The function saves data to Cosmos DB → **Cosmos DB output binding** (declarative, no SDK code needed).
- **B is wrong:** The function reads FROM Service Bus (trigger), not writes TO it.
- **D is wrong:** Cosmos DB trigger fires when documents change — the function isn't triggered by Cosmos DB.
- **E is wrong:** Event Grid trigger fires when events arrive — the function sends TO Event Grid, not triggered BY it.

Note: For the Event Grid notification, the developer would need to use the Event Grid SDK (there's no built-in Event Grid output binding in the standard binding set), or use an HTTP output to call the Event Grid API.

---

## Score Interpretation

| Score | Assessment | Recommendation |
|---|---|---|
| 13–15 | 🟢 Excellent — Exam ready for Functions topics | Move to M03 |
| 11–12 | 🟡 Good — Review the questions you missed | Re-read specific sections, then proceed |
| 8–10 | 🟠 Needs work — Review the Concept document | Re-read M02 Concept, redo scenarios, retake quiz |
| Below 8 | 🔴 Not ready — Start from the beginning | Re-read M02 Concept thoroughly, then retake |
