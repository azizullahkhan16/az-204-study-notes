# AZ-204 Practice Questions & Answers

---

## Q1: APIM - Subscription & Access Control

**Scenario:** APIs need subscription keys, terms of use, admin approval, and subscription limits.

**Options:**
- A. Create and publish a product
- B. Query string-based versioning
- C. Header-based versioning
- D. Add a new revision
- E. Make revisions current with change log

### ✅ Answer: A. Create and publish a product

**Why:** Products in APIM handle all access control features:
- Subscription keys ✓
- Terms of use (Legal Terms field) ✓
- Admin approval workflow ✓
- Subscription limits ✓

**Why not others:**
- **Versions (B, C)** = Managing breaking API changes (v1→v2), not access control
- **Revisions (D, E)** = Testing non-breaking changes safely, not subscriptions

**Quick Remember:**
| Products | Versions | Revisions |
|----------|----------|-----------|
| WHO can access | WHAT version | HOW to update safely |

---

## Q2: Azure Functions - HTTP Trigger Timeout

**Scenario:** HTTP triggered function times out after 4 min. Solution: Set `functionTimeout` in host.json to 10 minutes.

**Does this meet the goal?**
- A. Yes
- B. No

### ✅ Answer: B. No

**Why:** HTTP triggers have a **hard 230-second (~3.8 min) limit** enforced by Azure Load Balancer — regardless of `functionTimeout` setting.

**Key Concept - Function Timeouts:**
| Plan | Default | Max `functionTimeout` |
|------|---------|----------------------|
| Consumption | 5 min | 10 min |
| Premium | 30 min | Unlimited |
| Dedicated | 30 min | Unlimited |

⚠️ **But HTTP triggers are limited to ~230 seconds by Azure Load Balancer** — this cannot be changed.

**Better solutions for long-running work:**
- Use **Durable Functions** (async pattern)
- Use **Blob/Queue trigger** instead of HTTP
- Return HTTP response immediately, process in background

---

## Q3: Azure Monitor - Identify Configuration Changes

**Scenario:** Need to identify configuration changes made to web apps using Azure Monitor.

**Options:**
- A. AppServiceEnvironmentPlatformLogs
- B. AppServiceAppLogs
- C. AppServiceAuditLogs
- D. AppServiceConsoleLogs

### ✅ Answer: C. AppServiceAuditLogs

**Why:** AuditLogs track configuration/management changes to web apps.

**App Service Log Types:**
| Log | Purpose |
|-----|---------|
| **AuditLogs** | Config changes, admin actions |
| **AppLogs** | Application stdout/stderr |
| **ConsoleLogs** | Console output |
| **EnvironmentPlatformLogs** | ASE platform-level logs |

---

## Q4: Compute Solution - Infrequent Web Requests

**Scenario:** Small app receiving web requests with encoded coordinates. Calls occur **infrequently**.

**Options:**
- A. Azure Functions
- B. Azure App Service
- C. Azure Batch
- D. Azure API Management

### ✅ Answer: A. Azure Functions

**Why:** 
- **Infrequent calls** = Functions consumption plan is ideal (pay-per-execution, scales to zero)
- Small app + HTTP trigger = perfect Functions use case

**Why not others:**
- **App Service** = Always-on, pay even when idle
- **Batch** = Large-scale parallel/HPC jobs, not web requests
- **APIM** = API gateway, not a compute solution

**Quick Rule:** Infrequent/sporadic workloads → Azure Functions (Consumption)

---

## Q5: Event Hub - IoT Device Data Ingestion

**Scenario:** 2,000 stores, 1-5 POS devices each, 2MB/device/day. Need to store in Blob, correlate by device ID, scale for future.

**Solution:** Azure Event Hub with device ID as partition key + enable capture.

**Does this meet the goal?**
- A. Yes
- B. No

### ✅ Answer: A. Yes

**Why this works:**
- **Event Hub** = Designed for high-throughput streaming from many devices ✓
- **Partition key = device ID** = All data from same device goes to same partition (enables correlation) ✓
- **Capture enabled** = Auto-delivers data to Blob Storage ✓
- Scales easily for future stores ✓

**Key Concept - Event Hub Capture:**
Automatically streams Event Hub data to Azure Blob Storage or Data Lake without writing code.

---

## Q6: Application Insights - Distributed Tracing with Custom Data

**Scenario:** Web app with App Insights calls external OpenTelemetry services. Need customer ID associated with all operations across the system.

**Options:**
- A. Create SpanContext with TraceFlags = customer ID
- B. Set TraceId = customer ID
- C. Add customer ID to CorrelationContext
- D. Set Ocp-Apim-Trace header = customer ID

### ✅ Answer: C. Add the customer ID to CorrelationContext

**Why:** CorrelationContext (called "Baggage" in OpenTelemetry) is designed to propagate custom key-value pairs across service boundaries.

**Why not others:**
- **TraceFlags** = Bit field for sampling decisions, not custom data
- **TraceId** = Unique trace identifier (128-bit), not for business data
- **Ocp-Apim-Trace** = APIM debugging header, not correlation

**Key Concept - CorrelationContext/Baggage:**
Propagates custom data (user ID, tenant ID, etc.) automatically across all services in a distributed trace.

---

## Q7: Azure AD - Group-Based Authorization

**Scenario:** Web app needs permission levels (admin, normal, reader) based on Azure AD group membership.

**Solution:** Set `groupMembershipClaims = "All"` in app manifest, use `groups` claim from JWT.

**Does this meet the goal?**
- A. Yes
- B. No

### ✅ Answer: A. Yes

**Why this works:**
- `groupMembershipClaims = "All"` → Includes group IDs in the token ✓
- `groups` claim in JWT → Contains user's group Object IDs ✓
- App maps group IDs to permission levels ✓

**Key Concept - groupMembershipClaims values:**
| Value | Includes |
|-------|----------|
| None | No groups |
| SecurityGroup | Security groups only |
| All | Security groups + distribution lists + roles |

---

## Q8: Application Insights - Call Stack & Performance

**Scenario:** Web app on Basic plan has performance issues. Need complete call stack, correlated across instances, minimize cost/impact.

**Options (pick 3):**
- A. Enable Application Insights site extensions
- B. Enable Profiler
- C. Restart all apps in App Service plan
- D. Enable Snapshot debugger
- E. Enable remote debugging
- F. Enable Always On setting
- G. Upgrade to Premium

### ✅ Answer: A, B, C

**Why these three:**
- **A. App Insights extensions** = Required for telemetry collection
- **B. Profiler** = Captures call stacks for performance analysis ✓
- **C. Restart** = Needed to apply extension/profiler changes

**Why not others:**
- **D. Snapshot debugger** = For exceptions, not performance profiling
- **E. Remote debugging** = Manual debugging, impacts users
- **F. Always On** = Not available on Basic plan + adds cost
- **G. Premium** = Unnecessary, increases cost

**Key Concept:**
| Tool | Purpose |
|------|---------|
| **Profiler** | Performance call stacks |
| **Snapshot Debugger** | Exception snapshots |

---

## Q9: Cosmos DB - Multi-Tier SaaS Application

**Scenario:** SaaS app with key-value data. Low-cost edition = best-effort, single region. Higher editions = guaranteed performance, multi-region.

**Options:**
- A. Core (NoSQL)
- B. MongoDB
- C. Cassandra

### ✅ Answer: A. Core (NoSQL)

**Why Core API:**
- **Serverless mode** = Best-effort, single region, lowest cost (for low tier)
- **Provisioned throughput** = Guaranteed RU/s (for higher tiers)
- **Multi-region support** = Built-in
- Native Cosmos DB API = Best optimized, lowest cost

**Key Concept - Cosmos DB Capacity Modes:**
| Mode | Performance | Regions | Cost |
|------|-------------|---------|------|
| **Serverless** | Best-effort | Single | Lowest |
| **Provisioned** | Guaranteed RU/s | Multi-region | Higher |

MongoDB/Cassandra APIs are for compatibility, not cost optimization.

---

## Q10: Messaging - FIFO Transactional Messages

**Scenario:** Microservices need transactional messages in FIFO order.

**Options:**
- A. Azure Storage Queue
- B. Azure Event Hub
- C. Azure Service Bus
- D. Azure Event Grid

### ✅ Answer: C. Azure Service Bus

**Why Service Bus:**
- **FIFO guaranteed** via Sessions ✓
- **Transactions support** ✓
- Enterprise-grade reliable messaging

**Why not others:**
| Service | FIFO | Transactions | Purpose |
|---------|------|--------------|---------|
| **Storage Queue** | ❌ Best-effort | ❌ | Simple queuing |
| **Event Hub** | Partition only | ❌ | High-throughput streaming |
| **Event Grid** | ❌ | ❌ | Event routing/pub-sub |
| **Service Bus** | ✅ Sessions | ✅ | Enterprise messaging |

**Quick Rule:** FIFO + Transactions → Service Bus

---

## Q11: Secure VM Management - Remote Desktop

**Scenario:** Multi-tier app on VMs. Need RDP access for admin. Minimize internet exposure. Backend VMs not directly accessible from internet.

**Options:**
- A. Azure Bastion
- B. Service Endpoint
- C. Azure Private Link
- D. Azure Front Door

### ✅ Answer: A. Azure Bastion

**Why Bastion:**
- Secure RDP/SSH via Azure Portal over HTTPS (443) ✓
- VMs don't need public IP addresses ✓
- No need to expose RDP port 3389 to internet ✓
- Built-in security, no jump box needed

**Why not others:**
- **Service Endpoint** = Secures PaaS services to VNet, not VM management
- **Private Link** = Private access to PaaS services, not RDP
- **Front Door** = Web traffic load balancing/CDN, not VM admin

**Quick Rule:** Secure RDP/SSH without public IPs → Azure Bastion

---

## Q12: APIM Caching - Per-User Cache

**Scenario:** Azure Function with HTTP trigger behind APIM (consumption plan). OAuth auth. Need caching but customers must NOT see other customers' cached data.

**How to configure caching policy?**

### ✅ Answer:
```xml
<cache-lookup caching-type="internal" downstream-caching-type="private">
    <vary-by-header>Authorization</vary-by-header>
</cache-lookup>
```

**Key settings:**
| Setting | Value | Why |
|---------|-------|-----|
| `caching-type` | **internal** | Use built-in APIM cache |
| `downstream-caching-type` | **private** | Prevents shared caching (per-user) |
| `vary-by-header` | **Authorization** | Cache varies by user's auth token |

**Key Concept - APIM Cache Types:**
- `internal` = Built-in APIM cache
- `external` = Azure Cache for Redis
- `prefer-external` = External if configured, else internal

---

## Q13: Logic App - Azure Monitor Diagnostics Setup

**Scenario:** Logic App needs Azure Monitor logs stored in Blob storage. Set up logging.

**Steps in order:**
1. **Create a Log Analytics workspace**
2. **Install the Logic Apps Management solution**
3. **Add a diagnostic setting to the Logic App**

**Key Concept:** Log Analytics workspace is prerequisite for Azure Monitor logs. Management solution provides Logic App-specific insights.

---

## Q14: Azure Table Storage - CORS Configuration

**Scenario:** Web app getting CORS error: "No 'Access-Control-Allow-Origin' header present"

### ✅ Answer:
```xml
<CorsRule>
    <AllowedOrigins>http://your-domain.com</AllowedOrigins>
    <AllowedMethods>GET,POST</AllowedMethods>
    <AllowedHeaders>*</AllowedHeaders>
</CorsRule>
```

**Key Concept - CORS:**
- Cross-Origin Resource Sharing allows browsers to make requests to different domains
- Must configure `AllowedOrigins` with the requesting domain
- `AllowedMethods` specifies which HTTP methods are permitted

---

## Q15: Azure Function Security - Azure AD

**Scenario:** Secure Function app endpoints using Azure AD.

### ✅ Answer:
| Setting | Value |
|---------|-------|
| Authorization level | **Function** |
| User claims | **JSON Web Token (JWT)** |
| Trigger type | **HTTP** |

**Key Concept:**
- Azure AD uses JWT tokens for authentication
- HTTP trigger required for web-based auth flow
- Function-level auth provides per-function security

---

## Q16: Container Instance - OS Type

**Scenario:** Deploy container that must only be accessible from internal VNets.

### ✅ Answer: `osType: Linux`

**Key Concept:** Azure Container Instances:
- Linux containers support VNet integration
- Windows containers have limited VNet support
- Private = internal VNet only

---

## Q17: Container Logs - HTTP 502 Errors

**Scenario:** ContentUploadService showing HTTP 502 errors. Need to investigate.

**Which command?**
- A. az webapp log
- B. az ams live-output
- C. az monitor activity-log
- D. az container attach

### ✅ Answer: D. az container attach

**Why:** `az container attach` connects to a container's stdout/stderr to view real-time logs for troubleshooting.

