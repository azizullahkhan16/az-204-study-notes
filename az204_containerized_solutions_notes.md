# AZ-204: Implement Containerized Solutions - Complete Study Notes

## Learning Path Overview
This learning path covers implementing containerized solutions in Azure with 3 modules:
1. Manage container images in Azure Container Registry
2. Run container images in Azure Container Instances
3. Implement Azure Container Apps

**Duration:** ~1.5 hours | **Level:** Intermediate

---

## MODULE 1: AZURE CONTAINER REGISTRY (ACR)

### 1.1 Introduction to Azure Container Registry

**What is Azure Container Registry?**
- Managed, private Docker registry service
- Based on open-source Docker Registry 2.0
- Store and manage container images and artifacts
- Works with existing container development and deployment pipelines
- Supports Docker and OCI (Open Container Initiative) images

**Key Use Cases:**
- Store container images for deployment to:
  - Azure Container Instances (ACI)
  - Azure Kubernetes Service (AKS)
  - Azure App Service
  - Azure Container Apps
  - Azure Functions
- Build container images in Azure
- Automate image builds on source code commit or base image update
- Geo-replicate registry for global distribution

**Key Features:**
- Private registry for container images
- Automated container builds (ACR Tasks)
- Geo-replication for global availability
- Content trust (image signing)
- Integrated security with Azure AD
- Webhooks for automation
- VNet integration and private endpoints

### 1.2 Azure Container Registry Service Tiers

**ACR Service Tiers:**

| Feature | Basic | Standard | Premium |
|---------|-------|----------|---------|
| Storage | 10 GB | 100 GB | 500 GB |
| Read ops/min | 1,000 | 3,000 | 10,000 |
| Write ops/min | 100 | 500 | 2,000 |
| Download bandwidth | 30 MBps | 60 MBps | 100 MBps |
| Upload bandwidth | 10 MBps | 20 MBps | 50 MBps |
| Webhooks | 2 | 10 | 500 |
| Geo-replication | ❌ | ❌ | ✅ |
| Content Trust | ❌ | ❌ | ✅ |
| Private Link | ❌ | ❌ | ✅ |
| Customer-managed keys | ❌ | ❌ | ✅ |
| Repository-scoped tokens | ❌ | ❌ | ✅ |
| Dedicated agent pools | ❌ | ❌ | ✅ |
| Availability zones | ❌ | ❌ | ✅ |

**Tier Selection Guidelines:**
- **Basic**: Development and testing, low usage
- **Standard**: Production workloads, most scenarios
- **Premium**: High-scale production, geo-replication, enhanced security

### 1.3 ACR Storage Concepts

**Storage Features:**
- **Encryption at rest**: All images encrypted automatically
- **Regional storage**: Stored in region of registry
- **Zone redundancy**: Premium tier with availability zones
- **Scalable storage**: Grows automatically within tier limits

**Supported Content:**
- Docker-compatible container images
- OCI artifacts (Helm charts, Singularity, etc.)
- OCI image format
- Multi-architecture images (manifests)

**Image Naming:**
```
<registry-name>.azurecr.io/<repository>:<tag>
<registry-name>.azurecr.io/<repository>@<digest>

# Examples
myregistry.azurecr.io/myapp:v1.0
myregistry.azurecr.io/samples/myapp:latest
myregistry.azurecr.io/myapp@sha256:abc123...
```

**Repository Structure:**
```
Registry: myregistry.azurecr.io
    └── Repository: myapp
            └── Tags: v1.0, v1.1, latest
            └── Manifests (by digest)
    └── Repository: samples/webapp
            └── Tags: dev, staging, prod
```

### 1.4 Creating and Managing ACR

**Create Registry - Azure CLI:**

```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# Create container registry
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Standard

# Create with admin user enabled
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Premium \
  --admin-enabled true

# Create with zone redundancy (Premium only)
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Premium \
  --zone-redundancy enabled
```

**Create Registry - Azure Portal:**
```
Azure Portal → Create a resource → Container Registry
- Basics: Name, Resource Group, Location, SKU
- Networking: Public/Private endpoint
- Encryption: Microsoft-managed or customer-managed keys
- Tags: Optional tags
```

**Manage Registry:**

```bash
# List registries
az acr list --resource-group myResourceGroup --output table

# Show registry details
az acr show --name myregistry --resource-group myResourceGroup

# Update registry SKU
az acr update --name myregistry --sku Premium

# Enable admin user
az acr update --name myregistry --admin-enabled true

# Delete registry
az acr delete --name myregistry --resource-group myResourceGroup
```

### 1.5 ACR Authentication

**Authentication Options:**

#### **1. Azure AD Authentication (Recommended)**
- Individual identities (users, service principals)
- Managed identities for Azure resources
- Role-based access control (RBAC)

**Built-in Roles:**

| Role | Description |
|------|-------------|
| Owner | Full access including role assignments |
| Contributor | Push/pull/delete images, no role assignments |
| Reader | Pull images only |
| AcrPush | Push images |
| AcrPull | Pull images |
| AcrDelete | Delete images |
| AcrImageSigner | Sign images (content trust) |

```bash
# Login with Azure AD
az acr login --name myregistry

# Assign role to service principal
az role assignment create \
  --assignee <service-principal-id> \
  --role AcrPull \
  --scope /subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.ContainerRegistry/registries/<registry>

# Login with service principal
docker login myregistry.azurecr.io \
  --username <service-principal-id> \
  --password <service-principal-password>
```

#### **2. Admin Account**
- Single user per registry
- Disabled by default
- Not recommended for production
- Useful for development/testing

```bash
# Enable admin account
az acr update --name myregistry --admin-enabled true

# Get admin credentials
az acr credential show --name myregistry

# Login with admin
docker login myregistry.azurecr.io \
  --username myregistry \
  --password <admin-password>
```

#### **3. Repository-Scoped Tokens (Premium)**
- Fine-grained access to specific repositories
- Token-based authentication
- Limited to Premium tier

```bash
# Create token
az acr token create \
  --name mytoken \
  --registry myregistry \
  --scope-map _repositories_pull

# Get token password
az acr token credential generate \
  --name mytoken \
  --registry myregistry
```

### 1.6 Working with Container Images

**Push Images to ACR:**

```bash
# Login to ACR
az acr login --name myregistry

# Tag local image for ACR
docker tag myapp:v1 myregistry.azurecr.io/myapp:v1

# Push image to ACR
docker push myregistry.azurecr.io/myapp:v1

# Push all tags
docker push myregistry.azurecr.io/myapp --all-tags
```

**Pull Images from ACR:**

```bash
# Login to ACR
az acr login --name myregistry

# Pull image
docker pull myregistry.azurecr.io/myapp:v1

# Pull by digest
docker pull myregistry.azurecr.io/myapp@sha256:abc123...
```

**List and Manage Images:**

```bash
# List repositories
az acr repository list --name myregistry --output table

# List tags in repository
az acr repository show-tags --name myregistry --repository myapp

# Show image manifest
az acr repository show-manifests --name myregistry --repository myapp

# Delete image by tag
az acr repository delete --name myregistry --image myapp:v1

# Delete image by digest
az acr repository delete --name myregistry --image myapp@sha256:abc123...

# Delete entire repository
az acr repository delete --name myregistry --repository myapp
```

### 1.7 ACR Tasks

**What are ACR Tasks?**
- Cloud-based container image building
- Automated builds on source code commit
- Automated builds on base image update
- Multi-step tasks for complex workflows
- No local Docker installation needed

**Task Types:**

#### **1. Quick Tasks**
- On-demand build in the cloud
- Similar to `docker build` but in Azure

```bash
# Build image from current directory
az acr build \
  --registry myregistry \
  --image myapp:v1 .

# Build from Git repository
az acr build \
  --registry myregistry \
  --image myapp:v1 \
  https://github.com/myuser/myrepo.git

# Build with specific Dockerfile
az acr build \
  --registry myregistry \
  --image myapp:v1 \
  --file ./docker/Dockerfile .

# Build with build arguments
az acr build \
  --registry myregistry \
  --image myapp:v1 \
  --build-arg VERSION=1.0 .
```

#### **2. Automatically Triggered Tasks**
- Trigger on source code commit
- Trigger on base image update
- Scheduled triggers (cron)

```bash
# Create task triggered by Git commit
az acr task create \
  --registry myregistry \
  --name buildtask \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/myuser/myrepo.git \
  --branch main \
  --file Dockerfile \
  --git-access-token <PAT>

# Create task triggered by base image update
az acr task create \
  --registry myregistry \
  --name baseupdatetask \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/myuser/myrepo.git \
  --branch main \
  --file Dockerfile \
  --base-image-trigger-enabled true \
  --git-access-token <PAT>

# Create scheduled task (daily at midnight)
az acr task create \
  --registry myregistry \
  --name scheduledtask \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/myuser/myrepo.git \
  --branch main \
  --file Dockerfile \
  --schedule "0 0 * * *" \
  --git-access-token <PAT>
```

#### **3. Multi-Step Tasks**
- Complex workflows with multiple steps
- Defined in YAML file
- Build, test, push multiple images

```yaml
# acr-task.yaml
version: v1.1.0
steps:
  # Build the application image
  - build: -t {{.Run.Registry}}/myapp:{{.Run.ID}} -f Dockerfile .
  
  # Run tests
  - cmd: {{.Run.Registry}}/myapp:{{.Run.ID}}
    args:
      - npm
      - test
  
  # Push if tests pass
  - push:
      - {{.Run.Registry}}/myapp:{{.Run.ID}}
      - {{.Run.Registry}}/myapp:latest
```

```bash
# Run multi-step task
az acr run \
  --registry myregistry \
  --file acr-task.yaml \
  https://github.com/myuser/myrepo.git

# Create multi-step task
az acr task create \
  --registry myregistry \
  --name multisteptask \
  --context https://github.com/myuser/myrepo.git \
  --file acr-task.yaml \
  --git-access-token <PAT>
```

**Manage Tasks:**

```bash
# List tasks
az acr task list --registry myregistry --output table

# Show task details
az acr task show --registry myregistry --name buildtask

# Run task manually
az acr task run --registry myregistry --name buildtask

# View task runs (logs)
az acr task logs --registry myregistry --name buildtask

# List task runs
az acr task list-runs --registry myregistry --output table

# Delete task
az acr task delete --registry myregistry --name buildtask
```

### 1.8 Geo-Replication (Premium)

**What is Geo-Replication?**
- Replicate registry across multiple Azure regions
- Pull images from nearest region
- Single management endpoint
- Automatic sync between replicas

**Benefits:**
- Network-close registry access
- Lower latency for image pulls
- Disaster recovery
- Global availability

```bash
# Add replication
az acr replication create \
  --registry myregistry \
  --location westus

# List replications
az acr replication list --registry myregistry --output table

# Delete replication
az acr replication delete --registry myregistry --location westus
```

### 1.9 ACR Security Features

**Content Trust (Premium):**
- Sign images to verify publisher
- Verify image integrity
- Based on Docker Content Trust

```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1
export DOCKER_CONTENT_TRUST_SERVER=https://myregistry.azurecr.io

# Push signed image
docker push myregistry.azurecr.io/myapp:v1
```

**Private Link (Premium):**
- Access registry over private endpoint
- No public internet exposure
- VNet integration

**Firewall Rules:**
- Restrict access by IP address
- Allow/deny specific IP ranges

```bash
# Add network rule
az acr network-rule add \
  --name myregistry \
  --ip-address 203.0.113.0/24

# Remove network rule
az acr network-rule remove \
  --name myregistry \
  --ip-address 203.0.113.0/24
```

---

## MODULE 2: AZURE CONTAINER INSTANCES (ACI)

### 2.1 Introduction to Azure Container Instances

**What is Azure Container Instances?**
- Fastest and simplest way to run containers in Azure
- No VM management required
- No orchestration needed
- Serverless containers
- Per-second billing

**Key Features:**
- Fast startup times (seconds)
- Custom sizes (CPU and memory)
- Public IP connectivity
- Persistent storage (Azure Files)
- Linux and Windows containers
- Container groups (pod-like)
- Virtual network deployment
- GPU support

**Use Cases:**
- Simple applications
- Task automation
- Build jobs
- Event-driven applications
- Batch processing
- Dev/test environments
- CI/CD agents

**When to Use ACI vs Other Services:**

| Use Case | Best Service |
|----------|-------------|
| Simple container, no orchestration | ACI |
| Microservices with orchestration | AKS or Container Apps |
| Event-driven, auto-scale | Container Apps |
| Full Kubernetes | AKS |
| Web apps with PaaS features | App Service |

### 2.2 Container Groups

**What is a Container Group?**
- Collection of containers scheduled on same host
- Share lifecycle, resources, network, storage
- Similar to Kubernetes pod
- Multi-container support

**Container Group Characteristics:**
- Share public IP address
- Share localhost network
- Share storage volumes
- Scheduled together
- Deployed together

**Container Group Deployment Methods:**
- ARM templates (JSON)
- YAML file (recommended)
- Azure CLI
- Azure Portal

**Example Container Group (YAML):**

