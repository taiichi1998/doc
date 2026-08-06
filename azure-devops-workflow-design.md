# Azure DevOps パイプライン・ワークフロー設計

## 1. 目的

Azure DevOps、開発環境、本番環境を使用し、各アプリを安全かつシンプルに継続デリバリーするための基本ワークフローを定義する。

---

## 2. 前提

- ソース管理・CI/CD：Azure DevOps Repos / Pipelines
- 環境：開発環境、本番環境
- アプリ構成：React + Node.js + TypeScript
- 配置先：フロントエンドとAPIを同一App Serviceへデプロイ
- DB：Azure SQL Database
- DB変更方式：SQL Database Projects + DACPACを第一候補として検討中
- 認証：Microsoft Entra ID
- E2Eテスト：Playwright
- IaC：Bicep
- アプリごとにリポジトリを分割
- 共通バックエンド基盤は独立リポジトリ・独立パイプライン
- 共通インフラも独立リポジトリ・独立パイプライン
- `main`への直接Pushは禁止し、Pull Requestを必須とする
- `main`マージ後、開発環境へ自動デプロイし、承認後に本番へデプロイする
- 当面は本番App Serviceへ直接デプロイし、将来的にDeployment Slotへ移行する

---

## 3. パイプライン構成

### 基本方針

**1リポジトリにつき1本のマルチステージYAMLパイプライン**を用意する。

システム全体を1本にまとめるのではなく、デプロイ単位ごとに分離する。

```text
各アプリリポジトリ       → 各1本
共通バックエンドリポジトリ → 1本
共通インフラリポジトリ     → 1本
```

各パイプラインのYAMLは共通テンプレートを利用し、リポジトリ固有部分だけを定義する。

```text
pipelines/
├─ azure-pipelines.yml
└─ templates/
   ├─ validate.yml
   ├─ build.yml
   ├─ test-unit.yml
   ├─ security-scan.yml
   ├─ deploy-app-service.yml
   ├─ deploy-database.yml
   ├─ smoke-test.yml
   ├─ playwright.yml
   └─ bicep.yml
```

---

## 4. アプリ用ワークフロー

```text
Feature Branch
    ↓
Pull Request
    ↓
Validate
    ├─ Lint
    ├─ Type Check
    ├─ Unit Test
    ├─ Build
    ├─ Security Scan
    ├─ SQL Project Build
    └─ Bicep Validate / What-If
    ↓
レビュー・mainへマージ
    ↓
Build Artifact
    ├─ アプリ成果物
    ├─ DACPAC
    └─ Bicepテンプレート
    ↓
Deploy Development
    ├─ Bicep適用（変更時のみ）
    ├─ DB変更適用
    └─ App Serviceへデプロイ
    ↓
Test Development
    ├─ Smoke Test
    ├─ API / DB Integration Test
    └─ Playwright E2E Test
    ↓
Production Approval
    ↓
Deploy Production
    ├─ DB変更適用
    └─ App Serviceへデプロイ
    ↓
Production Smoke Test
```

### ステージ例

```text
Validate
Build
Deploy_Development
Smoke_Development
Integration_Test
E2E_Development
Approve_Production
Deploy_Production
Smoke_Production
```

同一の成果物を開発環境から本番環境へ昇格させる。本番用に再ビルドは行わない。

---

## 5. Pull Request時のテスト

Pull Requestでは、環境へデプロイせずに実行できる高速な検証を中心に行う。

| テスト・検証 | 内容 |
|---|---|
| Lint | コーディング規約違反を検出 |
| Type Check | TypeScriptの型エラーを検出 |
| Unit Test | 関数・クラス・コンポーネント単位の検証 |
| Build | フロント・APIが正常にビルドできるか確認 |
| SAST | ソースコードの脆弱性検査 |
| SCA | 依存ライブラリの脆弱性検査 |
| Secret Scan | シークレットや接続情報の混入検査 |
| SQL Project Build | DB定義が正常にDACPAC化できるか確認 |
| Bicep Lint / Validate | Bicepの構文・デプロイ前検証 |
| Bicep What-If | インフラ変更内容を事前表示 |

