# AZ-204: Azure Cosmos DB - Complete Study Notes (Exam-Ready Edition)

## Learning Path Overview
This learning path covers developing solutions that use Azure Cosmos DB with 2 modules:
1. Explore Azure Cosmos DB
2. Work with Azure Cosmos DB

**Duration:** ~1 hour | **Level:** Intermediate | **XP:** ~1400

> **⚠️ Note:** This document includes topics BEYOND the MS Learn career path — covering every concept that appears in the actual AZ-204 exam including advanced SDK options, TTL, backup policies, data modeling, networking, security, emulator, analytical store, integrated cache, ARM/Bicep provisioning, and more.

---

## MODULE 1: EXPLORE AZURE COSMOS DB

### 1.1 Introduction to Azure Cosmos DB

**What is Azure Cosmos DB?**
- Globally distributed, multi-model database service
- Fully managed NoSQL and relational database
- Designed for modern app development
- Guarantees single-digit millisecond response times
- Provides automatic and instant scalability
- Offers 99.999% availability SLA (with multi-region)

**Key Benefits:**
- **Turnkey Global Distribution**: Replicate data across any Azure region with a click
- **Always On**: 99.99% availability for single-region, 99.999% for multi-region
- **Elastic Scalability**: Scale throughput and storage independently and elastically
- **Low Latency**: <10ms reads and <10ms writes at 99th percentile
- **Multiple Data Models**: Support for document, key-value, graph, and column-family
- **Multiple APIs**: SQL, MongoDB, Cassandra, Gremlin, and Table APIs

**Use Cases:**
- IoT and telematics
- Retail and marketing
- Gaming applications
- Web and mobile applications
- Real-time personalization
- Social media applications
- Global content distribution

### 1.2 Azure Cosmos DB Resource Hierarchy

**Resource Hierarchy:**
```
Azure Cosmos DB Account
    └── Database
            └── Container
                    └── Items (Documents, Rows, Nodes, Edges)
```

#### **Azure Cosmos DB Account**
- Top-level resource
- Unit of global distribution and high availability
- Unique DNS name: `<account-name>.documents.azure.com`
- Can have multiple databases
- Choose API type at account creation (cannot change later)
- Configure default consistency level
- Maximum 50 accounts per Azure subscription (can be increased via support)

#### **Database**
- Namespace for containers
- Unit of management for containers
- Analogous to a database in relational systems
- Can provision throughput at database level (shared throughput)
- Unlimited databases per account

#### **Container**
- Unit of scalability for throughput and storage
- Horizontally partitioned and replicated across regions
- Items added are automatically distributed across partitions
- Throughput provisioned at container or database level
- Unlimited storage (with proper partitioning)

**Container Equivalents Across APIs:**

| API | Container Name |
|-----|---------------|
| SQL (Core) | Container |
| MongoDB | Collection |
| Cassandra | Table |
| Gremlin | Graph |
| Table | Table |

#### **Items**
- Content stored within containers
- Automatically indexed by default
- Schema-agnostic (no fixed schema required)
- **Maximum item size: 2 MB** (for SQL API)

**Item Equivalents Across APIs:**

| API | Item Name |
|-----|-----------|
| SQL (Core) | Item (Document) |
| MongoDB | Document |
| Cassandra | Row |
| Gremlin | Node or Edge |
| Table | Item (Entity) |

### 1.2b System Properties (Reserved Properties)

Every item in Cosmos DB has **system-generated properties** that are read-only (except `id`):

| Property | Description | Exam Notes |
|----------|-------------|------------|
| `id` | Unique identifier within a logical partition (user-provided) | Required, max 255 characters, case-sensitive |
| `_rid` | System-generated unique resource identifier | Internal use, hierarchical, used in resource URI |
| `_self` | Addressable URI of the resource | Used for REST API direct access |
| `_etag` | Entity tag for optimistic concurrency control | Changes on every update, used with `If-Match` header |
| `_ts` | Last modified timestamp (Unix epoch seconds) | Auto-updated, used by default in Last Writer Wins conflict resolution |
| `_attachments` | Addressable path for attachments resource | Legacy feature, deprecated |

```json
{
    "id": "product-001",
    "name": "Laptop",
    "price": 999.99,
    "_rid": "qHVdAImeNAQBAAAAAAAAAA==",
    "_self": "dbs/qHVdAA==/colls/qHVdAImeNAQ=/docs/qHVdAImeNAQBAAAAAAAAAA==/",
    "_etag": "\"00000000-0000-0000-abcd-1234567890ab\"",
    "_ts": 1706745600,
    "_attachments": "attachments/"
}
```

**Key Exam Points:**
- `id` + partition key = unique identifier for an item
- `id` is user-defined; all others are system-generated
- `_etag` is critical for optimistic concurrency (HTTP 412 on mismatch)
- `_ts` is used as the default conflict resolution path in LWW
- **Partition key value maximum size: 2 KB**

### 1.3 Azure Cosmos DB APIs

**Overview:**
- Choose API when creating account
- API choice is permanent (cannot change after creation)
- Each API optimized for specific data models and query patterns

#### **1. API for NoSQL (SQL API / Core API)**
- **Default and most commonly used API**
- Native Cosmos DB API
- Query using SQL-like syntax
- Stores data as JSON documents
- Best for new projects without existing database
- Full support for all Cosmos DB features first
- Best performance and feature support

```sql
-- Example SQL Query
SELECT c.id, c.name, c.email
FROM customers c
WHERE c.city = "Seattle"
```

**Use Cases:**
- New applications without legacy requirements
- Applications requiring complex queries
- Maximum flexibility and feature support

#### **2. API for MongoDB**
- Wire protocol compatible with MongoDB
- Use existing MongoDB drivers, tools, and expertise
- Stores data as BSON documents
- Migration path for MongoDB applications

**Use Cases:**
- Existing MongoDB applications
- Teams with MongoDB experience
- Applications using MongoDB ecosystem tools

#### **3. API for Apache Cassandra**
- Wire protocol compatible with Cassandra
- CQL (Cassandra Query Language) support
- Wide-column store data model
- Existing Cassandra drivers work

**Use Cases:**
- Existing Cassandra applications
- Time-series data
- IoT applications
- High-write throughput scenarios

#### **4. API for Apache Gremlin**
- Graph database API
- Gremlin query language
- Stores nodes (vertices) and relationships (edges)
- Property graph model

**Use Cases:**
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs

#### **5. API for Table**
- Key-value store
- Compatible with Azure Table Storage
- Migration path from Azure Table Storage
- Better performance than Table Storage

**Use Cases:**
- Applications using Azure Table Storage
- Simple key-value access patterns
- Cost-effective storage for structured data

**API Selection Guidelines:**

| Requirement | Recommended API |
|-------------|----------------|
| New application, no constraints | API for NoSQL |
| Existing MongoDB code | API for MongoDB |
| Existing Cassandra code | API for Cassandra |
| Graph relationships | API for Gremlin |
| Azure Table Storage migration | API for Table |
| Complex queries | API for NoSQL |
| Maximum features | API for NoSQL |

### 1.4 Request Units (RU/s)

**What are Request Units?**
- Currency for Cosmos DB operations
- Abstracts CPU, IOPS, and memory resources
- 1 RU = cost of reading a single 1 KB item by ID and partition key
- All operations measured in RUs
- Predictable performance model

**RU Consumption Factors:**

| Factor | Impact on RU Cost |
|--------|------------------|
| Item size | Larger items = more RUs |
| Item indexing | More indexed properties = more RUs |
| Item property count | More properties = more RUs |
| Query complexity | Complex queries = more RUs |
| Script/procedure execution | Logic complexity affects RUs |
| Consistency level | Stronger consistency = more RUs |

**Approximate RU Costs:**

| Operation | Approximate RU Cost |
|-----------|-------------------|
| Point read (1 KB item, by ID + partition key) | 1 RU |
| Write (1 KB item) | ~5-10 RUs |
| Query (simple, returning 1 KB) | ~2.5+ RUs |
| Query (complex, cross-partition) | Significantly higher |

**Throughput Provisioning Models:**

#### **1. Provisioned Throughput**
- Pre-allocate RU/s capacity
- Pay per hour for provisioned capacity
- Guaranteed performance
- Two modes:
  - **Standard (Manual)**: Fixed RU/s, manual scaling
  - **Autoscale**: Automatic scaling between min and max

**Standard Provisioned:**
```bash
# Minimum: 400 RU/s per container
# Increments: 100 RU/s
# Example: 400, 500, 600, ... RU/s
```

**Autoscale:**
```bash
# Set maximum RU/s
# Automatically scales between 10% and 100% of max
# Example: Max 10,000 RU/s → scales from 1,000 to 10,000 RU/s
# Billed for highest RU/s used per hour
```

#### **2. Serverless**
- Pay per RU consumed
- No capacity provisioning
- Good for development and sporadic workloads
- Maximum 5,000 RU/s burst capacity
- Single-region only
- No SLA guarantee

**Throughput Provisioning Comparison:**

| Aspect | Provisioned (Standard) | Provisioned (Autoscale) | Serverless |
|--------|----------------------|------------------------|------------|
| Billing | Per hour (provisioned) | Per hour (peak used) | Per RU consumed |
| Scaling | Manual | Automatic | Automatic |
| Minimum | 400 RU/s | 10% of max | None |
| Best for | Predictable workloads | Variable workloads | Dev/test, sporadic |
| Multi-region | Yes | Yes | No |
| SLA | Yes | Yes | No |

#### **3. Shared Throughput (Database-level)**
- Provision throughput at database level
- Shared among containers in database
- Minimum 400 RU/s for database
- Good for multiple containers with variable usage
- Cannot mix with container-level provisioning for same containers

```csharp
// Create database with shared throughput
DatabaseResponse database = await client.CreateDatabaseIfNotExistsAsync(
    id: "myDatabase",
    throughput: 1000);  // 1000 RU/s shared across containers
```

### 1.5 Partitioning in Cosmos DB

**Why Partitioning?**
- Enables horizontal scaling
- Distributes data and throughput
- Automatically managed by Cosmos DB
- Critical for performance and cost

**Partition Types:**

#### **Logical Partition**
- Set of items with same partition key value
- Maximum size: 20 GB
- All items with same partition key in same logical partition
- Unit of scope for transactions (stored procedures)

#### **Physical Partition**
- Internal storage and compute unit
- Can contain multiple logical partitions
- Managed automatically by Cosmos DB
- Each has up to 50 GB storage and 10,000 RU/s throughput
- Cosmos DB splits and merges as needed

**Partition Key:**
- Property path in your documents
- Immutable (cannot change after container creation)
- Should exist in every document
- Determines how data is distributed

**Choosing a Good Partition Key:**

**Characteristics of a Good Partition Key:**
1. **High Cardinality**: Many distinct values
2. **Even Distribution**: Data spread evenly across partitions
3. **Common in Queries**: Used in WHERE clause filters
4. **Avoids Hot Partitions**: No single partition gets all traffic

**Good Partition Key Examples:**

| Scenario | Good Partition Key | Reason |
|----------|-------------------|--------|
| E-commerce | `/userId` or `/customerId` | High cardinality, queries filter by user |
| IoT | `/deviceId` | Each device generates data, queries by device |
| Multi-tenant | `/tenantId` | Isolates tenant data, queries within tenant |
| Logs | `/date` (combined with ID) | Prevents hot partition on current date alone |

**Bad Partition Key Examples:**

| Partition Key | Problem |
|--------------|---------|
| `/country` | Low cardinality (few values), hot partitions |
| `/status` | Very low cardinality (e.g., "active", "inactive") |
| `/timestamp` | All writes go to current time partition (hot partition) |
| Static value | All data in single partition |

**Synthetic Partition Keys:**
- Combine multiple properties for better distribution
- Useful when no single property is suitable

```csharp
// Create synthetic partition key
var item = new {
    id = Guid.NewGuid().ToString(),
    userId = "user123",
    orderId = "order456",
    partitionKey = "user123-order456"  // Synthetic key
};
```

**Hierarchical Partition Keys (Preview):**
- Up to 3 levels of partition keys
- Better for multi-tenant scenarios
- Example: `/tenantId`, `/userId`, `/sessionId`

**Cross-Partition Queries:**
- Queries without partition key filter
- Fan out to all partitions
- More expensive (higher RU cost)
- Avoid when possible

```sql
-- Single partition query (efficient)
SELECT * FROM c WHERE c.userId = "user123"

-- Cross-partition query (expensive)
SELECT * FROM c WHERE c.status = "active"
```

### 1.6 Consistency Levels

**Overview:**
- Trade-off between consistency, availability, latency, and throughput
- Five consistency levels to choose from
- Can be overridden at request level
- Weaker consistency = lower latency, higher availability
- Stronger consistency = higher latency, more RU cost

**Consistency Spectrum:**
```
Strong ←→ Bounded Staleness ←→ Session ←→ Consistent Prefix ←→ Eventual
(Strongest)                                                      (Weakest)
```

#### **1. Strong Consistency**
- **Linearizable reads**: Always returns most recent committed write
- Guaranteed to read latest data globally
- Highest consistency guarantee
- Higher latency (waits for replication)
- Not available for multi-region writes
- Costs 2x RU for reads

**Guarantees:**
- Reads guaranteed to return most recent committed version
- Write is only visible after committed to majority of replicas
- Reader never sees uncommitted or partial write

**Use Cases:**
- Financial applications requiring absolute accuracy
- Inventory management
- Scenarios where stale data is unacceptable

#### **2. Bounded Staleness Consistency**
- Reads may lag behind writes by:
  - **K** versions (updates), OR
  - **T** time interval
- Outside staleness window: Strong consistency
- Within window: Consistent prefix (ordered)

**Configuration:**
- Minimum: 10 seconds or 10 operations
- For single-region: K ≥ 10, T ≥ 5 seconds
- For multi-region: K ≥ 100,000, T ≥ 300 seconds

**Use Cases:**
- Applications tolerating bounded staleness
- Stock/inventory with slight delay acceptable
- Gaming leaderboards with acceptable lag

#### **3. Session Consistency (Default)**
- **Most popular choice**
- Consistent within a single client session
- Session reads its own writes (read-your-writes)
- Other sessions may see older data
- Monotonic reads within session
- Honors write order within session

**Guarantees (within session):**
- Read your own writes
- Monotonic reads
- Monotonic writes
- Consistent prefix

**Use Cases:**
- User profile management
- Shopping cart
- Most web/mobile applications
- Social media feeds

#### **4. Consistent Prefix Consistency**
- Reads never see out-of-order writes
- If writes are A, B, C → reads see A, A-B, or A-B-C (never B-A)
- No staleness guarantee (may be arbitrarily behind)
- Guarantees order is preserved

**Use Cases:**
- Social media updates
- Activity feeds
- Comment threads

#### **5. Eventual Consistency**
- Weakest consistency
- No ordering guarantees
- Reads may be out of order
- Eventually converges to final state
- Lowest latency, highest availability
- Lowest RU cost

**Use Cases:**
- Non-critical updates
- Likes/views counters
- Recommendations
- Analytics data

**Consistency Level Comparison:**

| Level | Read Consistency | Availability | Latency | RU Cost |
|-------|-----------------|--------------|---------|---------|
| Strong | Latest write | Lower | Higher | 2x |
| Bounded Staleness | Within bounds | High | Medium | 1x |
| Session | Own writes | High | Low | 1x |
| Consistent Prefix | In order | High | Low | 1x |
| Eventual | May be stale | Highest | Lowest | 1x |

**Setting Consistency Level:**

**Account-level (Default):**
```bash
# Azure CLI
az cosmosdb update --name <account> --resource-group <rg> \
  --default-consistency-level Session
```