```yaml
# container-group.yaml
apiVersion: '2021-10-01'
location: eastus
name: mycontainergroup
properties:
  containers:
  - name: myapp
    properties:
      image: myregistry.azurecr.io/myapp:v1
      ports:
      - port: 80
        protocol: TCP
      resources:
        requests:
          cpu: 1.0
          memoryInGb: 1.5
      environmentVariables:
      - name: ENV_VAR
        value: 'myvalue'
      - name: SECRET_VAR
        secureValue: 'mysecret'
  - name: sidecar
    properties:
      image: nginx:latest
      ports:
      - port: 8080
        protocol: TCP
      resources:
        requests:
          cpu: 0.5
          memoryInGb: 0.5
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - port: 80
      protocol: TCP
    - port: 8080
      protocol: TCP
  imageRegistryCredentials:
  - server: myregistry.azurecr.io
    username: myregistry
    password: <password>
tags:
  environment: production
type: Microsoft.ContainerInstance/containerGroups
```

```bash
# Deploy container group from YAML
az container create \
  --resource-group myResourceGroup \
  --file container-group.yaml
```

### 2.3 Creating Container Instances

**Create Container - Azure CLI:**

```bash
# Create simple container
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --ports 80 \
  --dns-name-label myapp-demo

# Create with specific resources
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myregistry.azurecr.io/myapp:v1 \
  --cpu 2 \
  --memory 4 \
  --ports 80 443 \
  --dns-name-label myapp

# Create from ACR with managed identity
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myregistry.azurecr.io/myapp:v1 \
  --acr-identity [system] \
  --assign-identity

# Create from ACR with service principal
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-login-server myregistry.azurecr.io \
  --registry-username <sp-id> \
  --registry-password <sp-password>
```

**Container Configuration Options:**

| Option | Description |
|--------|-------------|
| --cpu | Number of CPU cores (default: 1) |
| --memory | Memory in GB (default: 1.5) |
| --ports | Exposed ports |
| --dns-name-label | DNS label for public IP |
| --restart-policy | Always, OnFailure, Never |
| --environment-variables | Environment variables |
| --secure-environment-variables | Secret env vars |
| --command-line | Override container entrypoint |
| --azure-file-volume-* | Mount Azure Files |

### 2.4 Restart Policies

**Available Restart Policies:**

| Policy | Description | Use Case |
|--------|-------------|----------|
| Always (default) | Always restart on exit | Long-running services |
| OnFailure | Restart only on failure (non-zero exit) | Batch jobs with retry |
| Never | Never restart | One-time tasks |

```bash
# Create with restart policy
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myapp:v1 \
  --restart-policy OnFailure

# Create one-time task
az container create \
  --resource-group myResourceGroup \
  --name mybatchjob \
  --image mybatch:v1 \
  --restart-policy Never
```

**Restart Policy Behavior:**

```
Always:
  Container exits → Restart immediately → Repeat indefinitely

OnFailure:
  Exit code 0 → Stop (success)
  Exit code non-zero → Restart → Repeat until success or manual stop

Never:
  Container exits → Stop (regardless of exit code)
```

### 2.5 Environment Variables

**Setting Environment Variables:**

```bash
# Plain environment variables
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myapp:v1 \
  --environment-variables \
    'DATABASE_HOST=myserver.database.windows.net' \
    'APP_ENV=production'

# Secure environment variables (not shown in logs/portal)
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myapp:v1 \
  --secure-environment-variables \
    'DATABASE_PASSWORD=MySecretPassword123' \
    'API_KEY=secret-api-key'
```

**YAML Environment Variables:**

```yaml
apiVersion: '2021-10-01'
name: mycontainer
properties:
  containers:
  - name: myapp
    properties:
      image: myapp:v1
      environmentVariables:
      - name: DATABASE_HOST
        value: 'myserver.database.windows.net'
      - name: APP_ENV
        value: 'production'
      - name: DATABASE_PASSWORD
        secureValue: 'MySecretPassword123'
      - name: API_KEY
        secureValue: 'secret-api-key'
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
  osType: Linux
```

### 2.6 Mounting Volumes

**Supported Volume Types:**
- Azure Files (SMB file share)
- Empty directory
- Git repository
- Secret volumes

**Mount Azure Files:**

```bash
# Create storage account and file share
az storage account create \
  --resource-group myResourceGroup \
  --name mystorageaccount \
  --sku Standard_LRS

az storage share create \
  --account-name mystorageaccount \
  --name myshare

# Get storage account key
STORAGE_KEY=$(az storage account keys list \
  --resource-group myResourceGroup \
  --account-name mystorageaccount \
  --query '[0].value' \
  --output tsv)

# Create container with Azure Files volume
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myapp:v1 \
  --azure-file-volume-account-name mystorageaccount \
  --azure-file-volume-account-key $STORAGE_KEY \
  --azure-file-volume-share-name myshare \
  --azure-file-volume-mount-path /mnt/data
```

**YAML Volume Mount:**

```yaml
apiVersion: '2021-10-01'
name: mycontainer
properties:
  containers:
  - name: myapp
    properties:
      image: myapp:v1
      volumeMounts:
      - mountPath: /mnt/data
        name: datavolume
      - mountPath: /mnt/secrets
        name: secretvolume
        readOnly: true
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
  volumes:
  - name: datavolume
    azureFile:
      shareName: myshare
      storageAccountName: mystorageaccount
      storageAccountKey: <storage-key>
  - name: secretvolume
    secret:
      mysecret: <base64-encoded-value>
  osType: Linux
```

### 2.7 Container Operations

**Manage Containers:**

```bash
# List containers
az container list --resource-group myResourceGroup --output table

# Show container details
az container show \
  --resource-group myResourceGroup \
  --name mycontainer

# Get container logs
az container logs \
  --resource-group myResourceGroup \
  --name mycontainer

# Get logs from specific container in group
az container logs \
  --resource-group myResourceGroup \
  --name mycontainergroup \
  --container-name myapp

# Stream logs (follow)
az container logs \
  --resource-group myResourceGroup \
  --name mycontainer \
  --follow

# Execute command in container
az container exec \
  --resource-group myResourceGroup \
  --name mycontainer \
  --exec-command "/bin/sh"

# Start stopped container
az container start \
  --resource-group myResourceGroup \
  --name mycontainer

# Stop container
az container stop \
  --resource-group myResourceGroup \
  --name mycontainer

# Restart container
az container restart \
  --resource-group myResourceGroup \
  --name mycontainer

# Delete container
az container delete \
  --resource-group myResourceGroup \
  --name mycontainer \
  --yes
```

**View Container Status:**

```bash
# Get container state
az container show \
  --resource-group myResourceGroup \
  --name mycontainer \
  --query "containers[0].instanceView.currentState"

# Get IP address
az container show \
  --resource-group myResourceGroup \
  --name mycontainer \
  --query ipAddress.ip \
  --output tsv

# Get FQDN
az container show \
  --resource-group myResourceGroup \
  --name mycontainer \
  --query ipAddress.fqdn \
  --output tsv
```

### 2.8 Networking

**Public IP Address:**

```bash
# Create with public IP and DNS label
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myapp:v1 \
  --ports 80 443 \
  --dns-name-label myapp-demo \
  --ip-address public

# Access via: myapp-demo.eastus.azurecontainer.io
```

**Private IP (VNet Integration):**

```bash
# Create subnet for ACI
az network vnet create \
  --resource-group myResourceGroup \
  --name myVnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name aciSubnet \
  --subnet-prefix 10.0.0.0/24

# Delegate subnet to ACI
az network vnet subnet update \
  --resource-group myResourceGroup \
  --vnet-name myVnet \
  --name aciSubnet \
  --delegations Microsoft.ContainerInstance/containerGroups

# Create container in VNet
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myapp:v1 \
  --vnet myVnet \
  --subnet aciSubnet \
  --ip-address private
```

### 2.9 Container Instance Limitations

**Resource Limits:**

| Resource | Linux | Windows |
|----------|-------|---------|
| Max CPU | 4 cores | 4 cores |
| Max Memory | 16 GB | 16 GB |
| Max GPU | 4 (K80/P100/V100) | Not supported |

**General Limitations:**
- No built-in scaling (must create new instances)
- No load balancing (use Application Gateway)
- Single availability zone
- No rolling updates
- Container groups can't be modified after creation

---

## MODULE 3: AZURE CONTAINER APPS

### 3.1 Introduction to Azure Container Apps

**What is Azure Container Apps?**
- Fully managed serverless container service
- Built on Kubernetes and open-source technologies
- Abstracts infrastructure management
- Automatic scaling (including to zero)
- Pay only for resources used

**Built On:**
- Azure Kubernetes Service (AKS)
- KEDA (Kubernetes Event-Driven Autoscaling)
- Dapr (Distributed Application Runtime)
- Envoy (service proxy)

**Key Features:**
- Serverless containers
- Automatic horizontal scaling
- HTTPS ingress without infrastructure
- Traffic splitting for blue/green and A/B testing
- Internal service discovery
- Dapr integration
- Event-driven processing
- Long-running background services

**Use Cases:**
- Microservices and APIs
- Event-driven processing
- Background processing
- Web applications
- Long-running processes
- Jobs and scheduled tasks

### 3.2 Container Apps vs Other Services

**Comparison:**

| Feature | Container Apps | ACI | AKS | App Service |
|---------|---------------|-----|-----|-------------|
| Serverless | ✅ | ✅ | ❌ | Partial |
| Auto-scaling | ✅ (KEDA) | ❌ | Manual/HPA | ✅ |
| Scale to zero | ✅ | ❌ | ❌ | ❌ |
| Kubernetes | Abstracted | ❌ | Full access | ❌ |
| Dapr support | ✅ | ❌ | Manual | ❌ |
| Ingress | Built-in | Manual | Manual | Built-in |
| Microservices | ✅ | Basic | ✅ | Limited |
| Management | Low | Low | High | Low |

**When to Use Container Apps:**
- Microservices needing easy orchestration
- Event-driven applications
- Apps requiring scale-to-zero
- Teams wanting serverless with containers
- Dapr-based applications
- HTTP APIs and web apps

### 3.3 Container Apps Architecture

**Core Concepts:**

```
Container Apps Environment
    └── Container App
            └── Revision (immutable snapshot)
                    └── Replica (running instance)
                            └── Container(s)
```

#### **Container Apps Environment**
- Secure boundary around apps
- Shared virtual network
- Shared logging (Log Analytics)
- Can contain multiple apps
- Apps can communicate internally

#### **Container App**
- Single application definition
- Multiple revisions possible
- Configuration and secrets
- Scaling rules
- Ingress configuration

#### **Revision**
- Immutable snapshot of app version
- Created on container/config change
- Multiple revisions can run simultaneously
- Traffic splitting between revisions

#### **Replica**
- Running instance of a revision
- Automatically scaled based on rules
- Can scale to zero
- Contains one or more containers

### 3.4 Creating Container Apps

**Create Container App Environment:**

```bash
# Create environment
az containerapp env create \
  --name myenvironment \
  --resource-group myResourceGroup \
  --location eastus

# Create with Log Analytics
az containerapp env create \
  --name myenvironment \
  --resource-group myResourceGroup \
  --location eastus \
  --logs-workspace-id <workspace-id> \
  --logs-workspace-key <workspace-key>
```

**Create Container App:**

```bash
# Create simple container app
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
  --target-port 80 \
  --ingress external

# Create from ACR
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-server myregistry.azurecr.io \
  --registry-username <username> \
  --registry-password <password> \
  --target-port 8080 \
  --ingress external

# Create with system-assigned managed identity
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-server myregistry.azurecr.io \
  --registry-identity system \
  --target-port 8080 \
  --ingress external

# Create with environment variables
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myapp:v1 \
  --env-vars "DB_HOST=mydb.database.windows.net" "APP_ENV=production" \
  --target-port 8080 \
  --ingress external
```

**Create with YAML:**

```yaml
# container-app.yaml
properties:
  managedEnvironmentId: /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.App/managedEnvironments/myenvironment
  configuration:
    ingress:
      external: true
      targetPort: 80
      transport: auto
      traffic:
        - weight: 100
          latestRevision: true
    secrets:
      - name: db-password
        value: secretpassword
    registries:
      - server: myregistry.azurecr.io
        username: myregistry
        passwordSecretRef: registry-password
  template:
    containers:
      - image: myregistry.azurecr.io/myapp:v1
        name: myapp
        resources:
          cpu: 0.5
          memory: 1Gi
        env:
          - name: DB_HOST
            value: mydb.database.windows.net
          - name: DB_PASSWORD
            secretRef: db-password
    scale:
      minReplicas: 1
      maxReplicas: 10
      rules:
        - name: http-scale
          http:
            metadata:
              concurrentRequests: '50'
```

### 3.5 Ingress Configuration

**Ingress Types:**

| Type | Access |
|------|--------|
| External | Internet and internal |
| Internal | Internal to environment only |
| None | No HTTP ingress (background services) |

**Configure Ingress:**

```bash
# Enable external ingress
az containerapp ingress enable \
  --name myapp \
  --resource-group myResourceGroup \
  --type external \
  --target-port 80

# Enable internal ingress
az containerapp ingress enable \
  --name myapp \
  --resource-group myResourceGroup \
  --type internal \
  --target-port 80

# Disable ingress
az containerapp ingress disable \
  --name myapp \
  --resource-group myResourceGroup

# Show ingress FQDN
az containerapp show \
  --name myapp \
  --resource-group myResourceGroup \
  --query properties.configuration.ingress.fqdn
```

