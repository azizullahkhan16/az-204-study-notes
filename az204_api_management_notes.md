# AZ-204: Implement API Management - Complete Study Notes (Exam-Ready Edition)

## Learning Path Overview
This learning path covers implementing Azure API Management with 1 module:
1. Explore API Management

**Level:** Intermediate | **Product:** Azure API Management | **Role:** Developer

> **⚠️ Note:** This document includes topics BEYOND the MS Learn career path — covering Named Values, Groups/Users, Backend Entities, Certificates, VNet Integration, Application Insights/Monitoring, OAuth 2.0 in Developer Portal, Policy Fragments, Validation Policies, Concurrency/Parallelism, Liquid Templates, Azure Functions/Logic Apps Integration, GraphQL APIs, Bicep/ARM, CI/CD, SOAP-to-REST, Multi-Region, Advanced Caching, and comprehensive Context Object reference.

---

## MODULE 1: EXPLORE API MANAGEMENT

### 1.1 Introduction to Azure API Management

**What is Azure API Management (APIM)?**
- Fully managed service for publishing, securing, transforming, and monitoring APIs
- Acts as a facade/gateway in front of backend APIs
- Provides consistent API experience for consumers
- Decouples API implementation from API consumption

**Key Benefits:**
- **Centralized management**: Single place to manage all APIs
- **Security**: Authentication, authorization, rate limiting
- **Transformation**: Modify requests/responses without changing backend
- **Analytics**: Monitor API usage and performance
- **Developer portal**: Self-service API discovery and onboarding
- **Versioning**: Manage multiple API versions
- **Documentation**: Auto-generated API documentation

**Use Cases:**
- Expose backend services as APIs
- Protect APIs from abuse
- Transform legacy services
- Aggregate multiple APIs
- Enable third-party developer access
- Monetize APIs
- Multi-cloud API management

### 1.2 API Management Components

**Three Main Components:**

```
┌─────────────────────────────────────────────────────────────────┐
│                    Azure API Management                          │
├─────────────────────┬─────────────────────┬─────────────────────┤
│   API Gateway       │   Management Plane  │   Developer Portal  │
│   (Runtime)         │   (Admin)           │   (Consumer)        │
├─────────────────────┼─────────────────────┼─────────────────────┤
│ • Request routing   │ • API configuration │ • API discovery     │
│ • Policy execution  │ • Policy management │ • Documentation     │
│ • Authentication    │ • User management   │ • Try-it console    │
│ • Rate limiting     │ • Analytics/reports │ • Subscription mgmt │
│ • Caching           │ • Product setup     │ • Code samples      │
│ • Logging           │ • Subscription mgmt │ • Self-service      │
└─────────────────────┴─────────────────────┴─────────────────────┘
```

#### 1. API Gateway (Runtime)
- Entry point for all API requests
- Routes requests to backend services
- Executes policies
- Handles authentication and authorization
- Enforces usage quotas and rate limits
- Caches responses
- Logs requests for analytics

#### 2. Management Plane (Azure Portal / REST API)
- Configure API Management instance
- Define APIs, operations, policies
- Manage users, groups, subscriptions
- Set up products
- Configure security
- View analytics and reports
- Import APIs (OpenAPI, WSDL, WADL)

#### 3. Developer Portal
- Customizable website for API consumers
- Auto-generated API documentation
- Interactive API console (try APIs)
- Subscription key management
- Usage analytics for developers
- Can be customized or replaced

### 1.3 API Management Service Tiers

| Feature | Consumption | Developer | Basic | Standard | Premium |
|---------|-------------|-----------|-------|----------|---------|
| **SLA** | 99.95% | None | 99.95% | 99.95% | 99.99% |
| **Scale Units** | Auto | 1 | 2 | 4 | 12+ per region |
| **Cache** | External only | 10 MB | 50 MB | 1 GB | 5 GB |
| **Multi-region** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **VNet** | ❌ | ✅ | ❌ | ❌ | ✅ |
| **Self-hosted Gateway** | ✅ | ✅ | ❌ | ❌ | ✅ |
| **Availability Zones** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **Dev Portal** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Built-in Cache** | ❌ | ✅ | ✅ | ✅ | ✅ |
| **Active Directory** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Pricing Model** | Per call | Monthly | Monthly | Monthly | Monthly |

**Tier Selection Guide:**

- **Consumption**: Serverless, pay-per-call, auto-scaling, no idle cost
- **Developer**: Development/testing, no SLA, includes VNet
- **Basic**: Small production workloads
- **Standard**: Medium production workloads, more scale
- **Premium**: Enterprise, multi-region, VNet, maximum scale

### 1.4 APIs, Operations, and Products

**API Structure:**

```
API Management Instance
│
├── Product 1 (Published)
│   ├── API A
│   │   ├── Operation: GET /users
│   │   ├── Operation: POST /users
│   │   └── Operation: GET /users/{id}
│   └── API B
│       └── Operation: GET /orders
│
├── Product 2 (Published)
│   └── API C
│       └── Operation: GET /products
│
└── Unpublished APIs (for testing)
```

#### APIs
- Represent a backend service
- Collection of operations
- Have base URL and version
- Import from: OpenAPI, WSDL, WADL, Azure resources

```bash
# Import API from OpenAPI spec
az apim api import \
  --resource-group myRG \
  --service-name myapim \
  --api-id myapi \
  --path /api \
  --specification-format OpenApi \
  --specification-url "https://example.com/swagger.json"

# Create blank API
az apim api create \
  --resource-group myRG \
  --service-name myapim \
  --api-id myapi \
  --display-name "My API" \
  --path /myapi \
  --protocols https
```

#### Operations
- Specific actions within an API
- HTTP method + URL path
- Request/response definitions
- Can have individual policies

```bash
# Create operation
az apim api operation create \
  --resource-group myRG \
  --service-name myapim \
  --api-id myapi \
  --operation-id getusers \
  --display-name "Get Users" \
  --method GET \
  --url-template "/users"
```

#### Products
- Group of one or more APIs
- Define access rights and usage limits
- Subscription required to access
- Can be open or require approval

**Product States:**
- **Published**: Visible in developer portal, can be subscribed
- **Not Published**: Hidden, only visible to administrators

```bash
# Create product
az apim product create \
  --resource-group myRG \
  --service-name myapim \
  --product-id starter \
  --display-name "Starter" \
  --description "Entry level product" \
  --subscription-required true \
  --approval-required false \
  --subscriptions-limit 1 \
  --state published

# Add API to product
az apim product api add \
  --resource-group myRG \
  --service-name myapim \
  --product-id starter \
  --api-id myapi
```

### 1.5 Subscriptions and Keys

**What are Subscriptions?**
- Named containers for subscription keys
- Provide access to APIs through products
- Track API usage per subscriber
- Enable rate limiting per subscription

**Subscription Scopes:**

| Scope | Description |
|-------|-------------|
| **All APIs** | Access to all APIs in the instance |
| **Single API** | Access to specific API only |
| **Product** | Access to all APIs in the product |

**Subscription Keys:**
- Primary and secondary keys per subscription
- Passed in request header or query string
- Default header: `Ocp-Apim-Subscription-Key`
- Alternative header: `Subscription-Key`
- Query string: `subscription-key`

```bash
# Create subscription
az apim subscription create \
  --resource-group myRG \
  --service-name myapim \
  --subscription-id mysubscription \
  --display-name "My Subscription" \
  --scope "/products/starter"

# Get subscription keys
az apim subscription show \
  --resource-group myRG \
  --service-name myapim \
  --subscription-id mysubscription \
  --query "{primary:primaryKey, secondary:secondaryKey}"

# Regenerate key
az apim subscription regenerate-key \
  --resource-group myRG \
  --service-name myapim \
  --subscription-id mysubscription \
  --key-type primary
```

**Using Subscription Keys:**

```bash
# In header (recommended)
curl -H "Ocp-Apim-Subscription-Key: abc123" \
  https://myapim.azure-api.net/api/users

# In query string
curl "https://myapim.azure-api.net/api/users?subscription-key=abc123"
```

### 1.6 API Management Policies

**What are Policies?**
- XML statements that modify API behavior
- Execute on request or response
- Can transform, validate, route, cache
- Applied at different scopes

**Policy Execution Order:**

```
Client Request
      │
      ▼
┌─────────────────┐
│  Global Policy  │  (All APIs scope)
│    <inbound>    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Product Policy  │  (Product scope)
│    <inbound>    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   API Policy    │  (API scope)
│    <inbound>    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│Operation Policy │  (Operation scope)
│    <inbound>    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    <backend>    │  → Backend Service
└────────┬────────┘
         │
         ▼
   (Reverse order for outbound)
         │
         ▼
   Client Response
```

**Policy Structure:**

```xml
<policies>
    <inbound>
        <!-- Policies applied to the request -->
        <base />  <!-- Include parent scope policies -->
    </inbound>
    <backend>
        <!-- Policies applied before forwarding to backend -->
        <base />
    </backend>
    <outbound>
        <!-- Policies applied to the response -->
        <base />
    </outbound>
    <on-error>
        <!-- Policies applied when an error occurs -->
        <base />
    </on-error>
</policies>
```

**The `<base>` Element:**
- Includes policies from parent scope
- Position determines order of execution
- Can be placed anywhere in section
- Can be omitted to override parent policies

```xml
<!-- Execute parent policies first, then this -->
<inbound>
    <base />
    <set-header name="X-Custom" exists-action="override">
        <value>custom-value</value>
    </set-header>
</inbound>

<!-- Execute this first, then parent policies -->
<inbound>
    <set-header name="X-Custom" exists-action="override">
        <value>custom-value</value>
    </set-header>
    <base />
</inbound>

<!-- Override parent policies completely -->
<inbound>
    <set-header name="X-Custom" exists-action="override">
        <value>custom-value</value>
    </set-header>
    <!-- No base element -->
</inbound>
```

### 1.7 Common Policies - Inbound

#### Access Restriction Policies

**check-header** - Validate header exists/value:
```xml
<check-header name="Authorization" failed-check-httpcode="401" 
              failed-check-error-message="Authorization header missing"
              ignore-case="true">
    <value>Bearer valid-token</value>
</check-header>
```

**ip-filter** - Allow/deny IP addresses:
```xml
<ip-filter action="allow">
    <address>10.0.0.1</address>
    <address-range from="10.0.1.0" to="10.0.1.255" />
</ip-filter>

<ip-filter action="forbid">
    <address>192.168.1.100</address>
</ip-filter>
```

**rate-limit** - Limit calls per subscription:
```xml
<!-- 100 calls per 60 seconds per subscription -->
<rate-limit calls="100" renewal-period="60" />

<!-- With specific keys -->
<rate-limit-by-key calls="100" renewal-period="60" 
                   counter-key="@(context.Subscription.Id)" />
```

**quota** - Limit calls over longer period:
```xml
<!-- 10000 calls per week, max 2MB bandwidth -->
<quota calls="10000" bandwidth="2097152" renewal-period="604800" />

<!-- Per subscription key -->
<quota-by-key calls="10000" renewal-period="604800"
              counter-key="@(context.Subscription.Id)" />
```

**validate-jwt** - Validate JWT token:
```xml
<validate-jwt header-name="Authorization" 
              failed-validation-httpcode="401"
              failed-validation-error-message="Unauthorized">
    <openid-config url="https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration" />
    <required-claims>
        <claim name="aud">
            <value>api://my-api</value>
        </claim>
    </required-claims>
</validate-jwt>
```

