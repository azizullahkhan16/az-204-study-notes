# AZ-204: Implement Secure Azure Solutions - Complete Study Notes

## Learning Path Overview
This learning path covers implementing secure solutions in Azure with 3 modules:
1. Implement Azure Key Vault
2. Implement managed identities
3. Implement Azure App Configuration

**Level:** Intermediate | **Focus:** Cloud Security

---

## MODULE 1: IMPLEMENT AZURE KEY VAULT

### 1.1 Introduction to Azure Key Vault

**What is Azure Key Vault?**
- Centralized cloud service for storing secrets securely
- Helps safeguard cryptographic keys and secrets
- Reduces risk of accidental secret exposure
- Provides secure access with authentication and authorization

**Key Vault Can Store:**
- **Secrets**: Passwords, connection strings, API keys, tokens
- **Keys**: Cryptographic keys for encryption
- **Certificates**: SSL/TLS certificates

**Key Benefits:**
- Centralized secret management
- Secure access control (RBAC + access policies)
- Monitoring and logging
- Integration with Azure services
- Hardware Security Module (HSM) support
- Automatic certificate renewal
- Simplified application deployment

**Use Cases:**
- Store database connection strings
- Store API keys and tokens
- Manage SSL/TLS certificates
- Encrypt application data
- Store encryption keys
- Secure DevOps pipelines

### 1.2 Azure Key Vault Service Tiers

**Key Vault Tiers:**

| Feature | Standard | Premium |
|---------|----------|---------|
| Secrets | ✅ | ✅ |
| Keys (Software) | ✅ | ✅ |
| Keys (HSM) | ❌ | ✅ |
| Certificates | ✅ | ✅ |
| Managed HSM | ❌ | ✅ |
| Price | Lower | Higher |

**Hardware Security Module (HSM):**
- FIPS 140-2 Level 2 validated (Standard)
- FIPS 140-2 Level 3 validated (Premium HSM)
- Keys never leave HSM boundary
- Required for high-security scenarios

### 1.3 Key Vault Best Practices

**Security Best Practices:**

1. **Use separate vaults for different environments**
   - Development, Staging, Production
   - Reduces blast radius if compromised

2. **Enable soft-delete and purge protection**
   - Prevents accidental deletion
   - Required for some compliance scenarios

3. **Use managed identities for access**
   - No credentials in code
   - Automatic credential rotation

4. **Enable logging and monitoring**
   - Azure Monitor integration
   - Alert on suspicious access

5. **Apply principle of least privilege**
   - Grant minimum required permissions
   - Use separate access policies

6. **Backup important keys and secrets**
   - Regular backups for disaster recovery
   - Store backups securely

7. **Use private endpoints**
   - Restrict network access
   - Keep traffic within Azure backbone

**Operational Best Practices:**

```bash
# Enable soft-delete (now enabled by default)
az keyvault update --name mykeyvault --enable-soft-delete true

# Enable purge protection
az keyvault update --name mykeyvault --enable-purge-protection true

# Set soft-delete retention (7-90 days)
az keyvault update --name mykeyvault --retention-days 90
```

### 1.4 Authentication and Authorization

**Authentication Methods:**

1. **Azure AD Authentication (Required)**
   - All requests must be authenticated with Azure AD
   - Service principals, managed identities, or users

2. **Access Control Models:**

#### **Azure RBAC (Recommended)**
- Built-in roles for Key Vault
- Granular permissions
- Inherited from management groups/subscriptions

| Role | Secrets | Keys | Certificates |
|------|---------|------|--------------|
| Key Vault Administrator | Full | Full | Full |
| Key Vault Secrets Officer | Full | ❌ | ❌ |
| Key Vault Secrets User | Read | ❌ | ❌ |
| Key Vault Crypto Officer | ❌ | Full | ❌ |
| Key Vault Crypto User | ❌ | Use | ❌ |
| Key Vault Certificates Officer | ❌ | ❌ | Full |
| Key Vault Reader | List | List | List |

```bash
# Assign RBAC role
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <principal-id> \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault>
```

#### **Access Policies (Legacy)**
- Vault-level permissions
- Assigned to security principals
- All-or-nothing per operation type

```bash
# Set access policy
az keyvault set-policy \
  --name mykeyvault \
  --object-id <principal-id> \
  --secret-permissions get list \
  --key-permissions get list \
  --certificate-permissions get list
```

**Permission Model Comparison:**

| Aspect | RBAC | Access Policies |
|--------|------|-----------------|
| Granularity | Role-based | Vault-level |
| Inheritance | Yes | No |
| Management | Azure-wide | Per-vault |
| Recommended | Yes | Legacy |

### 1.5 Creating and Managing Key Vault

**Create Key Vault - Azure CLI:**

```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# Create key vault
az keyvault create \
  --name mykeyvault \
  --resource-group myResourceGroup \
  --location eastus \
  --sku standard \
  --enable-rbac-authorization true

# Create with soft-delete and purge protection
az keyvault create \
  --name mykeyvault \
  --resource-group myResourceGroup \
  --location eastus \
  --enable-soft-delete true \
  --enable-purge-protection true \
  --retention-days 90
```

**Create Key Vault - Azure Portal:**
```
Azure Portal → Create a resource → Key Vault
- Basics: Name, Region, Pricing tier
- Access configuration: RBAC or Access policies
- Networking: Public or Private endpoint
- Tags: Optional tags
```

### 1.6 Working with Secrets

**Secret Operations - Azure CLI:**