**Ingress Features:**
- HTTPS by default (automatic certificates)
- Custom domains supported
- Traffic splitting
- Session affinity

### 3.6 Scaling Rules

**Scale Triggers:**
- HTTP concurrent requests
- CPU utilization
- Memory utilization
- Event-driven (KEDA scalers)
- Azure Queue Storage
- Azure Service Bus
- Custom metrics

**Configure Scaling:**

```bash
# Set scale rules
az containerapp update \
  --name myapp \
  --resource-group myResourceGroup \
  --min-replicas 1 \
  --max-replicas 10

# HTTP scaling rule
az containerapp update \
  --name myapp \
  --resource-group myResourceGroup \
  --scale-rule-name http-scaling \
  --scale-rule-type http \
  --scale-rule-http-concurrency 50
```

**YAML Scaling Configuration:**

```yaml
scale:
  minReplicas: 0
  maxReplicas: 30
  rules:
    # HTTP concurrent requests
    - name: http-rule
      http:
        metadata:
          concurrentRequests: '100'
    
    # CPU-based scaling
    - name: cpu-rule
      custom:
        type: cpu
        metadata:
          type: Utilization
          value: '70'
    
    # Azure Queue scaling
    - name: queue-rule
      azureQueue:
        queueName: myqueue
        queueLength: 20
        auth:
          - secretRef: queue-connection
            triggerParameter: connection
    
    # Azure Service Bus scaling
    - name: servicebus-rule
      custom:
        type: azure-servicebus
        metadata:
          queueName: myqueue
          messageCount: '5'
        auth:
          - secretRef: sb-connection
            triggerParameter: connection
```

### 3.7 Revisions and Traffic Splitting

**Revision Modes:**

| Mode | Description |
|------|-------------|
| Single | Only one revision active, new deploys replace |
| Multiple | Multiple revisions can receive traffic |

**Manage Revisions:**

```bash
# List revisions
az containerapp revision list \
  --name myapp \
  --resource-group myResourceGroup \
  --output table

# Show revision details
az containerapp revision show \
  --name myapp \
  --resource-group myResourceGroup \
  --revision myapp--abc123

# Activate revision
az containerapp revision activate \
  --name myapp \
  --resource-group myResourceGroup \
  --revision myapp--abc123

# Deactivate revision
az containerapp revision deactivate \
  --name myapp \
  --resource-group myResourceGroup \
  --revision myapp--abc123

# Restart revision
az containerapp revision restart \
  --name myapp \
  --resource-group myResourceGroup \
  --revision myapp--abc123

# Set revision mode
az containerapp revision set-mode \
  --name myapp \
  --resource-group myResourceGroup \
  --mode multiple
```

**Traffic Splitting:**

```bash
# Split traffic between revisions
az containerapp ingress traffic set \
  --name myapp \
  --resource-group myResourceGroup \
  --revision-weight myapp--v1=80 myapp--v2=20

# Route all traffic to latest
az containerapp ingress traffic set \
  --name myapp \
  --resource-group myResourceGroup \
  --revision-weight latest=100

# View traffic distribution
az containerapp ingress traffic show \
  --name myapp \
  --resource-group myResourceGroup
```

**Traffic Splitting Use Cases:**
- **Blue/Green Deployments**: 100% to new or old
- **Canary Releases**: Small percentage to new version
- **A/B Testing**: Split traffic for testing

### 3.8 Secrets and Environment Variables

**Manage Secrets:**

```bash
# Set secrets
az containerapp secret set \
  --name myapp \
  --resource-group myResourceGroup \
  --secrets "db-password=secretvalue" "api-key=myapikey"

# List secrets
az containerapp secret list \
  --name myapp \
  --resource-group myResourceGroup

# Remove secret
az containerapp secret remove \
  --name myapp \
  --resource-group myResourceGroup \
  --secret-names db-password
```

**Reference Secrets in Environment Variables:**

```yaml
containers:
  - name: myapp
    image: myapp:v1
    env:
      - name: DB_PASSWORD
        secretRef: db-password
      - name: PLAIN_VAR
        value: "plain-value"
```

### 3.9 Dapr Integration

**What is Dapr?**
- Distributed Application Runtime
- Simplifies microservices development
- Provides building blocks for common patterns
- Sidecar architecture

**Dapr Building Blocks:**
- Service-to-service invocation
- State management
- Pub/sub messaging
- Bindings (input/output)
- Actors
- Secrets management
- Configuration

**Enable Dapr:**

```bash
# Create app with Dapr enabled
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myapp:v1 \
  --enable-dapr \
  --dapr-app-id myapp \
  --dapr-app-port 3000
```

**YAML Dapr Configuration:**

```yaml
properties:
  configuration:
    dapr:
      enabled: true
      appId: myapp
      appPort: 3000
      appProtocol: http
```

**Service-to-Service Invocation:**

```csharp
// Call another service via Dapr
var client = new HttpClient();
var response = await client.GetAsync("http://localhost:3500/v1.0/invoke/otherservice/method/api/data");
```

### 3.10 Container App Jobs

**What are Container App Jobs?**
- Run containers that perform a task and exit
- Not continuously running
- Scheduled or manual triggers

**Job Types:**
- **Manual**: Triggered on-demand
- **Scheduled**: Run on cron schedule
- **Event-driven**: Triggered by events (queue messages, etc.)

```bash
# Create scheduled job
az containerapp job create \
  --name myjob \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --trigger-type Schedule \
  --cron-expression "0 0 * * *" \
  --image myregistry.azurecr.io/myjob:v1 \
  --cpu 0.5 \
  --memory 1Gi

# Create manual job
az containerapp job create \
  --name myjob \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --trigger-type Manual \
  --image myregistry.azurecr.io/myjob:v1

# Start manual job
az containerapp job start \
  --name myjob \
  --resource-group myResourceGroup

# List job executions
az containerapp job execution list \
  --name myjob \
  --resource-group myResourceGroup
```

---

## MODULE 4: DOCKERFILE & CONTAINER IMAGE CONCEPTS

### 4.1 Dockerfile Instructions Reference

**What is a Dockerfile?**
- Text file with instructions to build a Docker image
- Each instruction creates a layer in the image
- Layers are cached for faster subsequent builds
- Read and executed top-to-bottom

**Complete Dockerfile Instructions:**

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image (required, must be first) | `FROM mcr.microsoft.com/dotnet/aspnet:8.0` |
| `RUN` | Execute command during build | `RUN apt-get update && apt-get install -y curl` |
| `COPY` | Copy files from build context | `COPY ./src /app/src` |
| `ADD` | Copy files (supports URLs and tar extraction) | `ADD https://example.com/file.tar.gz /tmp/` |
| `CMD` | Default command when container starts | `CMD ["dotnet", "myapp.dll"]` |
| `ENTRYPOINT` | Configure container as executable | `ENTRYPOINT ["dotnet", "myapp.dll"]` |
| `ENV` | Set environment variable | `ENV ASPNETCORE_URLS=http://+:80` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `EXPOSE` | Document which ports to publish | `EXPOSE 80 443` |
| `ARG` | Build-time variable | `ARG BUILD_VERSION=1.0` |
| `VOLUME` | Create mount point | `VOLUME /data` |
| `USER` | Set user for subsequent instructions | `USER appuser` |
| `LABEL` | Add metadata | `LABEL version="1.0" maintainer="dev@co.com"` |
| `HEALTHCHECK` | Container health check | `HEALTHCHECK CMD curl -f http://localhost/ \|\| exit 1` |
| `ONBUILD` | Trigger for child image builds | `ONBUILD COPY . /app` |
| `STOPSIGNAL` | Signal to stop the container | `STOPSIGNAL SIGTERM` |
| `SHELL` | Override default shell | `SHELL ["powershell", "-Command"]` |

### 4.2 CMD vs ENTRYPOINT (Critical Exam Topic)

**CMD:**
- Provides default arguments/command
- Can be **overridden** at `docker run`
- Only the **last** CMD takes effect
- Three forms: exec, shell, as ENTRYPOINT parameters

```dockerfile
# Exec form (preferred)
CMD ["dotnet", "myapp.dll"]

# Shell form
CMD dotnet myapp.dll

# As ENTRYPOINT parameters
ENTRYPOINT ["dotnet"]
CMD ["myapp.dll"]
```

**ENTRYPOINT:**
- Configures container as executable
- **Cannot** be easily overridden (need `--entrypoint` flag)
- Combined with CMD for default arguments

```dockerfile
# Container always runs dotnet, CMD provides default dll
ENTRYPOINT ["dotnet"]
CMD ["myapp.dll"]

# docker run myimage              → dotnet myapp.dll
# docker run myimage otherapp.dll → dotnet otherapp.dll
```

**Key Distinction (Exam Favorite):**

| Scenario | CMD | ENTRYPOINT |
|----------|-----|------------|
| `docker run myimage` | Runs CMD | Runs ENTRYPOINT |
| `docker run myimage /bin/sh` | CMD is **replaced** | ENTRYPOINT stays, `/bin/sh` becomes argument |
| Override mechanism | Easy to override | Requires `--entrypoint` flag |
| Use case | Default arguments | Fixed executable |

**Shell Form vs Exec Form:**

```dockerfile
# Exec form (preferred) - PID 1 is the process itself
CMD ["dotnet", "myapp.dll"]
# Runs: dotnet myapp.dll

# Shell form - PID 1 is /bin/sh -c
CMD dotnet myapp.dll
# Runs: /bin/sh -c "dotnet myapp.dll"
```

> ⚠️ **Exam Tip**: Shell form wraps command in `/bin/sh -c`, which means signals (like SIGTERM for graceful shutdown) go to the shell, NOT the app. Always use exec form for production.

### 4.3 COPY vs ADD

| Feature | COPY | ADD |
|---------|------|-----|
| Copy local files | ✅ | ✅ |
| Copy from URL | ❌ | ✅ |
| Auto-extract tar archives | ❌ | ✅ |
| Recommended for most cases | ✅ | ❌ |

```dockerfile
# COPY (preferred for local files)
COPY package.json /app/
COPY . /app/

# ADD (only when you need URL or tar extraction)
ADD https://example.com/file.tar.gz /tmp/
ADD archive.tar.gz /app/  # Auto-extracts
```

> ⚠️ **Best Practice**: Always use `COPY` unless you specifically need URL download or tar extraction.

### 4.4 Multi-Stage Builds (Frequently Tested)

**Purpose:**
- Reduce final image size dramatically
- Separate build environment from runtime
- Keep secrets/build tools out of production image
- Single Dockerfile for build + runtime

**Example - .NET Application:**

```dockerfile
# Stage 1: Build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.csproj ./
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

# Stage 2: Runtime (much smaller image)
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 80
ENTRYPOINT ["dotnet", "myapp.dll"]
```

**Example - Node.js Application:**

```dockerfile
# Stage 1: Build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Runtime
FROM node:18-alpine AS runtime
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

**Key Points:**
- `AS build` names the build stage
- `COPY --from=build` copies from named stage
- Final image only contains runtime stage layers
- SDK/build tools are NOT in the final image
- Can have multiple build stages

### 4.5 Image Layering and Caching

**How Layers Work:**
- Each instruction creates a new layer
- Layers are stacked and cached
- If a layer changes, all subsequent layers are rebuilt
- Order instructions from least to most frequently changing

**Layer Caching Best Practices:**

```dockerfile
# ❌ BAD - Changes to any source file invalidate npm install cache
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "index.js"]

# ✅ GOOD - npm install is cached unless package.json changes
FROM node:18
WORKDIR /app
COPY package*.json ./       # Changes rarely
RUN npm install              # Cached if package.json unchanged
COPY . .                     # Source code changes often
CMD ["node", "index.js"]
```

**Reduce Image Size:**
```dockerfile
# Combine RUN commands to reduce layers
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Use alpine-based images
FROM node:18-alpine    # ~50MB vs ~350MB for node:18

# Use .dockerignore
```

### 4.6 .dockerignore File

**Purpose:** Exclude files from the build context (faster builds, smaller images, no secrets in image)

```
# .dockerignore
.git
.gitignore
node_modules
npm-debug.log
Dockerfile
.dockerignore
docker-compose*.yml
.env
*.md
.vscode
bin/
obj/
*.pdb
tests/
```

### 4.7 Docker Compose (Basics for Exam)

**What is Docker Compose?**
- Tool for defining multi-container applications
- Uses YAML file (`docker-compose.yml`)
- Single command to start all services
- Useful for local development and testing

```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8080:80"
    environment:
      - ConnectionStrings__Default=Server=db;Database=mydb;User=sa;Password=MyPass123
    depends_on:
      - db
  
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=MyPass123
    ports:
      - "1433:1433"
    volumes:
      - sqldata:/var/opt/mssql

volumes:
  sqldata:
```

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# Build and start
docker-compose up --build

# View logs
docker-compose logs -f web
```

### 4.8 Essential Docker CLI Commands (Exam Reference)

