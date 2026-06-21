# Lab Guide: Cosmos DB Web + AI Integration

| **Module** | Module 5: Cosmos DB for NoSQL |
| :--- | :--- |
| **Lab Title** | Building a Hybrid Web + AI User Profile Store |
| **Prerequisites** | Azure CLI installed, .NET 8 SDK, an active Azure Subscription |
| **Estimated Time** | 45 minutes |
| **Cost Estimate** | 💰 **COST WARNING:** This lab uses Cosmos DB Serverless tier. Estimated cost: < $0.10 if cleaned up within 24 hours. |
| **What you build** | A globally distributed NoSQL database and a .NET console app that performs traditional point-writes and AI vector similarity searches. |
| **What you learn** | Creating Serverless Cosmos DB, provisioning containers, inserting JSON profiles, inserting AI vectors, querying with SQL, and performing vector distance search. |

---

## Part 1: Create a Cosmos DB Account

First, we will create a resource group and a Cosmos DB account using the Azure CLI. To save costs, we will use the Serverless capability.

1. Open your terminal and log in to Azure:
```bash
az login
```

2. Create a resource group:
```bash
az group create \
  --name rg-cosmos-lab \     # The name of our resource group
  --location eastus          # The Azure region to deploy into
```

3. Create the Cosmos DB NoSQL Account. 
*(Note: Cosmos DB account names must be globally unique. Replace `yourname` with a unique string).*
```bash
az cosmosdb create \
  --name cosmos-lab-yourname \          # Globally unique account name
  --resource-group rg-cosmos-lab \      # The resource group we just created
  --locations regionName=eastus \       # Primary region for the database
  --capabilities EnableServerless \     # Sets billing to Serverless (pay-per-request)
  --default-consistency-level Session   # Default consistency (guarantees read-your-own-writes)
```
*Expected Output: A massive JSON response detailing the account creation. This command may take 5-10 minutes to run.*

✅ **CHECKPOINT**
Why did we use the `EnableServerless` capability instead of defining RU/s?
*Answer:* Serverless is cheaper for labs and unpredictable workloads because you only pay for the exact queries you run, rather than paying an hourly rate for provisioned capacity.

---

## Part 2: Create a Database and Container

1. Create a database named `AppDB` inside your account:
```bash
az cosmosdb sql database create \
  --account-name cosmos-lab-yourname \  # Your globally unique account name
  --resource-group rg-cosmos-lab \      # Your resource group
  --name AppDB                          # The name of the logical database
```

2. Create a container named `Users` with a partition key of `/country`.
```bash
az cosmosdb sql container create \
  --account-name cosmos-lab-yourname \  # Your globally unique account name
  --resource-group rg-cosmos-lab \      # Your resource group
  --database-name AppDB \               # The database we just created
  --name Users \                        # The name of the container
  --partition-key-path "/country"       # The JSON property used to distribute data across physical servers
```

---

## Part 3: Setup the .NET Console App & Get Credentials

1. Get your Cosmos DB connection string (Primary Key):
```bash
az cosmosdb keys list \
  --name cosmos-lab-yourname \
  --resource-group rg-cosmos-lab \
  --type keys
```
*Copy the `primaryMasterKey` value from the output.*

2. Create a new .NET Console application:
```bash
dotnet new console -n CosmosWebAIApp
cd CosmosWebAIApp
```

3. Install the Azure Cosmos DB SDK:
```bash
dotnet add package Microsoft.Azure.Cosmos
```

✅ **CHECKPOINT**
What is the purpose of the `/country` partition key?
*Answer:* Cosmos DB uses this key to group all users from the same country together onto the same physical servers, allowing the database to scale horizontally.

---

## Part 4: Code the Application (Web + AI Inserts)

Open `Program.cs` in your editor and replace the contents with the following C# code. Replace the `EndpointUri` and `PrimaryKey` with your specific values.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Azure.Cosmos;

class Program
{
    // Replace with your actual Cosmos DB URI and Primary Key
    private static readonly string EndpointUri = "https://cosmos-lab-yourname.documents.azure.com:443/";
    private static readonly string PrimaryKey = "YOUR_PRIMARY_KEY_HERE";

    static async Task Main(string[] args)
    {
        // 1. Initialize the Cosmos Client
        CosmosClient client = new CosmosClient(EndpointUri, PrimaryKey);
        
        // 2. Get database and container references
        Database database = client.GetDatabase("AppDB");
        Container container = database.GetContainer("Users");

        Console.WriteLine("Connected to Cosmos DB...");

        // 3. Create a Web User Profile with an AI Vector Embedding
        // In a real app, an AI model (like OpenAI) would generate the float[] array based on the user's bio.
        var userProfile = new
        {
            id = Guid.NewGuid().ToString(),
            country = "USA", // This is our Partition Key
            name = "Alex Developer",
            bio = "Loves writing C# and building cloud architectures.",
            // A mock vector embedding representing the semantic meaning of the bio
            bioVector = new float[] { 0.82f, -0.11f, 0.45f, 0.99f, -0.32f } 
        };

        // 4. Insert the User Profile (Point Write)
        // We MUST pass the partition key value ("USA") when creating the item.
        await container.CreateItemAsync(userProfile, new PartitionKey(userProfile.country));
        
        Console.WriteLine($"Inserted User: {userProfile.name}");

        // 5. Query the database using standard SQL
        await QueryTraditionalSql(container);

        // 6. Perform a Vector Distance Search
        await QueryVectorSearch(container);
    }