**Note:** The dump says C, but for container instances, `az container attach` or `az container logs` is correct for viewing container output.

---

## Q18: Monitor Alerts - CPU Percentage

**Scenario:** Alert when service uses more than 80% CPU.

### ✅ Answer:
```bash
az monitor metrics alert create -n alert -g ... --scopes ... --condition "avg Percentage CPU > 80"
```

**Key Concept:**
- Metric name is `Percentage CPU` (not "CPU Usage")
- Value is percentage (80), not multiplied (800)
- Use `avg` aggregation for CPU metrics

---

## Q19: Azure AD App Manifest - OAuth2 Permissions

**Scenario:** Users need to review content using ContentAnalysisService with OAuth.

### ✅ Answer:
```json
{
    "oauth2Permissions": ["login"],
    "oauth2AllowImplicitFlow": true
}
```

**Key Concept:**
- `oauth2Permissions` = Permission scopes exposed to client apps
- `oauth2AllowImplicitFlow` = Required for SPA apps (Angular, React, etc.)

---

## Q20: Event Grid - Container Registry Events

**Scenario:** Trigger validation testing when new container image version is available.

### ✅ Answer:
```javascript
if (event.eventType === 'RepositoryUpdated' 
    && event.data.target.repository === 'contentanalysisservice'
    && event.topic.contains('contosoimages')) {
    startValidationTesting();
}
```

**Key Concept - Container Registry Events:**
| Event | When |
|-------|------|
| `ImagePushed` | New image pushed |
| `RepositoryUpdated` | Repository changes |
| `ImageDeleted` | Image removed |

---

## Q21: Blob Storage - Lifecycle Management

**Scenario:** Automatically move blobs to cool/archive storage based on age.

### ✅ Answer: Configure Blob Lifecycle Management Policy

```json
{
  "rules": [
    {
      "name": "moveToCool",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```

**Key Concept - Storage Tiers:**
| Tier | Access | Cost |
|------|--------|------|
| Hot | Frequent | High storage, low access |
| Cool | Infrequent (30+ days) | Lower storage, higher access |
| Archive | Rare (180+ days) | Lowest storage, highest access |

---

## Q22: Key Vault - Managed Identity Access

**Scenario:** App Service needs to access Key Vault secrets without storing credentials.

### ✅ Answer: Enable System-assigned Managed Identity + Grant Key Vault access policy

**Steps:**
1. Enable managed identity on App Service
2. Add access policy in Key Vault for the identity
3. Use `DefaultAzureCredential` in code

**Key Concept:**
- Managed Identity = Azure-managed credentials (no secrets to store)
- System-assigned = Tied to resource lifecycle
- User-assigned = Independent, can be shared

---

## Q23: App Configuration - Feature Flags

**Scenario:** Implement feature flags for gradual rollout.

### ✅ Answer: Use Azure App Configuration with Feature Management

```csharp
if (await featureManager.IsEnabledAsync("BetaFeature"))
{
    // New feature code
}
```

**Key Concept:**
- Feature flags enable/disable features without deployment
- Supports percentage rollout, targeting filters
- Integrates with .NET Feature Management library

---

## Q24: Durable Functions - Fan-out/Fan-in Pattern

**Scenario:** Process multiple items in parallel, then aggregate results.

### ✅ Answer: Use Fan-out/Fan-in pattern with `Task.WhenAll`

```csharp
[FunctionName("Orchestrator")]
public static async Task<int[]> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var tasks = new List<Task<int>>();
    var items = await context.CallActivityAsync<string[]>("GetItems", null);
    
    foreach (var item in items)
    {
        tasks.Add(context.CallActivityAsync<int>("ProcessItem", item));
    }
    
    var results = await Task.WhenAll(tasks);
    return results;
}
```

**Key Concept - Durable Function Patterns:**
| Pattern | Use Case |
|---------|----------|
| **Function chaining** | Sequential steps |
| **Fan-out/Fan-in** | Parallel processing |
| **Async HTTP APIs** | Long-running operations |
| **Monitor** | Polling/recurring |
| **Human interaction** | Approval workflows |

---

## Q25: Service Bus - Dead Letter Queue

**Scenario:** Messages failing processing need to be captured for investigation.

### ✅ Answer: Use Dead Letter Queue (DLQ)

**Messages go to DLQ when:**
- Max delivery count exceeded
- TTL expired
- Filter evaluation exception
- Explicit dead-lettering in code

**Access DLQ:**
```csharp
var dlqPath = EntityNameHelper.FormatDeadLetterPath(queueName);
var receiver = new ServiceBusReceiver(client, dlqPath);
```

**Key Concept:** DLQ is a sub-queue that holds messages that can't be processed. Always monitor DLQ for failed messages.

---

## Q26: Azure CDN - Cache Purge

**Scenario:** Updated content on origin but CDN still serving stale content.

### ✅ Answer: Purge the CDN endpoint

```bash
az cdn endpoint purge --resource-group myRG --profile-name myProfile --name myEndpoint --content-paths '/*'
```

**Key Concept:**
- CDN caches content at edge locations
- Purge forces CDN to fetch fresh content from origin
- Use `/*` for all content or specific paths

---

## Q27: Cosmos DB - Consistency Levels

**Scenario:** Need to balance consistency vs performance for global app.

### ✅ Answer: Choose appropriate consistency level

| Level | Guarantee | Performance |
|-------|-----------|-------------|
| **Strong** | Linearizable | Slowest |
| **Bounded Staleness** | Consistent prefix + bounded lag | |
| **Session** | Consistent for session | Default ⭐ |
| **Consistent Prefix** | Reads never see out-of-order | |
| **Eventual** | No ordering guarantee | Fastest |

**Key Concept:** Session consistency is default - guarantees consistency within a user session, good balance for most apps.

---

## Q28: App Service - Deployment Slots

**Scenario:** Deploy new version without downtime, ability to rollback instantly.

### ✅ Answer: Use Deployment Slots with Swap

**Steps:**
1. Deploy to staging slot
2. Test staging
3. Swap staging ↔ production
4. If issues, swap back (instant rollback)

**Key Concept:**
- Slots are live apps with own hostnames
- Swap is instant (no cold start)
- Slot settings can be "sticky" (not swapped)

---

## Q29: Azure Functions - Binding Expressions

**Scenario:** Create blob output with dynamic name based on input.

### ✅ Answer: Use binding expressions

```csharp
[FunctionName("ProcessOrder")]
public static void Run(
    [QueueTrigger("orders")] Order order,
    [Blob("receipts/{Id}.txt", FileAccess.Write)] out string receipt)
{
    receipt = $"Order {order.Id} processed";
}
```

**Key Concept:**
- `{property}` = Bind to input property
- `{DateTime}` = Current datetime
- `{rand-guid}` = Random GUID

---

## Q30: Authentication - MSAL Token Cache

**Scenario:** Desktop app needs to cache tokens for silent authentication.

### ✅ Answer: Configure token cache serialization

```csharp
var app = PublicClientApplicationBuilder
    .Create(clientId)
    .Build();

// Enable token cache
var cacheHelper = await MsalCacheHelper.CreateAsync(storageProperties);
cacheHelper.RegisterCache(app.UserTokenCache);
```

**Key Concept:**
- Token cache enables silent auth (no re-login)
- Desktop apps use `PublicClientApplication`
- Web apps use `ConfidentialClientApplication`

---

## Q31: Event Grid - Custom Topics

**Scenario:** Publish custom events from your application.

### ✅ Answer:

```csharp
var client = new EventGridPublisherClient(
    new Uri(topicEndpoint), 
    new AzureKeyCredential(topicKey));

var events = new List<EventGridEvent>
{
    new EventGridEvent(
        subject: "orders/new",
        eventType: "Order.Created",
        dataVersion: "1.0",
        data: new { OrderId = 123 })
};

await client.SendEventsAsync(events);
```

**Key Concept - Event Grid:**
- **Topics** = Endpoints to receive events
- **Subscriptions** = Where to route events
- **Event Handlers** = Functions, Webhooks, Queues, etc.

---

## Q32: Azure Redis Cache - Connection

**Scenario:** Connect to Azure Cache for Redis from .NET app.

### ✅ Answer:

```csharp
var redis = ConnectionMultiplexer.Connect(connectionString);
var db = redis.GetDatabase();

// Set value
await db.StringSetAsync("key", "value", TimeSpan.FromMinutes(10));

// Get value
var value = await db.StringGetAsync("key");
```

**Key Concept:**
- Use `StackExchange.Redis` library
- Connection string from Azure Portal
- Always use async methods
- Set expiration to avoid memory issues

---

## Q33: Blob Storage - SAS Token

**Scenario:** Generate time-limited access to specific blob.

### ✅ Answer:

```csharp
var sasBuilder = new BlobSasBuilder
{
    BlobContainerName = containerName,
    BlobName = blobName,
    Resource = "b", // b = blob, c = container
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};
sasBuilder.SetPermissions(BlobSasPermissions.Read);

var sasToken = sasBuilder.ToSasQueryParameters(
    new StorageSharedKeyCredential(accountName, accountKey));
    
var sasUri = $"{blobUri}?{sasToken}";
```

**Key Concept - SAS Types:**
| Type | Scope |
|------|-------|
| **Service SAS** | Single service (blob, queue, etc.) |
| **Account SAS** | Multiple services |
| **User Delegation SAS** | Azure AD-based (most secure) |

---

## Q34: Container Registry - Authentication

**Scenario:** Pull images from Azure Container Registry.

### ✅ Answer Options:

1. **Admin account** (not recommended for production)
2. **Service Principal** (CI/CD pipelines)
3. **Managed Identity** (Azure services) ✓ Best
4. **Azure AD individual** (developers)

```bash
# Using managed identity
az acr login --name myregistry
```

**Key Concept:** Use managed identity when possible - no credentials to manage.

---

## Q35: App Service - Custom Domain SSL

**Scenario:** Configure custom domain with SSL certificate.

### ✅ Answer Steps:
1. Add custom domain to App Service
2. Verify domain ownership (CNAME or TXT record)
3. Upload/create SSL certificate
4. Add SSL binding (SNI or IP-based)

**Key Concept - SSL Binding Types:**
| Type | Use Case |
|------|----------|
| **SNI SSL** | Multiple domains, single IP (modern) |
| **IP SSL** | Legacy clients, dedicated IP |

---

## Q36: Azure Functions - Premium Plan Benefits

**Scenario:** Need Functions with VNet integration, no cold start.

### ✅ Answer: Use Premium Plan

**Premium Plan features:**
- ✅ VNet integration
- ✅ No cold start (pre-warmed instances)
- ✅ Unlimited execution time
- ✅ More powerful instances
- ✅ Private site access

**When to use:**
- Long-running functions
- VNet connectivity required
- Predictable performance needed

---

## Q37: Cosmos DB - Partition Key Selection

**Scenario:** Design partition key for e-commerce orders.

### ✅ Answer: Use high-cardinality field (e.g., `customerId` or `orderId`)

**Good partition keys:**
- ✅ High cardinality (many distinct values)
- ✅ Even distribution
- ✅ Commonly used in queries

**Bad partition keys:**
- ❌ Low cardinality (status, category)
- ❌ Timestamp alone (hot partition)
- ❌ Boolean values

**Key Concept:** Partition key determines data distribution and query performance. Choose based on access patterns.

---

## Q38: Application Insights - Availability Tests

**Scenario:** Monitor web app availability from multiple locations.

### ✅ Answer: Create URL ping test or multi-step test

**Test types:**
| Type | Purpose |
|------|---------|
| **URL ping** | Simple availability check |
| **Multi-step** | Complex scenarios (deprecated) |
| **Standard test** | Advanced with custom headers |
| **Custom TrackAvailability** | Code-based tests |

**Key Concept:** Availability tests run from Azure data centers worldwide to detect regional outages.

---

## Q39: Service Bus - Sessions for FIFO

**Scenario:** Ensure messages for same entity processed in order.

### ✅ Answer: Enable sessions on queue/subscription

```csharp
// Send with session
var message = new ServiceBusMessage("data")
{
    SessionId = "order-123" // All messages with same SessionId processed in order
};

// Receive with session
var receiver = client.AcceptSessionAsync(queueName, "order-123");
```

**Key Concept:**
- Session = Group of related messages
- FIFO guaranteed within session
- One receiver per session at a time

---

## Q40: Azure AD B2C - User Flows

**Scenario:** Implement sign-up/sign-in for consumer-facing app.

### ✅ Answer: Use Azure AD B2C with User Flows

**User Flow types:**
| Flow | Purpose |
|------|---------|
| **Sign up and sign in** | Combined registration/login |
| **Profile editing** | Update user attributes |
| **Password reset** | Self-service reset |

**Key Concept:**
- B2C = Consumer identity (social logins, local accounts)
- B2B = Business partner access
- Azure AD = Enterprise employees

