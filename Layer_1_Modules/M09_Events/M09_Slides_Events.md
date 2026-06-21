# Module 9: Event-Driven Architecture
## Slide Deck (20 Slides)

<!-- slide -->
## Slide 1: Event-Driven Architecture
**Exam Domain:** AZ-204: Message/Event Routing
**Weight:** 15-20%
**Learning Objectives:**
- Distinguish between Events and Messages
- Architect solutions using Azure Event Grid
- Design decoupled systems with Azure Service Bus
- Implement asynchronous AI pipelines using Queues

**Speaker Notes:**
Welcome to Module 9. Today we are tackling one of the most critical concepts in modern cloud architecture: Event-Driven systems. If you've ever wondered how Amazon handles millions of orders on Black Friday without crashing, the answer is queues and messages. Let's dive in.

---
<!-- slide -->
## Slide 2: The Problem This Service Solves
### Tightly Coupled Systems
- **Frontend** waits for **Backend**
- **Result:** Web pages freeze, timeouts occur, systems crash under load.
- **AI Context:** AI models take seconds/minutes to process. You cannot make a web browser wait that long.

**Speaker Notes:**
Why do we need event routers and messaging systems at all? Think about the problem. If a user clicks 'Process AI Video', and your web app literally waits for the AI model to finish rendering for 5 minutes, the browser will time out. The frontend is tightly coupled to the backend. We need to cut that cord.

---
<!-- slide -->
## Slide 3: The Solution: Decoupling
- Introduce a **Middleman** (Queue or Event Router)
- Web App sends request to the Middleman.
- Web App immediately tells user: "Processing!"
- Backend pulls work from the Middleman at its own pace.

**Speaker Notes:**
The solution is decoupling. By putting a queue in the middle, the web app just drops off a ticket and goes back to work. The backend AI workers pull the tickets and work at their maximum safe speed without crashing.

---
<!-- slide -->
## Slide 4: Events vs. Messages
| Feature | Event | Message |
|---------|-------|---------|
| **Analogy** | Newspaper | Addressed Letter |
| **Publisher Expectation** | Doesn't care who reads it | Expects specific processing |
| **Payload** | Lightweight (e.g., "File Uploaded") | Heavy (e.g., "Invoice Data") |
| **Azure Service** | Azure Event Grid | Azure Service Bus |

**Speaker Notes:**
This is a massive exam trap. Events and Messages are different. An Event is a broadcast: "Hey, a file was created!" The sender doesn't care if anyone is listening. A Message is a command: "Process this $500 payment." The sender absolutely cares that it is processed and not lost.

---
<!-- slide -->
## Slide 5: Azure Event Grid
### The Push Model
- **Reactive Programming:** Code runs exactly when something happens.
- **No Polling:** No more checking "is it ready yet?"
- **Fully Serverless:** Scales instantly to millions of events.

**Speaker Notes:**
Event Grid is our Event handler. It uses a Push model. Think of it like a push notification on your phone. You don't have to keep opening the app to check for messages; the app pushes the alert to your screen the second it happens.

---
<!-- slide -->
## Slide 6: Event Grid Architecture
- **Event Source:** Blob Storage, Resource Group, Custom App
- **Topic:** Where the source sends the event
- **Subscription:** The filter (e.g., "Only .jpg files")
- **Event Handler:** Azure Function, Webhook, Event Hub

**Speaker Notes:**
This is the anatomy of Event Grid. The Source is where it happens. The Topic is the category. Your code creates a Subscription to listen to that Topic. And the Handler is the code that actually runs when the event fires.

---
<!-- slide -->
## Slide 7: ⚠️ EXAM ALERT: Event Grid
**Red Background / White Text**
If an exam question asks for:
- "Lightweight notifications"
- "Reacting to state changes"
- "No polling"
**Answer: Azure Event Grid**

**Speaker Notes:**
Pay close attention here. Microsoft loves testing your ability to choose between Event Grid and Service Bus. If the scenario involves reacting to an Azure infrastructure change, like a VM starting or a Blob being created, it's Event Grid every time.

---
<!-- slide -->
## Slide 8: Azure Service Bus
### The Pull Model
- Enterprise-grade message broker.
- Secures high-value data (financials, healthcare, heavy AI workloads).
- **Pull Model:** Receiver pulls when ready.

**Speaker Notes:**
Now we switch to Azure Service Bus. This is for Messages. It's a highly secure, reliable post office. Senders drop messages in, and receivers pull them out. Because it's a pull model, the backend is never overwhelmed. It pulls exactly what it can handle.

---
<!-- slide -->
## Slide 9: Service Bus: Queues
- **1-to-1 Relationship**
- First In, First Out (FIFO)
- Exactly ONE receiver processes the message.
- *Use Case:* Processing e-commerce payments.

**Speaker Notes:**
Within Service Bus, we have two routing methods. The first is a Queue. A Queue is strictly 1-to-1. If you have 5 backend servers listening to a queue, only ONE of them will get the message. This prevents double-billing a customer.

---
<!-- slide -->
## Slide 10: Service Bus: Topics & Subscriptions
- **1-to-Many Relationship**
- Publish / Subscribe pattern.
- EVERY subscription gets a copy of the message.
- *Use Case:* Sending an order to the Payment API, Inventory API, and Email API simultaneously.

**Speaker Notes:**
The second method is a Topic. Topics are 1-to-Many. When a message lands in a topic, Service Bus copies it and drops it into multiple Subscriptions. This is perfect when multiple independent systems need to know about the same event without interfering with each other.

