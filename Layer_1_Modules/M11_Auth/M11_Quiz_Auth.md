# Module 11: Identity & Authorization Quiz

## Foundation Questions

**Question 1**
You are explaining cloud security to a junior developer. They ask what service acts as the Identity Provider (IdP) in Microsoft Azure. Which service should you tell them?
A) Azure Key Vault
B) Microsoft Entra ID
C) Azure Role-Based Access Control (RBAC)
D) Azure Application Gateway

**Correct Answer: B**
- **A is incorrect:** Key Vault stores secrets; it does not issue identities.
- **B is correct:** Microsoft Entra ID (formerly Azure AD) is Microsoft's centralized Identity Provider.
- **C is incorrect:** Azure RBAC handles Authorization, not Authentication (Identity).
- **D is incorrect:** App Gateway is a network load balancer.

---

**Question 2**
What is the primary difference between Authentication and Authorization?
A) Authentication determines what you can do; Authorization determines who you are.
B) Authentication and Authorization are the same thing in Azure.
C) Authentication determines who you are; Authorization determines what you can do.
D) Authentication uses RBAC; Authorization uses Entra ID.

**Correct Answer: C**
- **A is incorrect:** This has the definitions backwards.
- **B is incorrect:** They are distinct security concepts handled by different systems.
- **C is correct:** AuthN verifies identity (who are you). AuthZ verifies permissions (what can you do).
- **D is incorrect:** It is the reverse. Authentication uses Entra ID; Authorization uses RBAC.

---

**Question 3**
Which term describes the global blueprint or definition of an application in Entra ID?
A) Service Principal
B) Managed Identity
C) App Service Plan
D) App Registration

**Correct Answer: D**
- **A is incorrect:** A Service Principal is the local instance/badge.
- **B is incorrect:** A Managed Identity is a specific type of automated Service Principal.
- **C is incorrect:** App Service Plan relates to compute hosting, not identity.
- **D is correct:** The App Registration is the global blueprint for an application in Entra ID.

---

**Question 4**
You need to authenticate a background worker process running on a single Azure VM to access an Azure SQL Database. The identity must be automatically deleted if the VM is deleted. Which identity type should you choose?
A) User-Assigned Managed Identity
B) Shared Access Signature (SAS)
C) System-Assigned Managed Identity
D) Service Principal with a Client Secret

**Correct Answer: C**
- **A is incorrect:** User-Assigned identities persist after the resource is deleted.
- **B is incorrect:** SAS is a token for Storage, not an Entra identity for SQL.
- **C is correct:** System-Assigned identities are tied 1:1 to the resource lifecycle and are deleted automatically.
- **D is incorrect:** Managing a client secret manually is insecure and requires manual cleanup.

---

**Question 5**
What is the primary purpose of the Microsoft Authentication Library (MSAL)?
A) To encrypt data at rest.
B) To acquire security tokens from Entra ID.
C) To manage Azure infrastructure via code.
D) To scan application code for security vulnerabilities.

**Correct Answer: B**
- **A is incorrect:** Data encryption is handled by Azure Storage/Key Vault.
- **B is correct:** MSAL is the SDK used by developers to negotiate and acquire authentication tokens from Entra ID.
- **C is incorrect:** This describes Azure Resource Manager (ARM) templates or Bicep.
- **D is incorrect:** This describes Microsoft Defender for Cloud.

---

## Applied Questions

**Question 6**
You are developing a C# application that needs to securely connect to Azure OpenAI. You want the code to use your personal Visual Studio credentials when running locally, but automatically switch to a Managed Identity when deployed to Azure App Service. Which Azure SDK class should you use?
A) `EnvironmentCredential`
B) `ManagedIdentityCredential`
C) `SharedTokenCacheCredential`
D) `DefaultAzureCredential`

**Correct Answer: D**
- **A is incorrect:** Requires environment variables to be explicitly set.
- **B is incorrect:** Only works when deployed to Azure; will fail locally.
- **C is incorrect:** Only checks local developer caches.
- **D is correct:** `DefaultAzureCredential` automatically chains multiple credential types, checking the local environment first, then falling back to Managed Identity in Azure.

