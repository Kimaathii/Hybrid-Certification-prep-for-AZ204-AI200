# F03 — Developer Basics: Terminal, Code, Git, and APIs

> **Foundation Document 3 of 8** | **Pages:** 22–28 | **Study Time:** 3–4 hours
> **THE MOST IMPORTANT FOUNDATION DOCUMENT**
>
> **Prerequisites:** F01 (How the Cloud Works), F02 (Azure Hierarchy, ARM, and IaC)
> **What you'll be able to do after this document:** Open a terminal and navigate confidently, run Azure CLI commands, read JSON, understand environment variables, make HTTP requests, use Git for source control, and create .NET projects — everything you need to follow every AZ-204 lab.

---

## How to Use This Document

This is the longest foundation document because it covers **eight essential developer skills** that every AZ-204 lab assumes you already have. If you've never opened a terminal, never written code, and never used Git — that's perfectly fine. This document assumes **zero prior hands-on experience**.

Work through each section in order. Every section builds on the one before it. Do the hands-on exercises — reading about commands is not the same as typing them yourself.

---

# Section 1: The Terminal — Your New Best Friend

**Exam Relevance:** ⚪ CONTEXT ONLY — The terminal itself isn't tested, but you **cannot do any AZ-204 lab** without it.

---

## 1.1 What Is a Terminal?

### The Analogy: Texting Your Computer

Imagine you could **text your computer** the same way you text a friend. Instead of clicking through menus and buttons with your mouse, you type a short message, press Enter, and your computer does exactly what you asked.

That's what a terminal is. It's a **text-based conversation with your computer**.

When you use the regular graphical interface (clicking icons, dragging files), you're having a *face-to-face conversation* with your computer — pointing, gesturing, tapping things visually. The terminal is like switching to *text messaging* — faster, more precise, and you can send multiple instructions quickly.

💡 **KEY CONCEPT**
> A **terminal** (also called a **command line**, **command prompt**, or **shell**) is a text-based interface where you type commands and your computer executes them. Every AZ-204 lab uses the terminal for Azure CLI commands, Git operations, and running code.

### Why Not Just Click Around?

You might wonder: "If I can do everything by clicking, why learn the terminal?" Three reasons:

1. **Speed** — Creating an Azure resource group takes 8 clicks in the portal. In the terminal, it's one line: `az group create --name myRG --location eastus`
2. **Repeatability** — You can save terminal commands in a file and run them again. You can't save a sequence of mouse clicks.
3. **Automation** — Deployment pipelines (which deploy your code automatically) use terminal commands, not mouse clicks.

---

## 1.2 How to Open a Terminal

### Windows (Windows Terminal / PowerShell)

Windows has multiple terminal options. We'll use **Windows Terminal** with **PowerShell**:

1. Press the **Windows key** on your keyboard
2. Type **Terminal**
3. Click **Terminal** (the app with a dark icon showing `>_`)
4. A window opens with a blinking cursor — you're in!

If you don't have Windows Terminal installed:
1. Press the **Windows key**
2. Type **PowerShell**
3. Click **Windows PowerShell**

📝 **NOTE**
> **PowerShell** is the modern shell (command language) for Windows. The older **Command Prompt** (`cmd`) also works but has fewer features. We'll use PowerShell throughout this course. When you see `PS C:\Users\YourName>`, that's PowerShell's prompt telling you it's ready for your command.

### Mac (Terminal)

1. Press **Command + Space** to open Spotlight search
2. Type **Terminal**
3. Press **Enter**
4. A window opens with a blinking cursor — you're in!

### Linux

1. Press **Ctrl + Alt + T** (works on most Linux distributions)
2. Or search for "Terminal" in your applications menu

### Azure Cloud Shell (Works Everywhere — No Installation!)

Azure provides a terminal **inside your browser** — no installation needed:

