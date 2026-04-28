# AZ-204: Implement User Authentication and Authorization - Complete Study Notes (Exam-Ready Edition)

## Learning Path Overview
This learning path covers implementing authentication and authorization in Azure with 4 modules:
1. Explore the Microsoft identity platform
2. Implement authentication by using the Microsoft Authentication Library (MSAL)
3. Implement shared access signatures (SAS)
4. Explore Microsoft Graph

**Level:** Intermediate | **Focus:** Identity and Access

> **⚠️ Note:** This document includes topics BEYOND the MS Learn career path — covering Azure AD Fundamentals (editions, objects, hybrid identity, MFA, SSO, B2B/B2C, PIM, Identity Protection, Security Defaults), Azure RBAC (role definitions, custom roles, deny assignments, control vs data plane, Actions/DataActions), Managed Identities, Azure Key Vault, App Service Easy Auth, JWT/Token deep dive, App Roles, Claims-based Auth, DefaultAzureCredential, Microsoft.Identity.Web, Token Validation, Multi-tenant patterns, App Manifest, Conditional Access, SAML vs OIDC, and Zero Trust.

---

## FOUNDATIONAL CONCEPTS: AZURE AD & RBAC FUNDAMENTALS

### F.1 What is Azure AD (Microsoft Entra ID)?

**Azure Active Directory = Microsoft's cloud-based identity and access management service**

- **NOT** the same as on-premises Active Directory Domain Services (AD DS)
- Renamed to **Microsoft Entra ID** (but Azure AD still widely used on exams)
- Every Azure subscription **trusts** one Azure AD tenant
- Provides: authentication, authorization, SSO, MFA, conditional access, app management

**Core Functions:**

| Function | Description |
|----------|-------------|
| **Authentication** | Verify who you are (identity) |
| **Authorization** | Verify what you can do (permissions) |
| **Single Sign-On (SSO)** | One login for multiple applications |
| **Multi-Factor Authentication (MFA)** | Extra verification beyond password |
| **Application Management** | Register and manage apps |
| **Device Management** | Register and manage devices |
| **B2B Collaboration** | Guest users from other organizations |
| **B2C Identity** | Consumer-facing identity for apps |

### F.2 Authentication vs Authorization

| Concept | Authentication (AuthN) | Authorization (AuthZ) |
|---------|----------------------|---------------------|
| **Question** | "Who are you?" | "What can you do?" |
| **Protocol** | OpenID Connect (OIDC) | OAuth 2.0 |
| **Token** | ID Token | Access Token |
| **Result** | Identity verified | Permissions granted |
| **Example** | User logs in with password + MFA | User can read blobs but not delete |

```
User → Authenticates (OIDC) → Gets ID Token (who you are)
                              → Gets Access Token (what you can do)
                                    → Sends Access Token to API
                                          → API authorizes based on token claims
```

**⚠️ Exam Tip:** OAuth 2.0 = **authorization** protocol (grants access). OpenID Connect = **authentication** layer ON TOP of OAuth 2.0 (proves identity). They work together.

### F.3 Azure AD Objects

**Users:**
- Represent people in the organization
- Types: Cloud-only users, Synced users (from on-prem AD), Guest users (B2B)
- Each user has an Object ID (GUID), UPN (user@domain.com), and Display Name

**Groups:**

| Group Type | Purpose | Members Can Be |
|-----------|---------|---------------|
| **Security Group** | Manage access to resources | Users, devices, service principals, other groups |
| **Microsoft 365 Group** | Collaboration (Teams, SharePoint) | Users only |
| **Dynamic Group** | Auto-membership based on attributes | Defined by rules (e.g., department = "Sales") |

```bash
# Create security group
az ad group create --display-name "Developers" --mail-nickname "developers"

# Add member to group
az ad group member add --group "Developers" --member-id <user-object-id>
```

**Enterprise Applications vs App Registrations:**

| Concept | App Registration | Enterprise Application |
|---------|-----------------|----------------------|
| **What** | The app's identity definition (global) | The app's local presence in a tenant |
| **Maps to** | Application Object | Service Principal |
| **Where** | Home tenant only | Every tenant where app is used |
| **Configure** | Redirect URIs, credentials, API permissions | User assignments, SSO, conditional access |
| **Analogy** | The app's "blueprint" | The app's "installation" in a tenant |

### F.4 Azure AD Editions

| Edition | Key Features |
|---------|-------------|
| **Free** | User/group management, SSO (up to 10 apps), basic reports, self-service password change |
| **P1** | All Free + dynamic groups, self-service password reset, conditional access, hybrid identity (Azure AD Connect), unlimited SSO |
| **P2** | All P1 + Identity Protection (risk-based policies), Privileged Identity Management (PIM), access reviews |

**⚠️ Exam Tip:** Conditional Access requires at least **P1** license. PIM and Identity Protection require **P2**.

### F.5 Single Sign-On (SSO)

- User authenticates ONCE, then accesses multiple applications without re-entering credentials
- Azure AD supports SSO for thousands of pre-integrated SaaS apps (Salesforce, ServiceNow, etc.)
- Custom apps use OIDC/OAuth 2.0 or SAML for SSO
- Azure AD acts as the central identity provider

```
User signs in once to Azure AD
    ├── App 1 (SSO — no login prompt)
    ├── App 2 (SSO — no login prompt)
    └── App 3 (SSO — no login prompt)
```

### F.6 Multi-Factor Authentication (MFA)

**Something you know + Something you have + Something you are**

| Factor | Type | Examples |
|--------|------|---------|
| Knowledge | Something you know | Password, PIN, security questions |
| Possession | Something you have | Phone, hardware key, authenticator app |
| Biometric | Something you are | Fingerprint, face recognition, iris scan |

**Azure AD MFA Methods:**
- Microsoft Authenticator app (push notification or code)
- SMS verification
- Phone call
- FIDO2 security keys
- Windows Hello for Business
- Certificate-based authentication

**MFA is enforced via:**
- Security Defaults (basic, free)
- Conditional Access policies (flexible, requires P1)
- Per-user MFA (legacy)

### F.7 Guest Users (Azure AD B2B)

