# AZ-204: Develop Message-Based Solutions - Complete Study Notes

## Learning Path Overview
This learning path covers developing message-based solutions in Azure with 1 module:
1. Discover Azure message queues

**Level:** Intermediate | **Products:** Azure Service Bus, Azure Queue Storage | **Role:** Developer

---

## MODULE 1: DISCOVER AZURE MESSAGE QUEUES

### 1.1 Introduction to Message-Based Architecture

**What is Message-Based Architecture?**
- Asynchronous communication between components
- Decouples sender from receiver
- Enables reliable, scalable applications
- Messages stored until processed

**Key Benefits:**
- **Loose coupling**: Components independent of each other
- **Load leveling**: Handle traffic spikes gracefully
- **Reliability**: Messages persist if receiver unavailable
- **Scalability**: Scale senders/receivers independently
- **Temporal decoupling**: Sender/receiver don't need to be online simultaneously

**Azure Messaging Services:**

| Service | Type | Use Case |
|---------|------|----------|
| Azure Service Bus | Enterprise messaging | Complex workflows, transactions |
| Azure Queue Storage | Simple queuing | Basic async processing |
| Azure Event Grid | Event routing | Reactive event handling |
| Azure Event Hubs | Event streaming | Big data ingestion |

---

## AZURE SERVICE BUS

### 1.2 Introduction to Azure Service Bus

**What is Azure Service Bus?**
- Fully managed enterprise message broker
- Supports queues (point-to-point) and topics (publish-subscribe)
- Advanced features: transactions, sessions, dead-lettering
- Supports AMQP 1.0 and HTTP protocols

**Key Features:**
- Message sessions (ordered processing)
- Dead-letter queue
- Scheduled delivery
- Message deferral
- Batching
- Transactions
- Duplicate detection
- Auto-forwarding
- Message expiration (TTL)

**Use Cases:**
- Order processing
- Financial transactions
- Workflow orchestration
- Cross-service communication
- Legacy system integration
- Application decoupling

### 1.3 Service Bus Components

**Architecture:**

```
                    Service Bus Namespace
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   ┌──────────────────┐        ┌──────────────────────────────┐  │
│   │      Queues      │        │          Topics              │  │
│   │                  │        │                              │  │
│   │  ┌────────────┐  │        │  ┌────────────────────────┐  │  │
│   │  │   Queue A  │  │        │  │       Topic A          │  │  │
│   │  │ [Messages] │  │        │  │                        │  │  │
│   │  └────────────┘  │        │  │  ┌─────────────────┐   │  │  │
│   │                  │        │  │  │ Subscription 1  │   │  │  │
│   │  ┌────────────┐  │        │  │  │   [Messages]    │   │  │  │
│   │  │   Queue B  │  │        │  │  └─────────────────┘   │  │  │
│   │  │ [Messages] │  │        │  │  ┌─────────────────┐   │  │  │
│   │  └────────────┘  │        │  │  │ Subscription 2  │   │  │  │
│   │                  │        │  │  │   [Messages]    │   │  │  │
│   └──────────────────┘        │  │  └─────────────────┘   │  │  │
│                               │  └────────────────────────┘   │  │
│                               └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

**Components:**

| Component | Description |
|-----------|-------------|
| Namespace | Container for queues/topics, management boundary |
| Queue | Point-to-point messaging, one receiver |
| Topic | Publish-subscribe, multiple subscriptions |
| Subscription | Receives copy of topic messages |
| Message | Data being transferred |

### 1.4 Service Bus Tiers

| Feature | Basic | Standard | Premium |
|---------|-------|----------|---------|
| **Queues** | ✅ | ✅ | ✅ |
| **Topics** | ❌ | ✅ | ✅ |
| **Message Size** | 256 KB | 256 KB | 100 MB |
| **Transactions** | ❌ | ✅ | ✅ |
| **Sessions** | ❌ | ✅ | ✅ |
| **Duplicate Detection** | ❌ | ✅ | ✅ |
| **Scheduled Messages** | ❌ | ✅ | ✅ |
| **Dead-lettering** | ✅ | ✅ | ✅ |
| **VNet Integration** | ❌ | ❌ | ✅ |
| **Private Link** | ❌ | ❌ | ✅ |
| **Dedicated Resources** | ❌ | ❌ | ✅ |
| **Geo-DR** | ❌ | ✅ | ✅ |
| **Availability Zones** | ❌ | ❌ | ✅ |

**Tier Selection:**
- **Basic**: Simple queuing, cost-effective
- **Standard**: Topics, sessions, transactions
- **Premium**: High throughput, isolation, large messages

### 1.5 Queues - Point-to-Point Messaging

**Queue Characteristics:**
- FIFO ordering (best effort, guaranteed with sessions)
- Single consumer per message
- Messages stored until processed or expired
- At-least-once delivery

**Queue Operations:**

```bash
# Create namespace
az servicebus namespace create \
  --name mynamespace \
  --resource-group myRG \
  --location eastus \
  --sku Standard

# Create queue
az servicebus queue create \
  --name myqueue \
  --namespace-name mynamespace \
  --resource-group myRG \
  --max-size 5120 \
  --default-message-time-to-live P14D \
  --lock-duration PT1M \
  --enable-dead-lettering-on-message-expiration true

# List queues
az servicebus queue list \
  --namespace-name mynamespace \
  --resource-group myRG

# Delete queue
az servicebus queue delete \
  --name myqueue \
  --namespace-name mynamespace \
  --resource-group myRG
```

**Queue Properties:**

| Property | Description | Default |
|----------|-------------|---------|
| Max Size | Queue size in MB | 1024 MB |
| Message TTL | Time-to-live | 14 days |
| Lock Duration | Peek-lock timeout | 60 seconds |
| Max Delivery Count | Before dead-letter | 10 |
| Duplicate Detection | Time window | None |
| Sessions Required | Enable sessions | false |

### 1.6 Topics and Subscriptions - Publish-Subscribe

**Topic Characteristics:**
- One-to-many communication
- Publishers send to topic
- Subscribers receive from subscriptions
- Each subscription gets copy of message
- Filtering per subscription

**Topic Flow:**

```
Publisher → Topic → Subscription 1 → Consumer 1
                 → Subscription 2 → Consumer 2
                 → Subscription 3 → Consumer 3
```

**Topic Operations:**

```bash
# Create topic
az servicebus topic create \
  --name mytopic \
  --namespace-name mynamespace \
  --resource-group myRG \
  --max-size 5120 \
  --default-message-time-to-live P14D

# Create subscription
az servicebus topic subscription create \
  --name mysubscription \
  --topic-name mytopic \
  --namespace-name mynamespace \
  --resource-group myRG \
  --lock-duration PT1M \
  --max-delivery-count 10 \
  --enable-dead-lettering-on-message-expiration true

# List subscriptions
az servicebus topic subscription list \
  --topic-name mytopic \
  --namespace-name mynamespace \
  --resource-group myRG
```

### 1.7 Subscription Filters

**Filter Types:**

#### 1. Boolean Filter (True/False)
```bash
# Match all messages (default)
az servicebus topic subscription rule create \
  --name AllMessages \
  --subscription-name mysubscription \
  --topic-name mytopic \
  --namespace-name mynamespace \
  --resource-group myRG \
  --filter-type TrueFilter

# Match no messages
az servicebus topic subscription rule create \
  --name NoMessages \
  --subscription-name mysubscription \
  --topic-name mytopic \
  --namespace-name mynamespace \
  --resource-group myRG \
  --filter-type FalseFilter
```

#### 2. SQL Filter
```bash
# Filter by property
az servicebus topic subscription rule create \
  --name HighPriority \
  --subscription-name mysubscription \
  --topic-name mytopic \
  --namespace-name mynamespace \
  --resource-group myRG \
  --filter-sql-expression "Priority = 'High'"

# Complex filter
az servicebus topic subscription rule create \
  --name ComplexFilter \
  --subscription-name mysubscription \
  --topic-name mytopic \
  --namespace-name mynamespace \
  --resource-group myRG \
  --filter-sql-expression "Priority = 'High' AND Region IN ('US', 'EU')"
```

**SQL Filter Operators:**
- Comparison: `=`, `<>`, `<`, `>`, `<=`, `>=`
- Logical: `AND`, `OR`, `NOT`
- Pattern: `LIKE` with `%` and `_` wildcards
- Existence: `IS NULL`, `IS NOT NULL`
- Collection: `IN (value1, value2, ...)`

#### 3. Correlation Filter
```bash
# Filter by correlation ID
az servicebus topic subscription rule create \
  --name CorrelationFilter \
  --subscription-name mysubscription \
  --topic-name mytopic \
  --namespace-name mynamespace \
  --resource-group myRG \
  --correlation-filter correlation-id=order-123
```

**Correlation Filter Properties:**
- CorrelationId
- ContentType
- Label
- MessageId
- ReplyTo
- ReplyToSessionId
- SessionId
- To
- Custom properties

#### 4. Filter Actions
```bash
# Add action to modify message
az servicebus topic subscription rule create \
  --name FilterWithAction \
  --subscription-name mysubscription \
  --topic-name mytopic \
  --namespace-name mynamespace \
  --resource-group myRG \
  --filter-sql-expression "Priority = 'High'" \
  --action-sql-expression "SET Processed = true"
```

### 1.8 Message Receive Modes

**Two Receive Modes:**

#### 1. Receive and Delete
- Message removed immediately when received
- Simpler, faster
- Risk of message loss if processing fails
- No duplicate processing

```csharp
var receiver = client.CreateReceiver(queueName, new ServiceBusReceiverOptions
{
    ReceiveMode = ServiceBusReceiveMode.ReceiveAndDelete
});
```

#### 2. Peek-Lock (Default)
- Message locked, not removed
- Must complete, abandon, or dead-letter
- Lock timeout (default 60 seconds)
- Safer, guaranteed processing

```csharp
var receiver = client.CreateReceiver(queueName, new ServiceBusReceiverOptions
{
    ReceiveMode = ServiceBusReceiveMode.PeekLock
});

// After processing
await receiver.CompleteMessageAsync(message);  // Remove from queue
// OR
await receiver.AbandonMessageAsync(message);   // Return to queue
// OR
await receiver.DeadLetterMessageAsync(message); // Move to DLQ
// OR
await receiver.DeferMessageAsync(message);     // Defer for later
```

**Lock Management:**

```csharp
// Renew lock (before timeout)
await receiver.RenewMessageLockAsync(message);

// Auto-renew with processor
var processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions
{
    AutoCompleteMessages = false,
    MaxAutoLockRenewalDuration = TimeSpan.FromMinutes(10)
});
```

### 1.9 Dead-Letter Queue (DLQ)

**What is Dead-Letter Queue?**
- Sub-queue for unprocessable messages
- Automatic dead-lettering on conditions
- Manual dead-lettering by application
- Each queue/subscription has own DLQ

**Automatic Dead-Lettering Conditions:**
- Message expired (TTL reached)
- Max delivery count exceeded
- Filter evaluation exception
- Message size exceeded

**DLQ Path:**
```
Queue: myqueue/$DeadLetterQueue
Subscription: mytopic/Subscriptions/mysubscription/$DeadLetterQueue
```

**Working with DLQ:**

```csharp
// Receive from DLQ
var dlqReceiver = client.CreateReceiver(queueName, new ServiceBusReceiverOptions
{
    SubQueue = SubQueue.DeadLetter
});

await foreach (var message in dlqReceiver.ReceiveMessagesAsync())
{
    // Get dead-letter reason
    var reason = message.DeadLetterReason;
    var description = message.DeadLetterErrorDescription;
    
    Console.WriteLine($"DLQ Reason: {reason}");
    Console.WriteLine($"Description: {description}");
    
    // Process or discard
    await dlqReceiver.CompleteMessageAsync(message);
}
```

**Manual Dead-Lettering:**

```csharp
await receiver.DeadLetterMessageAsync(message, new Dictionary<string, object>
{
    { "DeadLetterReason", "ProcessingError" },
    { "DeadLetterErrorDescription", "Invalid message format" }
});
```

### 1.10 Message Sessions

**What are Sessions?**
- Enable FIFO ordering guarantee
- Group related messages
- Single receiver per session
- Required for ordered processing

**Session Characteristics:**
- SessionId property identifies session
- Session state can be stored
- Only one receiver can lock session
- Session lock (not message lock)

**Enable Sessions:**

```bash
# Create session-enabled queue
az servicebus queue create \
  --name mysessionqueue \
  --namespace-name mynamespace \
  --resource-group myRG \
  --enable-session true
```

**Session Processing:**

```csharp
// Send with session ID
var sender = client.CreateSender(queueName);
var message = new ServiceBusMessage("Order item 1")
{
    SessionId = "order-123"
};
await sender.SendMessageAsync(message);

// Receive from specific session
var sessionReceiver = await client.AcceptSessionAsync(queueName, "order-123");

// Receive from next available session
var sessionReceiver = await client.AcceptNextSessionAsync(queueName);

// Process session messages
await foreach (var message in sessionReceiver.ReceiveMessagesAsync())
{
    Console.WriteLine($"Session: {message.SessionId}");
    Console.WriteLine($"Data: {message.Body}");
    await sessionReceiver.CompleteMessageAsync(message);
}

// Session state management
await sessionReceiver.SetSessionStateAsync(new BinaryData("processing"));
var state = await sessionReceiver.GetSessionStateAsync();
```

**Session Processor:**

```csharp
var processor = client.CreateSessionProcessor(queueName, new ServiceBusSessionProcessorOptions
{
    AutoCompleteMessages = false,
    MaxConcurrentSessions = 5,
    SessionIdleTimeout = TimeSpan.FromSeconds(10)
});

processor.ProcessMessageAsync += async args =>
{
    Console.WriteLine($"Session: {args.SessionId}");
    Console.WriteLine($"Message: {args.Message.Body}");
    await args.CompleteMessageAsync(args.Message);
};

processor.ProcessErrorAsync += async args =>
{
    Console.WriteLine($"Error: {args.Exception}");
};

