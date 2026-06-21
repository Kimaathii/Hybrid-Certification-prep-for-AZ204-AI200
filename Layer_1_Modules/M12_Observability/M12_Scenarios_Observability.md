# Module 12: Scenario Cards - Observability

---

## SCENARIO 1: The Global Menu (Beginner)
**Business Context:** "BistroCloud" runs an online food ordering platform. They just launched a massive marketing campaign in Japan, the UK, and the USA simultaneously.
**The Problem:** The marketing team is worried that the website might crash under the load or be unreachable from certain countries. They want to know instantly if the site goes offline in any of these regions.
**Constraints:** 
1. The test must check the homepage from servers located in Tokyo, London, and Seattle.
2. The test must alert the team if the site takes longer than 2 seconds to load.
**Your Task:** Choose the correct Application Insights feature to configure this monitoring.
**Hints:** 
1. You are checking from the "outside in", rather than relying on internal server metrics.
2. What feature acts like a "secret shopper"?
**Solution:** You should configure a **Standard Availability Test**. Standard tests allow you to specify an endpoint (the homepage URL) and select specific geographic test locations (Tokyo, London, Seattle). You can configure the success criteria to mandate an HTTP 200 OK response within a 2-second timeout window.
**Exam Connection:** AZ-204 often asks you to choose between standard availability tests, custom track availability, and multi-step tests. For single URL checks from multiple regions, standard availability tests are always the correct answer.

---

## SCENARIO 2: The Bottleneck Hunt (Intermediate)
**Business Context:** "FinTech Fast" has a microservices architecture. When a user transfers money, the request hits a Web App, an Auth Service, a Ledger Database, and an external Fraud Detection API.
**The Problem:** Users are complaining that transferring money takes 15 seconds. The database team says the Ledger is fast. The Web team says the frontend is fast. You need to find the specific service causing the delay.
**Constraints:** 
1. You cannot spend hours reading raw text logs.
2. You need a visual representation of the entire request path.
**Your Task:** Identify the specific Azure Monitor feature used to visualize the flow and find the bottleneck.
**Hints:** 
1. What feature draws a topology graph based on correlation IDs?
**Solution:** You should use **Application Map**. By ensuring Distributed Tracing (W3C Trace Context) is enabled across all services, Application Insights will stitch the requests together. Application Map will draw a visual graph with nodes for the Web App, Auth, Ledger, and Fraud API. The lines between the nodes will show average latency. You will quickly see a red or thick line pointing to the Fraud API indicating it is responsible for the 15-second delay.
**Exam Connection:** Knowing that "Application Map" is the visual tool for distributed tracing, and that it relies on correlation headers, is heavily tested.

---

## SCENARIO 3: Tracking Token Taxes (Intermediate)
**Business Context:** "DocuSummarizer" built a .NET app that sends large PDFs to Azure OpenAI to generate 1-page summaries.
**The Problem:** Their Azure bill for OpenAI skyrocketed this month. They need to figure out exactly which users are submitting massive documents that consume the most tokens.
**Constraints:** 
1. You must find the sum of all prompt tokens used, grouped by User ID.
2. You must write a query to extract this data from Log Analytics.
**Your Task:** Determine how to query custom token metrics using Kusto Query Language (KQL).
**Hints:** 
1. Token counts are not standard HTTP metrics; where do custom AI metrics get stored in an Application Insights log entry?
2. What KQL operator is used for grouping and summing data?
**Solution:** You need to write a KQL query against the `dependencies` or `requests` table. Because token counts are custom telemetry, they are stored in the `customDimensions` property bag. 
```kql
dependencies
| where target contains "openai"
| extend tokens = customDimensions["ai.prompt.tokens"], userId = customDimensions["user.id"]
| summarize TotalTokens = sum(toint(tokens)) by tostring(userId)
```
**Exam Connection:** AI-200 focuses heavily on tracing AI costs. You must know that AI-specific telemetry like tokens and model names reside in the `customDimensions` bag and must be extracted using the `extend` operator before they can be summed using `summarize`.

---

## SCENARIO 4: The Proprietary Prison Break (Intermediate)
**Business Context:** "LegacyCorp" wrote a Java app 5 years ago using the native Application Insights SDK. They are now moving to a multi-cloud strategy and want to use Datadog for some apps and Azure Monitor for others.
**The Problem:** They do not want to rewrite their monitoring code every time they switch cloud providers or monitoring tools.
**Constraints:** 
1. The application code must be instrumented only once.
2. The telemetry format must be vendor-neutral.
**Your Task:** Design the new telemetry architecture for LegacyCorp.
**Hints:** 
1. Think of the "universal travel adapter" analogy.
**Solution:** LegacyCorp should migrate to **OpenTelemetry**. They will remove the proprietary Application Insights SDK and replace it with the OpenTelemetry API and SDK. They will then configure an Exporter. When running in Azure, they use the `Azure.Monitor.OpenTelemetry.Exporter`. If they move to another cloud, they simply swap the Exporter package without touching the core instrumentation code.
**Exam Connection:** AZ-204 tests your knowledge of modern telemetry standards. Recognize OpenTelemetry as the vendor-neutral answer for logging, metrics, and distributed tracing.

---

## SCENARIO 5: The Alerting Avalanche (Advanced)
**Business Context:** "HealthSync" processes millions of medical records nightly via a background worker process. 
**The Problem:** Last night, the database went down for 5 minutes. The worker process threw 50,000 `SqlException` errors. The IT team received 50,000 separate email alerts, crashing the IT director's inbox.
**Constraints:** 
1. You must still be alerted if a `SqlException` occurs.
2. You must receive exactly ONE email if the database goes down, not one email per exception.
**Your Task:** Configure the Alert Rule in Azure Monitor to prevent alert fatigue.
**Hints:** 
1. When configuring a log-search alert, how do you handle thousands of rows matching the query?
**Solution:** You must configure a **Log Search Alert Rule** with a specific evaluation configuration. Instead of triggering an alert for *every single result* row, you configure the alert to measure the **Count of results**. For example, you set the condition: "If the number of `SqlException` results is greater than 0 within the last 5 minutes, fire an alert." Azure will run the KQL query, see 50,000 rows, evaluate that 50,000 > 0, and fire a *single* alert summarizing the condition.
**Exam Connection:** Managing Alert Rules is a core AZ-204 objective. You must understand the difference between metric alerts and log alerts, and how to aggregate log alert results to prevent alert storms.
