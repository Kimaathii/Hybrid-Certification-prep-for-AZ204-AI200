# Module 10: Secrets & Configuration (Hybrid Web + AI Edition)

## Title Page
**Module Name:** Module 10 — Secrets & Configuration (Hybrid Web + AI Edition)  
**Exam Domain:** AZ-204 (App Configuration) + AI-200 (AI Infrastructure)  
**Weight:** 10-15%  
**Estimated Study Time:** 2.5 hours  
**Prerequisites:** Basic understanding of Azure App Service, GitHub or basic source control concepts.

---

## 1. The Problem: Hardcoding Passwords in Source Control

### The Analogy: Keys Under the Doormat
Imagine you own a highly secure smart home. You've installed state-of-the-art alarms, reinforced doors, and camera systems. However, to let the dog walker in, you leave the master key under the front doormat. It doesn’t matter how good your security system is; anyone who knows where the key is can walk right in. 

### The Technical Reality
In software development, your application often needs "keys" to function. These are **secrets**: database passwords, API keys (like your Azure OpenAI key), or connection strings. 
A common, but dangerous, beginner mistake is **hardcoding** these secrets directly into the source code (the text files that make up your program) and saving them into **Source Control** (a system like GitHub that tracks code changes). 

If you put a secret in your code, it becomes like the key under the doormat. Anyone with access to your code (or if your code is accidentally made public) now has access to your databases and paid AI services. 

**The Solution?** A secure vault where you keep the keys, and the application only asks for them exactly when needed, without ever saving them in the code.

---

## 2. Azure Key Vault: Centralized Secret Management

### The Analogy: The Bank Safety Deposit Box
Instead of leaving your key under the doormat, you place it in a heavily guarded bank safety deposit box. When you need the key, you must show the bank teller your verified ID. Only then does the teller fetch the exact key you need.

### The Technical Definition
**Azure Key Vault** is a centralized cloud service for safeguarding cryptographic keys and other secrets used by cloud applications and services. 

Instead of storing secrets in your code or app settings, you store them in Key Vault. Your application then securely authenticates (proves its identity) to Azure and retrieves the secrets at runtime (when the application is running).

💡 **KEY CONCEPT**
> Azure Key Vault completely removes secrets from your source code and centralizes them, making it easier to manage, rotate, and secure them.

### Secrets vs. Keys vs. Certificates
Key Vault stores three different types of sensitive information. You must know the difference for the exam:

| Item Type | What is it? | Real-world Example | Use Case |
|-----------|-------------|--------------------|----------|
| **Secrets** | A generic string of text that Key Vault securely stores and retrieves. | Passwords, Database Connection Strings, API Keys (e.g., Azure OpenAI key). | Storing application passwords so they aren't in code. |
| **Keys** | Cryptographic keys used to encrypt or decrypt data. | RSA or Elliptic Curve keys. | Encrypting a database disk or securing a storage account. |
| **Certificates** | Digital certificates (like an ID card for a website) used to prove identity and secure web traffic. | SSL/TLS certificates (the padlock in your browser). | Securing a website with HTTPS. |

🚨 **EXAM ALERT**
> If an exam question asks where to store an *API key* or a *database password*, the answer is a **Secret**. If it asks where to store something for *data encryption*, the answer is a **Key**.

---

## 3. Access Policies vs RBAC in Key Vault

Once your secrets are in the Key Vault, you need to control *who* or *what* can read them. There are two ways to do this in Azure.

### The Analogy: The Bouncer vs. The VIP List
- **Access Policies** are like giving a bouncer a specific rule: "Allow John to read secrets, but he cannot delete them." It's managed directly at the door of the club (the Key Vault itself).
- **RBAC (Role-Based Access Control)** is like a massive, corporate-wide VIP list managed by headquarters. Headquarters says "All managers can read secrets in any club."

### The Technical Definition
Azure Key Vault supports two authorization systems:
1. **Vault Access Policies:** The older, legacy method. You configure permissions directly on the Key Vault resource. You grant specific permissions (e.g., Get, List, Set, Delete) to a specific user, app, or managed identity.
2. **Azure RBAC (Role-Based Access Control):** The modern, recommended method. It integrates Key Vault permissions with the rest of Azure's permission system. You assign roles like "Key Vault Secrets User" to an identity.

| Feature | Vault Access Policies | Azure RBAC |
|---------|-----------------------|------------|
| **Scope** | Applies only to the specific Key Vault. | Can be applied to the Vault, the Resource Group, or the whole Subscription. |
| **Granularity** | Can specify exact permissions (Get, List, etc.). | Uses predefined roles (e.g., Key Vault Secrets Officer). |
| **Recommendation** | Legacy, but still widely used and tested. | **Recommended** best practice by Microsoft. |

🚨 **EXAM ALERT**
> The exam will test if you know that **Azure RBAC** is the modern, recommended approach over Access Policies for managing access to Key Vault data. Also, know the role **Key Vault Secrets User** is exactly what an application needs to *read* a secret.

