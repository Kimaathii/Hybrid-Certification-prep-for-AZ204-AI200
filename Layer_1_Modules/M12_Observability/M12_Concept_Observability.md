# Module 12: Observability (Hybrid Web + AI Edition)

## Cover Page
**Module Number:** 12
**Module Title:** Observability (Hybrid Web + AI Edition)
**Exam Domain:** AZ-204 (Monitoring, Troubleshooting, and Optimizing Azure Solutions) + AI-200 (AI Infrastructure Observability)
**Weight %:** 15-20%
**Estimated Study Time:** 3 Hours
**Prerequisites:** AZ-900 or equivalent cloud concepts, basic knowledge of application development and APIs.

---

## 1. The Problem: The Need for Observability

### Analogy: The Detective at the Restaurant

Imagine you run a massive, incredibly busy restaurant. One night, a VIP guest orders a complex meal. They click the "Order" button on their tablet, but 10 minutes later, the waiter brings out an empty plate with a sticky note saying, "Error 500: Meal Not Found." 

The guest is furious. You need to figure out what happened. 
- Did the tablet fail to send the order to the kitchen? 
- Did the sous-chef burn the steak? 
- Did the ingredient supplier run out of truffles? 
- Or did the waiter trip and drop the plate on the way to the table?

If you don't have cameras, logs, or communication between your staff, you have absolutely no idea where the failure occurred. 

In the software world, a user clicks a "Submit Order" button on your website, and they get an error. The request travels from the user's browser, to your web server, to a backend API, to a database, and maybe even out to a third-party AI service like OpenAI to generate a receipt summary. Where did it fail? Without tracking, you are guessing blindly. 

**Observability** is the camera system, the communication radios, and the logs in your restaurant. It allows you to trace exactly what happened to the request at every single step, measure how long each step took, and pinpoint the exact line of code or server that caused the failure.

### Technical Definition
**Observability** is the measure of how well internal states of a system can be inferred from knowledge of its external outputs. In Azure, observability means instrumenting your code and infrastructure so that metrics, logs, and traces are collected, allowing you to monitor health, detect anomalies, and troubleshoot application issues efficiently.

> 🚨 **EXAM ALERT**
> The AZ-204 exam frequently tests your ability to distinguish between simply "knowing an app is down" (monitoring) and "knowing exactly why the app is down by tracing the request across microservices" (observability and distributed tracing).

---

## 2. Azure Monitor vs Application Insights vs Log Analytics

To build our camera system in Azure, we use three distinct but interconnected services.

### Analogy: The Command Center
- **Log Analytics Workspace:** The massive filing cabinet where all the restaurant's security camera footage, receipt logs, and staff timesheets are stored in one massive pile.
- **Azure Monitor:** The overall command center room. It's the building that houses the filing cabinet and provides the screens to view the data.
- **Application Insights:** The specialized detective who knows exactly how to read the kitchen's specific receipt formats and understands the chef's shorthand. It focuses exclusively on *application code*.

### Technical Definitions

1. **Log Analytics Workspace (LAW):** The centralized data storage repository where telemetry (logs and metrics) from various Azure resources are collected, stored, and queried. It uses the Kusto Query Language (KQL) to extract insights. 
2. **Azure Monitor:** The overarching service in Azure that collects, analyzes, and acts on telemetry from your cloud and on-premises environments. It is the umbrella service.
3. **Application Insights (App Insights):** A feature of Azure Monitor focused specifically on Application Performance Management (APM). It monitors live web apps, tracks exceptions, traces requests, and provides deep code-level diagnostics.

### Comparison Table

| Feature | Azure Monitor | Application Insights | Log Analytics Workspace |
| :--- | :--- | :--- | :--- |
| **Scope** | Umbrella service for all Azure resources (VMs, Networking, Apps). | Specifically for application code (.NET, Java, Node.js, Python). | The storage engine where the data actually lives. |
| **Primary Use** | Platform health, infrastructure metrics. | Code exceptions, page load times, dependency failures. | Storing logs, running KQL queries, long-term retention. |
| **Data Type** | Metrics, Logs. | Application Telemetry (Requests, Traces, Exceptions). | Structured Log Data. |

> 💡 **KEY CONCEPT**
> Application Insights does *not* store its own data anymore. When you create an App Insights resource, it **must** be connected to a Log Analytics Workspace. The workspace stores the data; App Insights provides the specialized lens to view the application-centric data.

> 🚨 **EXAM ALERT**
> If an exam question asks where Application Insights data is stored, the answer is the **Log Analytics Workspace**. Workspace-based Application Insights is the standard.

> ✅ **CHECKPOINT**
> You need to monitor CPU usage on a Virtual Machine, and you also need to see if a specific SQL query inside your web app is taking too long. Which service handles the VM CPU? Which service handles the slow SQL query? 
> *(Answer: Azure Monitor handles the VM metrics. Application Insights handles the slow SQL query telemetry.)*

---

## 3. Distributed Tracing and Application Map

### The Problem
In modern microservices architectures, a single user request might touch 5 different services (e.g., Web Frontend -> Auth Service -> Inventory API -> Payment Gateway -> Database). If the request takes 10 seconds, which of those 5 services is the bottleneck?

