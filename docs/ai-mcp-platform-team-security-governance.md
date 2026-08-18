# AI + MCP Platform Team — Security & Governance Design

## 1. 目的

AI Agent と MCP を利用し、Team Topologies における Platform Team の役割をセルフサービス化する。

単に AI に Azure の操作権限を与えるのではなく、組織が承認した Golden Path、Policy、Approval、最小権限、監査を通してのみクラウドを変更できる Internal Developer Platform を構築する。

基本原則は以下とする。

> AI は変更を要求・計画する。Platform が可否を判断する。Executor だけが実環境を変更する。

---

## 2. 基本アーキテクチャ

```text
User
  ↓
AI Agent
  ↓
Policy / Guardrails
  ↓
MCP Gateway
  ↓
Platform API
  ├─ Validation
  ├─ Risk Evaluation
  ├─ Cost Check
  └─ Approval
  ↓
Executor
  ↓
Azure / Azure DevOps

全操作 → Audit Log / Monitoring
```

Agent から Azure CLI、Azure API、Azure DevOps などを無制限に直接操作する経路は原則提供しない。

---

## 3. Platform Team として提供する主要機能

### 3.1 Resource Provisioning

- 単体 Azure リソース作成
- 複数 Azure リソース一括作成
- 既存リソースへの追加
- リソース変更
- リソース停止・削除
- Bicep 等の IaC による再現可能な構築

例：

- App Service
- Azure SQL
- Storage Account
- Key Vault
- Functions
- Application Insights
- Private Endpoint

### 3.2 Service Management

リソース単位だけではなく、サービスを上位概念として管理する。

リソース作成時に以下を選択可能とする。

1. 新規親 Service を作成
2. 既存 Service を選択

```text
Service
 ├─ Application
 ├─ Database
 ├─ Storage
 ├─ Network
 ├─ Identity
 ├─ Monitoring
 └─ CI/CD
```

Service を単位として Owner、Environment、Cost、Security Policy、リソース群を関連付ける。

### 3.3 Golden Path / Service Catalog

組織が承認した標準構成を Catalog として提供する。

例：

- Web Application
- SPA + API
- API Service
- Batch / Worker
- Function
- Database付きApplication

ユーザーは Azure の詳細構成を毎回設計せず、標準構成と必要なパラメータだけを選択する。

### 3.4 Environment Management

- Dev / Staging / Production 作成
- 環境複製
- 一時環境作成
- TTLによる自動削除
- Environment単位のPolicy適用

### 3.5 Repository / CI/CD Automation

- Repository作成
- 標準ディレクトリ生成
- Bicep配置
- Pipeline生成
- Branch Policy設定
- Security Scan組込み
- Deployment設定

### 3.6 Day 2 Operations

構築後の運用もPlatform機能として提供する。

- Scale Up / Down
- Scale Out / In
- Configuration変更
- Deployment
- Restart
- Log確認
- 障害診断
- 証明書・Secret更新
- Drift Detection
- Compliance Check
- Rollback

---

## 4. セキュリティ設計

AI Agent は信頼境界の外側に存在するものとして扱う。

```text
Agent → Policy / Approval → MCP → Restricted Platform API → Executor → Cloud
```

Agent、MCP、Platform API のいずれかが想定外の動作をしても、次のレイヤーで被害を制限できる多層防御を採用する。

### 4.1 Tool Allowlist

Agent に任意の Azure 操作を許可しない。

公開する MCP Tool は Platform Team が承認したものだけとする。

例：

```text
create_resource()
create_service()
create_environment()
deploy_service()
scale_service()
estimate_cost()
delete_environment()
```

汎用 Shell、任意 Azure CLI、任意 REST API 実行などは原則公開しない。

### 4.2 Parameter Allowlist / Validation

Tool が許可されていても任意パラメータを受け付けない。

例：

```yaml
environment:
  - dev
  - staging
  - prod

allowed_regions:
  - japaneast
  - japanwest

app_service_sku:
  - B1
  - P0v3
  - P1v3
```

以下をサーバー側で検証する。

- Resource Type
- SKU
- Region
- Naming Convention
- Tags
- Network設定
- Public Access
- Environment
- Owner
- Cost上限

AIによる入力値を信用せず、Platform API側で必ず再検証する。

### 4.3 Least Privilege

