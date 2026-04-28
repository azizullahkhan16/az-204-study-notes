# AZ-204: Azure Blob Storage - Complete Study Notes (Exam-Ready Edition)

## Learning Path Overview
This learning path covers developing solutions that use Azure Blob Storage with 3 modules:
1. Explore Azure Blob Storage
2. Manage the Azure Blob Storage Lifecycle
3. Work with Azure Blob Storage

**Duration:** 1 hour 34 minutes | **Level:** Intermediate | **XP:** 2400

> **⚠️ Note:** This document includes topics BEYOND the MS Learn career path — covering leases, immutable storage, static websites, change feed, AzCopy, Data Lake Gen2, Event Grid integration, Azure Functions bindings, point-in-time restore, concurrency control, and all exam gotchas.

---

## MODULE 1: EXPLORE AZURE BLOB STORAGE

### 1.1 Introduction to Azure Blob Storage

**What is Azure Blob Storage?**
- Massively scalable object storage for unstructured data
- Optimized for storing massive amounts of unstructured data
- Data that doesn't adhere to a particular data model or definition
- HTTP/HTTPS accessible from anywhere in the world
- Also known as Object Storage

**Key Use Cases:**
- Serving images or documents directly to a browser
- Storing files for distributed access
- Streaming video and audio
- Writing to log files
- Storing data for backup, restore, disaster recovery, and archiving
- Storing data for analysis by on-premises or Azure-hosted services
- Storage for virtual machine disks (Page blobs)

**Key Features:**
- Supports thousands of simultaneous uploads
- Designed for high availability
- Geo-redundancy options
- Hot, Cool, Cold, and Archive access tiers
- Lifecycle management for cost optimization
- Multiple authentication and authorization options
- Client libraries available for .NET, Java, Python, JavaScript, and more

### 1.2 Azure Storage Account Types

**What is a Storage Account?**
- Provides a unique namespace in Azure for your data
- Every object stored has an address that includes your unique account name
- Combination of account name and Blob Storage endpoint forms the base address
- Example: `https://<storage-account-name>.blob.core.windows.net`

**Storage Account Types:**

#### 1. **Standard General-Purpose v2**
- Recommended for most scenarios
- Supports Blob, File, Queue, and Table storage
- Supports all redundancy options (LRS, GRS, RA-GRS, ZRS, GZRS, RA-GZRS)
- Supports all access tiers (Hot, Cool, Cold, Archive)
- Standard performance tier (HDD-based)
- Use case: Most workloads, general storage needs

#### 2. **Premium Block Blobs**
- Premium performance tier (SSD-based)
- For block blobs and append blobs only
- Low latency, high transaction rates
- Supports LRS and ZRS redundancy only
- Use case: Applications requiring fast storage transactions, low latency
- Examples: Interactive workloads, IoT, analytics, AI/ML

#### 3. **Premium Page Blobs**
- Premium performance tier (SSD-based)
- For page blobs only
- Supports LRS and ZRS redundancy only
- Use case: Index-based and sparse data structures (OS and data disks for VMs)

#### 4. **Premium File Shares**
- Premium performance for Azure Files
- For file shares only
- Supports LRS and ZRS redundancy only
- Use case: Enterprise applications, high-performance file sharing

**Legacy Storage Account Types (Not Recommended):**
- **Blob Storage accounts**: Legacy, use General-Purpose v2 instead
- **General-Purpose v1**: Legacy, lacks features, use v2 instead

**Key Storage Account Properties:**

| Property | Description |
|----------|-------------|
| Name | Globally unique, 3-24 characters, lowercase letters and numbers only |
| Performance | Standard (HDD) or Premium (SSD) |
| Replication | LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS |
| Access Tier | Hot, Cool, Cold, or Archive (Standard only) |

### 1.3 Storage Account Redundancy Options

**Redundancy in Primary Region:**

#### 1. **Locally Redundant Storage (LRS)**
- Replicates data 3 times within a single data center in primary region
- 99.999999999% (11 nines) durability
- Lowest cost redundancy option
- Protects against: Server rack and drive failures
- Does NOT protect against: Data center disasters
- Use case: Non-critical data, easily reconstructable data

#### 2. **Zone-Redundant Storage (ZRS)**
- Replicates data across 3 Azure availability zones in primary region
- 99.9999999999% (12 nines) durability
- Data accessible for read/write even if a zone becomes unavailable
- Protects against: Data center-level failures
- Use case: High availability requirements, data residency requirements

**Redundancy in Secondary Region:**

#### 3. **Geo-Redundant Storage (GRS)**
- Replicates data 3 times in primary region (LRS)
- Then asynchronously replicates to secondary region (LRS)
- 99.99999999999999% (16 nines) durability
- Secondary region is paired region (determined by Azure)
- Use case: Disaster recovery, regional outage protection

#### 4. **Geo-Zone-Redundant Storage (GZRS)**
- Replicates data across 3 availability zones in primary region (ZRS)
- Then asynchronously replicates to secondary region (LRS)
- 99.99999999999999% (16 nines) durability
- Best durability and availability option
- Use case: Critical applications needing maximum durability

**Read Access to Secondary Region:**

#### 5. **Read-Access Geo-Redundant Storage (RA-GRS)**
- Same as GRS but with read access to secondary region
- Secondary region readable even when primary is available
- Use case: Applications needing to read from secondary region

#### 6. **Read-Access Geo-Zone-Redundant Storage (RA-GZRS)**
- Same as GZRS but with read access to secondary region
- Maximum durability + read availability
- Use case: Maximum availability and durability requirements

**Redundancy Comparison:**

| Redundancy | Durability (nines) | Regions | AZ Support | Read Secondary |
|------------|-------------------|---------|------------|----------------|
| LRS | 11 | 1 | No | No |
| ZRS | 12 | 1 | Yes | No |
| GRS | 16 | 2 | No | No |
| GZRS | 16 | 2 | Primary Only | No |
| RA-GRS | 16 | 2 | No | Yes |
| RA-GZRS | 16 | 2 | Primary Only | Yes |

### 1.4 Azure Blob Storage Resource Types

**Resource Hierarchy:**
```
Storage Account
    └── Container (unlimited)
            └── Blob (unlimited)
                    └── Block / Page / Append
```

#### **Storage Account**
- Top-level resource
- Provides unique namespace for data
- Contains all Azure Storage data objects
- Endpoint: `https://<storage-account>.blob.core.windows.net`

#### **Container**
- Organizes a set of blobs (similar to a directory in a file system)
- Storage account can contain unlimited containers
- Container can store unlimited blobs
- Container names must be:
  - 3-63 characters long
  - Start with letter or number
  - Contain only lowercase letters, numbers, and hyphens
  - Cannot have consecutive hyphens

**Container Access Levels:**

| Access Level | Description |
|--------------|-------------|
| **Private** (default) | No anonymous access, requires authentication |
| **Blob** | Anonymous read access for blobs only, cannot list blobs |
| **Container** | Anonymous read and list access for entire container |

#### **Blob Types**

**1. Block Blobs:**
- Composed of blocks, each identified by block ID
- Maximum size: ~190.7 TiB (4000 blocks × ~50,000 MiB per block)
- Can upload/replace individual blocks
- Optimized for upload efficiency
- Use case: Text files, images, videos, documents, binary data

**2. Append Blobs:**
- Optimized for append operations
- Composed of blocks like block blobs
- Blocks can only be added to end (no update/delete of existing blocks)
- Maximum size: ~195 GiB (50,000 blocks × 4 MiB per block)
- Use case: Logging scenarios, audit logs, streaming data

**3. Page Blobs:**
- Collection of 512-byte pages
- Optimized for random read/write operations
- Maximum size: 8 TiB
- Foundation for Azure IaaS disks
- Use case: Virtual machine OS disks, data disks, databases

**Blob Naming Rules:**
- 1-1024 characters
- Case-sensitive
- Can contain any combination of characters
- Reserved URL characters must be properly escaped
- Avoid names ending with dot (.) or forward slash (/)

### 1.5 Access Tiers for Blob Storage

**Overview:**
- Azure provides different access tiers to optimize storage costs
- Tiers based on how frequently data is accessed
- Can be set at blob level or default at account level

#### **Hot Access Tier**
- Optimized for frequent access
- Highest storage cost, lowest access cost
- Default tier for new storage accounts
- Use case: Data in active use, frequently accessed data
- Minimum storage duration: None

#### **Cool Access Tier**
- Optimized for infrequent access
- Lower storage cost, higher access cost than Hot
- Data should be stored for at least 30 days
- Use case: Short-term backup, older content not frequently viewed
- Minimum storage duration: 30 days
- Early deletion fee if deleted before 30 days

#### **Cold Access Tier**
- Optimized for rarely accessed data
- Lower storage cost than Cool, higher access cost
- Data should be stored for at least 90 days
- Use case: Long-term backup, archive-ready data
- Minimum storage duration: 90 days
- Early deletion fee if deleted before 90 days

#### **Archive Access Tier**
- Lowest storage cost, highest access cost
- Offline storage - data must be rehydrated before access
- Data should be stored for at least 180 days
- Use case: Long-term archive, compliance data, historical data
- Minimum storage duration: 180 days
- Early deletion fee if deleted before 180 days
- Rehydration required (can take hours)

**Access Tier Comparison:**

| Tier | Storage Cost | Access Cost | Min Duration | Latency |
|------|--------------|-------------|--------------|---------|
| Hot | Highest | Lowest | None | Milliseconds |
| Cool | Medium | Medium | 30 days | Milliseconds |
| Cold | Low | High | 90 days | Milliseconds |
| Archive | Lowest | Highest | 180 days | Hours (rehydration) |

**Setting Access Tiers:**

```bash
# Set default access tier for storage account
az storage account create --name <account-name> \
  --resource-group <rg> --access-tier Cool

# Set access tier for individual blob
az storage blob set-tier --account-name <account> \
  --container-name <container> --name <blob-name> --tier Cool

# Set tier during upload
az storage blob upload --account-name <account> \
  --container-name <container> --name <blob-name> \
  --file <file-path> --tier Cool
```

**Rehydration from Archive:**
- Must change tier to Hot, Cool, or Cold before accessing
- Two rehydration priorities:
  - **Standard**: Up to 15 hours
  - **High**: Under 1 hour for objects <10 GB
- Can copy blob to different tier instead of changing in place

### 1.6 Azure Storage Security Features

**Security Layers:**

#### **1. Encryption at Rest**
- **Azure Storage Service Encryption (SSE)**:
  - All data automatically encrypted using 256-bit AES encryption
  - Enabled by default, cannot be disabled
  - FIPS 140-2 compliant
- **Key Management Options**:
  - Microsoft-managed keys (default)
  - Customer-managed keys (stored in Azure Key Vault)
  - Customer-provided keys (for Blob storage operations)

#### **2. Encryption in Transit**
- **HTTPS**: All REST API operations use HTTPS
- **TLS 1.2**: Minimum TLS version configurable
- **SMB 3.0 encryption**: For Azure Files

```bash
# Require secure transfer (HTTPS only)
az storage account update --name <account-name> \
  --resource-group <rg> --https-only true

# Set minimum TLS version
az storage account update --name <account-name> \
  --resource-group <rg> --min-tls-version TLS1_2
```

#### **3. Authentication Methods**

**Azure Active Directory (Microsoft Entra ID):**
- Recommended for applications
- Uses OAuth 2.0 tokens
- Supports managed identities
- Fine-grained access control via RBAC
- Works with user accounts and service principals

**Shared Key (Account Key):**
- Storage account provides 2 access keys
- Full access to storage account
- Should be rotated regularly
- Use Azure Key Vault to store keys

**Shared Access Signatures (SAS):**
- Provides delegated access to resources
- Time-limited, scoped permissions
- Three types:
  - **User delegation SAS**: Secured with Azure AD credentials (most secure)
  - **Service SAS**: Secured with storage account key, scoped to one service
  - **Account SAS**: Secured with storage account key, scoped to account-level operations

**Anonymous Public Access:**
- Can enable read-only access to containers and blobs
- No authentication required
- Should be disabled unless explicitly needed

#### **4. Authorization Options**

**Role-Based Access Control (RBAC):**
- Assign roles to security principals
- Built-in roles for storage:
  - **Storage Blob Data Owner**: Full access to blob data
  - **Storage Blob Data Contributor**: Read, write, delete blob data
  - **Storage Blob Data Reader**: Read blob data only
  - **Storage Blob Delegator**: Generate user delegation SAS

**Stored Access Policies:**
- Define policies on containers
- Control SAS token permissions
- Can modify/revoke without regenerating account keys

#### **5. Network Security**

**Firewall and Virtual Networks:**
- Restrict access to specific VNets and IP addresses
- Service endpoints for private connectivity
- Private endpoints for complete network isolation

```bash
# Configure firewall rules
az storage account network-rule add \
  --account-name <account> --resource-group <rg> \
  --ip-address <ip-address>

# Enable service endpoint
az storage account network-rule add \
  --account-name <account> --resource-group <rg> \
  --vnet-name <vnet> --subnet <subnet>
```

**Advanced Threat Protection:**
- Detects unusual and potentially harmful access patterns
- Security alerts for suspicious activities
- Integrated with Azure Security Center

### 1.7 Shared Access Signatures (SAS) Deep Dive

**What is a SAS?**
- URI that grants restricted access rights to Azure Storage resources
- Specifies how resources can be accessed
- Includes start/end time, permissions, and resource scope

**SAS Token Components:**

| Component | Description |
|-----------|-------------|
| sv | Storage service version |
| ss | Services (b=blob, f=file, q=queue, t=table) |
| srt | Resource types (s=service, c=container, o=object) |
| sp | Permissions (r=read, w=write, d=delete, l=list, etc.) |
| se | Expiry time (UTC) |
| st | Start time (UTC) |
| spr | Protocol (https or https,http) |
| sig | Signature |

**Types of SAS:**

#### **1. User Delegation SAS (Recommended)**
- Secured with Azure AD credentials
- Only works for Blob storage
- Most secure option
- Recommended by Microsoft

```csharp
// Generate User Delegation SAS
UserDelegationKey key = await blobServiceClient.GetUserDelegationKeyAsync(
    DateTimeOffset.UtcNow,
    DateTimeOffset.UtcNow.AddHours(1));

BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = containerName,
    BlobName = blobName,
    Resource = "b",
    StartsOn = DateTimeOffset.UtcNow,
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};
sasBuilder.SetPermissions(BlobSasPermissions.Read);

string sasToken = sasBuilder.ToSasQueryParameters(key, accountName).ToString();
```

