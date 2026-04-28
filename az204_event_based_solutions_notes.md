# AZ-204: Develop Event-Based Solutions - Complete Study Notes

## Learning Path Overview
This learning path covers developing event-based solutions in Azure with 2 modules:
1. Explore Azure Event Grid
2. Explore Azure Event Hubs

**Level:** Intermediate | **Products:** Azure Event Grid, Azure Event Hubs | **Role:** Developer

---

## MODULE 1: EXPLORE AZURE EVENT GRID

### 1.1 Introduction to Azure Event Grid

**What is Azure Event Grid?**
- Fully managed event routing service
- Enables event-driven, reactive programming
- Uses publish-subscribe model
- Supports near real-time event delivery
- Massively scalable (millions of events per second)

**Key Benefits:**
- **Simplicity**: Easy to connect sources to handlers
- **Advanced filtering**: Filter events by type, subject, data
- **Fan-out**: Multiple endpoints can subscribe to same events
- **Reliability**: 24-hour retry with exponential backoff
- **Pay-per-event**: No idle costs, pay only for events
- **High throughput**: Millions of events per second

**Use Cases:**
- Serverless application architectures
- Operational automation
- Application integration
- IoT event processing
- Real-time notifications
- State change reactions

### 1.2 Event Grid Concepts

**Core Components:**

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Azure Event Grid                              │
├───────────────┬───────────────────────────┬─────────────────────────┤
│    Sources    │         Topics            │       Handlers          │
│   (Publishers)│    (Event Channels)       │     (Subscribers)       │
├───────────────┼───────────────────────────┼─────────────────────────┤
│ • Azure Blob  │ • System Topics           │ • Azure Functions       │
│ • Resource    │   (Built-in Azure events) │ • Event Hubs            │
│   Groups      │                           │ • Service Bus           │
│ • Event Hubs  │ • Custom Topics           │ • Storage Queues        │
│ • IoT Hub     │   (Your own events)       │ • Webhooks              │
│ • Service Bus │                           │ • Logic Apps            │
│ • Custom Apps │ • Partner Topics          │ • Automation Runbooks   │
│               │   (Third-party SaaS)      │ • Hybrid Connections    │
└───────────────┴───────────────────────────┴─────────────────────────┘
```

#### Events
- What happened (smallest unit of information)
- Contains: source, time, type, unique ID, data
- Maximum size: 1 MB per event
- Events >64 KB charged in 64 KB increments

#### Event Sources (Publishers)
- Where the event originated
- Azure services, custom applications, SaaS providers

#### Topics
- Endpoint where events are sent
- Three types: System, Custom, Partner

#### Event Subscriptions
- Defines which events to route where
- Can filter events
- Routes events to handlers

#### Event Handlers
- Where events are sent
- React to and process events

### 1.3 Topic Types

**1. System Topics**
- Built-in topics for Azure services
- Automatically available
- Subscribe to Azure resource events
- Examples: Blob created, Resource deleted, IoT device connected

```bash
# Create system topic
az eventgrid system-topic create \
  --name mysystemtopic \
  --resource-group myRG \
  --source /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{account} \
  --topic-type Microsoft.Storage.StorageAccounts \
  --location eastus
```

**2. Custom Topics**
- Your own event endpoints
- Publish custom events from your applications
- Full control over event schema

```bash
# Create custom topic
az eventgrid topic create \
  --name mycustomtopic \
  --resource-group myRG \
  --location eastus

# Get topic endpoint
az eventgrid topic show \
  --name mycustomtopic \
  --resource-group myRG \
  --query "endpoint" --output tsv

# Get topic key
az eventgrid topic key list \
  --name mycustomtopic \
  --resource-group myRG \
  --query "key1" --output tsv
```

**3. Partner Topics**
- Events from third-party SaaS providers
- Auth0, SAP, Twilio, etc.
- Registered partner namespaces

**4. Domains**
- Manage large numbers of topics
- Single endpoint for thousands of topics
- Up to 100,000 topics per domain
- Good for multi-tenant scenarios

```bash
# Create event domain
az eventgrid domain create \
  --name mydomain \
  --resource-group myRG \
  --location eastus

# Create topic within domain
az eventgrid domain topic create \
  --domain-name mydomain \
  --name mytopic \
  --resource-group myRG