- External users invited to your Azure AD tenant
- Can access your apps and resources
- Authenticate with their OWN identity provider (their org's Azure AD, Google, etc.)
- Appear as `user@externaldomain.com#EXT#@yourtenant.onmicrosoft.com`

```bash
# Invite guest user
az ad user create --display-name "External Partner" \
  --user-principal-name "partner@external.com" \
  --invite-redirect-url "https://myapp.com"
```

**B2B vs B2C:**

| Aspect | B2B | B2C |
|--------|-----|-----|
| **Who** | Partners, vendors, external orgs | Consumers, customers |
| **Identity** | Their org's Azure AD | Social accounts, local accounts |
| **Tenant** | Guest in YOUR tenant | Separate B2C tenant |
| **Use case** | Collaboration | Consumer-facing apps |

### F.8 OAuth 2.0 and OpenID Connect — Protocol Fundamentals

**OAuth 2.0 (Authorization):**
- Grants **access** to resources on behalf of a user
- Issues **Access Tokens**
- Does NOT tell you WHO the user is
- Defines flows: Authorization Code, Client Credentials, Device Code, etc.

**OpenID Connect (Authentication):**
- Identity layer **built on top of OAuth 2.0**
- Issues **ID Tokens** (JWT containing user identity claims)
- Adds `/userinfo` endpoint and discovery document
- Tells you WHO the user is

**How They Work Together:**

```
1. App redirects to Azure AD login (OIDC + OAuth combined request)
2. User authenticates (OIDC — proves identity)
3. Azure AD returns:
   - ID Token (OIDC — who the user is)
   - Access Token (OAuth — what the user can access)
   - (Optional) Refresh Token (OAuth — get new tokens)
4. App uses ID Token for user info display
5. App sends Access Token to API (Authorization: Bearer <token>)
6. API validates token and checks permissions
```

**Bearer Tokens:**
```http
GET https://graph.microsoft.com/v1.0/me
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIs...
```
- "Bearer" means whoever **bears** (holds) the token gets access
- Token is sent in the `Authorization` HTTP header
- Must be protected (use HTTPS, short-lived tokens)

### F.9 Azure AD Administrative Roles vs Azure RBAC

**TWO SEPARATE ROLE SYSTEMS:**

| System | Scope | Purpose | Examples |
|--------|-------|---------|----------|
| **Azure AD Roles** | Azure AD tenant (identity) | Manage identity objects (users, groups, apps) | Global Admin, Application Admin, User Admin |
| **Azure RBAC** | Azure resources (infrastructure) | Manage Azure resources (VMs, storage, etc.) | Owner, Contributor, Reader, Storage Blob Data Reader |

```
Azure AD Roles → Manage IDENTITY (users, groups, app registrations)
Azure RBAC     → Manage RESOURCES (storage accounts, VMs, databases)
```

**Common Azure AD Administrative Roles:**

| Role | What They Can Do |
|------|-----------------|
| **Global Administrator** | Full access to EVERYTHING in Azure AD (most powerful) |
| **Application Administrator** | Manage all app registrations and enterprise apps |
| **Cloud Application Administrator** | Like Application Admin but cannot manage app proxy |
| **User Administrator** | Manage users and groups, reset passwords |
| **Privileged Role Administrator** | Manage role assignments in Azure AD |
| **Security Administrator** | Read security info, manage security policies |

**⚠️ Exam Tip:** Global Administrator does NOT automatically have Azure RBAC access to resources. They must **elevate access** or be explicitly assigned an Azure RBAC role. These are two separate systems.

### F.10 Azure RBAC — Role Definitions Deep Dive

**Role Definition JSON Structure:**

```json
{
  "Name": "Custom Blob Manager",
  "Description": "Can manage blobs but not containers",
  "Actions": [
    "Microsoft.Storage/storageAccounts/read"
  ],
  "NotActions": [],
  "DataActions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read",
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/write",
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete"
  ],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/<subscription-id>"
  ]
}
```

**Permission Types:**

| Property | Plane | Description |
|----------|-------|-------------|
| `Actions` | Control plane | Resource management operations (create, delete, configure) |
| `NotActions` | Control plane | Excluded management operations |
| `DataActions` | Data plane | Data operations (read/write data inside resources) |
| `NotDataActions` | Data plane | Excluded data operations |

```bash
# Create custom role
az role definition create --role-definition custom-role.json

# List custom roles
az role definition list --custom-role-only true

# List actions for a built-in role
az role definition list --name "Storage Blob Data Reader" --output json
```

**Built-in Role Examples — Actions vs DataActions:**

| Role | Actions (Control Plane) | DataActions (Data Plane) |
|------|------------------------|-------------------------|
| **Contributor** | `*/` (all management) | None |
| **Reader** | `*/read` | None |
| **Storage Blob Data Reader** | None | `blobs/read` |
| **Storage Blob Data Contributor** | None | `blobs/read, write, delete` |
| **Storage Blob Data Owner** | None | `blobs/*` + assign roles |

**⚠️ Key insight:** Contributor has `Actions: [*/*]` but `DataActions: []` — that's WHY Contributor cannot read Key Vault secrets or Storage blob data.

### F.11 Conditional Access — Policy Details

**Policy Structure:**

```
IF (Assignments match)     →    THEN (Apply Access Controls)
   ├── Users/Groups               ├── Grant (Allow + require MFA/device)
   ├── Cloud Apps                  ├── Block
   ├── Conditions                  └── Session (enforce controls)
   │   ├── Sign-in risk
   │   ├── Device platform
   │   ├── Location (IP/country)
   │   ├── Client app type
   │   └── Device state
```

**Common Policies:**

| Scenario | Assignment | Control |
|----------|-----------|---------|
| Require MFA for admins | Admin roles | Grant: Require MFA |
| Block legacy auth | All users, client apps = "Other clients" | Block |
| Require managed device for sensitive apps | All users, specific apps | Grant: Require compliant device |
| Block access from certain countries | All users, location = blocked countries | Block |

```bash
# Conditional Access policies are managed via Azure Portal or Microsoft Graph API
# Not available via Azure CLI
# Graph API endpoint:
# GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies
```

**⚠️ Exam Tip:** Conditional Access requires **Azure AD P1** license. If multiple policies apply, ALL conditions must be satisfied (policies are ANDed, not ORed).

### F.12 Supported Account Types (App Registration)

When registering an app in Azure AD, you choose **who can sign in**:

| Option | Value in Manifest | Who Can Sign In |
|--------|------------------|----------------|
| **Single tenant** | `AzureADMyOrg` | Users in YOUR Azure AD tenant only |
| **Multi-tenant** | `AzureADMultipleOrgs` | Users in ANY Azure AD tenant |
| **Multi-tenant + Personal** | `AzureADandPersonalMicrosoftAccount` | Any Azure AD tenant + personal Microsoft accounts (Outlook, Xbox) |
| **Personal only** | `PersonalMicrosoftAccount` | Personal Microsoft accounts only |

**Impact on Authority URL:**

```
Single tenant:   https://login.microsoftonline.com/<tenant-id>
Multi-tenant:    https://login.microsoftonline.com/common
Multi-tenant+PA: https://login.microsoftonline.com/common
Personal only:   https://login.microsoftonline.com/consumers
Azure AD only:   https://login.microsoftonline.com/organizations
```

**⚠️ Exam Tip:** If the question says "users from multiple organizations", use `organizations` or `common`. If it says "single organization only", use the tenant ID.

### F.13 Multi-Tenant Applications

**How Multi-Tenant Apps Work:**

```
Home Tenant (where app is registered)
    └── Application Object + Service Principal (home)

Tenant B (consents to use the app)
    └── Service Principal (copy) ← created on first user consent

Tenant C (consents to use the app)
    └── Service Principal (copy) ← created on first user consent
```

- App is registered ONCE in the home tenant
- When a user from another tenant signs in and consents → a **Service Principal** is created in THEIR tenant
- Each tenant's Service Principal has its own permissions and assignments
- The Application Object stays in the home tenant only

**Multi-Tenant Code Considerations:**
```csharp
// Multi-tenant app MSAL configuration
var app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, "common") // "common" for multi-tenant
    .WithClientSecret(clientSecret)
    .Build();
```

**Token Validation for Multi-Tenant:**
- Do NOT validate `iss` (issuer) against a single tenant — it will vary
- Validate `iss` format: `https://login.microsoftonline.com/{tenantId}/v2.0`
- Check `tid` (tenant ID) against your list of allowed tenants
- Use `aud` (audience) to verify token is for your app

### F.14 SAML vs OAuth 2.0 vs OpenID Connect

| Protocol | Purpose | Token Type | Use Case |
|----------|---------|-----------|----------|
| **SAML 2.0** | Authentication + Authorization | XML assertion | Enterprise SSO (older), ADFS federation |
| **OAuth 2.0** | Authorization only | Access Token (JWT) | API access, delegated permissions |
| **OpenID Connect** | Authentication (on OAuth 2.0) | ID Token + Access Token (JWT) | Modern web/mobile sign-in |

```
SAML = XML-based, older, widely used in enterprise SSO
OAuth 2.0 = JSON/JWT-based, modern, for API authorization
OIDC = OAuth 2.0 + identity layer (ID token), for modern sign-in
```

**When to Use What:**
- **Enterprise SSO with legacy apps** → SAML
- **Modern web/mobile app sign-in** → OpenID Connect
- **API-to-API access** → OAuth 2.0 (Client Credentials)
- **User accessing API via app** → OpenID Connect + OAuth 2.0

**⚠️ Exam Tip:** AZ-204 focuses heavily on OAuth 2.0 and OIDC. SAML awareness is enough — you won't write SAML code, but you should know it's XML-based and used for enterprise SSO.

### F.15 Azure AD Hybrid Identity

**Azure AD Connect** — Synchronizes on-premises Active Directory with Azure AD:

```
On-Premises AD DS ──── Azure AD Connect ──── Azure AD (Cloud)
   (users, groups)       (sync tool)          (cloud identities)
```

**Authentication Methods:**

| Method | Description |
|--------|-------------|
| **Password Hash Synchronization (PHS)** | Hash of on-prem password synced to Azure AD; Azure AD performs authentication |
| **Pass-Through Authentication (PTA)** | Azure AD sends auth request to on-prem agent; password never stored in cloud |
| **Federation (ADFS)** | Azure AD redirects to on-prem ADFS server for authentication |

**⚠️ Exam Tip:** PHS is the simplest and recommended default. PTA keeps passwords strictly on-premises. Federation uses ADFS. For AZ-204, just know these exist — deep details are AZ-500/SC-300.

### F.16 Azure AD Application Proxy

- Publishes on-premises web applications for external access
- No need for VPN or inbound firewall rules
- Users authenticate via Azure AD, then traffic is relayed to on-prem app via a lightweight connector

```
External User → Azure AD (authenticate) → Application Proxy (cloud) → Connector (on-prem) → On-prem Web App
```

**⚠️ Exam Tip:** Application Proxy requires Azure AD **P1** license. The connector runs on-premises. Mainly an AZ-500 topic, but may appear in AZ-204 as a distractor.

### F.17 Azure AD Deny Assignments (RBAC)

- **Deny assignments** block users from performing actions even if a role assignment grants access
- **Deny takes precedence** over allow (role assignments)
- Cannot be created directly by users — created by Azure Blueprints or managed apps
- Used to protect critical resources

```
Evaluation Order:
1. Check Deny Assignments → If DENY found → ACCESS DENIED (stops here)
2. Check Role Assignments → If matching ALLOW → ACCESS GRANTED
3. No matching assignment → ACCESS DENIED (implicit deny)
```

**⚠️ Exam Tip:** Azure uses **implicit deny** by default. If no role assignment matches, access is denied. Explicit deny assignments override explicit allow.

### F.18 Custom RBAC Roles

**When to Use:**
- Built-in roles don't exactly match your needs
- Need to combine permissions from multiple built-in roles
- Need to restrict to very specific actions

**Creating Custom Roles:**

```json
{
  "Name": "App Service Plan Manager",
  "IsCustom": true,
  "Description": "Can manage App Service plans but not web apps",
  "Actions": [
    "Microsoft.Web/serverfarms/*",
    "Microsoft.Resources/subscriptions/resourceGroups/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/<subscription-id>"
  ]
}
```

```bash
# Create custom role from JSON
az role definition create --role-definition custom-role.json

# Update custom role
az role definition update --role-definition updated-role.json

# Delete custom role
az role definition delete --name "App Service Plan Manager"
```

**Custom Role Limits:**
- Up to **5000** custom roles per Azure AD tenant
- `AssignableScopes` defines WHERE the role can be assigned
- Can assign at management group, subscription, or resource group scope
- Cannot use wildcard (`*`) in DataActions for custom roles

**⚠️ Exam Tip:** Custom roles require Azure AD **P1** or **P2** for Azure AD custom roles. Azure RBAC custom roles require the ability to create them (Owner or User Access Administrator). NotActions is NOT deny — it's subtraction from Actions.

### F.19 Custom Azure AD Directory Roles

- Different from Azure RBAC custom roles
- Manage Azure AD objects (users, groups, app registrations)
- Requires Azure AD **P1** license

```json
{
  "displayName": "Application Registration Creator",
  "description": "Can create app registrations",
  "rolePermissions": [
    {
      "allowedResourceActions": [
        "microsoft.directory/applications/create"
      ]
    }
  ],
  "isEnabled": true
}
```

**Common Azure AD Permission Actions:**
- `microsoft.directory/users/create` — Create users
- `microsoft.directory/groups/create` — Create groups
- `microsoft.directory/applications/create` — Create app registrations
- `microsoft.directory/servicePrincipals/create` — Create service principals

### F.20 Azure AD Privileged Identity Management (PIM)

- Provides **just-in-time** (JIT) privileged access
- Users don't permanently hold admin roles
- Must **activate** the role when needed (with justification and optional approval)
- Requires Azure AD **P2** license

```
Without PIM: User → permanently assigned Global Admin
With PIM:    User → eligible for Global Admin → must activate → time-limited (e.g., 8 hours)
```

**Key Features:**
- Time-bound access (activate role for X hours)
- Approval workflow
- MFA requirement on activation
- Audit trail of all activations
- Access reviews

**⚠️ Exam Tip:** PIM is mainly an AZ-500/SC-300 topic, but know the concept: just-in-time access reduces standing privileges. Requires P2.

### F.21 Azure AD Identity Protection

- Uses machine learning to detect risky sign-ins and compromised users
- Requires Azure AD **P2** license

**Risk Types:**

| Risk Type | Examples |
|-----------|---------|
| **Sign-in Risk** | Sign-in from unfamiliar location, anonymous IP, malware-linked IP, impossible travel |
| **User Risk** | Leaked credentials found on dark web, anomalous user activity |

**Risk-Based Policies:**
- Require MFA for medium/high sign-in risk
- Require password change for high user risk
- Block access for very high risk

### F.22 Token Endpoints and Key URLs

**Azure AD Endpoints (v2.0):**

| Endpoint | URL |
|----------|-----|
| **Authorization** | `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize` |
| **Token** | `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token` |
| **Discovery** | `https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration` |
| **JWKS (Signing Keys)** | `https://login.microsoftonline.com/{tenant}/discovery/v2.0/keys` |
| **UserInfo** | `https://graph.microsoft.com/oidc/userinfo` |

**v1.0 vs v2.0 Endpoints:**

| Aspect | v1.0 | v2.0 |
|--------|------|------|
| Path | `/oauth2/authorize` | `/oauth2/v2.0/authorize` |
| Token format | Access token may not be JWT | Access token is always JWT |
| Scopes | Resource-based (`resource=`) | Scope-based (`scope=`) |
| Personal accounts | Not supported | Supported |
| MSAL support | ADAL (deprecated) | MSAL |

**⚠️ Exam Tip:** Always use **v2.0** endpoints with MSAL. ADAL (v1.0) is deprecated. If you see `resource=` parameter, it's v1.0. If you see `scope=`, it's v2.0.

### F.23 Azure AD Application Registration — Key Settings

**Essential Settings When Registering an App:**

| Setting | Purpose | Example |
|---------|---------|---------|
| **Application (client) ID** | Unique identifier for the app | `12345678-abcd-...` |
| **Directory (tenant) ID** | Azure AD tenant identifier | `87654321-dcba-...` |
| **Redirect URI** | Where tokens are sent after authentication | `https://myapp.com/callback` |
| **Client Secret** | Password-like credential (string) | Expires in 6/12/24 months or custom |
| **Certificate** | Stronger credential (X.509) | Thumbprint stored, private key held by app |
| **API Permissions** | What APIs the app can access | `Microsoft Graph: User.Read` |
| **Expose an API** | Define scopes that YOUR API offers | `api://<client-id>/access_as_user` |
| **App Roles** | Custom roles for authorization | `Admin`, `Reader`, `Writer` |

**Client Secret vs Certificate:**

| Aspect | Client Secret | Certificate |
|--------|--------------|-------------|
| Security | Lower (string can be leaked) | Higher (private key stays with app) |
| Rotation | Manual or auto-expire | Certificate renewal |
| Max expiry | 2 years (recommended: shorter) | Depends on certificate |
| Production | Acceptable | Recommended |
| Format | Plain string | X.509 certificate (.pfx/.pem) |

**Redirect URI Types:**

| Platform | Type | Example |
|----------|------|---------|
| Web | HTTPS URL | `https://myapp.com/auth/callback` |
| SPA | HTTPS URL | `https://myapp.com` (uses PKCE) |
| Mobile/Desktop | Custom scheme or localhost | `msauth://callback`, `http://localhost` |

```bash
# Register app via CLI
az ad app create --display-name "My API" \
  --sign-in-audience "AzureADMyOrg" \
  --web-redirect-uris "https://myapp.com/callback"

# Create client secret
az ad app credential reset --id <app-id> --years 1

# Upload certificate
az ad app credential reset --id <app-id> --cert @mycert.pem
```

### F.24 Azure AD vs Azure AD B2C vs Azure AD B2B — Complete Comparison

| Feature | Azure AD | Azure AD B2B | Azure AD B2C |
|---------|---------|-------------|-------------|
| **Purpose** | Workforce identity | Partner collaboration | Consumer identity |
| **Users** | Employees | External guests | Customers/consumers |
| **Identity providers** | Azure AD | Guest's own IdP | Social (Google, Facebook), local accounts |
| **Tenant** | Organization tenant | Same org tenant (as guests) | Separate B2C tenant |
| **Customization** | Company branding | Limited | Full UI customization (user flows, policies) |
| **Sign-up** | Admin creates accounts | Admin invites | Self-service registration |
| **MFA** | Azure AD MFA | Home tenant MFA | Built-in MFA |
| **Pricing** | Per-user license | Free (first 50K MAU) | Per authentication |

### F.25 Security Defaults

**What Are Security Defaults?**
- Pre-configured security settings for Azure AD tenants
- **Free** — available to all Azure AD editions
- Cannot be used alongside Conditional Access policies

**What Security Defaults Enforce:**
1. All users must register for MFA within 14 days
2. Admins must perform MFA on every sign-in
3. Users must perform MFA when necessary (risky sign-in)
4. Block legacy authentication protocols
5. Protect privileged actions (Azure portal, Azure PowerShell, CLI)

```
Security Defaults = simple, free, all-or-nothing
Conditional Access = flexible, granular, requires P1
```

**⚠️ Exam Tip:** Security Defaults and Conditional Access are **mutually exclusive**. If you enable Conditional Access, Security Defaults must be disabled. Security Defaults are great for small orgs that need baseline security without P1.

---

## MODULE 1: EXPLORE THE MICROSOFT IDENTITY PLATFORM

### 1.1 Introduction to Microsoft Identity Platform

**What is Microsoft Identity Platform?**
- Cloud identity service **for developers** (built on top of Azure AD)
- Allows sign-in with Microsoft identities or social accounts
- Provides authorized access to APIs (Microsoft Graph, custom APIs)
- Evolution of Azure Active Directory (now Microsoft Entra ID)
- Based on industry standards (OAuth 2.0, OpenID Connect)

**Key Components:**
1. **Authentication Service**: Validates identity, issues tokens
2. **Open-source Libraries**: MSAL for various platforms
3. **Application Management Portal**: Azure portal, App registrations
4. **Configuration APIs**: Microsoft Graph API, PowerShell

**What You Can Build:**
- Web applications with user sign-in
- Single-page applications (SPAs)
- Web APIs protected by identity
- Mobile and desktop applications
- Daemon/service applications
- Server-side scripts

### 1.2 Service Principals and Application Objects

**Application Object:**
- Global representation of your application
- Defined in the "home" tenant where app is registered
- Template for creating service principals
- One-to-one relationship with software application
- One-to-many relationship with service principals

**Properties defined in Application Object:**
- Application (client) ID
- Redirect URIs
- Credentials (certificates, secrets)
- Permissions (API permissions)
- Application roles

**Service Principal:**
- Local representation in a specific tenant
- Created when app is used in a tenant
- Defines what app can do in that tenant
- Used for authentication and authorization

**Types of Service Principals:**

| Type | Description |
|------|-------------|
| **Application** | Local instance of global app object |
| **Managed Identity** | Represents managed identity resource |
| **Legacy** | Created before app registrations existed |

**Relationship:**

```
Application Object (Home Tenant)
    │
    ├── Service Principal (Tenant A)
    │       └── Permissions, Roles, Policies
    │
    ├── Service Principal (Tenant B)
    │       └── Permissions, Roles, Policies
    │
    └── Service Principal (Tenant C)
            └── Permissions, Roles, Policies
```

### 1.3 Permissions and Consent

**Permission Types:**

#### **1. Delegated Permissions**
- App acts on behalf of signed-in user
- User (or admin) consents at runtime
- App can only do what user can do
- Used by apps with signed-in users

```
User Signs In → Consents → App Gets Token → App Calls API (as User)
```

#### **2. Application Permissions**
- App acts as itself (no signed-in user)
- Always requires admin consent
- Used by daemon services, background jobs
- Full access without user context

```
Admin Consents → App Gets Token → App Calls API (as Application)
```

**Permission Comparison:**

| Aspect | Delegated | Application |
|--------|-----------|-------------|
| User Required | Yes | No |
| Consent | User or Admin | Admin only |
| Access Level | User's permissions | App's permissions |
| Use Case | User-facing apps | Background services |

**Consent Types:**

#### **1. Static User Consent**
- All permissions defined at registration
- User sees all permissions at first sign-in
- Admin can consent on behalf of organization

#### **2. Incremental/Dynamic Consent**
- Request permissions as needed
- Better user experience
- Users see only what's needed

```csharp
// Request additional permissions dynamically
var scopes = new[] { "User.Read", "Mail.Read" };
var result = await app.AcquireTokenInteractive(scopes).ExecuteAsync();
```

#### **3. Admin Consent**
- Required for high-privilege permissions
- Admin consents for entire organization
- Users don't see consent prompt

**Consent Levels:**

| Level | Description |
|-------|-------------|
| User Consent | Individual user consents for themselves |
| Admin Consent (User) | Admin consents for single user |
| Admin Consent (Org) | Admin consents for all users in organization |

### 1.4 Conditional Access

**What is Conditional Access?**
- Zero Trust access control
- "If-then" policies for access decisions
- Evaluates signals to make decisions

**Common Signals:**
- User or group membership
- IP location
- Device (platform, compliance state)
- Application being accessed
- Risk detection (sign-in, user risk)

**Common Decisions:**
- Allow access
- Block access
- Require MFA
- Require compliant device
- Require managed device
- Require password change

**Impact on Applications:**

Apps may need to handle Conditional Access challenges:

```csharp
try
{
    var result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync();
}
catch (MsalUiRequiredException ex) when (ex.Classification == UiRequiredExceptionClassification.ConditionalAccessRequired)
{
    // Conditional Access policy requires additional action
    var result = await app.AcquireTokenInteractive(scopes)
        .WithClaims(ex.Claims)
        .ExecuteAsync();
}
```

### 1.5 Application Registration

**Registering an Application:**

```
Azure Portal → Microsoft Entra ID → App registrations → New registration
```

**Registration Settings:**

| Setting | Description |
|---------|-------------|
| Name | Display name for the app |
| Supported Account Types | Single tenant, multi-tenant, personal accounts |
| Redirect URI | Where tokens are sent after authentication |
| Application ID (Client ID) | Unique identifier for the app |
| Directory ID (Tenant ID) | Identifier for the Azure AD tenant |

**Supported Account Types:**

| Type | Description | Use Case |
|------|-------------|----------|
| Single tenant | This org only | Internal LOB apps |
| Multitenant | Any org | SaaS apps |
| Multitenant + Personal | Any org + personal MS accounts | Consumer + enterprise apps |
| Personal only | Personal MS accounts only | Consumer apps |

**Redirect URIs:**

| Platform | Example URI |
|----------|-------------|
| Web | `https://myapp.com/callback` |
| SPA | `https://myapp.com/` |
| Mobile/Desktop | `myapp://callback` or `http://localhost` |
| Public client | `https://login.microsoftonline.com/common/oauth2/nativeclient` |

**Credentials:**

**Client Secrets:**
- String value (password)
- Easier to create, less secure
- Has expiration date
- Stored securely, never in code

**Certificates:**
- More secure than secrets
- Public key registered with app
- Private key used by app to prove identity
- Recommended for production

```bash
# Register app with Azure CLI
az ad app create \
  --display-name "My App" \
  --sign-in-audience AzureADMyOrg \
  --web-redirect-uris "https://myapp.com/callback"

# Create client secret
az ad app credential reset \
  --id <app-id> \
  --append

# Upload certificate
az ad app credential reset \
  --id <app-id> \
  --cert @certificate.pem
```

### 1.6 Authentication Flows

**OAuth 2.0 / OpenID Connect Flows:**

#### **1. Authorization Code Flow**
- Most secure for web apps
- Backend exchanges code for tokens
- Tokens never exposed to browser

```
User → App → Identity Provider
            ↓
      Authorization Code
            ↓
App Backend → Exchange for Tokens
```

**Use Cases:** Web applications, server-side apps

#### **2. Authorization Code Flow with PKCE**
- For public clients (mobile, SPA)
- Adds Proof Key for Code Exchange
- Prevents authorization code interception

```
Client generates:
  - code_verifier (random string)
  - code_challenge = SHA256(code_verifier)
  
Send code_challenge with auth request
Send code_verifier with token request
```

**Use Cases:** Mobile apps, SPAs, desktop apps

#### **3. Implicit Flow (Legacy)**
- Tokens returned directly in redirect
- No longer recommended (security issues)
- Use Authorization Code with PKCE instead

#### **4. Client Credentials Flow**
- For daemon/service applications
- No user interaction
- App authenticates with its own credentials

```
App → Identity Provider (with client credentials)
            ↓
      Access Token (app permissions only)
```

**Use Cases:** Background services, microservices, APIs

#### **5. Device Code Flow**
- For devices without browsers/keyboards
- User authenticates on different device

```
Device → Identity Provider → Device Code + User Code
User → Browser → Enter User Code → Authenticate
Device polls for tokens
```

**Use Cases:** IoT devices, CLI tools, TV apps

#### **6. On-Behalf-Of (OBO) Flow**
- API calls another API on user's behalf
- Preserves user context through chain

```
User → App → API 1 (with user token)
              ↓
          API 1 → Identity Provider (exchange token)
              ↓
          API 1 → API 2 (with new token)
```

**Use Cases:** Microservices, API chains

**Flow Selection Guide:**

| Scenario | Recommended Flow |
|----------|-----------------|
| Web app with backend | Authorization Code |
| Single-page application | Auth Code with PKCE |
| Mobile/Desktop app | Auth Code with PKCE |
| Daemon/Background service | Client Credentials |
| IoT/CLI tool | Device Code |
| API calling API | On-Behalf-Of |

---

## MODULE 2: MICROSOFT AUTHENTICATION LIBRARY (MSAL)

### 2.1 Introduction to MSAL

**What is MSAL?**
- Microsoft Authentication Library
- Handles authentication with Microsoft identity platform
- Manages tokens (acquire, cache, refresh)
- Supports multiple platforms

**MSAL Benefits:**
- Handles OAuth 2.0 / OpenID Connect complexity
- Automatic token caching
- Automatic token refresh
- Silent authentication support
- Conditional Access handling

**MSAL Libraries:**

| Platform | Library | Package |
|----------|---------|---------|
| .NET | MSAL.NET | Microsoft.Identity.Client |
| JavaScript | MSAL.js | @azure/msal-browser |
| Python | MSAL Python | msal |
| Java | MSAL Java | msal4j |
| Android | MSAL Android | com.microsoft.identity.client |
| iOS | MSAL iOS | MSAL |
| Node.js | MSAL Node | @azure/msal-node |

### 2.2 Public vs Confidential Client Applications

**Public Client Applications:**
- Cannot securely store secrets
- Run on user's device
- Examples: Desktop apps, mobile apps, SPAs
- Use Authorization Code with PKCE

```csharp
// Public client application
IPublicClientApplication app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(authority)
    .WithRedirectUri(redirectUri)
    .Build();
```

**Confidential Client Applications:**
- Can securely store secrets
- Run on secure servers
- Examples: Web apps, Web APIs, daemon services
- Use client secrets or certificates

```csharp
// Confidential client with secret
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)
    .WithAuthority(authority)
    .Build();

// Confidential client with certificate
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithCertificate(certificate)
    .WithAuthority(authority)
    .Build();
```

**Comparison:**

| Aspect | Public Client | Confidential Client |
|--------|--------------|---------------------|
| Secret Storage | Cannot store | Can store securely |
| Environment | User device | Server |
| Examples | Mobile, Desktop, SPA | Web app, API, Daemon |
| Credentials | PKCE | Secret or Certificate |

### 2.3 MSAL.NET Application Builders

**PublicClientApplicationBuilder:**

```csharp
IPublicClientApplication app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .WithRedirectUri("http://localhost")
    .Build();

// With specific options
IPublicClientApplication app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(authority)
    .WithDefaultRedirectUri()
    .WithLogging(MyLoggingMethod, LogLevel.Info, enablePiiLogging: false)
    .Build();
```

**ConfidentialClientApplicationBuilder:**

```csharp
// With client secret
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(authority)
    .WithClientSecret(clientSecret)
    .Build();

// With certificate
X509Certificate2 cert = new X509Certificate2("certificate.pfx", "password");
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(authority)
    .WithCertificate(cert)
    .Build();

// From configuration
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder
    .CreateWithApplicationOptions(options)
    .Build();
```

**Common Builder Methods:**

| Method | Description |
|--------|-------------|
| `.Create(clientId)` | Initialize with client ID |
| `.WithAuthority()` | Set authority URL |
| `.WithTenantId()` | Set specific tenant |
| `.WithRedirectUri()` | Set redirect URI |
| `.WithClientSecret()` | Set client secret |
| `.WithCertificate()` | Set certificate |
| `.WithLogging()` | Enable logging |
| `.Build()` | Create the application |

### 2.4 Acquiring Tokens

**Token Acquisition Methods:**

#### **Interactive (User Sign-in):**

```csharp
// Public client - interactive
string[] scopes = { "User.Read", "Mail.Read" };

AuthenticationResult result = await app
    .AcquireTokenInteractive(scopes)
    .WithPrompt(Prompt.SelectAccount)
    .ExecuteAsync();

string accessToken = result.AccessToken;
```

#### **Silent (From Cache):**

```csharp
// Try silent first, fall back to interactive
IAccount account = (await app.GetAccountsAsync()).FirstOrDefault();

AuthenticationResult result;
try
{
    result = await app
        .AcquireTokenSilent(scopes, account)
        .ExecuteAsync();
}
catch (MsalUiRequiredException)
{
    result = await app
        .AcquireTokenInteractive(scopes)
        .ExecuteAsync();
}
```

#### **Client Credentials (Daemon):**

```csharp
// Confidential client - no user
string[] scopes = { "https://graph.microsoft.com/.default" };

AuthenticationResult result = await app
    .AcquireTokenForClient(scopes)
    .ExecuteAsync();
```

#### **On-Behalf-Of (API to API):**

```csharp
// Exchange user token for new token
string[] scopes = { "api://downstream-api/.default" };

AuthenticationResult result = await app
    .AcquireTokenOnBehalfOf(scopes, userAssertion)
    .ExecuteAsync();
```

#### **Device Code (Input-constrained devices):**

```csharp
AuthenticationResult result = await app
    .AcquireTokenWithDeviceCode(scopes, async deviceCodeResult =>
    {
        Console.WriteLine(deviceCodeResult.Message);
        // "To sign in, use a web browser to open the page 
        //  https://microsoft.com/devicelogin and enter the code ABC123"
        await Task.FromResult(0);
    })
    .ExecuteAsync();
```

### 2.5 Token Cache

**Why Token Caching?**
- Avoid unnecessary authentication prompts
- Reduce calls to identity provider
- Improve application performance
- Handle token refresh automatically

**Default Caching:**
- Public clients: In-memory cache (lost on restart)
- Confidential clients: In-memory cache

**Custom Token Cache (For Web Apps):**

```csharp
// In-memory cache (for development)
app.AddInMemoryTokenCache();

// Distributed cache (for production)
app.AddDistributedTokenCache(services =>
{
    services.AddStackExchangeRedisCache(options =>
    {
        options.Configuration = "redis-connection-string";
    });
});

// SQL Server cache
app.AddDistributedTokenCache(services =>
{
    services.AddDistributedSqlServerCache(options =>
    {
        options.ConnectionString = "sql-connection-string";
        options.SchemaName = "dbo";
        options.TableName = "TokenCache";
    });
});
```

**Serialization for Desktop/Mobile:**

```csharp
// Enable token cache serialization
var storageProperties = new StorageCreationPropertiesBuilder(
    "myapp.cache",
    MsalCacheHelper.UserRootDirectory)
    .Build();

var cacheHelper = await MsalCacheHelper.CreateAsync(storageProperties);
cacheHelper.RegisterCache(app.UserTokenCache);
```

### 2.6 Handling MSAL Exceptions

**Exception Types:**

| Exception | Description |
|-----------|-------------|
| MsalClientException | SDK errors, configuration issues |
| MsalServiceException | Identity provider errors |
| MsalUiRequiredException | User interaction required |

**Error Handling Pattern:**

```csharp
try
{
    var result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync();
    return result.AccessToken;
}
catch (MsalUiRequiredException ex)
{
    // User interaction required - show login UI
    try
    {
        var result = await app.AcquireTokenInteractive(scopes).ExecuteAsync();
        return result.AccessToken;
    }
    catch (MsalException msalEx)
    {
        // Handle MSAL-specific error
        Console.WriteLine($"MSAL error: {msalEx.ErrorCode}");
        throw;
    }
}
catch (MsalServiceException ex) when (ex.ErrorCode == "invalid_grant")
{
    // Token expired or revoked
    // Clear cache and re-authenticate
    var accounts = await app.GetAccountsAsync();
    foreach (var acc in accounts)
    {
        await app.RemoveAsync(acc);
    }
    throw;
}
catch (MsalClientException ex)
{
    // Configuration or network error
    Console.WriteLine($"Client error: {ex.Message}");
    throw;
}
```

**Common Error Codes:**

| Error Code | Description | Action |
|------------|-------------|--------|
| invalid_grant | Token expired/revoked | Re-authenticate |
| interaction_required | MFA or consent needed | Interactive auth |
| consent_required | Permissions not granted | Request consent |
| invalid_client | Wrong credentials | Check app config |
| temporarily_unavailable | Service issue | Retry with backoff |

### 2.7 Complete MSAL Examples

**Example 1: Desktop Application**

```csharp
public class AuthService
{
    private readonly IPublicClientApplication _app;
    private readonly string[] _scopes = { "User.Read", "Mail.Read" };

    public AuthService(string clientId)
    {
        _app = PublicClientApplicationBuilder
            .Create(clientId)
            .WithAuthority(AzureCloudInstance.AzurePublic, "common")
            .WithDefaultRedirectUri()
            .Build();

        // Enable token cache persistence
        TokenCacheHelper.EnableSerialization(_app.UserTokenCache);
    }

    public async Task<string> GetTokenAsync()
    {
        var accounts = await _app.GetAccountsAsync();
        var firstAccount = accounts.FirstOrDefault();

        try
        {
            // Try silent authentication first
            var result = await _app
                .AcquireTokenSilent(_scopes, firstAccount)
                .ExecuteAsync();
            return result.AccessToken;
        }
        catch (MsalUiRequiredException)
        {
            // Fall back to interactive
            var result = await _app
                .AcquireTokenInteractive(_scopes)
                .WithAccount(firstAccount)
                .WithPrompt(Prompt.SelectAccount)
                .ExecuteAsync();
            return result.AccessToken;
        }
    }

    public async Task SignOutAsync()
    {
        var accounts = await _app.GetAccountsAsync();
        foreach (var account in accounts)
        {
            await _app.RemoveAsync(account);
        }
    }
}
```

**Example 2: Web API (Daemon Service)**

```csharp
public class GraphService
{
    private readonly IConfidentialClientApplication _app;
    private readonly string[] _scopes = { "https://graph.microsoft.com/.default" };

    public GraphService(IConfiguration configuration)
    {
        _app = ConfidentialClientApplicationBuilder
            .Create(configuration["AzureAd:ClientId"])
            .WithClientSecret(configuration["AzureAd:ClientSecret"])
            .WithAuthority(new Uri($"https://login.microsoftonline.com/{configuration["AzureAd:TenantId"]}"))
            .Build();
    }

    public async Task<string> GetTokenAsync()
    {
        var result = await _app
            .AcquireTokenForClient(_scopes)
            .ExecuteAsync();
        return result.AccessToken;
    }

    public async Task<HttpResponseMessage> CallGraphApiAsync(string endpoint)
    {
        var token = await GetTokenAsync();
        
        using var client = new HttpClient();
        client.DefaultRequestHeaders.Authorization = 
            new AuthenticationHeaderValue("Bearer", token);
        
        return await client.GetAsync($"https://graph.microsoft.com/v1.0/{endpoint}");
    }
}
```

**Example 3: ASP.NET Core Web App**

```csharp
// Program.cs / Startup.cs
builder.Services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(builder.Configuration.GetSection("AzureAd"))
    .EnableTokenAcquisitionToCallDownstreamApi()
    .AddMicrosoftGraph(builder.Configuration.GetSection("Graph"))
    .AddInMemoryTokenCaches();

builder.Services.AddAuthorization();

// Controller
[Authorize]
public class HomeController : Controller
{
    private readonly GraphServiceClient _graphClient;

    public HomeController(GraphServiceClient graphClient)
    {
        _graphClient = graphClient;
    }

    public async Task<IActionResult> Profile()
    {
        var user = await _graphClient.Me.GetAsync();
        return View(user);
    }
}

// appsettings.json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "contoso.onmicrosoft.com",
    "TenantId": "your-tenant-id",
    "ClientId": "your-client-id",
    "ClientSecret": "your-client-secret",
    "CallbackPath": "/signin-oidc"
  },
  "Graph": {
    "BaseUrl": "https://graph.microsoft.com/v1.0",
    "Scopes": "User.Read"
  }
}
```

---

## MODULE 3: IMPLEMENT SHARED ACCESS SIGNATURES

### 3.1 Introduction to Shared Access Signatures

**What is a Shared Access Signature (SAS)?**
- Signed URI providing delegated access to Azure Storage
- Grants limited permissions for limited time
- No need to share account keys
- Fine-grained access control

**SAS Use Cases:**
- Grant temporary access to storage resources
- Allow clients to upload files directly
- Share specific blobs/containers without full access
- Delegate access without exposing credentials

### 3.2 Types of Shared Access Signatures

**1. User Delegation SAS (Recommended)**
- Secured with Azure AD credentials
- Most secure option
- Blob storage only
- Created using user delegation key

```csharp
// Get user delegation key
UserDelegationKey key = await blobServiceClient.GetUserDelegationKeyAsync(
    DateTimeOffset.UtcNow,
    DateTimeOffset.UtcNow.AddHours(1));

// Create SAS
BlobSasBuilder sasBuilder = new BlobSasBuilder
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

**2. Service SAS**
- Secured with storage account key
- Delegates access to single service
- Scoped to Blob, Queue, Table, or File service

```csharp
BlobSasBuilder sasBuilder = new BlobSasBuilder
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

**3. Account SAS**
- Secured with storage account key
- Delegates access to multiple services
- Can specify service-level operations

```bash
# Create account SAS with Azure CLI
az storage account generate-sas \
  --account-name myaccount \
  --account-key $key \
  --services bfqt \
  --resource-types sco \
  --permissions rwdlacup \
  --expiry 2024-12-31
```

**SAS Type Comparison:**

| Type | Security | Scope | Best For |
|------|----------|-------|----------|
| User Delegation | Azure AD | Blob only | Production, highest security |
| Service | Account Key | Single service | Service-specific access |
| Account | Account Key | Multiple services | Broad access patterns |

### 3.3 SAS Components and Parameters

**SAS URI Format:**
```
https://myaccount.blob.core.windows.net/container/blob?
  sv=2021-06-08&
  ss=b&
  srt=o&
  sp=r&
  se=2024-01-01T00:00:00Z&
  st=2023-01-01T00:00:00Z&
  spr=https&
  sig=xxxxx
```

**SAS Parameters:**

| Parameter | Name | Description |
|-----------|------|-------------|
| sv | Signed Version | Storage service version |
| ss | Signed Services | b=blob, f=file, q=queue, t=table |
| srt | Signed Resource Types | s=service, c=container, o=object |
| sp | Signed Permissions | r=read, w=write, d=delete, l=list, etc. |
| st | Start Time | When SAS becomes valid (UTC) |
| se | Expiry Time | When SAS expires (UTC) |
| sip | Signed IP | Allowed IP address or range |
| spr | Signed Protocol | https or https,http |
| sig | Signature | HMAC-SHA256 signature |

**Permissions by Service:**

| Permission | Blob | Container | File | Queue | Table |
|------------|------|-----------|------|-------|-------|
| Read (r) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Write (w) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Delete (d) | ✅ | ✅ | ✅ | ✅ | ✅ |
| List (l) | - | ✅ | ✅ | - | - |
| Add (a) | ✅ | - | - | ✅ | ✅ |
| Create (c) | ✅ | ✅ | ✅ | - | - |
| Update (u) | - | - | - | ✅ | ✅ |
| Process (p) | - | - | - | ✅ | - |

### 3.4 Stored Access Policies

**What are Stored Access Policies?**
- Named policy on a container, queue, table, or file share
- Groups SAS parameters in reusable definition
- Can revoke SAS by modifying/deleting policy
- Up to 5 policies per resource

**Benefits:**
- Revoke SAS without regenerating keys
- Modify expiry or permissions
- Better management of multiple SAS tokens

**Creating Stored Access Policy:**

```bash
# Create stored access policy
az storage container policy create \
  --container-name mycontainer \
  --account-name myaccount \
  --name mypolicy \
  --permissions rl \
  --expiry 2024-12-31T23:59:59Z

# List policies
az storage container policy list \
  --container-name mycontainer \
  --account-name myaccount

# Update policy
az storage container policy update \
  --container-name mycontainer \
  --account-name myaccount \
  --name mypolicy \
  --permissions r \
  --expiry 2025-12-31T23:59:59Z

# Delete policy (revokes all SAS using it)
az storage container policy delete \
  --container-name mycontainer \
  --account-name myaccount \
  --name mypolicy
```

**Using Stored Access Policy with SAS:**

```csharp
// Reference stored access policy
BlobSasBuilder sasBuilder = new BlobSasBuilder
{
    BlobContainerName = containerName,
    Identifier = "mypolicy"  // Reference policy by name
};

string sasToken = sasBuilder.ToSasQueryParameters(
    new StorageSharedKeyCredential(accountName, accountKey)).ToString();
```

### 3.5 SAS Best Practices

**Security Best Practices:**

1. **Use User Delegation SAS when possible**
   - Most secure option
   - No storage keys required

2. **Use HTTPS only**
   - Prevent token interception
   - Set `spr=https`

3. **Shortest practical expiry time**
   - Minimize exposure window
   - Generate new SAS as needed

4. **Use stored access policies**
   - Enable revocation
   - Centralized management

5. **Principle of least privilege**
   - Grant only needed permissions
   - Scope to specific resources

6. **Use near-term expiration**
   - SAS without stored policy can't be revoked
   - Short expiry limits damage if compromised

7. **Validate SAS parameters**
   - Check client-generated SAS before using
   - Ensure permissions match requirements

**Generation Best Practices:**

```csharp
public string GenerateSafeSas(string containerName, string blobName)
{
    // Use user delegation SAS
    var userDelegationKey = await _blobServiceClient.GetUserDelegationKeyAsync(
        DateTimeOffset.UtcNow,
        DateTimeOffset.UtcNow.AddMinutes(15));  // Short key lifetime

    var sasBuilder = new BlobSasBuilder
    {
        BlobContainerName = containerName,
        BlobName = blobName,
        Resource = "b",
        StartsOn = DateTimeOffset.UtcNow,
        ExpiresOn = DateTimeOffset.UtcNow.AddMinutes(10),  // Short expiry
        Protocol = SasProtocol.Https  // HTTPS only
    };
    
    // Minimum permissions
    sasBuilder.SetPermissions(BlobSasPermissions.Read);
    
    // Optional: Restrict IP
    sasBuilder.IPRange = new SasIPRange(IPAddress.Parse("203.0.113.0"), IPAddress.Parse("203.0.113.255"));

    return sasBuilder.ToSasQueryParameters(userDelegationKey, _accountName).ToString();
}
```

### 3.6 SAS Code Examples

**Generate Blob SAS URI:**

```csharp
public Uri GetBlobSasUri(BlobClient blobClient, TimeSpan validity, BlobSasPermissions permissions)
{
    // Check if BlobClient can generate SAS
    if (!blobClient.CanGenerateSasUri)
    {
        throw new InvalidOperationException("BlobClient must be authenticated with shared key");
    }

    BlobSasBuilder sasBuilder = new BlobSasBuilder
    {
        BlobContainerName = blobClient.BlobContainerName,
        BlobName = blobClient.Name,
        Resource = "b",
        ExpiresOn = DateTimeOffset.UtcNow.Add(validity)
    };
    sasBuilder.SetPermissions(permissions);

    return blobClient.GenerateSasUri(sasBuilder);
}

// Usage
var sasUri = GetBlobSasUri(blobClient, TimeSpan.FromHours(1), BlobSasPermissions.Read);
```

**Generate Container SAS:**

```csharp
public Uri GetContainerSasUri(BlobContainerClient containerClient)
{
    BlobSasBuilder sasBuilder = new BlobSasBuilder
    {
        BlobContainerName = containerClient.Name,
        Resource = "c",
        ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
    };
    sasBuilder.SetPermissions(BlobContainerSasPermissions.Read | BlobContainerSasPermissions.List);

    return containerClient.GenerateSasUri(sasBuilder);
}
```

**Generate Account SAS:**

```csharp
public string GetAccountSasToken(StorageSharedKeyCredential credential)
{
    AccountSasBuilder sasBuilder = new AccountSasBuilder
    {
        Services = AccountSasServices.Blobs | AccountSasServices.Files,
        ResourceTypes = AccountSasResourceTypes.Service | AccountSasResourceTypes.Container | AccountSasResourceTypes.Object,
        ExpiresOn = DateTimeOffset.UtcNow.AddHours(1),
        Protocol = SasProtocol.Https
    };
    sasBuilder.SetPermissions(AccountSasPermissions.Read | AccountSasPermissions.Write | AccountSasPermissions.List);

    return sasBuilder.ToSasQueryParameters(credential).ToString();
}
```

---

## MODULE 4: EXPLORE MICROSOFT GRAPH

### 4.1 Introduction to Microsoft Graph

**What is Microsoft Graph?**
- Unified API for Microsoft 365 services
- Single endpoint: `https://graph.microsoft.com`
- Access users, groups, mail, files, calendar, etc.
- RESTful API and client SDKs

**What You Can Access:**

| Service | Examples |
|---------|----------|
| Users | Profile, photo, manager |
| Groups | Members, conversations |
| Mail | Messages, folders, attachments |
| Calendar | Events, availability |
| OneDrive | Files, folders, sharing |
| Teams | Channels, messages, meetings |
| SharePoint | Sites, lists, documents |
| Outlook | Contacts, tasks |
| Security | Alerts, incidents |

**API Endpoint:**
```
https://graph.microsoft.com/{version}/{resource}

Examples:
https://graph.microsoft.com/v1.0/me
https://graph.microsoft.com/v1.0/users
https://graph.microsoft.com/v1.0/me/messages
https://graph.microsoft.com/beta/me/insights
```

**API Versions:**
- **v1.0**: Production-ready, stable
- **beta**: Preview features, may change

### 4.2 Querying Microsoft Graph

**Basic Query Structure:**

```http
{HTTP method} https://graph.microsoft.com/{version}/{resource}?{query-parameters}
```

**HTTP Methods:**

| Method | Description |
|--------|-------------|
| GET | Read resources |
| POST | Create resources |
| PUT | Replace resources |
| PATCH | Update resources |
| DELETE | Remove resources |

**Common Queries:**

```http
# Get current user
GET https://graph.microsoft.com/v1.0/me

# Get user's profile photo
GET https://graph.microsoft.com/v1.0/me/photo/$value

# Get user's messages
GET https://graph.microsoft.com/v1.0/me/messages

# Get user's calendar events
GET https://graph.microsoft.com/v1.0/me/events

# Get user's files in OneDrive
GET https://graph.microsoft.com/v1.0/me/drive/root/children

# Get specific user by ID
GET https://graph.microsoft.com/v1.0/users/{user-id}

# Get group members
GET https://graph.microsoft.com/v1.0/groups/{group-id}/members
```

### 4.3 Query Parameters

**OData Query Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| $select | Choose properties to return | `$select=displayName,mail` |
| $filter | Filter results | `$filter=startsWith(displayName,'A')` |
| $orderby | Sort results | `$orderby=displayName desc` |
| $top | Limit number of results | `$top=10` |
| $skip | Skip results (pagination) | `$skip=20` |
| $count | Include count of items | `$count=true` |
| $expand | Include related resources | `$expand=manager` |
| $search | Full-text search | `$search="displayName:John"` |

**Query Examples:**

```http
# Select specific properties
GET https://graph.microsoft.com/v1.0/users?$select=displayName,mail,jobTitle

# Filter users by department
GET https://graph.microsoft.com/v1.0/users?$filter=department eq 'Sales'

# Filter with startsWith
GET https://graph.microsoft.com/v1.0/users?$filter=startsWith(displayName,'A')

# Sort by name
GET https://graph.microsoft.com/v1.0/users?$orderby=displayName

# Pagination
GET https://graph.microsoft.com/v1.0/users?$top=10&$skip=20

# Get count
GET https://graph.microsoft.com/v1.0/users?$count=true

# Expand related resources
GET https://graph.microsoft.com/v1.0/me?$expand=manager

# Complex query
GET https://graph.microsoft.com/v1.0/users
  ?$select=displayName,mail,department
  &$filter=department eq 'Engineering'
  &$orderby=displayName
  &$top=25
```

### 4.4 Microsoft Graph SDK for .NET

**Installation:**

```bash
dotnet add package Microsoft.Graph
dotnet add package Azure.Identity
```

**Initialize GraphServiceClient:**

```csharp
using Azure.Identity;
using Microsoft.Graph;

// With client credentials (daemon app)
var credential = new ClientSecretCredential(
    tenantId, 
    clientId, 
    clientSecret);

var graphClient = new GraphServiceClient(credential);

// With interactive authentication
var credential = new InteractiveBrowserCredential(
    new InteractiveBrowserCredentialOptions
    {
        TenantId = tenantId,
        ClientId = clientId,
        RedirectUri = new Uri("http://localhost")
    });

var graphClient = new GraphServiceClient(credential);

// With device code (CLI/IoT)
var credential = new DeviceCodeCredential(
    new DeviceCodeCredentialOptions
    {
        TenantId = tenantId,
        ClientId = clientId,
        DeviceCodeCallback = (code, cancellation) =>
        {
            Console.WriteLine(code.Message);
            return Task.CompletedTask;
        }
    });

var graphClient = new GraphServiceClient(credential);
```

### 4.5 Graph SDK Operations

**Get Current User:**

```csharp
// Get user profile
var user = await graphClient.Me.GetAsync();
Console.WriteLine($"Name: {user.DisplayName}");
Console.WriteLine($"Email: {user.Mail}");

// Select specific properties
var user = await graphClient.Me.GetAsync(config =>
{
    config.QueryParameters.Select = new[] { "displayName", "mail", "jobTitle" };
});
```

**Get Users:**

```csharp
// Get all users
var users = await graphClient.Users.GetAsync();
foreach (var user in users.Value)
{
    Console.WriteLine($"{user.DisplayName} - {user.Mail}");
}

// With filters
var users = await graphClient.Users.GetAsync(config =>
{
    config.QueryParameters.Filter = "department eq 'Engineering'";
    config.QueryParameters.Select = new[] { "displayName", "mail" };
    config.QueryParameters.Top = 25;
});
```

**Work with Messages:**

```csharp
// Get messages
var messages = await graphClient.Me.Messages.GetAsync(config =>
{
    config.QueryParameters.Top = 10;
    config.QueryParameters.Select = new[] { "subject", "from", "receivedDateTime" };
    config.QueryParameters.Orderby = new[] { "receivedDateTime desc" };
});

// Send message
var message = new Message
{
    Subject = "Hello from Graph",
    Body = new ItemBody
    {
        ContentType = BodyType.Text,
        Content = "This is a test message."
    },
    ToRecipients = new List<Recipient>
    {
        new Recipient
        {
            EmailAddress = new EmailAddress { Address = "user@example.com" }
        }
    }
};

await graphClient.Me.SendMail.PostAsync(new SendMailPostRequestBody
{
    Message = message,
    SaveToSentItems = true
});
```

**Work with Calendar:**

```csharp
// Get events
var events = await graphClient.Me.Events.GetAsync(config =>
{
    config.QueryParameters.Top = 10;
    config.QueryParameters.Select = new[] { "subject", "start", "end", "location" };
});

// Create event
var newEvent = new Event
{
    Subject = "Team Meeting",
    Start = new DateTimeTimeZone
    {
        DateTime = "2024-03-20T14:00:00",
        TimeZone = "Pacific Standard Time"
    },
    End = new DateTimeTimeZone
    {
        DateTime = "2024-03-20T15:00:00",
        TimeZone = "Pacific Standard Time"
    },
    Location = new Location { DisplayName = "Conference Room A" }
};

await graphClient.Me.Events.PostAsync(newEvent);
```

**Work with OneDrive:**

```csharp
// List files in root
var files = await graphClient.Me.Drive.Root.Children.GetAsync();
foreach (var item in files.Value)
{
    Console.WriteLine($"{item.Name} ({item.Size} bytes)");
}

// Upload file
using var stream = File.OpenRead("document.pdf");
await graphClient.Me.Drive.Root
    .ItemWithPath("document.pdf")
    .Content
    .PutAsync(stream);

// Download file
var fileStream = await graphClient.Me.Drive.Root
    .ItemWithPath("document.pdf")
    .Content
    .GetAsync();
```

### 4.6 Handling Pagination

**Graph API Pagination:**

```csharp
// Manual pagination
var users = await graphClient.Users.GetAsync(config =>
{
    config.QueryParameters.Top = 25;
});

// Process first page
foreach (var user in users.Value)
{
    Console.WriteLine(user.DisplayName);
}

// Get next page using @odata.nextLink
while (users.OdataNextLink != null)
{
    users = await graphClient.Users
        .WithUrl(users.OdataNextLink)
        .GetAsync();
    
    foreach (var user in users.Value)
    {
        Console.WriteLine(user.DisplayName);
    }
}
```

**Using PageIterator:**

```csharp
// Automatic pagination with PageIterator
var users = await graphClient.Users.GetAsync(config =>
{
    config.QueryParameters.Top = 25;
});

var pageIterator = PageIterator<User, UserCollectionResponse>.CreatePageIterator(
    graphClient,
    users,
    (user) =>
    {
        Console.WriteLine(user.DisplayName);
        return true;  // Continue iteration
    });

await pageIterator.IterateAsync();
```

### 4.7 Batch Requests

**Batching Multiple Requests:**

```csharp
// Create batch request content
var batchRequestContent = new BatchRequestContent(graphClient);

// Add requests to batch
var userRequest = graphClient.Me.ToGetRequestInformation();
var messagesRequest = graphClient.Me.Messages.ToGetRequestInformation(config =>
{
    config.QueryParameters.Top = 5;
});
var eventsRequest = graphClient.Me.Events.ToGetRequestInformation(config =>
{
    config.QueryParameters.Top = 5;
});

var userRequestId = await batchRequestContent.AddBatchRequestStepAsync(userRequest);
var messagesRequestId = await batchRequestContent.AddBatchRequestStepAsync(messagesRequest);
var eventsRequestId = await batchRequestContent.AddBatchRequestStepAsync(eventsRequest);

// Execute batch
var batchResponse = await graphClient.Batch.PostAsync(batchRequestContent);

// Get responses
var user = await batchResponse.GetResponseByIdAsync<User>(userRequestId);
var messages = await batchResponse.GetResponseByIdAsync<MessageCollectionResponse>(messagesRequestId);
var events = await batchResponse.GetResponseByIdAsync<EventCollectionResponse>(eventsRequestId);
```

### 4.8 Microsoft Graph Permissions

**Permission Types:**

| Type | Description | Consent |
|------|-------------|---------|
| Delegated | User context | User or Admin |
| Application | App context (no user) | Admin only |

**Common Permissions:**

| Permission | Type | Description |
|------------|------|-------------|
| User.Read | Delegated | Read user profile |
| User.Read.All | Both | Read all users' profiles |
| Mail.Read | Both | Read mail |
| Mail.Send | Both | Send mail |
| Calendars.Read | Both | Read calendars |
| Calendars.ReadWrite | Both | Read/write calendars |
| Files.Read | Delegated | Read user's files |
| Files.Read.All | Both | Read all files |
| Group.Read.All | Both | Read groups |
| Directory.Read.All | Both | Read directory data |

**Request Permissions in App Registration:**
```
Azure Portal → App registrations → API permissions → Add permission → Microsoft Graph
```

---

## MODULE 5: MANAGED IDENTITIES (EXAM CRITICAL)

### 5.1 Introduction to Managed Identities

**What is a Managed Identity?**
- Azure-managed identity in Microsoft Entra ID
- Eliminates need for credentials in code
- Azure manages the identity lifecycle
- Used to authenticate to any service supporting Azure AD authentication
- No secrets, certificates, or keys to manage

**Key Benefits:**
- No credential management
- No code changes needed (via DefaultAzureCredential)
- Automatic token refresh
- Supported by Azure Key Vault, Storage, SQL, Service Bus, Event Hubs, etc.

### 5.2 System-Assigned vs User-Assigned Managed Identity

| Feature | System-Assigned | User-Assigned |
|---------|----------------|---------------|
| Creation | Created with the Azure resource | Created as standalone resource |
| Lifecycle | Tied to resource (deleted with resource) | Independent lifecycle |
| Sharing | Cannot be shared | Can be shared across resources |
| Use case | Single resource needs access | Multiple resources need same identity |
| Enabled | Toggle on existing resource | Create then assign to resource(s) |

```bash
# Enable system-assigned managed identity on App Service
az webapp identity assign --name myapp --resource-group myRG

# Create user-assigned managed identity
az identity create --name myIdentity --resource-group myRG

# Assign user-assigned identity to App Service
az webapp identity assign --name myapp --resource-group myRG \
  --identities /subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myIdentity
```

### 5.3 DefaultAzureCredential & Azure.Identity

**What is DefaultAzureCredential?**
- Simplified, opinionated credential chain from Azure.Identity
- Tries multiple authentication methods in order
- Works in BOTH local development AND Azure-hosted environments

**Credential Chain Order:**

```
1. EnvironmentCredential          → Environment variables (AZURE_CLIENT_ID, etc.)
2. WorkloadIdentityCredential     → Kubernetes workload identity
3. ManagedIdentityCredential      → System/User-assigned managed identity
4. SharedTokenCacheCredential     → Visual Studio shared token cache
5. VisualStudioCredential         → Visual Studio logged-in account
6. VisualStudioCodeCredential     → VS Code Azure Account extension
7. AzureCliCredential             → az login account
8. AzurePowerShellCredential      → Connect-AzAccount
9. AzureDeveloperCliCredential    → azd auth login
10. InteractiveBrowserCredential  → (only if explicitly enabled)
```

```csharp
using Azure.Identity;

// Simplest — works everywhere
var credential = new DefaultAzureCredential();

// With options (e.g., specify user-assigned identity)
var credential = new DefaultAzureCredential(new DefaultAzureCredentialOptions
{
    ManagedIdentityClientId = "<user-assigned-client-id>"
});

// Use with any Azure SDK client
var blobClient = new BlobServiceClient(
    new Uri("https://myaccount.blob.core.windows.net"),
    credential);

var secretClient = new SecretClient(
    new Uri("https://myvault.vault.azure.net"),
    credential);

var cosmosClient = new CosmosClient(
    "https://myaccount.documents.azure.com:443/",
    credential);
```

**Other Credential Classes:**

| Credential | Use Case |
|-----------|----------|
| `ManagedIdentityCredential` | Explicitly use managed identity only |
| `ClientSecretCredential` | App with client secret |
| `ClientCertificateCredential` | App with certificate |
| `InteractiveBrowserCredential` | Interactive sign-in |
| `DeviceCodeCredential` | Device code flow |
| `EnvironmentCredential` | Reads from env vars |
| `ChainedTokenCredential` | Custom chain of credentials |

```csharp
// Custom credential chain
var credential = new ChainedTokenCredential(
    new ManagedIdentityCredential(),
    new AzureCliCredential()
);
```

**Environment Variables for EnvironmentCredential:**

| Variable | Description |
|----------|-------------|
| `AZURE_CLIENT_ID` | Application (client) ID |
| `AZURE_TENANT_ID` | Directory (tenant) ID |
| `AZURE_CLIENT_SECRET` | Client secret value |
| `AZURE_CLIENT_CERTIFICATE_PATH` | Path to PFX/PEM certificate |

**⚠️ Exam Tip:** DefaultAzureCredential is the **recommended** way to authenticate in AZ-204. It automatically uses managed identity in Azure and developer credentials locally — NO code changes needed between environments.

### 5.4 Assigning RBAC Roles to Managed Identities

```bash
# Assign Storage Blob Data Contributor role to managed identity
az role assignment create \
  --assignee <managed-identity-principal-id> \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/myAccount

# Assign Key Vault Secrets User role
az role assignment create \
  --assignee <principal-id> \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myVault
```

**Common RBAC Roles for AZ-204:**

| Role | Service | Access |
|------|---------|--------|
| Storage Blob Data Reader | Blob Storage | Read blobs |
| Storage Blob Data Contributor | Blob Storage | Read/write/delete blobs |
| Storage Blob Data Owner | Blob Storage | Full access + RBAC |
| Storage Blob Delegator | Blob Storage | Generate user delegation SAS |
| Key Vault Secrets User | Key Vault | Read secrets |
| Key Vault Secrets Officer | Key Vault | Full secret management |
| Key Vault Crypto User | Key Vault | Use keys for crypto ops |
| Cosmos DB Account Reader | Cosmos DB | Read account metadata |
| Service Bus Data Sender | Service Bus | Send messages |
| Service Bus Data Receiver | Service Bus | Receive messages |

### 5.5 Azure RBAC Fundamentals (Quick Reference)

> **📌 See Foundation Section F.9–F.10 for detailed Azure RBAC concepts including role definitions, Actions vs DataActions, and custom roles.**

**Key Points for Managed Identity Context:**

| Component | Description |
|-----------|-------------|
| **Security Principal** | The managed identity itself (has an Object ID / Principal ID) |
| **Role Definition** | What the identity can do (e.g., Storage Blob Data Contributor) |
| **Scope** | Where the role applies (e.g., specific storage account) |

```bash
# Assign at resource scope (most common for managed identities)
az role assignment create --assignee <managed-identity-principal-id> \
  --role "Storage Blob Data Reader" \
  --scope /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>
```

**⚠️ Exam Tip:** Contributor can create/delete resources but CANNOT assign RBAC roles and CANNOT access data plane. For managed identities, always assign the **least privilege data-plane role** (e.g., Storage Blob Data Reader, not Contributor).

### 5.6 Control Plane vs Data Plane (Quick Reference)

> **📌 See Foundation Section F.10 for detailed role definitions, Actions vs DataActions breakdown.**

| Aspect | Control Plane | Data Plane |
|--------|--------------|------------|
| **What** | Manage resources (create, delete, configure) | Access data inside resources |
| **URL** | `management.azure.com` | Service-specific (e.g., `*.blob.core.windows.net`) |
| **Roles** | Owner, Contributor, Reader | Storage Blob Data Reader, Key Vault Secrets User |

**⚠️ Exam Tip:** Contributor role lets you create a Key Vault but does NOT let you read secrets inside it. You need a data-plane role (Key Vault Secrets User) for that.

### 5.7 Azure AD Tenant Basics

> **📌 See Foundation Section F.1 for Azure AD overview and F.3 for Azure AD objects.**

**Tenant ↔ Subscription ↔ Resource Relationship:**

```
Azure AD Tenant (Organization Identity)
    ├── Subscription 1 (billing boundary)
    │       ├── Resource Group A → Resources
    │       └── Resource Group B → Resources
    ├── Subscription 2
    │       └── Resource Group C → Resources
    └── (A subscription trusts exactly ONE tenant)
```

- Each subscription trusts **one** Azure AD tenant
- A tenant can have **multiple** subscriptions
- Managed identities exist within the tenant and can access resources across subscriptions (with proper RBAC)

### 5.8 Managed Identity Token Acquisition (Behind the Scenes)

**How Managed Identity Gets Tokens:**
- Azure VMs and services use the **Instance Metadata Service (IMDS)**
- Endpoint: `http://169.254.169.254/metadata/identity/oauth2/token`
- Only accessible from within the Azure resource (link-local address)
- No credentials needed — Azure platform handles authentication

```http
GET http://169.254.169.254/metadata/identity/oauth2/token
  ?api-version=2018-02-01
  &resource=https://vault.azure.net
Header: Metadata: true
```

**Response:**
```json
{
  "access_token": "eyJ0eXAi...",
  "expires_on": "1706003600",
  "resource": "https://vault.azure.net",
  "token_type": "Bearer"
}
```

**For App Service (different endpoint):**
```http
GET {IDENTITY_ENDPOINT}?api-version=2019-08-01&resource=https://vault.azure.net
Header: X-IDENTITY-HEADER: {IDENTITY_HEADER}
```

**⚠️ Exam Tip:** You rarely call these endpoints directly — the Azure SDKs (via `DefaultAzureCredential` / `ManagedIdentityCredential`) handle this automatically. But know that IMDS (169.254.169.254) is how it works under the hood.

---

## MODULE 6: AZURE KEY VAULT (EXAM CRITICAL)

### 6.1 Introduction to Azure Key Vault

**What is Azure Key Vault?**
- Cloud service for securely storing and managing secrets, keys, and certificates
- Centralized secret management
- Hardware Security Modules (HSM) for keys
- Auditing and logging

**What Key Vault Stores:**

| Object Type | Description | Use Case |
|-------------|-------------|----------|
| **Secrets** | String values (passwords, connection strings, API keys) | App configuration |
| **Keys** | Cryptographic keys (RSA, EC) | Encryption, signing |
| **Certificates** | X.509 certificates | TLS/SSL, code signing |

### 6.2 Key Vault Access Control

**Two Access Models:**

#### **1. Vault Access Policy (Legacy)**
- Per-vault permission model
- Assign get, list, set, delete per object type
- Applied to a security principal (user, app, managed identity)

```bash
# Set access policy
az keyvault set-policy --name myVault \
  --object-id <principal-id> \
  --secret-permissions get list set delete \
  --key-permissions get list create \
  --certificate-permissions get list
```

#### **2. Azure RBAC (Recommended)**
- Uses standard Azure RBAC model
- More granular, supports conditions
- Consistent with other Azure services

```bash
# Enable RBAC on Key Vault
az keyvault update --name myVault --resource-group myRG \
  --enable-rbac-authorization true

# Assign role
az role assignment create \
  --assignee <principal-id> \
  --role "Key Vault Secrets User" \
  --scope /subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myVault
```

**Key Vault RBAC Roles:**

| Role | Permissions |
|------|-------------|
| Key Vault Administrator | Full management of all Key Vault objects |
| Key Vault Secrets Officer | All operations on secrets |
| Key Vault Secrets User | Read secret contents |
| Key Vault Crypto Officer | All operations on keys |
| Key Vault Crypto User | Encrypt/decrypt/sign/verify with keys |
| Key Vault Certificates Officer | All operations on certificates |
| Key Vault Reader | Read metadata (not secret values) |

**⚠️ Exam Tip:** Key Vault Reader can see metadata but CANNOT read secret values. You need Key Vault Secrets User for that.

### 6.3 Key Vault SDK (.NET)

**Packages:**

```bash
dotnet add package Azure.Security.KeyVault.Secrets
dotnet add package Azure.Security.KeyVault.Keys
dotnet add package Azure.Security.KeyVault.Certificates
dotnet add package Azure.Identity
```

**SecretClient — Working with Secrets:**

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

// Create client
var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net"),
    new DefaultAzureCredential());

