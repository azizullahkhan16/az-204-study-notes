# AZ-204: Azure Functions - Complete Study Notes

## Learning Path Overview
This learning path covers implementing Azure Functions with 2 modules:
1. Explore Azure Functions
2. Develop Azure Functions

---

## MODULE 1: EXPLORE AZURE FUNCTIONS

### 1.1 Introduction to Azure Functions

**What are Azure Functions?**
- Serverless compute service that lets you run event-driven code
- Execute code on-demand without managing infrastructure
- Automatically scales based on demand
- Pay only for compute time consumed
- Part of Azure's serverless offerings

**Key Characteristics:**
- **Event-driven**: Code runs in response to events/triggers
- **Stateless (default)**: Each execution is independent
- **Stateful (Durable Functions)**: Can maintain state across executions
- **Serverless**: No server management required
- **Automatic scaling**: Scales automatically based on load
- **Flexible hosting**: Multiple hosting options available

**Supported Languages:**
- C# (.NET)
- Java
- JavaScript/TypeScript (Node.js)
- Python
- PowerShell
- Custom handlers (Go, Rust, any language with HTTP primitives)

**Common Use Cases:**

1. **Data Processing**:
   - File/blob processing
   - Data transformation
   - Real-time stream processing
   - ETL operations

2. **Webhooks and APIs**:
   - HTTP APIs and REST endpoints
   - Webhook receivers
   - Microservices

3. **Scheduled Tasks**:
   - Cleanup jobs
   - Batch processing
   - Scheduled reports
   - Database maintenance

4. **System Integration**:
   - Connect disparate systems
   - Message queue processing
   - Event processing
   - IoT data processing

5. **Automation**:
   - DevOps automation
   - Infrastructure management
   - Notification systems
   - Alert processing

**Benefits:**

1. **Cost-Effective**:
   - Pay per execution
   - No idle compute costs (Consumption plan)
   - Free grant included monthly

2. **Developer Productivity**:
   - Focus on code, not infrastructure
   - Multiple language support
   - Rich development tools
   - Local development support

3. **Scalability**:
   - Automatic scaling
   - Handle variable workloads
   - Scale to zero when idle

4. **Integration**:
   - Built-in triggers and bindings
   - Connect to Azure services easily
   - Third-party integrations

### 1.2 Azure Functions Hosting Options

Azure Functions offers multiple hosting plans, each with different features, scaling behavior, and pricing models.

#### 1. **Consumption Plan (Dynamic Hosting)**

**Overview:**
- True serverless plan
- Pay only for execution time
- Automatic scaling (scale to zero)
- Default plan

**Key Features:**
- **Automatic Scaling**: Scales out automatically based on events
- **Scale to Zero**: When idle, no instances running (no cost)
- **Dynamic Resource Allocation**: Resources allocated per execution
- **Timeout**: 5 minutes (default), configurable up to 10 minutes
- **Memory**: Up to 1.5 GB per instance
- **Instance Limit**: 200 instances (default), 300 in portal

**Pricing:**
- Pay per execution
- Pay per compute time (GB-s)
- **Monthly Free Grant**:
  - 1 million executions
  - 400,000 GB-s compute time

**Best For:**
- Sporadic or unpredictable workloads
- Short-running functions
- Cost optimization when idle
- Development and testing

**Limitations:**
- Cold starts possible
- 10-minute maximum timeout
- Limited instance memory
- No VNet integration (use Premium for VNet)

**Cold Start:**
- When function app scales to zero, first request experiences delay
- App and runtime must be loaded
- Typically 1-3 seconds but can be longer for large dependencies
- Subsequent requests are faster (warm instances)

---

#### 2. **Flex Consumption Plan** (Newer)

**Overview:**
- Enhanced consumption plan
- Fast horizontal scaling
- Flexible compute options
- Virtual network integration support

**Key Features:**
- **Fast Scaling**: Optimized for rapid scale-out
- **Per-Instance Concurrency**: Configure concurrent executions per instance
- **VNet Integration**: Full VNet support
- **Always Ready Instances**: Optional pre-warmed instances
- **Identity-Based Connections**: Microsoft Entra ID authentication

**Billing Modes:**
1. **On-Demand**: Pay per execution and resource consumption
2. **Always Ready**: Pay for baseline provisioned capacity + usage

**Pricing:**
- **Monthly Free Grant**:
  - 250,000 executions
  - 100,000 GB-s resource consumption

**Configuration Options:**
- Instance memory size (configurable)
- Maximum instance count
- Per-instance concurrency
- Always Ready instance count

**Best For:**
- Apps requiring VNet integration
- Fast, unpredictable scaling needs
- Identity-based connections
- Modern serverless apps

**Advantages over Consumption:**
- Faster scaling
- VNet integration
- Better concurrency control
- More flexible resource allocation

---

#### 3. **Premium Plan (Elastic Premium)**

**Overview:**
- Enhanced performance with pre-warmed instances
- No cold starts
- VNet connectivity
- Unlimited execution duration

**SKUs:**
- **EP1**: 1 core, 3.5 GB RAM
- **EP2**: 2 cores, 7 GB RAM
- **EP3**: 4 cores, 14 GB RAM

**Key Features:**
- **Pre-Warmed Instances**: Always-ready instances eliminate cold starts
- **VNet Integration**: Full VNet connectivity
- **Unlimited Duration**: Run duration defaults to 30 minutes, configurable to unbounded
- **Faster Scaling**: Rapid scale-out when needed
- **Private Site Access**: Private endpoints supported
- **Larger Instances**: More CPU and memory

**Pricing:**
- Pay per core-seconds and memory allocated
- Always running at least one instance (minimum cost)
- No execution charges (unlike Consumption)
- More expensive than Consumption but predictable

**Scaling:**
- Minimum instances: 1 (configurable)
- Maximum instances: 20-30 (Linux can go to 100 in some regions)
- Event-driven auto-scaling

**Best For:**
- Production applications
- Latency-sensitive workloads
- Apps requiring VNet connectivity
- Long-running functions
- Avoiding cold starts

**Premium Features Summary:**
- No cold starts (pre-warmed instances)
- VNet integration
- Unlimited execution duration (with caveats)
- Premium hardware (faster CPUs)
- More memory options
- Better for production workloads

**Important Limitations:**
- Minimum monthly cost (at least 1 instance always running)
- Platform updates can terminate after 10 minutes (grace period)
- Scale-in can shutdown after 60 minutes
- Idle timer stops worker after 60 minutes with no executions

---

#### 4. **Dedicated (App Service) Plan**

**Overview:**
- Run functions on dedicated VMs
- Same infrastructure as App Service
- Full control over scaling
- Predictable billing

**Key Features:**
- **Dedicated VMs**: Functions run on dedicated resources
- **Manual/Auto Scaling**: Configure scale rules or scale manually
- **Always On**: Must enable Always On for continuous availability
- **No Execution Limits**: No timeout limits (unbounded)
- **Full VNet Integration**: Complete network isolation
- **Deployment Slots**: Support for staging/production slots

**Pricing:**
- Pay for App Service plan (not per execution)
- Same pricing as App Service
- Predictable monthly cost
- Good for utilizing existing App Service capacity

**Best For:**
- Long-running functions
- Predictable, continuous workloads
- When you have underutilized App Service plans
- Need for deployment slots
- Regulatory requirements for dedicated resources

**Scaling:**
- Manual scaling (increase/decrease instance count)
- Auto-scaling based on metrics (CPU, memory, etc.)
- Scale settings apply to entire plan

**Configuration:**
- **Always On**: Must be enabled (prevents idle timeout)
- **ARR Affinity**: Session affinity if needed
- Standard App Service plan features apply

---

#### 5. **Container Apps**

**Overview:**
- Run containerized function apps
- Kubernetes-based hosting
- Full container customization

**Key Features:**
- Container customization
- Kubernetes orchestration
- Microservices support
- Dapr integration

**Best For:**
- Custom container requirements
- Microservices architecture
- When you need full OS control

**Scaling:**
- Default: 10 instances
- Maximum: 1,000 instances
- Event-driven scaling

---

#### 6. **Kubernetes (KEDA)**

**Overview:**
- Run functions on Kubernetes clusters
- On-premises or cloud
- Complete control over infrastructure

**Supported Platforms:**
- Azure Kubernetes Service (AKS)
- On-premises Kubernetes
- Azure Arc-enabled Kubernetes
- Azure IoT Edge

**Best For:**
- Existing Kubernetes infrastructure
- On-premises requirements
- Edge computing scenarios
- Hybrid cloud deployments

**Note:**
- Customer manages Kubernetes infrastructure
- Responsible for scaling node pools
- Billing for underlying infrastructure

---

### 1.3 Hosting Plan Comparison

| Feature | Consumption | Flex Consumption | Premium | Dedicated | Container Apps |
|---------|-------------|------------------|---------|-----------|----------------|
| **Pricing** | Per execution + GB-s | Per execution + GB-s | Per core/memory | App Service plan | Container resources |
| **Scaling** | Automatic (0-200) | Automatic, configurable | Automatic (1-30+) | Manual/auto | Automatic (0-1000) |
| **Cold Starts** | Yes | Possible | No (pre-warmed) | No (Always On) | Possible |
| **Timeout** | 5 min (10 max) | 30 min default | 30 min (unbounded) | Unbounded | 30 min default |
| **VNet** | No | Yes | Yes | Yes | Yes |
| **Free Grant** | 1M exec, 400K GB-s | 250K exec, 100K GB-s | No | No | No |
| **Min Instances** | 0 | 0 | 1+ | 1+ | 0 |
| **Max Instances** | 200 (300 portal) | Configurable | 20-100 | Plan limit | 1000 |
| **Best For** | Sporadic loads | Modern serverless | Production, low latency | Continuous, predictable | Containers |

**Key Differences Summary:**

**Scale to Zero:**
- Consumption: Yes
- Flex Consumption: Yes
- Premium: No (minimum 1 instance)
- Dedicated: No
- Container Apps: Yes

**Cold Start Mitigation:**
- Consumption: No mitigation
- Flex Consumption: Always Ready instances (optional)
- Premium: Pre-warmed instances
- Dedicated: Always On setting
- Container Apps: Depends on configuration

**Cost Model:**
- Consumption: Pay per use (most cost-effective for sporadic)
- Flex Consumption: Pay per use with optional baseline
- Premium: Always-on cost (predictable but higher minimum)
- Dedicated: Fixed monthly cost
- Container Apps: Container-based pricing

### 1.4 Azure Functions Runtime and Versions

**Runtime Versions:**

**Functions Runtime 4.x (Current, Recommended):**
- .NET 6, .NET 7, .NET 8 (isolated worker or in-process for .NET 6)
- Node.js 18, 20
- Python 3.9, 3.10, 3.11
- Java 11, 17, 21
- PowerShell 7.2, 7.4
- End of support: To be determined

**Functions Runtime 3.x:**
- .NET Core 3.1, .NET 5 (in-process)
- Node.js 10, 12, 14
- Python 3.6, 3.7, 3.8, 3.9
- Java 8, 11
- PowerShell 7.0
- **End of support**: December 13, 2022 (extended support available)

**Functions Runtime 2.x:**
- .NET Core 2.x
- Node.js 8, 10
- Python 3.6, 3.7
- Java 8
- **End of support**: December 3, 2021

**Functions Runtime 1.x:**
- .NET Framework 4.8
- Node.js 6 (Windows only)
- **End of support**: September 14, 2026 (maintenance only)
- Windows only

**Important Notes:**
- Always use the latest runtime version (4.x)
- Runtime 1.x supports only .NET Framework (not .NET Core)
- Runtime 4.x provides best performance and latest features
- Migration paths available from older versions

### 1.5 Triggers and Bindings

**What are Triggers?**
- Event that causes function to execute
- Every function must have exactly ONE trigger
- Trigger defines how function is invoked
- Contains data about the event (passed to function as parameters)

**What are Bindings?**
- Declarative way to connect to resources
- Simplifies connecting to data sources
- **Input Bindings**: Read data from external sources
- **Output Bindings**: Write data to external destinations
- Optional (functions can have zero or more bindings)

**Benefits of Triggers and Bindings:**
- No need to write connection code
- Declarative (configured, not coded)
- Simplified data access
- Handles connection management
- Retries and error handling built-in

**Binding Direction:**
- **in**: Input binding (read data)
- **out**: Output binding (write data)
- **inout**: Both (read and write) - rare, some triggers support this

**Common Triggers:**

1. **HTTP Trigger**:
   - Triggered by HTTP request
   - Build REST APIs and webhooks
   - Supports GET, POST, PUT, DELETE, etc.
   - Returns HTTP response

2. **Timer Trigger**:
   - Scheduled execution (cron expressions)
   - Run on schedule (e.g., daily, hourly)
   - CRON format: `{second} {minute} {hour} {day} {month} {day-of-week}`
   - Example: `0 */5 * * * *` = every 5 minutes

3. **Blob Trigger**:
   - Triggered when blob is added/updated in Azure Storage
   - Process uploaded files
   - Monitors blob container for changes
   - Can cause delays (polling-based)

4. **Queue Trigger**:
   - Triggered by message in Azure Storage Queue
   - Processes queue messages
   - Supports parallel processing
   - Poison message handling

5. **Event Grid Trigger**:
   - Event-driven, reactive
   - Low latency
   - Responds to Azure events
   - Push-based (no polling delays)

6. **Event Hub Trigger**:
   - Stream processing
   - High-throughput event ingestion
   - IoT scenarios
   - Batch processing

7. **Service Bus Trigger**:
   - Enterprise messaging
   - Queue or Topic/Subscription
   - Supports sessions, transactions
   - Advanced messaging features

8. **Cosmos DB Trigger**:
   - Change feed processing
   - React to document changes
   - Near real-time
   - Partition-aware

9. **Durable Functions Orchestration Trigger**:
   - Orchestrates workflows
   - Long-running processes
   - Stateful functions

**Common Input Bindings:**
- **Blob Storage**: Read blob content
- **Table Storage**: Read table rows
- **Cosmos DB**: Query documents
- **SQL**: Query SQL databases
- **Event Grid**: Receive events
- **SignalR**: Receive messages

**Common Output Bindings:**
- **HTTP**: Return HTTP response
- **Queue Storage**: Add queue messages
- **Blob Storage**: Write blobs
- **Table Storage**: Write table rows
- **Cosmos DB**: Insert/update documents
- **Event Grid**: Publish events
- **Event Hub**: Send events
- **Service Bus**: Send messages
- **SignalR**: Send messages to clients
- **SendGrid**: Send emails
- **Twilio**: Send SMS

**Binding Configuration:**

**For .NET (In-Process):**
```csharp
[FunctionName("HttpTriggerWithBlobOutput")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    [Blob("output-container/{rand-guid}.txt", FileAccess.Write)] Stream outputBlob,
    ILogger log)
{
    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    await outputBlob.WriteAsync(Encoding.UTF8.GetBytes(requestBody));
    return new OkObjectResult("Success");
}
```

**For .NET (Isolated Worker):**
```csharp
[Function("HttpTriggerWithBlobOutput")]
public static MultiResponse Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequestData req)
{
    var response = req.CreateResponse(HttpStatusCode.OK);
    return new MultiResponse
    {
        HttpResponse = response,
        BlobOutput = req.ReadAsString()
    };
}

public class MultiResponse
{
    [BlobOutput("output-container/{rand-guid}.txt")]
    public string BlobOutput { get; set; }
    
    public HttpResponseData HttpResponse { get; set; }
}
```

**For JavaScript (function.json):**
```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    },
    {
      "type": "blob",
      "direction": "out",
      "name": "outputBlob",
      "path": "output-container/{rand-guid}.txt",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

**For Python:**
```python
import azure.functions as func
import logging

app = func.FunctionApp()

@app.function_name(name="HttpTriggerWithBlobOutput")
@app.route(route="upload", methods=["POST"])
@app.blob_output(arg_name="outputBlob", 
                 path="output-container/{rand-guid}.txt",
                 connection="AzureWebJobsStorage")
def main(req: func.HttpRequest, outputBlob: func.Out[str]) -> func.HttpResponse:
    body = req.get_body().decode('utf-8')
    outputBlob.set(body)
    return func.HttpResponse("Success", status_code=200)
```

**Binding Expressions:**
- **{rand-guid}**: Random GUID
- **{DateTime}**: Current datetime
- **{DateTime:yyyy-MM-dd}**: Formatted datetime
- **{name}**: From trigger data (e.g., blob name, queue message property)
- **{sys.methodName}**: Function name
- **{sys.utcNow}**: Current UTC time

### 1.6 Function App Structure and Configuration

**Function App:**
- Deployment unit for functions
- Container for one or more individual functions
- All functions in app share same hosting plan
- Shared configuration, resources, and runtime version

**Function App Components:**

1. **host.json**:
   - Global configuration for entire function app
   - Applies to all functions in the app
   - Runtime-wide settings
   - Extension configurations

```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "maxTelemetryItemsPerSecond": 20
      }
    }
  },
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.*, 5.0.0)"
  },
  "functionTimeout": "00:05:00",
  "retry": {
    "strategy": "fixedDelay",
    "maxRetryCount": 2,
    "delayInterval": "00:00:03"
  }
}
```

2. **local.settings.json** (Local Development Only):
   - Not deployed to Azure
   - Local environment variables
   - Connection strings for local testing
   - Storage emulator settings

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "MyCustomSetting": "value"
  },
  "ConnectionStrings": {
    "SqlConnectionString": "Server=localhost;Database=mydb"
  }
}
```

3. **function.json** (Non-.NET):
   - Function-specific configuration
   - Trigger and binding definitions
   - Not used in .NET (uses attributes instead)

4. **Application Settings**:
   - Environment variables
   - Connection strings
   - Feature flags
   - Configured in Azure Portal or via CLI
   - Override local.settings.json in cloud

**Important App Settings:**

```bash
# Runtime version
FUNCTIONS_EXTENSION_VERSION = ~4

# Worker runtime
FUNCTIONS_WORKER_RUNTIME = dotnet|node|python|java|powershell

# Storage account
AzureWebJobsStorage = <connection-string>

# Application Insights
APPINSIGHTS_INSTRUMENTATIONKEY = <key>
APPLICATIONINSIGHTS_CONNECTION_STRING = <connection-string>

# Timezone (default UTC)
WEBSITE_TIME_ZONE = Pacific Standard Time
```

**Project Structure:**

**.NET (In-Process):**
```
MyFunctionApp/
├── MyHttpFunction.cs
├── MyTimerFunction.cs
├── host.json
├── local.settings.json
└── MyFunctionApp.csproj
```

**JavaScript/TypeScript:**
```
MyFunctionApp/
├── HttpTrigger/
│   ├── function.json
│   └── index.js
├── TimerTrigger/
│   ├── function.json
│   └── index.js
├── host.json
├── package.json
└── local.settings.json
```

**Python:**
```
MyFunctionApp/
├── HttpTrigger/
│   ├── function.json
│   └── __init__.py
├── TimerTrigger/
│   ├── function.json
│   └── __init__.py
├── host.json
├── requirements.txt
└── local.settings.json
```

### 1.7 Function Execution and Scaling

**How Functions Scale:**

**Consumption and Premium Plans:**
- Azure Functions Scale Controller monitors event rate
- Decides when to add/remove instances
- Each instance runs single instance of function host
- All functions in app scale together
- Scale decisions based on:
  - Event rate
  - Queue length
  - Message age
  - CPU/memory (Premium only)

**Scale-Out Behavior:**
1. Monitor trigger source for events
2. Determine if current instances can handle load
3. If not, allocate new instance
4. Deploy function app to new instance
5. Start processing events
6. Continue monitoring

**Scale-In Behavior:**
1. Monitor for decreased load
2. Wait for instances to become idle
3. Remove instances (gradual)
4. Minimum instances maintained (0 for Consumption, 1+ for Premium)

**Scaling Limits:**

| Plan | Max Instances | Scale-Out Limit |
|------|---------------|-----------------|
| Consumption (Windows) | 200 | 10 per 10 seconds |
| Consumption (Linux) | 200 | 10 per 10 seconds |
| Premium | 20-100 | 10 per 20 seconds |
| Dedicated | Plan limit | Based on plan |

