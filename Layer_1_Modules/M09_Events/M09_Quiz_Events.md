# Module 9 Quiz: Event-Driven Architecture

## Foundation Questions

**1. You need to implement an Azure service that pushes a lightweight notification when a new Virtual Machine is created in your resource group. The system receiving the alert just needs to know the state changed. Which service should you choose?**
A) Azure Service Bus Queue
B) Azure Event Grid
C) Azure Event Hubs
D) Azure Relay
*Correct Answer: B*
*Explanation:* 
- A is wrong because Service Bus uses a Pull model and is for heavy messaging.
- B is correct because Event Grid is designed for pushing lightweight notifications of state changes (like resource creation).
- C is wrong because Event Hubs is for massive telemetry data ingestion (e.g., millions of IoT sensors).
- D is wrong because Relay is for securely exposing on-premises services.

**2. What is the fundamental difference between an Event and a Message in Azure architecture?**
A) Events are XML, Messages are JSON.
B) Events expect a specific action to be taken; Messages do not.
C) Events are lightweight notifications of state changes; Messages contain raw data and expect specific processing.
D) Events are pulled; Messages are pushed.
*Correct Answer: C*
*Explanation:*
- A is wrong because both can be JSON.
- B is backwards.
- C is correct. Events are publisher-agnostic (Newspaper), Messages mandate specific processing (Addressed Letter).
- D is backwards. Events (Event Grid) are pushed. Messages (Service Bus) are pulled.

**3. In Azure Service Bus, what is the default behavior when a worker retrieves a message from a queue?**
A) Receive-and-Delete
B) Peek-Lock
C) Dead-Lettering
D) Auto-Forwarding
*Correct Answer: B*
*Explanation:*
- A is wrong because it is an optional setting that deletes the message instantly, risking data loss.
- B is correct. Peek-Lock hides the message for a default duration to allow processing without risking data loss if the worker crashes.
- C & D are advanced features, not default retrieval behaviors.

**4. You need to ensure that an incoming e-commerce order is processed by EXACTLY one backend server. Which routing mechanism should you use?**
A) Service Bus Topic
B) Event Grid Topic
C) Service Bus Queue
D) Notification Hub
*Correct Answer: C*
*Explanation:*
- A is wrong because Topics duplicate messages to multiple subscriptions (1:Many).
- C is correct because Queues enforce a strict 1:1 relationship. Only one receiver gets the message.

**5. What is the purpose of a Dead-Letter Queue (DLQ)?**
A) To permanently delete successfully processed messages.
B) To hold messages that cannot be processed successfully after a maximum number of delivery attempts.
C) To store expired API keys.
D) To archive old messages for compliance purposes.
*Correct Answer: B*
*Explanation:*
- B is correct. Poison messages that continually crash the worker are moved to the DLQ after `MaxDeliveryCount` is exceeded.
- A, C, and D describe archiving, security, and deletion, not DLQ behavior.

## Applied Questions

**6. A web application receives an image upload, puts a request on a Service Bus Queue, and returns a fast response to the user. A backend AI model retrieves the message and processes it. If the AI model crashes mid-processing, what happens to the message? (Assuming default queue settings)**
A) It is deleted permanently to protect the queue.
B) The message lock expires, and the message becomes available on the queue again for another worker to retry.
C) The web application receives a 500 Internal Server Error.
D) It is immediately forwarded to Azure Event Grid.
*Correct Answer: B*
*Explanation:*
- B is correct because the default behavior is Peek-Lock. If the worker crashes, it fails to call `Complete()`. The lock expires, and the message reappears.
- A describes Receive-and-Delete. C is wrong because the web app already returned a success response (decoupling). 

**7. You are building an Azure Service Bus application in .NET. You want the message to be deleted from the queue immediately upon retrieval to maximize throughput, even though data loss is acceptable. Which receive mode should you configure?**
A) `ReceiveMode.PeekLock`
B) `ReceiveMode.ReceiveAndDelete`
C) `ReceiveMode.DeadLetter`
D) `ReceiveMode.Session`
*Correct Answer: B*
*Explanation:*
- B is correct. Receive-and-Delete maximizes speed but offers At-Most-Once delivery, risking data loss if a crash occurs.

**8. Select all that apply: Which of the following scenarios are appropriate use cases for Azure Service Bus Topics?**
A) Sending a single notification that a user registered, which must update the CRM, the Marketing system, and the Analytics dashboard.
B) Processing financial bank transfers sequentially to prevent double-spending.
C) Implementing a Publish/Subscribe pattern.
D) Pushing a high-frequency stream of 10 million temperature readings per second.
*Correct Answer: A, C*
*Explanation:*
- A and C both describe 1:Many (Publish/Subscribe) scenarios perfect for Topics.
- B requires a Queue (1:1). D requires Event Hubs (massive telemetry ingestion).