// Set a secret
KeyVaultSecret secret = await client.SetSecretAsync("MySecret", "MySecretValue");

// Get a secret
KeyVaultSecret retrieved = await client.GetSecretAsync("MySecret");
string value = retrieved.Value;

// Get specific version
KeyVaultSecret versioned = await client.GetSecretAsync("MySecret", version: "abc123");

// List secrets (metadata only, not values)
await foreach (SecretProperties props in client.GetPropertiesOfSecretsAsync())
{
    Console.WriteLine($"{props.Name}, Enabled: {props.Enabled}");
}

// Update secret properties
SecretProperties properties = retrieved.Properties;
properties.ExpiresOn = DateTimeOffset.UtcNow.AddYears(1);
await client.UpdateSecretPropertiesAsync(properties);

// Delete a secret (soft delete)
DeleteSecretOperation operation = await client.StartDeleteSecretAsync("MySecret");
await operation.WaitForCompletionAsync();

// Purge a deleted secret (permanent)
await client.PurgeDeletedSecretAsync("MySecret");

// Recover a deleted secret
RecoverDeletedSecretOperation recovery = await client.StartRecoverDeletedSecretAsync("MySecret");
```

**KeyClient — Working with Keys:**

```csharp
using Azure.Security.KeyVault.Keys;
using Azure.Security.KeyVault.Keys.Cryptography;