**Function Instance:**
- Single VM or container running function host
- All functions in app run on same instance
- Multiple executions of same function can run concurrently on single instance
- Concurrency depends on trigger type

**Event-Driven Scaling Patterns:**

**Queue-Based (Queue, Service Bus):**
- Monitors queue length
- Scales out when queue grows
- Scales in when queue shrinks
- Batch processing support

**Timer-Based:**
- No scaling (single instance processes)
- Multiple instances = multiple timer executions (avoid)
- Use `UseLock` setting to prevent concurrent runs

**HTTP:**
- Monitors request rate
- Scales based on concurrent requests
- Target: 1,000 requests per instance

**Event Hub:**
- Partition-based scaling
- One instance per partition
- Maximum instances = partition count

**Cosmos DB (Change Feed):**
- Lease-based scaling
- Monitors change feed lag
- Distributes partitions across instances

### 1.8 Function Performance and Best Practices

**Cold Start Optimization:**

1. **Reduce Package Size**:
   - Minimize dependencies
   - Tree-shaking for JavaScript
   - Remove unused libraries

2. **Pre-Warm Instances** (Premium Plan):
   - Configure minimum instances
   - Always-ready instances

3. **Optimize Initialization**:
   - Lazy load dependencies
   - Initialize outside function handler
   - Use static/global variables carefully

4. **Choose Appropriate Plan**:
   - Premium: No cold starts
   - Consumption: Accept cold starts for cost savings

**Performance Best Practices:**

1. **Function Design**:
   - Keep functions small and focused
   - Single responsibility principle
   - Avoid long-running operations (use Durable Functions)

2. **Concurrency**:
   - Configure appropriate batch sizes
   - Set concurrency limits
   - Understand trigger behavior

3. **Connection Management**:
   - Reuse HTTP clients
   - Use static connection instances
   - Connection pooling
   - Avoid opening connections per execution

```csharp
// BAD: Creates new HttpClient per execution
public static async Task Run(string input)
{
    using (var client = new HttpClient()) // DON'T DO THIS
    {
        await client.GetAsync("https://api.example.com");
    }
}

// GOOD: Reuses HttpClient across executions
private static HttpClient httpClient = new HttpClient();

public static async Task Run(string input)
{
    await httpClient.GetAsync("https://api.example.com");
}
```

4. **Async/Await**:
   - Use async patterns
   - Don't block on async calls
   - Avoid `.Result` or `.Wait()`

5. **Logging**:
   - Use appropriate log levels
   - Avoid excessive logging
   - Use Application Insights
   - Structured logging

**Resource Limits:**

**Consumption Plan:**
- Memory: 1.5 GB per instance
- Timeout: 5 min default (10 max)
- Execution count: 1 million free monthly

**Premium Plan:**
- Memory: Based on SKU (3.5-14 GB)
- Timeout: 30 min default (unbounded configurable)
- Instance size: EP1, EP2, EP3

**Common Limits:**
- HTTP request size: 100 MB
- HTTP response size: 100 MB
- Request timeout: 230 seconds (HTTP)
- Outbound connections: Depends on plan

**Cost Optimization:**

1. **Choose Right Plan**:
   - Consumption for sporadic workloads
   - Premium for production/low-latency
   - Dedicated for continuous workloads

2. **Optimize Execution Time**:
   - Reduce function duration
   - Efficient code
   - Minimize external calls

3. **Monitor and Optimize**:
   - Use Application Insights
   - Identify bottlenecks
   - Optimize slow functions

4. **Batch Processing**:
   - Process multiple items per execution
   - Reduce total executions

---

## MODULE 2: DEVELOP AZURE FUNCTIONS

### 2.1 Creating and Deploying Azure Functions

**Development Tools:**

1. **Azure Portal**:
   - Quick function creation
   - Code editing in browser
   - Testing and monitoring
   - Limited for production use

2. **Visual Studio**:
   - Full IDE experience
   - IntelliSense and debugging
   - Templates and wizards
   - .NET development

3. **Visual Studio Code**:
   - Lightweight, cross-platform
   - Azure Functions extension
   - Multiple language support
   - Integrated debugging

4. **Azure Functions Core Tools**:
   - Command-line tool
   - Local development and testing
   - Template generation
   - Deployment capabilities

**Installing Azure Functions Core Tools:**

```bash
# Windows (npm)
npm install -g azure-functions-core-tools@4 --unsafe-perm true

# macOS (Homebrew)
brew tap azure/functions
brew install azure-functions-core-tools@4

# Linux (apt)
wget -q https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install azure-functions-core-tools-4
```

**Creating Function App Locally:**

```bash
# Create new function app
func init MyFunctionApp --worker-runtime dotnet

# Navigate to project
cd MyFunctionApp

# Create HTTP trigger function
func new --name HttpTrigger --template "HTTP trigger"

# Create Timer trigger function
func new --name TimerTrigger --template "Timer trigger"

# Run locally
func start
```

**Function Templates:**
- HTTP trigger
- Timer trigger
- Blob trigger
- Queue trigger
- Event Hub trigger
- Cosmos DB trigger
- Service Bus Queue trigger
- Service Bus Topic trigger
- Event Grid trigger
- Durable Functions orchestrator
- And more...

**Local Development:**

```bash
# Run function locally
func start

# Run with specific port
func start --port 7072

# Build project
func build

# List functions
func list

# Run specific function
func run <FunctionName>
```

**Testing Locally:**

**HTTP Function:**
```bash
# Default URL
http://localhost:7071/api/HttpTrigger

# With query parameter
http://localhost:7071/api/HttpTrigger?name=Azure

# With body (POST)
curl -X POST http://localhost:7071/api/HttpTrigger \
  -H "Content-Type: application/json" \
  -d '{"name":"Azure"}'
```

**Using Azurite (Storage Emulator):**
```bash
# Install Azurite
npm install -g azurite

# Start Azurite
azurite --silent --location c:\azurite --debug c:\azurite\debug.log

# Connection string for local.settings.json
"AzureWebJobsStorage": "UseDevelopmentStorage=true"
```

### 2.2 Deploying Azure Functions

**Prerequisites:**
- Azure subscription
- Resource group
- Storage account (required for all plans)
- Function app created in Azure

**Creating Function App via Azure CLI:**

```bash
# Create resource group
az group create --name MyResourceGroup --location eastus

# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group MyResourceGroup \
  --location eastus \
  --sku Standard_LRS

# Create Consumption plan function app
az functionapp create \
  --resource-group MyResourceGroup \
  --consumption-plan-location eastus \
  --runtime dotnet \
  --runtime-version 8 \
  --functions-version 4 \
  --name MyFunctionApp \
  --storage-account mystorageaccount

# Create Premium plan
az functionapp plan create \
  --resource-group MyResourceGroup \
  --name MyPremiumPlan \
  --location eastus \
  --sku EP1 \
  --is-linux

az functionapp create \
  --resource-group MyResourceGroup \
  --plan MyPremiumPlan \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4 \
  --name MyPythonFunctionApp \
  --storage-account mystorageaccount \
  --os-type Linux
```

**Deployment Methods:**

#### 1. **Azure Functions Core Tools**

```bash
# Deploy to Azure
func azure functionapp publish MyFunctionApp

# Deploy with build on remote
func azure functionapp publish MyFunctionApp --build remote

# Deploy specific slot
func azure functionapp publish MyFunctionApp --slot staging
```

#### 2. **Visual Studio**
- Right-click project → Publish
- Select Azure → Azure Function App
- Create new or select existing
- Publish

#### 3. **Visual Studio Code**
- Azure Functions extension
- Deploy to Function App button
- Select subscription and function app
- Confirm deployment

#### 4. **ZIP Deployment**

```bash
# Create ZIP of function app
zip -r functionapp.zip .

# Deploy ZIP using Azure CLI
az functionapp deployment source config-zip \
  --resource-group MyResourceGroup \
  --name MyFunctionApp \
  --src functionapp.zip

# Deploy using REST API
curl -X POST -u <username>:<password> \
  https://<app-name>.scm.azurewebsites.net/api/zipdeploy \
  --data-binary @functionapp.zip
```

#### 5. **Continuous Deployment (CI/CD)**

**Azure DevOps:**
```yaml
# azure-pipelines.yml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: UseDotNet@2
  inputs:
    version: '8.x'

- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  inputs:
    command: 'publish'
    publishWebProjects: false
    projects: '**/*.csproj'
    arguments: '--output $(Build.ArtifactStagingDirectory)'
    zipAfterPublish: true

- task: AzureFunctionApp@1
  inputs:
    azureSubscription: '<subscription>'
    appType: 'functionApp'
    appName: 'MyFunctionApp'
    package: '$(Build.ArtifactStagingDirectory)/**/*.zip'
```

**GitHub Actions:**
```yaml
# .github/workflows/deploy.yml
name: Deploy to Azure Functions

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '8.0.x'
    
    - name: Build
      run: dotnet build --configuration Release
    
    - name: Publish
      run: dotnet publish --configuration Release --output ./output
    
    - name: Deploy to Azure Functions
      uses: Azure/functions-action@v1
      with:
        app-name: 'MyFunctionApp'
        package: './output'
        publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
```

#### 6. **Container Deployment**

```bash
# Create Docker image
docker build -t myfunctionapp:latest .

# Push to Azure Container Registry
az acr login --name myregistry
docker tag myfunctionapp:latest myregistry.azurecr.io/myfunctionapp:latest
docker push myregistry.azurecr.io/myfunctionapp:latest

# Create function app from container
az functionapp create \
  --resource-group MyResourceGroup \
  --name MyContainerFunctionApp \
  --storage-account mystorageaccount \
  --plan MyPremiumPlan \
  --deployment-container-image-name myregistry.azurecr.io/myfunctionapp:latest
```

**Deployment Slots:**
- Available in Premium and Dedicated plans
- Not available in Consumption plan
- Staging/production workflow
- Swap with zero downtime
- Same as App Service deployment slots

```bash
# Create deployment slot
az functionapp deployment slot create \
  --name MyFunctionApp \
  --resource-group MyResourceGroup \
  --slot staging

# Deploy to slot
func azure functionapp publish MyFunctionApp --slot staging

# Swap slots
az functionapp deployment slot swap \
  --resource-group MyResourceGroup \
  --name MyFunctionApp \
  --slot staging \
  --target-slot production
```

### 2.3 Configuring Application Settings

**Application Settings in Azure:**
- Environment variables for function app
- Configuration values
- Connection strings
- Feature flags

**Managing Settings via Portal:**
```
Function App → Settings → Environment variables → App settings
```

**Managing Settings via CLI:**

```bash
# List app settings
az functionapp config appsettings list \
  --name MyFunctionApp \
  --resource-group MyResourceGroup

# Set app setting
az functionapp config appsettings set \
  --name MyFunctionApp \
  --resource-group MyResourceGroup \
  --settings MyCustomSetting=value

# Delete app setting
az functionapp config appsettings delete \
  --name MyFunctionApp \
  --resource-group MyResourceGroup \
  --setting-names MyCustomSetting
```

**Accessing Settings in Code:**

**C#:**
```csharp
string setting = Environment.GetEnvironmentVariable("MyCustomSetting");

// Or via configuration
string setting = _configuration["MyCustomSetting"];
```

**JavaScript:**
```javascript
const setting = process.env.MyCustomSetting;
```

**Python:**
```python
import os
setting = os.environ["MyCustomSetting"]
```

**Connection Strings:**
- Stored as app settings
- Prefixed based on type: `SQLCONNSTR_`, `MYSQLCONNSTR_`, `SQLAZURECONNSTR_`, `CUSTOMCONNSTR_`
- More secure than hardcoding

**Key Vault References:**
```bash
# App setting value
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)

# With version
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/ec96f02080254f109c51a1f14cdb1931)

# Requires managed identity with Key Vault access
```

### 2.4 Monitoring and Diagnostics

**Application Insights Integration:**
- Automatic telemetry collection
- Performance monitoring
- Exception tracking
- Custom metrics and events
- Dependency tracking

**Enabling Application Insights:**

```bash
# Create Application Insights
az monitor app-insights component create \
  --app MyAppInsights \
  --location eastus \
  --resource-group MyResourceGroup

# Connect to function app
az functionapp config appsettings set \
  --name MyFunctionApp \
  --resource-group MyResourceGroup \
  --settings APPLICATIONINSIGHTS_CONNECTION_STRING="<connection-string>"
```

**Logging in Functions:**

**C#:**
```csharp
public static void Run(HttpRequest req, ILogger log)
{
    log.LogInformation("Processing request");
    log.LogWarning("This is a warning");
    log.LogError("This is an error");
    
    // Structured logging
    log.LogInformation("User {UserId} performed {Action}", userId, action);
}
```

**JavaScript:**
```javascript
module.exports = async function (context, req) {
    context.log('Processing request');
    context.log.warn('This is a warning');
    context.log.error('This is an error');
};
```

**Python:**
```python
import logging

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Processing request')
    logging.warning('This is a warning')
    logging.error('This is an error')
```

**Custom Metrics:**

```csharp
// Track custom metric
var telemetry = new TelemetryClient();
telemetry.TrackMetric("OrdersProcessed", orderCount);

// Track custom event
telemetry.TrackEvent("OrderCompleted", 
    properties: new Dictionary<string, string> { { "OrderId", orderId } },
    metrics: new Dictionary<string, double> { { "Amount", amount } });
```

**Log Levels:**
- Trace (0)
- Debug (1)
- Information (2)
- Warning (3)
- Error (4)
- Critical (5)
- None (6)

**Configuring Log Levels (host.json):**
```json
{
  "logging": {
    "logLevel": {
      "default": "Information",
      "Function": "Information",
      "Host.Results": "Error",
      "Host.Aggregator": "Information"
    }
  }
}
```

**Monitoring Metrics:**
- Execution count
- Execution units (GB-s)
- Success rate
- Error rate
- Average duration
- Active instances
- Connection count

**Live Metrics Stream:**
- Real-time telemetry
- Live request count
- Live failure rate
- Live server stats
- Sample telemetry

---

## MODULE 3: DURABLE FUNCTIONS (ADVANCED)

### 3.1 Introduction to Durable Functions

**What are Durable Functions?**
- Extension of Azure Functions
- Write stateful workflows in serverless environment
- Orchestrate function execution
- Manage state, checkpoints, and restarts automatically
- Long-running, complex workflows

**Key Benefits:**
- **Stateful**: Maintain state across function executions
- **Orchestration**: Chain multiple functions together
- **Checkpointing**: Automatic state persistence
- **Replay**: Rebuild state after interruptions
- **Timeout Management**: Handle long-running operations
- **Human Interaction**: Wait for external events

**Use Cases:**
- Long-running workflows
- Human approval processes
- Monitor and alert patterns
- Fan-out/fan-in processing
- Complex orchestrations
- Sagas and compensation logic

**Durable Functions Patterns:**
1. Function chaining
2. Fan-out/fan-in
3. Async HTTP APIs
4. Monitor
5. Human interaction
6. Aggregator (stateful entities)

### 3.2 Durable Functions Types

There are **4 types** of durable functions:

#### 1. **Orchestrator Functions**

**Purpose:**
- Define workflow logic
- Coordinate activity functions
- Describe how and when actions execute
- Control flow of the orchestration

**Characteristics:**
- Must be **deterministic** (predictable, repeatable)
- Triggered by orchestration trigger
- Can call activity functions, sub-orchestrations
- Can wait for external events
- Can set timers
- Single-threaded (no race conditions)

**Restrictions (Must Follow):**
- Cannot make non-deterministic calls (random numbers, DateTime.Now, HTTP calls directly)
- Cannot do I/O operations
- Cannot create threads or tasks directly
- Cannot call activity functions directly (must use context methods)
- Must be deterministic (same input = same output)
- No infinite loops without checkpointing

**Allowed Operations:**
- Call activity functions via context
- Call sub-orchestrations
- Schedule durable timers
- Wait for external events
- Return values
- Conditional logic and loops (deterministic)