```

### 1.4 Event Schemas

**Event Grid Schema (Default):**

```json
[
  {
    "id": "unique-event-id",
    "topic": "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.EventGrid/topics/{topic}",
    "subject": "/myapp/vehicles/cars",
    "eventType": "MyApp.Vehicles.CarCreated",
    "eventTime": "2024-01-15T10:30:00.0000000Z",
    "data": {
      "make": "Toyota",
      "model": "Camry",
      "year": 2024
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

| Property | Required | Description |
|----------|----------|-------------|
| id | Yes | Unique event identifier |
| topic | No | Full resource path (set by Event Grid) |
| subject | Yes | Publisher-defined event path |
| eventType | Yes | Event type (e.g., Microsoft.Storage.BlobCreated) |
| eventTime | Yes | UTC time event was generated |
| data | No | Event-specific data |
| dataVersion | No | Schema version of data |
| metadataVersion | No | Schema version of metadata (set by Event Grid) |

**Cloud Events Schema (v1.0):**

```json
{
  "specversion": "1.0",
  "type": "MyApp.Vehicles.CarCreated",
  "source": "/myapp/vehicles",
  "id": "unique-event-id",
  "time": "2024-01-15T10:30:00.0000000Z",
  "datacontenttype": "application/json",
  "data": {
    "make": "Toyota",
    "model": "Camry"
  }
}
```

**Cloud Events Benefits:**
- CNCF standard
- Cross-platform compatibility
- Supported by multiple cloud providers

```bash
# Create topic with Cloud Events schema
az eventgrid topic create \
  --name mycustomtopic \
  --resource-group myRG \
  --location eastus \
  --input-schema cloudeventschemav1_0
```

### 1.5 Event Subscriptions

**Subscription Properties:**
- Event types to receive
- Endpoint for handler
- Event filtering
- Retry policy
- Dead-lettering

```bash
# Create subscription to custom topic
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.EventGrid/topics/{topic} \
  --endpoint https://myfunction.azurewebsites.net/api/EventHandler \
  --endpoint-type webhook

# Create subscription with Azure Function
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.EventGrid/topics/{topic} \
  --endpoint /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Web/sites/{functionapp}/functions/{function} \
  --endpoint-type azurefunction

# Create subscription with Event Hub
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.EventGrid/topics/{topic} \
  --endpoint /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/{namespace}/eventhubs/{eventhub} \
  --endpoint-type eventhub
```

### 1.6 Event Filtering

**Filter Types:**

#### 1. Event Type Filtering
```bash
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {resource-id} \
  --endpoint {endpoint} \
  --included-event-types Microsoft.Storage.BlobCreated Microsoft.Storage.BlobDeleted
```

#### 2. Subject Filtering
```bash
# Subject begins with
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {resource-id} \
  --endpoint {endpoint} \
  --subject-begins-with "/blobServices/default/containers/images/"

# Subject ends with
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {resource-id} \
  --endpoint {endpoint} \
  --subject-ends-with ".jpg"

# Both
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {resource-id} \
  --endpoint {endpoint} \
  --subject-begins-with "/blobServices/default/containers/images/" \
  --subject-ends-with ".jpg"
```

#### 3. Advanced Filtering
```bash
# Advanced filter with JSON
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {resource-id} \
  --endpoint {endpoint} \
  --advanced-filter data.color StringIn Red Blue Green
```

**Advanced Filter Operators:**

| Operator | Description |
|----------|-------------|
| NumberGreaterThan | Number comparison |
| NumberGreaterThanOrEquals | Number comparison |
| NumberLessThan | Number comparison |
| NumberLessThanOrEquals | Number comparison |
| NumberIn | Number in array |
| NumberNotIn | Number not in array |
| BoolEquals | Boolean comparison |
| StringContains | String contains value |
| StringBeginsWith | String starts with |
| StringEndsWith | String ends with |
| StringIn | String in array |
| StringNotIn | String not in array |
| IsNullOrUndefined | Value is null or undefined |
| IsNotNull | Value is not null |

**Advanced Filter Example:**
```json
{
  "filter": {
    "advancedFilters": [
      {
        "operatorType": "NumberGreaterThan",
        "key": "data.counter",
        "value": 5
      },
      {
        "operatorType": "StringIn",
        "key": "data.color",
        "values": ["red", "blue", "green"]
      }
    ]
  }
}
```

### 1.7 Event Delivery and Retry

**Delivery Guarantees:**
- At-least-once delivery
- Events may be delivered out of order
- Events may be delivered multiple times

**Retry Policy:**
- Exponential backoff
- Default: 24-hour expiration, 30 retry attempts
- Configurable retry count and time-to-live

```bash
# Custom retry policy
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {resource-id} \
  --endpoint {endpoint} \
  --max-delivery-attempts 10 \
  --event-ttl 1440  # Minutes (24 hours)
```

**Retry Schedule:**
- 10 seconds
- 30 seconds
- 1 minute
- 5 minutes
- 10 minutes
- 30 minutes
- 1 hour
- Then hourly for up to 24 hours

**Delivery Failure Responses:**
| HTTP Code | Behavior |
|-----------|----------|
| 400 Bad Request | Drop event (no retry) |
| 401 Unauthorized | Drop after 5 minutes |
| 403 Forbidden | Drop event (no retry) |
| 404 Not Found | Drop after 5 minutes |
| 408 Request Timeout | Retry with backoff |
| 413 Payload Too Large | Drop event (no retry) |
| 429 Too Many Requests | Retry with backoff |
| 503 Service Unavailable | Retry with backoff |
| 504 Gateway Timeout | Retry with backoff |

**Dead-Lettering:**
- Store undelivered events
- Requires Azure Storage account
- Enables event replay

```bash
# Enable dead-lettering
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {resource-id} \
  --endpoint {endpoint} \
  --deadletter-endpoint /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{account}/blobServices/default/containers/{container}
```

### 1.8 Event Handler Types

**Supported Handlers:**

| Handler Type | Description |
|--------------|-------------|
| Webhooks | HTTP endpoints (custom or Azure Functions) |
| Azure Functions | Native integration |
| Event Hubs | For high-throughput streaming |
| Service Bus Queues | Reliable messaging |
| Service Bus Topics | Pub/sub messaging |
| Storage Queues | Simple queuing |
| Logic Apps | Workflow automation |
| Automation Runbooks | Infrastructure automation |
| Hybrid Connections | On-premises endpoints |
| Partner Destinations | Third-party services |

**Webhook Validation:**
- Event Grid validates webhook endpoints
- Two methods: Synchronous and Asynchronous

**Synchronous Validation (Preferred):**
```csharp
// Respond with validation code in response body
if (eventType == "Microsoft.EventGrid.SubscriptionValidationEvent")
{
    var validationCode = data["validationCode"].ToString();
    return new OkObjectResult(new { validationResponse = validationCode });
}
```

**Asynchronous Validation:**
- Event Grid sends validation URL in event
- Manually GET the URL to validate
- Useful when you can't modify endpoint code

### 1.9 Publishing Events

**Publishing to Custom Topic:**

```csharp
using Azure;
using Azure.Messaging.EventGrid;

// Create client
var endpoint = new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events");
var credential = new AzureKeyCredential("topic-key");
var client = new EventGridPublisherClient(endpoint, credential);

// Create event (Event Grid schema)
var events = new List<EventGridEvent>
{
    new EventGridEvent(
        subject: "/myapp/vehicles/cars",
        eventType: "MyApp.Vehicles.CarCreated",
        dataVersion: "1.0",
        data: new { Make = "Toyota", Model = "Camry" })
};

// Publish
await client.SendEventsAsync(events);
```

**Publishing Cloud Events:**

```csharp
using Azure.Messaging.CloudEvent;

var client = new EventGridPublisherClient(
    new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events"),
    new AzureKeyCredential("topic-key"));

var cloudEvents = new List<CloudEvent>
{
    new CloudEvent(
        source: "/myapp/vehicles",
        type: "MyApp.Vehicles.CarCreated",
        jsonSerializableData: new { Make = "Toyota", Model = "Camry" })
};

await client.SendEventsAsync(cloudEvents);
```

**HTTP POST (REST):**

```bash
# Event Grid schema
curl -X POST \
  -H "aeg-sas-key: <topic-key>" \
  -H "Content-Type: application/json" \
  -d '[{
    "id": "'$(uuidgen)'",
    "eventType": "MyApp.Events.ItemCreated",
    "subject": "/items/1",
    "eventTime": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'",
    "data": {"itemId": "1", "name": "Item 1"},
    "dataVersion": "1.0"
  }]' \
  "https://mytopic.eastus-1.eventgrid.azure.net/api/events"
```

### 1.10 Security and Access Control

**Authentication Methods:**

#### 1. Access Keys
```bash
# Get keys
az eventgrid topic key list \
  --name mytopic \
  --resource-group myRG

# Regenerate key
az eventgrid topic key regenerate \
  --name mytopic \
  --resource-group myRG \
  --key-name key1
```

#### 2. SAS Tokens
```csharp
// Generate SAS token
var builder = new EventGridSasBuilder(
    new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events"),
    DateTimeOffset.UtcNow.AddHours(1));
var sasToken = builder.GenerateSas(new AzureKeyCredential("topic-key"));

// Use token
var client = new EventGridPublisherClient(
    new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events"),
    new AzureSasCredential(sasToken));
```

#### 3. Azure AD Authentication
```csharp
var client = new EventGridPublisherClient(
    new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events"),
    new DefaultAzureCredential());
```

**RBAC Roles:**

| Role | Description |
|------|-------------|
| EventGrid Contributor | Manage Event Grid resources |
| EventGrid Data Sender | Send events to topics |
| EventGrid EventSubscription Contributor | Manage subscriptions |
| EventGrid EventSubscription Reader | Read subscriptions |

```bash
# Assign data sender role
az role assignment create \
  --role "EventGrid Data Sender" \
  --assignee <principal-id> \
  --scope /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.EventGrid/topics/{topic}
```

**Webhook Security:**
- Validation handshake (automatic)
- Delivery with Azure AD token
- Custom header values

```bash
# Subscription with Azure AD auth to webhook
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {resource-id} \
  --endpoint {endpoint} \
  --azure-active-directory-tenant-id {tenant-id} \
  --azure-active-directory-application-id-or-uri {app-id}
```

---

## MODULE 2: EXPLORE AZURE EVENT HUBS

### 2.1 Introduction to Azure Event Hubs

**What is Azure Event Hubs?**
- Big data streaming platform
- Event ingestion service
- Receives and processes millions of events per second
- Low latency, highly scalable
- Apache Kafka compatible

**Key Benefits:**
- **Scalable**: Millions of events per second
- **Real-time & Batch**: Process events in real-time or batch
- **Capture**: Automatic data archival to storage
- **Kafka compatible**: Use existing Kafka applications
- **Partitioning**: Parallel processing
- **Geo-DR**: Cross-region disaster recovery

**Use Cases:**
- Application logging
- Analytics pipelines
- Live dashboards
- IoT telemetry
- Transaction processing
- Clickstream analysis
- Anomaly detection

### 2.2 Event Hubs Architecture

**Core Components:**

```
                    Event Hubs Namespace
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │                    Event Hub                              │  │
│   │  ┌────────────┐ ┌────────────┐ ┌────────────┐            │  │
│   │  │ Partition 0│ │ Partition 1│ │ Partition 2│  ...       │  │
│   │  │  ┌──────┐  │ │  ┌──────┐  │ │  ┌──────┐  │            │  │
│   │  │  │Event │  │ │  │Event │  │ │  │Event │  │            │  │
│   │  │  │Event │  │ │  │Event │  │ │  │Event │  │            │  │
│   │  │  │Event │  │ │  │Event │  │ │  │Event │  │            │  │
│   │  │  │ ...  │  │ │  │ ...  │  │ │  │ ...  │  │            │  │
│   │  │  └──────┘  │ │  └──────┘  │ │  └──────┘  │            │  │
│   │  └────────────┘ └────────────┘ └────────────┘            │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│   Consumer Groups:  $Default  │  analytics  │  dashboard        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
         │                                         │
         ▼                                         ▼
    Producers                                 Consumers
   (Send Events)                            (Receive Events)
```

**Components:**

| Component | Description |
|-----------|-------------|
| Namespace | Container for Event Hubs, management unit |
| Event Hub | Equivalent to Kafka topic |
| Partition | Ordered sequence of events, enables parallelism |
| Consumer Group | View of the event stream, enables multiple readers |
| Event | Data unit being transferred |
| Producer | Entity that sends events |
| Consumer | Entity that reads events |

### 2.3 Event Hubs Tiers

| Feature | Basic | Standard | Premium | Dedicated |
|---------|-------|----------|---------|-----------|
| Partitions | 32 | 32 | 100 | 1024 |
| Consumer Groups | 1 | 20 | 100 | 1000 |
| Retention | 1 day | 7 days | 90 days | 90 days |
| Capture | ❌ | ✅ | ✅ | ✅ |
| Kafka Support | ❌ | ✅ | ✅ | ✅ |
| Private Link | ❌ | ❌ | ✅ | ✅ |
| Zone Redundancy | ❌ | ✅ | ✅ | ✅ |
| Throughput | Shared | 1-40 TU | PU-based | CU-based |

**Throughput Units (TU) - Standard:**
- **Ingress**: 1 MB/sec or 1000 events/sec (per TU)
- **Egress**: 2 MB/sec or 4096 events/sec (per TU)
- Auto-inflate: Automatically scale TUs

**Processing Units (PU) - Premium:**
- More resources per unit
- Better isolation
- Higher limits

**Capacity Units (CU) - Dedicated:**
- Dedicated hardware
- No noisy neighbor
- Highest performance

### 2.4 Partitions

**What are Partitions?**
- Ordered sequence of events
- New events added to end (append-only log)
- Events retained for configured period
- Enable parallel processing

**Partition Key:**
- Determines which partition receives event
- Same key = Same partition = Ordered delivery
- No key = Round-robin distribution

```csharp
// Send with partition key (ordered)
var eventData = new EventData(Encoding.UTF8.GetBytes("Hello"));
await producer.SendAsync(new[] { eventData }, new SendEventOptions
{
    PartitionKey = "customer-123"
});

// Send to specific partition
await producer.SendAsync(new[] { eventData }, new SendEventOptions
{
    PartitionId = "0"
});

// Send without partition key (round-robin)
await producer.SendAsync(new[] { eventData });
```

**Partition Count:**
- Set at creation, cannot change (Basic/Standard)
- Premium allows partition count changes
- More partitions = More parallelism = More consumers
- Typically: 4-32 partitions

**Best Practices:**
- Use partition key for related events requiring order
- Don't use too many unique partition keys
- Plan partition count based on consumers

### 2.5 Consumer Groups

**What are Consumer Groups?**
- Independent view (state, position, offset) of event stream
- Allows multiple consumers to read same events
- Each group tracks its own position
- Default group: `$Default`

```
Event Hub Stream: [E1, E2, E3, E4, E5, E6, E7, E8, E9, E10]
                          │           │
Consumer Group A ─────────┘           │
Position: E3                          │
                                      │
Consumer Group B ─────────────────────┘
Position: E7
```

**Consumer Group Rules:**
- Maximum 5 concurrent readers per partition per group
- Each consumer reads from assigned partitions
- Load balancing distributes partitions among consumers

```csharp
// Read from specific consumer group
var consumerGroup = "my-consumer-group";
await using var consumer = new EventHubConsumerClient(
    consumerGroup,
    connectionString,
    eventHubName);
```

### 2.6 Event Retention

**Retention Period:**
- How long events are stored
- Basic: 1 day (fixed)
- Standard: 1-7 days
- Premium: 1-90 days
- Dedicated: 1-90 days

**Event Position:**
- Offset: Byte position in partition
- Sequence Number: Order number
- Enqueued Time: When event was stored

```bash
# Set retention period
az eventhubs eventhub update \
  --resource-group myRG \
  --namespace-name mynamespace \
  --name myeventhub \
  --message-retention 7
```

### 2.7 Event Hubs Capture

**What is Capture?**
- Automatically saves events to storage
- Azure Blob Storage or Azure Data Lake
- Avro format
- No code required
- Enables batch analytics

**Capture Flow:**

```
Producers → Event Hub → Automatic Capture → Blob Storage / Data Lake
                │
                └──→ Real-time Consumers
```

**Capture Settings:**
- **Time Window**: 1-15 minutes
- **Size Window**: 10-500 MB
- **Naming**: Customizable path/filename

**Default Path Format:**
```
{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}
```

**Custom Path Format:**
```
{Year}/{Month}/{Day}/{EventHub}/{PartitionId}/{Hour}-{Minute}-{Second}
```

**Enable Capture:**

```bash
# Enable capture to blob storage
az eventhubs eventhub update \
  --resource-group myRG \
  --namespace-name mynamespace \
  --name myeventhub \
  --enable-capture true \
  --capture-destination "name=EventHubArchive.AzureBlockBlob" \
  --storage-account /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{account} \
  --blob-container capture \
  --capture-interval 300 \
  --capture-size-limit 314572800
```

**Capture File Format (Avro):**
```json
{
  "type": "record",
  "name": "EventData",
  "namespace": "Microsoft.ServiceBus.Messaging",
  "fields": [
    {"name": "SequenceNumber", "type": "long"},
    {"name": "Offset", "type": "string"},
    {"name": "EnqueuedTimeUtc", "type": "string"},
    {"name": "SystemProperties", "type": {"type": "map", "values": "string"}},
    {"name": "Properties", "type": {"type": "map", "values": "string"}},
    {"name": "Body", "type": ["null", "bytes"]}
  ]
}
```

### 2.8 Event Hubs SDK for .NET

**Installation:**
```bash
dotnet add package Azure.Messaging.EventHubs
dotnet add package Azure.Messaging.EventHubs.Processor
```

**Producer - EventHubProducerClient:**

```csharp
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Producer;

// Create producer
await using var producer = new EventHubProducerClient(connectionString, eventHubName);

// Create batch
using EventDataBatch batch = await producer.CreateBatchAsync();

// Add events to batch
for (int i = 0; i < 100; i++)
{
    var eventData = new EventData(Encoding.UTF8.GetBytes($"Event {i}"));
    
    // Add custom properties
    eventData.Properties["EventType"] = "MyEventType";
    eventData.Properties["Priority"] = "High";
    
    if (!batch.TryAdd(eventData))
    {
        // Batch is full, send it
        await producer.SendAsync(batch);
        batch = await producer.CreateBatchAsync();
        batch.TryAdd(eventData);
    }
}

// Send remaining events
await producer.SendAsync(batch);
```

**Producer with Partition Key:**

```csharp
// Send to specific partition key
var options = new SendEventOptions { PartitionKey = "customer-123" };
using var batch = await producer.CreateBatchAsync(options);

foreach (var order in customerOrders)
{
    var eventData = new EventData(JsonSerializer.SerializeToUtf8Bytes(order));
    batch.TryAdd(eventData);
}

await producer.SendAsync(batch);
```

**Consumer - EventHubConsumerClient:**

```csharp
using Azure.Messaging.EventHubs.Consumer;

// Create consumer
await using var consumer = new EventHubConsumerClient(
    EventHubConsumerClient.DefaultConsumerGroupName,
    connectionString,
    eventHubName);

// Read events
await foreach (PartitionEvent partitionEvent in consumer.ReadEventsAsync())
{
    var data = Encoding.UTF8.GetString(partitionEvent.Data.Body.ToArray());
    Console.WriteLine($"Partition: {partitionEvent.Partition.PartitionId}");
    Console.WriteLine($"Data: {data}");
    Console.WriteLine($"Sequence: {partitionEvent.Data.SequenceNumber}");
    Console.WriteLine($"Offset: {partitionEvent.Data.Offset}");
}

// Read from specific partition
await foreach (PartitionEvent partitionEvent in consumer.ReadEventsFromPartitionAsync(
    "0",
    EventPosition.Earliest))
{
    // Process events from partition 0
}
```

**Event Position Options:**
```csharp
// Read from beginning
EventPosition.Earliest

// Read from end (new events only)
EventPosition.Latest

// Read from specific offset
EventPosition.FromOffset(12345)

// Read from specific sequence number
EventPosition.FromSequenceNumber(100)

// Read from specific time
EventPosition.FromEnqueuedTime(DateTimeOffset.UtcNow.AddHours(-1))
```

**Processor - EventProcessorClient (Recommended):**

```csharp
using Azure.Messaging.EventHubs.Processor;
using Azure.Storage.Blobs;

// Create blob container client for checkpointing
var blobClient = new BlobContainerClient(storageConnectionString, "checkpoints");

// Create processor
var processor = new EventProcessorClient(
    blobClient,
    consumerGroup,
    eventHubConnectionString,
    eventHubName);

// Register handlers
processor.ProcessEventAsync += ProcessEventHandler;
processor.ProcessErrorAsync += ProcessErrorHandler;

// Start processing
await processor.StartProcessingAsync();

// Wait for processing
await Task.Delay(TimeSpan.FromMinutes(5));

// Stop processing
await processor.StopProcessingAsync();

// Event handler
async Task ProcessEventHandler(ProcessEventArgs args)
{
    var data = Encoding.UTF8.GetString(args.Data.Body.ToArray());
    Console.WriteLine($"Received: {data}");
    
    // Checkpoint after processing
    await args.UpdateCheckpointAsync();
}

// Error handler
Task ProcessErrorHandler(ProcessErrorEventArgs args)
{
    Console.WriteLine($"Error: {args.Exception.Message}");
    Console.WriteLine($"Partition: {args.PartitionId}");
    return Task.CompletedTask;
}
```

### 2.9 Checkpointing

**What is Checkpointing?**
- Records position in event stream
- Enables resume after failure
- Stored in Azure Blob Storage
- Managed by EventProcessorClient

**Checkpoint Content:**
- Consumer group
- Partition ID
- Offset
- Sequence number

**Checkpoint Strategy:**
```csharp
// Checkpoint every event (slowest, most reliable)
async Task ProcessEventHandler(ProcessEventArgs args)
{
    await ProcessEvent(args.Data);
    await args.UpdateCheckpointAsync();
}

// Checkpoint every N events (balanced)
private int eventCount = 0;
async Task ProcessEventHandler(ProcessEventArgs args)
{
    await ProcessEvent(args.Data);
    eventCount++;
    
    if (eventCount % 100 == 0)
    {
        await args.UpdateCheckpointAsync();
    }
}

// Checkpoint based on time
private DateTimeOffset lastCheckpoint = DateTimeOffset.UtcNow;
async Task ProcessEventHandler(ProcessEventArgs args)
{
    await ProcessEvent(args.Data);
    
    if (DateTimeOffset.UtcNow - lastCheckpoint > TimeSpan.FromMinutes(1))
    {
        await args.UpdateCheckpointAsync();
        lastCheckpoint = DateTimeOffset.UtcNow;
    }
}
```

### 2.10 Security and Access Control

**Authentication Methods:**

#### 1. Connection Strings
```csharp
var connectionString = "Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=xxx";
var producer = new EventHubProducerClient(connectionString, "myeventhub");
```

#### 2. Shared Access Signatures (SAS)
```bash
# Create authorization rule
az eventhubs eventhub authorization-rule create \
  --resource-group myRG \
  --namespace-name mynamespace \
  --eventhub-name myeventhub \
  --name SendRule \
  --rights Send

az eventhubs eventhub authorization-rule create \
  --resource-group myRG \
  --namespace-name mynamespace \
  --eventhub-name myeventhub \
  --name ListenRule \
  --rights Listen
```

#### 3. Azure AD Authentication (Recommended)
```csharp
var credential = new DefaultAzureCredential();
var producer = new EventHubProducerClient(
    "mynamespace.servicebus.windows.net",
    "myeventhub",
    credential);
```

**RBAC Roles:**

| Role | Permissions |
|------|-------------|
| Azure Event Hubs Data Owner | Full data access |
| Azure Event Hubs Data Sender | Send events |
| Azure Event Hubs Data Receiver | Receive events |

```bash
# Assign sender role
az role assignment create \
  --role "Azure Event Hubs Data Sender" \
  --assignee <principal-id> \
  --scope /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/{namespace}/eventhubs/{eventhub}
```

### 2.11 Scaling Event Hubs

**Scaling Strategies:**

#### 1. Throughput Units (Standard)
```bash
# Update throughput units
az eventhubs namespace update \
  --resource-group myRG \
  --name mynamespace \
  --capacity 10

# Enable auto-inflate
az eventhubs namespace update \
  --resource-group myRG \
  --name mynamespace \
  --enable-auto-inflate true \
  --maximum-throughput-units 20
```

#### 2. Processing Units (Premium)
- Scale PUs for more processing power
- Better isolation

#### 3. Partitions
- More partitions = More parallel consumers
- Set at creation (Standard) or adjust later (Premium)

**Consumer Scaling:**
- One consumer per partition per consumer group
- Multiple consumers share partitions via load balancing
- EventProcessorClient handles distribution

### 2.12 Event Hubs and Kafka

**Kafka Compatibility:**
- Event Hubs exposes Kafka endpoint
- Use existing Kafka clients
- Standard tier and above

**Kafka Producer Example:**
```java
Properties props = new Properties();
props.put("bootstrap.servers", "mynamespace.servicebus.windows.net:9093");
props.put("security.protocol", "SASL_SSL");
props.put("sasl.mechanism", "PLAIN");
props.put("sasl.jaas.config", 
    "org.apache.kafka.common.security.plain.PlainLoginModule required " +
    "username=\"$ConnectionString\" " +
    "password=\"Endpoint=sb://...;SharedAccessKeyName=...;SharedAccessKey=...\";");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
producer.send(new ProducerRecord<>("myeventhub", "key", "value"));
```

---

## MODULE 3: EVENT GRID ADVANCED CONCEPTS

### 3.1 System Topic Event Types Reference

**Commonly Tested Azure Service Events:**

**Azure Storage:**

| Event Type | Description |
|-----------|-------------|
| `Microsoft.Storage.BlobCreated` | Blob created or replaced |
| `Microsoft.Storage.BlobDeleted` | Blob deleted |
| `Microsoft.Storage.BlobRenamed` | Blob renamed (Data Lake only) |
| `Microsoft.Storage.BlobTierChanged` | Blob access tier changed |
| `Microsoft.Storage.DirectoryCreated` | Directory created (Data Lake only) |
| `Microsoft.Storage.DirectoryDeleted` | Directory deleted (Data Lake only) |

**Azure Resource Groups:**

| Event Type | Description |
|-----------|-------------|
| `Microsoft.Resources.ResourceWriteSuccess` | Resource created or updated |
| `Microsoft.Resources.ResourceWriteFailure` | Resource create/update failed |
| `Microsoft.Resources.ResourceDeleteSuccess` | Resource deleted |
| `Microsoft.Resources.ResourceDeleteFailure` | Resource delete failed |
| `Microsoft.Resources.ResourceActionSuccess` | Resource action succeeded |

**Azure Subscriptions:**
- Same event types as Resource Groups, but at subscription scope

**Azure Container Registry:**

| Event Type | Description |
|-----------|-------------|
| `Microsoft.ContainerRegistry.ImagePushed` | Docker image pushed |
| `Microsoft.ContainerRegistry.ImageDeleted` | Docker image deleted |
| `Microsoft.ContainerRegistry.ChartPushed` | Helm chart pushed |
| `Microsoft.ContainerRegistry.ChartDeleted` | Helm chart deleted |

**Azure IoT Hub:**

| Event Type | Description |
|-----------|-------------|
| `Microsoft.Devices.DeviceCreated` | Device registered |
| `Microsoft.Devices.DeviceDeleted` | Device deleted |
| `Microsoft.Devices.DeviceConnected` | Device connected |
| `Microsoft.Devices.DeviceDisconnected` | Device disconnected |
| `Microsoft.Devices.DeviceTelemetry` | Telemetry message sent |

**Azure App Configuration:**

| Event Type | Description |
|-----------|-------------|
| `Microsoft.AppConfiguration.KeyValueModified` | Key-value pair modified |
| `Microsoft.AppConfiguration.KeyValueDeleted` | Key-value pair deleted |

**Azure Key Vault:**

| Event Type | Description |
|-----------|-------------|
| `Microsoft.KeyVault.SecretNewVersionCreated` | New secret version created |
| `Microsoft.KeyVault.SecretNearExpiry` | Secret near expiration |
| `Microsoft.KeyVault.SecretExpired` | Secret expired |
| `Microsoft.KeyVault.CertificateNewVersionCreated` | New certificate version |
| `Microsoft.KeyVault.CertificateNearExpiry` | Certificate near expiry |
| `Microsoft.KeyVault.CertificateExpired` | Certificate expired |

### 3.2 Webhook Validation Deep Dive

**Why Validation?**
- Prevents malicious actors from flooding your endpoint
- Event Grid validates ownership of the webhook endpoint before delivering events

**Method 1: Synchronous Handshake (Preferred)**

```
Event Grid → POST validation event → Your Endpoint
Your Endpoint → Return validationResponse in JSON body → Event Grid
Subscription Created ✅
```

**Full Implementation:**
```csharp
using Azure.Messaging.EventGrid;
using Azure.Messaging.EventGrid.SystemEvents;

[HttpPost("api/events")]
public async Task<IActionResult> HandleEvents()
{
    // Read the request body
    string requestContent;
    using (var reader = new StreamReader(Request.Body))
    {
        requestContent = await reader.ReadToEndAsync();
    }

    var events = EventGridEvent.ParseMany(BinaryData.FromString(requestContent));

    foreach (var eventGridEvent in events)
    {
        // Handle validation
        if (eventGridEvent.EventType == SystemEventNames.EventGridSubscriptionValidation)
        {
            var validationEventData = eventGridEvent.Data
                .ToObjectFromJson<SubscriptionValidationEventData>();
            
            // Return the validation code
            var response = new SubscriptionValidationResponse
            {
                ValidationResponse = validationEventData.ValidationCode
            };
            return new OkObjectResult(response);
        }

        // Handle regular events
        if (eventGridEvent.EventType == "MyApp.Items.Created")
        {
            // Process event
            var data = eventGridEvent.Data.ToObjectFromJson<MyItemData>();
            await ProcessItem(data);
        }
    }

    return Ok();
}
```

**Method 2: Asynchronous Handshake (Manual)**
- Event Grid sends a `validationUrl` in the validation event
- You make a GET request to that URL within **5 minutes**
- URL format: `https://<region>.eventgrid.azure.net:553/eventsubscriptions/{sub}/validate?...&validationCode=<code>`
- Used when you cannot modify endpoint code (third-party services)

```csharp
// Async validation — extract URL and call it manually
if (eventGridEvent.EventType == SystemEventNames.EventGridSubscriptionValidation)
{
    var validationData = eventGridEvent.Data
        .ToObjectFromJson<SubscriptionValidationEventData>();
    
    // Option 1: Return code synchronously (preferred)
    // Option 2: GET the validationUrl manually within 5 minutes
    string validationUrl = validationData.ValidationUrl;
    
    // Call this URL with HTTP GET to validate
    using var httpClient = new HttpClient();
    await httpClient.GetAsync(validationUrl);
}
```

**Azure Functions Auto-Validation:**
- Azure Functions with Event Grid trigger handle validation AUTOMATICALLY
- No code needed — the runtime handles the handshake
- This is the simplest approach

### 3.3 Event Delivery Properties (Custom Headers)

**Add custom headers to delivered events:**
```bash
# Add custom delivery properties (static)
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {resource-id} \
  --endpoint {endpoint} \
  --delivery-attribute-mapping "header1" static "myvalue" false \
  --delivery-attribute-mapping "header2" dynamic data.myProperty

# Static: fixed value
# Dynamic: value from event data property
```

**Delivery Attribute Types:**

| Type | Description | Example |
|------|-------------|---------|
| `static` | Fixed value | `--delivery-attribute-mapping "X-Custom" static "value123" false` |
| `dynamic` | From event data | `--delivery-attribute-mapping "X-ItemId" dynamic data.itemId` |

**Secret flag:** The last boolean parameter — `true` means the value is a secret (won't show in logs/queries).

### 3.4 Event Grid Networking & Security

**IP Firewall:**
```bash
# Configure IP firewall on topic
az eventgrid topic update \
  --name mytopic \
  --resource-group myRG \
  --public-network-access enabled \
  --inbound-ip-rules "10.0.0.0/8" allow \
  --inbound-ip-rules "203.0.113.0/24" allow
```

**Private Endpoints:**
```bash
# Create private endpoint for Event Grid topic
az network private-endpoint create \
  --name myPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet mySubnet \
  --private-connection-resource-id <topic-resource-id> \
  --group-id topic \
  --connection-name myConnection
```

**Managed Identity for Event Delivery:**
```bash
# Enable system-assigned identity on topic
az eventgrid topic update \
  --name mytopic \
  --resource-group myRG \
  --identity systemassigned

# Create subscription with managed identity delivery
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {topic-resource-id} \
  --endpoint-type servicebusqueue \
  --endpoint {service-bus-queue-resource-id} \
  --delivery-identity-endpoint-type servicebusqueue \
  --delivery-identity systemassigned
```

### 3.5 Event Grid Domains (Multi-Tenant)

**When to Use Domains:**
- Multi-tenant applications
- Thousands of topics
- Single management endpoint
- Per-tenant event routing

**Architecture:**
```
Event Grid Domain (1 endpoint, 1 set of keys)
├── Topic: tenant-a      → Subscription → Tenant A's handler
├── Topic: tenant-b      → Subscription → Tenant B's handler  
├── Topic: tenant-c      → Subscription → Tenant C's handler
└── ... up to 100,000 topics
```

```bash
# Create domain
az eventgrid domain create --name mydomain --resource-group myRG --location eastus

# Publish to domain topic
# POST to: https://{domain-endpoint}/api/events
# Include "topic" property in the event to route to specific domain topic
```

**Publishing to Domain:**
```csharp
var events = new List<EventGridEvent>
{
    new EventGridEvent(
        subject: "/orders/123",
        eventType: "Order.Created",
        dataVersion: "1.0",
        data: new { OrderId = 123, TenantId = "tenant-a" })
    {
        Topic = "tenant-a"  // Routes to "tenant-a" topic within domain
    }
};

await client.SendEventsAsync(events);
```

### 3.6 Custom Input Schema Mapping

**Map custom event format to Event Grid schema:**

```bash
# Create topic with custom input schema
az eventgrid topic create \
  --name mytopic \
  --resource-group myRG \
  --location eastus \
  --input-schema customeventschema \
  --input-mapping-fields \
    id=myId \
    topic=myTopic \
    eventTime=myEventTime \
    eventType=myEventType \
    subject=mySubject \
    dataVersion=myDataVersion \
  --input-mapping-default-values \
    dataVersion=1.0 \
    subject="default-subject"
```

**Use case:** When your event producer uses a different schema and you don't want to change it.

### 3.7 Output Batching

**Batch delivery to Event Hubs and Storage Queue handlers:**

```bash
# Configure batched delivery
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id {resource-id} \
  --endpoint-type eventhub \
  --endpoint {eventhub-resource-id} \
  --max-events-per-batch 10 \
  --preferred-batch-size-in-kilobytes 64
```

**Note:** Batching is best-effort. May deliver fewer events per batch. Default is 1 event per batch.

### 3.8 Event Grid Quotas & Limits

| Resource | Limit |
|----------|-------|
| Custom topics per subscription | 100 |
| Event subscriptions per topic | 500 |
| Publish rate per topic | 5,000 events/sec |
| Event size | 1 MB max |
| Events >64 KB | Charged in 64 KB increments |
| Advanced filter values | 25 per filter |
| Advanced filters per subscription | 25 |
| Domain topics | 100,000 per domain |
| Event subscriptions per domain topic | 500 |
| Domain scope subscriptions | 50 |

### 3.9 Event Grid Retry Behavior — Drop vs Retry

**Events that are DROPPED (no retry):**

| HTTP Status | Reason |
|-------------|--------|
| 400 Bad Request | Malformed request, won't succeed on retry |
| 401 Unauthorized | Dropped after 5-minute delay |
| 403 Forbidden | Permanent access denial |
| 404 Not Found | Dropped after 5-minute delay |
| 413 Payload Too Large | Event too big for endpoint |

**Events that are RETRIED:**

| HTTP Status | Behavior |
|-------------|----------|
| 408 Request Timeout | Retry with exponential backoff |
| 429 Too Many Requests | Retry after delay (respects Retry-After header) |
| 503 Service Unavailable | Retry with exponential backoff |
| 504 Gateway Timeout | Retry with exponential backoff |

**Important:** For 401 and 404, if the issue persists beyond 5 minutes, the event is dead-lettered (if configured) or dropped.

---

## MODULE 4: EVENT HUBS ADVANCED CONCEPTS

### 4.1 EventProcessorClient Deep Dive

**Load Balancing:**
- Distributes partitions evenly across processor instances
- Uses **ownership** stored in Azure Blob Storage
- Automatically rebalances when processors join/leave
- Each partition owned by exactly ONE processor at a time

**Ownership Model:**
```
Event Hub: 8 partitions
Processor A: owns partitions 0, 1, 2, 3
Processor B: owns partitions 4, 5, 6, 7

Processor C joins:
Processor A: owns partitions 0, 1, 2
Processor B: owns partitions 3, 4, 5
Processor C: owns partitions 6, 7
```

**Important Configuration:**
```csharp
var processor = new EventProcessorClient(
    blobClient,
    consumerGroup,
    connectionString,
    eventHubName,
    new EventProcessorClientOptions
    {
        // How often to claim/release partitions
        LoadBalancingUpdateInterval = TimeSpan.FromSeconds(10),
        
        // How long before a partition ownership expires
        PartitionOwnershipExpirationInterval = TimeSpan.FromSeconds(30),
        
        // Load balancing strategy
        LoadBalancingStrategy = LoadBalancingStrategy.Greedy, // or Balanced
        
        // Prefetch count for performance
        PrefetchCount = 300,
        
        // Max events per batch
        MaximumWaitTime = TimeSpan.FromSeconds(5)
    });
```

**LoadBalancingStrategy:**
- **Balanced** (default): Claims one partition at a time, gradual rebalancing
- **Greedy**: Claims as many partitions as needed immediately

### 4.2 Idempotent Processing Patterns

**Why Idempotency Matters:**
- Event Hubs guarantees **at-least-once** delivery
- Events MAY be delivered more than once
- Your handler must handle duplicates

**Pattern 1: Sequence Number Tracking**
```csharp
async Task ProcessEventHandler(ProcessEventArgs args)
{
    var sequenceNumber = args.Data.SequenceNumber;
    var partitionId = args.Partition.PartitionId;
    
    // Check if already processed
    var key = $"{partitionId}:{sequenceNumber}";
    if (await _cache.ExistsAsync(key))
    {
        // Already processed, skip
        return;
    }
    
    // Process the event
    await ProcessEvent(args.Data);
    
    // Mark as processed
    await _cache.SetAsync(key, "processed", TimeSpan.FromHours(24));
    
    // Checkpoint
    await args.UpdateCheckpointAsync();
}
```

**Pattern 2: Business Key Deduplication**
```csharp
async Task ProcessEventHandler(ProcessEventArgs args)
{
    var order = JsonSerializer.Deserialize<Order>(args.Data.Body.Span);
    
    // Use business key for deduplication
    if (await _repository.ExistsAsync(order.OrderId))
    {
        return; // Already processed
    }
    
    await _repository.SaveAsync(order);
    await args.UpdateCheckpointAsync();
}
```

### 4.3 Schema Registry

**What is Schema Registry?**
- Central repository for event schemas
- Schema validation at producer/consumer
- Schema evolution management
- Supports **Avro** schemas
- Part of Event Hubs namespace (Standard+)

**Benefits:**
- Producers serialize with registered schema
- Consumers deserialize with schema from registry
- Prevents breaking changes
- Schema versioning and compatibility

```csharp
using Azure.Data.SchemaRegistry;
using Microsoft.Azure.Data.SchemaRegistry.ApacheAvro;

// Create schema registry client
var schemaRegistryClient = new SchemaRegistryClient(
    "mynamespace.servicebus.windows.net",
    new DefaultAzureCredential());

// Create Avro serializer
var serializer = new SchemaRegistryAvroSerializer(
    schemaRegistryClient,
    "my-schema-group",
    new SchemaRegistryAvroSerializerOptions
    {
        AutoRegisterSchemas = true
    });

// Serialize with schema
var order = new Order { OrderId = 123, Amount = 99.99m };
var eventData = await serializer.SerializeAsync<EventData, Order>(order);

// Send
await producer.SendAsync(new[] { eventData });

// Deserialize with schema
var deserializedOrder = await serializer.DeserializeAsync<Order>(receivedEventData);
```

**Schema Compatibility Modes:**
- **None**: No compatibility checking
- **Backward**: New schema can read old data
- **Forward**: Old schema can read new data
- **Full**: Both backward and forward compatible

### 4.4 Geo-Disaster Recovery

**What is Geo-DR?**
- Paired namespaces in different regions
- Automatic metadata replication
- Manual failover
- **Does NOT replicate event data** (metadata only!)

```bash
# Create alias (pairing)
az eventhubs georecovery-alias set \
  --resource-group myRG \
  --namespace-name primary-namespace \
  --alias my-alias \
  --partner-namespace /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/secondary-namespace

# Initiate failover
az eventhubs georecovery-alias fail-over \
  --resource-group myRG \
  --namespace-name secondary-namespace \
  --alias my-alias

# Break pairing
az eventhubs georecovery-alias break-pair \
  --resource-group myRG \
  --namespace-name primary-namespace \
  --alias my-alias
```

**Key Points:**
- Only **metadata** is replicated (namespace, Event Hub, consumer group config)
- **Event data is NOT replicated** — active events remain in primary
- Use **Capture** if you need event data backup
- Alias connection string works for both primary and secondary
- Failover is **manual** (not automatic)

### 4.5 Auto-Inflate (Standard Tier)

**What is Auto-Inflate?**
- Automatically increases throughput units (TUs) based on load
- Prevents throttling during traffic spikes
- Only scales UP, never DOWN
- Specify maximum TU limit

```bash
# Enable auto-inflate
az eventhubs namespace update \
  --resource-group myRG \
  --name mynamespace \
  --enable-auto-inflate true \
  --maximum-throughput-units 20

# Check current settings
az eventhubs namespace show \
  --resource-group myRG \
  --name mynamespace \
  --query '{autoInflate: isAutoInflateEnabled, maxTU: maximumThroughputUnits}'
```

**Important:** Auto-inflate only scales UP. To scale down, you must manually reduce TUs. Unused TUs still incur charges.

### 4.6 Event Hubs Networking & Security

**IP Firewall:**
```bash
# Set IP rules
az eventhubs namespace network-rule-set update \
  --resource-group myRG \
  --namespace-name mynamespace \
  --default-action Deny \
  --ip-rules '[{"ipMask":"10.0.0.0/8","action":"Allow"}]'
```

**VNet Service Endpoints:**
```bash
# Add VNet rule
az eventhubs namespace network-rule-set update \
  --resource-group myRG \
  --namespace-name mynamespace \
  --default-action Deny \
  --virtual-network-rules '[{"subnet":{"id":"/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/{vnet}/subnets/{subnet}"},"ignoreMissingVnetServiceEndpoint":false}]'
```

**Private Endpoints:**
```bash
# Create private endpoint for Event Hubs
az network private-endpoint create \
  --name myPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet mySubnet \
  --private-connection-resource-id <namespace-resource-id> \
  --group-id namespace \
  --connection-name myConnection
```

### 4.7 Event Hubs with Azure Functions

**EventHubTrigger:**
```csharp
// Single event processing
[Function("EventHubProcessor")]
public async Task Run(
    [EventHubTrigger("myeventhub", Connection = "EventHubConnection")] string message,
    FunctionContext context)
{
    var logger = context.GetLogger("EventHubProcessor");
    logger.LogInformation($"Event: {message}");
}

// Batch processing (recommended for throughput)
[Function("EventHubBatchProcessor")]
public async Task RunBatch(
    [EventHubTrigger("myeventhub", Connection = "EventHubConnection", 
        IsBatched = true)] EventData[] events,
    FunctionContext context)
{
    var logger = context.GetLogger("EventHubBatchProcessor");
    
    foreach (var eventData in events)
    {
        logger.LogInformation($"Event: {Encoding.UTF8.GetString(eventData.Body.Span)}");
        logger.LogInformation($"Partition: {eventData.PartitionKey}");
        logger.LogInformation($"Sequence: {eventData.SequenceNumber}");
    }
}
```

**EventHub Output Binding:**
```csharp
[Function("SendToEventHub")]
[EventHubOutput("myeventhub", Connection = "EventHubConnection")]
public string Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req)
{
    return "{\"message\": \"Hello Event Hub\"}";
}
```

**host.json Configuration for Event Hub Trigger:**
```json
{
  "extensions": {
    "eventHubs": {
      "maxEventBatchSize": 100,
      "minEventBatchSize": 1,
      "maxWaitTime": "00:00:05",
      "prefetchCount": 300,
      "transportType": "amqpWebSockets",
      "clientRetryOptions": {
        "mode": "Exponential",
        "tryTimeout": "00:01:00",
        "delay": "00:00:00.80",
        "maxDelay": "00:01:00",
        "maxRetries": 3
      }
    }
  }
}
```

### 4.8 Event Hubs Quotas & Limits

| Resource | Basic | Standard | Premium | Dedicated |
|----------|-------|----------|---------|-----------|
| **Namespaces per subscription** | 100 | 100 | 100 | 100 |
| **Event Hubs per namespace** | 10 | 10 | 100 | 1000 |
| **Partitions per Event Hub** | 32 | 32 | 100 | 1024 |
| **Consumer groups per Event Hub** | 1 | 20 | 100 | 1000 |
| **Max event size** | 256 KB | 1 MB | 1 MB | 1 MB |
| **Brokered connections** | 100 | 5,000 | — | — |
| **Retention** | 1 day | 7 days | 90 days | 90 days |
| **Throughput** | — | 1-40 TU | 1-16 PU | 1-20 CU |
| **Ingress per TU** | — | 1 MB/s | — | — |
| **Egress per TU** | — | 2 MB/s | — | — |

**Throttling:** When you exceed TU limits, Event Hubs returns `ServerBusyException`. Use auto-inflate or increase TUs.

**Max Event Size:**
- **Basic**: 256 KB
- **Standard/Premium/Dedicated**: 1 MB
- This is a CRITICAL exam gotcha!

### 4.9 Buffered Producer (High Throughput)

**For applications sending many events:**
```csharp
using Azure.Messaging.EventHubs.Producer;

await using var producer = new EventHubBufferedProducerClient(
    connectionString, eventHubName);

// Register handlers
producer.SendEventBatchSucceededAsync += args =>
{
    Console.WriteLine($"Published {args.EventBatch.Count} events to partition {args.PartitionId}");
    return Task.CompletedTask;
};

producer.SendEventBatchFailedAsync += args =>
{
    Console.WriteLine($"Failed to publish: {args.Exception.Message}");
    return Task.CompletedTask;
};

// Enqueue events (automatically batched and sent)
for (int i = 0; i < 10000; i++)
{
    await producer.EnqueueEventAsync(new EventData($"Event {i}"));
}

// Flush remaining
await producer.FlushAsync();
```

**EventHubBufferedProducerClient vs EventHubProducerClient:**

| Feature | EventHubProducerClient | EventHubBufferedProducerClient |
|---------|------------------------|--------------------------------|
| Batching | Manual (CreateBatchAsync) | Automatic |
| Partition assignment | Manual or automatic | Automatic |
| Best for | Control, small volume | High throughput |
| Error handling | Try/catch | Event handlers |
| Ordering guarantee | Per batch | Per partition key |

---

## MODULE 5: MESSAGING COMPARISON & ARCHITECTURE PATTERNS

### 5.1 Event Grid vs Event Hubs vs Service Bus — The BIG Comparison

| Aspect | Event Grid | Event Hubs | Service Bus |
|--------|------------|------------|-------------|
| **Purpose** | Event distribution/routing | Event streaming/ingestion | Enterprise messaging |
| **Model** | Publish-Subscribe (push) | Publish-Subscribe (pull) | Queue + Topic (pull) |
| **Delivery** | Push to subscribers | Consumer pulls events | Consumer pulls messages |
| **Protocol** | HTTP/HTTPS | AMQP, HTTPS, Kafka | AMQP, HTTPS |
| **Ordering** | ❌ Not guaranteed | ✅ Per partition | ✅ FIFO (with sessions) |
| **Retention** | 24h retry period | 1-90 days | Until consumed |
| **Volume** | Millions events/sec | Millions events/sec | Moderate throughput |
| **Event size** | 1 MB | 256 KB (Basic) / 1 MB | 256 KB / 100 MB (Premium) |
| **Deduplication** | ❌ No | ❌ No (consumer handles) | ✅ Built-in |
| **Transactions** | ❌ No | ❌ No | ✅ Yes |
| **Sessions** | ❌ No | ❌ No | ✅ Yes |
| **Dead-lettering** | ✅ Yes (Storage) | ❌ No (use Capture) | ✅ Yes (built-in) |
| **Scheduled delivery** | ❌ No | ❌ No | ✅ Yes |
| **Filtering** | ✅ Advanced | ❌ No (consumer-side) | ✅ SQL filters on topics |
| **Consumption** | Reactive/push | Streaming/pull | Queue/pull |
| **Replay** | ❌ No | ✅ Yes (offset-based) | ❌ No (once consumed) |
| **Pricing** | Per event | Per throughput unit | Per operations |

### 5.2 When to Use Which Service

**Use Event Grid When:**
- ✅ Reacting to state changes in Azure resources (blob created, resource deleted)
- ✅ Serverless event-driven architectures
- ✅ Fan-out: one event → many subscribers
- ✅ Low-latency event notifications
- ✅ You need server-side filtering (event type, subject, advanced)
- ✅ Pay-per-event pricing model
- ❌ NOT for high-volume data streaming
- ❌ NOT when you need message ordering

**Use Event Hubs When:**
- ✅ High-volume event streaming (telemetry, logs, clickstreams)
- ✅ Real-time analytics pipelines
- ✅ Need to replay/reprocess events
- ✅ Multiple consumer groups reading same stream
- ✅ Time-series data ingestion
- ✅ Kafka compatibility needed
- ✅ Event archival with Capture
- ❌ NOT for request-reply patterns
- ❌ NOT when you need per-message transactions

**Use Service Bus When:**
- ✅ Reliable, ordered message delivery
- ✅ Transactional processing (financial, orders)
- ✅ Request-reply messaging
- ✅ Duplicate detection needed
- ✅ Message sessions (group related messages)
- ✅ Scheduled message delivery
- ✅ Dead-letter queue with inspection
- ✅ Long-running workflows
- ❌ NOT for high-volume streaming
- ❌ NOT for event broadcasting to many subscribers

### 5.3 Azure Queue Storage vs Service Bus Queue

| Feature | Queue Storage | Service Bus Queue |
|---------|---------------|-------------------|
| **Max message size** | 64 KB | 256 KB (Standard) / 100 MB (Premium) |
| **Max queue size** | 500 TB | 1-80 GB |
| **Delivery guarantee** | At-least-once | At-least-once or At-most-once (PeekLock/ReceiveAndDelete) |
| **Ordering** | FIFO (best-effort) | FIFO (guaranteed with sessions) |
| **Duplicate detection** | ❌ No | ✅ Yes |
| **Transactions** | ❌ No | ✅ Yes |
| **Dead-lettering** | ❌ No | ✅ Yes |
| **Sessions** | ❌ No | ✅ Yes |
| **Lease/Lock** | Visibility timeout | PeekLock |
| **Receive modes** | Polling | Long polling (AMQP) |
| **Batch receive** | ✅ Yes (32 max) | ✅ Yes |
| **Scheduled delivery** | ❌ No | ✅ Yes |
| **Auto-forwarding** | ❌ No | ✅ Yes |
| **Pricing** | Very low (per transaction) | Higher (per operations) |
| **Server-side log** | ✅ Yes | ❌ No |
| **Use when** | Simple queuing, >80 GB, audit trail | Enterprise messaging, reliability |

### 5.4 Event-Driven Architecture Patterns

**Pattern 1: Simple Event Notification**
```
Azure Storage (Blob Created) → Event Grid → Azure Function (process)
```
- Simplest pattern
- Loosely coupled
- Event Grid handles routing

**Pattern 2: Event Streaming Pipeline**
```
IoT Devices → Event Hubs → Stream Analytics → Power BI
                    └──→ Capture → Data Lake → Batch Analytics
```
- High throughput
- Real-time + batch processing
- Event replay capability

**Pattern 3: Competing Consumers**
```
Producers → Service Bus Queue → Consumer 1
                              → Consumer 2 (load balanced)
                              → Consumer 3
```
- Load distribution
- Each message processed once
- Scale consumers independently

**Pattern 4: Event Grid Fan-Out**
```
                         ┌→ Azure Function A (email)
Storage Blob Created → Event Grid ─→ Azure Function B (thumbnail)
                         └→ Event Hub (analytics)
```
- One event, multiple handlers
- Each handler gets a copy
- Parallel processing

**Pattern 5: Saga / Choreography**
```
Order Service → Event Grid → Payment Service
Payment Service → Event Grid → Shipping Service
Shipping Service → Event Grid → Notification Service
```
- No central coordinator
- Each service reacts to events
- Loosely coupled

**Pattern 6: Event Sourcing with Event Hubs**
```
Commands → Event Hub (append-only log)
                 ↓
         Consumer Group A → Read model (current state)
         Consumer Group B → Analytics
         Consumer Group C → Audit log
```
- Store events as source of truth
- Rebuild state from events
- Multiple projections via consumer groups

### 5.5 Integration Patterns (Event Grid + Event Hubs)

**Common: Event Grid → Event Hubs**
```bash
# Route Event Grid events to Event Hub for streaming
az eventgrid event-subscription create \
  --name grid-to-hub \
  --source-resource-id {storage-account-resource-id} \
  --endpoint-type eventhub \
  --endpoint {eventhub-resource-id} \
  --included-event-types Microsoft.Storage.BlobCreated
```

**Use case:** Storage events need both real-time processing AND long-term analytics.

**Common: Event Grid → Service Bus**
```bash
# Route Event Grid events to Service Bus for reliable processing
az eventgrid event-subscription create \
  --name grid-to-bus \
  --source-resource-id {storage-account-resource-id} \
  --endpoint-type servicebusqueue \
  --endpoint {queue-resource-id}
```

**Use case:** Storage events need reliable, ordered, transactional processing.

---

## EVENT GRID VS EVENT HUBS COMPARISON (Expanded)

| Aspect | Event Grid | Event Hubs |
|--------|------------|------------|
| **Purpose** | Event routing/distribution | Event streaming/ingestion |
| **Volume** | Millions/sec (discrete events) | Millions/sec (high-throughput stream) |
| **Retention** | 24 hours retry | 1-90 days |
| **Order** | No guarantee | Per partition |
| **Consumer Model** | Push (webhooks) | Pull (consumer reads) |
| **Use Case** | React to events | Process event streams |
| **Protocol** | HTTP | AMQP, HTTPS, Kafka |
| **Pricing** | Per event | Per throughput unit |
| **Replay** | ❌ No | ✅ Yes (rewind offset) |
| **Filtering** | ✅ Server-side (subject, type, advanced) | ❌ Consumer-side only |
| **Schema** | Event Grid or Cloud Events | Custom (+ Schema Registry) |
| **Dead-letter** | ✅ To Blob Storage | ❌ Use Capture instead |
| **Max event size** | 1 MB | 256 KB (Basic) / 1 MB |
| **Networking** | Private endpoints, IP firewall | Private endpoints, VNet, IP firewall |

**When to Use:**
- **Event Grid**: Discrete events, reactive processing, Azure integration, fan-out
- **Event Hubs**: High-volume streaming, analytics, time-series data, event replay

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**Event Grid:**
- Topics (System, Custom, Domain, Partner)
- Event schemas (Event Grid, Cloud Events)
- Filtering (Event type, Subject, Advanced)
- Retry and dead-lettering (which HTTP codes retry vs drop)
- Security (Keys, SAS, Azure AD, Managed Identity)
- Webhook validation (synchronous vs asynchronous handshake)
- System event types (Blob, Resource Group, Key Vault, ACR, App Config)
- Domains for multi-tenant scenarios
- Custom input schema mapping
- Event delivery properties (static, dynamic)
- Networking (IP firewall, Private Endpoints)

**Event Hubs:**
- Partitions, partition keys, ordering guarantees
- Consumer groups (5 concurrent readers per partition per group)
- Checkpointing (Blob Storage, strategies)
- Capture (Avro format, time/size windows)
- Scaling (TU, PU, CU, auto-inflate)
- EventProcessorClient (load balancing, ownership)
- Tiers (Basic vs Standard vs Premium vs Dedicated)
- Max event size per tier (256 KB Basic, 1 MB others)
- Schema Registry (Avro, compatibility modes)
- Geo-DR (metadata only, NOT event data)
- Networking (Private Endpoints, IP firewall)
- Kafka compatibility (Standard tier+)
- Integration with Azure Functions (EventHubTrigger)

**Messaging Comparison (HEAVILY TESTED):**
- Event Grid vs Event Hubs vs Service Bus — when to use which
- Queue Storage vs Service Bus Queue differences
- Push vs Pull delivery models
- Ordering, deduplication, transactions support per service

### Sample Exam Questions

#### Event Grid

**Q1:** What is the maximum size for a single event in Event Grid?
- A) 64 KB
- B) 256 KB
- C) 1 MB
- D) 5 MB