PR検証に失敗した場合、`main`へマージしない。

---

## 6. 開発環境デプロイ後のテスト

### 6.1 スモークテスト

デプロイ直後に、システムが最低限利用可能かを短時間で確認する。

想定実行時間は1〜3分程度とする。

```text
- アプリURLへアクセスできる
- ヘルスチェックAPIが成功する
- Entra ID認証済みユーザーとして画面を表示できる
- 主要APIが正常応答する
- Azure SQL Databaseから基本データを取得できる
- 共通バックエンドの必須APIへ接続できる
```

スモークテストはPlaywrightまたはAPIテストで実装する。

### 6.2 統合テスト

```text
- APIとAzure SQL Databaseの接続
- DBへの登録・参照・更新
- 外部または共通バックエンドAPIとの連携
- 認証情報・権限情報の受け渡し
```

開発環境にはテスト用シードデータを投入可能とする。

### 6.3 Playwright E2Eテスト

主要なユーザー操作をブラウザから検証する。

```text
- 未認証ユーザーがログインへ誘導される
- 認証済みユーザーがトップ画面を表示できる
- 権限を持つユーザーが主要機能を利用できる
- 権限がないユーザーの操作が拒否される
- 一覧表示
- 新規登録
- 詳細表示
- 更新
- 状態変更または削除
```

Microsoftのログイン画面をすべてのE2Eテストで操作するのではなく、テスト専用ユーザーの認証状態を作成・再利用する。

ログイン自体が可能かどうかは、スモークテストまたは認証専用テストで最低1ケース確認する。

---

## 7. 本番環境デプロイ後のテスト

本番では原則として、データを変更しない読み取り中心のスモークテストを行う。

```text
- ヘルスチェックAPI
- Readiness確認
- アプリのトップページ表示
- Entra ID認証後の画面表示
- 参照専用API
- 参照専用画面
- Azure SQL Databaseへの接続
- 共通バックエンドへの接続
```

本番で登録・更新・削除を確認する場合は、専用テストデータと自動クリーンアップ処理を用意する。

---

## 8. テスト実行タイミング

| タイミング | 実行内容 |
|---|---|
| ローカル開発 | Lint、Type Check、対象Unit Test |
| Pull Request | Lint、Type Check、Unit Test、Build、Security Scan、SQL/Bicep検証 |
| 開発環境デプロイ直後 | Smoke Test |
| 開発環境スモーク成功後 | Integration Test、主要Playwright E2E |
| 本番デプロイ前 | 開発環境のテスト結果確認、手動承認 |
| 本番デプロイ直後 | 読み取り中心のSmoke Test |
| 夜間・定期実行 | 全E2E、重いDAST、詳細なセキュリティ検査 |

E2Eの実行時間が長くなった場合は、次のように分割する。

```text
主要E2E → mainマージ後に毎回実行
全E2E   → 夜間またはリリース前に実行
```

---

## 9. DBデプロイ

SQL Database Projectsを採用した場合、DACPACを成果物として扱う。

```text
Pull Request
    ↓
SQL Project Build
    ↓
DACPAC生成確認
    ↓
mainマージ
    ↓
DACPACを一度だけ生成
    ↓
開発DBへ適用
    ↓
DB統合テスト
    ↓
同じDACPACを本番DBへ適用
```

破壊的な変更は自動適用しない。

### 自動適用しやすい変更

```text
- テーブル追加
- NULL許可列の追加
- インデックス追加
- View・Stored Procedure追加
```

### 手動確認が必要な変更

```text
- テーブル削除
- 列削除
- データ型縮小
- NOT NULL化
- 大量データ更新
- データ移行を伴う変更
```

複雑なデータ移行はDACPACだけで処理せず、バージョン管理した移行SQLを併用する。

---

## 10. Bicepワークフロー

### Pull Request

```text
Bicep Lint
    ↓
Bicep Build
    ↓
Azure Resource Manager Validate
    ↓
What-If
```

### mainマージ後