**Example (C#):**
```csharp
[FunctionName("OrchestratorFunction")]
public static async Task<List<string>> RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var outputs = new List<string>();

    // Call activity functions in sequence
    outputs.Add(await context.CallActivityAsync<string>("SayHello", "Tokyo"));
    outputs.Add(await context.CallActivityAsync<string>("SayHello", "Seattle"));
    outputs.Add(await context.CallActivityAsync<string>("SayHello", "London"));

    return outputs;
}
```

**Replay Mechanism:**
- Orchestrator replays from start on each continuation
- Checks execution history
- If activity already completed, replays result (doesn't re-execute)
- Continues from last checkpoint
- Ensures state consistency

#### 2. **Activity Functions**

**Purpose:**
- Perform actual work
- Individual tasks in workflow
- Called by orchestrator functions

**Characteristics:**
- Basic unit of work
- No restrictions (can do anything)
- Can perform I/O operations
- Can be non-deterministic
- Can make HTTP calls, access databases
- Return results to orchestrator
- Can only be called from orchestrator functions

**Example (C#):**
```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] string name, ILogger log)
{
    log.LogInformation($"Saying hello to {name}.");
    return $"Hello {name}!";
}
```

**Example (JavaScript):**
```javascript
module.exports = async function (context, name) {
    context.log(`Saying hello to ${name}`);
    return `Hello ${name}!`;
};
```

**Multiple Parameters:**
- Activity functions accept single input
- Use objects/arrays for multiple values

```csharp
// Using object
public class ActivityInput
{
    public string Name { get; set; }
    public int Age { get; set; }
}

[FunctionName("ProcessData")]
public static string ProcessData([ActivityTrigger] ActivityInput input, ILogger log)
{
    return $"Name: {input.Name}, Age: {input.Age}";
}
```

#### 3. **Entity Functions**

**Purpose:**
- Read and update small pieces of state
- Stateful entities (durable entities)
- Similar to actors pattern
- Encapsulate state and operations

**Characteristics:**
- Have unique identity (entity ID)
- State persisted automatically
- Operations are serialized (one at a time)
- Can be invoked from orchestrators or clients
- Support for signals and operations

**Use Cases:**
- Counters
- Shopping carts
- User sessions
- Device states
- Aggregations

**Example (C#):**
```csharp
[FunctionName("Counter")]
public static void Counter([EntityTrigger] IDurableEntityContext context)
{
    int currentValue = context.GetState<int>();

    switch (context.OperationName.ToLowerInvariant())
    {
        case "add":
            int amount = context.GetInput<int>();
            currentValue += amount;
            break;
        case "reset":
            currentValue = 0;
            break;
        case "get":
            context.Return(currentValue);
            break;
    }

    context.SetState(currentValue);
}
```

**Calling Entity from Orchestrator:**
```csharp
[FunctionName("CounterOrchestrator")]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var entityId = new EntityId(nameof(Counter), "myCounter");
    
    // Signal entity (fire-and-forget)
    context.SignalEntity(entityId, "add", 5);
    
    // Call entity and get result
    int currentValue = await context.CallEntityAsync<int>(entityId, "get");
}
```

#### 4. **Client Functions**

**Purpose:**
- Start orchestrations
- Query orchestration status
- Terminate orchestrations
- Raise events to orchestrations
- Entry point for durable functions

**Characteristics:**
- Can be any trigger type (HTTP, Queue, Timer, etc.)
- Uses durable client binding
- Manages orchestration instances
- Not part of orchestration itself

**Example (C#):**
```csharp
[FunctionName("HttpStart")]
public static async Task<HttpResponseMessage> HttpStart(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")] HttpRequestMessage req,
    [DurableClient] IDurableOrchestrationClient starter,
    ILogger log)
{
    // Start new orchestration instance
    string instanceId = await starter.StartNewAsync("OrchestratorFunction", null);

    log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

    // Return HTTP response with status query URLs
    return starter.CreateCheckStatusResponse(req, instanceId);
}
```

**Client Operations:**

```csharp
// Start orchestration
string instanceId = await client.StartNewAsync("OrchestratorName", input);

// Start with custom instance ID
await client.StartNewAsync("OrchestratorName", "myCustomId", input);

// Get status
var status = await client.GetStatusAsync(instanceId);

// Terminate orchestration
await client.TerminateAsync(instanceId, "User cancelled");

// Raise event to running orchestration
await client.RaiseEventAsync(instanceId, "ApprovalReceived", true);

// Purge instance history
await client.PurgeInstanceHistoryAsync(instanceId);

// Query all instances
var queryResult = await client.ListInstancesAsync(
    new OrchestrationStatusQueryCondition
    {
        RuntimeStatus = new[]
        {
            OrchestrationRuntimeStatus.Running,
            OrchestrationRuntimeStatus.Pending
        }
    },
    CancellationToken.None);
```

### 3.3 Durable Functions Patterns

#### Pattern 1: **Function Chaining**

**Description:**
- Execute functions in sequence
- Output of one function is input to next
- Linear workflow

**Diagram:**
```
F1 → F2 → F3 → F4
```

**Example:**
```csharp
[FunctionName("Chaining")]
public static async Task<object> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var x = await context.CallActivityAsync<object>("F1", null);
    var y = await context.CallActivityAsync<object>("F2", x);
    var z = await context.CallActivityAsync<object>("F3", y);
    return await context.CallActivityAsync<object>("F4", z);
}
```

**Use Cases:**
- ETL pipelines
- Order processing
- Sequential approvals
- Multi-step transformations

---

#### Pattern 2: **Fan-Out/Fan-In**

**Description:**
- Execute multiple functions in parallel (fan-out)
- Wait for all to complete (fan-in)
- Aggregate results

**Diagram:**
```
     ┌→ F1 ┐
     ├→ F2 ┤
F0 → ├→ F3 ┤ → Aggregate → Result
     ├→ F4 ┤
     └→ F5 ┘
```

**Example:**
```csharp
[FunctionName("FanOutFanIn")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var parallelTasks = new List<Task<int>>();

    // Fan-out: Start parallel executions
    parallelTasks.Add(context.CallActivityAsync<int>("ProcessFile", "file1.txt"));
    parallelTasks.Add(context.CallActivityAsync<int>("ProcessFile", "file2.txt"));
    parallelTasks.Add(context.CallActivityAsync<int>("ProcessFile", "file3.txt"));

    // Fan-in: Wait for all to complete
    var results = await Task.WhenAll(parallelTasks);

    // Aggregate results
    var total = results.Sum();
    return total;
}
```

**Use Cases:**
- Batch processing
- MapReduce patterns
- Parallel data processing
- Aggregating data from multiple sources

---

#### Pattern 3: **Async HTTP APIs**

**Description:**
- Handle long-running operations
- Return status endpoint immediately
- Client polls for completion
- Built-in status query endpoints

**Flow:**
1. Client triggers orchestration
2. Receives 202 Accepted with status URLs
3. Client polls statusQueryGetUri
4. Orchestration completes
5. Client receives final result

**Example:**
```csharp
[FunctionName("HttpAsyncStart")]
public static async Task<HttpResponseMessage> Start(
    [HttpTrigger(AuthorizationLevel.Anonymous, "post")] HttpRequestMessage req,
    [DurableClient] IDurableOrchestrationClient starter)
{
    string instanceId = await starter.StartNewAsync("LongRunningOrchestrator", null);
    
    // Returns 202 with status URLs
    return starter.CreateCheckStatusResponse(req, instanceId);
}
```

**Response Format:**
```json
{
  "id": "d5b0a6b0f5db4c9a8f8f8f8f8f8f8f8f",
  "statusQueryGetUri": "https://.../instances/d5b0a6b0f5db4c9a8f8f8f8f8f8f8f8f",
  "sendEventPostUri": "https://.../instances/d5b0a6b0f5db4c9a8f8f8f8f8f8f8f8f/raiseEvent/{eventName}",
  "terminatePostUri": "https://.../instances/d5b0a6b0f5db4c9a8f8f8f8f8f8f8f8f/terminate",
  "purgeHistoryDeleteUri": "https://.../instances/d5b0a6b0f5db4c9a8f8f8f8f8f8f8f8f"
}
```

**Use Cases:**
- Long-running jobs
- Report generation
- Data exports
- Video processing

---

#### Pattern 4: **Monitor**

**Description:**
- Recurring polling process
- Flexible recurrence intervals
- Conditional execution
- Self-terminating

**Example:**
```csharp
[FunctionName("MonitorJobStatus")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string jobId = context.GetInput<string>();
    DateTime expiryTime = context.CurrentUtcDateTime.AddHours(1);

    while (context.CurrentUtcDateTime < expiryTime)
    {
        var jobStatus = await context.CallActivityAsync<string>("GetJobStatus", jobId);
        
        if (jobStatus == "Completed")
        {
            await context.CallActivityAsync("SendNotification", jobId);
            break;
        }

        // Wait before next check
        var nextCheck = context.CurrentUtcDateTime.AddMinutes(5);
        await context.CreateTimer(nextCheck, CancellationToken.None);
    }
}
```

**Use Cases:**
- Polling external systems
- Health checks
- Price monitoring
- Resource state monitoring

---

#### Pattern 5: **Human Interaction**

**Description:**
- Wait for human input/approval
- Timeout if no response
- External event-driven

**Example:**
```csharp
[FunctionName("ApprovalWorkflow")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    await context.CallActivityAsync("RequestApproval", null);

    using (var cts = new CancellationTokenSource())
    {
        // Wait for approval or timeout
        DateTime dueTime = context.CurrentUtcDateTime.AddHours(24);
        Task approvalEvent = context.WaitForExternalEvent<bool>("ApprovalEvent");
        Task timeoutEvent = context.CreateTimer(dueTime, cts.Token);

        Task winner = await Task.WhenAny(approvalEvent, timeoutEvent);

        if (winner == approvalEvent)
        {
            cts.Cancel(); // Cancel timer
            bool approved = await approvalEvent;
            
            if (approved)
            {
                await context.CallActivityAsync("ProcessApproval", null);
            }
            else
            {
                await context.CallActivityAsync("HandleRejection", null);
            }
        }
        else
        {
            // Timeout occurred
            await context.CallActivityAsync("EscalateApproval", null);
        }
    }
}
```

**Raising External Events:**
```csharp
// From client function
await client.RaiseEventAsync(instanceId, "ApprovalEvent", true);
```

**Use Cases:**
- Approval workflows
- User confirmations
- Manual interventions
- Time-sensitive operations

---

#### Pattern 6: **Aggregator (Stateful Entities)**

**Description:**
- Aggregate data from multiple sources
- Stateful computation
- Multiple events contribute to single result

**Example:**
```csharp
// Entity function
[FunctionName("AverageCalculator")]
public static void AverageCalculator([EntityTrigger] IDurableEntityContext context)
{
    var state = context.GetState<AverageState>(() => new AverageState());

    switch (context.OperationName.ToLowerInvariant())
    {
        case "add":
            int value = context.GetInput<int>();
            state.Sum += value;
            state.Count++;
            break;
        case "getaverage":
            context.Return(state.Count == 0 ? 0 : state.Sum / state.Count);
            break;
    }

    context.SetState(state);
}

public class AverageState
{
    public int Sum { get; set; }
    public int Count { get; set; }
}
```

**Use Cases:**
- Running averages
- Vote counting
- Aggregating sensor data
- Shopping cart totals

### 3.4 Durable Functions Configuration

**Storage Providers:**
- **Azure Storage** (default): Tables and Queues
- **Netherite**: High-performance alternative
- **MSSQL**: SQL Server provider

**Task Hub:**
- Logical container for orchestrations
- Multiple function apps can share task hub
- Configured in host.json

**host.json Configuration:**
```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "hubName": "MyTaskHub",
      "storageProvider": {
        "type": "azureStorage",
        "connectionStringName": "AzureWebJobsStorage"
      },
      "maxConcurrentActivityFunctions": 10,
      "maxConcurrentOrchestratorFunctions": 10,
      "extendedSessionsEnabled": true,
      "extendedSessionIdleTimeoutInSeconds": 30
    }
  }
}
```

**Important Settings:**

- **hubName**: Task hub name (default: function app name)
- **maxConcurrentActivityFunctions**: Max concurrent activity executions
- **maxConcurrentOrchestratorFunctions**: Max concurrent orchestrator executions
- **extendedSessionsEnabled**: Reuse orchestrator instances (reduces latency)
- **maxQueuePollingInterval**: Queue polling frequency

### 3.5 Durable Functions Best Practices

**Orchestrator Code Constraints:**

**❌ DON'T:**
```csharp
// Non-deterministic - DON'T DO THIS
var random = new Random();
int value = random.Next();

// Non-deterministic - DON'T DO THIS
DateTime now = DateTime.Now;

// I/O operations - DON'T DO THIS
var data = await httpClient.GetAsync("https://api.example.com");

// Direct threading - DON'T DO THIS
var task = Task.Run(() => DoWork());
```

**✅ DO:**
```csharp
// Use orchestration context for random
int value = context.NewGuid().GetHashCode();

// Use orchestration context for time
DateTime now = context.CurrentUtcDateTime;

// Call activity functions for I/O
var data = await context.CallActivityAsync<string>("FetchData", url);

// Use proper async patterns
var result = await context.CallActivityAsync("DoWork", null);
```

**Error Handling:**
```csharp
[FunctionName("OrchestratorWithRetry")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var retryOptions = new RetryOptions(
        firstRetryInterval: TimeSpan.FromSeconds(5),
        maxNumberOfAttempts: 3);

    try
    {
        await context.CallActivityWithRetryAsync("UnreliableActivity", retryOptions, null);
    }
    catch (Exception ex)
    {
        // Handle failure after all retries
        await context.CallActivityAsync("HandleError", ex.Message);
    }
}
```

**Sub-Orchestrations:**
```csharp
[FunctionName("ParentOrchestrator")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    // Call sub-orchestration
    var result1 = await context.CallSubOrchestratorAsync<int>("ChildOrchestrator", "input1");
    var result2 = await context.CallSubOrchestratorAsync<int>("ChildOrchestrator", "input2");
    
    return result1 + result2;
}
```

**Performance Optimization:**
1. Use extended sessions (reduces checkpointing)
2. Minimize orchestration size
3. Use sub-orchestrations for large workflows
4. Proper concurrency settings
5. Monitor and tune based on metrics

---

## MODULE 4: FUNCTION ACCESS KEYS & HTTP AUTHORIZATION

### 4.1 Function Access Keys

Azure Functions uses **access keys** to secure HTTP-triggered functions. Keys are stored encrypted and managed by the Functions runtime.

**Types of Keys:**

| Key Type | Scope | Usage |
|----------|-------|-------|
| **Function Keys** | Single function | Access specific function only |
| **Host Keys** | All functions in app | Access any function in the function app |
| **Master Key (`_master`)** | All functions + admin APIs | Full admin access; cannot be revoked, only renewed |
| **System Keys** | Extension-specific | Used by internal extensions (e.g., Event Grid, Durable Functions) |

**Key Details:**

1. **Function Keys:**
   - Scoped to individual function
   - Each function can have multiple named keys
   - Default function key named `default`
   - Pass via query string `?code=<key>` or `x-functions-key` header

2. **Host Keys:**
   - Scoped to entire function app
   - Can access any function within the app
   - Named keys (e.g., `default`)
   - Pass same way as function keys

3. **Master Key (`_master`):**
   - Special host key with elevated permissions
   - Provides admin access to runtime REST APIs
   - **Cannot be revoked** (only renewed/rotated)
   - Should be treated with extra care
   - Provides access to Durable Functions HTTP management APIs
   - **Not recommended for production** use as a regular auth mechanism

4. **System Keys:**
   - Used by specific binding extensions
   - Example: `eventgrid_extension` key for Event Grid webhook validation
   - `durabletask` key for Durable Functions HTTP APIs
   - Managed by extensions, not directly by users

**How Keys Are Passed:**

```bash
# Via query string parameter
GET https://myfunctionapp.azurewebsites.net/api/HttpTrigger?code=<API_KEY>

# Via HTTP header
GET https://myfunctionapp.azurewebsites.net/api/HttpTrigger
x-functions-key: <API_KEY>
```

**Key Storage:**
- Stored encrypted in Azure (blob storage by default)
- In local development: stored in local.settings.json (not encrypted)
- Keys are shared across all instances of function app
- Can be managed via Azure Portal, CLI, or Key Management APIs

**Key Management via CLI:**
```bash
# List function keys
az functionapp function keys list \
  --name <app-name> \
  --resource-group <rg> \
  --function-name <function-name>

# Set function key
az functionapp function keys set \
  --name <app-name> \
  --resource-group <rg> \
  --function-name <function-name> \
  --key-name myKey \
  --key-value <key-value>

# List host keys
az functionapp keys list \
  --name <app-name> \
  --resource-group <rg>

# Renew master key
az functionapp keys set \
  --name <app-name> \
  --resource-group <rg> \
  --key-name _master \
  --key-type masterKey
```

### 4.2 HTTP Trigger Authorization Levels

Authorization levels determine what key (if any) is required to invoke an HTTP-triggered function.

**Authorization Levels:**

| Level | Value | Description |
|-------|-------|-------------|
| **Anonymous** | `anonymous` | No key required. Any request is accepted. |
| **Function** | `function` | Requires a function-specific key OR host key |
| **Admin** | `admin` | Requires the **master key** (`_master`) only |

**Setting Authorization Level:**

**C# (In-Process):**
```csharp
[FunctionName("MyFunction")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequest req,
    ILogger log)
{
    // AuthorizationLevel.Anonymous - no key needed
    // AuthorizationLevel.Function - function key or host key needed  
    // AuthorizationLevel.Admin    - master key needed
    return new OkObjectResult("Hello");
}
```

**C# (Isolated Worker):**
```csharp
[Function("MyFunction")]
public HttpResponseData Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequestData req)
{
    var response = req.CreateResponse(HttpStatusCode.OK);
    response.WriteString("Hello");
    return response;
}
```

**function.json (JavaScript/Python/Java):**
```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "authLevel": "function",
      "methods": ["get", "post"]
    }
  ]
}
```

**Python (v2 model):**
```python
import azure.functions as func

app = func.FunctionApp()

@app.route(route="hello", auth_level=func.AuthLevel.FUNCTION)
def hello(req: func.HttpRequest) -> func.HttpResponse:
    return func.HttpResponse("Hello!")
```

**Authorization Level Decision Guide:**

| Scenario | Level | Reason |
|----------|-------|--------|
| Public API, open endpoint | Anonymous | No auth needed |
| Internal API, service-to-service | Function | Key-based auth |
| Administrative operations | Admin | Master key protection |
| API behind API Management | Anonymous | APIM handles auth |
| Webhooks from third parties | Function or Anonymous | Depends on webhook provider |

**Important Exam Notes:**
- Default authorization level is **Function** (not Anonymous)
- Anonymous means the function runtime doesn't validate keys - you can still implement your own auth
- When using API Management in front of Functions, set level to **Anonymous** and let APIM handle auth
- Master key provides admin access to management APIs (status, terminate, etc.)
- In production, prefer **Managed Identity** or **Azure AD/Entra ID authentication** over function keys

### 4.3 Disabling Functions

You can disable individual functions without removing them:

```bash
# Disable a function via app setting
az functionapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --settings "AzureWebJobs.<FUNCTION_NAME>.Disabled=true"

# Enable a function
az functionapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --settings "AzureWebJobs.<FUNCTION_NAME>.Disabled=false"
```

**C# Attribute:**
```csharp
[FunctionName("MyFunction")]
[Disable("MY_DISABLE_SETTING")]  // Reads app setting to determine if disabled
public static void Run([TimerTrigger("0 */5 * * * *")] TimerInfo timer) { }
```

---

## MODULE 5: .NET EXECUTION MODELS (In-Process vs Isolated Worker)

### 5.1 Overview of .NET Models

Azure Functions supports two .NET execution models:

| Feature | In-Process Model | Isolated Worker Model |
|---------|-----------------|----------------------|
| **Process** | Same process as Functions host | Separate worker process |
| **.NET Versions** | .NET 6 only (end of support) | .NET 6, 7, 8, 9+ |
| **NuGet Package** | `Microsoft.NET.Sdk.Functions` | `Microsoft.Azure.Functions.Worker` |
| **Entry Point** | Function methods only | `Program.cs` with `HostBuilder` |
| **Dependency Injection** | `Startup` class | `Program.cs` (standard .NET DI) |
| **Middleware** | Not supported | Fully supported |
| **Binding Types** | Rich .NET types (HttpRequest, ILogger) | Serializable types, HttpRequestData |
| **Output Bindings** | Return values, out parameters | Return types, property bags |
| **Future** | Being retired | Recommended going forward |

**CRITICAL: In-Process model is being retired. Isolated Worker is the recommended model for all new development.**

### 5.2 In-Process Model (Legacy)

**NuGet Package:** `Microsoft.NET.Sdk.Functions`

**Project File (.csproj):**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <AzureFunctionsVersion>v4</AzureFunctionsVersion>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Sdk.Functions" Version="4.1.1" />
  </ItemGroup>
</Project>
```

**Function Example:**
```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;

public static class MyFunction
{
    [FunctionName("HttpExample")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req,
        ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");
        string name = req.Query["name"];
        return new OkObjectResult($"Hello, {name}");
    }
}
```

**Key Attributes (In-Process):**
- `[FunctionName("name")]` - Defines function name
- Uses `Microsoft.Azure.WebJobs` namespace
- Uses ASP.NET Core types directly (`HttpRequest`, `IActionResult`)
- `ILogger` injected directly as parameter

**Dependency Injection (In-Process) - Startup Class:**
```csharp
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection;

[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            // Register services
            builder.Services.AddSingleton<IMyService, MyService>();
            builder.Services.AddHttpClient();
            builder.Services.AddScoped<IDataRepository, DataRepository>();
        }
    }
}
```

**Using Injected Services (In-Process):**
```csharp
public class MyFunction
{
    private readonly IMyService _myService;
    
    public MyFunction(IMyService myService)
    {
        _myService = myService;
    }

    [FunctionName("HttpExample")]
    public async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req,
        ILogger log)
    {
        var result = await _myService.GetDataAsync();
        return new OkObjectResult(result);
    }
}
```

### 5.3 Isolated Worker Model (Recommended)

**NuGet Packages:**
- `Microsoft.Azure.Functions.Worker`
- `Microsoft.Azure.Functions.Worker.Sdk`
- `Microsoft.Azure.Functions.Worker.Extensions.Http`
- `Microsoft.Azure.Functions.Worker.Extensions.Http.AspNet` (for ASP.NET Core integration)

**Project File (.csproj):**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <AzureFunctionsVersion>v4</AzureFunctionsVersion>
    <OutputType>Exe</OutputType>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.Azure.Functions.Worker" Version="1.20.0" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Sdk" Version="1.16.2" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.Http" Version="3.1.0" />
    <PackageReference Include="Microsoft.Azure.Functions.Worker.Extensions.Http.AspNet" Version="1.2.0" />
  </ItemGroup>
</Project>
```

**Program.cs (Entry Point):**
```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

var host = new HostBuilder()
    .ConfigureFunctionsWebApplication()  // For ASP.NET Core integration
    // OR
    // .ConfigureFunctionsWorkerDefaults()  // Without ASP.NET Core
    .ConfigureServices(services =>
    {
        // Register Application Insights
        services.AddApplicationInsightsTelemetryWorkerService();
        services.ConfigureFunctionsApplicationInsights();
        
        // Register custom services (Dependency Injection)
        services.AddSingleton<IMyService, MyService>();
        services.AddHttpClient();
    })
    .Build();

host.Run();
```

**Function Example (Isolated Worker):**
```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;
using Microsoft.Extensions.Logging;
using System.Net;

public class MyFunction
{
    private readonly ILogger<MyFunction> _logger;
    private readonly IMyService _myService;

    public MyFunction(ILogger<MyFunction> logger, IMyService myService)
    {
        _logger = logger;
        _myService = myService;
    }

    [Function("HttpExample")]
    public async Task<HttpResponseData> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequestData req)
    {
        _logger.LogInformation("C# HTTP trigger function processed a request.");
        
        var response = req.CreateResponse(HttpStatusCode.OK);
        await response.WriteStringAsync("Hello from isolated worker!");
        return response;
    }
}
```

**Key Differences in Code:**
- `[Function("name")]` instead of `[FunctionName("name")]`
- `HttpRequestData` / `HttpResponseData` instead of `HttpRequest` / `IActionResult`
- Logger injected via constructor, not as function parameter
- `Microsoft.Azure.Functions.Worker` namespace instead of `Microsoft.Azure.WebJobs`
- Must have `Program.cs` with `HostBuilder`
- Has `OutputType` set to `Exe` in project file

### 5.4 Middleware (Isolated Worker Only)

Middleware allows you to intercept and modify function invocations — similar to ASP.NET Core middleware.

**Creating Middleware:**
```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Middleware;

public class ExceptionHandlingMiddleware : IFunctionsWorkerMiddleware
{
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(ILogger<ExceptionHandlingMiddleware> logger)
    {
        _logger = logger;
    }

    public async Task Invoke(FunctionContext context, FunctionExecutionDelegate next)
    {
        try
        {
            // Code before function execution
            _logger.LogInformation("Function {Name} is being invoked", context.FunctionDefinition.Name);
            
            await next(context);  // Call the actual function
            
            // Code after function execution
            _logger.LogInformation("Function {Name} completed", context.FunctionDefinition.Name);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error in function {Name}", context.FunctionDefinition.Name);
            throw;
        }
    }
}
```

**Registering Middleware in Program.cs:**
```csharp
var host = new HostBuilder()
    .ConfigureFunctionsWorkerDefaults(workerApplication =>
    {
        // Middleware executes in order registered
        workerApplication.UseMiddleware<ExceptionHandlingMiddleware>();
        workerApplication.UseMiddleware<AuthenticationMiddleware>();
        workerApplication.UseMiddleware<LoggingMiddleware>();
    })
    .Build();

host.Run();
```

**Common Middleware Use Cases:**
- Global exception handling
- Authentication/authorization validation
- Request/response logging
- Correlation ID tracking
- Input validation
- Performance monitoring

### 5.5 ASP.NET Core Integration (Isolated Worker)

With `ConfigureFunctionsWebApplication()`, you can use ASP.NET Core types:

```csharp
// Program.cs
var host = new HostBuilder()
    .ConfigureFunctionsWebApplication()
    .Build();

host.Run();
```

```csharp
// Function with ASP.NET Core integration
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.Functions.Worker;

public class MyFunction
{
    [Function("HttpExample")]
    public IActionResult Run(
        [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req)
    {
        return new OkObjectResult("Hello with ASP.NET Core!");
    }
}
```

### 5.6 Multiple Output Bindings (Isolated Worker)

In isolated worker, use a class with properties for multiple outputs:

```csharp
public class MultiOutput
{
    [HttpResult]
    public HttpResponseData HttpResponse { get; set; }
    
    [QueueOutput("output-queue")]
    public string QueueMessage { get; set; }
    
    [BlobOutput("output-container/output.txt")]
    public string BlobContent { get; set; }
}

public class MyFunction
{
    [Function("MultiOutputFunction")]
    public MultiOutput Run(
        [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequestData req)
    {
        var response = req.CreateResponse(HttpStatusCode.OK);
        response.WriteString("Done!");
        
        return new MultiOutput
        {
            HttpResponse = response,
            QueueMessage = "New message for queue",
            BlobContent = "Content for blob"
        };
    }
}
```

---

## MODULE 6: ADVANCED TRIGGER & BINDING CONFIGURATIONS

### 6.1 HTTP Trigger Advanced Configuration

**Route Templates:**
```csharp
// Custom route (instead of default /api/FunctionName)
[HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "products/{category}/{id:int?}")]
```

**Route Constraints:**
```csharp
// Supported constraints
Route = "users/{id:int}"           // Integer only
Route = "users/{id:guid}"          // GUID only
Route = "users/{name:alpha}"       // Alpha chars only
Route = "users/{name:minlength(3)}" // Min 3 characters
Route = "products/{id:int?}"       // Optional int parameter
```

**Accessing Route Parameters:**

**C# In-Process:**
```csharp
[FunctionName("GetProduct")]
public static IActionResult Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", 
     Route = "products/{category}/{id:int}")] HttpRequest req,
    string category,
    int id,
    ILogger log)
{
    return new OkObjectResult($"Category: {category}, ID: {id}");
}
```

**C# Isolated Worker:**
```csharp
[Function("GetProduct")]
public HttpResponseData Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", 
     Route = "products/{category}/{id:int}")] HttpRequestData req,
    string category,
    int id)
{
    var response = req.CreateResponse(HttpStatusCode.OK);
    response.WriteString($"Category: {category}, ID: {id}");
    return response;
}
```

**HTTP Trigger Request/Response Details:**

```csharp
// Reading request body
string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
dynamic data = JsonConvert.DeserializeObject(requestBody);

// Reading query parameters
string name = req.Query["name"];

// Reading headers
string authHeader = req.Headers["Authorization"];

// Reading route data
string id = req.RouteValues["id"]?.ToString();

// Setting response headers
response.Headers.Add("X-Custom-Header", "value");
```

**HTTP Output Binding with Status Codes:**
```csharp
// Return specific status codes
return new StatusCodeResult(StatusCodes.Status202Accepted);
return new BadRequestObjectResult("Invalid input");
return new NotFoundResult();
return new UnauthorizedResult();
return new ConflictObjectResult("Resource already exists");
```

### 6.2 Timer Trigger Advanced Configuration

**CRON Expression Format:**
```
{second} {minute} {hour} {day} {month} {day-of-week}
```

**CRON Expression Examples:**

| Expression | Description |
|-----------|-------------|
| `0 */5 * * * *` | Every 5 minutes |
| `0 0 * * * *` | Every hour (top of hour) |
| `0 0 */2 * * *` | Every 2 hours |
| `0 0 9-17 * * *` | Every hour from 9 AM to 5 PM |
| `0 30 9 * * 1-5` | 9:30 AM weekdays only |
| `0 0 0 * * *` | Midnight every day |
| `0 0 0 1 * *` | Midnight on 1st of every month |
| `0 0 0 * * 0` | Midnight every Sunday |
| `0 0 0 1 1 *` | Midnight on January 1st |
| `0 0 9 * * 1` | 9:00 AM every Monday |

**Special CRON values:**
- `*` = every value
- `*/n` = every n intervals
- `n-m` = range from n to m
- `n,m` = specific values n and m

**Timer Trigger Properties:**

```csharp
[FunctionName("TimerFunction")]
public static void Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo myTimer,
    ILogger log)
{
    // TimerInfo properties:
    log.LogInformation($"Schedule: {myTimer.Schedule}");
    log.LogInformation($"Is Past Due: {myTimer.IsPastDue}");
    log.LogInformation($"Last Run: {myTimer.ScheduleStatus?.Last}");
    log.LogInformation($"Next Run: {myTimer.ScheduleStatus?.Next}");
    log.LogInformation($"Last Updated: {myTimer.ScheduleStatus?.LastUpdated}");
}
```

**RunOnStartup:**
```csharp
// Function runs immediately on startup AND on schedule
[TimerTrigger("0 */5 * * * *", RunOnStartup = true)]
```
⚠️ **Warning:** `RunOnStartup` should **rarely be used in production** — especially with Consumption plan, as it causes unexpected executions on scale-out.

**UseMonitor:**
```csharp
// UseMonitor = true (default): Persists schedule status to detect missed executions
// UseMonitor = false: Don't track missed executions
[TimerTrigger("0 */5 * * * *", UseMonitor = true)]
```

**Timer Trigger host.json:**
```json
{
  "extensions": {
    "timers": {
      "schedule": {
        "adjustForDST": true
      }
    }
  }
}
```

**Timezone Configuration:**
- Timer triggers use UTC by default
- Set `WEBSITE_TIME_ZONE` app setting to change timezone
- Windows: `"WEBSITE_TIME_ZONE": "Eastern Standard Time"`
- Linux: `"WEBSITE_TIME_ZONE": "America/New_York"` (IANA format)

### 6.3 Queue Trigger Advanced Configuration

**Queue Trigger host.json Settings:**
```json
{
  "extensions": {
    "queues": {
      "maxPollingInterval": "00:02:00",
      "visibilityTimeout": "00:00:30",
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8,
      "messageEncoding": "base64"
    }
  }
}
```

**Key Settings Explained:**

| Setting | Default | Description |
|---------|---------|-------------|
| `batchSize` | 16 | Number of messages retrieved simultaneously |
| `maxDequeueCount` | 5 | Times to try processing before moving to poison queue |
| `visibilityTimeout` | 00:00:00 | Time before failed message becomes visible again |
| `maxPollingInterval` | 00:01:00 | Maximum interval between queue polls |
| `newBatchThreshold` | batchSize/2 | Threshold for fetching new batch |
| `messageEncoding` | base64 | Encoding format (base64 or none) |

**Poison Message Handling:**
- After `maxDequeueCount` failed attempts, message moves to poison queue
- Poison queue name: `{original-queue-name}-poison`
- Example: `myqueue` → `myqueue-poison`
- Monitor poison queues for failed messages
- Implement alerting on poison queue message count

```csharp
// Queue trigger function
[FunctionName("ProcessQueue")]
public static void Run(
    [QueueTrigger("myqueue")] string myQueueItem,
    int dequeueCount,       // Current dequeue attempt count
    string id,              // Message ID
    string popReceipt,      // Pop receipt
    DateTimeOffset insertionTime,
    DateTimeOffset expirationTime,
    DateTimeOffset nextVisibleTime,
    ILogger log)
{
    log.LogInformation($"Processing message {id}, attempt {dequeueCount}");
    
    if (dequeueCount > 3)
    {
        log.LogWarning($"Message {id} has been dequeued {dequeueCount} times");
    }
}
```

**Queue Message Metadata (Available as parameters):**
- `dequeueCount` — Number of times message has been dequeued
- `expirationTime` — Message expiration time
- `id` — Message ID
- `insertionTime` — When message was added
- `nextVisibleTime` — Next visibility time
- `popReceipt` — Pop receipt of the message

### 6.4 Blob Trigger Advanced Configuration

**Blob Trigger host.json:**
```json
{
  "extensions": {
    "blobs": {
      "maxDegreeOfParallelism": 8,
      "poisonBlobThreshold": 3
    }
  }
}
```

**Blob Trigger Scanning Methods:**

1. **Storage Logs (Default for general-purpose accounts):**
   - Scans blob storage logs for changes
   - Can have up to **10-minute delay** in processing
   - Logs are created on a best-effort basis

2. **Event Grid (Recommended):**
   - Uses Event Grid subscription for blob events
   - **Near real-time** processing (seconds)
   - Requires Event Grid subscription configuration
   - Much more efficient than log-based scanning

**Setting up Event Grid-based Blob Trigger:**
```json
{
  "extensions": {
    "blobs": {
      "source": "EventGrid"
    }
  }
}
```

**Blob Trigger vs Event Grid Trigger for Blobs:**

| Feature | Blob Trigger | Event Grid Trigger |
|---------|-------------|-------------------|
| **Latency** | Up to 10 min (polling) | Near real-time |
| **Scaling** | Slower | Faster |
| **Blob Types** | Block blobs | Block + Page + Append |
| **Filtering** | Container path only | Advanced filtering |
| **Reliability** | Best effort | Guaranteed delivery |
| **Cost** | Included | Event Grid costs |
| **Setup** | Simple | Requires Event Grid subscription |

**Blob Receipt Tracking:**
- Azure Functions tracks which blobs have been processed
- Uses blob receipts stored in `azure-webjobs-hosts` container
- If function app restarts, already-processed blobs are not reprocessed
- Blob receipts contain: function ID, blob ETag, timestamp

**Blob Trigger Path Patterns:**
```csharp
// Trigger on any blob in container
[BlobTrigger("input/{name}")] 

// Trigger on specific file extension
[BlobTrigger("input/{name}.csv")]

// Trigger on subfolder
[BlobTrigger("input/images/{name}")]

// Binding expressions
[BlobTrigger("input/{blobname}.{blobextension}")]
```

**Blob Input/Output Binding Types:**
```csharp
// Read as string
[Blob("container/{name}", FileAccess.Read)] string content

// Read as byte array
[Blob("container/{name}", FileAccess.Read)] byte[] content

// Read as Stream
[Blob("container/{name}", FileAccess.Read)] Stream stream

// Read as BlobClient (SDK type)
[Blob("container/{name}")] BlobClient blobClient

// Read as CloudBlockBlob (legacy)
[Blob("container/{name}")] CloudBlockBlob blob
```

### 6.5 Service Bus Trigger Advanced Configuration

**Service Bus Trigger host.json:**
```json
{
  "extensions": {
    "serviceBus": {
      "prefetchCount": 0,
      "messageHandlerOptions": {
        "autoComplete": true,
        "maxConcurrentCalls": 16,
        "maxAutoRenewDuration": "00:05:00"
      },
      "sessionHandlerOptions": {
        "autoComplete": false,
        "messageWaitTimeout": "00:00:30",
        "maxAutoRenewDuration": "00:55:00",
        "maxConcurrentSessions": 2000
      },
      "batchOptions": {
        "maxMessageCount": 1000,
        "operationTimeout": "00:01:00",
        "autoComplete": true
      }
    }
  }
}
```

**Key Settings:**

| Setting | Default | Description |
|---------|---------|-------------|
| `maxConcurrentCalls` | 16 | Max concurrent messages processed per instance |
| `autoComplete` | true | Auto-complete message after function success |
| `prefetchCount` | 0 | Number of messages prefetched |
| `maxAutoRenewDuration` | 00:05:00 | Max time to auto-renew message lock |

**Service Bus Trigger Properties:**
```csharp
[FunctionName("ServiceBusQueueFunction")]
public static void Run(
    [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] 
    ServiceBusReceivedMessage message,
    ServiceBusMessageActions messageActions,
    ILogger log)
{
    // Message properties
    log.LogInformation($"Message ID: {message.MessageId}");
    log.LogInformation($"Body: {message.Body}");
    log.LogInformation($"Content Type: {message.ContentType}");
    log.LogInformation($"Correlation ID: {message.CorrelationId}");
    log.LogInformation($"Delivery Count: {message.DeliveryCount}");
    log.LogInformation($"Enqueued Time: {message.EnqueuedTime}");
    log.LogInformation($"Lock Token: {message.LockToken}");
    log.LogInformation($"Session ID: {message.SessionId}");
    
    // Manual complete/abandon/dead-letter
    // await messageActions.CompleteMessageAsync(message);
    // await messageActions.AbandonMessageAsync(message);
    // await messageActions.DeadLetterMessageAsync(message);
}
```

**Service Bus Topic/Subscription Trigger:**
```csharp
[FunctionName("TopicFunction")]
public static void Run(
    [ServiceBusTrigger("mytopic", "mysubscription", 
     Connection = "ServiceBusConnection")] string messageBody,
    ILogger log)
{
    log.LogInformation($"Topic message: {messageBody}");
}
```

### 6.6 Event Hub Trigger Advanced Configuration

**Event Hub Trigger host.json:**
```json
{
  "extensions": {
    "eventHubs": {
      "maxEventBatchSize": 100,
      "minEventBatchSize": 1,
      "maxWaitTime": "00:00:05",
      "batchCheckpointFrequency": 1,
      "prefetchCount": 300,
      "transportType": "amqpWebSockets",
      "clientRetryOptions": {
        "mode": "exponential",
        "tryTimeout": "00:01:00",
        "delay": "00:00:00.80",
        "maxDelay": "00:01:00",
        "maxRetries": 3
      }
    }
  }
}
```

**Event Hub Trigger Batch Processing:**
```csharp
[FunctionName("EventHubBatch")]
public static async Task Run(
    [EventHubTrigger("myeventhub", Connection = "EventHubConnection")] 
    EventData[] events,
    ILogger log)
{
    foreach (EventData eventData in events)
    {
        string messageBody = Encoding.UTF8.GetString(eventData.Body.ToArray());
        log.LogInformation($"Event: {messageBody}");
        log.LogInformation($"Partition: {eventData.PartitionKey}");
        log.LogInformation($"Offset: {eventData.Offset}");
        log.LogInformation($"Sequence: {eventData.SequenceNumber}");
        log.LogInformation($"Enqueued: {eventData.EnqueuedTime}");
    }
}
```

**Key Concept: Event Hub scaling = based on partitions. Max instances = partition count.**

### 6.7 Cosmos DB Trigger Advanced Configuration

**Cosmos DB Trigger host.json:**
```json
{
  "extensions": {
    "cosmosDB": {
      "connectionMode": "Direct",
      "protocol": "Tcp",
      "leaseOptions": {
        "feedPollDelay": "00:00:05",
        "acquireInterval": "00:00:13",
        "expirationInterval": "00:01:00",
        "renewInterval": "00:00:17"
      }
    }
  }
}
```

**Cosmos DB Trigger with Lease Container:**
```csharp
[FunctionName("CosmosDBTrigger")]
public static void Run(
    [CosmosDBTrigger(
        databaseName: "mydb",
        containerName: "mycontainer",
        Connection = "CosmosDBConnection",
        LeaseContainerName = "leases",
        CreateLeaseContainerIfNotExists = true,
        LeaseContainerPrefix = "myprefix",
        StartFromBeginning = false,
        MaxItemsPerInvocation = 100,
        FeedPollDelay = 5000,
        PreferredLocations = "East US"
    )] IReadOnlyList<Document> documents,
    ILogger log)
{
    if (documents != null && documents.Count > 0)
    {
        log.LogInformation($"Documents modified: {documents.Count}");
        log.LogInformation($"First document Id: {documents[0].Id}");
    }
}
```

**Key Cosmos DB Trigger Settings:**

| Setting | Description |
|---------|-------------|
| `LeaseContainerName` | Container to store lease state (required) |
| `CreateLeaseContainerIfNotExists` | Auto-create lease container |
| `StartFromBeginning` | Read all changes from start (false = only new) |
| `MaxItemsPerInvocation` | Max items per function call |
| `FeedPollDelay` | Delay in ms between change feed polls |
| `LeaseContainerPrefix` | Prefix for lease documents (multi-function sharing) |
| `PreferredLocations` | Comma-separated preferred regions |

**Change Feed Key Concepts:**
- Uses **lease container** to track processing state
- Multiple functions can share a lease container using `LeaseContainerPrefix`
- Change feed processes **inserts and updates** only (NOT deletes)
- To handle deletes: use soft-delete pattern with TTL
- Documents are processed **in order within a partition**
- Guaranteed **at-least-once** delivery

### 6.8 Extension Bundles

Extension bundles automatically manage trigger/binding extensions without needing NuGet packages (for non-.NET languages).

**host.json Configuration:**
```json
{
  "version": "2.0",
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.*, 5.0.0)"
  }
}
```

**Key Points:**
- Required for JavaScript, Python, Java, PowerShell
- NOT required for .NET (uses NuGet packages instead)
- Automatically downloads appropriate extensions
- Version range syntax: `[min, max)` (inclusive min, exclusive max)
- Simplifies project setup — no manual extension installation needed

**Without Extension Bundles (.NET):**
```bash
# Install binding extension manually
dotnet add package Microsoft.Azure.WebJobs.Extensions.Storage
dotnet add package Microsoft.Azure.WebJobs.Extensions.CosmosDB
dotnet add package Microsoft.Azure.WebJobs.Extensions.ServiceBus
```

---

## MODULE 7: NETWORKING, SECURITY & IDENTITY

### 7.1 Azure Functions Networking

**Inbound Features (controlling traffic INTO function app):**

| Feature | Plans |
|---------|-------|
| Inbound IP restrictions | All plans |
| Private endpoints | Premium, Dedicated |
| Service endpoints | Premium, Dedicated |
| Access restrictions | All plans |

**Outbound Features (controlling traffic FROM function app):**

| Feature | Plans |
|---------|-------|
| VNet integration | Premium, Dedicated, Flex Consumption |
| Hybrid connections | Premium, Dedicated |
| NAT gateway | Premium, Dedicated |
| Outbound IP addresses | All plans |

**IP Restrictions:**
```bash
# Add IP restriction
az functionapp config access-restriction add \
  --name <app-name> \
  --resource-group <rg> \
  --rule-name "AllowMyIP" \
  --priority 100 \
  --action Allow \
  --ip-address 203.0.113.0/24

# Remove IP restriction
az functionapp config access-restriction remove \
  --name <app-name> \
  --resource-group <rg> \
  --rule-name "AllowMyIP"
```

**VNet Integration (Outbound):**
- Function app can access resources in a VNet
- Available in **Premium**, **Dedicated**, and **Flex Consumption** plans
- NOT available in Consumption plan
- Enables access to databases, VMs, other services in VNet
- Supports Network Security Groups (NSGs) and route tables

**Private Endpoints (Inbound):**
- Makes function app accessible only through private VNet addresses
- Disables public internet access (if configured)
- Available in **Premium** and **Dedicated** plans

### 7.2 Function App Authentication (EasyAuth)

Azure Functions supports built-in authentication/authorization (App Service Authentication / EasyAuth).

**Supported Identity Providers:**
- Microsoft Entra ID (Azure AD)
- Facebook
- Google
- Twitter
- GitHub
- Apple
- Any OpenID Connect provider

**How EasyAuth Works:**
1. Runs as a separate module alongside your function
2. Intercepts incoming HTTP requests
3. Validates tokens, manages sessions
4. Injects identity information into request headers
5. Your code receives authenticated user info

**Authentication Flow:**
- **Server-directed flow**: Provider login page (web apps)
- **Client-directed flow**: App handles login, sends token to server (mobile/SPA)

**Configuring via CLI:**
```bash
# Enable AAD authentication
az webapp auth update \
  --name <app-name> \
  --resource-group <rg> \
  --enabled true \
  --action LoginWithAzureActiveDirectory \
  --aad-client-id <client-id> \
  --aad-client-secret <client-secret> \
  --aad-token-issuer-url https://sts.windows.net/<tenant-id>/
```

**Accessing User Claims in Code:**
```csharp
// Claims are available in request headers
string userId = req.Headers["X-MS-CLIENT-PRINCIPAL-ID"];
string userName = req.Headers["X-MS-CLIENT-PRINCIPAL-NAME"];
string idpToken = req.Headers["X-MS-TOKEN-AAD-ACCESS-TOKEN"];

// Or use ClaimsPrincipal (in-process model)
ClaimsPrincipal principal = req.HttpContext.User;
string email = principal.FindFirst(ClaimTypes.Email)?.Value;
```

**EasyAuth vs Function Keys:**

| Feature | Function Keys | EasyAuth |
|---------|--------------|----------|
| **Auth Type** | API key | Identity-based (OAuth/OIDC) |
| **User Identity** | No user info | Full user identity |
| **Granularity** | Function or app level | User/role level |
| **Token** | Static key | OAuth tokens |
| **Best For** | Service-to-service | User-facing apps |

### 7.3 CORS (Cross-Origin Resource Sharing)

**Configuring CORS:**
```bash
# Add allowed origin
az functionapp cors add \
  --name <app-name> \
  --resource-group <rg> \
  --allowed-origins "https://mywebapp.com" "https://localhost:3000"

# Remove allowed origin
az functionapp cors remove \
  --name <app-name> \
  --resource-group <rg> \
  --allowed-origins "https://mywebapp.com"

# List allowed origins
az functionapp cors show \
  --name <app-name> \
  --resource-group <rg>
```

**host.json CORS (local development):**
```json
{
  "Host": {
    "CORS": "*",
    "CORSCredentials": false
  }
}
```

**Key Points:**
- `*` allows all origins (not recommended for production)
- Wildcard cannot be used with credentials
- CORS is handled at the platform level (before function code)
- For Azure Portal testing, add `https://portal.azure.com` as allowed origin

### 7.4 Managed Identity with Azure Functions

**Types of Managed Identity:**

| Feature | System-Assigned | User-Assigned |
|---------|----------------|---------------|
| **Creation** | Created with resource | Created independently |
| **Lifecycle** | Tied to resource | Independent lifecycle |
| **Sharing** | 1 resource only | Multiple resources |
| **Deletion** | Deleted with resource | Must delete separately |

**Enabling Managed Identity:**
```bash
# Enable system-assigned managed identity
az functionapp identity assign \
  --name <app-name> \
  --resource-group <rg>

# Assign user-assigned managed identity
az functionapp identity assign \
  --name <app-name> \
  --resource-group <rg> \
  --identities <identity-resource-id>
```

**Using Managed Identity in Code:**
```csharp
using Azure.Identity;
using Azure.Storage.Blobs;

// DefaultAzureCredential uses managed identity in Azure, 
// falls back to other methods locally
var credential = new DefaultAzureCredential();

// Access Blob Storage with managed identity
var blobClient = new BlobServiceClient(
    new Uri("https://mystorageaccount.blob.core.windows.net"),
    credential);

// Access Key Vault with managed identity
var keyVaultClient = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net"),
    credential);
```

**DefaultAzureCredential Chain (Order):**
1. Environment variables
2. Workload Identity
3. Managed Identity
4. Visual Studio
5. Azure CLI
6. Azure PowerShell
7. Azure Developer CLI
8. Interactive browser

### 7.5 Identity-Based Connections (No Connection Strings)

Instead of connection strings, Azure Functions can use managed identity for trigger/binding connections.

**Traditional (Connection String):**
```json
{
  "Values": {
    "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=...;AccountKey=...",
    "ServiceBusConnection": "Endpoint=sb://mybus.servicebus.windows.net/;SharedAccessKeyName=...;SharedAccessKey=..."
  }
}
```

**Identity-Based Connection:**
```json
{
  "Values": {
    "AzureWebJobsStorage__accountName": "mystorageaccount",
    "ServiceBusConnection__fullyQualifiedNamespace": "mybus.servicebus.windows.net"
  }
}
```

**Naming Convention for Identity-Based:**
- `<CONNECTION_NAME>__accountName` for Storage
- `<CONNECTION_NAME>__fullyQualifiedNamespace` for Service Bus / Event Hub
- `<CONNECTION_NAME>__accountEndpoint` for Cosmos DB
- Double underscore `__` is used as separator

**Required RBAC Roles:**

| Service | Role for Trigger | Role for Output |
|---------|-----------------|-----------------|
| Blob Storage | Storage Blob Data Owner | Storage Blob Data Contributor |
| Queue Storage | Storage Queue Data Contributor | Storage Queue Data Contributor |
| Table Storage | Storage Table Data Contributor | Storage Table Data Contributor |
| Service Bus | Service Bus Data Receiver | Service Bus Data Sender |
| Event Hub | Event Hub Data Receiver | Event Hub Data Sender |
| Cosmos DB | Cosmos DB Built-in Data Reader | Cosmos DB Built-in Data Contributor |

**Benefits of Identity-Based Connections:**
- No secrets to manage or rotate
- No connection strings in configuration
- Uses Azure RBAC for fine-grained access control
- Supports system-assigned and user-assigned managed identities
- Works with `DefaultAzureCredential` for local development

---

## MODULE 8: ADVANCED DEPLOYMENT & RUNTIME CONFIGURATION

### 8.1 Run-From-Package Deployment

**WEBSITE_RUN_FROM_PACKAGE Setting:**

| Value | Description |
|-------|-------------|
| `1` | Run from local package (d:\home\data\SitePackages) |
| `<URL>` | Run from external URL (Blob storage) |
| Not set | Normal deployment (d:\home\site\wwwroot) |

**Setting Run-From-Package:**
```bash
# Enable run from local package
az functionapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --settings WEBSITE_RUN_FROM_PACKAGE=1

# Run from blob URL (with SAS token)
az functionapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --settings WEBSITE_RUN_FROM_PACKAGE="https://mystorage.blob.core.windows.net/deploy/package.zip?sv=..."
```

**Benefits of Run-From-Package:**
- Files are NOT extracted to wwwroot (mounted read-only)
- Reduces cold start time
- Atomic deployment (no partial deployments)
- Consistent deployment state
- Reduces storage usage

**Limitations:**
- File system is read-only (cannot write to wwwroot)
- Maximum package size: 1 GB
- Some scenarios require writable file system

### 8.2 Deployment Slots (Advanced)

**Slot-Specific (Sticky) Settings:**
- Settings that DON'T swap with the slot
- Marked as "deployment slot setting" in portal
- Important for settings that are environment-specific

**Common Sticky Settings:**
```
- Connection strings for slot-specific databases
- Application Insights keys (different per slot)
- Feature flags for testing
- Custom domain SSL bindings
- WEBSITE_RUN_FROM_PACKAGE
```

**Configuring Sticky Settings via CLI:**
```bash
# Mark setting as slot-specific
az functionapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --slot-settings MY_SLOT_SETTING=value

# Note: --slot-settings makes it sticky (doesn't swap)
# Regular --settings will swap with the slot
```

**Swap with Preview (Multi-Phase Swap):**
1. Apply target slot settings to source
2. Validate source with target settings
3. Complete or cancel swap

```bash
# Start swap with preview
az functionapp deployment slot swap \
  --resource-group <rg> \
  --name <app-name> \
  --slot staging \
  --target-slot production \
  --action preview

# Complete the swap
az functionapp deployment slot swap \
  --resource-group <rg> \
  --name <app-name> \
  --slot staging \
  --target-slot production

# Cancel the swap
az functionapp deployment slot swap \
  --resource-group <rg> \
  --name <app-name> \
  --slot staging \
  --action reset
```

**Auto Swap:**
- Automatically swaps slot after deployment succeeds
- Configure in slot configuration settings
- Useful for zero-downtime continuous deployment

**Traffic Routing:**
```bash
# Route 20% of production traffic to staging slot
az functionapp traffic-routing set \
  --name <app-name> \
  --resource-group <rg> \
  --distribution staging=20
```

### 8.3 Retry Policies

**Function-Level Retry Policy (host.json):**
```json
{
  "version": "2.0",
  "retry": {
    "strategy": "fixedDelay",
    "maxRetryCount": 3,
    "delayInterval": "00:00:05"
  }
}
```

**Retry Strategies:**

**1. Fixed Delay:**
```json
{
  "retry": {
    "strategy": "fixedDelay",
    "maxRetryCount": 5,
    "delayInterval": "00:00:10"
  }
}
```

**2. Exponential Backoff:**
```json
{
  "retry": {
    "strategy": "exponentialBackoff",
    "maxRetryCount": 5,
    "minimumInterval": "00:00:02",
    "maximumInterval": "00:15:00"
  }
}
```

**Function-Level Retry (C# Attribute):**
```csharp
// Fixed delay retry
[FunctionName("RetryFunction")]
[FixedDelayRetry(5, "00:00:10")]
public static void Run(
    [QueueTrigger("myqueue")] string message, ILogger log)
{
    // Will retry up to 5 times with 10 second delay
}

// Exponential backoff retry
[FunctionName("RetryFunction")]
[ExponentialBackoffRetry(5, "00:00:04", "00:15:00")]
public static void Run(
    [QueueTrigger("myqueue")] string message, ILogger log)
{
    // Will retry with exponential backoff
}
```

**Retry Behavior per Trigger:**

| Trigger | Built-in Retry | Custom Retry |
|---------|---------------|--------------|
| HTTP | No | Yes |
| Timer | No | Yes |
| Queue | Yes (via dequeue) | Yes |
| Blob | Yes (via receipt) | Yes |
| Service Bus | Yes (via lock) | Yes |
| Event Hub | Yes (via checkpoint) | Yes |
| Cosmos DB | Yes (via lease) | Yes |

**Important:** HTTP triggers do NOT have built-in retry. The caller must implement retry logic.

### 8.4 Function Timeout Configuration

**Configuring Timeout in host.json:**
```json
{
  "functionTimeout": "00:10:00"
}
```

**Timeout Limits by Plan:**

| Plan | Default | Minimum | Maximum |
|------|---------|---------|---------|
| Consumption | 5 min | 1 sec | 10 min |
| Flex Consumption | 30 min | — | Varies |
| Premium | 30 min | 1 sec | Unlimited* |
| Dedicated | 30 min | 1 sec | Unlimited* |

*Unlimited means no enforced maximum, but set `-1` for unbounded.

```json
// Unlimited timeout (Premium/Dedicated only)
{
  "functionTimeout": "-1"
}
```

### 8.5 Concurrency Configuration

**host.json Concurrency Settings:**
```json
{
  "extensions": {
    "http": {
      "maxConcurrentRequests": 100,
      "routePrefix": "api",
      "dynamicThrottlesEnabled": false
    },
    "queues": {
      "batchSize": 16,
      "newBatchThreshold": 8
    },
    "serviceBus": {
      "messageHandlerOptions": {
        "maxConcurrentCalls": 16
      }
    },
    "eventHubs": {
      "maxEventBatchSize": 100
    },
    "cosmosDB": {
      "connectionMode": "Direct"
    }
  },
  "concurrency": {
    "dynamicConcurrencyEnabled": true,
    "snapshotPersistenceEnabled": true
  }
}
```

**Dynamic Concurrency (Preview):**
- Automatically adjusts concurrency limits
- Learns optimal concurrency over time
- Reduces manual tuning
- Enabled via `dynamicConcurrencyEnabled`

**HTTP Route Prefix:**
```json
{
  "extensions": {
    "http": {
      "routePrefix": "api"
    }
  }
}
```
- Default prefix is `api` → URLs are `https://.../api/functionname`
- Set to `""` (empty string) to remove prefix → `https://.../functionname`
- Set to custom value → `https://.../custom/functionname`

---

## MODULE 9: ADDITIONAL AZURE FUNCTIONS CONCEPTS

### 9.1 Custom Handlers

Custom handlers let you use **any language** with Azure Functions by implementing a lightweight web server.

**How Custom Handlers Work:**
1. Functions host receives trigger event
2. Host forwards HTTP request to your custom handler (web server)
3. Your handler processes the request
4. Handler returns HTTP response to Functions host
5. Host translates response to output bindings

**Architecture:**
```
[Trigger] → [Functions Host] → HTTP → [Custom Handler (Your Code)] → HTTP → [Functions Host] → [Output Bindings]
```

**host.json for Custom Handler:**
```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "myapp",
      "workingDirectory": "",
      "arguments": []
    },
    "enableForwardingHttpRequest": true
  },
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.*, 5.0.0)"
  }
}
```

**Custom Handler Languages:**
- Go
- Rust
- Ruby
- PHP
- Any language with HTTP server capability

**Example: Go Custom Handler:**
```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "os"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    fmt.Fprintf(w, `{"message": "Hello from Go!"}`)
}

func main() {
    port, exists := os.LookupEnv("FUNCTIONS_CUSTOMHANDLER_PORT")
    if !exists {
        port = "8080"
    }
    http.HandleFunc("/api/HttpTrigger", helloHandler)
    log.Fatal(http.ListenAndServe(":"+port, nil))
}
```

**Key Points:**
- Custom handler listens on port defined by `FUNCTIONS_CUSTOMHANDLER_PORT`
- `enableForwardingHttpRequest: true` forwards the original HTTP request directly
- Without forwarding, Functions host wraps trigger data in a custom payload
- Still requires `function.json` for each function

### 9.2 Azure Functions Proxies (Deprecated but May Be Tested)

⚠️ **Deprecated:** Azure Functions Proxies is deprecated. Use **Azure API Management** instead.

**What Proxies Did:**
- Unified API surface across multiple function apps
- URL rewriting and response modification
- Mock APIs for development
- Lightweight API gateway

**proxies.json Example:**
```json
{
  "$schema": "http://json.schemastore.org/proxies",
  "proxies": {
    "proxy1": {
      "matchCondition": {
        "methods": ["GET"],
        "route": "/api/users/{id}"
      },
      "backendUri": "https://other-function-app.azurewebsites.net/api/GetUser/{id}"
    },
    "mockApi": {
      "matchCondition": {
        "methods": ["GET"],
        "route": "/api/mock"
      },
      "responseOverrides": {
        "response.statusCode": "200",
        "response.body": "{\"message\": \"This is a mock response\"}"
      }
    }
  }
}
```

**Migration Path:** Replace proxies with **Azure API Management** policies.

### 9.3 OpenAPI / Swagger for Azure Functions

**NuGet Package:** `Microsoft.Azure.WebJobs.Extensions.OpenApi`

```csharp
// Add OpenAPI attributes to your functions
[FunctionName("GetProducts")]
[OpenApiOperation(operationId: "GetProducts", tags: new[] { "products" })]
[OpenApiParameter(name: "category", In = ParameterLocation.Query, Required = false)]
[OpenApiResponseWithBody(statusCode: HttpStatusCode.OK, contentType: "application/json", bodyType: typeof(Product[]))]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "products")] HttpRequest req)
{
    // Implementation
}
```

**Key Points:**
- Generates OpenAPI/Swagger specification automatically
- Available at `/api/swagger/ui` endpoint
- Useful for API documentation and client code generation

### 9.4 Azure Functions with Azure App Configuration

**Centralized Configuration Management:**

```csharp
// Program.cs (Isolated Worker)
var host = new HostBuilder()
    .ConfigureAppConfiguration(config =>
    {
        var settings = config.Build();
        var connection = settings["AppConfigConnectionString"];
        
        config.AddAzureAppConfiguration(options =>
        {
            options.Connect(connection)
                   .Select("MyApp:*")           // Key filter
                   .ConfigureRefresh(refresh =>
                   {
                       refresh.Register("MyApp:Settings:Sentinel", refreshAll: true)
                              .SetCacheExpiration(TimeSpan.FromSeconds(30));
                   });
        });
    })
    .ConfigureFunctionsWorkerDefaults()
    .Build();
```

**Feature Flags with App Configuration:**
```csharp
// Check feature flag
if (await _featureManager.IsEnabledAsync("BetaFeature"))
{
    // Feature is enabled
}
```

### 9.5 Durable Functions Advanced Topics

#### Eternal Orchestrations (ContinueAsNew)

Long-running orchestrations that never end — use `ContinueAsNew` to reset history:

```csharp
[FunctionName("EternalOrchestrator")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var counterState = context.GetInput<int>() ?? 0;
    
    // Do some work
    counterState = await context.CallActivityAsync<int>("DoWork", counterState);
    
    // Wait before next iteration
    var nextCheck = context.CurrentUtcDateTime.AddMinutes(5);
    await context.CreateTimer(nextCheck, CancellationToken.None);
    
    // Restart orchestration with new input (clears history)
    context.ContinueAsNew(counterState);
    // Code after ContinueAsNew is never executed
}
```

**Why ContinueAsNew?**
- Prevents unbounded execution history growth
- Resets orchestrator state
- Allows true infinite loops without memory issues
- History table doesn't grow indefinitely

#### Singleton Orchestrations

Ensure only one instance of an orchestration runs at a time:

```csharp
[FunctionName("StartSingleton")]
public static async Task<HttpResponseMessage> Start(
    [HttpTrigger(AuthorizationLevel.Function)] HttpRequestMessage req,
    [DurableClient] IDurableOrchestrationClient client)
{
    string instanceId = "MySingletonOrchestration";
    
    // Check if already running
    var existingInstance = await client.GetStatusAsync(instanceId);
    
    if (existingInstance == null || 
        existingInstance.RuntimeStatus == OrchestrationRuntimeStatus.Completed ||
        existingInstance.RuntimeStatus == OrchestrationRuntimeStatus.Failed ||
        existingInstance.RuntimeStatus == OrchestrationRuntimeStatus.Terminated)
    {
        // Start new instance with fixed ID
        await client.StartNewAsync("MyOrchestrator", instanceId, null);
        return client.CreateCheckStatusResponse(req, instanceId);
    }
    
    // Already running - return existing status
    return client.CreateCheckStatusResponse(req, instanceId);
}
```

#### Durable Functions Versioning

**Problem:** Changing orchestrator code while instances are running can break replay.

**Strategies:**

1. **Side-by-Side Deployment:**
```csharp
// Version 1
[FunctionName("MyOrchestrator_v1")]
public static async Task RunV1([OrchestrationTrigger] IDurableOrchestrationContext ctx) { }

// Version 2 (new logic)
[FunctionName("MyOrchestrator_v2")]
public static async Task RunV2([OrchestrationTrigger] IDurableOrchestrationContext ctx) { }
```

2. **Wait for Completion:** Let all running instances complete before deploying new version.

3. **Deployment Slots:** Deploy new version to staging, let production drain, then swap.

#### Durable Functions HTTP Management APIs

Built-in HTTP APIs for managing orchestration instances:

**Status Query:**
```
GET /runtime/webhooks/durabletask/instances/{instanceId}
    ?taskHub={taskHub}
    &connection={connection}
    &code={systemKey}
```

**Start New Instance:**
```
POST /runtime/webhooks/durabletask/orchestrators/{functionName}
     ?code={systemKey}
Body: <input data>
```

**Terminate Instance:**
```
POST /runtime/webhooks/durabletask/instances/{instanceId}/terminate
     ?reason={reason}&code={systemKey}
```

**Raise Event:**
```
POST /runtime/webhooks/durabletask/instances/{instanceId}/raiseEvent/{eventName}
     ?code={systemKey}
Body: <event data>
```

**Purge Instance:**
```
DELETE /runtime/webhooks/durabletask/instances/{instanceId}
       ?code={systemKey}
```

**Query All Instances:**
```
GET /runtime/webhooks/durabletask/instances
    ?runtimeStatus=Running,Pending
    &createdTimeFrom=2024-01-01T00:00:00Z
    &code={systemKey}
```

**Orchestration Status Values:**
- `Running` — Currently executing
- `Completed` — Finished successfully
- `ContinuedAsNew` — Restarted via ContinueAsNew
- `Failed` — Failed with error
- `Canceled` — Gracefully cancelled
- `Terminated` — Force terminated
- `Pending` — Scheduled but not yet started
- `Suspended` — Suspended by user

#### Durable Entities (Class-Based Syntax)

Alternative to function-based entity:

```csharp
[JsonObject(MemberSerialization.OptIn)]
public class Counter : ICounter
{
    [JsonProperty("value")]
    public int Value { get; set; }

    public void Add(int amount)
    {
        Value += amount;
    }

    public void Reset()
    {
        Value = 0;
    }

    public int Get()
    {
        return Value;
    }

    [FunctionName(nameof(Counter))]
    public static Task Run([EntityTrigger] IDurableEntityContext ctx)
        => ctx.DispatchAsync<Counter>();
}

// Interface
public interface ICounter
{
    void Add(int amount);
    void Reset();
    int Get();
}
```

**Calling Class-Based Entity:**
```csharp
// Using typed proxy (strongly typed)
var entityId = new EntityId(nameof(Counter), "myCounter");
var proxy = context.CreateEntityProxy<ICounter>(entityId);
proxy.Add(5);
int current = await proxy.Get();
```

### 9.6 Azure Functions with Application Insights (Deep Dive)

**Sampling Configuration (host.json):**
```json
{
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "maxTelemetryItemsPerSecond": 20,
        "excludedTypes": "Request;Exception",
        "includedTypes": ""
      },
      "enableLiveMetrics": true,
      "httpAutoCollectionOptions": {
        "enableHttpTriggerExtendedInfoCollection": true,
        "enableW3CDistributedTracing": true,
        "enableResponseHeaderInjection": true
      }
    },
    "logLevel": {
      "default": "Information",
      "Host.Results": "Error",
      "Function": "Error",
      "Host.Aggregator": "Trace"
    }
  }
}
```

**Disabling Application Insights:**
- Remove `APPINSIGHTS_INSTRUMENTATIONKEY` and `APPLICATIONINSIGHTS_CONNECTION_STRING`
- Or set sampling to exclude all types

**Custom Telemetry (Isolated Worker):**
```csharp
// Program.cs
services.AddApplicationInsightsTelemetryWorkerService();
services.ConfigureFunctionsApplicationInsights();
```

**Kusto Query Language (KQL) for Functions:**
```kusto
// Find failed function executions
requests
| where success == false
| where timestamp > ago(24h)
| summarize count() by name
| order by count_ desc

// Average function duration
requests
| where timestamp > ago(1h)
| summarize avgDuration = avg(duration) by name
| order by avgDuration desc

// Function execution count over time
requests
| where timestamp > ago(24h)
| summarize count() by bin(timestamp, 1h), name
| render timechart

// Exceptions
exceptions
| where timestamp > ago(24h)
| summarize count() by type, outerMessage
| order by count_ desc

// Dependencies (external calls)
dependencies
| where timestamp > ago(1h)
| summarize avgDuration = avg(duration) by target, type
| order by avgDuration desc
```

### 9.7 Important App Settings Reference

| Setting | Description | Example |
|---------|-------------|---------|
| `FUNCTIONS_EXTENSION_VERSION` | Runtime version | `~4` |
| `FUNCTIONS_WORKER_RUNTIME` | Language runtime | `dotnet`, `node`, `python`, `java`, `powershell`, `dotnet-isolated`, `custom` |
| `AzureWebJobsStorage` | Storage connection | Connection string or identity-based |
| `WEBSITE_RUN_FROM_PACKAGE` | Run from package | `1` or URL |
| `APPINSIGHTS_INSTRUMENTATIONKEY` | App Insights key (legacy) | GUID |
| `APPLICATIONINSIGHTS_CONNECTION_STRING` | App Insights connection | `InstrumentationKey=...` |
| `WEBSITE_TIME_ZONE` | Timezone for timer triggers | `Eastern Standard Time` |
| `FUNCTIONS_WORKER_PROCESS_COUNT` | Worker process count | `1` (default) |
| `WEBSITE_MAX_DYNAMIC_APPLICATION_SCALE_OUT` | Max scale-out instances | `200` (Consumption) |
| `AzureWebJobs.<FUNCTION_NAME>.Disabled` | Disable specific function | `true` / `false` |
| `WEBSITE_CONTENTAZUREFILECONNECTIONSTRING` | Content share connection | Connection string |
| `WEBSITE_CONTENTSHARE` | Content share name | Auto-generated |
| `SCM_DO_BUILD_DURING_DEPLOYMENT` | Remote build | `true` / `false` |

**Worker Runtime Values:**
- `dotnet` — .NET in-process
- `dotnet-isolated` — .NET isolated worker
- `node` — JavaScript/TypeScript
- `python` — Python
- `java` — Java
- `powershell` — PowerShell
- `custom` — Custom handler

### 9.8 Azure Functions with Azure Key Vault

**Key Vault Reference Syntax:**
```
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/ec96f02080254f109c51a1f14cdb1931)
@Microsoft.KeyVault(VaultName=myvault;SecretName=mysecret)
@Microsoft.KeyVault(VaultName=myvault;SecretName=mysecret;SecretVersion=ec96f02080254f109c51a1f14cdb1931)
```

**Prerequisites for Key Vault References:**
1. Function app must have a **managed identity** (system or user-assigned)
2. Managed identity must have **Get** secret permission in Key Vault access policy (or Secret User RBAC role)
3. Key Vault reference syntax in app settings

**Resolution Behavior:**
- Resolved at app startup and during configuration refresh
- If Key Vault is unavailable, app keeps the last resolved value
- Secrets are cached; rotation requires app restart or configuration refresh
- Supports both access policies and RBAC-based access

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**1. Hosting Plans:**
- Differences between Consumption, Premium, Dedicated
- When to use each plan
- Scaling characteristics
- Pricing models
- Feature availability per plan

**2. Triggers and Bindings:**
- Common trigger types
- Input vs output bindings
- Binding configuration
- Trigger-specific behaviors

**3. Durable Functions:**
- Four function types
- Orchestrator constraints
- Durable patterns
- Deterministic requirement

**4. Development and Deployment:**
- Local development tools
- Deployment methods
- CI/CD pipelines
- Configuration management

**5. Performance and Scaling:**
- Cold start mitigation
- Scaling behavior
- Connection management
- Best practices

### Sample Exam Questions

#### Hosting Plans

**Q1:** You need to run Azure Functions that require VNet integration and have predictable costs. Which hosting plan should you use?
- A) Consumption
- B) Flex Consumption
- C) Premium
- D) Dedicated