**Answer: C) 1 MB**
Explanation: Event Grid supports events up to 1 MB. Events >64 KB are charged in 64 KB increments.

---

**Q2:** Which topic type is used to receive events from Azure services like Blob Storage?
- A) Custom Topic
- B) System Topic
- C) Partner Topic
- D) Domain Topic

**Answer: B) System Topic**
Explanation: System topics are built-in topics for Azure services that emit events.

---

**Q3:** You want to receive only blob events from the "images" container ending with ".jpg". Which filtering should you use?
- A) Event type filtering only
- B) Subject filtering only
- C) Both event type and subject filtering
- D) Advanced filtering only

**Answer: C) Both event type and subject filtering**
Explanation: Use event type filtering for BlobCreated and subject filtering for path and extension.

---

**Q4:** How long does Event Grid retry event delivery by default?
- A) 1 hour
- B) 4 hours
- C) 12 hours
- D) 24 hours

**Answer: D) 24 hours**
Explanation: Event Grid retries delivery for up to 24 hours with exponential backoff.

---

**Q5:** Which role allows sending events to an Event Grid topic?
- A) EventGrid Contributor
- B) EventGrid Data Sender
- C) EventGrid EventSubscription Contributor
- D) EventGrid Owner

**Answer: B) EventGrid Data Sender**
Explanation: EventGrid Data Sender role grants permission to send events to topics.

