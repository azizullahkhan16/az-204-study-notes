# AZ-204: Troubleshoot Solutions Using Application Insights - Complete Study Notes

## Learning Path Overview
This learning path covers instrumenting solutions for monitoring and logging with 1 module:
1. Monitor app performance

**Level:** Intermediate | **Product:** Azure Monitor | **Role:** Developer

---

## MODULE 1: MONITOR APP PERFORMANCE

### 1.1 Introduction to Azure Monitor

**What is Azure Monitor?**
- Comprehensive monitoring solution for Azure
- Collects, analyzes, and acts on telemetry
- Monitors applications, infrastructure, and networks
- Includes Application Insights for application monitoring

**Azure Monitor Components:**

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           Azure Monitor                                   │
├───────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐         │
│  │   Data Sources  │   │   Data Platform │   │   Consumption   │         │
│  ├─────────────────┤   ├─────────────────┤   ├─────────────────┤         │
│  │ • Applications  │   │ • Metrics       │   │ • Insights      │         │
│  │ • OS Guest      │──▶│ • Logs          │──▶│ • Visualize     │         │
│  │ • Azure Resources│  │                 │   │ • Analyze       │         │
│  │ • Subscriptions │   │                 │   │ • Respond       │         │
│  │ • Tenant        │   │                 │   │ • Integrate     │         │
│  │ • Custom        │   │                 │   │                 │         │
│  └─────────────────┘   └─────────────────┘   └─────────────────┘         │
│                                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

**Data Types:**
- **Metrics**: Numerical values at regular intervals (time-series)
- **Logs**: Structured/unstructured data stored in Log Analytics

### 1.2 Introduction to Application Insights

**What is Application Insights?**
- Application Performance Management (APM) service
- Part of Azure Monitor
- Monitors live web applications
- Automatically detects performance anomalies
- Includes analytics tools for diagnostics

**Key Capabilities:**
- Request monitoring and failure diagnosis
- Dependency tracking
- Exception tracking
- Distributed tracing
- Performance counters
- Custom telemetry
- Usage analytics
- Availability monitoring

**Supported Platforms:**
- .NET / .NET Core / .NET Framework
- Java
- Node.js
- Python
- JavaScript (browser)
- Mobile apps (via SDKs)

**Use Cases:**
- Monitor request rates, response times, failures
- Track dependency calls (databases, REST APIs)
- Detect and diagnose exceptions
- Analyze user behavior and page views
- Set up alerts on metrics
- Troubleshoot performance issues

### 1.3 Application Insights Architecture

**Data Flow:**

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│   Application   │     │  Application Insights │     │    Azure        │
│   (with SDK)    │────▶│     Endpoint         │────▶│    Storage      │
└─────────────────┘     └──────────────────────┘     └────────┬────────┘
                                                              │
                        ┌──────────────────────┐              │
                        │   Log Analytics      │◀─────────────┘
                        │   Workspace          │
                        └──────────┬───────────┘
                                   │
          ┌────────────────────────┼────────────────────────┐
          │                        │                        │
          ▼                        ▼                        ▼
   ┌──────────────┐      ┌──────────────┐        ┌──────────────┐
   │   Alerts     │      │  Dashboards  │        │   Workbooks  │
   └──────────────┘      └──────────────┘        └──────────────┘
```

**Resource Types:**
- **Workspace-based** (recommended): Data stored in Log Analytics workspace
- **Classic**: Standalone data storage (deprecated for new resources)

### 1.4 Creating Application Insights

**Azure CLI:**

```bash
# Create Application Insights resource
az monitor app-insights component create \
  --app myappinsights \
  --resource-group myRG \
  --location eastus \
  --workspace /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.OperationalInsights/workspaces/{workspace}

# Get instrumentation key
az monitor app-insights component show \
  --app myappinsights \
  --resource-group myRG \
  --query instrumentationKey

# Get connection string
az monitor app-insights component show \
  --app myappinsights \
  --resource-group myRG \
  --query connectionString
```

**Connection String vs Instrumentation Key:**
- **Connection String** (recommended): Contains endpoint and key
- **Instrumentation Key** (legacy): Just the key, uses default endpoint

```
Connection String Format:
InstrumentationKey=xxx;IngestionEndpoint=https://eastus-0.in.applicationinsights.azure.com/;LiveEndpoint=https://eastus.livediagnostics.monitor.azure.com/
```

### 1.5 Telemetry Types

**Built-in Telemetry:**

| Type | Description | Auto-collected |
|------|-------------|----------------|
| **Request** | Incoming HTTP requests | Yes |
| **Dependency** | Calls to external services | Yes |
| **Exception** | Runtime exceptions | Yes |
| **Trace** | Diagnostic logs | Partial |
| **Event** | Custom business events | No |
| **Metric** | Performance measurements | Partial |
| **PageView** | Browser page loads | JS SDK |
| **Availability** | Availability test results | Yes |

**Request Telemetry:**
- Name, URL, duration
- Response code, success
- Custom properties
- User/session context

**Dependency Telemetry:**
- HTTP calls, SQL queries
- Azure service calls
- Duration, success
- Target, type, data

**Exception Telemetry:**
- Exception type, message
- Stack trace
- Associated request
- Custom properties

**Trace Telemetry:**
- Log messages
- Severity level
- Custom properties

### 1.6 Instrumentation Methods

**Two Approaches:**

#### 1. Auto-Instrumentation (Agent-based)
- No code changes required
- Limited to supported scenarios
- Azure App Service: Enable in portal
- Azure VMs: Install agent
- Kubernetes: Use operator

**Enable for App Service:**
```bash
# Enable Application Insights for App Service
az webapp config appsettings set \
  --name mywebapp \
  --resource-group myRG \
  --settings APPLICATIONINSIGHTS_CONNECTION_STRING="InstrumentationKey=xxx;..."

# Or use Azure Portal:
# App Service → Settings → Application Insights → Enable
```

#### 2. SDK-based Instrumentation
- Add NuGet package
- Full control over telemetry
- Custom events and metrics
- Required for custom telemetry

### 1.7 Application Insights SDK for .NET

**Installation:**
```bash
dotnet add package Microsoft.ApplicationInsights.AspNetCore
```

**Configuration - Program.cs:**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add Application Insights
builder.Services.AddApplicationInsightsTelemetry();

var app = builder.Build();
app.Run();
```

**Configuration - appsettings.json:**

```json
{
  "ApplicationInsights": {
    "ConnectionString": "InstrumentationKey=xxx;IngestionEndpoint=https://..."
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    },
    "ApplicationInsights": {
      "LogLevel": {
        "Default": "Information"
      }
    }
  }
}
```

**Environment Variable:**
```bash
APPLICATIONINSIGHTS_CONNECTION_STRING="InstrumentationKey=xxx;..."
```

**Telemetry Configuration:**

```csharp
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = "InstrumentationKey=xxx;...";
    options.EnableAdaptiveSampling = true;
    options.EnableQuickPulseMetricStream = true;  // Live Metrics
    options.EnableHeartbeat = true;
    options.EnableDependencyTrackingTelemetryModule = true;
    options.EnableRequestTrackingTelemetryModule = true;
});
```

### 1.8 Custom Telemetry

**TelemetryClient:**

```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;

public class OrderService
{
    private readonly TelemetryClient _telemetryClient;

    public OrderService(TelemetryClient telemetryClient)
    {
        _telemetryClient = telemetryClient;
    }

    public void ProcessOrder(Order order)
    {
        // Track custom event
        _telemetryClient.TrackEvent("OrderProcessed", new Dictionary<string, string>
        {
            { "OrderId", order.Id.ToString() },
            { "CustomerId", order.CustomerId },
            { "ItemCount", order.Items.Count.ToString() }
        });

        // Track custom metric
        _telemetryClient.TrackMetric("OrderValue", order.TotalValue);

        // Track trace (log)
        _telemetryClient.TrackTrace($"Processing order {order.Id}", SeverityLevel.Information);
    }
}
```

**Tracking Methods:**

```csharp
// Track custom event
_telemetryClient.TrackEvent("ButtonClicked", 
    properties: new Dictionary<string, string> { { "ButtonName", "Submit" } },
    metrics: new Dictionary<string, double> { { "ClickCount", 1 } });

// Track metric
_telemetryClient.TrackMetric("QueueLength", 42);
_telemetryClient.TrackMetric(new MetricTelemetry("ResponseTime", 150));

// Track exception
try
{
    // Code that might throw
}
catch (Exception ex)
{
    _telemetryClient.TrackException(ex, new Dictionary<string, string>
    {
        { "Operation", "ProcessOrder" },
        { "OrderId", orderId }
    });
    throw;
}

// Track dependency
var startTime = DateTime.UtcNow;
var timer = Stopwatch.StartNew();
try
{
    // Call external service
    var result = await httpClient.GetAsync("https://api.example.com/data");
    timer.Stop();
    
    _telemetryClient.TrackDependency(
        dependencyTypeName: "HTTP",
        dependencyName: "api.example.com",
        data: "GET /data",
        startTime: startTime,
        duration: timer.Elapsed,
        success: result.IsSuccessStatusCode);
}
catch (Exception ex)
{
    timer.Stop();
    _telemetryClient.TrackDependency("HTTP", "api.example.com", "GET /data", 
        startTime, timer.Elapsed, false);
    throw;
}

// Track request (usually automatic)
_telemetryClient.TrackRequest(new RequestTelemetry
{
    Name = "ProcessOrder",
    Url = new Uri("https://myapp.com/api/orders"),
    Duration = TimeSpan.FromMilliseconds(250),
    Success = true,
    ResponseCode = "200"
});

// Track page view (browser)
_telemetryClient.TrackPageView("HomePage");

// Track availability
_telemetryClient.TrackAvailability(new AvailabilityTelemetry
{
    Name = "API Health Check",
    RunLocation = "East US",
    Success = true,
    Duration = TimeSpan.FromMilliseconds(100)
});

// Flush telemetry (ensure delivery before shutdown)
_telemetryClient.Flush();
await Task.Delay(5000);  // Wait for flush
```

### 1.9 Custom Properties and Context

**Adding Custom Properties:**

```csharp
// Global properties (all telemetry)
_telemetryClient.Context.GlobalProperties["Environment"] = "Production";
_telemetryClient.Context.GlobalProperties["Version"] = "1.0.0";

// User context
_telemetryClient.Context.User.Id = userId;
_telemetryClient.Context.User.AuthenticatedUserId = authenticatedUserId;

// Session context
_telemetryClient.Context.Session.Id = sessionId;

// Operation context (for correlation)
_telemetryClient.Context.Operation.Id = correlationId;
_telemetryClient.Context.Operation.ParentId = parentId;
_telemetryClient.Context.Operation.Name = "ProcessOrder";
```

**Telemetry Initializer (Add Properties Globally):**

```csharp
public class CustomTelemetryInitializer : ITelemetryInitializer
{
    private readonly IHttpContextAccessor _httpContextAccessor;

    public CustomTelemetryInitializer(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    public void Initialize(ITelemetry telemetry)
    {
        // Add custom properties to all telemetry
        if (telemetry is ISupportProperties propTelemetry)
        {
            propTelemetry.Properties["MachineName"] = Environment.MachineName;
            propTelemetry.Properties["Environment"] = 
                Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");
        }

        // Add user info
        var user = _httpContextAccessor.HttpContext?.User;
        if (user?.Identity?.IsAuthenticated == true)
        {
            telemetry.Context.User.AuthenticatedUserId = 
                user.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        }
    }
}

// Register in Program.cs
builder.Services.AddSingleton<ITelemetryInitializer, CustomTelemetryInitializer>();
```

### 1.10 Telemetry Processor and Filtering

**Telemetry Processor:**