var keyClient = new KeyClient(
    new Uri("https://myvault.vault.azure.net"),
    new DefaultAzureCredential());

// Create RSA key
KeyVaultKey rsaKey = await keyClient.CreateRsaKeyAsync(new CreateRsaKeyOptions("myRsaKey")
{
    KeySize = 2048,
    ExpiresOn = DateTimeOffset.UtcNow.AddYears(1)
});

// Use key for encryption
var cryptoClient = new CryptographyClient(rsaKey.Id, new DefaultAzureCredential());
byte[] plaintext = Encoding.UTF8.GetBytes("Hello, Key Vault!");
EncryptResult encrypted = await cryptoClient.EncryptAsync(EncryptionAlgorithm.RsaOaep, plaintext);
DecryptResult decrypted = await cryptoClient.DecryptAsync(EncryptionAlgorithm.RsaOaep, encrypted.Ciphertext);
```

### 6.4 Key Vault Features

**Soft Delete:**
- Deleted objects retained for 7-90 days (default 90)
- Can recover deleted objects within retention period
- Enabled by default (mandatory since Feb 2025)

**Purge Protection:**
- Prevents permanent deletion during retention period
- Even Key Vault admins cannot purge
- Must wait for retention period to expire

```bash
# Create Key Vault with purge protection
az keyvault create --name myVault --resource-group myRG \
  --location eastus \
  --enable-soft-delete true \
  --soft-delete-retention-days 90 \
  --enable-purge-protection true