---

**Q6:** What is the purpose of dead-lettering in Event Grid?
- A) Delete failed events permanently
- B) Store undelivered events for later processing
- C) Retry events indefinitely
- D) Log event delivery failures

**Answer: B) Store undelivered events for later processing**
Explanation: Dead-lettering stores events that couldn't be delivered after all retries.

---

#### Event Hubs

**Q7:** What determines which partition receives an event in Event Hubs?
- A) Consumer group
- B) Partition key
- C) Throughput unit
- D) Event size

**Answer: B) Partition key**
Explanation: Events with the same partition key go to the same partition, ensuring order.

---

**Q8:** How many concurrent readers are allowed per partition per consumer group?
- A) 1
- B) 5
- C) 10
- D) Unlimited

**Answer: B) 5**
Explanation: Maximum 5 concurrent readers per partition per consumer group to prevent conflicts.

---

**Q9:** What is the purpose of checkpointing in Event Hubs?
- A) Count processed events
- B) Store event data
- C) Track position in event stream
- D) Authenticate consumers

**Answer: C) Track position in event stream**
Explanation: Checkpointing records the last processed position so consumers can resume after failure.

---

**Q10:** What file format does Event Hubs Capture use?
- A) JSON
- B) CSV
- C) Avro
- D) Parquet