#### **2. Service SAS**
- Secured with storage account key
- Delegates access to single service (Blob, Queue, Table, or Files)
- Can be scoped to container or blob

```csharp
// Generate Service SAS
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = containerName,
    BlobName = blobName,
    Resource = "b",
    StartsOn = DateTimeOffset.UtcNow,
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};
sasBuilder.SetPermissions(BlobSasPermissions.Read | BlobSasPermissions.Write);

string sasToken = sasBuilder.ToSasQueryParameters(
    new StorageSharedKeyCredential(accountName, accountKey)).ToString();
```

#### **3. Account SAS**
- Secured with storage account key
- Delegates access to resources in one or more services
- Can specify service-level operations

**SAS Best Practices:**
1. Use User Delegation SAS when possible
2. Set shortest practical expiration time
3. Grant minimum required permissions
4. Use HTTPS only
5. Have a revocation plan
6. Use stored access policies for Service SAS
7. Validate SAS before distributing

**Stored Access Policies:**
- Provide additional control over Service SAS
- Can modify permissions without regenerating SAS
- Can revoke SAS by deleting or modifying policy
- Up to 5 policies per container

```bash
# Create stored access policy
az storage container policy create \
  --container-name <container> --name <policy-name> \
  --account-name <account> \
  --permissions racwdl --expiry 2025-01-01
```

---

## MODULE 2: MANAGE THE AZURE BLOB STORAGE LIFECYCLE

### 2.1 Blob Lifecycle Management Overview

**What is Lifecycle Management?**
- Rule-based policy to automatically manage blob lifecycle
- Transition blobs to different access tiers
- Delete blobs and blob versions
- Optimize costs by moving data to appropriate tiers
- Runs once per day

**Key Benefits:**
- Cost optimization through automatic tiering
- Automated deletion of outdated data
- Compliance with data retention requirements
- Reduced manual management overhead

### 2.2 Lifecycle Management Policies

**Policy Structure:**
```json
{
  "rules": [
    {
      "enabled": true,
      "name": "rule-name",
      "type": "Lifecycle",
      "definition": {
        "actions": { ... },
        "filters": { ... }
      }
    }
  ]
}
```

**Rule Components:**

#### **Filters**
Define which blobs the rule applies to:

| Filter | Description |
|--------|-------------|
| blobTypes | Array of blob types (blockBlob, appendBlob) |
| prefixMatch | Array of prefix strings to match |
| blobIndexMatch | Array of blob index tag conditions |

```json
"filters": {
  "blobTypes": ["blockBlob"],
  "prefixMatch": ["logs/", "data/archive/"],
  "blobIndexMatch": [
    {
      "name": "Project",
      "op": "==",
      "value": "Contoso"
    }
  ]
}
```

#### **Actions**
Define what happens to matching blobs:

**For Base Blobs:**

| Action | Description |
|--------|-------------|
| tierToCool | Move to Cool tier |
| tierToCold | Move to Cold tier |
| tierToArchive | Move to Archive tier |
| delete | Delete the blob |
| enableAutoTierToHotFromCool | Auto-tier to Hot when accessed |

**For Snapshots:**

| Action | Description |
|--------|-------------|
| tierToCool | Move snapshot to Cool |
| tierToCold | Move snapshot to Cold |
| tierToArchive | Move snapshot to Archive |
| delete | Delete the snapshot |

**For Versions:**

| Action | Description |
|--------|-------------|
| tierToCool | Move version to Cool |
| tierToCold | Move version to Cold |
| tierToArchive | Move version to Archive |
| delete | Delete the version |

**Action Conditions:**

| Condition | Description |
|-----------|-------------|
| daysAfterModificationGreaterThan | Days since last modification |
| daysAfterCreationGreaterThan | Days since creation |
| daysAfterLastAccessTimeGreaterThan | Days since last access (requires access tracking) |
| daysAfterLastTierChangeGreaterThan | Days since last tier change |

### 2.3 Lifecycle Policy Examples

**Example 1: Move to Cool after 30 days, Archive after 90 days, Delete after 365 days**

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "lifecycle-rule",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["data/"]
        }
      }
    }
  ]
}
```

**Example 2: Delete old snapshots and versions**

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "delete-old-snapshots",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "snapshot": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          },
          "version": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"]
        }
      }
    }
  ]
}
```

**Example 3: Archive based on blob index tags**

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "archive-by-tag",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 0
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "blobIndexMatch": [
            {
              "name": "Status",
              "op": "==",
              "value": "Archive"
            }
          ]
        }
      }
    }
  ]
}
```

### 2.4 Implementing Lifecycle Policies

**Via Azure Portal:**
```
Storage Account → Data Management → Lifecycle Management → Add Rule
```

**Via Azure CLI:**

```bash
# Create policy from JSON file
az storage account management-policy create \
  --account-name <storage-account> \
  --resource-group <rg> \
  --policy @policy.json

# Show current policy
az storage account management-policy show \
  --account-name <storage-account> \
  --resource-group <rg>

# Delete policy
az storage account management-policy delete \
  --account-name <storage-account> \
  --resource-group <rg>
```

**Via PowerShell:**

```powershell
# Create policy
$rule = New-AzStorageAccountManagementPolicyRule -Name "rule1" `
  -Action (New-AzStorageAccountManagementPolicyAction `
    -BaseBlobAction tierToCool -BaseBlobActionDaysAfterModificationGreaterThan 30) `
  -Filter (New-AzStorageAccountManagementPolicyFilter `
    -BlobType blockBlob -PrefixMatch "logs/")

Set-AzStorageAccountManagementPolicy -ResourceGroupName <rg> `
  -AccountName <storage-account> -Rule $rule
```

### 2.5 Lifecycle Management Considerations

**Tier Transition Rules:**
- Can only move from hotter to colder tiers (Hot → Cool → Cold → Archive)
- Cannot automatically move from colder to hotter tiers
- Archive requires rehydration before access

**Minimum Days Between Tier Changes:**
- Cool tier: 30 days minimum storage
- Cold tier: 90 days minimum storage  
- Archive tier: 180 days minimum storage
- Early deletion incurs charges

**Last Access Time Tracking:**
- Must enable "Access tracking" on storage account
- Required for `daysAfterLastAccessTimeGreaterThan` condition
- Has performance impact (small overhead per access)

```bash
# Enable last access time tracking
az storage account blob-service-properties update \
  --account-name <storage-account> \
  --resource-group <rg> \
  --enable-last-access-tracking true
```

**Policy Execution:**
- Runs once per day
- May take up to 24 hours to apply changes
- Processes blobs in batches
- Check Activity Log for execution status

### 2.6 Blob Versioning

**What is Blob Versioning?**
- Automatically maintains previous versions of blob
- Each write creates new version
- Previous versions are read-only
- Enabled at storage account level

**Enabling Versioning:**

```bash
# Enable versioning
az storage account blob-service-properties update \
  --account-name <storage-account> \
  --resource-group <rg> \
  --enable-versioning true
```

**Version ID:**
- Format: `<blob-name>?versionId=<datetime>`
- DateTime in format: `2024-01-15T12:30:45.1234567Z`

**Working with Versions:**

```csharp
// List all versions
await foreach (BlobItem blob in containerClient.GetBlobsAsync(
    traits: BlobTraits.None,
    states: BlobStates.Version))
{
    Console.WriteLine($"Version: {blob.VersionId}");
}

// Access specific version
BlobClient versionedBlob = containerClient.GetBlobClient(blobName)
    .WithVersion(versionId);
```

**Versioning Costs:**
- Each version stored separately
- Billed for storage of all versions
- Use lifecycle policies to manage version retention

### 2.7 Soft Delete

**Blob Soft Delete:**
- Retains deleted blobs for specified period
- Can recover accidentally deleted data
- Retention period: 1-365 days

```bash
# Enable blob soft delete
az storage blob service-properties delete-policy update \
  --account-name <storage-account> \
  --days-retained 7 \
  --enable true
```

**Container Soft Delete:**
- Retains deleted containers
- Separate from blob soft delete
- Retention period: 1-365 days

```bash
# Enable container soft delete
az storage account blob-service-properties update \
  --account-name <storage-account> \
  --resource-group <rg> \
  --enable-container-delete-retention true \
  --container-delete-retention-days 7
```

**Recovering Deleted Items:**

```csharp
// List deleted blobs
await foreach (BlobItem blob in containerClient.GetBlobsAsync(
    states: BlobStates.Deleted))
{
    Console.WriteLine($"Deleted: {blob.Name}, Deleted Time: {blob.Deleted}");
}

// Undelete blob
BlobClient blobClient = containerClient.GetBlobClient(blobName);
await blobClient.UndeleteAsync();
```

### 2.8 Blob Index Tags

**What are Blob Index Tags?**
- Key-value attributes on blobs
- Enables filtering and querying across blobs
- Up to 10 tags per blob
- Tag name: 1-128 characters
- Tag value: 0-256 characters

**Setting Tags:**

```csharp
// Set tags during upload
var options = new BlobUploadOptions
{
    Tags = new Dictionary<string, string>
    {
        { "Project", "Marketing" },
        { "Status", "Active" }
    }
};
await blobClient.UploadAsync(content, options);

// Set tags on existing blob
await blobClient.SetTagsAsync(new Dictionary<string, string>
{
    { "Project", "Marketing" },
    { "Status", "Archived" }
});

// Get tags
Response<IDictionary<string, string>> tags = await blobClient.GetTagsAsync();
```

**Querying by Tags:**

```csharp
// Find blobs by tag
string query = @"""Project"" = 'Marketing' AND ""Status"" = 'Active'";
await foreach (TaggedBlobItem blob in blobServiceClient.FindBlobsByTagsAsync(query))
{
    Console.WriteLine($"Found: {blob.BlobContainerName}/{blob.BlobName}");
}
```

---

## MODULE 3: WORK WITH AZURE BLOB STORAGE

### 3.1 Azure Storage Client Library for .NET

**Package Installation:**

```bash
# Install via NuGet
dotnet add package Azure.Storage.Blobs

# Additional packages for specific scenarios
dotnet add package Azure.Identity  # For Azure AD authentication
dotnet add package Azure.Storage.Blobs.Batch  # For batch operations
```

**Key Namespaces:**
```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using Azure.Storage.Blobs.Specialized;
using Azure.Identity;
```

### 3.2 Client Classes Overview

**BlobServiceClient**
- Represents the storage account
- Entry point for operations
- Used for account-level operations

```csharp
// Create using connection string
BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

// Create using Azure AD (DefaultAzureCredential)
BlobServiceClient blobServiceClient = new BlobServiceClient(
    new Uri("https://<account>.blob.core.windows.net"),
    new DefaultAzureCredential());

// Create using account key
BlobServiceClient blobServiceClient = new BlobServiceClient(
    new Uri("https://<account>.blob.core.windows.net"),
    new StorageSharedKeyCredential(accountName, accountKey));
```

**BlobContainerClient**
- Represents a single container
- Used for container and blob operations

```csharp
// Get from BlobServiceClient
BlobContainerClient containerClient = blobServiceClient.GetBlobContainerClient("mycontainer");

// Create directly
BlobContainerClient containerClient = new BlobContainerClient(connectionString, "mycontainer");

// Create with URI
BlobContainerClient containerClient = new BlobContainerClient(
    new Uri("https://<account>.blob.core.windows.net/mycontainer"),
    new DefaultAzureCredential());
```

**BlobClient**
- Represents a single blob
- Used for blob operations (upload, download, delete)

```csharp
// Get from BlobContainerClient
BlobClient blobClient = containerClient.GetBlobClient("myblob.txt");

// Create directly
BlobClient blobClient = new BlobClient(connectionString, "mycontainer", "myblob.txt");
```

**Specialized Blob Clients:**

| Client | Purpose |
|--------|---------|
| BlockBlobClient | Block blob specific operations |
| AppendBlobClient | Append blob specific operations |
| PageBlobClient | Page blob specific operations |
| BlobLeaseClient | Manage blob leases |

### 3.3 Authentication Methods

**1. Connection String:**
```csharp
string connectionString = "DefaultEndpointsProtocol=https;AccountName=<account>;AccountKey=<key>;EndpointSuffix=core.windows.net";
BlobServiceClient client = new BlobServiceClient(connectionString);
```

**2. DefaultAzureCredential (Recommended):**
```csharp
// Uses managed identity in Azure, developer credentials locally
BlobServiceClient client = new BlobServiceClient(
    new Uri("https://<account>.blob.core.windows.net"),
    new DefaultAzureCredential());
```

**3. Managed Identity:**
```csharp
// System-assigned managed identity
var credential = new ManagedIdentityCredential();
BlobServiceClient client = new BlobServiceClient(
    new Uri("https://<account>.blob.core.windows.net"),
    credential);

// User-assigned managed identity
var credential = new ManagedIdentityCredential("<client-id>");
```

**4. Storage Account Key:**
```csharp
var credential = new StorageSharedKeyCredential(accountName, accountKey);
BlobServiceClient client = new BlobServiceClient(
    new Uri("https://<account>.blob.core.windows.net"),
    credential);
```

**5. SAS Token:**
```csharp
// Using SAS URI
BlobClient blobClient = new BlobClient(
    new Uri("https://<account>.blob.core.windows.net/container/blob?<sas-token>"));
```

### 3.4 Container Operations

**Create Container:**
```csharp
// Create if not exists
BlobContainerClient containerClient = await blobServiceClient
    .CreateBlobContainerAsync("mycontainer");

// Create with public access
await blobServiceClient.CreateBlobContainerAsync(
    "publiccontainer",
    PublicAccessType.Blob);

// Check existence and create
BlobContainerClient containerClient = blobServiceClient.GetBlobContainerClient("mycontainer");
await containerClient.CreateIfNotExistsAsync();
```

**List Containers:**
```csharp
// List all containers
await foreach (BlobContainerItem container in blobServiceClient.GetBlobContainersAsync())
{
    Console.WriteLine($"Container: {container.Name}");
}

// List with prefix
await foreach (BlobContainerItem container in blobServiceClient.GetBlobContainersAsync(
    prefix: "log"))
{
    Console.WriteLine($"Container: {container.Name}");
}

// Include metadata
await foreach (BlobContainerItem container in blobServiceClient.GetBlobContainersAsync(
    traits: BlobContainerTraits.Metadata))
{
    Console.WriteLine($"Container: {container.Name}");
    foreach (var metadata in container.Properties.Metadata)
    {
        Console.WriteLine($"  {metadata.Key}: {metadata.Value}");
    }
}
```