```bash
# Create/Set secret
az keyvault secret set \
  --vault-name mykeyvault \
  --name MySecret \
  --value "SuperSecretValue123"

# Get secret
az keyvault secret show \
  --vault-name mykeyvault \
  --name MySecret

# Get secret value only
az keyvault secret show \
  --vault-name mykeyvault \
  --name MySecret \
  --query value \
  --output tsv

# List secrets
az keyvault secret list \
  --vault-name mykeyvault

# Delete secret (soft-delete)
az keyvault secret delete \
  --vault-name mykeyvault \
  --name MySecret

# Recover deleted secret
az keyvault secret recover \
  --vault-name mykeyvault \
  --name MySecret

# Purge secret permanently
az keyvault secret purge \
  --vault-name mykeyvault \
  --name MySecret

# Set secret with expiration
az keyvault secret set \
  --vault-name mykeyvault \
  --name MySecret \
  --value "SecretValue" \
  --expires "2025-12-31T23:59:59Z"

# Set secret with content type
az keyvault secret set \
  --vault-name mykeyvault \
  --name ConnectionString \
  --value "Server=myserver;Database=mydb" \
  --content-type "application/x-connection-string"
```

**Secret Versioning:**
- Each update creates new version
- Previous versions retained
- Access specific version by version ID
- Latest version is default

```bash
# List secret versions
az keyvault secret list-versions \
  --vault-name mykeyvault \
  --name MySecret

# Get specific version
az keyvault secret show \
  --vault-name mykeyvault \
  --name MySecret \
  --version <version-id>
```

### 1.7 Working with Keys

**Key Types:**
- **RSA**: Asymmetric encryption, signing
- **EC**: Elliptic curve cryptography
- **oct**: Symmetric keys (HSM only)

**Key Operations:**

```bash
# Create RSA key
az keyvault key create \
  --vault-name mykeyvault \
  --name MyKey \
  --kty RSA \
  --size 2048

# Create EC key
az keyvault key create \
  --vault-name mykeyvault \
  --name MyECKey \
  --kty EC \
  --curve P-256

# Create HSM-protected key (Premium)
az keyvault key create \
  --vault-name mykeyvault \
  --name MyHSMKey \
  --kty RSA-HSM \
  --size 2048

# List keys
az keyvault key list --vault-name mykeyvault

# Get key
az keyvault key show \
  --vault-name mykeyvault \
  --name MyKey

# Delete key
az keyvault key delete \
  --vault-name mykeyvault \
  --name MyKey

# Backup key
az keyvault key backup \
  --vault-name mykeyvault \
  --name MyKey \
  --file mykey-backup.blob

# Restore key
az keyvault key restore \
  --vault-name mykeyvault \
  --file mykey-backup.blob
```

### 1.8 Working with Certificates

**Certificate Sources:**
- Self-signed certificates
- Certificate Authority (CA) integrated
- Import existing certificates

**Certificate Operations:**

```bash
# Create self-signed certificate
az keyvault certificate create \
  --vault-name mykeyvault \
  --name MyCert \
  --policy "$(az keyvault certificate get-default-policy)"

# Import certificate
az keyvault certificate import \
  --vault-name mykeyvault \
  --name MyCert \
  --file certificate.pfx \
  --password "certpassword"

# List certificates
az keyvault certificate list --vault-name mykeyvault

# Get certificate
az keyvault certificate show \
  --vault-name mykeyvault \
  --name MyCert

# Download certificate
az keyvault certificate download \
  --vault-name mykeyvault \
  --name MyCert \
  --file mycert.pem

# Delete certificate
az keyvault certificate delete \
  --vault-name mykeyvault \
  --name MyCert
```

### 1.9 Key Vault SDK for .NET

**Installation:**

```bash
dotnet add package Azure.Security.KeyVault.Secrets
dotnet add package Azure.Security.KeyVault.Keys
dotnet add package Azure.Security.KeyVault.Certificates
dotnet add package Azure.Identity
```

**SecretClient Operations:**

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

// Create client with managed identity
var client = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net/"),
    new DefaultAzureCredential());

// Create client with specific credential
var client = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net/"),
    new ManagedIdentityCredential());

// Set secret
KeyVaultSecret secret = await client.SetSecretAsync("MySecret", "SecretValue");
Console.WriteLine($"Secret created: {secret.Name}, Version: {secret.Properties.Version}");

// Get secret
KeyVaultSecret secret = await client.GetSecretAsync("MySecret");
Console.WriteLine($"Secret value: {secret.Value}");

// Get specific version
KeyVaultSecret secret = await client.GetSecretAsync("MySecret", version: "abc123");

// List secrets
await foreach (SecretProperties secretProperties in client.GetPropertiesOfSecretsAsync())
{
    Console.WriteLine($"Secret: {secretProperties.Name}");
}

// List secret versions
await foreach (SecretProperties version in client.GetPropertiesOfSecretVersionsAsync("MySecret"))
{
    Console.WriteLine($"Version: {version.Version}, Created: {version.CreatedOn}");
}

// Delete secret
DeleteSecretOperation operation = await client.StartDeleteSecretAsync("MySecret");
await operation.WaitForCompletionAsync();

// Recover deleted secret
RecoverDeletedSecretOperation operation = await client.StartRecoverDeletedSecretAsync("MySecret");
await operation.WaitForCompletionAsync();

// Purge deleted secret
await client.PurgeDeletedSecretAsync("MySecret");
```

**KeyClient Operations:**

```csharp
using Azure.Security.KeyVault.Keys;
using Azure.Security.KeyVault.Keys.Cryptography;

var keyClient = new KeyClient(
    new Uri("https://mykeyvault.vault.azure.net/"),
    new DefaultAzureCredential());

// Create key
KeyVaultKey key = await keyClient.CreateKeyAsync("MyKey", KeyType.Rsa);

// Create RSA key with options
var rsaKeyOptions = new CreateRsaKeyOptions("MyRsaKey")
{
    KeySize = 2048,
    ExpiresOn = DateTimeOffset.UtcNow.AddYears(1)
};
KeyVaultKey key = await keyClient.CreateRsaKeyAsync(rsaKeyOptions);

// Get key
KeyVaultKey key = await keyClient.GetKeyAsync("MyKey");

// Encrypt/Decrypt with CryptographyClient
var cryptoClient = new CryptographyClient(key.Id, new DefaultAzureCredential());