```csharp
public class FilteringTelemetryProcessor : ITelemetryProcessor
{
    private readonly ITelemetryProcessor _next;

    public FilteringTelemetryProcessor(ITelemetryProcessor next)
    {
        _next = next;
    }

    public void Process(ITelemetry item)
    {
        // Filter out health check requests
        if (item is RequestTelemetry request)
        {
            if (request.Url?.AbsolutePath == "/health")
            {
                return;  // Don't send this telemetry
            }
        }

        // Filter successful dependencies (only track failures)
        if (item is DependencyTelemetry dependency)
        {
            if (dependency.Success == true && dependency.Duration < TimeSpan.FromSeconds(1))
            {
                return;  // Don't send fast successful dependencies
            }
        }

        // Pass to next processor
        _next.Process(item);
    }
}

// Register in Program.cs
builder.Services.AddApplicationInsightsTelemetryProcessor<FilteringTelemetryProcessor>();
```

### 1.11 Sampling

**What is Sampling?**
- Reduces telemetry volume
- Lowers costs
- Maintains statistical accuracy
- Keeps related items together

**Sampling Types:**

#### 1. Adaptive Sampling (Default)
- Automatically adjusts rate
- Targets specific volume
- Enabled by default in SDK

```csharp
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.EnableAdaptiveSampling = true;
});

// Or configure in appsettings.json
{
  "ApplicationInsights": {
    "SamplingSettings": {
      "IsEnabled": true,
      "MaxTelemetryItemsPerSecond": 5
    }
  }
}
```

#### 2. Fixed-Rate Sampling
- Constant sampling percentage
- Predictable volume reduction

```csharp
builder.Services.Configure<TelemetryConfiguration>(config =>
{
    var builder = config.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    builder.UseSampling(50);  // 50% sampling
    builder.Build();
});
```

#### 3. Ingestion Sampling
- Server-side sampling
- Configured in Azure portal
- Applied after data received

**Excluded Telemetry Types:**
```csharp
builder.Services.Configure<TelemetryConfiguration>(config =>
{
    var builder = config.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    builder.UseAdaptiveSampling(excludedTypes: "Exception;Trace");
    builder.Build();
});
```

### 1.12 Application Map

**What is Application Map?**
- Visual representation of application topology
- Shows components and dependencies
- Displays health and performance
- Helps identify bottlenecks

**Components Shown:**
- Application instances
- External dependencies
- Databases
- HTTP endpoints
- Queue services

**Cloud Role Name:**
```csharp
// Set cloud role name for identification
builder.Services.AddApplicationInsightsTelemetry();
builder.Services.Configure<TelemetryConfiguration>(config =>
{
    config.TelemetryInitializers.Add(new CloudRoleNameInitializer("OrderService"));
});

public class CloudRoleNameInitializer : ITelemetryInitializer
{
    private readonly string _roleName;

    public CloudRoleNameInitializer(string roleName)
    {
        _roleName = roleName;
    }

    public void Initialize(ITelemetry telemetry)
    {
        telemetry.Context.Cloud.RoleName = _roleName;
    }
}
```

### 1.13 Live Metrics (Live Metrics Stream)

**What is Live Metrics?**
- Real-time telemetry view
- 1-second latency
- No sampling applied
- Free (no additional cost)

**Features:**
- Live request rate
- Live failure rate
- Live dependency calls
- Server health metrics
- Incoming requests stream
- Quick filtering

**Enable Live Metrics:**
```csharp
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.EnableQuickPulseMetricStream = true;  // Enabled by default
});
```

### 1.14 Availability Tests

**Types of Availability Tests:**

#### 1. URL Ping Test (Standard Test)
- Simple HTTP GET request
- Check response code
- Check response content
- SSL certificate validation
- From multiple locations

```bash
# Create availability test (portal is recommended)
# Settings:
# - URL to test
# - Test frequency (5, 10, 15 minutes)
# - Test locations
# - Success criteria
# - Alerts
```

#### 2. Multi-Step Web Test
- Record of user actions
- Visual Studio Web Test
- Complex scenarios
- Being deprecated (use TrackAvailability)

#### 3. Custom Track Availability
- Programmatic availability testing
- Custom test logic
- Use TrackAvailability API

```csharp
public class CustomAvailabilityTest
{
    private readonly TelemetryClient _telemetryClient;
    private readonly HttpClient _httpClient;

    public async Task RunTestAsync()
    {
        var testName = "API Availability";
        var testLocation = Environment.GetEnvironmentVariable("REGION") ?? "Unknown";
        var startTime = DateTime.UtcNow;
        var timer = Stopwatch.StartNew();
        var success = false;
        var message = "";

        try
        {
            var response = await _httpClient.GetAsync("https://myapi.com/health");
            success = response.IsSuccessStatusCode;
            message = $"Response: {response.StatusCode}";
        }
        catch (Exception ex)
        {
            message = ex.Message;
        }
        finally
        {
            timer.Stop();
            _telemetryClient.TrackAvailability(
                name: testName,
                timeStamp: startTime,
                duration: timer.Elapsed,
                runLocation: testLocation,
                success: success,
                message: message);
        }
    }
}
```

**Availability Test Locations:**
- Multiple Azure regions worldwide
- Select based on user geography
- Minimum 5 locations recommended

### 1.15 Log Analytics and Kusto Queries

**What is Log Analytics?**
- Query engine for Application Insights data
- Uses Kusto Query Language (KQL)
- Analyze and visualize telemetry

**Common Tables:**

| Table | Description |
|-------|-------------|
| requests | HTTP requests |
| dependencies | External calls |
| exceptions | Errors and exceptions |
| traces | Log messages |
| customEvents | Custom events |
| customMetrics | Custom metrics |
| pageViews | Browser page views |
| availabilityResults | Availability tests |
| performanceCounters | System metrics |

**Basic Kusto Queries:**

```kusto
// Recent requests
requests
| where timestamp > ago(1h)
| limit 100

// Failed requests
requests
| where success == false
| where timestamp > ago(24h)
| summarize count() by name, resultCode
| order by count_ desc

// Slowest requests
requests
| where timestamp > ago(1h)
| summarize avg(duration), percentile(duration, 95), percentile(duration, 99) by name
| order by avg_duration desc

// Request rate over time
requests
| where timestamp > ago(1h)
| summarize requestCount = count() by bin(timestamp, 5m)
| render timechart

// Failed dependencies
dependencies
| where success == false
| where timestamp > ago(24h)
| summarize count() by name, type, target
| order by count_ desc

// Exceptions by type
exceptions
| where timestamp > ago(24h)
| summarize count() by type, outerMessage
| order by count_ desc

// Exception details
exceptions
| where timestamp > ago(1h)
| project timestamp, type, outerMessage, innermostMessage, details
| limit 50

// Custom events
customEvents
| where name == "OrderProcessed"
| where timestamp > ago(24h)
| extend orderId = tostring(customDimensions.OrderId)
| extend customerId = tostring(customDimensions.CustomerId)
| summarize count() by customerId
| order by count_ desc

// Trace logs
traces
| where severityLevel >= 3  // Warning and above
| where timestamp > ago(1h)
| project timestamp, message, severityLevel, customDimensions
| order by timestamp desc

// End-to-end transaction (distributed tracing)
union requests, dependencies, exceptions
| where operation_Id == "specific-operation-id"
| order by timestamp asc

// Performance percentiles
requests
| where timestamp > ago(24h)
| summarize 
    p50 = percentile(duration, 50),
    p90 = percentile(duration, 90),
    p95 = percentile(duration, 95),
    p99 = percentile(duration, 99)
  by name
| order by p95 desc
```

**Advanced Queries:**

```kusto
// Response time trend
requests
| where timestamp > ago(7d)
| summarize avg(duration) by bin(timestamp, 1h)
| render timechart

// Failure rate by hour
requests
| where timestamp > ago(24h)
| summarize 
    totalCount = count(),
    failedCount = countif(success == false)
  by bin(timestamp, 1h)
| extend failureRate = (failedCount * 100.0) / totalCount
| project timestamp, failureRate
| render timechart

// Dependency performance
dependencies
| where timestamp > ago(24h)
| summarize 
    avgDuration = avg(duration),
    callCount = count(),
    failureRate = countif(success == false) * 100.0 / count()
  by name, type
| order by avgDuration desc

// User sessions
requests
| where timestamp > ago(24h)
| summarize requestCount = count(), 
            uniqueUsers = dcount(user_Id),
            uniqueSessions = dcount(session_Id)

// Correlate requests and exceptions
requests
| where success == false
| join kind=inner exceptions on operation_Id
| project timestamp, requestName = name, exceptionType = type1, exceptionMessage = outerMessage
| limit 100
```

### 1.16 Alerts

**Alert Types:**

#### 1. Metric Alerts
- Based on metric thresholds
- Near real-time (1 minute)
- Static or dynamic thresholds

```bash
# Create metric alert
az monitor metrics alert create \
  --name "High Response Time" \
  --resource-group myRG \
  --scopes "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Insights/components/{app}" \
  --condition "avg requests/duration > 5000" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Insights/actionGroups/{ag}"
```

#### 2. Log Alerts
- Based on Kusto query results
- Custom conditions
- Scheduled evaluation

```bash
# Example log alert query
requests
| where success == false
| where timestamp > ago(5m)
| summarize failedCount = count()
| where failedCount > 10
```

#### 3. Smart Detection
- Automatic anomaly detection
- No configuration needed
- Alerts on:
  - Failure anomalies
  - Performance degradation
  - Memory leak
  - Exception volume increase

### 1.17 Distributed Tracing

**What is Distributed Tracing?**
- Track requests across services
- Correlate telemetry
- Visualize call chains
- Identify bottlenecks

**Correlation IDs:**
- **Operation ID**: Unique ID for entire transaction
- **Parent ID**: ID of calling operation
- **Request ID**: Unique ID for specific request

**Automatic Correlation:**
- W3C Trace Context (default)
- Request-Id header (legacy)

```csharp
// Headers automatically propagated:
// traceparent: 00-<trace-id>-<parent-id>-<trace-flags>
// tracestate: <vendor-specific>
```

**View in Portal:**
1. Select a request in Application Insights
2. Click "View in Transaction Diagnostics"
3. See full call chain with timing

**Kusto Query for Tracing:**

```kusto
// All telemetry for an operation
union requests, dependencies, exceptions, traces
| where operation_Id == "abc123"
| project timestamp, itemType, name, duration, success
| order by timestamp asc
```

### 1.18 Workbooks

**What are Workbooks?**
- Interactive reports
- Combine queries, visualizations, text
- Shareable dashboards
- Template gallery available

**Common Workbook Templates:**
- Failure Analysis
- Performance Analysis
- Usage Analysis
- Reliability (SLA)

### 1.19 Application Insights for JavaScript

**Browser SDK:**

```html
<!-- Add to <head> -->
<script type="text/javascript">
!function(T,l,y){/* Application Insights snippet */}
(window,document,{
  connectionString: "InstrumentationKey=xxx;..."
});
</script>
```

**npm Package:**

```bash
npm install @microsoft/applicationinsights-web
```

```javascript
import { ApplicationInsights } from '@microsoft/applicationinsights-web';

const appInsights = new ApplicationInsights({
  config: {
    connectionString: 'InstrumentationKey=xxx;...',
    enableAutoRouteTracking: true,  // SPA route tracking
    enableCorsCorrelation: true,    // Cross-origin correlation
    enableRequestHeaderTracking: true,
    enableResponseHeaderTracking: true
  }
});

appInsights.loadAppInsights();
appInsights.trackPageView();

// Custom tracking
appInsights.trackEvent({ name: 'ButtonClicked' });
appInsights.trackException({ exception: new Error('Test error') });
appInsights.trackTrace({ message: 'Trace message' });
appInsights.trackMetric({ name: 'CustomMetric', average: 42 });
```

### 1.20 Best Practices

**Implementation Best Practices:**

