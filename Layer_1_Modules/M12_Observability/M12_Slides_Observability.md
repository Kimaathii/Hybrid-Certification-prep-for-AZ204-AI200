# Module 12: Observability (Hybrid Web + AI Edition) - Slide Deck

---

## Slide 1: Title
**Title:** Module 12: Observability (Hybrid Web + AI Edition)
**Exam Domain:** AZ-204 (15-20%) + AI-200 (AI Infrastructure)
**Learning Objectives:**
- Understand Azure Monitor, Application Insights, and Log Analytics.
- Trace requests across microservices using Application Map.
- Write basic Kusto Query Language (KQL) queries.
- Configure Availability Tests.
- Implement OpenTelemetry for Web and AI Workloads.

**Speaker Notes:**
Welcome to Module 12. Today we are tackling Observability. For the AZ-204 exam, this is a heavy hitter, making up a significant portion of the troubleshooting and monitoring domain. We're also blending in AI-200 concepts, showing you how monitoring traditional web apps differs from monitoring expensive AI models. Let's dive in.

---

## Slide 2: The Problem This Service Solves
**Visual:** A frustrated user looking at an "Error 500" screen. A maze of servers behind the screen.
**Text:** 
- User clicks a button -> Gets an error.
- The request touched 5 different servers, a database, and an AI API.
- Which one failed? 
- Without Observability, you are guessing blindly.

**Speaker Notes:**
Imagine a user clicking 'submit' and getting an error. Behind the scenes, that request hopped from a web frontend, to a backend API, to a database, and maybe out to Azure OpenAI. Where did the failure happen? Without proper instrumentation, you have no idea. You need a system that tracks the request at every single hop. That system is Observability.

---

## Slide 3: Analogy: The Restaurant Detective
**Visual:** A detective in a restaurant kitchen looking at order tickets.
**Text:**
- **The Problem:** The waiter brings an empty plate to the VIP. 
- **The Solution:** Security cameras, staff radios, and receipt logs.
- Observability is the camera system for your code. It tracks exactly what happened, when, and where.

**Speaker Notes:**
Think of running a massive restaurant. If an order gets lost, how do you find out whose fault it was? You check the security cameras, the order logs, and the kitchen receipts. Observability provides exactly this for your applications: logs (the receipts), metrics (the camera system), and traces (the step-by-step path of the waiter).

---

## Slide 4: The Three Pillars of Azure Observability
**Visual:** A Venn diagram or three pillars showing Azure Monitor, Application Insights, and Log Analytics.
**Text:**
- **Log Analytics Workspace:** The Database. Where data lives.
- **Azure Monitor:** The Command Center. The umbrella service.
- **Application Insights:** The Detective. Specialized lens for application code.

**Speaker Notes:**
It's critical to understand the separation of duties here. Log Analytics is just storage; it's the filing cabinet. Azure Monitor is the overarching service for everything in Azure, including infrastructure. Application Insights is a specific feature within Azure Monitor designed purely for application code—like your C# or Python apps.

---

## Slide 5: Application Insights vs Azure Monitor
**Visual:** Comparison table.
**Text:**
| Feature | Azure Monitor | Application Insights |
| :--- | :--- | :--- |
| **Focus** | Infrastructure (VMs, Network) | App Code (Exceptions, Requests) |
| **Data** | Platform Metrics | Application Telemetry |

**Speaker Notes:**
To make it simple: if you want to know if a Virtual Machine is using 100% of its CPU, use Azure Monitor. If you want to know if a specific SQL query in your code threw a Null Reference Exception, use Application Insights. 

---

## Slide 6: ⚠️ EXAM ALERT: Storage ⚠️
**Background Color:** #D13438 (Red)
**Text:**
**EXAM ALERT:** Where is Application Insights data stored?
- Historically: It had its own isolated storage.
- **NOW (Exam Answer):** Log Analytics Workspace.
- All App Insights resources must be "Workspace-based".

**Speaker Notes:**
Pay close attention here. Microsoft retired the old "classic" Application Insights. On the exam, if they ask where Application Insights data is stored, the answer is always a Log Analytics Workspace. It does not store its own data anymore.