---

**Question 7**
You are building an iOS mobile application that requires users to sign in. You are integrating MSAL into the mobile app's codebase. Which MSAL application type must you configure?
A) ConfidentialClientApplication
B) PublicClientApplication
C) DaemonClientApplication
D) SystemClientApplication

**Correct Answer: B**
- **A is incorrect:** Mobile apps cannot securely store a client secret, so they cannot be confidential clients.
- **B is correct:** Devices you do not control (phones, browsers) are Public Clients in MSAL.
- **C is incorrect:** Daemon apps run on secure servers.
- **D is incorrect:** Not a valid MSAL class.

---

**Question 8**
You have an Azure Web App. A user successfully signs in to the Web App using Entra ID. However, when the Web App tries to read a file from Azure Storage on behalf of the user, it receives an HTTP 403 Forbidden error. What is the most likely cause?
A) The user's Entra ID password has expired.
B) The Web App has a System-Assigned identity instead of User-Assigned.
C) The user has not been granted an Azure RBAC role on the Storage account.
D) The Storage account is in a different Azure region than the Web App.

**Correct Answer: C**
- **A is incorrect:** If the password expired, they could not sign in to the Web App (Authentication would fail).
- **B is incorrect:** The identity type does not cause 403 errors inherently.
- **C is correct:** The user successfully authenticated, but lacks Authorization (RBAC permissions) to read the storage data.
- **D is incorrect:** Cross-region access is allowed by default.

---

**Question 9**
You are creating a custom Web API to wrap your proprietary machine learning model. You want to restrict access so that only one specific internal web application can call it. You create an App Registration for the custom API. What is your next step?
A) Expose an API and define an App Role in the custom API's App Registration.
B) Create a System-Assigned Managed Identity on the custom API.
C) Upload a Client Secret to the custom API's App Registration.
D) Change the custom API to a Public Client.

**Correct Answer: A**
- **A is correct:** To protect a custom API, you must define permissions (App Roles/Scopes) on its App Registration so other apps can request them.
- **B is incorrect:** An identity on the API lets the API call *other* things; it doesn't protect the API itself from incoming callers.
- **C is incorrect:** The caller needs a secret to get a token, not the API receiving the request.
- **D is incorrect:** APIs running on servers are Confidential clients, not Public.

---

**Question 10 (Select all that apply)**
Your architecture includes an Azure Virtual Machine Scale Set containing 10 VMs. All 10 VMs run an AI processing daemon that must authenticate to an Azure Cognitive Service. Which of the following statements about using a User-Assigned Managed Identity are TRUE?
A) You only need to create one identity and assign it to the entire Scale Set.
B) If you delete the Scale Set, the identity is automatically permanently deleted.
C) You can use the same identity for a separate Azure Web App in a different resource group.
D) The identity cannot be assigned Azure RBAC roles.

**Correct Answers: A and C**
- **A is correct:** User-assigned identities can be attached to a Scale Set, and all instances will inherit it.
- **B is incorrect:** User-assigned identities have independent lifecycles and survive resource deletion.
- **C is correct:** Because it is independent, it can be shared across multiple completely different Azure resources.
- **D is incorrect:** You MUST assign RBAC roles to it, otherwise it has no permissions.

---

## Exam-Level Questions

**Question 11**
You are deploying a legacy application to an Azure VM. The application must communicate with an external third-party API that requires a static API key. The third-party API does not support Microsoft Entra ID. You must ensure the API key is not stored on the VM's disk or in its environment variables. How should you architect this solution?
A) Store the API key in Entra ID as a custom user attribute.
B) Store the API key in Azure Key Vault. Enable a System-Assigned identity on the VM to retrieve it at runtime.
C) Store the API key in an Azure Storage Blob. Grant the VM public access to the blob.
D) Hardcode the API key in the application binary using obfuscation.