**cors** - Enable CORS:
```xml
<cors allow-credentials="true">
    <allowed-origins>
        <origin>https://myapp.com</origin>
        <origin>https://localhost:3000</origin>
    </allowed-origins>
    <allowed-methods>
        <method>GET</method>
        <method>POST</method>
        <method>PUT</method>
        <method>DELETE</method>
    </allowed-methods>
    <allowed-headers>
        <header>Content-Type</header>
        <header>Authorization</header>
    </allowed-headers>
</cors>
```

#### Transformation Policies

**set-header** - Add/modify headers:
```xml
<set-header name="X-Custom-Header" exists-action="override">
    <value>custom-value</value>
</set-header>

<set-header name="X-Request-Id" exists-action="skip">
    <value>@(Guid.NewGuid().ToString())</value>
</set-header>

<!-- Remove header -->
<set-header name="X-Internal" exists-action="delete" />
```

**set-query-parameter** - Add/modify query params:
```xml
<set-query-parameter name="api-version" exists-action="override">
    <value>2024-01-01</value>
</set-query-parameter>
```

**set-body** - Modify request body:
```xml
<!-- Set static body -->
<set-body>{ "modified": true }</set-body>

<!-- Transform body with template -->
<set-body template="liquid">
{
    "user": "{{ body.username }}",
    "email": "{{ body.email | downcase }}"
}
</set-body>

<!-- Transform with C# expression -->
<set-body>@{
    var body = context.Request.Body.As<JObject>();
    body["timestamp"] = DateTime.UtcNow;
    return body.ToString();
}</set-body>
```

**rewrite-uri** - Change request path:
```xml
<!-- Rewrite /external/users to /internal/v2/users -->
<rewrite-uri template="/internal/v2/users" />

<!-- With path parameters -->
<rewrite-uri template="/api/v2/users/{id}" copy-unmatched-params="true" />
```

**set-backend-service** - Route to different backend:
```xml
<set-backend-service base-url="https://different-backend.com" />

<!-- Conditional routing -->
<choose>
    <when condition="@(context.Request.Headers.GetValueOrDefault("X-Test","") == "true")">
        <set-backend-service base-url="https://test-backend.com" />
    </when>
</choose>
```

### 1.8 Common Policies - Outbound

**set-header** - Modify response headers:
```xml
<outbound>
    <!-- Add custom header -->
    <set-header name="X-Powered-By" exists-action="override">
        <value>Azure API Management</value>
    </set-header>
    
    <!-- Remove sensitive headers -->
    <set-header name="X-AspNet-Version" exists-action="delete" />
    <set-header name="Server" exists-action="delete" />
</outbound>
```

**set-body** - Transform response:
```xml
<outbound>
    <!-- Wrap response -->
    <set-body>@{
        var response = context.Response.Body.As<JObject>();
        return new JObject(
            new JProperty("data", response),
            new JProperty("status", "success"),
            new JProperty("timestamp", DateTime.UtcNow)
        ).ToString();
    }</set-body>
</outbound>
```

**xml-to-json** - Convert XML response to JSON:
```xml
<outbound>
    <xml-to-json kind="direct" apply="always" consider-accept-header="true" />
</outbound>
```

**json-to-xml** - Convert JSON response to XML:
```xml
<outbound>
    <json-to-xml apply="always" consider-accept-header="true" />
</outbound>
```

**find-and-replace** - Replace text in response:
```xml
<outbound>
    <find-and-replace from="internal-server" to="api-server" />
</outbound>
```

### 1.9 Common Policies - Backend

**forward-request** - Control backend request:
```xml
<backend>
    <forward-request timeout="60" 
                     follow-redirects="true" 
                     buffer-request-body="true" />
</backend>
```

**send-request** - Make additional requests:
```xml
<send-request mode="new" response-variable-name="tokenResponse" 
              timeout="20" ignore-error="false">
    <set-url>https://auth.example.com/token</set-url>
    <set-method>POST</set-method>
    <set-header name="Content-Type" exists-action="override">
        <value>application/x-www-form-urlencoded</value>
    </set-header>
    <set-body>grant_type=client_credentials&amp;client_id=xxx&amp;client_secret=xxx</set-body>
</send-request>
```

**retry** - Retry failed requests:
```xml
<retry condition="@(context.Response.StatusCode == 503)" 
       count="3" 
       interval="1" 
       first-fast-retry="true">
    <forward-request />
</retry>
```

### 1.10 Common Policies - Advanced

#### Caching Policies

**cache-lookup** / **cache-store** - Response caching:
```xml
<inbound>
    <!-- Check cache first -->
    <cache-lookup vary-by-developer="false" 
                  vary-by-developer-groups="false"
                  downstream-caching-type="none">
        <vary-by-header>Accept</vary-by-header>
        <vary-by-query-parameter>version</vary-by-query-parameter>
    </cache-lookup>
</inbound>

<outbound>
    <!-- Store in cache for 1 hour -->
    <cache-store duration="3600" />
</outbound>
```

**cache-lookup-value** / **cache-store-value** - Value caching:
```xml
<inbound>
    <!-- Get cached token -->
    <cache-lookup-value key="auth-token" variable-name="cachedToken" />
    <choose>
        <when condition="@(context.Variables.GetValueOrDefault<string>("cachedToken") == null)">
            <!-- Get new token and cache it -->
            <send-request mode="new" response-variable-name="tokenResponse">
                <set-url>https://auth.example.com/token</set-url>
                <set-method>POST</set-method>
            </send-request>
            <set-variable name="newToken" 
                          value="@(((IResponse)context.Variables["tokenResponse"]).Body.As<JObject>()["access_token"].ToString())" />
            <cache-store-value key="auth-token" 
                              value="@((string)context.Variables["newToken"])" 
                              duration="3600" />
        </when>
    </choose>
</inbound>
```

#### Control Flow Policies

**choose** - Conditional execution:
```xml
<choose>
    <when condition="@(context.Request.Method == "POST")">
        <set-header name="Content-Type" exists-action="override">
            <value>application/json</value>
        </set-header>
    </when>
    <when condition="@(context.Request.Method == "GET")">
        <cache-lookup vary-by-developer="false" />
    </when>
    <otherwise>
        <return-response>
            <set-status code="405" reason="Method Not Allowed" />
        </return-response>
    </otherwise>
</choose>
```

**set-variable** - Store values:
```xml
<set-variable name="userId" value="@(context.Request.Headers.GetValueOrDefault("X-User-Id", "anonymous"))" />
<set-variable name="isAdmin" value="@(context.User.Groups.Any(g => g.Name == "Administrators"))" />
```

**return-response** - Return immediately:
```xml
<return-response>
    <set-status code="200" reason="OK" />
    <set-header name="Content-Type" exists-action="override">
        <value>application/json</value>
    </set-header>
    <set-body>{ "message": "Mock response" }</set-body>
</return-response>
```

**mock-response** - Return mock data:
```xml
<mock-response status-code="200" content-type="application/json" />
```

#### Error Handling

**on-error** section:
```xml
<on-error>
    <base />
    <set-header name="X-Error" exists-action="override">
        <value>@(context.LastError.Message)</value>
    </set-header>
    <set-body>@{
        return new JObject(
            new JProperty("error", context.LastError.Message),
            new JProperty("reason", context.LastError.Reason),
            new JProperty("requestId", context.RequestId)
        ).ToString();
    }</set-body>
</on-error>
```

### 1.11 Policy Expressions

**Context Object:**
- `context.Request` - Incoming request
- `context.Response` - Backend response (outbound only)
- `context.User` - Authenticated user
- `context.Subscription` - Current subscription
- `context.Api` - Current API
- `context.Operation` - Current operation
- `context.Product` - Current product
- `context.Variables` - Custom variables
- `context.LastError` - Last error (on-error only)

**Common Expressions:**

```xml
<!-- Get header value -->
@(context.Request.Headers.GetValueOrDefault("X-Custom", "default"))

<!-- Get query parameter -->
@(context.Request.Url.Query.GetValueOrDefault("param", "default"))

<!-- Get body as string -->
@(context.Request.Body.As<string>())

<!-- Get body as JSON -->
@(context.Request.Body.As<JObject>())

<!-- Get path parameter (from URL template) -->
@(context.Request.MatchedParameters["id"])

<!-- User information -->
@(context.User?.Email ?? "anonymous")
@(context.User?.Id ?? "unknown")

<!-- Subscription info -->
@(context.Subscription.Key)
@(context.Subscription.Id)

<!-- Generate GUID -->
@(Guid.NewGuid().ToString())

<!-- Current timestamp -->
@(DateTime.UtcNow.ToString("o"))

<!-- Check user group -->
@(context.User.Groups.Any(g => g.Name == "Premium"))

<!-- Base64 encode -->
@(Convert.ToBase64String(Encoding.UTF8.GetBytes("text")))

<!-- HMAC signature -->
@{
    var key = "secret-key";
    var message = context.Request.Body.As<string>();
    using (var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(key)))
    {
        var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(message));
        return Convert.ToBase64String(hash);
    }
}
```

### 1.12 API Versioning

**Versioning Schemes:**

| Scheme | Example | Description |
|--------|---------|-------------|
| Path | `/v1/users` | Version in URL path |
| Query | `/users?api-version=1.0` | Version in query string |
| Header | `Api-Version: 1.0` | Version in header |

**Create Versioned API:**

```bash
# Create API version set
az apim api versionset create \
  --resource-group myRG \
  --service-name myapim \
  --version-set-id myapi-versions \
  --display-name "My API Versions" \
  --versioning-scheme Path

# Create versioned API
az apim api create \
  --resource-group myRG \
  --service-name myapim \
  --api-id myapi-v1 \
  --display-name "My API v1" \
  --path /myapi \
  --api-version v1 \
  --api-version-set-id myapi-versions
```

### 1.13 API Revisions

**What are Revisions?**
- Non-breaking changes to an API
- Test changes before making them live
- Keep history of API configurations
- Roll back to previous revisions

**Revision vs Version:**

| Aspect | Revision | Version |
|--------|----------|---------|
| Purpose | Non-breaking changes | Breaking changes |
| URL | Same | Different |
| Testing | Yes, before release | No |
| Rollback | Yes | No |
| Example | Bug fix, new optional field | New endpoint structure |

```bash
# Create new revision
az apim api revision create \
  --resource-group myRG \
  --service-name myapim \
  --api-id myapi \
  --api-revision 2 \
  --api-revision-description "Added new optional field"

# Make revision current
az apim api release create \
  --resource-group myRG \
  --service-name myapim \
  --api-id myapi \
  --release-id release-2 \
  --api-revision 2 \
  --notes "Released revision 2"
```

### 1.14 Securing APIs

**Authentication Options:**

#### 1. Subscription Keys (Default)
```xml
<!-- Already enabled by default -->
<!-- Key passed in Ocp-Apim-Subscription-Key header -->
```

#### 2. OAuth 2.0 / OpenID Connect
```xml
<validate-jwt header-name="Authorization" 
              failed-validation-httpcode="401"
              failed-validation-error-message="Invalid token">
    <openid-config url="https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration" />
    <audiences>
        <audience>api://my-api-id</audience>
    </audiences>
    <issuers>
        <issuer>https://login.microsoftonline.com/{tenant}/v2.0</issuer>
    </issuers>
    <required-claims>
        <claim name="scp" match="any">
            <value>API.Read</value>
            <value>API.Write</value>
        </claim>
    </required-claims>
</validate-jwt>
```

#### 3. Client Certificates
```xml
<choose>
    <when condition="@(context.Request.Certificate == null || 
                       context.Request.Certificate.Thumbprint != "expected-thumbprint")">
        <return-response>
            <set-status code="403" reason="Invalid certificate" />
        </return-response>
    </when>
</choose>
```