```

**Secret Versioning:**
- Each update creates a new version
- Previous versions remain accessible
- Can set activation and expiration dates per version

**Key Vault References in App Service:**
```
@Microsoft.KeyVault(VaultName=myVault;SecretName=MySecret)
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/MySecret/)
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/MySecret/abc123)
```

**⚠️ Exam Tip:** Key Vault References in App Service require the app to have a managed identity with `Get` secret permission. The syntax uses `@Microsoft.KeyVault(...)` — know both VaultName and SecretUri formats.

---

## MODULE 7: APP SERVICE AUTHENTICATION (EASY AUTH)

### 7.1 Built-in Authentication

**What is Easy Auth?**
- Built-in authentication/authorization in Azure App Service
- No code changes required
- Runs as middleware in the App Service platform
- Handles token validation, session management, identity provider integration

**How It Works:**
```
Client Request → App Service Platform (Easy Auth Middleware) → Your App
                      │
                      ├── Validates tokens
                      ├── Manages authenticated sessions
                      ├── Injects identity info into request headers
                      └── Redirects unauthenticated users (if configured)
```

### 7.2 Supported Identity Providers

| Provider | Sign-in Endpoint |
|----------|-----------------|
| Microsoft Entra ID | `/.auth/login/aad` |
| Facebook | `/.auth/login/facebook` |
| Google | `/.auth/login/google` |
| Twitter | `/.auth/login/twitter` |
| GitHub | `/.auth/login/github` |
| Apple | `/.auth/login/apple` |
| Any OpenID Connect provider | `/.auth/login/<provider>` |

### 7.3 Authentication Flow

**Two Flows:**

#### **1. Server-directed Flow (No SDK)**
- App Service handles the entire flow
- Browser-based apps (redirects)
- App Service manages tokens in a token store

```
User → /.auth/login/aad → Login Page → Redirect back with session cookie
```

#### **2. Client-directed Flow (With SDK)**
- Client app handles authentication (e.g., MSAL)
- Posts token to App Service for validation
- Common for mobile/SPA apps

```
Client → MSAL → Get token → POST /.auth/login/aad (with token) → Session
```

### 7.4 Authentication Settings

**Action for Unauthenticated Requests:**

| Setting | Behavior |
|---------|----------|
| Allow unauthenticated | Pass through (app handles auth) |
| Require authentication | Redirect to login page (web) or return 401 (API) |

**Token Store:**
- Built-in token store (filesystem-based)
- Stores tokens from identity providers
- Accessible via `/.auth/me` endpoint
- Returns user claims as JSON

**Request Headers (Injected by Easy Auth):**

| Header | Description |
|--------|-------------|
| `X-MS-CLIENT-PRINCIPAL` | Base64-encoded claims |
| `X-MS-CLIENT-PRINCIPAL-ID` | User/app object ID |
| `X-MS-CLIENT-PRINCIPAL-NAME` | User display name |
| `X-MS-CLIENT-PRINCIPAL-IDP` | Identity provider name |
| `X-MS-TOKEN-AAD-ACCESS-TOKEN` | Azure AD access token |
| `X-MS-TOKEN-AAD-ID-TOKEN` | Azure AD ID token |
| `X-MS-TOKEN-AAD-REFRESH-TOKEN` | Azure AD refresh token |

```csharp
// Access Easy Auth claims in ASP.NET Core
[HttpGet("me")]
public IActionResult GetMe()
{
    var principal = Request.HttpContext.User;
    var name = principal.Identity?.Name;
    var claims = principal.Claims.Select(c => new { c.Type, c.Value });
    return Ok(new { name, claims });
}

// Access via headers
var principalId = Request.Headers["X-MS-CLIENT-PRINCIPAL-ID"].FirstOrDefault();
```

```bash
# Enable Easy Auth with Azure AD
az webapp auth update --name myapp --resource-group myRG \
  --enabled true \
  --action LoginWithAzureActiveDirectory \
  --aad-client-id <client-id> \
  --aad-token-issuer-url "https://sts.windows.net/<tenant-id>/"
```

**⚠️ Exam Tip:** Easy Auth runs BEFORE your application code. If set to "Require authentication," unauthenticated requests NEVER reach your code. For APIs, it returns 401; for web apps, it redirects to login.

---

## MODULE 8: TOKENS, JWT, AND CLAIMS (EXAM CRITICAL)

### 8.1 OAuth 2.0 Token Types

| Token | Purpose | Lifetime | Audience |
|-------|---------|----------|----------|
| **Access Token** | Authorize API calls | ~1 hour (default) | API (resource server) |
| **ID Token** | Prove user identity | Session-based | Client application |
| **Refresh Token** | Get new access tokens | Up to 90 days | Authorization server |

**Token Flow:**
```
Login → Receive: ID Token + Access Token + Refresh Token
           │           │              │
           │           │              └── Use to get new access/ID tokens silently
           │           └── Send to API in Authorization: Bearer <token>
           └── Used by client app to show user info
```

### 8.2 JWT (JSON Web Token) Structure

```
eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0.signature
       HEADER              PAYLOAD                   SIGNATURE
```

**Header:**
```json
{
  "alg": "RS256",       // Signing algorithm
  "typ": "JWT",         // Token type
  "kid": "key-id-123"   // Key ID (used to find public key)
}
```

**Payload (Claims):**
```json
{
  "aud": "api://my-api-id",            // Audience - who token is for
  "iss": "https://login.microsoftonline.com/<tenant-id>/v2.0",  // Issuer
  "iat": 1706000000,                    // Issued at (Unix timestamp)
  "nbf": 1706000000,                    // Not before
  "exp": 1706003600,                    // Expiration
  "sub": "user-object-id",              // Subject (unique user ID)
  "oid": "user-object-id",              // Object ID in Azure AD
  "tid": "tenant-id",                   // Tenant ID
  "preferred_username": "user@contoso.com",
  "name": "John Doe",
  "scp": "User.Read Mail.Read",         // Scopes (delegated permissions)
  "roles": ["Admin", "Reader"],          // App roles (application permissions)
  "azp": "client-app-id"                // Authorized party (client ID)
}
```

### 8.3 Key Claims for AZ-204

| Claim | Description | Example |
|-------|-------------|---------|
| `aud` | Audience — API the token is for | `api://my-api` or resource URI |
| `iss` | Issuer — who issued the token | `https://login.microsoftonline.com/<tid>/v2.0` |
| `sub` | Subject — unique user identifier | GUID |
| `oid` | Object ID — user's ID in Azure AD | GUID |
| `tid` | Tenant ID — organization's Azure AD | GUID |
| `scp` | Scopes — delegated permissions | `"User.Read Mail.Read"` |
| `roles` | App roles — assigned roles | `["Admin"]` |
| `azp` / `appid` | Client app ID that requested token | GUID |
| `exp` | Expiration time (Unix timestamp) | `1706003600` |
| `nbf` | Not valid before | `1706000000` |
| `iat` | Issued at | `1706000000` |
| `preferred_username` | Username | `user@contoso.com` |

**⚠️ Exam Tip:**
- `scp` (scopes) = **delegated** permissions (user context)
- `roles` = **application** permissions or app roles (no user)
- Know the difference — exam frequently tests this

### 8.4 The `.default` Scope

