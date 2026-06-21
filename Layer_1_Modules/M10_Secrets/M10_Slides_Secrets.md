# Module 10: Secrets & Configuration (Hybrid Web + AI Edition) - Slide Deck

---

## Slide 1: Title Slide
**Title:** Secrets & Configuration (Hybrid Web + AI Edition)
**Exam Domain:** AZ-204 (App Configuration) + AI-200 (AI Infrastructure)
**Weight:** 10-15%
**Colours:** Azure Blue (#0078D4) background, white text.

**Learning Objectives:**
- Understand why hardcoding secrets is a massive security risk.
- Differentiate between Azure Key Vault Secrets, Keys, and Certificates.
- Compare Vault Access Policies vs Azure RBAC.
- Use Azure App Configuration for Feature Flags.
- Secure Azure OpenAI API keys using Key Vault and App Service References.

**Speaker Notes:**
> "Welcome to Module 10. Today we are tackling one of the most critical aspects of modern cloud development: managing secrets and configuration. We'll be covering both the standard AZ-204 requirements and the AI-200 additions for securing expensive AI API keys."

---

## Slide 2: The Problem This Service Solves
**Title:** The Danger of Hardcoding
**Visual:** A diagram showing a developer saving a file containing `password="mySuperSecret123"` into GitHub. A hacker then downloads the GitHub code and accesses the database.

**Speaker Notes:**
> "Before we talk about Azure services, we have to talk about the problem. When you write code, your app needs to connect to databases, storage, and AI services. To do this, it needs passwords and API keys. The biggest beginner mistake is typing these passwords directly into your code. If you commit this code to a source control system like GitHub, anyone who can see the code can steal your passwords."

---

## Slide 3: Analogy — Keys Under the Doormat
**Title:** Keys Under the Doormat
**Visual:** A beautiful, highly secure house with a glaringly obvious key peeking out from under the front doormat.

**Speaker Notes:**
> "Think of it like building a fortress, but leaving the front door key under the doormat. It doesn't matter how secure your cloud environment is; if your code contains the passwords, your security is entirely compromised. This is what we call 'hardcoding' secrets."

---

## Slide 4: Azure Key Vault Overview
**Title:** Introducing Azure Key Vault
**Visual:** The Azure Key Vault icon in the center. An application authenticates with Entra ID, gets a token, and uses it to retrieve a secret from the vault.

**Speaker Notes:**
> "The solution to the doormat problem is Azure Key Vault. This is a centralized cloud service for safeguarding cryptographic keys and other secrets used by cloud apps. Instead of putting secrets in your code, your code authenticates securely to Azure and asks Key Vault for the secret exactly when it needs it."

---

## Slide 5: Analogy — The Bank Safety Deposit Box
**Title:** The Bank Safety Deposit Box
**Visual:** A bank teller verifying an ID before handing over a safety deposit box.

**Speaker Notes:**
> "If code is the doormat, Key Vault is the bank. To get the key, your app has to prove its identity to the 'teller'—which is Azure Active Directory, now called Microsoft Entra ID. Only authorized applications can retrieve the secrets."

---

## Slide 6: Secrets vs. Keys vs. Certificates
**Title:** Three Types of Key Vault Items
**Visual:** Three columns with icons: a lock with a string of text (Secrets), a cryptographic key (Keys), and an ID badge (Certificates).

**Speaker Notes:**
> "Key Vault stores three different things. A 'Secret' is just a string of text, like a password or an API key. A 'Key' is a cryptographic key used for encrypting data. A 'Certificate' is a digital certificate used to secure websites with HTTPS."

---

## Slide 7: ⚠️ EXAM ALERT: Secrets vs Keys 
**Background:** Red (#D13438)
**Text:** White

**Exam Trap:**
- Need to store an Azure OpenAI API key? -> **Secret**
- Need to store a database connection string? -> **Secret**
- Need to encrypt a database disk? -> **Key**

**Speaker Notes:**
> "Pay close attention here. The exam loves to test if you know the difference between a Key and a Secret. Even though we say 'API Key' in English, in Azure Key Vault, it is stored as a 'Secret'. Only choose 'Key' on the exam if the question explicitly talks about data encryption."

---

## Slide 8: Controlling Access to Key Vault
**Title:** Who Can Read the Secrets?
**Visual:** A scale balancing "Granular Control" vs "Centralized Management".

**Speaker Notes:**
> "Once your secrets are safely inside Key Vault, you have to decide who is allowed to read them. You don't want every app in your company to have access to the HR database password. Azure gives you two different authorization systems to control this."

---

## Slide 9: Access Policies vs Azure RBAC
**Title:** Vault Access Policies vs Azure RBAC
**Visual:** 
- Access Policies: A bouncer standing at the door of the vault with a specific list.
- Azure RBAC: A central corporate HQ distributing "Role" badges to employees.

**Speaker Notes:**
> "The first method is Vault Access Policies. This is the legacy method. It's like a bouncer at the specific vault door. The second is Azure RBAC, or Role-Based Access Control. This is like a corporate-wide VIP system where you assign a role, like 'Key Vault Secrets User', to an identity."

---

## Slide 10: ⚠️ EXAM ALERT: RBAC is Recommended
**Background:** Red (#D13438)
**Text:** White

**Exam Fact:**
- Azure RBAC is the **Microsoft Recommended** approach.
- The role to read a secret is **Key Vault Secrets User**.

**Speaker Notes:**
> "For the exam, if you are asked to choose the best way to secure a new Key Vault, always choose Azure RBAC. It is the modern, recommended approach. Also, memorize the specific role name: 'Key Vault Secrets User'. This role allows an app to read secrets, but not change or delete them."

---

## Slide 11: The Problem — Redeploying for Configuration
**Title:** The Danger of Static Configuration
**Visual:** A developer changing a background color in code, waiting 30 minutes for a build pipeline, just to see the change live.

**Speaker Notes:**
> "We've solved secrets, but what about non-sensitive settings? Let's say you want to change the color of a button, or turn off a broken feature. If that setting is hardcoded, you have to rewrite the code, recompile the app, and redeploy it. This can cause downtime and takes too long."

---

## Slide 12: Azure App Configuration Overview
**Title:** Azure App Configuration
**Visual:** Azure App Configuration icon sending settings to a web app, a mobile app, and a serverless function simultaneously.

**Speaker Notes:**
> "Enter Azure App Configuration. This service centrally manages application settings and feature flags. It allows you to change your application's behavior in real-time, instantly, without restarting or redeploying the code."

---

## Slide 13: Feature Flags (Feature Management)
**Title:** Feature Flags
**Visual:** A toggle switch labeled "New AI Chat Interface - ON/OFF".

**Speaker Notes:**
> "A massive benefit of App Configuration is Feature Flags. A feature flag is a simple true or false toggle. If true, the code runs the new feature. If false, it hides it. This is perfect for A/B testing or slowly rolling out a new feature to only a small percentage of your users."

---

## Slide 14: Analogy — The Restaurant Menu
**Title:** The Restaurant Menu
**Visual:** A digital fast-food menu board updating its prices instantly without the store closing.

**Speaker Notes:**
> "Think of App Configuration like a digital menu board. If you want to change a price, you don't rebuild the restaurant. You just update the central system, and the menu board instantly updates. Feature flags are like a secret menu item you can instantly turn on for VIPs."

---

## Slide 15: AI-200: Securing AI Keys
**Title:** Securing AI Service Keys
**Visual:** An Azure OpenAI Service icon linked to a Key Vault.

**Speaker Notes:**
> "Now let's switch gears to AI-200 topics. AI services like Azure OpenAI are incredibly powerful, but they cost money. To use them, you get an API key. If this key gets into the wild, it can be abused."

---

## Slide 16: The Architecture of AI API Keys
**Title:** The Secure Pattern
**Visual:** 
1. App gets OpenAI Endpoint from App Configuration.
2. App gets OpenAI API Key from Key Vault.
3. App calls OpenAI securely.

**Speaker Notes:**
> "The best practice is a hybrid approach. Store your AI endpoints—the URLs, which aren't sensitive—in App Configuration. Store the actual API Key in Key Vault. Your application fetches both at runtime."

---

## Slide 17: ⚠️ EXAM ALERT: Leaking AI Keys
**Background:** Red (#D13438)
**Text:** White

**Exam Trap:**
- Never commit `appsettings.json` with AI keys to source control.
- Use **Managed Identities** to allow your app to read from Key Vault.

**Speaker Notes:**
> "Exam questions will test your workflow. If you put an AI key in an appsettings.json file on your local machine, and commit it to GitHub, you've leaked the key. Always use Managed Identities so your app can fetch the key from the vault without needing a password to the vault."

---

## Slide 18: Key Vault References in App Service
**Title:** Key Vault References
**Visual:** An App Service configuration panel showing `@Microsoft.KeyVault(SecretUri=...)`

**Speaker Notes:**
> "Writing the code to fetch secrets can be tedious for beginners. Azure App Service has a brilliant feature called Key Vault References. It allows your web app to fetch secrets from Key Vault without writing any custom SDK code."

---

## Slide 19: Analogy — The Magic Sticky Note
**Title:** The Magic Sticky Note
**Visual:** A sticky note that says "Go look in the vault" attached to a configuration file.

**Speaker Notes:**
> "Think of it as a magic sticky note. Instead of pasting the password into your web app's settings, you paste a special Key Vault Reference string. The App Service sees this string, automatically goes to the vault, grabs the real secret, and gives it to your app. Your code never knows the difference!"

---

## Slide 20: Module Summary Table
**Title:** Module Summary
**Visual:** A summary table

| Service / Feature | Purpose | Key Exam Fact |
|-------------------|---------|---------------|
| **Key Vault** | Store sensitive secrets & keys. | Use Azure RBAC over Access Policies. |
| **App Configuration** | Store non-sensitive settings. | Use for Feature Flags and A/B testing. |
| **Secrets vs Keys** | Passwords vs Crypto Keys. | API Keys = Secrets! |
| **Key Vault References** | Read vault without code. | Requires a System-Assigned Managed Identity. |

**Speaker Notes:**
> "To wrap up, this table covers the exact points you must know for the exam. Memorize the difference between Key Vault for secrets and App Config for settings. Know that API keys are Secrets, and that Key Vault References require a Managed Identity to work. Thank you for joining this module."