#### 4. IP Filtering
```xml
<ip-filter action="allow">
    <address-range from="10.0.0.0" to="10.0.0.255" />
</ip-filter>
```

#### 5. Rate Limiting
```xml
<rate-limit calls="100" renewal-period="60" />
<quota calls="10000" renewal-period="604800" />
```

### 1.15 Backend Authentication

**Pass-through Authentication:**
```xml
<!-- Forward client certificate to backend -->
<authentication-certificate certificate-id="my-cert" />
```

**Managed Identity Authentication:**
```xml
<authentication-managed-identity resource="https://management.azure.com/" />
```

**Basic Authentication:**
```xml
<authentication-basic username="user" password="pass" />
```

**OAuth 2.0 to Backend:**
```xml
<send-request mode="new" response-variable-name="bearerToken" 
              timeout="20" ignore-error="true">
    <set-url>https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token</set-url>
    <set-method>POST</set-method>
    <set-header name="Content-Type" exists-action="override">
        <value>application/x-www-form-urlencoded</value>
    </set-header>
    <set-body>@{
        return "client_id=xxx&client_secret=xxx&scope=api://backend/.default&grant_type=client_credentials";
    }</set-body>
</send-request>
<set-header name="Authorization" exists-action="override">
    <value>@("Bearer " + ((IResponse)context.Variables["bearerToken"]).Body.As<JObject>()["access_token"].ToString())</value>
</set-header>
```

### 1.16 Self-Hosted Gateway

**What is Self-Hosted Gateway?**
- Containerized version of APIM gateway
- Run gateway close to your backends
- Hybrid and multi-cloud scenarios
- Same policies as cloud gateway
- Requires connection to APIM control plane

**Use Cases:**
- Low latency requirements
- Data residency requirements
- On-premises APIs
- Multi-cloud deployments
- Edge computing

**Deployment:**

```bash
# Get gateway token
az apim gateway token create \
  --resource-group myRG \
  --service-name myapim \
  --gateway-id my-gateway \
  --expiry "2024-12-31"

# Deploy to Kubernetes
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apim-gateway
spec:
  replicas: 2
  selector:
    matchLabels:
      app: apim-gateway
  template:
    metadata:
      labels:
        app: apim-gateway
    spec:
      containers:
      - name: apim-gateway
        image: mcr.microsoft.com/azure-api-management/gateway:latest
        env:
        - name: config.service.endpoint
          value: "https://myapim.configuration.azure-api.net"
        - name: config.service.auth
          valueFrom:
            secretKeyRef:
              name: apim-gateway-token
              key: value
        ports:
        - containerPort: 8080
        - containerPort: 8081
EOF
```

### 1.17 Importing APIs

**Import Sources:**

| Source | Format | Description |
|--------|--------|-------------|
| OpenAPI | JSON/YAML | Swagger/OpenAPI 3.0 specs |
| WSDL | XML | SOAP web services |
| WADL | XML | RESTful service descriptions |
| Azure Resources | - | Functions, Logic Apps, App Service |
| GraphQL | Schema | GraphQL APIs |

**Import Commands:**

```bash
# Import from OpenAPI URL
az apim api import \
  --resource-group myRG \
  --service-name myapim \
  --api-id petstore \
  --path /petstore \
  --specification-format OpenApi \
  --specification-url "https://petstore.swagger.io/v2/swagger.json"

# Import from local file
az apim api import \
  --resource-group myRG \
  --service-name myapim \
  --api-id myapi \
  --path /myapi \
  --specification-format OpenApi \
  --specification-path "./swagger.json"

# Import Azure Function
az apim api import \
  --resource-group myRG \
  --service-name myapim \
  --api-id myfunc \
  --path /func \
  --specification-format OpenApi \
  --service-url "https://myfunc.azurewebsites.net/api"
```

### 1.18 Developer Portal

**Features:**
- Auto-generated API documentation
- Interactive API console
- User registration and sign-in
- Subscription management
- Usage analytics
- Customizable appearance

**Customization Options:**
- **Visual editor**: Drag-and-drop UI customization
- **Code-based**: Custom widgets and themes
- **Self-hosted**: Deploy your own instance
- **Disable**: Turn off if not needed

**Portal URLs:**
- **Managed**: `https://{apim-name}.developer.azure-api.net`
- **Custom domain**: Configure in portal settings

```bash
# Publish developer portal
az apim portalsetting update \
  --resource-group myRG \
  --service-name myapim \
  --enabled true
```

### 1.19 Named Values (Named Properties)

**What are Named Values?**
- Key-value pairs stored in APIM instance
- Reusable across all policies
- Acts as global constants or configuration
- Can reference Azure Key Vault secrets

**Types:**

| Type | Description |
|------|-------------|
| **Plain** | Literal string value, visible in portal |
| **Secret** | Encrypted at rest, hidden in portal (masked with asterisks) |
| **Key Vault** | References a secret stored in Azure Key Vault |

```xml
<!-- Reference named value in policy using double curly braces -->
<set-header name="X-Api-Key" exists-action="override">
    <value>{{api-key}}</value>
</set-header>

<!-- Or using policy expression -->
<set-header name="X-Api-Key" exists-action="override">
    <value>@(context.NamedValues.GetValueOrDefault("api-key", "default"))</value>
</set-header>

<!-- Use in backend URL -->
<set-backend-service base-url="{{backend-url}}" />

<!-- Use in validate-jwt -->
<validate-jwt header-name="Authorization">
    <issuer-signing-keys>
        <key>{{jwt-signing-key}}</key>
    </issuer-signing-keys>
</validate-jwt>
```

```bash
# Create named value (plain)
az apim nv create \
  --resource-group myRG \
  --service-name myapim \
  --named-value-id api-key \
  --display-name "API Key" \
  --value "my-secret-key"

# Create secret named value
az apim nv create \
  --resource-group myRG \
  --service-name myapim \
  --named-value-id db-connection \
  --display-name "DB Connection" \
  --value "Server=..." \
  --secret true

# Create Key Vault reference
az apim nv create \
  --resource-group myRG \
  --service-name myapim \
  --named-value-id kv-secret \
  --display-name "KV Secret" \
  --key-vault "https://myvault.vault.azure.net/secrets/my-secret"
```

**⚠️ Exam Tip:** Named values are referenced with `{{name}}` syntax in policies. Key Vault-backed named values require APIM managed identity with Key Vault access. Secret named values are NOT shown in portal or exported — they're one-way.

### 1.20 Groups and Users

**Built-in Groups (Cannot be deleted):**

| Group | Description |
|-------|-------------|
| **Administrators** | Azure subscription administrators. Full control of APIM. |
| **Developers** | Authenticated developer portal users. Can view APIs, create subscriptions, test APIs. |
| **Guests** | Unauthenticated developer portal visitors. Read-only access to some content. |

- Custom groups can be created
- Groups from Azure AD can be mapped to APIM groups
- Users are assigned to groups
- Products are granted to groups (controls visibility)

```
Product "Premium APIs" → Granted to Group "Premium Developers"
    ↳ Only users in "Premium Developers" group see and subscribe to this product
```

```bash
# Create custom group
az apim group create \
  --resource-group myRG \
  --service-name myapim \
  --group-id premium-devs \
  --display-name "Premium Developers" \
  --description "Developers with premium access"

# Add user to group
az apim group user create \
  --resource-group myRG \
  --service-name myapim \
  --group-id premium-devs \
  --user-id user@example.com
```

**⚠️ Exam Tip:** Products → Groups → Users is the access model. Products are visible to groups. Users subscribe to products through groups they belong to. Administrators group members manage the APIM instance.

### 1.21 Backend Entities

**What is a Backend Entity?**
- Named backend service configuration in APIM
- Encapsulates backend URL, credentials, circuit breaker settings
- Reusable across APIs and policies
- Supports load balancing across multiple backends

```json
// Backend entity definition
{
  "url": "https://my-backend.azurewebsites.net",
  "protocol": "http",
  "credentials": {
    "header": {
      "x-api-key": ["secret-value"]
    }
  },
  "tls": {
    "validateCertificateChain": true,
    "validateCertificateName": true
  },
  "circuitBreaker": {
    "rules": [{
      "failureCondition": {
        "count": 5,
        "interval": "PT1M",
        "statusCodeRanges": [{ "min": 500, "max": 599 }]
      },
      "tripDuration": "PT1M"
    }]
  }
}
```

```xml
<!-- Reference backend entity in policy -->
<set-backend-service backend-id="my-backend" />

<!-- Load balancer with multiple backends -->
<set-backend-service backend-id="load-balanced-pool" />
```

**Backend Pool (Load Balancing):**
- Round-robin or weighted distribution across backends
- Circuit breaker per backend
- Health probes

### 1.22 Certificates Management

**Certificate Uses in APIM:**

| Use | Description |
|-----|-------------|
| **Client certificate to backend** | APIM authenticates to backend using certificate |
| **Client certificate from client** | Clients authenticate to APIM using certificate |
| **Custom domains** | TLS/SSL for custom gateway domains |
| **CA certificates** | Trust custom certificate authorities |

**Client Certificate Validation Policy:**

```xml
<inbound>
    <choose>
        <!-- Check certificate exists -->
        <when condition="@(context.Request.Certificate == null)">
            <return-response>
                <set-status code="403" reason="No client certificate" />
            </return-response>
        </when>
        <!-- Validate thumbprint -->
        <when condition="@(context.Request.Certificate.Thumbprint != "desired-thumbprint")">
            <return-response>
                <set-status code="403" reason="Invalid certificate" />
            </return-response>
        </when>
        <!-- Validate issuer -->
        <when condition="@(!context.Request.Certificate.Verify())">
            <return-response>
                <set-status code="403" reason="Certificate validation failed" />
            </return-response>
        </when>
    </choose>
</inbound>
```

**Certificate Properties in Policy Expressions:**

| Property | Description |
|----------|-------------|
| `context.Request.Certificate` | Client certificate object |
| `.Thumbprint` | SHA-1 thumbprint |
| `.Subject` | Subject distinguished name |
| `.Issuer` | Issuer distinguished name |
| `.NotBefore` | Valid from date |
| `.NotAfter` | Valid to date |
| `.SerialNumber` | Serial number |
| `.Verify()` | Validates chain trust |

**Consumption Tier Note:** In Consumption tier, you must explicitly enable client certificates and upload trusted CA root certificates.

```bash
# Upload certificate for backend authentication
az apim certificate create \
  --resource-group myRG \
  --service-name myapim \
  --certificate-id my-cert \
  --pfx-file-path ./cert.pfx \
  --pfx-password "certpass"
```

### 1.23 Application Insights & Monitoring

**Built-in Analytics:**
- Request/response metrics
- Subscription-level usage
- Product-level usage
- API-level performance
- Geographic distribution

**Application Insights Integration:**

```bash
# Enable Application Insights logging
az apim api-management-service update \
  --name myapim \
  --resource-group myRG

# Create logger for Application Insights
az apim logger create \
  --resource-group myRG \
  --service-name myapim \
  --logger-id appinsights-logger \
  --logger-type applicationInsights \
  --instrumentation-key "<app-insights-key>"

# Create diagnostic for all APIs
az apim diagnostic create \
  --resource-group myRG \
  --service-name myapim \
  --diagnostic-id applicationinsights \
  --logger-id appinsights-logger \
  --sampling-percentage 100
```

**Diagnostic Logging Levels:**

| Level | What's Logged |
|-------|--------------|
| **Errors only** | 4xx, 5xx responses |
| **All** | All requests |
| **Custom** | Based on sampling percentage |

**Log-to-Event Hub Policy:**