**Answer: C) Premium**
Explanation: Premium plan provides VNet integration, pre-warmed instances, and more predictable costs than Consumption. Dedicated also supports VNet but Premium is specifically designed for serverless with these features.

---

**Q2:** What is the maximum execution timeout for a function in the Consumption plan?
- A) 5 minutes
- B) 10 minutes
- C) 30 minutes
- D) Unlimited

**Answer: B) 10 minutes**
Explanation: Consumption plan has a default timeout of 5 minutes, configurable up to 10 minutes maximum.

---

**Q3:** Which hosting plan scales to zero instances when idle?
- A) Premium plan only
- B) Consumption plan only
- C) Both Consumption and Flex Consumption
- D) Dedicated plan

**Answer: C) Both Consumption and Flex Consumption**
Explanation: Both Consumption and Flex Consumption plans can scale to zero. Premium requires minimum of 1 instance.

---

**Q4:** You need to eliminate cold starts for your production function app. What should you do?
- A) Use Consumption plan with Always On
- B) Use Premium plan with pre-warmed instances
- C) Use Dedicated plan only
- D) Increase timeout value

**Answer: B) Use Premium plan with pre-warmed instances**
Explanation: Premium plan with pre-warmed instances eliminates cold starts. Consumption plan doesn't support Always On. Dedicated plan needs Always On enabled.

