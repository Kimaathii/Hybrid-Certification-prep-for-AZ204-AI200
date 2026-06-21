# M11 Slide Deck: Identity & Authorization (Hybrid Web + AI Edition)

**Design Guidelines:** Use Azure blue (#0078D4), white, and light grey. Keep text minimal on slides; put details in speaker notes.

---

## Slide 1: Title
**Title:** Module 11: Identity & Authorization
**Subtitle:** AZ-204 (App Security) & AI-200 (AI Infrastructure) - Weight: 15-20%
**Visual:** Microsoft Entra ID logo and a lock icon.

**Speaker Notes:**
Welcome to Module 11. Today we are tackling one of the most critical topics in cloud development: Identity and Security. In the modern cloud, the firewall is no longer the primary security boundary; identity is. By the end of this module, you will understand how to secure your apps and connect to AI services without ever using a password.

---

## Slide 2: The Problem This Solves
**Title:** The Password Problem
**Visual:** A hacker stealing a post-it note with "DB_PASSWORD=123" off a developer's monitor, compared to a sleek smart-badge system.

**Speaker Notes:**
Why do we need complex identity systems? Because hardcoded passwords are dangerous. If a developer hardcodes an API key in their code to connect to an AI service, anyone who can read that code can steal that key and run up your Azure bill. We need a system that issues temporary, secure access tokens instead of permanent passwords.

---

## Slide 3: Identity Provider vs Service Provider
**Title:** The Airport Analogy
**Visual:** 
- Government (Identity Provider) issuing a Passport.
- Airline (Service Provider) checking the Passport and issuing a Boarding Pass.

**Speaker Notes:**
Think of Entra ID as the government issuing a passport. It is your Identity Provider (IdP). The application you want to access is the airline, the Service Provider (SP). The application doesn't verify your DNA or fingerprint; it trusts the passport issued by the IdP. 

---

## Slide 4: Authentication (AuthN)
**Title:** Authentication: Who Are You?
**Visual:** A person showing their ID badge to a security guard at the front desk of a building.

**Speaker Notes:**
Authentication, often abbreviated as AuthN, answers the question: "Are you who you say you are?" When you type your username and password, or use multi-factor authentication, Entra ID is handling the Authentication. It proves your identity and lets you in the front door.

---

## Slide 5: Authorization (AuthZ)
**Title:** Authorization: What Can You Do?
**Visual:** The same person trying to swipe their ID badge on the server room door, but access is denied (red light).

**Speaker Notes:**
Authorization, abbreviated as AuthZ, answers: "Are you allowed to do this specific action?" You might be authenticated to be inside the building, but you are not authorized to enter the server room. In Azure, Authorization is handled by Azure Role-Based Access Control (RBAC), not Entra ID.

---

## Slide 6: ⚠️ EXAM ALERT: AuthN vs AuthZ
**Design:** Red background (#D13438), White text.
**Title:** EXAM TRAP: AuthN vs AuthZ
**Text:** 
- Cannot log in? = Authentication issue (Entra ID).
- Logged in but cannot see the database? = Authorization issue (Azure RBAC).

**Speaker Notes:**
Pay close attention here for the exam. You will see troubleshooting questions where a user successfully logs into an application but gets an "Access Denied" error when trying to read a storage account. The trap is selecting an answer about Entra ID passwords. The correct answer will always be about checking Azure RBAC roles, because the issue is Authorization, not Authentication.

---

## Slide 7: Giving Applications an Identity
**Title:** Humans Have Accounts, Apps Have Registrations
**Visual:** A human user with a profile picture vs a robot holding a certificate.

**Speaker Notes:**
We know how humans log in. But how does an automated background job or a web API log in? Applications need identities too. We can't give them a human username and password, because apps can't complete MFA prompts. We need a secure, programmatic way for apps to authenticate.

---

## Slide 8: App Registrations
**Title:** App Registrations: The Master Blueprint
**Visual:** A blueprint document representing an application globally.

**Speaker Notes:**
The first step in giving an app an identity is creating an App Registration in Entra ID. This is the global definition of your application. It tells Microsoft, "I have built an application, here is its name, and here are the permissions it might eventually need."

---

## Slide 9: Service Principals
**Title:** Service Principals: The Local Badge
**Visual:** The blueprint from the previous slide printing out a local security badge.

**Speaker Notes:**
While the App Registration is the global blueprint, the Service Principal is the actual local instance of that identity in your specific Azure subscription. Think of it as the physical badge printed out for the app so it can access resources locally. When you assign RBAC permissions, you assign them to the Service Principal.

---

## Slide 10: Managed Identities
**Title:** The Ultimate Solution: Managed Identities
**Visual:** A robot automatically generating its own temporary badges, discarding passwords.

**Speaker Notes:**
Managing App Registrations and Service Principals manually means you still have to handle client secrets (passwords) for the app. Enter Managed Identities. Azure handles the identity, the secrets, and the automatic password rotation for you. Your code never sees a password. This is the standard for modern Azure development.

---

## Slide 11: System-Assigned Managed Identity
**Title:** System-Assigned Managed Identity
**Visual:** A keycard glued permanently to a specific server rack.

**Speaker Notes:**
A System-Assigned Identity is tied 1:1 to an Azure resource. If you enable it on a Web App, that Web App gets a dedicated identity. The lifecycle is strictly tied: if you delete the Web App, Azure automatically deletes the identity. It cannot be shared with any other resource.

---

## Slide 12: User-Assigned Managed Identity
**Title:** User-Assigned Managed Identity
**Visual:** A universal master key that can be handed to multiple servers, standing independently.

**Speaker Notes:**
A User-Assigned Identity is created as a standalone Azure resource. You can assign this single identity to multiple Web Apps or VMs. Its lifecycle is independent. If you delete the Web Apps using it, the identity remains safe in Azure until you explicitly delete it.

---

## Slide 13: ⚠️ EXAM ALERT: Identity Lifecycle
**Design:** Red background (#D13438), White text.
**Title:** EXAM TRAP: Deleting Resources
**Text:** 
- Need identity to survive resource deletion? -> User-Assigned.
- Need identity shared across 5 VMs? -> User-Assigned.
- Need strict 1:1 mapping? -> System-Assigned.

**Speaker Notes:**
This is guaranteed to be tested. When you see words like "shared across multiple resources" or "must persist if the VM is deleted", immediately look for User-Assigned Managed Identity in the multiple-choice options.

---

## Slide 14: Microsoft Authentication Library (MSAL)
**Title:** Don't Write Security Code. Use MSAL.
**Visual:** A developer pushing away a messy whiteboard of OAuth math, embracing a neat MSAL toolkit.

**Speaker Notes:**
Authentication protocols like OAuth 2.0 and OpenID Connect are complex. You should never write raw HTTP requests to negotiate tokens. Microsoft provides the Microsoft Authentication Library (MSAL). It handles token caching, refreshing, and secure storage with just a few lines of code.

---

## Slide 15: Public vs Confidential Clients
**Title:** Where is your app running?
**Visual:** A mobile phone (Public) vs a locked server rack (Confidential).

**Speaker Notes:**
MSAL separates apps into two categories. Public clients run on devices you don't control, like a user's mobile phone or browser. You cannot hide a secret key there. Confidential clients run on servers you control, like an Azure Web App. You can safely store secrets on a confidential client.

---

## Slide 16: [AI-200] Passwordless AI Architecture
**Title:** AI Without API Keys
**Visual:** Azure Web App -> (Managed Identity) -> Azure OpenAI. (Red X over a traditional API Key).

**Speaker Notes:**
Historically, AI integration required copying an API key from Azure OpenAI and pasting it into your app. If leaked, hackers steal your AI quota. By using a Managed Identity, your Web App authenticates securely with Entra ID to get a temporary token to call the AI service. The API key is entirely eliminated from your code.

---

## Slide 17: DefaultAzureCredential
**Title:** Magic Code: DefaultAzureCredential
**Visual:** Code snippet showing `new DefaultAzureCredential()` with arrows pointing to Local Dev and Azure Prod.

**Speaker Notes:**
How do we write code for this? The Azure SDK provides the `DefaultAzureCredential` class. It is brilliant because it adapts. When you run your code locally on your laptop, it uses your personal Visual Studio login. When deployed to Azure, it automatically switches to using the Managed Identity. Zero code changes required!

---

## Slide 18: Securing Custom AI APIs
**Title:** Protecting Your Own AI Wrappers
**Visual:** Front-End App -> Entra ID (Token) -> Custom AI API.

**Speaker Notes:**
If you build a custom API that calls AI models, you must protect your API from unauthorized callers. You achieve this by creating an App Registration for your API, defining an App Role, and requiring the Front-End app to acquire an Entra ID token before it is allowed to call your API endpoint.

---

## Slide 19: Putting it all together
**Title:** The Secure Flow
**Visual:** Flowchart showing User -> Web App (AuthN) -> RBAC check (AuthZ) -> MSAL token acquisition -> AI Service access via Managed Identity.

**Speaker Notes:**
Let's review the complete flow. A user authenticates via Entra ID. The web app uses MSAL to handle the token. The web app then uses its System-Assigned Managed Identity via `DefaultAzureCredential` to call Azure OpenAI without keys. Azure RBAC ensures the web app is authorized to make that call. Complete, zero-trust security.

---

## Slide 20: Module Summary
**Title:** What We Covered
**Visual:** Summary Table.

| Concept | Description | Analogy |
|---------|-------------|---------|
| Entra ID | Identity Provider (IdP) | Government issuing passport |
| AuthN | Who are you? | Checking the ID badge |
| AuthZ (RBAC)| What can you do? | Swiping badge on secure door |
| System Identity | 1:1, deleted with resource | Keycard glued to room |
| User Identity | Shared, independent lifecycle | Universal master key |
| MSAL | Library to get tokens | Automatic translator app |
| DefaultAzureCredential | Code to go passwordless | Auto-switching badge |

**Speaker Notes:**
Here is your cheat sheet for the exam. Remember the difference between Authentication and Authorization, and memorize the lifecycle differences of Managed Identities. In our upcoming lab, we will build exactly this: a passwordless application calling an AI service.
