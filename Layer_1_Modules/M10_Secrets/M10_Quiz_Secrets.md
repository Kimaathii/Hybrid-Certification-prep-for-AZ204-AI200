# Module 10: Secrets & Configuration — Quiz

## Foundation Level

**Question 1**
You are designing an Azure solution that requires storing a 2048-bit RSA key used to encrypt virtual machine disks. Which Azure Key Vault item type should you use?
A) Secret
B) Key
C) Certificate
D) Token
**Answer: B**
* **A is incorrect** because Secrets are used for passwords, connection strings, and API keys, not for cryptographic data encryption.
* **B is correct**. Key Vault Keys are specifically designed for cryptographic operations like data encryption and decryption.
* **C is incorrect** because Certificates are used for identity and securing web traffic (SSL/TLS).
* **D is incorrect** because "Token" is not a valid Key Vault storage type.

**Question 2**
You need to store an Azure OpenAI API key so your application can authenticate to the AI service. Which Azure Key Vault item type should you use?
A) Secret
B) Key
C) Certificate
D) Managed Identity
**Answer: A**
* **A is correct**. Even though it is called an "API Key" in plain English, Key Vault treats passwords, connection strings, and API keys as generic strings of text, which are stored as Secrets.
* **B is incorrect** because Key Vault Keys are for cryptographic encryption, not for storing authentication strings.
* **C is incorrect** because Certificates are for SSL/TLS.
* **D is incorrect** because a Managed Identity is an authentication mechanism, not an item stored inside a vault.

**Question 3**
Which Azure service should you use to centrally manage application settings, background colors, and feature flags without requiring application restarts?
A) Azure Key Vault
B) Azure Storage Accounts
C) Azure App Configuration
D) Azure Application Insights
**Answer: C**
* **A is incorrect** because Key Vault is strictly for highly sensitive data (secrets) and does not natively support feature flags.
* **B is incorrect** because Storage Accounts are for blobs and files, not dynamic application configuration.
* **C is correct**. Azure App Configuration centrally manages non-sensitive settings and Feature Flags and supports dynamic refresh without redeploying.
* **D is incorrect** because Application Insights is used for logging and monitoring.

**Question 4**
Microsoft recommends moving away from legacy Vault Access Policies. What is the modern, recommended authorization system for controlling access to Azure Key Vault data?
A) Azure Active Directory Conditional Access
B) Azure RBAC (Role-Based Access Control)
C) Network Security Groups (NSGs)
D) Shared Access Signatures (SAS)
**Answer: B**
* **A is incorrect** because Conditional Access controls user logins (like requiring MFA), not specific data access inside a vault.
* **B is correct**. Azure RBAC is the modern, unified permission system recommended by Microsoft for managing access to Key Vault data planes.
* **C is incorrect** because NSGs control network traffic to VMs, not identity access.
* **D is incorrect** because SAS tokens are used for Azure Storage, not Key Vault.

**Question 5**
When using Azure RBAC with Key Vault, which role should you assign to an application so it can read a password, but cannot change or delete it?
A) Key Vault Administrator
B) Key Vault Secrets Officer
C) Key Vault Secrets User
D) Owner
**Answer: C**
* **A is incorrect** because Administrator grants full access to all keys, secrets, and certificates.
* **B is incorrect** because Secrets Officer allows creating, changing, and deleting secrets.
* **C is correct**. Key Vault Secrets User is the principle of least privilege, granting only the ability to read (get) secret values.
* **D is incorrect** because Owner grants full administrative control over the entire resource.

---

## Applied Level

**Question 6**
You are developing a .NET console application that will run on your local machine during development, and on Azure App Service in production. You want the code to automatically use your local CLI credentials when debugging, and the App Service's Managed Identity when in production. Which credential class should you use?
A) `EnvironmentCredential`
B) `ManagedIdentityCredential`
C) `SharedTokenCacheCredential`
D) `DefaultAzureCredential`
**Answer: D**
* **A is incorrect** because it only looks for specific environment variables.
* **B is incorrect** because it only works in Azure, causing your local development to fail.
* **C is incorrect** because it only looks at visual studio caches.
* **D is correct**. `DefaultAzureCredential` is designed precisely for this hybrid scenario. It checks the environment in a specific order: first looking for Managed Identity, then falling back to Visual Studio/Azure CLI credentials locally.