```xml
<log-to-eventhub logger-id="eventhub-logger">@{
    return new JObject(
        new JProperty("EventTime", DateTime.UtcNow.ToString()),
        new JProperty("ServiceName", context.Deployment.ServiceName),
        new JProperty("RequestId", context.RequestId),
        new JProperty("RequestIp", context.Request.IpAddress),
        new JProperty("OperationName", context.Operation.Name)
    ).ToString();
}</log-to-eventhub>
```

**Emit Metric Policy (Custom Metrics):**

```xml
<emit-metric name="custom-request-metric" namespace="apim-custom-metrics">
    <dimension name="API" value="@(context.Api.Name)" />
    <dimension name="Operation" value="@(context.Operation.Name)" />
    <dimension name="Client" value="@(context.User?.Email ?? "anonymous")" />
</emit-metric>
```

**Trace Policy (Debugging):**

```xml
<trace source="my-api" severity="information">
    <message>@($"Processing request {context.RequestId} for {context.Operation.Name}")</message>
    <metadata name="userId" value="@(context.User?.Id ?? "anonymous")" />
</trace>
```

**⚠️ Exam Tip:** `log-to-eventhub` sends data to Event Hub for custom analytics. `emit-metric` sends custom metrics to Azure Monitor. `trace` writes to API Inspector tracing (visible in test console). Application Insights provides end-to-end distributed tracing.

### 1.24 VNet Integration

**Two Modes:**

| Mode | Description | Inbound | Outbound |
|------|-------------|---------|----------|
| **External** | Gateway accessible from internet; backends in VNet | Public | VNet |
| **Internal** | Gateway ONLY accessible from VNet | VNet | VNet |

```
External Mode:
  Internet → APIM Gateway (public IP) → VNet → Backend Services

Internal Mode:
  VNet Only → APIM Gateway (private IP) → VNet → Backend Services
  (Requires VPN/ExpressRoute for external access)
```

**Tier Support:**
- VNet integration available only in **Developer** and **Premium** tiers
- NOT available in Consumption, Basic, or Standard tiers
- Premium supports both External and Internal modes

```bash
# Create APIM with External VNet
az apim create \
  --name myapim \
  --resource-group myRG \
  --publisher-email admin@example.com \
  --publisher-name "My Company" \
  --sku-name Premium \
  --virtual-network External

# Update to Internal VNet
az apim update \
  --name myapim \
  --resource-group myRG \
  --virtual-network Internal
```

**Private Endpoint (Alternative to VNet):**
- Available in more tiers than full VNet integration
- Inbound private connectivity only
- Backend calls still go over internet unless VNet-integrated

**⚠️ Exam Tip:** External VNet = accessible from internet but backend is in VNet. Internal VNet = only accessible from within VNet. If question says "API must not be accessible from internet", choose Internal mode + Premium tier.

### 1.25 OAuth 2.0 Authorization in Developer Portal

**Scenario:** Protect the "Try it" console in developer portal with OAuth 2.0

**Setup Steps:**
1. Register a backend API app in Azure AD
2. Register a developer portal app in Azure AD
3. Grant the developer portal app permission to call the backend API
4. Configure OAuth 2.0 in APIM developer portal settings
5. Add `validate-jwt` policy on the API

```xml
<!-- Validate tokens from developer portal -->
<inbound>
    <validate-jwt header-name="Authorization" 
                  failed-validation-httpcode="401"
                  failed-validation-error-message="Unauthorized">
        <openid-config url="https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration" />
        <audiences>
            <audience>api://{backend-app-id}</audience>
        </audiences>
        <issuers>
            <issuer>https://sts.windows.net/{tenant-id}/</issuer>
        </issuers>
    </validate-jwt>
</inbound>
```

**⚠️ Exam Tip:** OAuth 2.0 configured in APIM developer portal does NOT enforce authorization — it only enables the "Try it" console to get tokens. You MUST add `validate-jwt` policy to actually enforce token validation on the API.

### 1.26 Policy Fragments

**What are Policy Fragments?**
- Reusable pieces of policy XML
- Define once, use across multiple APIs/operations
- Reduces duplication
- Managed independently

```xml
<!-- Define a policy fragment named "standard-headers" -->
<!-- Fragment content: -->
<fragment>
    <set-header name="X-Request-Id" exists-action="skip">
        <value>@(Guid.NewGuid().ToString())</value>
    </set-header>
    <set-header name="X-Forwarded-For" exists-action="override">
        <value>@(context.Request.IpAddress)</value>
    </set-header>
    <cors allow-credentials="true">
        <allowed-origins>
            <origin>https://myapp.com</origin>
        </allowed-origins>
        <allowed-methods preflight-result-max-age="300">
            <method>*</method>
        </allowed-methods>
        <allowed-headers>
            <header>*</header>
        </allowed-headers>
    </cors>
</fragment>

<!-- Reference the fragment in any policy -->
<inbound>
    <base />
    <include-fragment fragment-id="standard-headers" />
    <!-- API-specific policies -->
</inbound>
```

**⚠️ Exam Tip:** Policy fragments use `<include-fragment fragment-id="name" />` to reference. Great for DRY principle. Each fragment is a self-contained policy snippet.

### 1.27 Additional Important Policies

#### Validation Policies

**validate-content** — Validate request/response body:

```xml
<inbound>
    <validate-content unspecified-content-type-action="prevent"
                       max-size="102400"
                       size-exceeded-action="prevent"
                       errors-variable-name="validationErrors">
        <content type="application/json" validate-as="json" 
                 action="prevent" />
    </validate-content>
</inbound>
```

**validate-parameters** — Validate query/path parameters:

```xml
<inbound>
    <validate-parameters specified-parameter-action="prevent"
                          unspecified-parameter-action="prevent"
                          errors-variable-name="paramErrors" />
</inbound>
```

**validate-headers** — Validate request headers:

```xml
<inbound>
    <validate-headers specified-header-action="prevent"
                       unspecified-header-action="ignore"
                       errors-variable-name="headerErrors" />
</inbound>
```

**validate-status-code** — Validate backend response status:

```xml
<outbound>
    <validate-status-code unspecified-status-code-action="prevent"
                           errors-variable-name="statusErrors" />
</outbound>
```

#### Concurrency and Parallelism

**limit-concurrency** — Limit parallel executions:

```xml
<inbound>
    <limit-concurrency key="@(context.Subscription.Id)" max-count="10">
        <forward-request />
    </limit-concurrency>
</inbound>
```

**wait** — Execute multiple policies in parallel:

```xml
<inbound>
    <wait for="all">
        <send-request mode="new" response-variable-name="response1">
            <set-url>https://service1.com/api</set-url>
            <set-method>GET</set-method>
        </send-request>
        <send-request mode="new" response-variable-name="response2">
            <set-url>https://service2.com/api</set-url>
            <set-method>GET</set-method>
        </send-request>
    </wait>
    <!-- Aggregate responses -->
    <return-response>
        <set-status code="200" />
        <set-body>@{
            var r1 = ((IResponse)context.Variables["response1"]).Body.As<JObject>();
            var r2 = ((IResponse)context.Variables["response2"]).Body.As<JObject>();
            return new JObject(
                new JProperty("service1", r1),
                new JProperty("service2", r2)
            ).ToString();
        }</set-body>
    </return-response>
</inbound>
```

**⚠️ `wait` `for` attribute:**
- `for="all"` — Wait for ALL requests to complete
- `for="any"` — Proceed when ANY ONE request completes

#### Other Important Policies