✅ **CHECKPOINT**
> Can you name the three types of items stored in a Key Vault? Which one would hold a database password? *(Answer: Secrets, Keys, Certificates. A password is a Secret.)*

---

## 4. Azure App Configuration: Dynamic Configuration Without Redeploying

### The Analogy: The Restaurant Menu
Imagine a digital menu board at a fast-food restaurant. If they want to change the price of a burger, they don't tear down the whole building and rebuild it (redeploying an app). They just update a central control panel, and the screen instantly shows the new price (dynamic configuration). If they want to test a new "Spicy Burger", they can hide it behind a toggle switch and only show it to 10% of customers (feature flags).

### The Technical Definition
**Azure App Configuration** is a service that centrally manages application settings and feature flags. 

Unlike Key Vault (which is strictly for highly sensitive *secrets*), App Configuration is for *non-sensitive* configuration data like background colors, retry limits, or feature toggles. It allows you to change your application's behavior in real-time without restarting or redeploying the code.

### Feature Flags (Feature Management)
A **Feature Flag** is a boolean (true/false) toggle that allows you to turn a feature in your application on or off remotely.

**Why use Feature Flags?**
- **A/B Testing:** Show a new AI chat interface to only 20% of your users to see if they like it.
- **Kill Switch:** If a new feature is causing your app to crash, you can instantly turn it off without writing any new code.
- **Gradual Rollout:** Release a feature to a small group first, then expand it.

💡 **KEY CONCEPT**
> Use **Key Vault** for sensitive data (passwords, keys). Use **App Configuration** for non-sensitive settings and feature flags. App Configuration can even integrate with Key Vault to provide a single place for your app to fetch *both* settings and secrets.

---

## 5. [AI-200 ADDITION] Securing AI Keys

### The Problem with AI Keys
When you provision an Azure OpenAI service or any Cognitive Service, you are given an Endpoint (a URL) and an API Key (a long string of characters). Every time you ask the AI to generate text or an image, you must send this API Key to prove you are authorized and to bill your account.

If this AI key is leaked to GitHub, malicious users can steal it and run up tens of thousands of dollars in AI compute charges on your bill.

### The Solution: Store AI Keys in Key Vault
As an Azure Developer and AI Engineer, your absolute priority is securing these keys. 

1. **Create the Key Vault.**
2. **Store the Azure OpenAI Key** as a **Secret** inside the Key Vault.
3. **Store the Azure OpenAI Endpoint** as a setting in **Azure App Configuration**.
4. Give your application a **Managed Identity** (an automatically managed Azure identity) and assign it the "Key Vault Secrets User" role.

Your code never sees the actual text of the API key. It just asks Azure, "I am the authorized app, please talk to OpenAI for me using the secret in the vault."

💰 **COST WARNING**
> Leaked Azure OpenAI keys are one of the fastest ways to incur massive unexpected cloud bills. Never put them in local configuration files like `appsettings.json` that might get committed to Git.

---

## 6. [AI-200 ADDITION] Key Vault References in App Service

### The Analogy: The Magic Sticky Note
Imagine you have an instruction manual (your code) that says "Type the password here." Instead of writing the password on the manual, you place a magic sticky note that says "Go look in the bank vault." When the machine reads the manual, it automatically goes to the vault, gets the password, and uses it, without you having to write any special "go to the bank" instructions.

### The Technical Definition
**Key Vault References** allow Azure App Service (where you host your web apps) to use secrets from Key Vault directly in its configuration, **without requiring you to change your application code.**

Normally, to read from Key Vault, you have to write SDK code. But with Key Vault References, you just put a special string in your App Service configuration settings.

**The Syntax:**
```kql
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)
```

When App Service sees this string, it automatically reaches out to Key Vault, grabs the secret, and injects it into your app as a normal environment variable. Your app just reads the environment variable normally, completely unaware that it came from Key Vault.

🚨 **EXAM ALERT**
> To use Key Vault References, your App Service **MUST** have a System-Assigned Managed Identity enabled, and that identity **MUST** have permission to read the secret in Key Vault. If either is missing, the reference will fail.

---

## 🔗 MODULE CONNECTIONS
- **Module 3 (App Service):** Key Vault references rely heavily on the Managed Identity concepts we covered when hosting web apps.
- **Module 12 (Security & Identity):** We will dive deeper into Azure Active Directory, Managed Identities, and RBAC roles that make Key Vault authentication work.

---

## What We Covered
- [x] Why hardcoding secrets is dangerous (The "Keys Under the Doormat" analogy).
- [x] Azure Key Vault as the centralized solution for Secrets, Keys, and Certificates.
- [x] The difference between Vault Access Policies (legacy) and Azure RBAC (modern).
- [x] Azure App Configuration for non-sensitive settings and Feature Flags.
- [x] How to protect expensive Azure OpenAI keys by storing them in Key Vault.
- [x] Using Key Vault References in App Service to fetch secrets without writing SDK code.
