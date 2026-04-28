# AZ-204: Azure App Service Web Apps - Complete Study Notes

## Learning Path Overview
This learning path covers implementing Azure App Service web apps with 4 modules:
1. Explore Azure App Service
2. Configure web app settings
3. Scale apps in Azure App Service
4. Explore Azure App Service deployment slots

---

## MODULE 1: EXPLORE AZURE APP SERVICE

### 1.1 Introduction to Azure App Service

**What is Azure App Service?**
- Fully managed Platform-as-a-Service (PaaS) for building, deploying, and scaling web apps
- HTTP-based service for hosting web applications, REST APIs, and mobile backends
- Supports multiple programming languages: .NET, .NET Core, Java, Node.js, PHP, Python, Ruby
- Runs on both Windows and Linux platforms

**Key Features:**
- Built-in auto-scaling and load balancing
- High availability with 99.95% SLA
- Continuous deployment and integration support
- Built-in authentication and authorization
- DevOps integration (Azure DevOps, GitHub, BitBucket)
- Security and compliance certifications
- Application templates and marketplace

**App Service Types:**
1. **Web Apps** - Host web applications
2. **Web Apps for Containers** - Deploy containerized applications
3. **API Apps** - Build and host RESTful APIs
4. **Mobile Apps** - Backend for mobile applications

### 1.2 Azure App Service Plans

**What is an App Service Plan?**
- Defines the compute resources (CPU, memory, storage) for your web apps
- Acts as a container for one or more apps
- All apps in the same plan share the same compute resources
- Pricing is based on the plan, not individual apps

**Pricing Tiers Categories:**

#### 1. **Shared Compute Tiers**

**Free (F1) Tier:**
- CPU: 60 minutes/day quota on shared infrastructure
- Storage: 1 GB
- RAM: 1 GB shared
- Max apps: 10
- Custom domains: No
- SSL: No custom SSL
- Auto-scale: No
- Deployment slots: No
- Always On: No
- Use case: Development, testing, learning
- Cost: Free

**Shared (D1) Tier:**
- CPU: 240 minutes/day quota on shared infrastructure
- Storage: 1 GB
- RAM: 1 GB shared
- Custom domains: Yes
- SSL: SNI SSL supported
- Auto-scale: No
- Deployment slots: No
- Always On: No
- Use case: Low-traffic sites, personal projects
- Cost: Very low (pay per app)

#### 2. **Dedicated Compute Tiers**

**Basic (B1, B2, B3) Tier:**
- CPU: Dedicated (1, 2, 4 cores)
- RAM: 1.75 GB, 3.5 GB, 7 GB
- Storage: 10 GB
- Custom domains: Yes
- SSL: Yes (custom certificates)
- Auto-scale: No
- Deployment slots: No
- Always On: Available
- Use case: Small production workloads, dev/test
- Approximate cost: Starting ~$0.013/hour

**Standard (S1, S2, S3) Tier:**
- CPU: Dedicated (1, 2, 4 cores)
- RAM: 1.75 GB, 3.5 GB, 7 GB
- Storage: 50 GB
- Custom domains: Yes (unlimited)
- SSL: Yes (SNI + IP-based)
- Auto-scale: Yes (up to 10 instances)
- Deployment slots: 5 slots
- Always On: Yes
- Traffic Manager integration: Yes
- Backups: Daily (manual + scheduled)
- VNet integration: Hybrid connections
- Use case: Production apps with moderate traffic
- Approximate cost: Starting ~$0.10/hour

**Premium (P1v2, P2v2, P3v2, P1v3, P2v3, P3v3, P1v4, P2v4, P3v4) Tier:**
- CPU: Faster dedicated cores (2, 4, 8 cores)
- RAM: Higher memory allocation (8-32 GB depending on version)
- Storage: 250 GB
- Custom domains: Yes
- SSL: Yes
- Auto-scale: Yes (up to 20-30 instances)
- Deployment slots: 20 slots
- Always On: Yes
- VNet integration: Full VNet integration, Private Endpoints
- Daily backups: Yes (50 per app)
- Use case: High-traffic production applications, enterprise workloads
- **PremiumV3**: Best cost-to-performance ratio
- **PremiumV4**: Latest generation with enhanced performance
- Approximate cost: Varies significantly by version

#### 3. **Isolated Tier**

**Isolated (I1v2, I2v2, I3v2) Tier:**
- Runs in dedicated Azure Virtual Networks (App Service Environment v3)
- Complete network isolation
- Dedicated infrastructure (single-tenant)
- CPU: Dedicated high-performance (2, 4, 8 cores)
- RAM: 8-16 GB
- Storage: 1 TB
- Auto-scale: Yes (up to 100+ instances)
- Deployment slots: 20 slots
- Use case: Mission-critical applications requiring maximum security and isolation
- No Stamp Fee (eliminated in v3)
- Highest security and compliance requirements
- Approximate cost: Highest pricing tier

**Key Pricing Considerations:**
- Only Free tier has no charges
- Shared tier charges per app based on CPU quota
- Dedicated tiers (Basic, Standard, Premium) charge per VM instance
- Isolated tier charges per worker instance
- Stopped apps still incur charges (except Free tier)
- Windows plans are more expensive than Linux (due to OS licensing)

### 1.3 How Apps Run in App Service Plans

**Resource Sharing:**
- All apps in the same plan run on the same VM instances
- Apps share CPU, memory, disk space
- Multiple deployment slots also share the same VM instances

**Scaling Behavior:**
- **Scale up/down (Vertical)**: Change the plan tier
- **Scale out/in (Horizontal)**: Increase/decrease instance count
- All apps in the plan scale together
- Cannot scale individual apps separately (must move to different plan)

**Isolation:**
- To isolate compute resources for an app, move it to a separate App Service Plan
- Apps in different plans can still be in the same resource group and region

### 1.4 Deploying to App Service

**Deployment Methods:**

1. **Automated Deployment (CI/CD)**:
   - Azure DevOps Services
   - GitHub Actions
   - Bitbucket
   - Continuous deployment from container registry

2. **Manual Deployment**:
   - Git (local repository push)
   - Azure CLI (`az webapp up`)
   - Visual Studio / Visual Studio Code
   - ZIP deploy
   - FTP/FTPS
   - WAR deploy (for Java apps)

**Deployment Considerations:**
- **Deployment Center**: Centralized place to configure deployment
- **Kudu**: Built-in deployment engine
- **SCM endpoint**: Separate endpoint for source control (e.g., `<app-name>.scm.azurewebsites.net`)
- **Deployment credentials**: User-level or app-level credentials

**Key Deployment Commands:**

```bash
# Create and deploy web app
az webapp up --name <app-name> --resource-group <resource-group> --html

# Deploy from local Git
git remote add azure <deployment-git-url>
git push azure main:master

# Deploy ZIP file
az webapp deploy --resource-group <rg> --name <app-name> --src-path <zip-file> --type zip

# Deploy to slot
az webapp deploy --resource-group <rg> --name <app-name> --src-path <zip-file> --slot <slot-name>
```

### 1.5 Authentication and Authorization (Easy Auth)

**Overview:**
- Built-in authentication and authorization module
- Also called "Easy Auth"
- Runs as middleware on the same VM as application code
- Handles authentication before code execution
- No SDKs, specific languages, or code changes required

**Supported Identity Providers:**
1. **Microsoft Entra ID (Azure AD)** - Enterprise authentication
2. **Microsoft Account** - Consumer Microsoft accounts
3. **Facebook** - Social authentication
4. **Google** - Social authentication
5. **X (Twitter)** - Social authentication
6. **Apple ID** - Apple users
7. **OpenID Connect** - Any OIDC-compliant provider
8. **GitHub** - Developer authentication

**How It Works:**

**Authentication Flow - Without Provider SDK:**
1. User clicks sign-in button
2. App Service redirects to provider's sign-in page (`/.auth/login/<provider>`)
3. User authenticates with provider
4. Provider redirects back to App Service with token
5. App Service validates token and creates session
6. App Service redirects to application with authenticated session

**Authentication Flow - With Provider SDK:**
1. Client app uses provider SDK for authentication
2. Client obtains authentication token
3. Client POSTs token to App Service for validation (`/.auth/login/<provider>`)
4. App Service validates token and creates session
5. App Service returns its own authentication token
6. Client uses token for authenticated requests

**Token Store:**
- App Service automatically manages tokens
- Tokens are cached in storage associated with user session
- Accessible without additional coding

**Session Management:**
- **Web browsers**: Cookie-based session (persistent authentication)
- **Mobile clients**: JSON Web Token (JWT) in `X-ZUMO-AUTH` header
- **APIs**: Bearer token in `Authorization` header

**Authorization Behaviors:**

1. **Allow unauthenticated requests (Anonymous)**:
   - App handles authentication logic
   - Authentication information passed in HTTP headers
   - Provides flexibility for multiple sign-in providers
   - Requires custom authorization code

2. **Require authentication**:
   - Rejects unauthenticated traffic
   - Redirects to `/.auth/login/<provider>` for browser clients
   - Returns HTTP 401 Unauthorized for APIs
   - Can specify identity provider

**Important Configuration Settings:**

```bash
# Enable authentication in portal:
# App Service → Settings → Authentication → Add identity provider

# Configure Microsoft Entra ID
# Select Microsoft → Create new app registration or use existing
# Supported account types: Single tenant / Multi-tenant / External

# Configure slot-specific settings
# Mark settings as "Deployment slot setting" to prevent swap
```

**Authorization Claims:**
- User claims available in request headers (JSON-encoded)
- Headers include: `X-MS-CLIENT-PRINCIPAL`, `X-MS-CLIENT-PRINCIPAL-ID`, `X-MS-TOKEN-<provider>-ACCESS-TOKEN`
- Access claims programmatically via `/.auth/me` endpoint

**Key Security Features:**
- **Token validation**: Automatic validation of provider tokens
- **Token refresh**: Automatic refresh of OAuth tokens
- **Session management**: Secure session cookies
- **HTTPS enforcement**: Redirect HTTP to HTTPS
- **Encryption**: Tokens encrypted at rest

### 1.6 Networking Features

**App Service Networking Overview:**
- Apps run in multi-tenant environment (except Isolated tier)
- Default: Apps are internet-accessible
- Can control both inbound and outbound traffic

**Inbound Features (Controlling Access TO App):**

1. **App-assigned Address**:
   - Apps get a shared public IP by default
   - Custom domain names point to this IP
   - IP address shared among all apps in plan (except Isolated)

2. **Access Restrictions (IP Restrictions)**:
   - Define allowed/denied IP address ranges
   - Rules processed in priority order
   - Can restrict by:
     - IPv4/IPv6 addresses
     - CIDR ranges
     - Service Tags (Azure services)
     - VNet/Subnet

3. **Service Endpoints**:
   - Secure access from Azure VNets only
   - Traffic from VNet subnets to App Service
   - Available in Standard tier and above

4. **Private Endpoints**:
   - Deploy app into your VNet with private IP
   - App accessible only from within VNet
   - Requires Premium v2, Premium v3, or Isolated tier
   - Replaces public endpoint with private IP

**Outbound Features (Controlling Access FROM App):**

1. **Hybrid Connections**:
   - Connect to resources in on-premises or other networks
   - Uses Azure Relay service
   - Requires agent on target network
   - Only TCP connectivity (no UDP)
   - Each connection targets specific host:port combination
   - Available in Basic tier and above

2. **VNet Integration**:
   - App can make outbound calls into Azure VNet
   - Access resources in or connected to VNet
   - Does not give VNet access TO app (use private endpoint for that)
   - **Regional VNet Integration**: Same region VNet (Standard tier+)
   - **Gateway-required VNet Integration**: Different region VNet (Premium tier+)
   - Can access:
     - Resources in VNet
     - Resources in peered VNets
     - Service Endpoint secured services
     - On-premises resources via ExpressRoute/VPN

3. **App Service Environment (ASE)**:
   - Isolated tier only
   - Fully deployed into your VNet
   - Complete network isolation
   - Dedicated environment
   - Can have internal or external load balancer

**Default Outbound Addresses:**
- Apps have default outbound IP addresses
- These IPs are shared among apps in same deployment unit
- Can view outbound IPs in portal or CLI
- May change when scaling across different plan tiers

```bash
# View outbound IPs
az webapp show --resource-group <rg> --name <app-name> --query outboundIpAddresses --output tsv

# View all possible outbound IPs
az webapp show --resource-group <rg> --name <app-name> --query possibleOutboundIpAddresses --output tsv
```

**Important Networking Limits:**
- Free/Shared: No VNet integration or Hybrid Connections
- Basic: Hybrid Connections supported
- Standard+: VNet integration, Hybrid Connections
- Premium: All networking features
- Isolated: Full network isolation with ASE

---

## MODULE 2: CONFIGURE WEB APP SETTINGS

### 2.1 Application Settings and Configuration

**App Settings Overview:**
- Environment variables passed to application code
- Encrypted at rest
- Similar to web.config `<appSettings>` or appsettings.json
- Override app config files

**Configuring App Settings:**

**In Azure Portal:**
```
App Service → Settings → Environment variables → Application settings
```

**Via Azure CLI:**
```bash
# List app settings
az webapp config appsettings list --name <app-name> --resource-group <rg>

# Set app setting
az webapp config appsettings set --name <app-name> --resource-group <rg> \
  --settings KEY1=VALUE1 KEY2=VALUE2

# Delete app setting
az webapp config appsettings delete --name <app-name> --resource-group <rg> \
  --setting-names KEY1 KEY2
```

**Accessing App Settings in Code:**

**ASP.NET/ASP.NET Core:**
```csharp
// App settings override Web.config values
System.Configuration.ConfigurationManager.AppSettings["MySetting"]

// ASP.NET Core - via IConfiguration
_configuration["MySetting"]
```

**Node.js:**
```javascript
process.env.MySetting
```

**Python:**
```python
import os
os.environ['MySetting']
```

**Java:**
```java
System.getenv("MySetting")
```

**Deployment Slot Settings:**
- Mark settings as "Deployment slot setting" (sticky)
- Sticky settings DON'T swap with slots
- Non-sticky settings swap with the code
- Use for environment-specific values (connection strings, feature flags)

### 2.2 Connection Strings

**Connection String Configuration:**
- Configured similar to app settings
- Support for different types: SQL Database, MySQL, SQL Server, Custom, etc.
- Always encrypted at rest
- More secure than storing in code/config files

**Connection String Types:**
1. **SQL Database** / **SQL Azure**
2. **MySQL**
3. **SQL Server**
4. **PostgreSQL**
5. **Custom**

**Accessing Connection Strings:**

**ASP.NET:**
```csharp
ConfigurationManager.ConnectionStrings["MyDbConnection"].ConnectionString
```

**ASP.NET Core:**
```json
// appsettings.json structure
{
  "ConnectionStrings": {
    "MyDbConnection": "connection-string-value"
  }
}
```

```csharp
// Access via IConfiguration
_configuration.GetConnectionString("MyDbConnection")
```

**Environment Variable Format:**
- Windows: `SQLCONNSTR_<name>`, `MYSQLCONNSTR_<name>`, `SQLAZURECONNSTR_<name>`, `POSTGRESQLCONNSTR_<name>`, `CUSTOMCONNSTR_<name>`
- Linux: Provided without prefix

**Best Practices:**
- Use Azure Key Vault references for sensitive data
- Never commit connection strings to source control
- Use managed identities when possible
- Mark database-specific strings as slot settings to prevent production DB access from staging

### 2.3 General Settings (Configuration)

**Platform Settings:**

1. **Stack Settings**:
   - Runtime stack (e.g., .NET, Java, Node.js, PHP, Python)
   - Runtime version
   - Platform bitness (32-bit / 64-bit)
   - Web sockets protocol
   - Always On feature
   - ARR affinity (Application Request Routing)

2. **Always On**:
   - Keeps app loaded even when no traffic
   - Prevents idle timeout (20 minutes by default)
   - Required for continuous WebJobs
   - Required for triggered WebJobs with CRON
   - Available in Basic tier and above
   - Disabled in Free/Shared tiers

3. **ARR Affinity (Session Affinity)**:
   - Routes client to same instance for entire session
   - Uses cookies to maintain session state
   - Enable for stateful apps
   - Disable for stateless apps (better scaling)
   - **Recommendation**: Disable for stateless apps, use external session state (Redis Cache, SQL)

4. **HTTP Version**:
   - HTTP/1.1 (default)
   - HTTP/2 support available

5. **Web Sockets**:
   - Enable for real-time communication (SignalR)
   - Required for apps using WebSocket protocol

**Debugging:**
- **Remote Debugging**: Attach Visual Studio debugger
- Available for .NET, .NET Core, Node.js
- **Auto-off**: Automatically turns off after 48 hours