```bash
# Image Management
docker build -t myapp:v1 .                    # Build image
docker build -t myapp:v1 -f Dockerfile.prod .  # Build with specific Dockerfile
docker images                                   # List images
docker rmi myapp:v1                            # Remove image
docker tag myapp:v1 myregistry.azurecr.io/myapp:v1  # Tag for registry
docker push myregistry.azurecr.io/myapp:v1     # Push to registry
docker pull myregistry.azurecr.io/myapp:v1     # Pull from registry

# Container Management
docker run -d -p 8080:80 --name mycontainer myapp:v1  # Run detached
docker run -it myapp:v1 /bin/sh                 # Interactive shell
docker run --rm myapp:v1                        # Auto-remove on exit
docker run -e "ENV_VAR=value" myapp:v1          # With env vars
docker run -v /host/path:/container/path myapp:v1  # Mount volume
docker ps                                        # List running containers
docker ps -a                                     # List all containers
docker stop mycontainer                          # Stop container
docker start mycontainer                         # Start container
docker rm mycontainer                            # Remove container
docker logs mycontainer                          # View logs
docker exec -it mycontainer /bin/sh              # Execute in running container
docker inspect mycontainer                       # Detailed info

# System
docker system prune                              # Clean up unused resources
docker system df                                 # Disk usage
```

---

## MODULE 5: ACR ADVANCED CONCEPTS

### 5.1 Import Images (az acr import)

**Import images from other registries without Docker CLI:**

```bash
# Import from Docker Hub
az acr import \
  --name myregistry \
  --source docker.io/library/nginx:latest \
  --image nginx:latest

# Import from another ACR
az acr import \
  --name myregistry \
  --source sourceregistry.azurecr.io/myapp:v1 \
  --image myapp:v1

# Import from MCR (Microsoft Container Registry)
az acr import \
  --name myregistry \
  --source mcr.microsoft.com/dotnet/aspnet:8.0 \
  --image dotnet/aspnet:8.0

# Import with credentials for source registry
az acr import \
  --name myregistry \
  --source sourceregistry.azurecr.io/myapp:v1 \
  --image myapp:v1 \
  --username <source-username> \
  --password <source-password>

# Import by digest
az acr import \
  --name myregistry \
  --source docker.io/library/nginx@sha256:abc123... \
  --image nginx:imported
```

**Key Benefits:**
- No local Docker needed
- Doesn't use Docker pull/push bandwidth
- Works directly between registries
- Useful for air-gapped environments
- Can import from any Docker V2 compatible registry

### 5.2 ACR Webhooks

**Webhook Events:**

| Event | Trigger |
|-------|---------|
| `push` | Image pushed or manifest pushed |
| `delete` | Image or manifest deleted |
| `quarantine` | Image quarantined (with content trust) |
| `chart_push` | Helm chart pushed |
| `chart_delete` | Helm chart deleted |

```bash
# Create webhook
az acr webhook create \
  --registry myregistry \
  --name mywebhook \
  --uri https://myapp.azurewebsites.net/api/webhook \
  --actions push delete \
  --scope "myapp:*"

# List webhooks
az acr webhook list --registry myregistry

# Test webhook (ping)
az acr webhook ping --registry myregistry --name mywebhook

# Get webhook delivery logs
az acr webhook list-events --registry myregistry --name mywebhook
```

### 5.3 ACR Image Retention and Purge

**Retention Policy (Preview - Premium):**

```bash
# Set retention policy (delete untagged manifests after 7 days)
az acr config retention update \
  --registry myregistry \
  --status enabled \
  --days 7 \
  --type UntaggedManifests
```

**ACR Purge (via ACR Tasks):**

```bash
# Purge images older than 30 days
az acr run \
  --cmd "acr purge --filter 'myapp:.*' --ago 30d --untagged" \
  --registry myregistry \
  /dev/null

# Schedule automatic purge
az acr task create \
  --name purgeTask \
  --registry myregistry \
  --cmd "acr purge --filter 'myapp:.*' --ago 90d --untagged --keep 5" \
  --schedule "0 0 * * *" \
  --context /dev/null

# Purge with dry-run first
az acr run \
  --cmd "acr purge --filter 'myapp:.*' --ago 30d --dry-run" \
  --registry myregistry \
  /dev/null
```

**Purge Parameters:**

| Parameter | Description |
|-----------|-------------|
| `--filter` | Repository and tag pattern |
| `--ago` | Duration string (e.g., `30d`, `2h`) |
| `--untagged` | Also purge untagged manifests |
| `--keep` | Number of latest tags to keep |
| `--dry-run` | Preview what would be deleted |

### 5.4 Vulnerability Scanning (Microsoft Defender for Containers)

**Features:**
- Automatic scanning on image push
- Continuous scanning of images in registry
- Vulnerability assessment reports
- Integration with Azure Security Center
- Recommendations and remediation guidance

**Enable:**
```bash
# Enable Defender for Containers
az security pricing create \
  --name Containers \
  --tier Standard
```

**Key Concepts:**
- Scans for OS and application vulnerabilities
- Provides severity ratings (Critical, High, Medium, Low)
- Works with Linux-based images
- Generates security recommendations in Azure portal
- Part of Microsoft Defender for Cloud

### 5.5 ACR Managed Identities

**Use Managed Identity for ACR Authentication:**

```bash
# Create ACR with system-assigned managed identity
az acr create \
  --name myregistry \
  --resource-group myRG \
  --sku Premium \
  --identity

# Assign AcrPull role to AKS managed identity
az role assignment create \
  --assignee <aks-managed-identity-client-id> \
  --role AcrPull \
  --scope $(az acr show --name myregistry --query id -o tsv)

# Assign AcrPush role to a user-assigned managed identity
az role assignment create \
  --assignee <managed-identity-client-id> \
  --role AcrPush \
  --scope $(az acr show --name myregistry --query id -o tsv)
```

### 5.6 ACR Customer-Managed Keys (Premium)

```bash
# Create key vault and key
az keyvault create --name mykeyvault --resource-group myRG --location eastus
az keyvault key create --vault-name mykeyvault --name mykey

# Create ACR with CMK
az acr create \
  --name myregistry \
  --resource-group myRG \
  --sku Premium \
  --identity \
  --key-encryption-key https://mykeyvault.vault.azure.net/keys/mykey
```

### 5.7 ACR Private Link and Network Rules (Premium)

```bash
# Create private endpoint for ACR
az network private-endpoint create \
  --name myACRPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVnet \
  --subnet mySubnet \
  --private-connection-resource-id $(az acr show --name myregistry --query id -o tsv) \
  --group-ids registry \
  --connection-name myConnection

# Disable public network access
az acr update \
  --name myregistry \
  --public-network-enabled false

# Add IP-based network rule
az acr network-rule add \
  --name myregistry \
  --ip-address 203.0.113.0/24
```

### 5.8 ACR Connected Registries

**What are Connected Registries?**
- On-premises or remote mirror of your ACR
- Supports disconnected/IoT scenarios
- Sync images from cloud ACR to edge
- Works in connected or occasionally connected modes

**Modes:**
- **ReadOnly**: Pulls only, mirrors cloud registry
- **ReadWrite**: Push and pull locally, sync with cloud

---

## MODULE 6: ACI ADVANCED CONCEPTS

### 6.1 Liveness and Readiness Probes

**Liveness Probe:**
- Detects if container is healthy
- Failed probe → container restart
- Prevents stuck/deadlocked containers

**Readiness Probe:**
- Detects if container is ready to serve traffic
- Failed probe → removed from load balancer (not restarted)

**YAML Configuration:**

```yaml
apiVersion: '2021-10-01'
name: mycontainer
properties:
  containers:
  - name: myapp
    properties:
      image: myapp:v1
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      livenessProbe:
        httpGet:
          path: /health
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 30
        failureThreshold: 3
        successThreshold: 1
        timeoutSeconds: 5
      readinessProbe:
        httpGet:
          path: /ready
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
      ports:
      - port: 80
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - port: 80
```

**Probe Types:**

| Type | Configuration |
|------|--------------|
| HTTP GET | `httpGet: { path: /health, port: 80 }` |
| Exec | `exec: { command: ["cat", "/tmp/healthy"] }` |

**Probe Parameters:**

| Parameter | Description | Default |
|-----------|-------------|---------|
| `initialDelaySeconds` | Delay before first probe | 0 |
| `periodSeconds` | How often to probe | 10 |
| `failureThreshold` | Failures before action | 3 |
| `successThreshold` | Successes to be healthy | 1 |
| `timeoutSeconds` | Timeout per probe | 1 |

### 6.2 Init Containers

**What are Init Containers?**
- Run before application containers start
- Complete a specific task and exit
- Run sequentially (one at a time)
- Must complete successfully before app containers start

**Use Cases:**
- Database migrations
- Download configuration
- Wait for dependent services
- Set up permissions

```yaml
apiVersion: '2021-10-01'
name: mycontainergroup
properties:
  initContainers:
  - name: init-db
    properties:
      image: myapp-migration:v1
      command:
        - "dotnet"
        - "ef"
        - "database"
        - "update"
      environmentVariables:
      - name: DB_CONNECTION
        secureValue: 'Server=mydb;...'
  containers:
  - name: myapp
    properties:
      image: myapp:v1
      ports:
      - port: 80
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - port: 80
```

### 6.3 ACI with Managed Identity

```bash
# Create ACI with system-assigned managed identity
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image myapp:v1 \
  --assign-identity

# Create ACI with user-assigned managed identity
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image myapp:v1 \
  --assign-identity /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myIdentity

# Pull from ACR using managed identity
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image myregistry.azurecr.io/myapp:v1 \
  --acr-identity [system] \
  --assign-identity
```

**Using Managed Identity in Code:**

```csharp
// Access Azure resources from within the container
var credential = new DefaultAzureCredential();
var secretClient = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net/"), 
    credential);
var secret = await secretClient.GetSecretAsync("MySecret");
```

### 6.4 ACI GPU Support

```bash
# Create container with GPU (Linux only)
az container create \
  --resource-group myRG \
  --name mygpucontainer \
  --image tensorflow/tensorflow:latest-gpu \
  --cpu 4 \
  --memory 16 \
  --gpu-count 1 \
  --gpu-sku K80
```

**Available GPU SKUs:**
- K80
- P100
- V100

> ⚠️ GPU is only available in select regions and for Linux containers.

### 6.5 ACI Diagnostics and Logging

**Azure Monitor Integration:**

```bash
# View container logs
az container logs --resource-group myRG --name mycontainer

# Attach to container output (streaming)
az container attach --resource-group myRG --name mycontainer

# Get container events
az container show \
  --resource-group myRG \
  --name mycontainer \
  --query "containers[0].instanceView.events"
```

**Diagnostic Settings (send to Log Analytics):**

```bash
# Create diagnostic setting
az monitor diagnostic-settings create \
  --name myDiagSetting \
  --resource $(az container show --resource-group myRG --name mycontainer --query id -o tsv) \
  --workspace <log-analytics-workspace-id> \
  --logs '[{"category": "ContainerInstanceLog", "enabled": true}]'
```

### 6.6 ACI Deployment with ARM Templates

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2021-10-01",
      "name": "mycontainergroup",
      "location": "[resourceGroup().location]",
      "properties": {
        "containers": [
          {
            "name": "myapp",
            "properties": {
              "image": "myregistry.azurecr.io/myapp:v1",
              "ports": [{ "port": 80, "protocol": "TCP" }],
              "resources": {
                "requests": {
                  "cpu": 1.0,
                  "memoryInGb": 1.5
                }
              },
              "environmentVariables": [
                { "name": "ASPNETCORE_URLS", "value": "http://+:80" }
              ]
            }
          }
        ],
        "osType": "Linux",
        "restartPolicy": "OnFailure",
        "ipAddress": {
          "type": "Public",
          "ports": [{ "port": 80, "protocol": "TCP" }],
          "dnsNameLabel": "myapp-aci"
        },
        "imageRegistryCredentials": [
          {
            "server": "myregistry.azurecr.io",
            "username": "[parameters('registryUsername')]",
            "password": "[parameters('registryPassword')]"
          }
        ]
      }
    }
  ]
}
```

### 6.7 ACI Confidential Containers

**What are Confidential Containers?**
- Run containers in a hardware-based Trusted Execution Environment (TEE)
- Memory encrypted and isolated from host OS
- Protects data in use
- Based on AMD SEV-SNP technology
- No code changes required

**Use Cases:**
- Process sensitive data (PII, financial, healthcare)
- Multi-party computation
- Protect against privileged admin access
- Regulatory compliance

```bash
# Create confidential container group
az container create \
  --resource-group myRG \
  --name myconfidential \
  --image myapp:v1 \
  --sku Confidential \
  --cce-policy <base64-encoded-policy>
```

### 6.8 ACI Container Group Limitations and Networking Deep Dive

**Multi-container Group Constraints:**
- Linux only (Windows supports single container)
- All containers share: IP address, port namespace, storage volumes
- Resource allocation: total CPU/memory split among containers
- Cannot update after creation (must delete and recreate)
- Maximum 60 containers per group

**Port Mapping:**
```yaml
# Each container must use DIFFERENT ports
containers:
  - name: web
    properties:
      ports:
      - port: 80      # Web on port 80
  - name: api
    properties:
      ports:
      - port: 8080    # API on port 8080