### Analogy: The Package Tracking Number
When you mail a package across the country, it gets a unique tracking number. Every time the package arrives at a post office, gets put on a truck, or reaches a sorting facility, it is scanned using that same tracking number. You can look up the number and see the exact path and where it got stuck.

### Technical Definition
**Distributed Tracing** is the process of attaching a unique Correlation ID (tracking number) to a request when it first enters your system. This Correlation ID is passed along to every downstream service in the HTTP headers. Azure Monitor uses this ID to stitch together the entire journey of the request.

#### Application Map
The visual representation of Distributed Tracing in Azure is called the **Application Map**. It automatically discovers the topology of your application and generates a visual graph.

- **Nodes:** Represent individual services (e.g., Web App, SQL Database, external API).
- **Edges (Lines):** Represent the calls between them, displaying the success rate (e.g., 99%) and average latency (e.g., 45ms).
- **Red Nodes:** Indicate a service with a high failure rate.

If your restaurant app is slow, Application Map will show you a green web frontend, a green backend API, but a big red line pointing to the external Payment API indicating it's returning 500 errors.

> 🚨 **EXAM ALERT**
> For AZ-204, remember the specific HTTP headers used for distributed tracing. Historically this was `Request-Id` and `Correlation-Context`, but the industry standard is now the **W3C Trace Context** standard, which uses the `traceparent` and `tracestate` headers. Application Insights supports W3C Trace Context by default.

---

## 4. KQL (Kusto Query Language) Basics

### The Problem
You have millions of logs flowing into your Log Analytics Workspace every day. How do you find the exact error message that happened at 2:00 PM yesterday for user 'kimathi'?

### Analogy: The SQL-Like Filter Funnel
Imagine pouring dirty water through a series of filters. The top filter removes big rocks, the next removes pebbles, and the final removes sand, leaving you with clean water. KQL works the exact same way with the pipe `|` operator. You start with a massive table of logs, and pass it through filters until you get the exact record you need.

### Technical Definition
**KQL (Kusto Query Language)** is a read-only query language used to process data and return results from Log Analytics Workspaces. It is heavily optimized for fast searching over massive amounts of structured and unstructured text data.

### Basic KQL Syntax

All KQL queries start with a table name, followed by the pipe character `|` which passes the results to the next operator.

```kql
// Step 1: Start with the exceptions table
exceptions
// Step 2: Filter by a specific time range
| where timestamp > ago(1h)
// Step 3: Filter by a specific error type
| where problemId == "System.NullReferenceException"
// Step 4: Sort by the newest errors first
| order by timestamp desc
// Step 5: Only show the top 10 results to save screen space
| take 10
```

#### Core KQL Operators for the Exam:
- `where`: Filters rows based on a condition (like SQL WHERE).
- `project`: Selects specific columns to return (like SQL SELECT).
- `summarize`: Aggregates data (like SQL GROUP BY). Useful for counting errors.
- `order by`: Sorts the results.
- `take` / `limit`: Returns a specific number of rows.
- `render`: Visualizes the results as a chart (e.g., `| render timechart`).

> 💡 **KEY CONCEPT**
> Unlike SQL, which starts with the columns (`SELECT name FROM users`), KQL starts with the data source (`users | project name`). Think "Data Source first, filtering second."

> 🚨 **EXAM ALERT**
> You must know how to construct a basic KQL query to find errors or count events. A common exam drag-and-drop question will ask you to order the lines of a KQL query. The table name *always* comes first!

> ✅ **CHECKPOINT**
> Write a conceptual KQL query that takes the `requests` table, filters it to only show requests where `success == false`, and outputs a bar chart.
> *(Answer: requests | where success == false | render barchart)*

---

## 5. Availability Tests

### The Problem
You deployed your web app, and the server metrics look fine. The CPU is at 10%. But the DNS routing broke, and users outside your corporate network can't actually reach the site. Your internal metrics say everything is green, but your customers say it's dead.

### Analogy: The Secret Shopper
A restaurant manager thinks the food is great because the kitchen looks busy. But to be sure the customer experience is actually good, they hire a "secret shopper" to walk through the front door, sit down, order food, and report back. 

### Technical Definition
**Availability Tests** (also known as Synthetic Monitoring) are automated tests run from Microsoft's global datacenters that attempt to access your web application from the "outside" (the public internet) to verify it is responsive and functioning correctly.

### Types of Availability Tests

1. **Standard Test (Ping Test):** 
   - Makes a single HTTP/HTTPS request to your endpoint.
   - Checks if it returns a 200 OK status code within a certain time frame.
   - Great for checking if an API is alive.
2. **Custom TrackAvailability Test:**
   - Instead of Azure pinging your site, your code runs its own custom logic to determine health, and then uses the App Insights SDK (`TrackAvailability()`) to send the result to Azure.
3. **Multi-step Web Test (Classic / Deprecated):**
   - Recorded a sequence of browser actions (login, click here, submit form). 
   - *Note: Microsoft is deprecating Classic Multi-step tests in favor of Standard Tests or custom Playwright/Selenium tests run via Azure Functions.*