**Startup Command:**
- Custom startup command for Linux apps
- Specify container startup script
- Example: `python manage.py migrate && python manage.py runserver`

### 2.4 Path Mappings (Handler Mappings & Virtual Applications)

**Virtual Applications and Directories:**
- Map virtual paths to physical paths
- Root application: `/` maps to `site\wwwroot`
- Can create sub-applications under root

**Structure:**
- **Virtual Path**: URL path (e.g., `/api`)
- **Physical Path**: Actual folder (e.g., `site\wwwroot\api`)
- **Application**: Checkbox indicating if path is an application

**Handler Mappings (Windows only):**
- Configure custom script processors
- Example: Process PHP files with custom PHP version
- Define file extension and script processor path

### 2.5 Certificates and TLS/SSL

**Certificate Types:**

1. **Free App Service Managed Certificate**:
   - Automatically renewed
   - SNI SSL only
   - Limited to Standard tier and above
   - No wildcard certificates
   - Restrictions: No export, no IP-based SSL, no Isolated tier

2. **App Service Certificate** (Purchased):
   - Purchased from Azure
   - Stored in Azure Key Vault
   - Supports wildcard domains
   - Can export
   - Automatically renewed
   - Annual purchase and renewal required

3. **Import Certificate**:
   - Upload your own certificate
   - PFX format required
   - Password-protected
   - Minimum 2048-bit key
   - Must contain private key

**Certificate Requirements:**
- PFX format
- Password protected
- Contains private key
- Created for key exchange
- Minimum 2048-bit encryption
- All intermediate certificates included

**Binding Types:**

1. **SNI SSL (Server Name Indication)**:
   - Modern standard
   - Multiple SSL certificates on same IP
   - Supported by modern browsers
   - Free with App Service Managed Certificate
   - Recommended for most scenarios

2. **IP-based SSL**:
   - Traditional SSL binding
   - Dedicated IP address for app
   - Only one certificate per IP
   - Additional cost
   - Needed for older clients

**TLS/SSL Configuration:**
```bash
# Upload certificate
az webapp config ssl upload --certificate-file <path-to-pfx> \
  --certificate-password <password> --name <app-name> --resource-group <rg>

# Bind certificate to app
az webapp config ssl bind --certificate-thumbprint <thumbprint> \
  --ssl-type SNI --name <app-name> --resource-group <rg>

# Set minimum TLS version
az webapp config set --min-tls-version 1.2 --name <app-name> --resource-group <rg>
```

**Security Settings:**
- **HTTPS Only**: Redirect all HTTP to HTTPS
- **Minimum TLS Version**: Set to 1.2 or 1.3
- **Client Certificates**: Require client cert authentication (mutual TLS)

**Custom Domains:**
- Add custom domain before binding certificate
- Verify domain ownership via TXT or CNAME record
- Steps:
  1. Create CNAME record: `<subdomain> → <app-name>.azurewebsites.net`
  2. Create TXT record for verification: `asuid.<subdomain> → <verification-id>`
  3. Add custom domain in portal
  4. Bind SSL certificate

### 2.6 Feature Management

**Azure App Configuration Integration:**
- Centralized configuration management
- Feature flags for controlled rollouts
- Dynamic configuration without redeployment
- A/B testing capabilities

**Feature Flags:**
- Enable/disable features without code deployment
- Percentage-based rollouts
- Target specific users or groups
- Integration with App Configuration service

### 2.7 Diagnostic Logging

**Log Types:**

1. **Application Logging**:
   - Logs generated by application code
   - **Windows**: File system (temporary, 12 hours) or Azure Blob Storage
   - **Linux**: File system only
   - Levels: Error, Warning, Information, Verbose
   - Framework support: ASP.NET, ASP.NET Core, Node.js, Java, Python

2. **Web Server Logging (HTTP Logs)**:
   - Raw HTTP request data
   - W3C Extended Log File Format
   - Includes: time, client IP, method, URI, status code, bytes
   - **Windows**: File system or Blob storage
   - **Linux**: File system only

3. **Detailed Error Messages**:
   - Detailed error page HTML
   - Windows only
   - Saved to file system

4. **Failed Request Tracing**:
   - IIS Failed Request Tracing logs
   - XML format with XSL stylesheet
   - Windows only
   - Detailed trace of request pipeline

5. **Deployment Logging**:
   - Automatically enabled
   - Tracks deployment operations
   - Cannot be disabled

**Enabling Diagnostic Logging:**

**In Portal:**
```
App Service → Monitoring → App Service logs
```

**Via CLI:**
```bash
# Enable application logging
az webapp log config --name <app-name> --resource-group <rg> \
  --application-logging azureblobstorage \
  --level information \
  --detailed-error-messages true \
  --failed-request-tracing true \
  --web-server-logging filesystem

# Enable web server logging
az webapp log config --name <app-name> --resource-group <rg> \
  --web-server-logging filesystem

# Stream logs
az webapp log tail --name <app-name> --resource-group <rg>

# Download logs
az webapp log download --name <app-name> --resource-group <rg> \
  --log-file <local-file-path>
```

**Log Storage Locations:**
- **File System**: `/home/LogFiles` (Linux/Windows)
- **Blob Storage**: Requires storage account configuration
- **Retention**: File system - automatic cleanup, Blob - manual cleanup

**Streaming Logs:**
- Real-time log viewing
- Useful for development/debugging
- `az webapp log tail` or portal Log stream

**Diagnostic Logging Best Practices:**
- Use appropriate log levels
- Configure blob storage for long-term retention
- Use Application Insights for advanced monitoring
- Set retention policies to manage costs
- Don't log sensitive information

### 2.8 Security Features

**Managed Identity:**
- Provides identity for app to access Azure resources
- No credentials in code
- Two types:
  - **System-assigned**: Tied to app lifecycle, deleted with app
  - **User-assigned**: Independent lifecycle, can be shared

```bash
# Enable system-assigned managed identity
az webapp identity assign --name <app-name> --resource-group <rg>

# Assign user-assigned managed identity
az webapp identity assign --name <app-name> --resource-group <rg> \
  --identities <identity-resource-id>
```

**Key Vault References:**
- Reference secrets from Azure Key Vault
- Syntax: `@Microsoft.KeyVault(SecretUri=<secret-uri>)`
- Requires managed identity with Key Vault access
- App restarts when referenced secret is updated

```bash
# Example app setting using Key Vault reference
az webapp config appsettings set --name <app-name> --resource-group <rg> \
  --settings MySecret="@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)"
```

---

## MODULE 3: SCALE APPS IN AZURE APP SERVICE

### 3.1 Scale Up vs Scale Out

**Scale Up (Vertical Scaling):**
- Change to higher App Service plan tier
- Get more CPU, RAM, disk space, features
- Manual operation
- No downtime (brief restart)
- Examples:
  - Free → Basic
  - Basic → Standard
  - Standard → Premium
- Limitations: Hardware limits, regional availability

**Scale Out (Horizontal Scaling):**
- Increase number of VM instances
- Distribute load across multiple instances
- Can be manual or automatic
- Supported in Basic tier and above
- Benefits:
  - Better availability
  - Load distribution
  - Reduced response time

**Key Differences:**

| Aspect | Scale Up | Scale Out |
|--------|----------|-----------|
| Resource Type | CPU/RAM/Disk per instance | Number of instances |
| Direction | Vertical | Horizontal |
| Limit | Hardware availability | Plan tier limits |
| Cost Impact | Higher per-instance cost | More instances |
| Availability | No (single instance) | Yes (multiple instances) |
| Flexibility | Limited | Highly flexible |

### 3.2 Autoscale Overview

**What is Autoscale?**
- Automatically adjust instance count based on rules
- Responds to load changes without manual intervention
- Available in Standard tier and above
- Two types:
  1. **Automatic Scaling** (Platform-managed)
  2. **Azure Autoscale** (Rule-based)

**Automatic Scaling (Platform-Managed):**
- App Service handles scaling automatically
- Based on HTTP traffic patterns
- No rules or schedules needed
- Platform prewarms instances
- Available in Premium v2, Premium v3, Premium v4
- Simpler but less control

**Azure Autoscale (Rule-Based):**
- Define custom rules and schedules
- Based on metrics (CPU, memory, queue length, etc.)
- More control and flexibility
- Create complex scaling conditions
- Available Standard tier and above

### 3.3 Autoscale Conditions and Rules

**Autoscale Settings Structure:**
- **Profile**: Container for rules and schedules
- **Rules**: Define when and how to scale
- **Conditions**: Metric thresholds that trigger scaling

**Profile Types:**

1. **Default Profile**:
   - Always active
   - Fallback when other profiles inactive
   - Must always exist

2. **Fixed Date Profile**:
   - Active during specific date/time range
   - Example: Holiday traffic spike
   - Start date, end date, time zone

3. **Recurrence Profile**:
   - Repeats on schedule
   - Example: Weekday vs weekend scaling
   - Days of week, time ranges

**Autoscale Rules:**

**Components:**
1. **Metric Source**: Where data comes from
2. **Metric Name**: What to measure
3. **Time Aggregation**: How to aggregate (Average, Minimum, Maximum, Total, Count)
4. **Operator**: Comparison (>, >=, <, <=, ==, !=)
5. **Threshold**: Value to compare against
6. **Duration**: Time window for evaluation
7. **Scale Action**: What to do
8. **Cool down**: Wait period after scaling

**Common Metrics for Autoscale:**

**App Service Plan Metrics:**
- CPU Percentage
- Memory Percentage
- Disk Queue Length
- HTTP Queue Length
- Data In/Out

**Application Insights Metrics:**
- Request count
- Response time
- Failed requests
- Custom metrics

**Storage Queue:**
- Queue length
- Message count

**Service Bus:**
- Queue/topic message count
- Active message count

**Example Rule Configuration:**

**Scale-Out Rule:**
```
Metric: CPU Percentage
Time Aggregation: Average
Operator: Greater than
Threshold: 70
Duration: 10 minutes
Scale Action: Increase count by 1
Cool down: 5 minutes
```

**Scale-In Rule:**
```
Metric: CPU Percentage
Time Aggregation: Average
Operator: Less than
Threshold: 25
Duration: 10 minutes
Scale Action: Decrease count by 1
Cool down: 5 minutes
```

### 3.4 Autoscale Best Practices

**1. Always Define Both Scale-Out and Scale-In Rules:**
- Prevent resource waste
- Ensure cost optimization
- Avoid reaching max limits
- Use same metric for both directions

**2. Choose Appropriate Metrics:**
- CPU: General application load
- Memory: Memory-intensive applications
- HTTP Queue: Request backlog
- Custom metrics: Business-specific logic
- Multiple metrics: Use AND (scale-in) / OR (scale-out) logic

**3. Set Appropriate Thresholds:**
- Scale-out threshold: 70-80% (give safety margin)
- Scale-in threshold: 25-30% (avoid flapping)
- Leave gap between scale-out and scale-in thresholds
- Consider metric variability

**4. Ensure Sufficient Time Aggregation:**
- Minimum: 5 minutes
- Recommended: 10-15 minutes
- Longer duration = more stable scaling
- Shorter duration = faster response (but more churn)

**5. Configure Cool Down Periods:**
- Prevents scaling flapping
- Scale-out cool down: 3-5 minutes (can be shorter)
- Scale-in cool down: 5-10 minutes (should be longer)
- Allow new instances to stabilize
- Let metrics reflect new capacity

**6. Set Appropriate Instance Limits:**
- **Minimum**: Never less than what you need for availability (usually 2+)
- **Maximum**: Budget and resource limits
- **Default**: Safe middle ground
- Consider costs

**7. Multiple Rule Logic:**
- **Scale-Out**: OR logic - any rule triggers scale-out
- **Scale-In**: AND logic - all rules must be true
- If scale-out and scale-in triggered simultaneously, scale-out wins

**8. Flapping Prevention:**
- Autoscale detects flapping (rapid scale-out/scale-in cycles)
- Aborts scale attempts if flapping detected
- Logs to Activity Log
- Fix: Adjust thresholds, increase cool down, widen gap between rules

**9. Default Instance Count:**
- Used when metrics unavailable
- Should be safe for typical load
- Not too high (cost) or too low (availability)

**10. Monitor and Adjust:**
- Review Activity Log for scale events
- Check Run History
- Monitor actual vs target instance count
- Adjust rules based on patterns

**Autoscale Limitations:**
- Free/Shared: No autoscale
- Basic: No autoscale
- Standard: Up to 10 instances
- Premium: Up to 20-30 instances
- Isolated: Up to 100+ instances

### 3.5 Configuring Autoscale

**Via Azure Portal:**

1. Navigate to App Service Plan
2. Select **Scale out (App Service plan)**
3. Choose **Custom autoscale**
4. Configure default profile:
   - Set instance limits (min, max, default)
   - Add rules
5. Add additional profiles (optional)
6. Configure notifications
7. Save

**Via Azure CLI:**

```bash
# Create autoscale settings
az monitor autoscale create \
  --resource-group <rg> \
  --resource <app-service-plan-resource-id> \
  --resource-type Microsoft.Web/serverfarms \
  --name <autoscale-name> \
  --min-count 1 \
  --max-count 10 \
  --count 2

# Add scale-out rule (CPU > 70%)
az monitor autoscale rule create \
  --resource-group <rg> \
  --autoscale-name <autoscale-name> \
  --condition "Percentage CPU > 70 avg 10m" \
  --scale out 1

# Add scale-in rule (CPU < 25%)
az monitor autoscale rule create \
  --resource-group <rg> \
  --autoscale-name <autoscale-name> \
  --condition "Percentage CPU < 25 avg 10m" \
  --scale in 1

# List autoscale settings
az monitor autoscale list --resource-group <rg>

# Show autoscale settings
az monitor autoscale show --name <autoscale-name> --resource-group <rg>

# Delete autoscale settings
az monitor autoscale delete --name <autoscale-name> --resource-group <rg>
```

**Autoscale Notifications:**
- Email notifications to admins/co-admins
- Additional email addresses
- Webhook notifications
- Configured per autoscale setting
- Triggered on scale operations

**Monitoring Autoscale:**

**Activity Log:**
- All scale operations logged
- Success/failure status
- Reason for scaling
- Old vs new instance count
- Access via: Monitor → Activity Log

**Run History:**
- Visual timeline of scale events
- Shows instance count over time
- Trigger conditions met
- Access via: App Service Plan → Scale out → Run history

**Metrics:**
- Instance count over time
- Autoscale evaluations
- Custom metrics if used

### 3.6 Automatic Scaling (Platform-Managed)

**Overview:**
- App Service handles scaling without rules
- Based on HTTP traffic load
- Pre-warms instances for faster scaling
- Available: Premium v2, Premium v3, Premium v4
- Simpler alternative to Azure Autoscale

**Configuration:**

**Instance Limits:**
- **Minimum instances**: Always running (cost consideration)
- **Maximum instances**: Upper limit (cost protection)
- **Scale-out limit per app**: App-specific limit within plan

**Platform Behavior:**
- Monitors HTTP traffic patterns
- Automatically adds instances under load
- Pre-warms instances (buffer capacity)
- Scales in when load decreases
- Faster response than metric-based scaling

**Pre-warmed Instances:**
- Always maintained as buffer
- Default: 1 pre-warmed instance
- Ensures instant capacity for traffic spikes
- Billed per second

**Limitations:**
- Cannot mix with Azure Autoscale
- Only one scaling method per plan
- No custom metrics
- No schedule-based scaling
- Less control than rule-based autoscale

**When to Use:**
- HTTP-driven workloads
- Unpredictable traffic patterns
- Don't need schedule-based scaling
- Want simplicity over control

**When NOT to Use:**
- Need CPU/memory-based scaling
- Need schedule-based scaling  
- Need precise control over scaling behavior
- Using non-HTTP workloads (background jobs)

---

## MODULE 4: EXPLORE AZURE APP SERVICE DEPLOYMENT SLOTS

### 4.1 Deployment Slots Overview

**What are Deployment Slots?**
- Live apps with their own hostnames
- Separate instances of your app
- Run on same App Service plan (share resources)
- Each slot can have different code and configuration
- Available in Standard tier and above

**Key Characteristics:**
- Each slot is a full App Service instance
- Has unique URL: `<app-name>-<slot-name>.azurewebsites.net`
- Shares App Service plan resources (CPU, memory)
- Can have different code versions
- Can have slot-specific configuration

**Number of Slots by Tier:**
- Free/Shared: 0 (not supported)
- Basic: 0 (not supported)
- Standard: 5 slots
- Premium: 20 slots
- Isolated: 20 slots

**Common Slot Usage:**
- **Production**: Main slot (default, always exists)
- **Staging**: Pre-production testing
- **Development**: Developer testing
- **QA**: Quality assurance
- **UAT**: User acceptance testing

**Benefits:**