1. **Use Connection String**: Prefer connection string over instrumentation key
2. **Set Cloud Role Name**: Identify services in Application Map
3. **Add Custom Properties**: Include business context in telemetry
4. **Use Telemetry Initializers**: Add global properties
5. **Implement Sampling**: Control costs at scale
6. **Filter Health Checks**: Reduce noise from monitoring endpoints
7. **Flush on Shutdown**: Ensure telemetry delivery
8. **Enable Live Metrics**: For real-time monitoring
9. **Set Up Alerts**: Proactive issue detection
10. **Use Availability Tests**: Monitor uptime

**Cost Optimization:**
- Enable sampling (adaptive or fixed-rate)
- Filter unnecessary telemetry
- Use daily cap if needed
- Reduce retention period
- Ingestion sampling as last resort

---

## MODULE 2: SNAPSHOT DEBUGGER & PROFILER

### 2.1 Snapshot Debugger

**What is Snapshot Debugger?**
- Automatically collects debug snapshots when exceptions occur in production
- Allows viewing local variables and call stack at the point of the exception
- No impact on application performance (lightweight)
- Works with .NET and .NET Core applications
- Requires Application Insights SDK or auto-instrumentation

**When Snapshots Are Collected:**
- Only on **first-chance exceptions** that are tracked by Application Insights
- Applies a **cooldown period** (default: ~15 minutes between snapshots for the same exception)
- Has a **daily limit** per app (configurable)
- Snapshots are NOT collected for every single exception occurrence

**Enabling Snapshot Debugger:**

**Method 1: NuGet Package (SDK-based)**
```bash
dotnet add package Microsoft.ApplicationInsights.SnapshotCollector
```

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddApplicationInsightsTelemetry();
builder.Services.AddSnapshotCollector(config =>
{
    config.IsEnabledInDeveloperMode = false;   // Don't collect in dev
    config.ThresholdForSnapshotting = 1;        // Snapshots after N exceptions
    config.MaximumSnapshotsRequired = 3;        // Max snapshots per problem
    config.MaximumCollectionPlanSize = 50;       // Max tracked problems
    config.SnapshotInLowPriorityThread = true;  // Don't block main thread
    config.ProvideAnonymousTelemetry = true;
    config.ReconnectInterval = TimeSpan.FromMinutes(15);
});

var app = builder.Build();
app.Run();
```

**Method 2: Azure Portal (No code changes)**
- Navigate to Application Insights → Settings → Snapshot Debugger → Enable
- Works with App Service auto-instrumentation
- No NuGet package needed

**Method 3: ARM Template / App Settings**
```json
{
  "APPINSIGHTS_SNAPSHOTFEATURE_VERSION": "1.0.0",
  "DiagnosticServices_EXTENSION_VERSION": "~3",
  "SnapshotDebugger_EXTENSION_VERSION": "disabled"  // or "~1" to enable
}
```

**Viewing Snapshots:**
1. Application Insights → Failures → Select an exception
2. Click "Open debug snapshot" on the right panel
3. View call stack, local variables, and parameters
4. Optionally download `.diagsession` file for Visual Studio

**Key Exam Points:**
- Snapshot Debugger is for **production debugging** — no need to reproduce bugs
- It captures snapshots of **local variables** at exception time
- Works with both App Service and VMs
- Requires `Microsoft.ApplicationInsights.SnapshotCollector` NuGet package for SDK-based
- Supported: .NET Framework 4.6.2+, .NET Core 2.0+, .NET 6+
- The snapshot is a **minidump** — not a full memory dump

### 2.2 Application Insights Profiler

**What is Profiler?**
- Traces the execution path of live web requests
- Identifies **hot paths** — code that takes the most time
- Lightweight, designed for production use
- Helps find performance bottlenecks without reproducing issues
- Works with .NET / .NET Core applications on App Service

**Key Difference: Profiler vs Snapshot Debugger**

| Feature | Profiler | Snapshot Debugger |
|---------|----------|-------------------|
| **Purpose** | Performance analysis | Exception debugging |
| **Captures** | Execution traces (hot paths) | Local variables at exception |
| **Trigger** | Sampling-based / on-demand | Exception occurrence |
| **Output** | Flame graph / call tree | Debug snapshot (minidump) |
| **Use When** | "Why is this request slow?" | "Why did this crash?" |

**Enabling Profiler:**

**For App Service:**
```bash
# Enable via App Settings
az webapp config appsettings set \
  --name mywebapp \
  --resource-group myRG \
  --settings APPINSIGHTS_PROFILERFEATURE_VERSION="1.0.0" \
             DiagnosticServices_EXTENSION_VERSION="~3"
```

**Or via Portal:**
- Application Insights → Performance → Profiler → Configure
- Enable profiling and set triggers

**Profiler Triggers:**
- **Sampling trigger**: Profiles a percentage of requests automatically
- **Manual trigger**: Profile now for a set duration
- **CPU trigger**: Profile when CPU exceeds threshold
- **On-demand**: Start/stop profiling via portal

**Viewing Profiler Traces:**
1. Application Insights → Performance → Select a slow request
2. Click "Profiler traces" button
3. View flame graph showing time spent in each method
4. Identify hot path (longest execution path)

**Reading the Flame Graph:**
```
┌──────────────────────────────────────────────────────┐
│ GET /api/orders                                 250ms │
├──────────────────────────────────────────────────────┤
│ OrderController.Get()                           248ms │
├────────────────────────────┬─────────────────────────┤
│ OrderService.GetOrders()   │ OrderService.Validate() │
│ 200ms                      │ 45ms                    │
├────────────────────────────┤                         │
│ DbContext.QueryAsync()     │                         │
│ 195ms ← HOT PATH          │                         │
└────────────────────────────┴─────────────────────────┘
```

**Supported Platforms for Profiler:**
- Azure App Service (Windows and Linux)
- Azure Virtual Machines
- Azure Cloud Services
- Azure Service Fabric
- NOT supported: Azure Functions Consumption plan (Premium/Dedicated only)

### 2.3 Transaction Diagnostics View

**What is Transaction Diagnostics?**
- Unified view of an end-to-end transaction
- Shows all telemetry correlated by operation_Id
- Includes requests, dependencies, exceptions, traces
- Timeline view showing sequence and duration
- Part of the portal experience (no code needed)

**How to Access:**
1. Application Insights → select any telemetry item
2. Click "View in transaction diagnostics"
3. See the full call chain with timing

**What it Shows:**
```
[Request] GET /api/orders ─── 250ms (success)
  ├── [Dependency] SQL query ─── 195ms (success)
  ├── [Dependency] Redis cache ─── 5ms (success)
  ├── [Trace] "Fetching orders for user X"
  └── [Dependency] HTTP POST /api/notifications ─── 45ms (success)
        ├── [Trace] "Notification sent"
        └── [Exception] TimeoutException (handled)
```

---

## MODULE 3: PRE-AGGREGATED METRICS vs LOG-BASED METRICS

### 3.1 Two Types of Metrics

**This is a critical exam concept that trips up many candidates.**

| Aspect | Log-Based Metrics | Pre-Aggregated Metrics |
|--------|-------------------|----------------------|
| **Storage** | Stored as individual log events | Stored as pre-aggregated time series |
| **Query via** | Kusto (KQL) on `customMetrics` table | Metrics Explorer / Metrics API |
| **Affected by sampling** | ✅ Yes — sampling reduces data | ❌ No — aggregated before sampling |
| **Accuracy** | Lower at high volume (due to sampling) | Higher — exact counts |
| **Latency** | Higher (log query) | Lower (near real-time) |
| **Cost** | Higher (stores every event) | Lower (pre-aggregated) |
| **API** | `TrackMetric()` | `GetMetric().TrackValue()` |
| **Dimensions** | Unlimited custom properties | Up to 10 dimensions |

### 3.2 GetMetric() API (Pre-Aggregated — Recommended)

```csharp
public class OrderService
{
    private readonly TelemetryClient _telemetryClient;

    public OrderService(TelemetryClient telemetryClient)
    {
        _telemetryClient = telemetryClient;
    }

    public void ProcessOrder(Order order)
    {
        // Pre-aggregated metric — efficient, not affected by sampling
        Metric orderMetric = _telemetryClient.GetMetric("OrdersProcessed");
        orderMetric.TrackValue(1);

        // With dimensions (up to 10)
        Metric orderByRegion = _telemetryClient.GetMetric("OrderValue", "Region");
        orderByRegion.TrackValue(order.TotalValue, order.Region);

        // Multi-dimensional
        Metric orderDetailed = _telemetryClient.GetMetric(
            "OrderProcessingTime", "Region", "ProductCategory");
        orderDetailed.TrackValue(
            processingTime.TotalMilliseconds, order.Region, order.Category);
    }
}
```

**Key Points about GetMetric():**
- Pre-aggregates locally before sending to Azure
- Sends aggregated values (sum, count, min, max) every 60 seconds
- NOT affected by sampling — always accurate
- Maximum 10 dimensions per metric
- Much more efficient at high volume than `TrackMetric()`
- Available in SDK 2.7.2+

### 3.3 TrackMetric() (Log-Based — Legacy)

```csharp
// Log-based metric — stored as individual event, affected by sampling
_telemetryClient.TrackMetric("OrderValue", order.TotalValue);

// Equivalent to:
_telemetryClient.TrackMetric(new MetricTelemetry("OrderValue", order.TotalValue));
```

**When to Use Which:**

| Scenario | Use |
|----------|-----|
| High-frequency metrics (1000s/sec) | `GetMetric()` |
| Need exact counts | `GetMetric()` |
| Alerting on metrics | `GetMetric()` (not affected by sampling) |
| Need unlimited custom properties | `TrackMetric()` |
| Ad-hoc analysis with KQL | `TrackMetric()` |
| New code / greenfield | `GetMetric()` (recommended) |

### 3.4 Metrics Explorer

**What is Metrics Explorer?**
- Portal tool to visualize pre-aggregated metrics
- Real-time charting
- Supports splitting by dimensions
- Pin charts to dashboards

**Standard Metrics (Auto-collected):**
- `requests/count` — Number of requests
- `requests/duration` — Average response time
- `requests/failed` — Failed request count
- `dependencies/count` — Dependency call count
- `dependencies/duration` — Average dependency duration
- `dependencies/failed` — Failed dependency count
- `exceptions/count` — Exception count
- `performanceCounters/processCpuPercentage` — CPU %
- `performanceCounters/memoryAvailableBytes` — Available memory
- `performanceCounters/requestsPerSecond` — Request rate

**Custom Metrics Namespaces:**
- `azure.applicationinsights` — Standard metrics
- `customMetrics` — Your custom metrics (log-based)
- Custom namespace — Your pre-aggregated metrics

### 3.5 Telemetry Channels

**Two Channel Types:**

| Channel | Description | Use Case |
|---------|-------------|----------|
| `InMemoryChannel` | In-memory buffer, no retry | Local development, short-lived processes |
| `ServerTelemetryChannel` | Disk-backed, with retry | Production web apps (default for ASP.NET Core) |

```csharp
// ServerTelemetryChannel is default for web apps
// For non-web (console, worker) you may need to configure:
builder.Services.AddApplicationInsightsTelemetryWorkerService(options =>
{
    options.ConnectionString = "InstrumentationKey=xxx;...";
});