**Delete Container:**
```csharp
// Delete container
await containerClient.DeleteAsync();

// Delete if exists
await containerClient.DeleteIfExistsAsync();
```

**Container Properties and Metadata:**
```csharp
// Get properties
BlobContainerProperties properties = await containerClient.GetPropertiesAsync();
Console.WriteLine($"Last Modified: {properties.LastModified}");
Console.WriteLine($"ETag: {properties.ETag}");

// Set metadata
var metadata = new Dictionary<string, string>
{
    { "Department", "Marketing" },
    { "Project", "Campaign2024" }
};
await containerClient.SetMetadataAsync(metadata);

// Set access policy
await containerClient.SetAccessPolicyAsync(PublicAccessType.Blob);
```

### 3.5 Blob Operations

**Upload Blob:**
```csharp
// Upload from file path
await blobClient.UploadAsync("local-file.txt", overwrite: true);

// Upload from stream
using FileStream stream = File.OpenRead("local-file.txt");
await blobClient.UploadAsync(stream, overwrite: true);

// Upload from BinaryData
BinaryData data = BinaryData.FromString("Hello, World!");
await blobClient.UploadAsync(data, overwrite: true);

// Upload with options
var options = new BlobUploadOptions
{
    HttpHeaders = new BlobHttpHeaders
    {
        ContentType = "text/plain",
        CacheControl = "max-age=3600"
    },
    Metadata = new Dictionary<string, string>
    {
        { "Author", "John Doe" }
    },
    Tags = new Dictionary<string, string>
    {
        { "Status", "Active" }
    },
    AccessTier = AccessTier.Cool
};
await blobClient.UploadAsync("local-file.txt", options);
```

**Download Blob:**
```csharp
// Download to file
await blobClient.DownloadToAsync("downloaded-file.txt");

// Download to stream
using MemoryStream stream = new MemoryStream();
await blobClient.DownloadToAsync(stream);

// Download content
BlobDownloadResult result = await blobClient.DownloadContentAsync();
string content = result.Content.ToString();

// Download with options (partial download)
var options = new BlobDownloadOptions
{
    Range = new HttpRange(0, 1024)  // First 1KB
};
Response<BlobDownloadStreamingResult> response = await blobClient.DownloadStreamingAsync(options);
```

**List Blobs:**
```csharp
// List all blobs in container
await foreach (BlobItem blob in containerClient.GetBlobsAsync())
{
    Console.WriteLine($"Blob: {blob.Name}, Size: {blob.Properties.ContentLength}");
}

// List with prefix (virtual directory)
await foreach (BlobItem blob in containerClient.GetBlobsAsync(prefix: "logs/2024/"))
{
    Console.WriteLine($"Blob: {blob.Name}");
}

// List by hierarchy (like folders)
await foreach (BlobHierarchyItem item in containerClient.GetBlobsByHierarchyAsync(
    delimiter: "/", 
    prefix: "logs/"))
{
    if (item.IsPrefix)
    {
        Console.WriteLine($"Virtual Directory: {item.Prefix}");
    }
    else
    {
        Console.WriteLine($"Blob: {item.Blob.Name}");
    }
}

// Include additional information
await foreach (BlobItem blob in containerClient.GetBlobsAsync(
    traits: BlobTraits.Metadata | BlobTraits.Tags))
{
    Console.WriteLine($"Blob: {blob.Name}");
    Console.WriteLine($"  Content-Type: {blob.Properties.ContentType}");
    Console.WriteLine($"  Access Tier: {blob.Properties.AccessTier}");
}
```

**Delete Blob:**
```csharp
// Delete blob
await blobClient.DeleteAsync();

// Delete if exists
await blobClient.DeleteIfExistsAsync();

// Delete with options (include snapshots)
await blobClient.DeleteAsync(DeleteSnapshotsOption.IncludeSnapshots);

// Delete only snapshots
await blobClient.DeleteAsync(DeleteSnapshotsOption.OnlySnapshots);
```

**Copy Blob:**
```csharp
// Copy from URI (async operation)
Uri sourceUri = new Uri("https://source-account.blob.core.windows.net/container/blob");
CopyFromUriOperation operation = await blobClient.StartCopyFromUriAsync(sourceUri);
await operation.WaitForCompletionAsync();

// Synchronous copy (for small blobs within same account)
await blobClient.SyncCopyFromUriAsync(sourceUri);
```

### 3.6 Blob Properties and Metadata

**Blob Properties (System-defined):**

| Property | Description |
|----------|-------------|
| ContentType | MIME type (e.g., "text/plain") |
| ContentLength | Size in bytes |
| ContentEncoding | Encoding (e.g., "gzip") |
| ContentLanguage | Language |
| ContentDisposition | How to display |
| CacheControl | Caching directives |
| ContentHash | MD5 hash |
| LastModified | Last modification time |
| CreatedOn | Creation time |
| ETag | Entity tag for concurrency |
| AccessTier | Current access tier |

**Get Properties:**
```csharp
BlobProperties properties = await blobClient.GetPropertiesAsync();

Console.WriteLine($"Content Type: {properties.ContentType}");
Console.WriteLine($"Content Length: {properties.ContentLength}");
Console.WriteLine($"Last Modified: {properties.LastModified}");
Console.WriteLine($"Created On: {properties.CreatedOn}");
Console.WriteLine($"Access Tier: {properties.AccessTier}");
Console.WriteLine($"ETag: {properties.ETag}");

// Access metadata
foreach (var kvp in properties.Metadata)
{
    Console.WriteLine($"Metadata - {kvp.Key}: {kvp.Value}");
}
```

**Set HTTP Headers:**
```csharp
var headers = new BlobHttpHeaders
{
    ContentType = "application/json",
    CacheControl = "max-age=3600",
    ContentDisposition = "attachment; filename=\"data.json\""
};
await blobClient.SetHttpHeadersAsync(headers);
```

**Set Metadata:**
```csharp
var metadata = new Dictionary<string, string>
{
    { "Author", "Jane Smith" },
    { "ReviewDate", "2024-06-15" }
};
await blobClient.SetMetadataAsync(metadata);
```

**Get and Set Tags:**
```csharp
// Get tags
Response<IDictionary<string, string>> tagsResponse = await blobClient.GetTagsAsync();
foreach (var tag in tagsResponse.Value)
{
    Console.WriteLine($"Tag - {tag.Key}: {tag.Value}");
}

// Set tags
var tags = new Dictionary<string, string>
{
    { "Project", "Alpha" },
    { "Status", "InProgress" }
};
await blobClient.SetTagsAsync(tags);
```

### 3.7 Working with Blob URIs

**Get Blob URI:**
```csharp
// Get blob URI
Uri blobUri = blobClient.Uri;
Console.WriteLine($"Blob URI: {blobUri}");
// Output: https://<account>.blob.core.windows.net/<container>/<blob>

// Get container URI
Uri containerUri = containerClient.Uri;
Console.WriteLine($"Container URI: {containerUri}");
```

**Generate SAS URI:**
```csharp
// Generate SAS for blob
BlobSasBuilder sasBuilder = new BlobSasBuilder
{
    BlobContainerName = containerClient.Name,
    BlobName = blobClient.Name,
    Resource = "b",  // b = blob, c = container
    StartsOn = DateTimeOffset.UtcNow,
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};
sasBuilder.SetPermissions(BlobSasPermissions.Read);

// Generate with account key
Uri sasUri = blobClient.GenerateSasUri(sasBuilder);

// Or manually construct
string sasToken = sasBuilder.ToSasQueryParameters(
    new StorageSharedKeyCredential(accountName, accountKey)).ToString();
Uri fullUri = new Uri($"{blobClient.Uri}?{sasToken}");
```

**User Delegation SAS:**
```csharp
// Get user delegation key
UserDelegationKey userDelegationKey = await blobServiceClient.GetUserDelegationKeyAsync(
    DateTimeOffset.UtcNow,
    DateTimeOffset.UtcNow.AddHours(1));

// Create SAS builder
BlobSasBuilder sasBuilder = new BlobSasBuilder
{
    BlobContainerName = containerName,
    BlobName = blobName,
    Resource = "b",
    StartsOn = DateTimeOffset.UtcNow,
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};
sasBuilder.SetPermissions(BlobSasPermissions.Read | BlobSasPermissions.Write);

// Generate SAS token
string sasToken = sasBuilder.ToSasQueryParameters(
    userDelegationKey, 
    blobServiceClient.AccountName).ToString();
```

### 3.8 Error Handling and Best Practices

**Common Exceptions:**

| Exception | Cause |
|-----------|-------|
| RequestFailedException | General Azure Storage error |
| AuthenticationFailedException | Authentication issues |
| Azure.RequestFailedException with Status 404 | Blob/Container not found |
| Azure.RequestFailedException with Status 409 | Conflict (e.g., container exists) |
| Azure.RequestFailedException with Status 412 | Precondition failed |

**Error Handling Pattern:**
```csharp
try
{
    await blobClient.UploadAsync(content);
}
catch (RequestFailedException ex) when (ex.Status == 404)
{
    Console.WriteLine("Blob or container not found");
}
catch (RequestFailedException ex) when (ex.Status == 409)
{
    Console.WriteLine("Conflict - blob may already exist");
}
catch (RequestFailedException ex)
{
    Console.WriteLine($"Azure Storage error: {ex.Status} - {ex.Message}");
    Console.WriteLine($"Error Code: {ex.ErrorCode}");
}
catch (AuthenticationFailedException ex)
{
    Console.WriteLine($"Authentication failed: {ex.Message}");
}
```

**Retry Configuration:**
```csharp
var options = new BlobClientOptions
{
    Retry = 
    {
        MaxRetries = 5,
        Delay = TimeSpan.FromSeconds(2),
        MaxDelay = TimeSpan.FromSeconds(30),
        Mode = RetryMode.Exponential,
        NetworkTimeout = TimeSpan.FromSeconds(100)
    }
};

BlobServiceClient client = new BlobServiceClient(connectionString, options);
```

**Best Practices:**

1. **Use Async Methods:**
```csharp
// Always prefer async
await blobClient.UploadAsync(content);
// Avoid sync methods in production
```

2. **Reuse Clients:**
```csharp
// Create once and reuse
private static readonly BlobServiceClient _blobServiceClient = 
    new BlobServiceClient(connectionString);
```

3. **Use Managed Identity in Production:**
```csharp
// In production environments
var client = new BlobServiceClient(
    new Uri("https://<account>.blob.core.windows.net"),
    new DefaultAzureCredential());
```

4. **Set Appropriate Content-Type:**
```csharp
var options = new BlobUploadOptions
{
    HttpHeaders = new BlobHttpHeaders
    {
        ContentType = "application/json"
    }
};
```

5. **Handle Large Files:**
```csharp
// For large files, use streams
using var fileStream = File.OpenRead(largeFile);
await blobClient.UploadAsync(fileStream, overwrite: true);
```

### 3.9 Complete Code Examples

**Example 1: Complete Blob Operations**
```csharp
using Azure.Identity;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

public class BlobStorageService
{
    private readonly BlobServiceClient _blobServiceClient;

    public BlobStorageService(string accountUrl)
    {
        _blobServiceClient = new BlobServiceClient(
            new Uri(accountUrl),
            new DefaultAzureCredential());
    }

    public async Task<string> UploadBlobAsync(
        string containerName, 
        string blobName, 
        Stream content,
        string contentType)
    {
        BlobContainerClient containerClient = 
            _blobServiceClient.GetBlobContainerClient(containerName);
        await containerClient.CreateIfNotExistsAsync();

        BlobClient blobClient = containerClient.GetBlobClient(blobName);
        
        var options = new BlobUploadOptions
        {
            HttpHeaders = new BlobHttpHeaders { ContentType = contentType }
        };

        await blobClient.UploadAsync(content, options);
        return blobClient.Uri.ToString();
    }

    public async Task<Stream> DownloadBlobAsync(string containerName, string blobName)
    {
        BlobContainerClient containerClient = 
            _blobServiceClient.GetBlobContainerClient(containerName);
        BlobClient blobClient = containerClient.GetBlobClient(blobName);

        var stream = new MemoryStream();
        await blobClient.DownloadToAsync(stream);
        stream.Position = 0;
        return stream;
    }

    public async Task<IEnumerable<string>> ListBlobsAsync(string containerName)
    {
        BlobContainerClient containerClient = 
            _blobServiceClient.GetBlobContainerClient(containerName);

        var blobs = new List<string>();
        await foreach (BlobItem blob in containerClient.GetBlobsAsync())
        {
            blobs.Add(blob.Name);
        }
        return blobs;
    }

    public async Task DeleteBlobAsync(string containerName, string blobName)
    {
        BlobContainerClient containerClient = 
            _blobServiceClient.GetBlobContainerClient(containerName);
        BlobClient blobClient = containerClient.GetBlobClient(blobName);
        
        await blobClient.DeleteIfExistsAsync();
    }
}
```

**Example 2: Retrieve and Display Metadata**
```csharp
public async Task DisplayBlobMetadataAsync(string containerName, string blobName)
{
    BlobClient blobClient = _blobServiceClient
        .GetBlobContainerClient(containerName)
        .GetBlobClient(blobName);

    BlobProperties properties = await blobClient.GetPropertiesAsync();

    Console.WriteLine($"=== Blob Properties ===");
    Console.WriteLine($"Name: {blobClient.Name}");
    Console.WriteLine($"URI: {blobClient.Uri}");
    Console.WriteLine($"Content Type: {properties.ContentType}");
    Console.WriteLine($"Content Length: {properties.ContentLength} bytes");
    Console.WriteLine($"Created On: {properties.CreatedOn}");
    Console.WriteLine($"Last Modified: {properties.LastModified}");
    Console.WriteLine($"Access Tier: {properties.AccessTier}");
    Console.WriteLine($"ETag: {properties.ETag}");

    Console.WriteLine($"\n=== Custom Metadata ===");
    foreach (var metadata in properties.Metadata)
    {
        Console.WriteLine($"{metadata.Key}: {metadata.Value}");
    }

    Response<IDictionary<string, string>> tagsResponse = await blobClient.GetTagsAsync();
    Console.WriteLine($"\n=== Tags ===");
    foreach (var tag in tagsResponse.Value)
    {
        Console.WriteLine($"{tag.Key}: {tag.Value}");
    }
}
```