**Answer: C) Avro**
Explanation: Event Hubs Capture stores events in Apache Avro format.

---

**Q11:** Which tier is required for Event Hubs Capture?
- A) Basic
- B) Standard or higher
- C) Premium only
- D) Dedicated only

**Answer: B) Standard or higher**
Explanation: Capture is available in Standard, Premium, and Dedicated tiers, not Basic.

---

**Q12:** What is the maximum retention period for events in Event Hubs Standard tier?
- A) 1 day
- B) 7 days
- C) 30 days
- D) 90 days

**Answer: B) 7 days**
Explanation: Standard tier supports 1-7 days retention. Premium/Dedicated support up to 90 days.

---

**Q13:** What is a throughput unit in Event Hubs?
- A) Number of partitions
- B) Measure of ingress/egress capacity
- C) Number of consumer groups
- D) Event size limit

**Answer: B) Measure of ingress/egress capacity**
Explanation: Each TU provides 1 MB/sec ingress and 2 MB/sec egress.

---

**Q14:** Which class should you use for processing events with automatic load balancing and checkpointing?
- A) EventHubProducerClient
- B) EventHubConsumerClient
- C) EventProcessorClient
- D) EventHubClient

**Answer: C) EventProcessorClient**
Explanation: EventProcessorClient provides automatic load balancing across partitions and checkpoint management.

