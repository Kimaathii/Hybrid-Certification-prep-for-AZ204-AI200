# Module 10: Secrets & Configuration — Scenario Cards

## Scenario 1: The GitHub Leak (Beginner)
**Business Context:** "FreshStart AI" is a new startup building a resume-writing application powered by Azure OpenAI.
**The Problem:** A junior developer accidentally committed the `appsettings.json` file containing the Azure OpenAI API key to a public GitHub repository. Within 10 minutes, malicious bots found the key and ran up $5,000 in AI compute charges.
**Constraints:** 
- The API key must never be stored in plain text in the source code again.
- The application code needs a way to securely retrieve a new API key when running locally on developer machines and in the cloud.
**Your Task:** Design a secure storage mechanism for the AI key and determine the best Azure SDK class to use for authentication.
**Hints:** 
- Where do we store "secrets" in Azure?
- What Azure SDK credential class adapts to the local environment and the cloud automatically?
**Solution:** 
1. Regenerate the Azure OpenAI API key immediately in the Azure Portal to invalidate the leaked key.
2. Create an **Azure Key Vault** and store the new API key as a **Secret**.
3. In the application code, use the Azure SDK for Key Vault and authenticate using `DefaultAzureCredential`. This allows developers to read the vault using their local `az login` credentials, while the cloud app uses its Managed Identity.
**Exam Connection:** The exam will frequently present a scenario where code is pushed to GitHub with passwords. The answer is always to move the password to a Key Vault Secret and use Managed Identities / `DefaultAzureCredential`.

---

## Scenario 2: Connecting App Service to Key Vault Without Code (Intermediate)
**Business Context:** "LegacyTrade", a financial services company, has an old ASP.NET web app hosted on Azure App Service.
**The Problem:** The app connects to a SQL database using a password hardcoded in the App Service "Application Settings" menu in the Azure Portal. Security auditors have flagged this as a violation.
**Constraints:**
- The password must be moved to Azure Key Vault.
- The development team is currently busy; you **cannot** change the application's source code or deploy new code.
**Your Task:** Configure the Azure App Service to read the password from Key Vault without modifying the application code.
**Hints:**
- What feature allows App Service to "pretend" a secret is a normal application setting?
- What identity does the App Service need to talk to the Vault?
**Solution:**
1. Enable a **System-Assigned Managed Identity** on the Azure App Service.
2. Grant that Managed Identity the **Key Vault Secrets User** role (Azure RBAC) on the Key Vault.
3. In the App Service Configuration, replace the plaintext password with a **Key Vault Reference** using the syntax: `@Microsoft.KeyVault(SecretUri=https://<vault-name>.vault.azure.net/secrets/<secret-name>/)`. The App Service will seamlessly inject the secret at runtime.
**Exam Connection:** You will see questions asking how to inject secrets into an App Service without rewriting code. The answer is Key Vault References, and you must remember that it *requires* a Managed Identity.

---

## Scenario 3: A/B Testing the New AI Chatbot (Intermediate)
**Business Context:** "ShopBot" is an e-commerce platform transitioning from a traditional search bar to an AI-powered conversational chatbot.
**The Problem:** The marketing team is nervous that the AI chatbot might confuse older customers. They want to test the chatbot on only 15% of the live users to gather metrics before a full rollout.
**Constraints:**
- The application cannot be redeployed to change the percentage of users seeing the chatbot.
- The routing of 15% of traffic must be handled randomly but consistently per user session.
**Your Task:** Choose the appropriate Azure service to manage this rollout and identify the specific feature required.
**Hints:**
- Key Vault is for secrets. What service is for settings?
- How do you turn a feature on for only a subset of traffic?
**Solution:** Use **Azure App Configuration** and implement **Feature Flags** (Feature Management). You can create a feature flag named `EnableAIChat` and apply a **Percentage Filter** set to 15%. The application checks this flag at runtime; Azure App Configuration automatically handles the math to ensure 15% of users receive `true` and the rest receive `false`.
**Exam Connection:** Knowing when to use App Configuration over Key Vault is critical. "A/B Testing", "Percentage rollouts", and "changing behavior without redeploying" are keywords for App Configuration Feature Flags.

---

## Scenario 4: Rotating Database Passwords (Intermediate)
**Business Context:** "HealthSecure", a medical records company, requires strict compliance with healthcare regulations.
**The Problem:** Security policies mandate that the master password for their Azure SQL Database must be changed (rotated) every 30 days. Currently, when the password changes, the web application crashes until an administrator manually restarts the App Service with the new password.
**Constraints:**
- The password rotation must happen with zero downtime for the web application.
- The web app must automatically pick up the new password.
**Your Task:** Design a solution using Key Vault and App Configuration to handle seamless secret rotation.
**Hints:**
- Can App Configuration tell your app when something changes?
- Can App Configuration link to a Key Vault secret?
**Solution:** 
1. Store the SQL Password as a Secret in **Key Vault**.
2. Create a Key Vault Reference inside **Azure App Configuration** (not App Service directly).
3. The application connects to App Configuration and uses the `AddAzureAppConfiguration` SDK with dynamic refresh enabled. 
When the password is rotated in Key Vault, the application automatically pulls the new secret via App Configuration's dynamic refresh without needing a restart.
**Exam Connection:** The exam tests dynamic configuration refresh. Remember that App Configuration can wrap Key Vault references, allowing apps to dynamically reload secrets without restarting.

---

## Scenario 5: Cross-Region AI Redundancy (Advanced)
**Business Context:** "GlobalAI" provides real-time translation services to international airlines. They use Azure OpenAI hosted in both `East US` and `West Europe`.
**The Problem:** If the `East US` AI region goes down, the application needs to instantly failover and start sending requests to the `West Europe` AI endpoint. 
**Constraints:**
- The failover must not require a code deployment.
- The application uses multiple settings (Endpoint URL, Region Name, Timeout limits) that must all change together during a failover.
**Your Task:** Configure Azure App Configuration to manage these region-specific settings and switch between them.
**Hints:**
- How can you group related settings together in App Configuration?
**Solution:** Use **Labels** in Azure App Configuration. 
Create settings for `AI_Endpoint` and `AI_Timeout`. Create two versions of each: one with the label `EastUS` and one with the label `WestEurope`. 
In the application, configure it to load settings based on an active label. If a region fails, an administrator changes one master setting or environment variable to swap the active label, and the application instantly reloads the configuration for the backup region.
**Exam Connection:** Labels are an advanced App Configuration feature heavily tested on AZ-204. They are the correct answer for managing environment-specific (Dev/Test/Prod) or region-specific (East/West) settings under the same key names.