**Example 3: List Containers with Properties**
```csharp
public async Task ListContainersWithPropertiesAsync()
{
    await foreach (BlobContainerItem container in _blobServiceClient.GetBlobContainersAsync(
        traits: BlobContainerTraits.Metadata))
    {
        Console.WriteLine($"=== Container: {container.Name} ===");
        Console.WriteLine($"  URI: https://{_blobServiceClient.AccountName}.blob.core.windows.net/{container.Name}");
        Console.WriteLine($"  Last Modified: {container.Properties.LastModified}");
        Console.WriteLine($"  Public Access: {container.Properties.PublicAccess}");
        
        if (container.Properties.Metadata != null)
        {
            foreach (var metadata in container.Properties.Metadata)
            {
                Console.WriteLine($"  Metadata - {metadata.Key}: {metadata.Value}");
            }
        }
    }
}
```

---

## MODULE 4: ADVANCED CONCEPTS (BEYOND MS LEARN - EXAM CRITICAL)

### 4.1 Blob Leases (Pessimistic Concurrency)

**What is a Blob Lease?**
- Lock mechanism for exclusive write/delete access to a blob or container
- Prevents other clients from modifying or deleting the blob
- Used for pessimistic concurrency control

**Lease States:**

| State | Description |
|-------|-------------|
| Available | No active lease, can be acquired |
| Leased | Active lease, only lease holder can modify/delete |
| Expired | Lease duration elapsed, blob is available |
| Breaking | Lease is being broken, transitions to Broken |
| Broken | Lease was broken, blob is available |

**Lease Duration:**
- **Fixed**: 15-60 seconds
- **Infinite**: -1 (must be explicitly released or broken)

**Lease Operations:**

| Operation | Description |
|-----------|-------------|
| Acquire | Obtain a lease on the blob |
| Renew | Extend the lease before expiry |
| Change | Change the lease ID |
| Release | Release the lease (blob becomes available) |
| Break | Force-break the lease (regardless of holder) |

```csharp
// Acquire a lease
BlobLeaseClient leaseClient = blobClient.GetBlobLeaseClient();
BlobLease lease = await leaseClient.AcquireAsync(TimeSpan.FromSeconds(30));
string leaseId = lease.LeaseId;

// Modify blob with lease (required while leased)
BlobUploadOptions options = new BlobUploadOptions
{
    Conditions = new BlobRequestConditions { LeaseId = leaseId }
};
await blobClient.UploadAsync(content, options);

// Renew lease before expiry
await leaseClient.RenewAsync();

// Release lease when done
await leaseClient.ReleaseAsync();

// Break lease (force unlock)
await leaseClient.BreakAsync();
```

```csharp
// Container lease (prevents container deletion)
BlobLeaseClient containerLease = containerClient.GetBlobLeaseClient();
await containerLease.AcquireAsync(TimeSpan.FromSeconds(-1)); // Infinite
// Container cannot be deleted until lease is released
await containerLease.ReleaseAsync();
```

**⚠️ Exam Tip:** Leases = pessimistic concurrency. ETags = optimistic concurrency. Know the difference.

### 4.2 Concurrency Control with ETags

**Optimistic Concurrency (ETags):**
- Every blob has an `ETag` that changes on each modification
- Use `If-Match` to ensure blob hasn't changed since you read it
- Use `If-None-Match` to ensure blob doesn't already exist

```csharp
// Read blob and capture ETag
BlobProperties props = await blobClient.GetPropertiesAsync();
ETag originalETag = props.ETag;

// Update only if blob hasn't changed (optimistic concurrency)
BlobUploadOptions options = new BlobUploadOptions
{
    Conditions = new BlobRequestConditions
    {
        IfMatch = originalETag  // Fails with 412 if ETag changed
    }
};

try
{
    await blobClient.UploadAsync(newContent, options);
}
catch (RequestFailedException ex) when (ex.Status == 412)
{
    Console.WriteLine("Blob was modified by another process!");
}

// Create only if blob doesn't exist
BlobUploadOptions createOnly = new BlobUploadOptions
{
    Conditions = new BlobRequestConditions
    {
        IfNoneMatch = ETag.All  // "*" - fails if blob exists
    }
};
```

### 4.3 Immutable Storage (WORM - Write Once, Read Many)

**What is Immutable Storage?**
- Prevents modification and deletion of blobs for a specified period
- Used for regulatory compliance (SEC, FINRA, CFTC, etc.)
- Two types of policies:

#### **1. Time-Based Retention Policy**
- Blobs cannot be modified or deleted for N days
- Can be **locked** (making retention period immutable) or **unlocked** (can extend but not decrease)
- After lock, policy cannot be deleted, period can only be extended

```bash
# Set time-based retention (90 days)
az storage container immutability-policy create \
  --account-name <account> --container-name <container> \
  --period 90

# Lock the policy (IRREVERSIBLE!)
az storage container immutability-policy lock \
  --account-name <account> --container-name <container> \
  --if-match <etag>
```

#### **2. Legal Hold**
- Indefinite hold until explicitly cleared
- Multiple legal holds can be applied (tagged by name)
- Blobs can be created and read but NOT modified or deleted
- No expiration period

```bash
# Set legal hold
az storage container legal-hold set \
  --account-name <account> --container-name <container> \
  --tags "Investigation2026" "AuditHold"

# Clear legal hold
az storage container legal-hold clear \
  --account-name <account> --container-name <container> \
  --tags "Investigation2026"
```

**Immutable Storage Comparison:**

| Feature | Time-Based Retention | Legal Hold |
|---------|---------------------|------------|
| Duration | Fixed period (days) | Indefinite (until cleared) |
| Can extend? | Yes (if unlocked or locked) | N/A |
| Can shorten? | Only if unlocked | N/A |
| Can delete policy? | Only if unlocked | Yes (clear tags) |
| Lockable? | Yes (irreversible) | N/A |
| Use case | Compliance retention | Legal investigations |

**⚠️ Exam Tip:** Immutable = cannot delete or overwrite. You CAN still create new blobs. Versioning-level immutability is also available (per-version policies).

### 4.4 Static Website Hosting

**What is it?**
- Serve static content (HTML, CSS, JS, images) directly from Blob Storage
- No web server needed
- Uses a special container named `$web`
- Supports custom domains and Azure CDN

**Configuration:**

```bash
# Enable static website hosting
az storage blob service-properties update \
  --account-name <account> \
  --static-website \
  --index-document index.html \
  --404-document error.html

# Upload files to $web container
az storage blob upload-batch \
  --account-name <account> \
  --source ./website-files \
  --destination '$web'
```

**Endpoints:**

| Endpoint | Format |
|----------|--------|
| Primary | `https://<account>.z<zone>.web.core.windows.net/` |
| Blob endpoint | `https://<account>.blob.core.windows.net/$web/index.html` |

**Key Details:**
- Container is always named `$web` (auto-created when enabled)
- Index document: served for `/` and directory paths
- 404 document: served for missing pages
- **Server-side code NOT supported** (static content only)
- Combine with **Azure CDN** for custom domain + HTTPS
- Supports custom domains via CNAME mapping

**⚠️ Exam Tip:** Static website hosting uses a DIFFERENT endpoint than normal blob endpoint. The `$web` container has **anonymous read access** automatically.

### 4.5 Blob Snapshots

**What are Snapshots?**
- Read-only copy of a blob at a point in time
- Snapshot = base blob URI + `?snapshot=<DateTime>`
- Creating a snapshot is instant (metadata operation)
- Billed only for unique data (differences from base blob)

```csharp
// Create snapshot
BlobSnapshotInfo snapshot = await blobClient.CreateSnapshotAsync();
string snapshotTime = snapshot.Snapshot; // DateTime string

// Access snapshot
BlobClient snapshotClient = blobClient.WithSnapshot(snapshotTime);
BlobDownloadResult content = await snapshotClient.DownloadContentAsync();

// List snapshots
await foreach (BlobItem blob in containerClient.GetBlobsAsync(
    states: BlobStates.Snapshots))
{
    Console.WriteLine($"Name: {blob.Name}, Snapshot: {blob.Snapshot}");
}

// Delete blob and all snapshots
await blobClient.DeleteAsync(DeleteSnapshotsOption.IncludeSnapshots);

// Delete only snapshots (keep base blob)
await blobClient.DeleteAsync(DeleteSnapshotsOption.OnlySnapshots);
```

**Snapshots vs Versioning:**

| Feature | Snapshots | Versioning |
|---------|-----------|-----------|
| Creation | Manual (explicit call) | Automatic (on every write) |
| Mutability | Read-only | Read-only (previous versions) |
| Identifier | `?snapshot=<datetime>` | `?versionId=<datetime>` |
| Deletion tracking | No | Yes (previous version retained) |
| Lifecycle mgmt | Yes | Yes |
| Enable | Per operation | Account-level setting |

### 4.6 Object Replication

**What is it?**
- Asynchronous copy of blobs between storage accounts
- Source and destination can be in different regions
- Requires: blob versioning + change feed enabled on both accounts

**Requirements:**
- Both accounts must have **versioning enabled**
- Source must have **change feed enabled**
- Supports block blobs only
- Cannot replicate to/from Archive tier

```bash
# Enable prerequisites
az storage account blob-service-properties update \
  --account-name <source> --resource-group <rg> \
  --enable-versioning true --enable-change-feed true

az storage account blob-service-properties update \
  --account-name <destination> --resource-group <rg> \
  --enable-versioning true
```

### 4.7 Blob Storage Change Feed

**What is it?**
- Transaction log of all changes to blobs and blob metadata
- Stored as blobs in a special `$blobchangefeed` container
- Ordered, guaranteed, durable
- Events available within minutes of change

**Tracked Events:** Create, Update, Delete, SetMetadata, SetBlobTier, CopyBlob, SetBlobProperties, CreateSnapshot, UndeleteBlob

```bash
# Enable change feed
az storage account blob-service-properties update \
  --account-name <account> --resource-group <rg> \
  --enable-change-feed true
```

**Use Cases:**
- Audit trail / compliance
- Trigger downstream processing
- Feed into data analytics
- Replicate data changes

**⚠️ Exam Tip:** Blob change feed ≠ Cosmos DB change feed. Blob change feed is stored as Avro-format blobs in `$blobchangefeed`. Different from Event Grid events (which are real-time push).

### 4.8 Event Grid Integration (Blob Events)

**Blob Storage generates events for Event Grid:**

| Event Type | When |
|------------|------|
| `Microsoft.Storage.BlobCreated` | Blob created or replaced |
| `Microsoft.Storage.BlobDeleted` | Blob deleted |
| `Microsoft.Storage.BlobTierChanged` | Blob tier changed |
| `Microsoft.Storage.BlobRenamed` | Blob renamed (Data Lake Gen2 only) |
| `Microsoft.Storage.DirectoryCreated` | Directory created (Data Lake Gen2) |
| `Microsoft.Storage.DirectoryDeleted` | Directory deleted (Data Lake Gen2) |

```bash
# Create Event Grid subscription for blob events
az eventgrid event-subscription create \
  --source-resource-id "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>" \
  --name blob-events \
  --endpoint <webhook-url> \
  --included-event-types Microsoft.Storage.BlobCreated
```

**Event Grid vs Change Feed:**

| Feature | Event Grid | Change Feed |
|---------|-----------|-------------|
| Delivery | Push (real-time) | Pull (batch) |
| Latency | Seconds | Minutes |
| Ordering | Not guaranteed | Ordered per blob |
| Completeness | At-least-once | Guaranteed complete |
| Format | JSON event | Avro records |
| Use case | Real-time reactions | Auditing, replication |

### 4.9 Azure Functions Blob Storage Bindings

#### **1. Blob Trigger**
```csharp
// Triggered when blob is created/updated in a container
[FunctionName("ProcessBlob")]
public static void Run(
    [BlobTrigger("input-container/{name}", Connection = "StorageConnection")]
    Stream inputBlob,
    string name,
    ILogger log)
{
    log.LogInformation($"Processing blob: {name}, Size: {inputBlob.Length}");
}

// With BlobClient for more control
[FunctionName("ProcessBlobClient")]
public static async Task Run(
    [BlobTrigger("input/{name}", Connection = "StorageConnection")]
    BlobClient blobClient,
    string name,
    ILogger log)
{
    BlobProperties props = await blobClient.GetPropertiesAsync();
    log.LogInformation($"Content type: {props.ContentType}");
}
```

**⚠️ Blob Trigger Latency:** Can take up to **10 minutes** to detect new blobs on Consumption plan. For faster processing, use **Event Grid trigger** instead.

#### **2. Blob Input Binding**
```csharp
[FunctionName("ReadBlob")]
public static IActionResult Run(
    [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req,
    [Blob("configs/settings.json", FileAccess.Read, Connection = "StorageConnection")]
    string blobContent,
    ILogger log)
{
    return new OkObjectResult(blobContent);
}

// With stream
[FunctionName("ReadBlobStream")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req,
    [Blob("images/{Query.name}", FileAccess.Read, Connection = "StorageConnection")]
    Stream blobStream,
    ILogger log)
{
    // Process stream
    return new FileStreamResult(blobStream, "image/jpeg");
}
```

#### **3. Blob Output Binding**
```csharp
[FunctionName("WriteBlob")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    [Blob("output/{rand-guid}.txt", FileAccess.Write, Connection = "StorageConnection")]
    TextWriter outputBlob,
    ILogger log)
{
    string body = await new StreamReader(req.Body).ReadToEndAsync();
    await outputBlob.WriteAsync(body);
    return new OkResult();
}
```

### 4.10 Large File Upload — Block Staging Pattern

**How Block Blob Upload Works Internally:**
1. Split file into blocks (up to 4000 blocks)
2. Upload each block with a unique Block ID (`PutBlock`)
3. Commit the block list to finalize the blob (`PutBlockList`)

```csharp
// Manual block upload for large files
BlockBlobClient blockBlobClient = containerClient.GetBlockBlobClient("large-file.zip");

int blockSize = 4 * 1024 * 1024; // 4 MB blocks
List<string> blockIds = new List<string>();
int blockNumber = 0;

using FileStream fs = File.OpenRead("large-file.zip");
byte[] buffer = new byte[blockSize];
int bytesRead;

while ((bytesRead = await fs.ReadAsync(buffer, 0, blockSize)) > 0)
{
    string blockId = Convert.ToBase64String(
        BitConverter.GetBytes(blockNumber++));
    blockIds.Add(blockId);

    using var blockStream = new MemoryStream(buffer, 0, bytesRead);
    await blockBlobClient.StageBlockAsync(blockId, blockStream);
}

// Commit all blocks
await blockBlobClient.CommitBlockListAsync(blockIds);
```