```text
infra/配下に変更あり
    ↓
開発環境へBicep適用
    ↓
動作確認
    ↓
本番承認
    ↓
本番環境へBicep適用
```

`infra/`配下に変更がない場合は、Bicep関連処理をスキップする。

What-Ifで削除・再作成が表示された場合は、自動デプロイせず手動レビューを必須とする。

---

## 11. 共通バックエンド基盤

共通バックエンドは、各アプリとは独立してデプロイする。

```text
Validate
    ↓
Build
    ↓
Deploy Development
    ↓
Smoke / Integration / Contract Test
    ↓
Production Approval
    ↓
Deploy Production
    ↓
Production Smoke Test
```

各アプリのパイプラインから共通バックエンドをデプロイしない。

各アプリ側では、必須APIへの接続確認とAPI契約の互換性を検証する。

将来的にはOpenAPIの破壊的変更検知やConsumer Contract Testを導入する。

---

## 12. 共通インフラ

共通インフラは独立パイプラインで管理する。

対象例：

```text
- VNet
- Private DNS
- 共通監視
- 共通Key Vault
- 共通ログ基盤
- 共通バックエンド基盤の土台
```

アプリ固有パイプラインでは、アプリ固有のApp Service、設定、DBなどのみを管理する。

---

## 13. Azure DevOps Environmentと承認

Azure DevOps Environmentsとして次を作成する。

```text
development
production
```

### development

```text
- `main`マージ後に自動デプロイ
- 原則として手動承認なし
```

### production

```text
- 手動承認あり
- 当面はセルフ承認を許可
- 代理承認者を設定
- 同時デプロイを防ぐ排他制御を設定
```

承認設定はYAML内ではなく、Azure DevOps Environment側に設定する。

`main`ブランチには最低限、次のBranch Policyを設定する。

```text
- 直接Push禁止
- Pull Request必須
- Build Validation必須
- コメント解決必須
- 原則1名以上のレビュー
```

将来的に承認者が増えた場合は、通常リリースでのセルフ承認を禁止する。

---

## 14. Deployment Slotへの移行

### 初期構成

```text
開発環境へ直接デプロイ
    ↓
テスト
    ↓
本番環境へ直接デプロイ
    ↓
本番スモークテスト
```

### 将来構成

```text
本番App Serviceのstaging slotへデプロイ
    ↓
Slot上でスモークテスト
    ↓
Production SlotとSwap
    ↓
本番URLでスモークテスト
    ↓
問題時はSwap Back
```

次の条件が発生したらDeployment Slotへ切り替える。

```text
- 利用者が増加した
- デプロイ停止時間を短縮したい
- アプリ起動に時間がかかる
- デプロイ直後の初期化処理がある
- 短時間でロールバックできる構成が必要
```

---

## 15. この設計を採用する理由

- **1リポジトリ1パイプライン**にすることで、アプリごとの独立リリースを維持できる。
- CIとCDを1本のマルチステージパイプラインにまとめることで、現時点では運用を単純化できる。
- ステージを分離することで、将来的にCI/CDを別パイプラインへ分割しやすい。
- 一度生成した成果物を開発・本番で共通利用するため、検証済み成果物と本番成果物の差異を防止できる。
- スモーク、統合、E2Eを分離することで、障害の早期検知とテスト時間のバランスを取れる。
- 共通バックエンドと共通インフラを独立させることで、アプリ変更による不要な共通基盤デプロイを防止できる。
- Bicep What-Ifと本番承認により、インフラ変更や本番変更のリスクを抑えられる。
- 現在は直接デプロイで開始し、利用規模に応じてDeployment Slotへ段階的に移行できる。

---

## 16. 今後決定する項目

```text
- SQL Database Projectsを正式採用するか
- DACPACと移行SQLの運用ルール
- Playwright用Entra IDテストユーザー
- 認証状態の作成・保管・更新方法
- 本番スモークテストの具体的な確認項目
- E2Eテストの実行時間と分割基準
- 本番承認者・代理承認者
- Deployment Slotへ移行する時期
- 障害発生時のロールバック手順
```