```csharp
// For daemon apps (client credentials) — always use .default
string[] scopes = { "https://graph.microsoft.com/.default" };
var result = await app.AcquireTokenForClient(scopes).ExecuteAsync();

// .default means: "Give me all permissions that have been consented for this app"
// NOT the same as specific scopes like "User.Read"
```

**When to use `.default`:**
- Client credentials flow (daemon apps) — **always**
- When you want all consented permissions at once
- When the resource doesn't support individual scopes

### 8.5 Authority URLs

| Authority | URL | Use Case |
|-----------|-----|----------|
| Single tenant | `https://login.microsoftonline.com/{tenant-id}/v2.0` | LOB apps |
| Multi-tenant (orgs) | `https://login.microsoftonline.com/organizations/v2.0` | Any org |
| Multi-tenant + personal | `https://login.microsoftonline.com/common/v2.0` | Any account |
| Personal only | `https://login.microsoftonline.com/consumers/v2.0` | Personal MS |
| Azure AD B2C | `https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}` | B2C |

**v1.0 vs v2.0 Endpoints:**

| Feature | v1.0 | v2.0 |
|---------|------|------|
| Scopes | Resources | Scopes (granular) |
| Personal accounts | No | Yes |
| Incremental consent | No | Yes |
| App registrations | Azure AD | App Registrations portal |

**⚠️ Exam Tip:** v2.0 endpoint uses **scopes** (e.g., `User.Read`). v1.0 uses **resources** (e.g., `https://graph.microsoft.com`). AZ-204 focuses on v2.0.

---

## MODULE 9: APP ROLES & CLAIMS-BASED AUTHORIZATION

### 9.1 App Roles

**What are App Roles?**
- Custom roles defined in app registration
- Assigned to users, groups, or applications
- Appear in `roles` claim in token
- Enable role-based access control in your app

**Defining App Roles in Manifest:**

```json
"appRoles": [
  {
    "allowedMemberTypes": ["User"],
    "displayName": "Admin",
    "id": "a1b2c3d4-...",
    "isEnabled": true,
    "description": "Administrators can manage all aspects",
    "value": "Admin"
  },
  {
    "allowedMemberTypes": ["User", "Application"],
    "displayName": "Reader",
    "id": "e5f6g7h8-...",
    "isEnabled": true,
    "description": "Readers can view data",
    "value": "Reader"
  }
]
```

**Using App Roles in Code:**

```csharp
// ASP.NET Core — check role in controller
[Authorize(Roles = "Admin")]
[HttpDelete("data/{id}")]
public IActionResult DeleteData(string id)
{
    return Ok("Deleted");
}

// Check role programmatically
[HttpGet("data")]
public IActionResult GetData()
{
    if (User.IsInRole("Admin"))
    {
        return Ok("Admin data");
    }
    return Ok("Regular data");
}

// Access all roles from claims
var roles = User.Claims
    .Where(c => c.Type == ClaimTypes.Role)
    .Select(c => c.Value);
```

### 9.2 Application Manifest

**Key Manifest Properties for AZ-204:**

| Property | Description |
|----------|-------------|
| `appId` | Application (client) ID |
| `appRoles` | Custom app roles |
| `groupMembershipClaims` | Groups included in token (None, SecurityGroup, All) |
| `optionalClaims` | Additional claims in ID/Access tokens |
| `signInAudience` | Supported account types |
| `requiredResourceAccess` | API permissions the app needs |
| `oauth2AllowImplicitFlow` | Allow implicit flow (legacy) |
| `accessTokenAcceptedVersion` | Token version (1 or 2) |

**Optional Claims Configuration:**

```json
"optionalClaims": {
  "idToken": [
    { "name": "email", "essential": false },
    { "name": "upn", "essential": false }
  ],
  "accessToken": [
    { "name": "ipaddr", "essential": false }
  ]
}
```

**Group Membership Claims:**

```json
"groupMembershipClaims": "SecurityGroup"
// Options: "None", "SecurityGroup", "DirectoryRole", "All", "ApplicationGroup"
// Groups appear in "groups" claim in token
// If too many groups (>200), token includes _claim_names with a URL to fetch groups
```

### 9.3 Claims-Based Authorization

```csharp
// ASP.NET Core — Policy-based authorization with claims
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireClaim("roles", "Admin"));

    options.AddPolicy("PremiumUser", policy =>
        policy.RequireClaim("extension_SubscriptionLevel", "Premium"));

    options.AddPolicy("SpecificTenant", policy =>
        policy.RequireClaim("tid", "your-tenant-id"));
});

// Use in controller
[Authorize(Policy = "AdminOnly")]
[HttpPost("admin/action")]
public IActionResult AdminAction() { ... }
```

---

## MODULE 10: TOKEN VALIDATION & WEB API PROTECTION

### 10.1 Protecting a Web API

**ASP.NET Core — Microsoft.Identity.Web:**

```bash
dotnet add package Microsoft.Identity.Web
```

```csharp
// Program.cs
using Microsoft.Identity.Web;

var builder = WebApplication.CreateBuilder(args);

// Add JWT Bearer authentication
builder.Services.AddMicrosoftIdentityWebApiAuthentication(
    builder.Configuration, "AzureAd");

builder.Services.AddAuthorization();
builder.Services.AddControllers();

var app = builder.Build();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

```json
// appsettings.json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "your-tenant-id",
    "ClientId": "your-api-client-id",
    "Audience": "api://your-api-client-id"
  }
}
```

```csharp
// Protected API controller
[Authorize]
[ApiController]
[Route("api/[controller]")]
public class DataController : ControllerBase
{
    [HttpGet]
    [RequiredScope("Data.Read")]  // Require specific scope
    public IActionResult Get()
    {
        // Access user claims
        var userId = User.FindFirst("oid")?.Value;
        var tenantId = User.FindFirst("tid")?.Value;
        return Ok(new { userId, tenantId });
    }

    [HttpPost]
    [RequiredScope("Data.Write")]
    [Authorize(Roles = "Admin")]  // Require role AND scope
    public IActionResult Post([FromBody] object data)
    {
        return Created("", data);
    }
}
```

### 10.2 Manual Token Validation

```csharp
// Manual JWT validation (for understanding — SDK handles this normally)
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;

var tokenHandler = new JwtSecurityTokenHandler();
var validationParameters = new TokenValidationParameters
{
    ValidateIssuer = true,
    ValidIssuer = $"https://login.microsoftonline.com/{tenantId}/v2.0",
    
    ValidateAudience = true,
    ValidAudience = "api://my-api-id",
    
    ValidateLifetime = true,
    
    ValidateIssuerSigningKey = true,
    IssuerSigningKeys = signingKeys,  // From OpenID Connect metadata
    
    ClockSkew = TimeSpan.FromMinutes(5)
};

var principal = tokenHandler.ValidateToken(token, validationParameters, out var validatedToken);
```

**What Gets Validated:**
1. **Signature** — Token wasn't tampered with
2. **Issuer (`iss`)** — Token came from expected Azure AD tenant
3. **Audience (`aud`)** — Token was meant for this API
4. **Expiration (`exp`)** — Token hasn't expired
5. **Not Before (`nbf`)** — Token is already valid

**⚠️ Exam Tip:** Always validate BOTH issuer AND audience. For multi-tenant apps, use `ValidateIssuer = false` or custom issuer validation logic.

### 10.3 Multi-Tenant Token Validation

```csharp
// Multi-tenant: validate issuer dynamically
var validationParameters = new TokenValidationParameters
{
    ValidateIssuer = true,
    ValidIssuers = new[]
    {
        "https://login.microsoftonline.com/{tenantId1}/v2.0",
        "https://login.microsoftonline.com/{tenantId2}/v2.0"
    },
    // OR use custom validation
    IssuerValidator = (issuer, token, parameters) =>
    {
        // Check against allowed tenant list
        var tid = ((JwtSecurityToken)token).Claims
            .First(c => c.Type == "tid").Value;
        if (IsAllowedTenant(tid))
            return issuer;
        throw new SecurityTokenInvalidIssuerException("Tenant not allowed");
    }
};
```

---

## MODULE 11: MICROSOFT.IDENTITY.WEB & ASP.NET CORE INTEGRATION

### 11.1 Microsoft.Identity.Web Overview

**What is it?**
- Higher-level library on top of MSAL
- Simplifies ASP.NET Core integration
- Handles token acquisition, caching, and API protection
- Works with App Service Easy Auth

**Packages:**

| Package | Purpose |
|---------|---------|
| `Microsoft.Identity.Web` | Core library, API protection |
| `Microsoft.Identity.Web.UI` | UI for sign-in/sign-out |
| `Microsoft.Identity.Web.MicrosoftGraph` | Graph SDK integration |
| `Microsoft.Identity.Web.DownstreamApi` | Call downstream APIs |

### 11.2 Web App Configuration (Sign-in Users)

```csharp
// Program.cs — Web app that signs in users and calls Graph
var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddMicrosoftIdentityWebAppAuthentication(builder.Configuration)
    .EnableTokenAcquisitionToCallDownstreamApi(new[] { "User.Read" })
    .AddMicrosoftGraph(builder.Configuration.GetSection("Graph"))
    .AddDistributedTokenCaches();  // For production

builder.Services.AddControllersWithViews()
    .AddMicrosoftIdentityUI();  // Adds /MicrosoftIdentity/Account/SignIn etc.

var app = builder.Build();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

### 11.3 Web API Configuration (Protect API)

```csharp
// Program.cs — Protected Web API
var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddMicrosoftIdentityWebApiAuthentication(builder.Configuration)
    .EnableTokenAcquisitionToCallDownstreamApi()
    .AddDownstreamApi("DownstreamApi", builder.Configuration.GetSection("DownstreamApi"))
    .AddInMemoryTokenCaches();

builder.Services.AddControllers();

var app = builder.Build();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

### 11.4 Calling Downstream APIs

```csharp
// Inject and use IDownstreamApi
[Authorize]
[ApiController]
public class MyController : ControllerBase
{
    private readonly IDownstreamApi _downstreamApi;

    public MyController(IDownstreamApi downstreamApi)
    {
        _downstreamApi = downstreamApi;
    }

    [HttpGet]
    public async Task<IActionResult> Get()
    {
        var result = await _downstreamApi.CallApiForUserAsync<object>(
            "DownstreamApi",
            options => { options.RelativePath = "api/data"; });
        return Ok(result);
    }
}
```

---

## MODULE 12: ADVANCED AUTHENTICATION TOPICS

### 12.1 Zero Trust Principles

**Three Pillars (Commonly Tested):**

| Principle | Description |
|-----------|-------------|
| **Verify explicitly** | Always authenticate and authorize based on all data points |
| **Least privilege access** | Limit user access with JIT/JEA, risk-based policies |
| **Assume breach** | Minimize blast radius, encrypt, verify end-to-end |

### 12.2 Azure AD B2C (Basic Awareness)

- Separate Azure AD tenant for **consumer-facing** apps
- Custom sign-up/sign-in flows (user flows or custom policies)
- Social identity providers (Google, Facebook, etc.)
- Different from Azure AD B2B (which is for partner/guest access)
- Authority URL: `https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}`

### 12.3 Workload Identity Federation (Passwordless for External)

- Allows external workloads (GitHub Actions, Kubernetes) to access Azure without secrets
- Uses federated credentials instead of client secrets
- Configured in app registration → Certificates & secrets → Federated credentials

```bash
# Create federated credential for GitHub Actions
az ad app federated-credential create --id <app-id> --parameters '{
  "name": "github-main",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:myorg/myrepo:ref:refs/heads/main",
  "audiences": ["api://AzureADTokenExchange"]
}'
```

### 12.4 Certificate vs Secret — Detailed Comparison

| Aspect | Client Secret | Certificate |
|--------|--------------|-------------|
| Security | Lower (string can be copied) | Higher (private key stays local) |
| Rotation | Must update secret value | Upload new cert |
| Max lifetime | 2 years (recommended < 6 months) | Configurable |
| Recommended for | Development, testing | Production |
| Configuration | `.WithClientSecret(secret)` | `.WithCertificate(cert)` |

### 12.5 Token Lifetime and Refresh

| Token | Default Lifetime | Configurable? |
|-------|-----------------|---------------|
| Access Token | ~1 hour (60-90 min) | Yes (via Token Lifetime Policy, 10 min - 1 day) |
| ID Token | ~1 hour | Yes |
| Refresh Token | Up to 90 days | Yes (sliding window) |
| User Delegation Key | Up to 7 days | Yes (at creation) |

**MSAL Handles Refresh Automatically:**
```csharp
// MSAL silently refreshes expired access tokens using refresh tokens
// You just call AcquireTokenSilent — MSAL does the rest
var result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync();
// If access token expired but refresh token valid → new access token automatically
// If refresh token expired → MsalUiRequiredException
```

### 12.6 Secure Token Storage Patterns

| Environment | Storage | Method |
|-------------|---------|--------|
| ASP.NET Core (dev) | In-memory | `.AddInMemoryTokenCaches()` |
| ASP.NET Core (prod) | Redis / SQL | `.AddDistributedTokenCaches()` |
| Desktop / Mobile | Encrypted file | `MsalCacheHelper` |
| Daemon (single machine) | In-memory | Default (no user tokens) |

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**1. Microsoft Identity Platform:**
- Application vs Service Principal (1:many relationship)
- Delegated vs Application permissions (scp vs roles claims)
- Consent types (user, admin, incremental/dynamic)
- Authentication flows (Auth Code, PKCE, Client Credentials, Device Code, OBO)

**2. MSAL:**
- Public vs Confidential clients (what makes them different)
- Always try Silent first, then Interactive
- Token caching (in-memory vs distributed)
- Error handling (MsalUiRequiredException → interactive)

**3. Managed Identities:**
- System-assigned (lifecycle tied to resource) vs User-assigned (shared, independent)
- DefaultAzureCredential chain order
- How to assign RBAC roles to managed identities
- Common role assignments for Storage, Key Vault, Cosmos DB

**4. Azure Key Vault:**
- Secrets vs Keys vs Certificates
- Access Policies (legacy) vs RBAC (recommended)
- SDK: SecretClient, KeyClient, CertificateClient
- Key Vault References in App Service (@Microsoft.KeyVault(...))
- Soft delete & purge protection

**5. App Service Easy Auth:**
- Server-directed vs Client-directed flow
- Request headers injected (X-MS-CLIENT-PRINCIPAL-*)
- Action for unauthenticated requests (allow vs require)
- Runs BEFORE your application code

**6. Tokens & JWT:**
- ID Token vs Access Token vs Refresh Token
- JWT structure (header.payload.signature)
- Key claims: aud, iss, sub, oid, tid, scp, roles, exp
- `.default` scope for client credentials flow

**7. App Roles & Claims:**
- Define in app manifest (appRoles)
- Appear in `roles` claim
- [Authorize(Roles = "...")] in ASP.NET Core
- groupMembershipClaims for group-based access

**8. API Protection:**
- Microsoft.Identity.Web for ASP.NET Core
- Token validation (issuer, audience, signature, expiration)
- [RequiredScope] attribute
- Multi-tenant issuer validation

**9. SAS Tokens:**
- User Delegation (most secure) > Service > Account
- Stored access policies (revocable, max 5)
- SAS parameters (sv, ss, srt, sp, se, st, spr, sig)

**10. Microsoft Graph:**
- Base URL, v1.0 vs beta
- OData parameters ($select, $filter, $orderby, $top, $expand)
- SDK: GraphServiceClient
- Pagination: @odata.nextLink, PageIterator

### Sample Exam Questions

#### Microsoft Identity Platform

**Q1:** Which type of permission requires admin consent and allows an app to access resources without a signed-in user?
- A) Delegated permission
- B) Application permission
- C) User permission
- D) Implicit permission

**Answer: B) Application permission**
Explanation: Application permissions allow apps to act as themselves without a user, always requiring admin consent.

---

**Q2:** What is the relationship between an application object and a service principal?
- A) One application object per service principal
- B) One service principal per application object
- C) One application object can have multiple service principals
- D) They are the same thing

**Answer: C) One application object can have multiple service principals**
Explanation: An application object in the home tenant can have service principals in multiple tenants where the app is used.

---

**Q3:** Which authentication flow should you use for a single-page application (SPA)?
- A) Client credentials flow
- B) Implicit flow
- C) Authorization code flow with PKCE
- D) Device code flow

**Answer: C) Authorization code flow with PKCE**
Explanation: SPAs should use Authorization Code flow with PKCE as it's more secure than the legacy Implicit flow.

---

**Q4:** What is the purpose of consent in the Microsoft identity platform?
- A) To encrypt data
- B) To grant an application permission to access resources
- C) To create application objects
- D) To manage token lifetime

**Answer: B) To grant an application permission to access resources**
Explanation: Consent is the process where users or admins grant applications permission to access protected resources.

---

**Q5:** Which flow should a daemon service use to authenticate?
- A) Authorization code flow
- B) Device code flow
- C) Client credentials flow
- D) On-behalf-of flow

**Answer: C) Client credentials flow**
Explanation: Daemon services with no user interaction should use client credentials flow with application permissions.

---

#### MSAL

**Q6:** What is the difference between a public client and a confidential client in MSAL?
- A) Public clients are faster
- B) Confidential clients can securely store secrets
- C) Public clients support more platforms
- D) There is no difference

**Answer: B) Confidential clients can securely store secrets**
Explanation: Confidential clients run on secure servers and can store secrets, while public clients run on user devices and cannot.

---

**Q7:** Which method should you call first when trying to acquire a token in MSAL?
- A) AcquireTokenInteractive
- B) AcquireTokenSilent
- C) AcquireTokenForClient
- D) AcquireTokenWithDeviceCode

**Answer: B) AcquireTokenSilent**
Explanation: Always try AcquireTokenSilent first to get tokens from cache, falling back to interactive methods if it fails.

---

**Q8:** What exception indicates that user interaction is required to acquire a token?
- A) MsalClientException
- B) MsalServiceException
- C) MsalUiRequiredException
- D) TokenExpiredException

**Answer: C) MsalUiRequiredException**
Explanation: MsalUiRequiredException is thrown when silent authentication fails and user interaction is needed.

---

**Q9:** Which credential type is more secure for production confidential client applications?
- A) Client secret
- B) Certificate
- C) Username/password
- D) They are equally secure

**Answer: B) Certificate**
Explanation: Certificates are more secure than client secrets because the private key never leaves the application.

---

**Q10:** What does the `.default` scope represent when calling Microsoft Graph APIs?
- A) Default user permissions
- B) All permissions consented for the app
- C) Read-only permissions
- D) Admin permissions

**Answer: B) All permissions consented for the app**
Explanation: The `.default` scope requests all permissions that have been consented for the application.

---

#### Shared Access Signatures

**Q11:** Which SAS type is recommended for production workloads?
- A) Account SAS
- B) Service SAS
- C) User delegation SAS
- D) All are equally recommended

**Answer: C) User delegation SAS**
Explanation: User delegation SAS is most secure as it's secured with Azure AD credentials rather than storage account keys.

---

**Q12:** What is the primary benefit of using stored access policies with SAS?
- A) Better performance
- B) Ability to revoke SAS tokens
- C) Longer expiration times
- D) Cross-account access

**Answer: B) Ability to revoke SAS tokens**
Explanation: Stored access policies allow you to revoke SAS tokens by modifying or deleting the policy without regenerating account keys.

---

**Q13:** What does the `sp=r` parameter in a SAS URI represent?
- A) Service protocol = required
- B) Signed permissions = read
- C) Storage path = root
- D) Start point = recent

**Answer: B) Signed permissions = read**
Explanation: `sp` is signed permissions and `r` indicates read permission.

---

**Q14:** How many stored access policies can you create per container?
- A) 1
- B) 3
- C) 5
- D) Unlimited

**Answer: C) 5**
Explanation: You can create a maximum of 5 stored access policies per container, queue, table, or file share.

---

**Q15:** Which parameter should you set to restrict a SAS to HTTPS only?
- A) sp
- B) ss
- C) spr
- D) sig

**Answer: C) spr**
Explanation: The `spr` (signed protocol) parameter should be set to `https` to restrict access to HTTPS only.

---

#### Microsoft Graph

**Q16:** What is the base URL for Microsoft Graph API?
- A) `https://api.microsoft.com`
- B) `https://graph.microsoft.com`
- C) `https://microsoft.graph.com`
- D) `https://msgraph.microsoft.com`