// Encrypt
byte[] plaintext = Encoding.UTF8.GetBytes("Hello, World!");
EncryptResult encryptResult = await cryptoClient.EncryptAsync(EncryptionAlgorithm.RsaOaep, plaintext);

// Decrypt
DecryptResult decryptResult = await cryptoClient.DecryptAsync(EncryptionAlgorithm.RsaOaep, encryptResult.Ciphertext);
string decrypted = Encoding.UTF8.GetString(decryptResult.Plaintext);
```

**CertificateClient Operations:**

```csharp
using Azure.Security.KeyVault.Certificates;

var certClient = new CertificateClient(
    new Uri("https://mykeyvault.vault.azure.net/"),
    new DefaultAzureCredential());

// Create self-signed certificate
CertificateOperation operation = await certClient.StartCreateCertificateAsync(
    "MyCert",
    CertificatePolicy.Default);

KeyVaultCertificateWithPolicy certificate = await operation.WaitForCompletionAsync();

// Get certificate
KeyVaultCertificateWithPolicy cert = await certClient.GetCertificateAsync("MyCert");

// Download certificate with private key (as secret)
SecretClient secretClient = new SecretClient(vaultUri, credential);
KeyVaultSecret certSecret = await secretClient.GetSecretAsync(cert.Name);
byte[] pfxBytes = Convert.FromBase64String(certSecret.Value);
X509Certificate2 x509 = new X509Certificate2(pfxBytes);
```

### 1.10 Key Vault References in App Configuration

**App Service / Azure Functions:**

```bash
# Reference secret in app settings
@Microsoft.KeyVault(SecretUri=https://mykeyvault.vault.azure.net/secrets/MySecret/)

# With specific version
@Microsoft.KeyVault(SecretUri=https://mykeyvault.vault.azure.net/secrets/MySecret/abc123)

# Alternative syntax
@Microsoft.KeyVault(VaultName=mykeyvault;SecretName=MySecret)
```

**Requirements:**
- App Service must have managed identity
- Identity must have access to Key Vault
- Key Vault reference syntax in app settings

---

## MODULE 2: IMPLEMENT MANAGED IDENTITIES

### 2.1 Introduction to Managed Identities

**What are Managed Identities?**
- Azure AD identities for Azure resources
- Eliminates need to manage credentials
- Automatic credential rotation
- No secrets in code or configuration

**Benefits:**
- No credential management
- Secure by default
- Automatic rotation
- Works with Azure services
- Simplified application code
- Reduced attack surface

### 2.2 Types of Managed Identities

**System-Assigned Managed Identity:**
- Created as part of Azure resource
- Tied to resource lifecycle
- Deleted when resource is deleted
- One-to-one relationship with resource
- Cannot be shared between resources

```bash
# Enable system-assigned identity on App Service
az webapp identity assign \
  --name mywebapp \
  --resource-group myResourceGroup

# Enable on VM
az vm identity assign \
  --name myvm \
  --resource-group myResourceGroup

# Enable on Azure Function
az functionapp identity assign \
  --name myfunctionapp \
  --resource-group myResourceGroup
```

**User-Assigned Managed Identity:**
- Standalone Azure resource
- Independent lifecycle
- Can be assigned to multiple resources
- Managed separately from resources
- Reusable across resources

```bash
# Create user-assigned identity
az identity create \
  --name myuseridentity \
  --resource-group myResourceGroup

# Get identity client ID
az identity show \
  --name myuseridentity \
  --resource-group myResourceGroup \
  --query clientId

# Assign to App Service
az webapp identity assign \
  --name mywebapp \
  --resource-group myResourceGroup \
  --identities /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuseridentity

# Assign to VM
az vm identity assign \
  --name myvm \
  --resource-group myResourceGroup \
  --identities /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuseridentity
```

**Comparison:**

| Aspect | System-Assigned | User-Assigned |
|--------|----------------|---------------|
| Creation | With resource | Standalone |
| Lifecycle | Tied to resource | Independent |
| Sharing | Not shareable | Shareable |
| Use Case | Single resource | Multiple resources |
| Management | Automatic | Manual |
| Deletion | With resource | Manual |

### 2.3 How Managed Identities Work

**Architecture:**

```
Azure Resource (App Service, VM, etc.)
    │
    │ 1. Request token for resource
    ▼
Azure Instance Metadata Service (IMDS)
    │
    │ 2. Request token from Azure AD
    ▼
Azure Active Directory
    │
    │ 3. Return access token
    ▼
Azure Resource
    │
    │ 4. Use token to access target resource
    ▼
Target Resource (Key Vault, Storage, etc.)
```

**Token Acquisition:**
- Azure resources contact IMDS endpoint
- IMDS endpoint: `http://169.254.169.254/metadata/identity/oauth2/token`
- No credentials needed - identity verified by Azure
- Token returned for requested resource

**Manual Token Request (for understanding):**

```bash
# Request token from IMDS (within Azure VM/container)
curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net' \
  -H 'Metadata: true'
```

### 2.4 Using Managed Identities with Azure Services

**Azure SDK - DefaultAzureCredential:**

```csharp
using Azure.Identity;

// Automatically uses managed identity in Azure, developer credentials locally
var credential = new DefaultAzureCredential();

// Key Vault
var secretClient = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net/"),
    credential);

// Storage
var blobClient = new BlobServiceClient(
    new Uri("https://mystorage.blob.core.windows.net/"),
    credential);

// Cosmos DB
var cosmosClient = new CosmosClient(
    "https://mycosmosdb.documents.azure.com:443/",
    credential);

// Service Bus
var serviceBusClient = new ServiceBusClient(
    "myservicebus.servicebus.windows.net",
    credential);

// Event Hubs
var eventHubClient = new EventHubProducerClient(
    "myeventhub.servicebus.windows.net",
    "myeventhub",
    credential);
```

**Specific Managed Identity Credential:**

```csharp
// System-assigned managed identity
var credential = new ManagedIdentityCredential();

// User-assigned managed identity (by client ID)
var credential = new ManagedIdentityCredential("client-id-guid");

// User-assigned managed identity (by resource ID)
var credential = new ManagedIdentityCredential(
    new ManagedIdentityCredentialOptions
    {
        ResourceIdentifier = new ResourceIdentifier("/subscriptions/.../myidentity")
    });
```

**DefaultAzureCredential Chain:**

```
1. EnvironmentCredential (environment variables)
2. WorkloadIdentityCredential (Kubernetes)
3. ManagedIdentityCredential (Azure resources)
4. SharedTokenCacheCredential (cached tokens)
5. VisualStudioCredential (Visual Studio)
6. VisualStudioCodeCredential (VS Code)
7. AzureCliCredential (Azure CLI)
8. AzurePowerShellCredential (PowerShell)
9. AzureDeveloperCliCredential (azd)
10. InteractiveBrowserCredential (browser login)
```

### 2.5 Granting Access to Resources

**Key Vault Access:**

```bash
# Using RBAC (recommended)
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <managed-identity-principal-id> \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.KeyVault/vaults/<vault>

# Using Access Policy
az keyvault set-policy \
  --name mykeyvault \
  --object-id <managed-identity-principal-id> \
  --secret-permissions get list
```

**Storage Account Access:**

```bash
# Blob data access
az role assignment create \
  --role "Storage Blob Data Contributor" \
  --assignee <managed-identity-principal-id> \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>

# Queue data access
az role assignment create \
  --role "Storage Queue Data Contributor" \
  --assignee <managed-identity-principal-id> \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>
```

**SQL Database Access:**

```bash
# Grant access in SQL
# Connect to database and run:
CREATE USER [managed-identity-name] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [managed-identity-name];
ALTER ROLE db_datawriter ADD MEMBER [managed-identity-name];
```

```csharp
// Connect with managed identity
var connectionString = "Server=myserver.database.windows.net;Database=mydb;Authentication=Active Directory Managed Identity;";
using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();
```

**Service Bus Access:**

```bash
az role assignment create \
  --role "Azure Service Bus Data Sender" \
  --assignee <managed-identity-principal-id> \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ServiceBus/namespaces/<namespace>
```

### 2.6 Managed Identity Code Examples

**Example 1: Access Key Vault from App Service**

```csharp
public class SecretsService
{
    private readonly SecretClient _secretClient;

    public SecretsService(IConfiguration configuration)
    {
        var vaultUri = new Uri(configuration["KeyVaultUri"]);
        _secretClient = new SecretClient(vaultUri, new DefaultAzureCredential());
    }

    public async Task<string> GetSecretAsync(string secretName)
    {
        KeyVaultSecret secret = await _secretClient.GetSecretAsync(secretName);
        return secret.Value;
    }
}

// Startup.cs / Program.cs
builder.Services.AddSingleton<SecretsService>();
```

**Example 2: Access Blob Storage from Azure Function**

```csharp
public class BlobFunction
{
    private readonly BlobServiceClient _blobClient;

    public BlobFunction()
    {
        _blobClient = new BlobServiceClient(
            new Uri("https://mystorage.blob.core.windows.net/"),
            new DefaultAzureCredential());
    }

    [FunctionName("ProcessBlob")]
    public async Task Run([TimerTrigger("0 */5 * * * *")] TimerInfo timer)
    {
        var containerClient = _blobClient.GetBlobContainerClient("mycontainer");
        await foreach (var blob in containerClient.GetBlobsAsync())
        {
            Console.WriteLine($"Blob: {blob.Name}");
        }
    }
}
```

**Example 3: Access Multiple Services**

```csharp
public class MultiServiceClient
{
    private readonly SecretClient _secretClient;
    private readonly BlobServiceClient _blobClient;
    private readonly CosmosClient _cosmosClient;

    public MultiServiceClient(IConfiguration config)
    {
        var credential = new DefaultAzureCredential();

        _secretClient = new SecretClient(
            new Uri(config["KeyVaultUri"]),
            credential);

        _blobClient = new BlobServiceClient(
            new Uri(config["StorageUri"]),
            credential);

        _cosmosClient = new CosmosClient(
            config["CosmosDbEndpoint"],
            credential);
    }

    public async Task<string> GetSecretAsync(string name) =>
        (await _secretClient.GetSecretAsync(name)).Value.Value;

    public async Task UploadBlobAsync(string container, string name, Stream content)
    {
        var containerClient = _blobClient.GetBlobContainerClient(container);
        await containerClient.UploadBlobAsync(name, content);
    }

    public async Task<T> ReadCosmosItemAsync<T>(string database, string container, string id, string partitionKey)
    {
        var cosmosContainer = _cosmosClient.GetContainer(database, container);
        var response = await cosmosContainer.ReadItemAsync<T>(id, new PartitionKey(partitionKey));
        return response.Resource;
    }
}
```

### 2.7 Virtual Machine Managed Identity

**Enable Managed Identity on VM:**

```bash
# Enable system-assigned identity
az vm identity assign \
  --name myvm \
  --resource-group myResourceGroup

# Get principal ID
az vm identity show \
  --name myvm \
  --resource-group myResourceGroup \
  --query principalId
```

**Access Token from Within VM:**

```python
# Python example - get token from IMDS
import requests

def get_access_token(resource):
    url = "http://169.254.169.254/metadata/identity/oauth2/token"
    params = {
        "api-version": "2018-02-01",
        "resource": resource
    }
    headers = {"Metadata": "true"}
    
    response = requests.get(url, params=params, headers=headers)
    return response.json()["access_token"]

# Get token for Key Vault
token = get_access_token("https://vault.azure.net")

# Get token for Storage
token = get_access_token("https://storage.azure.com")
```

```csharp
// C# example - using Azure SDK
var credential = new ManagedIdentityCredential();
var token = await credential.GetTokenAsync(
    new TokenRequestContext(new[] { "https://vault.azure.net/.default" }));
Console.WriteLine($"Token: {token.Token}");
```

---

## MODULE 3: IMPLEMENT AZURE APP CONFIGURATION

### 3.1 Introduction to Azure App Configuration

**What is Azure App Configuration?**
- Centralized service for managing application settings
- Separate configuration from code
- Feature flag management
- Dynamic configuration updates
- Hierarchical key-value store

**Key Benefits:**
- Centralized configuration management
- Dynamic configuration without redeployment
- Feature flag support
- Environment-specific settings
- Integration with Key Vault for secrets
- Change history and versioning
- Native integration with Azure services

**Use Cases:**
- Microservices configuration
- Feature flag management
- A/B testing
- Environment-specific settings
- Dynamic updates without restart
- Centralized secrets management (via Key Vault references)

### 3.2 App Configuration vs Other Services

**Comparison:**

| Feature | App Configuration | App Settings | Key Vault |
|---------|------------------|--------------|-----------|
| Purpose | Config management | App-specific settings | Secret storage |
| Feature Flags | ✅ | ❌ | ❌ |
| Dynamic Updates | ✅ | Restart required | ✅ |
| Secrets | Via Key Vault ref | ❌ | ✅ |
| Versioning | ✅ | Limited | ✅ |
| Labels | ✅ | ❌ | ❌ |
| Cross-app | ✅ | ❌ | ✅ |

**When to Use:**
- **App Configuration**: Centralized config, feature flags, dynamic updates
- **App Settings**: Simple, app-specific settings
- **Key Vault**: Sensitive secrets, certificates, keys

### 3.3 Creating App Configuration Store

**Create Store - Azure CLI:**

```bash
# Create App Configuration store
az appconfig create \
  --name myappconfig \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard

# Create with system-assigned identity
az appconfig create \
  --name myappconfig \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard \
  --assign-identity
```

**App Configuration Tiers:**

| Feature | Free | Standard |
|---------|------|----------|
| Storage | 10 MB | 1 GB |
| Requests/day | 1,000 | Unlimited |
| SLA | None | 99.9% |
| Private endpoints | ❌ | ✅ |
| Managed identity | ❌ | ✅ |
| Encryption | Microsoft keys | Customer keys |

### 3.4 Working with Key-Value Pairs

**Key Naming Conventions:**

```
# Hierarchical keys (recommended)
AppName:Service:Setting
MyApp:Database:ConnectionString
MyApp:Cache:Enabled

# With labels for environments
Key: MyApp:Database:ConnectionString
Labels: Development, Staging, Production
```

**Key Operations - Azure CLI:**

```bash
# Set key-value
az appconfig kv set \
  --name myappconfig \
  --key "MyApp:Settings:Message" \
  --value "Hello World"

# Set with label
az appconfig kv set \
  --name myappconfig \
  --key "MyApp:Settings:Message" \
  --value "Hello Dev" \
  --label Development

az appconfig kv set \
  --name myappconfig \
  --key "MyApp:Settings:Message" \
  --value "Hello Prod" \
  --label Production

# Get key-value
az appconfig kv show \
  --name myappconfig \
  --key "MyApp:Settings:Message"

# Get with specific label
az appconfig kv show \
  --name myappconfig \
  --key "MyApp:Settings:Message" \
  --label Development

# List keys
az appconfig kv list \
  --name myappconfig

# List with filter
az appconfig kv list \
  --name myappconfig \
  --key "MyApp:*"

# Delete key
az appconfig kv delete \
  --name myappconfig \
  --key "MyApp:Settings:Message"

# Lock key (prevent modifications)
az appconfig kv lock \
  --name myappconfig \
  --key "MyApp:Settings:Message"

# Unlock key
az appconfig kv unlock \
  --name myappconfig \
  --key "MyApp:Settings:Message"
```

**Content Types:**

```bash
# JSON content
az appconfig kv set \
  --name myappconfig \
  --key "MyApp:Settings:Complex" \
  --value '{"timeout": 30, "retries": 3}' \
  --content-type "application/json"

# Key Vault reference
az appconfig kv set-keyvault \
  --name myappconfig \
  --key "MyApp:Secrets:DbPassword" \
  --secret-identifier "https://mykeyvault.vault.azure.net/secrets/DbPassword"
```

### 3.5 Feature Flags

**What are Feature Flags?**
- Toggle features on/off without code deployment
- Gradual rollouts
- A/B testing
- Kill switches for features
- Environment-specific features

**Feature Flag Properties:**
- **Key**: Feature identifier (prefixed with `.appconfig.featureflag/`)
- **Enabled**: Boolean state
- **Label**: Environment/variant
- **Filters**: Conditional activation

**Feature Flag Operations - Azure CLI:**

```bash
# Create feature flag
az appconfig feature set \
  --name myappconfig \
  --feature Beta \
  --label Production

# Enable feature
az appconfig feature enable \
  --name myappconfig \
  --feature Beta \
  --label Production

# Disable feature
az appconfig feature disable \
  --name myappconfig \
  --feature Beta \
  --label Production

# List features
az appconfig feature list \
  --name myappconfig

# Show feature
az appconfig feature show \
  --name myappconfig \
  --feature Beta

# Delete feature
az appconfig feature delete \
  --name myappconfig \
  --feature Beta
```

**Feature Filters:**

| Filter | Description |
|--------|-------------|
| Percentage | Enabled for percentage of users |
| TimeWindow | Enabled during time window |
| Targeting | Enabled for specific users/groups |
| Custom | Custom filter logic |

```bash
# Add percentage filter (50% rollout)
az appconfig feature filter add \
  --name myappconfig \
  --feature Beta \
  --filter-name "Microsoft.Percentage" \
  --filter-parameters Value=50
```

### 3.6 App Configuration SDK for .NET

**Installation:**

```bash
dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
dotnet add package Microsoft.FeatureManagement.AspNetCore
```

**Basic Configuration:**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add App Configuration
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(builder.Configuration["ConnectionStrings:AppConfig"])
           .Select(KeyFilter.Any)
           .Select(KeyFilter.Any, "Production");  // Load labeled values
});

