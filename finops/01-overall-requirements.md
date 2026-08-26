# FinOps 全体要件・前提

## ステータス

**完了**

基本方針・対象範囲・データ更新・保持・アクセス方針・将来統合方針まで確定済み。

## 1. 目的

Azureを中心としたクラウドコストを可視化・分析し、予算管理、コスト削減、配賦、将来コストのシミュレーションを行えるFinOps基盤を構築する。

## 2. 対象範囲

### 現在

- Azureを主対象とする
- 会社側でSubscriptionを管理しているため、現在の主な管理単位はResource Group（RG）
- 部門管理対象RG内の全Azureリソースを対象とする
- Subscription単位の分析機能も維持する

### 将来

- 部門専用Subscriptionを取得した場合はSubscription単位でも管理できるようにする
- GitHub費用
- GitHub Copilot費用
- Azure DevOps費用

上記は将来統合対象とする。

## 3. 基盤構成

- Azure Cost Management
- Cost Management Exports
- Azure Storage Account
- Microsoft FinOps Toolkit
- Power BI

### 初期導入しないもの

- FinOps Hub
- Azure Data Explorer（ADX）

Power BIは社内ライセンスを利用し、追加コストを極力抑える。

## 4. データ取得・更新

### Cost Managementデータ

- Cost Management Exportを使用
- 日次でStorage Accountへ出力
- Power BIも日次更新
- 必要時に手動更新可能とする
- リアルタイム更新は行わない

### Azure価格データ

コストシミュレーション用にAzure Retail Prices APIを使用する。

- Retail Prices APIから価格を取得
- Storage Accountへ価格データを保存
- Power BIは保存済み価格データを参照
- 価格データは週1回更新

## 5. データ保持

- 保持期間：3年間
- 0〜6か月：Hot
- 6か月超〜1年：Cool
- 1年超〜3年：Cold
- 3年経過後：削除

Storage Lifecycle Managementで自動階層化・削除を行う。

### Lifecycle Management管理方式

- Azure Portalでの手動設定を正としない
- BicepでIaC管理する
- 保持期間・階層移行条件をコードとして管理し、設定漏れや環境差分を防止する

## 6. タグ方針

既存Bicepテンプレートで以下のタグを管理している。

- Department
- ManagedBy
- Repository
- NetworkMode
- NetworkConfigurationStatus
- Owner
- Project
- CostCenter
- Environment
- Service

### FinOpsで主に使用するタグ

- Owner
- Project
- CostCenter
- Environment
- Department
- Service

### 配賦・分析方針

- RG + Tagを併用する
- Ownerは原則として現在の担当者・責任者のメールアドレスを設定する
- 作成者そのものではなく、現在の責任者をOwnerとして扱う
- タグ未設定データはメイン分析画面では `Other` または `Untagged` として集約する
- タグ不足の詳細確認はタグ品質ページで行う

## 7. 社内RGコスト負担ルール

現在の会社制度として、RG単位・月単位で20万円まで会社側が負担する。

### 判定ルール

- RG月額コストが200,000円以下：実質負担額 0円
- RG月額コストが200,000円超：RG月額コスト全額を部門負担

例：

| RG月額コスト | 実質負担額 |
|---:|---:|
| 190,000円 | 0円 |
| 200,000円 | 0円 |
| 210,000円 | 210,000円 |

超過分のみを負担する方式ではない。

### 実装方針

- 200,000円は固定値としてハードコードしない
- 設定値として管理し、将来の制度変更に対応可能とする
- Power BIでは「実コスト」と「実質負担額」を両方表示する

## 8. アクセス権

- 部門内ユーザーは対象RG全体を閲覧可能
- 初期段階ではProject / Department単位のRow-Level Security（RLS）は不要

## 9. データ更新失敗時

- 前回正常取得したデータをそのまま表示する
- レポート上に最終更新日時を必ず表示する
- 初期要件では更新失敗専用の通知・エラー画面は必須としない

## 10. 将来コスト統合方針

### GitHub / GitHub Copilot / Azure DevOps

- Microsoft Cost Managementで取得可能な費用はCost Management側を優先して利用する
- Cost Managementで取得できない費用・利用情報のみ、各サービスのAPI等で補完する
- 補完データはStorageへ集約し、最終的にPower BIで分析可能とする
- 初期リリースでは統合対象外とし、将来拡張とする

### Invoice / 請求情報

- 初期はAzure Cost Management側で確認する
- 将来Power BIへ主要な請求情報のみ統合する
- Invoice明細の全項目をPower BIへ複製することは初期方針としない

## 11. 初期対象外・将来機能

- Unit Cost
  - ユーザー数、案件数などCost Management外の業務データが必要になるため初期対象外
- Invoice / 請求情報のPower BI統合
  - 初期はAzure Cost Management側で確認
  - 将来的に主要請求情報をPower BIへ統合
- GitHub / GitHub Copilot / Azure DevOpsコスト統合

## 確定事項

- [x] FinOps対象範囲
- [x] 基盤構成
- [x] データ更新方式
- [x] Storage保持期間3年
- [x] Hot / Cool / Cold移行タイミング
- [x] Lifecycle ManagementをBicepで管理
- [x] タグ方針
- [x] RG社内負担ルール
- [x] アクセス方針
- [x] 更新失敗時の表示方針
- [x] GitHub / Copilot / Azure DevOps費用統合方針
- [x] 将来のInvoice統合方針