---
<!-- slide -->
## Slide 11: AI Pipeline Architecture
**Web App -> Service Bus Queue -> AI Worker -> Cosmos DB**
1. User requests AI image generation.
2. Web app puts request on Queue.
3. Web app returns 202 Accepted.
4. AI Worker pulls queue, processes 30 seconds.
5. Saves to DB.

**Speaker Notes:**
This is how we merge AZ-204 with AI-200. Modern AI takes time. We use Service Bus Queues to hold the AI requests. This creates an asynchronous pipeline. The user gets a loading spinner on the web app, while the heavy lifting happens in the background.

---
<!-- slide -->
## Slide 12: Receiving Messages: Receive-and-Delete
- Message is pulled by worker.
- Message is instantly deleted from the Queue.
- **Risk:** If worker crashes, data is lost forever.
- "At-Most-Once" delivery.

**Speaker Notes:**
When pulling messages, we have two modes. First is Receive and Delete. It's fast, but dangerous. The second the worker grabs the message, the Queue deletes it. If the worker's CPU crashes 2 seconds later, that message is gone forever.

---
<!-- slide -->
## Slide 13: Receiving Messages: Peek-Lock (Default)
- Message is pulled by worker.
- Message is **Locked** (hidden) for 1 minute.
- Worker processes it, then sends `Complete()`.
- If worker crashes, lock expires, message reappears.
- "At-Least-Once" delivery.

**Speaker Notes:**
Peek-Lock is the default and the safest way to read messages. It checks out the message like a library book. It's locked. If the worker succeeds, it tells the queue to delete it. If the worker dies, the queue notices the lock expired and puts the message back for someone else to process.

---
<!-- slide -->
## Slide 14: ⚠️ EXAM ALERT: Peek-Lock
**Red Background / White Text**
If an exam scenario requires **Guaranteed Delivery** and must survive worker node crashes, you MUST select **Peek-Lock**.

**Speaker Notes:**
Exam tip: Always default to Peek-Lock for enterprise applications. Only use Receive-and-Delete for things like IoT temperature sensors where missing one reading out of a thousand doesn't matter.

---
<!-- slide -->
## Slide 15: Poison Messages
- What happens if a message is corrupted?
- The worker tries to process it, crashes, lock expires.
- Message reappears. Worker pulls it again. Crashes again.
- Result: **Infinite Loop** blocking the queue.

**Speaker Notes:**
Let's talk about failures. Imagine someone uploads a corrupted PDF to our AI analyzer. The worker pulls it, tries to read it, throws a fatal error, and crashes. The Peek-Lock expires. The PDF goes back on the queue. The next worker pulls it, and also crashes. This is a poison message, and it will destroy your system.

---
<!-- slide -->
## Slide 16: Dead-Letter Queue (DLQ)
- The Quarantine Zone.
- If a message fails `MaxDeliveryCount` times (default 10).
- Service Bus automatically moves it to the DLQ.
- Clears the main queue for healthy traffic.

**Speaker Notes:**
To solve poison messages, Service Bus has a Dead Letter Queue. If a message fails 10 times, Service Bus says "Enough is enough," rips it out of the main queue, and dumps it into the DLQ. The system stays healthy, and a human engineer can review the DLQ later to see what went wrong.

---
<!-- slide -->
## Slide 17: Securing Service Bus
- **Azure Role-Based Access Control (RBAC):** Azure Service Bus Data Owner.
- **Managed Identities:** Web App uses its identity, no passwords in code!
- **Shared Access Signatures (SAS):** Used for legacy or external clients.

**Speaker Notes:**
Security is paramount. Never put a connection string with a password in your code. Give your Web App a Managed Identity, and grant that identity the 'Service Bus Data Sender' role. It's secure, automatic, and passwordless.

---
<!-- slide -->
## Slide 18: Message Sessions
- Need guaranteed **Ordered Delivery**?
- Default Queues are First In, First Out (mostly), but at scale, order can get mixed up.
- **Sessions** group messages together to guarantee absolute order (FIFO).

**Speaker Notes:**
If you absolutely must guarantee that Message A is processed before Message B—like processing bank deposits before withdrawals—you enable Message Sessions.

---
<!-- slide -->
## Slide 19: Advanced Features
- **Scheduled Delivery:** "Don't show this message on the queue until 5:00 PM."
- **Message Deferral:** "I can't process this right now, put it aside, but don't dead-letter it."
- **Auto-Forwarding:** Automatically move messages from a Queue to a Topic.

**Speaker Notes:**
Just briefly touching on some advanced features. Scheduled delivery is great for reminder emails. Deferral is useful when the worker is waiting for a database lock to release but doesn't want to fail the message.

---
<!-- slide -->
## Slide 20: Module Summary Table
| Service | Model | Use Case | Guarantee |
|---------|-------|----------|-----------|
| **Event Grid** | Push | State changes (Blobs) | At-least-once |
| **Service Bus Queue** | Pull (1:1) | E-commerce, AI buffers | Peek-Lock (No loss) |
| **Service Bus Topic** | Pull (1:Many) | Pub/Sub, Multiple backends | Peek-Lock (No loss) |

**Speaker Notes:**
We made it. Take a screenshot of this table. Event grid is push, Service Bus is pull. Queues are 1 to 1, Topics are 1 to Many. Master this table and you will ace the messaging portion of the AZ-204 exam. See you in the lab!