AI Agent 自体には Azure Contributor / Owner 権限を付与しない。

```text
AI Agent
   ↓
MCP
   ↓
Platform API
   ↓
Executor Managed Identity
   ↓
Azure RBAC
```

Executor の Managed Identity も必要な Resource Group、Resource Type、Action のみに制限する。

可能であれば環境ごとに Identity を分離する。

```text
Dev Executor MI
  └─ Dev Resource Groups only

Prod Executor MI
  └─ Production Resource Groups only
```

### 4.4 Short-lived Credential

長期間有効な Client Secret や Personal Access Token を Agent に保持させない。

優先順位：

1. Managed Identity
2. Workload Identity / Federation
3. 短命Token
4. Secret（必要な場合のみ）

### 4.5 Sandbox

AIが生成したコード・IaC・設定は直接Productionで実行しない。

```text
Generate
  ↓
Sandbox
  ↓
Validate
  ↓
Lint / Security Scan
  ↓
Bicep What-if
  ↓
Policy Check
  ↓
Approval
  ↓
Deploy
```

Sandboxから不要なInternetアクセスや社内ネットワークへのアクセスを許可しない。

### 4.6 Network Restriction

- Platform API は必要に応じPrivate化
- ExecutorからのOutboundを制限
- Private Endpointを標準化
- Public Network AccessをPolicyで制御
- 許可されたEndpointのみ通信可能にする

AIのプロンプトや生成コードによってネットワーク境界を変更できない設計とする。

---

## 5. Policy / Approval

すべての操作を同じ承認フローにしない。

操作内容・対象環境・影響範囲からRisk Levelを算出する。

| 操作 | 推奨制御 |
|---|---|
| Devリソース作成 | 自動承認 |
| Dev設定変更 | 自動またはOwner承認 |
| Dev削除 | Owner承認 |
| Production作成 | Owner承認 |
| Production変更 | Owner承認 |
| Production削除 | Owner + Platform Admin承認 |
| RBAC変更 | Platform Admin承認 |
| Public Access有効化 | 原則Deny / Security承認 |
| 高額SKU | Cost/Owner承認 |
| Policy変更 | Platform Admin承認 |

重要なのは「AIが安全だと判断したか」ではなく、Platform側の決定論的PolicyでRiskを判定することである。

---

## 6. Plan Before Apply

変更系Toolは原則として即時実行しない。

```text
Request
 ↓
Generate Plan
 ↓
Validate
 ↓
What-if
 ↓
Risk Evaluation
 ↓
Cost Estimate
 ↓
Approval
 ↓
Apply
 ↓
Verify
```

Planには最低限以下を含める。

- 作成リソース
- 更新リソース
- 削除リソース
- RBAC変更
- Network変更
- Public Exposure変更
- Cost影響
- Policy違反
- Deployment対象Environment

特に Delete、RBAC、Network、Production はPlanとApplyを分離する。

---

## 7. Governance

### 7.1 Naming / Tagging

Platformから作成されるリソースには標準Naming ConventionとTagを強制する。

推奨Tag：

```text
service-id
owner
team
environment
cost-center
managed-by
created-by
created-at
expires-at
```

### 7.2 Azure Policy

Platform APIだけに制御を依存せずAzure側にもPolicyを設定する。

例：

- 許可Region制限
- Public Access制限
- 必須Tag
- 許可SKU
- Diagnostic Settings
- TLS設定
- Private Endpoint

Platform層が突破された場合でもAzure側で拒否できる構造にする。

### 7.3 Cost Governance

- 作成前Cost Estimate
- SKU Allowlist
- Service単位Budget
- Environment単位Budget
- 高額変更時Approval
- 未使用Resource検出
- Temporary Environment TTL

### 7.4 Ownership

すべてのServiceにOwnerを必須とする。

Owner不明のリソースを作らない。

```text
Service
 ├─ Owner
 ├─ Team
 ├─ Cost Center
 ├─ Environment
 └─ Resources
```

---

## 8. Audit / Observability

AI Agent経由の操作は通常のユーザー操作以上に詳細な監査ログを残す。

最低限記録する項目：

- User
- Agent / Agent Version
- Session / Correlation ID
- MCP Tool
- Input Parameters
- Requested Operation
- Generated Plan
- Policy Decision
- Risk Level
- Approval者
- Executor Identity
- Azure Operation
- Result
- Timestamp