---

**Q5:** What is the monthly free grant for the Consumption plan?
- A) 500,000 executions and 200,000 GB-s
- B) 1 million executions and 400,000 GB-s
- C) 1 million executions and 500,000 GB-s
- D) 250,000 executions and 100,000 GB-s

**Answer: B) 1 million executions and 400,000 GB-s**
Explanation: Consumption plan provides 1 million free executions and 400,000 GB-s per month per subscription. Flex Consumption has different grants (250K executions, 100K GB-s).

---

#### Triggers and Bindings

**Q6:** How many triggers can a single function have?
- A) 0
- B) 1
- C) Multiple
- D) Unlimited

**Answer: B) 1**
Explanation: Every function must have exactly ONE trigger. Functions can have zero or more input/output bindings, but only one trigger.

---

**Q7:** Which trigger type is best for processing messages with guaranteed delivery and dead-letter queue support?
- A) Queue trigger
- B) Service Bus trigger
- C) Event Hub trigger
- D) HTTP trigger

**Answer: B) Service Bus trigger**
Explanation: Service Bus provides enterprise messaging features including guaranteed delivery, dead-letter queues, sessions, and transactions.

---

**Q8:** What is the CRON expression for a Timer trigger that runs every day at 9:00 AM?
- A) `0 0 9 * * *`
- B) `0 9 * * * *`
- C) `* 0 9 * * *`
- D) `0 * 9 * * *`