// Configure channel
builder.Services.Configure<TelemetryConfiguration>(config =>
{
    var channel = new ServerTelemetryChannel();
    channel.StorageFolder = @"C:\temp\appinsights";  // Disk buffer location
    channel.MaxTransmissionBufferCapacity = 5 * 1024 * 1024;  // 5 MB
    config.TelemetryChannel = channel;
});
```

**Key Differences:**
- `InMemoryChannel` — Loses data if app crashes before flush
- `ServerTelemetryChannel` — Persists to disk, retries on failure
- Web apps default: `ServerTelemetryChannel`
- Console apps / Functions default: `InMemoryChannel`
- Always call `Flush()` + delay for `InMemoryChannel` before process exit

---

## MODULE 4: USAGE ANALYTICS

### 4.1 Users, Sessions, Events

**Users:**
- Unique users by `user_Id` or anonymous ID (cookie-based)
- Browser SDK auto-generates anonymous IDs
- Set authenticated user: `telemetryClient.Context.User.AuthenticatedUserId`
- Tracks how many users interact with your app

**Sessions:**
- Groups of user interactions within a time window
- Default session timeout: **30 minutes** of inactivity
- New session on: timeout, new day, or explicit reset
- Tracked by `session_Id`

**Events:**
- Custom business events tracked with `TrackEvent()`
- Shows which features are popular
- Events per user / per session analysis

**Portal Location:**
- Application Insights → Usage → Users / Sessions / Events

### 4.2 Funnels

**What are Funnels?**
- Track conversion rates through multi-step processes
- E.g., Home → Product → Cart → Checkout → Purchase
- Shows drop-off at each step
- Helps identify where users abandon

**Example Funnel:**
```
Step 1: PageView "ProductList"     → 10,000 users (100%)
Step 2: Event "AddToCart"          → 3,500 users  (35%)
Step 3: PageView "Checkout"        → 2,000 users  (20%)
Step 4: Event "PurchaseComplete"   → 800 users    (8%)
```

**Portal:** Application Insights → Usage → Funnels

### 4.3 Cohorts

**What are Cohorts?**
- Groups of users defined by shared characteristics
- E.g., "Users who experienced an error last week"
- Reusable across analysis tools
- Can be defined by: events, page views, custom properties

**Built-in Cohort Definitions:**
- Users who used feature X in last 7 days
- Users from specific countries
- Users on specific browsers/devices
- Users who encountered specific exceptions

### 4.4 Impact Analysis

**What is Impact?**
- Analyzes how load times and performance affect conversion
- Answers: "Does slow load time reduce purchases?"
- Correlates page view durations with custom events
- Portal: Application Insights → Usage → Impact

### 4.5 Retention Analysis

**What is Retention?**
- How many users return to your app over time
- Shows: "Of users who did X on Day 1, how many returned by Day 7?"
- Configurable time periods
- Portal: Application Insights → Usage → Retention

### 4.6 User Flows

**What is User Flows?**
- Visual representation of how users navigate your app
- Shows common paths between pages and events
- Identifies where users go after a specific page/event
- Helps find unexpected navigation patterns
- Portal: Application Insights → Usage → User Flows

---

## MODULE 5: OPENTELEMETRY & AZURE MONITOR

### 5.1 OpenTelemetry Overview

**What is OpenTelemetry?**
- Vendor-neutral observability framework
- Standard for traces, metrics, and logs
- Supported by CNCF (Cloud Native Computing Foundation)
- Replaces vendor-specific SDKs (like classic Application Insights SDK)

**Azure Monitor OpenTelemetry Distro:**
- Microsoft's packaging of OpenTelemetry for Azure Monitor
- Includes exporters, instrumentations, and configuration
- Recommended for new projects going forward

**Installation:**
```bash
dotnet add package Azure.Monitor.OpenTelemetry.AspNetCore
```

**Configuration:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add Azure Monitor OpenTelemetry
builder.Services.AddOpenTelemetry().UseAzureMonitor(options =>
{
    options.ConnectionString = "InstrumentationKey=xxx;...";
});

var app = builder.Build();
app.Run();
```

### 5.2 Classic SDK vs OpenTelemetry

| Aspect | Classic SDK | OpenTelemetry Distro |
|--------|-------------|---------------------|
| **Package** | `Microsoft.ApplicationInsights.AspNetCore` | `Azure.Monitor.OpenTelemetry.AspNetCore` |
| **Setup** | `AddApplicationInsightsTelemetry()` | `AddOpenTelemetry().UseAzureMonitor()` |
| **Custom Telemetry** | `TelemetryClient` | OpenTelemetry APIs (`ActivitySource`, `Meter`) |
| **Vendor Lock-in** | Azure-specific | Vendor-neutral |
| **Status** | Supported (maintenance) | Recommended for new projects |
| **Initializers** | `ITelemetryInitializer` | Resource attributes, processors |
| **Processors** | `ITelemetryProcessor` | OpenTelemetry processors |

### 5.3 Custom Telemetry with OpenTelemetry

```csharp
using System.Diagnostics;
using System.Diagnostics.Metrics;

// Traces (replaces TelemetryClient operations)
private static readonly ActivitySource _activitySource = new("MyApp.Orders");

public void ProcessOrder(Order order)
{
    using var activity = _activitySource.StartActivity("ProcessOrder");
    activity?.SetTag("order.id", order.Id);
    activity?.SetTag("order.region", order.Region);
    
    // Activity = Span in OpenTelemetry terms
    // Automatically correlated across services
}

// Metrics (replaces GetMetric / TrackMetric)
private static readonly Meter _meter = new("MyApp.Orders");
private static readonly Counter<long> _orderCounter = _meter.CreateCounter<long>("orders.processed");
private static readonly Histogram<double> _orderDuration = _meter.CreateHistogram<double>("orders.duration");

public void RecordMetrics(Order order, TimeSpan duration)
{
    _orderCounter.Add(1, new KeyValuePair<string, object?>("region", order.Region));
    _orderDuration.Record(duration.TotalMilliseconds);
}

// Logs (uses ILogger — automatically captured)
private readonly ILogger<OrderService> _logger;

public void LogOrder(Order order)
{
    _logger.LogInformation("Order {OrderId} processed for {Region}", order.Id, order.Region);
    // Automatically sent to Application Insights as trace telemetry
}
```

### 5.4 When to Use Which

| Scenario | Recommendation |
|----------|---------------|
| New .NET 6+ project | OpenTelemetry Distro |
| Existing app with classic SDK | Keep classic SDK (migration optional) |
| Multi-cloud / vendor-neutral needed | OpenTelemetry |
| Need Snapshot Debugger / Profiler | Classic SDK (not yet in OpenTelemetry) |
| Azure Functions | Classic SDK (better integration) |
| Java / Python / Node.js | OpenTelemetry (preferred) |

---

## MODULE 6: DATA EXPORT, RETENTION & COST MANAGEMENT

### 6.1 Data Retention

**Retention Periods:**
- Default: **90 days**
- Configurable: 30, 60, 90, 120, 180, 270, 365, 550, 730 days
- Longer retention = higher cost
- Set per Application Insights resource or Log Analytics workspace

```bash
# Set retention via CLI
az monitor app-insights component update \
  --app myappinsights \
  --resource-group myRG \
  --retention-time 180  # days
```

**Key Points:**
- Data older than retention period is automatically deleted
- Retention applies to ALL telemetry types equally
- Cannot selectively retain only certain telemetry types
- For long-term archival, export data before retention expires

### 6.2 Daily Cap

**What is Daily Cap?**
- Limits daily data ingestion volume (in GB)
- When cap is reached: data collection **stops** until next day (UTC midnight)
- Prevents unexpected cost spikes
- **WARNING**: Data is LOST when cap is reached — not queued

```bash
# Set daily cap
az monitor app-insights component billing update \
  --app myappinsights \
  --resource-group myRG \
  --cap 10  # GB per day
```

**Best Practices:**
- Set daily cap alerts to warn before reaching limit
- Use sampling instead of daily cap when possible (sampling preserves data accuracy)
- Daily cap resets at UTC midnight

### 6.3 Diagnostic Settings (Export)

**Continuous Export is DEPRECATED → Use Diagnostic Settings**

**Diagnostic Settings allows exporting to:**
1. **Log Analytics Workspace** — for KQL queries and long-term analysis
2. **Storage Account** — for archival and compliance
3. **Event Hubs** — for real-time streaming to external systems
4. **Partner Solutions** — third-party integrations

```bash
# Create diagnostic setting to export to Storage Account
az monitor diagnostic-settings create \
  --name "export-to-storage" \
  --resource "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Insights/components/{app}" \
  --storage-account "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{sa}" \
  --logs '[{"category": "AppRequests", "enabled": true}, 
           {"category": "AppExceptions", "enabled": true},
           {"category": "AppDependencies", "enabled": true}]'

# Export to Event Hub
az monitor diagnostic-settings create \
  --name "export-to-eventhub" \
  --resource "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Insights/components/{app}" \
  --event-hub-rule "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/{ns}/authorizationRules/RootManageSharedAccessKey" \
  --logs '[{"category": "AppTraces", "enabled": true}]'
```

**Available Log Categories for Export:**
| Category | Description |
|----------|-------------|
| `AppRequests` | Request telemetry |
| `AppDependencies` | Dependency telemetry |
| `AppExceptions` | Exception telemetry |
| `AppTraces` | Trace/log telemetry |
| `AppEvents` | Custom event telemetry |
| `AppMetrics` | Custom metric telemetry |
| `AppPageViews` | Page view telemetry |
| `AppAvailabilityResults` | Availability test results |
| `AppPerformanceCounters` | Performance counter data |
| `AppBrowserTimings` | Browser timing data |
| `AppSystemEvents` | System events |

### 6.4 Pricing & Cost Optimization

**Pricing Model:**
- **Pay-As-You-Go**: Per GB ingested (first 5 GB/month free per billing account)
- **Commitment Tiers**: 100, 200, 300, 400, 500 GB/day at discounted rates

**Cost Optimization Strategies:**

| Strategy | Savings | Trade-off |
|----------|---------|-----------|
| Adaptive sampling | High | May miss rare events |
| Fixed-rate sampling | Predictable | Consistent data reduction |
| Telemetry processors (filtering) | Medium | Must filter carefully |
| Daily cap | Last resort | Data LOSS when reached |
| Shorter retention | Low-Medium | Historical data deleted |
| Ingestion sampling | Medium | Applied server-side |
| Commitment tiers | Medium | Requires consistent volume |

**Estimating Costs:**
- Average telemetry item: ~1 KB
- 1 million requests/day ≈ 1 GB/day (without dependencies, traces, etc.)
- Dependencies, traces, exceptions add significantly
- Enable sampling to reduce by 50-95%

### 6.5 Workspace-Based vs Classic Resources

| Aspect | Classic (Deprecated) | Workspace-Based (Current) |
|--------|---------------------|--------------------------|
| **Data storage** | Application Insights internal | Log Analytics workspace |
| **Query** | Application Insights portal | Log Analytics + App Insights |
| **Cross-resource queries** | Limited | Full support |
| **Data export** | Continuous Export | Diagnostic Settings |
| **Access control** | Resource-level | Workspace-level RBAC |
| **Pricing** | Per GB | Log Analytics pricing |
| **New resources** | ❌ Can't create new classic | ✅ Default for new resources |

---

## MODULE 7: AZURE MONITOR ALERTS & ACTION GROUPS (DEEP DIVE)

### 7.1 Alert Types

#### Metric Alerts
- Evaluate metric conditions at regular intervals
- **Static threshold**: Fixed value (e.g., avg response time > 5000ms)
- **Dynamic threshold**: ML-based, adapts to metric patterns automatically
- Near real-time evaluation (1-minute frequency)

```bash
# Static threshold alert
az monitor metrics alert create \
  --name "HighResponseTime" \
  --resource-group myRG \
  --scopes <app-insights-resource-id> \
  --condition "avg requests/duration > 5000" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --severity 2 \
  --action <action-group-id>

# Dynamic threshold alert
az monitor metrics alert create \
  --name "AnomalousFailures" \
  --resource-group myRG \
  --scopes <app-insights-resource-id> \
  --condition "count requests/failed > dynamic medium of 4 violations out of 5 evaluations" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action <action-group-id>
```

#### Log Alerts (KQL-based)
- Run KQL query on schedule
- Alert when results meet criteria
- Can query across multiple workspaces

```bash
# Create log alert rule
az monitor scheduled-query create \
  --name "TooManyExceptions" \
  --resource-group myRG \
  --scopes <log-analytics-workspace-id> \
  --condition "count > 50" \
  --condition-query "exceptions | where timestamp > ago(5m) | summarize count()" \
  --evaluation-frequency 5m \
  --window-size 5m \
  --severity 1 \
  --action <action-group-id>
```