**9. Your architecture features Azure Event Grid. An event handler (Azure Function) goes completely offline for 2 hours. What does Event Grid do with the events generated during this time?**
A) Deletes them instantly because Event Grid does not store events.
B) Automatically reroutes them to a Service Bus Queue.
C) Retries delivery based on a backoff schedule for up to 24 hours (default).
D) Pauses the Azure resources generating the events.
*Correct Answer: C*
*Explanation:*
- C is correct. While Event Grid is a push system, it does possess a retry mechanism using exponential backoff to handle temporary handler outages.

**10. You need to guarantee that all messages associated with `AccountID: 12345` are processed in the exact order they were sent. Which feature of Service Bus Queues must you enable?**
A) Duplicate Detection
B) Dead-Letter Queues
C) Message Sessions
D) Scheduled Delivery
*Correct Answer: C*
*Explanation:*
- C is correct. Message Sessions guarantee strict FIFO (First-In, First-Out) ordering for messages sharing the same Session ID.

## Exam-Level Questions

**11. You are designing an architecture for a hybrid application. The on-premises billing system must receive a guaranteed copy of an invoice when a cloud web app generates it. The cloud marketing system also needs a copy. You provision an Azure Service Bus Topic. How should you configure the architecture to ensure both systems receive the invoice?**
A) Create one Subscription on the Topic. Have both systems pull from that single subscription.
B) Create two separate Queues and abandon the Topic.
C) Create two separate Subscriptions on the Topic. Connect the billing system to Subscription A, and the marketing system to Subscription B.
D) Configure Event Grid to listen to the Service Bus Topic.
*Correct Answer: C*
*Explanation:*
- C is correct. A Topic duplicates the message into individual Subscriptions. Each backend system needs its own Subscription to independently pull the message.
- A is wrong because if both pull from the same subscription, it acts like a queue; they will steal messages from each other.

**12. A backend Azure Function processes messages from a Service Bus Queue using Peek-Lock. Occasionally, the AI processing takes 90 seconds, but the default Peek-Lock duration is only 60 seconds. Consequently, another worker pulls the same message, resulting in duplicate processing. How can you solve this with the LEAST amount of code changes?**
A) Change the receive mode to Receive-and-Delete.
B) Increase the `LockDuration` property on the Service Bus Queue to 2 minutes.
C) Move the messages to a Dead-Letter Queue immediately.
D) Use Event Hubs instead.
*Correct Answer: B*
*Explanation:*
- B is correct. The Queue configuration has a `LockDuration` property. Increasing it to 2 minutes (120 seconds) accommodates the 90-second AI processing time before the lock expires.
- A causes data loss. C and D do not solve the problem.

**13. Select all that apply: In Azure Service Bus, which actions explicitly remove a message from a Queue when using Peek-Lock mode?**
A) The client application calling `CompleteMessageAsync()`.
B) The client application calling `AbandonMessageAsync()`.
C) The client application calling `DeadLetterMessageAsync()`.
D) The Peek-Lock timer expiring.
*Correct Answer: A, C*
*Explanation:*
- A completely removes it because processing succeeded.
- C completely removes it from the main queue and places it in the DLQ.
- B and D release the lock, making the message visible again on the main queue (it is NOT removed).

**14. An Event Grid Subscription is configured to send events to a Webhook. For security reasons, you must prove that you own the Webhook endpoint before Event Grid will send events to it. What is this process called?**
A) Shared Access Signature validation
B) Endpoint Validation (Validation Code handshake)
C) Managed Identity Role Assignment
D) Event Hub Capture
*Correct Answer: B*
*Explanation:*
- B is correct. Event Grid requires a validation handshake (sending a validation code that your webhook must echo back) to prevent people from using Event Grid to DDoS random URLs.

**15. You are reviewing the Dead-Letter Queue of a Service Bus Queue and find 50 messages. You notice the `DeadLetterReason` is `MaxDeliveryCountExceeded`. What is the most likely cause?**
A) The messages expired before any worker attempted to process them (TTL expired).
B) The worker successfully processed the messages but forgot to call `Complete()`, causing them to retry until failure.
C) The Service Bus namespace exceeded its storage quota.
D) The sender lacked IAM permissions to send the messages.
*Correct Answer: B*
*Explanation:*
- B is correct. If a worker processes a message but crashes or forgets to call `Complete()`, the lock expires, and it retries. If this happens repeatedly up to the max count, it goes to the DLQ with `MaxDeliveryCountExceeded`.
- A would show a reason of `TTLExpiredException`.