await processor.StartProcessingAsync();
```

### 1.11 Advanced Features

#### Scheduled Messages
```csharp
// Schedule message for future delivery
var message = new ServiceBusMessage("Scheduled message");
var sequenceNumber = await sender.ScheduleMessageAsync(
    message,
    DateTimeOffset.UtcNow.AddMinutes(30));

// Cancel scheduled message
await sender.CancelScheduledMessageAsync(sequenceNumber);
```

#### Message Deferral
```csharp
// Defer message for later processing
await receiver.DeferMessageAsync(message);

// Receive deferred message by sequence number
var deferredMessage = await receiver.ReceiveDeferredMessageAsync(message.SequenceNumber);
await receiver.CompleteMessageAsync(deferredMessage);
```

#### Duplicate Detection
```bash
# Enable duplicate detection
az servicebus queue create \
  --name myqueue \
  --namespace-name mynamespace \
  --resource-group myRG \
  --enable-duplicate-detection true \
  --duplicate-detection-history-time-window P1D
```

```csharp
// Set MessageId for duplicate detection
var message = new ServiceBusMessage("Data")
{
    MessageId = "unique-id-123"  // Duplicates with same ID are rejected
};
```

#### Auto-Forwarding
```bash
# Forward to another queue
az servicebus queue create \
  --name sourcequeue \
  --namespace-name mynamespace \
  --resource-group myRG \
  --forward-to destinationqueue

# Forward dead-letters
az servicebus queue create \
  --name myqueue \
  --namespace-name mynamespace \
  --resource-group myRG \
  --forward-dead-lettered-messages-to errorqueue
```

#### Transactions
```csharp
// Transactional send (all or nothing)
using var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);

await sender.SendMessageAsync(new ServiceBusMessage("Message 1"));
await sender.SendMessageAsync(new ServiceBusMessage("Message 2"));

scope.Complete();  // Commit both or neither
```

### 1.12 Service Bus SDK for .NET

**Installation:**
```bash
dotnet add package Azure.Messaging.ServiceBus
```

**ServiceBusClient:**

```csharp
using Azure.Messaging.ServiceBus;

// Create client with connection string
var client = new ServiceBusClient(connectionString);

// Create client with Azure AD
var client = new ServiceBusClient("mynamespace.servicebus.windows.net", new DefaultAzureCredential());
```

**Sending Messages:**

```csharp
// Create sender
var sender = client.CreateSender(queueOrTopicName);

// Send single message
var message = new ServiceBusMessage("Hello, Service Bus!");
message.ApplicationProperties["Priority"] = "High";
message.ContentType = "application/json";
message.Subject = "Greeting";
message.CorrelationId = "correlation-123";
message.TimeToLive = TimeSpan.FromMinutes(5);

await sender.SendMessageAsync(message);

// Send batch
using var batch = await sender.CreateMessageBatchAsync();
for (int i = 0; i < 100; i++)
{
    if (!batch.TryAddMessage(new ServiceBusMessage($"Message {i}")))
    {
        // Batch full, send it
        await sender.SendMessagesAsync(batch);
        batch = await sender.CreateMessageBatchAsync();
        batch.TryAddMessage(new ServiceBusMessage($"Message {i}"));
    }
}
await sender.SendMessagesAsync(batch);

// Send list of messages
var messages = Enumerable.Range(0, 10)
    .Select(i => new ServiceBusMessage($"Message {i}"))
    .ToList();
await sender.SendMessagesAsync(messages);
```

**Receiving Messages:**

```csharp
// Create receiver
var receiver = client.CreateReceiver(queueName);

// Receive single message
var message = await receiver.ReceiveMessageAsync(TimeSpan.FromSeconds(30));
if (message != null)
{
    Console.WriteLine($"Received: {message.Body}");
    await receiver.CompleteMessageAsync(message);
}

// Receive batch
var messages = await receiver.ReceiveMessagesAsync(maxMessages: 10, maxWaitTime: TimeSpan.FromSeconds(30));
foreach (var msg in messages)
{
    Console.WriteLine($"Received: {msg.Body}");
    await receiver.CompleteMessageAsync(msg);
}

// Receive from subscription
var receiver = client.CreateReceiver(topicName, subscriptionName);
```

**ServiceBusProcessor (Recommended):**

```csharp
// Create processor
var processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions
{
    AutoCompleteMessages = false,
    MaxConcurrentCalls = 10,
    PrefetchCount = 20
});

// Register handlers
processor.ProcessMessageAsync += async args =>
{
    Console.WriteLine($"Received: {args.Message.Body}");
    
    try
    {
        // Process message
        await ProcessMessageAsync(args.Message);
        
        // Complete on success
        await args.CompleteMessageAsync(args.Message);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error: {ex.Message}");
        // Message will be abandoned automatically
    }
};

processor.ProcessErrorAsync += async args =>
{
    Console.WriteLine($"Error: {args.Exception.Message}");
    Console.WriteLine($"Source: {args.ErrorSource}");
};

// Start processing
await processor.StartProcessingAsync();

// Stop processing
await processor.StopProcessingAsync();
```

### 1.13 Security and Access Control

**Authentication Methods:**

#### 1. Connection Strings
```csharp
var connectionString = "Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=xxx";
var client = new ServiceBusClient(connectionString);
```

#### 2. Shared Access Signatures (SAS)
```bash
# Create authorization rule
az servicebus namespace authorization-rule create \
  --name SendOnlyRule \
  --namespace-name mynamespace \
  --resource-group myRG \
  --rights Send

az servicebus queue authorization-rule create \
  --name QueueRule \
  --queue-name myqueue \
  --namespace-name mynamespace \
  --resource-group myRG \
  --rights Send Listen
```

**SAS Rights:**
- **Send**: Send messages
- **Listen**: Receive messages
- **Manage**: Create/delete queues, topics, subscriptions

#### 3. Azure AD Authentication (Recommended)
```csharp
var client = new ServiceBusClient(
    "mynamespace.servicebus.windows.net",
    new DefaultAzureCredential());
```

**RBAC Roles:**

| Role | Permissions |
|------|-------------|
| Azure Service Bus Data Owner | Full data access |
| Azure Service Bus Data Sender | Send messages |
| Azure Service Bus Data Receiver | Receive messages |

```bash
# Assign sender role
az role assignment create \
  --role "Azure Service Bus Data Sender" \
  --assignee <principal-id> \
  --scope /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.ServiceBus/namespaces/{namespace}
```

---

## AZURE QUEUE STORAGE

### 1.14 Introduction to Azure Queue Storage

**What is Azure Queue Storage?**
- Simple, cost-effective queue service
- Part of Azure Storage account
- REST API accessible
- Millions of messages, up to 500 TB total

**Key Characteristics:**
- Simple HTTP-based access
- Messages up to 64 KB
- Message TTL up to 7 days
- Visibility timeout for processing
- No ordering guarantees
- At-least-once delivery

**Use Cases:**
- Background job processing
- Decoupled components
- Work queues
- Simple async messaging

### 1.15 Queue Storage vs Service Bus

| Feature | Queue Storage | Service Bus |
|---------|--------------|-------------|
| **Max Message Size** | 64 KB | 256 KB / 100 MB |
| **Max Queue Size** | 500 TB | 1-80 GB |
| **Message TTL** | 7 days | Unlimited |
| **Ordering** | No guarantee | FIFO with sessions |
| **Duplicate Detection** | No | Yes |
| **Transactions** | No | Yes |
| **Dead-Letter Queue** | No | Yes |
| **Topics/Subscriptions** | No | Yes |
| **Protocol** | HTTP/HTTPS | AMQP, HTTP |
| **Cost** | Very low | Higher |

**When to Use:**
- **Queue Storage**: Simple queuing, high volume, cost-sensitive
- **Service Bus**: Complex messaging, ordering, transactions, topics

### 1.16 Queue Storage Components

**URL Format:**
```
https://<storage-account>.queue.core.windows.net/<queue-name>
```

**Components:**
- **Storage Account**: Container for queues
- **Queue**: Container for messages
- **Message**: Data being stored (up to 64 KB)

### 1.17 Queue Storage Operations

**Create Queue - CLI:**

```bash
# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group myRG \
  --location eastus \
  --sku Standard_LRS

# Create queue
az storage queue create \
  --name myqueue \
  --account-name mystorageaccount

# List queues
az storage queue list \
  --account-name mystorageaccount

# Delete queue
az storage queue delete \
  --name myqueue \
  --account-name mystorageaccount
```

**Message Operations - CLI:**

```bash
# Add message
az storage message put \
  --queue-name myqueue \
  --account-name mystorageaccount \
  --content "Hello, Queue!"

# Get messages (peek, don't delete)
az storage message peek \
  --queue-name myqueue \
  --account-name mystorageaccount

# Get messages (dequeue)
az storage message get \
  --queue-name myqueue \
  --account-name mystorageaccount \
  --visibility-timeout 30

# Delete message
az storage message delete \
  --queue-name myqueue \
  --account-name mystorageaccount \
  --id <message-id> \
  --pop-receipt <pop-receipt>

# Clear all messages
az storage message clear \
  --queue-name myqueue \
  --account-name mystorageaccount
```

### 1.18 Queue Storage SDK for .NET

**Installation:**
```bash
dotnet add package Azure.Storage.Queues
```

**QueueClient Operations:**

```csharp
using Azure.Storage.Queues;
using Azure.Storage.Queues.Models;

// Create client
var queueClient = new QueueClient(connectionString, queueName);

// Create with Azure AD
var queueClient = new QueueClient(
    new Uri("https://mystorageaccount.queue.core.windows.net/myqueue"),
    new DefaultAzureCredential());

// Create queue if not exists
await queueClient.CreateIfNotExistsAsync();

// Send message
await queueClient.SendMessageAsync("Hello, Queue!");

// Send with visibility delay (invisible for 60 seconds)
await queueClient.SendMessageAsync(
    "Delayed message",
    visibilityTimeout: TimeSpan.FromSeconds(60));

// Send with TTL
await queueClient.SendMessageAsync(
    "Short-lived message",
    timeToLive: TimeSpan.FromMinutes(5));

// Send Base64 encoded message (for binary or special chars)
var base64Message = Convert.ToBase64String(Encoding.UTF8.GetBytes("Hello"));
await queueClient.SendMessageAsync(base64Message);
```

**Receiving Messages:**

```csharp
// Receive single message
QueueMessage message = await queueClient.ReceiveMessageAsync();
if (message != null)
{
    Console.WriteLine($"Message: {message.Body}");
    Console.WriteLine($"ID: {message.MessageId}");
    Console.WriteLine($"PopReceipt: {message.PopReceipt}");
    Console.WriteLine($"DequeueCount: {message.DequeueCount}");
    
    // Delete after processing
    await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
}

// Receive multiple messages
QueueMessage[] messages = await queueClient.ReceiveMessagesAsync(maxMessages: 10);
foreach (var msg in messages)
{
    Console.WriteLine($"Message: {msg.Body}");
    await queueClient.DeleteMessageAsync(msg.MessageId, msg.PopReceipt);
}

// Peek without removing
PeekedMessage peeked = await queueClient.PeekMessageAsync();
Console.WriteLine($"Peeked: {peeked.Body}");

// Peek multiple
PeekedMessage[] peekedMessages = await queueClient.PeekMessagesAsync(maxMessages: 5);
```

**Queue Management:**

```csharp
// Get queue properties
QueueProperties properties = await queueClient.GetPropertiesAsync();
Console.WriteLine($"Approximate message count: {properties.ApproximateMessagesCount}");

// Update message (extend visibility, change content)
await queueClient.UpdateMessageAsync(
    message.MessageId,
    message.PopReceipt,
    "Updated content",
    TimeSpan.FromSeconds(60));

// Clear all messages
await queueClient.ClearMessagesAsync();

// Delete queue
await queueClient.DeleteIfExistsAsync();
```

**Processing Pattern:**

```csharp
// Continuous processing loop
while (true)
{
    QueueMessage[] messages = await queueClient.ReceiveMessagesAsync(
        maxMessages: 10,
        visibilityTimeout: TimeSpan.FromMinutes(5));
    
    if (messages.Length == 0)
    {
        // No messages, wait before polling again
        await Task.Delay(TimeSpan.FromSeconds(10));
        continue;
    }
    
    foreach (var message in messages)
    {
        try
        {
            // Process message
            await ProcessMessageAsync(message.Body.ToString());
            
            // Delete on success
            await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error processing: {ex.Message}");
            // Message becomes visible again after timeout
            // Check DequeueCount to implement dead-letter logic
            if (message.DequeueCount > 5)
            {
                // Move to error queue manually
                await errorQueueClient.SendMessageAsync(message.Body.ToString());
                await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
            }
        }
    }
}
```

### 1.19 Visibility Timeout

**What is Visibility Timeout?**
- Period message is invisible after dequeue
- Prevents other consumers from processing same message
- Default: 30 seconds
- Maximum: 7 days

**Visibility Timeout Flow:**

```
1. Consumer A receives message → Message becomes invisible
2. Consumer A processes message (within timeout)
3a. Success: Consumer A deletes message → Message removed
3b. Failure: Visibility timeout expires → Message visible again
4. Consumer B (or A retry) receives message
```

```csharp
// Custom visibility timeout
var message = await queueClient.ReceiveMessageAsync(
    visibilityTimeout: TimeSpan.FromMinutes(5));

// Extend visibility timeout during long processing
await queueClient.UpdateMessageAsync(
    message.MessageId,
    message.PopReceipt,
    message.Body.ToString(),
    TimeSpan.FromMinutes(10));  // New timeout from now
```

### 1.20 Queue Storage Security

**Authentication Methods:**

#### 1. Storage Account Keys
```csharp
var connectionString = "DefaultEndpointsProtocol=https;AccountName=mystorageaccount;AccountKey=xxx;EndpointSuffix=core.windows.net";
var queueClient = new QueueClient(connectionString, queueName);
```

#### 2. Shared Access Signature (SAS)
```bash
# Generate SAS token
az storage queue generate-sas \
  --name myqueue \
  --account-name mystorageaccount \
  --permissions raup \
  --expiry 2024-12-31T23:59:59Z