**Request-level Override:**
```csharp
// Override at request level (can only weaken, not strengthen)
ItemRequestOptions options = new ItemRequestOptions
{
    ConsistencyLevel = ConsistencyLevel.Eventual
};

ItemResponse<MyItem> response = await container.ReadItemAsync<MyItem>(
    id: "item1",
    partitionKey: new PartitionKey("pk1"),
    requestOptions: options);
```

### 1.7 Global Distribution

**Multi-Region Writes:**
- Write to any region
- Automatic conflict resolution
- Lower write latency (write to nearest region)
- Higher availability

**Single-Region Writes (Multi-Region Reads):**
- One region accepts writes
- All regions can serve reads
- Automatic failover for reads
- Manual failover for writes

**Configuring Global Distribution:**

```bash
# Add region
az cosmosdb update --name <account> --resource-group <rg> \
  --locations regionName=eastus failoverPriority=0 \
  --locations regionName=westus failoverPriority=1

# Enable multi-region writes
az cosmosdb update --name <account> --resource-group <rg> \
  --enable-multiple-write-locations true
```

**Automatic Failover:**
- Cosmos DB automatically fails over in case of region outage
- Priority-based failover
- Configure failover priorities per region
- For writes: manual or automatic failover

**Conflict Resolution (Multi-Region Writes):**
- **Last Writer Wins (LWW)**: Based on `_ts` timestamp property
- **Custom**: Using stored procedure
- **Async**: Via conflict feed

```csharp
// Configure conflict resolution policy
ContainerProperties properties = new ContainerProperties("container1", "/partitionKey")
{
    ConflictResolutionPolicy = new ConflictResolutionPolicy
    {
        Mode = ConflictResolutionMode.LastWriterWins,
        ResolutionPath = "/_ts"  // Default timestamp
    }
};
```

### 1.8 Indexing in Cosmos DB

**Default Indexing:**
- All properties automatically indexed by default
- No schema or secondary indexes required
- Enables efficient queries on any property
- Can customize for optimization

**Index Types:**

#### **1. Range Index**
- Default for strings and numbers
- Supports equality, range, ORDER BY queries
- Efficient for: =, >, <, >=, <=, ORDER BY

#### **2. Spatial Index**
- For geospatial data (Point, Polygon, LineString)
- Supports distance and within queries
- Must be explicitly enabled

#### **3. Composite Index**
- Index on multiple properties together
- Required for ORDER BY on multiple properties
- Improves query performance for specific patterns

**Indexing Policy:**

```json
{
  "indexingMode": "consistent",
  "automatic": true,
  "includedPaths": [
    {
      "path": "/*"
    }
  ],
  "excludedPaths": [
    {
      "path": "/largeTextField/*"
    },
    {
      "path": "/_etag/?"
    }
  ],
  "compositeIndexes": [
    [
      { "path": "/name", "order": "ascending" },
      { "path": "/age", "order": "descending" }
    ]
  ],
  "spatialIndexes": [
    {
      "path": "/location/*",
      "types": ["Point", "Polygon"]
    }
  ]
}
```

**Indexing Modes:**

| Mode | Description |
|------|-------------|
| Consistent | Index updated synchronously with writes |
| None | Indexing disabled, only point reads |

**Optimizing Indexing:**
- Exclude large text fields not used in queries
- Exclude properties never queried
- Add composite indexes for complex ORDER BY
- Use spatial indexes only when needed

```csharp
// Set indexing policy
ContainerProperties properties = new ContainerProperties("container1", "/partitionKey")
{
    IndexingPolicy = new IndexingPolicy
    {
        IndexingMode = IndexingMode.Consistent,
        Automatic = true,
        IncludedPaths = { new IncludedPath { Path = "/*" } },
        ExcludedPaths = { new ExcludedPath { Path = "/description/*" } }
    }
};
```

### 1.9 Time-to-Live (TTL)

**What is TTL?**
- Automatically delete items after a period of time (in seconds)
- Uses `_ts` (last modified timestamp) to calculate expiration
- Deletes consume RUs but are background operations
- No additional cost for TTL beyond the RUs used for deletion

**TTL Configuration Levels:**

| Level | Setting | Behavior |
|-------|---------|----------|
| **Container default OFF** | TTL not set (null) | Items never expire (default) |
| **Container default ON, no default value** | TTL = -1 | Items never expire unless item has its own TTL |
| **Container default ON, with value** | TTL = N seconds | Items expire N seconds after last modification |
| **Item-level override** | `ttl` property on item | Overrides container default for that specific item |

**TTL Behavior Matrix:**

| Container TTL | Item TTL | Result |
|---------------|----------|--------|
| Not set (null/off) | Any value | Item **never expires** (container TTL must be enabled first) |
| -1 (on, no expiry) | Not set | Item **never expires** |
| -1 (on, no expiry) | 10 | Item **expires in 10 seconds** |
| 100 | Not set (null) | Item **expires in 100 seconds** (inherits container) |
| 100 | -1 | Item **never expires** (overrides container) |
| 100 | 50 | Item **expires in 50 seconds** (overrides container) |

**⚠️ Key Exam Rule:** Container-level TTL must be enabled FIRST before any item-level TTL takes effect.

**Setting TTL in Code:**

```csharp
// Enable TTL on container with 90-day default
ContainerProperties properties = new ContainerProperties("myContainer", "/partitionKey")
{
    DefaultTimeToLive = 90 * 24 * 60 * 60  // 90 days in seconds
};

// Enable TTL on container with NO default (items must set their own)
ContainerProperties properties2 = new ContainerProperties("myContainer2", "/partitionKey")
{
    DefaultTimeToLive = -1  // Enabled but no default expiration
};
```

```json
// Item with custom TTL (overrides container default)
{
    "id": "session-token-123",
    "partitionKey": "user1",
    "token": "abc123",
    "ttl": 3600  // Expires in 1 hour (3600 seconds)
}

// Item that never expires (even if container has default TTL)
{
    "id": "permanent-record",
    "partitionKey": "user1",
    "data": "important",
    "ttl": -1  // Never expires
}
```

**CLI:**
```bash
# Enable TTL with 30-day default
az cosmosdb sql container create \
  --account-name <account> --resource-group <rg> \
  --database-name <db> --name <container> \
  --partition-key-path "/pk" \
  --analytical-storage-ttl -1 \
  --default-ttl 2592000  # 30 days
```

### 1.10 Data Modeling: Embedding vs. Referencing

**Embedding (Denormalization):**
- Store related data INSIDE the same document
- Best for Cosmos DB because of single-document reads
- **One-to-few** relationships (addresses, phone numbers)
- Data that is **read together** should be **stored together**

```json
// EMBEDDED model - Order with line items inside
{
    "id": "order-001",
    "customerId": "cust-123",
    "customerName": "John Doe",
    "customerEmail": "john@example.com",
    "items": [
        { "productId": "prod-1", "name": "Laptop", "price": 999.99, "qty": 1 },
        { "productId": "prod-2", "name": "Mouse", "price": 29.99, "qty": 2 }
    ],
    "shippingAddress": {
        "street": "123 Main St",
        "city": "Seattle",
        "state": "WA",
        "zip": "98101"
    },
    "total": 1059.97
}
```

**Referencing (Normalization):**
- Store related data in **separate documents** with references (IDs)
- Requires **multiple reads/queries** to assemble full data
- Use for **one-to-many** or **many-to-many** with large/unbounded child data
- Use when embedded data would **exceed 2 MB item limit**

```json
// REFERENCED model - Separate documents
// Order document
{
    "id": "order-001",
    "customerId": "cust-123",
    "itemIds": ["item-001", "item-002"],
    "total": 1059.97
}

// Item documents (separate)
{
    "id": "item-001",
    "orderId": "order-001",
    "productId": "prod-1",
    "name": "Laptop",
    "price": 999.99
}
```

**When to Embed vs. Reference:**

| Criteria | Embed | Reference |
|----------|-------|-----------|
| Relationship | One-to-few | One-to-many (unbounded) |
| Data read pattern | Read together frequently | Read independently |
| Data update pattern | Updated together | Updated independently |
| Data size | Fits within 2 MB limit | Would exceed 2 MB |
| Data volatility | Relatively static | Frequently changing child data |
| Query pattern | Single query retrieves all | Multiple queries acceptable |

**Hybrid Approach (Common in Practice):**
- Embed **frequently accessed** data (denormalize)
- Reference **infrequently accessed** or **large** data
- Duplicate some data for read performance (accept update cost)

```json
// Hybrid - embed summary, reference details
{
    "id": "order-001",
    "customerId": "cust-123",
    "customerName": "John Doe",       // Denormalized (duplicate)
    "itemCount": 3,                    // Summary
    "total": 1059.97,
    "topItem": "Laptop",              // Most relevant embedded
    "itemIds": ["item-001", "item-002", "item-003"]  // References for details
}
```

### 1.11 Unique Keys

**What are Unique Keys?**
- Enforce uniqueness constraint on one or more properties **within a logical partition**
- Set at container creation time and **cannot be changed after creation**
- Scoped to logical partition (same unique value CAN exist in different partitions)

```csharp
// Create container with unique key policy
ContainerProperties properties = new ContainerProperties("myContainer", "/tenantId")
{
    UniqueKeyPolicy = new UniqueKeyPolicy
    {
        UniqueKeys =
        {
            // Each tenant can have only one user with this email
            new UniqueKey { Paths = { "/email" } },
            // Composite unique key: combination must be unique per partition
            new UniqueKey { Paths = { "/department", "/employeeId" } }
        }
    }
};

await database.CreateContainerAsync(properties, 400);
```

**Key Exam Points:**
- Unique keys are per **logical partition**, NOT globally unique
- Must be defined at **container creation** (immutable)
- `null` values participate in uniqueness (only one null allowed per unique key per partition)
- Composite unique keys: the **combination** of paths must be unique
- Maximum 16 paths per unique key, maximum 10 unique keys per policy

```bash
# CLI: Create container with unique keys
az cosmosdb sql container create \
  --account-name <account> --resource-group <rg> \
  --database-name <db> --name <container> \
  --partition-key-path "/tenantId" \
  --unique-key-policy '{"uniqueKeys": [{"paths": ["/email"]}, {"paths": ["/dept", "/empId"]}]}'
```

---

## MODULE 2: WORK WITH AZURE COSMOS DB

### 2.1 Azure Cosmos DB .NET SDK V3

**Package Installation:**

```bash
# Install via NuGet
dotnet add package Microsoft.Azure.Cosmos
```

**Key Namespaces:**
```csharp
using Microsoft.Azure.Cosmos;
using Microsoft.Azure.Cosmos.Linq;
using Microsoft.Azure.Cosmos.Scripts;
```

### 2.2 Client Classes Overview

**CosmosClient**
- Entry point for all operations
- Represents connection to Cosmos DB account
- Thread-safe, should be singleton
- Manages connections and resources

```csharp
// Create CosmosClient (recommended: singleton)
CosmosClient cosmosClient = new CosmosClient(
    connectionString: "<connection-string>");

// Or with endpoint and key
CosmosClient cosmosClient = new CosmosClient(
    accountEndpoint: "https://<account>.documents.azure.com:443/",
    authKeyOrResourceToken: "<primary-key>");

// With options
CosmosClientOptions options = new CosmosClientOptions
{
    ApplicationRegion = Regions.EastUS,
    ConnectionMode = ConnectionMode.Direct,
    ConsistencyLevel = ConsistencyLevel.Session
};
CosmosClient cosmosClient = new CosmosClient(connectionString, options);
```

**Database**
- Represents a database resource
- Manages containers within the database

```csharp
// Get database reference (doesn't make network call)
Database database = cosmosClient.GetDatabase("myDatabase");

// Create database
DatabaseResponse response = await cosmosClient.CreateDatabaseIfNotExistsAsync(
    id: "myDatabase",
    throughput: 400);  // Optional: shared throughput
Database database = response.Database;
```

**Container**
- Represents a container resource
- Primary interface for CRUD operations

```csharp
// Get container reference
Container container = database.GetContainer("myContainer");

// Create container
ContainerProperties properties = new ContainerProperties(
    id: "myContainer",
    partitionKeyPath: "/partitionKey");

ContainerResponse response = await database.CreateContainerIfNotExistsAsync(
    containerProperties: properties,
    throughput: 400);
Container container = response.Container;
```

### 2.3 Authentication Methods

**1. Connection String:**
```csharp
// From Azure Portal: Cosmos DB Account → Keys
string connectionString = "AccountEndpoint=https://<account>.documents.azure.com:443/;AccountKey=<key>;";
CosmosClient client = new CosmosClient(connectionString);
```

**2. Primary/Secondary Key:**
```csharp
string endpoint = "https://<account>.documents.azure.com:443/";
string key = "<primary-or-secondary-key>";
CosmosClient client = new CosmosClient(endpoint, key);
```

**3. Azure AD (DefaultAzureCredential):**
```csharp
// Requires Azure.Identity package
using Azure.Identity;

string endpoint = "https://<account>.documents.azure.com:443/";
CosmosClient client = new CosmosClient(endpoint, new DefaultAzureCredential());

// Requires RBAC role assignment:
// - Cosmos DB Built-in Data Reader
// - Cosmos DB Built-in Data Contributor
// - Cosmos DB Built-in Data Owner
```

**4. Resource Tokens:**
```csharp
// Limited-scope tokens for specific resources
string resourceToken = "<resource-token>";
CosmosClient client = new CosmosClient(endpoint, resourceToken);
```

### 2.4 Connection Modes: Direct vs. Gateway

**Two connection modes available:**

| Aspect | Direct Mode | Gateway Mode |
|--------|------------|-------------|
| **Protocol** | TCP | HTTPS |
| **Connections** | Directly to backend replicas | Through Cosmos DB gateway |
| **Performance** | **Lower latency** (fewer network hops) | Higher latency (extra hop) |
| **Port requirements** | Range of ports (10000-20000) | Only port 443 |
| **Best for** | Production workloads | Restricted network environments |
| **Default** | **Yes (default in .NET SDK v3)** | No |
| **Firewall friendly** | Requires port range | Yes (only HTTPS 443) |

```csharp
// Direct Mode (default, best performance)
CosmosClientOptions directOptions = new CosmosClientOptions
{
    ConnectionMode = ConnectionMode.Direct
};

// Gateway Mode (use when behind firewall/proxy that blocks TCP)
CosmosClientOptions gatewayOptions = new CosmosClientOptions
{
    ConnectionMode = ConnectionMode.Gateway,
    GatewayModeMaxConnectionLimit = 50  // Max concurrent connections
};

CosmosClient client = new CosmosClient(connectionString, directOptions);
```

**⚠️ Exam Tip:** Use Gateway mode when network restrictions prevent opening TCP port ranges. Use Direct mode (default) for lowest latency in production.

### 2.5 CosmosClientOptions Deep Dive

```csharp
CosmosClientOptions options = new CosmosClientOptions
{
    // Connection
    ConnectionMode = ConnectionMode.Direct,
    
    // Region configuration
    ApplicationRegion = Regions.EastUS,               // Single preferred region
    // OR use multiple preferred regions (ordered preference):
    ApplicationPreferredRegions = new List<string>
    {
        Regions.EastUS,
        Regions.WestUS,
        Regions.NorthEurope
    },
    
    // Performance
    AllowBulkExecution = true,        // Enable bulk mode for high-throughput
    MaxRetryAttemptsOnRateLimitedRequests = 9,   // Default: 9
    MaxRetryWaitTimeOnRateLimitedRequests = TimeSpan.FromSeconds(30),  // Default: 30s
    
    // Consistency (override account default - can only WEAKEN)
    ConsistencyLevel = ConsistencyLevel.Eventual,
    
    // Serialization
    SerializerOptions = new CosmosSerializationOptions
    {
        PropertyNamingPolicy = CosmosPropertyNamingPolicy.CamelCase
    },
    
    // Diagnostics threshold
    CosmosClientTelemetryOptions = new CosmosClientTelemetryOptions
    {
        DisableDistributedTracing = false
    }
};
```