#### Activity Log Alerts
- Triggered by Azure resource management events
- E.g., "Resource deleted", "Deployment failed"
- NOT for application telemetry — for Azure platform events

#### Smart Detection Alerts
- Automatic anomaly detection (no configuration needed)
- Detects:
  - **Failure anomalies**: Unusual spike in failed requests
  - **Performance anomalies**: Response time degradation
  - **Memory leak**: Steady increase in memory consumption
  - **Exception volume increase**: Sudden spike in exceptions
  - **Dependency duration increase**: External calls getting slower

### 7.2 Alert Severity Levels

| Severity | Level | Description |
|----------|-------|-------------|
| Sev 0 | Critical | Immediate action required |
| Sev 1 | Error | Urgent attention needed |
| Sev 2 | Warning | Potential issue |
| Sev 3 | Informational | For awareness |
| Sev 4 | Verbose | Debug/detailed info |

### 7.3 Action Groups

**What are Action Groups?**
- Define WHO to notify and WHAT to do when alert fires
- Reusable across multiple alert rules
- Support multiple notification and action types

**Notification Types:**
| Type | Description |
|------|-------------|
| Email | Send to email addresses |
| SMS | Send text messages |
| Push (Azure app) | Azure mobile app notification |
| Voice | Phone call |

**Action Types:**
| Type | Description |
|------|-------------|
| Azure Function | Trigger a function |
| Logic App | Start a Logic App workflow |
| Webhook | HTTP POST to a URL |
| ITSM | Create ticket in ITSM tool |
| Automation Runbook | Run an Azure Automation runbook |
| Event Hub | Send to Event Hub |
| Secure Webhook | Webhook with Azure AD auth |

```bash
# Create action group
az monitor action-group create \
  --name "DevOpsTeam" \
  --resource-group myRG \
  --short-name "DevOps" \
  --action email devopsLead devops@company.com \
  --action webhook opsWebhook "https://hooks.slack.com/services/xxx" \
  --action azurefunction autoRemediate <function-app-resource-id> <function-name> <http-trigger-url>
```

### 7.4 Alert Processing Rules

**What are Alert Processing Rules?**
- Apply actions/suppressions to alerts AFTER they fire
- Use for:
  - **Suppression**: Suppress alerts during maintenance windows
  - **Action routing**: Add action groups to alerts based on filters
- Filter by: resource type, resource group, alert severity, etc.

```bash
# Suppress all alerts during maintenance window
az monitor alert-processing-rule create \
  --name "MaintenanceWindow" \
  --resource-group myRG \
  --rule-type RemoveAllActionGroups \
  --scopes <resource-group-id> \
  --schedule-recurrence-type Weekly \
  --schedule-start-datetime "2026-02-15 01:00:00" \
  --schedule-end-datetime "2026-02-15 05:00:00"
```

---

## MODULE 8: INTEGRATION WITH AZURE SERVICES

### 8.1 App Service Auto-Instrumentation

**Enable Without Code Changes:**
- Navigate to App Service → Settings → Application Insights → Enable
- Or via app settings:

```bash
az webapp config appsettings set \
  --name mywebapp \
  --resource-group myRG \
  --settings \
    APPLICATIONINSIGHTS_CONNECTION_STRING="InstrumentationKey=xxx;..." \
    ApplicationInsightsAgent_EXTENSION_VERSION="~3" \
    XDT_MicrosoftApplicationInsights_Mode="Recommended"
```

**Auto-Collected Data:**
- Incoming HTTP requests
- Outgoing HTTP dependency calls
- SQL queries
- Exceptions
- Performance counters (CPU, memory, request rate)

**Instrumentation Modes:**
| Mode | Description |
|------|-------------|
| `Recommended` | Full auto-collection (default) |
| `Basic` | Minimal collection |
| `disabled` | Turn off auto-instrumentation |

### 8.2 Azure Functions Monitoring

**Configuration in host.json:**
```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "maxTelemetryItemsPerSecond": 20,
        "excludedTypes": "Request"
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

**Key Settings:**
- `samplingSettings.isEnabled` — Enable/disable sampling (default: true)
- `maxTelemetryItemsPerSecond` — Sampling target rate (default: 20)
- `excludedTypes` — Telemetry types to exclude from sampling: `Dependency`, `Event`, `Exception`, `PageView`, `Request`, `Trace`
- `enableLiveMetrics` — Enable Live Metrics Stream

**Functions-Specific Telemetry:**
- Automatic request tracking for each function invocation
- Automatic dependency tracking
- ILogger integration — logs go directly to Application Insights `traces` table
- Function-level metrics (invocations, errors, duration)

### 8.3 Application Insights for Worker Services

**For non-web apps (console, background workers, Windows Services):**

```bash
dotnet add package Microsoft.ApplicationInsights.WorkerService
```

```csharp
var builder = Host.CreateDefaultBuilder(args);

builder.ConfigureServices(services =>
{
    services.AddApplicationInsightsTelemetryWorkerService(options =>
    {
        options.ConnectionString = "InstrumentationKey=xxx;...";
    });
    
    services.AddHostedService<MyBackgroundWorker>();
});

var host = builder.Build();
await host.RunAsync();
```

**Key Differences from Web SDK:**
- Uses `InMemoryChannel` by default (not `ServerTelemetryChannel`)
- No automatic HTTP request tracking
- Must manually track operations
- Must call `Flush()` before shutdown

### 8.4 Application Insights SDK Configuration Priority

**Connection string resolution order (highest to lowest):**
1. Code: `options.ConnectionString = "..."` in `AddApplicationInsightsTelemetry()`
2. Environment variable: `APPLICATIONINSIGHTS_CONNECTION_STRING`
3. Config file: `appsettings.json` → `ApplicationInsights:ConnectionString`

**This is commonly tested — code always wins over env vars over config files.**

---

## MODULE 9: SECURITY, ACCESS CONTROL & COMPLIANCE

### 9.1 RBAC Roles for Application Insights

| Role | Permissions |
|------|------------|
| **Monitoring Reader** | Read monitoring data (metrics, logs, alerts) |
| **Monitoring Contributor** | Read + create/edit alerts, action groups, diagnostic settings |
| **Application Insights Component Contributor** | Full management of Application Insights resources |
| **Log Analytics Reader** | Read Log Analytics data |
| **Log Analytics Contributor** | Read + manage Log Analytics workspace |

```bash
# Assign Monitoring Reader role
az role assignment create \
  --assignee user@company.com \
  --role "Monitoring Reader" \
  --scope "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Insights/components/{app}"