1. Go to [https://shell.azure.com](https://shell.azure.com)
2. Log in with your Azure account
3. Choose **Bash** (recommended) or **PowerShell**
4. First time only: Azure creates a small storage account to save your files
5. You're in a fully working terminal, already logged into Azure!

💡 **KEY CONCEPT**
> **Azure Cloud Shell** is a terminal that runs in your web browser. It has the Azure CLI, Git, .NET, Node.js, and Python already installed. It's perfect when you don't want to install anything locally. Access it at [https://shell.azure.com](https://shell.azure.com) or by clicking the `>_` icon in the Azure Portal's top bar.

---

## 1.3 Essential Terminal Commands

Every terminal command follows this basic pattern:

```
command  argument1  argument2  --flag value
```

- **command** — the action you want to perform (like a verb: "go", "make", "show")
- **arguments** — what to perform the action on (like a noun: "to the store", "a folder")
- **flags** — optional modifiers that change how the command behaves (like an adverb: "quietly", "in detail")

Here are the commands you'll use every day:

### `pwd` — Print Working Directory (Where Am I?)

Tells you which folder you're currently inside. Think of it as asking "What's my current address?"

```bash
pwd
# Output: /Users/yourname
```

On Windows PowerShell, use:

```powershell
pwd
# Output: Path
#         ----
#         C:\Users\YourName
```

### `ls` (Mac/Linux) / `dir` (Windows) — List Contents (What's Here?)

Shows all files and folders in your current location. Like looking around the room you're in.

```bash
# Mac/Linux
ls
# Output: Desktop  Documents  Downloads  Music  Pictures

# Windows PowerShell
dir
# Output: (list of files and folders with dates and sizes)
```

📝 **NOTE**
> PowerShell actually understands `ls` too — it's an alias (shortcut name) for `Get-ChildItem`. So `ls` works on all three operating systems!

### `cd foldername` — Change Directory (Go There)

Moves you into a folder. Like walking into a room.

```bash
cd Desktop
# No output means success! You're now inside the Desktop folder.
```

```bash
cd Documents
# Now you're inside the Documents folder.
```

### `cd ..` — Go Up One Level (Go Back)

The two dots `..` mean "the parent folder" — the folder that contains your current folder. Like walking out of a room into the hallway.

```bash
cd ..
# You moved up one level.
```

### `cd ~` — Go Home

The tilde `~` means "my home folder" — the starting point for your user account. Like teleporting home no matter where you are.

```bash
cd ~
# You're now in /Users/yourname (Mac/Linux) or C:\Users\YourName (Windows)
```

### `mkdir foldername` — Make Directory (Create a Folder)

Creates a new folder. Like building a new room.

```bash
mkdir my-project
# Created a folder called "my-project" inside your current location.
```

### `clear` (Mac/Linux) / `cls` (Windows) — Clear the Screen

Wipes the screen clean. Your history isn't deleted — you just get a fresh view. Like erasing a whiteboard.

```bash
# Mac/Linux
clear

# Windows PowerShell
cls
```

### `cat filename` — Display File Contents

Shows the contents of a file right in your terminal. Like reading a document without opening an app.

```bash
cat notes.txt
# Output: (the contents of notes.txt appear on screen)
```

On Windows PowerShell:

```powershell
cat notes.txt
# PowerShell also understands 'cat' — it's an alias for Get-Content.
```

### `echo "text"` — Print Text

Prints text to the screen or writes text to a file. Like saying something out loud.

```bash
echo "Hello, AZ-204!"
# Output: Hello, AZ-204!
```

You can also create a file with echo:

```bash
echo "This is my notes file" > notes.txt
# Creates notes.txt with the text inside. The > means "write to this file."
```

### Quick Reference Table

| Command | Mac/Linux | Windows PowerShell | What It Does |
|---|---|---|---|
| Where am I? | `pwd` | `pwd` | Shows current folder path |
| What's here? | `ls` | `dir` (or `ls`) | Lists files and folders |
| Go into folder | `cd foldername` | `cd foldername` | Changes to that folder |
| Go back | `cd ..` | `cd ..` | Moves up one level |
| Go home | `cd ~` | `cd ~` | Goes to home folder |
| Create folder | `mkdir name` | `mkdir name` | Creates a new folder |
| Clear screen | `clear` | `cls` | Clears the terminal display |
| Show file | `cat filename` | `cat filename` | Displays file contents |
| Print text | `echo "text"` | `echo "text"` | Prints text to screen |

---

## 1.4 How to Read Output vs Errors

When you run a command, you'll see one of three things:

**1. Success with output** — The command worked and shows you results:
```bash
pwd
# /Users/yourname/Desktop    ← This is the result. Everything is fine.
```

**2. Success with no output** — The command worked, but there's nothing to show. **Silence means success** in the terminal world:
```bash
mkdir my-folder
# (no output)    ← This means it worked! The folder was created.
```

**3. Error** — Something went wrong. Errors usually contain words like "error", "not found", "denied", or "failed":
```bash
cd nonexistent-folder
# bash: cd: nonexistent-folder: No such file or directory   ← ERROR!
```

💡 **KEY CONCEPT**
> In the terminal, **no output usually means success**. This is the opposite of what you might expect. If you create a folder and see nothing, that's good! Errors are always explicitly shown.

---

## 1.5 The #1 Beginner Mistake: Running Commands in the Wrong Directory

The most common mistake beginners make is running a command **in the wrong folder**. The terminal always operates **relative to your current location**.

For example, if you're in your home folder and type `cd my-project`, the terminal looks for a folder called `my-project` **inside your home folder**. If `my-project` is actually on your Desktop, this fails.

**The fix: Always check where you are first!**

```bash
pwd                    # Step 1: Check where you are
# /Users/yourname

cd Desktop             # Step 2: Navigate to where you need to be
cd my-project          # Step 3: NOW this works because my-project is on Desktop
```

🚨 **EXAM ALERT**
> Throughout all AZ-204 labs, if a command fails with "file not found" or "directory not found," the first thing to check is your current directory with `pwd`. This single habit will save you hours of frustration.

---

## 1.6 Hands-On Exercise: Your First Terminal Adventure

🔬 **HANDS-ON**

**Goal:** Navigate to your Desktop, create a course folder, create a subfolder, and verify everything exists.

**Step 1: Open your terminal** (see Section 1.2 above for your operating system)

**Step 2: Check where you are**
```bash
pwd
```
Expected output: Your home directory (e.g., `/Users/yourname` or `C:\Users\YourName`)

**Step 3: Navigate to your Desktop**
```bash
cd Desktop
```

**Step 4: Create the course folder**
```bash
mkdir az204-labs
```

**Step 5: Go into the new folder**
```bash
cd az204-labs
```

**Step 6: Create a module subfolder**
```bash
mkdir module-01
```

**Step 7: Verify the subfolder exists**
```bash
ls
# Expected output: module-01
```

**Step 8: Verify your full path**
```bash
pwd
# Expected output: /Users/yourname/Desktop/az204-labs (Mac/Linux)
# Expected output: C:\Users\YourName\Desktop\az204-labs (Windows)
```

**Step 9: Go back to Desktop**
```bash
cd ..
```

**Step 10: Verify az204-labs is visible**
```bash
ls
# You should see az204-labs in the list
```

✅ **CHECKPOINT**
> Can you answer these?
> 1. What command shows your current folder?
> 2. What does `cd ..` do?
> 3. If you type `mkdir test` and see no output, what happened?
>
> **Answers:** 1. `pwd` 2. Moves up one folder level 3. It worked — the folder was created successfully!

---

# Section 2: The Azure CLI — Commanding Azure from Your Terminal

**Exam Relevance:** 🔴 HIGH — CLI commands appear in many exam questions and are used in every lab.

---

## 2.1 Why Use the CLI Instead of the Portal?

You already know the Azure Portal — the graphical website where you click through menus to create resources. So why learn a text-based alternative?

| Portal (Graphical) | CLI (Text Commands) |
|---|---|
| Click through 6–10 screens to create a resource | One command creates it |
| Can't easily repeat steps | Save commands and run again |
| Can't automate | Put commands in scripts for automation |
| Good for exploring and learning | Good for speed and production |
| You must remember where settings are | Commands are self-documenting |

Think of it this way: the Portal is like driving to the bank in person. The CLI is like mobile banking — faster for things you do repeatedly.

💡 **KEY CONCEPT**
> The **Azure CLI** (`az`) is a command-line tool that lets you create, manage, and delete Azure resources by typing commands instead of clicking through the portal. Every AZ-204 lab uses CLI commands extensively.

---

## 2.2 Installing the Azure CLI

### Windows (using winget)

```powershell
winget install -e --id Microsoft.AzureCLI
# winget       ← Windows package manager (like an app store in the terminal)
# install      ← action: install something
# -e           ← exact match (don't guess what I mean)
# --id         ← the unique identifier of the package
# Microsoft.AzureCLI  ← the package to install
```

After installation, **close and reopen your terminal** for the `az` command to become available.

### Mac (using Homebrew)

```bash
brew install azure-cli
# brew         ← Mac package manager (install it from https://brew.sh first)
# install      ← action: install something
# azure-cli    ← the package to install
```

### Linux (using curl)

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
# curl         ← downloads content from a URL
# -s           ← silent mode (don't show download progress)
# -L           ← follow redirects (if the URL points somewhere else, follow it)
# |            ← pipe: take the output and feed it as input to the next command
# sudo bash    ← run the downloaded script with administrator privileges
```

### Already Installed in Azure Cloud Shell

If you use Azure Cloud Shell ([https://shell.azure.com](https://shell.azure.com)), the CLI is already installed and you're already logged in!

---

## 2.3 Verifying Installation and Logging In

### Verify the CLI is installed

```bash
az --version
# Shows the Azure CLI version and all installed extensions.
# You should see something like: azure-cli  2.x.x
```

If you see "az: command not found" — the CLI isn't installed or your terminal needs to be restarted.

### Log in to Azure

```bash
az login
# Opens your default web browser with a Microsoft login page.
# Sign in with the same account you use for the Azure Portal.
# After signing in, your terminal shows your subscription information.
```

📝 **NOTE**
> `az login` opens a browser window for you to authenticate. If you're working on a server without a browser, you can use `az login --use-device-code` instead — it gives you a code to enter at [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin) on any device.

---

## 2.4 The CLI Command Structure Pattern

**This is the single most important thing to learn about the Azure CLI.** Every single command follows the same pattern:

```
az  [service]  [action]  --parameter value  --parameter value
```

Let's break this down:

| Part | What It Means | Example |
|---|---|---|
| `az` | The Azure CLI tool itself | Always starts with `az` |
| `[service]` | Which Azure service you're talking to | `group`, `webapp`, `functionapp`, `storage` |
| `[action]` | What you want to do | `create`, `list`, `show`, `delete`, `update` |
| `--parameter` | Named settings for the command | `--name`, `--resource-group`, `--location` |
| `value` | The value for that parameter | `myRG`, `eastus`, `S1` |

### Five Real Examples with Every Flag Explained

**Example 1: Create a Resource Group**
```bash
az group create --name az204-practice-rg --location eastus
# az           ← the Azure CLI tool
# group        ← service: resource groups
# create       ← action: create a new one
# --name       ← the name for this resource group
# az204-practice-rg  ← the name value we chose
# --location   ← which Azure region to create it in
# eastus       ← the East US datacenter
```

**Example 2: Create an App Service Plan**
```bash
az appservice plan create --name myPlan --resource-group az204-practice-rg --sku S1 --is-linux
# az           ← the Azure CLI tool
# appservice plan  ← service: App Service plans
# create       ← action: create a new one
# --name       ← the name for this plan
# myPlan       ← the name value
# --resource-group  ← which resource group to put it in
# az204-practice-rg  ← the resource group name
# --sku        ← the pricing tier (S1 = Standard, cheapest tier with deployment slots)
# S1           ← the tier value
# --is-linux   ← use a Linux host instead of Windows (no value needed, it's a boolean flag)
```

**Example 3: Create a Web App**
```bash
az webapp create --name myapp-$RANDOM --resource-group az204-practice-rg --plan myPlan --runtime "NODE:18-lts"
# az           ← the Azure CLI tool
# webapp       ← service: web apps
# create       ← action: create a new one
# --name       ← the name (must be globally unique — $RANDOM adds a random number)
# myapp-$RANDOM  ← name with random suffix for uniqueness
# --resource-group  ← which resource group
# --plan       ← which App Service plan to host it on
# --runtime    ← which programming language/version to use
# "NODE:18-lts"  ← Node.js version 18, Long Term Support
```

**Example 4: List All Resource Groups**
```bash
az group list --output table
# az           ← the Azure CLI tool
# group        ← service: resource groups
# list         ← action: show all of them
# --output     ← how to format the results
# table        ← display as a readable table (instead of raw JSON)
```

**Example 5: Delete a Resource Group**
```bash
az group delete --name az204-practice-rg --yes --no-wait
# az           ← the Azure CLI tool
# group        ← service: resource groups
# delete       ← action: delete it
# --name       ← which resource group to delete
# az204-practice-rg  ← the name of the group to delete
# --yes        ← don't ask for confirmation (normally it asks "Are you sure?")
# --no-wait    ← don't wait for deletion to complete (it can take minutes)
```

### Key Flags You'll See in ALL AZ-204 Labs

| Flag | What It Does | Example |
|---|---|---|
| `--name` or `-n` | Names the resource | `--name myapp` |
| `--resource-group` or `-g` | Which resource group | `--resource-group myRG` |
| `--location` or `-l` | Azure region | `--location eastus` |
| `--output` or `-o` | Output format | `--output table`, `--output json` |
| `--yes` or `-y` | Skip confirmation prompts | Used with delete commands |
| `--no-wait` | Return immediately, don't wait | Useful for long operations |
| `--query` | Filter JSON output | `--query "[].name"` |

---

## 2.5 How to Get Help

Forgot which flags a command needs? Add `--help` to any command:

```bash
az webapp --help
# Shows all available actions for web apps: create, delete, list, show, etc.

az webapp create --help
# Shows all available flags for creating a web app, with descriptions.
```

💡 **KEY CONCEPT**
> You don't need to memorize every CLI command. Use `--help` on any command to see what's available. The CLI itself is your reference guide. For example, `az functionapp create --help` shows every option for creating a Function App.

---

## 2.6 Hands-On Exercise: Your First Azure CLI Session

🔬 **HANDS-ON**

💰 **COST WARNING**
> This exercise creates and deletes a resource group. Resource groups are free — you're only charged for resources inside them. Since we create nothing inside, the cost is **$0.00**.

**Step 1: Verify Azure CLI is installed**
```bash
az --version
```
Expected: A version number like `azure-cli  2.x.x`

**Step 2: Log in to Azure**
```bash
az login
```
Expected: A browser opens, you sign in, and your subscription details appear in the terminal.

**Step 3: List your subscriptions**
```bash
az account list --output table
```
Expected: A table showing your subscription name, ID, and state (Enabled).

**Step 4: Create a practice resource group**
```bash
az group create --name az204-practice-rg --location eastus
```
Expected: A JSON response with `"provisioningState": "Succeeded"`.

**Step 5: List all resource groups**
```bash
az group list --output table
```
Expected: A table showing `az204-practice-rg` with location `eastus`.

**Step 6: Delete the practice resource group**
```bash
az group delete --name az204-practice-rg --yes --no-wait
```
Expected: No output (returns immediately because of `--no-wait`).

**Step 7: Verify deletion (wait 30 seconds first)**
```bash
az group list --output table
```
Expected: `az204-practice-rg` should no longer appear (or show as "Deleting").

✅ **CHECKPOINT**
> Can you answer these?
> 1. What pattern does every Azure CLI command follow?
> 2. What does `--output table` do?
> 3. What command shows help for creating a web app?
>
> **Answers:** 1. `az [service] [action] --parameter value` 2. Formats the output as a readable table instead of JSON 3. `az webapp create --help`

---

# Section 3: JSON — Reading and Writing Azure's Language

**Exam Relevance:** 🟡 MEDIUM — You must read JSON in ARM templates, CLI output, and configuration files.

---

## 3.1 What Is JSON?

### The Analogy: A Form with Labelled Fields

Imagine filling out a government form. Each field has a **label** (Name, Date of Birth, Address) and a **value** you write in (John Smith, 1990-01-15, 123 Main St). The form has a fixed structure so that any clerk can read it consistently.

**JSON** (JavaScript Object Notation, pronounced "JAY-son") is the digital equivalent of that form. It's a way of organizing data into **labelled fields** so that computers — and humans — can read it reliably. It looks like this:

```json
{
  "name": "John Smith",
  "dateOfBirth": "1990-01-15",
  "address": "123 Main St"
}
```

💡 **KEY CONCEPT**
> **JSON** is a text format for storing and transmitting structured data. Azure uses JSON everywhere: CLI output, ARM templates, API responses, configuration files, and app settings. If you learn to read JSON, you can read anything Azure throws at you.

---

## 3.2 Complete JSON Anatomy

Here's a realistic JSON object representing an Azure resource. Every element is labelled:

```json
{                                          // ← Opening curly brace: start of an OBJECT
  "name": "az204-webapp",                  // ← KEY: "name", VALUE: "az204-webapp" (a STRING)
  "resourceGroup": "az204-rg",             // ← Another key-value pair (STRING value)
  "location": "eastus",                    // ← Strings are ALWAYS wrapped in double quotes
  "isRunning": true,                       // ← BOOLEAN value: true or false (no quotes!)
  "instanceCount": 3,                      // ← NUMBER value: whole number (no quotes!)
  "costPerHour": 0.075,                    // ← NUMBER value: decimal number (no quotes!)
  "customDomain": null,                    // ← NULL: means "no value" or "not set"
  "tags": {                                // ← NESTED OBJECT: an object inside an object
    "environment": "development",          // ← Keys inside the nested object
    "team": "backend"
  },                                       // ← Closing brace for the nested object
  "enabledFeatures": [                     // ← ARRAY (list): square brackets contain a list
    "https",                               // ← First item in the array
    "customDomain",                        // ← Second item
    "deploymentSlots"                      // ← Third item (no comma after the LAST item!)
  ]                                        // ← Closing bracket for the array
}                                          // ← Closing curly brace: end of the object
```

### JSON Element Summary Table

| Element | Symbol | Example | Rule |
|---|---|---|---|
| **Object** | `{ }` | `{ "key": "value" }` | Curly braces. Contains key-value pairs. |
| **Array** (list) | `[ ]` | `["a", "b", "c"]` | Square brackets. Ordered list of values. |
| **String** (text) | `" "` | `"hello"` | Always in double quotes. Never single quotes. |
| **Number** | none | `42` or `3.14` | No quotes. Can be integer or decimal. |
| **Boolean** | none | `true` or `false` | No quotes. Only two possible values. |
| **Null** | none | `null` | No quotes. Means "empty" or "not set." |
| **Key** | `"key":` | `"name":` | Always a string. Followed by a colon. |
| **Comma** | `,` | after each pair | Separates items. **No comma after the last item.** |

🚨 **EXAM ALERT**
> When reading ARM templates or policy definitions in the exam, you'll need to identify specific values inside nested JSON. The skill you need: trace the path from the outer `{ }` through nested objects to find a specific value. Practice: `"tags" → "environment" → "development"` means: look for the `tags` key, look inside that object, find the `environment` key, and read its value.

---

## 3.3 Reading Azure CLI JSON Output

When you run an Azure CLI command, the default output format is JSON. Here's what `az group create` returns:

```json
{
  "id": "/subscriptions/abc123/resourceGroups/az204-rg",
  "location": "eastus",
  "name": "az204-rg",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

### Using `--output table` for Readability

Raw JSON can be hard to scan. Add `--output table` to get a clean table:

```bash
az group list --output table
```

Output:
```
Name            Location    Status
--------------  ----------  ---------
az204-rg        eastus      Succeeded
my-other-rg     westus2     Succeeded
```

### Using `--query` to Extract Specific Fields

The `--query` flag uses a language called **JMESPath** to filter JSON output. Think of it as asking a specific question about the data:

```bash
# Get just the names of all resource groups
az group list --query "[].name" --output tsv
# [].name  ← [] means "every item in the list", .name means "get the name field"
# --output tsv  ← tab-separated values: plain text without quotes or brackets

# Get the location of a specific resource group
az group show --name az204-rg --query "location" --output tsv
# "location"  ← get just the location field from the result

# Get names AND locations together
az group list --query "[].{Name:name, Location:location}" --output table
# {Name:name, Location:location}  ← create a custom table with column headers Name and Location
```

💡 **KEY CONCEPT**
> Use `--output table` when you want to quickly scan results visually. Use `--query` when you need to extract a specific piece of data from the JSON response. Use `--output tsv` when you want a plain value without formatting (useful in scripts).

---

## 3.4 Hands-On Exercise: Reading JSON

🔬 **HANDS-ON**

Given this JSON blob (which represents an Azure Function App), answer the five questions below:

```json
{
  "name": "order-processor-func",
  "resourceGroup": "ecommerce-rg",
  "location": "westeurope",
  "kind": "functionapp",
  "state": "Running",
  "hostNames": [
    "order-processor-func.azurewebsites.net"
  ],
  "httpsOnly": true,
  "dailyMemoryTimeQuota": 0,
  "siteConfig": {
    "numberOfWorkers": 1,
    "linuxFxVersion": "DOTNET-ISOLATED|8.0",
    "alwaysOn": false,
    "functionAppScaleLimit": 200
  },
  "tags": {
    "project": "ecommerce",
    "costCenter": "CC-4521",
    "environment": "production"
  }
}
```

**Questions:**

1. What is the name of this Function App?
2. Is HTTPS enforced?
3. What programming runtime does it use?
4. What is the cost center tag value?
5. How many host names does it have, and what is the first one?

**Answers:**

1. `"order-processor-func"` — found at the top-level `"name"` key
2. `true` — the `"httpsOnly"` field is set to `true`
3. `"DOTNET-ISOLATED|8.0"` — found inside `"siteConfig"` → `"linuxFxVersion"`
4. `"CC-4521"` — found inside `"tags"` → `"costCenter"`
5. It has 1 host name (the array has one item): `"order-processor-func.azurewebsites.net"`

✅ **CHECKPOINT**
> Can you answer these?
> 1. What symbols surround a JSON object?
> 2. What symbols surround a JSON array?
> 3. Are boolean values (`true`/`false`) wrapped in quotes?
>
> **Answers:** 1. Curly braces `{ }` 2. Square brackets `[ ]` 3. No — booleans and numbers have no quotes.

---

# Section 4: Environment Variables — The Secret Plumbing of Azure Apps

**Exam Relevance:** 🔴 HIGH — App Settings, Key Vault references, and environment variables appear in many exam questions.

---

## 4.1 What Are Environment Variables?

### The Analogy: Employee ID Badge

Imagine you work at a company. Your employee ID badge has information on it: your name, department, access level. You don't carry your entire personnel file around — you just carry the badge, and systems look up the information when they need it.

**Environment variables** work the same way. They're small **named values** stored outside your code that your program can look up when it runs. Instead of putting a database password directly in your code (dangerous!), you store it as an environment variable and tell your code: "look up the value called `DATABASE_PASSWORD`."

💡 **KEY CONCEPT**
> An **environment variable** is a named value (like `DATABASE_PASSWORD = "abc123"`) stored at the operating system level, outside your application code. Your code reads the value by name at runtime. This is the standard way Azure apps access configuration and secrets.

---

## 4.2 Why NEVER Hard-Code Secrets

A **secret** is any value that must stay private: database passwords, API keys, connection strings, encryption keys.

**Never put secrets directly in your code.** Here are three reasons why:

| Reason | What Happens |
|---|---|
| **1. GitHub Exposure** | If you push code to GitHub with a password in it, anyone on the internet can see it. Bots actively scan GitHub for leaked secrets. They can compromise your Azure account within minutes. |
| **2. Change Management** | If the password changes, you must edit code, rebuild, and redeploy. With environment variables, you just update the variable — no code change needed. |
| **3. Environment Differences** | Development uses a test database. Production uses the real database. With environment variables, the same code reads different values in each environment automatically. |

```csharp
// ❌ WRONG — secret hard-coded in source code
string connectionString = "Server=prod.database.net;Password=SuperSecret123";

// ✅ RIGHT — secret read from environment variable
string connectionString = Environment.GetEnvironmentVariable("DATABASE_CONNECTION");
```

---

## 4.3 Setting Environment Variables Locally

### Mac/Linux (Bash/Zsh)

```bash
# Set a variable for the current terminal session
export DATABASE_CONNECTION="Server=localhost;Password=devpassword"
# export     ← makes the variable available to programs you run from this terminal
# DATABASE_CONNECTION  ← the variable name (convention: ALL_CAPS with underscores)
# =          ← assignment (no spaces around the equals sign!)
# "Server=..." ← the value

# Verify it was set
echo $DATABASE_CONNECTION
# $DATABASE_CONNECTION  ← the $ sign reads the variable's value
# Output: Server=localhost;Password=devpassword
```

### Windows PowerShell

```powershell
# Set a variable for the current terminal session
$env:DATABASE_CONNECTION = "Server=localhost;Password=devpassword"
# $env:      ← tells PowerShell this is an environment variable
# DATABASE_CONNECTION  ← the variable name
# =          ← assignment (spaces around equals are okay in PowerShell)
# "Server=..." ← the value

# Verify it was set
echo $env:DATABASE_CONNECTION
# Output: Server=localhost;Password=devpassword
```

📝 **NOTE**
> Environment variables set with `export` (Mac/Linux) or `$env:` (Windows) only last for your **current terminal session**. When you close the terminal, they're gone. This is actually a security feature — secrets don't linger on your machine.

---

## 4.4 Reading Environment Variables in Code

### C# (.NET)

```csharp
// Read an environment variable by name
string dbConnection = Environment.GetEnvironmentVariable("DATABASE_CONNECTION");
// Environment  ← a built-in .NET class for system-level operations
// GetEnvironmentVariable  ← method that reads a variable by name
// "DATABASE_CONNECTION"   ← the name of the variable to read

if (dbConnection == null)
{
    Console.WriteLine("WARNING: DATABASE_CONNECTION is not set!");
    // If the variable doesn't exist, the method returns null (nothing).
}
else
{
    Console.WriteLine($"Connected to: {dbConnection}");
    // $"..."  ← string interpolation: inserts the variable value into the text
}
```

### Node.js (JavaScript)

```javascript
// Read an environment variable by name
const dbConnection = process.env.DATABASE_CONNECTION;
// process.env  ← built-in Node.js object containing all environment variables
// .DATABASE_CONNECTION  ← access the variable by name using dot notation

if (!dbConnection) {
    console.log("WARNING: DATABASE_CONNECTION is not set!");
} else {
    console.log(`Connected to: ${dbConnection}`);
    // `...${variable}...`  ← template literal: inserts the variable value (backticks, not quotes)
}
```

---

## 4.5 How Azure App Settings Become Environment Variables

Here's the magic: when you configure **App Settings** in the Azure Portal or CLI for an App Service or Function App, Azure **automatically injects them as environment variables** into your running application.

```
Azure Portal → App Service → Configuration → App Settings
   ↓
You add:  Name = DATABASE_CONNECTION,  Value = Server=prod.db.net;Password=ProdSecret
   ↓
Azure injects this as an environment variable when your app starts
   ↓
Your C# code reads it:  Environment.GetEnvironmentVariable("DATABASE_CONNECTION")
   ↓
Returns: "Server=prod.db.net;Password=ProdSecret"
```

**Setting App Settings via CLI:**

```bash
az webapp config appsettings set \
  --name myapp \
  --resource-group az204-rg \
  --settings DATABASE_CONNECTION="Server=prod.db.net;Password=ProdSecret"
# az webapp config appsettings set  ← service + action: set app settings on a web app
# --name        ← which web app to configure
# --resource-group  ← which resource group it's in
# --settings    ← the key=value pair(s) to set
```

💡 **KEY CONCEPT**
> Azure App Settings = environment variables. When you set `DATABASE_CONNECTION = "some value"` in App Settings, your code reads it using `Environment.GetEnvironmentVariable("DATABASE_CONNECTION")` in C# or `process.env.DATABASE_CONNECTION` in Node.js. No special Azure SDK is needed — standard environment variable reading is all it takes.

---

## 4.6 The .env File Pattern

For local development, typing `export` commands every time you open a terminal is tedious. The `.env` file pattern solves this:

1. Create a file named `.env` in your project root
2. Add your variables, one per line
3. Use a library to load them automatically when your app starts

```bash
# .env file — NEVER commit this to Git!
DATABASE_CONNECTION=Server=localhost;Password=devpassword
API_KEY=abc123-dev-key
STORAGE_ACCOUNT=devstorageaccount
```

In Node.js, use the `dotenv` package:

```javascript
require('dotenv').config();        // Loads .env file into process.env
// Now process.env.DATABASE_CONNECTION works as if you typed 'export' manually
```

In .NET, the `local.settings.json` file serves a similar purpose for Azure Functions:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
    "DATABASE_CONNECTION": "Server=localhost;Password=devpassword"
  }
}
```

🚨 **EXAM ALERT**
> The exam tests the **secrets management chain**: Secrets should be stored in **Azure Key Vault**, referenced from **App Settings** using Key Vault references (`@Microsoft.KeyVault(SecretUri=https://...)`), and read in your code as **environment variables**. The code never touches Key Vault directly in this pattern — it just reads an environment variable. This is the recommended production pattern.

---

# Section 5: HTTP and REST APIs — From Theory to Practice

**Exam Relevance:** 🔴 HIGH — Every Azure SDK call is an HTTP request under the hood. API Management, Functions HTTP triggers, and webhooks all require HTTP understanding.

---

## 5.1 What Is HTTP?

### The Analogy: The Postal System for the Internet

Imagine the postal system. When you mail a letter, you:
1. **Write the letter** (the content)
2. **Put it in an envelope** (formatting)
3. **Write the destination address** (URL)
4. **Mark it as a regular letter, certified mail, or package** (the type of request)
5. **Drop it at the post office** (send it)
6. **Receive a response** (or get a "return to sender" notice if something went wrong)

**HTTP** (HyperText Transfer Protocol) works the same way. It's the system that web browsers, mobile apps, and Azure services use to send and receive data over the internet.

Every time you visit a website, your browser sends an HTTP request to a server, and the server sends back an HTTP response with the page content.

💡 **KEY CONCEPT**
> **HTTP** is the communication protocol (set of rules) for sending and receiving data over the internet. When your code talks to Azure services, it uses HTTP. When the Azure CLI creates a resource, it sends HTTP requests to Azure's servers. When a user visits your web app, their browser sends HTTP requests to your app.

---

## 5.2 The Five HTTP Methods

HTTP defines **methods** (also called **verbs**) that describe what you want to do. There are five you need to know:

| Method | Analogy | What It Does | Azure Example |
|---|---|---|---|
| **GET** | "Give me something" — like asking a librarian for a book | Retrieves data. Never changes anything. | Get a list of blobs in a container |
| **POST** | "Here is something new" — like dropping a letter in the mailbox | Creates a new resource | Create a new Azure Function App |
| **PUT** | "Replace this entirely" — like replacing an entire book on the shelf | Replaces a resource completely | Update all settings on a web app |
| **PATCH** | "Update part of this" — like correcting one page in a book | Updates part of a resource | Change just the runtime version of an app |
| **DELETE** | "Remove this" — like throwing a book in the recycling bin | Deletes a resource | Delete a resource group |

📝 **NOTE**
> **GET** is the default method. When you type a URL in your browser's address bar, your browser sends a GET request. That's why visiting `https://portal.azure.com` loads the Azure Portal — your browser sent a GET request, and Azure's server responded with the page content.

---

## 5.3 HTTP Status Codes — The Full Reference

When a server responds to your HTTP request, it includes a **status code** — a three-digit number that tells you what happened. Think of it as the server's short answer: "OK," "not found," "you're not allowed," etc.

### Success Codes (2xx) — "It Worked!"

| Code | Name | Meaning | When You'll See It |
|---|---|---|---|
| **200** | OK | Request succeeded, here's the data | GET request returned data |
| **201** | Created | New resource was created successfully | POST request created a resource |
| **204** | No Content | Succeeded, but nothing to return | DELETE request removed a resource |

### Redirect Codes (3xx) — "Go Look Over There"

| Code | Name | Meaning | When You'll See It |
|---|---|---|---|
| **301** | Moved Permanently | Resource moved to a new URL | Old API endpoint redirecting to new one |

### Client Error Codes (4xx) — "You Made a Mistake"

| Code | Name | Meaning | When You'll See It |
|---|---|---|---|
| **400** | Bad Request | Your request is malformed or invalid | Missing required fields in JSON body |
| **401** | Unauthorized | You need to log in first | Missing or expired authentication token |
| **403** | Forbidden | You're logged in but not allowed | Insufficient RBAC permissions |
| **404** | Not Found | The resource doesn't exist | Typo in resource name or URL |
| **409** | Conflict | Conflicts with current state | Resource name already taken |
| **429** | Too Many Requests | You're sending too many requests (rate limited) | Exceeding Cosmos DB RU limit |

### Server Error Codes (5xx) — "The Server Broke"

| Code | Name | Meaning | When You'll See It |
|---|---|---|---|
| **500** | Internal Server Error | Something crashed on the server side | Bug in your deployed application code |
| **503** | Service Unavailable | Server is temporarily overloaded or down | Azure service experiencing an outage |

🚨 **EXAM ALERT**
> Know the difference between **401 (Unauthorized)** and **403 (Forbidden)**.
> - **401** = You haven't proven who you are (authentication problem). Fix: provide a valid token.
> - **403** = Azure knows who you are, but you don't have permission (authorization problem). Fix: assign the correct RBAC role.
> 
> The exam often presents scenarios where a developer gets a 401 or 403 and asks what to do.

---

## 5.4 REST APIs — The Waiter Analogy

### What Is a REST API?

Imagine you're at a restaurant. You (the **client**) don't walk into the kitchen to make your own food. Instead:

1. You look at the **menu** (the API documentation — tells you what's available)
2. You tell the **waiter** what you want (send an HTTP request to the API)
3. The waiter takes your order to the **kitchen** (the API forwards the request to the server)
4. The kitchen prepares your food (the server processes the request)
5. The waiter brings your food back (the API returns an HTTP response with data)

A **REST API** (Representational State Transfer Application Programming Interface) is a standardized way to design these "waiter" services. REST APIs use:

- **URLs** to identify resources (like table numbers identify where food goes)
- **HTTP methods** to define actions (GET = read menu, POST = place order, DELETE = cancel order)
- **JSON** for request and response bodies (the language of the conversation)

💡 **KEY CONCEPT**
> A **REST API** is a service that accepts HTTP requests at specific URLs and returns HTTP responses (usually in JSON). Azure's entire management layer is a REST API. When you run `az group create`, the CLI sends an HTTP POST request to `https://management.azure.com/subscriptions/.../resourceGroups/...`. Every Azure SDK call translates to HTTP under the hood.

### REST API Example — The Full Picture

Here's what happens when the Azure CLI creates a resource group:

```
Your Terminal                          Azure's REST API
─────────────                          ────────────────
az group create                  →     POST https://management.azure.com
  --name myRG                          /subscriptions/{id}/resourceGroups/myRG
  --location eastus                    ?api-version=2024-03-01

                                       Request Body (JSON):
                                       {
                                         "location": "eastus"
                                       }

                                 ←     Response: 201 Created
                                       {
                                         "name": "myRG",
                                         "location": "eastus",
                                         "properties": {
                                           "provisioningState": "Succeeded"
                                         }
                                       }
```

---

## 5.5 HTTP Headers

HTTP requests and responses include **headers** — metadata that describes the message itself. Think of headers as the information written on the outside of an envelope (not the letter inside).

Three headers you'll encounter constantly:

| Header | Purpose | Common Values |
|---|---|---|
| `Content-Type` | Tells the server what format your data is in | `application/json` (almost always in Azure) |
| `Authorization` | Proves who you are | `Bearer eyJhbGciOiJSUz...` (an access token) |
| `Accept` | Tells the server what format you want the response in | `application/json` |

### Example HTTP Request with Headers

```
POST https://management.azure.com/subscriptions/.../resourceGroups/myRG
Content-Type: application/json          ← "I'm sending you JSON data"
Authorization: Bearer eyJhbGciOiJ...   ← "Here's proof that I'm allowed to do this"
Accept: application/json                ← "Please send the response back in JSON"

{
  "location": "eastus"                  ← This is the request body (the actual data)
}
```

---

## 5.6 Request Body

Some HTTP methods (POST, PUT, PATCH) include a **request body** — the data you're sending to the server. GET and DELETE usually don't have a request body.

Think of the request body as the contents of the envelope. The URL is the address on the outside, the headers are the postmarks and stamps, and the body is the letter inside.

```json
// Example: Creating a Cosmos DB container via REST API
// Method: POST
// URL: https://management.azure.com/.../containers/orders
// Body:
{
  "properties": {
    "resource": {
      "id": "orders",
      "partitionKey": {
        "paths": ["/customerId"],
        "kind": "Hash"
      }
    },
    "options": {
      "throughput": 400
    }
  }
}
```

---

## 5.7 Why HTTP and REST Matter for AZ-204

1. **Azure SDK calls = HTTP requests** — When your C# code calls `blobClient.UploadAsync()`, it sends an HTTP PUT request to Azure Storage's REST API
2. **Azure Functions HTTP triggers** — You build APIs that receive HTTP requests
3. **API Management** — You manage and protect REST APIs
4. **Webhooks** — Event Grid sends HTTP POST requests to your endpoint when events occur
5. **Authentication** — OAuth tokens travel in HTTP Authorization headers
6. **Troubleshooting** — When something fails, the HTTP status code tells you why (401? Check auth. 429? Slow down.)

🔗 **MODULE LINK**
> HTTP concepts are used in every AZ-204 module. Module 1 (App Service) serves HTTP requests. Module 2 (Functions) has HTTP triggers. Module 8 (API Management) is entirely about managing HTTP APIs. Module 7 (Key Vault) reads secrets via HTTP.

✅ **CHECKPOINT**
> Can you answer these?
> 1. What HTTP method retrieves data without changing anything?
> 2. What status code means "you're logged in but not allowed"?
> 3. What header tells the server your data is in JSON format?
>
> **Answers:** 1. GET 2. 403 Forbidden 3. `Content-Type: application/json`

---

# Section 6: REST API Testing with Postman and curl

**Exam Relevance:** ⚪ CONTEXT ONLY — These tools aren't on the exam, but you'll use them in every lab to test your work.

---

## 6.1 Postman — The Visual API Tool

**Postman** is a desktop application that lets you send HTTP requests and see responses visually — no code needed. It's like having a web browser that lets you control exactly what request to send.

### Installation

1. Go to [https://www.postman.com/downloads/](https://www.postman.com/downloads/)
2. Download and install for your operating system
3. Create a free account (or skip sign-in for local use)
4. Open Postman

### Interface Walkthrough

When you open Postman, here's what you see:

```
┌──────────────────────────────────────────────────────────────┐
│  [GET ▼]  [https://api.example.com/users          ]  [Send] │
│                                                              │
│  Params │ Authorization │ Headers │ Body │ Pre-req │ Tests  │
│  ─────────────────────────────────────────────────────────── │
│  (tabs for configuring your request)                         │
│                                                              │
│  ═══════════════════════════════════════════════════════════  │
│  Response                                              200 OK│
│  Body │ Cookies │ Headers │ Test Results                     │
│  ─────────────────────────────────────────────────────────── │
│  [                                                           │
│    { "id": 1, "name": "Leanne Graham", ... },               │
│    { "id": 2, "name": "Ervin Howell", ... }                 │
│  ]                                                           │
└──────────────────────────────────────────────────────────────┘
```

| Part | What It Is |
|---|---|
| **Method dropdown** (`GET ▼`) | Choose GET, POST, PUT, PATCH, or DELETE |
| **URL bar** | Type the API URL here |
| **Send button** | Sends the request |
| **Params tab** | Add URL query parameters |
| **Authorization tab** | Configure authentication (tokens, API keys) |
| **Headers tab** | Add HTTP headers |
| **Body tab** | Write the request body (for POST/PUT/PATCH) |
| **Response section** | Shows the status code and response body |

### First GET Request

1. Set the method dropdown to **GET**
2. Type this URL in the URL bar: `https://jsonplaceholder.typicode.com/users`
3. Click **Send**
4. See the response: a JSON array of 10 fake users with names, emails, and addresses

```
Status: 200 OK    Time: 234ms    Size: 5.32 KB
```

This worked because:
- You sent a **GET** request (retrieve data)
- To a public test API that returns fake user data
- The server responded with **200 OK** (success) and the data in **JSON** format

### First POST Request

1. Set the method dropdown to **POST**
2. Type this URL: `https://jsonplaceholder.typicode.com/posts`
3. Click the **Body** tab
4. Select **raw** and choose **JSON** from the format dropdown
5. Type this JSON in the body area:

```json
{
  "title": "My First API Request",
  "body": "Learning REST APIs for AZ-204!",
  "userId": 1
}
```

6. Click **Send**
7. See the response:

```json
{
  "title": "My First API Request",
  "body": "Learning REST APIs for AZ-204!",
  "userId": 1,
  "id": 101
}
```

```
Status: 201 Created
```

The server returned **201 Created** — it accepted your data and created a new resource (with `id: 101`).

---

## 6.2 curl — The Terminal API Tool

**curl** (pronounced "curl," short for "Client URL") is a command-line tool that sends HTTP requests from your terminal. It's already installed on Mac, Linux, and modern Windows.

### Example 1: Simple GET Request

```bash
curl https://jsonplaceholder.typicode.com/users/1
# curl       ← the tool name
# https://...  ← the URL to send a GET request to (GET is the default method)
```

Output: A JSON object for user #1 (name, email, address, etc.)

### Example 2: GET with Readable Formatting

```bash
curl -s https://jsonplaceholder.typicode.com/users/1 | python -m json.tool
# -s         ← silent mode: hide the download progress bar
# |          ← pipe: send curl's output to the next command
# python -m json.tool  ← Python's built-in JSON formatter (pretty-prints the JSON)
```

📝 **NOTE**
> On Windows, if `python` isn't installed, you can use `curl -s https://... | ConvertFrom-Json | ConvertTo-Json -Depth 10` in PowerShell to format the JSON output.

### Example 3: POST with JSON Body

```bash
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"Test","body":"Hello from curl","userId":1}'
# -X POST    ← use the POST method (instead of the default GET)
# \          ← line continuation: the command continues on the next line (readability)
# -H         ← add a header to the request
# "Content-Type: application/json"  ← tell the server we're sending JSON
# -d         ← data: the request body to send
# '{"title":"Test",...}'  ← the JSON data (wrapped in single quotes to protect the double quotes inside)
```

### Example 4: Calling an Azure Function

```bash
curl "https://myapp.azurewebsites.net/api/ProcessOrder?code=myFunctionKey123"
# The URL includes:
# myapp.azurewebsites.net  ← your Function App's domain
# /api/ProcessOrder        ← the function name
# ?code=myFunctionKey123   ← the function key for authentication (query parameter)
```

### curl Flag Reference Table

| Flag | Full Name | What It Does |
|---|---|---|
| `-X` | `--request` | Sets the HTTP method (POST, PUT, DELETE, PATCH) |
| `-H` | `--header` | Adds an HTTP header |
| `-d` | `--data` | Sends data in the request body |
| `-s` | `--silent` | Hides the progress bar |
| `-i` | `--include` | Shows response headers along with the body |
| `-o` | `--output` | Saves response to a file instead of displaying it |
| `-v` | `--verbose` | Shows the full request and response details (for debugging) |

---

## 6.3 Hands-On Exercise: Same GET Request in Postman AND curl

🔬 **HANDS-ON**

**Goal:** Make the same API call using both tools and compare the experience.

**In Postman:**
1. Method: **GET**
2. URL: `https://jsonplaceholder.typicode.com/users/1`
3. Click **Send**
4. Note the response body (user #1's data)

**In curl (your terminal):**
```bash
curl https://jsonplaceholder.typicode.com/users/1
```

**Compare:** Both give you the same JSON response. Postman is visual and great for exploring. curl is fast and works in scripts.

✅ **CHECKPOINT**
> Can you answer these?
> 1. What curl flag sets the HTTP method?
> 2. What does the `-H` flag do?
> 3. In Postman, where do you write the JSON body for a POST request?
>
> **Answers:** 1. `-X` (e.g., `-X POST`) 2. Adds a header to the request 3. In the Body tab, with "raw" and "JSON" selected.

---

# Section 7: Git and Source Control

**Exam Relevance:** 🟡 MEDIUM — Git itself isn't deeply tested, but deployment from GitHub, CI/CD, and `.gitignore` concepts appear in exam questions.

---

## 7.1 What Is Git?

### The Analogy: A Time Machine for Your Code

Imagine you're writing a research paper. You save a copy called `paper_v1.docx`, then make changes and save `paper_v2.docx`, then `paper_v3_final.docx`, then `paper_v3_final_FINAL.docx`. This gets messy fast, and you can never easily compare versions.

**Git** is a **time machine for your code**. Instead of creating multiple copies, Git tracks every change you make. At any point, you can:
- **Go back in time** — restore your project to any previous version
- **See what changed** — compare today's code with last week's code
- **Work simultaneously** — multiple people edit the same project without overwriting each other

Every save point in Git is called a **commit** — like taking a snapshot of your entire project at that moment.

💡 **KEY CONCEPT**
> **Git** is a version control system that tracks changes to your files over time. Every "save point" (called a **commit**) records what changed, who changed it, and when. You can go back to any previous commit at any time — like having unlimited undo.

---

## 7.2 Why AZ-204 Students Need Git

You need Git for three specific reasons in this course:

1. **Deployment labs deploy FROM GitHub** — Several AZ-204 labs deploy your application to Azure by connecting Azure to your GitHub repository. When you push code to GitHub, Azure automatically deploys it.

2. **CI/CD is triggered by Git push** — Continuous Integration / Continuous Deployment (CI/CD) pipelines watch your Git repository. When you push new code, the pipeline automatically builds, tests, and deploys it.

3. **All sample code is on GitHub** — Microsoft's official AZ-204 lab code is hosted on GitHub at [https://github.com/MicrosoftLearning/AZ-204-DevelopingSolutionsforMicrosoftAzure](https://github.com/MicrosoftLearning/AZ-204-DevelopingSolutionsforMicrosoftAzure). You'll clone (copy) these repos to follow along.

---

## 7.3 Core Git Concepts

Before learning commands, understand these seven concepts:

| Concept | Analogy | Definition |
|---|---|---|
| **Repository (repo)** | A project folder with history tracking | A folder that Git is tracking. Contains all files and the complete history of changes. |
| **Commit** | A snapshot / save point | A recorded set of changes with a message describing what you did. |
| **Branch** | A parallel timeline | An independent line of development. The default branch is called `main`. |
| **Remote** | A cloud backup | A copy of your repository stored online (e.g., on GitHub). |
| **Push** | Upload to the cloud | Send your local commits to the remote repository (GitHub). |
| **Pull** | Download from the cloud | Get the latest commits from the remote repository to your local machine. |
| **Clone** | Download a copy | Create a local copy of a remote repository (including all history). |

### How These Concepts Connect

```
Your Computer (LOCAL)                    GitHub (REMOTE)
┌───────────────────┐                   ┌───────────────────┐
│  Working Files     │     push →       │  Remote Repository │
│  (what you edit)   │                   │  (cloud backup)    │
│        ↓           │     ← pull       │                    │
│  Staging Area      │                   │  Other people can  │
│  (ready to save)   │     clone →      │  see and contribute│
│        ↓           │                   │                    │
│  Local Repository  │                   │                    │
│  (saved history)   │                   │                    │
└───────────────────┘                   └───────────────────┘
```

---

## 7.4 Essential Git Commands

### First-Time Setup

```bash
# Check if Git is installed
git --version
# Expected output: git version 2.x.x
# If not installed: download from https://git-scm.com/downloads

# Set your name (used in commit history)
git config --global user.name "Your Name"
# --global  ← applies to all Git repositories on this computer

# Set your email (used in commit history)
git config --global user.email "your.email@example.com"
# Use the same email as your GitHub account
```

### Creating a New Repository

```bash
# Turn the current folder into a Git repository
git init
# init       ← initialize: sets up Git tracking in this folder
# Creates a hidden .git folder that stores all version history
```

### Checking Status

```bash
# See what's changed since your last commit
git status
# Shows: which files are new, modified, or ready to commit
# RED files    = changed but not staged (not ready to commit)
# GREEN files  = staged (ready to commit)
```

### Staging Changes (Adding to Snapshot)

```bash
# Stage ALL changed files (prepare them for a commit)
git add .
# add        ← stage files for the next commit
# .          ← the dot means "everything in the current folder and subfolders"

# Stage a single specific file
git add Program.cs
# Only stages Program.cs — other changed files stay unstaged
```

### Committing (Saving a Snapshot)

```bash
# Create a commit with a message describing what you changed
git commit -m "Add Azure Blob Storage upload function"
# commit     ← save a snapshot of all staged changes
# -m         ← message flag: the text that follows is the commit message
# "Add Azure..."  ← a short description of what changed (always use present tense)
```

### Viewing History

```bash
# See a list of all commits
git log --oneline
# log        ← show the commit history
# --oneline  ← show each commit on a single line (compact view)
# Output:
# a1b2c3d Add Azure Blob Storage upload function
# e4f5g6h Initial project setup
# 7h8i9j0 First commit
```

### Connecting to GitHub (Remote)

```bash
# Link your local repository to a GitHub repository
git remote add origin https://github.com/yourusername/your-repo.git
# remote add  ← add a new remote (cloud) connection
# origin      ← the name for this remote (convention: "origin" = the main remote)
# https://...  ← the URL of your GitHub repository
```

### Pushing (Uploading to GitHub)

```bash
# Upload your commits to GitHub
git push -u origin main
# push       ← send your local commits to the remote
# -u         ← set upstream: remember this remote/branch combo for future pushes
# origin     ← which remote to push to
# main       ← which branch to push
```

After the first push with `-u`, future pushes only need:

```bash
git push
# Git remembers where to push because of the -u flag
```

### Pulling (Downloading from GitHub)

```bash
# Download the latest changes from GitHub
git pull
# Fetches and merges changes from the remote into your local copy
```

### Cloning (Downloading a Copy)

```bash
# Download a complete copy of a GitHub repository
git clone https://github.com/MicrosoftLearning/AZ-204-DevelopingSolutionsforMicrosoftAzure.git
# clone      ← copy an entire remote repository to your local machine
# https://...  ← the URL of the repository to copy
# Creates a new folder with the repository name
```

### Complete Command Reference

| Command | What It Does |
|---|---|
| `git --version` | Check if Git is installed |
| `git config --global user.name "Name"` | Set your name for commits |
| `git config --global user.email "email"` | Set your email for commits |
| `git init` | Initialize Git in current folder |
| `git clone <url>` | Download a repository |
| `git status` | See what's changed |
| `git add .` | Stage all changes |
| `git add <file>` | Stage a specific file |
| `git commit -m "message"` | Save a snapshot |
| `git push` | Upload to remote |
| `git pull` | Download from remote |
| `git log --oneline` | View commit history |
| `git remote add origin <url>` | Connect to GitHub |

---

## 7.5 GitHub Setup Step-by-Step

**GitHub** is a website that hosts Git repositories online. It's where your code lives in the cloud.

### Step 1: Create a GitHub Account
1. Go to [https://github.com](https://github.com)
2. Click **Sign up**
3. Choose a username, enter your email, create a password
4. Complete the verification
5. Choose the **Free** plan (it's all you need)

### Step 2: Create a New Repository on GitHub
1. Click the **+** icon in the top right → **New repository**
2. Repository name: `az204-labs`
3. Description: "AZ-204 certification lab work"
4. Select **Private** (keeps your code visible only to you)
5. ✅ Check **Add a README file**
6. Click **Create repository**

### Step 3: Clone the Repository to Your Computer
```bash
cd ~/Desktop
git clone https://github.com/yourusername/az204-labs.git
cd az204-labs
```

### Step 4: Make a Change and Push
```bash
echo "# Module 01 Notes" > module01-notes.md
git add .
git commit -m "Add module 01 notes file"
git push
```

Go back to your GitHub repository page in the browser — you should see `module01-notes.md`!

---

## 7.6 The .gitignore File

Some files should **never** be tracked by Git. A `.gitignore` file tells Git which files and folders to ignore completely.

Here's a `.gitignore` file designed for AZ-204 labs:

```bash
# .gitignore — files and folders Git should NEVER track

# === SECRETS — NEVER COMMIT THESE ===
local.settings.json          # Azure Functions local config (contains connection strings!)
.env                         # Environment variables file (contains secrets!)
appsettings.Development.json # .NET development settings (may contain secrets)

# === BUILD OUTPUT — no need to track compiled code ===
bin/                         # .NET compiled output folder
obj/                         # .NET build intermediates folder
node_modules/                # Node.js dependencies (downloaded, not committed)
dist/                        # JavaScript build output

# === IDE AND OS FILES — editor-specific, not part of your project ===
.vs/                         # Visual Studio settings
.vscode/                     # VS Code settings (optional — some people DO commit this)
*.suo                        # Visual Studio user options
*.user                       # Visual Studio user preferences
.DS_Store                    # Mac OS folder metadata
Thumbs.db                    # Windows image cache

# === LOGS AND TEMP ===
*.log                        # Log files
*.tmp                        # Temporary files
```

🚨 **EXAM ALERT**
> **CRITICAL RULE: Never commit `local.settings.json` or `.env` to Git.** These files contain connection strings, API keys, and secrets. If you accidentally push them to a public GitHub repository, your Azure account can be compromised within minutes. Always add them to `.gitignore` BEFORE your first commit.
>
> If you already committed secrets to Git: **the secret is compromised even if you delete the file**, because Git remembers the entire history. You must rotate (change) the compromised credentials immediately.

💡 **KEY CONCEPT**
> The `.gitignore` file is a list of file patterns that Git should pretend don't exist. It prevents accidental commits of secrets, build output, and operating-system files. Create this file in the root of every repository before your first commit.

---

## 7.7 Hands-On Exercise: Clone, Change, Commit, Push

🔬 **HANDS-ON**

**Prerequisites:** GitHub account created, `az204-labs` repository created (Section 7.5)

**Step 1: Clone the repository (if you haven't already)**
```bash
cd ~/Desktop
git clone https://github.com/yourusername/az204-labs.git
cd az204-labs
```

**Step 2: Create the .gitignore file**
```bash
echo "local.settings.json" > .gitignore
echo ".env" >> .gitignore
echo "bin/" >> .gitignore
echo "obj/" >> .gitignore
echo "node_modules/" >> .gitignore
```

📝 **NOTE**
> The `>` symbol creates a new file (or overwrites existing). The `>>` symbol **appends** to an existing file. So the first line creates `.gitignore` with one entry, and the remaining lines add more entries.

**Step 3: Create a sample project file**
```bash
echo "Console.WriteLine(\"Hello AZ-204!\");" > Program.cs
```

**Step 4: Check status**
```bash
git status
# Expected: .gitignore and Program.cs shown in red (unstaged)
```

**Step 5: Stage all changes**
```bash
git add .
```

**Step 6: Commit**
```bash
git commit -m "Add .gitignore and first C# file"
```

**Step 7: Push to GitHub**
```bash
git push
```

**Step 8: Verify on GitHub**

Open `https://github.com/yourusername/az204-labs` in your browser. You should see:
- `.gitignore`
- `Program.cs`
- `README.md`

✅ **CHECKPOINT**
> Can you answer these?
> 1. What command saves a snapshot of your changes?
> 2. What file tells Git which files to ignore?
> 3. Why should you never commit `local.settings.json`?
>
> **Answers:** 1. `git commit -m "message"` 2. `.gitignore` 3. It contains connection strings and secrets that could be exposed if pushed to a public repository.

---

# Section 8: .NET Project Structure — Reading the Labs

**Exam Relevance:** 🟡 MEDIUM — You must understand .NET project structure to follow AZ-204 labs and read exam code snippets.

---

## 8.1 Installing the .NET SDK

The **.NET SDK** (Software Development Kit) is the toolset that lets you create, build, and run .NET applications. AZ-204 labs use .NET extensively.

### Installation

1. Go to [https://dotnet.microsoft.com/download](https://dotnet.microsoft.com/download)
2. Download **.NET 8.0 SDK** (or the latest LTS version)
3. Run the installer
4. Restart your terminal

### Verify Installation

```bash
dotnet --version
# Expected output: 8.0.xxx (the exact number doesn't matter — 8.0+ is fine)
```

---

## 8.2 Creating Projects

The `dotnet new` command creates new projects from templates. Two project types you'll use constantly:

### Console Application (most lab exercises)

```bash
dotnet new console -n MyFirstApp
# dotnet new  ← create a new project from a template
# console     ← template type: a command-line application
# -n          ← name: what to call the project
# MyFirstApp  ← the project name (also creates a folder with this name)
```

### Web API (REST API projects)

```bash
dotnet new webapi -n MyApi
# dotnet new  ← create a new project
# webapi      ← template type: a REST API application
# -n MyApi    ← project name
```

---

## 8.3 Project Folder Anatomy — Every File Labelled

After running `dotnet new console -n MyFirstApp`, here's what gets created:

```
MyFirstApp/                    ← Project root folder (named after your project)
├── MyFirstApp.csproj          ← PROJECT FILE: lists dependencies and build settings
│                                 (like a recipe card listing ingredients)
│
├── Program.cs                 ← ENTRY POINT: where your code starts running
│                                 (like the front door of a building)
│
├── bin/                       ← COMPILED OUTPUT: the finished, runnable program
│   └── Debug/                    (generated by 'dotnet build' — you never edit these)
│       └── net8.0/
│           └── MyFirstApp.dll
│
└── obj/                       ← BUILD INTERMEDIATES: temporary files used during compilation
                                  (generated automatically — you never edit these either)
```

📝 **NOTE**
> The `bin/` and `obj/` folders are generated by the build process. You never edit them directly, and they should be in your `.gitignore` file. If you delete them, they'll be recreated the next time you run `dotnet build`.

---

## 8.4 The .csproj File Annotated

The `.csproj` (C# Project) file is XML that tells .NET how to build your project and what packages it needs:

```xml
<Project Sdk="Microsoft.NET.Sdk">
<!-- Project: the root element of every .csproj file -->
<!-- Sdk="Microsoft.NET.Sdk": tells .NET which build system to use -->

  <PropertyGroup>
  <!-- PropertyGroup: contains project-level settings -->

    <OutputType>Exe</OutputType>
    <!-- OutputType: Exe = executable program, Library = reusable code -->

    <TargetFramework>net8.0</TargetFramework>
    <!-- TargetFramework: which .NET version to build for -->

    <ImplicitUsings>enable</ImplicitUsings>
    <!-- ImplicitUsings: auto-imports common namespaces (so you write less 'using' statements) -->

    <Nullable>enable</Nullable>
    <!-- Nullable: enables warnings when you might accidentally use null values -->

  </PropertyGroup>

  <ItemGroup>
  <!-- ItemGroup: lists NuGet packages (external libraries) your project depends on -->

    <PackageReference Include="Azure.Identity" Version="1.11.0" />
    <!-- PackageReference: a NuGet package dependency -->
    <!-- Include: the package name -->
    <!-- Version: which version to use -->

  </ItemGroup>

</Project>
```

💡 **KEY CONCEPT**
> The `.csproj` file is your project's configuration and dependency list. When you add a NuGet package, it appears here. You rarely edit this file manually — the `dotnet add package` command does it for you.

---

## 8.5 NuGet Packages: Adding Libraries

**NuGet** is .NET's package manager — a system for downloading and using code libraries written by others. Think of it as an app store for code: instead of writing everything from scratch, you install pre-built packages.

```bash
# Add a NuGet package to your project
dotnet add package Azure.Storage.Blobs
# dotnet add package  ← command to install a NuGet package
# Azure.Storage.Blobs  ← the package name (Microsoft's Azure Blob Storage SDK)

# This modifies your .csproj to include:
# <PackageReference Include="Azure.Storage.Blobs" Version="12.x.x" />
```

```bash
# Add a package with a specific version
dotnet add package Azure.Identity --version 1.11.0
# --version   ← specify an exact version (optional; latest is used by default)
```

---

## 8.6 Building vs Running

Two commands you'll use constantly:

### `dotnet build` — Compile Only

```bash
dotnet build
# Compiles your code into a runnable program.
# Checks for errors but does NOT run the program.
# Output goes to bin/Debug/net8.0/
# Use this to check if your code has any errors.
```

If there are errors, the output shows:
```
Build FAILED.
Program.cs(5,13): error CS1002: ; expected
    1 Error(s)
```

The error message tells you: **file** (Program.cs), **line** (5), **column** (13), and **what's wrong** (; expected — you forgot a semicolon).

### `dotnet run` — Build AND Execute

```bash
dotnet run
# First builds (compiles) your code, then runs it.
# Equivalent to: dotnet build + execute the compiled program.
# Use this when you want to see your program actually run.
```

| Command | What It Does | When to Use |
|---|---|---|
| `dotnet build` | Compiles, checks for errors | When you want to check if code is valid |
| `dotnet run` | Compiles AND runs | When you want to see the program's output |

---

## 8.7 Full Annotated Program.cs

Here's a complete `Program.cs` file with **every syntax element labelled**:

```csharp
// A 'using' statement imports a library (namespace) so you can use its classes.
// Think of it as: "I want to use tools from this toolbox."
using Azure.Identity;                  // Import Azure authentication classes
using Azure.Security.KeyVault.Secrets; // Import Key Vault secrets classes

// 'namespace' is a named container that groups related code together.
// Think of it as a department name: "This code belongs to the MyApp department."
namespace MyApp
{
    // A 'class' is a blueprint that groups methods (functions) and data together.
    // Think of it as a chapter in a book.
    class Program
    {
        // 'static' = this method belongs to the class itself, not an individual copy.
        // 'async' = this method can wait for long operations without freezing.
        // 'Task' = the return type for async methods (like a promise to return later).
        // 'Main' = the ENTRY POINT: the first method that runs when your program starts.
        // 'string[] args' = command-line arguments (values passed when running the program).
        static async Task Main(string[] args)
        {
            // Declare a variable.
            // 'string' = the data type (text).
            // 'vaultUrl' = the variable name (you choose this).
            // '=' = assignment: store this value in the variable.
            string vaultUrl = "https://my-vault.vault.azure.net/";

            // Create an object using 'new'.
            // 'var' = let the compiler figure out the type automatically.
            // 'new DefaultAzureCredential()' = create a new authentication credential object.
            var credential = new DefaultAzureCredential();

            // Create another object, passing the URL and credential as inputs.
            // 'SecretClient' = a class from the Azure Key Vault SDK.
            // 'new Uri(vaultUrl)' = converts the string URL to a Uri object.
            var client = new SecretClient(new Uri(vaultUrl), credential);

            // 'await' = pause HERE until the long operation finishes.
            // Without 'await', the program would continue before the result is ready.
            // 'GetSecretAsync' = get a secret from Key Vault (the Async suffix = async method).
            KeyVaultSecret secret = await client.GetSecretAsync("MySecret");

            // '$"..."' = string interpolation: insert variable values inside text.
            // '{secret.Value}' = access the Value property of the secret object.
            Console.WriteLine($"Secret value: {secret.Value}");
            // Console.WriteLine = print text to the terminal.

            // '//' = a comment. Everything after // on the same line is ignored by the compiler.
            // Comments are notes for humans reading the code.
        }
    }
}
```

---

## 8.8 async/await — The Restaurant Ordering Analogy

The `async` and `await` keywords appear in almost every AZ-204 lab. Here's how they work:

### The Analogy: Restaurant Ordering

Imagine you're at a restaurant with friends. When you place your food order:

- **Without async (synchronous):** You place your order, then you **stand at the kitchen door and wait** until your food is ready. Nobody else at your table can order until you sit back down. Everything stops while you wait.

- **With async (asynchronous):** You place your order, the waiter says "I'll bring it when it's ready," and you **sit down and chat with your friends**. The waiter brings the food when it's done. Meanwhile, your friends can also place their orders.

In code:
- **The waiter** = `async` method (handles the request and comes back later)
- **"I'll bring it when it's ready"** = `Task` (a promise that a result is coming)
- **Sitting down and waiting for the tap on the shoulder** = `await` (pause here until the result arrives)

```csharp
// ❌ SYNCHRONOUS (Blocking) — Like standing at the kitchen door
// The entire program freezes until GetSecret finishes.
KeyVaultSecret secret = client.GetSecret("MySecret");

// ✅ ASYNCHRONOUS (Non-blocking) — Like sitting down while the waiter works
// The program can do other work while waiting for the result.
KeyVaultSecret secret = await client.GetSecretAsync("MySecret");
```

### The Rules

1. Any method that uses `await` **must** be marked with `async`
2. Async methods return `Task` (no result) or `Task<T>` (result of type T)
3. Almost every Azure SDK method has an `Async` suffix version

### The Async Suffix Pattern

Azure SDK methods come in pairs. Always use the `Async` version:

| Synchronous (avoid) | Asynchronous (use this) |
|---|---|
| `client.GetSecret()` | `await client.GetSecretAsync()` |
| `container.Upload()` | `await container.UploadAsync()` |
| `client.CreateDatabase()` | `await client.CreateDatabaseAsync()` |
| `sender.SendMessage()` | `await sender.SendMessageAsync()` |

🚨 **EXAM ALERT**
> When exam questions show code, look for the **`Async` suffix** on method names. If a method ends with `Async`, it must be called with `await`, and the calling method must be marked `async`. Missing `await` = the operation starts but nobody waits for it to finish, causing bugs.

---

## 8.9 How to Read a Stack Trace

When your program crashes, .NET shows a **stack trace** — a list of breadcrumbs showing exactly where the error happened and the path the code took to get there.

```
Unhandled exception. Azure.RequestFailedException: 
Status: 403 (Forbidden)
ErrorCode: Forbidden

   at Azure.Security.KeyVault.Secrets.SecretClient.GetSecretAsync(String name)
   at MyApp.Program.Main(String[] args) in Program.cs:line 18
```

**How to read this:**

| Line | What It Tells You |
|---|---|
| `Unhandled exception` | The program crashed because nobody caught this error |
| `Azure.RequestFailedException` | **The error type** — a failed Azure API call |
| `Status: 403 (Forbidden)` | **THE ACTUAL PROBLEM** — you don't have permission |
| `ErrorCode: Forbidden` | Azure's specific error code |
| `at ...SecretClient.GetSecretAsync` | The Azure SDK method that failed |
| `at MyApp.Program.Main ... line 18` | **YOUR code**, line 18 in Program.cs, called the method that failed |

💡 **KEY CONCEPT**
> When reading a stack trace: **the first line is the actual error**. Everything below is the call path (how the program got there). Start at the top, find the error message, and look for the line in **your** code (it will show a file name and line number you recognize).

### Common Azure Error Types

| Error | HTTP Status | What It Means | How to Fix |
|---|---|---|---|
| `403 Forbidden` | 403 | No permission | Assign the correct RBAC role |
| `404 Not Found` | 404 | Resource doesn't exist | Check the resource name and URL |
| `CredentialUnavailableException` | — | Authentication failed | Run `az login` locally or check Managed Identity |
| `409 Conflict` | 409 | Name already taken | Use a different name or add `$RANDOM` suffix |

---

## 8.10 Node.js Alternative

Not every lab uses .NET. Some use **Node.js** (JavaScript runtime). Here's the quick equivalent:

```bash
# Check if Node.js is installed
node --version
# Expected: v18.x.x or higher

# Check if npm (Node Package Manager) is installed
npm --version
# Expected: 9.x.x or higher

# If not installed: download from https://nodejs.org
```

### Creating a Node.js Project

```bash
# Create a project folder
mkdir my-node-app
cd my-node-app

# Initialize a new Node.js project
npm init -y
# npm init   ← create a package.json file (Node.js equivalent of .csproj)
# -y         ← accept all defaults (skip the questionnaire)

# Install an Azure SDK package
npm install @azure/storage-blob
# npm install  ← download and install a package
# @azure/storage-blob  ← the Azure Blob Storage SDK for Node.js

# Run a JavaScript file
node index.js
# node       ← the Node.js runtime
# index.js   ← the file to run
```

### Side-by-Side Comparison

| Task | .NET (C#) | Node.js (JavaScript) |
|---|---|---|
| Check version | `dotnet --version` | `node --version` |
| Create project | `dotnet new console -n MyApp` | `npm init -y` |
| Add package | `dotnet add package Azure.Storage.Blobs` | `npm install @azure/storage-blob` |
| Run project | `dotnet run` | `node index.js` |
| Project config | `.csproj` (XML) | `package.json` (JSON) |
| Dependencies folder | `obj/` (auto-managed) | `node_modules/` (auto-managed) |

---

## 8.11 Complete NuGet Package Reference Table for All 13 AZ-204 Modules

This table lists every Azure SDK package you'll install across all 13 AZ-204 modules. Bookmark this page — you'll refer to it throughout the course.

| Module | Service | NuGet Package | Install Command |
|---|---|---|---|
| All modules | **Azure Authentication** | `Azure.Identity` | `dotnet add package Azure.Identity` |
| M01 | App Service | *(deployed apps, no SDK needed)* | — |
| M02 | Azure Functions | `Microsoft.Azure.Functions.Worker` | `dotnet add package Microsoft.Azure.Functions.Worker` |
| M03 | Container Registry | *(CLI-based, no SDK needed)* | — |
| M04 | Blob Storage | `Azure.Storage.Blobs` | `dotnet add package Azure.Storage.Blobs` |
| M05 | Cosmos DB | `Microsoft.Azure.Cosmos` | `dotnet add package Microsoft.Azure.Cosmos` |
| M06 | Microsoft Graph | `Microsoft.Graph` | `dotnet add package Microsoft.Graph` |
| M07 | Key Vault Secrets | `Azure.Security.KeyVault.Secrets` | `dotnet add package Azure.Security.KeyVault.Secrets` |
| M07 | App Configuration | `Azure.Data.AppConfiguration` | `dotnet add package Azure.Data.AppConfiguration` |
| M08 | API Management | *(portal/policy-based, no SDK needed)* | — |
| M09 | Event Grid / Event Hubs | `Azure.Messaging.EventHubs` | `dotnet add package Azure.Messaging.EventHubs` |
| M10 | Service Bus | `Azure.Messaging.ServiceBus` | `dotnet add package Azure.Messaging.ServiceBus` |
| M11 | Application Insights | `Microsoft.ApplicationInsights` | `dotnet add package Microsoft.ApplicationInsights` |
| M12 | Redis Cache | `StackExchange.Redis` | `dotnet add package StackExchange.Redis` |
| M13 | CDN / Front Door | *(portal/CLI-based, no SDK needed)* | — |

🚨 **EXAM ALERT**
> The `Azure.Identity` package with `DefaultAzureCredential` is used in **every** module that connects to an Azure service via SDK. `DefaultAzureCredential` automatically detects your authentication method: it uses your `az login` session locally and **Managed Identity** when running in Azure. This is the recommended authentication approach — zero code changes between development and production.

---

## 8.12 Hands-On Exercise: Create Your First .NET Project

🔬 **HANDS-ON**

**Goal:** Create a .NET console project, add the Azure.Identity package, write a "Hello AZ-204" program, and run it.

**Step 1: Navigate to your lab folder**
```bash
cd ~/Desktop/az204-labs
```

**Step 2: Create a new console project**
```bash
dotnet new console -n HelloAZ204
```

Expected: A new folder `HelloAZ204` with `Program.cs` and `HelloAZ204.csproj`.

**Step 3: Navigate into the project**
```bash
cd HelloAZ204
```

**Step 4: Add the Azure.Identity package**
```bash
dotnet add package Azure.Identity
```

Expected: Package is downloaded and added to `.csproj`.

**Step 5: Open Program.cs and replace its content**

Use your text editor (VS Code, Notepad, or any editor) to replace the content of `Program.cs` with:

```csharp
// Import the Azure.Identity library
using Azure.Identity;

// Print a greeting
Console.WriteLine("Hello AZ-204!");

// Show that Azure.Identity is installed and working
var credential = new DefaultAzureCredential();
Console.WriteLine($"Azure.Identity loaded successfully!");
Console.WriteLine($"Credential type: {credential.GetType().Name}");
Console.WriteLine("You're ready for AZ-204 labs!");
```

**Step 6: Build the project (check for errors)**
```bash
dotnet build
```

Expected output:
```
Build succeeded.
    0 Warning(s)
    0 Error(s)
```

**Step 7: Run the project**
```bash
dotnet run
```

Expected output:
```
Hello AZ-204!
Azure.Identity loaded successfully!
Credential type: DefaultAzureCredential
You're ready for AZ-204 labs!
```

🎉 **Congratulations!** You just created, built, and ran a .NET project with an Azure SDK package!

💰 **COST WARNING**
> This exercise runs entirely on your local machine — no Azure resources are created. Cost: **$0.00**.

✅ **CHECKPOINT**
> Can you answer these?
> 1. What command creates a new .NET console project?
> 2. What's the difference between `dotnet build` and `dotnet run`?
> 3. What keyword must appear before calling an `Async` method?
>
> **Answers:** 1. `dotnet new console -n ProjectName` 2. `build` only compiles and checks for errors; `run` compiles AND executes 3. `await`

---

# Self-Check Quiz: 8 Questions Across All 8 Sections

Test yourself on everything covered in this document. Try to answer each question before looking at the answer.

---

**Question 1 (Section 1 — Terminal)**
You type `mkdir my-project` in the terminal and see **no output**. What happened?

**A)** The command failed silently
**B)** The folder was created successfully
**C)** You need to restart the terminal
**D)** You're in the wrong directory

> **Answer: B** — In the terminal, silence means success. The folder was created. If there was a problem, you would see an error message.

---

**Question 2 (Section 2 — Azure CLI)**
What is the correct Azure CLI command pattern?

**A)** `az --parameter value [service] [action]`
**B)** `az [action] [service] --parameter value`
**C)** `az [service] [action] --parameter value`
**D)** `[service] az [action] --parameter value`

> **Answer: C** — The pattern is always `az [service] [action] --parameter value`. For example: `az group create --name myRG --location eastus`.

---

**Question 3 (Section 3 — JSON)**
Which of these is valid JSON?

**A)** `{'name': 'John'}` — single quotes
**B)** `{"name": "John",}` — trailing comma
**C)** `{"name": "John"}` — double quotes, no trailing comma
**D)** `{name: "John"}` — key without quotes

