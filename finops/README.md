# FinOps

Azure中心のFinOps基盤に関する要件・設計ドキュメントです。

## ドキュメント構成

| No. | ドキュメント | ステータス | 内容 |
|---|---|---|---|
| 1 | [全体要件・前提](./01-overall-requirements.md) | 完了 | 対象範囲、基盤、データ取得、保持、タグ、RG負担ルール、アクセス |
| 2 | [機能要件](./02-functional-requirements.md) | 完了 | コスト分析、削減候補、配賦、Forecast、シミュレーション、タグ品質等 |
| 3 | [画面・運用要件](./03-screen-operation-requirements.md) | 完了 | Power BIページ構成、各画面詳細、更新・運用 |

## 現在の構成

- Azure Cost Management
- Cost Management Exports
- Azure Storage Account
- Microsoft FinOps Toolkit
- Power BI
- Azure Retail Prices API（コストシミュレーション用）

初期段階ではFinOps Hub / Azure Data Explorer（ADX）は使用しない。

## 現在の主要決定事項

### 対象

- 現在の主管理単位：Resource Group
- Subscription単位にも対応
- 部門管理RG内の全Azureリソースを対象
- 将来：GitHub / GitHub Copilot / Azure DevOps費用を統合

### データ更新・保持

- Cost Management Export：日次
- Power BI：日次更新 + 必要時手動更新
- Azure価格データ：週1回
- データ保持：3年
- 0〜6か月：Hot
- 6か月超〜1年：Cool
- 1年超〜3年：Cold
- 3年経過後：削除
- Lifecycle Management：Bicepで管理

### 将来コスト統合

- GitHub / GitHub Copilot / Azure DevOps費用はMicrosoft Cost Managementで取得可能な範囲を優先
- 不足分のみ各サービスのAPI等で補完
- Invoiceは初期はCost Managementで確認し、将来Power BIへ主要情報のみ統合

### 削減・分析ロジック

- 未使用・低利用リソース：Azure Advisorを基本判定とし、必要な場合のみ独自ルールを追加
- Reservation / Savings Plan：Microsoft推奨値を基本に1年・3年の削減額・削減率を比較
- Forecast：Azure Cost Management標準Forecastを利用
- 予算超過原因分析：前月比を基本とし、前日比 / 前週比へ切替可能
- 変更前後比較：Azure Activity Logから変更日時を取得し、手動指定も可能

### 画面設計

- 経営サマリー：Department / Project等はTop N制限をせず全件表示、金額降順
- コスト分析：月次をメインとし、今月 / 先月 / 直近7日 / 直近30日 / 直近3か月 / 直近12か月 / 任意期間を選択可能
- コスト分析の比較：前月比を基本とし、比較対象月を選択可能
- 共通フィルター：Subscription / RG / Service / Resource / SKUに加え、管理対象タグを原則すべて利用可能
- タグ品質の必須タグ：Owner / Project / CostCenter / Environment / Department / Service
- コストシミュレーション：Power BIパラメータ / スライサー入力
- シミュレーション対象：実利用中Resource TypeをAPI等から取得し、将来候補はBicep / 設定ファイルで管理
- 変更前後比較：7日 vs 7日をデフォルト、30日 vs 30日へ切替可能

### RG社内負担ルール

現在はRGごと・月ごとに200,000円まで会社負担。

- 200,000円以下：部門実質負担額 0円
- 200,000円超：RGコスト全額を部門負担

閾値は設定値として管理し、将来変更可能とする。

### 初期Power BIページ

1. 経営サマリー
2. コスト分析
3. 削減候補
4. タグ品質
5. コストシミュレーション
6. 変更前後比較
7. 予算超過原因分析

## 完了状況

### 完了

- [x] FinOps対象範囲
- [x] 基盤構成
- [x] RG中心 + Subscription対応方針
- [x] Cost Management Export日次更新
- [x] Power BI日次更新方針
- [x] データ保持期間3年
- [x] Hot / Cool / Cold移行タイミング
- [x] Lifecycle ManagementをBicepで管理
- [x] RG + Tagによる配賦方針
- [x] Ownerの扱い
- [x] タグ品質機能
- [x] 未使用・低利用リソースの判定方針
- [x] Reservation / Savings Plan比較ロジック
- [x] コストシミュレーション対象・入力方式
- [x] Retail Prices API → Storage → Power BI方針
- [x] 予算超過原因分析の粒度・比較期間・表示方式
- [x] 変更前後比較の粒度・変更日時取得方式・比較期間
- [x] Forecastの分析粒度・計算方式
- [x] 経営向けサマリーの主要項目・表示方針
- [x] コスト分析画面の期間・比較・フィルター方針
- [x] 削減候補画面の表示方針
- [x] RG月20万円社内負担ルール
- [x] 独自異常検知の初期要件からの削除
- [x] 独自異常通知の初期要件からの削除
- [x] Unit Costの初期対象外化
- [x] Invoice Power BI統合の将来方針
- [x] GitHub / Copilot / Azure DevOps費用統合方針
- [x] Power BIページ構成
- [x] 部門内全RG閲覧方針
- [x] 更新失敗時は前回成功データを表示 + 最終更新日時表示

## 現在の状態

初期FinOps要件・機能要件・画面運用要件は確定済み。

次フェーズでは、実装設計として以下を具体化する。

- Power BIデータモデル / DAX
- Cost Management Export設定
- Storage構成 / Lifecycle Bicep
- Azure Retail Prices API取得処理
- Azure Activity Log連携
- Advisorデータ取得
- Power BI各ページの実レイアウト
