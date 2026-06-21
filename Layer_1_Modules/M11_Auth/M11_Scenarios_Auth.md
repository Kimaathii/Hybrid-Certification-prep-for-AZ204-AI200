# Module 11: Scenario Cards - Identity & Authorization

## SCENARIO 1: The One-to-One Connection (Beginner)
**Business Context:** Globomantics is a retail company moving their legacy inventory system to the cloud.
**The Problem:** An existing background worker application has been migrated to a single Azure Virtual Machine (VM). This worker needs to read images from an Azure Storage Account.
**Constraints:**
1. Passwords or storage account keys must NOT be used.
2. The identity must be automatically cleaned up if the VM is ever decommissioned.
**Your Task:** Choose the correct identity type to assign to the Virtual Machine.
**Hints:** 
- The identity is only needed by *one* resource.
- Look at the cleanup requirement constraint.
**Solution:** Enable a **System-Assigned Managed Identity** on the Virtual Machine. Then, use Azure RBAC to assign the "Storage Blob Data Reader" role to that identity on the Storage Account. System-Assigned identities are tied 1:1 to the resource and are automatically deleted when the parent resource is deleted.
**Exam Connection:** The AZ-204 exam frequently asks for the simplest identity solution that handles its own lifecycle cleanup.

---

## SCENARIO 2: The Multi-Node AI Cluster (Intermediate)
**Business Context:** Contoso AI processes millions of customer support transcripts using a custom natural language processing pipeline.
**The Problem:** Contoso has 15 Azure Web Apps running simultaneously to process the load. All 15 Web Apps need to authenticate to a single Azure OpenAI resource securely.
**Constraints:**
1. No API keys allowed in the codebase.
2. You must minimize administrative overhead (do not manage 15 separate identity assignments).
3. If one Web App node crashes and is recreated by the auto-scaler, it must instantly have access without manual intervention.
**Your Task:** Design the identity architecture for this AI cluster.
**Hints:** 
- System-Assigned identities cannot be shared. 
- You need a central master key that you can stamp onto new servers.
**Solution:** Create a **User-Assigned Managed Identity**. Assign the "Cognitive Services User" RBAC role to this single identity. Then, attach this User-Assigned identity to all 15 Web Apps (and configure the auto-scaler template to attach it to any newly spun-up nodes). 
**Exam Connection:** When an exam question specifies a cluster, a VM scale set, or resources needing *shared* access that persists through resource deletion, the answer is always a User-Assigned Managed Identity.

---

## SCENARIO 3: Choosing the Right MSAL Client (Intermediate)
**Business Context:** Fabrikam is building two distinct applications: a secure backend Web API running on Azure, and a public-facing mobile app available in the iOS App Store.
**The Problem:** Both applications need to authenticate users against Microsoft Entra ID. You are setting up the Microsoft Authentication Library (MSAL) in the code for both projects.
**Constraints:**
1. The backend API has a secure vault to store a client secret.
2. The mobile app can be decompiled by hackers, meaning no secrets can be hidden in its code.
**Your Task:** Determine which MSAL client type (`PublicClientApplication` vs `ConfidentialClientApplication`) should be used for each app.
**Hints:** 
- Can the device keep a secret? 
- "Confidential" means it can hide things securely.
**Solution:** 
- The backend Web API should use `ConfidentialClientApplication` because it runs on a secure server where client secrets can be safely stored.
- The mobile app must use `PublicClientApplication` because mobile devices are inherently untrusted environments where secrets can be extracted by malicious users.
**Exam Connection:** AZ-204 tests your knowledge of MSAL classes. If the scenario involves a Mobile App, Desktop App (WPF/Windows Forms), or Single-Page Application (SPA) running in a browser, select Public Client.

---

## SCENARIO 4: Securing the Internal AI Wrapper API (Advanced)
**Business Context:** Northwind Traders developed a proprietary "Pricing AI" model and exposed it as an Azure Function (`https://pricing-ai.northwind.com`). 
**The Problem:** Right now, the Azure Function accepts requests from anyone. Northwind only wants their internal "Sales Dashboard Web App" to be able to call this Pricing AI Function.
**Constraints:**
1. Only Entra ID authenticated apps can call the API.
2. You must explicitly define a permission (like `AI.Calculate`) that the caller must possess.
**Your Task:** Explain the steps to secure the Azure Function so only authorized applications can call it.
**Hints:** 
- Both the calling app and the receiving API need representation in Entra ID.
- You need to create a custom role.
**Solution:** 
1. Create an **App Registration** for the Pricing AI Function and define a custom App Role named `AI.Calculate` in its manifest.
2. Require Entra ID authentication on the Azure Function.
3. Create an **App Registration** for the Sales Dashboard Web App.
4. In Entra ID, grant the Sales Dashboard's App Registration permission to use the `AI.Calculate` role on the Pricing AI API. 
5. The Sales App must acquire a token using MSAL and pass it as a Bearer token in the HTTP Authorization header to the Function.
**Exam Connection:** You will see questions about "App Roles" and "Expose an API". Know that the API defines the roles, and the calling client requests permission to use those roles.

---

## SCENARIO 5: The Third-Party Integration (Advanced)
**Business Context:** AdventureWorks is integrating their internal HR software with a third-party SaaS provider (Workday).
**The Problem:** The HR Web App needs to read employee data from Workday's external API. Workday does NOT support Microsoft Entra ID or Managed Identities. They gave AdventureWorks a long, static API Key.
**Constraints:**
1. The API key must never touch the application's source code.
2. The API key must never be visible in the Azure App Service configuration environment variables.
**Your Task:** Design a secure way for the HR Web App to use this third-party API key.
**Hints:** 
- If you can't use a Managed Identity to connect to the destination directly, what secure Azure service can the Managed Identity read from?
**Solution:** 
1. Store the Workday API Key inside an **Azure Key Vault** as a Secret.
2. Enable a System-Assigned Managed Identity on the HR Web App.
3. Grant the Web App's identity RBAC permission (`Key Vault Secrets User`) to read from the Key Vault.
4. The application code uses `DefaultAzureCredential` to authenticate to the Key Vault, retrieves the API key at runtime, and uses it to call Workday.
**Exam Connection:** A classic AZ-204 trap. If a service does not support Managed Identity natively, you combine Managed Identity + Key Vault to achieve the most secure architecture possible.