**send-one-way-request** — Fire-and-forget (don't wait for response):

```xml
<send-one-way-request mode="new">
    <set-url>https://webhook.example.com/notify</set-url>
    <set-method>POST</set-method>
    <set-body>@{
        return new JObject(
            new JProperty("event", "api-called"),
            new JProperty("api", context.Api.Name)
        ).ToString();
    }</set-body>
</send-one-way-request>
```

**redirect** — HTTP redirect:

```xml
<return-response>
    <set-status code="301" reason="Moved Permanently" />
    <set-header name="Location" exists-action="override">
        <value>https://new-api.example.com/v2</value>
    </set-header>
</return-response>
```

**jsonp** — Add JSONP support (wrap JSON in callback):

```xml
<outbound>
    <jsonp callback-parameter-name="callback" />
</outbound>
```

### 1.28 Liquid Templates in Policies

**What are Liquid Templates?**
- Templating language for transforming request/response bodies
- Alternative to C# expressions for body transformation
- Supports loops, conditionals, filters
- Useful for complex JSON/XML transformations

```xml
<!-- Transform JSON response using Liquid -->
<set-body template="liquid">
{
    "results": [
        {% JSONArrayFor item in body.items %}
        {
            "name": "{{ item.name }}",
            "price": {{ item.price | times: 1.1 }},
            "category": "{{ item.category | upcase }}"
        }
        {% endJSONArrayFor %}
    ],
    "count": {{ body.items.size }},
    "generated": "{{ 'now' | date: '%Y-%m-%dT%H:%M:%S' }}"
}
</set-body>
```

```xml
<!-- Convert XML to JSON with Liquid -->
<set-body template="liquid">
{
    "orderId": "{{ body.order.id }}",
    "customer": "{{ body.order.customer.name }}",
    "items": [
        {% JSONArrayFor item in body.order.items %}
        {
            "product": "{{ item.product }}",
            "quantity": {{ item.qty }}
        }
        {% endJSONArrayFor %}
    ]
}
</set-body>
```

**Liquid Filters:**
- `upcase` / `downcase` — Change case
- `replace` — Replace text
- `size` — Array/string length
- `times` / `divided_by` / `plus` / `minus` — Math
- `date` — Format dates
- `split` / `join` — String operations

**⚠️ Exam Tip:** Liquid templates use `template="liquid"` attribute on `<set-body>`. Use `{% JSONArrayFor %}` for JSON array iteration (NOT standard Liquid `{% for %}`). Liquid is good for transformations; C# expressions are better for logic.

### 1.29 APIM with Azure Functions, Logic Apps, and App Service

#### Azure Functions Integration

```bash
# Import Azure Function as API
az apim api import \
  --resource-group myRG \
  --service-name myapim \
  --api-id func-api \
  --path /func \
  --specification-format OpenApi \
  --specification-url "https://myfunc.azurewebsites.net/api/swagger.json"
```

**Function App as Backend:**
- Import directly from Azure portal → "Add API" → "Function App"
- Auto-generates operations from HTTP-triggered functions
- APIM can use Function host key for authentication

```xml
<!-- Use function key for authentication -->
<set-header name="x-functions-key" exists-action="override">
    <value>{{function-host-key}}</value>
</set-header>
```

#### Logic Apps Integration

- Import Logic App request trigger as API
- APIM manages access; Logic App handles orchestration
- Logic App provides the OpenAPI spec automatically

#### App Service Integration

- Import App Service Web API as backend
- Use managed identity for APIM → App Service authentication
- App Service can require APIM certificate

```xml
<!-- Authenticate to App Service using managed identity -->
<authentication-managed-identity resource="api://{app-service-app-id}" />
```

### 1.30 GraphQL APIs in APIM

**Two Types:**

| Type | Description |
|------|-------------|
| **Pass-through GraphQL** | Forward GraphQL requests to existing GraphQL backend |
| **Synthetic GraphQL** | APIM resolves GraphQL against REST/SOAP backends |

**Synthetic GraphQL Resolvers:**

```xml
<!-- Resolver for a GraphQL field using REST backend -->
<http-data-source>
    <http-request>
        <set-method>GET</set-method>
        <set-url>@($"https://rest-backend.com/users/{context.GraphQL.Arguments["id"]}")</set-url>
    </http-request>
</http-data-source>
```

**WebSocket API Support:**
- Pass-through WebSocket connections
- Available in Standard and Premium tiers
- Cannot apply policies to individual WebSocket messages

### 1.31 Bicep/ARM Deployment

```bicep
resource apim 'Microsoft.ApiManagement/service@2023-03-01-preview' = {
  name: 'myapim'
  location: resourceGroup().location
  sku: {
    name: 'Developer'
    capacity: 1
  }
  properties: {
    publisherEmail: 'admin@example.com'
    publisherName: 'My Company'
  }
}

// Define a named value
resource namedValue 'Microsoft.ApiManagement/service/namedValues@2023-03-01-preview' = {
  parent: apim
  name: 'api-key'
  properties: {
    displayName: 'API Key'
    value: 'my-secret-key'
    secret: true
  }
}

// Define an API
resource api 'Microsoft.ApiManagement/service/apis@2023-03-01-preview' = {
  parent: apim
  name: 'myapi'
  properties: {
    displayName: 'My API'
    path: 'myapi'
    protocols: ['https']
    serviceUrl: 'https://mybackend.azurewebsites.net'
  }
}

// Define a product
resource product 'Microsoft.ApiManagement/service/products@2023-03-01-preview' = {
  parent: apim
  name: 'starter'
  properties: {
    displayName: 'Starter'
    subscriptionRequired: true
    approvalRequired: false
    state: 'published'
  }
}
```

### 1.32 APIM DevOps and CI/CD

**Approaches:**

| Approach | Description |
|----------|-------------|
| **ARM/Bicep templates** | Infrastructure as Code for APIM resources |
| **APIOps** | Git-based CI/CD for API definitions and policies |
| **Azure Resource Manager API** | REST API for programmatic management |
| **Extract → Transform → Deploy** | Export current config, modify, deploy to target |

**Git Integration (Built-in):**
- APIM has a built-in Git repository
- Export current configuration to Git
- Make changes in Git
- Deploy from Git back to APIM

```bash
# Save current APIM configuration to Git
az apim api-management-service backup \
  --resource-group myRG \
  --name myapim \
  --backup-name apim-backup \
  --storage-account-name mystorage \
  --storage-account-container backups

# Restore from backup
az apim api-management-service restore \
  --resource-group myRG \
  --name myapim \
  --backup-name apim-backup \
  --storage-account-name mystorage \
  --storage-account-container backups
```

### 1.33 Consumption Tier Deep Dive

**Unique Characteristics:**

| Feature | Consumption Tier Behavior |
|---------|--------------------------|
| **Pricing** | Pay per call (no idle cost) |
| **Scaling** | Automatic |
| **Cold start** | Yes — first request may have latency |
| **VNet** | NOT supported |
| **Built-in cache** | NOT supported (use external Redis) |
| **Self-hosted gateway** | Supported |
| **Dev portal** | Supported |
| **SLA** | 99.95% |
| **Max throughput** | Dynamically allocated |
| **Subscription keys** | Optional (can disable) |
| **Named values** | Supported |
| **Custom domains** | 1 gateway domain |

**When to Use Consumption:**
- Microservices architectures
- Serverless / event-driven workloads
- Variable or unpredictable traffic
- Cost-sensitive (no idle cost)
- Don't need VNet integration
- Accept possible cold start latency

**⚠️ Exam Tip:** If the question mentions "minimize cost" + "no idle charges" + "serverless" → Consumption tier. If it mentions "VNet" or "multi-region" → Premium tier. Cold start is a trade-off of Consumption tier.

### 1.34 Rate Limiting vs Quotas (Deep Comparison)

| Aspect | Rate Limit | Quota |
|--------|-----------|-------|
| **Purpose** | Prevent short-term bursts | Limit total usage over time |
| **Period** | Short (seconds/minutes) | Long (hours/days/weeks) |
| **Counter reset** | Rolling window or fixed | Fixed period |
| **HTTP response** | 429 Too Many Requests | 403 Forbidden |
| **Headers returned** | `Retry-After` | `Retry-After` |
| **Scope** | Per subscription or per key | Per subscription or per key |
| **Bandwidth** | Not tracked | Can limit bandwidth (bytes) |

```xml
<!-- Rate limit: 100 calls per 60 seconds per subscription -->
<rate-limit calls="100" renewal-period="60" 
            retry-after-header-name="Retry-After"
            remaining-calls-header-name="X-RateLimit-Remaining" />

<!-- Rate limit by custom key (e.g., per IP) -->
<rate-limit-by-key calls="50" renewal-period="60"
                   counter-key="@(context.Request.IpAddress)"
                   retry-after-variable-name="retryAfter" />

<!-- Quota: 10000 calls per week per subscription, max 5MB bandwidth -->
<quota calls="10000" bandwidth="5242880" renewal-period="604800" />

<!-- Quota by custom key -->
<quota-by-key calls="5000" renewal-period="604800"
              counter-key="@(context.Subscription.Id)" />
```

**⚠️ Exam Tip:** `rate-limit` resets per sliding window. `quota` resets per fixed period. Use `rate-limit-by-key` / `quota-by-key` for custom counters (per IP, per user, etc.). HTTP 429 = rate limited. HTTP 403 = quota exceeded.

### 1.35 Context Object — Complete Reference

```
context
├── .Api
│   ├── .Id                    // API identifier
│   ├── .Name                  // API display name
│   ├── .Path                  // API URL suffix
│   ├── .ServiceUrl            // Backend service URL
│   ├── .Version               // API version
│   └── .Revision              // API revision
│
├── .Deployment
│   ├── .Region                // Azure region
│   ├── .ServiceName           // APIM instance name
│   ├── .ServiceUrl            // APIM gateway URL
│   └── .GatewayUrl            // Gateway URL
│
├── .Operation
│   ├── .Id                    // Operation identifier
│   ├── .Name                  // Operation display name
│   └── .Method                // HTTP method
│
├── .Product
│   ├── .Id                    // Product identifier
│   ├── .Name                  // Product display name
│   └── .Apis                  // APIs in the product
│
├── .Request
│   ├── .Body.As<T>()          // Body as type T (JObject, string, etc.)
│   ├── .Certificate           // Client certificate
│   ├── .Headers               // Request headers collection
│   ├── .IpAddress             // Client IP
│   ├── .MatchedParameters     // URL template parameters
│   ├── .Method                // HTTP method
│   ├── .OriginalUrl           // Original request URL
│   ├── .Url                   // Current request URL
│   └── .Url.Query             // Query parameters
│
├── .Response  (outbound/on-error only)
│   ├── .Body.As<T>()          // Response body
│   ├── .Headers               // Response headers
│   └── .StatusCode            // HTTP status code
│
├── .Subscription
│   ├── .Id                    // Subscription identifier
│   ├── .Key                   // Subscription key
│   ├── .Name                  // Subscription name
│   └── .CreatedDate           // Creation date
│
├── .User
│   ├── .Email                 // User email
│   ├── .FirstName / .LastName // Name
│   ├── .Groups                // User groups
│   ├── .Id                    // User identifier
│   └── .Identities            // Identity providers
│
├── .Variables                  // Custom variables (set-variable)
├── .RequestId                  // Unique request ID
├── .Elapsed                    // Time elapsed
├── .LastError (on-error only)
│   ├── .Message               // Error message
│   ├── .Reason                // Error reason
│   ├── .Source                // Error source
│   └── .Section               // Policy section where error occurred
│
└── .NamedValues                // Named values collection
```

### 1.36 APIM Multi-Region Deployment (Premium Only)

**What it provides:**
- Deploy API gateway to multiple Azure regions
- Reduce latency for global consumers
- Automatic failover for high availability
- Single management plane, multiple gateway instances

```
                    ┌──────── Management Plane (Primary Region) ────────┐
                    │                                                     │
        ┌───────────┴──────────┐                          ┌──────────────┴──────────┐
        │ Gateway (East US)    │                          │ Gateway (West Europe)    │
        │ (serves US clients)  │                          │ (serves EU clients)      │
        └──────────────────────┘                          └─────────────────────────┘
```

```bash
# Add secondary region
az apim update \
  --name myapim \
  --resource-group myRG \
  --set additionalLocations[0].location="West Europe" \
        additionalLocations[0].sku.name="Premium" \
        additionalLocations[0].sku.capacity=1
```

### 1.37 Response Caching — Advanced Patterns

**Cache Vary Strategies:**

```xml
<inbound>
    <cache-lookup vary-by-developer="true"
                  vary-by-developer-groups="true"
                  caching-type="prefer-external"
                  downstream-caching-type="public"
                  must-revalidate="true">
        <vary-by-header>Accept</vary-by-header>
        <vary-by-header>Accept-Encoding</vary-by-header>
        <vary-by-query-parameter>version</vary-by-query-parameter>
        <vary-by-query-parameter>format</vary-by-query-parameter>
    </cache-lookup>
</inbound>
<outbound>
    <cache-store duration="3600" cache-response="true" />
</outbound>
```

**External Cache (Redis):**
- Required for Consumption tier (no built-in cache)
- Optional for other tiers (prefer-external caching-type)

```bash
# Configure external cache
az apim external-cache create \
  --resource-group myRG \
  --service-name myapim \
  --connection-string "mycache.redis.cache.windows.net:6380,password=...,ssl=True" \
  --description "Redis cache"
```

**Cache Invalidation:**

```xml
<!-- Remove specific cached item -->
<cache-remove-value key="@("response_" + context.Request.Url.Path)" />
```

**Downstream Caching Headers:**

| `downstream-caching-type` | Behavior |
|---------------------------|----------|
| `none` | No Cache-Control header sent to client |
| `private` | `Cache-Control: private` — only client can cache |
| `public` | `Cache-Control: public` — any proxy/CDN can cache |

### 1.38 Handling SOAP APIs

**Import WSDL:**

```bash
az apim api import \
  --resource-group myRG \
  --service-name myapim \
  --api-id soap-api \
  --path /soap \
  --specification-format Wsdl \
  --specification-url "https://example.com/service?wsdl" \
  --soap-api-type "soap"      # Keep as SOAP
  # OR
  --soap-api-type "http"      # Convert SOAP-to-REST
```

**SOAP-to-REST Conversion:**

| Type | `soap-api-type` | Description |
|------|-----------------|-------------|
| Pass-through SOAP | `soap` | Forward SOAP requests as-is |
| SOAP-to-REST | `http` | Accept REST, convert to SOAP envelope |

```xml
<!-- Convert REST request to SOAP envelope -->
<set-body template="liquid">
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:web="http://example.com/webservice">
    <soapenv:Header/>
    <soapenv:Body>
        <web:GetUser>
            <web:UserId>{{body.userId}}</web:UserId>
        </web:GetUser>
    </soapenv:Body>
</soapenv:Envelope>
</set-body>
<set-header name="Content-Type" exists-action="override">
    <value>text/xml</value>
</set-header>
```

### 1.39 API Inspector & Tracing (Debugging)

**Ocp-Apim-Trace Header:**
- Send `Ocp-Apim-Trace: true` header to enable request tracing
- Returns detailed trace in response header: `Ocp-Apim-Trace-Location`
- Trace URL points to a blob with full policy execution details
- **Requires subscription key** with tracing enabled

```bash
# Test with tracing enabled
curl -H "Ocp-Apim-Subscription-Key: <key>" \
     -H "Ocp-Apim-Trace: true" \
     https://myapim.azure-api.net/api/users
```

**Trace Output Shows:**
- Each policy section execution (inbound, backend, outbound)
- Time taken per policy
- Variable values
- Backend request/response details
- Errors and their source

**⚠️ Exam Tip:** Tracing must be explicitly enabled on the subscription (not enabled by default for security). The `trace` policy writes to this output. Never leave tracing enabled in production — it exposes internal details.

### 1.40 Open APIs (No Subscription Required)

**Subscription Requirement:**
- By default, APIs require a subscription key
- Can be disabled at the Product or API level

```bash
# Create product without subscription requirement
az apim product create \
  --resource-group myRG \
  --service-name myapim \
  --product-id open-api \
  --display-name "Open API" \
  --subscription-required false \
  --state published

# Create API without subscription requirement
az apim api update \
  --resource-group myRG \
  --service-name myapim \
  --api-id myapi \
  --subscription-required false
```

**When to Use Open APIs:**
- Public read-only APIs
- Health check endpoints
- APIs protected by other means (JWT, IP filtering, client certificates)

**⚠️ Exam Tip:** Even without subscription keys, you can still protect APIs with `validate-jwt`, `ip-filter`, or client certificates. Subscription keys and OAuth/JWT are independent security layers.

### 1.41 Custom Domains

**Configurable Endpoints:**

| Endpoint | Default Domain | Custom Domain Example |
|----------|---------------|----------------------|
| **Gateway** | `myapim.azure-api.net` | `api.contoso.com` |
| **Developer Portal** | `myapim.developer.azure-api.net` | `developer.contoso.com` |
| **Management** | `myapim.management.azure-api.net` | `management.contoso.com` |
| **SCM (Git)** | `myapim.scm.azure-api.net` | — |

**Requirements:**
- Own the domain (DNS CNAME or A record)
- TLS/SSL certificate (upload PFX or use Key Vault reference)
- Premium tier: supports multiple custom gateway domains (per region)

```bash
# Set custom domain for gateway
az apim update \
  --name myapim \
  --resource-group myRG \
  --set hostnameConfigurations[0].type="Proxy" \
        hostnameConfigurations[0].hostName="api.contoso.com" \
        hostnameConfigurations[0].certificateSource="KeyVault" \
        hostnameConfigurations[0].keyVaultId="https://myvault.vault.azure.net/secrets/api-cert"
```

### 1.42 APIM Managed Identity

**System-Assigned vs User-Assigned on APIM:**

| Type | Use Case |
|------|----------|
| **System-assigned** | Single APIM instance; auto-created/deleted with instance |
| **User-assigned** | Share across resources; independent lifecycle |

**Common Uses of APIM Managed Identity:**
- Access Key Vault (for named values, certificates, custom domains)
- Authenticate to backend APIs (`authentication-managed-identity` policy)
- Access Event Hub (for `log-to-eventhub`)

```bash
# Enable system-assigned managed identity on APIM
az apim update \
  --name myapim \
  --resource-group myRG \
  --set identity.type="SystemAssigned"

# Then grant APIM access to Key Vault
az keyvault set-policy \
  --name myvault \
  --object-id <apim-principal-id> \
  --secret-permissions get list
```

### 1.43 Outbound IP Addresses

**Why It Matters:**
- Backends may need to whitelist APIM's outbound IP
- Outbound IPs differ per tier and region
- VNet mode changes outbound behavior

```bash
# Get APIM outbound IPs
az apim show \
  --name myapim \
  --resource-group myRG \
  --query "outboundPublicIPAddresses"
```

**Behavior by Tier:**
- **Consumption**: Shared IPs (unpredictable, may change)
- **Developer/Basic/Standard/Premium**: Dedicated outbound IPs (stable)
- **VNet (Internal)**: Outbound goes through VNet (use NSG/firewall)

### 1.44 APIM Soft-Delete and Recovery

- Deleted APIM instances are **soft-deleted** for 48 hours
- Instance name is reserved during soft-delete period
- Can be purged immediately or recovered

```bash
# List soft-deleted instances
az apim deletedservice list

# Recover (undelete) a soft-deleted instance
az apim deletedservice purge --service-name myapim --location "East US"
# Note: purge permanently deletes; to recover, use REST API or portal
```

**⚠️ Exam Tip:** Soft-delete is automatic. You cannot reuse the same APIM name until the soft-delete period expires or the instance is purged.

### 1.45 stv1 vs stv2 Platform

**APIM Platform Versions:**

| Aspect | stv1 (Legacy) | stv2 (Current) |
|--------|--------------|----------------|
| **Compute** | Cloud Services | Virtual Machine Scale Sets |
| **Availability Zones** | ❌ | ✅ (Premium) |
| **VNet v2** | ❌ | ✅ |
| **Private endpoint** | ❌ | ✅ |
| **Status** | Deprecated (retirement announced) | Recommended |

**⚠️ Exam Tip:** Microsoft is migrating all APIM instances from stv1 to stv2. New instances are stv2 by default. If a question mentions availability zones or private endpoints, the answer requires stv2/Premium.

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**1. API Management Components:**
- Gateway, Management Plane, Developer Portal
- Purpose of each component
- How they interact

**2. Policies:**
- Policy sections (inbound, backend, outbound, on-error)
- Common policies and their use cases
- Policy expressions and context object
- Order of execution with `<base>`
- Named values (`{{name}}` syntax)
- Policy fragments (`<include-fragment />`)

**3. Products, Groups, and Subscriptions:**
- Product states (published/not published)
- Built-in groups (Administrators, Developers, Guests)
- Subscription scopes (All APIs, Single API, Product)
- Subscription keys (header, query string)

**4. Security:**
- Authentication methods (subscription keys, JWT, certificates, IP, managed identity)
- `validate-jwt` policy
- Rate limiting vs quotas
- IP filtering
- Client certificate validation (properties and `.Verify()`)

**5. Versioning and Revisions:**
- When to use versions vs revisions
- Versioning schemes (Path, Query, Header)

**6. Tiers:**
- Consumption (serverless, cold start, no VNet, external cache only)
- Developer (no SLA, VNet support)
- Premium (multi-region, VNet, availability zones)

**7. Advanced Policies:**
- `wait` (parallel execution — `for="all"` vs `for="any"`)
- `send-one-way-request` (fire and forget)
- `limit-concurrency` (throttle parallel requests)
- `validate-content/parameters/headers/status-code`
- Liquid templates (`template="liquid"`)
- Caching (cache-lookup + cache-store, external Redis)

**8. Integrations:**
- Application Insights (diagnostics, logging)
- Event Hubs (`log-to-eventhub`)
- Azure Functions / Logic Apps / App Service as backends
- Azure AD OAuth 2.0 in developer portal

### Sample Exam Questions

#### API Management Basics

**Q1:** Which component of API Management handles policy execution and request routing?
- A) Management Plane
- B) Developer Portal
- C) API Gateway
- D) Azure Portal