var app = builder.Build();
```

**With Managed Identity:**

```csharp
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(new Uri("https://myappconfig.azconfig.io"), new DefaultAzureCredential())
           .Select(KeyFilter.Any, LabelFilter.Null)
           .Select(KeyFilter.Any, builder.Environment.EnvironmentName);
});
```

**With Key Vault Integration:**

```csharp
builder.Configuration.AddAzureAppConfiguration(options =>
{
    var credential = new DefaultAzureCredential();
    
    options.Connect(new Uri("https://myappconfig.azconfig.io"), credential)
           .Select(KeyFilter.Any)
           .ConfigureKeyVault(kv =>
           {
               kv.SetCredential(credential);
           });
});
```

**Dynamic Configuration Refresh:**

```csharp
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .Select(KeyFilter.Any)
           .ConfigureRefresh(refresh =>
           {
               refresh.Register("MyApp:Settings:Sentinel", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromSeconds(30));
           });
});

// Add middleware for refresh
builder.Services.AddAzureAppConfiguration();

var app = builder.Build();
app.UseAzureAppConfiguration();  // Enable dynamic refresh
```

**Feature Flags in ASP.NET Core:**

```csharp
// Program.cs
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .UseFeatureFlags(featureOptions =>
           {
               featureOptions.CacheExpirationInterval = TimeSpan.FromMinutes(5);
               featureOptions.Label = builder.Environment.EnvironmentName;
           });
});