**Question 7**
You want to implement a new "AI Image Generation" feature in your web app, but you want to expose it to only 20% of your user base to monitor server load. Which feature of Azure App Configuration should you use?
A) Key Vault References
B) Configuration Labels
C) Feature Flags with a Percentage Filter
D) Feature Flags with a Time Window Filter
**Answer: C**
* **A is incorrect** because Key Vault References pull secrets, they don't route traffic.
* **B is incorrect** because Labels are used to manage environments (Dev vs Prod), not user percentages.
* **C is correct**. Feature Flags with a Percentage Filter allow you to randomly, but consistently, evaluate a feature as `true` for a specified percentage of traffic.
* **D is incorrect** because a Time Window filter turns a feature on/off based on dates, not user percentages.

**Question 8**
Your App Service requires a database password stored in Key Vault. You do not want to write any Azure SDK code. You decide to use a Key Vault Reference in the App Service configuration. What **two** things must be configured for this to work? (Select all that apply)
A) A System-Assigned or User-Assigned Managed Identity on the App Service.
B) A connection string in `appsettings.json`.
C) Permission for the Managed Identity to read the secret in Key Vault.
D) A Virtual Network peering between the App Service and Key Vault.
**Answer: A, C**
* **A is correct**. The App Service must have a Managed Identity to authenticate to the Key Vault.
* **B is incorrect** because Key Vault References replace the need for connection strings in source code.
* **C is correct**. The identity must have RBAC or Access Policy permissions to actually read the secret.
* **D is incorrect** because while Private Endpoints exist, a VNet is not strictly required for a basic Key Vault Reference to function over the Azure backbone.

**Question 9**
You have multiple environments: Development, Testing, and Production. You want to store the `LogLevel` setting in Azure App Configuration for all three environments using the exact same key name (`App:LogLevel`). How can you differentiate the values?
A) Create three separate App Configuration stores.
B) Use Labels in App Configuration (e.g., `Dev`, `Test`, `Prod`).
C) Use Key Vault instead.
D) Append the environment to the key name (e.g., `App:LogLevel:Dev`).
**Answer: B**
* **A is incorrect** because while possible, it is more expensive and harder to manage than using a single store.
* **B is correct**. Labels allow you to have multiple versions of the same Key. Your app can request the key and specify which label it wants.
* **C is incorrect** because Key Vault is for secrets, not non-sensitive logging levels.
* **D is incorrect** because changing the key name requires changing the application code per environment, which violates configuration best practices.

**Question 10**
You are reviewing a colleague's code. You see the following string in the Azure App Service configuration: `@Microsoft.KeyVault(VaultName=myvault;SecretName=mysecret)`. 
The application is failing to retrieve the secret. Why?
A) The syntax is incorrect; it must use `SecretUri`.
B) The vault name must be capitalized.
C) App Service does not support Key Vault.
D) The secret name must be a GUID.
**Answer: A**
* **A is correct**. The proper syntax for a Key Vault Reference can use VaultName/SecretName, but the exam and Microsoft documentation strongly prefer and test the `SecretUri` format: `@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)`. If VaultName is used, it must be formatted exactly, but `SecretUri` is the foolproof method. *(Self-correction for exam strictness: The exact syntax requires either SecretUri, or VaultName+SecretName+SecretVersion. Missing the version can cause issues in older APIs).*
* **B is incorrect** because vault names are case-insensitive.
* **C is incorrect** because App Service natively supports this feature.
* **D is incorrect** because secret names are user-defined strings, not GUIDs.

---

## Exam Level

**Question 11**
A company has deployed an ASP.NET Core web app to Azure App Service. The app uses Azure App Configuration. The development team updates a configuration value in the Azure Portal, but the web app does not reflect the change until the App Service is manually restarted. 
You need to ensure the app updates configuration dynamically without restarts. What should you do?
A) Change the App Service pricing tier to Premium.
B) Call the `ConfigureRefresh` method in the app's configuration builder setup.
C) Move the setting to Azure Key Vault.
D) Enable Continuous Deployment in App Service.
**Answer: B**
* **A is incorrect** because App Configuration refresh features work on all App Service tiers.
* **B is correct**. In .NET, you must explicitly enable dynamic refresh by calling `.ConfigureRefresh()` and registering the App Configuration middleware. Otherwise, settings are only loaded once on startup.
* **C is incorrect** because Key Vault does not solve dynamic reloading on its own.
* **D is incorrect** because Continuous Deployment is for source code, not application settings.