**Answer: A) `0 0 9 * * *`**
Explanation: CRON format is `{second} {minute} {hour} {day} {month} {day-of-week}`. So `0 0 9 * * *` means at second 0, minute 0, hour 9, every day.

---

**Q9:** You want to read data from Cosmos DB and write to a Storage Queue. Which bindings do you need?
- A) Cosmos DB trigger, Queue output binding
- B) Cosmos DB input binding, Queue output binding
- C) HTTP trigger, Cosmos DB input, Queue output
- D) Cosmos DB trigger, Queue input binding

**Answer: B) Cosmos DB input binding, Queue output binding**
Explanation: To read from Cosmos DB, use input binding. To write to queue, use output binding. Trigger is separate and determines what invokes the function.

---

**Q10:** Which binding expression generates a random GUID?
- A) `{DateTime}`
- B) `{rand-guid}`
- C) `{newid}`
- D) `{guid}`

**Answer: B) `{rand-guid}`**
Explanation: The binding expression `{rand-guid}` generates a random GUID. Other expressions include `{DateTime}` for current time, `{name}` for trigger properties.

---

#### Durable Functions

**Q11:** Which type of durable function can directly perform I/O operations and make HTTP calls?
- A) Orchestrator function
- B) Activity function
- C) Entity function
- D) None of the above

**Answer: B) Activity function**
Explanation: Activity functions have no restrictions and can perform I/O, make HTTP calls, access databases, etc. Orchestrator functions must be deterministic and cannot do these operations directly.

---

**Q12:** What happens when an orchestrator function is replayed?
- A) All activity functions are re-executed
- B) Activity function results are retrieved from execution history
- C) The orchestration fails
- D) A new instance is created

**Answer: B) Activity function results are retrieved from execution history**
Explanation: During replay, the orchestrator checks execution history. If an activity has already run, it replays the result instead of re-executing the activity.

---

**Q13:** Which of the following is allowed in an orchestrator function?
- A) DateTime.Now
- B) Random.Next()
- C) HttpClient.GetAsync()
- D) context.CurrentUtcDateTime

**Answer: D) context.CurrentUtcDateTime**
Explanation: Orchestrators must be deterministic. Use context.CurrentUtcDateTime (deterministic) instead of DateTime.Now. For I/O operations, call activity functions.

---

**Q14:** How do you call an activity function from an orchestrator?
- A) Direct function call
- B) await context.CallActivityAsync()
- C) Task.Run()
- D) await activityFunction()

**Answer: B) await context.CallActivityAsync()**
Explanation: Activity functions must be called through the orchestration context using CallActivityAsync method, not directly.

---

**Q15:** Which durable function pattern is best for executing multiple tasks in parallel and aggregating results?
- A) Function chaining
- B) Fan-out/fan-in
- C) Async HTTP APIs
- D) Monitor

**Answer: B) Fan-out/fan-in**
Explanation: Fan-out/fan-in pattern executes multiple activities in parallel (fan-out), waits for all to complete, then aggregates results (fan-in).

---

**Q16:** You need to wait for user approval with a 24-hour timeout. Which pattern should you use?
- A) Function chaining
- B) Fan-out/fan-in
- C) Human interaction
- D) Monitor

**Answer: C) Human interaction**
Explanation: Human interaction pattern uses external events and timers to wait for human input with timeout capability.

---

**Q17:** Which durable function type maintains stateful entities?
- A) Orchestrator function
- B) Activity function
- C) Entity function
- D) Client function

**Answer: C) Entity function**
Explanation: Entity functions are specifically designed to maintain and update stateful entities with operations like counters, shopping carts, etc.

---

**Q18:** How many orchestrator functions can a client function start?
- A) Only one
- B) Maximum of 10
- C) Multiple (no limit)
- D) None

**Answer: C) Multiple (no limit)**
Explanation: A client function can start multiple orchestration instances. It's not limited to starting only one.

---

**Q19:** What storage does Durable Functions use by default for state management?
- A) Cosmos DB
- B) Azure Storage (Tables and Queues)
- C) SQL Database
- D) Redis Cache

**Answer: B) Azure Storage (Tables and Queues)**
Explanation: By default, Durable Functions uses Azure Storage (Tables for execution history, Queues for messaging). Other providers like Netherite and MSSQL are also supported.

---

**Q20:** Can an orchestrator function call another orchestrator function?
- A) No, never
- B) Yes, using CallSubOrchestratorAsync
- C) Yes, but only directly
- D) Only in Premium plan

**Answer: B) Yes, using CallSubOrchestratorAsync**
Explanation: Orchestrators can call sub-orchestrations using CallSubOrchestratorAsync method. This allows building complex workflows from smaller orchestrations.

---

#### Development and Configuration

**Q21:** Which file contains global configuration for all functions in a function app?
- A) function.json
- B) local.settings.json
- C) host.json
- D) appsettings.json

**Answer: C) host.json**
Explanation: host.json contains runtime-wide settings that apply to all functions. function.json is function-specific (non-.NET). local.settings.json is for local development only.

---

**Q22:** How do you access an app setting named "MyApiKey" in a C# function?
- A) AppSettings["MyApiKey"]
- B) ConfigurationManager.AppSettings["MyApiKey"]
- C) Environment.GetEnvironmentVariable("MyApiKey")
- D) context.Settings["MyApiKey"]

**Answer: C) Environment.GetEnvironmentVariable("MyApiKey")**
Explanation: In Azure Functions, app settings are exposed as environment variables. Access them using Environment.GetEnvironmentVariable().

---

**Q23:** What is the purpose of local.settings.json?
- A) Deployed to Azure with function app
- B) Local development configuration only
- C) Function-specific settings
- D) Trigger configuration

**Answer: B) Local development configuration only**
Explanation: local.settings.json is used only for local development and is NOT deployed to Azure. Production settings are configured as app settings in Azure.

---

**Q24:** Which command deploys a function app using Azure Functions Core Tools?
- A) func deploy
- B) func publish
- C) func azure functionapp publish <app-name>
- D) func push

**Answer: C) func azure functionapp publish <app-name>**
Explanation: The correct command is `func azure functionapp publish <app-name>` to deploy functions to Azure.

---

**Q25:** You want to reference a secret from Azure Key Vault in an app setting. What syntax should you use?
- A) `keyvault://<vault>/<secret>`
- B) `@Microsoft.KeyVault(SecretUri=<uri>)`
- C) `vault://<vault>/<secret>`
- D) `{KeyVault:<vault>:<secret>}`

**Answer: B) @Microsoft.KeyVault(SecretUri=<uri>)**
Explanation: The correct Key Vault reference syntax is `@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)`.

---

#### Performance and Best Practices

**Q26:** Which of the following is a best practice for HTTP client usage in Azure Functions?
- A) Create new HttpClient in each execution
- B) Use static HttpClient instance
- C) Dispose HttpClient after each use
- D) Create HttpClient in constructor

**Answer: B) Use static HttpClient instance**
Explanation: Reuse HttpClient instances across executions to avoid socket exhaustion. Don't create new instances per execution.

---

**Q27:** What causes a cold start in Azure Functions?
- A) Function execution error
- B) First execution after scaling to zero
- C) Timeout occurrence
- D) Large response size

**Answer: B) First execution after scaling to zero**
Explanation: Cold start occurs when a function app scales to zero and needs to be reloaded on the first subsequent request.

---

**Q28:** Which log level should be used for production to reduce noise?
- A) Trace
- B) Debug
- C) Information
- D) Warning

**Answer: C) Information**
Explanation: Information level is appropriate for production. Trace and Debug generate too much noise. Warning might miss important operational logs.

---

**Q29:** You have a function that processes 100 queue messages. How can you optimize execution count costs?
- A) Process messages one at a time
- B) Use batch processing in a single execution
- C) Increase timeout
- D) Use Premium plan

**Answer: B) Use batch processing in a single execution**
Explanation: Processing multiple messages in a single execution reduces the total number of executions and therefore costs.

---

**Q30:** What is the maximum HTTP request timeout in Azure Functions?
- A) 60 seconds
- B) 120 seconds
- C) 230 seconds
- D) 300 seconds

**Answer: C) 230 seconds**
Explanation: HTTP triggered functions have a maximum timeout of 230 seconds regardless of plan, due to Azure Load Balancer idle timeout.

---

#### Advanced Scenarios

**Q31:** You need to ensure your timer-triggered function runs only once across multiple instances. What should you configure?
- A) Set maxConcurrentInstances to 1
- B) Enable UseLock for the trigger
- C) Use Singleton pattern
- D) Configure RunOnStartup

**Answer: B) Enable UseLock for the trigger**
Explanation: Timer triggers support the UseLock setting to ensure only one instance executes the function, preventing concurrent runs.

---

**Q32:** Which statement about function.json is correct?
- A) Required for all languages
- B) Not used in .NET (attributes used instead)
- C) Contains runtime configuration
- D) Deployed only to production

**Answer: B) Not used in .NET (attributes used instead)**
Explanation: .NET functions use attributes for trigger and binding configuration. function.json is used for JavaScript, Python, Java, and other languages.

---

**Q33:** You want to run a long-running workflow that takes 45 minutes. Which hosting plan should you use?
- A) Consumption plan
- B) Premium plan or Dedicated plan
- C) Any plan with increased timeout
- D) Flex Consumption plan

**Answer: B) Premium plan or Dedicated plan**
Explanation: Consumption plan max timeout is 10 minutes. Premium (with unbounded timeout) or Dedicated plans support long-running functions.