---

## Q41: Blob Storage - Access Tiers (Hot/Cool/Archive)

**Scenario:** Which tier for data accessed rarely, stored for compliance (180+ days)?

### ✅ Answer: Archive tier

**Key Concept - Rehydration:**
- Archive blobs are offline
- Must rehydrate before reading (hours)
- Set priority: Standard (15 hrs) or High (1 hr)

---

## Q42: Functions - Trigger Types

**Scenario:** Match trigger to use case.

| Trigger | Use Case |
|---------|----------|
| **HTTP** | REST APIs, webhooks |
| **Timer** | Scheduled jobs (CRON) |
| **Blob** | React to file uploads |
| **Queue** | Message processing |
| **Cosmos DB** | Change feed processing |
| **Event Hub** | Streaming data |
| **Event Grid** | Event-driven reactions |

---

## Q43: APIM - Rate Limiting

**Scenario:** Limit API calls to 100 per minute per subscription.

### ✅ Answer: Use `rate-limit-by-key` policy

```xml
<rate-limit-by-key calls="100" renewal-period="60" 
    counter-key="@(context.Subscription.Id)" />
```

**Key Concept:**
- `rate-limit` = Hard limit, returns 429
- `quota` = Over longer period (day/week/month)

---

## Q44: Key Vault - Secret Versioning

**Scenario:** App needs to get latest secret version automatically.

### ✅ Answer: Reference secret without version

```
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)
```

**Key Concept:**
- Without version = Always latest
- With version = Specific version (for rollback)

---

## Q45: Container Apps - Scaling Rules

**Scenario:** Scale container based on HTTP traffic.

### ✅ Answer: Configure HTTP scaling rule

```yaml
scale:
  minReplicas: 1
  maxReplicas: 10
  rules:
    - name: http-rule
      http:
        metadata:
          concurrentRequests: "100"
```

**Key Concept:** Container Apps support KEDA scalers (HTTP, Queue length, CPU, Custom).

---

## Q46: Cosmos DB - Change Feed

**Scenario:** React to document changes in real-time.

### ✅ Answer: Use Change Feed with Azure Functions

```csharp
[FunctionName("ProcessChanges")]
public static void Run(
    [CosmosDBTrigger(
        databaseName: "db",
        collectionName: "items",
        LeaseCollectionName = "leases")] IReadOnlyList<Document> documents)
{
    // Process changed documents
}
```

**Key Concept:**
- Change feed = Ordered log of changes
- Requires lease collection for checkpointing
- Only inserts and updates (not deletes)

---

## Q47: App Service - Auto-scaling

**Scenario:** Scale web app based on CPU usage.

### ✅ Answer: Configure auto-scale rules

**Scale conditions:**
- Metric-based (CPU, Memory, HTTP queue)
- Schedule-based (time of day)

**Key settings:**
- Instance limits (min/max)
- Scale-out rule (when to add)
- Scale-in rule (when to remove)
- Cool-down period

---

## Q48: Azure AD - App Roles

**Scenario:** Implement role-based access in web app.

### ✅ Answer: Define app roles in manifest + assign to users

```json
"appRoles": [
    {
        "allowedMemberTypes": ["User"],
        "displayName": "Admin",
        "value": "Admin"
    }
]
```

**Access in code:**
```csharp
[Authorize(Roles = "Admin")]
public IActionResult AdminOnly() { }
```

---

## Q49: Storage Queue vs Service Bus Queue

**Scenario:** When to use which?

| Feature | Storage Queue | Service Bus |
|---------|--------------|-------------|
| **Max size** | 64 KB | 256 KB (Standard), 100 MB (Premium) |
| **FIFO** | Best-effort | Guaranteed (sessions) |
| **Transactions** | ❌ | ✅ |
| **Duplicate detection** | ❌ | ✅ |
| **Dead-lettering** | ❌ | ✅ |
| **Cost** | Lower | Higher |

**Quick Rule:** Simple/cheap → Storage Queue. Enterprise features → Service Bus.

---

## Q50: Azure Functions - Durable Entities

**Scenario:** Maintain state across function executions.

### ✅ Answer: Use Durable Entities (Actor pattern)

```csharp
[FunctionName("Counter")]
public static void Counter([EntityTrigger] IDurableEntityContext ctx)
{
    int current = ctx.GetState<int>();
    switch (ctx.OperationName)
    {
        case "add":
            ctx.SetState(current + ctx.GetInput<int>());
            break;
        case "get":
            ctx.Return(current);
            break;
    }
}
```

**Key Concept:** Entities = Stateful objects that persist automatically.

---

## Q51: Functions - Output Binding to Blob

**Scenario:** HTTP triggered function processes blob data using output binding on blob. Times out after 4 minutes.

**Solution:** Pass HTTP payload to Service Bus queue, process with queue trigger function, return immediate HTTP response.

### ✅ Answer: Yes, this meets the goal

**Why:** 
- HTTP triggers limited to ~230 seconds
- Queue trigger can run longer
- Async pattern = immediate response + background processing

**Best Practice:** Long-running work → Defer to queue/Durable Functions

---

## Q52: Cosmos DB - Point Read vs Query

**Scenario:** Need fastest way to retrieve single document.

### ✅ Answer: Use Point Read (ReadItemAsync)

```csharp
// Point read - 1 RU, fastest
var response = await container.ReadItemAsync<Item>(id, new PartitionKey(pk));

// Query - More RUs, slower
var query = container.GetItemQueryIterator<Item>("SELECT * FROM c WHERE c.id = @id");
```

**Key Concept:** Point read requires both `id` AND `partition key` - uses only 1 RU.

---

## Q53: App Service - Diagnostic Logs

**Scenario:** Enable application logging for troubleshooting.

### ✅ Answer: Enable diagnostic logs

| Log Type | Content |
|----------|---------|
| **Application logging** | App stdout/stderr |
| **Web server logging** | HTTP request logs |
| **Detailed error messages** | Error pages |
| **Failed request tracing** | Request pipeline trace |
| **Deployment logging** | Deployment operations |

---

## Q54: APIM - Validate JWT Policy

**Scenario:** Validate OAuth tokens in APIM.

### ✅ Answer:

```xml
<validate-jwt header-name="Authorization" require-scheme="Bearer">
    <openid-config url="https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration" />
    <required-claims>
        <claim name="aud" match="all">
            <value>{api-client-id}</value>
        </claim>
    </required-claims>
</validate-jwt>
```

**Key Concept:** Validate JWT at API gateway level - reject invalid tokens before hitting backend.

---

## Q55: Storage - Immutable Blob Storage

**Scenario:** Store data that cannot be modified or deleted for compliance.

### ✅ Answer: Configure immutability policy

**Policy Types:**
| Type | Description |
|------|-------------|
| **Time-based retention** | Cannot delete until period expires |
| **Legal hold** | Cannot delete until hold removed |

**Key Concept:** WORM (Write Once Read Many) - Required for SEC, FINRA compliance.

---

## Q56: Event Hub - Consumer Groups

**Scenario:** Multiple applications need to read same events independently.

### ✅ Answer: Use separate consumer groups

```csharp
// App 1
var processor1 = new EventProcessorClient(storageClient, "analytics-consumer", ...);

// App 2  
var processor2 = new EventProcessorClient(storageClient, "backup-consumer", ...);
```

**Key Concept:** Each consumer group maintains its own position - apps read independently without affecting each other.

---

## Q57: Functions - Managed Dependencies

**Scenario:** PowerShell function needs Az modules auto-updated.

### ✅ Answer: Enable managed dependencies in host.json

```json
{
  "managedDependency": {
    "enabled": true
  }
}
```

**requirements.psd1:**
```powershell
@{
    'Az' = '9.*'
}
```

---

## Q58: App Configuration - Sentinel Key

**Scenario:** Refresh configuration only when specific key changes.

### ✅ Answer: Configure sentinel key for refresh

```csharp
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .ConfigureRefresh(refresh =>
           {
               refresh.Register("Settings:Sentinel", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromSeconds(30));
           });
});
```

**Key Concept:** Sentinel key = Single key that triggers full refresh when changed.

---

## Q59: Container Instances - Restart Policy

**Scenario:** Run container task once then stop.

### ✅ Answer: Set restart policy to "Never"

| Policy | Behavior |
|--------|----------|
| **Always** | Restart on exit (default) |
| **OnFailure** | Restart only on non-zero exit |
| **Never** | Run once, don't restart |

---

## Q60: Blob Storage - Lease for Exclusive Access

**Scenario:** Ensure only one process modifies blob at a time.

### ✅ Answer: Acquire blob lease

```csharp
var leaseClient = blobClient.GetBlobLeaseClient();
var lease = await leaseClient.AcquireAsync(TimeSpan.FromSeconds(60));

// Use lease.LeaseId for operations
await blobClient.UploadAsync(data, new BlobUploadOptions 
{ 
    Conditions = new BlobRequestConditions { LeaseId = lease.LeaseId }
});

await leaseClient.ReleaseAsync();
```

**Key Concept:** Lease = Distributed lock for blobs.

---

## Q61: Azure AD - Incremental Consent

**Scenario:** Request additional permissions after initial sign-in.

### ✅ Answer: Use incremental/dynamic consent

```csharp
var scopes = new[] { "User.Read", "Mail.Read" }; // Request as needed

var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();
```

**Key Concept:** Don't request all permissions upfront - ask when needed.

---

## Q62: Service Bus - Duplicate Detection

**Scenario:** Prevent duplicate messages from being processed.

### ✅ Answer: Enable duplicate detection on queue

**Configuration:**
- Enable duplicate detection (up to 7 days window)
- Set `MessageId` on each message

```csharp
var message = new ServiceBusMessage("data")
{
    MessageId = "unique-order-id-123" // Duplicates with same ID rejected
};
```

---

## Q63: Functions - Proxies (Deprecated but may appear)

**Scenario:** Route requests to different backends.

### ✅ Answer: Use proxies.json (legacy) or APIM

**Note:** Function Proxies deprecated - use APIM for new projects.

---

## Q64: Cosmos DB - Stored Procedures

**Scenario:** Execute transaction across multiple documents.

### ✅ Answer: Use stored procedures

```javascript
function bulkUpdate(items) {
    var context = getContext();
    var container = context.getCollection();
    
    items.forEach(function(item) {
        container.replaceDocument(item._self, item);
    });
}
```

**Key Concept:** Stored procedures run in single partition - atomic transactions.

---

## Q65: App Service - Hybrid Connections

**Scenario:** Connect App Service to on-premises database without VPN.

### ✅ Answer: Use Hybrid Connections

**How it works:**
1. Install Hybrid Connection Manager on-premises
2. Create relay in Azure
3. App Service connects via relay (no inbound firewall rules)

**Key Concept:** Outbound connection from on-prem - no firewall changes needed.

---

## Q66: Key Vault - Soft Delete

**Scenario:** Recover accidentally deleted secret.

### ✅ Answer: Recover from soft delete

```bash
# List deleted secrets
az keyvault secret list-deleted --vault-name myvault

# Recover secret
az keyvault secret recover --vault-name myvault --name mysecret
```

**Key Concept:** Soft delete enabled by default - 90 day retention. Purge protection prevents permanent deletion.

---

## Q67: Event Grid - Filtering

**Scenario:** Subscribe only to specific event types.

### ✅ Answer: Configure event filters

```json
{
  "filter": {
    "subjectBeginsWith": "/blobServices/default/containers/images",
    "subjectEndsWith": ".jpg",
    "includedEventTypes": ["Microsoft.Storage.BlobCreated"]
  }
}
```

**Filter Types:**
- Subject prefix/suffix
- Event type
- Advanced filters (data field values)

---

## Q68: Functions - Custom Handlers

**Scenario:** Run non-.NET language (Go, Rust) in Azure Functions.

### ✅ Answer: Use custom handlers

**host.json:**
```json
{
  "customHandler": {
    "description": {
      "defaultExecutablePath": "myapp.exe"
    }
  }
}
```

**Key Concept:** Custom handlers run any language that can serve HTTP.

---

## Q69: APIM - Mock Responses

**Scenario:** Return mock response during development.

### ✅ Answer: Use mock-response policy

```xml
<inbound>
    <mock-response status-code="200" content-type="application/json" />
</inbound>
```

**Key Concept:** Mock responses from OpenAPI examples or inline - no backend needed.

---

## Q70: Blob Storage - Object Replication

**Scenario:** Automatically copy blobs to another storage account.

### ✅ Answer: Configure object replication

**Requirements:**
- Source: Hot or Cool tier, change feed enabled
- Destination: Any tier
- Asynchronous replication

**Use cases:** Geo-redundancy, data locality, disaster recovery

---

## Q71: Azure AD - Conditional Access

**Scenario:** Require MFA for specific app access.

### ✅ Answer: Configure Conditional Access policy