# Both accessible via same public IP
# Internally communicate via localhost:port
```

**DNS Name Label:**
- Format: `<dns-label>.<region>.azurecontainer.io`
- Must be unique within the Azure region
- Only available with public IP

---

## MODULE 7: CONTAINER APPS ADVANCED CONCEPTS

### 7.1 Health Probes in Container Apps

**Three Types of Probes:**

| Probe | Purpose | Failure Action |
|-------|---------|----------------|
| Startup | Check if app has started | Block other probes until success |
| Liveness | Check if app is running | Restart container |
| Readiness | Check if app can serve traffic | Remove from load balancer |

**YAML Configuration:**

```yaml
properties:
  template:
    containers:
      - image: myapp:v1
        name: myapp
        probes:
          - type: startup
            httpGet:
              path: /startup
              port: 8080
            initialDelaySeconds: 3
            periodSeconds: 5
            failureThreshold: 30
          - type: liveness
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 30
            failureThreshold: 3
          - type: readiness
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 15
            successThreshold: 1
```

**Probe Transport Options:**
- HTTP GET
- TCP socket
- gRPC (for gRPC services)

### 7.2 Workload Profiles

**What are Workload Profiles?**
- Define the compute resources available to apps
- Choose between Consumption and Dedicated profiles
- Mix different profiles in the same environment

**Profile Types:**

| Profile | Description | Scale to Zero | Use Case |
|---------|-------------|---------------|----------|
| Consumption | Serverless, shared compute | ✅ | General purpose, cost-effective |
| Dedicated-D4 | 4 vCPU, 16 GiB memory | ❌ | Compute-intensive |
| Dedicated-D8 | 8 vCPU, 32 GiB memory | ❌ | High-performance |
| Dedicated-D16 | 16 vCPU, 64 GiB memory | ❌ | Large workloads |
| Dedicated-D32 | 32 vCPU, 128 GiB memory | ❌ | Enterprise |
| Dedicated-E4 | 4 vCPU, 32 GiB memory | ❌ | Memory-intensive |
| Dedicated-E8 | 8 vCPU, 64 GiB memory | ❌ | Memory-intensive |
| Dedicated-E16 | 16 vCPU, 128 GiB memory | ❌ | Memory-intensive |
| GPU | GPU-enabled profiles | ❌ | AI/ML workloads |

```bash
# Create environment with workload profiles
az containerapp env create \
  --name myenv \
  --resource-group myRG \
  --location eastus \
  --enable-workload-profiles

# Add a dedicated workload profile
az containerapp env workload-profile add \
  --name myenv \
  --resource-group myRG \
  --workload-profile-name my-d4-profile \
  --workload-profile-type D4 \
  --min-nodes 1 \
  --max-nodes 3

# Create app on specific workload profile
az containerapp create \
  --name myapp \
  --resource-group myRG \
  --environment myenv \
  --workload-profile-name my-d4-profile \
  --image myapp:v1
```

### 7.3 Managed Identity in Container Apps

```bash
# Enable system-assigned managed identity
az containerapp identity assign \
  --name myapp \
  --resource-group myRG \
  --system-assigned

# Enable user-assigned managed identity
az containerapp identity assign \
  --name myapp \
  --resource-group myRG \
  --user-assigned /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myIdentity

# Use managed identity to pull from ACR
az containerapp create \
  --name myapp \
  --resource-group myRG \
  --environment myenv \
  --image myregistry.azurecr.io/myapp:v1 \
  --registry-server myregistry.azurecr.io \
  --registry-identity system
```

**Reference in Code:**

```csharp
// Access Key Vault using managed identity
var credential = new DefaultAzureCredential();
var client = new SecretClient(new Uri("https://mykeyvault.vault.azure.net/"), credential);
```

### 7.4 Authentication (Easy Auth) in Container Apps

**Built-in Authentication:**
- No code changes required
- Similar to App Service Easy Auth
- Supports multiple identity providers

**Supported Providers:**
- Microsoft Identity Platform (Azure AD/Entra ID)
- Facebook
- GitHub
- Google
- Twitter
- OpenID Connect (custom)
- Apple

```bash
# Enable authentication with Azure AD
az containerapp auth update \
  --name myapp \
  --resource-group myRG \
  --unauthenticated-client-action RedirectToLoginPage \
  --set-string identityProviders.azureActiveDirectory.registration.clientId=<client-id> \
  --set-string identityProviders.azureActiveDirectory.registration.openIdIssuer=https://login.microsoftonline.com/<tenant-id>/v2.0
```

**Authentication Actions for Unauthenticated Requests:**

| Action | Description |
|--------|-------------|
| `AllowAnonymous` | Allow without authentication |
| `RedirectToLoginPage` | Redirect to identity provider |
| `Return401` | Return HTTP 401 Unauthorized |
| `Return403` | Return HTTP 403 Forbidden |

### 7.5 Custom Domains and Certificates

```bash
# Add custom domain
az containerapp hostname add \
  --name myapp \
  --resource-group myRG \
  --hostname www.mydomain.com

# Bind managed certificate
az containerapp hostname bind \
  --name myapp \
  --resource-group myRG \
  --hostname www.mydomain.com \
  --environment myenv \
  --validation-method CNAME

# Upload custom certificate
az containerapp ssl upload \
  --name myapp \
  --resource-group myRG \
  --environment myenv \
  --hostname www.mydomain.com \
  --certificate-file ./cert.pfx \
  --certificate-password <password>
```

**Certificate Types:**
- **Free Managed Certificate**: Auto-provisioned, auto-renewed, domain validation required
- **Custom Certificate**: Bring your own (.pfx), you manage renewal
- **Managed Certificate from Key Vault**: Reference a Key Vault certificate

### 7.6 Container Apps Networking (VNet Integration)

**Network Architecture:**

```
                    Internet
                       │
               ┌───────┴───────┐
               │  Envoy Proxy  │ (auto-managed)
               └───────┬───────┘
                       │
        ┌──────────────┴──────────────┐
        │   Container Apps Environment │
        │     (VNet integrated)        │
        │                              │
        │  ┌──────┐    ┌──────┐       │
        │  │App A │    │App B │       │
        │  └──────┘    └──────┘       │
        └──────────────────────────────┘
```

```bash
# Create environment with custom VNet
az containerapp env create \
  --name myenv \
  --resource-group myRG \
  --location eastus \
  --infrastructure-subnet-resource-id /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/infrastructure

# Container Apps environment requires a dedicated subnet
# Minimum subnet size: /23 (for workload profiles) or /27 (for consumption only)
```

**Network Types:**

| Setting | External Environment | Internal Environment |
|---------|---------------------|---------------------|
| External ingress apps | Internet accessible | VNet accessible only |
| Internal ingress apps | VNet accessible only | VNet accessible only |
| Default domain | `<app>.<unique-id>.<region>.azurecontainerapps.io` | `<app>.internal.<unique-id>.<region>.azurecontainerapps.io` |

### 7.7 Storage Mounts

**Supported Storage Types:**

| Storage Type | Description | Persistence |
|-------------|-------------|-------------|
| Azure Files | SMB file share | Persistent across revisions |
| Ephemeral | Temp storage in replica | Lost on replica restart |

```bash
# Add Azure Files storage to environment
az containerapp env storage set \
  --name myenv \
  --resource-group myRG \
  --storage-name mystorage \
  --azure-file-account-name mystorageaccount \
  --azure-file-account-key <key> \
  --azure-file-share-name myshare \
  --access-mode ReadWrite
```

**YAML Storage Mount:**

```yaml
properties:
  template:
    containers:
      - image: myapp:v1
        name: myapp
        volumeMounts:
          - volumeName: azure-files-volume
            mountPath: /mnt/data
          - volumeName: ephemeral-volume
            mountPath: /tmp/cache
    volumes:
      - name: azure-files-volume
        storageName: mystorage
        storageType: AzureFile
      - name: ephemeral-volume
        storageType: EmptyDir
```

### 7.8 Container Apps CI/CD

**GitHub Actions Deployment:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to Container Apps

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Login to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Build and push to ACR
        uses: azure/docker-login@v1
        with:
          login-server: myregistry.azurecr.io
          username: ${{ secrets.ACR_USERNAME }}
          password: ${{ secrets.ACR_PASSWORD }}
      
      - run: |
          docker build -t myregistry.azurecr.io/myapp:${{ github.sha }} .
          docker push myregistry.azurecr.io/myapp:${{ github.sha }}
      
      - name: Deploy to Container Apps
        uses: azure/container-apps-deploy-action@v1
        with:
          containerAppName: myapp
          resourceGroup: myRG
          imageToDeploy: myregistry.azurecr.io/myapp:${{ github.sha }}
```

**Azure DevOps Pipeline:**

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include: [main]

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: Docker@2
    inputs:
      containerRegistry: 'myACRConnection'
      repository: 'myapp'
      command: 'buildAndPush'
      Dockerfile: '**/Dockerfile'
      tags: '$(Build.BuildId)'
  
  - task: AzureContainerApps@1
    inputs:
      azureSubscription: 'myAzureConnection'
      containerAppName: 'myapp'
      resourceGroup: 'myRG'
      imageToDeploy: 'myregistry.azurecr.io/myapp:$(Build.BuildId)'
```

### 7.9 Container Apps Observability

**Built-in Logging:**
- All Container Apps environments use Log Analytics
- Console logs are automatically captured
- System logs for scaling, revision management

```bash
# View system logs (KQL)
az containerapp logs show \
  --name myapp \
  --resource-group myRG \
  --type system

# View console logs
az containerapp logs show \
  --name myapp \
  --resource-group myRG \
  --type console

# Stream logs in real-time
az containerapp logs show \
  --name myapp \
  --resource-group myRG \
  --type console \
  --follow
```

**KQL Queries for Container Apps:**

```kql
// View recent container app logs
ContainerAppConsoleLogs_CL
| where ContainerAppName_s == "myapp"
| where TimeGenerated > ago(1h)
| project TimeGenerated, Log_s, RevisionName_s
| order by TimeGenerated desc

// View system events (scaling, etc.)
ContainerAppSystemLogs_CL
| where ContainerAppName_s == "myapp"
| where Type_s == "Normal" or Type_s == "Warning"
| project TimeGenerated, Reason_s, Message_s

// Track replica count over time
ContainerAppSystemLogs_CL
| where Reason_s == "ScalingReplica"
| project TimeGenerated, Message_s
```

**Application Insights Integration:**
- Set `APPLICATIONINSIGHTS_CONNECTION_STRING` env var
- Or use OpenTelemetry Collector as a Dapr component

### 7.10 Sidecar Containers

**What are Sidecar Containers?**
- Additional containers running alongside the main app container
- Share the same revision lifecycle
- Common patterns: logging agents, proxies, configuration watchers

```yaml
properties:
  template:
    containers:
      - image: myapp:v1
        name: main-app
        resources:
          cpu: 0.5
          memory: 1Gi
        env:
          - name: LOG_PATH
            value: /shared/logs
        volumeMounts:
          - volumeName: shared-logs
            mountPath: /shared/logs
      - image: fluentd:latest
        name: log-shipper
        resources:
          cpu: 0.25
          memory: 0.5Gi
        volumeMounts:
          - volumeName: shared-logs
            mountPath: /shared/logs
    volumes:
      - name: shared-logs
        storageType: EmptyDir
```

### 7.11 Container Apps Session Affinity

**Sticky Sessions:**
- Route requests from same client to same replica
- Uses cookies for session tracking
- Useful for stateful applications

```bash
# Enable session affinity
az containerapp update \
  --name myapp \
  --resource-group myRG \
  --query properties.configuration.ingress.stickySessions

# YAML configuration
```

```yaml
properties:
  configuration:
    ingress:
      stickySessions:
        affinity: sticky
```

### 7.12 Container Apps Service Connectors

**What are Service Connectors?**
- Simplify connecting to Azure services (databases, storage, caches, etc.)
- Auto-configure connection strings and authentication
- Support managed identity authentication

```bash
# Connect to Azure SQL
az containerapp connection create sql \
  --name myapp \
  --resource-group myRG \
  --target-resource-group myRG \
  --server myserver \
  --database mydb \
  --system-identity

# Connect to Azure Storage
az containerapp connection create storage-blob \
  --name myapp \
  --resource-group myRG \
  --target-resource-group myRG \
  --account mystorageaccount \
  --system-identity
```

### 7.13 Container Apps Revision Labels

**What are Revision Labels?**
- Assign a stable URL to a specific revision
- Access a specific revision directly (bypass traffic splitting)
- Useful for testing a new revision before sending traffic

```bash
# Assign label to a revision
az containerapp revision label add \
  --name myapp \
  --resource-group myRG \
  --revision myapp--abc123 \
  --label staging

# Access via: staging---myapp.<env-id>.<region>.azurecontainerapps.io

# Remove label
az containerapp revision label remove \
  --name myapp \
  --resource-group myRG \
  --label staging