**Key Options for Exam:**

| Option | Purpose | Default |
|--------|---------|---------|
| `ConnectionMode` | Direct or Gateway | Direct |
| `ApplicationRegion` | Preferred region for reads | None (uses write region) |
| `ApplicationPreferredRegions` | Ordered list of preferred regions | None |
| `AllowBulkExecution` | Batch operations for throughput | false |
| `MaxRetryAttemptsOnRateLimitedRequests` | Retries on 429 errors | 9 |
| `MaxRetryWaitTimeOnRateLimitedRequests` | Max wait for 429 retries | 30 seconds |
| `ConsistencyLevel` | Override default (can only weaken) | Account default |

### 2.6 Bulk Operations

**When to use Bulk:**
- Ingesting large volumes of data
- Migrating data into Cosmos DB
- High-throughput insert/update scenarios

```csharp
// Enable bulk execution
CosmosClientOptions options = new CosmosClientOptions
{
    AllowBulkExecution = true  // REQUIRED for bulk operations
};
CosmosClient client = new CosmosClient(connectionString, options);
Container container = client.GetContainer("myDb", "myContainer");

// Create many items concurrently
List<Product> products = GetLargeProductList(); // thousands of items
List<Task> tasks = new List<Task>();

foreach (Product product in products)
{
    // All operations run concurrently and are batched by SDK
    tasks.Add(container.CreateItemAsync(product, new PartitionKey(product.Category))
        .ContinueWith(response =>
        {
            if (!response.IsCompletedSuccessfully)
            {
                Console.WriteLine($"Failed: {response.Exception}");
            }
        }));
}

await Task.WhenAll(tasks);
```

**Bulk vs Transactional Batch:**

| Feature | Bulk Operations | Transactional Batch |
|---------|----------------|-------------------|
| Atomicity | No (best-effort) | Yes (all or nothing) |
| Partition key scope | Any partition key | Single partition key |
| Max operations | Unlimited | 100 operations / 2 MB |
| Use case | Data migration, high throughput | Atomic multi-item operations |
| Enabled via | `AllowBulkExecution = true` | `CreateTransactionalBatch()` |

### 2.7 CRUD Operations

**Create Item:**

```csharp
public class Product
{
    [JsonProperty("id")]
    public string Id { get; set; }
    
    [JsonProperty("partitionKey")]
    public string PartitionKey { get; set; }
    
    [JsonProperty("name")]
    public string Name { get; set; }
    
    [JsonProperty("price")]
    public decimal Price { get; set; }
    
    [JsonProperty("category")]
    public string Category { get; set; }
}

// Create (insert) item
Product newProduct = new Product
{
    Id = Guid.NewGuid().ToString(),
    PartitionKey = "electronics",
    Name = "Laptop",
    Price = 999.99m,
    Category = "Computers"
};

ItemResponse<Product> response = await container.CreateItemAsync(
    item: newProduct,
    partitionKey: new PartitionKey(newProduct.PartitionKey));

Console.WriteLine($"Created item. RU charge: {response.RequestCharge}");
```

**Read Item (Point Read):**

```csharp
// Point read - most efficient (1 RU for 1 KB item)
ItemResponse<Product> response = await container.ReadItemAsync<Product>(
    id: "item-id",
    partitionKey: new PartitionKey("electronics"));

Product product = response.Resource;
Console.WriteLine($"Read item: {product.Name}. RU charge: {response.RequestCharge}");

// With request options
ItemRequestOptions options = new ItemRequestOptions
{
    ConsistencyLevel = ConsistencyLevel.Eventual
};

response = await container.ReadItemAsync<Product>(
    id: "item-id",
    partitionKey: new PartitionKey("electronics"),
    requestOptions: options);
```

**Update (Replace) Item:**

```csharp
// Full replacement (not partial update)
product.Price = 899.99m;
product.Name = "Gaming Laptop";

ItemResponse<Product> response = await container.ReplaceItemAsync(
    item: product,
    id: product.Id,
    partitionKey: new PartitionKey(product.PartitionKey));

Console.WriteLine($"Updated item. RU charge: {response.RequestCharge}");
```

**Upsert Item:**

```csharp
// Creates if not exists, updates if exists
ItemResponse<Product> response = await container.UpsertItemAsync(
    item: product,
    partitionKey: new PartitionKey(product.PartitionKey));

Console.WriteLine($"Upserted item. RU charge: {response.RequestCharge}");
```

**Patch Item (Partial Update):**

```csharp
// Partial update - update specific properties only
List<PatchOperation> patchOperations = new List<PatchOperation>
{
    PatchOperation.Replace("/price", 799.99),
    PatchOperation.Add("/discount", 0.1),
    PatchOperation.Remove("/oldProperty"),
    PatchOperation.Set("/lastUpdated", DateTime.UtcNow),
    PatchOperation.Increment("/viewCount", 1)
};

ItemResponse<Product> response = await container.PatchItemAsync<Product>(
    id: "item-id",
    partitionKey: new PartitionKey("electronics"),
    patchOperations: patchOperations);
```

**Delete Item:**

```csharp
ItemResponse<Product> response = await container.DeleteItemAsync<Product>(
    id: "item-id",
    partitionKey: new PartitionKey("electronics"));

Console.WriteLine($"Deleted item. RU charge: {response.RequestCharge}");
```

### 2.8 Querying Data

**SQL Query:**

```csharp
// Define query
string queryText = "SELECT * FROM c WHERE c.category = @category";
QueryDefinition query = new QueryDefinition(queryText)
    .WithParameter("@category", "Computers");

// Execute query
FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query);

List<Product> results = new List<Product>();
double totalRU = 0;

while (iterator.HasMoreResults)
{
    FeedResponse<Product> response = await iterator.ReadNextAsync();
    totalRU += response.RequestCharge;
    results.AddRange(response);
}

Console.WriteLine($"Found {results.Count} items. Total RU: {totalRU}");
```

**Query with Options:**

```csharp
QueryRequestOptions options = new QueryRequestOptions
{
    PartitionKey = new PartitionKey("electronics"),  // Single partition query
    MaxItemCount = 100,  // Max items per page
    MaxConcurrency = -1,  // Auto-parallelize cross-partition queries
    ConsistencyLevel = ConsistencyLevel.Eventual
};

FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(
    queryDefinition: query,
    requestOptions: options);
```

**LINQ Queries:**

```csharp
using Microsoft.Azure.Cosmos.Linq;

// Get LINQ queryable
IOrderedQueryable<Product> queryable = container.GetItemLinqQueryable<Product>();

// Build LINQ query
var linqQuery = queryable
    .Where(p => p.Category == "Computers")
    .Where(p => p.Price < 1000)
    .OrderBy(p => p.Name)
    .Select(p => new { p.Id, p.Name, p.Price });

// Convert to FeedIterator
using FeedIterator<dynamic> iterator = linqQuery.ToFeedIterator();

while (iterator.HasMoreResults)
{
    FeedResponse<dynamic> response = await iterator.ReadNextAsync();
    foreach (var item in response)
    {
        Console.WriteLine($"{item.Name}: ${item.Price}");
    }
}
```

**Common Query Patterns:**

```sql
-- Select all items
SELECT * FROM c

-- Filter by property
SELECT * FROM c WHERE c.category = "Electronics"

-- Filter with multiple conditions
SELECT * FROM c WHERE c.price > 100 AND c.price < 500

-- Select specific properties
SELECT c.id, c.name, c.price FROM c

-- ORDER BY (single property)
SELECT * FROM c ORDER BY c.name ASC

-- ORDER BY (multiple properties - requires composite index)
SELECT * FROM c ORDER BY c.category ASC, c.price DESC

-- TOP N results
SELECT TOP 10 * FROM c ORDER BY c.price DESC

-- OFFSET LIMIT (pagination)
SELECT * FROM c ORDER BY c.name OFFSET 10 LIMIT 10

-- Array contains
SELECT * FROM c WHERE ARRAY_CONTAINS(c.tags, "featured")

-- Join (within document)
SELECT p.name, t AS tag
FROM products p
JOIN t IN p.tags

-- Aggregate functions
SELECT COUNT(1) AS itemCount FROM c
SELECT AVG(c.price) AS avgPrice FROM c WHERE c.category = "Electronics"

-- String functions
SELECT * FROM c WHERE STARTSWITH(c.name, "Pro")
SELECT * FROM c WHERE CONTAINS(c.description, "gaming")

-- Mathematical functions
SELECT c.name, ROUND(c.price * 1.1) AS priceWithTax FROM c
```

### 2.9 Stored Procedures

**What are Stored Procedures?**
- JavaScript functions executed on server-side
- Run within partition key scope
- ACID transactions within partition
- Can batch operations for efficiency

**Creating a Stored Procedure:**

```javascript
// Stored procedure: createItems
function createItems(items) {
    var context = getContext();
    var container = context.getCollection();
    var response = context.getResponse();
    
    var createdItems = [];
    
    for (var i = 0; i < items.length; i++) {
        var accepted = container.createDocument(
            container.getSelfLink(),
            items[i],
            function(err, itemCreated) {
                if (err) throw new Error("Error creating item: " + err.message);
                createdItems.push(itemCreated);
            }
        );
        
        if (!accepted) {
            response.setBody({ 
                created: createdItems.length, 
                message: "Batch size exceeded" 
            });
            return;
        }
    }
    
    response.setBody({ created: createdItems.length, items: createdItems });
}
```

**Registering and Executing Stored Procedure:**

```csharp
// Register stored procedure
string sprocBody = File.ReadAllText("createItems.js");

StoredProcedureProperties sprocProperties = new StoredProcedureProperties
{
    Id = "createItems",
    Body = sprocBody
};

StoredProcedureResponse response = await container.Scripts.CreateStoredProcedureAsync(sprocProperties);

// Execute stored procedure
var items = new[]
{
    new { id = "1", partitionKey = "pk1", name = "Item 1" },
    new { id = "2", partitionKey = "pk1", name = "Item 2" }
};

StoredProcedureExecuteResponse<dynamic> execResponse = await container.Scripts.ExecuteStoredProcedureAsync<dynamic>(
    storedProcedureId: "createItems",
    partitionKey: new PartitionKey("pk1"),
    parameters: new dynamic[] { items });

Console.WriteLine($"Created: {execResponse.Resource.created}");
Console.WriteLine($"RU charge: {execResponse.RequestCharge}");
```

### 2.10 Triggers

**Pre-Triggers:**
- Execute before an operation
- Can validate or transform data
- Can cancel operation by throwing error

```javascript
// Pre-trigger: validate item
function validateItem() {
    var context = getContext();
    var request = context.getRequest();
    var itemToCreate = request.getBody();
    
    // Validation
    if (!itemToCreate.name) {
        throw new Error("Name is required");
    }
    
    // Add timestamp
    itemToCreate.createdAt = new Date().toISOString();
    
    // Update request body
    request.setBody(itemToCreate);
}
```

**Post-Triggers:**
- Execute after an operation
- Cannot modify the original operation result
- Can create additional documents

```javascript
// Post-trigger: create audit log
function createAuditLog() {
    var context = getContext();
    var container = context.getCollection();
    var response = context.getResponse();
    var createdItem = response.getBody();
    
    var auditLog = {
        id: "audit-" + createdItem.id,
        partitionKey: createdItem.partitionKey,
        operation: "CREATE",
        itemId: createdItem.id,
        timestamp: new Date().toISOString()
    };
    
    container.createDocument(container.getSelfLink(), auditLog);
}
```

**Using Triggers:**

```csharp
// Register trigger
TriggerProperties triggerProperties = new TriggerProperties
{
    Id = "validateItem",
    Body = File.ReadAllText("validateItem.js"),
    TriggerType = TriggerType.Pre,
    TriggerOperation = TriggerOperation.Create
};

await container.Scripts.CreateTriggerAsync(triggerProperties);

// Use trigger in operation
ItemRequestOptions options = new ItemRequestOptions
{
    PreTriggers = new[] { "validateItem" },
    PostTriggers = new[] { "createAuditLog" }
};

await container.CreateItemAsync(newItem, new PartitionKey(newItem.PartitionKey), options);
```

### 2.11 User-Defined Functions (UDFs)

**What are UDFs?**
- JavaScript functions for use in queries
- Extend SQL query capabilities
- Called from SELECT, WHERE, ORDER BY clauses

**Creating a UDF:**

```javascript
// UDF: Calculate discounted price
function calculateDiscount(price, discountPercent) {
    if (discountPercent < 0 || discountPercent > 100) {
        throw new Error("Invalid discount percentage");
    }
    return price * (1 - discountPercent / 100);
}
```

**Registering and Using UDF:**

```csharp
// Register UDF
UserDefinedFunctionProperties udfProperties = new UserDefinedFunctionProperties
{
    Id = "calculateDiscount",
    Body = "function calculateDiscount(price, discountPercent) { return price * (1 - discountPercent / 100); }"
};

await container.Scripts.CreateUserDefinedFunctionAsync(udfProperties);

// Use UDF in query
string query = @"
    SELECT c.name, 
           c.price, 
           udf.calculateDiscount(c.price, 10) AS discountedPrice
    FROM c 
    WHERE udf.calculateDiscount(c.price, 10) < 500";

FeedIterator<dynamic> iterator = container.GetItemQueryIterator<dynamic>(query);
```

### 2.12 Change Feed

**What is Change Feed?**
- Sorted list of changed documents in order of modification
- Persistent, incremental feed of changes
- Enables event-driven architectures
- Powers real-time scenarios

**Use Cases:**
- Real-time data synchronization
- Event sourcing
- Materialized views
- Search index updates
- Trigger Azure Functions

**Change Feed Processor:**

```csharp
// Create lease container for tracking
Container leaseContainer = database.GetContainer("leases");

// Build change feed processor
ChangeFeedProcessor processor = container
    .GetChangeFeedProcessorBuilder<Product>(
        processorName: "myProcessor",
        onChangesDelegate: HandleChangesAsync)
    .WithInstanceName("instance1")
    .WithLeaseContainer(leaseContainer)
    .WithStartTime(DateTime.UtcNow.AddHours(-1))  // Start from 1 hour ago
    .Build();

// Start processor
await processor.StartAsync();

// Handle changes
static async Task HandleChangesAsync(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<Product> changes,
    CancellationToken cancellationToken)
{
    Console.WriteLine($"Processing {changes.Count} changes from partition {context.LeaseToken}");
    
    foreach (Product item in changes)
    {
        Console.WriteLine($"Changed: {item.Id} - {item.Name}");
        // Process change: update cache, sync to another system, etc.
    }
}

// Stop processor
await processor.StopAsync();
```

**Change Feed with Azure Functions:**

```csharp
// Azure Function triggered by Cosmos DB change feed
[FunctionName("ProcessChanges")]
public static void Run(
    [CosmosDBTrigger(
        databaseName: "myDatabase",
        containerName: "myContainer",
        Connection = "CosmosDBConnection",
        LeaseContainerName = "leases",
        CreateLeaseContainerIfNotExists = true)]
    IReadOnlyList<Product> changes,
    ILogger log)
{
    if (changes != null && changes.Count > 0)
    {
        log.LogInformation($"Documents modified: {changes.Count}");
        foreach (var item in changes)
        {
            log.LogInformation($"Id: {item.Id}");
        }
    }
}
```

