# Module 12: Quiz - Observability

---

## Foundation Level

**Question 1**
You are designing a monitoring strategy for a new web application. You need a place to store all the telemetry data, including logs and metrics, for long-term querying.
Where is Application Insights telemetry data physically stored?
A) Azure SQL Database
B) Log Analytics Workspace
C) Azure Storage Account (Blob)
D) Application Insights Classic Storage

**Correct Answer:** B
**Explanations:**
- **A (Incorrect):** Telemetry is not stored in SQL; it is unstructured/semi-structured.
- **B (Correct):** Modern Application Insights resources are "Workspace-based", meaning all telemetry is stored in a centralized Log Analytics Workspace.
- **C (Incorrect):** While you can export data to Blob storage for archiving, it is not the primary queryable storage engine.
- **D (Incorrect):** Classic App Insights storage has been deprecated and retired.

---

**Question 2**
You are learning the Kusto Query Language (KQL) to search through application logs. You want to filter a table to only show errors.
Which character is used to pass data from one KQL operation to the next?
A) `>`
B) `&&`
C) `|` (Pipe)
D) `$`

**Correct Answer:** C
**Explanations:**
- **A (Incorrect):** Used for greater-than comparisons, not data piping.
- **B (Incorrect):** Used for logical AND operations.
- **C (Correct):** The pipe `|` character takes the output of the left side and feeds it as the input to the right side (e.g., `requests | where success == false`).
- **D (Incorrect):** Used for string interpolation or variables in some languages, not KQL piping.

---

**Question 3**
You are instrumenting a new microservice. Your company policy dictates that you must not lock your code into a specific cloud provider's proprietary monitoring SDK.
Which standard should you use to instrument your application telemetry?
A) WMI (Windows Management Instrumentation)
B) OpenTelemetry
C) Datadog SDK
D) Azure Monitor SDK

**Correct Answer:** B
**Explanations:**
- **A (Incorrect):** WMI is for Windows OS-level management, not application telemetry.
- **B (Correct):** OpenTelemetry is the open-source, vendor-neutral standard for logs, metrics, and traces.
- **C (Incorrect):** This is a proprietary SDK for a specific vendor.
- **D (Incorrect):** Microsoft recommends OpenTelemetry over their own proprietary SDKs now.

---

**Question 4**
You need to verify if an external API is currently online.
Which type of Availability Test simply sends a single HTTP request to check for a 200 OK status?
A) Standard Test (Ping)
B) Multi-step Web Test
C) Custom TrackAvailability Test
D) Load Test

**Correct Answer:** A
**Explanations:**
- **A (Correct):** A Standard Availability Test makes a single HTTP/HTTPS request to verify responsiveness.
- **B (Incorrect):** Multi-step tests execute a sequence of UI actions and are deprecated.
- **C (Incorrect):** Custom tests require you to write your own testing logic in code.
- **D (Incorrect):** Load testing stresses the server; availability testing just checks if it is alive.

---

**Question 5**
When tracking requests across multiple microservices, a unique identifier is passed in the HTTP headers. 
What is this identifier commonly called?
A) Session ID
B) Correlation ID
C) Tenant ID
D) JWT Token

**Correct Answer:** B
**Explanations:**
- **A (Incorrect):** Session IDs track a user's login session, not the specific backend telemetry trace.
- **B (Correct):** A Correlation ID (often implemented via W3C Trace Context) is the tracking number that links all parts of a distributed trace together.
- **C (Incorrect):** Tenant ID identifies a customer in a multi-tenant system.
- **D (Incorrect):** A JWT is used for authentication/authorization.

---

## Applied Level

**Question 6**
You are writing a KQL query to find all slow page loads in your web application. You need the results sorted so the slowest requests appear at the top.
Which KQL operator should you use?
A) `group by duration`
B) `order by duration desc`
C) `sort by duration asc`
D) `where duration > limit`

**Correct Answer:** B
**Explanations:**
- **A (Incorrect):** KQL uses `summarize` for grouping, not `group by`.
- **B (Correct):** `order by duration desc` sorts the results in descending order, placing the largest durations (slowest requests) at the top.
- **C (Incorrect):** Ascending order would put the fastest requests at the top.
- **D (Incorrect):** `where` is used for filtering, not sorting.