```

---

## EXAM PREPARATION: KEY TOPICS & QUESTIONS

### Critical Concepts to Master

**1. Azure Container Registry:**
- Service tiers and features (Premium-only: geo-replication, private link, content trust, CMK)
- Authentication methods (Azure AD, admin, service principal, managed identity, repository tokens)
- ACR Tasks (quick, triggered, multi-step YAML)
- Geo-replication and import images
- Security features (vulnerability scanning, content trust, private endpoints)
- Image retention and purge policies
- Webhooks (push, delete events)

**2. Azure Container Instances:**
- Container groups (shared network, storage, lifecycle)
- Restart policies (Always, OnFailure, Never)
- Environment variables (secure vs plain)
- Volume mounts (Azure Files, secret, emptyDir, gitRepo)
- Networking (public IP, VNet integration, DNS labels)
- Liveness and readiness probes
- Init containers
- Managed identity
- Confidential containers
- GPU support

**3. Azure Container Apps:**
- Environment and app architecture (Environment → App → Revision → Replica)
- Revisions and traffic splitting (single vs multiple revision mode)
- Scaling rules (HTTP, event-driven KEDA, CPU, memory, custom)
- Dapr integration (service invocation, pub/sub, state management)
- Secrets management and Key Vault references
- Health probes (startup, liveness, readiness)
- Workload profiles (Consumption vs Dedicated)
- Managed identity and authentication (Easy Auth)
- Custom domains and certificates
- VNet integration and networking
- Storage mounts (Azure Files, ephemeral)
- Container App Jobs (manual, scheduled, event-driven)
- Revision labels for targeted access
- Service connectors
- Sidecar containers and observability

**4. Dockerfile & Container Fundamentals:**
- Dockerfile instructions (FROM, RUN, COPY, ADD, CMD, ENTRYPOINT, etc.)
- CMD vs ENTRYPOINT (critical exam distinction)
- COPY vs ADD
- Multi-stage builds (reduce image size, separate build from runtime)
- Image layering and caching optimization
- .dockerignore file
- Docker CLI commands (build, run, push, pull, tag)
- Shell form vs exec form (PID 1 implications)

### Sample Exam Questions

#### Azure Container Registry

**Q1:** Which ACR service tier supports geo-replication?
- A) Basic
- B) Standard
- C) Premium
- D) All tiers

**Answer: C) Premium**
Explanation: Geo-replication is only available in the Premium tier, along with other advanced features like content trust and private link.

---

**Q2:** You need to automatically build a container image when code is pushed to a GitHub repository. Which ACR feature should you use?
- A) ACR Quick Task
- B) ACR Triggered Task
- C) ACR Multi-step Task
- D) ACR Webhook

**Answer: B) ACR Triggered Task**
Explanation: ACR Tasks with source triggers can automatically build images when code is committed to a repository.

---

**Q3:** Which command builds a container image in ACR without a local Docker installation?
- A) `docker build`
- B) `az acr build`
- C) `az container build`
- D) `az acr task run`

**Answer: B) `az acr build`**
Explanation: `az acr build` performs a quick task that builds the image in the cloud without requiring local Docker.

---

**Q4:** What is the recommended authentication method for production workloads accessing ACR?
- A) Admin account
- B) Azure AD with RBAC
- C) Anonymous access
- D) Shared access signature

**Answer: B) Azure AD with RBAC**
Explanation: Azure AD authentication with role-based access control is the recommended approach for production. Admin accounts are discouraged for production use.

---

**Q5:** Which ACR role allows pushing images but not deleting them?
- A) AcrPull
- B) AcrPush
- C) AcrDelete
- D) Contributor

**Answer: B) AcrPush**
Explanation: AcrPush allows pushing (and pulling) images. AcrDelete is needed for deletion. AcrPull is read-only.

---

#### Azure Container Instances

**Q6:** Which restart policy should you use for a one-time batch job that should not retry on failure?
- A) Always
- B) OnFailure
- C) Never
- D) Once

**Answer: C) Never**
Explanation: The "Never" restart policy ensures the container runs once and stops, regardless of exit code.

---

**Q7:** What is a container group in Azure Container Instances?
- A) A collection of containers in different regions
- B) A collection of containers scheduled on the same host machine
- C) A group of ACR repositories
- D) A Kubernetes cluster

**Answer: B) A collection of containers scheduled on the same host machine**
Explanation: A container group is similar to a Kubernetes pod - containers that share lifecycle, network, and storage, scheduled on the same host.

---

**Q8:** How do you pass sensitive information to a container instance without exposing it in logs?
- A) Environment variables
- B) Secure environment variables
- C) Command line arguments
- D) Configuration files

**Answer: B) Secure environment variables**
Explanation: Secure environment variables (`--secure-environment-variables` or `secureValue` in YAML) are not exposed in container logs or the Azure portal.

---

**Q9:** What is the maximum memory for a single container in Azure Container Instances?
- A) 4 GB
- B) 8 GB
- C) 16 GB
- D) 32 GB

**Answer: C) 16 GB**
Explanation: The maximum memory for a container in ACI is 16 GB. Maximum CPU is 4 cores.

---

**Q10:** Which volume type allows persisting data beyond the container lifecycle in ACI?
- A) Empty directory
- B) Secret volume
- C) Azure Files
- D) Git repository

**Answer: C) Azure Files**
Explanation: Azure Files provides persistent storage using SMB file shares that survive container restarts and deletions.

---

#### Azure Container Apps

**Q11:** What technology does Azure Container Apps use for event-driven autoscaling?
- A) Azure Autoscale
- B) KEDA
- C) Horizontal Pod Autoscaler
- D) Virtual Machine Scale Sets

**Answer: B) KEDA**
Explanation: Azure Container Apps uses KEDA (Kubernetes Event-Driven Autoscaling) for scaling based on events and metrics.

---

**Q12:** What is the minimum number of replicas you can scale to in Azure Container Apps?
- A) 0
- B) 1
- C) 2
- D) 3

**Answer: A) 0**
Explanation: Container Apps can scale to zero replicas when there's no traffic or events, which is a key serverless feature.

---

**Q13:** You want to deploy a new version of your app and send 20% of traffic to it while keeping 80% on the old version. What feature should you use?
- A) Revision mode
- B) Traffic splitting
- C) Dapr routing
- D) Load balancer

**Answer: B) Traffic splitting**
Explanation: Traffic splitting allows you to distribute traffic between multiple revisions, enabling canary deployments and A/B testing.

---

**Q14:** What is a revision in Azure Container Apps?
- A) A version control commit
- B) An immutable snapshot of a container app version
- C) A backup of the app
- D) A deployment script

**Answer: B) An immutable snapshot of a container app version**
Explanation: A revision is an immutable snapshot of the container app, created when the container image or configuration changes.

---

**Q15:** Which Dapr building block enables communication between microservices in Container Apps?
- A) State management
- B) Service-to-service invocation
- C) Pub/sub messaging
- D) All of the above

**Answer: D) All of the above**
Explanation: Dapr provides multiple building blocks for microservices communication, including service invocation, state management, and pub/sub.

---

**Q16:** What determines whether a Container App is accessible from the internet or only internally?
- A) Scale rules
- B) Ingress configuration
- C) Dapr settings
- D) Environment variables

**Answer: B) Ingress configuration**
Explanation: Ingress configuration (external vs internal) determines whether the app is accessible from the internet or only within the environment.

---

**Q17:** Which Container Apps scaling trigger monitors concurrent HTTP requests?
- A) CPU trigger
- B) Memory trigger
- C) HTTP trigger
- D) Queue trigger

**Answer: C) HTTP trigger**
Explanation: The HTTP scaling rule scales based on concurrent HTTP requests, automatically adding replicas when request volume increases.

---

**Q18:** How do you reference a secret in a Container App environment variable?
- A) `value: secretName`
- B) `secretRef: secretName`
- C) `secret: secretName`
- D) `valueFrom: secretName`

**Answer: B) `secretRef: secretName`**
Explanation: In Container Apps YAML, you use `secretRef` to reference a secret in an environment variable.

---

#### Advanced Scenarios

**Q19:** You need to run a batch processing job daily at midnight in Azure. Which Container Apps feature should you use?
- A) Container App with scaling rules
- B) Container App Job with scheduled trigger
- C) Container App with Dapr timer
- D) Azure Functions

**Answer: B) Container App Job with scheduled trigger**
Explanation: Container App Jobs with scheduled triggers allow running containers on a cron schedule for batch processing.

---

**Q20:** Which ACR feature allows you to run automated tests before pushing an image to the registry?
- A) Quick tasks
- B) Multi-step tasks
- C) Webhooks
- D) Content trust

**Answer: B) Multi-step tasks**
Explanation: Multi-step tasks defined in YAML can build, test, and conditionally push images in a single workflow.

---

**Q21:** How do containers in an ACI container group communicate with each other?
- A) Through the public IP
- B) Through localhost
- C) Through Azure Service Bus
- D) Through DNS names

**Answer: B) Through localhost**
Explanation: Containers in the same container group share the network namespace and can communicate via localhost on different ports.

---

**Q22:** What happens when you update a Container App in single revision mode?
- A) A new revision is created and receives all traffic
- B) Traffic is split between old and new revisions
- C) The update fails
- D) The old revision continues to receive traffic

**Answer: A) A new revision is created and receives all traffic**
Explanation: In single revision mode, the new revision automatically replaces the old one and receives all traffic.

---

**Q23:** Which command deploys a container group from a YAML file?
- A) `az container create --file`
- B) `az container deploy --yaml`
- C) `az aci create --template`
- D) `az container create --yaml`

**Answer: A) `az container create --file`**
Explanation: The `--file` parameter in `az container create` accepts a YAML file defining the container group.

---

**Q24:** What is required to pull images from ACR to Azure Container Instances?
- A) Only the registry URL
- B) Registry credentials or managed identity
- C) Public access enabled
- D) ACR webhook

**Answer: B) Registry credentials or managed identity**
Explanation: ACI needs authentication to pull from ACR, either through registry credentials (username/password) or managed identity.

---

**Q25:** Which Container Apps environment setting enables apps to communicate with each other using internal DNS?
- A) Virtual network integration
- B) Dapr service invocation
- C) Internal ingress
- D) All apps in the same environment can communicate by default

**Answer: D) All apps in the same environment can communicate by default**
Explanation: Apps in the same Container Apps environment can discover and communicate with each other using internal DNS names.

---

#### Dockerfile & Container Fundamentals

**Q26:** Given this Dockerfile, what command runs when you execute `docker run myimage`?
```
FROM node:18
WORKDIR /app
COPY . .
ENTRYPOINT ["node"]
CMD ["server.js"]
```
- A) `node`
- B) `server.js`
- C) `node server.js`
- D) Error - cannot have both ENTRYPOINT and CMD

**Answer: C) `node server.js`**
Explanation: When both ENTRYPOINT and CMD are present, CMD provides default arguments to ENTRYPOINT. The final command is `node server.js`.

---

**Q27:** What happens when you run `docker run myimage /bin/sh` with the Dockerfile from Q26?
- A) `node server.js`
- B) `node /bin/sh`
- C) `/bin/sh`
- D) Error

**Answer: B) `node /bin/sh`**
Explanation: The `docker run` argument `/bin/sh` **replaces CMD** but NOT ENTRYPOINT. So the command becomes `node /bin/sh`.

---

**Q28:** What is the primary benefit of multi-stage Docker builds?
- A) Faster container startup times
- B) Smaller final image size by separating build and runtime
- C) Better security through encryption
- D) Automatic scaling support

**Answer: B) Smaller final image size by separating build and runtime**
Explanation: Multi-stage builds let you use a full SDK image for building and copy only the output to a slim runtime image, dramatically reducing the final image size.

---

**Q29:** In this Dockerfile, what is the purpose of `AS build`?
```
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "myapp.dll"]
```
- A) Creates a tag for the final image
- B) Names the build stage for referencing with `COPY --from`
- C) Sets the container hostname
- D) Creates a Docker network alias

**Answer: B) Names the build stage for referencing with `COPY --from`**
Explanation: `AS build` names the stage so later stages can reference it with `COPY --from=build` to copy specific files from that stage.

---

**Q30:** Which Dockerfile instruction should you use to copy local files into the image? (Best practice)
- A) ADD
- B) COPY
- C) RUN cp
- D) VOLUME

**Answer: B) COPY**
Explanation: COPY is the recommended instruction for copying local files. ADD has extra features (URL download, tar extraction) that make it less predictable. Use COPY unless you specifically need ADD's features.

---

**Q31:** What is the difference between shell form and exec form in CMD?
- A) No difference
- B) Shell form runs as PID 1; exec form uses /bin/sh
- C) Exec form runs as PID 1; shell form wraps in /bin/sh -c
- D) Shell form is for Linux; exec form is for Windows

**Answer: C) Exec form runs as PID 1; shell form wraps in /bin/sh -c**
Explanation: Exec form `CMD ["dotnet", "app.dll"]` runs the process directly as PID 1, receiving signals properly. Shell form `CMD dotnet app.dll` wraps in `/bin/sh -c`, which means signals like SIGTERM go to the shell, not the app.

---

**Q32:** To optimize Docker layer caching for a Node.js app, which order of instructions is best?
- A) COPY . . → RUN npm install
- B) COPY package*.json ./ → RUN npm install → COPY . .
- C) RUN npm install → COPY . .
- D) Order doesn't matter

**Answer: B) COPY package*.json ./ → RUN npm install → COPY . .**
Explanation: By copying package.json first and running npm install before copying source code, the npm install layer is cached and only re-runs when dependencies change (not on every code change).

---

#### ACR Advanced

**Q33:** You need to copy an image from Docker Hub to your ACR without installing Docker locally. Which command should you use?
- A) `docker pull` then `docker push`
- B) `az acr import`
- C) `az acr build`
- D) `az acr copy`

**Answer: B) `az acr import`**
Explanation: `az acr import` directly imports images from any Docker V2 compatible registry to your ACR without requiring a local Docker installation.

---

**Q34:** You want to automatically delete untagged images older than 30 days from your ACR. What should you use?
- A) ACR retention policy
- B) ACR purge command via ACR Tasks
- C) Azure Policy
- D) Manual `az acr repository delete`

**Answer: B) ACR purge command via ACR Tasks**
Explanation: The ACR purge command (run via `az acr run`) can be scheduled as an ACR Task to automatically delete images based on age and tag patterns.

---

**Q35:** Which ACR event can trigger a webhook notification?
- A) Image push
- B) Image delete
- C) Chart push
- D) All of the above

**Answer: D) All of the above**
Explanation: ACR webhooks support multiple events: push, delete, quarantine, chart_push, and chart_delete.

---

#### ACI Advanced

**Q36:** A container in ACI becomes unresponsive but doesn't crash. How can you automatically detect and restart it?
- A) Readiness probe
- B) Liveness probe
- C) Startup probe
- D) Health check endpoint

**Answer: B) Liveness probe**
Explanation: Liveness probes detect stuck/deadlocked containers. When the probe fails, ACI restarts the container. Readiness probes only affect load balancing, not restarts.

---

**Q37:** You need to run a database migration before your main application starts in ACI. What feature should you use?
- A) Liveness probe with initial delay
- B) Init containers
- C) Startup probe
- D) Sidecar container

**Answer: B) Init containers**
Explanation: Init containers run sequentially before the main application containers start. They must complete successfully before the app containers begin.

---

**Q38:** How do you store secrets in ACI container instances without exposing them in logs or the portal?
- A) Environment variables with `value`
- B) Environment variables with `secureValue`
- C) Config maps
- D) Azure App Configuration

**Answer: B) Environment variables with `secureValue`**
Explanation: The `secureValue` property (or `--secure-environment-variables` in CLI) prevents values from appearing in logs, Azure portal, or container properties.

---

**Q39:** What is the minimum subnet size required when deploying ACI into a VNet?
- A) /28
- B) /29
- C) /24
- D) The subnet must be delegated to Microsoft.ContainerInstance/containerGroups

**Answer: D) The subnet must be delegated to Microsoft.ContainerInstance/containerGroups**
Explanation: The key requirement is subnet delegation. The subnet must be delegated to `Microsoft.ContainerInstance/containerGroups` and cannot contain other resources.

---

**Q40:** Multi-container groups in ACI are supported on which operating system(s)?
- A) Linux only
- B) Windows only
- C) Both Linux and Windows
- D) Depends on the container image

**Answer: A) Linux only**
Explanation: Multi-container groups (container groups with more than one container) are only supported on Linux. Windows container groups support only a single container.

---

**Q41:** You need to process sensitive healthcare data in ACI with protection against privileged admin access. What should you use?
- A) Private endpoints
- B) Confidential containers
- C) Network security groups
- D) Managed identity

**Answer: B) Confidential containers**
Explanation: Confidential containers run in a hardware-based Trusted Execution Environment (TEE) that encrypts memory and isolates it from the host OS, protecting data from even privileged administrators.

---

#### Container Apps Advanced

**Q42:** Which probe type should you use to prevent Container Apps from sending traffic to a container that is still initializing?
- A) Liveness probe
- B) Readiness probe
- C) Startup probe
- D) Health check

**Answer: C) Startup probe**
Explanation: Startup probes block liveness and readiness probes until the container has fully started. This prevents premature restarts of slow-starting containers.

---

**Q43:** You want to run a compute-intensive container app that requires dedicated resources and won't scale to zero. What should you use?
- A) Consumption workload profile
- B) Dedicated workload profile
- C) Premium Container App plan
- D) AKS instead

**Answer: B) Dedicated workload profile**
Explanation: Dedicated workload profiles (D-series for compute, E-series for memory) provide dedicated VM resources. Unlike Consumption profiles, they don't scale to zero.

---

**Q44:** How do you test a new Container Apps revision before routing production traffic to it?
- A) Deploy to a different environment
- B) Use revision labels to access the new revision directly
- C) Use A/B testing in Azure Front Door
- D) Deploy to a deployment slot

**Answer: B) Use revision labels to access the new revision directly**
Explanation: Revision labels provide a stable URL to access a specific revision directly (e.g., `staging---myapp.<env>.azurecontainerapps.io`), bypassing traffic splitting.

---

**Q45:** Which Container Apps scaling rule type uses KEDA?
- A) HTTP concurrent requests
- B) Azure Queue scaling
- C) Custom metrics
- D) Both B and C

**Answer: D) Both B and C**
Explanation: HTTP scaling is built-in to Container Apps. Azure Queue, Service Bus, and custom metric scalers all use KEDA (Kubernetes Event-Driven Autoscaling) under the hood.

---

**Q46:** What happens when you set `minReplicas: 0` in a Container App?
- A) The app is deleted
- B) The app scales to zero when idle (no traffic/events)
- C) The app always has at least one replica
- D) An error occurs

**Answer: B) The app scales to zero when idle (no traffic/events)**
Explanation: Setting minReplicas to 0 enables scale-to-zero, meaning the app has no running instances when there's no traffic. The first request may experience cold start latency.

---

**Q47:** You need to add built-in authentication to your Container App without code changes. Which feature should you use?
- A) Azure AD B2C integration
- B) Easy Auth (built-in authentication)
- C) Managed Identity
- D) API Management

**Answer: B) Easy Auth (built-in authentication)**
Explanation: Container Apps supports built-in authentication (Easy Auth) similar to App Service. It handles authentication at the platform level without code changes.

---

**Q48:** What is the minimum subnet size for a Container Apps environment with workload profiles?
- A) /27
- B) /24
- C) /23
- D) /16

**Answer: C) /23**
Explanation: Container Apps environments with workload profiles require a minimum /23 subnet. Consumption-only environments need a minimum /27 subnet.

---

**Q49:** How does Dapr service-to-service invocation work in Container Apps?
- A) Direct HTTP calls between containers
- B) Through the Dapr sidecar on localhost:3500
- C) Through Azure Service Bus
- D) Through the Envoy proxy

**Answer: B) Through the Dapr sidecar on localhost:3500**
Explanation: Dapr runs as a sidecar proxy. Service invocation uses `http://localhost:3500/v1.0/invoke/<app-id>/method/<endpoint>`, and Dapr handles service discovery and routing.

