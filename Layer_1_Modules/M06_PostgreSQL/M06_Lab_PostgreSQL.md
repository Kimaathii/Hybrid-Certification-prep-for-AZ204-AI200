# Lab Guide: Building a Hybrid Web + AI Database with PostgreSQL and pgvector

| Category | Details |
| :--- | :--- |
| **Module** | 6: Azure Database for PostgreSQL & pgvector |
| **Lab Title** | Hybrid Relational & AI Vector Database Implementation |
| **Prerequisites** | Azure CLI installed, psql command-line tool installed, active Azure subscription |
| **Estimated Time** | 45 minutes |
| **Cost Estimate** | ~$0.50 if cleaned up immediately (Uses Burstable B1ms tier) |
| **What You Build** | A cloud-hosted PostgreSQL database, secured with firewalls, enhanced with AI vector search capabilities. |
| **What You Learn** | Provisioning Flexible Server, managing IP firewalls, enabling extensions, storing vectors, executing cosine similarity queries. |

💰 **COST CALLOUT:** This lab uses Azure Database for PostgreSQL - Flexible Server on the Burstable (B1ms) tier. Estimated cost: $0.50 if cleaned up within 2 hours.

---

## Step 1: Create a PostgreSQL Flexible Server

First, we will provision the managed database server in Azure.

1. Open your terminal or Azure Cloud Shell.
2. Define standard variables to make our commands easier to read:

```bash
RESOURCE_GROUP="rg-az204-postgres-lab"
LOCATION="eastus"
SERVER_NAME="psql-hybrid-ai-$RANDOM"
ADMIN_USER="dbadmin"
ADMIN_PASS="P@ssw0rd1234!" # Note: Use a strong password in real scenarios
```

3. Create the resource group:

```bash
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION
```

4. Create the PostgreSQL Flexible Server:

```bash
az postgres flexible-server create \
  --resource-group $RESOURCE_GROUP \        # The group we just created
  --name $SERVER_NAME \                     # The globally unique server name
  --location $LOCATION \                    # Deployment region
  --admin-user $ADMIN_USER \                # Master administrator username
  --admin-password $ADMIN_PASS \            # Master administrator password
  --sku-name Standard_B1ms \                # Compute tier: Burstable 1 core (cheapest)
  --tier Burstable \                        # Explicitly set compute tier category
  --storage-size 32 \                       # Storage disk size in GB (32 is minimum)
  --version 14                              # PostgreSQL major version
```

**Expected Output:**
```json
{
  "connectionString": "postgresql://dbadmin:P@ssw0rd1234!@psql-hybrid-ai-12345.postgres.database.azure.com/postgres?sslmode=require",
  "host": "psql-hybrid-ai-12345.postgres.database.azure.com",
  "id": "/subscriptions/.../servers/psql-hybrid-ai-12345",
  "location": "East US",
  "skuname": "Standard_B1ms",
  "version": "14"
}
```

---

## Step 2: Configure the IP Firewall

By default, the server blocks all external connections. We must allow your specific IP address.

1. Find your current public IP address (you can also just google "What is my IP"):

```bash
# This command fetches your external IP address
curl ifconfig.me
```

2. Add your IP address to the firewall rules (replace `<YOUR_IP>` with the output from the previous step):

```bash
az postgres flexible-server firewall-rule create \
  --resource-group $RESOURCE_GROUP \        # Resource group containing the server
  --name $SERVER_NAME \                     # Name of your Postgres server
  --rule-name AllowMyIP \                   # A friendly name for this rule
  --start-ip-address <YOUR_IP> \            # The beginning of the allowed IP range
  --end-ip-address <YOUR_IP>                # The end of the range (same as start for a single IP)
```

✅ **CHECKPOINT 1**
Why did we have to explicitly add our IP address? 
*Answer: Azure Database for PostgreSQL Flexible Server is secure by default and blocks all public internet traffic until an explicit IP firewall rule is created.*

---

## Step 3: Connect to the Database

Now that the firewall allows our connection, let's connect using the `psql` command-line tool.

1. Retrieve the exact hostname of your server from Step 1's expected output (e.g., `psql-hybrid-ai-12345.postgres.database.azure.com`).
2. Run the `psql` connection command:

```bash
psql -h <YOUR_SERVER_HOSTNAME> \
     -U dbadmin \
     -d postgres
```
*(When prompted, enter your password: `P@ssw0rd1234!`)*