1. **Zero Downtime Deployments**:
   - Deploy to staging slot
   - Test thoroughly
   - Swap to production without downtime
   - Instant cutover

2. **Easy Rollbacks**:
   - Previous version preserved in slot
   - Swap back if issues detected
   - No re-deployment needed

3. **Staging Validation**:
   - Test in production-like environment
   - Same infrastructure as production
   - Validate before going live

4. **Warm-up Support**:
   - App warmed up before swap
   - Custom warm-up actions
   - No cold start in production

5. **A/B Testing / Traffic Routing**:
   - Route percentage of traffic to different slots
   - Test new features with subset of users
   - Gradual rollout capability

### 4.2 Creating and Managing Deployment Slots

**Creating a Slot:**

**Via Portal:**
```
App Service → Deployment → Deployment slots → Add slot
Name: staging
Clone settings from: production (or create new)
```

**Via CLI:**
```bash
# Create deployment slot
az webapp deployment slot create \
  --name <app-name> \
  --resource-group <rg> \
  --slot <slot-name>

# Create slot and clone configuration
az webapp deployment slot create \
  --name <app-name> \
  --resource-group <rg> \
  --slot <slot-name> \
  --configuration-source <source-slot-or-app-name>

# List slots
az webapp deployment slot list \
  --name <app-name> \
  --resource-group <rg>

# Delete slot
az webapp deployment slot delete \
  --name <app-name> \
  --resource-group <rg> \
  --slot <slot-name>
```

**Accessing Slots:**
- URL: `https://<app-name>-<slot-name>.azurewebsites.net`
- Can configure custom domains for slots
- Each slot has own publish profile

**Slot Configuration:**
- Can clone settings from another slot/production
- Or create with new configuration
- Each slot has independent:
  - App settings
  - Connection strings
  - Handler mappings
  - Authentication settings
  - Certificate bindings

### 4.3 Slot Settings (Sticky vs Swappable)

**Settings Behavior During Swap:**