builder.Services.AddFeatureManagement();

// Controller
public class HomeController : Controller
{
    private readonly IFeatureManager _featureManager;

    public HomeController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }

    public async Task<IActionResult> Index()
    {
        if (await _featureManager.IsEnabledAsync("Beta"))
        {
            return View("BetaIndex");
        }
        return View();
    }
}

// Using attribute
[FeatureGate("Beta")]
public IActionResult BetaFeature()
{
    return View();
}
```

**Feature Filter Registration:**

```csharp
// Custom feature filter
public class BrowserFilter : IFeatureFilter
{
    private readonly IHttpContextAccessor _httpContextAccessor;

    public BrowserFilter(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    public Task<bool> EvaluateAsync(FeatureFilterEvaluationContext context)
    {
        var userAgent = _httpContextAccessor.HttpContext?.Request.Headers["User-Agent"].ToString();
        var allowedBrowsers = context.Parameters.Get<string[]>("AllowedBrowsers");
        
        return Task.FromResult(allowedBrowsers?.Any(b => userAgent?.Contains(b) ?? false) ?? false);
    }
}

// Register filter
builder.Services.AddFeatureManagement()
    .AddFeatureFilter<BrowserFilter>();
```

### 3.7 App Configuration Complete Example

**appsettings.json:**

```json
{
  "ConnectionStrings": {
    "AppConfig": "Endpoint=https://myappconfig.azconfig.io;Id=xxxx;Secret=xxxx"
  },
  "AppConfigEndpoint": "https://myappconfig.azconfig.io"
}
```

**Program.cs:**

```csharp
var builder = WebApplication.CreateBuilder(args);

// Load from App Configuration with managed identity
var appConfigEndpoint = builder.Configuration["AppConfigEndpoint"];
if (!string.IsNullOrEmpty(appConfigEndpoint))
{
    var credential = new DefaultAzureCredential();
    
    builder.Configuration.AddAzureAppConfiguration(options =>
    {
        options.Connect(new Uri(appConfigEndpoint), credential)
               // Load all keys without label
               .Select(KeyFilter.Any, LabelFilter.Null)
               // Load environment-specific keys
               .Select(KeyFilter.Any, builder.Environment.EnvironmentName)
               // Configure Key Vault references
               .ConfigureKeyVault(kv => kv.SetCredential(credential))
               // Configure refresh
               .ConfigureRefresh(refresh =>
               {
                   refresh.Register("MyApp:Sentinel", refreshAll: true)
                          .SetCacheExpiration(TimeSpan.FromMinutes(1));
               })
               // Configure feature flags
               .UseFeatureFlags(features =>
               {
                   features.CacheExpirationInterval = TimeSpan.FromMinutes(5);
                   features.Label = builder.Environment.EnvironmentName;
               });
    });
}

// Add services
builder.Services.AddAzureAppConfiguration();
builder.Services.AddFeatureManagement();
builder.Services.AddControllers();

var app = builder.Build();

// Use App Configuration middleware
app.UseAzureAppConfiguration();
app.MapControllers();

app.Run();
```

**Configuration Class:**

```csharp
public class MyAppSettings
{
    public string Message { get; set; }
    public int Timeout { get; set; }
    public DatabaseSettings Database { get; set; }
}

public class DatabaseSettings
{
    public string ConnectionString { get; set; }
    public int MaxRetries { get; set; }
}

// Register in DI
builder.Services.Configure<MyAppSettings>(
    builder.Configuration.GetSection("MyApp:Settings"));

// Use in controller
public class HomeController : Controller
{
    private readonly MyAppSettings _settings;
    private readonly IFeatureManager _featureManager;