> 💰 **COST WARNING**
> Standard availability tests are charged per execution. If you ping your site every 5 minutes from 16 global regions, you are paying for 16 tests every 5 minutes. Optimize the frequency and number of regions to control costs.

> 🚨 **EXAM ALERT**
> If the exam asks how to test if a web app is reachable from multiple geographic locations (e.g., Tokyo, London, Seattle) and alerts you if it takes longer than 2 seconds to load, the answer is a **Standard Availability Test**.

---

## 6. [AI-200 ADDITION] OpenTelemetry: The New Standard

### The Problem
Historically, if you wanted to monitor an app in Azure, you installed the Application Insights SDK. If you moved to AWS, you had to rewrite your code to use the AWS X-Ray SDK. If you wanted to use Datadog, you installed the Datadog SDK. Developers were locked into proprietary monitoring tools.

### Analogy: The Universal Power Adapter
Imagine buying a laptop that only works with a specific brand of wall outlet. When you travel, you have to buy entirely new chargers. OpenTelemetry is the universal travel adapter. You instrument your application once, using a standard plug, and then you can connect it to any power outlet (Azure, AWS, Datadog) without changing the laptop.

### Technical Definition
**OpenTelemetry (OTel)** is an open-source, vendor-neutral standard for instrumenting, generating, collecting, and exporting telemetry data (metrics, logs, and traces). 

Azure Monitor fully embraces OpenTelemetry. Instead of using the proprietary Application Insights SDK, Microsoft now recommends using the OpenTelemetry SDKs with the **Azure Monitor Exporter**.

#### The OpenTelemetry Architecture
1. **Instrument:** You add the OpenTelemetry SDK to your code.
2. **Collect:** The SDK gathers spans, metrics, and logs.
3. **Export:** You add an "Exporter" package (e.g., `Azure.Monitor.OpenTelemetry.Exporter`). This takes the vendor-neutral data and translates it into the specific format Azure Monitor expects.

> 💡 **KEY CONCEPT**
> OpenTelemetry unifies traces, metrics, and logs. A "Trace" is made up of multiple "Spans" (a single operation, like a database query). 

---

## 7. [AI-200 ADDITION] Tracing AI Workloads

### The Problem
With generative AI, your web app might make a call to Azure OpenAI. But LLM calls are fundamentally different from standard database calls:
1. They take a long time (high latency).
2. They cost money based on *Tokens* (input and output text size).
3. The responses are non-deterministic (they change every time).

If an AI feature is costing you $1,000 a day and taking 15 seconds to load, how do you track down *which* prompts are causing the issue?

### Analogy: The Taxi Meter
Taking an AI API call is like taking a taxi. You don't just care that you arrived at the destination (success/fail). You care about how long the trip took (latency) and what the meter says at the end of the ride (token usage). You need to log the meter receipt for every trip.

### Technical Definition
Monitoring AI workloads involves capturing standard HTTP metrics (latency, status codes) AND AI-specific telemetry (Prompt Tokens, Completion Tokens, Model Version, and the raw text of the Prompt/Response).

#### Semantic Kernel Tracing
If you are building AI apps using the **Semantic Kernel** SDK, it has native support for OpenTelemetry. When you enable OTel in a Semantic Kernel app, it automatically generates spans for:
- Token usage (how many tokens were consumed by the prompt and the completion).
- The specific model deployed (e.g., `gpt-4o`).
- The execution time of specific AI Plugins and semantic functions.

By exporting this data to Application Insights, you can use KQL to query exactly how many tokens were used by a specific user across a specific timeframe.

```kql
// Example KQL to find slow AI requests
dependencies
| where target contains "openai.azure.com"
| where duration > 5000 // took longer than 5 seconds
| project timestamp, target, duration, customDimensions
```

> 🚨 **EXAM ALERT (AI-200)**
> Token usage is typically logged in the `customDimensions` or `Properties` bag of the telemetry data. To track AI costs, you must extract the token counts from these custom properties using KQL.

---

## 🔗 Module Connections
- **Connects from Module 3 (App Service):** You enable Application Insights on your App Services to monitor the web apps you built.
- **Connects from Module 11 (Azure OpenAI):** You use the tracing concepts here to monitor the tokens and latency of the AI models you deployed.

---

## What We Covered (Checklist)
- [ ] **Observability Problem:** Why we need logs, metrics, and traces.
- [ ] **Core Services:** Log Analytics Workspace (storage) vs Application Insights (code lens) vs Azure Monitor (umbrella).
- [ ] **Application Map:** Visualizing distributed tracing and identifying bottlenecks.
- [ ] **KQL Basics:** Using `|`, `where`, `project`, and `summarize` to search logs.
- [ ] **Availability Tests:** Using standard ping tests from global locations.
- [ ] **OpenTelemetry:** The vendor-neutral standard replacing proprietary SDKs.
- [ ] **AI Workload Tracing:** Monitoring token usage, prompt latency, and Semantic Kernel integration.

*End of Document*