**Block Limits:**

| Setting | Value |
|---------|-------|
| Max block size | 4000 MiB (for 2024+ API) |
| Max blocks per blob | 50,000 |
| Max blob size (block blob) | ~190.7 TiB |
| Max single upload (PutBlob) | 5000 MiB |
| Recommended: use blocks for files > | 256 MiB |

**SDK Automatic Chunking:**
```csharp
// SDK handles chunking automatically with transfer options
var transferOptions = new StorageTransferOptions
{
    MaximumConcurrency = 8,          // Parallel uploads
    MaximumTransferSize = 4 * 1024 * 1024,  // 4 MB per transfer
    InitialTransferSize = 4 * 1024 * 1024   // First chunk size
};

var uploadOptions = new BlobUploadOptions { TransferOptions = transferOptions };
await blobClient.UploadAsync("large-file.zip", uploadOptions);
```

### 4.11 AzCopy Tool

**What is AzCopy?**
- Command-line utility for copying data to/from Azure Storage
- High-performance, parallelized transfers
- Supports Blob, File, and Table storage
- Uses SAS tokens or Azure AD for authentication

**Common Commands:**

```bash
# Login with Azure AD
azcopy login

# Upload file to blob
azcopy copy "local-file.txt" "https://<account>.blob.core.windows.net/<container>/file.txt?<sas>"

# Upload directory
azcopy copy "local-dir" "https://<account>.blob.core.windows.net/<container>?<sas>" --recursive

# Download blob
azcopy copy "https://<account>.blob.core.windows.net/<container>/file.txt?<sas>" "local-file.txt"

# Copy between storage accounts (server-to-server, no local download)
azcopy copy "https://source.blob.core.windows.net/container?<sas>" \
  "https://dest.blob.core.windows.net/container?<sas>" --recursive

# Sync (only copies new/changed files)
azcopy sync "local-dir" "https://<account>.blob.core.windows.net/<container>?<sas>"

# Remove blobs
azcopy remove "https://<account>.blob.core.windows.net/<container>/file.txt?<sas>"
```

**Key Details:**
- Supports Azure AD auth and SAS tokens
- Server-to-server copy (no data through local machine)
- `sync` only copies differences (like rsync)
- Supports `--include-pattern` and `--exclude-pattern`
- Logs stored in `%USERPROFILE%\.azcopy` (Windows)

### 4.12 Azure Data Lake Storage Gen2

**What is it?**
- **Hierarchical namespace** on top of Blob Storage
- True directory structure (not just virtual directories)
- Enables POSIX-like permissions (ACLs)
- Optimized for big data analytics (Spark, Databricks, Synapse)

**Key Differences from Regular Blob Storage:**

| Feature | Blob Storage | Data Lake Gen2 |
|---------|-------------|---------------|
| Namespace | Flat | Hierarchical |
| Directories | Virtual (prefix-based) | Real directories |
| Rename directory | Copy + delete all blobs | Atomic rename |
| ACLs | No | Yes (POSIX) |
| Endpoint | `blob.core.windows.net` | `dfs.core.windows.net` |
| Analytics | Basic | Optimized for big data |

```bash
# Create storage account with hierarchical namespace
az storage account create \
  --name <account> --resource-group <rg> \
  --kind StorageV2 --enable-hierarchical-namespace true
```

**⚠️ Exam Tip:** Once hierarchical namespace is enabled, it **cannot be disabled**. Regular Blob API still works alongside Data Lake API.

### 4.13 Point-in-Time Restore for Blobs

**What is it?**
- Restore block blobs to a previous state
- Requires: soft delete, versioning, and change feed all enabled
- Restore scope: entire account, specific containers, or blob prefix ranges

```bash
# Enable prerequisites
az storage account blob-service-properties update \
  --account-name <account> --resource-group <rg> \
  --enable-delete-retention true --delete-retention-days 14 \
  --enable-versioning true --enable-change-feed true \
  --enable-restore-policy true --restore-days 13

# Restore to point in time
az storage blob restore \
  --account-name <account> --resource-group <rg> \
  --time-to-restore "2026-02-07T10:00:00Z"
```

**Limitations:**
- Only for **block blobs** in Standard GPv2 accounts
- Cannot restore if account has Data Lake Gen2 (hierarchical namespace)
- Restore period must be less than soft delete retention period

### 4.14 Encryption Key Options (Detailed)

| Key Type | Managed By | Scope | Use Case |
|----------|-----------|-------|----------|
| **Microsoft-managed keys (MMK)** | Microsoft | Entire account | Default, zero config |
| **Customer-managed keys (CMK)** | Customer (Key Vault) | Entire account or per-scope | Compliance requirements |
| **Customer-provided keys (CPK)** | Customer (per request) | Per blob operation | Maximum control |

```csharp
// Customer-provided key (per request)
CustomerProvidedKey cpk = new CustomerProvidedKey(Convert.FromBase64String("<256-bit-key>"));
BlobClientOptions options = new BlobClientOptions
{
    CustomerProvidedKey = cpk
};
BlobClient blobClient = new BlobClient(connectionString, "container", "blob", options);
await blobClient.UploadAsync(content);
```

**Key Differences:**
- **CMK**: Key stored in Azure Key Vault, configured on account level, transparent to SDK
- **CPK**: Key provided in EVERY request header, not stored by Azure, blob is unreadable without the key

### 4.15 Network Security — Additional Details

**Trusted Microsoft Services Exception:**
- Some Azure services need access even when firewall is enabled
- Enable "Allow trusted Microsoft services to access this storage account"
- Includes: Azure Backup, Event Grid, Azure Monitor, Azure DevOps, etc.

```bash
# Set firewall with trusted services exception
az storage account update \
  --name <account> --resource-group <rg> \
  --default-action Deny \
  --bypass AzureServices  # Allow trusted Microsoft services
```

**Disabling Public/Anonymous Access (Account-Level):**
```bash
# Disable all anonymous access at account level
az storage account update \
  --name <account> --resource-group <rg> \
  --allow-blob-public-access false
```

**⚠️ Exam Tip:** When `--allow-blob-public-access false` is set, container-level access (Blob/Container) settings are IGNORED. No anonymous access is possible.

### 4.16 Storage Account Endpoints

Every storage account has multiple service endpoints:

| Service | Endpoint Format |
|---------|----------------|
| Blob | `https://<account>.blob.core.windows.net` |
| Data Lake Gen2 | `https://<account>.dfs.core.windows.net` |
| File | `https://<account>.file.core.windows.net` |
| Queue | `https://<account>.queue.core.windows.net` |
| Table | `https://<account>.table.core.windows.net` |
| Static Website | `https://<account>.z<zone>.web.core.windows.net` |
| Secondary (RA-GRS) | `https://<account>-secondary.blob.core.windows.net` |

### 4.17 Metadata Naming Rules

**Container/Blob Metadata:**
- Names must be valid **C# identifiers**
- Names are **case-insensitive** (stored as lowercase in HTTP headers)
- Names cannot start with numbers
- Values are strings, max 8 KB total per blob
- Metadata is returned via `x-ms-meta-<name>` HTTP headers

```csharp
// Valid metadata names
{ "Author", "Jane" }        // ✅
{ "ProjectName", "Alpha" }  // ✅

// Invalid metadata names
{ "123Invalid", "value" }   // ❌ starts with number
{ "has spaces", "value" }   // ❌ spaces not allowed
{ "has-hyphens", "value" }  // ❌ hyphens not allowed (use underscores)
```

### 4.18 Storage Account Key Rotation

**Best Practice for Key Rotation:**
1. Storage accounts have **2 keys** (primary and secondary)
2. Update all apps to use secondary key
3. Regenerate primary key
4. Update all apps to use new primary key
5. Regenerate secondary key

```bash
# List keys
az storage account keys list --account-name <account> --resource-group <rg>

# Regenerate key1
az storage account keys renew --account-name <account> --resource-group <rg> --key key1

# Regenerate key2
az storage account keys renew --account-name <account> --resource-group <rg> --key key2
```

**⚠️ Better approach:** Use **Azure AD / Managed Identity** instead of keys — no rotation needed.

### 4.19 CORS (Cross-Origin Resource Sharing)

**What is it?**
- Allows web apps on one domain to access blob storage on another domain
- Required when JavaScript in a browser calls Blob Storage REST API directly
- Configured at the **service level** (Blob, File, Queue, Table)

```bash
# Enable CORS for blob service
az storage cors add --account-name <account> \
  --services b \
  --methods GET PUT POST DELETE HEAD OPTIONS \
  --origins "https://www.contoso.com" \
  --allowed-headers "*" \
  --exposed-headers "x-ms-*" \
  --max-age 3600

# List CORS rules
az storage cors list --account-name <account> --services b

# Clear all CORS rules
az storage cors clear --account-name <account> --services b
```

**CORS Rule Properties:**

| Property | Description |
|----------|-------------|
| AllowedOrigins | Domains allowed (e.g., `https://contoso.com`, or `*` for all) |
| AllowedMethods | HTTP methods (GET, PUT, POST, DELETE, HEAD, OPTIONS, MERGE, PATCH) |
| AllowedHeaders | Request headers the origin can send |
| ExposedHeaders | Response headers exposed to the client |
| MaxAgeInSeconds | Browser caches preflight response (max 3600 / 1 hour for some browsers) |

**⚠️ Exam Tip:** Up to **5 CORS rules** per storage service. CORS is checked BEFORE authentication — a failed CORS check returns no data even if the request is valid.

### 4.20 BlobBatchClient — Batch Operations

**What is it?**
- Perform multiple operations in a single HTTP request
- Supports: **DeleteBlobs** and **SetBlobAccessTier** in batch
- Up to **256 operations** per batch
- All blobs must be in the **same storage account**

```csharp
using Azure.Storage.Blobs.Specialized;

BlobBatchClient batchClient = blobServiceClient.GetBlobBatchClient();

// Batch delete multiple blobs
Uri[] blobUris = new Uri[]
{
    containerClient.GetBlobClient("file1.txt").Uri,
    containerClient.GetBlobClient("file2.txt").Uri,
    containerClient.GetBlobClient("file3.txt").Uri
};
await batchClient.DeleteBlobsAsync(blobUris);

// Batch set access tier
await batchClient.SetBlobsAccessTierAsync(blobUris, AccessTier.Cool);

// Advanced: custom batch with mixed operations
BlobBatch batch = batchClient.CreateBatch();
batch.DeleteBlob("container", "blob1.txt");
batch.DeleteBlob("container", "blob2.txt");
batch.SetBlobAccessTier("container", "blob3.txt", AccessTier.Archive);
await batchClient.SubmitBatchAsync(batch);
```

**⚠️ Exam Tip:** Batch only supports **delete** and **set tier**. You CANNOT batch uploads or downloads.

### 4.21 $root and $logs Special Containers

**$root Container:**
- Allows blobs at the root level of the storage account (no container in URL)
- URL: `https://<account>.blob.core.windows.net/myblob.txt` (no container segment)
- Must be explicitly created
- Not commonly used but can appear on exams

**$logs Container:**
- Storage Analytics logs container (auto-created when analytics logging enabled)
- Contains detailed logs of all storage operations
- Read-only (you cannot write to it)
- Lifecycle policies can manage log retention

**$blobchangefeed Container:**
- Stores change feed records (Avro format)
- Auto-created when change feed is enabled

### 4.22 Azurite — Local Development Emulator

**What is Azurite?**
- Open-source Azure Storage emulator for local development
- Supports Blob, Queue, and Table services
- Replaces the legacy Azure Storage Emulator

```bash
# Install via npm
npm install -g azurite

# Start all services
azurite --location ./azurite-data

# Start blob service only
azurite-blob --blobHost 127.0.0.1 --blobPort 10000
```

**Default Connection String:**
```
DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;
```

**Default Endpoints:**

| Service | Endpoint |
|---------|----------|
| Blob | `http://127.0.0.1:10000/devstoreaccount1` |
| Queue | `http://127.0.0.1:10001/devstoreaccount1` |
| Table | `http://127.0.0.1:10002/devstoreaccount1` |

**⚠️ Exam Tip:** Azurite uses `devstoreaccount1` as the account name. URL format differs from real Azure (account name is in the **path**, not subdomain).

### 4.23 Premium Storage — Tier Limitations

**Important Exam Trap:**
- **Premium Block Blob** accounts do **NOT** support Hot/Cool/Cold/Archive tiers
- Premium always runs at premium performance — no tier concept
- **Only Standard GPv2** supports access tiers
- Premium supports **only LRS and ZRS** redundancy (no GRS/GZRS)

**Archive tier limitation:**
- Archive can ONLY be set at the **blob level**, NOT as account default
- Account default tier can only be Hot or Cool

| Account Type | Tiers Supported | Redundancy |
|-------------|-----------------|------------|
| Standard GPv2 | Hot, Cool, Cold, Archive | LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS |
| Premium Block Blob | N/A (always premium) | LRS, ZRS only |
| Premium Page Blob | N/A (always premium) | LRS, ZRS only |

### 4.24 Storage Account Failover

**Customer-Managed Failover (for GRS/GZRS):**
- Manually trigger failover to secondary region
- Secondary becomes new primary after failover
- Storage account degrades to **LRS** after failover (must reconfigure geo-redundancy)
- **RPO** (Recovery Point Objective): Check `Last Sync Time` property

```bash
# Initiate failover
az storage account failover --name <account> --resource-group <rg>

# Check last sync time (data loss since this time)
az storage account show --name <account> --query "geoReplicationStats.lastSyncTime"
```

**⚠️ Exam Tip:** After failover, account becomes LRS. You must reconfigure GRS/GZRS. Data written after `lastSyncTime` may be lost.

### 4.25 Azure CDN with Blob Storage

**Integration Pattern:**
- Serve blob content through Azure CDN for global low-latency access
- CDN caches blobs at edge nodes worldwide
- Commonly combined with static website hosting

```bash
# Create CDN profile and endpoint pointing to blob storage
az cdn profile create --name <cdn-profile> --resource-group <rg> --sku Standard_Microsoft

az cdn endpoint create --name <endpoint> \
  --profile-name <cdn-profile> --resource-group <rg> \
  --origin <account>.blob.core.windows.net \
  --origin-host-header <account>.blob.core.windows.net
```