    public HomeController(
        IOptionsSnapshot<MyAppSettings> settings,
        IFeatureManager featureManager)
    {
        _settings = settings.Value;
        _featureManager = featureManager;
    }

    public async Task<IActionResult> Index()
    {
        ViewBag.Message = _settings.Message;
        ViewBag.BetaEnabled = await _featureManager.IsEnabledAsync("Beta");
        return View();
    }
}
```

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**1. Azure Key Vault:**
- Service tiers (Standard vs Premium)
- Authentication and authorization (RBAC vs Access Policies)
- Secrets, Keys, Certificates operations
- SDK usage
- Key Vault references

**2. Managed Identities:**
- System-assigned vs User-assigned
- DefaultAzureCredential
- Granting access to resources
- IMDS token acquisition

**3. App Configuration:**
- Key-value pairs with labels
- Feature flags and filters
- Dynamic refresh
- Key Vault integration

### Sample Exam Questions

#### Azure Key Vault

**Q1:** Which Key Vault tier supports Hardware Security Module (HSM) protected keys?
- A) Basic
- B) Standard
- C) Premium
- D) Enterprise

**Answer: C) Premium**
Explanation: Only the Premium tier supports HSM-protected keys with FIPS 140-2 Level 3 validation.

---

**Q2:** What is the recommended access control model for Azure Key Vault?
- A) Access Policies
- B) Azure RBAC
- C) Shared Access Signatures
- D) Anonymous access

**Answer: B) Azure RBAC**
Explanation: Azure RBAC is the recommended model as it provides granular permissions and inheritance from management groups/subscriptions.

---

**Q3:** Which RBAC role allows reading secrets but not modifying them?
- A) Key Vault Administrator
- B) Key Vault Secrets Officer
- C) Key Vault Secrets User
- D) Key Vault Reader

**Answer: C) Key Vault Secrets User**
Explanation: Key Vault Secrets User can read secret contents. Key Vault Reader can only list secrets without reading values.

---

**Q4:** What happens when you delete a secret with soft-delete enabled?
- A) Secret is permanently deleted
- B) Secret is moved to deleted state and can be recovered
- C) Secret is immediately purged
- D) Delete operation fails

**Answer: B) Secret is moved to deleted state and can be recovered**
Explanation: With soft-delete, secrets are recoverable during the retention period (7-90 days).

---

**Q5:** How do you reference a Key Vault secret in App Service app settings?
- A) `$(KeyVault:SecretName)`
- B) `@Microsoft.KeyVault(SecretUri=https://vault.vault.azure.net/secrets/secret/)`
- C) `keyvault://vault/secret`
- D) `{keyvault:secret}`