> **Answer: C** — JSON requires **double quotes** for both keys and string values. No trailing comma after the last item. Keys must always be quoted.

---

**Question 4 (Section 4 — Environment Variables)**
A developer stores a database password directly in their source code and pushes it to GitHub. What is the risk?

**A)** The code won't compile
**B)** Azure will reject the deployment
**C)** Bots can scan GitHub and steal the password within minutes
**D)** The password will automatically expire

> **Answer: C** — Bots actively scan public GitHub repositories for secrets. Storing a password in code that gets pushed to GitHub is a critical security vulnerability. Use environment variables and Key Vault instead.

---

**Question 5 (Section 5 — HTTP and REST APIs)**
A developer receives a `401 Unauthorized` response. What should they do?

**A)** Change the HTTP method from GET to POST
**B)** Provide valid authentication credentials (e.g., a Bearer token)
**C)** Assign an RBAC role to their identity
**D)** Retry the same request

> **Answer: B** — 401 means "you haven't proven who you are" (authentication problem). The fix is to provide valid credentials. 403 (Forbidden) would mean you're authenticated but lack permission, which is when you'd assign an RBAC role (Answer C).

---

**Question 6 (Section 6 — Postman and curl)**
What does the curl flag `-d` do?

**A)** Sets the HTTP method to DELETE
**B)** Downloads the response to a file
**C)** Sends data in the request body
**D)** Enables debug mode