---

**Question 7**
Your e-commerce application consists of a Web Frontend, an Inventory API, and an external Payment Gateway. You need to visually identify which of these services has the highest failure rate.
Which Azure Monitor feature provides this visualization?
A) Application Map
B) Log Analytics Dashboard
C) Metrics Explorer
D) Smart Detection

**Correct Answer:** A
**Explanations:**
- **A (Correct):** Application Map provides a visual topology graph of your microservices, showing call paths, latency, and failure rates (red nodes/lines).
- **B (Incorrect):** While you can build charts here, it does not automatically draw the distributed topology.
- **C (Incorrect):** Metrics Explorer creates line charts and bar graphs for numerical data, not topology maps.
- **D (Incorrect):** Smart Detection uses machine learning to alert you to anomalies, it is not a visual map.

---

**Question 8**
You have an application that makes calls to Azure OpenAI. You want to monitor the total number of tokens consumed by the prompts over the last 24 hours.
Where in the Application Insights logs are token metrics typically stored?
A) The `duration` column in the `requests` table.
B) The `message` column in the `exceptions` table.
C) The `customDimensions` property bag in the `dependencies` or `requests` table.
D) The `performanceCounters` table.

**Correct Answer:** C
**Explanations:**
- **A (Incorrect):** Duration tracks time (latency), not token counts.
- **B (Incorrect):** Token usage is not an exception.
- **C (Correct):** AI-specific metrics like tokens and model names are custom telemetry injected into the `customDimensions` JSON property bag.
- **D (Incorrect):** Performance counters track OS metrics like CPU and RAM.

---

**Question 9**
You are migrating a .NET 8 application to use OpenTelemetry. You have added the `OpenTelemetry` package and configured your traces. However, no data is appearing in Azure Monitor.
What missing component must be added to translate the OpenTelemetry data into Azure Monitor's format?
A) Log Analytics Agent
B) Azure Monitor Exporter
C) Application Insights Classic SDK
D) Telegraf Exporter

**Correct Answer:** B
**Explanations:**
- **A (Incorrect):** The Log Analytics Agent is an infrastructure-level VM agent, not a code library.
- **B (Correct):** The OpenTelemetry architecture requires an "Exporter" (specifically `Azure.Monitor.OpenTelemetry.Exporter`) to send the neutral data to a specific backend like Azure.
- **C (Incorrect):** You do not mix the Classic SDK with OpenTelemetry.
- **D (Incorrect):** Telegraf is a different agent system, not the Azure Monitor exporter.

---

**Question 10 (Select all that apply)**
You want to set up an alert if your web application's homepage becomes unreachable. 
Which two of the following are valid locations from which Azure can run a Standard Availability Test? (Select two).
A) A Virtual Machine inside your private corporate network.
B) Microsoft's East US Datacenter.
C) A user's local laptop running the Azure CLI.
D) Microsoft's Japan East Datacenter.

**Correct Answers:** B, D
**Explanations:**
- **A (Incorrect):** Azure Availability Tests originate from the public internet (Microsoft datacenters). They cannot run *from* your private internal network unless you write a custom TrackAvailability test on an internal worker.
- **B & D (Correct):** Standard availability tests are run from a list of predefined global Azure regions.
- **C (Incorrect):** Azure does not hijack end-user laptops to run availability tests.

---

## Exam Level

**Question 11**
You are troubleshooting a performance issue in a complex microservices architecture. You have a Correlation ID: `12345-abcde`. You need to see every single log entry across all tables (requests, dependencies, exceptions) that shares this Correlation ID.
Which KQL query represents the most efficient way to search across the entire workspace for this specific ID?
A) `search "12345-abcde"`
B) `requests | join exceptions on correlationId`
C) `union * | where correlationId == "12345-abcde"`
D) `find in (requests, dependencies, exceptions) where correlationId == "12345-abcde"`