### 2.13 Transactional Batch Operations

**What is Transactional Batch?**
- Execute multiple operations as single transaction
- All operations in same partition key
- All succeed or all fail (atomicity)
- Limited to 100 operations or 2 MB

```csharp
// Create batch
PartitionKey partitionKey = new PartitionKey("electronics");

TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey);

// Add operations
batch.CreateItem(new Product { Id = "1", PartitionKey = "electronics", Name = "Laptop" });
batch.CreateItem(new Product { Id = "2", PartitionKey = "electronics", Name = "Mouse" });
batch.UpsertItem(new Product { Id = "3", PartitionKey = "electronics", Name = "Keyboard" });
batch.ReplaceItem("4", new Product { Id = "4", PartitionKey = "electronics", Name = "Monitor" });
batch.DeleteItem("5");

// Execute batch
TransactionalBatchResponse response = await batch.ExecuteAsync();

if (response.IsSuccessStatusCode)
{
    Console.WriteLine($"Batch succeeded. RU: {response.RequestCharge}");
    
    // Access individual results
    TransactionalBatchOperationResult<Product> result1 = response.GetOperationResultAtIndex<Product>(0);
    Product createdProduct = result1.Resource;
}
else
{
    Console.WriteLine($"Batch failed: {response.StatusCode}");
}
```

### 2.14 Error Handling and Best Practices

**Common Exceptions:**

| Exception | Cause | Solution |
|-----------|-------|----------|
| CosmosException (429) | Rate limiting (RU exceeded) | Implement retry with backoff |
| CosmosException (404) | Item or resource not found | Check ID and partition key |
| CosmosException (409) | Conflict (item already exists) | Use upsert or handle conflict |
| CosmosException (412) | Precondition failed (ETag mismatch) | Refresh item and retry |
| CosmosException (413) | Request too large | Reduce item size |
| CosmosException (449) | Retry with (transient error) | Automatic retry by SDK |

**Error Handling Pattern:**

```csharp
try
{
    ItemResponse<Product> response = await container.CreateItemAsync(product, 
        new PartitionKey(product.PartitionKey));
}
catch (CosmosException ex) when (ex.StatusCode == HttpStatusCode.Conflict)
{
    // Item already exists - handle duplicate
    Console.WriteLine("Item already exists, using upsert instead");
    await container.UpsertItemAsync(product, new PartitionKey(product.PartitionKey));
}
catch (CosmosException ex) when (ex.StatusCode == HttpStatusCode.TooManyRequests)
{
    // Rate limited - implement backoff
    Console.WriteLine($"Rate limited. Retry after: {ex.RetryAfter}");
    await Task.Delay(ex.RetryAfter ?? TimeSpan.FromSeconds(1));
    // Retry operation
}
catch (CosmosException ex)
{
    Console.WriteLine($"Cosmos DB error: {ex.StatusCode} - {ex.Message}");
    Console.WriteLine($"Activity ID: {ex.ActivityId}");
    Console.WriteLine($"Request charge: {ex.RequestCharge}");
    throw;
}
```

**Optimistic Concurrency with ETags:**

```csharp
// Read item with ETag
ItemResponse<Product> response = await container.ReadItemAsync<Product>(
    id: "product1",
    partitionKey: new PartitionKey("electronics"));

Product product = response.Resource;
string etag = response.ETag;

// Modify product
product.Price = 899.99m;

// Update with ETag check
ItemRequestOptions options = new ItemRequestOptions
{
    IfMatchEtag = etag
};

try
{
    await container.ReplaceItemAsync(product, product.Id, 
        new PartitionKey(product.PartitionKey), options);
}
catch (CosmosException ex) when (ex.StatusCode == HttpStatusCode.PreconditionFailed)
{
    Console.WriteLine("Item was modified by another process. Refresh and retry.");
}
```

**Best Practices:**

1. **Use Singleton CosmosClient:**
```csharp
// Singleton pattern
public class CosmosDbService
{
    private static readonly Lazy<CosmosClient> _lazyClient = new Lazy<CosmosClient>(() =>
        new CosmosClient(connectionString, new CosmosClientOptions
        {
            ConnectionMode = ConnectionMode.Direct,
            ApplicationRegion = Regions.EastUS
        }));
    
    public static CosmosClient Client => _lazyClient.Value;
}
```

2. **Use Direct Mode for Best Performance:**
```csharp
CosmosClientOptions options = new CosmosClientOptions
{
    ConnectionMode = ConnectionMode.Direct  // Better than Gateway mode
};
```

3. **Use Partition Key in Queries:**
```csharp
// Always include partition key when possible
QueryRequestOptions options = new QueryRequestOptions
{
    PartitionKey = new PartitionKey("electronics")
};
```

4. **Prefer Point Reads Over Queries:**
```csharp
// Point read: 1 RU for 1 KB
var item = await container.ReadItemAsync<Product>(id, partitionKey);

// Query: More expensive
var query = "SELECT * FROM c WHERE c.id = @id";
```

5. **Use Async Methods:**
```csharp
// Always use async
await container.CreateItemAsync(item, partitionKey);
```

6. **Monitor RU Consumption:**
```csharp
ItemResponse<Product> response = await container.CreateItemAsync(item, partitionKey);
Console.WriteLine($"RU consumed: {response.RequestCharge}");
```

### 2.15 Complete Code Examples

**Example 1: Complete CRUD Operations**

```csharp
using Microsoft.Azure.Cosmos;

public class CosmosDbRepository<T> where T : class
{
    private readonly Container _container;

    public CosmosDbRepository(CosmosClient client, string databaseId, string containerId)
    {
        _container = client.GetContainer(databaseId, containerId);
    }

    public async Task<T> GetByIdAsync(string id, string partitionKey)
    {
        try
        {
            ItemResponse<T> response = await _container.ReadItemAsync<T>(
                id, new PartitionKey(partitionKey));
            return response.Resource;
        }
        catch (CosmosException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
        {
            return null;
        }
    }

    public async Task<IEnumerable<T>> GetAllAsync(string partitionKey)
    {
        var query = _container.GetItemQueryIterator<T>(
            new QueryDefinition("SELECT * FROM c"),
            requestOptions: new QueryRequestOptions
            {
                PartitionKey = new PartitionKey(partitionKey)
            });

        var results = new List<T>();
        while (query.HasMoreResults)
        {
            var response = await query.ReadNextAsync();
            results.AddRange(response);
        }
        return results;
    }

    public async Task<T> CreateAsync(T item, string partitionKey)
    {
        var response = await _container.CreateItemAsync(item, new PartitionKey(partitionKey));
        return response.Resource;
    }

    public async Task<T> UpdateAsync(string id, T item, string partitionKey)
    {
        var response = await _container.ReplaceItemAsync(item, id, new PartitionKey(partitionKey));
        return response.Resource;
    }

    public async Task DeleteAsync(string id, string partitionKey)
    {
        await _container.DeleteItemAsync<T>(id, new PartitionKey(partitionKey));
    }
}
```

**Example 2: Query with Pagination**

```csharp
public async Task<(List<Product> Items, string ContinuationToken)> GetProductsPagedAsync(
    string category, 
    int pageSize = 10, 
    string continuationToken = null)
{
    var query = new QueryDefinition("SELECT * FROM c WHERE c.category = @category")
        .WithParameter("@category", category);

    var options = new QueryRequestOptions
    {
        MaxItemCount = pageSize
    };

    using FeedIterator<Product> iterator = _container.GetItemQueryIterator<Product>(
        query, 
        continuationToken, 
        options);

    var items = new List<Product>();
    string newContinuationToken = null;

    if (iterator.HasMoreResults)
    {
        FeedResponse<Product> response = await iterator.ReadNextAsync();
        items.AddRange(response);
        newContinuationToken = response.ContinuationToken;
    }

    return (items, newContinuationToken);
}
```

---

## MODULE 3: ADVANCED CONCEPTS (BEYOND MS LEARN - EXAM CRITICAL)

### 3.1 Server-Side Programming Limits & Bounded Execution

**Execution Constraints:**
- Stored procedures, triggers, and UDFs run in a **sandboxed JavaScript runtime**
- **Maximum execution time: 5 seconds** (bounded execution)
- If execution doesn't complete within the time limit, it is rolled back
- All operations must complete within the time budget

**Bounded Execution Pattern (Continuation):**
- For operations on large datasets, use the `accepted` return value
- If `accepted` is `false`, the operation was NOT queued (resource budget exceeded)
- Return partial results and use a continuation token

```javascript
// Stored procedure with bounded execution
function bulkDelete(query) {
    var context = getContext();
    var container = context.getCollection();
    var response = context.getResponse();
    var responseBody = { deleted: 0, continuation: true };

    // Query for items to delete
    var accepted = container.queryDocuments(
        container.getSelfLink(),
        query,
        {},
        function(err, documents, responseOptions) {
            if (err) throw err;
            
            if (documents.length === 0) {
                responseBody.continuation = false;
                response.setBody(responseBody);
                return;
            }

            // Delete each document
            for (var i = 0; i < documents.length; i++) {
                var accepted = container.deleteDocument(
                    documents[i]._self,
                    {},
                    function(err) { if (err) throw err; }
                );

                // If not accepted, return with continuation = true
                if (!accepted) {
                    response.setBody(responseBody);
                    return;
                }
                responseBody.deleted++;
            }
            
            response.setBody(responseBody);
        }
    );

    // If initial query not accepted
    if (!accepted) {
        response.setBody(responseBody);
    }
}
```

**Client-side continuation loop:**
```csharp
// Keep calling until all items processed
bool continuation = true;
int totalDeleted = 0;

while (continuation)
{
    var result = await container.Scripts.ExecuteStoredProcedureAsync<dynamic>(
        "bulkDelete",
        new PartitionKey("partitionValue"),
        new dynamic[] { "SELECT * FROM c WHERE c.expired = true" });

    totalDeleted += (int)result.Resource.deleted;
    continuation = (bool)result.Resource.continuation;
}
```

**Key Limits:**

| Resource | Limit |
|----------|-------|
| Execution time | 5 seconds |
| Request size | 2 MB |
| Response size | 4 MB |
| String length in stored proc | 128,000 characters |
| Number of OR clauses in query | 20 |
| Number of AND clauses in query | 200 |

### 3.2 Change Feed - Advanced Details

**Change Feed Modes:**

| Mode | Description | Exam Notes |
|------|-------------|------------|
| **Latest version** (default) | Returns only the latest version of the changed item | Does NOT capture deletes |
| **All versions and deletes** | Returns all intermediate changes AND deletes | Requires continuous backup, captures deletes via `"metadata": {"operationType": "delete"}` |

**Change Feed Does NOT Capture (in Latest Version mode):**
- Deletes (items removed are NOT in the feed)
- Workaround: Use soft-delete pattern (set `isDeleted = true` and TTL)

```csharp
// Soft delete pattern for change feed
product.IsDeleted = true;
product.Ttl = 30 * 24 * 60 * 60; // TTL: 30 days to ensure change feed processes it
await container.UpsertItemAsync(product, new PartitionKey(product.PartitionKey));
```

**Change Feed Estimator:**
- Monitor the **lag** of your change feed processor
- Tells you how many changes are pending (not yet processed)

```csharp
// Build estimator
ChangeFeedEstimator estimator = container
    .GetChangeFeedEstimator(
        processorName: "myProcessor",
        leaseContainer: leaseContainer);

// Get estimation
using FeedIterator<ChangeFeedProcessorState> iterator = estimator.GetCurrentStateIterator();
while (iterator.HasMoreResults)
{
    FeedResponse<ChangeFeedProcessorState> states = await iterator.ReadNextAsync();
    foreach (ChangeFeedProcessorState state in states)
    {
        Console.WriteLine($"Partition: {state.LeaseToken}, Lag: {state.EstimatedLag}");
    }
}
```

**Change Feed Pull Model (alternative to processor):**
- Manually pull changes without a lease container
- More control over when and how to read changes

```csharp
// Pull model - read changes from a specific partition
FeedIterator<Product> iterator = container.GetChangeFeedIterator<Product>(
    ChangeFeedStartFrom.Beginning(),
    ChangeFeedMode.LatestVersion);

while (iterator.HasMoreResults)
{
    FeedResponse<Product> response = await iterator.ReadNextAsync();
    
    if (response.StatusCode == HttpStatusCode.NotModified)
    {
        // No new changes, wait and retry
        await Task.Delay(TimeSpan.FromSeconds(5));
    }
    else
    {
        foreach (Product item in response)
        {
            Console.WriteLine($"Changed: {item.Id}");
        }
    }
}
```

**Change Feed Processor Components:**

| Component | Purpose |
|-----------|---------|
| **Monitored container** | Container with the data that generates the change feed |
| **Lease container** | Stores state/checkpoints for each partition being processed |
| **Compute instance** | Hosts the change feed processor (e.g., VM, App Service, Azure Function) |
| **Delegate** | Your code that handles each batch of changes |

### 3.3 Azure Functions Cosmos DB Bindings

**Three Types of Bindings:**

#### **1. Cosmos DB Trigger (already covered - enhanced)**
```csharp
// Trigger: invoked when change feed has new changes
[FunctionName("ProcessChanges")]
public static void Run(
    [CosmosDBTrigger(
        databaseName: "myDatabase",
        containerName: "myContainer",
        Connection = "CosmosDBConnection",
        LeaseContainerName = "leases",
        CreateLeaseContainerIfNotExists = true,
        StartFromBeginning = false,        // false = only new changes
        MaxItemsPerInvocation = 100,       // Max items per batch
        FeedPollDelay = 5000)]             // Polling interval in ms
    IReadOnlyList<MyDocument> changes,
    ILogger log)
{
    foreach (var doc in changes)
    {
        log.LogInformation($"Modified: {doc.Id}");
    }
}
```

#### **2. Cosmos DB Input Binding (Read)**
```csharp
// Input binding: read a document by ID
[FunctionName("GetProduct")]
public static IActionResult Run(
    [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req,
    [CosmosDB(
        databaseName: "myDatabase",
        containerName: "myContainer",
        Connection = "CosmosDBConnection",
        Id = "{Query.id}",
        PartitionKey = "{Query.pk}")]
    Product product,
    ILogger log)
{
    if (product == null)
        return new NotFoundResult();
    
    return new OkObjectResult(product);
}

// Input binding: query for multiple documents
[FunctionName("GetProducts")]
public static IActionResult Run(
    [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req,
    [CosmosDB(
        databaseName: "myDatabase",
        containerName: "myContainer",
        Connection = "CosmosDBConnection",
        SqlQuery = "SELECT * FROM c WHERE c.category = {Query.category}")]
    IEnumerable<Product> products,
    ILogger log)
{
    return new OkObjectResult(products);
}
```

#### **3. Cosmos DB Output Binding (Write)**
```csharp
// Output binding: write a document
[FunctionName("CreateProduct")]
public static IActionResult Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    [CosmosDB(
        databaseName: "myDatabase",
        containerName: "myContainer",
        Connection = "CosmosDBConnection")]
    out dynamic document,
    ILogger log)
{
    string requestBody = new StreamReader(req.Body).ReadToEnd();
    document = JsonConvert.DeserializeObject(requestBody);
    document.id = Guid.NewGuid().ToString();
    
    return new OkObjectResult(document);
}

// Output binding: write multiple documents using IAsyncCollector
[FunctionName("CreateProducts")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    [CosmosDB(
        databaseName: "myDatabase",
        containerName: "myContainer",
        Connection = "CosmosDBConnection")]
    IAsyncCollector<Product> products,
    ILogger log)
{
    var product1 = new Product { Id = "1", Name = "Laptop" };
    var product2 = new Product { Id = "2", Name = "Mouse" };
    
    await products.AddAsync(product1);
    await products.AddAsync(product2);
    
    return new OkResult();
}
```

