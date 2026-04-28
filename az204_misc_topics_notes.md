# AZ-204: Miscellaneous Topics – Caching, Content Delivery, and IaaS Compute

## Learning Path Overview
This document captures **important AZ-204 topics that are only lightly covered across your other notes**:

1. Implement caching with **Azure Cache for Redis**
2. Configure **content delivery** with Azure CDN and Azure Front Door
3. Implement basic **IaaS compute solutions** with Virtual Machines and Virtual Machine Scale Sets

---

## MODULE 1: IMPLEMENT CACHING WITH AZURE CACHE FOR REDIS

### 1.1 What Is Azure Cache for Redis?

- Fully managed, in-memory **key-value cache** based on open-source Redis
- Sits between your app and slower data stores (SQL, Cosmos DB, storage)
- Improves **read latency**, **throughput**, and reduces load on backends

**Common Use Cases:**
- Caching frequently read data (product catalogs, reference data)
- Session state storage for web apps
- Caching output of expensive computations or queries
- Rate limiting and throttling counters

### 1.2 Tiers and Key Features

| Tier | Persistence | SLA | Network | Notes |
|------|-------------|-----|---------|------|
| Basic | No | Single node | Public | Dev/test only, no SLA for node failures |
| Standard | Optional RDB/AOF | 99.9% | Public/VNet | 2-node replicated cache, recommended minimum for prod |
| Premium | Persistence, clustering, active geo-replication | 99.9% | VNet, private endpoints | Larger sizes, better performance, Redis modules |
| Enterprise / Enterprise Flash | Advanced clustering, higher SLAs | 99.99% | VNet | Uses Redis Enterprise, Flash tier for large datasets |

**Exam angles:**
- Know **Basic vs Standard vs Premium** differences (replication, SLA, VNet, clustering, persistence).
- For **VNet integration, clustering, geo-replication** → choose **Premium or higher**.

### 1.3 Creating and Managing a Cache (CLI)

```bash
# Create a basic cache (dev/test)
az redis create \
  --name mycache \
  --resource-group myRG \
  --location eastus \
  --sku Basic \
  --vm-size C0

# Create a production-ready standard cache
az redis create \
  --name myprodcache \
  --resource-group myRG \
  --location eastus \
  --sku Standard \
  --vm-size C2 \
  --enable-non-ssl-port false

# List keys
az redis list-keys \
  --name myprodcache \
  --resource-group myRG
```

**Key configuration points:**
- Disable **non-SSL port** for production.
- Choose appropriate **capacity (C0–C7)** based on memory and throughput needs.
- For **virtual network isolation**, use Premium/Enterprise tiers and VNet integration or private endpoints.

### 1.4 Connecting from .NET (StackExchange.Redis)

```csharp
using StackExchange.Redis;

// Connection string from portal or Key Vault
var connectionString = "<cache-name>.redis.cache.windows.net:6380,password=<key>,ssl=True,abortConnect=False";

var muxer = await ConnectionMultiplexer.ConnectAsync(connectionString);
IDatabase cache = muxer.GetDatabase();

// Set a value with expiration
await cache.StringSetAsync("product:123", "Laptop", TimeSpan.FromMinutes(10));

// Get a value
string value = await cache.StringGetAsync("product:123");
```

**Exam tips:**
- Use **ConnectionMultiplexer** as a **singleton** in your app.
- Use **SSL** and store the connection string in **Key Vault** or App Configuration, not in code.

### 1.5 Caching Patterns for AZ-204

- **Cache-aside (Lazy loading)**:
  - App checks cache → if miss, load from DB → write to cache → return result.
  - Most common pattern and easiest to reason about.
- **Write-through / Write-behind**:
  - App writes to cache; cache then writes to DB synchronously (write-through) or asynchronously (write-behind).
  - More complex; less commonly emphasized on AZ-204 than cache-aside.
- **Session caching**:
  - Store ASP.NET session state or auth tokens in Redis for multi-instance web apps.

**Eviction policies:**
- Default: **volatile-lru** (least recently used among keys with TTL).
- Be aware that **keys can be evicted when memory fills**; design your app to handle cache misses gracefully.

---

## MODULE 2: CONTENT DELIVERY WITH AZURE CDN AND FRONT DOOR

### 2.1 Azure CDN – Static Content Acceleration

