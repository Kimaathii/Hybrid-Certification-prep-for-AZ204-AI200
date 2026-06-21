# Module 7 Lab: Implementing Azure Managed Redis

| Category | Details |
|----------|---------|
| **Module** | 7 - Azure Managed Redis |
| **Lab Title** | Implement Caching and Session State |
| **Prerequisites** | Azure CLI installed, .NET 8 SDK installed |
| **Estimated Time** | 45 Minutes |
| **Cost Estimate** | 💰 Basic Tier C0: ~$0.05 if cleaned up immediately |
| **What You Build** | A .NET Console App implementing the Cache-Aside pattern |
| **What You Learn** | Provisioning Redis, connecting via StackExchange.Redis, setting TTLs, Cache-Aside logic |

💰 **COST CALLOUT**
This lab uses the **Azure Cache for Redis (Basic C0 tier)**. Estimated cost is under $0.10 if cleaned up within 2 hours. Do NOT select Premium or Enterprise unless your employer is paying!

---

## Part 1: Provision Azure Cache for Redis

First, we will create our resource group and the Redis Cache instance using the Azure CLI.

```bash
# 1. Login to Azure
az login

# 2. Create a Resource Group
az group create \
  --name "rg-redis-lab" \
  --location "eastus"
```

Now, create the Redis Cache. *Note: Provisioning Redis can take 15-20 minutes! Take a coffee break after running this command.*

```bash
# 3. Create the Redis Cache
# --name must be globally unique (change 'mylabre' to include your initials)
# --sku Basic <-- The cheapest tier, no SLA, single node.
# --vm-size C0 <-- The smallest size available.
az redis create \
  --resource-group "rg-redis-lab" \
  --name "redis-lab-yourinitials" \
  --location "eastus" \
  --sku Basic \
  --vm-size C0
```

**Expected Output:** (After 15 minutes) A large JSON block showing `"provisioningState": "Succeeded"`.

While waiting, let's grab the Connection String. We will need this for our C# code.

```bash
# 4. Get the Primary Access Key
az redis list-keys \
  --resource-group "rg-redis-lab" \
  --name "redis-lab-yourinitials"
```

Construct your connection string like this:
`your-redis-name.redis.cache.windows.net:6380,password=YOUR_PRIMARY_KEY,ssl=True,abortConnect=False`

---

✅ **CHECKPOINT**
Why did we use `--sku Basic` instead of `--sku Premium`?
*(Answer: Basic is cheap and fine for lab environments. Premium is for production scenarios requiring VNet integration or data persistence.)*

---

## Part 2: Create the .NET Application

```bash
# 1. Create a new console application
dotnet new console -n RedisLabApp
cd RedisLabApp

# 2. Install the standard Redis SDK for .NET
dotnet add package StackExchange.Redis
```

Open `Program.cs` in your code editor. We will set up the connection.

```csharp
using System;
using System.Threading.Tasks;
using StackExchange.Redis;

class Program
{
    // Replace with your actual connection string from Part 1
    private static string connectionString = "redis-lab-yourinitials.redis.cache.windows.net:6380,password=YOUR_KEY,ssl=True,abortConnect=False";

    static async Task Main(string[] args)
    {
        Console.WriteLine("Connecting to Redis...");
        
        // ConnectionMultiplexer manages the network connection
        using var cache = ConnectionMultiplexer.Connect(connectionString);
        IDatabase db = cache.GetDatabase();
        
        Console.WriteLine("Connected!");
        
        // Test a simple ping
        var ping = await db.PingAsync();
        Console.WriteLine($"Ping response time: {ping.TotalMilliseconds} ms");
    }
}
```

Run the app using `dotnet run`. You should see a successful connection and a very fast ping response time.

---

## Part 3: Implement the Cache-Aside Pattern

Now we will write a function that simulates querying a slow database (like a 3-second delay), but checks Redis first.

Replace the contents of `Program.cs` with the following:

```csharp
using System;
using System.Threading.Tasks;
using StackExchange.Redis;

class Program
{
    private static string connectionString = "YOUR_CONNECTION_STRING";
    private static IDatabase db;

    static async Task Main(string[] args)
    {
        using var cache = ConnectionMultiplexer.Connect(connectionString);
        db = cache.GetDatabase();

        Console.WriteLine("--- First Request (Should be slow) ---");
        string result1 = await GetProductPriceAsync("Croissant");
        Console.WriteLine($"Result: {result1}\n");

        Console.WriteLine("--- Second Request (Should be instant) ---");
        string result2 = await GetProductPriceAsync("Croissant");
        Console.WriteLine($"Result: {result2}\n");
    }

    // The Cache-Aside implementation
    static async Task<string> GetProductPriceAsync(string productName)
    {
        string cacheKey = $"product_price:{productName}";

        // Step 1: Check Redis
        string cachedPrice = await db.StringGetAsync(cacheKey);

        if (!string.IsNullOrEmpty(cachedPrice))
        {
            Console.WriteLine("[CACHE HIT] Found price on the whiteboard!");
            return cachedPrice;
        }

        Console.WriteLine("[CACHE MISS] Not found. Walking to the filing cabinet...");
        
        // Step 2: Simulate slow database query
        await Task.Delay(3000); 
        string dbPrice = "$4.50"; 

        // Step 3: Write to Redis for the next time
        // We set a Time-To-Live (TTL) of 15 seconds!
        await db.StringSetAsync(cacheKey, dbPrice, TimeSpan.FromSeconds(15));
        Console.WriteLine("Wrote price to whiteboard with 15-second TTL.");

        return dbPrice;
    }
}
```

Run the app `dotnet run`.
**Expected Output:**
```text
--- First Request (Should be slow) ---
[CACHE MISS] Not found. Walking to the filing cabinet...
Wrote price to whiteboard with 15-second TTL.
Result: $4.50

--- Second Request (Should be instant) ---
[CACHE HIT] Found price on the whiteboard!
Result: $4.50
```

✅ **CHECKPOINT**
What would happen if we didn't include `TimeSpan.FromSeconds(15)`?
*(Answer: The data would stay in RAM forever until the cache filled up and an eviction policy was triggered. Setting a TTL prevents memory leaks.)*

---

## Part 4: Theory - Redis for AI Vector Search

*Note: You cannot execute this part of the lab because Vector Search requires the Enterprise Tier, which is too expensive for lab purposes. This is purely for exam context.*

To use Redis as a Vector Database (for semantic AI search), you must:
1. Provision an **Enterprise Tier** instance with the **RediSearch** module enabled.
2. Install the `NRedisStack` package instead of standard `StackExchange.Redis`.

**Code Snippet (Do Not Run):**
```csharp
// 1. You must create an Index to tell Redis how to search vectors
var schema = new Schema()
    .AddTextField("title")
    .AddVectorField("embedding", VectorField.VectorAlgo.HNSW, 
        new Dictionary<string, object>() {
            {"TYPE", "FLOAT32"},
            {"DIM", 1536}, // Standard size for OpenAI vectors
            {"DISTANCE_METRIC", "COSINE"}
        });
db.FT().Create("movie_index", new FTCreateParams().On(IndexDataType.HASH), schema);

// 2. Performing a Semantic Search (K-Nearest Neighbors)
var query = new Query("*=>[KNN 3 @embedding $vector AS score]")
    .AddParam("vector", myOpenAiVectorBytes)
    .Dialect(2);

var results = db.FT().Search("movie_index", query);
```

---

## Troubleshooting Top Errors

1. **Error:** `The message timed out...` or `Timeout performing GET`
   * **Fix:** Redis might still be provisioning (it takes 15+ mins). Wait longer. Also check your local firewall isn't blocking outbound port 6380.
2. **Error:** `No connection is available to service this operation`
   * **Fix:** Your connection string is wrong. Ensure it includes `,password=...` and `,ssl=True`.
3. **Error:** `Unknown command 'FT.SEARCH'`
   * **Fix:** You are trying to run a Vector Search query on a Basic/Standard tier cache. You must use Enterprise tier.

---

## Part 5: Clean Up

It is critical to delete your resource group to stop billing.

```bash
# Delete the entire resource group and the Redis instance
az group delete \
  --name "rg-redis-lab" \
  --yes \
  --no-wait

# Verify deletion
az group list -o table
```