**Swapped Settings (Move with code):**
- General settings (framework version, 32/64-bit, web sockets)
- App settings (unless marked as slot setting)
- Connection strings (unless marked as slot setting)
- Handler mappings
- Public certificates
- WebJobs content
- Hybrid connections *
- Service endpoints *
- Azure CDN *
- Path mappings
- Managed identities (user-assigned swap, system-assigned don't)

**Non-Swapped Settings (Slot-Specific):**
- Publishing endpoints
- Custom domain names
- Non-public certificates and TLS/SSL settings
- Scale settings
- Always On setting
- IP restrictions
- App settings marked as "Deployment slot setting"
- Connection strings marked as "Deployment slot setting"
- Diagnostic log settings
- CORS settings
- Virtual network integration
- Managed identities (system-assigned)

**Deployment Slot Setting (Sticky Setting):**
- Mark settings to NOT swap
- Remain with the slot
- Use for environment-specific values

**Configuring Slot Settings:**

**Via Portal:**
```
App Service → Settings → Environment variables
Select setting → Check "Deployment slot setting"
```

**Via CLI:**
```bash
# Set slot-specific app setting
az webapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --slot <slot-name> \
  --settings KEY=VALUE \
  --slot-settings KEY  # Makes it sticky

# Set slot-specific connection string
az webapp config connection-string set \
  --name <app-name> \
  --resource-group <rg> \
  --slot <slot-name> \
  --connection-string-type SQLAzure \
  --settings MyDB='<connection-string>' \
  --slot-settings MyDB  # Makes it sticky
```

**Common Sticky Settings:**
- Database connection strings (different DB per environment)
- API keys for different environments
- Feature flags
- Application Insights keys
- Logging levels

### 4.4 Swapping Deployment Slots

**What Happens During Swap:**

**Phase 1: Apply Target Slot Settings to Source**
1. Apply target slot settings to all source instances
2. Includes: slot-specific app settings, connection strings, configurations
3. Triggers restart of all source instances
4. Source instances initialize with target settings

**Phase 2: Wait for Warm-up (if enabled)**
1. Send HTTP request to source instances
2. Custom warm-up via `applicationInitialization` web.config
3. Wait for instances to respond successfully
4. If any instance fails warm-up, swap is aborted

**Phase 3: Switch Routing Rules**
1. If all source instances warm-up successful
2. Swap routing rules
3. Target slot now points to source app
4. Source slot points to target app (what was in production)

**Phase 4: Apply Source Settings to New Source**
1. Former production (now in source slot) gets source slot settings
2. Restart new source instances

**Swap Types:**

**1. Standard Swap (Production Swap):**
- Direct swap between slots
- Immediate cutover
- Most common operation

**2. Swap with Preview (Multi-phase Swap):**
- Two-phase swap
- Review changes before completing
- Phase 1: Apply settings, warm-up source
- Validation: Test source with production settings
- Phase 2: Complete swap (or cancel)

**3. Auto Swap:**
- Automatic swap after successful deployment
- Configured on source slot
- Swaps to target slot automatically
- Useful for CI/CD pipelines
- Zero manual intervention

**Performing Swaps:**

**Standard Swap - Portal:**
```
App Service → Deployment slots → Select Swap
Source: staging
Target: production
Review changes → Swap
```

**Standard Swap - CLI:**
```bash
# Swap slots
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg> \
  --slot <source-slot> \
  --target-slot <target-slot>  # Usually 'production'

# Example: Swap staging to production
az webapp deployment slot swap \
  --name myapp \
  --resource-group myrg \
  --slot staging \
  --target-slot production
```

**Swap with Preview - Portal:**
```
App Service → Deployment slots → Select Swap
Enable "Perform swap with preview"
Step 1: Initialize swap → Validate in staging
Step 2: Complete swap (or Cancel)
```

**Swap with Preview - CLI:**
```bash
# Phase 1: Start swap with preview
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg> \
  --slot <source-slot> \
  --target-slot <target-slot> \
  --action preview

# [Validate changes in source slot]

# Phase 2: Complete swap
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg> \
  --slot <source-slot> \
  --target-slot <target-slot> \
  --action swap

# OR: Cancel swap
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg> \
  --slot <source-slot> \
  --target-slot <target-slot> \
  --action reset
```

**Auto Swap Configuration:**

**Via Portal:**
```
Staging slot → Settings → Configuration → General settings
Auto swap enabled: On
Auto swap deployment slot: production
Save (restart required)
```

**Via CLI:**
```bash
# Enable auto swap
az webapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --slot <source-slot> \
  --settings WEBSITE_AUTO_SWAP_SLOT_NAME=production
```

**Auto Swap Considerations:**
- Enabled on source slot, not target
- Triggers after successful deployment
- Not supported on Linux/Containers
- Useful for continuous deployment
- Should test thoroughly before enabling

**Rollback After Swap:**
- Simply swap again in reverse direction
- Previous version preserved in source slot
- Instant rollback
- No need to redeploy code

```bash
# Rollback (swap back)
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg> \
  --slot staging \  # Was production before
  --target-slot production  # Now has new version
```

### 4.5 Custom Warm-up and Application Initialization

**Why Warm-up?**
- Prevent cold start in production
- Load caches/configuration before traffic
- Initialize database connections
- Compile code/assets
- Ensure app fully ready

**Application Initialization Module:**
- IIS module for custom warm-up
- Configured in web.config or applicationHost.config
- Sends requests to specified paths
- Waits for successful response before swap

**Web.config Configuration:**

```xml
<configuration>
  <system.webServer>
    <applicationInitialization 
      doAppInitAfterRestart="true" 
      skipManagedModules="true">
      <add initializationPage="/warmup" />
      <add initializationPage="/api/health" />
    </applicationInitialization>
  </system.webServer>
</configuration>
```

**App Settings for Warm-up:**

```bash
# Set warm-up path
az webapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --settings WEBSITE_SWAP_WARMUP_PING_PATH="/warmup"

# Set warm-up status check path
az webapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --settings WEBSITE_SWAP_WARMUP_PING_STATUSES="200,202"
```

**Custom Warm-up Endpoint:**
```csharp
// ASP.NET Core example
[HttpGet("/warmup")]
public IActionResult Warmup()
{
    // Perform warm-up tasks
    // Load cache
    _cache.LoadInitialData();
    
    // Initialize connections
    _dbContext.Database.CanConnect();
    
    // Return 200 when ready
    return Ok("Warmed up");
}
```

**Warm-up Best Practices:**
- Keep warm-up fast (<5 minutes)
- Return success status only when truly ready
- Don't depend on external services
- Test warm-up locally
- Monitor warm-up duration

### 4.6 Route Traffic to Slots (Testing in Production)

**Traffic Routing Feature:**
- Split traffic between slots
- Percentage-based distribution
- Users "stick" to assigned slot via cookie
- Also called "Testing in Production"

**Use Cases:**
- A/B testing
- Beta features
- Canary deployments
- Gradual rollout
- User feedback collection

**Configuring Traffic Routing:**

**Via Portal:**
```
App Service → Deployment slots
Set traffic percentage for each slot
Production: 80%
Staging: 20%
Save
```

**Via CLI:**
```bash
# Route 20% traffic to staging
az webapp traffic-routing set \
  --name <app-name> \
  --resource-group <rg> \
  --distribution staging=20

# Clear traffic routing (100% to production)
az webapp traffic-routing clear \
  --name <app-name> \
  --resource-group <rg>

# Show current routing
az webapp traffic-routing show \
  --name <app-name> \
  --resource-group <rg>
```

**How It Works:**
- User assigned to slot on first visit
- Cookie set: `x-ms-routing-name=<slot-name>`
- Subsequent requests route to same slot
- Cookie expires based on session
- Users can't switch slots during session

**Manual Routing:**
- Users can manually route to specific slot
- Append to URL: `?x-ms-routing-name=<slot-name>`
- Example: `https://myapp.azurewebsites.net/?x-ms-routing-name=staging`
- Useful for testers/developers

**Monitoring Traffic Distribution:**
- Application Insights: Track by slot
- Custom telemetry: Include slot name
- Analyze user experience per slot
- Compare metrics between slots

**Traffic Routing Best Practices:**
- Start with small percentage (5-10%)
- Monitor errors and performance
- Gradually increase percentage
- Use Application Insights for comparison
- Have rollback plan ready

### 4.7 Deployment Slot Best Practices

**1. Always Test in Staging:**
- Deploy to staging first
- Run full test suite
- Validate with production settings
- Only swap after successful tests

**2. Use Slot-Specific Settings:**
- Mark environment-specific settings as sticky
- Database connections
- API endpoints
- Feature flags
- Different Application Insights keys per slot

**3. Warm-up Configuration:**
- Configure applicationInitialization
- Ensure app fully ready before swap
- Test warm-up process
- Monitor warm-up duration

**4. Clone Configuration Carefully:**
- Understand what gets cloned
- Review settings after creation
- Adjust slot-specific values
- Document configuration differences

**5. Swap with Preview for Critical Changes:**
- Use multi-phase swap
- Validate in production environment
- Review before completing
- Can cancel if issues found

**6. Enable Auto Swap for CI/CD:**
- Automate deployment pipeline
- Reduce manual steps
- Faster deployments
- Ensure thorough automated testing first

**7. Monitor Swap Operations:**
- Check Activity Log
- Verify successful swap
- Monitor app health after swap
- Be ready to rollback

**8. Use Multiple Slots:**
- Development → QA → Staging → Production
- Each stage gates to next
- Reduces production issues
- More thorough testing

**9. Consider Costs:**
- All slots share App Service plan resources
- No additional cost for slots
- But all instances scaled together
- May need larger plan

**10. Naming Conventions:**
- Consistent naming: `dev`, `qa`, `staging`, `production`
- Clear purpose for each slot
- Document slot usage
- Include in deployment docs

### 4.8 Deployment Slot Limitations

**Resource Sharing:**
- All slots share same App Service plan
- Share CPU, memory, storage
- Scaling affects all slots equally
- Can impact performance if overused

**Plan Tier Limits:**
- Free/Shared: No slots
- Basic: No slots
- Standard: 5 slots
- Premium: 20 slots
- Isolated: 20 slots

**Swap Limitations:**
- Some settings don't swap
- System-assigned managed identities don't swap
- Custom domain names don't swap
- SSL certificates don't swap
- Scale settings don't swap

**Auto Swap Restrictions:**
- Not supported on Linux
- Not supported for containers
- Requires Standard tier or higher
- Must enable on source slot

**Testing in Production Limitations:**
- Cookie-based (can be cleared)
- No programmatic control of assignment
- All-or-nothing per user session
- Limited to percentage distribution

---

## MODULE 5: CUSTOM CONTAINERS & DOCKER DEPLOYMENT

### 5.1 Web App for Containers Overview

**What is Web App for Containers?**
- Deploy custom Docker containers to App Service
- Bring your own runtime, framework, or language
- Runs on App Service Linux or Windows (sidecar)
- Container image pulled from a registry
- Supports single-container and multi-container apps

**Supported Container Registries:**
- Azure Container Registry (ACR) — recommended
- Docker Hub (public and private)
- Any private registry with HTTP-based access

### 5.2 Deploying a Custom Container

**Via Azure CLI:**
```bash
# Create App Service plan (Linux)
az appservice plan create \
  --name myLinuxPlan \
  --resource-group myRG \
  --sku B1 \
  --is-linux

# Create web app from Docker Hub image
az webapp create \
  --resource-group myRG \
  --plan myLinuxPlan \
  --name mycontainerapp \
  --deployment-container-image-name nginx:latest

# Create web app from Azure Container Registry
az webapp create \
  --resource-group myRG \
  --plan myLinuxPlan \
  --name mycontainerapp \
  --deployment-container-image-name myregistry.azurecr.io/myapp:latest

# Configure ACR credentials
az webapp config container set \
  --name mycontainerapp \
  --resource-group myRG \
  --docker-custom-image-name myregistry.azurecr.io/myapp:latest \
  --docker-registry-server-url https://myregistry.azurecr.io \
  --docker-registry-server-user <username> \
  --docker-registry-server-password <password>
```

**Using Managed Identity for ACR (Recommended):**
```bash
# Enable system-assigned managed identity
az webapp identity assign --name mycontainerapp --resource-group myRG

# Grant ACR pull permission
az role assignment create \
  --assignee <principal-id> \
  --role AcrPull \
  --scope <acr-resource-id>

# Configure app to use managed identity for ACR
az webapp config set \
  --name mycontainerapp \
  --resource-group myRG \
  --generic-configurations '{"acrUseManagedIdentityCreds": true}'
```

### 5.3 Container Configuration Settings

**Key App Settings for Containers:**

| Setting | Description |
|---------|-------------|
| `DOCKER_REGISTRY_SERVER_URL` | Registry URL (e.g., `https://myregistry.azurecr.io`) |
| `DOCKER_REGISTRY_SERVER_USERNAME` | Registry username |
| `DOCKER_REGISTRY_SERVER_PASSWORD` | Registry password |
| `WEBSITES_PORT` | Port your container listens on (default: 80 or 8080) |
| `WEBSITES_ENABLE_APP_SERVICE_STORAGE` | `true` = mount `/home` as persistent shared storage |
| `DOCKER_ENABLE_CI` | `true` = enable CI/CD webhook for container updates |

**Container Port Configuration:**
```bash
# Set the port your container exposes
az webapp config appsettings set \
  --name mycontainerapp \
  --resource-group myRG \
  --settings WEBSITES_PORT=3000
```

**Important:** If your container listens on a port other than 80 or 8080, you MUST set `WEBSITES_PORT`.

### 5.4 Persistent Storage for Containers

**Storage Behavior:**
- By default, `/home` directory is **NOT** persistent across container restarts
- Enable persistence with `WEBSITES_ENABLE_APP_SERVICE_STORAGE=true`
- `/home` is then mapped to Azure Storage (shared across instances)
- Other directories are container-local (lost on restart)

```bash
# Enable persistent storage
az webapp config appsettings set \
  --name mycontainerapp \
  --resource-group myRG \
  --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=true
```

### 5.5 Multi-Container Apps (Docker Compose)

**Supported on Linux App Service:**
```bash
# Deploy multi-container app with Docker Compose
az webapp create \
  --resource-group myRG \
  --plan myLinuxPlan \
  --name mymulticontainerapp \
  --multicontainer-config-type compose \
  --multicontainer-config-file docker-compose.yml
```

**Example docker-compose.yml:**
```yaml
version: '3'
services:
  web:
    image: myregistry.azurecr.io/webapp:latest
    ports:
      - "80:3000"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
```

**Multi-Container Limitations:**
- Only Docker Compose supported (Kubernetes not supported in App Service)
- Limited to 4 containers per app
- Port mapping: Only one container can be exposed to external traffic on port 80/443
- No support for `build` directive — must use pre-built images
- `depends_on` is supported but `volumes` has limitations

### 5.6 Continuous Deployment for Containers

**Webhook-based CI/CD:**
```bash
# Enable CI/CD webhook
az webapp deployment container config \
  --enable-cd true \
  --name mycontainerapp \
  --resource-group myRG

# Get the webhook URL
az webapp deployment container show-cd-url \
  --name mycontainerapp \
  --resource-group myRG
```

**How it works:**
1. You push a new image to your registry
2. Registry triggers webhook to App Service
3. App Service pulls the new image
4. Container restarts with updated image

**ACR Webhook Setup:**
```bash
# Create ACR webhook
az acr webhook create \
  --registry myregistry \
  --name mywebhook \
  --uri <webhook-url-from-above> \
  --actions push \
  --scope myapp:latest
```

### 5.7 SSH into Linux Containers

**Built-in SSH:**
- App Service provides SSH access to Linux containers
- Access via Kudu: `https://<app-name>.scm.azurewebsites.net/webssh/host`
- Or via portal: App Service → Development Tools → SSH
- Custom containers need SSH server configured in Dockerfile

**Dockerfile SSH Setup:**
```dockerfile
# Install SSH server in your custom image
RUN apt-get update && apt-get install -y openssh-server \
    && echo "root:Docker!" | chpasswd \
    && mkdir /run/sshd

COPY sshd_config /etc/ssh/
EXPOSE 2222 8000

CMD ["/usr/sbin/sshd", "-D"]
```

**Note:** SSH password must be `Docker!` (with exclamation mark). Port is always 2222.

### 5.8 Container Startup Time & Health

**Startup Timeout:**
- Default: App Service waits **230 seconds** for container to start
- Configurable via `WEBSITES_CONTAINER_START_TIME_LIMIT` (max: 1800 seconds = 30 min)
- If container doesn't respond in time, it's restarted

```bash
# Increase container startup timeout
az webapp config appsettings set \
  --name mycontainerapp \
  --resource-group myRG \
  --settings WEBSITES_CONTAINER_START_TIME_LIMIT=600
```

---

## MODULE 6: WEBJOBS

### 6.1 WebJobs Overview

**What are WebJobs?**
- Background tasks that run in the context of an App Service web app
- Run on the same VM instances as the web app
- No additional cost (shares App Service plan resources)
- Supports scripts and executables: .exe, .cmd, .bat, .sh, .ps1, .py, .js, .php, .jar

**WebJob Types:**

| Type | Description | Trigger | Always On Required |
|------|-------------|---------|-------------------|
| **Continuous** | Runs continuously in a loop | Starts immediately, runs endlessly | ✅ Yes |
| **Triggered** | Runs on demand or on schedule | Manual, CRON, or webhook | ✅ For CRON-based |

### 6.2 Continuous WebJobs

**Behavior:**
- Starts immediately when created
- Runs in an endless loop
- Must implement loop logic in code
- Supports graceful shutdown
- Runs on ALL instances (or single instance)

**Single Instance vs All Instances:**
```bash
# settings.job file for single instance
{
  "is_singleton": true  # Run on only one instance
}
```

**Graceful Shutdown:**
- When app stops/restarts, WebJob receives a shutdown notification
- Creates file: `<webjob-path>\shutdown.txt`
- WebJob should check for this file and exit cleanly
- Grace period: **5 seconds** (then forcefully terminated)

### 6.3 Triggered WebJobs

**Manual Trigger:**
```bash
# Trigger via Kudu REST API
POST https://<app-name>.scm.azurewebsites.net/api/triggeredwebjobs/<job-name>/run
```

**CRON-Based Scheduling:**

Create a `settings.job` file in the WebJob directory:
```json
{
  "schedule": "0 */5 * * * *"
}
```

**CRON Format (6 fields — same as Azure Functions):**
```
{second} {minute} {hour} {day} {month} {day-of-week}
```

**Common CRON Examples:**

| Expression | Description |
|-----------|-------------|
| `0 */5 * * * *` | Every 5 minutes |
| `0 0 * * * *` | Every hour |
| `0 0 9 * * *` | Every day at 9:00 AM |
| `0 0 9 * * 1-5` | Weekdays at 9:00 AM |
| `0 30 9 1 * *` | 1st of every month at 9:30 AM |
| `0 0 0 * * 0` | Every Sunday at midnight |

**Important:** CRON-scheduled triggered WebJobs require **Always On** to be enabled (Basic tier+).

### 6.4 WebJobs SDK

**What is WebJobs SDK?**
- Framework for building background processing
- Simplified trigger and binding model (similar to Azure Functions)
- Uses same trigger/binding concepts

**NuGet Package:**
```bash
dotnet add package Microsoft.Azure.WebJobs
dotnet add package Microsoft.Azure.WebJobs.Extensions
```

**Example: Queue-Triggered WebJob:**
```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;

public class Functions
{
    // Triggered when message added to queue
    public static void ProcessQueueMessage(
        [QueueTrigger("myqueue")] string message,
        ILogger logger)
    {
        logger.LogInformation($"Processing: {message}");
    }

    // Timer-triggered (CRON)
    public static void TimerJob(
        [TimerTrigger("0 */5 * * * *")] TimerInfo timer,
        ILogger logger)
    {
        logger.LogInformation("Timer triggered!");
    }

    // Blob-triggered
    public static void ProcessBlob(
        [BlobTrigger("input/{name}")] Stream input,
        string name,
        ILogger logger)
    {
        logger.LogInformation($"Processing blob: {name}");
    }
}
```

**Program.cs for WebJobs SDK:**
```csharp
using Microsoft.Extensions.Hosting;

var builder = new HostBuilder();
builder.ConfigureWebJobs(b =>
{
    b.AddAzureStorageCoreServices();
    b.AddAzureStorage();      // Queue and Blob triggers
    b.AddTimers();             // Timer triggers
    b.AddServiceBus();         // Service Bus triggers
});
builder.ConfigureLogging(logging =>
{
    logging.AddConsole();
});

var host = builder.Build();
using (host)
{
    await host.RunAsync();
}
```

### 6.5 WebJobs vs Azure Functions

| Aspect | WebJobs | Azure Functions |
|--------|---------|-----------------|
| **Hosting** | Runs inside App Service | Standalone service (or App Service plan) |
| **Scaling** | Scales with App Service plan | Independent scaling (Consumption plan) |
| **Cost** | No additional cost | Pay-per-execution (Consumption) or plan-based |
| **Triggers** | Queue, Blob, Timer, Service Bus, manual | All WebJob triggers + HTTP, Event Grid, Cosmos DB, etc. |
| **Development** | Full .NET Host control | Simplified function model |
| **Monitoring** | Kudu dashboard, logs | Application Insights, portal metrics |
| **Event-driven scaling** | ❌ No | ✅ Yes (Consumption plan) |
| **Cold start** | ❌ No (always on same VM) | ✅ Yes (Consumption plan) |
| **Long-running** | ✅ Yes (continuous) | ⚠️ Timeout limits per plan |
| **Use when** | Background processing alongside web app | Standalone event-driven microservices |

**Key Decision:** Use **WebJobs** when you want background processing tightly coupled with your web app. Use **Azure Functions** when you want independent, event-driven, auto-scaling compute.

### 6.6 Deploying WebJobs

**Deployment Methods:**
1. **Portal**: App Service → Settings → WebJobs → Add
2. **ZIP Deploy**: Include in `App_Data/jobs/continuous/<name>` or `App_Data/jobs/triggered/<name>`
3. **Visual Studio**: Publish as WebJob project
4. **Kudu**: Upload via REST API

**Directory Structure:**
```
site/
└── wwwroot/
    └── App_Data/
        └── jobs/
            ├── continuous/
            │   └── mywebjob/
            │       ├── run.cmd (or .exe, .ps1, .py, .sh)
            │       └── settings.job (optional)
            └── triggered/
                └── myscheduledJob/
                    ├── run.cmd
                    └── settings.job (with CRON schedule)
```

---

## MODULE 7: ADVANCED DEPLOYMENT TECHNIQUES

### 7.1 Deployment Methods In-Depth

**Overview of All Methods:**

| Method | Description | Best For |
|--------|-------------|----------|
| ZIP Deploy | Upload ZIP file | CI/CD pipelines, simple deployments |
| Run from Package | Mount ZIP as read-only filesystem | Production (faster, predictable) |
| Local Git | Push from local Git repo | Development, small teams |
| External Git | Sync from GitHub/Bitbucket | CI/CD with repository integration |
| GitHub Actions | CI/CD workflow | Modern CI/CD |
| Azure DevOps | Azure Pipelines | Enterprise CI/CD |
| FTP/FTPS | File transfer | Legacy deployments, individual files |
| WAR Deploy | Java WAR file | Java applications |
| `az webapp up` | Quick deploy from local code | Rapid prototyping |

### 7.2 ZIP Deploy

```bash
# ZIP Deploy via CLI
az webapp deploy \
  --resource-group myRG \
  --name myapp \
  --src-path app.zip \
  --type zip

# ZIP Deploy via Kudu REST API
curl -X POST \
  -u <username>:<password> \
  --data-binary @app.zip \
  "https://<app-name>.scm.azurewebsites.net/api/zipdeploy"

# With async (for large deployments)
curl -X POST \
  -u <username>:<password> \
  --data-binary @app.zip \
  "https://<app-name>.scm.azurewebsites.net/api/zipdeploy?isAsync=true"
```

**ZIP Deploy Behavior:**
- Extracts ZIP contents to `D:\home\site\wwwroot` (Windows) or `/home/site/wwwroot` (Linux)
- Deletes old files and replaces with new ones
- Triggers Oryx build if `SCM_DO_BUILD_DURING_DEPLOYMENT=true`

### 7.3 Run from Package (WEBSITE_RUN_FROM_PACKAGE)

**Three Modes:**

| Value | Behavior |
|-------|----------|
| `0` | Normal deployment (default) — files are mutable |
| `1` | Run from local ZIP in `d:\home\data\SitePackages` — **read-only** filesystem |
| `<URL>` | Run from external ZIP URL (e.g., Blob Storage SAS URL) — **read-only** filesystem |

```bash
# Mode 1: Run from local package
az webapp config appsettings set \
  --name myapp \
  --resource-group myRG \
  --settings WEBSITE_RUN_FROM_PACKAGE=1

# Then deploy the ZIP
az webapp deploy --resource-group myRG --name myapp --src-path app.zip --type zip

# Mode 2: Run from external URL
az webapp config appsettings set \
  --name myapp \
  --resource-group myRG \
  --settings WEBSITE_RUN_FROM_PACKAGE="https://mystorage.blob.core.windows.net/deploys/app.zip?sv=..."
```

**Key Benefits of Run from Package:**
- **Faster cold starts** — no file extraction needed
- **Predictable deployments** — atomic, no partial deployments
- **Reduced storage locks** — no file locking issues
- **Read-only wwwroot** — more secure, no accidental modifications

**Limitations:**
- wwwroot is **read-only** — cannot write files to app directory
- Cannot use for apps that write to local filesystem
- External URL mode depends on blob availability

### 7.4 Deployment Credentials

**Two Types:**

| Type | Scope | Format |
|------|-------|--------|
| **User-level** | Entire Azure account | `<username>` |
| **App-level** | Specific app | `<app-name>\$<app-name>` |

```bash
# Set user-level deployment credentials
az webapp deployment user set \
  --user-name <username> \
  --password <password>

# Get app-level (publish profile) credentials
az webapp deployment list-publishing-credentials \
  --name myapp \
  --resource-group myRG
```

**App-level credentials are auto-generated** and can be found in the Publish Profile (download from portal).

### 7.5 Kudu (SCM Site)

**What is Kudu?**
- Deployment engine for App Service
- Provides REST API for deployment and management
- Accessible at `https://<app-name>.scm.azurewebsites.net`
- Provides file browser, console, process explorer, environment info

**Key Kudu Endpoints:**

| Endpoint | Description |
|----------|-------------|
| `/` | Kudu dashboard |
| `/api/zipdeploy` | ZIP deployment |
| `/api/deployments` | Deployment history |
| `/api/settings` | App settings |
| `/api/vfs/` | Virtual file system (browse/edit files) |
| `/api/triggeredwebjobs` | Manage triggered WebJobs |
| `/api/continuouswebjobs` | Manage continuous WebJobs |
| `/DebugConsole` | Interactive console (CMD/PowerShell) |
| `/ProcessExplorer` | View running processes |
| `/webssh/host` | SSH into Linux container |

### 7.6 Build Automation (Oryx)

**What is Oryx?**
- App Service's build system
- Automatically detects language and builds app
- Runs during deployment (not at runtime)
- Supports: .NET, Node.js, Python, PHP, Java, Ruby

**Key App Settings:**

| Setting | Description |
|---------|-------------|
| `SCM_DO_BUILD_DURING_DEPLOYMENT` | `true` = run Oryx build during deployment |
| `ENABLE_ORYX_BUILD` | `true` = enable Oryx build system |
| `PRE_BUILD_COMMAND` | Command to run before build |
| `POST_BUILD_COMMAND` | Command to run after build |
| `PROJECT` | Path to the project file to build |

```bash
# Enable build during deployment
az webapp config appsettings set \
  --name myapp \
  --resource-group myRG \
  --settings SCM_DO_BUILD_DURING_DEPLOYMENT=true ENABLE_ORYX_BUILD=true
```

### 7.7 CI/CD with GitHub Actions

**Basic GitHub Actions Workflow:**
```yaml
name: Deploy to Azure App Service

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'

    - name: Build
      run: dotnet publish -c Release -o ./publish

    - name: Deploy to Azure
      uses: azure/webapps-deploy@v3
      with:
        app-name: 'myapp'
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ./publish
```

**Deploy to Slot:**
```yaml
    - name: Deploy to Staging
      uses: azure/webapps-deploy@v3
      with:
        app-name: 'myapp'
        slot-name: 'staging'
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE_STAGING }}
        package: ./publish
```

### 7.8 Deployment Best Practices

1. **Use Run from Package** for production deployments (faster, predictable, read-only)
2. **Use deployment slots** — never deploy directly to production
3. **Automate with CI/CD** — GitHub Actions or Azure DevOps
4. **Use Managed Identity for ACR** — avoid storing registry credentials
5. **Enable build automation** only when needed (adds deployment time)
6. **Lock down SCM site** — apply access restrictions to `.scm.` endpoint separately

---

## MODULE 8: CORS, HEALTH CHECK, BACKUP & RESTORE, LOCAL CACHE

### 8.1 CORS (Cross-Origin Resource Sharing)

**What is CORS in App Service?**
- Built-in CORS support in the platform
- Controls which origins can call your APIs
- Configured at the App Service level (not in code)
- Processes CORS preflight (OPTIONS) requests automatically

**Configuration:**
```bash
# Set allowed origins
az webapp cors add \
  --name myapp \
  --resource-group myRG \
  --allowed-origins "https://example.com" "https://www.example.com"

# Allow all origins (NOT recommended for production)
az webapp cors add \
  --name myapp \
  --resource-group myRG \
  --allowed-origins "*"

# Show CORS settings
az webapp cors show --name myapp --resource-group myRG

# Remove an origin
az webapp cors remove \
  --name myapp \
  --resource-group myRG \
  --allowed-origins "https://example.com"
```

**Key Points:**
- App Service CORS and app-level CORS (e.g., ASP.NET CORS middleware) should NOT be used together
- If both are configured, **App Service CORS takes precedence** and app-level CORS is ignored
- App Service CORS does NOT support `Access-Control-Allow-Credentials` header — use app-level CORS for that
- For credentialed requests, configure CORS in your application code instead

### 8.2 Health Check

**What is Health Check?**
- Built-in feature that pings a path in your app at regular intervals
- If an instance fails health checks, it's removed from the load balancer
- Helps maintain availability by routing traffic away from unhealthy instances

**Configuration:**
```bash
# Set health check path
az webapp config set \
  --name myapp \
  --resource-group myRG \
  --generic-configurations '{"healthCheckPath": "/health"}'
```

**How It Works:**
1. App Service pings `/health` (or configured path) every **1 minute**
2. If instance returns status code **200-299** → healthy
3. If instance fails for **5 consecutive checks** (configurable, 2-10) → unhealthy
4. Unhealthy instance removed from load balancer rotation
5. Continues checking — if instance recovers, it's added back
6. If instance stays unhealthy for **1 hour**, it's **replaced** (only on 2+ instances)

**Health Check Best Practices:**
- Implement a `/health` endpoint that checks dependencies (DB, cache, etc.)
- Return 200 only when app is truly healthy
- Keep health check fast (<5 seconds)
- Don't use `/` as health check path
- Requires **2+ instances** for instance replacement (won't replace single instance)

**Example Health Check Endpoint:**
```csharp
[HttpGet("/health")]
public async Task<IActionResult> HealthCheck(
    [FromServices] DbContext dbContext,
    [FromServices] IDistributedCache cache)
{
    try
    {
        // Check database
        await dbContext.Database.CanConnectAsync();
        
        // Check cache
        await cache.GetStringAsync("health-check");
        
        return Ok("Healthy");
    }
    catch
    {
        return StatusCode(503, "Unhealthy");
    }
}
```

### 8.3 Backup and Restore

**What is Backup & Restore?**
- Create backups of your app's content and configuration
- Store backups in an Azure Storage account
- Available in **Standard tier and above**

**What's Backed Up:**
- App configuration (app settings, connection strings)
- File content (`/home/site/wwwroot`)
- Database content (if linked — Azure SQL, MySQL)
- Up to **10 GB** total (app + database)

**What's NOT Backed Up:**
- Linked storage (Azure Storage mounts)
- Traffic routing settings
- Managed Identity configuration
- Private certificates
- Scale settings

**Backup Types:**

| Type | Description | Frequency |
|------|-------------|-----------|
| **Manual** | On-demand backup | Anytime |
| **Automatic (Scheduled)** | Periodic backup | Every 1-24 hours or custom CRON |

**Configuration:**
```bash
# Create a manual backup
az webapp config backup create \
  --resource-group myRG \
  --webapp-name myapp \
  --container-url "https://mystorage.blob.core.windows.net/backups?sv=..." \
  --backup-name "manual-backup-2026"

# Configure automatic backups
az webapp config backup update \
  --resource-group myRG \
  --webapp-name myapp \
  --container-url "https://mystorage.blob.core.windows.net/backups?sv=..." \
  --frequency 1d \
  --retain-one true \
  --retention 30

# Restore from backup
az webapp config backup restore \
  --resource-group myRG \
  --webapp-name myapp \
  --container-url "https://mystorage.blob.core.windows.net/backups?sv=..." \
  --backup-name "manual-backup-2026" \
  --overwrite
```

**Backup Limitations:**
- Max **10 GB** (app + database combined)
- Requires Standard tier or above
- Needs Storage Account with SAS URL
- Automatic backups have a max retention of **30 days**
- If backup exceeds 10 GB, it fails silently
- Full backups (not incremental)

### 8.4 Local Cache

**What is Local Cache?**
- Copies site content from Azure Storage to local disk of the VM instance
- App runs from the local disk copy (faster I/O)
- Changes to remote content not reflected until restart

**When to Use:**
- Apps with high content read-to-write ratio
- Read-heavy workloads (serving static files)
- Want to eliminate Azure Storage dependency for content

**Configuration:**
```bash
# Enable local cache
az webapp config appsettings set \
  --name myapp \
  --resource-group myRG \
  --settings WEBSITE_LOCAL_CACHE_OPTION=Always

# Set local cache size (default: 1 GB, max: 2 GB)
az webapp config appsettings set \
  --name myapp \
  --resource-group myRG \
  --settings WEBSITE_LOCAL_CACHE_SIZEINMB=1000
```

**Key Points:**
- `WEBSITE_LOCAL_CACHE_OPTION=Always` enables local cache
- `WEBSITE_LOCAL_CACHE_OPTION=Default` disables it (default)
- Max size: **2 GB**
- Changes written to local cache are **lost on restart** (not synced back)
- Not suitable for apps that write content (CMS, file upload)
- Each instance has its own local cache copy
- App restarts when local cache is provisioned

### 8.5 App Service Managed Certificate Limitations

**Free Managed Certificate — What You CAN'T Do:**
- ❌ Wildcard certificates
- ❌ IP-based SSL binding
- ❌ Export the certificate
- ❌ Use with root (naked) domains directly
- ❌ Use in Isolated tier (ASE)
- ❌ Private DNS zones

**Free Managed Certificate — What You CAN Do:**
- ✅ SNI SSL binding
- ✅ Custom subdomains
- ✅ Auto-renewal
- ✅ Standard tier and above

---

## MODULE 9: ADVANCED NETWORKING DEEP DIVE

### 9.1 Access Restrictions (Deep Dive)

**Rule Processing:**
- Rules processed in **priority order** (lowest number = highest priority)
- If no rules match, default is **deny all** (when any allow rules exist) or **allow all** (when no rules exist)
- Rules apply to the **main site** — SCM/Kudu site has separate rules

**Restricting SCM Site Separately:**
```bash
# Set SCM access restrictions (independent of main site)
az webapp config access-restriction add \
  --name myapp \
  --resource-group myRG \
  --rule-name "AllowDevOps" \
  --action Allow \
  --ip-address 10.0.0.0/24 \
  --priority 100 \
  --scm-site true

# Or make SCM use the same rules as main site
az webapp config access-restriction set \
  --name myapp \
  --resource-group myRG \
  --use-same-restrictions-for-scm-site true
```

**Service Tag Restrictions:**
```bash
# Allow only from Azure Front Door
az webapp config access-restriction add \
  --name myapp \
  --resource-group myRG \
  --rule-name "AllowFrontDoor" \
  --action Allow \
  --service-tag AzureFrontDoor.Backend \
  --priority 100
```

### 9.2 VNet Integration (Deep Dive)

**Regional VNet Integration:**
- App can reach resources in the same-region VNet
- Requires an **empty, delegated subnet** (`Microsoft.Web/serverFarms`)
- Standard tier and above
- Does NOT give inbound access to the app

**Configuration:**
```bash
# Integrate with VNet
az webapp vnet-integration add \
  --name myapp \
  --resource-group myRG \
  --vnet myVNet \
  --subnet mySubnet

# List VNet integrations
az webapp vnet-integration list \
  --name myapp \
  --resource-group myRG

# Remove VNet integration
az webapp vnet-integration remove \
  --name myapp \
  --resource-group myRG
```

**Route All Traffic Through VNet:**
```bash
# By default, only RFC 1918 private traffic goes through VNet
# To route ALL traffic (including internet) through VNet:
az webapp config appsettings set \
  --name myapp \
  --resource-group myRG \
  --settings WEBSITE_VNET_ROUTE_ALL=1
```

**Key Point:** Without `WEBSITE_VNET_ROUTE_ALL=1`, only private IP ranges (10.x, 172.16-31.x, 192.168.x) route through VNet. Public internet calls still go directly from App Service.

**Subnet Requirements:**
- Subnet must be **empty** (no other resources)
- Subnet must be **delegated** to `Microsoft.Web/serverFarms`
- Minimum **/28** subnet recommended (provides 16 addresses minus 5 Azure-reserved = 11)
- Each App Service plan instance uses one IP from the subnet

### 9.3 Private Endpoints (Deep Dive)

**What Private Endpoints Provide:**
- App gets a **private IP** address in your VNet
- App accessible **only** from within VNet (or connected networks)
- Public endpoint becomes **inaccessible**
- Requires **Premium v2+** or Isolated tier

```bash
# Create private endpoint
az network private-endpoint create \
  --name myPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet mySubnet \
  --private-connection-resource-id <app-resource-id> \
  --group-id sites \
  --connection-name myConnection

# Create private DNS zone (required for name resolution)
az network private-dns zone create \
  --resource-group myRG \
  --name privatelink.azurewebsites.net

# Link DNS zone to VNet
az network private-dns zone virtual-network-link create \
  --resource-group myRG \
  --zone-name privatelink.azurewebsites.net \
  --name myDNSLink \
  --virtual-network myVNet \
  --registration-enabled false
```

**VNet Integration vs Private Endpoint:**

| Feature | VNet Integration | Private Endpoint |
|---------|-----------------|------------------|
| **Direction** | Outbound (app → VNet) | Inbound (VNet → app) |
| **Purpose** | App calls VNet resources | Users/services call app privately |
| **Public access** | Still accessible publicly | Public access blocked |
| **Minimum tier** | Standard | Premium v2 |
| **Use together** | ✅ Yes, commonly combined | ✅ Yes |

### 9.4 App Service Environment v3 (ASE v3)

**What is ASE v3?**
- Single-tenant App Service deployment
- Runs in YOUR VNet
- Complete network isolation
- Dedicated compute (not shared with others)
- Used for highest security/compliance requirements

**ASE v3 Types:**

| Type | Description |
|------|-------------|
| **External** | Has public-facing IP, apps accessible from internet |
| **Internal (ILB)** | Internal Load Balancer, apps only accessible from within VNet |

**Key ASE v3 Facts:**
- No separate stamp fee (unlike v1/v2)
- Uses Isolated v2 pricing tier
- Minimum 1 instance per plan (charged even if no apps)
- Supports zone redundancy
- Can have up to 100+ instances per plan
- Supports Private Endpoints (nested)

### 9.5 Hybrid Connections (Deep Dive)

**How Hybrid Connections Work:**
1. Install **Hybrid Connection Manager (HCM)** agent on on-premises machine
2. HCM establishes outbound connection to Azure Relay
3. App Service routes traffic through Azure Relay to on-premises
4. Supports **TCP only** (no UDP)
5. Targets specific **host:port** combination

**Configuration:**
```bash
# Create Hybrid Connection
az webapp hybrid-connection add \
  --name myapp \
  --resource-group myRG \
  --namespace <relay-namespace> \
  --hybrid-connection <connection-name>
```

**Key Points:**
- Available in Basic tier and above
- Each connection targets ONE host:port
- Multiple connections supported per app
- HCM can run on Windows Server 2012+ or Windows 10+
- Port 443 outbound required from HCM machine
- No firewall changes needed on-premises (outbound only)

### 9.6 Static Outbound IP with NAT Gateway

**Problem:** App Service outbound IPs can change when scaling across tiers.

**Solution:** Use NAT Gateway for predictable, static outbound IP.

```bash
# Create NAT Gateway
az network nat gateway create \
  --name myNatGateway \
  --resource-group myRG \
  --public-ip-addresses myPublicIP \
  --location eastus

# Associate with VNet Integration subnet
az network vnet subnet update \
  --name mySubnet \
  --vnet-name myVNet \
  --resource-group myRG \
  --nat-gateway myNatGateway
```

**Requires:** VNet Integration + `WEBSITE_VNET_ROUTE_ALL=1`

---

## MODULE 10: IMPORTANT APP SETTINGS, mTLS & MISCELLANEOUS

### 10.1 Critical App Settings Reference

| Setting | Description | Values |
|---------|-------------|--------|
| `WEBSITE_RUN_FROM_PACKAGE` | Run from mounted ZIP | `0`, `1`, or URL |
| `WEBSITE_LOCAL_CACHE_OPTION` | Enable local cache | `Default`, `Always` |
| `WEBSITE_LOCAL_CACHE_SIZEINMB` | Local cache size | 300-2000 MB |
| `WEBSITE_TIME_ZONE` | Set app timezone | TZ name (e.g., `Eastern Standard Time`) |
| `WEBSITE_SWAP_WARMUP_PING_PATH` | Warm-up path for swaps | URL path (e.g., `/health`) |
| `WEBSITE_SWAP_WARMUP_PING_STATUSES` | Accepted warm-up status codes | `200,202` |
| `WEBSITE_AUTO_SWAP_SLOT_NAME` | Target slot for auto swap | Slot name (e.g., `production`) |
| `WEBSITE_VNET_ROUTE_ALL` | Route all traffic through VNet | `0` or `1` |
| `WEBSITE_CONTAINER_START_TIME_LIMIT` | Container startup timeout | Seconds (max 1800) |
| `WEBSITES_PORT` | Container listening port | Port number |
| `WEBSITES_ENABLE_APP_SERVICE_STORAGE` | Persistent storage for containers | `true` / `false` |
| `SCM_DO_BUILD_DURING_DEPLOYMENT` | Run Oryx build during deploy | `true` / `false` |
| `ENABLE_ORYX_BUILD` | Enable Oryx build system | `true` / `false` |
| `DOCKER_REGISTRY_SERVER_URL` | Container registry URL | URL |
| `DOCKER_REGISTRY_SERVER_USERNAME` | Registry username | String |
| `DOCKER_REGISTRY_SERVER_PASSWORD` | Registry password | String |
| `DOCKER_ENABLE_CI` | Container CI/CD webhook | `true` / `false` |
| `PRE_BUILD_COMMAND` | Run before Oryx build | Command string |
| `POST_BUILD_COMMAND` | Run after Oryx build | Command string |
| `APPLICATIONINSIGHTS_CONNECTION_STRING` | App Insights connection | Connection string |
| `ASPNETCORE_ENVIRONMENT` | .NET environment | `Development`, `Staging`, `Production` |

### 10.2 Client Certificate Authentication (mTLS)

**What is mTLS?**
- Mutual TLS — both client and server authenticate with certificates
- Client must present a valid certificate to access the app
- App Service validates the client certificate

**Configuration:**
```bash
# Enable client certificates (required for all requests)
az webapp update \
  --name myapp \
  --resource-group myRG \
  --set clientCertEnabled=true

# Set client certificate mode
az webapp update \
  --name myapp \
  --resource-group myRG \
  --set clientCertMode=Required  # Required | Optional | OptionalInteractiveUser
```

**Client Certificate Modes:**

| Mode | Behavior |
|------|----------|
| `Required` | All requests MUST include a valid client certificate |
| `Optional` | Certificate forwarded if present, but not required |
| `OptionalInteractiveUser` | Required for browser requests, optional for API calls |

**Accessing Client Certificate in Code:**
```csharp
// ASP.NET Core
[HttpGet("/api/secure")]
public IActionResult SecureEndpoint()
{
    // Get client certificate from request header
    var certHeader = Request.Headers["X-ARR-ClientCert"].FirstOrDefault();
    
    if (string.IsNullOrEmpty(certHeader))
        return Unauthorized("No client certificate provided");
    
    var certBytes = Convert.FromBase64String(certHeader);
    var certificate = new X509Certificate2(certBytes);
    
    // Validate certificate
    if (certificate.Thumbprint != "EXPECTED_THUMBPRINT")
        return Unauthorized("Invalid certificate");
    
    // Validate expiration
    if (certificate.NotAfter < DateTime.UtcNow)
        return Unauthorized("Certificate expired");
    
    return Ok($"Authenticated: {certificate.Subject}");
}
```

**Key Points:**
- Certificate is forwarded in `X-ARR-ClientCert` HTTP header (Base64-encoded)
- App Service terminates TLS — your app receives the cert as a header, not in the TLS handshake
- You must validate the certificate in your code (thumbprint, issuer, expiration)
- Exclude specific paths from requiring certificates using `clientCertExclusionPaths`

```bash
# Exclude health check from requiring client cert
az webapp update \
  --name myapp \
  --resource-group myRG \
  --set clientCertExclusionPaths="/health;/api/public"
```

### 10.3 Custom Domains (Deep Dive)

**Steps to Configure Custom Domain:**

1. **Verify domain ownership** (create validation record):
```bash
# Create TXT record for verification
# Host: asuid.<subdomain>  or  asuid (for apex domain)
# Value: Custom Domain Verification ID (from portal)
```

2. **Create DNS records:**
```
# For subdomain (www.example.com):
CNAME www → myapp.azurewebsites.net

# For apex domain (example.com):
A record → <app-outbound-IP>
TXT asuid → <verification-id>

# OR use alias record (recommended for apex):
ALIAS/ANAME → myapp.azurewebsites.net
```

3. **Add domain in App Service:**
```bash
az webapp config hostname add \
  --webapp-name myapp \
  --resource-group myRG \
  --hostname "www.example.com"
```

4. **Bind SSL certificate** (required for HTTPS)

**Domain Verification Methods:**
- **TXT record**: `asuid.<domain>` → verification ID
- **CNAME record**: `<domain>` → `<app>.azurewebsites.net` (implicitly verifies)
- **A record + TXT**: For apex domains

### 10.4 Outbound IP Addresses

**Viewing Outbound IPs:**
```bash
# Current outbound IPs
az webapp show --name myapp --resource-group myRG \
  --query outboundIpAddresses --output tsv

# ALL possible outbound IPs (superset)
az webapp show --name myapp --resource-group myRG \
  --query possibleOutboundIpAddresses --output tsv
```

**When Outbound IPs Change:**
- Scaling to a different tier (e.g., Standard → Premium)
- Moving app to a different App Service plan in a different scale unit
- The underlying infrastructure redeployment

**When Outbound IPs DON'T Change:**
- Scaling instances within the same tier
- Restarting the app
- Deploying code

**For Static Outbound IP:**
- Use **NAT Gateway** with VNet integration
- Use **App Service Environment** (dedicated IPs)
- Use **Azure Front Door** or **Application Gateway**

### 10.5 App Service on Linux Specifics

**Key Differences from Windows:**

| Aspect | Windows | Linux |
|--------|---------|-------|
| **Runtime** | .NET Framework, .NET Core, Node.js, Java, PHP, Python | .NET Core, Node.js, Java, PHP, Python, Ruby, Go |
| **Custom containers** | Sidecar pattern (preview) | Full container support |
| **WebJobs** | ✅ Supported | ❌ Not supported (use Azure Functions) |
| **Auto swap** | ✅ Supported | ❌ Not supported |
| **Remote debugging** | ✅ Supported | ❌ Not supported |
| **Failed request tracing** | ✅ Supported | ❌ Not supported |
| **Detailed error pages** | ✅ Supported | ❌ Not supported |
| **File system** | Windows (NTFS) | Linux (ext4) |
| **SSH** | ❌ Not built-in | ✅ Built-in SSH |

**Linux Startup Commands:**

| Runtime | Startup Command |
|---------|----------------|
| .NET | `dotnet <assembly>.dll` |
| Node.js | `node server.js` or `pm2 start server.js` |
| Python | `gunicorn --bind 0.0.0.0:8000 app:app` |
| Java | Auto-configured based on JAR/WAR |
| PHP | `apache2ctl -D FOREGROUND` |
| Ruby | `bundle exec rails server -p 8080` |

```bash
# Set custom startup command for Linux
az webapp config set \
  --name myapp \
  --resource-group myRG \
  --startup-file "gunicorn --bind 0.0.0.0:8000 --workers 4 app:app"
```

### 10.6 App Service Plan Density & Best Practices

**Multiple Apps Per Plan:**
- All apps share CPU, memory, and disk
- Monitor per-app metrics to detect noisy neighbors
- Move resource-intensive apps to separate plans

**When to Separate Plans:**
- App has different scaling requirements
- App in different geographic region
- App needs different tier features
- You want to isolate costs
- App needs guaranteed resources

**Metric to Watch:** `CpuPercentage`, `MemoryPercentage` per app vs per plan

### 10.7 Azure Front Door / Application Gateway Integration

**Azure Front Door with App Service:**
- Global load balancing and CDN
- SSL offloading
- WAF (Web Application Firewall)
- Lock down App Service to only accept traffic from Front Door:

```bash
# Add access restriction for Front Door
az webapp config access-restriction add \
  --name myapp \
  --resource-group myRG \
  --rule-name "AllowFrontDoor" \
  --action Allow \
  --service-tag AzureFrontDoor.Backend \
  --priority 100 \
  --http-header x-azure-fdid=<front-door-id>
```

**Application Gateway with App Service:**
- Regional load balancing
- WAF protection
- SSL termination
- Commonly used with Private Endpoints

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**1. App Service Plans:**
- Tier differences and features (Free → Isolated)
- When to use each tier
- Scaling capabilities per tier
- Pricing model understanding
- Multiple apps in same plan (resource sharing)

**2. Authentication & Authorization:**
- Easy Auth capabilities
- Identity providers supported
- Authentication flow (with/without SDK)
- Token management
- Slot-specific auth settings

**3. Deployment Slots:**
- Swappable vs non-swappable settings
- Swap process phases (4 phases)
- Auto swap configuration
- Traffic routing (cookie-based)
- Warm-up with applicationInitialization

**4. Scaling:**
- Autoscale vs Automatic Scaling (platform-managed)
- Scale-out (OR logic) vs Scale-in (AND logic)
- Autoscale rules, metrics, cool down
- Instance limits per tier

**5. Configuration:**
- App settings vs connection strings
- Environment variable access (SQLCONNSTR_, CUSTOMCONNSTR_, etc.)
- Slot settings (sticky)
- Key Vault references (`@Microsoft.KeyVault(SecretUri=...)`)
- Managed identities (system vs user-assigned)

**6. Networking:**
- VNet Integration (outbound) vs Private Endpoints (inbound)
- Access restrictions (priority-based, SCM separate)
- Hybrid Connections (TCP only, host:port)
- ASE v3 (internal vs external LB)
- NAT Gateway for static outbound IP
- `WEBSITE_VNET_ROUTE_ALL` setting

**7. Containers:**
- Web App for Containers (Linux)
- ACR integration (Managed Identity preferred)
- `WEBSITES_PORT`, `WEBSITES_ENABLE_APP_SERVICE_STORAGE`
- Multi-container with Docker Compose
- Container CI/CD webhooks
- Container startup timeout

**8. WebJobs:**
- Continuous vs Triggered
- CRON expressions (6-field format)
- WebJobs SDK (triggers, bindings)
- WebJobs vs Azure Functions
- `settings.job` file
- Always On requirement

**9. Deployment Methods:**
- ZIP Deploy, Run from Package, Local Git, GitHub Actions
- `WEBSITE_RUN_FROM_PACKAGE` (0, 1, URL)
- Kudu/SCM site and REST API
- Oryx build system (`SCM_DO_BUILD_DURING_DEPLOYMENT`)
- Deployment credentials (user-level vs app-level)

**10. CORS, Health Check, Backup:**
- App Service CORS vs app-level CORS
- Health Check path, instance removal, replacement
- Backup & Restore (10 GB limit, Standard tier+)
- Local Cache (`WEBSITE_LOCAL_CACHE_OPTION`)

**11. Security:**
- Client certificates / mTLS (`X-ARR-ClientCert` header)
- TLS/SSL (SNI vs IP-based)
- Managed certificates (limitations)
- Custom domains (CNAME, A record, TXT verification)

**12. Diagnostic Logging:**
- Types: Application, Web Server, Detailed Error, Failed Request, Deployment
- Storage: File system, Blob storage
- Streaming logs (`az webapp log tail`)
- Application Insights integration

### Sample Exam Questions

#### App Service Plans

**Q1:** You need to deploy a web application that requires deployment slots for staging. What is the minimum App Service plan tier required?
- A) Free
- B) Shared
- C) Basic
- D) Standard