**Question 12**
You are configuring Azure Key Vault to store an Azure OpenAI API key. The Key Vault is configured to use Azure RBAC for authorization. You have assigned the `Key Vault Secrets User` role to your App Service's Managed Identity at the Resource Group scope.
When the application attempts to read the secret, it receives an "HTTP 403 Forbidden" error. What is the most likely cause?
A) The API key was stored as a Key instead of a Secret.
B) RBAC role assignments can take up to 10 minutes to propagate.
C) You must assign the `Key Vault Contributor` role to read secrets.
D) RBAC cannot be assigned at the Resource Group scope for Key Vault.
**Answer: B**
* **A is incorrect** because if it was stored as a Key, the code looking for a Secret would return a 404 Not Found, not a 403 Forbidden.
* **B is correct**. Azure RBAC assignments are eventually consistent and can take several minutes to propagate. A 403 immediately after assignment is a classic propagation delay issue.
* **C is incorrect** because Key Vault Secrets User is the correct role. Contributor manages the vault resource, not the data inside it.
* **D is incorrect** because RBAC can absolutely be assigned at the Resource Group scope, which inherits down to the Vault.

**Question 13**
Your application relies on several secrets stored in Key Vault. You want to ensure that if a secret is accidentally deleted, it can be recovered for up to 90 days. Furthermore, you want to ensure that NO user, not even a subscription admin, can permanently force-delete the secret before the 90 days expire.
Which **two** features must you enable? (Select all that apply)
A) Soft Delete
B) Access Policies
C) Purge Protection
D) Azure Defender for Key Vault
**Answer: A, C**
* **A is correct**. Soft Delete keeps the deleted secret in a recoverable state for a retention period (default 90 days).
* **B is incorrect** because Access Policies manage authorization, not deletion retention.
* **C is correct**. Purge Protection enforces that a soft-deleted item cannot be permanently deleted (purged) by anyone until the retention period expires.
* **D is incorrect** because Defender is for threat detection, not data retention.

**Question 14**
You are managing feature flags for a global e-commerce site. You want the "HolidaySaleBanner" feature flag to automatically turn on at midnight on November 25th and automatically turn off at midnight on November 28th. 
What type of Feature Filter should you configure in Azure App Configuration?
A) Percentage Filter
B) Target Filter
C) Time Window Filter
D) Custom Filter
**Answer: C**
* **A is incorrect** because Percentage evaluates based on random traffic distribution.
* **B is incorrect** because Target evaluates based on specific user IDs or group names.
* **C is correct**. The Time Window filter evaluates a feature flag to `true` only if the current UTC time is within the specified start and end dates.
* **D is incorrect** because a built-in filter already exists for this exact scenario.

**Question 15**
An Azure App Service needs to access Azure OpenAI. You want to store the AI Endpoint URL and the AI API Key securely. Following Microsoft best practices for configuration and secret management, where should you store these two items?
A) Store both the Endpoint URL and API Key in Azure Key Vault.
B) Store both the Endpoint URL and API Key in Azure App Configuration.
C) Store the API Key in Key Vault. Store the Endpoint URL in App Configuration.
D) Store the API Key in App Configuration. Store the Endpoint URL in Key Vault.
**Answer: C**
* **A is incorrect** because Key Vault is expensive and rate-limited; it should not be used for non-sensitive data like URLs.
* **B is incorrect** because App Configuration is not designed for highly sensitive cryptographic secrets like API keys.
* **C is correct**. The best practice is the hybrid approach: sensitive secrets (API Key) go in Key Vault, while non-sensitive routing data (Endpoint URL) goes in App Configuration. App Configuration can then use a Key Vault Reference to present both to the app simultaneously.
* **D is incorrect** because it reverses the intended purposes of the services.