**Answer: B) `@Microsoft.KeyVault(SecretUri=https://vault.vault.azure.net/secrets/secret/)`**
Explanation: Key Vault references in App Service use the @Microsoft.KeyVault syntax with the secret URI.

---

#### Managed Identities

**Q6:** What is the main difference between system-assigned and user-assigned managed identities?
- A) System-assigned is more secure
- B) User-assigned can be shared across multiple resources
- C) System-assigned supports more Azure services
- D) User-assigned is deprecated

**Answer: B) User-assigned can be shared across multiple resources**
Explanation: User-assigned identities are standalone resources that can be assigned to multiple Azure resources, while system-assigned identities are tied to a single resource.

---

**Q7:** What is the IMDS endpoint used by managed identities to obtain tokens?
- A) `https://login.microsoftonline.com/`
- B) `http://169.254.169.254/metadata/identity/oauth2/token`
- C) `https://management.azure.com/token`
- D) `http://localhost/identity/token`

**Answer: B) `http://169.254.169.254/metadata/identity/oauth2/token`**
Explanation: The Instance Metadata Service (IMDS) endpoint is a well-known, non-routable IP address that Azure resources use to obtain tokens.

---

**Q8:** Which credential class should you use in the Azure SDK to automatically work with managed identities in Azure and developer credentials locally?
- A) ManagedIdentityCredential
- B) DefaultAzureCredential
- C) ClientSecretCredential
- D) InteractiveBrowserCredential

**Answer: B) DefaultAzureCredential**
Explanation: DefaultAzureCredential tries multiple credential types in order, including managed identity in Azure and developer credentials locally.

---

**Q9:** How do you enable system-assigned managed identity on an Azure App Service using Azure CLI?
- A) `az webapp identity create`
- B) `az webapp identity assign`
- C) `az webapp identity enable`
- D) `az webapp managed-identity create`

**Answer: B) `az webapp identity assign`**
Explanation: The `az webapp identity assign` command enables system-assigned managed identity on an App Service.

---

**Q10:** Which SQL statement grants a managed identity access to an Azure SQL database?
- A) `GRANT ACCESS TO [identity-name]`
- B) `CREATE USER [identity-name] FROM EXTERNAL PROVIDER`
- C) `ADD USER [identity-name] FROM AZURE AD`
- D) `ENABLE MANAGED IDENTITY [identity-name]`

**Answer: B) `CREATE USER [identity-name] FROM EXTERNAL PROVIDER`**
Explanation: The CREATE USER FROM EXTERNAL PROVIDER statement creates a database user from an Azure AD identity.

---

#### Azure App Configuration

**Q11:** What is the purpose of labels in Azure App Configuration?
- A) To categorize keys alphabetically
- B) To version key-value pairs for different environments
- C) To encrypt sensitive values
- D) To set expiration dates

**Answer: B) To version key-value pairs for different environments**
Explanation: Labels allow you to have the same key with different values for different environments (Development, Production, etc.).

---

**Q12:** How do you configure dynamic configuration refresh in App Configuration?
- A) Call RefreshAsync() manually
- B) Use ConfigureRefresh() with a sentinel key
- C) Enable auto-refresh in Azure portal
- D) Set cache expiration to zero

**Answer: B) Use ConfigureRefresh() with a sentinel key**
Explanation: ConfigureRefresh registers a sentinel key that triggers refresh of all configurations when changed.

---

**Q13:** Which feature in App Configuration enables toggling functionality without code deployment?
- A) Key-Value Pairs
- B) Labels
- C) Feature Flags
- D) Snapshots

**Answer: C) Feature Flags**
Explanation: Feature flags allow you to enable/disable features dynamically without redeploying code.

---

**Q14:** How do you store a Key Vault secret reference in App Configuration?
- A) `az appconfig kv set --value @keyvault:secret`
- B) `az appconfig kv set-keyvault --secret-identifier <uri>`
- C) `az appconfig secret set --vault <name>`
- D) `az appconfig kv set --content-type keyvault`

**Answer: B) `az appconfig kv set-keyvault --secret-identifier <uri>`**
Explanation: The `az appconfig kv set-keyvault` command creates a Key Vault reference in App Configuration.

---