```

```csharp
// Use SAS token
var sasUri = new Uri("https://mystorageaccount.queue.core.windows.net/myqueue?sv=2021-06-08&ss=q&srt=sco&sp=rwdlacup&se=2024-12-31&st=2024-01-01&spr=https&sig=xxx");
var queueClient = new QueueClient(sasUri);
```

**SAS Permissions:**
- **r**: Read (peek messages)
- **a**: Add (send messages)
- **u**: Update (update messages)
- **p**: Process (get and delete messages)

#### 3. Azure AD Authentication
```csharp
var queueClient = new QueueClient(
    new Uri("https://mystorageaccount.queue.core.windows.net/myqueue"),
    new DefaultAzureCredential());
```

**RBAC Roles:**

| Role | Permissions |
|------|-------------|
| Storage Queue Data Contributor | Full access |
| Storage Queue Data Reader | Read (peek) |
| Storage Queue Data Message Processor | Process messages |
| Storage Queue Data Message Sender | Send messages |

---

## CHOOSING BETWEEN QUEUE SOLUTIONS

### Decision Matrix

| Requirement | Service Bus | Queue Storage |
|-------------|-------------|---------------|
| Simple async processing | ✓ | ✓✓ |
| Large messages (>64KB) | ✓✓ | ✗ |
| Message ordering (FIFO) | ✓✓ | ✗ |
| Duplicate detection | ✓✓ | ✗ |
| Dead-letter queue | ✓✓ | ✗ |
| Topics (pub/sub) | ✓✓ | ✗ |
| Transactions | ✓✓ | ✗ |
| Low cost | ✓ | ✓✓ |
| Long retention (>7 days) | ✓✓ | ✗ |
| High volume (TB of messages) | ✓ | ✓✓ |

### Recommendations

**Use Azure Service Bus when:**
- Need ordered delivery (FIFO)
- Need pub/sub with topics
- Need dead-letter handling
- Need transactions
- Messages larger than 64 KB
- Complex routing required

**Use Azure Queue Storage when:**
- Simple background processing
- Cost is primary concern
- Very high volume
- Simple polling model acceptable
- No ordering requirements
- Integration with Azure Functions

---

## MODULE 2: SERVICE BUS ADVANCED CONCEPTS

### 2.1 Service Bus Premium Deep Dive

**Why Premium Tier?**
- **Dedicated resources** — Fixed processing capacity, no noisy neighbor
- **Large messages** — Up to 100 MB (Standard: 256 KB)
- **VNet integration** — Private Endpoints, Service Endpoints
- **Availability Zones** — Zone-redundant deployment
- **Messaging Units (MUs)** — Scale processing capacity (1, 2, 4, 8, 16 MUs)
- **JMS 2.0 support** — Java Message Service over AMQP

**Messaging Units (MU):**

| MUs | CPU/Memory | Recommended Throughput |
|-----|-----------|----------------------|
| 1 | Dedicated baseline | ~1,000 messages/sec |
| 2 | 2× resources | ~2,000 messages/sec |
| 4 | 4× resources | ~4,000 messages/sec |
| 8 | 8× resources | ~8,000 messages/sec |
| 16 | 16× resources | ~16,000 messages/sec |

```bash
# Create Premium namespace with 2 messaging units
az servicebus namespace create \
  --name mypremiumns \
  --resource-group myRG \
  --location eastus \
  --sku Premium \
  --capacity 2

# Scale messaging units
az servicebus namespace update \
  --name mypremiumns \
  --resource-group myRG \
  --capacity 4
```

**Premium vs Standard — Key Differences:**

| Feature | Standard | Premium |
|---------|----------|---------|
| **Message size** | 256 KB | 100 MB |
| **Resources** | Shared (multi-tenant) | Dedicated (single-tenant) |
| **Throughput** | Variable | Predictable |
| **VNet/Private Link** | ❌ | ✅ |
| **Availability Zones** | ❌ | ✅ |
| **Geo-DR** | ✅ (metadata) | ✅ (metadata) |
| **Scale unit** | N/A | Messaging Units |
| **Auto-scaling** | N/A | ❌ (manual MU scaling) |

### 2.2 Service Bus Geo-Disaster Recovery

**What is Geo-DR?**
- Paired namespaces in different regions
- Automatic **metadata** replication (queues, topics, subscriptions, rules, SAS policies)
- Manual failover
- **Does NOT replicate message data** (only metadata!)

**Geo-DR Architecture:**
```
Primary Namespace (East US)  ←── Alias ──→  Secondary Namespace (West US)
├── Queue A (with messages)                  ├── Queue A (empty - no message data)
├── Topic B                                  ├── Topic B
│   ├── Subscription 1                       │   ├── Subscription 1
│   └── Subscription 2                       │   └── Subscription 2
└── SAS Policies                             └── SAS Policies
```

**Key Points:**
- Applications connect via **alias connection string** (not namespace directly)
- After failover, alias points to secondary
- **Messages in-flight in primary are LOST** on failover
- Secondary must be in a **different region**
- Secondary namespace **must be empty** (no existing entities) before pairing
- Premium tier provides better Geo-DR experience

```bash
# Create alias (pair namespaces)
az servicebus georecovery-alias set \
  --resource-group myRG \
  --namespace-name primary-namespace \
  --alias my-alias \
  --partner-namespace /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.ServiceBus/namespaces/secondary-namespace

# Get alias connection string
az servicebus georecovery-alias authorization-rule keys list \
  --resource-group myRG \
  --namespace-name primary-namespace \
  --alias my-alias \
  --name RootManageSharedAccessKey

# Initiate failover (run against secondary)
az servicebus georecovery-alias fail-over \
  --resource-group myRG \
  --namespace-name secondary-namespace \
  --alias my-alias

# Break pairing (without failover)
az servicebus georecovery-alias break-pair \
  --resource-group myRG \
  --namespace-name primary-namespace \
  --alias my-alias
```

### 2.3 Service Bus Networking & Security

**Network Isolation (Premium Only):**

#### IP Firewall
```bash
# Configure IP firewall
az servicebus namespace network-rule-set update \
  --resource-group myRG \
  --namespace-name mynamespace \
  --default-action Deny \
  --ip-rules '[{"ipMask":"10.0.0.0/8","action":"Allow"},{"ipMask":"203.0.113.5","action":"Allow"}]'
```

#### VNet Service Endpoints
```bash
# Enable service endpoint on subnet
az network vnet subnet update \
  --resource-group myRG \
  --vnet-name myVNet \
  --name mySubnet \
  --service-endpoints Microsoft.ServiceBus

# Add VNet rule to namespace
az servicebus namespace network-rule-set update \
  --resource-group myRG \
  --namespace-name mynamespace \
  --default-action Deny \
  --virtual-network-rules '[{"subnet":{"id":"/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/{vnet}/subnets/{subnet}"},"ignoreMissingVnetServiceEndpoint":false}]'
```

#### Private Endpoints
```bash
# Create private endpoint
az network private-endpoint create \
  --name myPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet mySubnet \
  --private-connection-resource-id /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.ServiceBus/namespaces/{namespace} \
  --group-id namespace \
  --connection-name myConnection
```

**Managed Identity with Service Bus:**
```csharp
// System-assigned managed identity
var client = new ServiceBusClient(
    "mynamespace.servicebus.windows.net",
    new DefaultAzureCredential());

// User-assigned managed identity
var client = new ServiceBusClient(
    "mynamespace.servicebus.windows.net",
    new ManagedIdentityCredential("client-id-of-user-assigned-identity"));
```

### 2.4 Service Bus Message Properties Deep Dive

**System Properties (Set by Service Bus or SDK):**

| Property | Description | Set By |
|----------|-------------|--------|
| `MessageId` | Unique identifier (for duplicate detection) | Sender |
| `SessionId` | Session identifier (for session-enabled entities) | Sender |
| `CorrelationId` | Correlation identifier (for request-reply) | Sender |
| `ContentType` | MIME type (e.g., `application/json`) | Sender |
| `Subject` / `Label` | Application-defined label | Sender |
| `To` | Destination address (for routing) | Sender |
| `ReplyTo` | Reply-to address | Sender |
| `ReplyToSessionId` | Reply session ID | Sender |
| `TimeToLive` | Message TTL | Sender |
| `ScheduledEnqueueTimeUtc` | Schedule delivery time | Sender |
| `PartitionKey` | Partition key (partitioned entities) | Sender |
| `SequenceNumber` | Unique number assigned by Service Bus | Service Bus |
| `EnqueuedTimeUtc` | Time message was enqueued | Service Bus |
| `ExpiresAtUtc` | Calculated expiry time | Service Bus |
| `LockedUntilUtc` | Lock expiry time (PeekLock) | Service Bus |
| `DeliveryCount` | Number of delivery attempts | Service Bus |
| `DeadLetterSource` | Original entity (when dead-lettered) | Service Bus |

**Custom (Application) Properties:**
```csharp
var message = new ServiceBusMessage("Order data")
{
    MessageId = "order-123",
    ContentType = "application/json",
    Subject = "NewOrder",
    CorrelationId = Guid.NewGuid().ToString(),
    TimeToLive = TimeSpan.FromHours(1),
    SessionId = "customer-456",
    ReplyTo = "response-queue",
    ScheduledEnqueueTimeUtc = DateTimeOffset.UtcNow.AddMinutes(30)
};

// Custom application properties
message.ApplicationProperties["Priority"] = "High";
message.ApplicationProperties["Region"] = "US-East";
message.ApplicationProperties["OrderType"] = "Express";
message.ApplicationProperties["Amount"] = 99.99;
```

### 2.5 Message Settlement Deep Dive

**Five Settlement Actions (PeekLock mode):**

| Action | Method | Effect |
|--------|--------|--------|
| **Complete** | `CompleteMessageAsync()` | Remove from queue (success) |
| **Abandon** | `AbandonMessageAsync()` | Return to queue, increment DeliveryCount |
| **Dead-Letter** | `DeadLetterMessageAsync()` | Move to DLQ with reason |
| **Defer** | `DeferMessageAsync()` | Keep in queue but only retrievable by SequenceNumber |
| **Renew Lock** | `RenewMessageLockAsync()` | Extend lock duration |

**Settlement Best Practices:**
```csharp
processor.ProcessMessageAsync += async args =>
{
    try
    {
        var body = args.Message.Body.ToString();
        
        // Validate message
        if (!IsValidMessage(body))
        {
            // Permanently reject — move to DLQ
            await args.DeadLetterMessageAsync(args.Message, 
                deadLetterReason: "InvalidFormat",
                deadLetterErrorDescription: "Message body failed validation");
            return;
        }
        
        // Check if we should process now
        if (!IsReadyToProcess(body))
        {
            // Defer for later retrieval by SequenceNumber
            await args.DeferMessageAsync(args.Message);
            return;
        }
        
        // Process message
        await ProcessAsync(body);
        
        // Success — complete
        await args.CompleteMessageAsync(args.Message);
    }
    catch (TransientException)
    {
        // Transient error — abandon (will retry)
        await args.AbandonMessageAsync(args.Message);
    }
    catch (PermanentException ex)
    {
        // Permanent error — dead-letter
        await args.DeadLetterMessageAsync(args.Message,
            deadLetterReason: "ProcessingError",
            deadLetterErrorDescription: ex.Message);
    }
};
```

**Retrieving Deferred Messages:**
```csharp
// Deferred messages can ONLY be retrieved by SequenceNumber
long sequenceNumber = 12345; // Stored earlier when message was deferred
ServiceBusReceivedMessage deferredMsg = await receiver.ReceiveDeferredMessageAsync(sequenceNumber);
await receiver.CompleteMessageAsync(deferredMsg);

// Receive multiple deferred messages
long[] sequenceNumbers = new long[] { 12345, 12346, 12347 };
IReadOnlyList<ServiceBusReceivedMessage> deferredMsgs = 
    await receiver.ReceiveDeferredMessagesAsync(sequenceNumbers);
```

### 2.6 Batching and Prefetch

**Client-Side Batching:**
```csharp
// ServiceBusSender automatically batches when sending a list
var messages = new List<ServiceBusMessage>();
for (int i = 0; i < 100; i++)
{
    messages.Add(new ServiceBusMessage($"Message {i}"));
}

// Option 1: Use MessageBatch (recommended for large batches)
using ServiceBusMessageBatch batch = await sender.CreateMessageBatchAsync();
foreach (var msg in messages)
{
    if (!batch.TryAddMessage(msg))
    {
        await sender.SendMessagesAsync(batch);
        batch = await sender.CreateMessageBatchAsync();
        batch.TryAddMessage(msg);
    }
}
if (batch.Count > 0) await sender.SendMessagesAsync(batch);

// Option 2: Send list directly (SDK handles batching)
await sender.SendMessagesAsync(messages);
```

**Prefetch:**
- Fetches extra messages in background when Receive is called
- Reduces round trips to Service Bus
- Improves throughput for high-volume scenarios

```csharp
// Enable prefetch on receiver
var receiver = client.CreateReceiver(queueName, new ServiceBusReceiverOptions
{
    PrefetchCount = 50  // Prefetch 50 messages
});

// Enable prefetch on processor
var processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions
{
    PrefetchCount = 100,
    MaxConcurrentCalls = 10
});
```

**Prefetch Guidelines:**
- Set `PrefetchCount` ≥ `MaxConcurrentCalls`
- Higher prefetch = better throughput but more memory
- Be cautious with short lock durations (messages may expire in prefetch buffer)
- For sessions: prefetch applies per session

### 2.7 Service Bus Correlation Patterns

**Request-Reply Pattern:**
```
Sender → Request Queue → Processor
                              ↓
Sender ← Reply Queue ← Processor
```

```csharp
// SENDER: Send request and wait for reply
var requestMessage = new ServiceBusMessage("Process this order")
{
    MessageId = Guid.NewGuid().ToString(),
    ReplyTo = "reply-queue",                    // Tell processor where to reply
    ReplyToSessionId = "request-session-123"    // Session ID for reply
};
await requestSender.SendMessageAsync(requestMessage);

// Wait for reply on reply queue
var replyReceiver = await client.AcceptSessionAsync("reply-queue", "request-session-123");
var reply = await replyReceiver.ReceiveMessageAsync(TimeSpan.FromSeconds(30));
Console.WriteLine($"Reply: {reply.Body}");
await replyReceiver.CompleteMessageAsync(reply);