---

**Q15:** What happens when events with the same partition key are sent to Event Hubs?
- A) They are distributed randomly
- B) They go to the same partition in order
- C) They are rejected as duplicates
- D) They go to different partitions

**Answer: B) They go to the same partition in order**
Explanation: Same partition key ensures events go to same partition, preserving order.

---

#### Advanced Scenarios

**Q16:** You need to process Azure Storage blob events and also capture them for batch analytics. What services should you use?
- A) Event Grid only
- B) Event Hubs only
- C) Event Grid to route events to Event Hubs
- D) Service Bus

**Answer: C) Event Grid to route events to Event Hubs**
Explanation: Use Event Grid to receive blob events and route them to Event Hubs, which can capture for batch analytics.

---

**Q17:** Which Event Hubs feature enables using Apache Kafka clients?
- A) Capture
- B) Kafka endpoint
- C) Kafka Protocol
- D) Event Grid integration

**Answer: B) Kafka endpoint**
Explanation: Event Hubs (Standard+) exposes a Kafka-compatible endpoint.

---

**Q18:** What is the Event Grid Cloud Events schema based on?
- A) Microsoft proprietary format
- B) CNCF standard
- C) Apache Avro
- D) JSON Schema

**Answer: B) CNCF standard**
Explanation: Cloud Events is a CNCF (Cloud Native Computing Foundation) specification.

---

**Q19:** How do you ensure Event Grid delivers events to a webhook securely?
- A) IP filtering only
- B) Validation handshake and optionally Azure AD auth
- C) Certificate authentication only
- D) No security needed

**Answer: B) Validation handshake and optionally Azure AD auth**
Explanation: Event Grid validates webhooks via handshake and can deliver with Azure AD tokens.

---

**Q20:** What is the default consumer group name in Event Hubs?
- A) Default
- B) $Default
- C) System
- D) Primary

**Answer: B) $Default**
Explanation: The default consumer group is named "$Default".

---

#### Event Grid Advanced

**Q21:** How does Event Grid validate webhook endpoints before delivering events?
- A) IP address verification
- B) Validation handshake — endpoint must return the validationCode
- C) SSL certificate validation only
- D) No validation required

**Answer: B) Validation handshake — endpoint must return the validationCode**
Explanation: Event Grid sends a SubscriptionValidationEvent with a validationCode. The endpoint must return it in the response body (synchronous) or GET the validationUrl within 5 minutes (asynchronous).

---

**Q22:** An Event Grid subscription returns HTTP 400 Bad Request for an event. What happens?
- A) Event is retried with exponential backoff
- B) Event is dead-lettered immediately
- C) Event is dropped (no retry)
- D) Event is retried after 5 minutes

**Answer: C) Event is dropped (no retry)**
Explanation: HTTP 400 indicates a permanent client error. Event Grid drops the event without retrying. 408, 429, 503, 504 are retried.

---

**Q23:** An Event Grid subscription returns HTTP 404 Not Found. What happens?
- A) Event is dropped immediately
- B) Event is retried for 24 hours
- C) Event is dropped after 5 minutes if 404 persists
- D) Event is dead-lettered immediately

**Answer: C) Event is dropped after 5 minutes if 404 persists**
Explanation: For 401 and 404, Event Grid waits up to 5 minutes. If the issue persists, the event is dead-lettered (if configured) or dropped.

---

**Q24:** You need to manage 10,000 different event topics for a multi-tenant SaaS application. What should you use?
- A) 10,000 custom topics
- B) Event Grid Domain with domain topics
- C) System topics
- D) Partner topics

**Answer: B) Event Grid Domain with domain topics**
Explanation: Event Grid Domains support up to 100,000 topics under a single management endpoint, ideal for multi-tenant scenarios.

---

**Q25:** You want events delivered to your webhook to include a custom HTTP header with the tenant ID from the event data. What should you configure?
- A) Event type filter
- B) Delivery attribute mapping with dynamic type
- C) Custom input schema
- D) Advanced filtering

**Answer: B) Delivery attribute mapping with dynamic type**
Explanation: Dynamic delivery attributes extract values from event data properties and include them as custom headers in the HTTP delivery.

---

**Q26:** Which two Event Grid schemas are supported for custom topics?
- A) Event Grid Schema and Apache Avro
- B) Event Grid Schema and Cloud Events v1.0
- C) Cloud Events and JSON Schema
- D) Custom Schema and Parquet

**Answer: B) Event Grid Schema and Cloud Events v1.0**
Explanation: Custom topics support Event Grid Schema (default), Cloud Events v1.0 (CNCF standard), and Custom Event Schema (with input mapping).

---

**Q27:** You have an Event Grid custom topic. You want to restrict who can publish events. Which Azure RBAC role should you assign?
- A) EventGrid Contributor
- B) EventGrid Data Sender
- C) EventGrid EventSubscription Contributor
- D) Owner

**Answer: B) EventGrid Data Sender**
Explanation: EventGrid Data Sender grants permission to send events to Event Grid topics. Contributor manages resources but doesn't grant data-plane access.

---

**Q28:** What is the maximum number of event subscriptions per Event Grid topic?
- A) 100
- B) 500
- C) 1,000
- D) 5,000

**Answer: B) 500**
Explanation: Each Event Grid topic supports up to 500 event subscriptions. Domains can have 500 subscriptions per domain topic.

---

#### Event Hubs Advanced

**Q29:** What is the maximum event size in Event Hubs Basic tier?
- A) 64 KB
- B) 256 KB
- C) 1 MB
- D) 5 MB

**Answer: B) 256 KB**
Explanation: Basic tier supports events up to 256 KB. Standard, Premium, and Dedicated tiers support up to 1 MB. This is a commonly tested distinction!

