# Resource Provisioning — Functional Requirements

## 1. Purpose

This document defines the functional requirements for Azure resource provisioning provided by the AI + MCP Platform.

The scope of this document is limited to **resource creation and provisioning**. Security controls, approval rules, audit, and governance details are defined separately.

Related document:

- [AI + MCP Platform Team — Security & Governance Design](./ai-mcp-platform-team-security-governance.md)

---

## 2. Scope

The platform must allow users and AI Agents to request Azure resources through MCP / Platform API without requiring direct manual configuration in the Azure Portal.

Target capabilities include:

- Create a single Azure resource
- Create multiple related Azure resources
- Add resources to an existing Service
- Create resources for a new Service
- Create resources per Environment
- Generate reproducible IaC
- Estimate provisioning results before execution
- Track provisioning status and results

---

## 3. Basic Flow

```text
User
  ↓
AI Agent
  ↓
MCP Tool
  ↓
Platform API
  ↓
Resource Definition
  ↓
IaC Generation / Selection
  ↓
Provisioning
  ↓
Azure
  ↓
Result / Status
```

---

## 4. Functional Requirements

### FR-01 Single Resource Creation

The platform must support creation of a single Azure resource.

Example resources:

- App Service
- App Service Plan
- Azure SQL Database
- Storage Account
- Key Vault
- Azure Functions
- Application Insights
- Log Analytics Workspace
- Private Endpoint

Required inputs should be limited to values that cannot be determined automatically.

Example:

```text
resource_type
service
resource_name
region
environment
sku
```

---

### FR-02 Multiple Resource Creation

The platform must support provisioning multiple related Azure resources in a single request.

Example:

```text
Web Application
 ├─ App Service Plan
 ├─ App Service
 ├─ Azure SQL Database
 ├─ Storage Account
 ├─ Key Vault
 └─ Application Insights
```

The platform must return one provisioning result representing the entire operation.

---

### FR-03 Service Association

Each created resource must be associated with a Service.

The requester must be able to choose either:

1. Create a new Service
2. Add resources to an existing Service

Example model:

```text
Service
 ├─ Application
 ├─ Database
 ├─ Storage
 ├─ Network
 ├─ Identity
 └─ Monitoring
```

---

### FR-04 Environment Selection

Resources must be created for a defined Environment.

Supported examples:

- dev
- staging
- prod

The platform must be able to use Environment-specific defaults such as SKU, naming, configuration, and deployment target.

---

### FR-05 Standard Templates / Golden Paths

The platform must support predefined resource templates.

Examples:

- Web Application
- SPA + API
- API Service
- Function
- Batch / Worker
- Database-backed Application

Users should specify only the required business parameters while the platform fills in standard infrastructure settings automatically.

---

### FR-06 Resource Parameter Handling

The platform must support configurable parameters for each resource type.

Examples:

```text
region
sku
capacity
runtime
storage_size
database_size
network_mode
backup_option
```

Each resource type must define:

- Required parameters
- Optional parameters
- Default values
- Allowed values

---

### FR-07 Automatic Naming

The platform must generate resource names automatically based on standardized naming rules.

Example inputs:

```text
service = procurement
environment = dev
resource_type = app-service
```

Example output:

```text
app-procurement-dev-jpe-001
```

The requester should be able to provide a logical name without constructing the full Azure resource name manually.

---

### FR-08 Dependency Resolution

The platform must resolve dependencies between resources automatically.

Example:

```text
App Service
  ↓ requires
App Service Plan

Private Endpoint
  ↓ requires
Target Resource + VNet/Subnet
```

Dependent resources must be created in the correct order.

---

### FR-09 Existing Resource Reference

When provisioning a new resource, the platform must support referencing existing resources.

Examples:

- Existing VNet
- Existing Subnet
- Existing Key Vault
- Existing Log Analytics Workspace
- Existing App Service Plan
- Existing Storage Account

The requester should be able to select an existing resource instead of creating a duplicate resource.

---

### FR-10 IaC Generation

Resource provisioning must be reproducible through Infrastructure as Code.

Primary format:

- Bicep

The platform must generate or select IaC definitions corresponding to the requested resource configuration.

Generated IaC should be storable in the related repository.

---

### FR-11 Plan Generation

Before provisioning, the platform must be able to generate a resource plan.

The plan should contain at least:

- Resources to create
- Resource types
- Resource names
- Region
- SKU
- Environment
- Dependencies
- Existing resources referenced
- Estimated cost when available