**Binding Summary:**

| Binding Type | Direction | Use Case |
|-------------|-----------|----------|
| Trigger | N/A (event) | React to change feed events |
| Input | In | Read documents by ID or SQL query |
| Output | Out | Write/Upsert documents |

### 3.4 Backup and Restore Policies

**Two Backup Modes:**

#### **1. Periodic Backup (Default)**
- Automatic full backups at regular intervals
- Backups stored in Azure Blob Storage (geo-redundant)
- **NOT** self-service restore → must contact Azure Support

| Setting | Default | Range |
|---------|---------|-------|
| Backup interval | 4 hours | 1 - 24 hours |
| Retention period | 8 hours | 8 hours - 30 days |
| Backup copies | 2 | 1 - unlimited |
| Storage redundancy | GRS | LRS, ZRS, GRS |

```bash
# Configure periodic backup
az cosmosdb update \
  --name <account> --resource-group <rg> \
  --backup-interval 240 \        # 4 hours (in minutes)
  --backup-retention 720 \       # 30 days (in hours)
  --backup-redundancy Geo
```

#### **2. Continuous Backup**
- **Point-in-time restore (PITR)**
- Self-service restore to any point in the last 7 or 30 days
- Can restore specific containers or entire database
- **No downtime** for restore operations
- Two tiers:
  - **Continuous 7 days**: Free
  - **Continuous 30 days**: Additional cost

```bash
# Create account with continuous backup
az cosmosdb create \
  --name <account> --resource-group <rg> \
  --backup-policy-type Continuous \
  --continuous-tier Continuous30Days

# Restore to point in time
az cosmosdb restore \
  --account-name <account> --resource-group <rg> \
  --target-database-account-name <restored-account> \
  --restore-timestamp "2026-02-07T10:00:00Z" \
  --location "eastus"
```

**Backup Mode Comparison:**

| Feature | Periodic | Continuous |
|---------|----------|-----------|
| Restore method | Contact support | Self-service |
| Restore granularity | Full account | Container/database level |
| Restore speed | Hours | Minutes |
| Data loss window | Up to backup interval | Seconds |
| Cost | Free (included) | Free (7-day) / Extra (30-day) |
| Migration | Can migrate TO continuous | Cannot go back to periodic |

### 3.5 Network Security

#### **IP Firewall**
- Configure allowed IP addresses for account access
- Block all public access or allow specific IPs

```bash
# Set IP firewall rules
az cosmosdb update \
  --name <account> --resource-group <rg> \
  --ip-range-filter "13.91.6.132,13.91.6.1/24"

# Allow Azure portal access (add Azure datacenter IPs)
az cosmosdb update \
  --name <account> --resource-group <rg> \
  --ip-range-filter "0.0.0.0"  # Allows all Azure services
```

#### **Virtual Network (VNet) Service Endpoints**
- Restrict access to specific VNet subnets
- Traffic stays on Azure backbone network

```bash
# Enable VNet service endpoint
az cosmosdb network-rule add \
  --name <account> --resource-group <rg> \
  --subnet <subnet-name> \
  --vnet-name <vnet-name> \
  --subnet-resource-group <subnet-rg>
```

#### **Private Endpoints (Azure Private Link)**
- Access Cosmos DB via a **private IP** in your VNet
- Traffic does NOT traverse public internet
- Highest level of network isolation

```bash
# Create private endpoint
az network private-endpoint create \
  --name <pe-name> --resource-group <rg> \
  --vnet-name <vnet> --subnet <subnet> \
  --private-connection-resource-id <cosmos-account-resource-id> \
  --group-id Sql \
  --connection-name myConnection
```

**Network Security Comparison:**

| Method | Isolation Level | Use Case |
|--------|----------------|----------|
| IP Firewall | Basic | Allow specific public IPs |
| VNet Service Endpoints | Medium | Restrict to VNet subnets |
| Private Endpoints | **Highest** | Full private network access |

### 3.6 RBAC & Security

#### **Control Plane vs Data Plane:**

| Plane | What it controls | Example |
|-------|-----------------|---------|
| **Control Plane** | Account/container management | Create database, configure throughput |
| **Data Plane** | Read/write data | CRUD operations on items |

#### **Data Plane RBAC (Built-in Roles):**

| Role | Permissions |
|------|------------|
| `Cosmos DB Built-in Data Reader` | Read-only access to data |
| `Cosmos DB Built-in Data Contributor` | Read + Write data |
| `Cosmos DB Built-in Data Owner` | Full data access + manage RBAC |

```bash
# Assign data plane role
az cosmosdb sql role assignment create \
  --account-name <account> --resource-group <rg> \
  --role-definition-id <role-definition-id> \
  --principal-id <aad-principal-id> \
  --scope "/"
```

```csharp
// Use Azure AD with RBAC (no keys!)
using Azure.Identity;

CosmosClient client = new CosmosClient(
    "https://<account>.documents.azure.com:443/",
    new DefaultAzureCredential());
// Requires one of the above RBAC roles assigned to the identity
```

#### **Key-Based Authentication:**
- **Primary/Secondary keys**: Full access (read + write)
- **Primary/Secondary read-only keys**: Read-only access
- Keys can be **regenerated** without downtime (use secondary while regenerating primary)

#### **Encryption:**

| Type | Default | Details |
|------|---------|---------|
| **Encryption at rest** | Always ON (free) | Microsoft-managed keys (service-managed) |
| **Customer-managed keys (CMK)** | Optional | Use Azure Key Vault to manage your own keys |
| **Encryption in transit** | Always ON | TLS 1.2 enforced |

### 3.7 Cosmos DB Emulator (Local Development)

**What is it?**
- Local emulator that simulates Azure Cosmos DB service
- **Free** for development and testing
- Supports SQL (NoSQL) API, MongoDB API, Table API, Gremlin API
- Runs on Windows, Linux (Docker), macOS (Docker)

**Key Details:**

| Feature | Details |
|---------|---------|
| Default endpoint | `https://localhost:8081` |
| Default key | `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==` |
| Data Explorer | `https://localhost:8081/_explorer/index.html` |
| Max containers | Unlimited (limited by local storage) |
| Consistency levels | All 5 supported (defaults to Session) |
| Multi-region | NOT supported (single-region only) |
| SSL Certificate | Self-signed (must trust or disable validation) |

```csharp
// Connect to emulator
CosmosClient client = new CosmosClient(
    "https://localhost:8081",
    "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==",
    new CosmosClientOptions
    {
        // Required for self-signed cert in emulator
        HttpClientFactory = () =>
        {
            HttpMessageHandler handler = new HttpClientHandler
            {
                ServerCertificateCustomValidationCallback = 
                    HttpClientHandler.DangerousAcceptAnyServerCertificateValidator
            };
            return new HttpClient(handler);
        },
        ConnectionMode = ConnectionMode.Gateway  // Emulator works best with Gateway
    });
```

```bash
# Run emulator via Docker
docker pull mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:latest

docker run -p 8081:8081 -p 10250-10255:10250-10255 \
  --name cosmos-emulator \
  -e AZURE_COSMOS_EMULATOR_PARTITION_COUNT=5 \
  -e AZURE_COSMOS_EMULATOR_ENABLE_DATA_PERSISTENCE=true \
  mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:latest
```

**⚠️ Exam Tip:** The emulator uses a well-known fixed key. Connection string format is the same as production — only the endpoint and key values change.

### 3.8 Integrated Cache & Dedicated Gateway

**Dedicated Gateway:**
- An in-memory caching layer in front of Cosmos DB
- Requires provisioning a **dedicated gateway** cluster
- Only works with **Gateway connection mode**

**Integrated Cache:**
- Built on top of the dedicated gateway
- Caches both **point reads** and **query results**
- Reduces RU consumption significantly for read-heavy workloads
- Cache entries have a configurable **TTL** (staleness tolerance)

| Feature | Details |
|---------|---------|
| Cache type | In-memory (within dedicated gateway) |
| Scope | Per-gateway node (not shared) |
| Consistency | Uses session or eventual only |
| Item cache | Caches point reads (by ID + partition key) |
| Query cache | Caches query results |
| RU savings | Cached reads cost **0 RU** from backend |

```csharp
// Connect through dedicated gateway for caching
CosmosClientOptions options = new CosmosClientOptions
{
    // Use dedicated gateway endpoint (not standard endpoint)
    ConnectionMode = ConnectionMode.Gateway,
    // Dedicated gateway connection string:
    // "AccountEndpoint=https://<account>.sqlx.cosmos.azure.com/;..."
};

// Requests are automatically cached
// Configure staleness tolerance per request:
ItemRequestOptions requestOptions = new ItemRequestOptions
{
    DedicatedGatewayRequestOptions = new DedicatedGatewayRequestOptions
    {
        MaxIntegratedCacheStaleness = TimeSpan.FromMinutes(5)
    }
};
```

### 3.9 Analytical Store & Azure Synapse Link

**What is Azure Synapse Link?**
- Cloud-native HTAP (Hybrid Transactional/Analytical Processing)
- No-ETL analytics over operational data in Cosmos DB
- Automatically syncs data from transactional store to **column-store** analytical store

**Analytical Store:**
- Fully isolated column-oriented store
- **Auto-synced** from transactional store (< 2 min latency)
- **No impact on transactional workload** (uses separate RUs)
- Optimized for analytical queries (aggregations, scans)
- Queried via Azure Synapse Analytics (Spark, SQL Serverless)

| Feature | Transactional Store | Analytical Store |
|---------|-------------------|-----------------|
| Format | Row-oriented (JSON) | Column-oriented |
| Optimized for | Point reads, writes | Analytical queries |
| Indexed | Yes (customizable) | Auto-indexed columns |
| RU impact | Yes | No (independent) |
| TTL | Configurable | Separate analytical TTL |

```bash
# Enable analytical store on container
az cosmosdb sql container create \
  --account-name <account> --resource-group <rg> \
  --database-name <db> --name <container> \
  --partition-key-path "/pk" \
  --analytical-storage-ttl -1  # -1 means infinite retention
```

**⚠️ Exam Tip:** Azure Synapse Link eliminates the need to build ETL pipelines for analytics. Know that it's column-store, auto-synced, and doesn't impact transactional RUs.

### 3.10 ARM Templates & Bicep for Cosmos DB

**ARM Template Example:**
```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.DocumentDB/databaseAccounts",
            "apiVersion": "2024-05-15",
            "name": "[parameters('accountName')]",
            "location": "[parameters('location')]",
            "kind": "GlobalDocumentDB",
            "properties": {
                "consistencyPolicy": {
                    "defaultConsistencyLevel": "Session"
                },
                "databaseAccountOfferType": "Standard",
                "locations": [
                    {
                        "locationName": "[parameters('location')]",
                        "failoverPriority": 0
                    }
                ],
                "enableAutomaticFailover": true,
                "enableMultipleWriteLocations": false,
                "backupPolicy": {
                    "type": "Continuous",
                    "continuousModeProperties": {
                        "tier": "Continuous30Days"
                    }
                }
            }
        }
    ]
}
```

**Bicep Example:**
```bicep
resource cosmosAccount 'Microsoft.DocumentDB/databaseAccounts@2024-05-15' = {
  name: accountName
  location: location
  kind: 'GlobalDocumentDB'
  properties: {
    consistencyPolicy: {
      defaultConsistencyLevel: 'Session'
    }
    databaseAccountOfferType: 'Standard'
    locations: [
      {
        locationName: location
        failoverPriority: 0
      }
    ]
    enableAutomaticFailover: true
    backupPolicy: {
      type: 'Continuous'
      continuousModeProperties: {
        tier: 'Continuous30Days'
      }
    }
  }
}

resource database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2024-05-15' = {
  parent: cosmosAccount
  name: databaseName
  properties: {
    resource: { id: databaseName }
    options: { throughput: 400 }
  }
}

resource container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2024-05-15' = {
  parent: database
  name: containerName
  properties: {
    resource: {
      id: containerName
      partitionKey: { paths: ['/partitionKey'], kind: 'Hash' }
      defaultTtl: 86400  // 1 day
      uniqueKeyPolicy: {
        uniqueKeys: [{ paths: ['/email'] }]
      }
    }
  }
}
```

**Key ARM/Bicep Resource Types:**

| Resource | Type |
|----------|------|
| Account | `Microsoft.DocumentDB/databaseAccounts` |
| SQL Database | `Microsoft.DocumentDB/databaseAccounts/sqlDatabases` |
| SQL Container | `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers` |
| Stored Procedure | `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/storedProcedures` |
| Trigger | `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/triggers` |
| UDF | `Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/userDefinedFunctions` |

### 3.11 Cosmos DB Capacity Calculator

- **Online tool** for estimating RU/s and cost: https://cosmos.azure.com/capacitycalculator/
- Input: number of documents, document size, reads/writes per second, regions
- Output: estimated RU/s needed, monthly cost

**Factors the Calculator Considers:**
- Document size (average)
- Number of reads per second
- Number of writes per second
- Number of regions (read/write)
- Indexing policy
- Consistency level chosen

### 3.12 Response Headers & Diagnostics

**Important Response Headers:**

| Header | Purpose | Exam Relevance |
|--------|---------|---------------|
| `x-ms-request-charge` | RU cost of the operation | Monitor cost per operation |
| `x-ms-activity-id` | Unique request identifier | Debugging with Azure support |
| `x-ms-session-token` | Session token for consistency | Pass to maintain session consistency |
| `x-ms-continuation` | Continuation token for paging | Used for paginated queries |
| `x-ms-retry-after-ms` | Wait time after 429 error | Implement proper back-off |
| `etag` | Entity tag for concurrency | Optimistic concurrency control |

```csharp
// Access diagnostics information
ItemResponse<Product> response = await container.CreateItemAsync(product, 
    new PartitionKey(product.PartitionKey));

// Request charge
double ruCost = response.RequestCharge;

// Activity ID (for support tickets)
string activityId = response.ActivityId;

// Session token (pass to other operations for session consistency)
string sessionToken = response.Headers.Session;

// Diagnostics (detailed breakdown)
CosmosDiagnostics diagnostics = response.Diagnostics;
Console.WriteLine(diagnostics.ToString());
// Shows: regions contacted, retries, latency breakdown, request timeline
```

### 3.13 Stream API (Low-Level Operations)

**When to use Stream API:**
- When you want to avoid deserialization overhead
- Custom serialization scenarios
- Maximum performance for high-throughput scenarios

```csharp
// Stream read (avoids deserialization)
using ResponseMessage response = await container.ReadItemStreamAsync(
    id: "item-id",
    partitionKey: new PartitionKey("pk"));

if (response.IsSuccessStatusCode)
{
    // Work with raw stream
    using StreamReader reader = new StreamReader(response.Content);
    string json = await reader.ReadToEndAsync();
    Console.WriteLine($"Raw JSON: {json}");
    Console.WriteLine($"RU: {response.Headers.RequestCharge}");
}
```

### 3.14 Cross-Partition Queries Detailed

**Behavior:**
- Queries without partition key in the WHERE clause → **fan-out** to ALL physical partitions
- Each partition is queried in parallel
- Results are merged client-side
- Costs significantly more RUs