**Answer: D) Standard**
Explanation: Deployment slots are available starting from the Standard tier. Free, Shared, and Basic tiers do not support deployment slots.

---

**Q2:** You have an App Service plan in the Basic tier with 3 web apps. You want to enable autoscale. What should you do?
- A) Configure autoscale rules in the portal
- B) Upgrade to Standard tier or higher
- C) Enable automatic scaling
- D) Create separate App Service plans

**Answer: B) Upgrade to Standard tier or higher**
Explanation: Autoscale is not available in the Basic tier. You must upgrade to Standard, Premium, or Isolated tier.

---

**Q3:** How many deployment slots are available in the Premium tier?
- A) 0
- B) 5
- C) 10
- D) 20

**Answer: D) 20**
Explanation: Premium and Isolated tiers support up to 20 deployment slots. Standard supports 5 slots.

---

**Q4:** You have 5 apps in the same App Service plan. You scale up to a larger instance size. How does this affect your apps?
- A) Only the primary app gets more resources
- B) All apps get more resources
- C) Apps must be reconfigured individually
- D) Apps are automatically moved to separate plans

**Answer: B) All apps get more resources**
Explanation: All apps in an App Service plan share the same compute resources. Scaling up affects all apps in the plan.

---

#### Authentication & Authorization

**Q5:** Which authentication flow does NOT require the provider's SDK?
- A) Server-directed flow
- B) Client-directed flow
- C) Token-based flow
- D) Certificate-based flow