// PROCESSOR: Process and send reply
processor.ProcessMessageAsync += async args =>
{
    var request = args.Message;
    
    // Process the request
    var result = await ProcessOrder(request.Body.ToString());
    
    // Send reply
    var replyMessage = new ServiceBusMessage(result)
    {
        CorrelationId = request.MessageId,      // Correlate reply to request
        SessionId = request.ReplyToSessionId    // Route to correct session
    };
    
    var replySender = client.CreateSender(request.ReplyTo);
    await replySender.SendMessageAsync(replyMessage);
    
    await args.CompleteMessageAsync(request);
};
```

**Transfer Dead-Letter Queue:**
- When auto-forwarding fails, messages go to the **transfer dead-letter queue**
- Path: `<entity>/$Transfer/$DeadLetterQueue`
- Separate from the regular DLQ

### 2.8 Programmatic Management with ServiceBusAdministrationClient

```csharp
using Azure.Messaging.ServiceBus.Administration;

// Create administration client
var adminClient = new ServiceBusAdministrationClient(connectionString);
// OR with Azure AD
var adminClient = new ServiceBusAdministrationClient(
    "mynamespace.servicebus.windows.net",
    new DefaultAzureCredential());

// Create queue programmatically
var queueOptions = new CreateQueueOptions("myqueue")
{
    MaxSizeInMegabytes = 5120,
    DefaultMessageTimeToLive = TimeSpan.FromDays(14),
    LockDuration = TimeSpan.FromMinutes(2),
    MaxDeliveryCount = 10,
    EnableBatchedOperations = true,
    DeadLetteringOnMessageExpiration = true,
    RequiresDuplicateDetection = true,
    DuplicateDetectionHistoryTimeWindow = TimeSpan.FromMinutes(10),
    RequiresSession = false
};
await adminClient.CreateQueueAsync(queueOptions);

// Create topic
var topicOptions = new CreateTopicOptions("mytopic")
{
    MaxSizeInMegabytes = 5120,
    DefaultMessageTimeToLive = TimeSpan.FromDays(14),
    EnableBatchedOperations = true,
    SupportOrdering = true
};
await adminClient.CreateTopicAsync(topicOptions);

// Create subscription with SQL filter
var subOptions = new CreateSubscriptionOptions("mytopic", "highpriority")
{
    MaxDeliveryCount = 10,
    LockDuration = TimeSpan.FromMinutes(1),
    DefaultMessageTimeToLive = TimeSpan.FromDays(7)
};
var ruleOptions = new CreateRuleOptions("HighPriorityFilter", new SqlRuleFilter("Priority = 'High'"))
{
    Action = new SqlRuleAction("SET Processed = true")
};
await adminClient.CreateSubscriptionAsync(subOptions, ruleOptions);

// Check if queue exists
bool exists = await adminClient.QueueExistsAsync("myqueue");

// Get queue runtime properties (message count, etc.)
QueueRuntimeProperties runtimeProps = await adminClient.GetQueueRuntimePropertiesAsync("myqueue");
Console.WriteLine($"Active messages: {runtimeProps.ActiveMessageCount}");
Console.WriteLine($"Dead-lettered: {runtimeProps.DeadLetterMessageCount}");
Console.WriteLine($"Scheduled: {runtimeProps.ScheduledMessageCount}");
Console.WriteLine($"Transfer DLQ: {runtimeProps.TransferDeadLetterMessageCount}");

// Update queue
QueueProperties queueProps = await adminClient.GetQueueAsync("myqueue");
queueProps.MaxDeliveryCount = 15;
await adminClient.UpdateQueueAsync(queueProps);

// Delete queue
await adminClient.DeleteQueueAsync("myqueue");

// List all queues
await foreach (QueueProperties queue in adminClient.GetQueuesAsync())
{
    Console.WriteLine($"Queue: {queue.Name}, MaxSize: {queue.MaxSizeInMegabytes} MB");
}
```

### 2.9 Service Bus Message Expiration & TTL

**TTL Hierarchy:**
```
Entity-level TTL (queue/topic default)
    └── Message-level TTL (overrides entity default)
        └── Effective expiry = EnqueuedTimeUtc + min(EntityTTL, MessageTTL)
```

```csharp
// Message-level TTL
var message = new ServiceBusMessage("Short-lived message")
{
    TimeToLive = TimeSpan.FromMinutes(5)
};

// Entity-level TTL (set at creation)
var queueOptions = new CreateQueueOptions("myqueue")
{
    DefaultMessageTimeToLive = TimeSpan.FromDays(14)
};
```

**When Message Expires:**
1. If `DeadLetteringOnMessageExpiration = true` → message moves to DLQ
2. If `DeadLetteringOnMessageExpiration = false` → message is deleted silently
3. Expired messages are not immediately removed — cleaned up during next receive attempt

### 2.10 Service Bus Quotas & Limits

| Resource | Standard | Premium |
|----------|----------|---------|
| **Queues/Topics per namespace** | 10,000 | 1,000 |
| **Queue/Topic size** | 1-80 GB | 1-80 GB (per MU) |
| **Message size** | 256 KB | 100 MB |
| **Concurrent connections** | 5,000 | 25,000 per MU |
| **Receive rate** | ~4,000 msg/sec | Higher (MU-based) |
| **SAS rules per entity** | 12 | 12 |
| **Subscriptions per topic** | 2,000 | 2,000 |
| **SQL filters per subscription** | 2,000 | 2,000 |
| **Correlation filters per subscription** | 100,000 | 100,000 |
| **Duplicate detection window** | 5 min - 7 days | 5 min - 7 days |
| **Max lock duration** | 5 minutes | 5 minutes |
| **Max sessions** | Unlimited | Unlimited |

---

## MODULE 3: SERVICE BUS WITH AZURE FUNCTIONS

### 3.1 ServiceBusTrigger

**Single Message Processing:**
```csharp
// Isolated worker model
[Function("ServiceBusProcessor")]
public async Task Run(
    [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] ServiceBusReceivedMessage message,
    ServiceBusMessageActions messageActions)
{
    _logger.LogInformation($"Message ID: {message.MessageId}");
    _logger.LogInformation($"Body: {message.Body}");
    _logger.LogInformation($"Content Type: {message.ContentType}");
    _logger.LogInformation($"Delivery Count: {message.DeliveryCount}");
    
    try
    {
        await ProcessMessageAsync(message.Body.ToString());
        await messageActions.CompleteMessageAsync(message);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error processing message");
        await messageActions.DeadLetterMessageAsync(message, 
            deadLetterReason: "ProcessingError");
    }
}
```

**Batch Processing:**
```csharp
[Function("ServiceBusBatchProcessor")]
public async Task Run(
    [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection", 
        IsBatched = true)] ServiceBusReceivedMessage[] messages,
    ServiceBusMessageActions messageActions)
{
    foreach (var message in messages)
    {
        try
        {
            await ProcessMessageAsync(message.Body.ToString());
            await messageActions.CompleteMessageAsync(message);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, $"Failed message: {message.MessageId}");
            await messageActions.AbandonMessageAsync(message);
        }
    }
}
```

**Topic Subscription Trigger:**
```csharp
[Function("TopicProcessor")]
public async Task Run(
    [ServiceBusTrigger("mytopic", "mysubscription", 
        Connection = "ServiceBusConnection")] ServiceBusReceivedMessage message,
    ServiceBusMessageActions messageActions)
{
    _logger.LogInformation($"Topic message: {message.Body}");
    await messageActions.CompleteMessageAsync(message);
}
```

**Session-Enabled Trigger:**
```csharp
[Function("SessionProcessor")]
public async Task Run(
    [ServiceBusTrigger("mysessionqueue", Connection = "ServiceBusConnection",
        IsSessionsEnabled = true)] ServiceBusReceivedMessage message,
    ServiceBusMessageActions messageActions)
{
    _logger.LogInformation($"Session: {message.SessionId}");
    _logger.LogInformation($"Body: {message.Body}");
    await messageActions.CompleteMessageAsync(message);
}
```

### 3.2 ServiceBusOutput Binding

```csharp
// Output binding — send message to queue
[Function("SendToQueue")]
[ServiceBusOutput("myqueue", Connection = "ServiceBusConnection")]
public ServiceBusMessage Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req)
{
    var message = new ServiceBusMessage("Hello from Function!")
    {
        ContentType = "application/json",
        Subject = "FunctionOutput"
    };
    message.ApplicationProperties["Source"] = "AzureFunction";
    return message;
}

// Output binding — send to topic
[Function("SendToTopic")]
[ServiceBusOutput("mytopic", Connection = "ServiceBusConnection")]
public string Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req)
{
    return "{\"message\": \"Hello from Function to Topic!\"}";
}

// Output binding — send multiple messages
[Function("SendMultiple")]
[ServiceBusOutput("myqueue", Connection = "ServiceBusConnection")]
public ServiceBusMessage[] Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo timer)
{
    return new ServiceBusMessage[]
    {
        new ServiceBusMessage("Message 1"),
        new ServiceBusMessage("Message 2"),
        new ServiceBusMessage("Message 3")
    };
}
```

### 3.3 host.json Configuration for Service Bus

```json
{
  "version": "2.0",
  "extensions": {
    "serviceBus": {
      "prefetchCount": 100,
      "messageHandlerOptions": {
        "autoComplete": true,
        "maxConcurrentCalls": 16,
        "maxAutoRenewDuration": "00:05:00"
      },
      "sessionHandlerOptions": {
        "autoComplete": true,
        "maxConcurrentSessions": 2000,
        "maxAutoRenewDuration": "00:05:00",
        "messageWaitTimeout": "00:00:30"
      },
      "batchOptions": {
        "maxMessageCount": 1000,
        "operationTimeout": "00:01:00",
        "autoComplete": true
      },
      "clientRetryOptions": {
        "mode": "Exponential",
        "tryTimeout": "00:01:00",
        "delay": "00:00:00.80",
        "maxDelay": "00:01:00",
        "maxRetries": 3
      },
      "transportType": "amqpWebSockets",
      "maxMessageBatchSize": 1000,
      "minMessageBatchSize": 1,
      "maxBatchWaitTime": "00:00:30"
    }
  }
}
```

**Key Settings Explained:**

| Setting | Description | Default |
|---------|-------------|---------|
| `prefetchCount` | Messages prefetched from Service Bus | 0 |
| `maxConcurrentCalls` | Max concurrent message handlers | 16 |
| `autoComplete` | Auto-complete messages on success | true |
| `maxAutoRenewDuration` | Auto-renew lock duration | 5 min |
| `maxConcurrentSessions` | Max concurrent sessions | 2,000 |
| `messageWaitTimeout` | Wait time for session message | 30 sec |

**Important:**
- When `autoComplete = true`: Function runtime completes message on success, abandons on exception
- When `autoComplete = false`: You must explicitly call `CompleteMessageAsync()` or use `ServiceBusMessageActions`
- For poison message handling: after `maxDeliveryCount` is exceeded, message goes to DLQ (Service Bus handles this automatically)

### 3.4 Connection Settings

```json
// local.settings.json
{
  "Values": {
    "ServiceBusConnection": "Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=xxx",
    
    // OR for Managed Identity (recommended):
    "ServiceBusConnection__fullyQualifiedNamespace": "mynamespace.servicebus.windows.net"
  }
}
```

**Azure AD / Managed Identity Connection:**
- Use `__fullyQualifiedNamespace` suffix instead of connection string
- Assign appropriate RBAC role (e.g., `Azure Service Bus Data Receiver`)
- Works with both System-assigned and User-assigned managed identity

```bash
# Assign role for Function App's managed identity
az role assignment create \
  --role "Azure Service Bus Data Receiver" \
  --assignee <function-app-managed-identity-principal-id> \
  --scope /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.ServiceBus/namespaces/{namespace}
```

### 3.5 Error Handling and Retry in Functions

**Default Behavior:**
1. Function throws exception → message is **abandoned** (if autoComplete = true)
2. DeliveryCount increments
3. When DeliveryCount > MaxDeliveryCount → message moves to **DLQ**
4. No built-in exponential backoff (relies on Service Bus retry)

**Custom Error Handling with ServiceBusMessageActions:**
```csharp
[Function("ErrorHandlingProcessor")]
public async Task Run(
    [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection",
        AutoCompleteMessages = false)] ServiceBusReceivedMessage message,
    ServiceBusMessageActions messageActions)
{
    try
    {
        await ProcessMessageAsync(message.Body.ToString());
        await messageActions.CompleteMessageAsync(message);
    }
    catch (TransientException)
    {
        // Abandon — will be retried
        await messageActions.AbandonMessageAsync(message, 
            new Dictionary<string, object>
            {
                { "RetryReason", "TransientError" },
                { "LastAttempt", DateTime.UtcNow.ToString() }
            });
    }
    catch (PermanentException ex)
    {
        // Dead-letter — no retry
        await messageActions.DeadLetterMessageAsync(message,
            deadLetterReason: "PermanentError",
            deadLetterErrorDescription: ex.Message);
    }
}
```

**DLQ Processor Function:**
```csharp
// Process dead-lettered messages
[Function("DLQProcessor")]
public async Task Run(
    [ServiceBusTrigger("myqueue/$DeadLetterQueue", Connection = "ServiceBusConnection")]
    ServiceBusReceivedMessage message)
{
    _logger.LogWarning($"DLQ Message: {message.MessageId}");
    _logger.LogWarning($"DLQ Reason: {message.DeadLetterReason}");
    _logger.LogWarning($"DLQ Description: {message.DeadLetterErrorDescription}");
    _logger.LogWarning($"Original entity: {message.DeadLetterSource}");
    
    // Alert, log, or reprocess
    await AlertOnDeadLetterAsync(message);
}
```

---

## MODULE 4: QUEUE STORAGE ADVANCED CONCEPTS

### 4.1 Queue Storage with Azure Functions

**QueueTrigger:**
```csharp
// Isolated worker model
[Function("QueueProcessor")]
public async Task Run(
    [QueueTrigger("myqueue", Connection = "AzureWebJobsStorage")] QueueMessage message)
{
    _logger.LogInformation($"Queue message: {message.Body}");
    _logger.LogInformation($"Message ID: {message.MessageId}");
    _logger.LogInformation($"Dequeue Count: {message.DequeueCount}");
    _logger.LogInformation($"Pop Receipt: {message.PopReceipt}");
    _logger.LogInformation($"Insertion Time: {message.InsertedOn}");
    _logger.LogInformation($"Expiration Time: {message.ExpiresOn}");
    _logger.LogInformation($"Next Visible: {message.NextVisibleOn}");
}

