# Scenario Cards: Module 9 - Event-Driven Architecture

## SCENARIO 1: The Reactive Storage Notification (Beginner)
**Business Context:** Contoso Media is a digital marketing agency. Photographers upload raw image files to an Azure Blob Storage container all day long.
**The Problem:** The current system has a script that runs every 10 minutes to check if new images have arrived. If an image arrives at 10:01, it sits idle until 10:10. Clients are complaining about delays.
**Constraints:** 
1. The solution must process the image immediately upon upload.
2. The solution must not require continuous polling or waste CPU cycles.
3. Must be as cost-effective as possible.
**Your Task:** Choose the appropriate Azure service to trigger the processing logic instantly.
**Hints:** 
- Which service is "Push" vs "Pull"?
- Which service specializes in reacting to infrastructure state changes (like a Blob being created)?
**Solution:** You should implement **Azure Event Grid**. By configuring an Event Grid Subscription on the Blob Storage container, Event Grid will push an event directly to an Azure Function the exact millisecond the file is uploaded. This eliminates polling, reduces cost (serverless), and processes images instantly.
**Exam Connection:** Exam questions often contrast Event Grid and Service Bus. If the trigger is a "state change" (Blob created, Resource Group updated) and requires "push" mechanics, the answer is always Event Grid.

---

## SCENARIO 2: The E-Commerce Checkout Jam (Intermediate)
**Business Context:** Tailwind Toys sells popular children's toys. During the holidays, web traffic spikes by 10,000%. 
**The Problem:** The web application directly calls the inventory database and payment gateway sequentially. When 5,000 users click "Checkout" at once, the database locks up, the web servers time out, and carts are abandoned.
**Constraints:** 
1. The web application must return a success response to the user in under 1 second.
2. Orders must be processed exactly once (no double billing).
3. If the payment gateway goes down temporarily, orders must not be lost.
**Your Task:** Design a decoupled architecture using an Azure messaging service.
**Hints:** 
- Which service guarantees enterprise message delivery?
- Should you use a Queue (1:1) or a Topic (1:Many)?
**Solution:** Implement an **Azure Service Bus Queue**. The Web App takes the order and instantly drops a JSON message onto the queue, returning a fast "Order Processing" HTML page to the user. A backend worker pulls messages from the queue using Peek-Lock to process the payment. If the payment API is down, the queue safely stores the messages until it comes back online. We use a Queue (not a Topic) to ensure 1:1 delivery, avoiding double-billing.
**Exam Connection:** Knowing when to decouple with a Service Bus Queue is the most highly tested concept in AZ-204 messaging.

---

## SCENARIO 3: The Pub/Sub Notification Broadcast (Intermediate)
**Business Context:** Alpine Ski Resort has an emergency weather alert system.
**The Problem:** When an avalanche warning is issued, the core system needs to notify the Mobile App Push Server, the Email Server, and the Digital Signage Server. Currently, the core system makes three separate API calls. If the Email Server is down, the whole script crashes, and the digital signs never update.
**Constraints:** 
1. The sender must only send ONE message.
2. Multiple independent backend systems must receive a copy of the message.
3. If one backend system goes offline, the others must remain unaffected.
**Your Task:** Select and configure the appropriate Service Bus routing mechanism.
**Hints:** 
- You need a 1-to-Many architecture.
- Think about magazine subscriptions.
**Solution:** Use an **Azure Service Bus Topic**. The core weather system publishes a single message to the Topic. You create three separate **Subscriptions** (Mobile, Email, Signage). Service Bus duplicates the message to all three. If the Email system crashes, its specific subscription queue safely holds the message until it recovers, while Mobile and Signage continue operating flawlessly.
**Exam Connection:** The "Publish/Subscribe" pattern is synonymous with Service Bus Topics. 

---

## SCENARIO 4: The Poison Image File (Intermediate)
**Business Context:** Fabrikam AI provides automated medical image scanning. 
**The Problem:** A hospital uploads a corrupted `.dcm` x-ray file. The backend Azure Function pulls the message off the Service Bus Queue, tries to open the file, hits an unhandled exception, and crashes. The Peek-Lock expires, the message returns to the queue, the Function pulls it again, and crashes again. The system is in an infinite loop.
**Constraints:** 
1. The system must automatically break the infinite loop without human intervention.
2. The broken message must not be deleted; engineers need to inspect it later to patch the bug.
**Your Task:** Describe the built-in Service Bus mechanism that solves this.
**Hints:** 
- Where do undeliverable letters go at the post office?
- Check the `MaxDeliveryCount` property.
**Solution:** Rely on the **Dead-Letter Queue (DLQ)**. By default, Service Bus tracks delivery attempts. When `MaxDeliveryCount` is exceeded (e.g., 10 failures), Service Bus automatically moves the poisonous message out of the main queue and into the DLQ sub-queue. The main queue resumes processing healthy x-rays, and developers can inspect the DLQ at their convenience.
**Exam Connection:** "Poison messages" and "Infinite processing loops" are keywords that strictly map to Dead-Letter Queues.

---

## SCENARIO 5: Assuring Sequential Processing (Advanced)
**Business Context:** Woodgrove Bank is processing financial ledger updates. 
**The Problem:** A user deposits $100 (Message A) and then withdraws $50 (Message B) five seconds later. Because there are multiple backend worker nodes processing the queue in parallel, Worker 2 grabs Message B and processes the withdrawal before Worker 1 finishes processing the deposit. The account temporarily overdraws.
**Constraints:** 
1. Messages from the same user account MUST be processed in exact chronological order.
2. The system must still support multiple concurrent worker nodes for high throughput (Worker 1 processes User X, Worker 2 processes User Y).
**Your Task:** Configure the Service Bus Queue to guarantee ordered processing for specific groups of messages.
**Hints:** 
- You need a way to group messages logically (e.g., by AccountID).
- Look for an advanced Service Bus feature that guarantees FIFO.
**Solution:** Enable **Message Sessions** on the Service Bus Queue. When the sender creates the message, it sets the `SessionId` property to the user's Account ID. Service Bus guarantees that all messages with the same Session ID are delivered in exact First-In-First-Out (FIFO) order to a single worker node. This prevents race conditions for financial transactions while still allowing global scale.
**Exam Connection:** "Guaranteed ordering", "FIFO strictness", and "grouping related messages" always point to Message Sessions.