**Conditions:**
- Users/groups
- Cloud apps
- Locations
- Device state

**Controls:**
- Block access
- Grant with MFA
- Require compliant device

---

## Q72: Functions - Identity-based Connections

**Scenario:** Connect to Storage without connection string.

### ✅ Answer: Use managed identity

**local.settings.json:**
```json
{
  "Values": {
    "AzureWebJobsStorage__accountName": "mystorageaccount"
  }
}
```

**Key Concept:** `__accountName` suffix = Use managed identity instead of connection string.

---

## Q73: Cosmos DB - Time to Live (TTL)

**Scenario:** Automatically delete documents after expiration.

### ✅ Answer: Configure TTL

```csharp
// Container level (default TTL)
var containerProperties = new ContainerProperties
{
    DefaultTimeToLive = 86400 // 24 hours in seconds
};

// Document level (override)
{
    "id": "1",
    "ttl": 3600 // 1 hour - overrides container default
}
```

**Special values:** `-1` = inherit from container, `null` = never expire

---

## Q74: App Service - Access Restrictions

**Scenario:** Allow only specific IPs to access web app.

### ✅ Answer: Configure access restrictions

```bash
az webapp config access-restriction add \
    --resource-group myRG \
    --name myApp \
    --rule-name "AllowOffice" \
    --priority 100 \
    --ip-address "203.0.113.0/24"
```

**Key Concept:** Rules evaluated by priority (lower = first). Default action = Deny.

---

## Q75: Service Bus - Scheduled Messages

**Scenario:** Send message to be delivered at specific time.

### ✅ Answer: Schedule message

```csharp
var message = new ServiceBusMessage("scheduled data");
var sequenceNumber = await sender.ScheduleMessageAsync(
    message, 
    DateTimeOffset.UtcNow.AddHours(1));

// Cancel if needed
await sender.CancelScheduledMessageAsync(sequenceNumber);
```

---

## Q76: Functions - Execution Context

**Scenario:** Get function invocation ID for correlation.

### ✅ Answer: Use ExecutionContext

```csharp
[FunctionName("MyFunction")]
public static void Run(
    [HttpTrigger] HttpRequest req,
    ExecutionContext context,
    ILogger log)
{
    log.LogInformation($"Invocation ID: {context.InvocationId}");
    log.LogInformation($"Function name: {context.FunctionName}");
}
```

---

## Q77: Blob Storage - Static Website

**Scenario:** Host static website from blob storage.

### ✅ Answer: Enable static website

```bash
az storage blob service-properties update \
    --account-name mystorageaccount \
    --static-website \
    --index-document index.html \
    --404-document 404.html
```

**Key Concept:** Creates `$web` container. Access via `https://<account>.z6.web.core.windows.net/`

---

## Q78: Event Hub - Checkpointing

**Scenario:** Resume processing from where left off after restart.

### ✅ Answer: Implement checkpointing

```csharp
async Task ProcessEventHandler(ProcessEventArgs args)
{
    // Process event
    Console.WriteLine(args.Data.EventBody.ToString());
    
    // Checkpoint every 10 events
    if (args.Data.SequenceNumber % 10 == 0)
    {
        await args.UpdateCheckpointAsync();
    }
}
```

**Key Concept:** Checkpoint = Save position in partition. Requires Blob Storage for checkpoint store.

---

## Q79: Container Registry - Tasks

**Scenario:** Automatically build image when code changes.

### ✅ Answer: Create ACR Task

```bash
az acr task create \
    --registry myregistry \
    --name buildtask \
    --image myapp:{{.Run.ID}} \
    --context https://github.com/myrepo.git \
    --file Dockerfile \
    --git-access-token $TOKEN
```

**Triggers:** Git commit, Base image update, Schedule

---

## Q80: Functions - Retry Policies

**Scenario:** Automatically retry failed function executions.

### ✅ Answer: Configure retry policy

**function.json:**
```json
{
  "retry": {
    "strategy": "exponentialBackoff",
    "maxRetryCount": 5,
    "minimumInterval": "00:00:10",
    "maximumInterval": "00:15:00"
  }
}
```

**Strategies:** `fixedDelay`, `exponentialBackoff`

---

## Q81: APIM - Backend Policy

**Scenario:** Route requests to different backend based on condition.

### ✅ Answer: Use set-backend-service policy

```xml
<choose>
    <when condition="@(context.Request.Headers.GetValueOrDefault("x-version","v1") == "v2")">
        <set-backend-service base-url="https://api-v2.contoso.com" />
    </when>
    <otherwise>
        <set-backend-service base-url="https://api-v1.contoso.com" />
    </otherwise>
</choose>
```

---

## Q82: Cosmos DB - Indexing Policy

**Scenario:** Optimize queries by customizing indexes.

### ✅ Answer: Configure indexing policy

```json
{
    "indexingMode": "consistent",
    "includedPaths": [
        { "path": "/category/*" }
    ],
    "excludedPaths": [
        { "path": "/description/*" }
    ],
    "compositeIndexes": [
        [
            { "path": "/category", "order": "ascending" },
            { "path": "/price", "order": "descending" }
        ]
    ]
}
```

**Key Concept:** Exclude large text fields, include query fields, use composite indexes for ORDER BY.

---

## Q83: Functions - Dependency Injection

**Scenario:** Inject services into Azure Functions.

### ✅ Answer: Use DI with Startup class

```csharp
[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

public class Startup : FunctionsStartup
{
    public override void Configure(IFunctionsHostBuilder builder)
    {
        builder.Services.AddSingleton<IMyService, MyService>();
        builder.Services.AddHttpClient();
    }
}
```

---

## Q84: App Service - TLS/SSL Settings

**Scenario:** Enforce minimum TLS version.

### ✅ Answer: Configure minimum TLS version

```bash
az webapp config set --resource-group myRG --name myApp --min-tls-version 1.2
```

**Key Concept:** TLS 1.2 minimum recommended. TLS 1.0/1.1 deprecated.

---

## Q85: Storage - AzCopy

**Scenario:** Copy large amounts of data to/from blob storage.

### ✅ Answer: Use AzCopy

```bash
# Copy local to blob
azcopy copy "C:\data\*" "https://account.blob.core.windows.net/container?SAS" --recursive

# Copy between storage accounts
azcopy copy "https://source?SAS" "https://dest?SAS" --recursive
```

**Key Concept:** High-performance, parallel transfers. Use SAS or Azure AD auth.

---

## Q86: APIM - Response Caching Headers

**Scenario:** Control client-side caching with headers.

### ✅ Answer: Use set-header policy

```xml
<outbound>
    <set-header name="Cache-Control" exists-action="override">
        <value>max-age=3600, must-revalidate</value>
    </set-header>
</outbound>
```

---

## Q87: Event Grid - Dead Letter

**Scenario:** Handle events that cannot be delivered.

### ✅ Answer: Configure dead-letter destination

**Events go to dead letter when:**
- Max retry attempts exceeded (30)
- TTL expired (24 hours default)

**Storage account required** for dead letter - stores failed events as blobs.

---

## Q88: Functions - SignalR Service Output

**Scenario:** Send real-time messages to connected clients.

### ✅ Answer: Use SignalR output binding

```csharp
[FunctionName("Broadcast")]
public static async Task Run(
    [HttpTrigger] HttpRequest req,
    [SignalR(HubName = "chat")] IAsyncCollector<SignalRMessage> messages)
{
    await messages.AddAsync(new SignalRMessage
    {
        Target = "newMessage",
        Arguments = new[] { "Hello everyone!" }
    });
}
```

---

## Q89: Blob Storage - Blob Index Tags

**Scenario:** Search blobs by custom metadata.

### ✅ Answer: Use blob index tags

```csharp
// Set tags
var tags = new Dictionary<string, string>
{
    { "project", "sales" },
    { "status", "processed" }
};
await blobClient.SetTagsAsync(tags);

// Query by tags
var blobs = containerClient.FindBlobsByTagsAsync("project = 'sales' AND status = 'processed'");
```

**Key Concept:** Tags are indexed and queryable across containers/accounts.

---

## Q90: Service Bus - Message Deferral

**Scenario:** Postpone processing of message until later.

### ✅ Answer: Defer message

```csharp
// Defer - saves sequence number
await receiver.DeferMessageAsync(message);
var sequenceNumber = message.SequenceNumber;

// Receive deferred later
var deferredMsg = await receiver.ReceiveDeferredMessageAsync(sequenceNumber);
```

**Key Concept:** Deferred messages must be received by sequence number.

---

## Q91: Azure AD - Client Credentials Flow

**Scenario:** Daemon/service app authenticating without user.

### ✅ Answer: Use client credentials flow

```csharp
var app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)
    .WithAuthority(authority)
    .Build();

var result = await app.AcquireTokenForClient(scopes).ExecuteAsync();
```

**Key Concept:** App-only access, no user interaction. Use for background services.

---

## Q92: Cosmos DB - Partial Document Update

**Scenario:** Update only specific properties without replacing entire document.

### ✅ Answer: Use Patch operations

```csharp
var patchOperations = new List<PatchOperation>
{
    PatchOperation.Set("/status", "shipped"),
    PatchOperation.Increment("/viewCount", 1),
    PatchOperation.Add("/tags/-", "urgent")
};

await container.PatchItemAsync<Document>(id, partitionKey, patchOperations);
```

**Key Concept:** Patch = Atomic partial updates, lower RU than full replace.

---

## Q93: App Service - WebJobs

**Scenario:** Run background tasks in App Service.

### ✅ Answer: Use WebJobs

| Type | Trigger |
|------|---------|
| **Continuous** | Runs constantly (while loop) |
| **Triggered** | Schedule (CRON) or manual |

**Note:** Consider Azure Functions for new projects - more features, better scaling.

---

## Q94: Key Vault - Certificate Management

**Scenario:** Store and manage SSL certificates.

### ✅ Answer: Import/create certificates in Key Vault

```bash
# Import existing certificate
az keyvault certificate import --vault-name myvault --name mycert --file cert.pfx

# Create self-signed (for testing)
az keyvault certificate create --vault-name myvault --name mycert --policy @policy.json
```

**Key Concept:** Key Vault can auto-renew certificates with integrated CAs.

---

## Q95: Functions - Durable Functions - Human Interaction

**Scenario:** Approval workflow with timeout.

### ✅ Answer: Use WaitForExternalEvent with timeout

```csharp
[FunctionName("ApprovalWorkflow")]
public static async Task<bool> Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var timeout = context.CurrentUtcDateTime.AddHours(24);
    var approvalTask = context.WaitForExternalEvent<bool>("ApprovalEvent");
    var timeoutTask = context.CreateTimer(timeout, CancellationToken.None);
    
    var winner = await Task.WhenAny(approvalTask, timeoutTask);
    
    if (winner == approvalTask)
        return approvalTask.Result;
    else
        return false; // Timed out
}
```

---

## Q96: Storage - Private Endpoints

**Scenario:** Access storage only from VNet.

### ✅ Answer: Configure private endpoint

**Steps:**
1. Create private endpoint for storage
2. Disable public access
3. Configure private DNS zone

**Key Concept:** Traffic stays on Microsoft backbone - never goes over public internet.

---

## Q97: APIM - Named Values

**Scenario:** Store reusable configuration values.

### ✅ Answer: Use named values

```xml
<set-header name="X-Api-Key" exists-action="override">
    <value>{{api-key-secret}}</value>
</set-header>
```

**Types:**
- Plain text
- Secret (hidden)
- Key Vault reference (recommended)

---

## Q98: Event Hub - Partition Receiver

**Scenario:** Read from specific partition.

### ✅ Answer: Use partition receiver

```csharp
var consumer = client.CreateConsumer("$Default", "0", EventPosition.Earliest);

await foreach (var partitionEvent in consumer.ReadEventsFromPartitionAsync("0", EventPosition.Latest))
{
    Console.WriteLine(partitionEvent.Data.EventBody);
}
```

**Key Concept:** Partition ID is string ("0", "1", etc.). Use for parallel processing.

---

## Q99: Functions - Bindings Expressions Runtime

**Scenario:** Access binding metadata at runtime.

### ✅ Answer: Use IBinder

```csharp
[FunctionName("DynamicBinding")]
public static async Task Run(
    [QueueTrigger("commands")] string command,
    IBinder binder)
{
    var containerName = GetContainerFromCommand(command);
    
    using var writer = await binder.BindAsync<TextWriter>(
        new BlobAttribute($"{containerName}/output.txt", FileAccess.Write));
    
    await writer.WriteAsync("Dynamic output");
}
```

---

## Q100: Cosmos DB - Conflict Resolution

**Scenario:** Handle conflicts in multi-region writes.

### ✅ Answer: Configure conflict resolution policy