---

**Q30:** You enable Geo-Disaster Recovery for Event Hubs. Which of the following is replicated to the secondary namespace?
- A) Event data
- B) Consumer group offsets/checkpoints
- C) Metadata (namespace, Event Hub, consumer group configuration)
- D) All of the above

**Answer: C) Metadata (namespace, Event Hub, consumer group configuration)**
Explanation: Geo-DR only replicates metadata, NOT event data or checkpoint information. Use Capture if you need event data backup.

---

**Q31:** How many concurrent readers are allowed per partition per consumer group in Event Hubs?
- A) 1
- B) 5
- C) 10
- D) 32

**Answer: B) 5**
Explanation: Maximum 5 concurrent readers per partition per consumer group. More than 5 leads to conflicts. Use EventProcessorClient for proper load balancing.

---

**Q32:** You're using EventProcessorClient. Where are checkpoints stored?
- A) Event Hub partition
- B) Azure Blob Storage
- C) Azure Table Storage
- D) In-memory cache

**Answer: B) Azure Blob Storage**
Explanation: EventProcessorClient stores checkpoints and ownership information in Azure Blob Storage, enabling resume after failure and load balancing.

---

**Q33:** What is the difference between LoadBalancingStrategy.Balanced and Greedy?
- A) Balanced is faster
- B) Balanced claims one partition at a time; Greedy claims all needed partitions immediately
- C) Greedy uses more memory
- D) They are identical

**Answer: B) Balanced claims one partition at a time; Greedy claims all needed partitions immediately**
Explanation: Balanced (default) gradually rebalances by claiming one partition per cycle. Greedy claims as many partitions as needed immediately, resulting in faster rebalancing.

---

**Q34:** Which Event Hubs feature automatically archives events to Azure Blob Storage or Data Lake?
- A) Geo-Disaster Recovery
- B) Auto-Inflate
- C) Capture
- D) Schema Registry

**Answer: C) Capture**
Explanation: Event Hubs Capture automatically saves events to Blob Storage or Data Lake in Avro format. It's available in Standard tier and above.

---

**Q35:** What file format does Event Hubs Capture use?
- A) JSON
- B) CSV
- C) Avro
- D) Parquet

**Answer: C) Avro**
Explanation: Capture uses Apache Avro format, which is a compact binary serialization format with schema included.

---

**Q36:** Event Hubs auto-inflate scales TUs up automatically. Does it scale DOWN automatically?
- A) Yes, it scales both up and down
- B) No, it only scales UP; you must manually scale down
- C) It scales down after 24 hours of low usage
- D) It depends on the tier

**Answer: B) No, it only scales UP; you must manually scale down**
Explanation: Auto-inflate only increases TUs to handle traffic spikes. You must manually reduce TUs to save costs.

---

**Q37:** You need to process events from Event Hubs in Azure Functions. Which trigger should you use for best throughput?
- A) HTTP trigger with manual Event Hub read
- B) EventHubTrigger with single event processing
- C) EventHubTrigger with batch processing (IsBatched = true)
- D) Timer trigger with Event Hub polling

**Answer: C) EventHubTrigger with batch processing (IsBatched = true)**
Explanation: Batch processing is recommended for throughput. The function receives an array of events, reducing overhead per invocation.

---

#### Messaging Comparison (CRITICAL for Exam)

**Q38:** You need to process millions of IoT telemetry events per second with the ability to replay past events. Which service should you use?
- A) Event Grid
- B) Event Hubs
- C) Service Bus Queue
- D) Queue Storage

**Answer: B) Event Hubs**
Explanation: Event Hubs is designed for high-throughput streaming with event replay capability (offset-based). It retains events for 1-90 days.

---

**Q39:** You need to react to blob creation events in Azure Storage with near real-time notifications to multiple webhook subscribers. Which service?
- A) Event Hubs
- B) Service Bus
- C) Event Grid
- D) Queue Storage

**Answer: C) Event Grid**
Explanation: Event Grid natively integrates with Azure Storage system topics and pushes events to multiple subscribers (fan-out) with filtering.

---

**Q40:** You need reliable, ordered message delivery with duplicate detection and dead-letter queue for a financial transaction system. Which service?
- A) Event Grid
- B) Event Hubs
- C) Queue Storage
- D) Service Bus

**Answer: D) Service Bus**
Explanation: Service Bus provides FIFO ordering (with sessions), built-in duplicate detection, dead-letter queues, and transactions — ideal for financial systems.

---

**Q41:** Which service supports transactions and sessions?
- A) Event Grid
- B) Event Hubs
- C) Queue Storage
- D) Service Bus

**Answer: D) Service Bus**
Explanation: Only Service Bus supports transactions and sessions among the Azure messaging services.

---

**Q42:** You need simple, cheap queue storage with a maximum queue size exceeding 80 GB. Which service?
- A) Service Bus Queue
- B) Azure Queue Storage
- C) Event Hubs
- D) Event Grid

**Answer: B) Azure Queue Storage**
Explanation: Azure Queue Storage supports up to 500 TB per queue (far beyond 80 GB). Service Bus queues max at 1-80 GB. Queue Storage is also much cheaper.

---

**Q43:** Which messaging service supports the Apache Kafka protocol?
- A) Event Grid
- B) Event Hubs (Standard tier+)
- C) Service Bus
- D) Queue Storage

**Answer: B) Event Hubs (Standard tier+)**
Explanation: Event Hubs exposes a Kafka-compatible endpoint. Existing Kafka applications can use Event Hubs without code changes. Requires Standard tier or above.

---

**Q44:** You need to schedule a message to be delivered at a specific time in the future. Which service supports this?
- A) Event Grid
- B) Event Hubs
- C) Queue Storage
- D) Service Bus

**Answer: D) Service Bus**
Explanation: Service Bus supports scheduled message delivery (ScheduledEnqueueTimeUtc). Event Grid, Event Hubs, and Queue Storage do not support scheduled delivery.

---

**Q45:** Which Event Grid component allows receiving events from third-party SaaS providers like Auth0 or SAP?
- A) System Topic
- B) Custom Topic
- C) Partner Topic
- D) Domain Topic

**Answer: C) Partner Topic**
Explanation: Partner Topics enable third-party SaaS providers to publish events to Event Grid through registered partner namespaces.

---

**Q46:** In Event Grid, events >64 KB are charged in what increments?
- A) 1 KB increments
- B) 32 KB increments
- C) 64 KB increments
- D) 128 KB increments

**Answer: C) 64 KB increments**
Explanation: Events larger than 64 KB are charged in 64 KB increments. A 65 KB event counts as 2 operations, a 130 KB event counts as 3.

---

**Q47:** You want to ensure all events for a specific customer are processed in order in Event Hubs. What should you do?
- A) Use a single partition
- B) Use the customer ID as partition key
- C) Use consumer groups
- D) Enable FIFO on the Event Hub

**Answer: B) Use the customer ID as partition key**
Explanation: Events with the same partition key go to the same partition, and ordering is guaranteed within a partition. Using customer ID as partition key ensures all events for a customer are ordered.

---

**Q48:** Which checkpoint strategy provides the BEST balance between reliability and performance?
- A) Checkpoint every event
- B) Checkpoint every N events (e.g., every 100)
- C) Never checkpoint
- D) Checkpoint at startup only

**Answer: B) Checkpoint every N events (e.g., every 100)**
Explanation: Checkpointing every event is reliable but slow (I/O overhead). Checkpointing every N events reduces overhead while limiting reprocessing on failure. Never checkpointing means reprocessing everything on restart.

---

**Q49:** How does Event Grid handle Azure Functions as event handlers differently from regular webhooks?
- A) No difference
- B) Azure Functions receive events via trigger, and validation handshake is handled automatically
- C) Azure Functions require SAS authentication
- D) Azure Functions can only receive Cloud Events schema

**Answer: B) Azure Functions receive events via trigger, and validation handshake is handled automatically**
Explanation: When using Azure Functions with Event Grid trigger, the runtime automatically handles the webhook validation handshake. No manual validation code needed.

---

**Q50:** Your Event Grid event subscription uses dead-lettering. Where are undelivered events stored?
- A) Azure SQL Database
- B) Event Hubs
- C) Azure Blob Storage
- D) Service Bus Queue

**Answer: C) Azure Blob Storage**
Explanation: Dead-lettered events are stored in Azure Blob Storage. You specify a storage account and container when configuring dead-lettering.

---

## Key CLI Commands Reference

### Event Grid
```bash
# Create custom topic
az eventgrid topic create --name mytopic --resource-group myRG --location eastus

# Create subscription
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.EventGrid/topics/{topic} \
  --endpoint https://myfunction.azurewebsites.net/api/handler

# Create system topic
az eventgrid system-topic create \
  --name mysystemtopic \
  --resource-group myRG \
  --source /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{account} \
  --topic-type Microsoft.Storage.StorageAccounts

# Get topic key
az eventgrid topic key list --name mytopic --resource-group myRG
```

### Event Hubs
```bash
# Create namespace
az eventhubs namespace create --name mynamespace --resource-group myRG --location eastus --sku Standard

# Create event hub
az eventhubs eventhub create --name myeventhub --resource-group myRG --namespace-name mynamespace --partition-count 4

# Enable capture
az eventhubs eventhub update \
  --name myeventhub --resource-group myRG --namespace-name mynamespace \
  --enable-capture true --storage-account {account-id} --blob-container capture

# Create consumer group
az eventhubs eventhub consumer-group create \
  --name myconsumergroup --resource-group myRG --namespace-name mynamespace --eventhub-name myeventhub

# Update throughput units
az eventhubs namespace update --name mynamespace --resource-group myRG --capacity 10

# Enable auto-inflate
az eventhubs namespace update --name mynamespace --resource-group myRG \
  --enable-auto-inflate true --maximum-throughput-units 20

# Create authorization rule (Send only)
az eventhubs eventhub authorization-rule create \
  --resource-group myRG --namespace-name mynamespace --eventhub-name myeventhub \
  --name SendRule --rights Send

# Set IP firewall
az eventhubs namespace network-rule-set update \
  --resource-group myRG --namespace-name mynamespace \
  --default-action Deny --ip-rules '[{"ipMask":"10.0.0.0/8","action":"Allow"}]'
```

### Event Grid Advanced
```bash
# Create topic with Cloud Events schema
az eventgrid topic create --name mytopic --resource-group myRG --location eastus \
  --input-schema cloudeventschemav1_0

# Create event domain (multi-tenant)
az eventgrid domain create --name mydomain --resource-group myRG --location eastus

# Subscription with dead-lettering
az eventgrid event-subscription create \
  --name mysubscription --source-resource-id {resource-id} --endpoint {endpoint} \
  --deadletter-endpoint {storage-container-resource-id}

# Subscription with custom retry policy
az eventgrid event-subscription create \
  --name mysubscription --source-resource-id {resource-id} --endpoint {endpoint} \
  --max-delivery-attempts 10 --event-ttl 1440

# Subscription with delivery attributes
az eventgrid event-subscription create \
  --name mysubscription --source-resource-id {resource-id} --endpoint {endpoint} \
  --delivery-attribute-mapping "X-Custom" static "myvalue" false

# Enable managed identity on topic
az eventgrid topic update --name mytopic --resource-group myRG --identity systemassigned

# Configure IP firewall on topic
az eventgrid topic update --name mytopic --resource-group myRG \
  --public-network-access enabled --inbound-ip-rules "10.0.0.0/8" allow

# Route Event Grid → Event Hub
az eventgrid event-subscription create \
  --name grid-to-hub --source-resource-id {storage-resource-id} \
  --endpoint-type eventhub --endpoint {eventhub-resource-id} \
  --included-event-types Microsoft.Storage.BlobCreated

# Route Event Grid → Service Bus Queue
az eventgrid event-subscription create \
  --name grid-to-bus --source-resource-id {storage-resource-id} \
  --endpoint-type servicebusqueue --endpoint {queue-resource-id}
```

---

## Exam Gotchas & Tricky Distinctions

