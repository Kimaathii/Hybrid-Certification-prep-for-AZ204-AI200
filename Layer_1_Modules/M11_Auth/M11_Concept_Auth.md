# Module 11: Identity & Authentication (Hybrid Web + AI Edition)

**Exam Domain:** AZ-204 (App Security) + AI-200 (AI Infrastructure)  
**Weight:** 15–20%  
**Estimated Study Time:** 2.5 hours  
**Prerequisites:** AZ-900 Cloud Concepts, Basic Web Architecture (Client/Server)

---

## The Problem This Module Solves

Imagine you manage a secure office building. You have employees who need to enter, but you also have delivery drivers, maintenance systems, and automated robots. If you don't have a central security desk, you would have to give everyone a universal master key. If a key is lost, your whole building is compromised. If an employee quits, you have to track them down and take their key back.

In the cloud, your applications, databases, and AI services are the "rooms" in your building. The "people" trying to get in are human users, but they are also *other applications*. For example, if a web app needs to talk to a database or an AI service, developers used to hardcode a password or an API key (a master key) directly into the web app's code. This is incredibly dangerous. If a hacker sees the code, they get the master key.

**The Solution:** A centralized security desk that hands out temporary, heavily restricted visitor badges instead of permanent keys. In Azure, this centralized security desk is called **Microsoft Entra ID**. By the end of this module, you will learn how to build applications that never use passwords or API keys.

---

## 1. Microsoft Entra ID: Identity Provider vs Service Provider

Before we secure applications, we must understand the core system managing identities.

**Analogy:** Think of an airport. The government issues your passport (Identity Provider). The airline checks your passport and gives you a boarding pass to let you on the plane (Service Provider).

- **Identity Provider (IdP):** The system that proves you are who you say you are. It issues the "passport." In Azure, **Microsoft Entra ID** (formerly Azure Active Directory) is your IdP. It stores user accounts, passwords, and security policies.
- **Service Provider (SP):** The application or resource you are trying to access. The SP doesn't know your password; it trusts the IdP. When you try to log in, the SP redirects you to the IdP. The IdP verifies you and sends you back with a digital "boarding pass" (a token).

> 💡 **KEY CONCEPT**
> Microsoft Entra ID (formerly Azure AD) is Microsoft's cloud-based Identity and Access Management (IAM) service. It acts as the Identity Provider (IdP) for all Azure services and your custom applications.

✅ **CHECKPOINT:** If an application asks Microsoft Entra ID to verify a user's password, is the application acting as the IdP or the SP? *(Answer: The application is the Service Provider. Entra ID is the Identity Provider.)*

---

## 2. Authentication (AuthN) vs Authorization (AuthZ)

It is critical to distinguish between these two steps, as they use different Azure services.

**Analogy:** 
- **Authentication (Who are you?):** You show your ID at the front desk of an office building. The guard checks your ID and gives you a badge. You are now authenticated.
- **Authorization (What can you do?):** You take your badge and try to swipe it on the door to the Server Room. The door flashes red and stays locked. You were authenticated (allowed in the building), but not authorized (allowed in the server room).

### Technical Definition
- **Authentication (AuthN):** The process of proving an identity. This is handled by **Microsoft Entra ID**. It answers: *Are you a valid user/app?*
- **Authorization (AuthZ):** The process of granting or denying access to a specific resource after authentication. This is handled by **Azure Role-Based Access Control (RBAC)**. It answers: *Does this valid user have permission to read this specific database?*

> 🚨 **EXAM ALERT**
> The AZ-204 exam will present scenarios where a user can log in but cannot see a database. The trick: The issue is NOT Authentication (Entra ID). The issue is Authorization (Azure RBAC). Always separate "logging in" from "accessing data".

---

## 3. App Registrations & Service Principals

How do we give an identity to an application instead of a human?

**Analogy:** When a new catering company is hired to deliver food to your office, the CEO of the catering company signs a contract with your company. But the CEO doesn't deliver the food. Instead, individual delivery drivers are given specific delivery badges to enter the building.