**CDN Endpoint URL:** `https://<endpoint>.azureedge.net/<container>/<blob>`

### 4.26 ARM Template / Bicep for Storage Account

```bicep
// Bicep template for storage account
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'mystorageaccount'
  location: resourceGroup().location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
  properties: {
    accessTier: 'Hot'
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
  }
}

// Container
resource container 'Microsoft.Storage/storageAccounts/blobServices/containers@2023-01-01' = {
  name: '${storageAccount.name}/default/mycontainer'
  properties: {
    publicAccess: 'None'
  }
}
```

```json
// ARM template excerpt
{
  "type": "Microsoft.Storage/storageAccounts",
  "apiVersion": "2023-01-01",
  "name": "[parameters('storageAccountName')]",
  "location": "[resourceGroup().location]",
  "kind": "StorageV2",
  "sku": { "name": "Standard_LRS" },
  "properties": {
    "accessTier": "Hot",
    "supportsHttpsTrafficOnly": true,
    "allowBlobPublicAccess": false
  }
}
```

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**1. Storage Account Types & Redundancy:**
- Standard GPv2 vs Premium Block/Page/File — when to use each
- LRS/ZRS/GRS/GZRS/RA-GRS/RA-GZRS — durability nines, region support
- Endpoints for each service (blob, dfs, file, queue, table, web)
- Data Lake Gen2: hierarchical namespace (cannot be disabled once enabled)

**2. Blob Types & Access Tiers:**
- Block (documents/media), Page (VM disks), Append (logs) — sizes & use cases
- Hot/Cool/Cold/Archive — costs, min storage durations, rehydration times
- Lifecycle management policies — JSON structure, actions, conditions
- `enableAutoTierToHotFromCool` action for last-access-time scenarios

**3. Security & Authentication:**
- Authentication hierarchy: Azure AD > User Delegation SAS > Service SAS > Shared Key
- SAS token types and creation (User Delegation = most secure)
- RBAC roles: Owner, Contributor, Reader, Delegator
- Encryption: MMK vs CMK (Key Vault) vs CPK (per-request)
- Network: Firewalls, private endpoints, trusted Microsoft services exception
- Account-level disable of public access (`--allow-blob-public-access false`)
- Key rotation strategy (2 keys, rotate safely)

**4. Concurrency & Leases:**
- **Leases** = pessimistic concurrency (lock, 15-60s or infinite)
- **ETags** = optimistic concurrency (If-Match / If-None-Match)
- Lease states: Available, Leased, Expired, Breaking, Broken

**5. Immutable Storage (WORM):**
- Time-based retention vs Legal hold
- Locked vs unlocked policies (locked = irreversible)
- Versioning-level immutability support

**6. Static Website Hosting:**
- `$web` container, separate web endpoint, index/error documents
- No server-side code, combine with CDN for custom domains

**7. SDK & Operations:**
- Client classes: BlobServiceClient → BlobContainerClient → BlobClient
- Block staging for large files (StageBlock + CommitBlockList)
- StorageTransferOptions for parallel upload/download
- Snapshots (manual) vs Versioning (automatic)
- Blob leases for exclusive access during modifications

**8. Advanced Features:**
- Change feed (Avro format, `$blobchangefeed` container)
- Event Grid integration (real-time push events)
- Azure Functions bindings (Blob trigger, input, output)
- Object replication (async, requires versioning + change feed)
- Point-in-time restore (requires soft delete + versioning + change feed)
- AzCopy tool (server-to-server copy, sync, SAS/Azure AD auth)

**9. Data Protection:**
- Soft delete (blob + container, 1-365 days retention)
- Versioning (automatic, per-write)
- Snapshots (manual, point-in-time)
- Blob index tags (query across containers)

### Sample Exam Questions

#### Storage Accounts

**Q1:** You need to create a storage account for storing virtual machine disks. Which storage account type should you use?
- A) Standard General-Purpose v2
- B) Premium Block Blobs
- C) Premium Page Blobs
- D) Premium File Shares

**Answer: C) Premium Page Blobs**
Explanation: Page blobs are optimized for random read/write operations and are the foundation for Azure IaaS disks. Premium provides SSD-based storage for better performance.

---

**Q2:** You need maximum durability and read access to data in a secondary region. Which redundancy option should you choose?
- A) LRS
- B) ZRS
- C) GRS
- D) RA-GZRS

**Answer: D) RA-GZRS**
Explanation: RA-GZRS provides zone-redundancy in the primary region, geo-redundancy to a secondary region, and read access to the secondary region - maximum durability and availability.

---

**Q3:** What is the minimum number of times data is replicated with locally redundant storage (LRS)?
- A) 1
- B) 2
- C) 3
- D) 6

**Answer: C) 3**
Explanation: LRS replicates data three times within a single data center in the primary region.

---

#### Blob Types

**Q4:** You need to implement logging for an application where log entries are continuously appended. Which blob type should you use?
- A) Block blob
- B) Page blob
- C) Append blob
- D) File share

**Answer: C) Append blob**
Explanation: Append blobs are optimized for append operations and are ideal for logging scenarios where data is continuously added to the end.

---

**Q5:** What is the maximum size of a block blob?
- A) 4.75 TiB
- B) 8 TiB
- C) ~190.7 TiB
- D) 500 TiB

**Answer: C) ~190.7 TiB**
Explanation: Block blobs can contain up to approximately 190.7 TiB (50,000 blocks × 4000 MiB per block for the largest block size).

---

**Q6:** Which blob type would you use for storing a virtual machine's operating system disk?
- A) Block blob
- B) Page blob
- C) Append blob
- D) File blob

**Answer: B) Page blob**
Explanation: Page blobs are optimized for random read/write operations and are the foundation for Azure IaaS disks, including OS disks.

---

#### Access Tiers

**Q7:** You have data that is rarely accessed but must be available immediately when needed. Which access tier should you use?
- A) Hot
- B) Cool
- C) Cold
- D) Archive

**Answer: C) Cold**
Explanation: Cold tier is for rarely accessed data (90+ days) but provides immediate access (milliseconds latency), unlike Archive which requires rehydration.

---

**Q8:** What is the minimum storage duration for the Archive access tier before early deletion charges apply?
- A) 30 days
- B) 90 days
- C) 180 days
- D) 365 days

**Answer: C) 180 days**
Explanation: Archive tier has a minimum storage duration of 180 days. Deleting or moving data before this incurs early deletion charges.

---

**Q9:** How long does it typically take to rehydrate a blob from the Archive tier with Standard priority?
- A) Milliseconds
- B) Minutes
- C) Up to 15 hours
- D) Up to 48 hours

**Answer: C) Up to 15 hours**
Explanation: Standard priority rehydration from Archive tier can take up to 15 hours. High priority rehydration for objects <10 GB can complete in under 1 hour.

---

**Q10:** You want to automatically move blobs to Cool tier after 30 days and Archive tier after 90 days. What should you configure?
- A) Azure Policy
- B) Blob lifecycle management policy
- C) Azure Automation runbook
- D) Storage account firewall rules

**Answer: B) Blob lifecycle management policy**
Explanation: Lifecycle management policies allow you to define rules that automatically transition blobs between tiers based on age or other conditions.

---

#### Security

**Q11:** Which type of SAS token is recommended by Microsoft as the most secure option?
- A) Account SAS
- B) Service SAS
- C) User delegation SAS
- D) Container SAS

**Answer: C) User delegation SAS**
Explanation: User delegation SAS is secured with Azure AD credentials rather than the storage account key, making it the most secure option.

---

**Q12:** You need to grant read-only access to a specific blob for 24 hours without sharing your storage account key. What should you use?
- A) Storage account connection string
- B) Shared Access Signature (SAS)
- C) Container public access
- D) Anonymous access

**Answer: B) Shared Access Signature (SAS)**
Explanation: SAS tokens provide delegated, time-limited, scoped access to storage resources without exposing account keys.

---

**Q13:** Which RBAC role allows reading blob data but not writing or deleting?
- A) Storage Blob Data Owner
- B) Storage Blob Data Contributor
- C) Storage Blob Data Reader
- D) Storage Blob Delegator

**Answer: C) Storage Blob Data Reader**
Explanation: Storage Blob Data Reader provides read-only access to blob data. Contributor allows read, write, and delete.

---

**Q14:** How is data encrypted at rest in Azure Blob Storage by default?
- A) 128-bit AES encryption
- B) 256-bit AES encryption
- C) RSA encryption
- D) Data is not encrypted by default

**Answer: B) 256-bit AES encryption**
Explanation: Azure Storage Service Encryption automatically encrypts data at rest using 256-bit AES encryption. This is enabled by default and cannot be disabled.

---

#### Lifecycle Management

**Q15:** In a lifecycle management policy, which filter can you use to apply rules to blobs based on their names?
- A) blobTypes
- B) prefixMatch
- C) containerMatch
- D) nameFilter

**Answer: B) prefixMatch**
Explanation: The prefixMatch filter allows you to specify blob name prefixes to match for the lifecycle rule.

---

**Q16:** How often do lifecycle management policies run?
- A) Every hour
- B) Every 6 hours
- C) Once per day
- D) Every minute

**Answer: C) Once per day**
Explanation: Lifecycle management policies run once per day. Changes may take up to 24 hours to be applied.

---

**Q17:** Which condition is required to use `daysAfterLastAccessTimeGreaterThan` in a lifecycle policy?
- A) Enable blob versioning
- B) Enable soft delete
- C) Enable last access time tracking
- D) Enable change feed

**Answer: C) Enable last access time tracking**
Explanation: Last access time tracking must be enabled on the storage account to use the daysAfterLastAccessTimeGreaterThan condition.

---

#### SDK Operations

**Q18:** Which client class should you use to manage operations at the storage account level?
- A) BlobClient
- B) BlobContainerClient
- C) BlobServiceClient
- D) BlockBlobClient

**Answer: C) BlobServiceClient**
Explanation: BlobServiceClient represents the storage account and is used for account-level operations like listing containers.

---

**Q19:** What is the correct way to upload a blob using the Azure Storage SDK for .NET?
- A) `blobClient.Upload(stream)`
- B) `await blobClient.UploadAsync(stream)`
- C) `containerClient.UploadBlob(stream)`
- D) `blobServiceClient.Upload(stream)`

**Answer: B) `await blobClient.UploadAsync(stream)`**
Explanation: The BlobClient class provides the UploadAsync method for uploading blob content. Async methods are preferred in production code.

---

**Q20:** How do you list blobs in a container using the .NET SDK?
- A) `containerClient.ListBlobs()`
- B) `containerClient.GetBlobs()`
- C) `await foreach (var blob in containerClient.GetBlobsAsync())`
- D) `blobServiceClient.ListBlobs(containerName)`

**Answer: C) `await foreach (var blob in containerClient.GetBlobsAsync())`**
Explanation: GetBlobsAsync() returns an async enumerable that you can iterate over with await foreach.

---

**Q21:** What authentication method is recommended for applications running in Azure?
- A) Connection string with account key
- B) Shared access signature
- C) DefaultAzureCredential / Managed Identity
- D) Username and password

**Answer: C) DefaultAzureCredential / Managed Identity**
Explanation: Managed identities eliminate the need to store credentials in code or configuration and are the recommended approach for Azure-hosted applications.

---

**Q22:** How do you set custom metadata on a blob after it has been uploaded?
- A) `await blobClient.SetMetadataAsync(metadata)`
- B) `await blobClient.UpdateMetadataAsync(metadata)`
- C) `await blobClient.AddMetadataAsync(metadata)`
- D) Metadata can only be set during upload

**Answer: A) `await blobClient.SetMetadataAsync(metadata)`**
Explanation: SetMetadataAsync allows you to set or update custom metadata on an existing blob.

---

**Q23:** Which method returns blob properties including content type, size, and last modified time?
- A) `GetMetadataAsync()`
- B) `GetPropertiesAsync()`
- C) `GetDetailsAsync()`
- D) `GetInfoAsync()`

**Answer: B) `GetPropertiesAsync()`**
Explanation: GetPropertiesAsync returns a BlobProperties object containing system properties like ContentType, ContentLength, and LastModified.

---

#### Advanced Scenarios

**Q24:** You need to find all blobs tagged with "Project=Marketing" across all containers. What should you use?
- A) List blobs with prefix filter
- B) FindBlobsByTagsAsync method
- C) GetBlobsAsync with metadata filter
- D) Azure Search

**Answer: B) FindBlobsByTagsAsync method**
Explanation: FindBlobsByTagsAsync on BlobServiceClient allows you to query blobs by their index tags across all containers.

---

**Q25:** What happens when you enable blob soft delete with a 7-day retention period?
- A) Deleted blobs are permanently removed after 7 days
- B) Deleted blobs can be recovered within 7 days
- C) Blobs automatically move to Archive tier after 7 days
- D) Blob versions are deleted after 7 days

**Answer: B) Deleted blobs can be recovered within 7 days**
Explanation: Soft delete retains deleted blobs for the specified retention period, allowing recovery of accidentally deleted data.

---

**Q26:** You want to copy a blob from one storage account to another. Which method should you use?
- A) `BlobClient.CopyAsync()`
- B) `BlobClient.StartCopyFromUriAsync()`
- C) `BlobContainerClient.CopyBlob()`
- D) `BlobServiceClient.CopyAsync()`

**Answer: B) `BlobClient.StartCopyFromUriAsync()`**
Explanation: StartCopyFromUriAsync initiates an asynchronous copy operation from a source URI to the destination blob.

---

**Q27:** What is the container access level that allows anonymous read access to blobs but does not allow listing blobs in the container?
- A) Private
- B) Blob
- C) Container
- D) Public

**Answer: B) Blob**
Explanation: Blob access level allows anonymous read access to individual blobs (if you know the URL) but does not allow listing blobs in the container.

---

**Q28:** You need to ensure that blob operations retry automatically on transient failures. What should you configure?
- A) Try-catch blocks
- B) BlobClientOptions with Retry settings
- C) Azure Policy
- D) Storage account diagnostic settings

**Answer: B) BlobClientOptions with Retry settings**
Explanation: BlobClientOptions allows you to configure retry policies including MaxRetries, Delay, and retry Mode (Exponential or Fixed).

---

**Q29:** What is the difference between blob properties and blob metadata?
- A) Properties are system-defined, metadata is user-defined
- B) Properties are user-defined, metadata is system-defined
- C) Properties are readable, metadata is writable
- D) There is no difference