```

### 9.2 API Keys (Programmatic Access)

**Types of API Access:**

| Method | Description | Use Case |
|--------|-------------|----------|
| **API Key** | Portal-generated key | Read telemetry, write annotations |
| **Azure AD (Entra ID)** | Token-based auth | Recommended for apps/services |
| **REST API** | HTTP endpoints | Custom integrations |

```bash
# Create API key (portal is easier)
# Application Insights → API Access → Create API Key
# Permissions: Read telemetry, Write annotations, Authenticate SDK
```

**API Key Permissions:**
- **Read telemetry**: Query data via REST API
- **Write annotations**: Add release annotations
- **Authenticate SDK control channel**: Used for Live Metrics and Profiler authentication

### 9.3 Release Annotations

**What are Release Annotations?**
- Visual markers on metrics charts showing deployments
- Correlate performance changes with deployments
- Can be created via REST API or Azure DevOps extension

```bash
# Create release annotation via REST API
$body = @{
    Id = [guid]::NewGuid().ToString()
    AnnotationName = "Release 1.2.0"
    EventTime = (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ss")
    Category = "Deployment"
    Properties = '{"ReleaseName":"1.2.0","Environment":"Production"}'
} | ConvertTo-Json

Invoke-RestMethod -Uri "https://aigs1.aisvc.visualstudio.com/applications/{app-id}/Annotations" `
  -Method Put `
  -Headers @{ "X-AIAPIKEY" = "<api-key>" } `
  -Body $body `
  -ContentType "application/json"
```

### 9.4 Data Privacy & PII

**PII Handling:**
- Application Insights may collect IP addresses, user agents, URLs with query params
- YOU are responsible for not sending PII in custom properties/events

**IP Anonymization:**
- By default, IP addresses are collected but the last octet is zeroed (e.g., 192.168.1.0)
- Fully anonymous: `0.0.0.0` stored when geo-lookup is disabled

**Telemetry Initializer to Remove PII:**
```csharp
public class PiiRemovalInitializer : ITelemetryInitializer
{
    public void Initialize(ITelemetry telemetry)
    {
        // Remove IP address
        telemetry.Context.Location.Ip = "0.0.0.0";
        
        // Clear user ID if needed
        telemetry.Context.User.Id = null;
        
        // Sanitize URLs
        if (telemetry is RequestTelemetry request)
        {
            if (request.Url != null)
            {
                var builder = new UriBuilder(request.Url) { Query = "" };
                request.Url = builder.Uri;
            }
        }
    }
}

// Register
builder.Services.AddSingleton<ITelemetryInitializer, PiiRemovalInitializer>();
```

### 9.5 Data Purge

**When You Need to Delete Data:**
- GDPR / data subject requests
- Accidental PII collection
- Compliance requirements

```bash
# Purge data via REST API
POST https://management.azure.com/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Insights/components/{app}/purge?api-version=2020-02-02

{
  "table": "requests",
  "filters": [
    {
      "column": "client_IP",
      "operator": "==",
      "value": "192.168.1.100"
    }
  ]
}
```

**Key Points:**
- Purge is **permanent** — data cannot be recovered
- Purge may take up to **30 days** to complete
- SLA: Data is no longer queryable within 5 days of purge request
- Only specific data matching filters is deleted

### 9.6 Private Link / Private Endpoints

**Azure Monitor Private Link Scope (AMPLS):**
- Connect Application Insights to your VNet via private endpoint
- Data ingestion and queries over private network
- No public internet exposure

```bash
# Create Azure Monitor Private Link Scope
az monitor private-link-scope create \
  --name myAMPLS \
  --resource-group myRG

# Connect Application Insights to Private Link Scope
az monitor private-link-scope scoped-resource create \
  --name myAppInsightsLink \
  --resource-group myRG \
  --scope-name myAMPLS \
  --linked-resource <app-insights-resource-id>

# Create private endpoint
az network private-endpoint create \
  --name myPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet mySubnet \
  --private-connection-resource-id <ampls-resource-id> \
  --group-id "azuremonitor" \
  --connection-name myConnection
```

---

## MODULE 10: ADVANCED KQL & TROUBLESHOOTING SCENARIOS

### 10.1 Advanced KQL Patterns

**let Statements (Variables):**
```kusto
let timeRange = 24h;
let threshold = 5000;

requests
| where timestamp > ago(timeRange)
| where duration > threshold
| summarize count() by name
| order by count_ desc
```

**join Operations:**
```kusto
// Inner join: Requests with their exceptions
requests
| where success == false
| join kind=inner (
    exceptions
    | project operation_Id, exceptionType = type, exceptionMessage = outerMessage
) on operation_Id
| project timestamp, name, resultCode, exceptionType, exceptionMessage
| limit 100

// Left outer join: All requests, optionally with exceptions
requests
| where timestamp > ago(1h)
| join kind=leftouter (
    exceptions
    | summarize exceptionCount = count() by operation_Id
) on operation_Id
| extend hasException = isnotnull(exceptionCount)
```

**make-series (Time Series):**
```kusto
// Create time series for anomaly detection
requests
| make-series requestCount = count() on timestamp from ago(7d) to now() step 1h
| extend anomalies = series_decompose_anomalies(requestCount)
| render anomalychart
```

**Dynamic Column Extraction:**
```kusto
// Extract custom dimensions
requests
| extend userId = tostring(customDimensions.UserId)
| extend region = tostring(customDimensions.Region)
| summarize count() by userId, region
| order by count_ desc

// Parse JSON from custom properties
traces
| where message contains "Order"
| extend orderData = parse_json(customDimensions.OrderDetails)
| extend orderId = tostring(orderData.id)
| extend orderTotal = todouble(orderData.total)
```

**Render Operations:**
```kusto
// Time chart
requests | summarize count() by bin(timestamp, 5m) | render timechart

// Bar chart
requests | summarize count() by name | render barchart

// Pie chart
requests | summarize count() by resultCode | render piechart

// Scatter chart
requests | project duration, customMetrics_value = todouble(customDimensions.PayloadSize) 
| render scatterchart
```

### 10.2 Cross-Resource Queries

```kusto
// Query across multiple Application Insights resources
app("other-app-insights-name").requests
| where timestamp > ago(1h)
| union (
    requests
    | where timestamp > ago(1h)
)
| summarize count() by appName

// Query Log Analytics workspace from Application Insights
workspace("my-workspace").AzureActivity
| where OperationNameValue == "MICROSOFT.WEB/SITES/WRITE"
| where timestamp > ago(24h)
```

### 10.3 Common Troubleshooting Scenarios

**Scenario 1: Identify Slow Dependencies**
```kusto
dependencies
| where timestamp > ago(1h)
| where success == true
| summarize 
    avgDuration = avg(duration),
    p95Duration = percentile(duration, 95),
    p99Duration = percentile(duration, 99),
    callCount = count()
  by name, type, target
| where p95Duration > 1000
| order by p95Duration desc
```

**Scenario 2: Find Error Spikes**
```kusto
requests
| where timestamp > ago(24h)
| summarize 
    totalCount = count(),
    failedCount = countif(success == false),
    failureRate = round(countif(success == false) * 100.0 / count(), 2)
  by bin(timestamp, 15m)
| where failureRate > 5  // Alert if >5% failure rate
| order by timestamp desc
```

**Scenario 3: Track User Journey**
```kusto
// All telemetry for a specific user
union requests, dependencies, exceptions, traces, pageViews
| where user_Id == "specific-user-id"
| where timestamp > ago(1h)
| project timestamp, itemType, name, duration, success
| order by timestamp asc
```

**Scenario 4: Dependency Failure Correlation**
```kusto
// Which dependency failures cause request failures?
requests
| where success == false
| join kind=inner (
    dependencies
    | where success == false
) on operation_Id
| summarize 
    requestFailures = dcount(operation_Id),
    affectedEndpoints = make_set(name)
  by dependencyName = name1, dependencyTarget = target
| order by requestFailures desc
```

### 10.4 Application Insights REST API

**Query API:**
```bash
# Query Application Insights via REST
GET https://api.applicationinsights.io/v1/apps/{app-id}/query?query=requests | take 10

Headers:
  x-api-key: <api-key>
  # OR
  Authorization: Bearer <aad-token>
```

**Metrics API:**
```bash
# Get metric values
GET https://api.applicationinsights.io/v1/apps/{app-id}/metrics/requests/count?timespan=PT1H&interval=PT5M

Headers:
  x-api-key: <api-key>
```

**Power BI Integration:**
- Export KQL query results directly to Power BI
- Use the M query connector for Application Insights
- Schedule refresh for live dashboards

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**1. Telemetry Types:**
- Requests, dependencies, exceptions, traces
- Custom events and metrics
- Automatic vs manual tracking

**2. Instrumentation:**
- SDK-based vs auto-instrumentation
- TelemetryClient methods
- Telemetry initializers and processors
- OpenTelemetry Distro vs Classic SDK

**3. Sampling:**
- Adaptive vs fixed-rate vs ingestion
- Excluded types
- Impact on pre-aggregated vs log-based metrics

**4. Querying:**
- Common Kusto queries (join, let, make-series, render)
- Tables (requests, dependencies, etc.)
- Correlation with operation_Id
- Cross-resource queries with `app()` and `workspace()`

**5. Features:**
- Application Map, Live Metrics, Availability Tests
- Snapshot Debugger (production debugging, minidumps)
- Profiler (hot path analysis, flame graphs)
- Transaction Diagnostics

**6. Metrics:**
- Pre-aggregated vs log-based metrics
- `GetMetric().TrackValue()` vs `TrackMetric()`
- Metrics Explorer, standard vs custom metrics
- Telemetry channels (InMemoryChannel vs ServerTelemetryChannel)

**7. Usage Analytics:**
- Users, Sessions, Events
- Funnels, Cohorts, Impact, Retention, User Flows

**8. Alerts & Actions:**
- Metric alerts (static and dynamic thresholds)
- Log alerts (KQL-based)
- Smart Detection alerts
- Action Groups (email, SMS, webhook, Azure Function, Logic App)
- Alert processing rules (suppression during maintenance)

**9. Data Management & Cost:**
- Data retention (default 90 days, up to 730)
- Daily cap (data LOST when reached)
- Diagnostic Settings (export to Storage, Event Hub, Log Analytics)
- Continuous Export deprecated → Diagnostic Settings
- Workspace-based vs Classic resources

**10. Security & Privacy:**
- RBAC roles (Monitoring Reader, Monitoring Contributor, Component Contributor)
- API Keys for programmatic access
- PII handling and IP anonymization
- Data purge API
- Private Link / AMPLS
- Release annotations

**11. Integration:**
- App Service auto-instrumentation (no code changes)
- Azure Functions monitoring (host.json config)
- Worker Service SDK
- Connection string priority: Code > Env var > Config file

### Sample Exam Questions

**Q1:** Which telemetry type tracks calls to external services like databases and HTTP APIs?
- A) Request
- B) Dependency
- C) Trace
- D) Event

**Answer: B) Dependency**
Explanation: Dependency telemetry tracks outgoing calls to external services, databases, and APIs.

---

**Q2:** What is the recommended way to configure Application Insights authentication?
- A) Instrumentation Key
- B) Connection String
- C) API Key
- D) Access Token

**Answer: B) Connection String**
Explanation: Connection String is recommended as it includes the instrumentation key and endpoint information.

---

**Q3:** Which method should you use to track custom business events?
- A) TrackRequest
- B) TrackDependency
- C) TrackEvent
- D) TrackTrace

**Answer: C) TrackEvent**
Explanation: TrackEvent is used for custom business events like "OrderPlaced" or "ButtonClicked".

---

**Q4:** What is the default sampling type in Application Insights SDK?
- A) Fixed-rate sampling
- B) Adaptive sampling
- C) Ingestion sampling
- D) No sampling

**Answer: B) Adaptive sampling**
Explanation: Adaptive sampling is enabled by default and automatically adjusts to maintain target volume.

---

**Q5:** Which Log Analytics table contains HTTP request data?
- A) traces
- B) dependencies
- C) requests
- D) customEvents

**Answer: C) requests**
Explanation: The requests table contains incoming HTTP request telemetry.

---

**Q6:** What is the purpose of a Telemetry Initializer?
- A) Filter telemetry
- B) Add properties to all telemetry items
- C) Sample telemetry
- D) Send telemetry to different endpoints

**Answer: B) Add properties to all telemetry items**
Explanation: Telemetry Initializers add or modify properties on all telemetry items before they're sent.

---

**Q7:** Which feature provides real-time telemetry with 1-second latency?
- A) Application Map
- B) Log Analytics
- C) Live Metrics
- D) Workbooks

**Answer: C) Live Metrics**
Explanation: Live Metrics Stream provides real-time telemetry with approximately 1-second latency.

---

**Q8:** How do you ensure telemetry is sent before application shutdown?
- A) Call TrackEvent
- B) Call Flush() and wait
- C) Set EnableAutoFlush = true
- D) Use async tracking

**Answer: B) Call Flush() and wait**
Explanation: Call TelemetryClient.Flush() and wait a few seconds to ensure pending telemetry is transmitted.

---

**Q9:** What property correlates telemetry across distributed services?
- A) session_Id
- B) operation_Id
- C) user_Id
- D) request_Id

**Answer: B) operation_Id**
Explanation: operation_Id is the correlation identifier that links all telemetry for a single transaction.

---

**Q10:** Which availability test type allows you to test complex user scenarios?
- A) URL Ping Test
- B) Standard Test
- C) Multi-Step Web Test
- D) Custom Track Availability

**Answer: D) Custom Track Availability**
Explanation: Custom Track Availability using TrackAvailability API allows implementing any test logic.

---

**Q11:** What is the purpose of cloud role name?
- A) Identify the Azure region
- B) Identify the service in Application Map
- C) Set the subscription
- D) Configure sampling

**Answer: B) Identify the service in Application Map**
Explanation: Cloud role name identifies different services/components in the Application Map visualization.

---

**Q12:** Which interface should you implement to filter telemetry?
- A) ITelemetryInitializer
- B) ITelemetryProcessor
- C) ITelemetryModule
- D) ITelemetryChannel

**Answer: B) ITelemetryProcessor**
Explanation: ITelemetryProcessor can filter (exclude) telemetry items from being sent.

---

**Q13:** What is the maximum severity level for trace telemetry?
- A) 3 (Warning)
- B) 4 (Error)
- C) 5 (Critical)
- D) No maximum

**Answer: C) 5 (Critical)**
Explanation: Severity levels are: 0=Verbose, 1=Information, 2=Warning, 3=Error, 4=Critical.

---

**Q14:** Which Kusto function calculates the 95th percentile?
- A) avg()
- B) median()
- C) percentile()
- D) p95()

**Answer: C) percentile()**
Explanation: Use percentile(column, 95) to calculate the 95th percentile.

---

**Q15:** How do you exclude specific telemetry types from sampling?
- A) Use ITelemetryProcessor
- B) Use excludedTypes parameter in sampling configuration
- C) Set sampling percentage to 100
- D) Use Telemetry Initializer

**Answer: B) Use excludedTypes parameter in sampling configuration**
Explanation: The excludedTypes parameter specifies telemetry types that shouldn't be sampled.

---

**Q16:** You need to debug an exception that occurs intermittently in production without attaching a debugger. Which feature should you enable?
- A) Application Insights Profiler
- B) Snapshot Debugger
- C) Live Metrics Stream
- D) Smart Detection

**Answer: B) Snapshot Debugger**
Explanation: Snapshot Debugger automatically captures debug snapshots (local variables, call stack) when exceptions occur in production, allowing you to debug without reproducing the issue.

---

**Q17:** Your web application has slow response times. You need to identify which methods are consuming the most execution time. Which feature should you use?
- A) Snapshot Debugger
- B) Application Map
- C) Application Insights Profiler
- D) Live Metrics

**Answer: C) Application Insights Profiler**
Explanation: Profiler traces execution paths and produces flame graphs showing "hot paths" — the methods that take the most time.

---

**Q18:** You are tracking a custom metric that is reported thousands of times per second. You need the metric to be accurate and NOT affected by sampling. Which API should you use?
- A) `_telemetryClient.TrackMetric("MetricName", value)`
- B) `_telemetryClient.TrackEvent("MetricName")`
- C) `_telemetryClient.GetMetric("MetricName").TrackValue(value)`
- D) `_telemetryClient.TrackTrace("MetricName")`

**Answer: C) `_telemetryClient.GetMetric("MetricName").TrackValue(value)`**
Explanation: `GetMetric()` creates pre-aggregated metrics that are NOT affected by sampling, aggregated locally, and much more efficient at high volume than `TrackMetric()`.

---

**Q19:** What is the key difference between `TrackMetric()` and `GetMetric().TrackValue()`?
- A) `TrackMetric()` is pre-aggregated; `GetMetric()` is log-based
- B) `GetMetric()` is pre-aggregated and not affected by sampling; `TrackMetric()` is log-based and affected by sampling
- C) They are identical
- D) `TrackMetric()` supports dimensions; `GetMetric()` does not

**Answer: B) `GetMetric()` is pre-aggregated and not affected by sampling; `TrackMetric()` is log-based and affected by sampling**
Explanation: `GetMetric()` pre-aggregates values locally before sending, making it accurate regardless of sampling. `TrackMetric()` stores each value as a log event and IS affected by sampling.

---

**Q20:** Which telemetry channel should you use for production web applications?
- A) InMemoryChannel
- B) ServerTelemetryChannel
- C) PersistentChannel
- D) BufferedChannel

**Answer: B) ServerTelemetryChannel**
Explanation: ServerTelemetryChannel uses disk-backed storage with retry logic, ensuring telemetry is not lost if the app crashes or network fails. InMemoryChannel is for development/short-lived processes only.

---

**Q21:** What happens when the Application Insights daily cap is reached?
- A) Data is queued and sent the next day
- B) Data collection stops and data is LOST until UTC midnight
- C) Sampling is automatically enabled
- D) An exception is thrown in the application

**Answer: B) Data collection stops and data is LOST until UTC midnight**
Explanation: When the daily cap is reached, ingestion stops completely. Data generated during this period is permanently lost. The cap resets at UTC midnight. Use sampling instead of daily cap when possible.

---

**Q22:** What is the default data retention period for Application Insights?
- A) 30 days
- B) 60 days
- C) 90 days
- D) 365 days

**Answer: C) 90 days**
Explanation: The default retention is 90 days. It can be configured from 30 to 730 days. Longer retention incurs additional costs.

---

**Q23:** You need to export Application Insights data to a Storage Account for long-term archival. Which feature should you use?
- A) Continuous Export
- B) Diagnostic Settings
- C) Data Export API
- D) Log Analytics Export

**Answer: B) Diagnostic Settings**
Explanation: Continuous Export is deprecated. Diagnostic Settings is the current recommended approach for exporting data to Storage Accounts, Event Hubs, or Log Analytics workspaces.

---

**Q24:** Which Application Insights feature uses machine learning to automatically detect anomalies without configuration?
- A) Metric Alerts
- B) Log Alerts
- C) Smart Detection
- D) Application Map

**Answer: C) Smart Detection**
Explanation: Smart Detection automatically analyzes telemetry and alerts on anomalies like failure rate spikes, performance degradation, and memory leaks without any manual configuration.

---

**Q25:** What is the correct way to set the cloud role name for Application Map identification?
- A) Set it in `appsettings.json` under `CloudRoleName`
- B) Use a Telemetry Initializer to set `telemetry.Context.Cloud.RoleName`
- C) Set `APPINSIGHTS_CLOUD_ROLE_NAME` environment variable
- D) Configure it in Azure Portal

**Answer: B) Use a Telemetry Initializer to set `telemetry.Context.Cloud.RoleName`**
Explanation: Cloud role name is set via a custom ITelemetryInitializer that sets `telemetry.Context.Cloud.RoleName` on all telemetry items.

---

**Q26:** You need to create an alert that triggers an Azure Function when the average response time exceeds 5 seconds. What two resources must you configure?
- A) Metric Alert and Diagnostic Setting
- B) Metric Alert and Action Group
- C) Log Alert and Webhook
- D) Smart Detection and Event Grid

**Answer: B) Metric Alert and Action Group**
Explanation: You need a Metric Alert to define the condition (avg response time > 5000ms) and an Action Group that specifies the Azure Function to trigger when the alert fires.

---

**Q27:** Which RBAC role allows a user to read Application Insights data but NOT modify alert rules?
- A) Monitoring Contributor
- B) Monitoring Reader
- C) Application Insights Component Contributor
- D) Reader

**Answer: B) Monitoring Reader**
Explanation: Monitoring Reader can read all monitoring data (metrics, logs, alerts) but cannot create or modify any monitoring resources.

---

**Q28:** What is the purpose of release annotations in Application Insights?
- A) Mark code changes in source control
- B) Add visual markers on metric charts showing when deployments occurred
- C) Annotate exceptions with release information
- D) Configure deployment slots

**Answer: B) Add visual markers on metric charts showing when deployments occurred**
Explanation: Release annotations appear as markers on metrics charts, making it easy to correlate performance or error changes with specific deployments.

---

**Q29:** How does Application Insights handle IP addresses by default?
- A) Full IP address is stored
- B) Last octet is zeroed (e.g., 192.168.1.0)
- C) IP address is not collected
- D) IP address is hashed

**Answer: B) Last octet is zeroed (e.g., 192.168.1.0)**
Explanation: By default, Application Insights anonymizes IP addresses by zeroing the last octet for geo-lookup purposes while preserving privacy.

---

**Q30:** You need to connect Application Insights to your VNet so data never traverses the public internet. What should you create?
- A) Service Endpoint
- B) VNet Integration
- C) Azure Monitor Private Link Scope (AMPLS) with Private Endpoint
- D) Network Security Group

**Answer: C) Azure Monitor Private Link Scope (AMPLS) with Private Endpoint**
Explanation: AMPLS allows Application Insights ingestion and queries to flow through a private endpoint in your VNet, avoiding public internet.

---

**Q31:** Which NuGet package is needed to enable Snapshot Debugger via SDK?
- A) Microsoft.ApplicationInsights.SnapshotDebugger
- B) Microsoft.ApplicationInsights.SnapshotCollector
- C) Microsoft.ApplicationInsights.Debugger
- D) Microsoft.Diagnostics.SnapshotCollector

**Answer: B) Microsoft.ApplicationInsights.SnapshotCollector**
Explanation: The `Microsoft.ApplicationInsights.SnapshotCollector` NuGet package enables SDK-based Snapshot Debugger functionality.

---

**Q32:** In an OpenTelemetry-based application using Azure Monitor, which class replaces `TelemetryClient.TrackEvent()`?
- A) EventSource
- B) ActivitySource
- C) Meter
- D) ILogger

**Answer: B) ActivitySource**
Explanation: In OpenTelemetry, `ActivitySource` (creates Activities/Spans) replaces the TelemetryClient tracking pattern. Custom events map to Activities with appropriate attributes.

---

**Q33:** Which KQL function do you use to query data from a different Application Insights resource?
- A) `resource()`
- B) `app()`
- C) `workspace()`
- D) `external()`

**Answer: B) `app()`**
Explanation: The `app()` function allows cross-resource queries. Example: `app("other-app-name").requests | take 10`. For Log Analytics workspaces, use `workspace()`.

---

**Q34:** What is the connection string configuration priority in Application Insights SDK?
- A) Config file > Environment variable > Code
- B) Environment variable > Code > Config file
- C) Code > Environment variable > Config file
- D) All sources are merged

**Answer: C) Code > Environment variable > Config file**
Explanation: Code (options in AddApplicationInsightsTelemetry) takes highest priority, then APPLICATIONINSIGHTS_CONNECTION_STRING environment variable, then appsettings.json.

---

**Q35:** You need to suppress all monitoring alerts during a scheduled maintenance window. What should you create?
- A) Alert rule with time-based condition
- B) Action Group with schedule
- C) Alert Processing Rule with suppression
- D) Diagnostic Setting with filter

**Answer: C) Alert Processing Rule with suppression**
Explanation: Alert Processing Rules can suppress alerts (RemoveAllActionGroups) based on a schedule, perfect for maintenance windows.

---

**Q36:** Which Application Insights feature shows how users navigate through your app and where they drop off in a multi-step process?
- A) User Flows
- B) Funnels
- C) Cohorts
- D) Retention

**Answer: B) Funnels**
Explanation: Funnels track conversion rates through multi-step processes, showing the percentage of users completing each step and where drop-off occurs.

---

**Q37:** You have a .NET console application (not a web app) that needs Application Insights. Which NuGet package should you use?
- A) Microsoft.ApplicationInsights.AspNetCore
- B) Microsoft.ApplicationInsights.WorkerService
- C) Microsoft.ApplicationInsights.Console
- D) Microsoft.ApplicationInsights

**Answer: B) Microsoft.ApplicationInsights.WorkerService**
Explanation: For non-web applications (console apps, background workers, Windows Services), use the WorkerService package. The AspNetCore package is specifically for web applications.

---

**Q38:** What type of alert uses machine learning to automatically determine the threshold based on historical patterns?
- A) Static metric alert
- B) Dynamic metric alert
- C) Log alert
- D) Smart Detection

**Answer: B) Dynamic metric alert**
Explanation: Dynamic threshold alerts use ML to learn metric behavior over time and alert on deviations from expected patterns, without you manually setting a fixed threshold.

---

**Q39:** How long can a data purge request take to complete in Application Insights?
- A) Immediately
- B) Up to 24 hours
- C) Up to 30 days
- D) Up to 90 days

**Answer: C) Up to 30 days**
Explanation: Data purge operations can take up to 30 days, though purged data becomes non-queryable within 5 days. Purge is permanent and cannot be reversed.

---

**Q40:** Which diagnostic setting export category contains custom events tracked with TrackEvent()?
- A) AppRequests
- B) AppTraces
- C) AppEvents
- D) AppMetrics

**Answer: C) AppEvents**
Explanation: Custom events tracked via `TrackEvent()` fall under the `AppEvents` category in Diagnostic Settings export.

---

**Q41:** In Azure Functions, which host.json setting controls the Application Insights sampling target rate?
- A) `logging.applicationInsights.samplingPercentage`
- B) `logging.applicationInsights.samplingSettings.maxTelemetryItemsPerSecond`
- C) `logging.applicationInsights.samplingSettings.samplingRate`
- D) `logging.samplingRate`

**Answer: B) `logging.applicationInsights.samplingSettings.maxTelemetryItemsPerSecond`**
Explanation: In Azure Functions host.json, `samplingSettings.maxTelemetryItemsPerSecond` controls the target rate for adaptive sampling. Default is 20 items per second.

---

**Q42:** Which Application Insights severity level value represents "Warning"?
- A) 1
- B) 2
- C) 3
- D) 4

**Answer: B) 2**
Explanation: Severity levels are: 0=Verbose, 1=Information, 2=Warning, 3=Error, 4=Critical. Note: The numbering starts at 0 and the highest is 4 (Critical), NOT 5.

---

**Q43:** Your application processes 50,000 requests per second. You notice Application Insights costs are very high. What is the BEST approach to reduce costs while maintaining data accuracy for alerting?
- A) Set a daily cap
- B) Enable adaptive sampling and use GetMetric() for critical metrics
- C) Disable Application Insights
- D) Reduce retention to 30 days

**Answer: B) Enable adaptive sampling and use GetMetric() for critical metrics**
Explanation: Adaptive sampling reduces volume while maintaining statistical accuracy. Pre-aggregated metrics via GetMetric() are NOT affected by sampling, so alerts remain accurate. Daily cap causes data loss and should be a last resort.

---

**Q44:** What is the difference between `app()` and `workspace()` functions in KQL?
- A) They are identical
- B) `app()` queries Application Insights resources; `workspace()` queries Log Analytics workspaces
- C) `app()` is for current resource; `workspace()` is for cross-resource
- D) `app()` is deprecated; use `workspace()` only

**Answer: B) `app()` queries Application Insights resources; `workspace()` queries Log Analytics workspaces**
Explanation: Use `app("name")` to query other Application Insights resources and `workspace("name")` to query Log Analytics workspaces in cross-resource queries.

---

**Q45:** Which feature should you use to see the FULL end-to-end call chain of a single request, including all dependencies, traces, and exceptions?
- A) Application Map
- B) Transaction Diagnostics
- C) Live Metrics
- D) Workbooks

**Answer: B) Transaction Diagnostics**
Explanation: Transaction Diagnostics provides a unified timeline view of all telemetry correlated by operation_Id for a single end-to-end transaction, including requests, dependencies, exceptions, and traces.

---

## Key CLI Commands Reference

```bash
# Create Application Insights
az monitor app-insights component create \
  --app myappinsights \
  --resource-group myRG \
  --location eastus \
  --workspace <workspace-resource-id>

# Get connection string
az monitor app-insights component show \
  --app myappinsights \
  --resource-group myRG \
  --query connectionString

# Create metric alert
az monitor metrics alert create \
  --name "High Failure Rate" \
  --resource-group myRG \
  --scopes <app-insights-resource-id> \
  --condition "count requests/failed > 10" \
  --window-size 5m \
  --evaluation-frequency 1m

# Query logs
az monitor app-insights query \
  --app myappinsights \
  --resource-group myRG \
  --analytics-query "requests | where success == false | take 10"
```

---

## Exam Gotchas & Tricky Distinctions

| Common Misconception | Reality |
|---------------------|---------|
| "Instrumentation Key is the recommended auth method" | ❌ **Connection String** is recommended. Instrumentation Key is legacy — connection string includes endpoint info and is more flexible |
| "Connection string in appsettings.json always wins" | ❌ Priority: **Code** > **Environment variable** (`APPLICATIONINSIGHTS_CONNECTION_STRING`) > **Config file** (`appsettings.json`). Code always overrides |
| "`TrackMetric()` and `GetMetric().TrackValue()` are the same" | ❌ `TrackMetric()` = log-based, affected by sampling. `GetMetric()` = **pre-aggregated, NOT affected by sampling**, more efficient at high volume |
| "Sampling affects all metrics equally" | ❌ **Pre-aggregated metrics** (via `GetMetric()`) are NOT affected by sampling. Only **log-based metrics** (via `TrackMetric()`) are affected |
| "Adaptive sampling and fixed-rate sampling can be used together" | ⚠️ Technically possible but NOT recommended. If both are enabled, the effective rate compounds and becomes unpredictable |
| "Daily cap queues data for later" | ❌ When daily cap is reached, data is **permanently LOST** until UTC midnight. Use sampling instead of daily cap when possible |
| "Snapshot Debugger captures full memory dumps" | ❌ It captures **minidumps** — local variables and call stack at the exception point. Not a full process dump |
| "Snapshot Debugger captures a snapshot for every exception" | ❌ Has a **cooldown period** (~15 min per exception type) and daily limits. Not every occurrence generates a snapshot |
| "Profiler and Snapshot Debugger do the same thing" | ❌ **Profiler** = performance (hot paths, flame graphs). **Snapshot Debugger** = debugging (local variables at exception time) |
| "Profiler works on Azure Functions Consumption plan" | ❌ Profiler requires **Premium or Dedicated plan**. NOT supported on Consumption plan |
| "ITelemetryInitializer can filter (exclude) telemetry" | ❌ Initializers can only **add/modify** properties. To **filter/exclude** telemetry, use `ITelemetryProcessor` |
| "ITelemetryProcessor can add properties to all telemetry" | ⚠️ It CAN, but that's not its primary purpose. Use `ITelemetryInitializer` for adding properties, `ITelemetryProcessor` for filtering |
| "InMemoryChannel is safe for production web apps" | ❌ `InMemoryChannel` **loses data** if the app crashes before flush. Use `ServerTelemetryChannel` (disk-backed with retry) for production |
| "`ServerTelemetryChannel` is the default for all app types" | ❌ Default for **web apps** only. Console apps, Workers, and Azure Functions default to `InMemoryChannel` |
| "Continuous Export is the way to export data" | ❌ Continuous Export is **deprecated**. Use **Diagnostic Settings** to export to Storage Account, Event Hub, or Log Analytics |
| "Classic Application Insights resources can still be created" | ❌ New resources must be **workspace-based**. Classic resources are deprecated and can't be created anymore |
| "Smart Detection requires configuration" | ❌ Smart Detection works **automatically** with zero configuration. It uses ML to detect anomalies in failure rate, performance, memory leaks, etc. |
| "Metric alerts and log alerts are the same" | ❌ **Metric alerts** evaluate numeric conditions at 1-minute frequency. **Log alerts** run KQL queries on a schedule (5 min+). Metric alerts are faster |
| "Dynamic threshold alerts require you to set a baseline" | ❌ Dynamic thresholds use **ML to automatically learn** the baseline from historical data. No manual configuration needed |
| "Action Groups can only send emails" | ❌ Action Groups support: email, SMS, voice, push, **webhook, Azure Function, Logic App, Automation Runbook, ITSM, Event Hub, Secure Webhook** |
| "Severity levels go from 0 to 5" | ❌ Severity levels are **0-4**: 0=Verbose, 1=Information, 2=Warning, 3=Error, **4=Critical** (not 5) |
| "Application Map requires manual configuration" | ❌ Application Map is **automatic** when distributed tracing is enabled. Set `Cloud.RoleName` via Telemetry Initializer to label components |
| "You can recover purged data" | ❌ Data purge is **permanent and irreversible**. Takes up to 30 days to complete. Data becomes non-queryable within ~5 days |
| "IP addresses are stored in full by default" | ❌ Last octet is **zeroed** by default (e.g., 192.168.1.0) for privacy. Can be fully anonymized with a Telemetry Initializer |
| "Azure Functions sampling is configured in appsettings.json" | ❌ For Azure Functions, sampling is configured in **`host.json`** under `logging.applicationInsights.samplingSettings`, NOT in appsettings.json |
| "The `app()` and `workspace()` KQL functions do the same thing" | ❌ `app()` = cross-query **Application Insights** resources. `workspace()` = cross-query **Log Analytics workspaces**. Different target types |
| "Release annotations are created automatically on deployment" | ❌ Release annotations must be created **explicitly** via REST API, Azure DevOps extension, or GitHub Action. They don't happen automatically |
| "Application Insights data retention is free regardless of duration" | ❌ Default 90 days is included in ingestion price. **Retention beyond 90 days incurs additional per-GB charges** |
| "Auto-instrumentation collects everything the SDK does" | ❌ Auto-instrumentation collects standard telemetry (requests, dependencies, exceptions). **Custom events, custom metrics, and custom properties require SDK code** |
| "OpenTelemetry Distro supports Snapshot Debugger and Profiler" | ❌ As of now, Snapshot Debugger and Profiler require the **classic Application Insights SDK**. They are NOT yet available in the OpenTelemetry Distro |
| "Availability tests can run every 1 minute" | ❌ Standard (URL ping) tests run at **5, 10, or 15 minute** intervals, not 1 minute. For higher frequency, use custom TrackAvailability with a Timer trigger |

---

## Study Tips for AZ-204 Exam

1. **Know telemetry types** - Request, dependency, exception, trace, event, metric — and what's auto-collected vs manual
2. **Understand TelemetryClient methods** - TrackEvent, TrackException, TrackDependency, TrackTrace, TrackMetric, GetMetric, Flush
3. **Master sampling concepts** - Adaptive vs fixed-rate vs ingestion; know that GetMetric() is NOT affected by sampling
4. **Learn KQL deeply** - Common tables, join, let, make-series, render, `app()`, `workspace()` functions
5. **Understand correlation** - `operation_Id`, distributed tracing, W3C Trace Context, Transaction Diagnostics
6. **Know instrumentation options** - Classic SDK vs OpenTelemetry Distro vs auto-instrumentation; connection string priority
7. **Remember key features** - Application Map, Live Metrics, Availability Tests, Snapshot Debugger, Profiler
8. **Distinguish Initializer vs Processor** - `ITelemetryInitializer` adds/modifies properties; `ITelemetryProcessor` filters/excludes
9. **Know cost management** - Sampling > daily cap; retention pricing; Diagnostic Settings for export
10. **Understand alerts** - Metric (static/dynamic) vs Log vs Smart Detection; Action Groups; Alert Processing Rules
11. **Know security** - RBAC roles (Monitoring Reader/Contributor), API Keys, PII handling, IP anonymization, AMPLS/Private Link
12. **Pre-aggregated vs log-based metrics** - GetMetric() = efficient & sampling-proof; TrackMetric() = log-based & sampled
13. **Usage Analytics** - Users, Sessions, Funnels, Cohorts, Impact, Retention, User Flows
14. **Channels** - ServerTelemetryChannel (production) vs InMemoryChannel (dev/console)
15. **Azure Functions integration** - host.json config, samplingSettings, excludedTypes, maxTelemetryItemsPerSecond

---

## Quick Reference Summary

### Telemetry Types
- **Request**: Incoming HTTP requests (auto-collected)
- **Dependency**: Outgoing calls — HTTP, SQL, Azure services (auto-collected)
- **Exception**: Errors and stack traces (auto-collected)
- **Trace**: Log messages / ILogger output (partial auto)
- **Event**: Custom business events (manual — `TrackEvent`)
- **Metric**: Custom measurements (manual — `TrackMetric` or `GetMetric`)
- **PageView**: Browser page loads (JS SDK auto)
- **Availability**: Test results (auto from availability tests)

### Key SDK Classes
- **TelemetryClient**: Main class for tracking
- **ITelemetryInitializer**: Add/modify properties on ALL telemetry
- **ITelemetryProcessor**: Filter/exclude telemetry (chain of responsibility)
- **ServerTelemetryChannel**: Disk-backed channel with retry (production)
- **InMemoryChannel**: In-memory only, no retry (development)

### Important Methods
```csharp
TrackEvent()                          // Custom business events
TrackException()                      // Exceptions with properties
TrackDependency()                     // External calls (manual)
TrackTrace()                          // Log messages
TrackMetric()                         // Log-based metric (affected by sampling)
GetMetric("name").TrackValue(value)   // Pre-aggregated metric (NOT affected by sampling)
TrackRequest()                        // HTTP requests (usually auto)
TrackAvailability()                   // Custom availability tests
Flush()                               // Send pending telemetry (call before shutdown)
```

### OpenTelemetry Equivalents
```csharp
ActivitySource → replaces TelemetryClient operations
Meter / Counter / Histogram → replaces GetMetric / TrackMetric
ILogger → replaces TrackTrace
```

### Sampling Types
| Type | Where | Default | Notes |
|------|-------|---------|-------|
| Adaptive | Client SDK | ✅ Enabled | Adjusts rate automatically |
| Fixed-Rate | Client SDK | ❌ Manual | Constant percentage |
| Ingestion | Server-side | ❌ Manual | Applied after ingestion |

### Common Kusto Tables
- `requests` — HTTP requests
- `dependencies` — External calls
- `exceptions` — Errors
- `traces` — Log messages
- `customEvents` — TrackEvent data
- `customMetrics` — TrackMetric data
- `pageViews` — Browser pages
- `availabilityResults` — Availability tests
- `performanceCounters` — System metrics

### Alert Types
| Type | Trigger | Speed |
|------|---------|-------|
| Metric (Static) | Fixed threshold | ~1 min |
| Metric (Dynamic) | ML-learned baseline | ~1 min |
| Log | KQL query result | 5+ min |
| Smart Detection | Automatic anomaly | Minutes-hours |
| Activity Log | Azure management events | Minutes |

### Key Diagnostic Features
| Feature | Purpose | Use When |
|---------|---------|----------|
| **Snapshot Debugger** | Production exception debugging | "Why did this crash?" |
| **Profiler** | Hot path / performance analysis | "Why is this slow?" |
| **Transaction Diagnostics** | End-to-end request view | "What happened in this request?" |
| **Application Map** | Service topology | "Which services depend on what?" |
| **Live Metrics** | Real-time telemetry | "What's happening right now?" |

### Correlation
- **operation_Id**: Links all telemetry for a transaction
- **parent_Id**: Links to calling operation
- **W3C Trace Context**: Standard for distributed tracing (traceparent header)

### Data Management
| Setting | Default | Range |
|---------|---------|-------|
| Retention | 90 days | 30-730 days |
| Daily cap | None (unlimited) | Configurable in GB |
| Free tier | 5 GB/month | Per billing account |
| Sampling default | Adaptive ON | 5 items/sec target |

### RBAC Roles
- **Monitoring Reader**: Read-only access to monitoring data
- **Monitoring Contributor**: Read + manage alerts, action groups
- **App Insights Component Contributor**: Full management of resources

### Connection String Priority
1. Code (`options.ConnectionString`)
2. Environment variable (`APPLICATIONINSIGHTS_CONNECTION_STRING`)
3. Config file (`appsettings.json`)