**What it is:**
- Global network of **edge POPs** that cache static content close to users.
- Backed by origin such as **App Service, Storage static website, or other HTTP endpoint**.

**Typical use cases:**
- Offload images, CSS/JS, videos from App Service or Storage.
- Reduce latency for static content; improve user-perceived performance.

**High-level steps:**
1. Create a **Storage account** or web app to host static assets.
2. Create a **CDN profile and endpoint**, specifying the origin.
3. Configure **caching rules**, compression, and HTTPS/custom domains.

Exam points:
- CDN is primarily for **static** content and **download acceleration**.
- For **global static website hosting**, pair **Storage static website + Azure CDN**.

### 2.2 Azure Front Door – Global HTTP Load Balancing + WAF

**What it is:**
- Global, anycast-based **reverse proxy and load balancer** for HTTP/HTTPS.
- Supports **routing**, **health probes**, **URL-based routing**, and **Web Application Firewall (WAF)**.

**Use cases:**
- Route users to the **nearest healthy backend** (multi-region deployments).
- Implement **path-based routing** (e.g., `/api` to API backend, `/static` to Storage/CDN).
- Add **WAF** for protection against common web attacks (SQL injection, XSS).

**Front Door vs CDN (exam-level):**
- **CDN**: Best for **static content caching** at the edge (images, files).
- **Front Door**: Best for **dynamic, HTTP/S traffic**, global load balancing, and WAF.
- Common pattern: **Front Door in front of regional App Services + CDN for static assets**.

---

## MODULE 3: BASIC IAAS COMPUTE – VIRTUAL MACHINES AND SCALE SETS

> These topics are lower emphasis for AZ-204 than PaaS/serverless, but they still appear in the official skills outline under “Develop Azure compute solutions”.

### 3.1 When to Use VMs vs PaaS/Serverless

- Use **App Service / Functions / Container Apps** when:
  - You want PaaS/serverless, minimal OS management.
  - Auto-scaling and deployment slots are needed.
- Use **Virtual Machines or Virtual Machine Scale Sets** when:
  - You need full control over OS/runtime (custom agents, specialized workloads).
  - You must run non-HTTP workloads or legacy applications that can’t be containerized easily.

### 3.2 Creating a Simple VM (CLI)

```bash
# Create a VM with SSH or password auth
az vm create \
  --resource-group myRG \
  --name myvm \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys

# Open HTTP port
az vm open-port \
  --resource-group myRG \
  --name myvm \
  --port 80
```

Exam focus:
- Recognize when a **VM-based solution** is required instead of Functions/App Service.
- Know that you can use **Azure SDKs, REST, or ARM/Bicep** to provision VMs programmatically.

### 3.3 Virtual Machine Scale Sets (VMSS) – Autoscale for VMs

**What they are:**
- A group of identical VMs managed as a single resource.
- Support **autoscaling** based on metrics (CPU, queue length, custom metrics).

```bash
# Create a basic scale set
az vmss create \
  --resource-group myRG \
  --name myScaleSet \
  --image UbuntuLTS \
  --instance-count 2 \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys

# Configure autoscale (example: CPU-based)
az monitor autoscale create \
  --resource-group myRG \
  --resource myScaleSet \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name myScaleSetAutoscale \
  --min-count 2 \
  --max-count 10 \
  --count 2
```

**Exam tips:**
- VMSS is the **IaaS equivalent** of scale-out that you use with App Service plans.
- For **stateless workloads** that must run on VMs (e.g., custom agents), VMSS + Autoscale is the right choice.

### 3.4 Accessing Other Azure Services from VMs (Managed Identity)

Although your `az204_secure_cloud_solutions_notes.md` already covers managed identity, remember:

```bash
# Enable system-assigned managed identity on a VM
az vm identity assign \
  --name myvm \
  --resource-group myRG
```

Then you can use **DefaultAzureCredential / ManagedIdentityCredential** in your app code on the VM to call **Key Vault, Storage, Cosmos DB, etc.**, without storing secrets.

---

## Quick Summary of “Previously Missing” Topics

- **Azure Cache for Redis**: Now has a dedicated section covering tiers, creation, and .NET usage.
- **Content delivery (CDN & Front Door)**: Summarized with when to use each and how they complement your existing App Service/blob notes.
- **IaaS compute (VMs + VM Scale Sets)**: Light but exam-relevant overview added for cases where VM-based compute is required instead of PaaS/serverless.


