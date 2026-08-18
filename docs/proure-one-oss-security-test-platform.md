# Proure One OSS セキュリティ・テスト基盤

## 1. 目的

Proure One におけるセキュリティ・品質・性能テストを、
OSSを中心として低コストかつ再現可能な形で構築する。

主な目的：

- セキュリティ品質向上
- CI/CDによる自動化
- OSS / 無料ツールを優先
- エンタープライズ利用可能な構成
- Private Azure環境へのテスト対応
- ツール環境の再現性・保守性確保
- VMへの直接インストールを最小化

---

## 2. 前提環境

### Azure VM

- OS：Linux
- VM：D4s v5
- CPU：4 vCPU
- Memory：16 GB
- Azure Virtual Network接続済み
- Azure DevOps Self-hosted Agentとして利用
- Private環境へのアクセス可能

### 想定ユーザー規模

- 最大：約1,000ユーザー
- 負荷試験は通常のPR CIでは実施しない
- 夜間 / リリース前 / 重要変更時に実施
- 必要に応じてVMをD8s v5等へ一時的にサイズアップ

---

## 3. 採用OSS

| 領域 | ツール | 用途 |
|---|---|---|
| SAST / Code Quality | SonarQube Community | 静的解析・コード品質 |
| SCA | Trivy | OSS依存関係の脆弱性検出 |
| Secret Scan | Trivy | API Key / Token / Password等の検出 |
| IaC Scan | Trivy | Bicep等のIaC設定検査 |
| SBOM | Trivy | SBOM生成 |
| DAST | OWASP ZAP | Web / API動的脆弱性診断 |
| Load Test | k6 OSS | Load / Stress / Spike / Soak |
| CSPM | Defender for Cloud Foundational CSPM | Azureセキュリティ推奨事項 |

---

## 4. 基本設計

OSSをVMへ直接大量にインストールするのではなく、
Dockerを中心に構築する。

構成：

Azure VM
│
├── Azure DevOps Agent
├── Docker Engine
├── Docker Compose
│
├── 常駐コンテナ
│   ├── SonarQube
│   └── PostgreSQL
│
└── 必要時のみ起動
    ├── Trivy
    ├── OWASP ZAP
    └── k6

---

## 5. コンテナ設計

基本方針：

- 1 OSS = 1コンテナ
- 1コンテナに複数OSSを詰め込まない
- バージョンを明示的に固定
- 実行後不要なコンテナは削除

### SonarQube

常駐。

SonarQube
    ↓
PostgreSQL

SonarQube本体とDBは別コンテナとする。

### PostgreSQL

SonarQubeの永続DBとして利用。

データはDocker Volume等へ永続化する。

### Trivy

常駐しない。

CI開始
  ↓
Trivyコンテナ起動
  ↓
Scan
  ↓
Report生成
  ↓
コンテナ削除

### OWASP ZAP

常駐しない。

Deploy
  ↓
ZAPコンテナ起動
  ↓
Dev環境をScan
  ↓
Report生成
  ↓
コンテナ削除

### k6

常駐しない。

Performance Test開始
  ↓
k6コンテナ起動
  ↓
負荷生成
  ↓
Result生成
  ↓
コンテナ削除

---

## 6. Docker Composeの役割

Docker Composeは主に常駐サービスの構成管理に利用する。

対象：

- SonarQube
- PostgreSQL
- Network
- Volume
- Environment Variables
- Port
- Restart Policy

例：

docker compose up -d

によりSonarQube環境をまとめて起動可能。

Trivy / ZAP / k6は必ずしもComposeに含めず、
PipelineまたはShell Scriptからdocker runで実行してよい。

---

## 7. Ansibleの役割

Docker ComposeとAnsibleは役割が異なる。

### Ansible

VMそのものの初期構築を自動化する。

対象例：

- Docker Engineインストール
- Docker Composeインストール
- Azure DevOps Agent構築
- Linuxユーザー / Group
- Directory作成
- Permission設定
- OS設定
- Composeファイル配置
- 必要な設定ファイル配置

### Docker Compose

Docker導入後のコンテナ環境を管理する。

そのため、

Bicep
  ↓
Azure VM作成
  ↓
Ansible
  ↓
VM初期設定
  ↓
Docker / Compose
  ↓
OSSコンテナ

という責務分離とする。

---

## 8. Git Repository構成

security-tools/
├── ansible/
│   ├── inventory/
│   │   └── hosts.yml
│   │
│   ├── playbooks/
│   │   └── setup.yml
│   │
│   └── roles/
│       ├── docker/
│       ├── azure-devops-agent/
│       └── security-tools/
│
├── compose/
│   ├── docker-compose.yml
│   └── .env.example
│
├── sonarqube/
│   └── config/
│
├── trivy/
│   └── trivy.yaml
│
├── zap/
│   └── config/
│
├── k6/
│   └── scripts/
│       ├── load.js
│       ├── stress.js
│       ├── spike.js
│       └── soak.js
│
├── scripts/
│   ├── run-trivy.sh
│   ├── run-zap.sh
│   └── run-k6.sh
│
├── pipelines/
│   ├── sonar.yml
│   ├── trivy.yml
│   ├── zap.yml
│   └── k6.yml
│
├── .gitignore
└── README.md

---

## 9. 各ディレクトリの責務