    static async Task QueryTraditionalSql(Container container)
    {
        Console.WriteLine("\n--- Running Traditional SQL Query ---");
        string sqlQueryText = "SELECT * FROM c WHERE c.country = 'USA'";
        
        QueryDefinition queryDefinition = new QueryDefinition(sqlQueryText);
        using FeedIterator<dynamic> queryResultSetIterator = container.GetItemQueryIterator<dynamic>(queryDefinition);

        while (queryResultSetIterator.HasMoreResults)
        {
            FeedResponse<dynamic> currentResultSet = await queryResultSetIterator.ReadNextAsync();
            foreach (dynamic user in currentResultSet)
            {
                Console.WriteLine($"\tFound standard user: {user.name} from {user.country}");
            }
        }
    }

    static async Task QueryVectorSearch(Container container)
    {
        Console.WriteLine("\n--- Running AI Vector Search ---");
        // A mock vector representing a search query for "cloud software engineer"
        float[] searchVector = new float[] { 0.80f, -0.10f, 0.40f, 0.95f, -0.30f };

        // VectorDistance compares the search vector to the bioVector stored in the database
        string sqlQueryText = @"
            SELECT TOP 1 c.name, VectorDistance(c.bioVector, @searchVector) AS similarityScore
            FROM c
            ORDER BY VectorDistance(c.bioVector, @searchVector)";

        QueryDefinition queryDefinition = new QueryDefinition(sqlQueryText)
            .WithParameter("@searchVector", searchVector);

        using FeedIterator<dynamic> queryResultSetIterator = container.GetItemQueryIterator<dynamic>(queryDefinition);

        while (queryResultSetIterator.HasMoreResults)
        {
            FeedResponse<dynamic> currentResultSet = await queryResultSetIterator.ReadNextAsync();
            foreach (dynamic user in currentResultSet)
            {
                Console.WriteLine($"\tAI Match: {user.name} with Similarity Score: {user.similarityScore}");
            }
        }
    }
}
```

Run the application:
```bash
dotnet run
```

*Expected Output:*
```text
Connected to Cosmos DB...
Inserted User: Alex Developer

--- Running Traditional SQL Query ---
        Found standard user: Alex Developer from USA

--- Running AI Vector Search ---
        AI Match: Alex Developer with Similarity Score: 0.985
```

✅ **CHECKPOINT**
In the Vector Search query, what does the `VectorDistance` function do?
*Answer:* It mathematically calculates how similar the user's `bioVector` is to the `searchVector`. A score closer to 1.0 means the text meanings are highly related.

---

## TROUBLESHOOTING

1. **Error: "401 Unauthorized"**
   *Fix:* Double-check that you copied the `primaryMasterKey` correctly and pasted it into your `Program.cs` file. Ensure there are no trailing spaces.
2. **Error: "PartitionKey extracted from document doesn't match the one specified in the header"**
   *Fix:* Ensure that the object you are inserting has `country = "USA"` and you are passing `new PartitionKey("USA")` in the `CreateItemAsync` call.
3. **Error: "Resource with specified id or name already exists"**
   *Fix:* Cosmos DB requires unique IDs. The code uses `Guid.NewGuid().ToString()`, but if you hardcoded the ID to "1" and ran the app twice, it will fail.
4. **Error: "Cannot establish connection to endpoint"**
   *Fix:* Ensure your corporate firewall is not blocking outbound traffic on port 443 (HTTPS) to Azure.
5. **Error: "Cosmos DB account name already exists" during CLI setup**
   *Fix:* The `--name` parameter in `az cosmosdb create` must be globally unique across all of Azure. Change "cosmos-lab-yourname" to something completely random.

---

## CLEANUP

Do not leave Cosmos DB accounts running. Even in Serverless, it's best practice to destroy resources you are no longer using.

1. Delete the resource group:
```bash
az group delete \
  --name rg-cosmos-lab \   # The resource group we created
  --yes \                  # Skips the confirmation prompt
  --no-wait                # Returns you to the prompt immediately while deletion happens in the background
```

2. Confirm deletion has started:
```bash
az group list -o table
```
*If `rg-cosmos-lab` is missing or marked as 'Deleting', your cleanup was successful.*