| Mode | Description |
|------|-------------|
| **Last Writer Wins (LWW)** | Highest _ts wins (default) |
| **Custom** | Stored procedure decides |
| **Custom (Async)** | Manual resolution via conflict feed |

```json
{
    "conflictResolutionPolicy": {
        "mode": "LastWriterWins",
        "conflictResolutionPath": "/_ts"
    }
}
```

---

## Q101: App Service - Health Check

**Scenario:** Automatically remove unhealthy instances.

### ✅ Answer: Configure health check

```bash
az webapp config set --resource-group myRG --name myApp --generic-configurations '{"healthCheckPath": "/health"}'
```

**Key Concept:** Instances failing health check are removed from load balancer after configurable threshold.

---

## Q102: Service Bus - Prefetch

**Scenario:** Improve message receive performance.

### ✅ Answer: Configure prefetch

```csharp
var options = new ServiceBusReceiverOptions
{
    PrefetchCount = 10 // Prefetch 10 messages
};
var receiver = client.CreateReceiver(queueName, options);
```

**Key Concept:** Prefetch = Background fetch, reduces latency. Don't set too high (memory).

---

## Q103: Functions - Extension Bundles

**Scenario:** Simplify binding extension management.

### ✅ Answer: Use extension bundles in host.json

```json
{
    "extensionBundle": {
        "id": "Microsoft.Azure.Functions.ExtensionBundle",
        "version": "[4.*, 5.0.0)"
    }
}
```

**Key Concept:** Bundles = Pre-packaged extensions, no need to manually install.

---

## Q104: Blob Storage - Customer-Managed Keys

**Scenario:** Encrypt blobs with your own keys.

### ✅ Answer: Configure CMK encryption

**Options:**
| Type | Key Location |
|------|-------------|
| Microsoft-managed | Azure manages (default) |
| Customer-managed (Key Vault) | Your Key Vault |
| Customer-provided | Per-request encryption key |

---

## Q105: Azure AD - On-Behalf-Of Flow

**Scenario:** Web API calls downstream API on behalf of user.

### ✅ Answer: Use OBO flow

```csharp
var app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithClientSecret(secret)
    .Build();

var result = await app.AcquireTokenOnBehalfOf(downstreamScopes, userAssertion)
    .ExecuteAsync();
```

**Key Concept:** OBO = API receives user token, exchanges for token to call another API.

---

## Q106: Cosmos DB - Request Unit (RU) Calculation

**Scenario:** Estimate RUs needed for operations.

### ✅ Answer: RU guidelines

| Operation | Approximate RUs |
|-----------|----------------|
| Point read (1KB) | 1 RU |
| Write (1KB) | 5-10 RUs |
| Query (depends) | 2.5+ RUs |

**Check actual:** `response.RequestCharge`

---

## Q107: APIM - Subscription Scopes

**Scenario:** Understand subscription key scopes.

### ✅ Answer:

| Scope | Access |
|-------|--------|
| **All APIs** | Every API in APIM |
| **Single API** | One specific API |
| **Product** | All APIs in product |

---

## Q108: Functions - Premium Plan Networking

**Scenario:** Function needs VNet access and private endpoints.

### ✅ Answer: Premium Plan with VNet Integration

**Features:**
- VNet integration (outbound)
- Private endpoints (inbound)
- Service endpoints
- No cold start

---

## Q109: Storage - Encryption Scopes

**Scenario:** Different encryption keys for different containers.

### ✅ Answer: Create encryption scopes

```bash
az storage account encryption-scope create \
    --account-name myaccount \
    --name scope1 \
    --key-source Microsoft.KeyVault \
    --key-uri "https://myvault.vault.azure.net/keys/mykey"
```

**Key Concept:** Each container/blob can use different encryption scope.

---

## Q110: Event Grid - CloudEvents Schema

**Scenario:** Use industry-standard event format.

### ✅ Answer: Configure CloudEvents schema

```json
{
    "specversion": "1.0",
    "type": "com.example.order.created",
    "source": "/orders/123",
    "id": "abc-123",
    "time": "2024-01-15T12:00:00Z",
    "data": { "orderId": 123 }
}
```

**Schemas available:** Event Grid schema (default), CloudEvents 1.0, Custom input

---

## Q111: App Service - ARR Affinity

**Scenario:** Should sticky sessions be enabled?

### ✅ Answer: Disable ARR Affinity for stateless apps

```bash
az webapp update --resource-group myRG --name myApp --client-affinity-enabled false
```

**Key Concept:**
- ARR Affinity = Sticky sessions (same instance)
- Disable for stateless apps (better load distribution)
- Enable only if app stores session state in memory

---

## Q112: Functions - Cold Start Mitigation

**Scenario:** Reduce function cold start time.

### ✅ Answer: Multiple options

| Solution | Plan |
|----------|------|
| Premium plan (pre-warmed) | Best option |
| Keep warm with timer trigger | Consumption workaround |
| Reduce package size | All plans |
| Use fewer dependencies | All plans |

---

## Q113: Cosmos DB - Hierarchical Partition Keys

**Scenario:** Multi-tenant app needs sub-partitioning.

### ✅ Answer: Use hierarchical partition keys (preview)

```csharp
var containerProperties = new ContainerProperties
{
    PartitionKeyPaths = new List<string> { "/tenantId", "/userId" }
};
```

**Key Concept:** Up to 3 levels of partition keys for multi-tenant scenarios.

---

## Q114: APIM - Quota Policy

**Scenario:** Limit total calls per day/week/month.

### ✅ Answer: Use quota-by-key policy

```xml
<quota-by-key calls="10000" 
              bandwidth="100000" 
              renewal-period="86400" 
              counter-key="@(context.Subscription.Id)" />
```

**Difference:**
- **Rate limit** = Per time window (e.g., 100/minute)
- **Quota** = Total over period (e.g., 10000/day)

---

## Q115: Blob Storage - Snapshot

**Scenario:** Create point-in-time backup of blob.

### ✅ Answer: Create blob snapshot

```csharp
var snapshot = await blobClient.CreateSnapshotAsync();
Console.WriteLine($"Snapshot: {snapshot.Value.Snapshot}");

// Access snapshot
var snapshotClient = blobClient.WithSnapshot(snapshot.Value.Snapshot);
```

**Key Concept:** Snapshots are read-only. Billed for changed blocks only.

---

## Q116: Service Bus - Auto-Forward

**Scenario:** Chain queues/topics together.

### ✅ Answer: Configure auto-forward

```bash
az servicebus queue create --name targetqueue --namespace-name myns --resource-group myRG

az servicebus queue update --name sourcequeue --namespace-name myns --resource-group myRG \
    --forward-to targetqueue
```

**Key Concept:** Messages automatically forwarded - no client code needed.

---

## Q117: Functions - HTTP Response Streaming

**Scenario:** Stream large responses without timeout.

### ✅ Answer: Enable response streaming (v4 runtime)

```csharp
[Function("StreamResponse")]
public async Task Run(
    [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req,
    FunctionContext context)
{
    var response = req.HttpContext.Response;
    response.ContentType = "text/plain";
    
    for (int i = 0; i < 100; i++)
    {
        await response.WriteAsync($"Line {i}\n");
        await response.Body.FlushAsync();
    }
}
```

---

## Q118: Azure AD - Token Lifetime

**Scenario:** Understand token validity periods.

### ✅ Answer:

| Token | Default Lifetime |
|-------|-----------------|
| Access token | 1 hour |
| ID token | 1 hour |
| Refresh token | 90 days |

**Key Concept:** Use refresh tokens to get new access tokens silently.

---

## Q119: Container Apps - Dapr Integration

**Scenario:** Enable service-to-service communication.

### ✅ Answer: Enable Dapr sidecar

```yaml
configuration:
  dapr:
    enabled: true
    appId: myapp
    appPort: 3000
```

**Dapr capabilities:** Service invocation, Pub/sub, State management, Secrets

---

## Q120: Event Hub - AMQP vs Kafka

**Scenario:** Which protocol to use?

### ✅ Answer:

| Protocol | When to Use |
|----------|-------------|
| **AMQP** | Default, .NET/Java SDK |
| **Kafka** | Existing Kafka apps (compatible endpoint) |
| **HTTPS** | Firewall restrictions |

---

## Q121: Functions - Durable Functions - Eternal Orchestrations

**Scenario:** Orchestration that runs indefinitely.

### ✅ Answer: Use ContinueAsNew

```csharp
[FunctionName("EternalOrchestrator")]
public static async Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    await context.CallActivityAsync("DoWork", null);
    await context.CreateTimer(context.CurrentUtcDateTime.AddMinutes(5), CancellationToken.None);
    
    context.ContinueAsNew(null); // Restart orchestration
}
```

**Key Concept:** ContinueAsNew prevents history from growing indefinitely.

---

## Q122: Key Vault - RBAC vs Access Policies

**Scenario:** Choose authorization model.

### ✅ Answer:

| Model | Description |
|-------|-------------|
| **Access Policies** | Legacy, per-vault permissions |
| **Azure RBAC** | Recommended, granular roles |

**RBAC Roles:**
- Key Vault Secrets User (read)
- Key Vault Secrets Officer (manage)
- Key Vault Administrator (full access)

---

## Q123: Blob Storage - Change Feed

**Scenario:** Track all blob changes for auditing.

### ✅ Answer: Enable change feed

```bash
az storage account blob-service-properties update \
    --account-name myaccount \
    --enable-change-feed true
```

**Key Concept:** Change feed = Ordered, guaranteed log of all changes. Stored in `$blobchangefeed` container.

---

## Q124: APIM - Transformation Policies

**Scenario:** Transform request/response XML to JSON.

### ✅ Answer: Use xml-to-json policy

```xml
<outbound>
    <xml-to-json kind="javascript-friendly" apply="always" />
</outbound>
```

**Other transformations:**
- `json-to-xml`
- `set-body` (template)
- `find-and-replace`

---

## Q125: Functions - Queue Trigger Concurrency

**Scenario:** Control how many messages processed in parallel.

### ✅ Answer: Configure in host.json

```json
{
    "extensions": {
        "queues": {
            "batchSize": 16,
            "newBatchThreshold": 8,
            "maxPollingInterval": "00:00:02"
        }
    }
}
```

**Key Concept:** `batchSize` = Max messages per instance. Scale out = more instances.

---

## Q126: Cosmos DB - Throughput Types

**Scenario:** Choose between throughput options.

### ✅ Answer:

| Type | Scope | Use Case |
|------|-------|----------|
| **Database throughput** | Shared across containers | Cost savings, predictable workload |
| **Container throughput** | Dedicated per container | Isolation, guaranteed performance |
| **Autoscale** | Automatic scaling | Variable workloads |
| **Serverless** | Pay per operation | Low/sporadic traffic |

---

## Q127: App Service - Network Injection

**Scenario:** Deploy App Service into VNet.

### ✅ Answer: Use App Service Environment (ASE) or VNet Integration

| Feature | ASE | VNet Integration |
|---------|-----|------------------|
| Inbound private | ✅ | Private Endpoints |
| Outbound VNet | ✅ | ✅ |
| Cost | Higher | Lower |
| Isolation | Full | Outbound only |

---

## Q128: Service Bus - Filters

**Scenario:** Route messages to specific subscriptions.

### ✅ Answer: Use subscription filters

```csharp
// SQL filter
var filter = new SqlRuleFilter("Priority = 'High'");

// Correlation filter (more efficient)
var correlationFilter = new CorrelationRuleFilter
{
    Subject = "HighPriority"
};

await adminClient.CreateRuleAsync(topicName, subscriptionName, 
    new CreateRuleOptions("HighPriorityRule", filter));
```

**Filter types:** SQL, Correlation (recommended), True (all messages)

---

## Q129: Functions - IP Restrictions

**Scenario:** Restrict function app access by IP.

### ✅ Answer: Configure access restrictions

```bash
az functionapp config access-restriction add \
    --resource-group myRG \
    --name myFunctionApp \
    --rule-name "AllowVNet" \
    --action Allow \
    --vnet-name myVNet \
    --subnet mySubnet \
    --priority 100
```

---

## Q130: Azure AD - Conditional Access - Sign-in Risk

**Scenario:** Require MFA for risky sign-ins.

### ✅ Answer: Use Identity Protection with Conditional Access

**Risk levels:**
- Low, Medium, High
- Real-time and offline detection

**Actions:**
- Block access
- Require MFA
- Require password change

---

## Q131: Cosmos DB - Bulk Operations

**Scenario:** Insert large number of documents efficiently.

### ✅ Answer: Enable AllowBulkExecution

```csharp
var options = new CosmosClientOptions
{
    AllowBulkExecution = true
};
var client = new CosmosClient(connectionString, options);

// Parallel inserts
var tasks = items.Select(item => container.CreateItemAsync(item, new PartitionKey(item.PartitionKey)));
await Task.WhenAll(tasks);
```

**Key Concept:** Bulk mode batches operations automatically.