- **App Registration:** The master contract between your application and Entra ID. You create an App Registration to tell Entra ID, "I have an application that exists." This is the global definition of your app.
- **Service Principal:** The local delivery badge. When your application actually needs to access resources in your specific Azure subscription, Entra ID creates a Service Principal. This is the local identity created from the App Registration that Azure RBAC uses to grant permissions.

> 💡 **KEY CONCEPT**
> Think of the **App Registration** as the blueprint or class definition, and the **Service Principal** as the actual instance of that class in your Azure directory. You assign RBAC permissions to the Service Principal, not the App Registration.

---

## 4. Managed Identities (System-assigned vs User-assigned)

Now we reach the most important security concept in Azure development.

**Analogy:** 
- A **System-assigned** identity is like a hotel room keycard that is permanently attached to a specific hotel room. When the room is destroyed or renovated, the keycard is destroyed with it. No one else can use it.
- A **User-assigned** identity is like a universal master key that you issue to a specific trusted maintenance worker. The worker can use it on multiple rooms. If the worker leaves, you can take the key back and give it to a different worker. The key has a life of its own, separate from the rooms.

### Technical Definition
A **Managed Identity** is a specialized type of Service Principal that Azure manages completely for you. You do not have to handle credentials, passwords, or secrets. It provides an automatically managed identity in Entra ID for applications to use when connecting to resources that support Microsoft Entra authentication.

There are two types:

**1. System-Assigned Managed Identity:**
- Created directly on the Azure resource (e.g., a specific Azure Web App or Virtual Machine).
- **Lifecycle:** Tied to the lifecycle of the resource. If you delete the Web App, the identity is automatically deleted.
- **Sharing:** Cannot be shared. It belongs only to that one specific resource.

**2. User-Assigned Managed Identity:**
- Created as a standalone Azure resource first, then assigned to one or more Azure services.
- **Lifecycle:** Independent lifecycle. If you delete the Web App using it, the identity still exists and can be used by other resources.
- **Sharing:** Can be shared across multiple resources. (e.g., You have 10 Virtual Machines that all need the exact same access to an AI service. You create ONE User-Assigned identity and attach it to all 10 VMs).

> 🚨 **EXAM ALERT**
> **KNOW THE LIFECYCLE DIFFERENCE.** If an exam question asks: "You have an application spread across 5 VMs. You want them all to access a storage account using the same identity. When a VM is deleted, the identity must NOT be deleted." The answer is **User-assigned Managed Identity**. 

✅ **CHECKPOINT:** You create an Azure Function and enable a System-Assigned Managed Identity. You later delete the Azure Function. What happens to the identity? *(Answer: It is permanently deleted along with the Azure Function.)*

---

## 5. Microsoft Authentication Library (MSAL)

How do we actually write code to perform authentication?

**Analogy:** If you want to order food in a foreign country where you don't speak the language, you use a translation app. You type what you want in English, and the app speaks the complex foreign language to the waiter.

In the cloud, authenticating with Entra ID involves complex security protocols (OAuth 2.0, OpenID Connect). Instead of writing thousands of lines of code to handle these protocols manually, you use **MSAL (Microsoft Authentication Library)**.

**Technical Definition:** MSAL is a code library provided by Microsoft that enables developers to acquire tokens from Microsoft Entra ID to access secured web APIs. It handles the heavy lifting: token caching, silent token refreshing, and standard protocol interactions.

### MSAL Core Concepts
- **Public Client Applications:** Apps running on devices where secrets cannot be hidden safely (e.g., Desktop apps, Mobile apps, single-page web apps).
- **Confidential Client Applications:** Apps running on secure servers where secrets can be hidden (e.g., Web Apps, Web APIs, daemon background worker services).

> 💡 **KEY CONCEPT**
> You will almost never write raw HTTP requests to authenticate against Entra ID. You will always import the MSAL SDK for your language (e.g., `Microsoft.Identity.Client` in .NET) to acquire tokens.