// Simple string binding
[Function("QueueStringProcessor")]
public void Run(
    [QueueTrigger("myqueue", Connection = "AzureWebJobsStorage")] string message)
{
    _logger.LogInformation($"Message: {message}");
}

// JSON deserialization
[Function("QueueOrderProcessor")]
public void Run(
    [QueueTrigger("order-queue", Connection = "AzureWebJobsStorage")] Order order)
{
    _logger.LogInformation($"Order: {order.OrderId}, Amount: {order.Amount}");
}
```

**QueueOutput Binding:**
```csharp
// Output binding — add message to queue
[Function("AddToQueue")]
[QueueOutput("output-queue", Connection = "AzureWebJobsStorage")]
public string Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req)
{
    return "{\"orderId\": 123, \"amount\": 99.99}";
}

// Multiple output messages
[Function("AddMultipleToQueue")]
[QueueOutput("output-queue", Connection = "AzureWebJobsStorage")]
public string[] Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo timer)
{
    return new string[]
    {
        "Message 1",
        "Message 2",
        "Message 3"
    };
}
```

### 4.2 Poison Queue Handling in Azure Functions

**How It Works:**
1. Function picks up message from `myqueue`
2. Function throws exception → message becomes visible again
3. After **5 failed attempts** (default), message moves to `myqueue-poison`
4. Poison queue is created **automatically** by the runtime

**Poison Queue Name Convention:**
```
Original Queue: myqueue
Poison Queue:   myqueue-poison
```

**Processing Poison Messages:**
```csharp
// Function triggered by poison queue
[Function("PoisonQueueProcessor")]
public void Run(
    [QueueTrigger("myqueue-poison", Connection = "AzureWebJobsStorage")] QueueMessage message)
{
    _logger.LogError($"Poison message received: {message.Body}");
    _logger.LogError($"Dequeue count: {message.DequeueCount}");
    _logger.LogError($"Message ID: {message.MessageId}");
    
    // Alert, log to App Insights, or move to dead-letter storage
}
```

**host.json Configuration:**
```json
{
  "version": "2.0",
  "extensions": {
    "queues": {
      "maxPollingInterval": "00:00:02",
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

| Setting | Description | Default |
|---------|-------------|---------|
| `maxPollingInterval` | Max time between queue polls | 1 min (Consumption), 1 sec (Premium) |
| `visibilityTimeout` | Visibility timeout on failure | 0 (immediate retry) |
| `batchSize` | Messages retrieved per poll | 16 |
| `maxDequeueCount` | Attempts before poison queue | 5 |
| `newBatchThreshold` | Fetch new batch when this many remaining | batchSize / 2 |
| `messageEncoding` | Encoding: `base64` or `none` | `base64` |

**Important `messageEncoding` Note:**
- Default is `base64` — messages sent via SDK should be Base64 encoded
- Set to `none` if messages are plain text (not Base64 encoded)
- Mismatch causes deserialization errors!

### 4.3 Queue Storage Message Encoding

**Base64 Encoding (Default for Functions):**
```csharp
// Send Base64 encoded message (compatible with Functions default)
string plainText = "Hello, World!";
string base64Message = Convert.ToBase64String(Encoding.UTF8.GetBytes(plainText));
await queueClient.SendMessageAsync(base64Message);

// OR use QueueClient with Base64 encoding option
var queueClient = new QueueClient(connectionString, queueName, 
    new QueueClientOptions
    {
        MessageEncoding = QueueMessageEncoding.Base64
    });
// Now sends are automatically Base64 encoded
await queueClient.SendMessageAsync("Hello, World!"); // Auto-encoded
```

**None Encoding:**
```csharp
// Send plain text (must set messageEncoding: "none" in host.json)
var queueClient = new QueueClient(connectionString, queueName,
    new QueueClientOptions
    {
        MessageEncoding = QueueMessageEncoding.None
    });
await queueClient.SendMessageAsync("Plain text message");
```

### 4.4 Stored Access Policies

**What are Stored Access Policies?**
- Pre-defined access policies stored on the queue
- Can be referenced by SAS tokens
- Allow revoking SAS tokens by modifying/deleting the policy
- Maximum 5 stored access policies per queue

```csharp
// Create stored access policy
var permissions = new List<QueueSignedIdentifier>
{
    new QueueSignedIdentifier
    {
        Id = "policy1",
        AccessPolicy = new QueueAccessPolicy
        {
            StartsOn = DateTimeOffset.UtcNow,
            ExpiresOn = DateTimeOffset.UtcNow.AddDays(30),
            Permissions = "raup"  // read, add, update, process
        }
    }
};
await queueClient.SetAccessPolicyAsync(permissions);

// Generate SAS with stored access policy
QueueSasBuilder sasBuilder = new QueueSasBuilder
{
    Identifier = "policy1"  // Reference stored policy
};
var sasUri = queueClient.GenerateSasUri(sasBuilder);
```

### 4.5 Claim-Check Pattern for Large Messages

**Problem:** Queue Storage max message size is 64 KB, Service Bus Standard is 256 KB.

**Solution:** Store message body in Blob Storage, put reference in queue.

```csharp
// SENDER: Upload large payload to blob, send reference to queue
public async Task SendLargeMessageAsync(string payload, QueueClient queueClient, BlobContainerClient blobContainer)
{
    // Upload to blob
    string blobName = $"messages/{Guid.NewGuid()}.json";
    var blobClient = blobContainer.GetBlobClient(blobName);
    await blobClient.UploadAsync(BinaryData.FromString(payload));
    
    // Send reference to queue
    var claimCheck = new { BlobName = blobName, Timestamp = DateTime.UtcNow };
    await queueClient.SendMessageAsync(JsonSerializer.Serialize(claimCheck));
}

// RECEIVER: Get reference from queue, download payload from blob
public async Task ProcessLargeMessageAsync(QueueMessage message, BlobContainerClient blobContainer)
{
    var claimCheck = JsonSerializer.Deserialize<ClaimCheck>(message.Body.ToString());
    
    // Download actual payload
    var blobClient = blobContainer.GetBlobClient(claimCheck.BlobName);
    var response = await blobClient.DownloadContentAsync();
    string payload = response.Value.Content.ToString();
    
    // Process the full payload
    await ProcessPayload(payload);
    
    // Clean up blob after processing
    await blobClient.DeleteIfExistsAsync();
}
```

### 4.6 Queue Storage Metadata and Properties

```csharp
// Set queue metadata
var metadata = new Dictionary<string, string>
{
    { "department", "sales" },
    { "environment", "production" }
};
await queueClient.SetMetadataAsync(metadata);

// Get queue properties and metadata
QueueProperties properties = await queueClient.GetPropertiesAsync();
Console.WriteLine($"Approximate message count: {properties.ApproximateMessagesCount}");
foreach (var kvp in properties.Metadata)
{
    Console.WriteLine($"Metadata: {kvp.Key} = {kvp.Value}");
}
```

### 4.7 Queue Storage Limits

| Resource | Limit |
|----------|-------|
| Max message size | 64 KB |
| Max message TTL | 7 days (or -1 for no expiry in newer API) |
| Max visibility timeout | 7 days |
| Max queue size | 500 TB |
| Max messages per queue | Unlimited (within 500 TB) |
| Max stored access policies | 5 |
| Max batch receive | 32 messages |
| Target throughput (single queue) | Up to 2,000 messages/sec |
| Max queues per storage account | Unlimited |

---

## MODULE 5: MESSAGE-BASED ARCHITECTURE PATTERNS

### 5.1 Queue-Based Load Leveling Pattern

**Problem:** Traffic spikes overwhelm backend services.

**Solution:** Use a queue as a buffer between sender and receiver.

```
Client → [Burst: 10,000 req/sec] → Queue → [Steady: 100 req/sec] → Backend Service
```

**Benefits:**
- Backend processes at own pace
- Queue absorbs spikes
- Prevents service overload
- Enables cost optimization (no need to scale backend for peak)

**Implementation:**
```csharp
// Producer: Add work items during traffic spike
foreach (var request in incomingRequests)
{
    await queueClient.SendMessageAsync(JsonSerializer.Serialize(request));
}

// Consumer: Process at steady rate
var processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions
{
    MaxConcurrentCalls = 5,   // Control processing rate
    PrefetchCount = 10
});
```

### 5.2 Competing Consumers Pattern

**Problem:** Single consumer can't keep up with message volume.

**Solution:** Multiple consumers process from the same queue.

```
                    ┌→ Consumer Instance 1
Producer → Queue ──→ Consumer Instance 2
                    └→ Consumer Instance 3
```

**Key Points:**
- Each message processed by exactly ONE consumer
- Consumers compete for messages
- Scale consumers independently
- Use with Service Bus queues or Queue Storage
- **Service Bus PeekLock** ensures safe processing

**Implementation with Service Bus:**
```csharp
// Each consumer instance runs this code
var processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions
{
    AutoCompleteMessages = false,
    MaxConcurrentCalls = 10  // Parallel processing per instance
});

processor.ProcessMessageAsync += async args =>
{
    await ProcessOrderAsync(args.Message.Body.ToString());
    await args.CompleteMessageAsync(args.Message);
};

await processor.StartProcessingAsync();
```

### 5.3 Priority Queue Pattern

**Problem:** Some messages need faster processing than others.

**Solution:** Use separate queues for different priorities.

```
                ┌→ High Priority Queue   → Consumer (fast, more instances)
Dispatcher ────→ Medium Priority Queue  → Consumer (normal)
                └→ Low Priority Queue    → Consumer (slow, fewer instances)
```

**Implementation with Service Bus Topics:**
```csharp
// SENDER: Set priority in application properties
var message = new ServiceBusMessage(orderData)
{
    Subject = "Order"
};
message.ApplicationProperties["Priority"] = isUrgent ? "High" : "Low";
await topicSender.SendMessageAsync(message);

// SUBSCRIPTION FILTERS:
// High priority subscription: SQL filter "Priority = 'High'"
// Low priority subscription: SQL filter "Priority = 'Low'"

// HIGH PRIORITY CONSUMER: More instances, higher concurrency
var highPriorityProcessor = client.CreateProcessor("orders", "high-priority",
    new ServiceBusProcessorOptions { MaxConcurrentCalls = 20 });

// LOW PRIORITY CONSUMER: Fewer instances
var lowPriorityProcessor = client.CreateProcessor("orders", "low-priority",
    new ServiceBusProcessorOptions { MaxConcurrentCalls = 2 });
```

### 5.4 Saga Pattern with Service Bus

**Problem:** Distributed transactions across multiple services.

**Choreography (Event-Driven):**
```
Order Service → "OrderCreated" → [Queue] → Payment Service
Payment Service → "PaymentCompleted" → [Queue] → Shipping Service
Shipping Service → "ShipmentCreated" → [Queue] → Notification Service

// Compensation (rollback):
Payment Service → "PaymentFailed" → [Queue] → Order Service (cancel order)
```

**Orchestration (Coordinator-Based):**
```
Saga Orchestrator → "ProcessPayment" → [Queue] → Payment Service
Payment Service → "PaymentResult" → [Queue] → Saga Orchestrator
Saga Orchestrator → "CreateShipment" → [Queue] → Shipping Service
Shipping Service → "ShipmentResult" → [Queue] → Saga Orchestrator
```

**Key Design Considerations:**
- Each step must be **idempotent** (safe to retry)
- Each step must have a **compensating action** (undo)
- Use `CorrelationId` to track saga across messages
- Use `SessionId` for ordering within a saga
- Dead-letter queue for failed steps that need manual intervention

### 5.5 Idempotent Message Processing

**Why Idempotency?**
- Both Service Bus (PeekLock) and Queue Storage deliver **at-least-once**
- Messages may be delivered more than once (lock timeout, retry, etc.)
- Processing must produce same result regardless of duplicate delivery

**Strategies:**

**1. Deduplication Table:**
```csharp
async Task ProcessMessageIdempotently(ServiceBusReceivedMessage message)
{
    // Check if already processed
    if (await _db.MessageLog.AnyAsync(m => m.MessageId == message.MessageId))
    {
        _logger.LogWarning($"Duplicate message: {message.MessageId}");
        return; // Skip
    }
    
    // Process
    await ProcessOrderAsync(message.Body.ToString());
    
    // Record processing
    await _db.MessageLog.AddAsync(new ProcessedMessage 
    { 
        MessageId = message.MessageId, 
        ProcessedAt = DateTime.UtcNow 
    });
    await _db.SaveChangesAsync();
}
```

**2. Natural Idempotency (Upsert):**
```csharp
// Instead of INSERT, use UPSERT
await _db.Database.ExecuteSqlRawAsync(
    "MERGE INTO Orders AS target " +
    "USING (SELECT @id AS OrderId, @amount AS Amount) AS source " +
    "ON target.OrderId = source.OrderId " +
    "WHEN MATCHED THEN UPDATE SET Amount = source.Amount " +
    "WHEN NOT MATCHED THEN INSERT (OrderId, Amount) VALUES (source.OrderId, source.Amount);",
    new SqlParameter("@id", order.OrderId),
    new SqlParameter("@amount", order.Amount));
```

**3. Service Bus Duplicate Detection (Built-in):**
```bash
# Enable at queue creation
az servicebus queue create \
  --name myqueue \
  --namespace-name mynamespace \
  --resource-group myRG \
  --enable-duplicate-detection true \
  --duplicate-detection-history-time-window P1D
```

### 5.6 Poison Message Handling Strategies

**Service Bus:**
- Built-in: after `MaxDeliveryCount` (default 10) → DLQ
- Custom: check `DeliveryCount` and dead-letter early
- DLQ processor function for investigation

**Queue Storage:**
- Functions: after `maxDequeueCount` (default 5) → `-poison` queue
- Manual: check `DequeueCount` and move to error queue

**Common Poison Message Handling Pattern:**
```csharp
// Service Bus: Custom poison handling before MaxDeliveryCount
processor.ProcessMessageAsync += async args =>
{
    if (args.Message.DeliveryCount > 3)
    {
        _logger.LogWarning($"Message {args.Message.MessageId} failed 3 times. Dead-lettering.");
        await args.DeadLetterMessageAsync(args.Message,
            deadLetterReason: "MaxRetries",
            deadLetterErrorDescription: "Failed after 3 processing attempts");
        return;
    }
    
    try
    {
        await ProcessMessageAsync(args.Message);
        await args.CompleteMessageAsync(args.Message);
    }
    catch (Exception ex)
    {
        await args.AbandonMessageAsync(args.Message);
    }
};
```

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**Service Bus:**
- Queues vs Topics vs Subscriptions
- Receive modes (ReceiveAndDelete, PeekLock) and settlement actions
- Dead-letter queue (automatic conditions, manual dead-lettering, DLQ path)
- Sessions for FIFO ordering (SessionId, session state, session lock)
- Subscription filters (SQL, Correlation, Boolean) and filter actions
- Message properties (system vs custom/application properties)
- Duplicate detection (MessageId, time window)
- Transactions (TransactionScope)
- Scheduled messages and message deferral
- Auto-forwarding and transfer DLQ
- Tiers (Basic vs Standard vs Premium) — what each supports
- Premium features (100 MB messages, VNet, Private Endpoints, MUs)
- Geo-DR (metadata only, alias, manual failover)
- Networking (IP firewall, VNet, Private Endpoints — Premium only)
- Batching, prefetch, ServiceBusMessageBatch
- ServiceBusAdministrationClient for programmatic management
- Correlation patterns (Request-Reply with ReplyTo/CorrelationId)
- Message expiration and TTL hierarchy

**Queue Storage:**
- Visibility timeout (default 30 sec, max 7 days, extend during processing)
- Message lifecycle (send → invisible → visible → delete)
- Dequeue count and poison message logic
- SDK operations (SendMessage, ReceiveMessage, DeleteMessage, UpdateMessage, PeekMessage)
- SAS permissions (r, a, u, p) and Stored Access Policies (max 5)
- Message encoding (Base64 vs None) — critical for Functions compatibility
- Claim-check pattern for large messages (>64 KB)
- Queue metadata and approximate message count

**Azure Functions Integration:**
- ServiceBusTrigger (single, batch, topic subscription, session-enabled)
- ServiceBusOutput binding (single, multiple messages)
- QueueTrigger and QueueOutput binding
- host.json configuration (prefetchCount, maxConcurrentCalls, autoComplete, maxDequeueCount, batchSize)
- Poison queue handling (Queue Storage: `-poison` suffix, auto after 5 attempts)
- DLQ processor functions for Service Bus
- Connection settings (connection string vs managed identity `__fullyQualifiedNamespace`)
- ServiceBusMessageActions (Complete, Abandon, DeadLetter, Defer)

**Architecture Patterns:**
- Queue-based Load Leveling
- Competing Consumers
- Priority Queue (separate queues/subscriptions)
- Claim-Check (blob + queue reference)
- Saga (choreography vs orchestration)
- Idempotent message processing (dedup table, upsert, built-in detection)
- Poison message handling strategies

### Sample Exam Questions

#### Service Bus

**Q1:** What is the default receive mode in Service Bus?
- A) ReceiveAndDelete
- B) PeekLock
- C) AtMostOnce
- D) ExactlyOnce