---

**Q34:** What is the purpose of Application Insights in Azure Functions?
- A) Function deployment
- B) Monitoring, diagnostics, and telemetry
- C) Authentication
- D) Scaling configuration

**Answer: B) Monitoring, diagnostics, and telemetry**
Explanation: Application Insights provides monitoring, performance metrics, exception tracking, custom telemetry, and diagnostics for Azure Functions.

---

**Q35:** You have an orchestrator function that needs to generate a random number. What should you use?
- A) Random.Next()
- B) Guid.NewGuid()
- C) context.NewGuid()
- D) DateTime.Now.Ticks

**Answer: C) context.NewGuid()**
Explanation: Orchestrators must be deterministic. Use context.NewGuid() which is deterministic on replay, not Random.Next() or Guid.NewGuid().

---

#### Function Access Keys & Authorization

**Q36:** Which key type provides access to ALL functions in a function app AND administrative runtime APIs?
- A) Function key
- B) Host key
- C) Master key (_master)
- D) System key

**Answer: C) Master key (_master)**
Explanation: The master key provides access to all functions plus administrative APIs (management endpoints). Host keys access all functions but not admin APIs. Function keys are scoped to one function.

---

**Q37:** You have an HTTP-triggered function with `AuthorizationLevel.Function`. Which keys can be used to invoke it?
- A) Function key only
- B) Host key only
- C) Function key or host key
- D) Master key only

**Answer: C) Function key or host key**
Explanation: AuthorizationLevel.Function accepts function-specific keys OR host keys (including master key). Host keys work for any function in the app.

---

**Q38:** You are creating an Azure Function that will be called by Azure API Management. What authorization level should you set?
- A) Function
- B) Admin
- C) Anonymous
- D) System

**Answer: C) Anonymous**
Explanation: When using API Management in front of Azure Functions, set the function to Anonymous and let APIM handle authentication. This avoids double-auth and lets APIM control access.

---

**Q39:** How do you pass a function key when calling an HTTP-triggered function?
- A) In the request body as JSON
- B) Via query parameter `?code=<key>` or `x-functions-key` header
- C) Via Authorization: Bearer header only
- D) Via cookie

**Answer: B) Via query parameter `?code=<key>` or `x-functions-key` header**
Explanation: Function keys can be passed as a query string parameter named `code` or in the `x-functions-key` HTTP header.

---

**Q40:** What is the default authorization level for HTTP-triggered functions?
- A) Anonymous
- B) Function
- C) Admin
- D) System

**Answer: B) Function**
Explanation: The default authorization level is Function, which requires a function key or host key to invoke.

---

#### .NET Execution Models

**Q41:** What is the key difference between Azure Functions in-process and isolated worker models?
- A) In-process supports more languages
- B) Isolated worker runs in a separate process from the Functions host
- C) In-process supports middleware
- D) Isolated worker doesn't support dependency injection

**Answer: B) Isolated worker runs in a separate process from the Functions host**
Explanation: The isolated worker model runs your function code in a separate .NET worker process, while in-process runs in the same process as the Functions host.

---

**Q42:** Which attribute is used to define a function name in the isolated worker model?
- A) `[FunctionName("name")]`
- B) `[Function("name")]`
- C) `[AzureFunction("name")]`
- D) `[WebJob("name")]`

**Answer: B) `[Function("name")]`**
Explanation: Isolated worker uses `[Function("name")]` attribute. In-process uses `[FunctionName("name")]`. This is a common exam trick question.

---

**Q43:** Where do you configure dependency injection in the isolated worker model?
- A) Startup.cs with FunctionsStartup
- B) Program.cs with HostBuilder
- C) host.json
- D) function.json

**Answer: B) Program.cs with HostBuilder**
Explanation: Isolated worker uses Program.cs with HostBuilder.ConfigureServices() for DI. In-process uses a Startup class inheriting from FunctionsStartup.

---

**Q44:** Which feature is available ONLY in the isolated worker model (not in-process)?
- A) Dependency injection
- B) Middleware pipeline
- C) Trigger bindings
- D) Application Insights

**Answer: B) Middleware pipeline**
Explanation: Middleware (IFunctionsWorkerMiddleware) is only available in the isolated worker model. DI, triggers, and App Insights are available in both models.

---

**Q45:** What HTTP types does the isolated worker model use by default (without ASP.NET Core integration)?
- A) HttpRequest / IActionResult
- B) HttpRequestData / HttpResponseData
- C) HttpRequestMessage / HttpResponseMessage
- D) HttpContext / HttpResponse

**Answer: B) HttpRequestData / HttpResponseData**
Explanation: Isolated worker model uses HttpRequestData/HttpResponseData by default. HttpRequest/IActionResult are used in the in-process model or with ASP.NET Core integration enabled.

---

**Q46:** In the isolated worker model, how do you configure dependency injection?
- A) Using a Startup class with `FunctionsStartup` attribute
- B) In `Program.cs` using `HostBuilder.ConfigureServices()`
- C) In `host.json` under "services"
- D) Using `[Inject]` attribute on parameters

**Answer: B) In `Program.cs` using `HostBuilder.ConfigureServices()`**
Explanation: In isolated worker model, DI is configured in Program.cs using standard .NET HostBuilder pattern: `new HostBuilder().ConfigureServices(services => { services.AddSingleton<IMyService, MyService>(); })`.

---

**Q47:** For the in-process model, how do you configure dependency injection?
- A) Program.cs with HostBuilder
- B) Startup class inheriting from `FunctionsStartup` with `[assembly: FunctionsStartup]`
- C) Directly in function constructor
- D) In host.json

**Answer: B) Startup class inheriting from `FunctionsStartup` with `[assembly: FunctionsStartup]`**
Explanation: In-process model uses a Startup class that inherits from FunctionsStartup and overrides Configure(IFunctionsHostBuilder builder) method, registered with the assembly attribute.

---

#### Advanced Triggers & Bindings

**Q48:** What happens to a queue message after it fails processing `maxDequeueCount` times?
- A) It is deleted permanently
- B) It is moved to a poison queue (`{queue-name}-poison`)
- C) It stays in the original queue forever
- D) It is sent back to the sender

**Answer: B) It is moved to a poison queue (`{queue-name}-poison`)**
Explanation: After maxDequeueCount failed attempts (default 5), the message is moved to a poison queue named `{original-queue-name}-poison`.

---

**Q49:** What is the default `maxDequeueCount` for queue-triggered functions?
- A) 1
- B) 3
- C) 5
- D) 10

**Answer: C) 5**
Explanation: The default maxDequeueCount is 5. After 5 failed processing attempts, the message moves to the poison queue.

---

**Q50:** Which approach provides the FASTEST blob processing for Azure Functions?
- A) Blob trigger with storage logs
- B) Blob trigger with Event Grid source
- C) Timer trigger polling for blobs
- D) HTTP trigger checking blob changes

**Answer: B) Blob trigger with Event Grid source**
Explanation: Event Grid-based blob trigger provides near real-time processing (seconds), while the default blob trigger using storage logs can have up to 10-minute delay.

---

**Q51:** In a Cosmos DB trigger, what container is required to track processing state?
- A) Monitor container
- B) Lease container
- C) Change container
- D) State container

**Answer: B) Lease container**
Explanation: Cosmos DB trigger uses a lease container to store lease documents that track the change feed processing state. It must be configured via `LeaseContainerName`.

---

**Q52:** The Cosmos DB change feed processes which types of operations?
- A) Inserts only
- B) Inserts and updates only
- C) Inserts, updates, and deletes
- D) All CRUD operations

**Answer: B) Inserts and updates only**
Explanation: The Cosmos DB change feed only processes inserts and updates. Deletes are NOT captured. Use soft-delete pattern with TTL to handle deletions.

---

**Q53:** What is the CRON expression for running a timer-triggered function at 2:30 PM every weekday?
- A) `0 30 14 * * 1-5`
- B) `30 14 * * 1-5 *`
- C) `0 30 2 * * MON-FRI`
- D) `0 14 30 * * 1-5`

**Answer: A) `0 30 14 * * 1-5`**
Explanation: CRON format is {second} {minute} {hour} {day} {month} {day-of-week}. 2:30 PM = hour 14, minute 30. Weekdays = 1-5 (Monday-Friday).

---

**Q54:** What does the `RunOnStartup = true` property do on a timer trigger?
- A) Runs the function once when the app starts
- B) Runs the function immediately on startup AND continues on schedule
- C) Replaces the schedule with startup-only execution
- D) Enables hot-start for the function

**Answer: B) Runs the function immediately on startup AND continues on schedule**
Explanation: RunOnStartup = true causes the function to execute immediately when the function app starts (or restarts), in addition to the normal schedule. Not recommended for production with Consumption plan.

---

**Q55:** How do you configure the default HTTP route prefix for all HTTP functions?
- A) In function.json `routePrefix` property
- B) In host.json under `extensions.http.routePrefix`
- C) In local.settings.json
- D) In the function attribute

**Answer: B) In host.json under `extensions.http.routePrefix`**
Explanation: The route prefix is configured globally in host.json. Default is "api". Set to "" (empty) to remove it. Example: `"extensions": { "http": { "routePrefix": "api" } }`.

---

#### Networking, Security & Identity

**Q56:** Which hosting plans support VNet integration for Azure Functions?
- A) All plans
- B) Premium and Dedicated only
- C) Premium, Dedicated, and Flex Consumption
- D) Premium only

**Answer: C) Premium, Dedicated, and Flex Consumption**
Explanation: VNet integration is available in Premium, Dedicated, and Flex Consumption plans. It is NOT available in the standard Consumption plan.

---

**Q57:** What is the name of Azure Functions' built-in authentication feature?
- A) Azure AD Connect
- B) EasyAuth (App Service Authentication)
- C) Function Guard
- D) Azure Sentinel

**Answer: B) EasyAuth (App Service Authentication)**
Explanation: Built-in authentication for Azure Functions is called EasyAuth (App Service Authentication). It runs as a sidecar module and supports multiple identity providers.

---

**Q58:** You want your function app to access Azure Blob Storage without using connection strings. What should you use?
- A) SAS tokens in app settings
- B) Managed identity with identity-based connection
- C) Anonymous access to the storage account
- D) API key in request header

**Answer: B) Managed identity with identity-based connection**
Explanation: Identity-based connections use managed identity instead of connection strings. Configure with `__accountName` suffix (e.g., `AzureWebJobsStorage__accountName`).

---

**Q59:** What is the naming convention for identity-based Storage connections in app settings?
- A) `ConnectionName.accountName`
- B) `ConnectionName__accountName` (double underscore)
- C) `ConnectionName-accountName`
- D) `ConnectionName:accountName`

**Answer: B) `ConnectionName__accountName` (double underscore)**
Explanation: Identity-based connections use double underscore `__` as separator. For storage: `AzureWebJobsStorage__accountName`, for Service Bus: `ServiceBusConnection__fullyQualifiedNamespace`.

---

**Q60:** Which Azure credential class should you use that works with managed identity in Azure and falls back to local development credentials?
- A) ManagedIdentityCredential
- B) DefaultAzureCredential
- C) ClientSecretCredential
- D) AzureCliCredential

**Answer: B) DefaultAzureCredential**
Explanation: DefaultAzureCredential tries multiple credential types in order: environment variables, workload identity, managed identity, Visual Studio, Azure CLI, etc. Works in both Azure and local development.

---

**Q61:** What RBAC role is needed for a managed identity to read blobs from Azure Storage?
- A) Storage Account Contributor
- B) Storage Blob Data Reader
- C) Reader
- D) Storage Blob Data Owner

**Answer: B) Storage Blob Data Reader**
Explanation: Storage Blob Data Reader provides read access to blob data. Storage Account Contributor manages the account but doesn't grant data access. Reader provides read access to management plane only.

---

#### Deployment & Configuration

**Q62:** What does setting `WEBSITE_RUN_FROM_PACKAGE=1` do?
- A) Compresses the function app at runtime
- B) Runs the function from a mounted read-only package file
- C) Enables package caching
- D) Deploys from a container image

**Answer: B) Runs the function from a mounted read-only package file**
Explanation: WEBSITE_RUN_FROM_PACKAGE=1 runs the function app from a ZIP package file. The file system becomes read-only, deployment is atomic, and cold start times are reduced.

---

**Q63:** What are "sticky settings" (slot settings) in deployment slots?
- A) Settings that persist after app restart
- B) Settings that don't swap with the slot (stay with the slot)
- C) Settings that are encrypted
- D) Settings shared between all slots

**Answer: B) Settings that don't swap with the slot (stay with the slot)**
Explanation: Sticky settings (deployment slot settings) remain with the slot during a swap. Common examples: database connection strings, Application Insights keys, feature flags.

---

**Q64:** Which retry strategy uses increasing delays between retry attempts?
- A) Fixed delay
- B) Exponential backoff
- C) Linear delay
- D) Immediate retry

**Answer: B) Exponential backoff**
Explanation: Exponential backoff increases the delay between retries exponentially (e.g., 2s, 4s, 8s, 16s...), up to a configured maximum. Fixed delay uses the same interval every time.

---

**Q65:** What is the correct value for `FUNCTIONS_WORKER_RUNTIME` when using .NET isolated worker model?
- A) `dotnet`
- B) `dotnet-isolated`
- C) `dotnet-worker`
- D) `net-isolated`

**Answer: B) `dotnet-isolated`**
Explanation: For isolated worker model, set FUNCTIONS_WORKER_RUNTIME to `dotnet-isolated`. For in-process, use `dotnet`. This distinction is critical.

---

#### Durable Functions Advanced

**Q66:** An orchestrator function needs to run indefinitely, checking a condition every 5 minutes. How should you prevent the execution history from growing unboundedly?
- A) Use a while(true) loop
- B) Use `context.ContinueAsNew()` to restart with cleared history
- C) Delete history manually
- D) Set maxHistorySize in host.json

**Answer: B) Use `context.ContinueAsNew()` to restart with cleared history**
Explanation: ContinueAsNew restarts the orchestration with a new execution, clearing the history. This prevents unbounded history growth in eternal/infinite orchestrations.

---

**Q67:** How do you ensure only one instance of an orchestration runs at a time (singleton)?
- A) Set maxInstances=1 in host.json
- B) Use a fixed instance ID and check status before starting
- C) Use the [Singleton] attribute
- D) Configure single-instance in Azure Portal

**Answer: B) Use a fixed instance ID and check status before starting**
Explanation: Start orchestrations with a deterministic instance ID. Before starting, check if an instance with that ID is already running. If so, skip starting a new one.

---

**Q68:** What HTTP status code does `CreateCheckStatusResponse` return when starting an orchestration?
- A) 200 OK
- B) 201 Created
- C) 202 Accepted
- D) 204 No Content

**Answer: C) 202 Accepted**
Explanation: CreateCheckStatusResponse returns 202 Accepted with a body containing management URLs (statusQueryGetUri, sendEventPostUri, terminatePostUri, purgeHistoryDeleteUri).

---

**Q69:** What key is required to access the Durable Functions HTTP management APIs?
- A) Function key
- B) Host key
- C) System key (or master key)
- D) No key required

**Answer: C) System key (or master key)**
Explanation: Durable Functions HTTP APIs require the `durabletask` system key or the master key (_master) for access. These are separate from regular function/host keys.

---

**Q70:** Which Durable Functions pattern should you use for a workflow that processes a dynamic list of items in parallel?
- A) Function chaining
- B) Fan-out/fan-in
- C) Monitor
- D) Human interaction

**Answer: B) Fan-out/fan-in**
Explanation: Fan-out/fan-in starts multiple activity functions in parallel (fan-out) and waits for all to complete (fan-in) using Task.WhenAll(). Perfect for processing dynamic collections in parallel.

---

#### Custom Handlers & Miscellaneous

**Q71:** What environment variable must a custom handler's web server listen on?
- A) `PORT`
- B) `FUNCTIONS_CUSTOMHANDLER_PORT`
- C) `HTTP_PLATFORM_PORT`
- D) `ASPNETCORE_URLS`

**Answer: B) `FUNCTIONS_CUSTOMHANDLER_PORT`**
Explanation: The Functions host sets FUNCTIONS_CUSTOMHANDLER_PORT to tell the custom handler which port to listen on. The custom handler must start an HTTP server on this port.

---

**Q72:** Which of the following is the correct Key Vault reference syntax for an app setting?
- A) `keyvault://myvault/mysecret`
- B) `@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)`
- C) `@KeyVault(myvault, mysecret)`
- D) `${keyvault:myvault:mysecret}`

**Answer: B) `@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)`**
Explanation: Key Vault references use the syntax @Microsoft.KeyVault(SecretUri=...) or @Microsoft.KeyVault(VaultName=...;SecretName=...).

---

**Q73:** What is required for Key Vault references to work in Azure Functions app settings?
- A) Application Insights connection
- B) Managed identity with Key Vault access
- C) Premium hosting plan
- D) VNet integration

**Answer: B) Managed identity with Key Vault access**
Explanation: The function app needs a managed identity (system or user-assigned) with Get permission for secrets in the Key Vault's access policy or the Key Vault Secrets User RBAC role.

---

**Q74:** Extension bundles in host.json are required for which languages?
- A) All languages including .NET
- B) Only JavaScript and Python
- C) Non-.NET languages (JavaScript, Python, Java, PowerShell)
- D) Only custom handler languages

**Answer: C) Non-.NET languages (JavaScript, Python, Java, PowerShell)**
Explanation: Extension bundles automatically manage binding extensions for non-.NET languages. .NET projects use NuGet packages directly instead.

---

**Q75:** You need to create an Azure Function using Go. What approach should you use?
- A) Install Go runtime on Azure Functions
- B) Use a custom handler
- C) Use the Go SDK for Azure Functions
- D) Deploy as a container with Go runtime

**Answer: B) Use a custom handler**
Explanation: Custom handlers allow any language with HTTP server capability (Go, Rust, Ruby, etc.) to work with Azure Functions by implementing a lightweight HTTP server.

---

**Q76:** What happens during a deployment slot swap?
- A) Code is copied from source to target
- B) Source slot gets target's settings, then DNS routing is swapped
- C) Both slots are deleted and recreated
- D) Only the code files are swapped

**Answer: B) Source slot gets target's settings, then DNS routing is swapped**
Explanation: During swap, the source slot is warmed up with target slot settings first, then the DNS routing is switched. Sticky settings stay with their respective slots.

---

**Q77:** Which of the following is NOT a valid orchestration status in Durable Functions?
- A) Running
- B) Completed
- C) Paused
- D) Suspended

**Answer: C) Paused**
Explanation: Valid statuses are: Running, Completed, ContinuedAsNew, Failed, Canceled, Terminated, Pending, and Suspended. "Paused" is not a valid status.

---

**Q78:** You want to disable a specific function without redeploying. What app setting should you create?
- A) `DISABLE_<FUNCTION_NAME>=true`
- B) `AzureWebJobs.<FUNCTION_NAME>.Disabled=true`
- C) `FUNCTIONS_DISABLE_<FUNCTION_NAME>=true`
- D) `<FUNCTION_NAME>_DISABLED=true`

**Answer: B) `AzureWebJobs.<FUNCTION_NAME>.Disabled=true`**
Explanation: The naming convention is `AzureWebJobs.<FUNCTION_NAME>.Disabled` set to `true` or `1`. This disables the function without deployment.

---

**Q79:** What is the maximum HTTP request/response body size supported by Azure Functions?
- A) 10 MB
- B) 50 MB
- C) 100 MB
- D) 500 MB

**Answer: C) 100 MB**
Explanation: Both HTTP request and response sizes are limited to 100 MB in Azure Functions. The HTTP request timeout is 230 seconds.

---

**Q80:** In the Service Bus trigger, what does `autoComplete: true` mean?
- A) Messages are auto-deleted from the queue
- B) Messages are automatically marked as completed after successful function execution
- C) The function automatically retries on failure
- D) Messages are auto-forwarded to dead-letter queue