Example:

```text
Create:
- App Service Plan: asp-procurement-dev-jpe-001
- App Service: app-procurement-dev-jpe-001
- Azure SQL Database: sqldb-procurement-dev-jpe-001
```

---

### FR-12 Provisioning Execution

The platform must execute the approved resource plan and create the required Azure resources.

The provisioning operation must have a unique operation ID.

Example:

```text
operation_id: op-20260826-000123
status: provisioning
```

---

### FR-13 Provisioning Status

The platform must expose provisioning status.

Supported states should include at least:

```text
planned
queued
provisioning
succeeded
partially_succeeded
failed
cancelled
```

The requester must be able to query the current state using the operation ID.

---

### FR-14 Provisioning Result

After provisioning completes, the platform must return the created resource information.

Example output:

```json
{
  "service": "procurement",
  "environment": "dev",
  "operation_id": "op-20260826-000123",
  "status": "succeeded",
  "resources": [
    {
      "type": "Microsoft.Web/sites",
      "name": "app-procurement-dev-jpe-001",
      "resource_id": "/subscriptions/..."
    }
  ]
}
```

---

### FR-15 Failure Handling

When resource creation fails, the platform must return:

- Failed resource
- Failure reason
- Current provisioning state
- Resources already created
- Whether retry is possible

The platform must support retrying a failed provisioning operation where applicable.

---

### FR-16 Idempotent Provisioning

Repeating the same provisioning request must not unintentionally create duplicate resources.

The platform should identify provisioning operations using identifiers such as:

```text
request_id
operation_id
service_id
environment
```

---

### FR-17 Resource Discovery

Before creating resources, the platform must be able to retrieve existing resources related to the target Service or Environment.

Example use cases:

- Determine whether an App Service Plan already exists
- Select an existing VNet/Subnet
- Avoid duplicate Storage Accounts
- Attach a new resource to an existing Service

---

### FR-18 Cost Estimation

The platform should provide an estimated monthly cost before resource creation when pricing information is available.

Example:

```text
App Service Plan P0v3: estimated monthly cost
Azure SQL Database: estimated monthly cost
Storage Account: usage-based estimate

Estimated total: xxx JPY / month
```

This estimate is informational and may differ from actual billing.

---

### FR-19 MCP Tool Interface

Resource provisioning should be exposed through high-level MCP tools rather than resource-specific low-level commands where possible.

Recommended tools:

```text
plan_resource
create_resource
create_resources
get_resource_plan
get_provisioning_status
get_provisioning_result
list_service_resources
estimate_resource_cost
```

Example:

```text
create_resource(
  service="procurement",
  environment="dev",
  resource_type="azure-sql-database"
)
```

---

## 5. Initial Supported Resource Types

Initial implementation should prioritize commonly used resources.

### Application

- App Service Plan
- App Service
- Azure Functions

### Data

- Azure SQL Server
- Azure SQL Database
- Storage Account

### Security / Configuration

- Key Vault

### Monitoring

- Application Insights
- Log Analytics Workspace

### Network

- Private Endpoint
- Private DNS Zone association

Additional resource types can be added incrementally through the same resource definition model.

---

## 6. Resource Definition Model

Each supported resource should be represented using a common definition model.

Example:

```yaml
resource_type: app-service
service: procurement
environment: dev
region: japaneast
parameters:
  runtime: node
  runtime_version: "24"
  sku: P0v3
references:
  app_service_plan: existing-or-create
```

This common model should be translated by the Platform into the required Azure / Bicep representation.

---

## 7. Out of Scope

The following are outside the scope of this document:

- Resource update
- Resource deletion
- Day 2 Operations
- Deployment of application code
- CI/CD pipeline creation
- Repository creation
- Approval design
- Authentication / authorization
- Security policy
- Audit design
- Compliance controls

These functions are defined in separate Platform requirements and governance documents.

---

## 8. Target User Experience

Example:

```text
User:
「ProcurementのDev環境にAzure SQL Databaseを追加して」

AI Agent:
1. Service / Environmentを確認
2. 既存Resourceを確認
3. Resource Definitionを生成
4. Resource Planを生成
5. Cost Estimateを表示
6. PlatformへProvisioning要求
7. Provisioning Statusを追跡
8. 作成結果を返却
```

The goal is for users to request the required infrastructure by intent, while the Platform translates that intent into standardized Azure resource provisioning.