**Answer: B) PeekLock**
Explanation: PeekLock is the default, requiring explicit Complete/Abandon after processing.

---

**Q2:** Which Service Bus tier supports Topics and Subscriptions?
- A) Basic
- B) Standard and Premium
- C) Premium only
- D) All tiers

**Answer: B) Standard and Premium**
Explanation: Basic tier only supports Queues. Standard and Premium support Topics.

---

**Q3:** What happens when a message exceeds MaxDeliveryCount?
- A) Message is deleted
- B) Message is moved to dead-letter queue
- C) Exception is thrown
- D) Message is returned to sender

**Answer: B) Message is moved to dead-letter queue**
Explanation: After exceeding MaxDeliveryCount, messages are automatically dead-lettered.

---

**Q4:** How do you ensure FIFO ordering in Service Bus?
- A) Use Basic tier
- B) Enable sessions
- C) Use single partition
- D) Set Priority property

**Answer: B) Enable sessions**
Explanation: Sessions with SessionId provide guaranteed FIFO ordering.

---

**Q5:** Which filter type is most efficient for matching exact property values?
- A) SQL Filter
- B) Boolean Filter
- C) Correlation Filter
- D) Custom Filter

**Answer: C) Correlation Filter**
Explanation: Correlation filters are optimized for exact matching and more efficient than SQL filters.

---

**Q6:** What is the lock duration used for in PeekLock mode?
- A) Time before message expires
- B) Time message is invisible to other receivers
- C) Time to send message
- D) Session lock duration

**Answer: B) Time message is invisible to other receivers**
Explanation: Lock duration determines how long the receiver has exclusive access to process the message.

---

**Q7:** How do you access the dead-letter queue path for a queue named "orders"?
- A) orders/deadletter
- B) orders/$DeadLetterQueue
- C) $DeadLetterQueue/orders
- D) orders.dlq

**Answer: B) orders/$DeadLetterQueue**
Explanation: The DLQ path is `<queue-name>/$DeadLetterQueue`.

---

#### Queue Storage

**Q8:** What is the maximum message size in Azure Queue Storage?
- A) 32 KB
- B) 64 KB
- C) 256 KB
- D) 1 MB

**Answer: B) 64 KB**
Explanation: Queue Storage supports messages up to 64 KB.

---

**Q9:** What is the maximum message TTL in Azure Queue Storage?
- A) 1 day
- B) 7 days
- C) 30 days
- D) Unlimited

**Answer: B) 7 days**
Explanation: Messages can have TTL up to 7 days, or -1 for no expiration (still limited to 7 days).

---

**Q10:** What is the purpose of the PopReceipt in Queue Storage?
- A) Track message count
- B) Authenticate the request
- C) Required for delete/update operations
- D) Store message metadata

**Answer: C) Required for delete/update operations**
Explanation: PopReceipt is a token received when dequeuing, required to delete or update the message.

---

**Q11:** What happens when visibility timeout expires before message is deleted?
- A) Message is deleted automatically
- B) Message becomes visible again
- C) Message is moved to error queue
- D) Exception is thrown

**Answer: B) Message becomes visible again**
Explanation: After visibility timeout, the message becomes visible for other consumers.

---

**Q12:** How do you implement dead-letter logic in Queue Storage?
- A) Use built-in DLQ
- B) Check DequeueCount and move to separate queue
- C) Set dead-letter property
- D) Enable dead-lettering in portal

**Answer: B) Check DequeueCount and move to separate queue**
Explanation: Queue Storage has no built-in DLQ. You must check DequeueCount and manually move messages.

---

#### Comparison Questions

**Q13:** Which service should you use for pub/sub messaging with multiple subscribers?
- A) Queue Storage
- B) Service Bus Queues
- C) Service Bus Topics
- D) Event Grid

**Answer: C) Service Bus Topics**
Explanation: Service Bus Topics support pub/sub with multiple subscriptions.

---

**Q14:** You need simple, low-cost background job processing without ordering requirements. Which service?
- A) Service Bus Premium
- B) Service Bus Standard
- C) Queue Storage
- D) Event Hubs

**Answer: C) Queue Storage**
Explanation: Queue Storage is ideal for simple, cost-effective background processing.

---

**Q15:** Which service supports transactions across multiple messages?
- A) Queue Storage
- B) Event Grid
- C) Service Bus Standard
- D) Event Hubs

**Answer: C) Service Bus Standard**
Explanation: Service Bus (Standard/Premium) supports transactions; Queue Storage does not.

---

**Q16:** You need to process messages larger than 64 KB. Which service must you use?
- A) Queue Storage
- B) Service Bus
- C) Either works
- D) Neither supports it

**Answer: B) Service Bus**
Explanation: Queue Storage max is 64 KB. Service Bus supports 256 KB (Standard) or 100 MB (Premium).

---

**Q17:** Which property must be unique for duplicate detection in Service Bus?
- A) CorrelationId
- B) SessionId
- C) MessageId
- D) Label

**Answer: C) MessageId**
Explanation: Duplicate detection uses MessageId to identify duplicate messages.

---

**Q18:** How do you filter messages to different subscriptions based on message content?
- A) Topic routing
- B) Subscription filters
- C) Queue partitioning
- D) Message sorting

**Answer: B) Subscription filters**
Explanation: Subscription filters (SQL, Correlation, Boolean) route messages to specific subscriptions.

---

#### Service Bus Advanced

**Q19:** Which Service Bus tier supports VNet integration and Private Endpoints?
- A) Basic
- B) Standard
- C) Premium
- D) All tiers

**Answer: C) Premium**
Explanation: VNet integration, Private Endpoints, and IP Firewall are only available in the Premium tier.

---

**Q20:** What does Service Bus Geo-Disaster Recovery replicate to the secondary namespace?
- A) Messages and metadata
- B) Only metadata (queues, topics, subscriptions, SAS policies)
- C) Only messages
- D) Nothing — it's manual backup

**Answer: B) Only metadata (queues, topics, subscriptions, SAS policies)**
Explanation: Geo-DR replicates metadata only. Message data is NOT replicated. Messages in-flight in primary are lost on failover.

---

**Q21:** What is the maximum message size in Service Bus Premium tier?
- A) 256 KB
- B) 1 MB
- C) 10 MB
- D) 100 MB

**Answer: D) 100 MB**
Explanation: Premium tier supports messages up to 100 MB. Standard tier max is 256 KB.

---

**Q22:** You need to scale Service Bus Premium for higher throughput. What do you adjust?
- A) Throughput Units (TU)
- B) Messaging Units (MU)
- C) Processing Units (PU)
- D) Capacity Units (CU)

**Answer: B) Messaging Units (MU)**
Explanation: Service Bus Premium uses Messaging Units (1, 2, 4, 8, 16 MUs) for scaling. TUs are for Event Hubs, PUs are for Event Hubs Premium.

---

**Q23:** What happens to a deferred message in Service Bus?
- A) It is deleted
- B) It can only be received by its SequenceNumber
- C) It goes to the dead-letter queue
- D) It becomes visible after a timeout

**Answer: B) It can only be received by its SequenceNumber**
Explanation: Deferred messages remain in the queue but can only be retrieved using `ReceiveDeferredMessageAsync(sequenceNumber)`. They won't appear in normal receive operations.

---

**Q24:** What happens when a Service Bus message expires (TTL reached) and `DeadLetteringOnMessageExpiration` is false?
- A) Message goes to DLQ
- B) Message is silently deleted
- C) Exception is thrown
- D) Message lock is extended

**Answer: B) Message is silently deleted**
Explanation: With `DeadLetteringOnMessageExpiration = false`, expired messages are simply deleted. Set to `true` to move them to DLQ for investigation.

---

**Q25:** What is the maximum lock duration in Service Bus?
- A) 60 seconds
- B) 2 minutes
- C) 5 minutes
- D) 30 minutes

**Answer: C) 5 minutes**
Explanation: Maximum lock duration is 5 minutes. For longer processing, use `RenewMessageLockAsync()` or the processor's `MaxAutoLockRenewalDuration`.

---

**Q26:** Which SDK class is used to programmatically create queues and topics in Service Bus?
- A) ServiceBusClient
- B) ServiceBusSender
- C) ServiceBusAdministrationClient
- D) ServiceBusProcessor

**Answer: C) ServiceBusAdministrationClient**
Explanation: ServiceBusAdministrationClient provides management operations (create, update, delete queues/topics/subscriptions). ServiceBusClient is for messaging operations.

---

**Q27:** What is the purpose of the `PrefetchCount` setting on a Service Bus receiver?
- A) Limit the number of messages in the queue
- B) Fetch extra messages in background to reduce round trips
- C) Set the maximum batch size for sending
- D) Control the number of concurrent sessions

**Answer: B) Fetch extra messages in background to reduce round trips**
Explanation: Prefetch fetches messages ahead of time, reducing latency. Set PrefetchCount ≥ MaxConcurrentCalls for best performance.

---

**Q28:** In Service Bus, what is the Transfer Dead-Letter Queue?
- A) DLQ for expired messages
- B) DLQ for messages that fail auto-forwarding
- C) DLQ for oversized messages
- D) DLQ for filtered messages

**Answer: B) DLQ for messages that fail auto-forwarding**
Explanation: When auto-forwarding fails (destination full, doesn't exist), messages go to the Transfer DLQ at path `<entity>/$Transfer/$DeadLetterQueue`.

---

**Q29:** How do you implement a Request-Reply pattern with Service Bus?
- A) Use two queues with CorrelationId and ReplyTo properties
- B) Use a single topic with filters
- C) Use sessions only
- D) Use HTTP endpoints

**Answer: A) Use two queues with CorrelationId and ReplyTo properties**
Explanation: Sender sets `ReplyTo` (reply queue name) and `MessageId`. Processor sends reply with `CorrelationId = original.MessageId` to the `ReplyTo` queue.

---

#### Queue Storage Advanced

**Q30:** What is the default number of dequeue attempts before a message goes to the poison queue in Azure Functions?
- A) 3
- B) 5
- C) 10
- D) 20

**Answer: B) 5**
Explanation: Default `maxDequeueCount` is 5 for Queue Storage triggers in Azure Functions. After 5 failed attempts, message goes to `<queuename>-poison`.

---

**Q31:** What is the name of the poison queue for a queue named "orders" in Azure Functions?
- A) orders-dlq
- B) orders-dead-letter
- C) orders-poison
- D) $DeadLetterQueue/orders

**Answer: C) orders-poison**
Explanation: Azure Functions creates a poison queue with `-poison` suffix. Service Bus uses `$DeadLetterQueue`. Don't confuse them!

---