**Answer: C) API Gateway**
Explanation: The API Gateway is the runtime component that routes requests, executes policies, and enforces security.

---

**Q2:** Which tier supports multi-region deployment?
- A) Consumption
- B) Developer
- C) Standard
- D) Premium

**Answer: D) Premium**
Explanation: Only Premium tier supports multi-region deployment for global scale and high availability.

---

**Q3:** Which tier has no built-in cache but can use external cache?
- A) Basic
- B) Standard
- C) Consumption
- D) Developer

**Answer: C) Consumption**
Explanation: The Consumption tier doesn't include built-in cache but supports external Redis cache.

---

#### Policies

**Q4:** In which policy section should you place rate limiting?
- A) `<backend>`
- B) `<outbound>`
- C) `<inbound>`
- D) `<on-error>`

**Answer: C) `<inbound>`**
Explanation: Rate limiting should be applied in the inbound section before the request reaches the backend.

---

**Q5:** What does the `<base />` element do in a policy?
- A) Resets all policies
- B) Includes policies from parent scope
- C) Sets the base URL
- D) Enables caching

**Answer: B) Includes policies from parent scope**
Explanation: The `<base />` element includes policies defined at parent scopes (global, product, API).

---

**Q6:** Which policy converts XML response to JSON?
- A) `<set-body>`
- B) `<json-to-xml>`
- C) `<xml-to-json>`
- D) `<find-and-replace>`

**Answer: C) `<xml-to-json>`**
Explanation: The `<xml-to-json>` policy transforms XML responses to JSON format.

---

**Q7:** You need to return a mock response without calling the backend. Which policy should you use?
- A) `<set-body>`
- B) `<return-response>`
- C) `<forward-request>`
- D) `<cache-lookup>`

**Answer: B) `<return-response>`**
Explanation: The `<return-response>` policy immediately returns a response without forwarding to the backend.

---

**Q8:** What is the correct order of policy execution?
- A) Operation → API → Product → Global
- B) Global → Product → API → Operation
- C) Product → Global → API → Operation
- D) API → Product → Global → Operation

**Answer: B) Global → Product → API → Operation**
Explanation: Policies execute from broadest to narrowest scope: Global → Product → API → Operation.

---

#### Subscriptions and Products

**Q9:** What is the default header name for subscription keys?
- A) `X-API-Key`
- B) `Authorization`
- C) `Ocp-Apim-Subscription-Key`
- D) `Subscription-Key`

**Answer: C) `Ocp-Apim-Subscription-Key`**
Explanation: By default, subscription keys are passed in the `Ocp-Apim-Subscription-Key` header.

---

**Q10:** A product must be in which state to be visible in the developer portal?
- A) Active
- B) Published
- C) Released
- D) Deployed

**Answer: B) Published**
Explanation: Products must be Published to be visible and subscribable in the developer portal.

---

**Q11:** Which subscription scope provides access to all APIs in the instance?
- A) Product
- B) Single API
- C) All APIs
- D) Global

**Answer: C) All APIs**
Explanation: The "All APIs" scope provides access to every API in the API Management instance.

---

#### Security

**Q12:** Which policy should you use to validate OAuth 2.0 tokens?
- A) `<authentication-basic>`
- B) `<validate-jwt>`
- C) `<check-header>`
- D) `<authentication-certificate>`

**Answer: B) `<validate-jwt>`**
Explanation: The `<validate-jwt>` policy validates JWT tokens from OAuth 2.0/OpenID Connect.

---

**Q13:** What is the difference between rate-limit and quota policies?
- A) Rate-limit is per user, quota is per subscription
- B) Rate-limit is short-term, quota is long-term limits
- C) Rate-limit counts bytes, quota counts calls
- D) They are identical

**Answer: B) Rate-limit is short-term, quota is long-term limits**
Explanation: Rate-limit handles short bursts (calls per second/minute), quota handles longer periods (per day/week).

---

**Q14:** Which policy allows/denies requests based on IP address?
- A) `<check-header>`
- B) `<ip-filter>`
- C) `<rate-limit>`
- D) `<access-restriction>`

**Answer: B) `<ip-filter>`**
Explanation: The `<ip-filter>` policy controls access based on client IP addresses.

---

#### Versioning and Revisions

**Q15:** When should you create a new API version instead of a revision?
- A) Adding an optional field
- B) Fixing a bug
- C) Changing the URL structure (breaking change)
- D) Updating documentation

**Answer: C) Changing the URL structure (breaking change)**
Explanation: Versions are for breaking changes. Revisions are for non-breaking changes.

---

**Q16:** Which versioning scheme puts the version in the URL path?
- A) Query
- B) Header
- C) Path
- D) Body

**Answer: C) Path**
Explanation: Path versioning includes the version in the URL (e.g., `/v1/users`).

---

#### Advanced Scenarios

**Q17:** How do you authenticate to a backend API using managed identity?
- A) `<authentication-basic>`
- B) `<authentication-managed-identity>`
- C) `<validate-jwt>`
- D) `<set-header>`

**Answer: B) `<authentication-managed-identity>`**
Explanation: The `<authentication-managed-identity>` policy obtains tokens using APIM's managed identity.