---

## 6. [AI-200 ADDITION] Passwordless AI Architecture

Historically, if your web app needed to call Azure OpenAI, you would go to the Azure Portal, copy the `API_KEY`, and paste it into your app's configuration. This is a massive security flaw if that key is leaked. 

We can now build a **Passwordless AI Architecture** using Managed Identities.

**The Workflow:**
1. Enable a **System-Assigned Managed Identity** on your Azure Web App.
2. Go to your Azure OpenAI service and assign a role (e.g., `Cognitive Services OpenAI User`) to the Web App's identity using Azure RBAC.
3. In your code, instead of providing an API Key, you use the `DefaultAzureCredential` class from the Azure SDK.

**How `DefaultAzureCredential` works:**
It is a magical class that automatically looks for an identity in its environment. When running locally on your laptop, it uses your personal Visual Studio or Azure CLI login. When deployed to Azure, it detects the Managed Identity on the Web App and uses that automatically! No code changes required between local testing and production.

```csharp
// C# Example: Passwordless AI Connection
using Azure.Identity;
using Azure.AI.OpenAI;

// 1. Create a credential object. 
// DefaultAzureCredential automatically finds the Managed Identity in production
// or your local developer credentials during local testing.
var credential = new DefaultAzureCredential();

// 2. Connect to the OpenAI Service using the endpoint URL and the credential
// Notice: NO API KEY IS USED HERE!
var endpoint = new Uri("https://my-ai-resource.openai.azure.com/");
var client = new OpenAIClient(endpoint, credential);

// 3. Make the API call
var response = await client.GetCompletionsAsync("my-model-deployment", "Hello AI!");
```

💰 **COST WARNING**
Managed Identities are free! There is no additional cost to use Entra ID for standard authentication to Azure Services like OpenAI or Storage.

---

## 7. [AI-200 ADDITION] Securing a Custom AI API

What if *you* build a custom API that wraps your AI logic, and you only want specific front-end applications to be able to call it?

**The Problem:** You built `https://my-smart-ai-api.azurewebsites.net`. Right now, anyone on the internet can call it.

**The Solution:**
1. Create an **App Registration** for your Custom AI API. This tells Entra ID, "I have an API that needs protecting."
2. In this App Registration, define an **App Role** (e.g., `AI.Caller`).
3. Create an **App Registration** for your Front-End Web App.
4. Grant the Front-End Web App permission to use the `AI.Caller` role on your API.
5. Require Entra ID authentication on your Custom API.

Now, when the Front-End App wants to call the API, it must first ask Entra ID for a token. Entra ID checks the permissions, sees the Front-End is allowed, and issues a token. The Front-End attaches this token to the HTTP request. Your Custom API validates the token and allows the request.

---

## What We Covered (Checklist)
- [x] **Entra ID Basics:** Understood the difference between an Identity Provider (issuer of passports) and a Service Provider (checker of passports).
- [x] **AuthN vs AuthZ:** Authentication is verifying *who* you are (Entra ID). Authorization is verifying *what* you can do (Azure RBAC).
- [x] **App Identities:** Learned that App Registrations are global blueprints, and Service Principals are local identities for apps.
- [x] **Managed Identities:** Mastered the critical exam difference: System-assigned (tied to one resource) vs User-assigned (shared across multiple, independent lifecycle).
- [x] **MSAL:** Discovered the Microsoft Authentication Library for token acquisition.
- [x] **Passwordless AI:** Learned how to use `DefaultAzureCredential` to connect to Azure OpenAI without API keys.
- [x] **Custom API Security:** Understood how to use App Registrations to secure your own APIs from unauthorized callers.

🔗 **MODULE CONNECTIONS**
This module connects directly to **Module 12: Azure Key Vault**. If you absolutely *must* use a password or API key (e.g., for a non-Azure third-party service), you will store it in Key Vault and use your Managed Identity to read it!