**Q32:** You send messages to Queue Storage using the SDK without Base64 encoding, but your Azure Function fails to read them. What is the likely cause?
- A) Queue Storage doesn't support the SDK
- B) The Function's host.json `messageEncoding` is set to `base64` (default) but messages aren't Base64 encoded
- C) Queue Storage has a bug
- D) The connection string is wrong

**Answer: B) The Function's host.json `messageEncoding` is set to `base64` (default) but messages aren't Base64 encoded**
Explanation: By default, Azure Functions expects Base64-encoded messages. If sending plain text, set `messageEncoding` to `none` in host.json, or use QueueClientOptions with `MessageEncoding = QueueMessageEncoding.Base64`.

---

**Q33:** What is the maximum number of stored access policies per Queue Storage queue?
- A) 3
- B) 5
- C) 10
- D) 25

**Answer: B) 5**
Explanation: Maximum 5 stored access policies per queue. This applies to all Azure Storage services (blobs, queues, tables, file shares).

---

**Q34:** You need to send a 2 MB message using Queue Storage. How should you handle this?
- A) Increase the queue's message size limit
- B) Use the Claim-Check pattern: store payload in Blob Storage, send reference in queue message
- C) Split into multiple 64 KB messages
- D) Use Base64 encoding to compress it

**Answer: B) Use the Claim-Check pattern: store payload in Blob Storage, send reference in queue message**
Explanation: Queue Storage max message size is 64 KB. For larger payloads, use Claim-Check: store in Blob Storage and send a reference (blob URI or name) in the queue message.

---

**Q35:** How do you extend the processing time for a Queue Storage message beyond the visibility timeout?
- A) Delete and re-queue the message
- B) Call `UpdateMessageAsync()` with a new visibility timeout
- C) Set `TimeToLive` to a higher value
- D) Use message sessions

**Answer: B) Call `UpdateMessageAsync()` with a new visibility timeout**
Explanation: `UpdateMessageAsync()` can update both the content and visibility timeout, effectively extending the processing window.

---

#### Azure Functions Integration

**Q36:** In an Azure Function with ServiceBusTrigger, what happens by default when the function throws an exception?
- A) Message is completed (removed)
- B) Message is abandoned (returned to queue)
- C) Message is dead-lettered
- D) Function retries immediately

**Answer: B) Message is abandoned (returned to queue)**
Explanation: With default `autoComplete = true`, the runtime abandons the message on exception (increments DeliveryCount). On success, it auto-completes.

---

**Q37:** How do you configure a ServiceBusTrigger Azure Function to process session-enabled queues?
- A) Set `IsSessionsEnabled = true` on the trigger attribute
- B) Use `SessionId` property on the message
- C) Create a separate session processor
- D) Enable sessions in host.json

**Answer: A) Set `IsSessionsEnabled = true` on the trigger attribute**
Explanation: Use `[ServiceBusTrigger("queue", IsSessionsEnabled = true)]` for session-enabled entities. The runtime handles session acceptance automatically.

---

**Q38:** Which host.json setting controls how many messages Azure Functions retrieves per poll from Queue Storage?
- A) maxConcurrentCalls
- B) batchSize
- C) prefetchCount
- D) maxPollingInterval

**Answer: B) batchSize**
Explanation: `batchSize` (default 16) controls messages retrieved per poll for Queue Storage triggers. `maxConcurrentCalls` is for Service Bus. `prefetchCount` is for Service Bus.

---

**Q39:** How do you connect an Azure Function to Service Bus using Managed Identity instead of a connection string?
- A) Use the `__fullyQualifiedNamespace` suffix in the connection setting
- B) Set `useManagedIdentity = true` in host.json
- C) Use `DefaultAzureCredential` in the trigger attribute
- D) Enable system identity in local.settings.json

**Answer: A) Use the `__fullyQualifiedNamespace` suffix in the connection setting**
Explanation: Instead of a connection string, set `ServiceBusConnection__fullyQualifiedNamespace = "mynamespace.servicebus.windows.net"` and assign the appropriate RBAC role.

---

**Q40:** You have an Azure Function triggered by a Service Bus queue. You want to manually control when messages are completed. What should you do?
- A) Set `autoComplete = false` in host.json and use `ServiceBusMessageActions`
- B) Use `ReceiveAndDelete` mode
- C) Set `MaxDeliveryCount` to 1
- D) Use a Timer trigger instead

**Answer: A) Set `autoComplete = false` in host.json and use `ServiceBusMessageActions`**
Explanation: Set `AutoCompleteMessages = false` on the trigger attribute or `autoComplete = false` in host.json. Then use `ServiceBusMessageActions.CompleteMessageAsync()` in your function code.

---

#### Architecture Patterns

**Q41:** You have a web application that receives traffic spikes during sales events. Processing each request takes 10 seconds. How should you handle this?
- A) Scale the web application to handle peak load
- B) Use the Queue-Based Load Leveling pattern with a message queue
- C) Use Event Grid for push notifications
- D) Increase request timeout

**Answer: B) Use the Queue-Based Load Leveling pattern with a message queue**
Explanation: Queue-Based Load Leveling buffers requests in a queue, allowing backend services to process at a steady rate regardless of traffic spikes.

---

**Q42:** In the Competing Consumers pattern, how do you ensure each message is processed by only one consumer?
- A) Use Service Bus Topics
- B) Use Service Bus Queues with PeekLock
- C) Use Event Grid
- D) Use a shared database lock

**Answer: B) Use Service Bus Queues with PeekLock**
Explanation: In PeekLock mode, a message is locked to one consumer. Other consumers cannot see it until the lock expires or the message is abandoned.

---

**Q43:** You need to process financial orders where some are urgent (same-day) and others are standard (next-day). How should you architect the message flow?
- A) Single queue with priority property
- B) Service Bus Topic with subscription filters routing to separate queues by priority
- C) Event Grid with subject filtering
- D) Queue Storage with visibility timeout

**Answer: B) Service Bus Topic with subscription filters routing to separate queues by priority**
Explanation: Use a Topic with SQL filters on priority property. High-priority subscription gets more consumer instances for faster processing (Priority Queue pattern).

---

**Q44:** In the Saga pattern using Service Bus, what ensures that a failed step can be rolled back?
- A) Transactions
- B) Compensating actions with CorrelationId tracking
- C) Dead-letter queue processing
- D) Message sessions

**Answer: B) Compensating actions with CorrelationId tracking**
Explanation: Each saga step has a compensating action (undo). CorrelationId tracks the saga across services. If a step fails, compensating messages are sent.

---

**Q45:** How does Service Bus built-in duplicate detection work?
- A) It compares message body content
- B) It checks MessageId within the DuplicateDetectionHistoryTimeWindow
- C) It uses CorrelationId for deduplication
- D) It compares SessionId values

**Answer: B) It checks MessageId within the DuplicateDetectionHistoryTimeWindow**
Explanation: Service Bus keeps a history of MessageIds. If a message with the same MessageId is sent within the window (5 min to 7 days), it's silently discarded.

---

#### Comparison & Scenario Questions

**Q46:** Your application sends 100-byte messages at 50,000 messages/second. You don't need ordering or deduplication. Which is the most cost-effective solution?
- A) Service Bus Premium
- B) Service Bus Standard
- C) Azure Queue Storage
- D) Event Hubs

**Answer: C) Azure Queue Storage**
Explanation: For simple, high-volume messaging without ordering, deduplication, or transactions, Queue Storage is the most cost-effective option.

---

**Q47:** You need to send a 50 MB payload as a message. Which approach should you use?
- A) Service Bus Standard queue
- B) Service Bus Premium queue (supports up to 100 MB)
- C) Queue Storage with multiple messages
- D) Claim-Check with Blob Storage and Service Bus Standard

**Answer: B) Service Bus Premium queue (supports up to 100 MB)**
Explanation: Service Bus Premium supports messages up to 100 MB. For Standard (256 KB) or Queue Storage (64 KB), you'd need Claim-Check pattern.

---

**Q48:** Which Service Bus feature allows you to chain queues so that messages flow automatically from one to another?
- A) Sessions
- B) Auto-forwarding
- C) Duplicate detection
- D) Dead-lettering

**Answer: B) Auto-forwarding**
Explanation: Auto-forwarding automatically forwards messages from one queue/subscription to another queue/topic. Useful for building processing pipelines.

---

**Q49:** You want to process Service Bus messages in an Azure Function but need to control when messages are completed, abandoned, or dead-lettered. Which interface do you use?
- A) ILogger
- B) ServiceBusMessageActions
- C) ServiceBusClient
- D) ServiceBusAdministrationClient

**Answer: B) ServiceBusMessageActions**
Explanation: `ServiceBusMessageActions` provides Complete, Abandon, DeadLetter, and Defer operations for messages within Azure Functions Service Bus triggers.

---

**Q50:** What is the difference between the poison queue in Queue Storage (`-poison`) and the dead-letter queue in Service Bus (`$DeadLetterQueue`)?
- A) They are the same thing with different names
- B) Poison queue is created by Azure Functions runtime (after maxDequeueCount); DLQ is built into Service Bus (after MaxDeliveryCount)
- C) Poison queue is for expired messages; DLQ is for rejected messages
- D) Poison queue supports sessions; DLQ does not

**Answer: B) Poison queue is created by Azure Functions runtime (after maxDequeueCount); DLQ is built into Service Bus (after MaxDeliveryCount)**
Explanation: Queue Storage has no built-in DLQ. Azure Functions creates a `-poison` queue after `maxDequeueCount` (default 5) failures. Service Bus DLQ is a built-in sub-queue that receives messages after `MaxDeliveryCount` (default 10) or manual dead-lettering.

---

---

## Key CLI Commands Reference

### Service Bus — Basic Operations
```bash
# Create namespace
az servicebus namespace create --name mynamespace --resource-group myRG --sku Standard

# Create Premium namespace with messaging units
az servicebus namespace create --name mypremiumns --resource-group myRG --sku Premium --capacity 2

# Scale messaging units (Premium)
az servicebus namespace update --name mypremiumns --resource-group myRG --capacity 4

# Create queue
az servicebus queue create --name myqueue --namespace-name mynamespace --resource-group myRG

# Create queue with advanced options
az servicebus queue create --name myqueue --namespace-name mynamespace --resource-group myRG \
  --max-size 5120 --default-message-time-to-live P14D --lock-duration PT1M \
  --max-delivery-count 10 --enable-dead-lettering-on-message-expiration true \
  --enable-duplicate-detection true --duplicate-detection-history-time-window P1D \
  --enable-session true

# Create topic
az servicebus topic create --name mytopic --namespace-name mynamespace --resource-group myRG

# Create subscription
az servicebus topic subscription create \
  --name mysubscription --topic-name mytopic --namespace-name mynamespace --resource-group myRG

# Create filter
az servicebus topic subscription rule create \
  --name myrule --subscription-name mysubscription --topic-name mytopic \
  --namespace-name mynamespace --resource-group myRG \
  --filter-sql-expression "Priority = 'High'"

# Create correlation filter
az servicebus topic subscription rule create \
  --name corrfilter --subscription-name mysubscription --topic-name mytopic \
  --namespace-name mynamespace --resource-group myRG \
  --correlation-filter correlation-id=order-123

# Get connection string
az servicebus namespace authorization-rule keys list \
  --name RootManageSharedAccessKey --namespace-name mynamespace --resource-group myRG

# Create SAS authorization rule
az servicebus namespace authorization-rule create \
  --name SendOnlyRule --namespace-name mynamespace --resource-group myRG --rights Send

az servicebus queue authorization-rule create \
  --name QueueSendListen --queue-name myqueue --namespace-name mynamespace \
  --resource-group myRG --rights Send Listen
```

### Service Bus — Geo-DR & Networking
```bash
# Create Geo-DR alias (pair namespaces)
az servicebus georecovery-alias set \
  --resource-group myRG --namespace-name primary-ns --alias my-alias \
  --partner-namespace /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.ServiceBus/namespaces/secondary-ns

# Get alias connection string
az servicebus georecovery-alias authorization-rule keys list \
  --resource-group myRG --namespace-name primary-ns --alias my-alias \
  --name RootManageSharedAccessKey

# Initiate failover
az servicebus georecovery-alias fail-over \
  --resource-group myRG --namespace-name secondary-ns --alias my-alias

# Break pairing
az servicebus georecovery-alias break-pair \
  --resource-group myRG --namespace-name primary-ns --alias my-alias

# Set IP firewall (Premium)
az servicebus namespace network-rule-set update \
  --resource-group myRG --namespace-name mynamespace \
  --default-action Deny \
  --ip-rules '[{"ipMask":"10.0.0.0/8","action":"Allow"}]'

# Create private endpoint (Premium)
az network private-endpoint create \
  --name myPrivateEndpoint --resource-group myRG \
  --vnet-name myVNet --subnet mySubnet \
  --private-connection-resource-id <namespace-resource-id> \
  --group-id namespace --connection-name myConnection

# Auto-forwarding
az servicebus queue create --name sourcequeue --namespace-name mynamespace \
  --resource-group myRG --forward-to destinationqueue

# Forward dead-letters to error queue
az servicebus queue create --name myqueue --namespace-name mynamespace \
  --resource-group myRG --forward-dead-lettered-messages-to errorqueue

# Assign RBAC role
az role assignment create \
  --role "Azure Service Bus Data Sender" \
  --assignee <principal-id> \
  --scope /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.ServiceBus/namespaces/{namespace}
```

### Queue Storage
```bash
# Create storage account
az storage account create --name mystorageaccount --resource-group myRG --sku Standard_LRS

# Create queue
az storage queue create --name myqueue --account-name mystorageaccount

# Send message
az storage message put --queue-name myqueue --account-name mystorageaccount --content "Hello"

# Receive messages (dequeue)
az storage message get --queue-name myqueue --account-name mystorageaccount --visibility-timeout 30

# Peek messages (don't remove)
az storage message peek --queue-name myqueue --account-name mystorageaccount

# Delete message (requires id and pop-receipt)
az storage message delete --queue-name myqueue --account-name mystorageaccount \
  --id <message-id> --pop-receipt <pop-receipt>

# Clear all messages
az storage message clear --queue-name myqueue --account-name mystorageaccount

# Generate SAS token
az storage queue generate-sas --name myqueue --account-name mystorageaccount \
  --permissions raup --expiry 2026-12-31T23:59:59Z

# Get connection string
az storage account show-connection-string --name mystorageaccount --resource-group myRG

# Assign RBAC role
az role assignment create \
  --role "Storage Queue Data Contributor" \
  --assignee <principal-id> \
  --scope /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{account}
```