```csharp
// Efficient: Single-partition query (includes partition key)
var query1 = new QueryDefinition("SELECT * FROM c WHERE c.userId = @uid AND c.status = 'active'")
    .WithParameter("@uid", "user123");
// This runs against ONE partition only

// Expensive: Cross-partition query (no partition key filter)
var query2 = new QueryDefinition("SELECT * FROM c WHERE c.status = 'active'");
// This fans out to ALL partitions

// Configure parallelism for cross-partition queries
QueryRequestOptions options = new QueryRequestOptions
{
    MaxConcurrency = -1,            // Maximum parallelism
    MaxBufferedItemCount = 100,     // Buffer size
    // NOTE: No PartitionKey specified = cross-partition
};
```

**Performance Tips:**
- Always include partition key in WHERE clause when possible
- Use `MaxConcurrency = -1` for parallel fan-out
- Use `MaxBufferedItemCount` to control memory usage
- Prefer point reads (`ReadItemAsync`) over queries for single items

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**1. Cosmos DB Architecture:**
- Resource hierarchy (Account → Database → Container → Item)
- APIs and when to use each
- Partitioning (logical vs physical)
- Choosing partition keys
- System properties (_rid, _self, _etag, _ts)
- Item size limit (2 MB), partition key value limit (2 KB)

**2. Consistency Levels:**
- All five levels and their guarantees
- Trade-offs between consistency, availability, latency
- Default vs request-level override (can only WEAKEN)
- Strong not available with multi-region writes

**3. Request Units:**
- What affects RU consumption
- Throughput provisioning models (Provisioned, Autoscale, Serverless)
- Shared vs dedicated throughput
- Capacity calculator for estimation

**4. SDK Operations:**
- CosmosClient, Database, Container classes
- CRUD operations (Create, Read, Replace, Upsert, Patch, Delete)
- Queries (SQL and LINQ)
- Transactional batch
- **Connection modes (Direct vs Gateway)**
- **CosmosClientOptions (regions, bulk, retries)**
- **Bulk operations vs Transactional batch**
- **Stream API for performance**

**5. Server-Side Programming:**
- Stored procedures (bounded execution, 5-second limit, continuation pattern)
- Triggers (pre and post)
- User-defined functions
- **All run within single partition key scope**

**6. TTL (Time-to-Live):**
- Container-level vs item-level TTL
- -1 means no expiry, null means inherit from container
- Container TTL must be enabled FIRST before item TTL works

**7. Data Modeling:**
- Embedding vs referencing (denormalization vs normalization)
- When to embed: one-to-few, read together, fits in 2 MB
- When to reference: one-to-many unbounded, updated independently

**8. Change Feed:**
- Change feed processor (monitored container, lease container)
- Change feed estimator (lag monitoring)
- Pull model vs processor
- Latest version mode (no deletes) vs all versions and deletes mode
- Soft-delete pattern workaround

**9. Azure Functions Bindings:**
- Trigger (change feed)
- Input binding (read by ID or SQL query)
- Output binding (write/upsert documents)

**10. Security & Networking:**
- RBAC built-in roles (Reader, Contributor, Owner)
- Key-based vs Azure AD authentication
- IP Firewall, VNet Service Endpoints, Private Endpoints
- Encryption at rest (always on), customer-managed keys
- Control plane vs data plane

**11. Backup & Restore:**
- Periodic (default, contact support) vs Continuous (self-service PITR)
- Continuous 7-day (free) vs 30-day (paid)

**12. Advanced Features:**
- Unique keys (per logical partition, immutable after creation)
- Integrated cache & dedicated gateway
- Analytical store & Azure Synapse Link (column-store, no-ETL)
- Cosmos DB Emulator (local dev, fixed key)
- ARM/Bicep provisioning

### Sample Exam Questions

#### Cosmos DB Fundamentals

**Q1:** What is the maximum size of a logical partition in Azure Cosmos DB?
- A) 10 GB
- B) 20 GB
- C) 50 GB
- D) Unlimited

**Answer: B) 20 GB**
Explanation: A logical partition can store up to 20 GB of data. Physical partitions can hold up to 50 GB and contain multiple logical partitions.

---

**Q2:** Which API should you choose for a new application with no existing database requirements that needs complex query capabilities?
- A) API for MongoDB
- B) API for Cassandra
- C) API for NoSQL (SQL API)
- D) API for Table

**Answer: C) API for NoSQL (SQL API)**
Explanation: The API for NoSQL is recommended for new applications as it provides the best performance, all features first, and supports SQL-like queries.

---

**Q3:** You cannot change the API type after creating a Cosmos DB account. True or False?
- A) True
- B) False

**Answer: A) True**
Explanation: The API type is selected at account creation and cannot be changed afterward. You would need to create a new account with the desired API.

---

#### Partitioning

**Q4:** Which of the following is NOT a characteristic of a good partition key?
- A) High cardinality
- B) Even distribution of data
- C) Frequently used in WHERE clauses
- D) Low cardinality with few distinct values

**Answer: D) Low cardinality with few distinct values**
Explanation: A good partition key should have high cardinality (many distinct values) to distribute data evenly. Low cardinality leads to hot partitions.

---

**Q5:** You have an e-commerce application with millions of customers. Which partition key would be BEST for a container storing customer orders?
- A) `/orderStatus`
- B) `/customerId`
- C) `/orderDate`
- D) `/country`

**Answer: B) `/customerId`**
Explanation: CustomerId has high cardinality, distributes data evenly, and most queries would filter by customer. OrderStatus has low cardinality, orderDate causes hot partitions on current date, and country has limited values.

---

**Q6:** What happens when a logical partition exceeds 20 GB?
- A) Data is automatically split across partitions
- B) Writes to that partition fail with an error
- C) The partition key must be changed
- D) Additional costs are charged

**Answer: B) Writes to that partition fail with an error**
Explanation: If a logical partition reaches 20 GB, further writes to that partition will fail. This is why choosing a partition key with high cardinality is critical.

---

#### Consistency Levels

**Q7:** Which consistency level is the DEFAULT for new Cosmos DB accounts?
- A) Strong
- B) Bounded Staleness
- C) Session
- D) Eventual

**Answer: C) Session**
Explanation: Session consistency is the default because it offers a good balance between consistency and performance for most applications.

---

**Q8:** You need guaranteed linearizable reads where every read returns the most recent write. Which consistency level should you use?
- A) Session
- B) Consistent Prefix
- C) Bounded Staleness
- D) Strong

**Answer: D) Strong**
Explanation: Strong consistency guarantees linearizable reads, meaning reads always return the most recently committed write globally.

---

**Q9:** Which consistency level guarantees that reads never see out-of-order writes but may be arbitrarily stale?
- A) Eventual
- B) Consistent Prefix
- C) Session
- D) Strong

**Answer: B) Consistent Prefix**
Explanation: Consistent Prefix guarantees that if writes are A, B, C, reads will see A, A-B, or A-B-C, but never out of order (like B-A). However, there's no staleness guarantee.

---

**Q10:** Your application needs "read your own writes" guarantee within a user session. Which is the MINIMUM consistency level required?
- A) Strong
- B) Session
- C) Consistent Prefix
- D) Eventual

**Answer: B) Session**
Explanation: Session consistency guarantees read-your-writes, monotonic reads, and monotonic writes within a single session.

---

**Q11:** Strong consistency is available for multi-region write configurations. True or False?
- A) True
- B) False

**Answer: B) False**
Explanation: Strong consistency is NOT available when multi-region writes are enabled. You must use Bounded Staleness or weaker for multi-region write scenarios.

---

#### Request Units

**Q12:** What is the approximate RU cost for a point read of a 1 KB item?
- A) 0.5 RU
- B) 1 RU
- C) 5 RU
- D) 10 RU

**Answer: B) 1 RU**
Explanation: A point read (read by ID and partition key) of a 1 KB item costs approximately 1 RU.

---

**Q13:** Which throughput model allows you to scale between 10% and 100% of maximum RU/s automatically?
- A) Standard Provisioned
- B) Autoscale
- C) Serverless
- D) Shared Throughput

**Answer: B) Autoscale**
Explanation: Autoscale automatically adjusts throughput between 10% and 100% of the configured maximum, based on workload demand.

---

**Q14:** What is the minimum provisioned throughput for a container with dedicated throughput?
- A) 100 RU/s
- B) 400 RU/s
- C) 1000 RU/s
- D) 4000 RU/s

**Answer: B) 400 RU/s**
Explanation: The minimum throughput for a container with dedicated throughput is 400 RU/s, with increments of 100 RU/s.

---

**Q15:** Which factor does NOT affect the RU cost of an operation?
- A) Item size
- B) Consistency level
- C) Container name
- D) Query complexity

**Answer: C) Container name**
Explanation: Container name doesn't affect RU cost. Item size, consistency level (Strong costs 2x for reads), and query complexity all impact RU consumption.

---

#### SDK Operations

**Q16:** Which class is the entry point for all Cosmos DB operations in the .NET SDK?
- A) Database
- B) Container
- C) CosmosClient
- D) DocumentClient

**Answer: C) CosmosClient**
Explanation: CosmosClient is the main entry point. DocumentClient was used in the older V2 SDK.

---

**Q17:** What is the recommended pattern for CosmosClient instantiation?
- A) Create new instance for each operation
- B) Use singleton pattern
- C) Create instance per container
- D) Create instance per database

**Answer: B) Use singleton pattern**
Explanation: CosmosClient is thread-safe and expensive to create. It should be created once and reused throughout the application lifetime.

---

**Q18:** Which method should you use to insert an item that may or may not already exist?
- A) CreateItemAsync
- B) ReplaceItemAsync
- C) UpsertItemAsync
- D) PatchItemAsync

**Answer: C) UpsertItemAsync**
Explanation: UpsertItemAsync creates the item if it doesn't exist, or replaces it if it does. CreateItemAsync throws an error if the item exists.

---

**Q19:** You want to update only specific properties of an item without replacing the entire document. Which method should you use?
- A) ReplaceItemAsync
- B) UpsertItemAsync
- C) PatchItemAsync
- D) UpdateItemAsync

**Answer: C) PatchItemAsync**
Explanation: PatchItemAsync allows partial updates to specific properties using patch operations (Add, Remove, Replace, Set, Increment).

---

**Q20:** What HTTP status code indicates rate limiting in Cosmos DB?
- A) 400
- B) 404
- C) 429
- D) 500

**Answer: C) 429**
Explanation: HTTP 429 (Too Many Requests) indicates that the provisioned throughput has been exceeded. The SDK automatically retries with backoff.

---

#### Server-Side Programming

**Q21:** What is the scope of a transaction in a Cosmos DB stored procedure?
- A) Entire container
- B) Single partition key
- C) Entire database
- D) Entire account

**Answer: B) Single partition key**
Explanation: Stored procedures run within the scope of a single logical partition. All operations in the procedure must use the same partition key.

---

**Q22:** Which type of trigger runs BEFORE an operation is committed?
- A) Pre-trigger
- B) Post-trigger
- C) Before-trigger
- D) Commit-trigger

**Answer: A) Pre-trigger**
Explanation: Pre-triggers execute before the operation and can validate or modify the incoming data. Post-triggers execute after the operation.

---

**Q23:** How are User-Defined Functions (UDFs) used in Cosmos DB?
- A) As standalone procedures
- B) Within SQL queries
- C) As triggers
- D) For authentication

**Answer: B) Within SQL queries**
Explanation: UDFs are JavaScript functions that can be called within SQL queries to extend query capabilities (e.g., `SELECT udf.myFunction(c.field) FROM c`).

---

#### Advanced Topics

**Q24:** What is the maximum number of operations in a transactional batch?
- A) 10
- B) 50
- C) 100
- D) 1000

**Answer: C) 100**
Explanation: A transactional batch can contain up to 100 operations, and the total size cannot exceed 2 MB.

---

**Q25:** Which feature enables real-time event-driven processing of data changes in Cosmos DB?
- A) Triggers
- B) Change Feed
- C) Stored Procedures
- D) Materialized Views

**Answer: B) Change Feed**
Explanation: Change Feed provides a persistent, ordered record of changes that enables real-time event-driven architectures, integrations, and data synchronization.

---

**Q26:** What is the default indexing behavior in Cosmos DB?
- A) No indexing
- B) Index only specified properties
- C) Index all properties automatically
- D) Index only the partition key

**Answer: C) Index all properties automatically**
Explanation: By default, Cosmos DB automatically indexes all properties in all documents without requiring schema or secondary indexes.

---

**Q27:** You need to ORDER BY multiple properties in a query. What must you configure?
- A) Range index
- B) Composite index
- C) Spatial index
- D) No additional configuration needed

**Answer: B) Composite index**
Explanation: ORDER BY on multiple properties requires a composite index defined in the indexing policy.

---

#### Global Distribution

**Q28:** What is the availability SLA for a multi-region Cosmos DB account with automatic failover?
- A) 99.9%
- B) 99.95%
- C) 99.99%
- D) 99.999%

**Answer: D) 99.999%**
Explanation: Multi-region accounts with automatic failover provide 99.999% availability SLA (five nines).

---

**Q29:** What is the default conflict resolution policy for multi-region writes?
- A) First Writer Wins
- B) Last Writer Wins
- C) Manual Resolution
- D) No conflict resolution

**Answer: B) Last Writer Wins**
Explanation: The default conflict resolution policy is Last Writer Wins (LWW) based on the `_ts` timestamp property.

---

**Q30:** Which consistency level is NOT available when multi-region writes are enabled?
- A) Session
- B) Consistent Prefix
- C) Strong
- D) Eventual

**Answer: C) Strong**
Explanation: Strong consistency requires all writes to be synchronously replicated, which is not compatible with multi-region writes. Bounded Staleness is the strongest available.

---

#### TTL, Data Modeling & Unique Keys

**Q31:** A container has `DefaultTimeToLive` set to 100 seconds. An item in the container has `ttl` set to -1. What happens to this item?
- A) Item expires in 100 seconds
- B) Item never expires
- C) Item expires immediately
- D) Item inherits the container default

**Answer: B) Item never expires**
Explanation: When an item has `ttl = -1`, it overrides the container default and will never expire, even though the container default TTL is 100 seconds.

---

**Q32:** A container does NOT have TTL enabled (DefaultTimeToLive is null/not set). An item in the container has `ttl` set to 60. What happens?
- A) Item expires in 60 seconds
- B) Item never expires
- C) An error occurs on write
- D) Item expires based on account default

**Answer: B) Item never expires**
Explanation: Container-level TTL must be ENABLED first before any item-level TTL takes effect. If the container TTL is not set, all items live forever regardless of their individual `ttl` property.

---

**Q33:** When should you use EMBEDDING (denormalization) vs REFERENCING in Cosmos DB data modeling?
- A) Embed when data is one-to-many unbounded; reference when data is one-to-few
- B) Embed when data is read together and one-to-few; reference when one-to-many unbounded
- C) Always embed because NoSQL databases don't support joins
- D) Always reference for normalized data

**Answer: B) Embed when data is read together and one-to-few; reference when one-to-many unbounded**
Explanation: In Cosmos DB, embed related data when it's read together, the relationship is one-to-few, and the total fits within the 2 MB item limit. Reference when relationships are unbounded or data changes independently.

---

**Q34:** You create a container with a unique key policy on `/email`. Two items with the email "john@test.com" are stored. The first item has partitionKey = "tenant1" and the second has partitionKey = "tenant2". What happens?
- A) The second insert fails with a conflict error
- B) Both items are stored successfully
- C) The unique key is ignored
- D) The second item overwrites the first

**Answer: B) Both items are stored successfully**
Explanation: Unique keys are scoped to the logical partition. Since the items are in different partitions ("tenant1" and "tenant2"), the same email is allowed in both.

---

**Q35:** When can you modify the unique key policy on a container?
- A) Anytime via the Azure Portal
- B) Using the SDK after deleting all items
- C) Never — it is immutable after container creation
- D) Only during a maintenance window