**Answer: B) `https://graph.microsoft.com`**
Explanation: Microsoft Graph uses the base URL `https://graph.microsoft.com` followed by version and resource path.

---

**Q17:** Which OData query parameter is used to return only specific properties?
- A) $filter
- B) $select
- C) $expand
- D) $top

**Answer: B) $select**
Explanation: The `$select` parameter specifies which properties to include in the response.

---

**Q18:** What version of Microsoft Graph API should you use for production applications?
- A) v1.0
- B) beta
- C) v2.0
- D) latest

**Answer: A) v1.0**
Explanation: The v1.0 version is production-ready and stable. Beta is for preview features and may change.

---

**Q19:** How do you get the next page of results when Microsoft Graph returns paginated data?
- A) Increment the $skip parameter
- B) Use the @odata.nextLink URL
- C) Call the same endpoint again
- D) Use the $page parameter

**Answer: B) Use the @odata.nextLink URL**
Explanation: Microsoft Graph includes an @odata.nextLink property containing the URL to fetch the next page of results.

---

**Q20:** Which permission type is required for a daemon application that reads all users' calendars?
- A) Delegated: Calendars.Read
- B) Application: Calendars.Read
- C) Delegated: Calendars.Read.All
- D) Application: Calendars.Read

**Answer: D) Application: Calendars.Read**
Explanation: Daemon apps need application permissions (not delegated) since there's no user context. Calendars.Read with application type can read all users' calendars.

---

#### Advanced Scenarios

**Q21:** An API receives a user token and needs to call another API on behalf of the user. Which flow should it use?
- A) Client credentials flow
- B) Authorization code flow
- C) On-behalf-of flow
- D) Device code flow

**Answer: C) On-behalf-of flow**
Explanation: The On-Behalf-Of flow allows an API to exchange an incoming token for a new token to call downstream APIs while preserving user context.

---

**Q22:** Which builder method in MSAL is used to handle Conditional Access challenges?
- A) WithClaims()
- B) WithConditionalAccess()
- C) WithMfa()
- D) WithChallenge()

**Answer: A) WithClaims()**
Explanation: The WithClaims() method is used to pass claims from a Conditional Access challenge back to the identity provider.

---

**Q23:** What is the maximum validity period for a user delegation key used to sign SAS tokens?
- A) 1 hour
- B) 7 days
- C) 30 days
- D) 1 year

**Answer: B) 7 days**
Explanation: User delegation keys can have a maximum validity period of 7 days from the start time.

---

**Q24:** Which Microsoft Graph SDK class handles automatic pagination?
- A) GraphServiceClient
- B) PageIterator
- C) BatchRequestContent
- D) CollectionPage

**Answer: B) PageIterator**
Explanation: The PageIterator class automatically handles pagination, following @odata.nextLink until all results are retrieved.

---

**Q25:** When creating an application registration, which account type should you choose for a SaaS application used by multiple organizations?
- A) Single tenant
- B) Multitenant
- C) Personal Microsoft accounts only
- D) Multitenant and personal accounts

**Answer: B) Multitenant**
Explanation: Multitenant allows users from any Azure AD organization to sign in, which is appropriate for SaaS applications.

---

#### Managed Identities

**Q26:** You want to access Azure Key Vault from an App Service without managing credentials. What should you use?
- A) Client secret stored in app settings
- B) Connection string
- C) Managed identity with DefaultAzureCredential
- D) Storage account key

**Answer: C) Managed identity with DefaultAzureCredential**
Explanation: Managed identity eliminates the need to manage credentials. DefaultAzureCredential automatically uses the managed identity when running in Azure.

---

**Q27:** What is the key difference between system-assigned and user-assigned managed identities?
- A) System-assigned is more secure
- B) System-assigned is tied to the resource lifecycle; user-assigned is independent
- C) User-assigned cannot be used with App Service
- D) System-assigned supports multiple resources

**Answer: B) System-assigned is tied to the resource lifecycle; user-assigned is independent**
Explanation: System-assigned identity is deleted when the resource is deleted. User-assigned identity has an independent lifecycle and can be shared across resources.

---

**Q28:** Which credential does DefaultAzureCredential try FIRST?
- A) ManagedIdentityCredential
- B) AzureCliCredential
- C) EnvironmentCredential
- D) InteractiveBrowserCredential

**Answer: C) EnvironmentCredential**
Explanation: DefaultAzureCredential tries EnvironmentCredential first (checking AZURE_CLIENT_ID, AZURE_TENANT_ID, AZURE_CLIENT_SECRET env vars), then WorkloadIdentityCredential, then ManagedIdentityCredential.

---

**Q29:** You have 5 Azure Functions that all need to access the same storage account. Which managed identity type is most appropriate?
- A) System-assigned on each function
- B) User-assigned shared across all functions
- C) Client secret per function
- D) Storage account key

**Answer: B) User-assigned shared across all functions**
Explanation: User-assigned identity can be shared across resources. This is ideal when multiple resources need the same access, reducing role assignments to manage.

---

#### Azure Key Vault

**Q30:** What RBAC role allows reading secret VALUES from Key Vault?
- A) Key Vault Reader
- B) Key Vault Secrets User
- C) Key Vault Contributor
- D) Key Vault Administrator

**Answer: B) Key Vault Secrets User**
Explanation: Key Vault Reader only reads metadata (not values). Key Vault Secrets User can read actual secret contents.

---

**Q31:** What is the correct syntax for a Key Vault reference in App Service application settings?
- A) `${KeyVault:MySecret}`
- B) `@Microsoft.KeyVault(VaultName=myVault;SecretName=MySecret)`
- C) `keyvault://myVault/MySecret`
- D) `@KeyVault(myVault, MySecret)`

**Answer: B) `@Microsoft.KeyVault(VaultName=myVault;SecretName=MySecret)`**
Explanation: App Service uses the `@Microsoft.KeyVault(...)` syntax with VaultName and SecretName (or SecretUri) to reference Key Vault secrets.

---

**Q32:** Your Key Vault has purge protection enabled. An admin accidentally deletes a secret. Can they permanently purge it before the retention period?
- A) Yes, with Owner role
- B) Yes, with Key Vault Administrator role
- C) No, purge protection prevents this until retention period expires
- D) Yes, by disabling purge protection first

**Answer: C) No, purge protection prevents this until retention period expires**
Explanation: Purge protection prevents ANY permanent deletion during the retention period, even by admins. It cannot be disabled once enabled.

---

**Q33:** Which method correctly creates a SecretClient for Key Vault using managed identity?
- A) `new SecretClient("https://vault.vault.azure.net", new ClientSecretCredential(...))`
- B) `new SecretClient(new Uri("https://vault.vault.azure.net"), new DefaultAzureCredential())`
- C) `SecretClient.Create("https://vault.vault.azure.net")`
- D) `new SecretClient("connection-string")`

**Answer: B) `new SecretClient(new Uri("https://vault.vault.azure.net"), new DefaultAzureCredential())`**
Explanation: SecretClient takes a Uri and a TokenCredential. DefaultAzureCredential uses managed identity in Azure automatically.

---

#### App Service Easy Auth

**Q34:** When Easy Auth is configured to "Require authentication," what happens to unauthenticated API requests?
- A) They are redirected to the login page
- B) They return HTTP 401 Unauthorized
- C) They are silently allowed through
- D) They return HTTP 403 Forbidden

**Answer: B) They return HTTP 401 Unauthorized**
Explanation: For API requests, Easy Auth returns 401. For browser requests, it redirects to the login page. The behavior depends on the client type.

---

**Q35:** Which HTTP header does Easy Auth inject to provide the authenticated user's claims?
- A) `Authorization`
- B) `X-MS-CLIENT-PRINCIPAL`
- C) `X-Auth-Claims`
- D) `X-Azure-Identity`

**Answer: B) `X-MS-CLIENT-PRINCIPAL`**
Explanation: Easy Auth injects X-MS-CLIENT-PRINCIPAL (Base64-encoded claims), X-MS-CLIENT-PRINCIPAL-ID, and other X-MS-* headers.

---

**Q36:** In Easy Auth, what is the difference between server-directed and client-directed flow?
- A) Server-directed uses the SDK; client-directed doesn't
- B) Server-directed: App Service handles login; Client-directed: client app authenticates then posts token
- C) Server-directed is for APIs; client-directed is for web apps
- D) There is no difference

**Answer: B) Server-directed: App Service handles login; Client-directed: client app authenticates then posts token**
Explanation: Server-directed: App Service redirects to login page. Client-directed: client (e.g., mobile app) uses MSAL to get a token and posts it to /.auth/login/aad.

---

#### JWT & Tokens

**Q37:** In a JWT access token, which claim contains the delegated permissions?
- A) `roles`
- B) `scp`
- C) `permissions`
- D) `aud`

**Answer: B) `scp`**
Explanation: The `scp` (scope) claim contains delegated permissions. The `roles` claim contains application permissions/app roles.

---

**Q38:** What is the purpose of the `aud` (audience) claim in an access token?
- A) Identifies the user
- B) Identifies the API the token is intended for
- C) Identifies the issuing authority
- D) Contains the token expiration

**Answer: B) Identifies the API the token is intended for**
Explanation: The `aud` claim specifies which resource/API should accept this token. The API must validate that `aud` matches its own identifier.

---

**Q39:** When should you use the `.default` scope?
- A) Only for user-facing applications
- B) Only for mobile applications
- C) For client credentials (daemon) flow — always
- D) Only when you don't know which permissions are needed

**Answer: C) For client credentials (daemon) flow — always**
Explanation: Client credentials flow requires the `.default` scope (e.g., `https://graph.microsoft.com/.default`), which returns all consented application permissions.

---

**Q40:** What are the three parts of a JWT token?
- A) Header, Body, Footer
- B) Header, Payload, Signature
- C) Issuer, Claims, Key
- D) Type, Data, Hash

**Answer: B) Header, Payload, Signature**
Explanation: JWT = Base64(Header).Base64(Payload).Signature, separated by dots.

---

#### App Roles & Claims

**Q41:** Where are custom app roles defined in Azure AD?
- A) In the application code
- B) In the app registration manifest (appRoles property)
- C) In Azure RBAC
- D) In the Key Vault

**Answer: B) In the app registration manifest (appRoles property)**
Explanation: App roles are defined in the `appRoles` array in the application manifest, then assigned to users/groups in Enterprise Applications.

---

**Q42:** In ASP.NET Core, how do you restrict a controller action to users with the "Admin" app role?
- A) `[RequireRole("Admin")]`
- B) `[Authorize(Roles = "Admin")]`
- C) `[ClaimRequired("Admin")]`
- D) `[AdminOnly]`

**Answer: B) `[Authorize(Roles = "Admin")]`**
Explanation: The standard `[Authorize(Roles = "...")]` attribute checks the `roles` claim in the token, which is populated from app role assignments.

---

**Q43:** If a user belongs to more than 200 groups, what happens to the `groups` claim in the token?
- A) All groups are included
- B) Token is rejected
- C) A `_claim_names` property with a URL to fetch groups is included instead
- D) Only the first 200 groups are included

**Answer: C) A `_claim_names` property with a URL to fetch groups is included instead**
Explanation: When groups exceed the token size limit (~200), Azure AD includes an overage indicator with a Graph API URL to fetch the full group list.

---

#### Token Validation & API Protection

**Q44:** What 5 things must be validated in a JWT access token?
- A) Signature, issuer, audience, expiration, not-before
- B) Header, payload, signature, algorithm, key
- C) Username, password, roles, scopes, tenant
- D) Client ID, secret, redirect URI, scope, grant type

**Answer: A) Signature, issuer, audience, expiration, not-before**
Explanation: Token validation must check: (1) signature integrity, (2) issuer matches expected authority, (3) audience matches the API, (4) token hasn't expired, (5) token is already valid.

---

**Q45:** In a multi-tenant web API, how should you validate the issuer?
- A) Set `ValidateIssuer = false`
- B) Hardcode a single issuer
- C) Use custom IssuerValidator that checks tenant ID against allowed list
- D) Issuer validation is not needed for multi-tenant

**Answer: C) Use custom IssuerValidator that checks tenant ID against allowed list**
Explanation: For multi-tenant apps, each tenant has a different issuer. Use custom validation logic to check the `tid` claim against your allowed tenant list.

---

**Q46:** Which NuGet package provides `AddMicrosoftIdentityWebApiAuthentication` for protecting ASP.NET Core APIs?
- A) Microsoft.Identity.Client
- B) Microsoft.Identity.Web
- C) Azure.Identity
- D) Microsoft.AspNetCore.Authentication.JwtBearer

**Answer: B) Microsoft.Identity.Web**
Explanation: Microsoft.Identity.Web is the higher-level library that simplifies ASP.NET Core integration. Microsoft.Identity.Client is MSAL (lower level).

---

**Q47:** What attribute in ASP.NET Core checks that the access token contains a specific scope?
- A) `[Authorize(Scopes = "Data.Read")]`
- B) `[RequiredScope("Data.Read")]`
- C) `[Scope("Data.Read")]`
- D) `[Permission("Data.Read")]`

**Answer: B) `[RequiredScope("Data.Read")]`**
Explanation: The `[RequiredScope]` attribute from Microsoft.Identity.Web validates that the token's `scp` claim contains the specified scope.

---

#### Advanced Scenarios

**Q48:** What does Workload Identity Federation enable?
- A) Users to sign in with multiple identities
- B) External workloads (e.g., GitHub Actions) to access Azure without secrets
- C) Multiple Azure subscriptions to share one identity
- D) Passwordless authentication for users

**Answer: B) External workloads (e.g., GitHub Actions) to access Azure without secrets**
Explanation: Workload identity federation uses federated credentials, allowing external identity providers to exchange tokens without client secrets.

---

**Q49:** What are the three Zero Trust principles?
- A) Encrypt, Authenticate, Authorize
- B) Verify explicitly, Least privilege, Assume breach
- C) Confidentiality, Integrity, Availability
- D) Defend, Detect, Respond

**Answer: B) Verify explicitly, Least privilege, Assume breach**
Explanation: Zero Trust pillars: (1) Verify explicitly (always authenticate), (2) Use least privilege access, (3) Assume breach (minimize blast radius).

---