---

**Q18:** What is the self-hosted gateway used for?
- A) Development only
- B) Running gateway close to backend APIs or on-premises
- C) Caching responses
- D) User authentication

**Answer: B) Running gateway close to backend APIs or on-premises**
Explanation: Self-hosted gateway enables hybrid and multi-cloud scenarios by running the gateway container near backend services.

---

**Q19:** Which policy section handles errors?
- A) `<inbound>`
- B) `<backend>`
- C) `<outbound>`
- D) `<on-error>`

**Answer: D) `<on-error>`**
Explanation: The `<on-error>` section handles policy execution errors.

---

**Q20:** You want to cache API responses for 1 hour. Which policies do you need?
- A) `<cache-store>` only
- B) `<cache-lookup>` only
- C) `<cache-lookup>` in inbound, `<cache-store>` in outbound
- D) `<cache-store>` in inbound, `<cache-lookup>` in outbound

**Answer: C) `<cache-lookup>` in inbound, `<cache-store>` in outbound**
Explanation: Cache-lookup checks the cache before calling backend (inbound), cache-store saves the response after (outbound).

---

#### Named Values and Fragments

**Q21:** How do you reference a named value called "api-key" in a policy?
- A) `@(api-key)`
- B) `${api-key}`
- C) `{{api-key}}`
- D) `#{api-key}`

**Answer: C) `{{api-key}}`**
Explanation: Named values use double curly braces `{{name}}` syntax in policies.

---

**Q22:** A named value backed by Azure Key Vault requires what additional configuration?
- A) APIM must have a client secret
- B) APIM must have a managed identity with Key Vault access
- C) Key Vault must be in the same resource group
- D) Key Vault must use access policies, not RBAC

**Answer: B) APIM must have a managed identity with Key Vault access**
Explanation: Key Vault-backed named values require APIM's managed identity to have read access to the Key Vault secrets.

---

**Q23:** How do you include a policy fragment named "standard-headers" in a policy?
- A) `<policy-fragment name="standard-headers" />`
- B) `<include-fragment fragment-id="standard-headers" />`
- C) `<import fragment="standard-headers" />`
- D) `{{standard-headers}}`

**Answer: B) `<include-fragment fragment-id="standard-headers" />`**
Explanation: Policy fragments are included using the `<include-fragment fragment-id="name" />` syntax.

---

#### Groups and Products

**Q24:** Which built-in APIM group represents unauthenticated developer portal visitors?
- A) Developers
- B) Administrators
- C) Guests
- D) Anonymous

**Answer: C) Guests**
Explanation: The Guests group represents unauthenticated visitors to the developer portal. They have read-only access to some content.

---

**Q25:** What controls which users can see and subscribe to a product?
- A) Subscription keys
- B) IP filtering
- C) Groups
- D) Named values

**Answer: C) Groups**
Explanation: Products are granted to groups. Only users in those groups can see and subscribe to the product.

---

#### VNet and Tiers

**Q26:** You need APIM to be accessible ONLY from within a virtual network. Which configuration should you use?
- A) External VNet mode
- B) Internal VNet mode
- C) Private endpoint
- D) IP filtering

**Answer: B) Internal VNet mode**
Explanation: Internal VNet mode makes the APIM gateway accessible only from within the VNet (requires VPN/ExpressRoute for external access).

---

**Q27:** Which tier supports VNet integration?
- A) Consumption and Basic
- B) Basic and Standard
- C) Developer and Premium
- D) All tiers

**Answer: C) Developer and Premium**
Explanation: VNet integration is available only in Developer (for testing) and Premium (for production) tiers.

---

**Q28:** Your company needs API Management with no idle costs and automatic scaling. Which tier should you choose?
- A) Developer
- B) Basic
- C) Consumption
- D) Standard

**Answer: C) Consumption**
Explanation: Consumption tier is pay-per-call with no idle costs and automatic scaling. It's the serverless option.

---

**Q29:** Which tier supports multi-region deployment?
- A) Standard
- B) Developer
- C) Basic
- D) Premium

**Answer: D) Premium**
Explanation: Only Premium tier supports multi-region deployment for global distribution and high availability.

---

#### Advanced Policies

**Q30:** You need to call two backend services in parallel and combine their responses. Which policy should you use?
- A) `<send-request>` twice sequentially
- B) `<wait for="all">` with two `<send-request>` inside
- C) `<retry>`
- D) `<forward-request>` twice

**Answer: B) `<wait for="all">` with two `<send-request>` inside**
Explanation: The `<wait>` policy executes multiple send-request calls in parallel. `for="all"` waits for all to complete.

---

**Q31:** What is the difference between `<send-request>` and `<send-one-way-request>`?
- A) `send-request` is async, `send-one-way-request` is sync
- B) `send-request` waits for response, `send-one-way-request` is fire-and-forget
- C) `send-one-way-request` sends to multiple backends
- D) No difference

**Answer: B) `send-request` waits for response, `send-one-way-request` is fire-and-forget**
Explanation: `send-one-way-request` sends the request but doesn't wait for or store the response. Useful for notifications/webhooks.

---

**Q32:** Which policy limits the number of parallel API calls being processed?
- A) `<rate-limit>`
- B) `<quota>`
- C) `<limit-concurrency>`
- D) `<throttle>`

**Answer: C) `<limit-concurrency>`**
Explanation: `limit-concurrency` limits how many requests are processed concurrently. `rate-limit` limits calls per time period (different concept).

---

**Q33:** In the `<wait>` policy, what does `for="any"` mean?
- A) Wait for all requests to complete
- B) Proceed when any one request completes
- C) Skip all requests
- D) Retry any failed request

**Answer: B) Proceed when any one request completes**
Explanation: `for="any"` proceeds as soon as the first request completes. `for="all"` waits for every request to finish.

---

#### Certificates

**Q34:** Which property validates the full certificate chain trust in a policy expression?
- A) `context.Request.Certificate.IsValid`
- B) `context.Request.Certificate.Verify()`
- C) `context.Request.Certificate.Validate()`
- D) `context.Request.Certificate.TrustChain`

**Answer: B) `context.Request.Certificate.Verify()`**
Explanation: The `.Verify()` method validates the certificate chain trust. `.Thumbprint` checks the specific certificate identity.

---

#### Monitoring

**Q35:** Which policy sends custom log data to Azure Event Hub?
- A) `<emit-metric>`
- B) `<trace>`
- C) `<log-to-eventhub>`
- D) `<send-request>`

**Answer: C) `<log-to-eventhub>`**
Explanation: `log-to-eventhub` sends data to an Event Hub logger. `emit-metric` sends custom metrics to Azure Monitor. `trace` writes to API Inspector.

---

**Q36:** Which policy sends custom metrics to Azure Monitor?
- A) `<log-to-eventhub>`
- B) `<emit-metric>`
- C) `<trace>`
- D) `<application-insights>`

**Answer: B) `<emit-metric>`**
Explanation: The `emit-metric` policy publishes custom metrics to Azure Monitor with custom dimensions.

---

#### Liquid Templates

**Q37:** How do you enable Liquid templating in a `<set-body>` policy?
- A) `<set-body language="liquid">`
- B) `<set-body template="liquid">`
- C) `<set-body format="liquid">`
- D) `<set-body type="liquid">`

**Answer: B) `<set-body template="liquid">`**
Explanation: The `template="liquid"` attribute enables Liquid template processing in the set-body policy.

---

**Q38:** In a Liquid template, which syntax is used for iterating over a JSON array?
- A) `{% for item in body.items %}`
- B) `{% JSONArrayFor item in body.items %}`
- C) `{% foreach item in body.items %}`
- D) `{% loop item in body.items %}`

**Answer: B) `{% JSONArrayFor item in body.items %}`**
Explanation: APIM uses the custom `{% JSONArrayFor %}` tag for JSON array iteration in Liquid templates, not standard `{% for %}`.

---

#### OAuth and Security

**Q39:** You configure OAuth 2.0 in the developer portal. Is the API automatically protected?
- A) Yes, all requests require a valid token
- B) No, you must also add a `validate-jwt` policy
- C) Yes, but only for published products
- D) No, OAuth is disabled by default

**Answer: B) No, you must also add a `validate-jwt` policy**
Explanation: Configuring OAuth 2.0 in the developer portal only enables the "Try it" console to get tokens. You MUST add `validate-jwt` policy to actually validate tokens.

---

**Q40:** How does APIM authenticate to a backend service using managed identity?
- A) `<validate-jwt>`
- B) `<authentication-managed-identity resource="..." />`
- C) `<set-header name="Authorization">`
- D) `<authentication-basic>`

**Answer: B) `<authentication-managed-identity resource="..." />`**
Explanation: The `authentication-managed-identity` policy uses APIM's managed identity to obtain an access token for the specified backend resource.

---

#### Rate Limit vs Quota

**Q41:** A user exceeds the rate limit. What HTTP status code is returned?
- A) 401 Unauthorized
- B) 403 Forbidden
- C) 429 Too Many Requests
- D) 503 Service Unavailable

**Answer: C) 429 Too Many Requests**
Explanation: Rate limiting returns 429 (Too Many Requests). Quota exceeded returns 403 (Forbidden).

---

**Q42:** You need to limit each IP address to 50 calls per minute, regardless of subscription. Which policy?
- A) `<rate-limit calls="50" renewal-period="60" />`
- B) `<rate-limit-by-key calls="50" renewal-period="60" counter-key="@(context.Request.IpAddress)" />`
- C) `<quota calls="50" renewal-period="60" />`
- D) `<limit-concurrency max-count="50" />`

**Answer: B) `<rate-limit-by-key calls="50" renewal-period="60" counter-key="@(context.Request.IpAddress)" />`**
Explanation: `rate-limit-by-key` with `counter-key` set to the IP address limits per-IP. Standard `rate-limit` is per subscription.

---

#### Context Object

**Q43:** How do you access the client's IP address in a policy expression?
- A) `@(context.User.IpAddress)`
- B) `@(context.Request.IpAddress)`
- C) `@(context.Request.Headers["X-Forwarded-For"])`
- D) `@(context.Connection.RemoteAddress)`

**Answer: B) `@(context.Request.IpAddress)`**
Explanation: `context.Request.IpAddress` provides the client IP address in policy expressions.

---

**Q44:** In the `<on-error>` section, how do you access the error message?
- A) `@(context.Error.Message)`
- B) `@(context.LastError.Message)`
- C) `@(context.Exception.Message)`
- D) `@(context.Response.Error)`

**Answer: B) `@(context.LastError.Message)`**
Explanation: `context.LastError` is available in the `<on-error>` section and contains `.Message`, `.Reason`, `.Source`, and `.Section`.

---

#### SOAP and Transformations

**Q45:** You need to expose a SOAP web service as a REST API through APIM. What `soap-api-type` should you use during import?
- A) `soap`
- B) `rest`
- C) `http`
- D) `wsdl`

**Answer: C) `http`**
Explanation: Setting `soap-api-type` to `http` during WSDL import converts SOAP operations to REST-like operations. `soap` keeps them as-is.

---

#### Caching

**Q46:** The Consumption tier needs caching. What should you configure?
- A) Built-in cache with 10 MB limit
- B) External Azure Cache for Redis
- C) Caching is not supported in Consumption tier
- D) Azure CDN

**Answer: B) External Azure Cache for Redis**
Explanation: Consumption tier has no built-in cache. You must configure an external Azure Cache for Redis for caching.

---

**Q47:** What does `downstream-caching-type="private"` do in `cache-lookup`?
- A) Only APIM caches the response
- B) Only the client can cache the response (sets `Cache-Control: private`)
- C) Only the backend can cache
- D) Disables caching