機密情報、Token、Secret、不要なPrompt内容そのものはログに残さない。

重要操作については「誰が依頼し、AIが何を要求し、Policyが何を判断し、誰が承認し、最終的に何がAzureへ適用されたか」を追跡可能にする。

---

## 9. Destructive Operation Protection

削除系は通常の更新操作より強く保護する。

- Production削除は複数承認
- Resource Lock確認
- Dependency確認
- What-if必須
- Backup確認
- Soft Delete利用
- Service単位の削除Plan生成
- 削除対象数の上限
- 一括削除Rate Limit

Agentに `delete anything matching ...` のような無制限削除機能を提供しない。

---

## 10. MCP Tool Design

MCP Toolは低レベルAzure APIのラッパーではなく、Platform Capabilityとして設計する。

推奨：

```text
create_service
add_resource_to_service
create_environment
deploy_service
update_service
scale_service
estimate_service_cost
validate_service
check_compliance
get_service_status
delete_environment
```

避ける：

```text
execute_shell
execute_az_cli
call_any_azure_api
execute_arbitrary_script
```

Agentが自由に「方法」を決めるのではなく、Platformが提供する安全な「能力」をAgentが選択する構造にする。

---

## 11. Defense in Depth

```text
User Identity
     ↓
Agent Guardrail
     ↓
Tool Allowlist
     ↓
Parameter Validation
     ↓
Policy Engine
     ↓
Risk Evaluation
     ↓
Human Approval
     ↓
Executor RBAC
     ↓
Azure Policy
     ↓
Resource Lock
     ↓
Audit / Detection
```

MCP自体をSecurity Boundaryとして信用しない。

各レイヤーが突破される可能性を前提に、次のレイヤーで操作を制限する。

---

## 12. Platform Team代替としての到達点

このPlatformの目的は単なるInfrastructure Automationではない。

Stream-aligned TeamがPlatform Teamへ依頼していた以下の作業を、セルフサービスとして提供することを目標とする。

- Infrastructure Provisioning
- Application Bootstrap
- Golden Path提供
- Repository作成
- CI/CD構築
- Security標準化
- Network標準化
- Identity / RBAC
- Monitoring
- Cost Management
- Environment Management
- Compliance
- Day 2 Operations
- Troubleshooting支援

最終的な利用イメージ：

```text
User:
「新しい社内Web APIを作成して」

AI Agent:
1. 要件確認
2. 既存Service選択 / 新規Service作成
3. Golden Path選択
4. Resource Plan生成
5. Cost Estimate
6. Security / Policy Check
7. Bicep What-if
8. 必要なApproval取得
9. Repository / IaC / CI/CD生成
10. ExecutorへDeployment要求
11. Deployment検証
12. Audit Log記録
```

これにより、Platform Teamの知識・標準・ガバナンスをコードとAPIとして提供し、AI Agentをそのセルフサービスインターフェースとして利用する。

---

## 13. 最重要設計原則

1. **AgentにCloud管理権限を直接持たせない**
2. **MCP ToolをAllowlist化する**
3. **任意コマンド実行を避ける**
4. **Platform APIで入力値を必ず再検証する**
5. **PlanとApplyを分離する**
6. **Production・Delete・RBAC・Network変更はHuman Approvalを要求する**
7. **Managed Identityと最小権限を使用する**
8. **Azure Policyでも再度制御する**
9. **すべての変更を監査可能にする**
10. **AIの判断ではなく決定論的Policyを最終的なSecurity Boundaryにする**

---

## 14. 推奨MVP

最初から全機能を実装せず、以下をMVPとする。

### Phase 1

- Service作成 / 選択
- 単体Resource作成
- 複数Resource作成
- Golden Path
- Bicep生成
- Parameter Validation
- Tool Allowlist
- What-if
- Managed Identity Executor
- Audit Log

### Phase 2

- Risk Evaluation
- Human Approval
- Cost Guardrail
- Azure Policy連携
- CI/CD / Repository自動生成
- Environment Management

### Phase 3

- Drift Detection
- Compliance Automation
- Day 2 Operations
- Troubleshooting Agent
- Rollback
- Service Catalog拡充
- Platform利用状況・品質メトリクス

この順序で実装することで、まず安全なProvisioning基盤を確立し、その上にPlatform Teamとしての運用能力を段階的に追加する。