**Q50:** What is the maximum lifetime for a User Delegation Key?
- A) 1 hour
- B) 24 hours
- C) 7 days
- D) 30 days

**Answer: C) 7 days**
Explanation: User delegation keys have a maximum validity of 7 days from the start time.

---

**Q51:** Which environment variable does EnvironmentCredential look for to find the client secret?
- A) `AZURE_SECRET`
- B) `AZURE_CLIENT_SECRET`
- C) `CLIENT_SECRET`
- D) `AAD_CLIENT_SECRET`

**Answer: B) `AZURE_CLIENT_SECRET`**
Explanation: EnvironmentCredential looks for AZURE_CLIENT_ID, AZURE_TENANT_ID, and AZURE_CLIENT_SECRET (or AZURE_CLIENT_CERTIFICATE_PATH).

---

**Q52:** What is the difference between Azure AD B2B and Azure AD B2C?
- A) B2B is for consumers; B2C is for businesses
- B) B2B is for partner/guest access; B2C is for consumer-facing apps
- C) B2B uses social logins; B2C uses organizational accounts
- D) There is no difference

**Answer: B) B2B is for partner/guest access; B2C is for consumer-facing apps**
Explanation: B2B enables guest users from other organizations. B2C is a separate identity solution for customer-facing apps with social login support.

---

**Q53:** Which credential type is recommended for production confidential client applications?
- A) Client secret
- B) Certificate
- C) Managed identity
- D) Username and password

**Answer: C) Managed identity**
Explanation: Managed identity is the most secure — no credentials to manage at all. If managed identity isn't available, certificate is preferred over client secret.

---

**Q54:** You're configuring an App Service to read a secret from Key Vault without code changes. What two things must you configure?
- A) Connection string and firewall rule
- B) Managed identity on App Service + Key Vault access for that identity + Key Vault reference in app settings
- C) SAS token and API key
- D) VNet integration and private endpoint

**Answer: B) Managed identity on App Service + Key Vault access for that identity + Key Vault reference in app settings**
Explanation: Enable managed identity, grant it Key Vault access (Secrets User role or Get secret policy), then use `@Microsoft.KeyVault(...)` syntax in app settings.

---

**Q55:** What does the `accessTokenAcceptedVersion` property in the app manifest control?
- A) The API version of Microsoft Graph
- B) The JWT token version (v1.0 or v2.0) the API accepts
- C) The MSAL library version
- D) The TLS version

**Answer: B) The JWT token version (v1.0 or v2.0) the API accepts**
Explanation: Setting `accessTokenAcceptedVersion` to 2 means the API expects v2.0 tokens. This affects the token format, claims, and issuer URL.

---

#### RBAC, Control Plane, Tenant

**Q56:** A user has the Contributor role at the subscription level. Can they assign RBAC roles to other users?
- A) Yes, Contributor has full access
- B) No, only Owner and User Access Administrator can assign roles
- C) Yes, but only at resource group scope
- D) Yes, for data plane roles only

**Answer: B) No, only Owner and User Access Administrator can assign roles**
Explanation: Contributor can create/delete/manage resources but CANNOT manage role assignments. That requires Owner or User Access Administrator.

---

**Q57:** You assign "Storage Blob Data Reader" to a managed identity at the resource group scope. The resource group has 3 storage accounts. What access does the identity have?
- A) Read access to blobs in the first storage account only
- B) Read access to blobs in ALL 3 storage accounts
- C) No access — roles must be assigned at resource scope
- D) Read access to the resource group metadata only

**Answer: B) Read access to blobs in ALL 3 storage accounts**
Explanation: RBAC roles inherit downward. A role at resource group scope applies to all resources in that resource group.

---

**Q58:** A developer has the Contributor role on a Key Vault. Can they read the secret values stored in it?
- A) Yes, Contributor has full access
- B) No, Contributor is a control plane role; reading secrets requires a data plane role
- C) Yes, but only through the Azure portal
- D) Yes, if they also have Reader role

**Answer: B) No, Contributor is a control plane role; reading secrets requires a data plane role**
Explanation: Contributor manages the Key Vault resource (create, delete, configure) but cannot access data inside it. Reading secrets requires Key Vault Secrets User (data plane role).

---

**Q59:** What are the three components of an Azure RBAC role assignment?
- A) User, Password, Resource
- B) Security Principal, Role Definition, Scope
- C) Subscription, Resource Group, Resource
- D) Application, Permission, Consent

**Answer: B) Security Principal, Role Definition, Scope**
Explanation: A role assignment combines WHO (security principal), WHAT (role definition/permissions), and WHERE (scope — subscription, resource group, or resource).

---

**Q60:** How does a managed identity running on an Azure VM obtain an access token?
- A) By reading credentials from environment variables
- B) By calling the Instance Metadata Service (IMDS) at 169.254.169.254
- C) By using a stored client secret
- D) By prompting the user to sign in

**Answer: B) By calling the Instance Metadata Service (IMDS) at 169.254.169.254**
Explanation: Azure VMs use the IMDS endpoint (169.254.169.254) to request tokens. This endpoint is only accessible from within the VM. The Azure SDK handles this automatically via ManagedIdentityCredential.

---

## EXAM GOTCHAS & TRICKY DISTINCTIONS

### Common Exam Traps

| Trap | Truth |
|------|-------|
| "DefaultAzureCredential tries managed identity first" | ❌ EnvironmentCredential is tried first |
| "Key Vault Reader can read secret values" | ❌ Reader only sees metadata; need Secrets User |
| "Client secrets are recommended for production" | ❌ Managed identity > Certificate > Secret |
| "Implicit flow is recommended for SPAs" | ❌ Use Auth Code with PKCE instead |
| "Easy Auth requires code changes" | ❌ It runs as middleware, no code needed |
| "Application permissions use the scp claim" | ❌ App permissions use `roles`; delegated use `scp` |
| "Managed identity requires storing credentials" | ❌ No credentials to manage, Azure handles it |
| "User delegation SAS works with all storage services" | ❌ User delegation SAS is Blob only |
| "Purge protection can be disabled" | ❌ Once enabled, it cannot be disabled |
| ".default scope returns specific permissions" | ❌ It returns ALL consented permissions |
| "AcquireTokenInteractive should be called first" | ❌ Always try AcquireTokenSilent first |
| "v1.0 and v2.0 tokens have same claims" | ❌ Different: v2.0 uses scp, v1.0 uses different format |
| "Contributor can assign roles" | ❌ Only Owner and User Access Administrator can assign roles |
| "Contributor on Key Vault = can read secrets" | ❌ Contributor is control plane; secrets need data plane role |
| "RBAC roles only apply at the exact scope" | ❌ Roles INHERIT downward (subscription → RG → resource) |
| "Managed identity calls Azure AD directly" | ❌ It calls IMDS (169.254.169.254) which handles auth |

### Key Numbers to Remember

| Item | Value |
|------|-------|
| Max stored access policies per container | 5 |
| User delegation key max lifetime | 7 days |
| Access token default lifetime | ~60-90 minutes |
| Refresh token max lifetime | Up to 90 days |
| Client secret max recommended lifetime | < 6 months (max 2 years) |
| DefaultAzureCredential chain | 10 credential types in order |
| Key Vault soft delete retention | 7-90 days (default 90) |
| Groups overage limit in token | ~200 groups |
| Max app roles in manifest | ~1200 |
| Max CORS rules per storage service | 5 |

### Authentication Method Selection Guide

| Scenario | Use |
|----------|-----|
| App in Azure needs to access Azure resources | **Managed Identity** |
| Local development needs to access Azure | **DefaultAzureCredential** (uses az login) |
| Daemon/background service | **Client Credentials** (cert or managed identity) |
| Web app signing in users | **Authorization Code** (Microsoft.Identity.Web) |
| SPA in browser | **Auth Code + PKCE** |
| Mobile/desktop app | **Auth Code + PKCE** |
| IoT/CLI tool | **Device Code** |
| API calling another API | **On-Behalf-Of** |
| GitHub Actions → Azure | **Workload Identity Federation** |

### Claims Cheat Sheet

| What You Need | Check This Claim |
|---------------|-----------------|
| Who is the user? | `oid`, `sub`, `preferred_username` |
| Which tenant? | `tid` |
| Which app requested? | `azp` (v2.0) or `appid` (v1.0) |
| What can user do? (delegated) | `scp` |
| What can app do? (application) | `roles` |
| When does it expire? | `exp` |
| Who issued it? | `iss` |
| Who should accept it? | `aud` |

---

## Key CLI Commands Reference

### App Registration
```bash
# Create app registration
az ad app create \
  --display-name "My App" \
  --sign-in-audience AzureADMyOrg \
  --web-redirect-uris "https://myapp.com/callback"

# Create service principal
az ad sp create --id <app-id>

# Create client secret
az ad app credential reset --id <app-id> --append

# Add API permission
az ad app permission add \
  --id <app-id> \
  --api 00000003-0000-0000-c000-000000000000 \
  --api-permissions e1fe6dd8-ba31-4d61-89e7-88639da4683d=Scope

# Grant admin consent
az ad app permission admin-consent --id <app-id>
```

### Storage SAS
```bash
# Generate account SAS
az storage account generate-sas \
  --account-name myaccount \
  --services b \
  --resource-types sco \
  --permissions rwdlacup \
  --expiry 2024-12-31

# Generate blob SAS
az storage blob generate-sas \
  --account-name myaccount \
  --container-name mycontainer \
  --name myblob \
  --permissions r \
  --expiry 2024-12-31

# Create stored access policy
az storage container policy create \
  --container-name mycontainer \
  --account-name myaccount \
  --name mypolicy \
  --permissions rl \
  --expiry 2024-12-31
```

---

## Study Tips for AZ-204 Exam

1. **OAuth 2.0 flows** — Know which flow for which scenario (Auth Code, PKCE, Client Creds, Device Code, OBO)
2. **MSAL patterns** — Always Silent first, then Interactive; know exception types
3. **Managed Identities** — System vs User-assigned; DefaultAzureCredential chain order
4. **Key Vault** — Secrets/Keys/Certs; Access Policy vs RBAC; SDK clients; Key Vault References
5. **Easy Auth** — Server vs Client-directed; X-MS-* headers; runs before code
6. **JWT tokens** — Structure (header.payload.signature); key claims (aud, iss, scp, roles, oid, tid)
7. **App Roles** — Defined in manifest; appear in `roles` claim; [Authorize(Roles = "...")]
8. **Permissions** — Delegated (`scp` claim) vs Application (`roles` claim); consent types
9. **SAS types** — User Delegation > Service > Account; stored access policies (max 5)
10. **Graph API** — Base URL, OData params, SDK, pagination, batch requests
11. **Token validation** — Validate issuer, audience, signature, expiration; multi-tenant considerations
12. **Security hierarchy** — Managed Identity > Certificate > Client Secret > Account Key
13. **Authority URLs** — common vs organizations vs consumers vs tenant-specific
14. **Zero Trust** — Verify explicitly, Least privilege, Assume breach
15. **Workload Identity Federation** — Passwordless for GitHub Actions, external workloads

---

## Important URLs and References

- Microsoft Identity Platform: https://learn.microsoft.com/en-us/entra/identity-platform/
- MSAL Documentation: https://learn.microsoft.com/en-us/entra/msal/overview
- Managed Identities: https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/
- Azure Key Vault: https://learn.microsoft.com/en-us/azure/key-vault/general/
- App Service Authentication: https://learn.microsoft.com/en-us/azure/app-service/overview-authentication-authorization
- Microsoft.Identity.Web: https://learn.microsoft.com/en-us/entra/msal/dotnet/microsoft-identity-web/
- Azure.Identity: https://learn.microsoft.com/en-us/dotnet/api/overview/azure/identity-readme
- Shared Access Signatures: https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview
- Microsoft Graph: https://learn.microsoft.com/en-us/graph/overview
- Graph Explorer: https://developer.microsoft.com/graph/graph-explorer
- Zero Trust: https://learn.microsoft.com/en-us/security/zero-trust/
- App Roles: https://learn.microsoft.com/en-us/entra/identity-platform/howto-add-app-roles-in-apps

---

## Quick Reference Summary

### Azure AD Fundamentals
- **Azure AD** = Cloud identity service (now Microsoft Entra ID)
- **Tenant** = Dedicated Azure AD instance for an organization
- **Editions**: Free, P1 (Conditional Access), P2 (PIM, Identity Protection)
- **Objects**: Users, Groups (Security/M365/Dynamic), Enterprise Apps, Service Principals
- **SSO**: One login for multiple apps
- **MFA**: Knowledge + Possession + Biometric factors

### Azure AD Roles vs Azure RBAC
- **Azure AD Roles** → Manage identity objects (users, groups, apps) — e.g., Global Admin
- **Azure RBAC** → Manage Azure resources (VMs, storage) — e.g., Contributor, Owner
- **Two separate systems!** Global Admin ≠ automatic resource access
- **Deny assignments** override allow; implicit deny by default

### Azure RBAC Quick Reference
- **Role Assignment** = Principal + Role Definition + Scope
- **Scope hierarchy** (inherits DOWN): Management Group → Subscription → Resource Group → Resource
- **Actions** = control plane; **DataActions** = data plane
- **Owner** = full + assign roles; **Contributor** = full - assign roles; **Reader** = read only
- **User Access Administrator** = manage role assignments only
- **Custom roles**: Up to 5000 per tenant; NotActions ≠ deny, it's subtraction

### Protocols
- **OAuth 2.0** = Authorization (Access Tokens) — "what can you do?"
- **OpenID Connect** = Authentication on OAuth (ID Tokens) — "who are you?"
- **SAML** = XML-based enterprise SSO (older)
- Always use **v2.0 endpoints** with MSAL; ADAL is deprecated

### Supported Account Types
- **Single tenant** (`AzureADMyOrg`) → authority uses tenant ID
- **Multi-tenant** (`AzureADMultipleOrgs`) → authority uses `organizations`
- **Multi-tenant + Personal** → authority uses `common`
- **Personal only** → authority uses `consumers`

### Authentication Flows
- **Authorization Code**: Web apps with backend
- **Auth Code + PKCE**: SPAs, mobile, desktop
- **Client Credentials**: Daemon services (uses `.default` scope)
- **Device Code**: Input-constrained devices (IoT, CLI)
- **On-Behalf-Of**: API calling API (preserves user context)

### MSAL Client Types
- **Public Client**: Desktop, mobile, SPA (cannot store secrets)
- **Confidential Client**: Web app, API, daemon (can store secrets)

### Managed Identities
- **System-assigned**: Tied to resource lifecycle, not shareable
- **User-assigned**: Independent lifecycle, shareable across resources
- **DefaultAzureCredential**: Works everywhere (Env → Managed Identity → CLI → VS)
- Token via **IMDS** (`169.254.169.254`) on VMs, environment vars on App Service

### Azure Key Vault
- **Secrets**: Connection strings, API keys (SecretClient)
- **Keys**: Cryptographic keys (KeyClient)
- **Certificates**: X.509 certs (CertificateClient)
- **Access**: RBAC (recommended) or Access Policies (legacy)
- **App Service**: `@Microsoft.KeyVault(VaultName=x;SecretName=y)`

### Easy Auth (App Service)
- Middleware — runs before your code, no code changes needed
- Server-directed (redirects) vs Client-directed (post token)
- Headers: X-MS-CLIENT-PRINCIPAL, X-MS-CLIENT-PRINCIPAL-ID

### Token Types
- **Access Token**: Authorizes API calls (~1 hour)
- **ID Token**: Proves identity (client-side)
- **Refresh Token**: Gets new tokens (up to 90 days)
- Bearer token sent in `Authorization: Bearer <token>` HTTP header

### JWT Claims
- `aud` = audience (API), `iss` = issuer, `exp` = expiry
- `scp` = delegated permissions, `roles` = app permissions
- `oid` = user object ID, `tid` = tenant ID

### SAS Types (Security Ranking)
- **User Delegation**: Most secure, Azure AD, Blob only
- **Service**: Single service, account key, stored access policies
- **Account**: Multiple services, account key, broadest access

### Credential Security Ranking
1. **Managed Identity** — No credentials at all
2. **Certificate** — Private key stays local
3. **Client Secret** — String that can be copied
4. **Account Key** — Full access, last resort

### Permissions
- **Delegated**: User context, `scp` claim, user or admin consent
- **Application**: No user, `roles` claim, admin consent only

### Graph API
- Base: `https://graph.microsoft.com/v1.0/`
- Params: `$select`, `$filter`, `$orderby`, `$top`, `$expand`
- Pagination: `@odata.nextLink` or `PageIterator`
- SDK: `GraphServiceClient` with credential

### App Registration Essentials
- **Client ID** + **Tenant ID** = identify the app
- **Redirect URI** = where tokens are sent back
- **Client Secret** (expires, weaker) vs **Certificate** (stronger, recommended for production)
- **Expose an API** = define scopes your API offers
- **API Permissions** = what APIs your app calls

### Security Features by Azure AD Edition
- **Free**: Users, groups, SSO (10 apps), basic MFA via Security Defaults
- **P1**: Conditional Access, dynamic groups, Azure AD App Proxy, unlimited SSO, custom RBAC/AD roles
- **P2**: PIM (just-in-time access), Identity Protection (risk-based), access reviews