> **Answer: C** — `-d` stands for "data" and sends the provided content as the request body. To set the method to DELETE, use `-X DELETE`. To download to a file, use `-o filename`. For verbose/debug output, use `-v`.

---

**Question 7 (Section 7 — Git)**
You accidentally committed `local.settings.json` to Git and pushed it to GitHub. The file contains your Azure Storage connection string. What should you do FIRST?

**A)** Delete the file from the repository and push again
**B)** Add `local.settings.json` to `.gitignore` and push again
**C)** Rotate (change) the compromised connection string immediately
**D)** Make the GitHub repository private

> **Answer: C** — The secret is already in Git's history. Deleting the file or adding it to `.gitignore` removes it from future commits but not from history. Making the repo private doesn't help if the secret was already exposed. The FIRST action is to rotate the compromised credential so the old one no longer works.

---

**Question 8 (Section 8 — .NET)**
A .NET console project has a method `client.UploadBlobAsync("data")`. What is the correct way to call this method?

**A)** `client.UploadBlobAsync("data");`
**B)** `await client.UploadBlobAsync("data");`
**C)** `async client.UploadBlobAsync("data");`
**D)** `Task client.UploadBlobAsync("data");`

> **Answer: B** — Methods ending in `Async` must be called with `await`. This tells the program to pause and wait for the operation to complete before continuing. Without `await`, the upload would start but the program wouldn't wait for it to finish, causing potential bugs.