**Answer: A) Properties are system-defined, metadata is user-defined**
Explanation: Blob properties are system-defined attributes (ContentType, ContentLength, etc.), while metadata consists of user-defined key-value pairs.

---

**Q30:** How do you generate a SAS URI that grants read access to a blob for 1 hour using the .NET SDK?
- A) `blobClient.GenerateSasUri(sasBuilder)`
- B) `blobClient.CreateSasUri(permissions, expiry)`
- C) `containerClient.GenerateSasUri(blobName)`
- D) `blobServiceClient.GenerateSas()`

**Answer: A) `blobClient.GenerateSasUri(sasBuilder)`**
Explanation: BlobClient.GenerateSasUri takes a BlobSasBuilder with permissions and expiry configured and returns a URI with the SAS token.

---

#### Leases & Concurrency

**Q31:** You need to prevent other processes from modifying a blob while your application updates it. What should you use?
- A) ETag with If-Match header
- B) Blob lease
- C) Stored access policy
- D) Blob versioning

**Answer: B) Blob lease**
Explanation: Leases provide pessimistic concurrency — acquiring a lease locks the blob for exclusive write/delete access. ETags provide optimistic concurrency (detect conflicts after the fact).

---

**Q32:** What is the valid range for a fixed-duration blob lease?
- A) 1-30 seconds
- B) 15-60 seconds
- C) 30-90 seconds
- D) 60-300 seconds

**Answer: B) 15-60 seconds**
Explanation: Blob leases can be 15-60 seconds fixed duration, or infinite (-1). There is no other range.

---

**Q33:** You read a blob and its ETag is "0x8D12345". You want to update it only if no one else has modified it. What condition should you use?
- A) `If-None-Match: "0x8D12345"`
- B) `If-Match: "0x8D12345"`
- C) `If-Modified-Since`
- D) `If-Unmodified-Since`

**Answer: B) `If-Match: "0x8D12345"`**
Explanation: If-Match with the original ETag ensures the update only succeeds if the blob hasn't been modified. If it has, the server returns 412 Precondition Failed.

---

**Q34:** You want to create a blob ONLY if it doesn't already exist. What should you set?
- A) `If-Match: *`
- B) `If-None-Match: *`
- C) `If-Modified-Since: <date>`
- D) Overwrite: false

**Answer: B) `If-None-Match: *`**
Explanation: Setting `If-None-Match: *` causes the operation to fail if the blob already exists. This prevents accidental overwrites.

---

#### Immutable Storage

**Q35:** Your organization must ensure that financial records stored in Blob Storage cannot be modified or deleted for 7 years due to SEC regulations. What should you configure?
- A) Blob versioning
- B) Soft delete with 365-day retention
- C) Time-based retention policy (locked)
- D) Legal hold

**Answer: C) Time-based retention policy (locked)**
Explanation: A locked time-based retention policy prevents modification and deletion for the specified period and cannot be shortened or deleted — meeting regulatory compliance requirements.

---

**Q36:** What is the key difference between a legal hold and a time-based retention policy?
- A) Legal hold has a fixed expiration; retention policy doesn't
- B) Legal hold has no expiration; retention policy expires after the set period
- C) Legal hold allows modifications; retention policy doesn't
- D) Legal hold applies to containers; retention policy applies to blobs

**Answer: B) Legal hold has no expiration; retention policy expires after the set period**
Explanation: Legal hold is indefinite until explicitly cleared. Time-based retention has a fixed period after which blobs can be modified/deleted.

---

**Q37:** You lock a time-based retention policy with a 90-day period. Can you change it to 60 days?
- A) Yes, the policy owner can modify it
- B) Yes, with an Azure support request
- C) No, but you can extend it to more than 90 days
- D) No, locked policies cannot be changed at all

**Answer: C) No, but you can extend it to more than 90 days**
Explanation: Once locked, a time-based retention policy CANNOT be decreased or deleted. It can only be extended.

---

#### Static Website Hosting

**Q38:** What is the name of the container used for static website hosting in Azure Blob Storage?
- A) `$root`
- B) `$web`
- C) `static`
- D) `website`

**Answer: B) `$web`**
Explanation: Static website hosting uses the special `$web` container that is auto-created when static website hosting is enabled.

---

**Q39:** Which endpoint format is used for static website hosting?
- A) `https://<account>.blob.core.windows.net`
- B) `https://<account>.web.core.windows.net`
- C) `https://<account>.z<zone>.web.core.windows.net`
- D) `https://<account>.static.core.windows.net`

**Answer: C) `https://<account>.z<zone>.web.core.windows.net`**
Explanation: Static websites use a different endpoint format that includes a zone identifier, separate from the normal blob endpoint.

---

#### Change Feed & Events

**Q40:** You need a complete, ordered audit trail of all blob changes. What should you enable?
- A) Event Grid subscription
- B) Blob change feed
- C) Azure Monitor diagnostic logs
- D) Storage Analytics logging

**Answer: B) Blob change feed**
Explanation: Change feed provides an ordered, guaranteed-complete log of all changes. Event Grid provides real-time events but doesn't guarantee ordering or completeness.

---

**Q41:** What format are blob change feed records stored in?
- A) JSON
- B) CSV
- C) Apache Avro
- D) Parquet

**Answer: C) Apache Avro**
Explanation: Blob change feed records are stored in Apache Avro format in the `$blobchangefeed` container.

---

**Q42:** You need to trigger an Azure Function immediately when a blob is created. Which is the FASTEST option?
- A) Blob trigger
- B) Event Grid trigger with BlobCreated event
- C) Timer trigger polling the container
- D) Change feed processing

**Answer: B) Event Grid trigger with BlobCreated event**
Explanation: Event Grid triggers fire in seconds. Blob triggers on Consumption plan can have up to 10-minute delay due to polling.

---

#### Azure Functions Bindings

**Q43:** Which binding attribute is used to read a blob in an Azure Function?
- A) `[BlobTrigger]`
- B) `[Blob]` with `FileAccess.Read`
- C) `[BlobInput]`
- D) `[QueueTrigger]`

**Answer: B) `[Blob]` with `FileAccess.Read`**
Explanation: The `[Blob]` attribute with `FileAccess.Read` is the input binding for reading blobs. `[BlobTrigger]` is for triggering on blob changes, not just reading.

---

**Q44:** In a Blob Trigger path `"input/{name}"`, what does `{name}` represent?
- A) The container name
- B) The blob name that triggered the function
- C) A query parameter
- D) The storage account name

**Answer: B) The blob name that triggered the function**
Explanation: `{name}` is a binding expression that captures the blob name and makes it available as a parameter in the function.

---

#### Large Files & AzCopy

**Q45:** You need to upload a 500 GB file to Blob Storage. What is the recommended approach?
- A) Single PUT request
- B) Stage blocks and commit block list
- C) Use page blobs
- D) Split into multiple small blobs

**Answer: B) Stage blocks and commit block list**
Explanation: For large files, the block staging pattern (PutBlock + PutBlockList) is the correct approach. The SDK does this automatically with parallel transfers via StorageTransferOptions.

---

**Q46:** What is the maximum number of blocks in a single block blob?
- A) 1,000
- B) 5,000
- C) 50,000
- D) 100,000

**Answer: C) 50,000**
Explanation: A block blob can contain up to 50,000 blocks.

---

**Q47:** You need to copy 10 TB of data between two storage accounts without downloading to your local machine. What should you use?
- A) Azure Data Factory
- B) AzCopy with server-to-server copy
- C) az storage blob download + upload
- D) Storage Explorer

**Answer: B) AzCopy with server-to-server copy**
Explanation: AzCopy supports server-to-server copy where data transfers directly between storage accounts without passing through the local machine.

---

**Q48:** What is the difference between `azcopy copy` and `azcopy sync`?
- A) `copy` is faster; `sync` is more reliable
- B) `copy` transfers all files; `sync` only transfers new/changed files
- C) `copy` uses HTTP; `sync` uses HTTPS
- D) `copy` is for blobs; `sync` is for files

**Answer: B) `copy` transfers all files; `sync` only transfers new/changed files**
Explanation: `azcopy sync` compares source and destination and only copies differences, similar to rsync.

---

#### Data Lake Gen2

**Q49:** What feature must be enabled to use Azure Data Lake Storage Gen2 on a storage account?
- A) Blob versioning
- B) Hierarchical namespace
- C) Change feed
- D) Soft delete

**Answer: B) Hierarchical namespace**
Explanation: Data Lake Gen2 requires hierarchical namespace enabled at account creation. Once enabled, it cannot be disabled.

---

**Q50:** What is the endpoint format for Data Lake Storage Gen2?
- A) `https://<account>.blob.core.windows.net`
- B) `https://<account>.dfs.core.windows.net`
- C) `https://<account>.lake.core.windows.net`
- D) `https://<account>.adls.core.windows.net`

**Answer: B) `https://<account>.dfs.core.windows.net`**
Explanation: Data Lake Gen2 uses the `dfs` (distributed file system) endpoint, though the regular blob endpoint still works too.

---

#### Encryption

**Q51:** What is the difference between Customer-Managed Keys (CMK) and Customer-Provided Keys (CPK)?
- A) CMK is stored in Key Vault; CPK is provided in every request
- B) CMK is per-blob; CPK is per-account
- C) CMK is free; CPK costs extra
- D) CMK uses AES-128; CPK uses AES-256

**Answer: A) CMK is stored in Key Vault; CPK is provided in every request**
Explanation: CMK is configured at account level and stored in Key Vault (transparent to SDK). CPK must be included in every API request header — the blob is unreadable without providing the key.

---

#### Point-in-Time Restore

**Q52:** What three features must be enabled to use point-in-time restore for blob storage?
- A) Versioning, change feed, soft delete
- B) Versioning, immutable storage, encryption
- C) Change feed, snapshots, soft delete
- D) Versioning, object replication, change feed

**Answer: A) Versioning, change feed, soft delete**
Explanation: Point-in-time restore requires all three: blob versioning, change feed, and blob soft delete to be enabled.

---

**Q53:** Can you use point-in-time restore on an account with hierarchical namespace (Data Lake Gen2) enabled?
- A) Yes, with any configuration
- B) Yes, but only for block blobs
- C) No, point-in-time restore is not supported with hierarchical namespace
- D) Yes, but requires premium tier

**Answer: C) No, point-in-time restore is not supported with hierarchical namespace**
Explanation: Point-in-time restore only works on Standard GPv2 accounts WITHOUT hierarchical namespace.

---

#### Snapshots & Versioning

**Q54:** What is the key difference between blob snapshots and blob versioning?
- A) Snapshots are automatic; versions are manual
- B) Snapshots are manual; versions are automatic on every write
- C) Snapshots support lifecycle management; versions don't
- D) Snapshots are mutable; versions are immutable

**Answer: B) Snapshots are manual; versions are automatic on every write**
Explanation: You must explicitly call CreateSnapshot. Versioning automatically creates a new version on every blob write/update.

---

**Q55:** What query string parameter identifies a blob snapshot?
- A) `?versionId=<datetime>`
- B) `?snapshot=<datetime>`
- C) `?snap=<id>`
- D) `?version=<number>`

**Answer: B) `?snapshot=<datetime>`**
Explanation: Snapshots are identified by `?snapshot=<DateTime>`. Versions use `?versionId=<DateTime>`.

---

#### Object Replication

**Q56:** What prerequisites are required for blob object replication?
- A) Versioning on both accounts, change feed on source
- B) Soft delete on both accounts
- C) Hierarchical namespace on both accounts
- D) Same region for both accounts

**Answer: A) Versioning on both accounts, change feed on source**
Explanation: Object replication requires versioning enabled on both source and destination, plus change feed on the source account.

---

#### Network Security

**Q57:** You set `--allow-blob-public-access false` on a storage account. A container has public access level set to "Blob". Can anonymous users access blobs?
- A) Yes, the container setting takes precedence
- B) No, the account-level setting overrides container settings
- C) Yes, but only with the direct blob URL
- D) Only if the firewall allows the IP

**Answer: B) No, the account-level setting overrides container settings**
Explanation: When public access is disabled at the account level, all container-level access settings are ignored.

---

**Q58:** Which option allows Azure Backup, Event Grid, and other Azure services to access your storage account even when the firewall denies all traffic?
- A) `--bypass AzureServices`
- B) `--allow-trusted-services true`
- C) Add Azure service IPs to the firewall
- D) Use a private endpoint

**Answer: A) `--bypass AzureServices`**
Explanation: The `--bypass AzureServices` flag (or "Allow trusted Microsoft services" in the portal) lets specific Azure services bypass firewall rules.

---

#### Stored Access Policies & SAS

**Q59:** What is the maximum number of stored access policies you can define on a single container?
- A) 3
- B) 5
- C) 10
- D) Unlimited

**Answer: B) 5**
Explanation: Each container can have up to 5 stored access policies.

---

**Q60:** How can you immediately revoke a Service SAS token that was created with a stored access policy?
- A) Regenerate the storage account key
- B) Delete or modify the stored access policy
- C) Both A and B
- D) SAS tokens cannot be revoked

**Answer: C) Both A and B**
Explanation: You can revoke by deleting/modifying the stored access policy OR regenerating the account key. Stored access policies give you finer control without affecting other SAS tokens.

---

#### CORS, Batch, Premium, Failover

**Q61:** A JavaScript web application hosted on `https://app.contoso.com` needs to read blobs directly from Azure Storage. What must you configure on the storage account?
- A) A firewall rule for the app's IP
- B) CORS rules allowing the origin
- C) A stored access policy
- D) Anonymous public access

**Answer: B) CORS rules allowing the origin**
Explanation: CORS must be configured to allow cross-origin requests from the web application's domain. Without it, the browser blocks the request.

---

**Q62:** What is the maximum number of CORS rules per storage service?
- A) 3
- B) 5
- C) 10
- D) 20

**Answer: B) 5**
Explanation: Each storage service (blob, file, queue, table) can have up to 5 CORS rules.

---

**Q63:** You need to delete 200 blobs as quickly as possible. What should you use?
- A) Loop calling DeleteAsync for each blob
- B) BlobBatchClient.DeleteBlobsAsync
- C) AzCopy remove
- D) Lifecycle management policy

**Answer: B) BlobBatchClient.DeleteBlobsAsync**
Explanation: BlobBatchClient sends up to 256 operations in a single HTTP request, making it the fastest approach for bulk delete/tier-change operations.

---