---

## Q132: Event Grid - Event Domains

**Scenario:** Manage events for multi-tenant app.

### ✅ Answer: Use Event Domains

```bash
az eventgrid domain create --name myDomain --resource-group myRG --location eastus

# Create topic per tenant
az eventgrid domain topic create --domain-name myDomain --name tenant1 --resource-group myRG
```

**Key Concept:** Domain = Container for thousands of topics (one per tenant).

---

## Q133: Blob Storage - Soft Delete

**Scenario:** Recover deleted blobs.

### ✅ Answer: Enable blob soft delete

```bash
az storage blob service-properties delete-policy update \
    --account-name myaccount \
    --enable true \
    --days-retained 14
```

**Types:**
- Blob soft delete (individual blobs)
- Container soft delete (entire containers)

---

## Q134: APIM - Self-Hosted Gateway

**Scenario:** Run APIM gateway on-premises or other clouds.

### ✅ Answer: Deploy self-hosted gateway

**Steps:**
1. Create gateway resource in APIM
2. Deploy container image on-premises
3. Gateway syncs config from cloud

**Use cases:** Multi-cloud, low latency, data sovereignty

---

## Q135: Functions - SignalR Connection Info

**Scenario:** Client needs SignalR connection endpoint.

### ✅ Answer: Use negotiate function

```csharp
[FunctionName("negotiate")]
public static SignalRConnectionInfo Negotiate(
    [HttpTrigger(AuthorizationLevel.Anonymous)] HttpRequest req,
    [SignalRConnectionInfo(HubName = "chat", UserId = "{headers.x-ms-client-principal-id}")] 
    SignalRConnectionInfo connectionInfo)
{
    return connectionInfo;
}
```

---

## Q136: Cosmos DB - Analytical Store

**Scenario:** Run analytics without affecting transactional workload.

### ✅ Answer: Enable analytical store (Azure Synapse Link)

**Benefits:**
- Auto-sync to column store
- No impact on transactional RUs
- Query with Synapse Spark/SQL

---

## Q137: App Service - Deployment Center

**Scenario:** Set up CI/CD for web app.

### ✅ Answer: Configure Deployment Center

**Sources:**
- GitHub Actions
- Azure DevOps
- Bitbucket
- Local Git
- Container Registry (for containers)

---

## Q138: Service Bus - Geo-Disaster Recovery

**Scenario:** High availability across regions.

### ✅ Answer: Configure Geo-DR pairing

**Features:**
- Metadata replication (queues, topics, subscriptions)
- Manual failover
- Active-passive model

**Note:** Messages NOT replicated - only metadata.

---

## Q139: Functions - Table Storage Binding

**Scenario:** Read/write Azure Table Storage.

### ✅ Answer:

```csharp
[FunctionName("TableBinding")]
public static void Run(
    [QueueTrigger("myqueue")] string id,
    [Table("MyTable", "PartitionKey", "{queueTrigger}")] MyEntity entity,
    [Table("MyTable")] IAsyncCollector<MyEntity> outputTable)
{
    // entity = single read
    // outputTable = write multiple
}
```

---

## Q140: Azure AD - Authorization Code Flow with PKCE

**Scenario:** Secure SPA authentication.

### ✅ Answer: Use Authorization Code with PKCE

```javascript
const msalConfig = {
    auth: {
        clientId: "your-client-id",
        redirectUri: "http://localhost:3000"
    }
};

const pca = new msal.PublicClientApplication(msalConfig);

// PKCE is automatic with MSAL.js v2+
await pca.loginRedirect({ scopes: ["User.Read"] });
```

**Key Concept:** PKCE prevents authorization code interception. Required for SPAs.

---

## Q141: Container Registry - Geo-Replication

**Scenario:** Deploy images faster across regions.

### ✅ Answer: Enable geo-replication (Premium tier)

```bash
az acr replication create --registry myregistry --location westeurope
```

**Benefits:**
- Pull from nearest region
- Single image URL
- Automatic sync

---

## Q142: APIM - IP Filtering Policy

**Scenario:** Allow/deny requests by IP address.

### ✅ Answer: Use ip-filter policy

```xml
<ip-filter action="allow">
    <address-range from="10.0.0.0" to="10.0.0.255" />
    <address>203.0.113.1</address>
</ip-filter>
```

---

## Q143: Functions - Deployment Slots

**Scenario:** Test function before going live.

### ✅ Answer: Use deployment slots (Premium/Dedicated)

```bash
az functionapp deployment slot create --name myFunctionApp --resource-group myRG --slot staging
```

**Key Concept:** Same as App Service slots - swap for zero-downtime deployment.

---

## Q144: Cosmos DB - Item-Level TTL

**Scenario:** Different TTL for different documents.

### ✅ Answer: Set TTL per document

```json
{
    "id": "1",
    "ttl": 3600,    // Expires in 1 hour
    "data": "temp"
}

{
    "id": "2",
    "ttl": -1,      // Never expires (ignore container default)
    "data": "permanent"
}
```

---

## Q145: Event Hub - Capture Time/Size Window

**Scenario:** Control when captured data is written.

### ✅ Answer: Configure capture windows

```bash
az eventhubs eventhub update --name myhub --namespace-name myns --resource-group myRG \
    --capture-interval 300 \
    --capture-size-limit 314572800
```

**Triggers:** First of time interval OR size limit reached.

---

## Q146: App Service - Custom Containers

**Scenario:** Run custom Docker container in App Service.

### ✅ Answer: Deploy container to App Service

```bash
az webapp create --resource-group myRG --plan myPlan --name myApp \
    --deployment-container-image-name myregistry.azurecr.io/myimage:latest
```

**Key Concept:** Web App for Containers = PaaS + custom containers.

---

## Q147: Key Vault - Firewall Rules

**Scenario:** Restrict Key Vault access to specific networks.

### ✅ Answer: Configure network rules

```bash
az keyvault update --name myvault --resource-group myRG \
    --default-action Deny

az keyvault network-rule add --name myvault --resource-group myRG \
    --vnet-name myVNet --subnet mySubnet
```

---

## Q148: Service Bus - Message Lock

**Scenario:** Extend message processing time.

### ✅ Answer: Renew message lock

```csharp
// Auto-renew during processing
var options = new ServiceBusProcessorOptions
{
    MaxAutoLockRenewalDuration = TimeSpan.FromMinutes(10)
};

// Or manually renew
await receiver.RenewMessageLockAsync(message);
```

**Default lock:** 30 seconds (queue) or 1 minute (subscription)

---

## Q149: Functions - OpenAPI Definition

**Scenario:** Generate Swagger documentation for HTTP functions.

### ✅ Answer: Use Azure Functions OpenAPI Extension

```csharp
[FunctionName("GetProducts")]
[OpenApiOperation("GetProducts")]
[OpenApiResponseWithBody(HttpStatusCode.OK, "application/json", typeof(Product[]))]
public async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get")] HttpRequest req)
{
    // Implementation
}
```

---

## Q150: Cosmos DB - Transactional Batch

**Scenario:** All-or-nothing operations on same partition.

### ✅ Answer: Use TransactionalBatch

```csharp
var batch = container.CreateTransactionalBatch(new PartitionKey("pk1"));
batch.CreateItem(new { id = "1", pk = "pk1" });
batch.CreateItem(new { id = "2", pk = "pk1" });
batch.DeleteItem("3");

var response = await batch.ExecuteAsync();
// All succeed or all fail
```

**Key Concept:** All items must be in same partition. Max 100 operations, 2MB total.

---

## Q151: Blob Storage - Premium Performance

**Scenario:** Need low-latency blob access.

### ✅ Answer: Use Premium Block Blobs

**Comparison:**
| Tier | Latency | Use Case |
|------|---------|----------|
| Standard | Higher | General purpose |
| Premium | Low (~ms) | AI/ML, analytics, high IOPS |

**Note:** Premium = Block blobs only, no Hot/Cool/Archive tiers.

---

## Q152: APIM - Revisions and Versions

**Scenario:** When to use each?

### ✅ Answer:

| Concept | Use Case | Breaking? |
|---------|----------|-----------|
| **Revision** | Test changes safely | No |
| **Version** | New contract (v1→v2) | Yes |

**Revision:** Same URL, different backend. Version: Different URL.

---

## Q153: Azure AD - App Consent

**Scenario:** Admin needs to consent for all users.

### ✅ Answer: Admin consent

**Types:**
- User consent: Per-user
- Admin consent: Tenant-wide

```
https://login.microsoftonline.com/{tenant}/adminconsent
    ?client_id={client-id}
    &redirect_uri={redirect-uri}
```

---

## Q154: Functions - Scaling Behavior

**Scenario:** How functions scale automatically.

### ✅ Answer:

| Plan | Scaling |
|------|---------|
| Consumption | 0-200 instances, event-driven |
| Premium | 1-100 instances, pre-warmed |
| Dedicated | Manual or auto-scale rules |

**Triggers affect scaling:** HTTP, Queue length, Event Hub partitions

---

## Q155: Storage - Account Kind

**Scenario:** Choose storage account type.

### ✅ Answer:

| Kind | Use Case |
|------|----------|
| **StorageV2** | General purpose (recommended) |
| **BlobStorage** | Blob only (legacy) |
| **FileStorage** | Premium files |
| **BlockBlobStorage** | Premium blobs |

---

## Q156: Service Bus - Transfer Dead Letter Queue

**Scenario:** Message fails during auto-forward.

### ✅ Answer: Check transfer dead letter queue

**Path:** `<queue>/$Transfer/$DeadLetterQueue`

**Cause:** Destination quota exceeded, permission denied, etc.

---

## Q157: Container Apps - Secrets

**Scenario:** Store sensitive values for container app.

### ✅ Answer: Configure secrets

```yaml
properties:
  configuration:
    secrets:
      - name: db-password
        value: secret123
    containers:
      - name: myapp
        env:
          - name: DATABASE_PASSWORD
            secretRef: db-password
```

**Or reference Key Vault with managed identity.**

---

## Q158: Event Grid - Retry Policy

**Scenario:** Configure event delivery retries.

### ✅ Answer: Customize retry policy

```json
{
    "eventDeliverySchema": "EventGridSchema",
    "retryPolicy": {
        "maxDeliveryAttempts": 30,
        "eventTimeToLiveInMinutes": 1440
    }
}
```

**Default:** 30 attempts, 24 hours TTL, exponential backoff

---

## Q159: Functions - Method Routing

**Scenario:** Handle multiple HTTP methods in one function.

### ✅ Answer: Specify methods in trigger

```csharp
[FunctionName("CrudFunction")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", "put", "delete", 
     Route = "items/{id?}")] HttpRequest req,
    string id)
{
    switch (req.Method)
    {
        case "GET": return HandleGet(id);
        case "POST": return HandlePost(req);
        // etc.
    }
}
```

---

## Q160: Cosmos DB - Consistency vs Availability

**Scenario:** Trade-offs in distributed systems.

### ✅ Answer: CAP theorem implications

| Consistency | Availability | Partition Tolerance |
|-------------|--------------|---------------------|
| Strong | Lower | ✅ |
| Eventual | Higher | ✅ |

**Key Concept:** Cosmos DB always has partition tolerance. Choose consistency vs availability.

---

## Q161: App Service - Always On

**Scenario:** Prevent app from going idle.

### ✅ Answer: Enable Always On

```bash
az webapp config set --resource-group myRG --name myApp --always-on true
```

**Requires:** Basic tier or higher

**Use case:** WebJobs, SignalR, apps that can't handle cold start

---

## Q162: APIM - OAuth 2.0 Authorization Server

**Scenario:** Configure OAuth in Developer Portal.

### ✅ Answer: Add OAuth 2.0 authorization server

**Steps:**
1. Create authorization server in APIM
2. Configure grant types, token endpoints
3. Associate with API
4. Developers can test with OAuth in portal

---

## Q163: Functions - Cosmos DB Output Binding

**Scenario:** Write to Cosmos DB from function.

### ✅ Answer:

```csharp
[FunctionName("WriteToCosmosDB")]
public static void Run(
    [HttpTrigger] HttpRequest req,
    [CosmosDB(
        databaseName: "mydb",
        containerName: "items",
        Connection = "CosmosDBConnection")] out dynamic document)
{
    document = new { id = Guid.NewGuid().ToString(), data = "test" };
}
```

---

## Q164: Key Vault - Backup and Restore

**Scenario:** Protect Key Vault contents.

### ✅ Answer: Backup secrets/keys/certificates

```bash
# Backup
az keyvault secret backup --vault-name myvault --name mysecret --file backup.blob

# Restore (to SAME tenant)
az keyvault secret restore --vault-name myvault --file backup.blob
```

**Key Concept:** Backup can only be restored to same Azure tenant.

---

## Q165: Event Hub - Epoch Receiver

**Scenario:** Ensure only one receiver per partition.