**Answer: A) Server-directed flow**
Explanation: In server-directed flow (without provider SDK), the application delegates sign-in to App Service, which redirects to the provider's login page.

---

**Q6:** Where are authentication tokens stored in App Service?
- A) Application memory
- B) Token store associated with the app
- C) Azure Key Vault
- D) Browser local storage

**Answer: B) Token store associated with the app**
Explanation: App Service maintains a token store that caches tokens associated with users of your app.

---

**Q7:** What HTTP header contains the authentication token for mobile clients using Easy Auth?
- A) Authorization
- B) X-MS-TOKEN
- C) X-ZUMO-AUTH
- D) Bearer

**Answer: C) X-ZUMO-AUTH**
Explanation: Mobile clients receive a JWT (JSON Web Token) that should be presented in the X-ZUMO-AUTH header.

---

**Q8:** Which identity provider is recommended for authenticating internal organizational users?
- A) Facebook
- B) Google
- C) Microsoft Entra ID
- D) Twitter

**Answer: C) Microsoft Entra ID**
Explanation: Microsoft Entra ID (formerly Azure AD) is ideal for authenticating employees and internal users within an organization.

---

#### Deployment Slots

**Q9:** Which of the following settings swaps when you swap deployment slots?
- A) Custom domain names
- B) App settings (not marked as slot setting)
- C) Publishing endpoints
- D) Scale settings

**Answer: B) App settings (not marked as slot setting)**
Explanation: App settings swap unless explicitly marked as "Deployment slot setting". Custom domains, publishing endpoints, and scale settings are slot-specific and don't swap.

---

**Q10:** You want to test changes in a production-like environment before swapping to production. What should you use?
- A) Standard swap
- B) Swap with preview
- C) Auto swap
- D) Clone slot

**Answer: B) Swap with preview**
Explanation: Swap with preview is a multi-phase swap that allows you to validate changes in the source slot with production settings before completing the swap.

---

**Q11:** After deploying code to the staging slot, you want the slot to automatically swap to production. What should you configure?
- A) Continuous deployment
- B) Auto swap
- C) Scheduled swap
- D) Preview swap

**Answer: B) Auto swap**
Explanation: Auto swap automatically swaps the slot to production after a successful deployment.

---

**Q12:** What happens to the previous production app after a swap?
- A) It is deleted
- B) It remains in the production slot
- C) It is moved to the staging slot
- D) It is archived

**Answer: C) It is moved to the staging slot**
Explanation: During a swap, the apps exchange slots. The previous production app is now in the staging slot, allowing for easy rollback.

---

**Q13:** Which setting should be marked as "Deployment slot setting" to prevent it from swapping?
- A) Framework version
- B) Database connection string for production
- C) Handler mappings
- D) WebJobs content

**Answer: B) Database connection string for production**
Explanation: Environment-specific settings like database connections should be marked as slot settings (sticky) so staging doesn't accidentally access production databases.

---

#### Scaling

**Q14:** You configure an autoscale rule: "Scale out by 1 instance when CPU > 80% for 10 minutes". What is the cool down period used for?
- A) Prevent immediate scale-in after scale-out
- B) Calculate average CPU
- C) Restart instances
- D) Update metrics

**Answer: A) Prevent immediate scale-in after scale-out**
Explanation: The cool down period prevents the system from scaling again immediately after a scale operation, allowing metrics to stabilize.

---

**Q15:** You have two autoscale rules for scale-out: one for CPU > 70% and one for Memory > 80%. What happens when CPU is 75% and Memory is 60%?
- A) No scaling occurs
- B) Scale out occurs (OR logic)
- C) Only if both conditions are met (AND logic)
- D) Error occurs

**Answer: B) Scale out occurs (OR logic)**
Explanation: For scale-out, autoscale uses OR logic. If any rule is triggered, scale-out occurs.

---

**Q16:** You have two autoscale rules for scale-in: one for CPU < 30% and one for Memory < 40%. What must happen for scale-in?
- A) Either condition (OR logic)
- B) Both conditions (AND logic)
- C) Only CPU matters
- D) Only Memory matters

**Answer: B) Both conditions (AND logic)**
Explanation: For scale-in, autoscale uses AND logic. All scale-in rules must be satisfied before scaling in.

---

**Q17:** What is the difference between "Scale up" and "Scale out"?
- A) Scale up increases instances, scale out increases resources per instance
- B) Scale up increases resources per instance, scale out increases instances
- C) They are the same thing
- D) Scale up is automatic, scale out is manual

**Answer: B) Scale up increases resources per instance, scale out increases instances**
Explanation: Scale up (vertical scaling) changes to a higher tier with more CPU/RAM per instance. Scale out (horizontal scaling) adds more instances.

---

**Q18:** Which tier supports Automatic Scaling (platform-managed)?
- A) Standard
- B) Basic
- C) Premium v3
- D) Free

**Answer: C) Premium v3**
Explanation: Automatic Scaling is available in Premium v2, Premium v3, and Premium v4 tiers only.

---

#### Configuration

**Q19:** How do you access an app setting named "MyApiKey" in an ASP.NET Core application?
- A) ConfigurationManager.AppSettings["MyApiKey"]
- B) _configuration["MyApiKey"]
- C) Environment.GetEnvironmentVariable("MyApiKey")
- D) Both B and C

**Answer: D) Both B and C**
Explanation: In ASP.NET Core, you can access app settings through IConfiguration or as environment variables.

---

**Q20:** What is the syntax to reference an Azure Key Vault secret in an app setting?
- A) keyvault://<vault>/<secret>
- B) @KeyVault(SecretUri=<uri>)
- C) @Microsoft.KeyVault(SecretUri=<uri>)
- D) vault://<vault>/<secret>

**Answer: C) @Microsoft.KeyVault(SecretUri=<uri>)**
Explanation: The correct syntax is @Microsoft.KeyVault(SecretUri=<secret-uri>).

---

**Q21:** You want to enable Always On for your web app. What is the minimum tier required?
- A) Free
- B) Shared
- C) Basic
- D) Standard

**Answer: C) Basic**
Explanation: Always On is available in Basic tier and above. It's not available in Free and Shared tiers.

---

**Q22:** Which type of certificate supports wildcard domains in App Service?
- A) App Service Managed Certificate
- B) App Service Certificate (purchased)
- C) Both A and B
- D) Neither

**Answer: B) App Service Certificate (purchased)**
Explanation: App Service Managed Certificates do not support wildcard certificates. You must purchase or import a certificate for wildcard domains.

---

#### Networking

**Q23:** Which feature allows your app to make outbound calls into an Azure VNet?
- A) Access restrictions
- B) Private endpoint
- C) VNet integration
- D) Service endpoint

**Answer: C) VNet integration**
Explanation: VNet integration allows your app to make outbound calls to resources in a VNet. Private endpoints control inbound access.

---

**Q24:** You want to restrict access to your app to only specific IP addresses. What should you configure?
- A) Private endpoint
- B) VNet integration
- C) Access restrictions (IP restrictions)
- D) Network security group

**Answer: C) Access restrictions (IP restrictions)**
Explanation: Access restrictions (IP restrictions) allow you to define allowed/denied IP address ranges for inbound traffic.

---

**Q25:** What is the minimum tier required for Private Endpoints?
- A) Basic
- B) Standard
- C) Premium v2
- D) Isolated

**Answer: C) Premium v2**
Explanation: Private Endpoints require Premium v2, Premium v3, or Isolated tier.

---

#### Diagnostic Logging

**Q26:** Which log type captures raw HTTP request data?
- A) Application logging
- B) Web server logging
- C) Failed request tracing
- D) Deployment logging

**Answer: B) Web server logging**
Explanation: Web server logging captures raw HTTP request data in W3C Extended Log File Format.

---

**Q27:** Where are logs stored in App Service?
- A) Azure SQL Database
- B) /home/LogFiles directory
- C) Azure Cosmos DB
- D) Table Storage

**Answer: B) /home/LogFiles directory**
Explanation: Logs are stored in the /home/LogFiles directory. They can also be sent to Blob Storage.

---

**Q28:** How do you stream real-time logs in Azure CLI?
- A) az webapp log stream
- B) az webapp log tail
- C) az webapp log show
- D) az webapp log download

**Answer: B) az webapp log tail**
Explanation: The command `az webapp log tail` streams live application logs.

---

#### Advanced Scenarios

**Q29:** You have an app in the Standard tier. You need to scale to 15 instances. What should you do?
- A) Enable autoscale to 15 instances
- B) Upgrade to Premium tier
- C) Create multiple App Service plans
- D) Enable Automatic Scaling

**Answer: B) Upgrade to Premium tier**
Explanation: Standard tier supports up to 10 instances. Premium tier supports up to 20-30 instances depending on the version.

---

**Q30:** Your app requires network isolation and runs in a dedicated environment. Which tier should you use?
- A) Premium v3
- B) Standard
- C) Isolated (App Service Environment)
- D) Basic

**Answer: C) Isolated (App Service Environment)**
Explanation: Isolated tier provides full network isolation with dedicated App Service Environment running in your VNet.

---

**Q31:** You want to route 10% of production traffic to a new feature in staging. What should you configure?
- A) Auto swap
- B) Traffic routing / Testing in Production
- C) Load balancer
- D) Multiple App Service plans

**Answer: B) Traffic routing / Testing in Production**
Explanation: Traffic routing allows you to split traffic percentage-wise between deployment slots.

---

**Q32:** What happens if an instance fails warm-up during a swap with preview?
- A) Swap continues anyway
- B) Swap is aborted
- C) Only healthy instances swap
- D) Instance is restarted

**Answer: B) Swap is aborted**
Explanation: If any instance fails warm-up during a swap with preview, the entire swap operation is aborted.

---

**Q33:** Which authentication method does NOT require code changes in the application?
- A) Custom authentication middleware
- B) OAuth libraries
- C) App Service built-in authentication (Easy Auth)
- D) JWT validation