---

## Slide 7: Distributed Tracing
**Visual:** A flowchart showing a single Correlation ID passed from Web -> API -> DB.
**Text:**
- **The Problem:** Tracking one user's journey across many microservices.
- **The Solution:** A unique "Tracking Number" (Correlation ID).
- Passed in HTTP Headers to every service.

**Speaker Notes:**
When a request enters your system, it gets a unique ID, much like a FedEx tracking number. As it hops from the web app to the API, it passes that ID along. This is what allows Azure to stitch the logs together into a single cohesive story.

---

## Slide 8: W3C Trace Context Standard
**Visual:** Diagram showing HTTP Headers: `traceparent` and `tracestate`.
**Text:**
- The industry standard for passing correlation IDs.
- Replaces proprietary headers (`Request-Id`).
- Supported natively by Application Insights.

**Speaker Notes:**
For the exam, know that the industry has standardized on the W3C Trace Context. The headers used to pass the correlation ID are called `traceparent` and `tracestate`. This is how different vendor tools can all agree on the same tracking number.

---

## Slide 9: Application Map
**Visual:** Screenshot of an Azure Application Map showing green circles and red lines.
**Text:**
- Visual topology of your architecture.
- Auto-discovered by telemetry.
- Instantly spot bottlenecks (red lines) and slow dependencies.

**Speaker Notes:**
Application Map is the visual result of distributed tracing. You don't have to draw this; Azure draws it for you based on the telemetry. If your database is failing, you'll see a big red line pointing to it, allowing you to instantly isolate the problem without reading thousands of log lines.

---

## Slide 10: Introduction to KQL
**Visual:** A water filter funnel. Dirty water at the top, clean water at the bottom.
**Text:**
- Kusto Query Language (KQL).
- Read-only language for querying Log Analytics.
- Works by passing data through filters using the pipe `|` operator.

**Speaker Notes:**
Logs are massive and messy. KQL is how we filter them down. Think of it like a water filter. You start with a massive pool of raw data, and pass it through a series of pipes, filtering out the noise until you have the exact error message you need.

---

## Slide 11: KQL vs SQL
**Visual:** Side-by-side code blocks.
**Text:**
**SQL:**
```sql
SELECT name FROM users WHERE age > 18
```
**KQL:**
```kql
users | where age > 18 | project name
```

**Speaker Notes:**
If you know SQL, you can learn KQL in ten minutes. The biggest difference is the order. SQL starts with what you want (SELECT). KQL starts with the data source (users table), and then pipes that data into a filter, and finally projects the columns. Data source first, filtering second.

---

## Slide 12: ⚠️ EXAM ALERT: KQL Syntax ⚠️
**Background Color:** #D13438 (Red)
**Text:**
**EXAM ALERT:** 
- Drag-and-drop KQL questions are very common.
- Rule #1: The table name ALWAYS goes on the first line.
- Rule #2: Use `where` to filter, `project` to select columns, `summarize` to count.

**Speaker Notes:**
On the AZ-204 exam, you will likely see a question asking you to drag blocks of KQL code into the correct order. Never put a pipe or a command on the first line. The first line is always the table name, like `exceptions` or `requests`.

---

## Slide 13: Availability Tests (Synthetic Monitoring)
**Visual:** A map of the world with ping lines targeting a central server.
**Text:**
- **The Problem:** Server metrics are green, but external users can't reach the site.
- **The Solution:** Azure tests your site from outside datacenters.
- Simulates real user traffic.

**Speaker Notes:**
Sometimes your servers are perfectly healthy, but a firewall rule is blocking traffic from the outside world. Availability tests are like hiring a secret shopper to try and use your app from the outside. Azure literally pings your website from datacenters around the world.

---

## Slide 14: Types of Availability Tests
**Visual:** Bulleted list.
**Text:**
- **Standard Test (Ping):** Single HTTP/HTTPS request. Checks for 200 OK.
- **Custom TrackAvailability:** Run custom code, push results to App Insights.
- *(Note: Multi-step web tests are being deprecated).*