### ✅ Answer: Use epoch-based receiver

```csharp
var options = new EventProcessorClientOptions
{
    // Epoch ensures exclusive access
    PartitionOwnershipExpirationInterval = TimeSpan.FromSeconds(30)
};
```

**Key Concept:** Higher epoch takes over partition from lower epoch.

---

## Q166: Service Bus - Message Sessions State

**Scenario:** Store state associated with session.

### ✅ Answer: Use session state

```csharp
var session = await receiver.AcceptSessionAsync("session-123");

// Set state
await session.SetSessionStateAsync(BinaryData.FromString("progress:50%"));

// Get state
var state = await session.GetSessionStateAsync();
```

**Use case:** Workflow progress, aggregation state

---

## Q167: Blob Storage - Point-in-Time Restore

**Scenario:** Restore blobs to previous state.

### ✅ Answer: Enable point-in-time restore

**Requirements:**
- Soft delete enabled
- Versioning enabled
- Change feed enabled

```bash
az storage blob restore --account-name myaccount \
    --time-to-restore "2024-01-15T10:00:00Z" \
    --blob-range '["",""]'  # All blobs
```

---

## Q168: Functions - Application Settings

**Scenario:** Store configuration values.

### ✅ Answer: Use app settings and local.settings.json

**local.settings.json:**
```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "...",
        "MyCustomSetting": "value"
    }
}
```

**Access in code:**
```csharp
var value = Environment.GetEnvironmentVariable("MyCustomSetting");
```

---

## Q169: Cosmos DB - Integrated Cache

**Scenario:** Cache frequently read data.

### ✅ Answer: Enable integrated cache (Dedicated Gateway)

**Benefits:**
- Automatic cache invalidation
- Reduced RU consumption
- No code changes

**Session consistency guaranteed for cached reads.**

---

## Q170: APIM - Error Handling Policy

**Scenario:** Return custom error response.

### ✅ Answer: Use on-error section

```xml
<policies>
    <inbound />
    <backend />
    <outbound />
    <on-error>
        <return-response>
            <set-status code="500" reason="Internal Error" />
            <set-body>{"error": "Something went wrong"}</set-body>
        </return-response>
    </on-error>
</policies>
```

---

## Q171: Container Apps - Revision Mode

**Scenario:** Control traffic between revisions.

### ✅ Answer: Configure revision mode

| Mode | Behavior |
|------|----------|
| **Single** | Only latest active |
| **Multiple** | Traffic split across revisions |

```yaml
configuration:
  activeRevisionsMode: multiple
  traffic:
    - revisionName: myapp--v1
      weight: 80
    - revisionName: myapp--v2
      weight: 20
```

---

## Q172: Azure AD - Device Code Flow

**Scenario:** Authenticate on device without browser.

### ✅ Answer: Use device code flow

```csharp
var result = await app.AcquireTokenWithDeviceCode(scopes, callback =>
{
    Console.WriteLine(callback.Message);
    // "To sign in, use a web browser to open..."
    return Task.CompletedTask;
}).ExecuteAsync();
```

**Use case:** CLI tools, IoT devices, smart TVs

---

## Q173: Storage Queue - Visibility Timeout

**Scenario:** Control when message becomes visible again.

### ✅ Answer: Set visibility timeout

```csharp
// Hide message for 5 minutes
await queueClient.ReceiveMessagesAsync(maxMessages: 1, 
    visibilityTimeout: TimeSpan.FromMinutes(5));

// Update timeout if processing takes longer
await queueClient.UpdateMessageAsync(message.MessageId, message.PopReceipt,
    visibilityTimeout: TimeSpan.FromMinutes(10));
```

---

## Q174: Functions - Timer Trigger Expression

**Scenario:** Schedule function execution.

### ✅ Answer: Use NCRONTAB expression

```json
{
    "schedule": "0 */5 * * * *"  // Every 5 minutes
}
```

**Format:** `{second} {minute} {hour} {day} {month} {day-of-week}`

| Expression | Schedule |
|------------|----------|
| `0 0 * * * *` | Every hour |
| `0 0 0 * * *` | Daily at midnight |
| `0 0 9 * * 1-5` | Weekdays at 9 AM |

---

## Q175: Cosmos DB - Cross-Partition Query

**Scenario:** Query without specifying partition key.

### ✅ Answer: Enable cross-partition query

```csharp
var options = new QueryRequestOptions
{
    MaxConcurrency = -1  // Maximum parallelism
};

var query = container.GetItemQueryIterator<Item>(
    "SELECT * FROM c WHERE c.status = 'active'",
    requestOptions: options);
```

**Warning:** Higher RU cost - use partition key when possible.

---

## Q176: App Service - VNET Integration

**Scenario:** Connect App Service to Azure resources in VNet.

### ✅ Answer: Enable VNet Integration

```bash
az webapp vnet-integration add --resource-group myRG --name myApp \
    --vnet myVNet --subnet mySubnet
```

**Types:**
- Regional VNet Integration (same region)
- Gateway-required VNet Integration (other regions, legacy)

---

## Q177: APIM - Rewrite URL Policy

**Scenario:** Change URL path before forwarding to backend.

### ✅ Answer: Use rewrite-uri policy

```xml
<inbound>
    <rewrite-uri template="/api/v2/{path}" />
    <!-- OR -->
    <set-backend-service base-url="https://api.backend.com/v2/" />
</inbound>
```

---

## Q178: Service Bus - Peek Lock vs Receive and Delete

**Scenario:** Choose receive mode.

### ✅ Answer:

| Mode | Behavior | Safety |
|------|----------|--------|
| **PeekLock** | Lock message, complete/abandon | Safe (default) |
| **ReceiveAndDelete** | Immediate delete | Fast, risk of loss |

```csharp
// PeekLock (default)
var receiver = client.CreateReceiver(queueName, new ServiceBusReceiverOptions
{
    ReceiveMode = ServiceBusReceiveMode.PeekLock
});
```

---

## Q179: Functions - Durable Functions Versioning

**Scenario:** Update orchestrator without breaking running instances.

### ✅ Answer: Use side-by-side versioning

```csharp
[FunctionName("OrchestratorV1")]
public static async Task RunV1([OrchestrationTrigger] IDurableOrchestrationContext ctx)
{
    // Old logic
}

[FunctionName("OrchestratorV2")]
public static async Task RunV2([OrchestrationTrigger] IDurableOrchestrationContext ctx)
{
    // New logic - new instances use this
}
```

**Key Concept:** Never change running orchestrator logic. Create new version.

---

## Q180: Blob Storage - Blob Types

**Scenario:** Choose correct blob type.

### ✅ Answer:

| Type | Max Size | Use Case |
|------|----------|----------|
| **Block blob** | 190.7 TB | Files, images, videos |
| **Append blob** | 195 GB | Logs (append only) |
| **Page blob** | 8 TB | VHDs, random access |

---

## Q181: Azure AD - Scopes vs Roles

**Scenario:** When to use which for authorization.

### ✅ Answer:

| Concept | Used For | Example |
|---------|----------|---------|
| **Scopes** | Delegated (user) permissions | User.Read |
| **App Roles** | Application permissions | Admin, Reader |

**Scopes:** What app can do on behalf of user
**Roles:** What user/app can do in the app

---

## Q182: Event Hub - Auto-Inflate

**Scenario:** Automatically scale throughput units.

### ✅ Answer: Enable auto-inflate

```bash
az eventhubs namespace update --name myns --resource-group myRG \
    --enable-auto-inflate \
    --maximum-throughput-units 20
```

**Key Concept:** Scales UP automatically. Manual scale down.

---

## Q183: Key Vault - Purge Protection

**Scenario:** Prevent permanent deletion of vault.

### ✅ Answer: Enable purge protection

```bash
az keyvault update --name myvault --enable-purge-protection true
```

**Key Concept:** Once enabled, cannot be disabled. Prevents malicious deletion.

---

## Q184: Functions - HTTP Trigger Route Templates

**Scenario:** Create RESTful routes.

### ✅ Answer: Use route parameter

```csharp
[FunctionName("GetProduct")]
public static IActionResult Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", 
     Route = "products/{category}/{id:int}")] HttpRequest req,
    string category,
    int id)
{
    // category and id extracted from URL
}
```

**Constraints:** `{id:int}`, `{name:alpha}`, `{id:guid}`

---

## Q185: Cosmos DB - SDK Connection Mode

**Scenario:** Choose connection mode for performance.

### ✅ Answer:

| Mode | Performance | Use Case |
|------|-------------|----------|
| **Gateway** | Lower | Firewalls, single endpoint |
| **Direct** | Higher | Best performance (default) |

```csharp
var options = new CosmosClientOptions
{
    ConnectionMode = ConnectionMode.Direct // Default
};
```

---

## Q186: App Service - Kudu

**Scenario:** Access deployment and diagnostic tools.

### ✅ Answer: Use Kudu (Advanced Tools)

**Access:** `https://<app-name>.scm.azurewebsites.net`

**Features:**
- Debug console (CMD/PowerShell)
- Process explorer
- Log streaming
- File browser

---

## Q187: APIM - Inbound/Outbound/Backend

**Scenario:** When does each policy section execute?

### ✅ Answer:

```
Client → [INBOUND] → APIM → [BACKEND] → Backend Service
                                              ↓
Client ← [OUTBOUND] ←───────────────── Response
```

| Section | Timing |
|---------|--------|
| **Inbound** | Before sending to backend |
| **Backend** | Modify backend call |
| **Outbound** | Before sending to client |
| **On-error** | When error occurs |

---

## Q188: Service Bus - Maximum Message Size

**Scenario:** Send large messages.

### ✅ Answer:

| Tier | Max Message Size |
|------|-----------------|
| Standard | 256 KB |
| Premium | 100 MB |

**Alternative:** Claim check pattern (store payload in blob, send reference)

---

## Q189: Functions - Isolated Process Model

**Scenario:** Run function in separate process.

### ✅ Answer: Use isolated worker model

**Benefits:**
- Full control over dependencies
- Latest .NET versions
- Middleware support

```csharp
var host = new HostBuilder()
    .ConfigureFunctionsWorkerDefaults()
    .ConfigureServices(services =>
    {
        services.AddApplicationInsightsTelemetryWorkerService();
    })
    .Build();
```

---

## Q190: Storage - Data Lake Gen2

**Scenario:** Hierarchical namespace for big data.

### ✅ Answer: Enable hierarchical namespace

**Features:**
- True directories (not virtual)
- Atomic rename operations
- ACL-based permissions
- Compatible with blob APIs

**Use case:** Analytics, big data, Hadoop

---

## Q191: Azure AD - Certificate Credentials

**Scenario:** Authenticate without client secret.

### ✅ Answer: Use certificate authentication

```csharp
var certificate = new X509Certificate2("cert.pfx", "password");

var app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithCertificate(certificate)
    .WithAuthority(authority)
    .Build();
```

**More secure than secrets** - no secret to leak.

---

## Q192: Cosmos DB - Unique Keys

**Scenario:** Ensure uniqueness within partition.

### ✅ Answer: Define unique key policy

```json
{
    "uniqueKeyPolicy": {
        "uniqueKeys": [
            { "paths": ["/email"] },
            { "paths": ["/firstName", "/lastName"] }
        ]
    }
}
```

**Key Concept:** Unique within partition, not globally. Set at container creation.

---

## Q193: Event Grid - System Topics vs Custom Topics

**Scenario:** When to use which.

### ✅ Answer:

| Type | Source | Example |
|------|--------|---------|
| **System Topic** | Azure services | Blob created, resource changed |
| **Custom Topic** | Your app | Order placed, user registered |

---

## Q194: Functions - Host.json Global Settings

**Scenario:** Configure function app behavior.

### ✅ Answer: Key host.json settings

```json
{
    "version": "2.0",
    "logging": {
        "logLevel": {
            "default": "Information",
            "Function": "Warning"
        }
    },
    "functionTimeout": "00:10:00",
    "healthMonitor": {
        "enabled": true
    }
}
```

---

## Q195: Container Registry - Webhooks

**Scenario:** Trigger action when image pushed.

### ✅ Answer: Configure ACR webhook

```bash
az acr webhook create --registry myregistry --name mywebhook \
    --uri https://myservice.com/webhook \
    --actions push delete
```

**Events:** push, delete, quarantine, chart_push, chart_delete

---

## Q196: APIM - Response Body Modification

**Scenario:** Modify JSON response.

### ✅ Answer: Use set-body policy with Liquid template

```xml
<outbound>
    <set-body template="liquid">
    {
        "items": [
        {% for item in body.data %}
            {
                "id": "{{ item.id }}",
                "name": "{{ item.name }}"
            }{% unless forloop.last %},{% endunless %}
        {% endfor %}
        ]
    }
    </set-body>
</outbound>
```

---

## Q197: App Service - Managed Certificates