**Answer: B) Messages are automatically marked as completed after successful function execution**
Explanation: When autoComplete is true (default), the runtime automatically completes the message after the function succeeds. Set to false if you want to manually complete/abandon/dead-letter messages.

---

## Key CLI Commands Reference

### Function App Management

```bash
# Create storage account
az storage account create \
  --name <storage-name> \
  --resource-group <rg> \
  --location <location> \
  --sku Standard_LRS

# Create Consumption function app
az functionapp create \
  --resource-group <rg> \
  --consumption-plan-location <location> \
  --runtime <dotnet|node|python|java|powershell> \
  --runtime-version <version> \
  --functions-version 4 \
  --name <app-name> \
  --storage-account <storage-name>

# Create Premium plan
az functionapp plan create \
  --resource-group <rg> \
  --name <plan-name> \
  --location <location> \
  --sku EP1

# Create Premium function app
az functionapp create \
  --resource-group <rg> \
  --plan <plan-name> \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4 \
  --name <app-name> \
  --storage-account <storage-name>

# List function apps
az functionapp list --resource-group <rg> --output table

# Delete function app
az functionapp delete --name <app-name> --resource-group <rg>
```

### Configuration

```bash
# Set app settings
az functionapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --settings KEY1=VALUE1 KEY2=VALUE2

# List app settings
az functionapp config appsettings list \
  --name <app-name> \
  --resource-group <rg>

# Delete app setting
az functionapp config appsettings delete \
  --name <app-name> \
  --resource-group <rg> \
  --setting-names KEY1

# Enable Application Insights
az functionapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --settings APPLICATIONINSIGHTS_CONNECTION_STRING="<connection-string>"
```

### Local Development (Core Tools)

```bash
# Initialize new project
func init <project-name> --worker-runtime <dotnet|node|python|java|powershell>

# Create new function
func new --name <function-name> --template "HTTP trigger"

# Run locally
func start

# Run on specific port
func start --port 7072

# Deploy to Azure
func azure functionapp publish <app-name>

# Deploy to slot
func azure functionapp publish <app-name> --slot <slot-name>

# List functions
func list

# Fetch app settings from Azure
func azure functionapp fetch-app-settings <app-name>
```

### Deployment and Scaling

```bash
# Deploy ZIP
az functionapp deployment source config-zip \
  --resource-group <rg> \
  --name <app-name> \
  --src <zip-file>

# Scale Premium plan
az functionapp plan update \
  --name <plan-name> \
  --resource-group <rg> \
  --sku EP2

# Create deployment slot
az functionapp deployment slot create \
  --name <app-name> \
  --resource-group <rg> \
  --slot staging

# Swap slots
az functionapp deployment slot swap \
  --resource-group <rg> \
  --name <app-name> \
  --slot staging \
  --target-slot production
```

### Monitoring

```bash
# View logs (streaming)
az webapp log tail --name <app-name> --resource-group <rg>

# Download logs
az webapp log download \
  --name <app-name> \
  --resource-group <rg> \
  --log-file logs.zip

# View metrics
az monitor metrics list \
  --resource <app-resource-id> \
  --metric FunctionExecutionCount

# Enable logging
az functionapp log config \
  --name <app-name> \
  --resource-group <rg> \
  --application-logging true
```

---

## Exam Gotchas & Tricky Distinctions

| Common Misconception | Reality |
|---------------------|---------|
| "Consumption plan supports VNet integration" | ❌ Only **Premium** and **Dedicated (App Service)** plans support VNet integration. Consumption does NOT |
| "Consumption plan supports deployment slots" | ❌ Deployment slots require **Premium** or **Dedicated** plan. Consumption does NOT support slots |
| "Consumption plan has no timeout limit" | ❌ Default timeout is **5 minutes**, max configurable is **10 minutes**. Premium/Dedicated default 30 min, max unlimited |
| "Azure Functions always need a storage account" | ✅ Mostly true — Consumption and Premium plans **always** require Azure Storage (for triggers, state, keys). Dedicated plan can run without it for non-Durable, non-Timer functions, but it's still recommended |
| "Durable Functions orchestrators can use `DateTime.Now`" | ❌ Orchestrators must be **deterministic**. Use `IDurableOrchestrationContext.CurrentUtcDateTime` instead of `DateTime.Now/UtcNow` |
| "Durable Functions orchestrators can call HTTP APIs directly" | ❌ Orchestrators must NOT do I/O directly. Use `CallHttpAsync()` on the context or delegate to an **Activity function** |
| "Durable Functions orchestrators can use `Task.Delay`" | ❌ Must use `context.CreateTimer()` — standard `Task.Delay` breaks deterministic replay |
| "Durable Functions orchestrators can use `Guid.NewGuid()`" | ❌ Use `context.NewGuid()` — random values break deterministic replay |
| "`[FunctionName]` and `[Function]` attributes are interchangeable" | ❌ `[FunctionName]` = **In-Process** model. `[Function]` = **Isolated Worker** model. Using the wrong one fails silently or throws errors |
| "In-Process and Isolated Worker use the same DI setup" | ❌ In-Process uses `FunctionsStartup` + `Startup.cs`. Isolated uses `HostBuilder` in `Program.cs` with `.ConfigureFunctionsWorkerDefaults()` |
| "Middleware is supported in both .NET models" | ❌ Middleware (`IFunctionsWorkerMiddleware`) is **only** available in the **Isolated Worker** model |
| "HTTP trigger with `AuthorizationLevel.Function` requires no key" | ❌ `Function` level **requires** a function key or host key in the request (`?code=` or `x-functions-key` header). Only `Anonymous` requires no key |
| "The master key is just another host key" | ❌ The master key (`_master`) grants **admin-level** access including runtime APIs. It cannot be revoked, only renewed. Treat it like a root password |
| "Timer trigger CRON `0 */5 * * * *` runs every 5 hours" | ❌ Azure Functions uses **6-field CRON** (includes seconds). `0 */5 * * * *` = every **5 minutes**. `0 0 */5 * * *` = every 5 hours |
| "Timer trigger with `RunOnStartup = true` is fine in production" | ❌ `RunOnStartup` causes execution on **every scale-out instance** and on **every restart/deployment**. Not recommended for production |
| "Blob trigger detects changes instantly" | ❌ Default blob trigger uses **polling** with up to **10-minute delay**. For near-instant, use **Event Grid trigger** with blob events |
| "Queue trigger `maxDequeueCount` defaults to 1" | ❌ Default `maxDequeueCount` is **5**. After 5 failures, the message moves to the **poison queue** (`<queuename>-poison`) |
| "Cosmos DB trigger detects deletes" | ❌ The Cosmos DB change feed only captures **inserts and updates**. Deletes are NOT detected unless you implement **soft-delete** pattern |
| "Cosmos DB trigger uses the main container for tracking" | ❌ It requires a separate **leases container** (default name: `leases`) to track the change feed position |
| "`WEBSITE_RUN_FROM_PACKAGE = 1` deploys to wwwroot normally" | ❌ `= 1` makes the deployment **read-only** (mounted from ZIP). The app cannot write to its own filesystem. Use `= 0` for normal mutable deployment |
| "`WEBSITE_RUN_FROM_PACKAGE` URL means the app downloads once" | ❌ With an external URL, the app mounts the package **directly from blob storage** at runtime. If the blob is deleted, the app breaks |
| "Slot swap moves app settings too" | ❌ By default, settings swap WITH the app. To keep settings **sticky** to a slot, mark them with `slotSetting = true` (or use `WEBSITE_OVERRIDE_STICKY_DIAGNOSTICS_SETTINGS`) |
| "Connection strings use `ConnectionStringName` in binding" | ❌ For identity-based connections, settings use **double underscore** prefix: `MyConn__serviceUri`, `MyConn__fullyQualifiedNamespace`, etc. |
| "Managed Identity is enabled by default" | ❌ You must **explicitly enable** Managed Identity (system-assigned or user-assigned) on the Function App, then assign the appropriate **RBAC roles** |
| "`function.json` is required for all languages" | ❌ **C# (compiled)** and **Java** use **attributes/annotations** in code — no `function.json` needed (it's auto-generated at build). Script languages (JS, Python, PowerShell) use `function.json` |
| "Extension bundles are always required" | ❌ Extension bundles are required for **non-.NET** languages (JS, Python, PowerShell, Java, Custom). .NET projects use NuGet packages instead |
| "`host.json` settings apply per function" | ❌ `host.json` is **app-level** configuration. It applies to ALL functions in the Function App. Per-function config goes in `function.json` or attributes |
| "Cold start only affects Consumption plan" | ❌ Premium plan eliminates cold starts with **pre-warmed instances**, but the **first request** after a long idle on Premium can still see slight delay. Dedicated plan has no cold start if always running |
| "Premium plan `minimumElasticInstanceCount` avoids all cold starts" | ⚠️ Only if set to ≥ 1. Default is 0 for Premium, which means cold starts CAN still happen. Set it to at least 1 for always-warm |
| "Azure Functions scale infinitely on Consumption" | ❌ Consumption plan has a **default limit of 200 instances**. Premium plan default is **100 instances**. These can be adjusted but are not infinite |
| "`ServiceBusTrigger` auto-completes messages always" | ❌ `autoComplete` defaults to `true`, but setting it to `false` means YOU must call `CompleteAsync()`, `AbandonAsync()`, or `DeadLetterAsync()` manually |
| "Retry policy works on all trigger types" | ❌ Retry policies are **NOT supported** on certain triggers like **Timer trigger** and **Kafka trigger**. They work on HTTP, Queue, Service Bus, Event Hub, Cosmos DB, Blob |
| "Fixed delay and exponential backoff retry are the same" | ❌ Fixed delay = same wait between each retry. Exponential backoff = wait **doubles** each time (e.g., 1s → 2s → 4s → 8s) with optional `maximumInterval` |
| "`local.settings.json` is deployed to Azure" | ❌ `local.settings.json` is for **local development only**. It is in `.gitignore` by default. Use **Application Settings** in Azure portal or CLI for production |
| "Functions in the same Function App can use different languages" | ❌ All functions in a Function App must use the **same language/runtime**. To use multiple languages, create separate Function Apps |
| "Durable Functions `WaitForExternalEvent` has no timeout" | ⚠️ It can wait **indefinitely** by default, which can cause orphaned orchestrations. Always use the overload with a `TimeSpan` timeout |
| "Durable entity state persists after orchestration completes" | ✅ True — Entity state is **durable** and persists independently. But entities are deleted if you call `Entity.Current.DeleteState()` |
| "Fan-out/fan-in uses `Task.WhenAll` directly" | ❌ You must use `Task.WhenAll()` with tasks returned by `context.CallActivityAsync()`. Don't mix in non-deterministic tasks or raw `Task.Run()` |
| "Function Proxies is the recommended API gateway" | ❌ Function Proxies is **deprecated**. Microsoft recommends using **Azure API Management (APIM)** instead |
| "Custom Handlers run inside the Functions runtime" | ❌ Custom Handlers run as a **separate process**. The Functions host forwards HTTP requests to your process on `FUNCTIONS_CUSTOMHANDLER_PORT` |
| "`az functionapp create` with `--consumption-plan-location` creates a named plan" | ❌ It creates a **serverless Consumption plan** automatically. You don't specify a plan name — Azure manages it |
| "You need to restart the app for Application Settings changes" | ⚠️ Azure Functions **automatically restarts** when you change Application Settings. This is by design but can cause brief downtime |
| "Event Grid trigger and Event Grid subscription are the same" | ❌ **Event Grid trigger** = function triggered by Event Grid events. **Event Grid subscription on blob** = using Event Grid to trigger the **Blob trigger** faster (replacing polling) |
| "Durable Functions work on Consumption plan with no limits" | ⚠️ Durable Functions work on Consumption but are subject to the **10-minute timeout** for activity functions. Long orchestrations work (they sleep), but individual activities must complete within the timeout |

---

## Study Tips for AZ-204 Exam

1. **Understand hosting plans deeply** - Know features, limits, and when to use each
2. **Practice with Core Tools** - Be comfortable creating and deploying functions locally
3. **Master Durable Functions** - Understand all 4 types, common patterns, and advanced topics (eternal, singleton, versioning)
4. **Know triggers and bindings** - Understand configuration, host.json settings, and trigger-specific behaviors
5. **Deterministic orchestrators** - Critical concept for Durable Functions (what IS and ISN'T allowed)
6. **Hands-on practice** - Create functions with different triggers and bindings
7. **Understand scaling** - How each plan scales, limits, and behaviors
8. **Configuration management** - host.json, app settings, Key Vault references, sticky settings
9. **Performance optimization** - Cold starts, connection management, best practices
10. **Monitoring and logging** - Application Insights integration, log levels, KQL queries
11. **Know .NET models** - In-process vs Isolated Worker model differences, Program.cs vs Startup.cs
12. **Authorization & keys** - Authorization levels (Anonymous/Function/Admin), key types, when to use each
13. **Identity-based connections** - Managed identity, DefaultAzureCredential, RBAC roles for each service
14. **Deployment strategies** - Run-from-package, deployment slots, sticky settings, traffic routing
15. **Networking** - VNet integration plan support, IP restrictions, private endpoints, CORS
16. **Poison messages & retry** - Queue poison handling, retry strategies, maxDequeueCount
17. **Custom handlers** - When and how to use them, FUNCTIONS_CUSTOMHANDLER_PORT
18. **HTTP trigger details** - Route templates, route constraints, request/response handling
19. **Middleware** - Only available in isolated worker model, IFunctionsWorkerMiddleware
20. **Extension bundles** - Required for non-.NET languages, version range syntax

---

## Quick Reference Summary

### Hosting Plans Comparison

| Feature | Consumption | Flex Consumption | Premium | Dedicated |
|---------|-------------|------------------|---------|-----------|
| **Pricing** | Per execution | Per execution | Always-on | Monthly plan |
| **Scale to Zero** | Yes | Yes | No (min 1) | No |
| **Cold Starts** | Yes | Possible | No | No |
| **Max Timeout** | 10 min | 30 min | Unbounded | Unbounded |
| **VNet** | No | Yes | Yes | Yes |
| **Free Grant** | Yes | Yes | No | No |
| **Max Instances** | 200 | Configurable | 20-100 | Plan limit |

### Authorization Levels

| Level | Key Required | Use Case |
|-------|-------------|----------|
| Anonymous | None | Public APIs, behind APIM |
| Function | Function key or Host key | Default, service-to-service |
| Admin | Master key only | Admin operations |

### Key Types

| Key | Scope | Notes |
|-----|-------|-------|
| Function key | Single function | Default key named `default` |
| Host key | All functions in app | Named keys |
| Master key (_master) | All + admin APIs | Cannot be revoked, only renewed |
| System key | Extension-specific | e.g., Event Grid, Durable Tasks |

### .NET Models Comparison

| Feature | In-Process | Isolated Worker |
|---------|-----------|----------------|
| Attribute | `[FunctionName]` | `[Function]` |
| HTTP Types | HttpRequest/IActionResult | HttpRequestData/HttpResponseData |
| DI Config | Startup : FunctionsStartup | Program.cs HostBuilder |
| Middleware | Not supported | Supported |
| Worker Runtime | `dotnet` | `dotnet-isolated` |
| Entry Point | Function methods | Program.cs (Exe) |

### Trigger Types

- **HTTP**: REST APIs, webhooks (auth levels: Anonymous/Function/Admin)
- **Timer**: Scheduled tasks (CRON: `{sec} {min} {hr} {day} {mon} {dow}`)
- **Blob**: File processing (polling-based or Event Grid-based)
- **Queue**: Message processing (poison queue: `{name}-poison`)
- **Service Bus**: Enterprise messaging (sessions, dead-letter, topics)
- **Event Hub**: Stream processing (partition-based scaling)
- **Cosmos DB**: Change feed (inserts + updates only, needs lease container)
- **Event Grid**: Event-driven (push-based, near real-time)

### Key host.json Settings

```json
{
  "extensions": {
    "http": { "routePrefix": "api", "maxConcurrentRequests": 100 },
    "queues": { "batchSize": 16, "maxDequeueCount": 5 },
    "serviceBus": { "messageHandlerOptions": { "maxConcurrentCalls": 16, "autoComplete": true } },
    "eventHubs": { "maxEventBatchSize": 100 },
    "blobs": { "source": "EventGrid" },
    "durableTask": { "hubName": "MyTaskHub" }
  },
  "functionTimeout": "00:05:00",
  "extensionBundle": { "id": "Microsoft.Azure.Functions.ExtensionBundle", "version": "[4.*, 5.0.0)" }
}
```

### Identity-Based Connection Suffixes

| Service | Setting Suffix | Example |
|---------|---------------|---------|
| Storage | `__accountName` | `AzureWebJobsStorage__accountName` |
| Service Bus | `__fullyQualifiedNamespace` | `ServiceBusConn__fullyQualifiedNamespace` |
| Event Hub | `__fullyQualifiedNamespace` | `EventHubConn__fullyQualifiedNamespace` |
| Cosmos DB | `__accountEndpoint` | `CosmosDBConn__accountEndpoint` |

### Durable Functions

**Four Types:**
1. **Orchestrator**: Workflow coordination (must be deterministic)
2. **Activity**: Actual work (no restrictions)
3. **Entity**: Stateful entities (class-based or function-based)
4. **Client**: Start/manage orchestrations

**Key Patterns:**
1. Function chaining (sequential)
2. Fan-out/fan-in (parallel with Task.WhenAll)
3. Async HTTP APIs (202 Accepted + polling)
4. Monitor (polling with timers)
5. Human interaction (external events + timeout)
6. Aggregator (stateful entities)

**Advanced Concepts:**
- Eternal orchestrations: `ContinueAsNew()` to prevent history growth
- Singleton: Fixed instance ID + status check before start
- Versioning: Side-by-side deployment or wait-for-completion
- HTTP Management APIs: System key required

### Best Practices

- Reuse HTTP clients (static instances)
- Use async/await properly (never .Result or .Wait())
- Minimize cold starts (Premium plan with pre-warmed instances)
- Configure appropriate log levels
- Use Application Insights with sampling
- Follow orchestrator constraints (deterministic only)
- Batch process when possible
- Choose right hosting plan for workload
- Use managed identity over connection strings
- Use identity-based connections
- Monitor poison queues for failures
- Use extension bundles for non-.NET languages
- Set route prefix in host.json (or remove with "")
- Use run-from-package for faster cold starts

### Retry Strategies

| Strategy | Behavior | Config |
|----------|----------|--------|
| Fixed Delay | Same delay between retries | `delayInterval` |
| Exponential Backoff | Increasing delay | `minimumInterval`, `maximumInterval` |
| Queue Built-in | Dequeue count + poison queue | `maxDequeueCount` |

### Key App Settings

| Setting | Value | Purpose |
|---------|-------|---------|
| `FUNCTIONS_EXTENSION_VERSION` | `~4` | Runtime version |
| `FUNCTIONS_WORKER_RUNTIME` | `dotnet-isolated`, `node`, `python`, etc. | Language |
| `AzureWebJobsStorage` | Connection string or identity | Required storage |
| `WEBSITE_RUN_FROM_PACKAGE` | `1` or URL | Package deployment |
| `WEBSITE_TIME_ZONE` | Timezone name | Timer trigger timezone |
| `AzureWebJobs.<NAME>.Disabled` | `true` | Disable function |

---

## Important URLs and References

- Azure Functions Documentation: https://learn.microsoft.com/en-us/azure/azure-functions/
- Durable Functions: https://learn.microsoft.com/en-us/azure/azure-functions/durable/
- Triggers and Bindings: https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings
- Best Practices: https://learn.microsoft.com/en-us/azure/azure-functions/functions-best-practices
- Pricing: https://azure.microsoft.com/en-us/pricing/details/functions/
- .NET Isolated Worker Guide: https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide
- Custom Handlers: https://learn.microsoft.com/en-us/azure/azure-functions/functions-custom-handlers
- Networking: https://learn.microsoft.com/en-us/azure/azure-functions/functions-networking-options
- Identity-Based Connections: https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference#configure-an-identity-based-connection
- AZ-204 Official Study Guide: https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-204

---

*Good luck with your AZ-204 exam preparation!*