**Answer: C) Never — it is immutable after container creation**
Explanation: Unique key policies are set at container creation and cannot be changed afterward. You would need to create a new container with the desired policy.

---

#### Connection Modes, SDK & Bulk Operations

**Q36:** What is the DEFAULT connection mode in the Azure Cosmos DB .NET SDK v3?
- A) Gateway
- B) Direct
- C) Hybrid
- D) TCP-only

**Answer: B) Direct**
Explanation: Direct mode (TCP protocol) is the default in the .NET SDK v3. It provides lower latency by connecting directly to backend replicas. Gateway mode uses HTTPS and is better for restricted network environments.

---

**Q37:** When should you use Gateway connection mode instead of Direct mode?
- A) When you need the lowest possible latency
- B) When you're behind a firewall that only allows HTTPS (port 443)
- C) When running in production
- D) When using read-only keys

**Answer: B) When you're behind a firewall that only allows HTTPS (port 443)**
Explanation: Gateway mode uses only HTTPS (port 443), making it suitable for restricted network environments. Direct mode requires a range of TCP ports (10000-20000).

---

**Q38:** You need to ingest 1 million documents into Cosmos DB as fast as possible. Which approach should you use?
- A) Loop through and call `CreateItemAsync` for each item
- B) Use `TransactionalBatch` for all items
- C) Enable `AllowBulkExecution = true` and use concurrent `CreateItemAsync` calls
- D) Use a stored procedure

**Answer: C) Enable `AllowBulkExecution = true` and use concurrent `CreateItemAsync` calls**
Explanation: Bulk execution groups concurrent operations into optimized batches for maximum throughput. Transactional batch is limited to 100 operations in a single partition. Sequential CreateItemAsync is too slow.

---

**Q39:** What is the recommended pattern for `CosmosClient` lifecycle management?
- A) Create and dispose per operation
- B) Create one per container
- C) Use singleton pattern (one instance for the application lifetime)
- D) Create a new one per HTTP request

**Answer: C) Use singleton pattern (one instance for the application lifetime)**
Explanation: CosmosClient is thread-safe and expensive to create. It manages connections internally and should be reused as a singleton throughout the application.

---

**Q40:** Which `CosmosClientOptions` property configures the ordered list of preferred read regions?
- A) `ApplicationRegion`
- B) `ApplicationPreferredRegions`
- C) `ReadRegions`
- D) `PreferredLocations`

**Answer: B) `ApplicationPreferredRegions`**
Explanation: `ApplicationPreferredRegions` takes a list of regions in order of preference. `ApplicationRegion` is for a single preferred region. You should use one or the other, not both.

---

#### Server-Side Programming

**Q41:** What is the maximum execution time for a stored procedure in Cosmos DB?
- A) 1 second
- B) 5 seconds
- C) 30 seconds
- D) No limit

**Answer: B) 5 seconds**
Explanation: Stored procedures have a 5-second execution time limit (bounded execution). If operations don't complete within this time, the transaction is rolled back.

---

**Q42:** In a stored procedure, the `container.createDocument()` method returns a boolean. What does `false` mean?
- A) The document was not valid
- B) The document already exists
- C) The operation was NOT accepted due to resource budget being exceeded
- D) The partition key was wrong

**Answer: C) The operation was NOT accepted due to resource budget being exceeded**
Explanation: When the resource budget (time or RU) is about to be exceeded, the acceptance callback returns false. The stored procedure should save its progress and return, allowing the client to continue with a new invocation.

---

#### Change Feed

**Q43:** The change feed (latest version mode) captures which of the following operations?
- A) Creates and updates only
- B) Creates, updates, and deletes
- C) Only creates
- D) Only updates

**Answer: A) Creates and updates only**
Explanation: In latest version mode (default), the change feed captures inserts and updates but NOT deletes. To capture deletes, use the "all versions and deletes" mode or implement a soft-delete pattern.

---

**Q44:** What is the purpose of the **lease container** in the change feed processor?
- A) Store the actual data changes
- B) Store processing state/checkpoints for each partition
- C) Store backup copies of changed documents
- D) Store conflict resolution logs

**Answer: B) Store processing state/checkpoints for each partition**
Explanation: The lease container tracks the progress (checkpoint) of the change feed processor for each partition. This enables resuming from where processing left off after a restart.

---

**Q45:** Which component monitors how many changes are pending (not yet processed) in the change feed?
- A) Change Feed Processor
- B) Change Feed Estimator
- C) Change Feed Monitor
- D) Lease Container

**Answer: B) Change Feed Estimator**
Explanation: The Change Feed Estimator reports the lag (number of unprocessed changes) for each partition, helping you monitor whether your processor is keeping up with changes.

---

#### Azure Functions Bindings

**Q46:** Which Cosmos DB Azure Functions binding allows you to READ a document by its ID and partition key?
- A) Trigger binding
- B) Input binding
- C) Output binding
- D) Query binding

**Answer: B) Input binding**
Explanation: The input binding reads documents from Cosmos DB. You can specify an ID and partition key for a point read, or a SQL query for multiple documents. The trigger binding reacts to change feed events.

---

**Q47:** In an Azure Function with Cosmos DB output binding, what operation does the SDK perform?
- A) CreateItemAsync
- B) ReplaceItemAsync
- C) UpsertItemAsync
- D) PatchItemAsync

**Answer: C) UpsertItemAsync**
Explanation: The Cosmos DB output binding performs an upsert — it creates the document if it doesn't exist, or replaces it if it does. This is by design to handle both new and existing documents.

---

#### Backup, Security & Networking

**Q48:** Which backup mode supports self-service point-in-time restore (PITR)?
- A) Periodic backup
- B) Continuous backup
- C) Snapshot backup
- D) Geo-redundant backup

**Answer: B) Continuous backup**
Explanation: Continuous backup enables self-service point-in-time restore. Periodic backup requires contacting Azure support for restoration.

---

**Q49:** Which RBAC role should you assign to allow an application to read AND write data, but NOT manage RBAC permissions?
- A) Cosmos DB Built-in Data Reader
- B) Cosmos DB Built-in Data Contributor
- C) Cosmos DB Built-in Data Owner
- D) Contributor

**Answer: B) Cosmos DB Built-in Data Contributor**
Explanation: Data Contributor allows read+write data operations. Data Reader is read-only. Data Owner includes managing RBAC. The "Contributor" role is a control-plane role, not data-plane.

---

**Q50:** Which network security method provides the HIGHEST level of isolation for Cosmos DB?
- A) IP Firewall rules
- B) VNet Service Endpoints
- C) Private Endpoints (Azure Private Link)
- D) Network Security Groups

**Answer: C) Private Endpoints (Azure Private Link)**
Explanation: Private Endpoints assign a private IP from your VNet to the Cosmos DB account. Traffic never traverses the public internet, providing the highest isolation level.

---

**Q51:** Is data in Cosmos DB encrypted at rest by default?
- A) No, you must enable it manually
- B) Yes, with Microsoft-managed keys
- C) Only if you use a Premium tier
- D) Only for multi-region accounts

**Answer: B) Yes, with Microsoft-managed keys**
Explanation: Azure Cosmos DB encrypts all data at rest by default using Microsoft-managed (service-managed) keys. You can optionally use customer-managed keys (CMK) via Azure Key Vault.

---

#### Advanced Features

**Q52:** You connect your application to the Cosmos DB Emulator for local development. What is the default endpoint?
- A) `http://localhost:8080`
- B) `https://localhost:8081`
- C) `http://localhost:10255`
- D) `https://localhost:443`

**Answer: B) `https://localhost:8081`**
Explanation: The Cosmos DB Emulator runs on `https://localhost:8081` by default with a well-known fixed authentication key.

---

**Q53:** What does Azure Synapse Link for Cosmos DB provide?
- A) Real-time replication to SQL Database
- B) No-ETL analytics over operational data using a column-store analytical store
- C) Automatic backup to Synapse storage
- D) SQL query support for MongoDB API

**Answer: B) No-ETL analytics over operational data using a column-store analytical store**
Explanation: Azure Synapse Link auto-syncs transactional data to a column-oriented analytical store that can be queried via Azure Synapse Analytics without impacting transactional workload RUs.

---

**Q54:** What does the integrated cache in Cosmos DB cache?
- A) Only query results
- B) Only point reads
- C) Both point reads and query results
- D) Only stored procedure results

**Answer: C) Both point reads and query results**
Explanation: The integrated cache (via dedicated gateway) caches both point reads and query results in memory, reducing backend RU consumption. Cached reads cost 0 RUs from the backend.

---

**Q55:** Which system property is used as the default conflict resolution path in Last Writer Wins (LWW) policy?
- A) `_rid`
- B) `_etag`
- C) `_ts`
- D) `id`

**Answer: C) `_ts`**
Explanation: The `_ts` (timestamp) property records when a document was last modified (Unix epoch seconds). By default, LWW uses `_ts` to determine which write wins in a conflict.

---

**Q56:** What is the maximum size of a single item (document) in Cosmos DB SQL API?
- A) 1 MB
- B) 2 MB
- C) 4 MB
- D) 10 MB

**Answer: B) 2 MB**
Explanation: The maximum item size in Cosmos DB SQL (NoSQL) API is 2 MB. If your data exceeds this, you need to split it across multiple items or use referencing patterns.

---

**Q57:** You have a stored procedure that needs to process 100,000 items. What pattern should you use?
- A) Increase the stored procedure timeout to 60 seconds
- B) Use bounded execution with continuation tokens (process items in batches across multiple invocations)
- C) Use async/await inside the stored procedure
- D) Process all items in a single stored procedure call

**Answer: B) Use bounded execution with continuation tokens (process items in batches across multiple invocations)**
Explanation: Stored procedures have a 5-second time limit. For large datasets, check the `accepted` return value, save progress, and call the stored procedure again from the client to continue.

---

**Q58:** Which of the following statements about the Cosmos DB change feed is TRUE?
- A) The change feed is push-only
- B) The change feed provides changes in order of modification time within each logical partition
- C) The change feed includes deleted items by default
- D) The change feed requires a Premium tier account

**Answer: B) The change feed provides changes in order of modification time within each logical partition**
Explanation: Within each logical partition, changes are in order. Across partitions, there is no guaranteed ordering. Deletes are NOT included by default (only in "all versions and deletes" mode).

---

**Q59:** You set `MaxRetryAttemptsOnRateLimitedRequests = 0` in CosmosClientOptions. What happens when you get a 429 error?
- A) The SDK retries once
- B) The SDK retries 9 times (default)
- C) The exception is thrown immediately to the caller
- D) The SDK waits indefinitely

**Answer: C) The exception is thrown immediately to the caller**
Explanation: Setting retries to 0 disables automatic retry for rate-limited (429) requests. The CosmosException is thrown immediately, and the caller must handle it.

---

**Q60:** Which throughput mode is single-region only and does NOT provide an SLA?
- A) Provisioned Standard
- B) Provisioned Autoscale
- C) Serverless
- D) Shared Database throughput

**Answer: C) Serverless**
Explanation: Serverless mode is limited to a single region, has a maximum burst of 5,000 RU/s, and does not provide an availability SLA. It's best for development/test or sporadic workloads.

---

## Key CLI Commands Reference

### Account Management
```bash
# Create Cosmos DB account
az cosmosdb create \
  --name <account-name> \
  --resource-group <rg> \
  --locations regionName=eastus failoverPriority=0 \
  --default-consistency-level Session \
  --enable-automatic-failover true

# Add region
az cosmosdb update \
  --name <account-name> \
  --resource-group <rg> \
  --locations regionName=eastus failoverPriority=0 \
  --locations regionName=westus failoverPriority=1

# Enable multi-region writes
az cosmosdb update \
  --name <account-name> \
  --resource-group <rg> \
  --enable-multiple-write-locations true

# Get connection string
az cosmosdb keys list \
  --name <account-name> \
  --resource-group <rg> \
  --type connection-strings
```

### Database Operations
```bash
# Create database
az cosmosdb sql database create \
  --account-name <account-name> \
  --resource-group <rg> \
  --name <database-name>

# Create database with shared throughput
az cosmosdb sql database create \
  --account-name <account-name> \
  --resource-group <rg> \
  --name <database-name> \
  --throughput 400

# List databases
az cosmosdb sql database list \
  --account-name <account-name> \
  --resource-group <rg>
```

### Container Operations
```bash
# Create container
az cosmosdb sql container create \
  --account-name <account-name> \
  --resource-group <rg> \
  --database-name <database-name> \
  --name <container-name> \
  --partition-key-path "/partitionKey" \
  --throughput 400

# Create container with autoscale
az cosmosdb sql container create \
  --account-name <account-name> \
  --resource-group <rg> \
  --database-name <database-name> \
  --name <container-name> \
  --partition-key-path "/partitionKey" \
  --max-throughput 4000  # Autoscale max

# Update throughput
az cosmosdb sql container throughput update \
  --account-name <account-name> \
  --resource-group <rg> \
  --database-name <database-name> \
  --name <container-name> \
  --throughput 1000

# Migrate to autoscale
az cosmosdb sql container throughput migrate \
  --account-name <account-name> \
  --resource-group <rg> \
  --database-name <database-name> \
  --name <container-name> \
  --throughput-type autoscale
```

### Stored Procedures
```bash
# Create stored procedure
az cosmosdb sql stored-procedure create \
  --account-name <account-name> \
  --resource-group <rg> \
  --database-name <database-name> \
  --container-name <container-name> \
  --name <sproc-name> \
  --body @sproc.js
```

---

## Study Tips for AZ-204 Exam

1. **Understand partition key selection** - Critical and commonly tested (high cardinality, even distribution)
2. **Know all five consistency levels** - Trade-offs, use cases, and which ones are not available with multi-region writes
3. **Master SDK operations** - CRUD, queries, error handling, connection modes, CosmosClientOptions
4. **Practice RU estimation** - Know what factors affect RU consumption and use the capacity calculator
5. **Understand server-side programming** - Stored procedures (bounded execution, 5s limit), triggers, UDFs
6. **Know throughput models** - Provisioned vs Autoscale vs Serverless (know limits of each)
7. **Global distribution concepts** - Multi-region, failover, conflict resolution (LWW, custom)
8. **Change feed mastery** - Processor, estimator, pull model, what it captures vs doesn't (deletes!)
9. **Indexing policies** - Default behavior, composite indexes for multi-ORDER BY, spatial indexes
10. **TTL configuration** - Container vs item level, -1 behavior, null behavior, container must be enabled first
11. **Data modeling** - Embedding vs referencing, when to use each, 2 MB item limit
12. **Security** - RBAC roles (Reader, Contributor, Owner), encryption, network security options
13. **Azure Functions bindings** - Know Trigger, Input, and Output bindings for Cosmos DB
14. **Backup policies** - Periodic vs Continuous, self-service PITR for continuous
15. **Unique keys** - Per logical partition, immutable after creation
16. **Emulator** - Default endpoint, well-known key, used for local development
17. **Hands-on practice** - Use the Azure portal, .NET SDK, CLI, and Emulator

---

## Important URLs and References