---

# What We Covered ✅

Use this checklist to verify you've understood every section:

- [ ] **Section 1 — Terminal:** I can open a terminal, navigate folders with `cd`, create folders with `mkdir`, and I know that silence means success
- [ ] **Section 2 — Azure CLI:** I can run `az login`, I understand the `az [service] [action] --parameter value` pattern, and I can create/list/delete resource groups
- [ ] **Section 3 — JSON:** I can identify objects `{}`, arrays `[]`, strings, numbers, booleans, and null in JSON. I can use `--output table` and `--query` with Azure CLI
- [ ] **Section 4 — Environment Variables:** I know why secrets should never be hard-coded, how to set environment variables on my OS, and how Azure App Settings become environment variables
- [ ] **Section 5 — HTTP and REST APIs:** I know all 5 HTTP methods, can read status codes (especially 200, 201, 401, 403, 404, 500), and understand that every Azure SDK call is HTTP under the hood
- [ ] **Section 6 — Postman and curl:** I can send a GET request in both Postman and curl, and I know the `-X`, `-H`, and `-d` curl flags
- [ ] **Section 7 — Git:** I can clone a repository, make changes, commit with a message, push to GitHub, and I NEVER commit secrets to Git
- [ ] **Section 8 — .NET:** I can create a project with `dotnet new console`, add packages with `dotnet add package`, build with `dotnet build`, run with `dotnet run`, and I understand `async`/`await`

---

🔗 **MODULE LINK**
> **Where to go next:**
> - **F04 (Azure Portal Navigation)** — Learn to navigate the Azure Portal for all lab exercises
> - **F05 (Lab Zero)** — Put everything together: create your first complete Azure resource via CLI
> - **F06 (C# Basics)** — Deep dive into reading and understanding C# code for AZ-204 labs
> - **Module 1 (App Service)** — Your first AZ-204 module, where you'll use the terminal, CLI, JSON, Git, and .NET from this document

---

*End of Document — F03: Developer Basics: Terminal, Code, Git, and APIs*