### ansible/

VM構築・設定自動化。

### compose/

常駐コンテナ構成。

主に：

- SonarQube
- PostgreSQL

### sonarqube/

SonarQube固有設定。

### trivy/

Trivyの：

- Severity
- Ignore
- Scan設定
- 出力設定

等を管理。

### zap/

ZAPの：

- 対象URL
- Authentication
- Scan Policy
- 除外URL
- API Scan設定

等を管理。

### k6/

性能試験シナリオを管理。

例：

- load.js
- stress.js
- spike.js
- soak.js

### scripts/

Docker実行コマンドをラップする。

例：

./scripts/run-trivy.sh

とすることでPipeline側に長いdocker runコマンドを書かない。

### pipelines/

Azure DevOps Pipelineテンプレート。

複数アプリから共通利用可能な構造とする。

---

## 10. CI/CDフロー

### Pull Request / CI

Azure DevOps
    ↓
Build / Unit Test
    ↓
Trivy
├── SCA
├── Secret
├── IaC
└── SBOM
    ↓
SonarQube Analysis

---

## 11. Deploy後

Development Environment
    ↓
Smoke Test
    ↓
E2E
    ↓
OWASP ZAP
├── Baseline Scan
└── API Scan

Active Scanは必要に応じて別途実施する。

---

## 12. 負荷試験

通常のPR Pipelineには組み込まない。

実施タイミング：

- リリース前
- 大規模API変更
- DB変更
- Query変更
- インフラ変更
- 性能劣化発生時
- 定期性能確認

k6
├── Load Test
├── Stress Test
├── Spike Test
└── Soak Test

利用者1,000人 = 1,000 VUとはしない。

実際の同時利用数を基準として負荷モデルを作成する。

例：

50 VU
 ↓
100 VU
 ↓
300 VU
 ↓
500 VU
 ↓
必要に応じてStress Test

---

## 13. VM負荷方針

通常：

D4s v5
- 4 vCPU
- 16 GB RAM

常時：

- Azure DevOps Agent
- SonarQube
- PostgreSQL

必要時：

- Trivy
- ZAP
- k6

重い処理：

- ZAP Active Scan
- k6 Stress / Soak

については可能な限り同時実行しない。

必要な場合のみVMを一時的にサイズアップする。

---

## 14. バージョン管理

latestタグは原則使用しない。

例：

NG

sonarqube:latest
aquasec/trivy:latest

OK

sonarqube:<固定バージョン>
aquasec/trivy:<固定バージョン>

ツール更新は意図的に実施する。

---

## 15. Secret管理

Git RepositoryにはSecretを保存しない。

保存しないもの：

- Password
- API Key
- Token
- Sonar Token
- Azure Credential

原則として以下を利用する。

- Azure DevOps Secret Variable
- Variable Group
- Azure Key Vault
- Managed Identity

.envはGit管理しない。

Gitには、

.env.example

のみ保存する。

---

## 16. Report管理

以下は原則Gitに保存しない。

- Trivy Report
- ZAP Report
- k6 Result
- SonarQube Data
- PostgreSQL Data

Azure DevOps Pipeline Artifactや、
必要に応じてAzure Storage等へ保存する。

---

## 17. 基本方針まとめ

VMそのもの
    ↓
Ansible

コンテナ基盤
    ↓
Docker / Docker Compose

常駐サービス
    ↓
SonarQube + PostgreSQL

CI Security Tool
    ↓
Trivy

DAST
    ↓
OWASP ZAP

Performance Test
    ↓
k6

実行制御
    ↓
Azure DevOps Pipeline

設定・IaC・スクリプト
    ↓
Git Repository

---

## 18. 最終アーキテクチャ

Azure
│
├── Proure One Dev Environment
│
└── Security / Test VM
    │
    ├── Azure DevOps Agent
    │
    ├── Docker
    │
    ├── Docker Compose
    │
    │
    ├── SonarQube Container
    │       │
    │       └── PostgreSQL Container
    │
    ├── Trivy Container      [On Demand]
    ├── ZAP Container        [On Demand]
    └── k6 Container         [On Demand]

Azure DevOps Pipeline
│
├── CI
│   ├── Unit Test
│   ├── Trivy
│   └── SonarQube
│
├── CD
│   ├── Deploy
│   ├── Smoke Test
│   ├── E2E
│   └── ZAP
│
└── Performance Pipeline
    └── k6

---

## 19. 設計原則

- VMへの直接インストールを最小化
- 1ツール1コンテナ
- 常駐コンテナを必要最小限にする
- CLI系ツールはOn-Demand実行
- Docker ImageのVersion固定
- Gitで設定を管理
- SecretはGitに保存しない
- ReportはArtifactとして管理
- Infrastructureはコード化
- Pipelineはテンプレート化
- 再構築可能な環境を維持する

---

## 20. 導入順序

1. Azure VM準備
2. Ansible構築
3. Docker / Docker Compose導入
4. Azure DevOps Agent構築
5. SonarQube + PostgreSQL構築
6. SonarQube CI連携
7. Trivy導入
8. Trivy CI連携
9. ZAP導入
10. ZAP CD連携
11. k6導入
12. Performance Pipeline構築
13. Report / Artifact運用整備
14. Version Update運用整備
15. セキュリティ・テスト基盤を部門標準化