---

## Exam Gotchas & Tricky Distinctions

| # | Common Misconception | Reality |
|---|---------------------|---------|
| 1 | "Service Bus Basic tier supports Topics" | ❌ Basic tier only supports **Queues**. Topics require Standard or Premium tier. |
| 2 | "Service Bus Basic tier supports Sessions and Transactions" | ❌ Sessions, Transactions, Duplicate Detection, and Scheduled Messages all require **Standard or Premium**. Basic tier has none of these. |
| 3 | "VNet/Private Endpoints work on Service Bus Standard" | ❌ Network isolation (VNet, Private Endpoints, IP Firewall) is **Premium tier only**. |
| 4 | "Service Bus Geo-DR replicates messages" | ❌ Geo-DR only replicates **metadata** (namespace config, queue/topic definitions, SAS policies). **Messages in-flight are LOST** on failover. |
| 5 | "PeekLock is the same as visibility timeout in Queue Storage" | ❌ PeekLock has explicit settlement (Complete/Abandon/DeadLetter/Defer/RenewLock). Visibility timeout just makes the message reappear after the timeout — no explicit acknowledgment needed before deletion. |
| 6 | "Queue Storage has a built-in Dead-Letter Queue" | ❌ Queue Storage has **no built-in DLQ**. You must check `DequeueCount` and manually move messages to a separate queue. Azure Functions creates a `-poison` queue, but this is a Functions runtime feature, not Queue Storage itself. |
| 7 | "Service Bus DLQ path is `$deadletter`" | ❌ The correct DLQ path is `<queuename>/$DeadLetterQueue` (case-sensitive). For subscriptions: `<topic>/Subscriptions/<sub>/$DeadLetterQueue`. |
| 8 | "Azure Functions poison queue suffix is `$DeadLetterQueue`" | ❌ Functions uses `-poison` suffix for Queue Storage (e.g., `myqueue-poison`). `$DeadLetterQueue` is Service Bus terminology. Don't confuse them! |
| 9 | "Queue Storage message TTL max is always 7 days" | ⚠️ Historically 7 days max, but newer API versions allow `-1` (TimeSpan.MaxValue) for messages that never expire. However, the exam typically tests the 7-day default. |
| 10 | "Correlation Filter and SQL Filter are equally efficient" | ❌ **Correlation filters are more efficient** than SQL filters. They match exact property values using hash-based lookup. SQL filters evaluate expressions at runtime. Always prefer Correlation filters for exact matching. |
| 11 | "AutoComplete in Service Bus Functions means you can't dead-letter messages" | ❌ Even with `autoComplete = true`, you can use `ServiceBusMessageActions` to explicitly dead-letter or abandon. The auto-complete only fires if the function succeeds without explicitly settling the message. |
| 12 | "Deferred messages are eventually returned to the queue automatically" | ❌ Deferred messages **stay deferred indefinitely**. They can only be retrieved by their `SequenceNumber`. They don't become visible again on their own. |
| 13 | "Queue Storage sends messages to the poison queue after 10 failures" | ❌ Azure Functions default `maxDequeueCount` is **5** (not 10). Service Bus default `MaxDeliveryCount` is 10. Don't mix them up! |
| 14 | "Service Bus max lock duration is 30 minutes" | ❌ Max lock duration is **5 minutes**. For longer processing, use `RenewMessageLockAsync()` or set `MaxAutoLockRenewalDuration` on the processor. |
| 15 | "MessageId is required for every Service Bus message" | ❌ `MessageId` is auto-generated by the SDK if not set. It's only critical when using **duplicate detection** — the same `MessageId` within the detection window is silently discarded. |
| 16 | "Service Bus Queues and Topics can hold unlimited messages" | ❌ Queue/Topic size is capped at **1-80 GB** (configurable). Queue Storage can hold **500 TB**. Know the limits! |
| 17 | "Queue Storage max message size is 256 KB" | ❌ Queue Storage max message size is **64 KB**. Service Bus Standard is 256 KB. Service Bus Premium is 100 MB. This is a commonly confused limit! |
| 18 | "You can change `RequiresSession` on an existing queue" | ❌ The session-enabled setting is **immutable after creation**. You must delete and recreate the queue to change this property. |
| 19 | "Service Bus PrefetchCount is always beneficial" | ❌ Prefetch can cause issues with short lock durations — messages may expire in the prefetch buffer before being processed. Set PrefetchCount ≥ MaxConcurrentCalls and ensure lock duration is sufficient. |
| 20 | "Functions `messageEncoding: base64` (default) works with any SDK" | ❌ If you send plain text messages (not Base64-encoded), you must set `messageEncoding: "none"` in host.json. Default `base64` expects Base64-encoded messages and will fail on plain text. |
| 21 | "Service Bus Standard and Premium have the same message size limit" | ❌ Standard: **256 KB**. Premium: **100 MB**. This is a 400× difference! |
| 22 | "Auto-forwarding works across namespaces" | ❌ Auto-forwarding only works within the **same namespace**. Source and destination must be in the same Service Bus namespace. |
| 23 | "The `__fullyQualifiedNamespace` connection pattern works with connection strings" | ❌ `__fullyQualifiedNamespace` is for **Managed Identity / Azure AD auth only**. It replaces the connection string, not supplements it. Set it to `mynamespace.servicebus.windows.net`. |
| 24 | "Claim-Check pattern stores message in the queue and data in blob" | ✅ Correct! But remember: the sender must upload to blob first, then send the blob reference to the queue. The receiver reads the queue, downloads from blob, then deletes the blob after processing. |
| 25 | "Competing Consumers pattern works with Service Bus Topics" | ❌ Competing Consumers uses **Queues** (each message to one consumer). Topics are for **Publish-Subscribe** (each message to all subscriptions). Multiple consumers on a single subscription achieve competing consumers. |

---

## Study Tips for AZ-204 Exam

1. **Know the differences** — Service Bus vs Queue Storage is heavily tested. Master the comparison table (message size, TTL, DLQ, sessions, transactions, ordering).
2. **Master receive modes** — PeekLock (lock, Complete/Abandon/DeadLetter/Defer/RenewLock) vs ReceiveAndDelete (remove immediately). Know all 5 settlement actions.
3. **Understand dead-lettering** — Automatic conditions (expired + flag, max delivery count, filter exception, size exceeded), DLQ path format, Transfer DLQ for auto-forward failures.
4. **Know session behavior** — Enable at creation (immutable), SessionId for grouping, session state, session lock vs message lock, `AcceptSessionAsync` vs `AcceptNextSessionAsync`.
5. **Understand filters** — SQL (flexible expressions), Correlation (hash-based, efficient for exact match), Boolean (TrueFilter/FalseFilter). Correlation > SQL for performance.
6. **Learn visibility timeout** — Queue Storage processing model. Default 30 sec, max 7 days. Extend with `UpdateMessageAsync()`. Compare to PeekLock with explicit settlement.
7. **Know tier features** — Basic (queues only, no topics/sessions/transactions), Standard (topics, sessions, transactions, 256 KB), Premium (100 MB, VNet, MUs, Geo-DR pairing).
8. **Practice SDK patterns** — ServiceBusClient → Sender/Receiver/Processor. ServiceBusAdministrationClient for management. QueueClient for Queue Storage. Know batch patterns.
9. **Security concepts** — SAS rights (Send, Listen, Manage), Azure AD + RBAC roles (Data Sender, Data Receiver, Data Owner), Managed Identity connections.
10. **Functions integration** — ServiceBusTrigger (single, batch, session), QueueTrigger, output bindings, host.json settings (autoComplete, maxDequeueCount, batchSize, messageEncoding), poison queue `-poison` suffix.
11. **Poison message handling** — Queue Storage: Functions creates `-poison` queue after 5 attempts. Service Bus: built-in DLQ after `MaxDeliveryCount`. Know both paths!
12. **Architecture patterns** — Queue-Based Load Leveling (buffer spikes), Competing Consumers (parallel processing from queue), Priority Queue (separate queues), Claim-Check (large messages), Saga (compensating actions).
13. **Connection patterns** — Connection string vs `__fullyQualifiedNamespace` for Managed Identity. Queue Storage connection via `AzureWebJobsStorage` or named connection.
14. **Message encoding** — Functions default is Base64 for Queue Storage. Mismatch causes failures. Set `messageEncoding: "none"` for plain text messages.
15. **Comparison scenarios** — When to use Service Bus vs Queue Storage vs Event Hubs vs Event Grid. Know that Event Grid/Event Hubs are covered in event-based solutions.

---

## Quick Reference Summary

### Service Bus Tiers

| Feature | Basic | Standard | Premium |
|---------|-------|----------|---------|
| Queues | ✅ | ✅ | ✅ |
| Topics/Subscriptions | ❌ | ✅ | ✅ |
| Sessions | ❌ | ✅ | ✅ |
| Transactions | ❌ | ✅ | ✅ |
| Duplicate Detection | ❌ | ✅ | ✅ |
| Scheduled Messages | ❌ | ✅ | ✅ |
| Dead-Lettering | ✅ | ✅ | ✅ |
| Max Message Size | 256 KB | 256 KB | 100 MB |
| VNet / Private Endpoints | ❌ | ❌ | ✅ |
| Availability Zones | ❌ | ❌ | ✅ |
| Geo-DR | ❌ | ✅ | ✅ |
| Scale Unit | — | — | Messaging Units (1-16) |

### Service Bus Features
- **Queues**: Point-to-point, single receiver, competing consumers
- **Topics**: Pub/sub, multiple subscriptions with filters
- **Sessions**: FIFO ordering, session state, session lock (immutable at creation)
- **DLQ**: Built-in dead-letter queue, path: `<queue>/$DeadLetterQueue`
- **Filters**: SQL (flexible), Correlation (efficient hash-based), Boolean (true/false)
- **Settlement**: Complete, Abandon, DeadLetter, Defer, RenewLock
- **Advanced**: Auto-forwarding, Transactions, Scheduled messages, Duplicate detection

### Receive Modes
- **PeekLock** (default): Lock message → process → Complete/Abandon/DeadLetter/Defer. Lock max 5 min, renewable.
- **ReceiveAndDelete**: Remove immediately on receive. Faster but risk of message loss.

### Queue Storage Characteristics
- **Max Message**: 64 KB (use Claim-Check for larger)
- **Max TTL**: 7 days (or -1 for no expiry)
- **Max Queue Size**: 500 TB
- **Visibility Timeout**: 30 seconds default, 7 days max
- **Max Batch Receive**: 32 messages
- **Stored Access Policies**: 5 per queue
- **No built-in DLQ**: Check DequeueCount, use `-poison` queue in Functions

### Key SDK Classes
- **Service Bus Messaging**: `ServiceBusClient` → `ServiceBusSender`, `ServiceBusReceiver`, `ServiceBusProcessor`, `ServiceBusSessionProcessor`
- **Service Bus Management**: `ServiceBusAdministrationClient`
- **Queue Storage**: `QueueClient`, `QueueServiceClient`
- **NuGet Packages**: `Azure.Messaging.ServiceBus`, `Azure.Storage.Queues`

### Azure Functions Integration Quick Reference

| Feature | Service Bus | Queue Storage |
|---------|-------------|---------------|
| Trigger | `[ServiceBusTrigger]` | `[QueueTrigger]` |
| Output | `[ServiceBusOutput]` | `[QueueOutput]` |
| Batch mode | `IsBatched = true` | N/A (one at a time) |
| Sessions | `IsSessionsEnabled = true` | N/A |
| Settlement | `ServiceBusMessageActions` | Auto (delete on success) |
| Poison handling | DLQ (after MaxDeliveryCount) | `-poison` queue (after maxDequeueCount=5) |
| Auto complete | `autoComplete` in host.json | Always auto-delete on success |
| Managed Identity | `__fullyQualifiedNamespace` | `__queueServiceUri` |

### Message Size Comparison

| Service | Max Message Size |
|---------|-----------------|
| Queue Storage | 64 KB |
| Service Bus Basic/Standard | 256 KB |
| Service Bus Premium | 100 MB |

### RBAC Roles

**Service Bus:**

| Role | Permissions |
|------|-------------|
| Azure Service Bus Data Owner | Full data access |
| Azure Service Bus Data Sender | Send messages |
| Azure Service Bus Data Receiver | Receive messages |

**Queue Storage:**

| Role | Permissions |
|------|-------------|
| Storage Queue Data Contributor | Full access |
| Storage Queue Data Reader | Read (peek) |
| Storage Queue Data Message Processor | Process messages |
| Storage Queue Data Message Sender | Send messages |

### Architecture Patterns Quick Reference

| Pattern | Use Case | Service |
|---------|----------|---------|
| Queue-Based Load Leveling | Buffer traffic spikes | Service Bus or Queue Storage |
| Competing Consumers | Parallel processing | Service Bus Queue |
| Priority Queue | Different priority processing | Service Bus Topic + Filters |
| Claim-Check | Messages > size limit | Blob Storage + Queue |
| Request-Reply | Synchronous-style over async | Service Bus (ReplyTo + CorrelationId) |
| Saga (Choreography) | Distributed transactions | Service Bus Topics |
| Saga (Orchestration) | Coordinated transactions | Service Bus Queues |

### Key Numbers to Remember

| Metric | Value |
|--------|-------|
| Service Bus max lock duration | 5 minutes |
| Service Bus default MaxDeliveryCount | 10 |
| Service Bus duplicate detection window | 5 min - 7 days |
| Service Bus subscriptions per topic | 2,000 |
| Queue Storage default visibility timeout | 30 seconds |
| Queue Storage max visibility timeout | 7 days |
| Queue Storage max batch receive | 32 |
| Queue Storage stored access policies | 5 |
| Functions default maxDequeueCount | 5 |
| Functions default batchSize (Queue Storage) | 16 |