**Answer: C) App Service built-in authentication (Easy Auth)**
Explanation: Easy Auth is built into the platform and requires no SDKs, specific languages, or code changes.

---

**Q34:** You enabled autoscale with min=2, max=10, default=3. Metrics are unavailable. How many instances will run?
- A) 0
- B) 2
- C) 3
- D) 10

**Answer: C) 3**
Explanation: When metrics are unavailable, autoscale uses the default instance count.

---

**Q35:** Your app uses session state stored in-memory. What should you configure?
- A) Disable ARR Affinity
- B) Enable ARR Affinity
- C) Use Redis Cache for session state
- D) Enable Always On

**Answer: C) Use Redis Cache for session state**
Explanation: In-memory session state doesn't work well with multiple instances. Best practice is to use external session state like Redis Cache, and disable ARR Affinity for better scaling.

---

#### Containers & Docker

**Q36:** You deploy a custom container that listens on port 3000. The app returns a 502 error. What should you configure?
- A) `WEBSITE_RUN_FROM_PACKAGE=1`
- B) `WEBSITES_PORT=3000`
- C) `DOCKER_REGISTRY_SERVER_URL`
- D) `WEBSITE_LOCAL_CACHE_OPTION=Always`

**Answer: B) `WEBSITES_PORT=3000`**
Explanation: If your container listens on a port other than 80 or 8080, you must set `WEBSITES_PORT` to tell App Service which port to forward to.

---

**Q37:** You want to use Managed Identity to pull images from Azure Container Registry. Which RBAC role must you assign?
- A) Contributor
- B) Reader
- C) AcrPull
- D) AcrPush

**Answer: C) AcrPull**
Explanation: The `AcrPull` role grants permission to pull images from ACR. `AcrPush` is for pushing images, which is not needed by the App Service.

---

**Q38:** What is the maximum number of containers supported in a multi-container App Service using Docker Compose?
- A) 2
- B) 4
- C) 8
- D) Unlimited

**Answer: B) 4**
Explanation: Multi-container apps in App Service are limited to 4 containers per app. Only Docker Compose is supported (not Kubernetes).

---

**Q39:** Your custom container takes 5 minutes to start. The default timeout causes it to restart before fully loading. What should you do?
- A) Set `WEBSITE_RUN_FROM_PACKAGE=1`
- B) Set `WEBSITES_CONTAINER_START_TIME_LIMIT=600`
- C) Enable Always On
- D) Scale to Premium tier

**Answer: B) `WEBSITES_CONTAINER_START_TIME_LIMIT=600`**
Explanation: The default container startup timeout is 230 seconds. You can increase it up to 1800 seconds (30 minutes) to allow slow-starting containers to complete initialization.

---

#### WebJobs

**Q40:** You need a background task that processes queue messages continuously alongside your web app. Which is the BEST option?
- A) Azure Functions Consumption plan
- B) Continuous WebJob with WebJobs SDK
- C) Triggered WebJob
- D) Logic App

**Answer: B) Continuous WebJob with WebJobs SDK**
Explanation: Continuous WebJobs run alongside your web app on the same VM at no additional cost. With WebJobs SDK, you get queue trigger support similar to Azure Functions.

---

**Q41:** You create a triggered WebJob with a CRON schedule. The job never executes. What is the most likely cause?
- A) The app is in Premium tier
- B) Always On is not enabled
- C) The CRON expression is wrong
- D) WebJobs require separate App Service plan

**Answer: B) Always On is not enabled**
Explanation: CRON-scheduled triggered WebJobs require Always On to be enabled (Basic tier+). Without it, the app may unload after idle timeout, and the scheduler won't fire.

---

**Q42:** What is the correct CRON expression for "every weekday at 9:00 AM" in Azure (6-field format)?
- A) `0 9 * * 1-5`
- B) `0 0 9 * * 1-5`
- C) `* * 9 * * MON-FRI`
- D) `0 0 9 * * *`

**Answer: B) `0 0 9 * * 1-5`**
Explanation: Azure uses 6-field CRON: `{second} {minute} {hour} {day} {month} {day-of-week}`. `0 0 9 * * 1-5` = second 0, minute 0, hour 9, every day, every month, Monday-Friday.

---

**Q43:** WebJobs are NOT supported on which platform?
- A) Windows App Service
- B) Linux App Service
- C) Premium tier
- D) Standard tier

**Answer: B) Linux App Service**
Explanation: WebJobs are NOT supported on Linux App Service. For background processing on Linux, use Azure Functions or container-based solutions.

---

#### Advanced Deployment

**Q44:** You set `WEBSITE_RUN_FROM_PACKAGE=1` and deploy a ZIP. What is the behavior of the wwwroot directory?
- A) Files are mutable (read-write)
- B) Files are read-only (mounted from ZIP)
- C) Files are cached to local disk
- D) Files are stored in Azure Blob Storage

**Answer: B) Files are read-only (mounted from ZIP)**
Explanation: `WEBSITE_RUN_FROM_PACKAGE=1` mounts the deployment package directly as the wwwroot directory, making it read-only. The app cannot write to its own directory.

---

**Q45:** Which deployment method provides the FASTEST cold starts?
- A) ZIP Deploy with `WEBSITE_RUN_FROM_PACKAGE=0`
- B) FTP deployment
- C) Run from Package (`WEBSITE_RUN_FROM_PACKAGE=1`)
- D) Local Git deployment

**Answer: C) Run from Package (`WEBSITE_RUN_FROM_PACKAGE=1`)**
Explanation: Run from Package is the fastest because the ZIP is mounted directly without extraction. It also provides atomic deployments and eliminates file locking issues.

---

**Q46:** What is the URL format for the Kudu/SCM site of an App Service?
- A) `https://<app-name>.kudu.azurewebsites.net`
- B) `https://<app-name>.scm.azurewebsites.net`
- C) `https://scm.<app-name>.azurewebsites.net`
- D) `https://<app-name>.debug.azurewebsites.net`

**Answer: B) `https://<app-name>.scm.azurewebsites.net`**
Explanation: Kudu (SCM site) is always accessible at `<app-name>.scm.azurewebsites.net`. It provides deployment tools, file browser, console, and REST APIs.

---

**Q47:** You want a deployment to automatically build your Node.js application (run `npm install`). Which setting enables this?
- A) `WEBSITE_RUN_FROM_PACKAGE=1`
- B) `SCM_DO_BUILD_DURING_DEPLOYMENT=true`
- C) `ENABLE_AUTO_BUILD=true`
- D) `NODE_BUILD_ENABLED=true`

**Answer: B) `SCM_DO_BUILD_DURING_DEPLOYMENT=true`**
Explanation: Setting `SCM_DO_BUILD_DURING_DEPLOYMENT=true` enables the Oryx build system, which auto-detects your language and runs the appropriate build (e.g., `npm install` for Node.js).

---

#### CORS, Health Check, Backup

**Q48:** You configure CORS in both App Service settings and ASP.NET Core middleware. What happens?
- A) Both CORS configurations are merged
- B) App Service CORS takes precedence, app-level is ignored
- C) App-level CORS takes precedence
- D) An error occurs

**Answer: B) App Service CORS takes precedence, app-level is ignored**
Explanation: When App Service CORS is configured, it takes precedence over application-level CORS. You should use one or the other, not both.

---

**Q49:** An App Service Health Check is configured. An instance fails health checks. When does App Service REPLACE the instance?
- A) After 5 consecutive failures
- B) After the instance is unhealthy for 10 minutes
- C) After the instance is unhealthy for 1 hour (only if 2+ instances)
- D) Immediately

**Answer: C) After the instance is unhealthy for 1 hour (only if 2+ instances)**
Explanation: After 5 consecutive failures, the instance is removed from load balancing. If it stays unhealthy for 1 hour AND there are 2+ instances, App Service replaces it. A single instance is never replaced.

---

**Q50:** What is the maximum backup size for App Service Backup & Restore?
- A) 1 GB
- B) 5 GB
- C) 10 GB
- D) 50 GB

**Answer: C) 10 GB**
Explanation: App Service backups are limited to 10 GB total (app content + database). If the backup exceeds this limit, it fails.

---

**Q51:** You want to improve file I/O performance for a read-heavy App Service. Which setting should you configure?
- A) `WEBSITE_RUN_FROM_PACKAGE=1`
- B) `WEBSITE_LOCAL_CACHE_OPTION=Always`
- C) `WEBSITE_VNET_ROUTE_ALL=1`
- D) Enable Always On

**Answer: B) `WEBSITE_LOCAL_CACHE_OPTION=Always`**
Explanation: Local cache copies site content to the local VM disk, providing faster I/O for read-heavy workloads. However, changes written to local cache are lost on restart.

---

#### Advanced Networking

**Q52:** You want your app to make outbound calls to resources in a VNet AND route ALL traffic (including internet) through the VNet. What must you configure?
- A) VNet Integration only
- B) VNet Integration + `WEBSITE_VNET_ROUTE_ALL=1`
- C) Private Endpoint
- D) App Service Environment

**Answer: B) VNet Integration + `WEBSITE_VNET_ROUTE_ALL=1`**
Explanation: VNet Integration alone only routes RFC 1918 private traffic through the VNet. Setting `WEBSITE_VNET_ROUTE_ALL=1` routes ALL traffic (including public internet calls) through the VNet.

---

**Q53:** Which feature makes your app accessible ONLY from within a VNet (blocks public access)?
- A) VNet Integration
- B) Access Restrictions
- C) Private Endpoint
- D) Service Endpoint

**Answer: C) Private Endpoint**
Explanation: Private Endpoints give your app a private IP in the VNet and block all public access. VNet Integration is for outbound traffic only. Access Restrictions limit but don't completely replace the public endpoint.

---

**Q54:** You need predictable static outbound IP addresses for your App Service. What should you implement?
- A) Premium tier
- B) VNet Integration + NAT Gateway
- C) App Service Managed Certificate
- D) Dedicated outbound IP setting

**Answer: B) VNet Integration + NAT Gateway**
Explanation: A NAT Gateway provides a static outbound IP. Combined with VNet Integration and `WEBSITE_VNET_ROUTE_ALL=1`, all outbound traffic uses the NAT Gateway's static IP.

---

**Q55:** What is the minimum subnet size recommended for VNet Integration?
- A) /30 (4 addresses)
- B) /29 (8 addresses)
- C) /28 (16 addresses)
- D) /24 (256 addresses)

**Answer: C) /28 (16 addresses)**
Explanation: A /28 subnet provides 16 addresses minus 5 Azure-reserved = 11 usable addresses. Each App Service plan instance uses one IP from the subnet.

---

#### Security & mTLS

**Q56:** You enable client certificate authentication. In which HTTP header does App Service forward the client certificate?
- A) `Authorization`
- B) `X-Client-Certificate`
- C) `X-ARR-ClientCert`
- D) `X-Forwarded-Cert`

**Answer: C) `X-ARR-ClientCert`**
Explanation: App Service terminates TLS and forwards the client certificate in the `X-ARR-ClientCert` header as a Base64-encoded string. Your code must read and validate it.

---

**Q57:** You want to require client certificates for API calls but NOT for browser-based login pages. Which `clientCertMode` should you use?
- A) Required
- B) Optional
- C) OptionalInteractiveUser
- D) Disabled

**Answer: C) OptionalInteractiveUser**
Explanation: `OptionalInteractiveUser` requires certificates for browser requests (interactive users) but makes them optional for API calls. However, if you want the opposite (required for API, not browser), you should use `Optional` and validate in code, or use `clientCertExclusionPaths`.

---

**Q58:** Which type of App Service certificate does NOT support wildcard domains?
- A) App Service Certificate (purchased)
- B) Imported PFX certificate
- C) App Service Managed Certificate (free)
- D) All support wildcard domains

**Answer: C) App Service Managed Certificate (free)**
Explanation: Free Managed Certificates do NOT support wildcard domains, IP-based SSL, or certificate export. You need a purchased or imported certificate for wildcards.

---

#### Linux & Miscellaneous

**Q59:** Which feature is supported on Windows App Service but NOT on Linux?
- A) Deployment slots
- B) Custom containers
- C) Auto swap
- D) VNet integration

**Answer: C) Auto swap**
Explanation: Auto swap is NOT supported on Linux or container-based App Service. It's only available on Windows. You must manually swap or use CI/CD pipelines on Linux.

---

**Q60:** What password must be configured for SSH into a custom Linux container on App Service?
- A) Any password you choose
- B) `Docker!` (with exclamation mark)
- C) No password needed
- D) The app deployment password

**Answer: B) `Docker!` (with exclamation mark)**
Explanation: App Service requires the root SSH password to be `Docker!` for custom containers. The SSH port is always 2222.

---

**Q61:** You need to set the timezone for your App Service. Which setting should you use?
- A) `TZ`
- B) `WEBSITE_TIME_ZONE`
- C) `APP_TIMEZONE`
- D) `AZURE_TIMEZONE`

**Answer: B) `WEBSITE_TIME_ZONE`**
Explanation: Set `WEBSITE_TIME_ZONE` to configure the timezone (e.g., `Eastern Standard Time` for Windows, or `America/New_York` for Linux). This affects CRON scheduling and time-based operations.

---

**Q62:** Your App Service's outbound IP addresses changed after a tier change. Which CLI command shows ALL possible outbound IPs?
- A) `az webapp show --query outboundIpAddresses`
- B) `az webapp show --query possibleOutboundIpAddresses`
- C) `az webapp config show --query ipAddresses`
- D) `az webapp networking show`

**Answer: B) `az webapp show --query possibleOutboundIpAddresses`**
Explanation: `possibleOutboundIpAddresses` returns ALL IPs that could potentially be used, regardless of current tier. `outboundIpAddresses` only shows the current set.

---

**Q63:** You configure access restrictions on the main site. Does this also restrict access to the Kudu/SCM site?
- A) Yes, automatically applies to both
- B) No, SCM site has separate access restriction rules
- C) SCM site cannot have access restrictions
- D) SCM site is always public

**Answer: B) No, SCM site has separate access restriction rules**
Explanation: By default, the main site and SCM (Kudu) site have SEPARATE access restriction rules. You can configure them independently or set `use-same-restrictions-for-scm-site` to use the same rules.

---

**Q64:** What is the difference between `WEBSITE_RUN_FROM_PACKAGE=1` and `WEBSITE_RUN_FROM_PACKAGE=<URL>`?
- A) They are identical
- B) `=1` runs from local ZIP in SitePackages; `=URL` runs from external blob storage
- C) `=1` enables caching; `=URL` doesn't
- D) `=URL` is faster than `=1`

**Answer: B) `=1` runs from local ZIP in SitePackages; `=URL` runs from external blob storage**
Explanation: `=1` mounts from a ZIP in `d:\home\data\SitePackages`. `=URL` mounts directly from the external blob storage URL. Both make wwwroot read-only. With URL mode, if the blob is deleted, the app breaks.

---

**Q65:** You want to lock down your App Service to ONLY accept traffic from Azure Front Door. What should you configure?
- A) Private Endpoint
- B) Access restriction with `AzureFrontDoor.Backend` service tag + Front Door ID header check
- C) VNet Integration
- D) Managed Identity

**Answer: B) Access restriction with `AzureFrontDoor.Backend` service tag + Front Door ID header check**
Explanation: Use an access restriction rule with the `AzureFrontDoor.Backend` service tag AND verify the `x-azure-fdid` header matches your Front Door instance ID, preventing other Front Door instances from accessing your app.

---

## Key CLI Commands Reference

### App Service Management
```bash
# Create App Service plan
az appservice plan create --name <plan-name> --resource-group <rg> --sku B1

# Create web app
az webapp create --name <app-name> --resource-group <rg> --plan <plan-name>

# Deploy app
az webapp up --name <app-name> --resource-group <rg> --html

# List web apps
az webapp list --resource-group <rg> --output table

# Delete web app
az webapp delete --name <app-name> --resource-group <rg>
```

### Configuration
```bash
# Set app settings
az webapp config appsettings set --name <app-name> --resource-group <rg> \
  --settings KEY=VALUE

# List app settings
az webapp config appsettings list --name <app-name> --resource-group <rg>

# Set connection string
az webapp config connection-string set --name <app-name> --resource-group <rg> \
  --connection-string-type SQLAzure --settings MyDB='<connection-string>'
```

### Deployment Slots
```bash
# Create slot
az webapp deployment slot create --name <app-name> --resource-group <rg> --slot staging

# List slots
az webapp deployment slot list --name <app-name> --resource-group <rg>

# Swap slots
az webapp deployment slot swap --name <app-name> --resource-group <rg> \
  --slot staging --target-slot production

# Delete slot
az webapp deployment slot delete --name <app-name> --resource-group <rg> --slot staging
```