**Expected Output:**
```text
Password for user dbadmin:
psql (14.x)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=>
```

---

## Step 4: Enable the pgvector Extension

By default, PostgreSQL is a traditional relational database. We must enable the AI extension.

1. Inside the `postgres=>` prompt, run:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

2. Verify the extension is installed:

```sql
\dx
```

**Expected Output:**
```text
                                     List of installed extensions
  Name   | Version |   Schema   |                              Description                               
---------+---------+------------+------------------------------------------------------------------------
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
 vector  | 0.5.1   | public     | vector data type and ivfflat and hnsw access methods
```

---

## Step 5: Create a Hybrid Table

We will create a table that holds both standard relational data and AI vector data. For this lab, we will use a small dimension size (`vector(3)`) so we can type the embeddings manually, rather than 1536.

```sql
CREATE TABLE products (
    id serial PRIMARY KEY,
    product_name varchar(100),
    category varchar(50),
    description_embedding vector(3)  -- An AI vector with 3 dimensions
);
```

✅ **CHECKPOINT 2**
If you were using OpenAI's `text-embedding-ada-002` model in a production app, what would you change in the table definition above?
*Answer: You would change `vector(3)` to `vector(1536)` because that specific model outputs exactly 1536 dimensions.*

---

## Step 6: Insert Mock Data

Let's insert some sample products. Imagine an AI model has already processed the product descriptions and generated these 3-number vectors representing their "meaning".

```sql
INSERT INTO products (product_name, category, description_embedding) VALUES 
('Cozy Wool Sweater', 'Apparel', '[0.9, 0.1, 0.1]'),
('Running Shoes', 'Footwear', '[0.1, 0.8, 0.2]'),
('Winter Coat', 'Apparel', '[0.8, 0.2, 0.1]'),
('Coffee Mug', 'Kitchen', '[0.0, 0.1, 0.9]');
```

---

## Step 7: Run a Cosine Similarity Search

Imagine a user searches for "Warm clothing for winter". Our backend application converts that search text into a vector using an AI API. Let's say the resulting search vector is `[0.85, 0.15, 0.1]`. 

Let's find the closest matches in our database using **Cosine Similarity (`<=>`)**.

```sql
SELECT product_name, category, 
       (description_embedding <=> '[0.85, 0.15, 0.1]') AS distance
FROM products
ORDER BY description_embedding <=> '[0.85, 0.15, 0.1]'
LIMIT 2;
```

**Expected Output:**
```text
   product_name    | category |        distance         
-------------------+----------+-------------------------
 Winter Coat       | Apparel  |  0.00124...
 Cozy Wool Sweater | Apparel  |  0.00845...
(2 rows)
```
*Notice how the query mathematically determined that the Winter Coat and Wool Sweater were semantically closest to the user's search query, ignoring the running shoes and coffee mug.*

---

## TROUBLESHOOTING

1. **Error:** `psql: error: connection to server... failed: FATAL: no pg_hba.conf entry for host`
   *Fix:* Your IP address changed, or the firewall rule wasn't applied correctly. Rerun Step 2 using your exact current public IP.
2. **Error:** `type "vector" does not exist`
   *Fix:* You forgot to run `CREATE EXTENSION vector;` in Step 4. Run it and try creating the table again.
3. **Error:** `operator does not exist: vector <=> unknown`
   *Fix:* Ensure you format the vector strictly as a string containing an array in brackets: `'[0.1, 0.2, 0.3]'`.
4. **Error:** `az postgres flexible-server create fails with ResourceNotFound`
   *Fix:* Ensure the resource group was created successfully in Step 1 and the location name is spelled correctly (e.g., `eastus`).
5. **Error:** `could not connect to server: Connection timed out`
   *Fix:* Corporate firewalls often block port 5432. Try connecting from a personal network or using Azure Cloud Shell directly in the browser.

---

## Step 8: CLEANUP

You MUST clean up these resources immediately to prevent ongoing charges to your Azure account.

1. Exit the psql prompt:
```sql
\q
```

2. Delete the entire resource group (which deletes the server and firewall rules):

```bash
az group delete \
  --name $RESOURCE_GROUP \
  --yes \
  --no-wait
```

3. Confirm deletion is in progress:

```bash
az group list -o table
```
*Verify that `rg-az204-postgres-lab` says "Deleting" or is completely gone from the list.*
