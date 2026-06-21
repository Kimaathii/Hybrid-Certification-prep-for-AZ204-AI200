# F06 — C# Basics for AZ-204 Labs

> **Foundation Document 6 of 8** | **Pages:** 18–22 | **Study Time:** 3–4 hours
> **Prerequisites:** F01–F05 completed | **Exam Relevance:** 🔴 HIGH — every lab uses C#
> **Goal:** Read, understand, modify, and run existing C# code in AZ-204 labs

---

> **🎯 YOUR MINDSET FOR THIS ENTIRE DOCUMENT:**
> You are **NOT** learning to become a C# developer. You are learning to **read** and **understand** existing lab code — like a cook who follows a recipe rather than inventing dishes from scratch. Every explanation in this document reinforces one thing: *"You are not expected to write this from scratch. You need to understand what each line does so you can follow the lab."*

---

## Table of Contents

1. [Why C# for AZ-204?](#section-1-why-c-for-az-204)
2. [C# File Anatomy — Reading Code Top to Bottom](#section-2-c-file-anatomy--reading-code-top-to-bottom)
3. [Data Types — The Values You Work With](#section-3-data-types--the-values-you-work-with)
4. [Methods — Doing Things](#section-4-methods--doing-things)
5. [Async/Await — The Pattern You Will See Everywhere](#section-5-asyncawait--the-pattern-you-will-see-everywhere)
6. [Exception Handling — What To Do When Things Break](#section-6-exception-handling--what-to-do-when-things-break)
7. [The Azure SDK Pattern — How ALL SDK Code Is Structured](#section-7-the-azure-sdk-pattern--how-all-sdk-code-is-structured)
8. [SDK Package Reference Table](#section-8-sdk-package-reference-table)

---

## Section 1: Why C# for AZ-204?

### The Microsoft Connection

Here is a fact that makes your learning journey easier: **Microsoft built both Azure AND the C# programming language.** They are designed to work together. The Azure SDK (Software Development Kit — a collection of pre-built code libraries that let your program talk to Azure services) has its best support, most examples, and deepest integration in C#. Every official AZ-204 lab from Microsoft uses C# code.

This is not a coincidence. When Microsoft writes documentation, sample code, and exam questions for Azure, they write them in C# first. If you can read C# code, you can follow every lab in this course.

### What "Reading Code" Actually Means

Think of it like this:

> **🍳 The Recipe Analogy**
>
> Imagine you are given a cookbook and asked to make a specific dish. You do **not** need to be a professional chef who invents recipes. You need to:
>
> 1. **Read** the recipe and understand each step
> 2. **Follow** the steps in order
> 3. **Recognise** when something goes wrong (burnt smell = too hot)
> 4. **Adjust** small things (a pinch more salt, different plate)
>
> That is exactly what you will do with C# in AZ-204 labs:
>
> 1. **Read** the provided code and understand each line
> 2. **Run** the code using terminal commands
> 3. **Recognise** error messages when something breaks
> 4. **Modify** small things (change a connection string, update a container name)

You will **never** be asked to write a complete C# program from scratch on the exam or in labs. The code is always provided. Your job is to understand it.

### What You Will Be Able to Do After This Document

By the end of this document, you will be able to:

- Open a `.cs` file and understand every element from top to bottom
- Recognise the data types used in Azure SDK code
- Understand what methods do and how to call them
- Read `async/await` code without confusion
- Understand `try/catch` error handling blocks
- Recognise the universal Azure SDK pattern used in every lab
- Know which NuGet package (pre-built library) to install for each Azure service

📝 **NOTE:** If you completed Section 8 of F03 (Developer Basics), some of this will feel familiar. That is intentional. F03 gave you the quick tour — this document gives you the complete guided walkthrough with Azure-specific examples.

---

## Section 2: C# File Anatomy — Reading Code Top to Bottom

### The Analogy: Reading a Letter

Think of a C# file like a formal letter. Every formal letter has the same structure: the address at the top, the greeting, the body, and the signature at the bottom. A C# file has a similar predictable structure. Once you know the structure, you can read **any** C# file.

### A Real Lab File — Complete Example

Here is a realistic C# file you might see in an AZ-204 lab. This file connects to Azure Blob Storage and uploads a file. **Do not panic** — we will explain every single line below.

```csharp
// Program.cs — Upload a file to Azure Blob Storage

using Azure.Storage.Blobs;            // Line 1: Import the Blob Storage library
using Azure.Storage.Blobs.Models;     // Line 2: Import Blob data models
using Azure.Identity;                 // Line 3: Import Azure authentication library

namespace BlobUploadApp               // Line 4: Name this code group "BlobUploadApp"
{                                     // Line 5: Start of the namespace block
    class Program                     // Line 6: Define a blueprint called "Program"
    {                                 // Line 7: Start of the class block
        static async Task Main(string[] args)   // Line 8: Entry point — where code starts running
        {                             // Line 9: Start of the Main method block
            // Step 1: Set up connection details
            string accountName = "az204aborage2024";                            // Line 10: Store the storage account name as text
            string containerName = "lab-files";                                  // Line 11: Store the container name as text
            Uri serviceUri = new Uri($"https://{accountName}.blob.core.windows.net");  // Line 12: Build the URL to the storage account

            // Step 2: Create a client to talk to Azure Blob Storage
            BlobServiceClient blobServiceClient = new BlobServiceClient(         // Line 13: Create a "client" object that talks to Blob Storage
                serviceUri,                                                      // Line 14: Tell it which storage account to connect to
                new DefaultAzureCredential());                                   // Line 15: Use automatic Azure authentication

            // Step 3: Get a reference to the container
            BlobContainerClient containerClient =                                // Line 16: Create a "client" for the specific container
                blobServiceClient.GetBlobContainerClient(containerName);          // Line 17: Ask for the container by name

            // Step 4: Upload a file
            string localFilePath = "data/hello.txt";                             // Line 18: Path to the local file to upload
            BlobClient blobClient = containerClient.GetBlobClient("hello.txt");   // Line 19: Create a "client" for the specific blob (file)
            await blobClient.UploadAsync(localFilePath, overwrite: true);         // Line 20: Upload the file, overwrite if it already exists

            // Step 5: Confirm success
            Console.WriteLine($"Uploaded {localFilePath} to container '{containerName}'");  // Line 21: Print a success message
        }                             // Line 22: End of the Main method block
    }                                 // Line 23: End of the class block
}                                     // Line 24: End of the namespace block
```

### Line-by-Line Annotation Table

Now let us break down **every element** you just saw:

| Code Element | What It Means | Why You See It in Labs |
|---|---|---|
| `// comment text` | A **comment** — notes for humans. The computer ignores everything after `//` on that line. | Lab authors use comments to explain each step. Read them! |
| `using Azure.Storage.Blobs;` | An **import statement** — brings in pre-built code from a library so you can use it. Like adding an ingredient from the pantry before cooking. | Every lab imports Azure SDK libraries at the top. |
| `namespace BlobUploadApp` | A **namespace** — a named folder that groups related code together. Prevents naming conflicts. | Organises the lab code. You rarely need to change this. |
| `class Program` | A **class** — a blueprint or template that holds your code. Think of it as a chapter heading in a book. | Every C# file has at least one class. Lab code lives inside it. |
| `{ }` (curly braces) | **Block markers** — they mark the beginning and end of a section of code. Everything between `{` and `}` belongs together. | Braces are everywhere. They show where things start and end. |
| `static async Task Main(string[] args)` | The **entry point** — the exact spot where the program starts running. Like the front door of a house. `static` = belongs to the class itself. `async` = this method will wait for long operations. `Task` = this method does work that takes time. `Main` = the special name .NET looks for to start. `string[] args` = optional inputs from the terminal. | Every lab has a `Main` method. This is where execution begins. |
| `string accountName = "az204storage2024";` | A **variable declaration** — creates a named container (`accountName`) that holds a text value (`"az204storage2024"`). `string` means it holds text. | Labs store Azure resource names, connection strings, and URLs in variables. |
| `new Uri(...)` | **Object creation** — `new` creates a fresh instance of something. `Uri` is a type that represents a web address. | Labs create objects for URLs, clients, and credentials. |
| `$"https://{accountName}.blob.core.windows.net"` | **String interpolation** — the `$` before the quote lets you embed variables inside text using `{curly braces}`. The variable's value replaces the `{variable}` placeholder. | Labs build URLs dynamically by inserting account names. |
| `new BlobServiceClient(...)` | Creates a new **client object** — a tool that knows how to talk to a specific Azure service. This client talks to Blob Storage. | Every Azure SDK lab creates a client object first. |
| `new DefaultAzureCredential()` | Creates an **authentication object** that automatically finds your Azure login credentials. It checks multiple locations (az login, environment variables, managed identity). | The recommended way to authenticate in all production Azure code. |
| `await client.UploadAsync(...)` | **Calls a method and waits for it to finish.** `await` = "pause here until Azure responds." `Async` suffix = this method talks to Azure over the network and takes time. | Almost every Azure SDK call uses `await` and ends with `Async`. |
| `Console.WriteLine(...)` | **Prints text** to the terminal window. | Labs use this to show you results, confirmations, and debugging info. |

✅ **CHECKPOINT:** Look back at the full code example above. Can you now identify: (1) the import statements, (2) the entry point, (3) where the client is created, and (4) where the upload happens? If yes, you are ready for the next section. If not, re-read the table above — it is worth understanding these before moving on.

---

## Section 3: Data Types — The Values You Work With

### The Analogy: Different Types of Containers

In your kitchen, you use different containers for different things: a bottle for liquids, a jar for dry goods, a box for eggs. Each container is designed for a specific type of content. C# works the same way — every piece of data has a **type** that tells the computer what kind of value it holds and what you can do with it.

You will see exactly **five data types** repeatedly in AZ-204 lab code. Here they are:

### The Five Data Types in AZ-204 Labs

#### 1. `string` — Text Values

A `string` holds text — any sequence of characters wrapped in double quotes.

```csharp
// Strings you will see in Azure labs:
string resourceGroupName = "az204-rg";          // A resource group name
string connectionString = "DefaultEndpoints..."; // A connection string to Azure
string secretName = "DatabasePassword";           // A Key Vault secret name
string blobContainerName = "images";              // A Blob Storage container name
string endpoint = "https://myapp.azurewebsites.net"; // A URL
```

**In Azure context:** Resource names, connection strings, URLs, secret names, container names — all stored as strings.

#### 2. `int` — Whole Numbers

An `int` (short for "integer") holds a whole number — no decimal point.

```csharp
// Integers you will see in Azure labs:
int maxRetries = 3;                  // Number of retry attempts
int statusCode = 200;                // HTTP response status code
int throughput = 400;                // Cosmos DB Request Units (RUs)
int messageCount = 10;               // Number of messages to send to a queue
int ttlSeconds = 3600;               // Time-to-live in seconds (1 hour)
```

**In Azure context:** Retry counts, status codes, throughput settings, time durations — all integers.

#### 3. `bool` — True or False

A `bool` (short for "Boolean") holds exactly one of two values: `true` or `false`.

```csharp
// Booleans you will see in Azure labs:
bool overwrite = true;               // Whether to overwrite an existing blob
bool enableLogging = false;          // Whether to turn on diagnostic logging
bool isSuccessful = response.IsSuccessStatusCode;  // Did the HTTP call succeed?
bool exists = await containerClient.ExistsAsync();   // Does this container exist?
```

**In Azure context:** Feature flags, overwrite options, existence checks, success/failure indicators — all booleans.

#### 4. `double` — Decimal Numbers

A `double` holds a number with a decimal point.

```csharp
// Doubles you will see in Azure labs:
double costPerUnit = 0.018;          // Storage cost per GB
double latitude = 47.6062;           // Geographic coordinate
double percentage = 85.5;            // A percentage value
```

**In Azure context:** You will see `double` less frequently than the other types. It appears in cost calculations and geographic data.

#### 5. `var` — Let the Compiler Figure It Out

`var` is not actually a data type. It is a shortcut that says: *"I do not want to write the full type name — you figure it out, computer."* The compiler (the tool that translates C# into something the computer can run) looks at the right side of the `=` sign and determines the type automatically.

```csharp
// These two lines do EXACTLY the same thing:
BlobServiceClient client = new BlobServiceClient(connectionString);   // Explicit type
var client = new BlobServiceClient(connectionString);                  // var — compiler infers the type

// More var examples from Azure labs:
var response = await client.GetSecretAsync("MySecret");   // var = Response<KeyVaultSecret>
var container = blobServiceClient.GetBlobContainerClient("images");  // var = BlobContainerClient
var messages = await receiver.ReceiveMessagesAsync(10);    // var = IReadOnlyList<ServiceBusReceivedMessage>
```

**Why labs use `var`:** The full type names in Azure SDK code are often very long (`IReadOnlyList<ServiceBusReceivedMessage>`). Using `var` makes the code shorter and easier to read. The computer still knows the exact type — it just figures it out automatically.

### Data Types Quick Reference

| Type | What It Holds | Example | Azure Lab Usage |
|---|---|---|---|
| `string` | Text (characters in double quotes) | `"az204-rg"` | Resource names, URLs, connection strings |
| `int` | Whole numbers | `400` | Retry counts, status codes, RUs |
| `bool` | True or false | `true` | Overwrite flags, existence checks |
| `double` | Decimal numbers | `0.018` | Costs, percentages |
| `var` | Compiler decides (shortcut) | `var client = new X();` | Used everywhere to shorten long type names |

✅ **CHECKPOINT:** When you see `string containerName = "uploads";` in lab code, you now know: a variable named `containerName` is being created to hold the text value `"uploads"`. When you see `var client = new BlobServiceClient(...)`, you know: a variable named `client` is being created and the computer will figure out its type automatically.

---

## Section 4: Methods — Doing Things

### The Analogy: Buttons on a Remote Control

Think of a TV remote control. Each button does one specific thing: Volume Up, Channel Down, Mute. You do not need to know how the electronics inside the remote work. You just press the button (with maybe some input, like a channel number) and it does its job.

A **method** in C# is exactly like a button on a remote control. It is a named block of code that performs one specific action. You "press the button" (call the method), sometimes give it input (parameters), and it does its job and sometimes gives something back (a return value).

### Anatomy of a Method Call

When you see a method being called in lab code, here is what each part means:

```csharp
//         Object          Method Name      Parameter (input)
//           ↓                  ↓                ↓
BlobContainerClient container = blobServiceClient.GetBlobContainerClient("images");
//          ↑                                          ↑
//    What you get back                        The text value "images" is
//    (the return value)                       passed into the method
```

Let us break that down:

| Part | What It Means |
|---|---|
| `blobServiceClient` | The object you are calling the method on (like picking up the remote) |
| `.GetBlobContainerClient` | The method name — the specific action to perform (like pressing a button) |
| `("images")` | The parameter — input the method needs to do its job (like typing a channel number) |
| `BlobContainerClient container =` | The return value — what the method gives back after it finishes (like seeing the channel change) |

### Methods With No Return Value

Some methods do their job without giving anything back. These use the keyword `void` (meaning "nothing returned"):

```csharp
Console.WriteLine("Hello AZ-204");   // Prints text to the screen, returns nothing
//      ↑                    ↑
//  Method name          Parameter (the text to print)
```

### Methods With Multiple Parameters

Some methods need more than one input. Parameters are separated by commas:

```csharp
await blobClient.UploadAsync(localFilePath, overwrite: true);
//                   ↑            ↑                ↑
//              Method name    First parameter   Second parameter
//                             (file path)       (named parameter: overwrite = yes)
```

The `overwrite: true` syntax is called a **named parameter** — it explicitly says which setting you are providing. This makes code easier to read because you can see what `true` means without guessing.

### Async Methods — The `Async` Suffix

In Azure lab code, you will constantly see methods that end with the word `Async`:

```csharp
// All of these are async methods — they talk to Azure over the network
await client.GetSecretAsync("MySecret");          // Gets a secret from Key Vault
await containerClient.CreateIfNotExistsAsync();    // Creates a container if it does not exist
await blobClient.UploadAsync(filePath);            // Uploads a file to Blob Storage
await sender.SendMessageAsync(message);            // Sends a message to Service Bus
```

**The rule is simple:** If a method name ends with `Async`, it talks to Azure over the network and you must put `await` in front of it. We will cover `async/await` in depth in Section 5.

### Three Real Azure SDK Method Calls — Labelled

Here are three real method calls you will see in AZ-204 labs, fully labelled:

**Call 1: Creating a Blob Container**

```csharp
await containerClient.CreateIfNotExistsAsync();
// ↑        ↑                    ↑
// Wait   The container       Method: create the container,
// for    client object       but only if it does not already exist.
// it                         Returns nothing useful — we just need it to finish.
```

**Call 2: Getting a Secret from Key Vault**

```csharp
KeyVaultSecret secret = await secretClient.GetSecretAsync("DatabasePassword");
//     ↑          ↑       ↑       ↑              ↑              ↑
//   Type of    Variable  Wait  Client       Method name     Parameter:
//   return     to store  for   object       (get a secret)  the name of
//   value      result    it                                 the secret
```

**Call 3: Sending a Message to Service Bus**

```csharp
await sender.SendMessageAsync(new ServiceBusMessage("Order #12345 received"));
//  ↑    ↑         ↑                        ↑
// Wait  The     Method name            Parameter: a new message object
// for   sender  (send a message)       containing the text "Order #12345 received"
// it    object
```

💡 **KEY CONCEPT:** You do not need to memorise every method name. In labs, the code is provided. Your job is to look at a method call and understand: **who** is calling it (the object), **what** it does (the method name), **what** it needs (the parameters), and **what** it gives back (the return value).

---

## Section 5: Async/Await — The Pattern You Will See Everywhere

### Why This Section Matters

If there is one C# pattern you absolutely must understand for AZ-204, it is `async/await`. **Every single lab** uses it. **Every Azure SDK call** uses it. If you skip this section, the lab code will look confusing. If you read this section carefully, every lab will make sense.

### The Restaurant Ordering Analogy (Complete)

Imagine you are at a restaurant with three friends. You all want to order food.

**The WRONG way (synchronous / blocking):**

1. You walk up to the kitchen and say "I want a burger."
2. You **stand at the kitchen door and wait** while the chef cooks your burger. Nobody else can order.
3. After 15 minutes, you get your burger and walk back to the table.
4. **Now** Friend #1 walks to the kitchen and orders pasta.
5. Friend #1 **stands and waits** 20 minutes for pasta.
6. **Now** Friend #2 walks to the kitchen...
7. Total time: 15 + 20 + 25 + 10 = **70 minutes** for 4 meals.

**The RIGHT way (asynchronous / non-blocking):**

1. You give your order to the waiter: "I want a burger." The waiter writes it down and takes it to the kitchen.
2. You **sit back down** at your table. You are free to talk, drink water, check your phone.
3. Friend #1 gives their order to the waiter. The waiter takes it to the kitchen.
4. Friend #2 gives their order. Friend #3 gives their order.
5. All four orders are being prepared at the same time.
6. The waiter **brings each meal when it is ready**.
7. Total time: **25 minutes** (the longest meal) instead of 70.

**This is exactly what `async/await` does in code:**

- **Without async/await:** Your program stands at the "kitchen door" (Azure's servers) and waits for each response. Nothing else can happen.
- **With async/await:** Your program places the order (sends the request to Azure), then sits back down (frees up the computer to do other things), and picks up the result when it arrives.

### The Three Rules to Remember

Every time you see `async` and `await` in lab code, three rules are always true:

| Rule | What It Means | Example |
|---|---|---|
| **Rule 1:** If you use `await` inside a method, that method must be marked `async` | The `async` keyword on the method is like a sign on the restaurant door that says "We do table service" — it tells the computer this method will do waiting. | `static async Task Main(string[] args)` |
| **Rule 2:** Always `await` methods that end with `Async` | If the method name ends with `Async`, it talks to a remote service. You must `await` it, or your code will try to use the result before it arrives — like opening an empty takeaway bag before the food is packed. | `await client.GetSecretAsync("name")` |
| **Rule 3:** An `async` method returns `Task` (no result) or `Task<T>` (with result) | `Task` is like a receipt the waiter gives you — proof that your order is being prepared. `Task<T>` is a receipt that comes with the actual food. `T` is the type of food (the type of result). | `async Task Main(...)` — no result. `async Task<string> GetNameAsync()` — returns text. |

### Side-by-Side: WRONG vs RIGHT for Three Azure Operations

Here are three real Azure operations showing the wrong and right way to use `async/await`:

**Operation 1: Creating a Blob Container**

```csharp
// ❌ WRONG — Missing await. The code continues before the container is created.
// The variable "response" will hold a Task object, not the actual response.
var response = containerClient.CreateIfNotExistsAsync();

// ✅ RIGHT — await pauses until Azure confirms the container is created.
var response = await containerClient.CreateIfNotExistsAsync();
```

**Operation 2: Getting a Secret from Key Vault**

```csharp
// ❌ WRONG — Missing await. "secret" holds a Task, not the actual secret.
// If you try to read secret.Value, your code will crash.
var secret = secretClient.GetSecretAsync("DatabasePassword");

// ✅ RIGHT — await pauses until Key Vault returns the secret.
// Now secret.Value contains the actual password text.
var secret = await secretClient.GetSecretAsync("DatabasePassword");
```

**Operation 3: Sending a Service Bus Message**

```csharp
// ❌ WRONG — Missing await. The message might not be sent before your program ends.
// No error appears immediately, but the message silently never arrives.
sender.SendMessageAsync(new ServiceBusMessage("Order received"));

// ✅ RIGHT — await ensures the message is sent before continuing.
await sender.SendMessageAsync(new ServiceBusMessage("Order received"));
```

### Async Naming Convention — Examples from All 13 Modules

The `Async` suffix is a naming convention used across the entire Azure SDK. Here are real method names you will encounter in different modules of this course:

| Module | Async Method Name | What It Does |
|---|---|---|
| M01 — App Service | `webApp.GetPublishingProfileAsync()` | Gets the publishing settings for deployment |
| M02 — Azure Functions | `client.StartNewAsync("Orchestrator", input)` | Starts a Durable Functions orchestration |
| M03 — Containers | `registryClient.GetRepositoryAsync("myimage")` | Gets info about a container image in ACR |
| M04 — Blob Storage | `blobClient.UploadAsync(stream)` | Uploads a file to Blob Storage |
| M05 — Cosmos DB | `container.CreateItemAsync(item)` | Creates a new document in Cosmos DB |
| M06 — Entra ID (Auth) | `client.GetTokenAsync(request)` | Gets an authentication token |
| M07 — Key Vault | `secretClient.GetSecretAsync("name")` | Retrieves a secret from Key Vault |
| M08 — API Management | `client.CreateOrUpdateAsync(apiId, data)` | Creates or updates an API definition |
| M09 — Event-Based | `producerClient.SendAsync(events)` | Sends events to Event Hubs |
| M10 — Service Bus | `sender.SendMessageAsync(message)` | Sends a message to a Service Bus queue |
| M11 — App Insights | `telemetryClient.TrackEvent("OrderPlaced")` | Logs a custom event (this one is synchronous — no `Async`!) |
| M12 — Redis Cache | `database.StringSetAsync("key", "value")` | Stores a value in Redis Cache |
| M13 — CDN/Blob Features | `blobClient.SetMetadataAsync(metadata)` | Sets metadata on a blob |

📝 **NOTE:** Notice that the App Insights `TrackEvent` method in M11 does **not** end with `Async`. This is one of the rare Azure methods that is synchronous (it sends telemetry in the background on its own). The naming convention helps you know: if it ends with `Async`, use `await`. If it does not, do not.

💡 **KEY CONCEPT:** Whenever you see `await` in lab code, mentally read it as **"pause here and wait for Azure to respond."** Whenever you see the `Async` suffix on a method, mentally read it as **"this method talks to Azure over the network."** These two go together — always.

✅ **CHECKPOINT:** Look at this line of code:
```csharp
var result = await cosmosContainer.CreateItemAsync(newOrder);
```
Can you identify: (1) the `await` keyword that pauses until Azure responds, (2) the method name that ends with `Async`, (3) the object calling the method (`cosmosContainer`), and (4) the parameter being passed (`newOrder`)? If yes, you understand `async/await` well enough for every lab in this course.

---

## Section 6: Exception Handling — What To Do When Things Break

### The Analogy: A Safety Net

Imagine a tightrope walker performing high above the ground. Below them is a safety net. If they slip, the net catches them so they don't hit the ground. Without the net, one mistake is fatal.

In code, things go wrong all the time — especially when your code talks to a cloud service over the internet. The network might be slow. The resource might not exist. Your credentials might have expired. **Exception handling** is the safety net. It catches errors so your program can respond gracefully instead of crashing.

### The `try-catch` Pattern

C# uses a `try-catch` block for exception handling. The idea is simple:

- **`try`** = "try to do this risky operation"
- **`catch`** = "if it goes wrong, do this instead"

Here is a real Azure example — reading a secret from Key Vault:

```csharp
try
{
    // TRY to do the risky operation:
    // Connect to Key Vault and retrieve the secret named "DatabasePassword"
    KeyVaultSecret secret = await secretClient.GetSecretAsync("DatabasePassword");

    // If we reach this line, it worked! Print the secret value.
    Console.WriteLine($"Secret value: {secret.Value}");
}
catch (Azure.RequestFailedException ex)    // CATCH: if Azure returns an error
{
    // "ex" is the exception object — it contains details about what went wrong
    // ex.Status = the HTTP status code (like 404 or 403)
    // ex.Message = a human-readable description of the error
    Console.WriteLine($"Azure error {ex.Status}: {ex.Message}");
}
catch (Azure.Identity.CredentialUnavailableException ex)   // CATCH: if authentication fails
{
    // This means Azure could not find valid login credentials
    Console.WriteLine($"Authentication failed: {ex.Message}");
    Console.WriteLine("Have you run 'az login' in your terminal?");
}
catch (Exception ex)    // CATCH: any other unexpected error
{
    // This is the general safety net — catches anything else
    Console.WriteLine($"Unexpected error: {ex.Message}");
}
```

Let us break down the structure:

| Part | What It Does |
|---|---|
| `try { ... }` | Contains the code that **might** fail. If everything works, the `catch` blocks are skipped entirely. |
| `catch (Azure.RequestFailedException ex)` | Catches errors specifically from Azure services (like "not found" or "permission denied"). The `ex` variable holds the error details. |
| `catch (Azure.Identity.CredentialUnavailableException ex)` | Catches authentication-specific errors (like "you are not logged in"). |
| `catch (Exception ex)` | The "catch-all" — catches any error not already handled above. Always put this last. |
| `ex.Status` | The HTTP status code from Azure (e.g., `404`). |
| `ex.Message` | A text description of what went wrong (e.g., `"Secret not found"`). |

### How to Read a Stack Trace

When your program crashes, it shows a **stack trace** — a log of where the error happened and the path the code took to get there. Stack traces look intimidating, but there is a simple rule: **read the first line — that is the actual error.**

Here is an example stack trace with labels:

```
❶ Azure.RequestFailedException: Secret not found: DatabasePassword
❷    Status: 404 (Not Found)
❸    at Azure.Security.KeyVault.Secrets.SecretClient.GetSecretAsync(String name)
❹    at BlobUploadApp.Program.Main(String[] args) in C:\labs\Program.cs:line 15
❺    at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification
```

| Line | What It Tells You |
|---|---|
| **❶ First line** | **THE ERROR.** Read this first. `RequestFailedException: Secret not found: DatabasePassword` = the secret does not exist in Key Vault. |
| **❷ Status** | The HTTP status code: `404` means "not found." |
| **❸ Location in SDK** | Which Azure SDK method threw the error: `SecretClient.GetSecretAsync`. |
| **❹ Location in YOUR code** | Where in YOUR code the error occurred: `Program.cs`, line 15. **This is the line to look at.** |
| **❺ System internals** | Internal .NET plumbing. **Ignore these lines.** |

💡 **KEY CONCEPT:** When you see a stack trace, read lines ❶ and ❹ only. Line ❶ tells you **what** went wrong. Line ❹ tells you **where** in your code it happened.

### Common Azure SDK Exceptions

You will encounter these errors repeatedly in labs. Recognising them saves you debugging time:

| Error Code / Exception | What It Means | What To Do |
|---|---|---|
| **403 — Forbidden** | You do not have **permission** to do this operation. Your Azure account has correct credentials but lacks the right role (RBAC permission). | Check RBAC role assignments. You may need "Storage Blob Data Contributor" or "Key Vault Secrets User" role. |
| **404 — Not Found** | The resource **does not exist**. The secret name is wrong, the container does not exist, or the Cosmos DB item is missing. | Check the resource name for typos. Make sure the resource was created before trying to access it. |
| **409 — Conflict** | The resource **already exists** and you are trying to create it again. | Use `CreateIfNotExistsAsync()` instead of `CreateAsync()`, or add `overwrite: true`. |
| **429 — Too Many Requests** | You have sent too many requests too quickly. Azure is **throttling** you (slowing you down to protect the service). | Wait and retry. Many SDKs do this automatically. In Cosmos DB, this means you exceeded your provisioned RUs. |
| `CredentialUnavailableException` | Azure cannot find valid **login credentials** on your machine. `DefaultAzureCredential` checked everywhere and found nothing. | Run `az login` in your terminal. Make sure Azure CLI is installed and you are logged in. |

🚨 **EXAM ALERT:** The exam may show you error messages and ask you to identify the cause. Remember: **403 = permission problem** (right identity, wrong role). **401 = authentication problem** (not logged in at all). **404 = the resource does not exist.** These three are the most common in exam scenarios.

---

## Section 7: The Azure SDK Pattern — How ALL SDK Code Is Structured

### The Universal Three-Step Pattern

Here is the most powerful insight in this entire document:

**Every Azure SDK code sample in every lab in every module follows the exact same three-step pattern:**

```
Step 1:  CREATE CLIENT      →  Build a "client" object that knows how to talk to the Azure service
Step 2:  PERFORM OPERATIONS  →  Call methods on the client to do things (create, read, upload, send)
Step 3:  HANDLE RESPONSE     →  Process the result or handle errors
```

Think of it like making a phone call:

1. **Dial the number** (create the client — connect to the right service)
2. **Have the conversation** (perform operations — ask for data, send data)
3. **Act on what you heard** (handle the response — use the results, deal with errors)

Once you recognise this pattern, every lab starts to look the same — just with different service names.

### The Pattern in Action — Four Azure Services

Let us see this same three-step pattern applied to four different Azure services you will use in this course:

#### Service 1: Azure Blob Storage (Module 4)

```csharp
// ============================================
// STEP 1: CREATE CLIENT
// ============================================
BlobServiceClient blobServiceClient = new BlobServiceClient(   // Create a client for Blob Storage
    new Uri("https://az204storage.blob.core.windows.net"),      // The URL of the storage account
    new DefaultAzureCredential());                               // Use automatic Azure authentication

// ============================================
// STEP 2: PERFORM OPERATIONS
// ============================================
BlobContainerClient containerClient =                            // Get a reference to the "images" container
    blobServiceClient.GetBlobContainerClient("images");
await containerClient.CreateIfNotExistsAsync();                  // Create the container if it doesn't exist
BlobClient blobClient = containerClient.GetBlobClient("photo.jpg"); // Get a reference to a specific blob
await blobClient.UploadAsync("local/photo.jpg", overwrite: true);   // Upload a local file

// ============================================
// STEP 3: HANDLE RESPONSE
// ============================================
Console.WriteLine($"Uploaded photo.jpg to 'images' container");  // Confirm success
```

#### Service 2: Azure Cosmos DB (Module 5)

```csharp
// ============================================
// STEP 1: CREATE CLIENT
// ============================================
CosmosClient cosmosClient = new CosmosClient(                    // Create a client for Cosmos DB
    "https://az204cosmos.documents.azure.com:443/",               // The Cosmos DB account endpoint
    new DefaultAzureCredential());                                 // Use automatic Azure authentication

// ============================================
// STEP 2: PERFORM OPERATIONS
// ============================================
Database database = cosmosClient.GetDatabase("OrdersDB");         // Get a reference to the database
Container container = database.GetContainer("Orders");             // Get a reference to the container
var newOrder = new { id = "order-001", product = "Widget", quantity = 5, partitionKey = "electronics" };
ItemResponse<dynamic> response =                                   // Create a new item (document)
    await container.CreateItemAsync(newOrder, new PartitionKey("electronics"));

// ============================================
// STEP 3: HANDLE RESPONSE
// ============================================
Console.WriteLine($"Created item. Cost: {response.RequestCharge} RUs");  // Show RU cost
```

#### Service 3: Azure Key Vault (Module 7)

```csharp
// ============================================
// STEP 1: CREATE CLIENT
// ============================================
SecretClient secretClient = new SecretClient(                     // Create a client for Key Vault Secrets
    new Uri("https://az204vault.vault.azure.net/"),                // The Key Vault URL
    new DefaultAzureCredential());                                  // Use automatic Azure authentication

// ============================================
// STEP 2: PERFORM OPERATIONS
// ============================================
KeyVaultSecret secret =                                            // Retrieve a secret by name
    await secretClient.GetSecretAsync("DatabasePassword");

// ============================================
// STEP 3: HANDLE RESPONSE
// ============================================
Console.WriteLine($"Secret value: {secret.Value}");                // Use the secret value
```

#### Service 4: Azure Service Bus (Module 10)

```csharp
// ============================================
// STEP 1: CREATE CLIENT
// ============================================
ServiceBusClient serviceBusClient = new ServiceBusClient(          // Create a client for Service Bus
    "az204bus.servicebus.windows.net",                              // The Service Bus namespace
    new DefaultAzureCredential());                                   // Use automatic Azure authentication
ServiceBusSender sender =                                           // Create a sender for a specific queue
    serviceBusClient.CreateSender("orders-queue");

// ============================================
// STEP 2: PERFORM OPERATIONS
// ============================================
ServiceBusMessage message =                                         // Create a message object
    new ServiceBusMessage("Order #12345 received");
await sender.SendMessageAsync(message);                             // Send the message to the queue

// ============================================
// STEP 3: HANDLE RESPONSE
// ============================================
Console.WriteLine("Message sent to orders-queue");                  // Confirm success
await sender.DisposeAsync();                                        // Clean up the sender
await serviceBusClient.DisposeAsync();                              // Clean up the client
```

✅ **CHECKPOINT:** Look at all four examples above. Can you see the identical three-step pattern? (1) Create a client with a URL and credentials. (2) Call methods on the client to do things. (3) Handle the results. Despite being four completely different Azure services, the code structure is the same.

### Two Ways to Create a Client — Connection String vs DefaultAzureCredential

In labs, you will see two different approaches to creating a client. Here they are side by side:

#### Approach 1: Connection String (Simple — Used in Early Labs)

A **connection string** is a long text value that contains everything needed to connect: the account URL, the access key (password), and other settings. It is stored in one string variable.

```csharp
// Connection string approach — simple but LESS secure
// The connection string contains the full access key (like a master password)
string connectionString = Environment.GetEnvironmentVariable("AZURE_STORAGE_CONNECTION_STRING");
//                         ↑ Reading the connection string from an environment variable
//                           (NEVER hard-code it directly in your code!)

BlobServiceClient client = new BlobServiceClient(connectionString);
//                                                 ↑ Pass the entire connection string
```

#### Approach 2: DefaultAzureCredential (Production — Recommended)

`DefaultAzureCredential` is a smart authentication object that automatically finds your Azure credentials. It looks in multiple places, in order:

1. Environment variables (for CI/CD pipelines)
2. Managed Identity (when running in Azure — like App Service or Functions)
3. Visual Studio / VS Code credentials
4. Azure CLI (`az login`) credentials
5. Azure PowerShell credentials

```csharp
// DefaultAzureCredential approach — secure and RECOMMENDED
// No secrets in code at all. Credentials are found automatically.
BlobServiceClient client = new BlobServiceClient(
    new Uri("https://az204storage.blob.core.windows.net"),   // Just the URL — no secret key
    new DefaultAzureCredential());                            // Azure figures out authentication automatically
//  ↑ This works locally (uses az login) AND in Azure (uses Managed Identity)
//    with ZERO code changes between environments
```

### Side-by-Side Comparison

| Aspect | Connection String | DefaultAzureCredential |
|---|---|---|
| **Contains secrets?** | ✅ Yes — the full access key is embedded | ❌ No secrets in code |
| **Easy to start with?** | ✅ Very simple — one string | 🟡 Slightly more setup (RBAC roles needed) |
| **Safe for production?** | ❌ Risky — key can leak | ✅ Secure — no key to leak |
| **Works locally?** | ✅ Yes | ✅ Yes (via `az login`) |
| **Works in Azure?** | ✅ Yes | ✅ Yes (via Managed Identity) |
| **Code changes needed between local and Azure?** | 🟡 Often — different connection strings | ❌ None — same code everywhere |
| **Microsoft recommendation** | ⚠️ For development/learning only | ✅ **Recommended for all production use** |

🚨 **EXAM ALERT:** `DefaultAzureCredential` is the **recommended approach for ALL production Azure SDK authentication**. It requires zero code changes between local development (where it uses `az login`) and Azure deployment (where it uses Managed Identity). The exam expects you to know this is the preferred approach over connection strings.

💡 **KEY CONCEPT:** The three-step pattern — **CREATE CLIENT → PERFORM OPERATIONS → HANDLE RESPONSE** — is the skeleton of every Azure SDK lab in this course. Learn to spot it, and you will never feel lost in lab code.

---

## Section 8: SDK Package Reference Table

### What Is a Package?

Before you can use Azure SDK code in your program, you need to **install the right package** (pre-built library). Think of it like apps on your phone — your phone can make calls and send texts out of the box, but to order food, you need to download the food delivery app first.

In .NET, packages are called **NuGet packages** (NuGet is the package manager for .NET, like an app store for code libraries). You install them using the `dotnet add package` command in your terminal.

### The Complete AZ-204 SDK Package Table

Here is every Azure SDK package used across the 13 modules of this course:

| Module | Azure Service | Package Name | Install Command | What It Does |
|---|---|---|---|---|
| All modules | Authentication | `Azure.Identity` | `dotnet add package Azure.Identity` | Provides `DefaultAzureCredential` and other authentication methods for all Azure services |
| M04 | Blob Storage | `Azure.Storage.Blobs` | `dotnet add package Azure.Storage.Blobs` | Upload, download, list, and manage blobs (files) in Azure Storage |
| M05 | Cosmos DB | `Microsoft.Azure.Cosmos` | `dotnet add package Microsoft.Azure.Cosmos` | Create, read, query, and delete documents in Cosmos DB |
| M07 | Key Vault Secrets | `Azure.Security.KeyVault.Secrets` | `dotnet add package Azure.Security.KeyVault.Secrets` | Store and retrieve secrets (passwords, API keys) from Key Vault |
| M07 | App Configuration | `Azure.Data.AppConfiguration` | `dotnet add package Azure.Data.AppConfiguration` | Read and manage configuration settings from Azure App Configuration |
| M09 | Event Hubs | `Azure.Messaging.EventHubs` | `dotnet add package Azure.Messaging.EventHubs` | Send and receive events from Azure Event Hubs for streaming data |
| M10 | Service Bus | `Azure.Messaging.ServiceBus` | `dotnet add package Azure.Messaging.ServiceBus` | Send and receive messages from Azure Service Bus queues and topics |
| M12 | Redis Cache | `StackExchange.Redis` | `dotnet add package StackExchange.Redis` | Connect to and interact with Azure Cache for Redis |

📝 **NOTE:** Notice that most packages follow the `Azure.*` naming pattern (like `Azure.Storage.Blobs`, `Azure.Messaging.ServiceBus`). The two exceptions are `Microsoft.Azure.Cosmos` (Cosmos DB uses an older naming convention) and `StackExchange.Redis` (Redis uses a popular open-source library, not a Microsoft-built one).

### How to Install a Package

When a lab tells you to install a package, you will run a command like this in your terminal:

```bash
dotnet add package Azure.Storage.Blobs
#  ↑       ↑         ↑
#  The     "Add a     The name of the package
#  .NET    package    (exactly as shown in the table above)
#  tool    to my
#          project"
```

This command does three things:
1. Downloads the package from the NuGet package store (the internet)
2. Adds it to your project's `.csproj` file (the project configuration)
3. Makes the library's code available for you to `using` in your `.cs` files

After installing, you will see a new line in your `.csproj` file:

```xml
<ItemGroup>
  <PackageReference Include="Azure.Storage.Blobs" Version="12.19.1" />
  <!--                  ↑                              ↑                -->
  <!--          The package name              The version number        -->
</ItemGroup>
```

---

## 🔬 Hands-On Exercise: Your First .NET Console Project with Azure.Identity

Let us put everything together. In this exercise, you will:
1. Create a new .NET console project
2. Install the Azure.Identity package
3. Write a simple program that prints a greeting
4. Build and run it

### Prerequisites

- .NET SDK 8.0+ installed (verify with `dotnet --version` — you set this up in F03)
- A terminal open (PowerShell on Windows, Terminal on Mac/Linux)

### Step 1: Create a New Project

Open your terminal and run:

```bash
dotnet new console -n HelloAZ204
# ↑      ↑     ↑    ↑    ↑
# .NET  "new  "console  "-n" = name     The project name
# tool  project" project   flag
#              type"
```

This creates a folder called `HelloAZ204` with a starter project inside.

### Step 2: Navigate Into the Project Folder

```bash
cd HelloAZ204
# ↑    ↑
# "change   The folder to move into
#  directory"
```

### Step 3: Install the Azure.Identity Package

```bash
dotnet add package Azure.Identity
# Installs the authentication library used in every Azure SDK lab
```

Expected output:

```
info : Adding PackageReference for 'Azure.Identity' into project 'HelloAZ204.csproj'.
info : Package 'Azure.Identity' is compatible with all the specified frameworks.
info : PackageReference for package 'Azure.Identity' version '1.x.x' added to file 'HelloAZ204.csproj'.
```

### Step 4: View the Generated Code

Open `Program.cs` in any text editor. It will contain:

```csharp
// See https://aka.ms/new-console-template for more information
Console.WriteLine("Hello, World!");
```

### Step 5: Replace With Our Code

Replace the contents of `Program.cs` with:

```csharp
// Program.cs — Our first AZ-204 practice program

using Azure.Identity;   // Import the Azure authentication library we just installed

// Print a greeting
Console.WriteLine("=== Hello AZ-204! ===");
Console.WriteLine("Azure.Identity package installed successfully.");
Console.WriteLine();

// Show that DefaultAzureCredential is available
// (We are just creating the object here — not actually authenticating yet)
var credential = new DefaultAzureCredential();
Console.WriteLine($"Credential type: {credential.GetType().Name}");
Console.WriteLine("This credential object can authenticate with ANY Azure service.");
Console.WriteLine();

// Display the three-step Azure SDK pattern
Console.WriteLine("The Azure SDK Pattern:");
Console.WriteLine("  Step 1: CREATE CLIENT   (connect to a service)");
Console.WriteLine("  Step 2: PERFORM OPS     (do things with the service)");
Console.WriteLine("  Step 3: HANDLE RESPONSE (use the results)");
Console.WriteLine();
Console.WriteLine("You are ready for AZ-204 labs!");
```

### Step 6: Build the Project

```bash
dotnet build
# ↑      ↑
# .NET  "compile my code and check for errors"
# tool
```

Expected output (last few lines):

```
Build succeeded.
    0 Warning(s)
    0 Error(s)
```

If you see errors, check for typos in `Program.cs`. The most common mistake is a missing semicolon (`;`) at the end of a line.

### Step 7: Run the Project

```bash
dotnet run
# ↑      ↑
# .NET  "build AND execute my program"
# tool
```

Expected output:

```
=== Hello AZ-204! ===
Azure.Identity package installed successfully.

Credential type: DefaultAzureCredential
This credential object can authenticate with ANY Azure service.

The Azure SDK Pattern:
  Step 1: CREATE CLIENT   (connect to a service)
  Step 2: PERFORM OPS     (do things with the service)
  Step 3: HANDLE RESPONSE (use the results)

You are ready for AZ-204 labs!
```

✅ **CHECKPOINT:** If you see the output above, congratulations! You have successfully: (1) created a .NET project, (2) installed an Azure SDK package, (3) written C# code that uses the package, and (4) built and run the project. This is the exact same process you will follow in every AZ-204 lab.

💰 **COST WARNING:** This exercise is completely free — no Azure resources are created. We only installed a library and ran local code.

---

## Annotated Code Exercise: Reading a Complete Lab File

Now let us test your understanding. Below is a realistic C# file from an AZ-204 lab. **Your task:** Read through it and try to identify every element before looking at the annotation table below.

```csharp
// Program.cs — Read a secret from Azure Key Vault and print it

using Azure.Identity;                           // ❶
using Azure.Security.KeyVault.Secrets;           // ❷

string vaultUrl = "https://az204-lab-vault.vault.azure.net/";    // ❸

SecretClient client = new SecretClient(          // ❹
    new Uri(vaultUrl),                            // ❺
    new DefaultAzureCredential());                // ❻

try                                               // ❼
{
    KeyVaultSecret secret =                       // ❽
        await client.GetSecretAsync("AppApiKey"); // ❾

    Console.WriteLine($"Secret name: {secret.Name}");       // ❿
    Console.WriteLine($"Secret value: {secret.Value}");     // ⓫
}
catch (Azure.RequestFailedException ex)           // ⓬
{
    Console.WriteLine($"Error {ex.Status}: {ex.Message}");  // ⓭
}
```

### Annotation Key

| Label | Code | Explanation |
|---|---|---|
| ❶ | `using Azure.Identity;` | Import the authentication library (provides `DefaultAzureCredential`) |
| ❷ | `using Azure.Security.KeyVault.Secrets;` | Import the Key Vault secrets library (provides `SecretClient`) |
| ❸ | `string vaultUrl = "https://..."` | Variable storing the Key Vault URL as text |
| ❹ | `SecretClient client = new SecretClient(...)` | **Step 1: CREATE CLIENT** — Create a client object that talks to Key Vault |
| ❺ | `new Uri(vaultUrl)` | Convert the URL text into a `Uri` object (web address type) |
| ❻ | `new DefaultAzureCredential()` | Use automatic Azure authentication (checks az login, Managed Identity, etc.) |
| ❼ | `try` | Start of the "try this risky operation" block |
| ❽ | `KeyVaultSecret secret =` | Declare a variable to hold the secret that comes back from Key Vault |
| ❾ | `await client.GetSecretAsync("AppApiKey")` | **Step 2: PERFORM OPERATION** — Ask Key Vault for the secret named "AppApiKey" and wait for the response |
| ❿ | `Console.WriteLine($"Secret name: {secret.Name}")` | **Step 3: HANDLE RESPONSE** — Print the secret's name |
| ⓫ | `Console.WriteLine($"Secret value: {secret.Value}")` | Print the secret's actual value |
| ⓬ | `catch (Azure.RequestFailedException ex)` | If Azure returns an error, catch it here instead of crashing |
| ⓭ | `Console.WriteLine($"Error {ex.Status}: {ex.Message}")` | Print the error code and message for debugging |

### Self-Check Questions

1. Which lines are the `using` import statements? → ❶ and ❷
2. Where is the client created? → ❹ (with ❺ and ❻ as parameters)
3. Which line performs the actual Azure operation? → ❾
4. Why is `await` used on line ❾? → Because `GetSecretAsync` talks to Azure over the network and takes time
5. What happens if the secret does not exist? → Line ⓬ catches the error, and line ⓭ prints the error details instead of crashing

---

## 🔗 Module Connections

This foundation document connects to every module in the course:

| Module | How This Document Helps |
|---|---|
| **M01 — App Service** | Understanding App Settings and Connection Strings in code |
| **M02 — Functions** | Reading function code, triggers, bindings, and async patterns |
| **M03 — Containers** | Dockerfile and container app code structure |
| **M04 — Blob Storage** | Blob SDK code: `BlobServiceClient`, upload/download operations |
| **M05 — Cosmos DB** | Cosmos SDK code: `CosmosClient`, create/query items |
| **M06 — Authentication** | `DefaultAzureCredential`, token acquisition patterns |
| **M07 — Key Vault** | `SecretClient`, reading secrets, exception handling |
| **M08 — API Management** | Understanding API code that APIM fronts |
| **M09 — Event-Based** | Event Hubs producer/consumer SDK code |
| **M10 — Service Bus** | Service Bus sender/receiver SDK code |
| **M11 — Monitoring** | Application Insights SDK and telemetry code |
| **M12 — Redis Cache** | Redis client connection and operations |
| **M13 — CDN & Blob Features** | Blob metadata and lifecycle management code |

---

## What We Covered — Checklist

Use this checklist to confirm you are ready to move on. You should be able to check every box:

- [ ] **Why C# for AZ-204** — Microsoft built Azure and C# together. You are reading and running lab code, not writing from scratch.
- [ ] **C# File Anatomy** — You can identify `using` statements, `namespace`, `class`, `Main` method, variables, object creation (`new`), method calls, `await`, `Console.WriteLine`, string interpolation (`$""`), and comments (`//`).
- [ ] **Data Types** — You recognise `string` (text), `int` (whole numbers), `bool` (true/false), `double` (decimals), and `var` (compiler-inferred type).
- [ ] **Methods** — You understand method calls: the object, the method name, the parameters (inputs), and the return value (output). You know that methods ending with `Async` talk to Azure over the network.
- [ ] **Async/Await** — You understand the restaurant analogy. You know the three rules: (1) `await` requires `async` on the method, (2) always `await` methods ending with `Async`, (3) async methods return `Task` or `Task<T>`.
- [ ] **Exception Handling** — You can read a `try/catch` block. You know how to read a stack trace (first line = the error, look for your file and line number). You recognise common Azure errors: 403 (permission), 404 (not found), `CredentialUnavailableException` (not logged in).
- [ ] **The Azure SDK Pattern** — You recognise the three-step pattern: CREATE CLIENT → PERFORM OPERATIONS → HANDLE RESPONSE. You understand the difference between connection string and `DefaultAzureCredential` authentication.
- [ ] **SDK Packages** — You know the `dotnet add package` command. You know which package to install for each Azure service (Blob, Cosmos, Key Vault, Service Bus, Event Hubs, Redis, App Configuration, Identity).
- [ ] **Hands-On** — You have created a .NET console project, installed a NuGet package, and run the project successfully.

---

> **🎯 FINAL REMINDER:** You do not need to memorise C# syntax. You need to **recognise** it. When you open a lab file and see `await client.GetSecretAsync("name")`, your brain should say: *"This line calls the Azure Key Vault service to get a secret named 'name', and await means we wait for Azure to respond."* That level of understanding is all you need for every lab and every exam question in AZ-204.

---

*Next up: **F07 — Azure Networking for Developers** — understanding VNet Integration, Private Endpoints, and Service Endpoints for the AZ-204 labs.*
