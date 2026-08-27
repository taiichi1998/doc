# AI Review & Security Platform

## 1. 方針

Azure DevOps Pipeline から Self-hosted Agent を経由し、Security Platform 上の各セキュリティツールを実行する。

Security Platform の実行基盤には AKS を使用する。

役割を以下のように分離する。

- Self-hosted Agent VM：Pipeline実行の入口
- AKS：Security Platform / セキュリティOSS実行基盤
- Azure Database for PostgreSQL：永続DB
- DefectDojo：セキュリティ結果の一元管理・ダッシュボード

---

## 2. 全体構成

```text
Azure DevOps
│
├─ Repos
│   ├─ Application Code
│   ├─ .claude/
│   └─ Pipeline YAML
│
└─ Pipelines
        │
        ▼
Self-hosted Agent VM
└─ Azure DevOps Agent
        │
        │ Kubernetes API / Pipeline実行
        ▼
AKS
└─ security-platform namespace
    │
    ├─ DefectDojo
    │   └─ Security Dashboard / Finding管理
    │
    ├─ SonarQube
    │
    ├─ AI Reviewer
    │   └─ Claude Code
    │
    ├─ Trivy
    │
    ├─ OWASP ZAP
    │
    ├─ k6
    │
    ├─ Renovate
    │
    ├─ SBOM Generator
    │
    └─ その他Security OSS
        │
        ▼
Azure Database for PostgreSQL
├─ DefectDojo DB
└─ SonarQube DB
```

---

## 3. Self-hosted Agent VM

VMには原則として Azure DevOps Self-hosted Agent のみを配置する。

セキュリティOSSはVM上では実行せず、AKS上で実行する。

```text
Agent VM
└─ ado-agent
    └─ Azure DevOps Self-hosted Agent
```

### Agentユーザー

専用ユーザーを使用する。

- sudo禁止
- SSHログイン禁止
- Agent専用Workspace
- 開発ユーザーと分離
- 本番Secretを保存しない
- Azure権限を最小化

---

## 4. Agent VMのセキュリティ前提

Self-hosted AgentはPipelineから任意コードを実行するため、VM自体が侵害される可能性を前提とする。

```text
悪意ある / 侵害されたPipeline
        ↓
Self-hosted Agent
        ↓
Agentユーザー権限で
VM上のコマンド実行
```

そのため、

「VMへ侵入されない構成」

ではなく、

「VMへ侵入されても被害範囲を限定する構成」

とする。

---

## 5. AKS権限

Self-hosted AgentからAKSへの権限は最小化する。

```text
Agent
 ↓
Managed Identity / Service Principal
 ↓
AKS RBAC
 ↓
security-platform namespaceのみ
```

原則禁止：

- cluster-admin
- 他Namespaceへの操作
- 本番Application Namespaceへの操作
- Azure Subscription全体へのContributor
- 不要なSecret参照

Pipelineが侵害された場合でも、

```text
Pipeline侵害
 ↓
Agent VM侵害
 ↓
Security Platform namespace
```

までで封じ込める設計とする。

Azure全体や本番アプリへ横展開できないことを重視する。

---

## 6. AKS Security Platform

```text
AKS
└─ security-platform
    ├─ DefectDojo
    ├─ SonarQube
    ├─ AI Reviewer
    ├─ Trivy
    ├─ ZAP
    ├─ k6
    ├─ Renovate
    ├─ SBOM
    └─ その他Security OSS
```

### 常駐系

- DefectDojo
- SonarQube

### Job / CronJob系

- AI Reviewer
- Trivy
- ZAP
- k6
- SBOM Generator
- Renovate

可能なものは実行終了後にPodを削除する。

---

## 7. DefectDojo

DefectDojoをSecurity Platformのメインダッシュボードとして利用する。

役割：

- 脆弱性結果の集約
- Severity管理
- Project / Product別管理
- Scanner別管理
- Finding管理
- 重複排除
- 対応状況管理
- 再テスト管理
- セキュリティダッシュボード

別途Security Dashboardは初期段階では作成しない。

---

## 8. PostgreSQL

DefectDojoおよびSonarQubeのPostgreSQLはAKS上で自前運用せず、Azure PaaSを使用する。

```text
Azure Database for PostgreSQL Flexible Server
├─ defectdojo
└─ sonarqube
```

初期段階では同一PostgreSQL Server内でDBを分離して利用可能。

必要に応じて将来Server自体を分離する。

---

## 9. AI Reviewer

```text
PR作成 / 更新
 ↓
Azure DevOps Pipeline
 ↓
Self-hosted Agent
 ↓
AKS Job
 ↓
AI Reviewer
 ↓
Claude Code
 ↓
review-result.json
```

入力：

- .claude/project-rules.md
- .claude/architecture.md
- .claude/security-rules.md
- .claude/review-rules.md
- PRタイトル
- PR説明
- Git diff
- 変更コード
- 必要な周辺コード
- テスト変更内容

判定：

| Severity | Pipeline |
|---|---|
| Critical | FAIL + 人レビュー |
| High | FAIL + 人レビュー |
| Medium | PASS + 指摘 |
| Low | PASS + 指摘 |
| None | PASS |

---

## 10. Security Scan

```text
PR / CI
 ↓
Pipeline
 ↓
Agent
 ↓
AKS Jobs
 ├─ Trivy
 ├─ ZAP
 ├─ SonarQube Scan
 ├─ SBOM
 └─ AI Review
 ↓
DefectDojo
 ↓
Finding一元管理
```

---

## 11. Renovate

RenovateはAKS CronJobとして定期実行する。

```text
Renovate CronJob
 ↓
Azure DevOps Repos
 ↓
依存関係確認
 ↓
更新PR作成
 ↓
CI
 ├─ Test
 ├─ Trivy
 ├─ SonarQube
 ├─ AI Review
 └─ その他Security Scan
```

---

## 12. Pipeline保護

Pipeline侵害対策として以下を必須とする。

- Pipeline YAML変更をBranch Policyで保護
- Pipeline変更時はレビュー必須
- Agent Poolを許可Pipelineのみに限定
- Secretは必要最小限
- Agent Credentialを本番用途と共用しない
- AKS権限をNamespace単位で限定
- Managed Identity / Service Principalを用途別に分離
- 本番環境への直接アクセス禁止
- IaC / Helm変更はレビュー後に適用

---

## 13. 最終構成

```text
Azure DevOps
     │
     ▼
Self-hosted Agent VM
     │
     │ 最小権限
     ▼
AKS
└─ security-platform
    ├─ DefectDojo
    ├─ SonarQube
    ├─ AI Reviewer
    ├─ Trivy
    ├─ ZAP
    ├─ k6
    ├─ Renovate
    └─ SBOM
     │
     ▼
Azure Database for PostgreSQL
├─ DefectDojo DB
└─ SonarQube DB
```

### 最終方針

- Security PlatformはAKSへ集約
- Self-hosted Agentは専用VMへ配置
- Agent VMには原則Agentのみ配置
- Agent VMは侵害される可能性を前提に設計
- Agentにはsudoを与えない
- AgentのAzure / AKS権限は最小化
- AKSはSecurity Platform専用Namespaceへ制限
- DefectDojoをSecurity Dashboardとして利用
- SonarQube / DefectDojoのDBはAzure PostgreSQLを利用
- Trivy / ZAP / AI Reviewer / k6等はAKS Jobとして実行
- RenovateはAKS CronJobとして実行
- Pipeline YAMLをBranch Policyで保護
- Pipeline侵害時もSecurity Platform内で影響を封じ込める
- 本番Application環境とは明確に信頼境界を分離する