**Answer: B) Only the client can cache the response (sets `Cache-Control: private`)**
Explanation: `private` means only the end client can cache (not intermediate proxies/CDNs). `public` allows any cache to store it.

---

#### Subscriptions

**Q48:** A subscription can be scoped to which levels?
- A) API, Operation, Product
- B) All APIs, Single API, Product
- C) Global, Regional, Local
- D) All APIs, Product, User

**Answer: B) All APIs, Single API, Product**
Explanation: Subscriptions can be scoped to All APIs, a single API, or a Product.

---

#### Versioning and Revisions

**Q49:** You need to test a policy change before making it live. What should you use?
- A) API Version
- B) API Revision
- C) Product
- D) Named Value

**Answer: B) API Revision**
Explanation: Revisions allow testing non-breaking changes before making them the current version. Versions are for breaking changes.

---

**Q50:** Which versioning scheme uses a URL like `/v1/users`?
- A) Query
- B) Header
- C) Path
- D) Body

**Answer: C) Path**
Explanation: Path versioning embeds the version in the URL path (e.g., `/v1/users`).

---

## Exam Gotchas & Tricky Distinctions

| Common Misconception | Reality |
|---------------------|---------|
| "OAuth 2.0 in developer portal protects the API" | ❌ It only enables "Try it" with tokens. You MUST add `validate-jwt` policy |
| "Rate-limit and quota are the same" | ❌ Rate-limit = short bursts (429). Quota = long-term total (403) |
| "Consumption tier has built-in cache" | ❌ Must use external Redis cache |
| "VNet is available in all tiers" | ❌ Only Developer and Premium |
| "Multi-region is available in Standard" | ❌ Only Premium |
| "Removing `<base>` has no effect" | ❌ Removing `<base>` OVERRIDES parent policies entirely |
| "Named values with `secret=true` can be exported" | ❌ Secret named values are one-way (cannot be read back) |
| "Policy execution order: API → Product → Global" | ❌ Correct order: Global → Product → API → Operation |
| "Contributor can manage APIM subscriptions" | ❌ Contributor manages APIM resources; Developers manage their own portal subscriptions |
| "`rate-limit` works per IP by default" | ❌ Default `rate-limit` is per subscription. Use `rate-limit-by-key` for per-IP |
| "`send-one-way-request` stores a response" | ❌ Fire-and-forget — no response is captured |
| "`wait for='any'` waits for all" | ❌ `for="any"` = first one wins. `for="all"` = wait for all |
| "Liquid uses `{% for %}` for JSON arrays" | ❌ Use `{% JSONArrayFor %}` for JSON arrays |
| "Client cert validation is automatic" | ❌ Must write policy to check thumbprint, issuer, or call `.Verify()` |
| "`limit-concurrency` is the same as `rate-limit`" | ❌ `limit-concurrency` = max parallel requests. `rate-limit` = max calls per time period |

---

## Key CLI Commands Reference

### API Management Instance
```bash
# Create APIM instance
az apim create \
  --name myapim \
  --resource-group myRG \
  --publisher-email admin@example.com \
  --publisher-name "My Company" \
  --sku-name Developer

# List APIM instances
az apim list --resource-group myRG

# Delete APIM instance
az apim delete --name myapim --resource-group myRG
```

### APIs
```bash
# Create API
az apim api create \
  --service-name myapim \
  --resource-group myRG \
  --api-id myapi \
  --display-name "My API" \
  --path /myapi

# Import API
az apim api import \
  --service-name myapim \
  --resource-group myRG \
  --api-id myapi \
  --path /myapi \
  --specification-format OpenApi \
  --specification-url "https://example.com/swagger.json"

# List APIs
az apim api list --service-name myapim --resource-group myRG

# Delete API
az apim api delete --service-name myapim --resource-group myRG --api-id myapi
```

### Products
```bash
# Create product
az apim product create \
  --service-name myapim \
  --resource-group myRG \
  --product-id starter \
  --display-name "Starter" \
  --subscription-required true \
  --state published

# Add API to product
az apim product api add \
  --service-name myapim \
  --resource-group myRG \
  --product-id starter \
  --api-id myapi
```

### Subscriptions
```bash
# Create subscription
az apim subscription create \
  --service-name myapim \
  --resource-group myRG \
  --subscription-id mysub \
  --display-name "My Subscription" \
  --scope "/products/starter"

# Show subscription keys
az apim subscription keys list \
  --service-name myapim \
  --resource-group myRG \
  --subscription-id mysub
```

---

## Study Tips for AZ-204 Exam

1. **Know policy placement** — Inbound, backend, outbound, on-error — and execution order (Global → Product → API → Operation)
2. **Understand `<base>` element** — Position matters; removing it overrides parent policies
3. **Master common policies** — rate-limit, quota, validate-jwt, set-header, cors, cache-lookup/store
4. **Know tier differences** — Consumption (serverless, no VNet, external cache) vs Premium (multi-region, VNet, zones)
5. **Understand subscription scopes** — All APIs, Single API, Product
6. **Version vs Revision** — Versions = breaking changes (different URL); Revisions = non-breaking (same URL, test first)
7. **Policy expressions** — `context.Request`, `context.Response`, `context.User`, `context.Variables`, `context.LastError`
8. **Security options** — JWT validation, client certificates (.Thumbprint, .Verify()), IP filtering, managed identity to backend
9. **Named values** — `{{name}}` syntax; Plain, Secret, Key Vault-backed; APIM needs managed identity for Key Vault
10. **Groups model** — Products → Groups → Users; built-in: Administrators, Developers, Guests
11. **Caching** — cache-lookup (inbound) + cache-store (outbound); Consumption tier = external Redis only
12. **Rate-limit vs Quota** — rate-limit = short burst (429); quota = long-term total (403); use `-by-key` for custom counters
13. **Parallel execution** — `<wait for="all">` or `<wait for="any">` with `<send-request>` inside
14. **Liquid templates** — `template="liquid"` on set-body; `{% JSONArrayFor %}` for arrays
15. **OAuth 2.0 in dev portal** — Does NOT protect API; must add `validate-jwt` policy separately

---

## Important URLs and References

- API Management Documentation: https://learn.microsoft.com/en-us/azure/api-management/
- Policy Reference: https://learn.microsoft.com/en-us/azure/api-management/api-management-policies
- Policy Expressions: https://learn.microsoft.com/en-us/azure/api-management/api-management-policy-expressions
- Named Values: https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-properties
- Pricing: https://azure.microsoft.com/en-us/pricing/details/api-management/

---

## Quick Reference Summary

### Components
- **API Gateway**: Request routing, policy execution, auth, caching, logging
- **Management Plane**: Configure APIs, policies, users, products (Azure Portal / REST API)
- **Developer Portal**: API discovery, documentation, "Try it" console, subscriptions

### Policy Sections & Execution Order
- **Inbound**: Before backend (auth, rate-limit, transform request)
- **Backend**: Backend configuration (routing, retry, managed identity)
- **Outbound**: After backend (transform response, caching)
- **On-error**: Error handling (`context.LastError`)
- **Order**: Global → Product → API → Operation (for inbound); reverse for outbound
- **`<base />`**: Position controls order; omitting = override parent

### Key Policies Cheat Sheet

| Policy | Section | Purpose |
|--------|---------|---------|
| `rate-limit` / `rate-limit-by-key` | Inbound | Short-term call limits (429) |
| `quota` / `quota-by-key` | Inbound | Long-term call/bandwidth limits (403) |
| `validate-jwt` | Inbound | OAuth/OIDC token validation |
| `ip-filter` | Inbound | Allow/deny by IP |
| `cors` | Inbound | Cross-origin resource sharing |
| `set-header` | Inbound/Outbound | Add/modify/remove headers |
| `set-body` | Inbound/Outbound | Transform body (C# or Liquid) |
| `rewrite-uri` | Inbound | Change request path |
| `set-backend-service` | Inbound | Route to different backend |
| `cache-lookup` | Inbound | Check cache |
| `cache-store` | Outbound | Store in cache |
| `forward-request` | Backend | Control backend request |
| `retry` | Backend | Retry failed requests |
| `send-request` | Any | Make additional HTTP call (waits for response) |
| `send-one-way-request` | Any | Fire-and-forget HTTP call |
| `wait` | Any | Parallel execution (`for="all"` / `for="any"`) |
| `limit-concurrency` | Any | Max parallel requests |
| `return-response` | Any | Return immediately (mock/short-circuit) |
| `choose` | Any | Conditional logic (if/else) |
| `set-variable` | Any | Store custom values |
| `validate-content` | Inbound | Validate request body schema |
| `log-to-eventhub` | Any | Send data to Event Hub |
| `emit-metric` | Any | Custom Azure Monitor metrics |
| `trace` | Any | Debugging output to API Inspector |
| `authentication-managed-identity` | Backend | Authenticate to backend via managed identity |
| `authentication-certificate` | Backend | Authenticate to backend via certificate |
| `include-fragment` | Any | Include a policy fragment |
| `xml-to-json` / `json-to-xml` | Outbound | Format conversion |
| `mock-response` | Inbound | Return mock data |

### Named Values
- Referenced with `{{name}}` in policies
- Types: Plain (visible), Secret (hidden), Key Vault (reference)
- Key Vault-backed requires APIM managed identity

### Subscription Keys
- Primary header: `Ocp-Apim-Subscription-Key`
- Alternative: `Subscription-Key`
- Query: `subscription-key`
- Scopes: All APIs, Single API, Product

### Groups (Access Model)
- **Administrators**: Full APIM control
- **Developers**: Authenticated portal users, create subscriptions
- **Guests**: Unauthenticated visitors, read-only
- Products → granted to Groups → Users subscribe

### Tiers

| Feature | Consumption | Developer | Basic | Standard | Premium |
|---------|-------------|-----------|-------|----------|---------|
| SLA | 99.95% | None | 99.95% | 99.95% | 99.99% |
| VNet | ❌ | ✅ | ❌ | ❌ | ✅ |
| Multi-region | ❌ | ❌ | ❌ | ❌ | ✅ |
| Built-in cache | ❌ | ✅ | ✅ | ✅ | ✅ |
| Cold start | ✅ | ❌ | ❌ | ❌ | ❌ |
| Pricing | Per call | Monthly | Monthly | Monthly | Monthly |

### Versioning vs Revisions
- **Version**: Breaking changes, different URL, new API version (Path/Query/Header scheme)
- **Revision**: Non-breaking, same URL, test before release, rollback possible

### Security Summary
- **Subscription keys**: Default, header or query
- **OAuth 2.0 JWT**: `validate-jwt` policy (openid-config, audiences, issuers, claims)
- **Client certificates**: `.Thumbprint`, `.Issuer`, `.Verify()` in policy expressions
- **IP filtering**: `ip-filter` policy (allow/forbid)
- **Managed identity to backend**: `authentication-managed-identity` policy
- **Certificate to backend**: `authentication-certificate` policy

### Rate Limit vs Quota
- **rate-limit**: Short period (seconds/minutes), resets rolling, 429 response
- **quota**: Long period (days/weeks), can limit bandwidth, 403 response
- **`-by-key`**: Custom counter (per IP, per user, per custom key)

### Liquid Templates
- Enable: `template="liquid"` on `<set-body>`
- Array iteration: `{% JSONArrayFor item in body.items %}`
- Filters: `upcase`, `downcase`, `replace`, `size`, `times`, `date`

### Client Certificate Properties
- `.Thumbprint`, `.Subject`, `.Issuer`, `.NotBefore`, `.NotAfter`, `.SerialNumber`, `.Verify()`