**Scenario:** Free SSL certificate for custom domain.

### ✅ Answer: Create App Service Managed Certificate

```bash
az webapp config ssl create --resource-group myRG --name myApp \
    --hostname www.contoso.com
```

**Benefits:** Free, auto-renew. **Limitation:** No wildcard, no export.

---

## Q198: Service Bus - Batch Send

**Scenario:** Send multiple messages efficiently.

### ✅ Answer: Use message batch

```csharp
var batch = await sender.CreateMessageBatchAsync();

batch.TryAddMessage(new ServiceBusMessage("Message 1"));
batch.TryAddMessage(new ServiceBusMessage("Message 2"));
batch.TryAddMessage(new ServiceBusMessage("Message 3"));

await sender.SendMessagesAsync(batch);
```

**Key Concept:** Batch = Single network call, better performance.

---

## Q199: Functions - Binding Data Types

**Scenario:** What types can bindings use?

### ✅ Answer:

| Binding | Supported Types |
|---------|----------------|
| Blob | Stream, string, byte[], BlobClient |
| Queue | string, byte[], POCO, CloudQueueMessage |
| HTTP | HttpRequest, POCO |
| Cosmos | Document, POCO, JObject |

---

## Q200: Azure AD - Logout

**Scenario:** Implement single sign-out.

### ✅ Answer: Redirect to logout endpoint

```
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/logout
    ?post_logout_redirect_uri={uri}
```

**Key Concept:** Clear session + redirect to Azure AD logout.

---

## Q201: Blob Storage - Blob Versioning

**Scenario:** Automatically maintain previous versions.

### ✅ Answer: Enable versioning

```bash
az storage account blob-service-properties update \
    --account-name myaccount \
    --enable-versioning true
```

**Access version:**
```csharp
var versionedBlob = containerClient.GetBlobClient("myblob").WithVersion(versionId);
```

---

## Q202: Cosmos DB - Server-Side Programming

**Scenario:** Compare server-side options.

### ✅ Answer:

| Type | Use Case | Partition Scope |
|------|----------|-----------------|
| **Stored Procedures** | Transactions | Single partition |
| **Triggers** | Pre/post operations | Single partition |
| **UDFs** | Query functions | N/A |

---

## Q203: Functions - Run From Package

**Scenario:** Deploy function as read-only package.

### ✅ Answer: Configure WEBSITE_RUN_FROM_PACKAGE

```json
{
    "WEBSITE_RUN_FROM_PACKAGE": "1"  // Use deployment package
    // OR URL to package
    "WEBSITE_RUN_FROM_PACKAGE": "https://storage.blob/package.zip?sas"
}
```

**Benefits:** Faster cold start, reliable deployments.

---

## Q204: APIM - Limit Call Rate by Key

**Scenario:** Different rate limits for different callers.

### ✅ Answer: Use rate-limit-by-key with custom key

```xml
<rate-limit-by-key 
    calls="10" 
    renewal-period="60"
    counter-key="@(context.Request.Headers.GetValueOrDefault("X-Customer-Id","default"))" />
```

---

## Q205: Azure AD - Permissions Classification

**Scenario:** Understand permission types.

### ✅ Answer:

| Type | Requested By | Consent |
|------|--------------|---------|
| **Delegated** | Apps acting as user | User or admin |
| **Application** | Apps acting as themselves | Admin only |

---

## Q206: Event Hub - Schema Registry

**Scenario:** Validate event schemas.

### ✅ Answer: Use Azure Schema Registry

**Benefits:**
- Schema versioning
- Validation at produce/consume
- Avro, JSON Schema support

---

## Q207: Service Bus - Topic Subscriptions

**Scenario:** Multiple consumers for same message.

### ✅ Answer: Use topics with subscriptions

```bash
# Create topic
az servicebus topic create --name mytopic --namespace-name myns

# Create subscriptions (each gets copy)
az servicebus topic subscription create --topic-name mytopic --name sub1 --namespace-name myns
az servicebus topic subscription create --topic-name mytopic --name sub2 --namespace-name myns
```

**Key Concept:** Pub/sub pattern. Each subscription = independent consumer.

---

## Q208: Key Vault - Secret Rotation

**Scenario:** Automatically rotate secrets.

### ✅ Answer: Use Event Grid for rotation notification

**Pattern:**
1. Key Vault emits "SecretNearExpiry" event
2. Event Grid triggers Function
3. Function rotates secret

---

## Q209: Functions - Triggers Comparison

**Scenario:** Quick reference for trigger selection.

### ✅ Answer:

| Scenario | Trigger |
|----------|---------|
| REST API | HTTP |
| Scheduled task | Timer |
| File uploaded | Blob |
| Message queue | Queue/Service Bus |
| Database changes | Cosmos DB |
| Real-time streaming | Event Hub |
| Event reaction | Event Grid |

---

## Q210: Cosmos DB - Throughput Calculator

**Scenario:** Estimate required RUs.

### ✅ Answer: RU estimation guidelines

| Operation | Est. RUs (1KB item) |
|-----------|---------------------|
| Point read | 1 RU |
| Point write | 5-10 RU |
| Query (simple) | 2.5+ RU |
| Query (complex) | 10-100+ RU |

**Tool:** Azure Cosmos DB Capacity Calculator

---

## Q211: App Service - Blue-Green Deployment

**Scenario:** Zero-downtime deployment pattern.

### ✅ Answer: Use deployment slots

1. Deploy to staging slot (blue)
2. Test staging
3. Swap staging ↔ production
4. Production now has new code (green)
5. Old code in staging for rollback

---

## Q212: Container Apps - Environment

**Scenario:** Share settings across container apps.

### ✅ Answer: Use Container Apps Environment

**Shared:**
- Virtual network
- Log Analytics workspace
- Dapr configuration

```bash
az containerapp env create --name myenv --resource-group myRG \
    --location eastus
```

---

## Q213: Storage - Redundancy Options

**Scenario:** Choose redundancy level.

### ✅ Answer:

| Type | Copies | Regions | Durability |
|------|--------|---------|------------|
| LRS | 3 | 1 | 99.999999999% |
| ZRS | 3 | 1 (zones) | 99.9999999999% |
| GRS | 6 | 2 | 99.99999999999999% |
| GZRS | 6 | 2 (zones) | Highest |

---

## Q214: APIM - Correlation ID

**Scenario:** Track requests across systems.

### ✅ Answer: Use correlation-id policy

```xml
<set-header name="x-correlation-id" exists-action="skip">
    <value>@(context.RequestId.ToString())</value>
</set-header>
```

---

## Q215: Functions - Cosmos DB Trigger Lease

**Scenario:** Understand lease collection purpose.

### ✅ Answer: Lease collection stores

- Checkpoint position
- Partition ownership
- Enables multiple instances

**Key Concept:** Without leases, no reliable change feed processing.

---

## Q216: Azure AD - App Registration vs Enterprise App

**Scenario:** Difference between the two.

### ✅ Answer:

| Concept | Purpose |
|---------|---------|
| **App Registration** | Define your app (client ID, secrets, permissions) |
| **Enterprise App** | Instance in tenant (service principal, user assignments) |

---

## Q217: Blob Storage - Access Control

**Scenario:** Compare access methods.

### ✅ Answer:

| Method | Scope | Duration |
|--------|-------|----------|
| Account Key | Full access | Permanent |
| SAS | Limited | Time-bound |
| Azure AD | RBAC | Session |
| Anonymous | Container/blob | Permanent |

**Recommendation:** Azure AD > SAS > Key

---

## Q218: Service Bus - Express Entities

**Scenario:** Lower latency messaging.

### ✅ Answer: Enable express mode (Standard tier)

**Behavior:** Messages kept in memory first, then persisted.

**Trade-off:** Higher throughput vs slight durability risk.

---

## Q219: Functions - Static Web Apps Integration

**Scenario:** Serverless API for static website.

### ✅ Answer: Use managed functions in Static Web Apps

**Benefits:**
- Auto-linked to static app
- Same domain (/api/*)
- No separate Function app needed

---

## Q220: Cosmos DB - Autoscale vs Manual

**Scenario:** When to use which.

### ✅ Answer:

| Mode | Best For |
|------|----------|
| **Manual** | Predictable workloads |
| **Autoscale** | Variable workloads |
| **Serverless** | Intermittent workloads |

**Autoscale:** 10-100% of max RU/s, billed for max used per hour.

---

## Q221: App Service - Virtual Applications

**Scenario:** Host multiple apps under one App Service.

### ✅ Answer: Configure virtual applications

**Path mapping:**
- `/` → site\wwwroot
- `/api` → site\api

---

## Q222: Event Hub - Retention Period

**Scenario:** How long events are stored.

### ✅ Answer:

| Tier | Retention |
|------|-----------|
| Basic | 1 day |
| Standard | 1-7 days |
| Premium/Dedicated | Up to 90 days |

**Alternative:** Capture to blob for long-term storage.

---

## Q223: Key Vault - Access from App Service

**Scenario:** Best way to access Key Vault.

### ✅ Answer: Use Managed Identity + Key Vault Reference

**App setting:**
```
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)
```

**Benefits:** No code changes, auto-refresh.

---

## Q224: APIM - Synthetic GraphQL

**Scenario:** Create GraphQL from REST APIs.

### ✅ Answer: Import GraphQL schema, configure resolvers

**Features:**
- GraphQL passthrough
- Synthetic (multiple REST → GraphQL)
- Validation and introspection

---

## Q225: Functions - Dapr Bindings

**Scenario:** Use Dapr with Azure Functions.

### ✅ Answer: Dapr input/output bindings

```csharp
[FunctionName("SaveState")]
public static void Run(
    [DaprServiceInvocationTrigger] string payload,
    [DaprState("statestore", Key = "mykey")] out string state)
{
    state = payload;
}
```

---

## Q226: Storage - Firewall Rules

**Scenario:** Restrict storage account access.

### ✅ Answer: Configure network rules

```bash
az storage account update --name myaccount \
    --default-action Deny \
    --bypass AzureServices

az storage account network-rule add --account-name myaccount \
    --vnet-name myVNet --subnet mySubnet
```

---

## Q227: Azure AD - Claims Mapping

**Scenario:** Include custom claims in token.

### ✅ Answer: Configure optional claims in manifest

```json
"optionalClaims": {
    "idToken": [
        { "name": "email", "essential": true },
        { "name": "upn" }
    ],
    "accessToken": [
        { "name": "groups" }
    ]
}
```

---

## Q228: Cosmos DB - SDK Best Practices

**Scenario:** Optimize SDK usage.

### ✅ Answer:

1. **Singleton CosmosClient** - Reuse across requests
2. **Direct mode** - For best performance
3. **Stream API** - Avoid serialization overhead
4. **Bulk execution** - For batch operations
5. **Partition key in queries** - Avoid cross-partition

---

## Q229: Service Bus - Premium Features

**Scenario:** When to use Premium tier.

### ✅ Answer:

| Feature | Standard | Premium |
|---------|----------|---------|
| Max message | 256 KB | 100 MB |
| Dedicated resources | No | Yes |
| VNet integration | No | Yes |
| Geo-DR | Limited | Full |
| Throughput | Variable | Predictable |

---

## Q230: Functions - Best Practices Summary

**Scenario:** Key recommendations.

### ✅ Answer:

1. **Keep functions small** - Single responsibility
2. **Avoid long-running** - Use Durable Functions
3. **Use async** - Don't block threads
4. **Minimize dependencies** - Faster cold starts
5. **Use managed identity** - No secrets in code
6. **Configure retries** - Handle transient failures
7. **Monitor with App Insights** - Observe behavior

---

# 📚 Quick Reference Tables

## Azure Messaging Comparison

| Service | Pattern | Ordering | Max Size |
|---------|---------|----------|----------|
| Storage Queue | Queue | Best-effort | 64 KB |
| Service Bus Queue | Queue | FIFO (sessions) | 256KB/100MB |
| Service Bus Topic | Pub/Sub | FIFO (sessions) | 256KB/100MB |
| Event Hub | Streaming | Per partition | 1 MB |
| Event Grid | Events | No | 1 MB |

## Function Hosting Plans

| Plan | Scale | Cold Start | Price |
|------|-------|------------|-------|
| Consumption | Auto (0-200) | Yes | Per execution |
| Premium | Auto (1-100) | No | Per instance |
| Dedicated | Manual/Auto | No | Always on |

## Cosmos DB Consistency Quick Guide

| Level | Read Your Writes | Monotonic Reads | RU Cost |
|-------|------------------|-----------------|---------|
| Strong | ✅ | ✅ | Highest |
| Bounded | ✅ (within bound) | ✅ | High |
| Session | ✅ (same session) | ✅ (same session) | Medium |
| Prefix | ❌ | ✅ | Low |
| Eventual | ❌ | ❌ | Lowest |

---