| # | Common Misconception | Reality |
|---|---------------------|---------|
| 1 | "Event Grid guarantees event ordering" | ❌ Event Grid does **NOT** guarantee ordering. Events may arrive out of order. Only Event Hubs guarantees order (per partition). |
| 2 | "Event Grid retries on HTTP 400 Bad Request" | ❌ HTTP 400 and 413 are **dropped immediately** (no retry). Only 408, 429, 503, 504 are retried with backoff. |
| 3 | "Event Grid retries on 404 like other errors" | ❌ HTTP 404 and 401 are dropped after a **5-minute delay**, not retried for 24 hours like 503/504. |
| 4 | "Event Hubs Geo-DR replicates event data" | ❌ Geo-DR only replicates **metadata** (namespace config, Event Hub config). Event data is NOT replicated. Use Capture for data backup. |
| 5 | "Event Hubs auto-inflate scales both up and down" | ❌ Auto-inflate only scales **UP**. You must manually reduce TUs to scale down and save costs. |
| 6 | "Event Hubs Basic tier supports 1 MB events" | ❌ Basic tier max event size is **256 KB**. Standard/Premium/Dedicated support 1 MB. |
| 7 | "Event Hubs Basic tier supports Capture" | ❌ Capture requires **Standard tier or above**. Basic tier does NOT support Capture or Kafka. |
| 8 | "Consumer groups in Basic tier allow 20 groups" | ❌ Basic tier only supports **1 consumer group** ($Default). Standard supports 20, Premium supports 100. |
| 9 | "EventHubConsumerClient handles load balancing" | ❌ `EventHubConsumerClient` is for simple reading. Use **`EventProcessorClient`** for load balancing, ownership, and checkpointing. |
| 10 | "Partition count can be changed after creation in Standard tier" | ❌ In Basic/Standard tiers, partition count is **fixed at creation**. Only Premium tier allows changing partitions. |
| 11 | "All Event Grid handlers use push delivery" | ✅ Correct! Event Grid is push-based. But note: Event Hubs as a handler receives pushed events, which consumers then pull. |
| 12 | "Event Hubs events are deleted after reading" | ❌ Events are **retained** for the configured period regardless of consumption. Multiple consumer groups can read the same events. |
| 13 | "Webhook validation requires code changes in Azure Functions" | ❌ Azure Functions with Event Grid trigger handle validation **automatically**. No manual handshake code needed. |
| 14 | "Event Grid dead-lettering is enabled by default" | ❌ Dead-lettering must be **explicitly configured** with a Blob Storage endpoint. Without it, undeliverable events are dropped. |
| 15 | "Event Hubs Capture uses JSON format" | ❌ Capture uses **Apache Avro** format (compact binary with embedded schema), not JSON. |
| 16 | "Queue Storage supports duplicate detection" | ❌ Only **Service Bus** supports duplicate detection. Queue Storage and Event Hubs do not. |
| 17 | "Event Grid supports message sessions" | ❌ Only **Service Bus** supports sessions and transactions. Event Grid and Event Hubs do not. |
| 18 | "Service Bus Queue max size is the same as Queue Storage" | ❌ Queue Storage: up to **500 TB**. Service Bus Queue: **1-80 GB** (Standard) or 100 GB (Premium). Very different scales. |
| 19 | "Event Grid custom topics have unlimited subscriptions" | ❌ Max **500 event subscriptions** per topic. Max **100 custom topics** per Azure subscription. |
| 20 | "Checkpoint every event is recommended for performance" | ❌ Checkpointing every event causes significant I/O overhead. Best practice is checkpoint every **N events** or **time-based** for balance. |
| 21 | "Event Hubs and Event Grid are interchangeable" | ❌ Event Grid = event **routing** (push, discrete events). Event Hubs = event **streaming** (pull, high-throughput, replay). Completely different use cases. |
| 22 | "More than 5 readers per partition per consumer group is fine" | ❌ Max **5 concurrent readers** per partition per consumer group. Exceeding this causes `ReceiverDisconnectedException`. |
| 23 | "Cloud Events schema is Microsoft-specific" | ❌ Cloud Events is a **CNCF standard** (Cloud Native Computing Foundation), not Microsoft-specific. Cross-cloud compatible. |
| 24 | "EventProcessorClient stores ownership in Event Hub" | ❌ Ownership and checkpoint data is stored in **Azure Blob Storage**, not in the Event Hub itself. |
| 25 | "Event Grid events can be replayed from the past" | ❌ Event Grid does NOT support event replay. Only **Event Hubs** supports replay via offset/sequence rewind. |
| 26 | "Service Bus and Queue Storage have the same message size limit" | ❌ Queue Storage: **64 KB** max. Service Bus Standard: **256 KB**. Service Bus Premium: up to **100 MB**. |
| 27 | "You need different connection strings for Event Hubs Kafka" | ❌ Kafka uses the **same Event Hubs connection string** with SASL_SSL. Username is `$ConnectionString`, password is the connection string itself. |
| 28 | "Async webhook validation has no time limit" | ❌ Async validation URL must be called via GET within **5 minutes** or the subscription creation fails. |
| 29 | "Event Grid maximum event size is 64 KB" | ❌ Max event size is **1 MB**. Events >64 KB are charged in 64 KB increments (billing distinction, not size limit). |
| 30 | "Event Hubs Schema Registry supports JSON Schema" | ❌ Schema Registry currently supports **Avro** schemas only (with compatibility modes: None, Backward, Forward, Full). |

---

## Study Tips for AZ-204 Exam

1. **Master the comparison** — Event Grid vs Event Hubs vs Service Bus is the #1 tested topic. Know when to use each.
2. **Event Grid retry behavior** — Memorize which HTTP codes retry (408, 429, 503, 504) vs drop (400, 403, 413) vs delay (401, 404).
3. **Event Hubs tier limits** — Basic: 256 KB events, 1 consumer group, no Capture, no Kafka. Standard: 1 MB, 20 groups, Capture, Kafka.
4. **Partition key = ordering** — Same partition key → same partition → guaranteed order. No key → round-robin.
5. **Consumer groups** — Max 5 readers per partition per group. Use EventProcessorClient for load balancing.
6. **Checkpointing** — Stored in Blob Storage. Checkpoint every N events for balance (not every event).
7. **Dead-lettering** — Event Grid: must configure (Blob Storage). Service Bus: built-in. Event Hubs: no dead-letter (use Capture).
8. **Webhook validation** — Synchronous (return validationCode) vs Asynchronous (GET validationUrl within 5 min). Functions handle automatically.
9. **Geo-DR** — Metadata only! Event data NOT replicated. Failover is manual.
10. **Schema knowledge** — Event Grid Schema vs Cloud Events (CNCF). Capture uses Avro. Schema Registry supports Avro only.
11. **Queue Storage vs Service Bus** — Queue Storage for simple/cheap/large. Service Bus for enterprise (sessions, transactions, dedup, dead-letter).
12. **SDK classes** — `EventGridPublisherClient`, `EventHubProducerClient`, `EventHubConsumerClient`, `EventProcessorClient`, `EventHubBufferedProducerClient`.
13. **Auto-inflate** — Only scales UP, never down. Must manually reduce TUs.
14. **Security** — Keys, SAS tokens, Azure AD + RBAC. Know the specific roles (Data Sender, Data Receiver, Data Owner).
15. **Architecture patterns** — Fan-out, competing consumers, event sourcing, event streaming pipeline. Know which service fits each.

---

## Important URLs and References

- Event Grid Documentation: https://learn.microsoft.com/en-us/azure/event-grid/
- Event Hubs Documentation: https://learn.microsoft.com/en-us/azure/event-hubs/
- Event Grid vs Event Hubs vs Service Bus: https://learn.microsoft.com/en-us/azure/event-grid/compare-messaging-services
- Event Grid System Topics: https://learn.microsoft.com/en-us/azure/event-grid/system-topics
- Event Hubs Capture: https://learn.microsoft.com/en-us/azure/event-hubs/event-hubs-capture-overview
- Event Hubs SDK: https://learn.microsoft.com/en-us/azure/event-hubs/sdks
- Schema Registry: https://learn.microsoft.com/en-us/azure/event-hubs/schema-registry-overview

---

## Quick Reference Summary

### Event Grid
- **Topics**: System (Azure), Custom (yours), Partner (SaaS), Domain (multi-tenant, up to 100K topics)
- **Schemas**: Event Grid (default), Cloud Events v1.0 (CNCF standard), Custom (with input mapping)
- **Filtering**: Event type, Subject (begins/ends with), Advanced (operators on data fields)
- **Retry**: 24 hours max, 30 attempts, exponential backoff. Drop: 400, 403, 413. Delay+drop: 401, 404.
- **Dead-letter**: Must configure explicitly → Azure Blob Storage
- **Handlers**: Functions, Event Hubs, Service Bus, Webhooks, Logic Apps, Storage Queues, Automation
- **Security**: Access Keys, SAS Tokens, Azure AD + RBAC (Data Sender role)
- **Limits**: 1 MB max event, 500 subscriptions/topic, 100 topics/subscription, 5000 events/sec/topic
- **Webhook validation**: Synchronous (return code) or Async (GET URL within 5 min)

### Event Hubs
- **Partitions**: Ordered sequences, parallel processing, partition key for ordering
- **Consumer Groups**: Independent views, $Default by default, max 5 readers/partition/group
- **Capture**: Automatic archival to Blob/Data Lake in **Avro** format (Standard tier+)
- **Throughput Units (Standard)**: 1 MB/sec ingress, 2 MB/sec egress per TU
- **Checkpointing**: Track position, stored in **Blob Storage** (via EventProcessorClient)
- **Auto-Inflate**: Scales UP only, must manually scale down
- **Kafka**: Compatible endpoint (Standard tier+), same connection string
- **Geo-DR**: Metadata only, NOT event data, manual failover
- **Schema Registry**: Avro schemas only, compatibility modes (None, Backward, Forward, Full)

### Event Hubs Tier Quick Reference

| Feature | Basic | Standard | Premium | Dedicated |
|---------|-------|----------|---------|-----------|
| Max event size | 256 KB | 1 MB | 1 MB | 1 MB |
| Consumer groups | 1 | 20 | 100 | 1000 |
| Partitions | 32 | 32 | 100 | 1024 |
| Retention | 1 day | 7 days | 90 days | 90 days |
| Capture | ❌ | ✅ | ✅ | ✅ |
| Kafka | ❌ | ✅ | ✅ | ✅ |
| Schema Registry | ❌ | ✅ | ✅ | ✅ |

### Messaging Service Decision Matrix

| Need | Use |
|------|-----|
| React to Azure resource changes | **Event Grid** |
| Fan-out one event to many handlers | **Event Grid** |
| High-volume telemetry/streaming | **Event Hubs** |
| Event replay/reprocessing | **Event Hubs** |
| Kafka compatibility | **Event Hubs** |
| Reliable ordered messaging | **Service Bus** |
| Transactions & sessions | **Service Bus** |
| Duplicate detection | **Service Bus** |
| Scheduled delivery | **Service Bus** |
| Simple cheap queue (>80 GB) | **Queue Storage** |
| Server-side audit trail | **Queue Storage** |

### Key SDK Classes
- **Event Grid**: `EventGridPublisherClient`, `EventGridEvent`, `CloudEvent`
- **Event Hubs Producer**: `EventHubProducerClient`, `EventHubBufferedProducerClient`
- **Event Hubs Consumer**: `EventHubConsumerClient` (simple), `EventProcessorClient` (recommended)
- **NuGet**: `Azure.Messaging.EventGrid`, `Azure.Messaging.EventHubs`, `Azure.Messaging.EventHubs.Processor`

### Event Grid Retry Quick Reference

| HTTP Code | Behavior |
|-----------|----------|
| 400 | Drop immediately |
| 401 | Drop after 5 min |
| 403 | Drop immediately |
| 404 | Drop after 5 min |
| 408 | Retry with backoff |
| 413 | Drop immediately |
| 429 | Retry with backoff |
| 503 | Retry with backoff |
| 504 | Retry with backoff |