**Q64:** Which operations does BlobBatchClient support?
- A) Delete and Upload
- B) Delete and Set Access Tier
- C) Upload and Download
- D) Delete, Upload, and Set Tier

**Answer: B) Delete and Set Access Tier**
Explanation: BlobBatchClient only supports DeleteBlob and SetBlobAccessTier operations. Uploads and downloads are not supported in batch.

---

**Q65:** You create a Premium Block Blob storage account. Which access tier should you set?
- A) Hot
- B) Cool
- C) Archive
- D) Premium accounts do not support access tiers

**Answer: D) Premium accounts do not support access tiers**
Explanation: Premium storage accounts always operate at premium performance. Access tiers (Hot/Cool/Cold/Archive) are only available on Standard GPv2 accounts.

---

**Q66:** Can you set Archive as the default access tier for a storage account?
- A) Yes, for any storage account
- B) Yes, for Standard GPv2 only
- C) No, account default can only be Hot or Cool
- D) No, Archive is not a valid tier

**Answer: C) No, account default can only be Hot or Cool**
Explanation: Archive can only be set at the individual blob level, NOT as the account default. Account default is limited to Hot or Cool.

---

**Q67:** After a customer-managed failover of a GRS storage account, what happens to the redundancy configuration?
- A) It remains GRS
- B) It changes to ZRS
- C) It degrades to LRS
- D) It changes to RA-GRS

**Answer: C) It degrades to LRS**
Explanation: After failover, the secondary becomes the new primary, but the account degrades to LRS. You must manually reconfigure geo-redundancy.

---

**Q68:** What is the default connection string account name when using Azurite for local development?
- A) `localhost`
- B) `azurite`
- C) `devstoreaccount1`
- D) `storageemulator`

**Answer: C) `devstoreaccount1`**
Explanation: Azurite uses `devstoreaccount1` as the default account name. The URL format puts the account name in the path, not the subdomain.

---

**Q69:** You want to serve blob content globally with low latency. What should you add in front of Blob Storage?
- A) Azure Application Gateway
- B) Azure CDN / Front Door
- C) Azure Load Balancer
- D) Azure Traffic Manager

**Answer: B) Azure CDN / Front Door**
Explanation: Azure CDN (or Azure Front Door) caches blob content at global edge nodes, reducing latency for users worldwide.

---

**Q70:** In an ARM template, what `kind` value should you use for a standard general-purpose v2 storage account?
- A) `Storage`
- B) `StorageV2`
- C) `BlobStorage`
- D) `BlockBlobStorage`

**Answer: B) `StorageV2`**
Explanation: `StorageV2` is the kind for Standard General-Purpose v2 accounts. `BlobStorage` is legacy, `BlockBlobStorage` is for premium block blobs, `Storage` is GPv1.

---

## Key CLI Commands Reference

### Storage Account Management
```bash
# Create storage account
az storage account create \
  --name <storage-account> \
  --resource-group <rg> \
  --location <location> \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot

# Show storage account
az storage account show \
  --name <storage-account> \
  --resource-group <rg>

# Get connection string
az storage account show-connection-string \
  --name <storage-account> \
  --resource-group <rg>

# Get account keys
az storage account keys list \
  --account-name <storage-account> \
  --resource-group <rg>
```

### Container Operations
```bash
# Create container
az storage container create \
  --name <container-name> \
  --account-name <storage-account>

# List containers
az storage container list \
  --account-name <storage-account>

# Delete container
az storage container delete \
  --name <container-name> \
  --account-name <storage-account>

# Set container access level
az storage container set-permission \
  --name <container-name> \
  --account-name <storage-account> \
  --public-access blob
```

### Blob Operations
```bash
# Upload blob
az storage blob upload \
  --account-name <storage-account> \
  --container-name <container> \
  --name <blob-name> \
  --file <local-file-path>

# Download blob
az storage blob download \
  --account-name <storage-account> \
  --container-name <container> \
  --name <blob-name> \
  --file <local-file-path>

# List blobs
az storage blob list \
  --account-name <storage-account> \
  --container-name <container>

# Delete blob
az storage blob delete \
  --account-name <storage-account> \
  --container-name <container> \
  --name <blob-name>

# Set blob tier
az storage blob set-tier \
  --account-name <storage-account> \
  --container-name <container> \
  --name <blob-name> \
  --tier Cool

# Show blob properties
az storage blob show \
  --account-name <storage-account> \
  --container-name <container> \
  --name <blob-name>
```

### Lifecycle Management
```bash
# Create lifecycle policy
az storage account management-policy create \
  --account-name <storage-account> \
  --resource-group <rg> \
  --policy @policy.json

# Show lifecycle policy
az storage account management-policy show \
  --account-name <storage-account> \
  --resource-group <rg>

# Delete lifecycle policy
az storage account management-policy delete \
  --account-name <storage-account> \
  --resource-group <rg>
```

### SAS Generation
```bash
# Generate blob SAS
az storage blob generate-sas \
  --account-name <storage-account> \
  --container-name <container> \
  --name <blob-name> \
  --permissions r \
  --expiry 2024-12-31T23:59:59Z \
  --https-only

# Generate container SAS
az storage container generate-sas \
  --account-name <storage-account> \
  --name <container> \
  --permissions rl \
  --expiry 2024-12-31T23:59:59Z
```

---

## EXAM GOTCHAS & TRICKY DISTINCTIONS

### Common Exam Traps

| Trap | Truth |
|------|-------|
| "Archive tier provides millisecond access" | ❌ Archive requires rehydration (hours) |
| "Blob trigger fires instantly" | ❌ Up to 10-minute delay on Consumption plan; use Event Grid trigger for instant |
| "Snapshots are automatic" | ❌ Snapshots are manual; Versioning is automatic |
| "You can disable hierarchical namespace after enabling" | ❌ Once enabled, it's permanent |
| "Container public access overrides account setting" | ❌ Account-level disable wins over container settings |
| "SAS tokens can be revoked directly" | ❌ Only via: stored access policy change, key regeneration, or token expiration |
| "Locking a retention policy can be undone" | ❌ Locking is IRREVERSIBLE |
| "Point-in-time restore works with Data Lake Gen2" | ❌ Not supported with hierarchical namespace |
| "Object replication is synchronous" | ❌ It's asynchronous |
| "CPK keys are stored by Azure" | ❌ Customer-provided keys are NOT stored; must be sent every request |
| "Premium Block Blob supports Hot/Cool tiers" | ❌ Premium has NO access tiers; tiers are Standard GPv2 only |
| "Archive can be the account default tier" | ❌ Account default is only Hot or Cool; Archive is blob-level only |
| "GRS is maintained after failover" | ❌ After failover, account degrades to LRS; must reconfigure |
| "Batch supports uploads" | ❌ BlobBatchClient only supports Delete and SetAccessTier |
| "Azurite uses account name in subdomain" | ❌ Azurite uses `devstoreaccount1` in the URL path, not subdomain |
| "CORS is checked after authentication" | ❌ CORS is checked BEFORE auth; failed CORS = no data returned |

### Key Numbers to Remember

| Item | Value |
|------|-------|
| Blob lease fixed range | 15-60 seconds (or infinite) |
| Max stored access policies per container | 5 |
| Max blob index tags per blob | 10 |
| Max block blob size | ~190.7 TiB |
| Max page blob size | 8 TiB |
| Max append blob size | ~195 GiB |
| Max blocks per block blob | 50,000 |
| Max single PutBlob | 5000 MiB |
| Cool minimum storage | 30 days |
| Cold minimum storage | 90 days |
| Archive minimum storage | 180 days |
| Archive rehydration (standard) | Up to 15 hours |
| Archive rehydration (high priority) | Under 1 hour (<10 GB) |
| Soft delete retention | 1-365 days |
| Blob trigger latency (Consumption) | Up to 10 minutes |
| Storage account name | 3-24 chars, lowercase + numbers only |
| Container name | 3-63 chars, lowercase + numbers + hyphens |
| Metadata max total size | 8 KB per blob |
| LRS durability | 11 nines |
| ZRS durability | 12 nines |
| GRS/GZRS durability | 16 nines |
| Max CORS rules per service | 5 |
| Max batch operations | 256 per request |
| Azurite default account | `devstoreaccount1` |
| Account default tiers | Hot or Cool ONLY (not Archive) |

### Concurrency Cheat Sheet

| Scenario | Use |
|----------|-----|
| Prevent concurrent modifications (lock) | **Blob Lease** (pessimistic) |
| Detect concurrent modifications (no lock) | **ETag + If-Match** (optimistic) |
| Create only if not exists | **If-None-Match: \*** |
| Update only if changed | **If-Match: <etag>** → 412 if changed |
| Update only if NOT changed | **If-None-Match: <etag>** |

### Authentication Hierarchy (Most → Least Secure)
1. **Azure AD + Managed Identity** → No credentials in code
2. **User Delegation SAS** → Azure AD-backed, blob only
3. **Service SAS + Stored Access Policy** → Revocable
4. **Service SAS** → Cannot revoke without key rotation
5. **Account SAS** → Broad access, avoid if possible
6. **Shared Key (Account Key)** → Full access, last resort

### Tier Transition Rules
```
Hot → Cool → Cold → Archive  (lifecycle mgmt can automate)
Archive → Cold/Cool/Hot       (requires rehydration, manual or copy)

CANNOT auto-transition from colder → hotter via lifecycle policy
CAN use enableAutoTierToHotFromCool (moves back on access)
```

### When to Use What

| Need | Use |
|------|-----|
| Real-time blob event reaction | Event Grid trigger |
| Audit trail of all blob changes | Change feed |
| Prevent blob deletion for compliance | Immutable storage (WORM) |
| Prevent accidental deletion | Soft delete |
| Restore blobs to earlier state | Point-in-time restore |
| Host a simple website cheaply | Static website hosting (`$web`) |
| Copy TB of data between accounts | AzCopy (server-to-server) |
| Upload files > 256 MB efficiently | Block staging / StorageTransferOptions |
| Lock blob during update | Blob lease |
| Detect conflicting updates | ETags |
| Big data analytics on blob data | Data Lake Gen2 (hierarchical namespace) |
| Replicate blobs across regions | Object replication |

---

## Study Tips for AZ-204 Exam

1. **Understand blob types thoroughly** — Block (general), Page (VMs), Append (logs) — know sizes
2. **Master access tiers** — costs, minimum durations, rehydration times, lifecycle policies
3. **Practice SDK code** — BlobServiceClient, BlobContainerClient, BlobClient, BlockBlobClient
4. **Know authentication methods** — Especially User Delegation SAS vs Service SAS vs Shared Key
5. **Understand lifecycle policies** — JSON structure, actions, conditions, tier transition rules
6. **Leases & ETags** — Pessimistic vs optimistic concurrency, lease durations, ETag conditions
7. **Immutable storage** — Time-based retention (locked/unlocked) vs Legal hold — compliance scenarios
8. **Static website** — `$web` container, different endpoint, no server-side code
9. **Event Grid vs Change Feed** — Push (real-time) vs Pull (ordered, complete)
10. **Azure Functions bindings** — Blob trigger (polling, slow), Event Grid trigger (fast), input/output
11. **AzCopy** — copy vs sync, server-to-server, SAS/Azure AD auth
12. **Data Lake Gen2** — hierarchical namespace, dfs endpoint, cannot disable once enabled
13. **Encryption** — MMK (default) vs CMK (Key Vault) vs CPK (per-request, not stored)
14. **Key numbers** — lease range, max sizes, tier minimums, stored access policy limits
15. **Network security** — trusted services bypass, account-level public access disable

---

## Important URLs and References

- Azure Blob Storage Documentation: https://learn.microsoft.com/en-us/azure/storage/blobs/
- Storage Redundancy: https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy
- Lifecycle Management: https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview
- SAS Documentation: https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview
- .NET SDK Reference: https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs
- Immutable Storage: https://learn.microsoft.com/en-us/azure/storage/blobs/immutable-storage-overview
- Static Website: https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website
- AzCopy: https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10
- Data Lake Gen2: https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction
- Blob Change Feed: https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-change-feed
- Event Grid + Blob: https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-event-overview

---

## Quick Reference Summary

### Storage Account Types
- **Standard General-Purpose v2**: Most scenarios, all blob types, all redundancy options
- **Premium Block Blobs**: High-performance block/append blobs, LRS/ZRS only
- **Premium Page Blobs**: VM disks, high-performance random I/O

### Blob Types
- **Block Blobs**: Documents, images, videos (up to ~190.7 TiB, 50,000 blocks)
- **Append Blobs**: Logging scenarios (up to ~195 GiB)
- **Page Blobs**: VM disks, random I/O (up to 8 TiB, 512-byte pages)

### Access Tiers
- **Hot**: Frequent access, highest storage cost, lowest access cost, no minimum
- **Cool**: Infrequent access, 30-day minimum
- **Cold**: Rarely accessed, 90-day minimum
- **Archive**: Long-term archive, 180-day minimum, requires rehydration (up to 15 hours)

### Redundancy Options
- **LRS**: 3 copies in single data center (11 nines)
- **ZRS**: 3 copies across availability zones (12 nines)
- **GRS**: LRS + async to secondary region (16 nines)
- **GZRS**: ZRS + async to secondary region (16 nines)
- **RA-GRS/RA-GZRS**: Add read access to secondary

### Authentication (Most → Least Secure)
- **Azure AD + Managed Identity** → DefaultAzureCredential (recommended)
- **User Delegation SAS** → Azure AD backed, most secure SAS
- **Service SAS** → Scoped to single service, use stored access policies
- **Shared Key** → Account keys, full access, rotate regularly

### SDK Client Hierarchy
- **BlobServiceClient** → Account-level (list containers, account info)
- **BlobContainerClient** → Container-level (CRUD, list blobs)
- **BlobClient** → Blob-level (upload, download, properties, metadata)
- **BlockBlobClient** → Block staging for large files
- **AppendBlobClient** → Append operations
- **BlobLeaseClient** → Lease management

### Concurrency
- **Pessimistic**: Blob lease (15-60s or infinite, lock blob)
- **Optimistic**: ETag + If-Match / If-None-Match (detect conflicts)

### Data Protection
- **Soft delete**: Recoverable deletion (1-365 days)
- **Versioning**: Auto-version on every write
- **Snapshots**: Manual point-in-time copies
- **Immutable storage**: WORM (time-based retention / legal hold)
- **Point-in-time restore**: Requires soft delete + versioning + change feed