**Speaker Notes:**
Standard tests are the most common. They simply ping a URL and wait for a 200 OK response. If it takes too long, or returns a 500, it triggers an alert. Remember that the old multi-step UI recording tests are being retired, so focus on Standard tests for the exam.

---

## Slide 15: OpenTelemetry: The New Standard
**Visual:** A universal travel power adapter.
**Text:**
- Vendor-neutral standard for telemetry.
- Write once, export anywhere.
- Replaces proprietary SDKs (like the old App Insights SDK).

**Speaker Notes:**
We are shifting from proprietary SDKs to OpenTelemetry. Think of it as a universal travel adapter. Instead of locking your code into Azure by using the Microsoft SDK, you use the OpenTelemetry standard. If you later move to AWS or Datadog, you don't have to rewrite your telemetry code.

---

## Slide 16: The OpenTelemetry Pipeline
**Visual:** Three boxes: Instrument Code -> OpenTelemetry SDK -> Azure Monitor Exporter.
**Text:**
1. **Instrument:** Generate Spans and Metrics.
2. **Collect:** OTel SDK gathers the data.
3. **Export:** Send to a backend using an Exporter package.

**Speaker Notes:**
The architecture is simple. You instrument your code, the OpenTelemetry SDK collects that data, and then you attach an Exporter. The Exporter acts as a translator, taking the generic OTel data and formatting it perfectly for Azure Monitor. 

---

## Slide 17: Tracing AI Workloads (AI-200)
**Visual:** A taxi meter showing a high fare.
**Text:**
- AI calls are different: High latency, non-deterministic, cost based on Tokens.
- You must log Token Usage, not just success/failure.
- Identify which prompts are driving up costs.

**Speaker Notes:**
Shifting to the AI side. Monitoring an LLM call is different than monitoring a database. A database call should be fast and cheap. An AI call might take 10 seconds and cost a dollar. You must monitor token usage to understand your costs. It's like checking the taxi meter after a ride.

---

## Slide 18: Semantic Kernel & OpenTelemetry
**Visual:** Code snippet showing Semantic Kernel and OTel integration.
**Text:**
- Semantic Kernel natively supports OpenTelemetry.
- Automatically creates Spans for:
  - Tokens consumed.
  - Model used (e.g., gpt-4o).
  - Plugin execution time.

**Speaker Notes:**
If you are building AI apps with the Semantic Kernel, good news: it has OpenTelemetry built right in. When you turn it on, it automatically logs how many tokens every prompt used and how long it took the AI to respond, and ships that data right into Application Insights.

---

## Slide 19: Querying AI Telemetry in KQL
**Visual:** KQL snippet querying customDimensions for tokens.
**Text:**
```kql
dependencies
| where target contains "openai"
| extend tokens = customDimensions.completionTokens
| summarize sum(toint(tokens)) by bin(timestamp, 1d)
```
- Extract token data from `customDimensions` property bag.

**Speaker Notes:**
Because token counts are specific to AI, they aren't standard HTTP metrics. They are stored in a property bag called `customDimensions`. You use KQL to extract those token numbers, cast them to integers, and sum them up to figure out your daily AI usage costs.

---

## Slide 20: Module Summary Table
**Visual:** A summary table of key concepts.
**Text:**
| Concept | Key Takeaway |
| :--- | :--- |
| **Log Analytics** | The storage backend for telemetry data. |
| **App Insights** | The APM tool for monitoring code and exceptions. |
| **KQL** | Data source first -> `|` -> filter (`where`). |
| **OpenTelemetry** | The vendor-neutral standard replacing proprietary SDKs. |
| **AI Observability** | Track token usage in `customDimensions`. |

**Speaker Notes:**
To wrap up: Log Analytics is storage, App Insights is the APM tool. Know your basic KQL syntax and remember the pipe operator. OpenTelemetry is the future, and when monitoring AI, always track your tokens to manage costs. Next up, we'll head into the lab to build this out!