### Scaling
```bash
# Scale up (change tier)
az appservice plan update --name <plan-name> --resource-group <rg> --sku S1

# Scale out (manual)
az appservice plan update --name <plan-name> --resource-group <rg> --number-of-workers 3

# Create autoscale setting
az monitor autoscale create --resource-group <rg> \
  --resource <plan-resource-id> --resource-type Microsoft.Web/serverfarms \
  --name <autoscale-name> --min-count 1 --max-count 10 --count 2

# Add autoscale rule
az monitor autoscale rule create --resource-group <rg> \
  --autoscale-name <autoscale-name> \
  --condition "Percentage CPU > 70 avg 10m" --scale out 1
```

### Logging
```bash
# Enable logging
az webapp log config --name <app-name> --resource-group <rg> \
  --application-logging azureblobstorage --level information

# Stream logs
az webapp log tail --name <app-name> --resource-group <rg>

# Download logs
az webapp log download --name <app-name> --resource-group <rg> \
  --log-file ./logs.zip
```

### Containers
```bash
# Create Linux App Service Plan
az appservice plan create --name <plan> --resource-group <rg> --sku B1 --is-linux

# Create container app from ACR
az webapp create --resource-group <rg> --plan <plan> --name <app> \
  --deployment-container-image-name <acr>.azurecr.io/<image>:<tag>

# Configure container settings
az webapp config container set --name <app> --resource-group <rg> \
  --docker-custom-image-name <acr>.azurecr.io/<image>:<tag> \
  --docker-registry-server-url https://<acr>.azurecr.io

# Enable container CI/CD webhook
az webapp deployment container config --enable-cd true \
  --name <app> --resource-group <rg>

# Get webhook URL
az webapp deployment container show-cd-url --name <app> --resource-group <rg>
```

### Networking
```bash
# Add VNet integration
az webapp vnet-integration add --name <app> --resource-group <rg> \
  --vnet <vnet> --subnet <subnet>

# Add access restriction
az webapp config access-restriction add --name <app> --resource-group <rg> \
  --rule-name "AllowMyIP" --action Allow --ip-address 203.0.113.0/24 --priority 100

# Add access restriction for SCM site
az webapp config access-restriction add --name <app> --resource-group <rg> \
  --rule-name "AllowDevOps" --action Allow --ip-address 10.0.0.0/24 \
  --priority 100 --scm-site true

# Set CORS origins
az webapp cors add --name <app> --resource-group <rg> \
  --allowed-origins "https://example.com"
```

### Health Check & Backup
```bash
# Set health check path
az webapp config set --name <app> --resource-group <rg> \
  --generic-configurations '{"healthCheckPath": "/health"}'

# Create manual backup
az webapp config backup create --resource-group <rg> --webapp-name <app> \
  --container-url "https://<storage>.blob.core.windows.net/backups?sv=..."

# Configure automatic backups
az webapp config backup update --resource-group <rg> --webapp-name <app> \
  --container-url "https://<storage>.blob.core.windows.net/backups?sv=..." \
  --frequency 1d --retain-one true --retention 30
```

### Security
```bash
# Enable client certificates (mTLS)
az webapp update --name <app> --resource-group <rg> \
  --set clientCertEnabled=true --set clientCertMode=Required

# Exclude paths from client cert
az webapp update --name <app> --resource-group <rg> \
  --set clientCertExclusionPaths="/health;/api/public"

# Enable managed identity
az webapp identity assign --name <app> --resource-group <rg>
```

---

## Exam Gotchas & Tricky Distinctions

| # | Common Misconception | Reality |
|---|---------------------|---------|
| 1 | "VNet Integration gives inbound access to my app" | ❌ VNet Integration is **outbound only** (app → VNet). For inbound private access, use **Private Endpoints**. |
| 2 | "Private Endpoints block internet access from the app" | ❌ Private Endpoints block inbound public access TO the app. Outbound internet from the app still works normally. |
| 3 | "Access restrictions on main site also protect Kudu/SCM" | ❌ Main site and SCM site have **separate** access restriction rules by default. You must configure SCM restrictions independently or explicitly use `use-same-restrictions-for-scm-site`. |
| 4 | "Deployment slots have separate VM instances" | ❌ All slots share the **same** App Service plan instances. They share CPU, memory, and disk. |
| 5 | "Scale-out uses AND logic (all rules must be met)" | ❌ Scale-out uses **OR logic** — any single rule triggers scale-out. Scale-**in** uses AND logic (all rules must be true). |
| 6 | "Auto swap works on Linux App Service" | ❌ Auto swap is **NOT supported** on Linux or container-based App Service. Windows only. |
| 7 | "WebJobs work on Linux App Service" | ❌ WebJobs are **NOT supported** on Linux. Use Azure Functions or container-based background processing. |
| 8 | "`WEBSITE_RUN_FROM_PACKAGE=1` allows writing files to wwwroot" | ❌ Run from Package makes wwwroot **read-only**. The ZIP is mounted directly as the filesystem. |
| 9 | "App Service CORS and app-level CORS work together" | ❌ When App Service CORS is configured, it **takes precedence** and app-level CORS middleware is ignored. Use one or the other. |
| 10 | "App Service CORS supports `Access-Control-Allow-Credentials`" | ❌ App Service built-in CORS does **NOT** support the credentials header. For credentialed requests, configure CORS in your application code instead. |
| 11 | "Free Managed Certificate supports wildcard domains" | ❌ Free Managed Certificates do NOT support wildcards, IP-based SSL, or certificate export. Purchase or import a certificate for wildcards. |
| 12 | "Always On is available in Free tier" | ❌ Always On requires **Basic tier** or above. Free and Shared tiers don't support it. |
| 13 | "Health Check replaces an unhealthy single-instance app" | ❌ Instance replacement only happens when there are **2+ instances**. A single instance is removed from load balancing but never replaced (no spare to maintain availability). |
| 14 | "Stopped App Service doesn't incur charges" | ❌ **Stopped apps still incur charges** for the App Service plan (except Free tier). You're paying for the plan, not the app's running state. |
| 15 | "System-assigned Managed Identity swaps with deployment slots" | ❌ System-assigned identities are **slot-specific** and do NOT swap. User-assigned identities DO swap. |
| 16 | "Custom domains swap with deployment slots" | ❌ Custom domain names, SSL certificates, and publishing endpoints are **slot-specific** and do NOT swap. |
| 17 | "VNet Integration routes all traffic through VNet by default" | ❌ By default, only **RFC 1918 private IP** traffic routes through VNet. Set `WEBSITE_VNET_ROUTE_ALL=1` to route ALL traffic. |
| 18 | "Client certificates are validated in the TLS handshake by App Service" | ❌ App Service **terminates TLS** and forwards the client certificate in the `X-ARR-ClientCert` header. Your code must validate it. |
| 19 | "Backup & Restore is available in Basic tier" | ❌ Backup & Restore requires **Standard tier** or above. Maximum backup size is **10 GB**. |
| 20 | "`WEBSITES_PORT` defaults to 8080 for all containers" | ❌ Default is port **80** (some frameworks default to 8080). You MUST set `WEBSITES_PORT` if your container listens on a different port. |
| 21 | "Outbound IPs never change" | ❌ Outbound IPs **can change** when scaling to a different tier or moving to a different scale unit. Use `possibleOutboundIpAddresses` for the full superset. |
| 22 | "Hybrid Connections support UDP" | ❌ Hybrid Connections support **TCP only**. No UDP support. Each connection targets a specific host:port. |
| 23 | "ASE v3 has a stamp fee like v1/v2" | ❌ ASE **v3 eliminated** the stamp fee. You pay only for the Isolated v2 instances you use. |
| 24 | "Deployment slots are available in Basic tier" | ❌ Deployment slots require **Standard tier** or above. Free, Shared, and Basic do NOT support slots. |
| 25 | "Container SSH password can be anything" | ❌ Custom container SSH password MUST be `Docker!` (with exclamation mark) and SSH port is always **2222**. |
| 26 | "Multi-container supports Kubernetes manifests" | ❌ App Service only supports **Docker Compose** for multi-container. Max **4 containers**. No Kubernetes manifests. |
| 27 | "`az webapp log tail` downloads logs" | ❌ `az webapp log tail` **streams** real-time logs. Use `az webapp log download` to download logs. |
| 28 | "Swap with preview skips warm-up" | ❌ Swap with preview **includes warm-up** in Phase 1. If warm-up fails, the swap is aborted. The preview phase lets you validate BEFORE completing. |
| 29 | "Traffic routing percentages apply per-request" | ❌ Traffic routing assigns users to a slot on **first visit** (via `x-ms-routing-name` cookie). Subsequent requests from the same user go to the **same slot**. |
| 30 | "Key Vault references update app settings automatically" | ❌ Key Vault references are resolved at app **start/restart**. Updating a secret in Key Vault requires an **app restart** for the new value to take effect. |
| 31 | "Local cache changes are synced back to Azure Storage" | ❌ Local cache (`WEBSITE_LOCAL_CACHE_OPTION=Always`) creates a **one-way copy**. Changes to local cache are **lost on restart** and NOT synced back. |
| 32 | "ZIP Deploy and Run from Package are the same" | ❌ ZIP Deploy extracts files to wwwroot (mutable). Run from Package mounts the ZIP directly (read-only, faster cold starts, atomic deployment). |
| 33 | "Autoscale is available in Basic tier" | ❌ Autoscale requires **Standard tier** or above. Basic supports manual scaling only. |
| 34 | "Automatic Scaling (platform-managed) works with Standard tier" | ❌ Automatic Scaling requires **Premium v2, v3, or v4** tiers only. Standard tier uses rule-based Azure Autoscale. |
| 35 | "Connection string environment variables have the same format on Windows and Linux" | ❌ Windows uses prefixed format: `SQLCONNSTR_`, `MYSQLCONNSTR_`, `SQLAZURECONNSTR_`, `CUSTOMCONNSTR_`. Linux provides them without prefixes. |

---

## Study Tips for AZ-204 Exam

1. **Understand tier limitations** — Know exactly what each pricing tier supports (slots, scaling, networking, backup)
2. **Practice CLI commands** — Be comfortable with Azure CLI for all App Service operations
3. **Know swap behavior** — Memorize what swaps and what doesn't (system identity, domains, certs DON'T swap)
4. **Master autoscale logic** — Scale-out uses OR, scale-in uses AND, cool down prevents flapping
5. **Authentication flows** — Server-directed (Easy Auth redirects) vs Client-directed (app uses SDK)
6. **VNet Integration vs Private Endpoints** — Outbound vs Inbound; know which requires which tier
7. **Slot settings (sticky)** — Mark environment-specific values (DB connections, API keys) as slot settings
8. **Container deployment** — `WEBSITES_PORT`, `WEBSITES_ENABLE_APP_SERVICE_STORAGE`, ACR with Managed Identity
9. **WebJobs vs Functions** — WebJobs for background tasks with web apps; Functions for independent, event-driven compute
10. **Run from Package** — Read-only wwwroot, faster cold starts, atomic deployment; `=1` vs `=URL`
11. **CORS** — App Service CORS overrides app-level; doesn't support credentials header
12. **Health Check** — Removes unhealthy instances; replaces only with 2+ instances after 1 hour
13. **Client certificates (mTLS)** — Cert in `X-ARR-ClientCert` header; YOU validate in code
14. **Access restrictions** — Main site and SCM have separate rules by default
15. **Build automation** — `SCM_DO_BUILD_DURING_DEPLOYMENT`, Oryx, `PRE/POST_BUILD_COMMAND`

---

## Important URLs and References

- Azure App Service Documentation: https://learn.microsoft.com/en-us/azure/app-service/
- App Service Pricing: https://azure.microsoft.com/en-us/pricing/details/app-service/
- Deployment Slots: https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots
- Autoscale: https://learn.microsoft.com/en-us/azure/azure-monitor/autoscale/autoscale-overview
- Authentication: https://learn.microsoft.com/en-us/azure/app-service/overview-authentication-authorization
- Custom Containers: https://learn.microsoft.com/en-us/azure/app-service/configure-custom-container
- WebJobs: https://learn.microsoft.com/en-us/azure/app-service/webjobs-create
- Health Check: https://learn.microsoft.com/en-us/azure/app-service/monitor-instances-health-check
- VNet Integration: https://learn.microsoft.com/en-us/azure/app-service/overview-vnet-integration
- Client Certificates: https://learn.microsoft.com/en-us/azure/app-service/app-service-web-configure-tls-mutual-auth

---

## Quick Reference Summary

### Pricing Tiers & Features Matrix

| Feature | Free | Shared | Basic | Standard | Premium | Isolated |
|---------|------|--------|-------|----------|---------|----------|
| **Custom domains** | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **SSL** | ❌ | SNI | ✅ | SNI+IP | SNI+IP | SNI+IP |
| **Always On** | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| **Deployment slots** | 0 | 0 | 0 | 5 | 20 | 20 |
| **Autoscale** | ❌ | ❌ | ❌ | ✅ (10) | ✅ (20-30) | ✅ (100+) |
| **Automatic Scaling** | ❌ | ❌ | ❌ | ❌ | ✅ (v2+) | ❌ |
| **VNet integration** | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| **Private endpoints** | ❌ | ❌ | ❌ | ❌ | ✅ (v2+) | ✅ |
| **Hybrid connections** | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| **Backup/Restore** | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| **WebJobs** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Auto swap** | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |

### Deployment Slots — What Swaps vs Doesn't

| Swaps (Moves with Code) | Doesn't Swap (Slot-Specific) |
|--------------------------|------------------------------|
| General settings | Publishing endpoints |
| App settings (unless sticky) | Custom domain names |
| Connection strings (unless sticky) | SSL certificates (non-public) |
| Handler mappings | Scale settings |
| Public certificates | Always On |
| WebJobs content | IP restrictions |
| Hybrid connections | Diagnostic logs |
| Path mappings | CORS settings |
| User-assigned managed identities | VNet integration |
| | System-assigned managed identities |

### Networking Cheat Sheet

| Direction | Feature | Tier Required | Purpose |
|-----------|---------|---------------|---------|
| **Inbound** | Access Restrictions | All | Filter by IP, service tag, VNet |
| **Inbound** | Service Endpoints | Standard+ | Restrict to VNet traffic |
| **Inbound** | Private Endpoints | Premium v2+ | Private IP, block public access |
| **Outbound** | Hybrid Connections | Basic+ | TCP to on-premises host:port |
| **Outbound** | VNet Integration | Standard+ | Access VNet resources |
| **Outbound** | NAT Gateway | Standard+ (with VNet) | Static outbound IP |
| **Both** | ASE v3 | Isolated | Complete network isolation |

### Key App Settings Quick Reference

| Setting | Purpose |
|---------|---------|
| `WEBSITE_RUN_FROM_PACKAGE` | `0`=normal, `1`=local ZIP (read-only), `URL`=remote ZIP |
| `WEBSITE_LOCAL_CACHE_OPTION` | `Always` = local disk copy for fast I/O |
| `WEBSITE_VNET_ROUTE_ALL` | `1` = route ALL traffic through VNet |
| `WEBSITE_TIME_ZONE` | Set timezone for the app |
| `WEBSITE_SWAP_WARMUP_PING_PATH` | Custom warm-up path for slot swaps |
| `WEBSITE_AUTO_SWAP_SLOT_NAME` | Target slot for auto swap |
| `WEBSITES_PORT` | Container listening port |
| `WEBSITES_ENABLE_APP_SERVICE_STORAGE` | Persistent `/home` for containers |
| `WEBSITES_CONTAINER_START_TIME_LIMIT` | Container startup timeout (max 1800s) |
| `SCM_DO_BUILD_DURING_DEPLOYMENT` | Enable Oryx build during deployment |

### Autoscale
- **Scale-out**: OR logic (any rule triggers)
- **Scale-in**: AND logic (all rules must be true)
- **Conflict**: If both trigger simultaneously, **scale-out wins**
- **Cool down**: Prevents flapping; longer for scale-in
- **Default count**: Used when metrics unavailable

### Authentication (Easy Auth)
- **Identity Providers**: Microsoft Entra ID, Facebook, Google, X, Apple, GitHub, OpenID Connect
- **Flows**: Server-directed (Easy Auth redirects) vs Client-directed (app uses SDK)
- **Tokens**: Automatically managed, stored in token store
- **Headers**: `X-MS-CLIENT-PRINCIPAL`, `X-MS-TOKEN-*-ACCESS-TOKEN`
- **Mobile**: JWT in `X-ZUMO-AUTH` header

### WebJobs vs Azure Functions
- **WebJobs**: Same VM, no extra cost, continuous/triggered, Windows only, Always On required
- **Functions**: Independent scaling, pay-per-execution, HTTP triggers, cross-platform, event-driven