---

**Q50:** What storage type in Container Apps persists data across revisions and replica restarts?
- A) EmptyDir (ephemeral)
- B) Azure Files
- C) Container local storage
- D) Dapr state store

**Answer: B) Azure Files**
Explanation: Azure Files mounted as a volume persists data across revisions and replica restarts. EmptyDir is ephemeral and lost when the replica is terminated.

---

**Q51:** In a Container App YAML, how do you reference a secret in an environment variable?
- A) `value: "{{secrets.my-secret}}"`
- B) `secretRef: my-secret`
- C) `valueFrom: { secretKeyRef: my-secret }`
- D) `secret: my-secret`

**Answer: B) `secretRef: my-secret`**
Explanation: In Container Apps YAML, use `secretRef` in environment variable definitions to reference a secret. The secret must be defined in the configuration.secrets section.

---

**Q52:** Which Container App revision mode should you use for canary deployments?
- A) Single revision mode
- B) Multiple revision mode
- C) Rolling update mode
- D) Blue/green mode

**Answer: B) Multiple revision mode**
Explanation: Multiple revision mode allows multiple revisions to run simultaneously with traffic splitting, which is required for canary deployments (e.g., 90/10 split).

---

**Q53:** You created a Container App Job with `--trigger-type Event`. When does the job execute?
- A) On a cron schedule
- B) When manually started
- C) When events arrive (e.g., queue messages)
- D) When the container app scales

**Answer: C) When events arrive (e.g., queue messages)**
Explanation: Event-driven jobs execute when specific events trigger them, such as messages appearing in a queue or events from Event Grid.

---

**Q54:** What is the FQDN format for a Container App with external ingress?
- A) `<app-name>.<region>.azurecontainer.io`
- B) `<app-name>.<unique-id>.<region>.azurecontainerapps.io`
- C) `<app-name>.azurecontainerapps.net`
- D) `<app-name>.<environment-name>.azurecontainerapps.io`

**Answer: B) `<app-name>.<unique-id>.<region>.azurecontainerapps.io`**
Explanation: Container Apps use the format `<app-name>.<unique-id>.<region>.azurecontainerapps.io` for external ingress FQDNs.

---

**Q55:** You need to connect your Container App to an Azure SQL Database using managed identity. What is the easiest approach?
- A) Manually set connection string as a secret
- B) Use a service connector
- C) Configure Dapr binding
- D) Use Azure Functions as a proxy

**Answer: B) Use a service connector**
Explanation: Service connectors simplify connecting to Azure services by automatically configuring connection strings, credentials, and managed identity authentication.

---

## Key CLI Commands Reference

### Docker CLI
```bash
# Build and tag image
docker build -t myapp:v1 .
docker build -t myapp:v1 -f Dockerfile.prod .

# Tag for ACR
docker tag myapp:v1 myregistry.azurecr.io/myapp:v1

# Push/Pull
docker push myregistry.azurecr.io/myapp:v1
docker pull myregistry.azurecr.io/myapp:v1

# Run containers
docker run -d -p 8080:80 --name mycontainer myapp:v1
docker run -it --rm myapp:v1 /bin/sh
docker run -e "ENV=prod" -v /host:/container myapp:v1

# Management
docker ps -a
docker logs mycontainer
docker exec -it mycontainer /bin/sh
docker stop mycontainer && docker rm mycontainer
docker system prune
```

### Azure Container Registry
```bash
# Create registry
az acr create --name myregistry --resource-group myRG --sku Standard

# Login to registry
az acr login --name myregistry

# Build image in ACR (no local Docker needed)
az acr build --registry myregistry --image myapp:v1 .

# Import image from Docker Hub
az acr import --name myregistry --source docker.io/library/nginx:latest --image nginx:latest

# Import from another ACR
az acr import --name myregistry --source other.azurecr.io/myapp:v1 --image myapp:v1

# List repositories and tags
az acr repository list --name myregistry
az acr repository show-tags --name myregistry --repository myapp

# Create triggered task
az acr task create --registry myregistry --name mytask --image myapp:{{.Run.ID}} \
  --context https://github.com/user/repo.git --branch main --file Dockerfile \
  --git-access-token <PAT>

# Run task manually
az acr task run --registry myregistry --name mytask

# Purge old images
az acr run --cmd "acr purge --filter 'myapp:.*' --ago 30d --untagged" \
  --registry myregistry /dev/null

# Create webhook
az acr webhook create --registry myregistry --name mywebhook \
  --uri https://myapp.azurewebsites.net/api/webhook --actions push delete

# Manage network rules
az acr update --name myregistry --public-network-enabled false
az acr network-rule add --name myregistry --ip-address 203.0.113.0/24

# Enable admin account
az acr update --name myregistry --admin-enabled true
az acr credential show --name myregistry

# Geo-replication (Premium)
az acr replication create --registry myregistry --location westus
```

### Azure Container Instances
```bash
# Create container
az container create --resource-group myRG --name mycontainer \
  --image myapp:v1 --ports 80 --dns-name-label myapp

# Create with restart policy
az container create --resource-group myRG --name mycontainer \
  --image myapp:v1 --restart-policy OnFailure

# Create with managed identity
az container create --resource-group myRG --name mycontainer \
  --image myapp:v1 --assign-identity

# Create from ACR with managed identity
az container create --resource-group myRG --name mycontainer \
  --image myregistry.azurecr.io/myapp:v1 --acr-identity [system] --assign-identity

# Create in VNet
az container create --resource-group myRG --name mycontainer \
  --image myapp:v1 --vnet myVnet --subnet aciSubnet --ip-address private

# Deploy from YAML
az container create --resource-group myRG --file container-group.yaml

# View logs
az container logs --resource-group myRG --name mycontainer
az container logs --resource-group myRG --name mygroup --container-name myapp

# Execute command / attach
az container exec --resource-group myRG --name mycontainer --exec-command "/bin/sh"
az container attach --resource-group myRG --name mycontainer

# Stop/Start/Restart/Delete
az container stop --resource-group myRG --name mycontainer
az container start --resource-group myRG --name mycontainer
az container restart --resource-group myRG --name mycontainer
az container delete --resource-group myRG --name mycontainer --yes
```