**Correct Answer:** A
**Explanations:**
- **A (Correct):** The `search` operator searches across all columns in all tables in the workspace for a specific string. It is the fastest way to find a needle in a haystack when you don't know exactly which table holds the data.
- **B (Incorrect):** Joining tables is computationally expensive and this only covers two tables.
- **C (Incorrect):** `union *` works, but `search` is optimized specifically for full-text searching across everything.
- **D (Incorrect):** `find` is a valid operator but `search` is the standard and most direct method for a global string lookup.

---

**Question 12**
You are writing a background worker application in .NET that runs every hour, processes files for 5 seconds, and then exits. You implemented OpenTelemetry, but you notice that no telemetry data ever arrives in Azure Monitor. You have verified the connection string is correct.
What is the most likely cause of the missing telemetry?
A) The Log Analytics workspace has reached its daily data cap.
B) You did not call `ForceFlush()` before the application exited.
C) OpenTelemetry does not support background worker processes.
D) The Correlation ID was not passed in the HTTP headers.

**Correct Answer:** B
**Explanations:**
- **A (Incorrect):** Possible, but less likely than B for a short-lived console app.
- **B (Correct):** Telemetry is batched in memory and sent asynchronously. If a short-lived process exits before the batch timer fires, the telemetry in memory is destroyed. You must explicitly call `ForceFlush()` before the program terminates.
- **C (Incorrect):** OpenTelemetry supports any type of application.
- **D (Incorrect):** Missing a correlation ID breaks distributed tracing, but standard logs and metrics would still appear.

---

**Question 13**
You are building an AI-powered chatbot using Semantic Kernel. You need to ensure that the execution time of individual AI plugins and the total token counts are recorded in Application Insights.
What is the recommended approach to achieve this?
A) Write custom `Stopwatch` code around every API call and manually log the time.
B) Enable OpenTelemetry support natively within Semantic Kernel and configure the Azure Monitor Exporter.
C) Install the Application Insights Classic SDK and use the `TrackDependency` method manually.
D) Configure Azure API Management in front of the OpenAI endpoint to log the payload.

**Correct Answer:** B
**Explanations:**
- **A (Incorrect):** Manual timing is error-prone and doesn't capture token usage cleanly.
- **B (Correct):** Semantic Kernel has native, built-in support for OpenTelemetry. Enabling it automatically generates detailed spans (including plugin execution times and token counts) without writing custom timing code.
- **C (Incorrect):** The Classic SDK is being phased out in favor of OpenTelemetry.
- **D (Incorrect):** While APIM can log requests, it is infrastructure overhead. The application code (Semantic Kernel) already has this capability built-in.

---

**Question 14**
You are reviewing a drag-and-drop question on the AZ-204 exam. You need to construct a KQL query that retrieves the top 10 longest-running requests.
Which is the correct order for the KQL statements?
A) `| project timestamp, name, duration` -> `requests` -> `| top 10 by duration`
B) `requests` -> `| top 10 by duration` -> `| project timestamp, name, duration`
C) `| top 10 by duration` -> `| project timestamp, name, duration` -> `requests`
D) `requests` -> `| project timestamp, name, duration` -> `| where top 10`

**Correct Answer:** B
**Explanations:**
- **A (Incorrect):** The table name must be first.
- **B (Correct):** 1. Start with the table (`requests`). 2. Use the `top` operator to sort by duration and limit to 10. 3. Use `project` to format the final output columns.
- **C (Incorrect):** The table name must be first.
- **D (Incorrect):** `where` is used for conditional filtering, not limiting rows by count.

---

**Question 15 (Select all that apply)**
You are configuring Distributed Tracing for a system that includes an Azure App Service web app and an Azure Functions backend. 
Which two HTTP headers are used by the W3C Trace Context standard to pass the tracing information between these services? (Select two).
A) `X-Correlation-Id`
B) `traceparent`
C) `Request-Id`
D) `tracestate`

**Correct Answers:** B, D
**Explanations:**
- **A (Incorrect):** This is a generic custom header often used historically, but it is not the W3C standard.
- **B (Correct):** `traceparent` contains the trace ID and the parent span ID.
- **C (Incorrect):** `Request-Id` was the proprietary header used by older versions of the Application Insights SDK.
- **D (Correct):** `tracestate` carries vendor-specific tracing information across boundaries.