**Q15:** What is the maximum storage size for the Standard tier of App Configuration?
- A) 10 MB
- B) 100 MB
- C) 1 GB
- D) 10 GB

**Answer: C) 1 GB**
Explanation: The Standard tier supports up to 1 GB of storage, while the Free tier is limited to 10 MB.

---

#### Advanced Scenarios

**Q16:** You need to grant an App Service access to both Key Vault secrets and Blob Storage. What is the best approach?
- A) Use connection strings for both services
- B) Create separate managed identities for each service
- C) Use a single managed identity with RBAC roles for both services
- D) Store both credentials in Key Vault

**Answer: C) Use a single managed identity with RBAC roles for both services**
Explanation: A single managed identity can be granted different RBAC roles for multiple Azure services, following the principle of least privilege.

---

**Q17:** What middleware must be added to enable dynamic App Configuration refresh in ASP.NET Core?
- A) `app.UseConfiguration()`
- B) `app.UseAzureAppConfiguration()`
- C) `app.UseRefresh()`
- D) `app.UseDynamicConfig()`

**Answer: B) `app.UseAzureAppConfiguration()`**
Explanation: The UseAzureAppConfiguration() middleware enables dynamic refresh of configuration values.

---

**Q18:** Which feature filter would you use to enable a feature for 10% of users?
- A) TimeWindowFilter
- B) TargetingFilter
- C) PercentageFilter
- D) RandomFilter

**Answer: C) PercentageFilter**
Explanation: The PercentageFilter (Microsoft.Percentage) enables features for a specified percentage of requests.

---

**Q19:** When a managed identity is deleted, what happens to the role assignments?
- A) They are automatically deleted
- B) They remain and must be manually deleted
- C) They are transferred to another identity
- D) They become orphaned and cause errors

**Answer: A) They are automatically deleted**
Explanation: When a managed identity is deleted, Azure automatically removes the associated role assignments.

---

**Q20:** What is the correct order of precedence when using DefaultAzureCredential in Azure?
- A) ManagedIdentityCredential, EnvironmentCredential, AzureCliCredential
- B) EnvironmentCredential, ManagedIdentityCredential, AzureCliCredential
- C) AzureCliCredential, ManagedIdentityCredential, EnvironmentCredential
- D) InteractiveBrowserCredential, ManagedIdentityCredential, EnvironmentCredential

**Answer: B) EnvironmentCredential, ManagedIdentityCredential, AzureCliCredential**
Explanation: DefaultAzureCredential tries EnvironmentCredential first, then ManagedIdentityCredential (in Azure), followed by various developer credentials.

---

## Key CLI Commands Reference

### Key Vault
```bash
# Create Key Vault
az keyvault create --name myvault --resource-group myRG --location eastus

# Set secret
az keyvault secret set --vault-name myvault --name MySecret --value "secretvalue"

# Get secret
az keyvault secret show --vault-name myvault --name MySecret

# Assign RBAC role
az role assignment create --role "Key Vault Secrets User" --assignee <id> --scope <vault-resource-id>

# Set access policy
az keyvault set-policy --name myvault --object-id <id> --secret-permissions get list
```

### Managed Identity
```bash
# Enable system-assigned identity
az webapp identity assign --name myapp --resource-group myRG

# Create user-assigned identity
az identity create --name myidentity --resource-group myRG

# Assign user identity to resource
az webapp identity assign --name myapp --resource-group myRG --identities <identity-resource-id>

# Get principal ID
az webapp identity show --name myapp --resource-group myRG --query principalId
```

### App Configuration
```bash
# Create App Configuration store
az appconfig create --name myappconfig --resource-group myRG --location eastus

# Set key-value
az appconfig kv set --name myappconfig --key "MyApp:Setting" --value "value"

# Set with label
az appconfig kv set --name myappconfig --key "MyApp:Setting" --value "value" --label Production

# Set Key Vault reference
az appconfig kv set-keyvault --name myappconfig --key "MyApp:Secret" --secret-identifier <uri>

# Enable feature flag
az appconfig feature enable --name myappconfig --feature Beta
```

---

## Study Tips for AZ-204 Exam

1. **Understand Key Vault tiers** - Standard vs Premium capabilities
2. **Know RBAC roles** - Specific roles for secrets, keys, certificates
3. **Master managed identities** - System vs user-assigned differences
4. **Understand DefaultAzureCredential** - Order of credential providers
5. **Learn App Configuration patterns** - Labels, feature flags, refresh
6. **Practice Key Vault references** - Syntax in App Service/Functions
7. **Know token acquisition** - IMDS endpoint and SDK usage
8. **Understand feature flags** - Filters and evaluation
9. **Practice CLI commands** - Common operations for all three services
10. **Hands-on practice** - Create resources and test access patterns

---

## Important URLs and References

- Azure Key Vault: https://learn.microsoft.com/en-us/azure/key-vault/
- Managed Identities: https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/
- App Configuration: https://learn.microsoft.com/en-us/azure/azure-app-configuration/
- DefaultAzureCredential: https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredential

---

## Quick Reference Summary

### Key Vault Tiers
- **Standard**: Software-protected keys, secrets, certificates
- **Premium**: HSM-protected keys, managed HSM

### Key Vault Access
- **RBAC (Recommended)**: Granular roles, inheritance
- **Access Policies (Legacy)**: Vault-level permissions

### Managed Identity Types
- **System-assigned**: Tied to resource, automatic lifecycle
- **User-assigned**: Standalone, shareable, manual management

### DefaultAzureCredential Order
1. EnvironmentCredential
2. WorkloadIdentityCredential
3. ManagedIdentityCredential
4. SharedTokenCacheCredential
5. VisualStudioCredential
6. VS Code / CLI / PowerShell credentials

### App Configuration Features
- **Key-Value Pairs**: Hierarchical keys with labels
- **Feature Flags**: Toggle features dynamically
- **Dynamic Refresh**: Update config without restart
- **Key Vault References**: Secure secret storage