### Azure Container Apps
```bash
# Create environment
az containerapp env create --name myenv --resource-group myRG --location eastus

# Create environment with workload profiles
az containerapp env create --name myenv --resource-group myRG \
  --location eastus --enable-workload-profiles

# Add workload profile
az containerapp env workload-profile add --name myenv --resource-group myRG \
  --workload-profile-name my-d4 --workload-profile-type D4 \
  --min-nodes 1 --max-nodes 3

# Create app
az containerapp create --name myapp --resource-group myRG --environment myenv \
  --image myapp:v1 --target-port 80 --ingress external

# Create with managed identity pulling from ACR
az containerapp create --name myapp --resource-group myRG --environment myenv \
  --image myregistry.azurecr.io/myapp:v1 --registry-server myregistry.azurecr.io \
  --registry-identity system --target-port 80 --ingress external

# Update with scaling
az containerapp update --name myapp --resource-group myRG \
  --min-replicas 1 --max-replicas 10

# Set traffic split
az containerapp ingress traffic set --name myapp --resource-group myRG \
  --revision-weight myapp--v1=80 myapp--v2=20

# Manage secrets
az containerapp secret set --name myapp --resource-group myRG \
  --secrets "my-secret=secretvalue"

# List revisions
az containerapp revision list --name myapp --resource-group myRG

# Revision labels
az containerapp revision label add --name myapp --resource-group myRG \
  --revision myapp--abc123 --label staging

# Set revision mode
az containerapp revision set-mode --name myapp --resource-group myRG --mode multiple

# Identity management
az containerapp identity assign --name myapp --resource-group myRG --system-assigned

# View logs
az containerapp logs show --name myapp --resource-group myRG --type console
az containerapp logs show --name myapp --resource-group myRG --type console --follow

# Custom domains
az containerapp hostname add --name myapp --resource-group myRG --hostname www.mydomain.com

# Container App Jobs
az containerapp job create --name myjob --resource-group myRG --environment myenv \
  --trigger-type Schedule --cron-expression "0 0 * * *" \
  --image myregistry.azurecr.io/myjob:v1 --cpu 0.5 --memory 1Gi

# Service connectors
az containerapp connection create sql --name myapp --resource-group myRG \
  --target-resource-group myRG --server myserver --database mydb --system-identity

# Storage
az containerapp env storage set --name myenv --resource-group myRG \
  --storage-name mystorage --azure-file-account-name myaccount \
  --azure-file-account-key <key> --azure-file-share-name myshare --access-mode ReadWrite

# Dapr
az containerapp create --name myapp --resource-group myRG --environment myenv \
  --image myapp:v1 --enable-dapr --dapr-app-id myapp --dapr-app-port 3000
```

---

## Exam Gotchas & Tricky Distinctions

| # | Common Misconception | Reality |
|---|---------------------|---------|
| 1 | "CMD and ENTRYPOINT do the same thing" | ❌ CMD provides default arguments (easily overridden at `docker run`). ENTRYPOINT sets the executable (requires `--entrypoint` to override). When both are present, CMD args are appended to ENTRYPOINT. |
| 2 | "COPY and ADD are interchangeable" | ❌ ADD supports URLs and auto-extracts tar archives. COPY is simpler and recommended for most cases. Use COPY unless you need ADD's extra features. |
| 3 | "Shell form and exec form produce the same result" | ❌ Shell form (`CMD dotnet app.dll`) wraps in `/bin/sh -c`, so PID 1 is the shell (signals like SIGTERM don't reach the app). Exec form (`CMD ["dotnet", "app.dll"]`) runs the process as PID 1 directly. |
| 4 | "Multi-stage builds result in multiple images" | ❌ Only the LAST `FROM` stage becomes the final image. Previous stages are discarded (unless referenced with `COPY --from`). |
| 5 | "EXPOSE actually publishes ports" | ❌ EXPOSE is **documentation only**. You still need `-p` at `docker run` or `ports` in YAML to actually publish ports. |
| 6 | "ACR Basic and Standard support geo-replication" | ❌ Geo-replication, content trust, private link, customer-managed keys, and repository-scoped tokens are **Premium only**. |
| 7 | "ACR admin account is recommended for production" | ❌ Admin account is for dev/test only. Production should use Azure AD with RBAC (service principals, managed identities). |
| 8 | "`az acr build` requires Docker installed locally" | ❌ `az acr build` performs the build entirely in the cloud. No local Docker needed. Similarly, `az acr import` moves images between registries without local Docker. |
| 9 | "ACI container groups support Windows multi-container" | ❌ Multi-container groups are **Linux only**. Windows supports only a single container per group. |
| 10 | "ACI containers can be updated after creation" | ❌ Container group properties (image, resources, ports) cannot be modified after creation. You must delete and recreate. You CAN restart/stop/start. |
| 11 | "ACI has built-in load balancing across multiple instances" | ❌ ACI has no built-in load balancer or auto-scaling. Each container group has its own IP. Use Application Gateway or Azure Front Door for load balancing. |
| 12 | "ACI restart policy 'OnFailure' restarts on exit code 0" | ❌ OnFailure only restarts on non-zero exit codes. Exit code 0 = success = stop. Always restarts on any exit. Never never restarts. |
| 13 | "Liveness and readiness probes do the same thing" | ❌ Liveness probe failure → **restart** container. Readiness probe failure → **remove from load balancer** (no restart). Startup probe → blocks other probes until app is ready. |
| 14 | "Container Apps and ACI are the same thing" | ❌ ACI = simple, no orchestration, no auto-scaling. Container Apps = serverless orchestration with auto-scaling (KEDA), revisions, traffic splitting, Dapr, scale-to-zero. |
| 15 | "Container Apps require managing Kubernetes" | ❌ Container Apps abstract Kubernetes entirely. Built on AKS/KEDA/Dapr/Envoy, but you never interact with Kubernetes directly. |
| 16 | "Container Apps single revision mode supports traffic splitting" | ❌ Single revision mode replaces the old revision entirely. You need **multiple revision mode** for traffic splitting and canary deployments. |
| 17 | "Container Apps Consumption profile cannot scale to zero" | ❌ Consumption workload profiles **can** scale to zero. Dedicated profiles cannot (minimum node count). |
| 18 | "Container Apps scale based on CPU/memory by default" | ❌ Default scaling is based on **HTTP concurrent requests**. CPU/memory scaling requires custom KEDA rules. |
| 19 | "Dapr is required for Container Apps" | ❌ Dapr is optional. You enable it per-app when you need its building blocks (service invocation, state management, pub/sub, etc.). |
| 20 | "Container Apps secrets are stored in Key Vault" | ❌ Container Apps has its own secrets management. Secrets are stored in the app configuration. You CAN reference Key Vault secrets, but it's not the default. |
| 21 | "ACI DNS label format is `<name>.azurecontainer.io`" | ❌ Correct format: `<dns-label>.<region>.azurecontainer.io`. The region is included. |
| 22 | "Container Apps FQDN includes the environment name" | ❌ Format: `<app-name>.<unique-id>.<region>.azurecontainerapps.io`. It uses a unique ID, not the environment name. |
| 23 | "`az acr import` and `docker pull/push` are equivalent" | ❌ `az acr import` is server-side (no bandwidth through your machine). `docker pull/push` transfers through your local machine. Import is faster and doesn't need Docker. |
| 24 | "Container App revision labels replace traffic splitting" | ❌ Labels provide direct access to a specific revision (for testing). Traffic splitting controls how production traffic is distributed. They serve different purposes. |
| 25 | "ACI and Container Apps both support Dapr" | ❌ Only Container Apps has built-in Dapr integration. ACI does not. |

---

## Study Tips for AZ-204 Exam

1. **Master Dockerfile instructions** — CMD vs ENTRYPOINT, COPY vs ADD, shell vs exec form are exam favorites
2. **Memorize multi-stage build pattern** — Know how `AS` and `COPY --from` work together
3. **Know ACR tiers cold** — Premium-only features (geo-replication, private link, content trust, CMK, tokens)
4. **Understand `az acr import` vs `az acr build`** — Import copies between registries; build compiles from source
5. **ACI container groups** — Linux-only for multi-container, shared localhost, cannot update after creation
6. **ACI restart policies** — Always (default), OnFailure (non-zero exit), Never (one-time)
7. **ACI probes** — Liveness (restart on failure), readiness (remove from LB), startup (delay other probes)
8. **Container Apps hierarchy** — Environment → App → Revision → Replica → Container
9. **Container Apps scaling** — HTTP (built-in), KEDA (queues, service bus, custom), scale-to-zero on Consumption
10. **Revision modes** — Single (auto-replace) vs Multiple (traffic splitting, canary)
11. **Workload profiles** — Consumption (scale-to-zero) vs Dedicated (guaranteed resources, D/E series)
12. **Container Apps networking** — /23 minimum for workload profiles, /27 for consumption-only
13. **Dapr is optional** — Enable per-app, uses sidecar on localhost:3500 for service invocation
14. **Revision labels** — For direct access to test revisions before sending traffic
15. **Authentication** — ACR uses Azure AD/RBAC, Container Apps supports Easy Auth, ACI uses managed identity

---

## Important URLs and References

- Azure Container Registry: https://learn.microsoft.com/en-us/azure/container-registry/
- Azure Container Instances: https://learn.microsoft.com/en-us/azure/container-instances/
- Azure Container Apps: https://learn.microsoft.com/en-us/azure/container-apps/
- ACR Tasks: https://learn.microsoft.com/en-us/azure/container-registry/container-registry-tasks-overview
- KEDA Scalers: https://keda.sh/docs/scalers/
- Dockerfile Reference: https://docs.docker.com/reference/dockerfile/
- Container Apps Workload Profiles: https://learn.microsoft.com/en-us/azure/container-apps/workload-profiles-overview
- ACI Container Groups: https://learn.microsoft.com/en-us/azure/container-instances/container-instances-container-groups

---

## Quick Reference Summary

### Dockerfile Key Instructions
- **FROM**: Base image (required first instruction)
- **RUN**: Execute during build (creates layer)
- **COPY**: Copy local files (preferred over ADD)
- **ADD**: Copy with URL/tar support (use COPY instead when possible)
- **CMD**: Default command (overridden by `docker run` args)
- **ENTRYPOINT**: Fixed executable (CMD becomes arguments)
- **ENV/ARG**: Runtime env vars / Build-time vars
- **EXPOSE**: Document ports (doesn't actually publish)
- **WORKDIR**: Set working directory
- **HEALTHCHECK**: Container health check
- **USER**: Run as non-root user

### Multi-Stage Build Pattern
```
FROM sdk AS build → COPY/RUN build → FROM runtime → COPY --from=build → ENTRYPOINT
```

### ACR Service Tiers
- **Basic**: Dev/test, 10 GB storage, 2 webhooks
- **Standard**: Production, 100 GB storage, 10 webhooks
- **Premium**: Enterprise, 500 GB, geo-replication, private link, content trust, CMK, tokens, 500 webhooks

### ACR Authentication
- **Azure AD + RBAC**: Recommended for production (AcrPull, AcrPush, AcrDelete roles)
- **Admin Account**: Simple but not recommended for production
- **Service Principal**: For automated CI/CD pipelines
- **Managed Identity**: For Azure services (ACI, AKS, Container Apps)
- **Repository Tokens**: Fine-grained access (Premium only)

### ACR Tasks
- **Quick Task** (`az acr build`): On-demand cloud build
- **Triggered Task**: On git commit, base image update, or schedule
- **Multi-step Task**: YAML-defined build/test/push workflow

### ACI Key Facts
- **Restart Policies**: Always (default, long-running), OnFailure (batch), Never (one-time)
- **Multi-container**: Linux only, share localhost/storage/lifecycle
- **Volumes**: Azure Files (persistent), secret, emptyDir, gitRepo
- **Max Resources**: 4 CPU, 16 GB memory per container
- **Probes**: Liveness (restart), readiness (LB removal)
- **Cannot update** container groups after creation

### Container Apps Concepts
- **Environment**: Shared boundary, networking, logging (Log Analytics)
- **Revision**: Immutable snapshot, traffic-splittable
- **Replica**: Running instance, auto-scaled (including to zero)
- **Ingress**: External (internet) or Internal (environment only), HTTPS by default
- **Jobs**: Manual, scheduled (cron), event-driven

### Container Apps Scaling
- **minReplicas**: Minimum (can be 0 for scale-to-zero)
- **maxReplicas**: Maximum limit
- **HTTP**: Built-in concurrent requests trigger
- **KEDA**: Event-driven (queues, service bus, custom metrics)
- **Workload Profiles**: Consumption (serverless) or Dedicated (D/E series VMs)

### Container Apps Networking
- **Consumption-only environment**: /27 minimum subnet
- **Workload profiles environment**: /23 minimum subnet
- **External**: Internet accessible
- **Internal**: VNet only
- **FQDN**: `<app>.<unique-id>.<region>.azurecontainerapps.io`

### Key Comparisons
| Feature | ACI | Container Apps | AKS |
|---------|-----|---------------|-----|
| Auto-scaling | ❌ | ✅ (KEDA) | ✅ (HPA) |
| Scale to zero | ❌ | ✅ | ❌ |
| Traffic splitting | ❌ | ✅ | Manual |
| Dapr support | ❌ | ✅ Built-in | Manual install |
| Kubernetes access | ❌ | Abstracted | Full |
| Management overhead | Low | Low | High |
| Multi-container | ✅ (Linux) | ✅ (sidecar) | ✅ (pods) |
| Probes | Liveness, Readiness | All 3 types | All 3 types |
| Best for | Simple tasks, batch | Microservices, APIs | Full orchestration |