**Correct Answer: B**
- **A is incorrect:** Entra ID is for identities, not storing third-party secrets.
- **B is correct:** Key Vault is the secure store. The VM uses its managed identity to authenticate to the Key Vault to read the secret in memory.
- **C is incorrect:** Public blob access is highly insecure.
- **D is incorrect:** Hardcoding, even obfuscated, is insecure and violates constraints.

---

**Question 12**
An application running on an Azure Web App needs to access an Azure SQL database using a System-Assigned Managed Identity. The identity has been enabled on the Web App. What must you do inside the Azure SQL Database to complete the authorization setup?
A) Create a contained database user mapped to the Web App's Managed Identity name.
B) Enable Transparent Data Encryption (TDE) on the database.
C) Add the Web App's IP address to the SQL server firewall.
D) Generate a SQL connection string containing a master username and password.

**Correct Answer: A**
- **A is correct:** In Azure SQL, you must run a SQL command (`CREATE USER [AppName] FROM EXTERNAL PROVIDER`) to map the Entra ID identity to a database user and grant it SQL roles (like `db_datareader`).
- **B is incorrect:** TDE encrypts data at rest; it does not handle authorization.
- **C is incorrect:** While a firewall rule might be needed for network access, it does not handle Entra ID authorization.
- **D is incorrect:** The goal of Managed Identity is to eliminate passwords.

---

**Question 13**
You are configuring an App Registration for a daemon application that runs in the background with no user interaction. The application needs to read user profiles from Microsoft Graph. What type of API permissions should you grant the App Registration?
A) Delegated Permissions
B) Application Permissions
C) Public Permissions
D) System Permissions

**Correct Answer: B**
- **A is incorrect:** Delegated permissions are used when an app acts *on behalf of a signed-in user*.
- **B is correct:** Application permissions are used when an app runs as a background service or daemon without a signed-in user present.
- **C is incorrect:** Not a valid Entra ID permission type.
- **D is incorrect:** Not a valid Entra ID permission type.

---

**Question 14**
A developer is trying to use `DefaultAzureCredential` to connect to Azure Storage locally on their workstation. They are receiving a "TokenCredentialAuthenticationFailedException". What is the fastest way to resolve this issue for local development?
A) Assign a System-Assigned Managed Identity to their local laptop.
B) Run `az login` in the terminal to authenticate the Azure CLI.
C) Create an App Registration and paste the client secret into their source code.
D) Disable Azure RBAC on the storage account.

**Correct Answer: B**
- **A is incorrect:** You cannot assign Azure Managed Identities to local developer laptops.
- **B is correct:** `DefaultAzureCredential` looks for developer credentials in the local environment, including the Azure CLI. Running `az login` caches a token it can use.
- **C is incorrect:** Pasting secrets in code is insecure and defeats the purpose of `DefaultAzureCredential`.
- **D is incorrect:** Disabling RBAC would remove all security and is not a valid solution.

---

**Question 15 (Select all that apply)**
Which of the following scenarios are valid use cases for a User-Assigned Managed Identity rather than a System-Assigned Managed Identity?
A) You have a fleet of 50 Virtual Machines that all need identical access to a Key Vault.
B) You have a single Azure Function that needs access to a Storage Account, and the identity should be deleted when the Function is deleted.
C) You need to provision the identity via Infrastructure-as-Code (Terraform/Bicep) before you create the Web Apps that will use it.
D) You are building a mobile application that needs an identity to call an API.

**Correct Answers: A and C**
- **A is correct:** Shared access across multiple resources requires a User-Assigned identity.
- **B is incorrect:** 1:1 mapping with automatic deletion is the exact definition of System-Assigned.
- **C is correct:** Because User-Assigned identities are standalone resources, they can be created first in an IaC script and then referenced by later resources. System-Assigned identities only exist *after* the parent is created.
- **D is incorrect:** Mobile apps cannot use Azure Managed Identities; they use App Registrations (Public Clients) to authenticate human users.
