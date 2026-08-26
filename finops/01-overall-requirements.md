# FinOps 全体要件・前提

## ステータス

**ほぼ完了**

基本方針・対象範囲・データ更新・保持・アクセス方針まで確定済み。実装方式の細部は設計フェーズで確定する。

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
- 直近データは高速アクセス可能な層に保持
- 古いデータは低コスト層へ移行

### 現時点の方針

- 直近6か月：Hot
- 6か月超：Cool
- さらに古いデータ：Coldの利用を検討
- 3年経過後に削除

Storage Lifecycle Managementによる自動階層化を前提とする。

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

## 10. 初期対象外・将来機能

- Unit Cost
  - ユーザー数、案件数などCost Management外の業務データが必要になるため初期対象外
- Invoice / 請求情報のPower BI統合
  - 初期はAzure Cost Management側で確認
  - 将来的に主要請求情報をPower BIへ統合
- GitHub / GitHub Copilot / Azure DevOpsコスト統合

## 未完了事項

- StorageのCold移行タイミングの最終決定
- 実装時のStorage Lifecycle Management具体設定
- GitHub / Copilot / Azure DevOpsコスト統合方式
- 将来のInvoice統合方式