- Azure Cosmos DB Documentation: https://learn.microsoft.com/en-us/azure/cosmos-db/
- Cosmos DB .NET SDK: https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/sdk-dotnet-v3
- Partitioning Best Practices: https://learn.microsoft.com/en-us/azure/cosmos-db/partitioning-overview
- Consistency Levels: https://learn.microsoft.com/en-us/azure/cosmos-db/consistency-levels
- Request Units: https://learn.microsoft.com/en-us/azure/cosmos-db/request-units
- Change Feed: https://learn.microsoft.com/en-us/azure/cosmos-db/change-feed
- TTL Configuration: https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/time-to-live
- Indexing Policies: https://learn.microsoft.com/en-us/azure/cosmos-db/index-policy
- Cosmos DB Emulator: https://learn.microsoft.com/en-us/azure/cosmos-db/emulator
- Backup and Restore: https://learn.microsoft.com/en-us/azure/cosmos-db/online-backup-and-restore
- Capacity Calculator: https://cosmos.azure.com/capacitycalculator/
- AZ-204 Official Study Guide: https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-204

---

## ⚠️ EXAM GOTCHAS & COMMONLY MISSED CONCEPTS

These are the concepts that trip people up on the actual exam. **Read carefully.**

### Gotcha 1: Triggers are NOT Automatically Invoked

- Triggers do **NOT** fire automatically when you write to a container
- You must **explicitly specify** the trigger name in `ItemRequestOptions` for every operation
- If you don't pass the trigger name, it simply won't run — no error, no warning

```csharp
// ❌ WRONG — trigger will NOT fire
await container.CreateItemAsync(item, new PartitionKey("pk"));

// ✅ CORRECT — trigger fires because explicitly specified
ItemRequestOptions options = new ItemRequestOptions
{
    PreTriggers = new[] { "validateItem" },
    PostTriggers = new[] { "createAuditLog" }
};
await container.CreateItemAsync(item, new PartitionKey("pk"), options);
```

### Gotcha 2: Server-Side Programming is JavaScript ONLY (SQL API)

- Stored procedures, triggers, and UDFs for the **SQL (NoSQL) API** must be written in **JavaScript only**
- No C#, Python, or other languages for server-side code
- They run in a **V8 JavaScript engine** within the Cosmos DB service
- For the MongoDB API: server-side code uses MongoDB's native JavaScript functions
- For Cassandra API: no custom server-side programming

### Gotcha 3: Container Properties — What's Mutable vs Immutable

| Property | Can Change After Creation? |
|----------|--------------------------|
| Partition key | ❌ **No** (immutable forever) |
| Unique key policy | ❌ **No** (immutable forever) |
| API type (account-level) | ❌ **No** (immutable forever) |
| Throughput (RU/s) | ✅ Yes |
| Indexing policy | ✅ Yes (updated in background, no downtime) |
| Default TTL | ✅ Yes |
| Conflict resolution policy | ✅ Yes |
| Stored procedures/triggers/UDFs | ✅ Yes (add/update/delete anytime) |
| Container name | ❌ **No** |

**Exam trap:** If a question asks "you chose the wrong partition key, what should you do?" → Answer: Create a NEW container with the correct partition key and migrate data.

### Gotcha 4: The `id` Property Rules

- **Required** in every document (if not provided, SDK auto-generates a GUID)
- **Maximum 255 characters**
- **Case-sensitive** (`"ABC"` ≠ `"abc"`)
- **Cannot contain**: `\`, `/`, `?`, `#` characters
- **Unique within a logical partition** (same `id` CAN exist in different partitions)
- Combined with partition key = globally unique identifier for the item

### Gotcha 5: Consistency Level Override — Can Only WEAKEN

- Account default consistency can be overridden at the **request level**
- But you can only **weaken** it, never **strengthen** it
- Example: If account default is `Session`, you can override to `Eventual` but NOT to `Strong`

```csharp
// Account default: Session
// ✅ Valid: weaken to Eventual
ItemRequestOptions options = new ItemRequestOptions
{
    ConsistencyLevel = ConsistencyLevel.Eventual  // Weaker than Session
};

// ❌ Invalid: cannot strengthen to Strong
ItemRequestOptions options2 = new ItemRequestOptions
{
    ConsistencyLevel = ConsistencyLevel.Strong  // Stronger than Session — NOT ALLOWED
};
```

### Gotcha 6: Partition Key Path Syntax

- Must start with `/`
- Can be **nested**: `/address/city`, `/metadata/region`
- Points to a JSON property path
- If the property doesn't exist in a document → partition key value is `null` (treated as undefined)
- Documents with `null`/undefined partition key all go to the SAME partition

```csharp
// Nested partition key path
ContainerProperties properties = new ContainerProperties("orders", "/customer/region");

// Document must have the nested property:
{
    "id": "order-1",
    "customer": {
        "name": "John",
        "region": "us-east"    // ← This is the partition key value
    }
}
```

### Gotcha 7: Indexing Mode — Only 2 Modes Exist

- **Consistent** (default): Index updated synchronously with every write
- **None**: Indexing completely disabled; container works as pure key-value store (only point reads by `id` + partition key work)
- ⚠️ **Lazy mode was REMOVED/DEPRECATED** — if you see it in old study materials, ignore it. Only `Consistent` and `None` exist now.

### Gotcha 8: Geo-Spatial Query Functions

If the exam asks about location/distance queries, know these built-in functions:

```sql
-- Distance between two points (returns meters)
SELECT * FROM c 
WHERE ST_DISTANCE(c.location, {"type": "Point", "coordinates": [-122.33, 47.60]}) < 5000

-- Check if point is within a polygon
SELECT * FROM c 
WHERE ST_WITHIN(c.location, {
    "type": "Polygon",
    "coordinates": [[[-122.4, 47.5], [-122.4, 47.7], [-122.2, 47.7], [-122.2, 47.5], [-122.4, 47.5]]]
})

-- Check if two geometries intersect
SELECT * FROM c WHERE ST_INTERSECTS(c.area, c.region)

-- Validate GeoJSON
SELECT ST_ISVALID({"type": "Point", "coordinates": [31.9, -4.8]}) AS isValid
SELECT ST_ISVALIDDETAILED({"type": "Point", "coordinates": [31.9, -4.8]}) AS validity
```

**Spatial indexes must be explicitly enabled** for these queries to be efficient:
```json
{
    "spatialIndexes": [
        {
            "path": "/location/*",
            "types": ["Point", "Polygon", "MultiPolygon", "LineString"]
        }
    ]
}
```

### Gotcha 9: Session Token for Cross-Client Consistency

- `Session` consistency guarantees read-your-writes only **within the same session**
- The **session token** must be explicitly passed between different client instances/services
- If Service A writes data and Service B needs to read it immediately, pass the session token

```csharp
// Service A: Write and capture session token
ItemResponse<Product> writeResponse = await containerA.CreateItemAsync(product, pk);
string sessionToken = writeResponse.Headers.Session;
// Pass sessionToken to Service B (via header, queue message, etc.)

// Service B: Read with the session token
ItemRequestOptions options = new ItemRequestOptions
{
    SessionToken = sessionToken
};
ItemResponse<Product> readResponse = await containerB.ReadItemAsync<Product>(
    "item-id", pk, options);
// Now guaranteed to see Service A's write
```

### Gotcha 10: Cosmos DB for Table vs Azure Table Storage

| Feature | Azure Table Storage | Cosmos DB Table API |
|---------|-------------------|-------------------|
| Latency | Variable (10s of ms) | < 10ms reads, < 15ms writes at p99 |
| Throughput | Max 20,000 ops/s per table | Unlimited (elastic) |
| Global distribution | GRS only (no active-active) | Multi-region read/write |
| Indexing | Primary key only | All properties auto-indexed |
| SLA | 99.9% | 99.99% (single-region), 99.999% (multi-region) |
| Consistency | Strong + Eventual only | All 5 levels |
| Pricing | Lower (storage-based) | Higher (RU-based) |

**Migration**: Existing Table Storage apps can migrate to Cosmos DB Table API with **no code changes** — just change the connection string.

### Gotcha 11: Advanced SQL Query Features (Exam Favorites)

```sql
-- DISTINCT (removes duplicates)
SELECT DISTINCT c.category FROM c

-- VALUE (returns scalar values, not objects)
SELECT VALUE c.name FROM c
-- Returns: ["Laptop", "Mouse"] instead of [{"name": "Laptop"}, {"name": "Mouse"}]

-- GROUP BY with aggregates
SELECT c.category, COUNT(1) AS count, AVG(c.price) AS avgPrice
FROM c
GROUP BY c.category

-- EXISTS (subquery)
SELECT * FROM c WHERE EXISTS (
    SELECT VALUE t FROM t IN c.tags WHERE t = "featured"
)

-- ARRAY expression
SELECT c.name, ARRAY(SELECT VALUE t FROM t IN c.tags WHERE t != "draft") AS activeTags
FROM c

-- Ternary operator
SELECT c.name, 
    (c.price > 100 ? "expensive" : "affordable") AS priceCategory
FROM c

-- IS_DEFINED / IS_NULL / IS_NUMBER / IS_STRING (type checking)
SELECT * FROM c WHERE IS_DEFINED(c.discount)
SELECT * FROM c WHERE NOT IS_NULL(c.email)

-- String functions
SELECT * FROM c WHERE STRINGEQUALS(c.name, "laptop", true)  -- case-insensitive
SELECT LOWER(c.name) AS name FROM c
SELECT CONCAT(c.firstName, " ", c.lastName) AS fullName FROM c
```

### Gotcha 12: Throughput — Shared vs Dedicated Interaction

- A database can have **shared throughput** (e.g., 1000 RU/s shared by all containers)
- Individual containers within that database can ALSO have **dedicated throughput**
- Containers with dedicated throughput do NOT consume from the shared pool
- **But**: A container must use one or the other — it cannot use both shared AND dedicated

```csharp
// Create database with 1000 RU/s shared throughput
await client.CreateDatabaseIfNotExistsAsync("myDb", throughput: 1000);

// Container using shared throughput (no throughput specified)
await database.CreateContainerAsync("sharedContainer", "/pk");

// Container with its own dedicated throughput
await database.CreateContainerIfNotExistsAsync(
    new ContainerProperties("dedicatedContainer", "/pk"),
    throughput: 400);  // 400 RU/s dedicated, does NOT use shared pool
```

---

## Quick Reference Summary

### Resource Hierarchy
- **Account**: Top-level, unique DNS endpoint (`<name>.documents.azure.com`), choose API (immutable)
- **Database**: Namespace for containers, optional shared throughput
- **Container**: Unit of scalability, has partition key, unique keys, indexing policy, TTL, conflict resolution
- **Item**: Actual data (max 2 MB), system props: `id`, `_rid`, `_self`, `_etag`, `_ts`

### APIs
- **NoSQL (SQL)**: Default, best for new apps, SQL queries, all features first
- **MongoDB**: Wire-compatible, BSON, for existing MongoDB apps
- **Cassandra**: CQL support, wide-column, for Cassandra migrations
- **Gremlin**: Graph database, vertices + edges, for relationship data
- **Table**: Key-value, for Azure Table Storage migration

### Consistency Levels (Strongest → Weakest)
1. **Strong**: Linearizable, 2x RU reads, **NOT available with multi-region writes**
2. **Bounded Staleness**: Within K versions or T seconds (K≥100K, T≥300s for multi-region)
3. **Session**: Read your writes within session **(DEFAULT)**
4. **Consistent Prefix**: In-order reads, no staleness guarantee
5. **Eventual**: Lowest latency, no ordering guarantee

### Throughput Models
- **Provisioned (Manual)**: Fixed RU/s, min 400 RU/s, increments of 100
- **Autoscale**: 10-100% of max, automatic scaling
- **Serverless**: Pay per RU, max 5000 RU/s burst, **single-region only, no SLA**
- **Shared (Database-level)**: min 400 RU/s shared across containers

### Partition Key Cheat Sheet
- **Good**: High cardinality, even distribution, used in queries
- **Bad**: Low cardinality, causes hot partitions
- **Logical partition max**: 20 GB
- **Physical partition max**: 50 GB storage, 10,000 RU/s
- **Partition key value max**: 2 KB
- **Synthetic keys**: Combine multiple properties for better distribution
- **Hierarchical keys**: Up to 3 levels

### SDK Client Classes
- **CosmosClient**: Entry point, **singleton pattern**, thread-safe
- **Database**: Database operations
- **Container**: CRUD and query operations
- **Scripts**: Stored procedures, triggers, UDFs

### Connection Modes
- **Direct (default)**: TCP, lowest latency, requires port range 10000-20000
- **Gateway**: HTTPS (port 443 only), for restricted networks

### CRUD Operations
- **Create**: `CreateItemAsync` (fails if exists, 409)
- **Read**: `ReadItemAsync` (point read, 1 RU for 1 KB)
- **Replace**: `ReplaceItemAsync` (full replacement)
- **Upsert**: `UpsertItemAsync` (create or replace)
- **Patch**: `PatchItemAsync` (partial update: Add, Remove, Replace, Set, Increment)
- **Delete**: `DeleteItemAsync`

### TTL Quick Rules
- Container TTL null → items never expire (item TTL ignored)
- Container TTL = -1 → enabled but no default; items use their own TTL
- Container TTL = N → items expire in N seconds unless overridden
- Item TTL = -1 → never expires (overrides container)
- Item TTL = M → expires in M seconds (overrides container)

### Change Feed Quick Rules
- **Captures**: Creates + Updates (latest version mode)
- **Does NOT capture**: Deletes (use soft-delete or "all versions and deletes" mode)
- **Lease container**: Stores checkpoints/state
- **Estimator**: Monitors processing lag
- **Pull model**: Manual control, no lease container needed

### Azure Functions Bindings
- **Trigger**: Reacts to change feed events
- **Input**: Reads documents (by ID or SQL query)
- **Output**: Writes/upserts documents

### Security Quick Reference
- **Keys**: Primary/Secondary (full access), Read-only keys
- **RBAC**: Data Reader, Data Contributor, Data Owner
- **Network**: IP Firewall < VNet Endpoints < **Private Endpoints** (highest isolation)
- **Encryption**: At rest (always on, free), in transit (TLS 1.2), CMK (optional)

### Backup Quick Reference
- **Periodic**: Default, 4h interval, contact support to restore
- **Continuous**: Self-service PITR, 7-day (free) or 30-day (paid)

### Key Limits Cheat Sheet

| Resource | Limit |
|----------|-------|
| Max item size (SQL API) | 2 MB |
| Max partition key value | 2 KB |
| Max logical partition | 20 GB |
| Max physical partition storage | 50 GB |
| Max physical partition throughput | 10,000 RU/s |
| Min container throughput | 400 RU/s |
| Max transactional batch operations | 100 |
| Max transactional batch size | 2 MB |
| Stored procedure timeout | 5 seconds |
| Max response size (stored proc) | 4 MB |
| Max accounts per subscription | 50 (default) |
| Serverless max burst | 5,000 RU/s |
| Bounded staleness (multi-region) | K ≥ 100,000, T ≥ 300s |
| Bounded staleness (single-region) | K ≥ 10, T ≥ 5s |
| Max unique keys per policy | 10 |
| Max paths per unique key | 16 |

### HTTP Status Codes Quick Reference

| Code | Meaning | Action |
|------|---------|--------|
| 200 | OK | Success |
| 201 | Created | Item created successfully |
| 204 | No Content | Delete successful |
| 304 | Not Modified | No new changes (change feed) |
| 400 | Bad Request | Fix request syntax/parameters |
| 401 | Unauthorized | Check authentication key/token |
| 403 | Forbidden | Insufficient permissions (check RBAC) |
| 404 | Not Found | Check item ID and partition key |
| 409 | Conflict | Item already exists (use Upsert) |
| 412 | Precondition Failed | ETag mismatch (refresh and retry) |
| 413 | Request Too Large | Reduce item size below 2 MB |
| 429 | Too Many Requests | Rate limited (retry with backoff) |
| 449 | Retry With | Transient error (SDK auto-retries) |
| 503 | Service Unavailable | Temporary outage (retry) |

