# Phase 02 — Cost Managementデータ取得・整形

## ステータス

**未着手**

## 目的

Azure Cost Managementを主データソースとして、Power BIで分析可能な日次コストデータを安定してStorageへ蓄積・整形する。

## 参照要件

- `../01-overall-requirements.md`
- `../02-functional-requirements.md`
- `../03-screen-operation-requirements.md`
- `../TASKS.md`
- `phase-01-foundation.md`

## 前提

Phase 01が完了し、Storage・Lifecycle・設定値管理の基盤が利用可能であること。

## 実装対象

### 1. Cost Management Export

- 対象RGを含む管理スコープからCost Managementデータを取得する
- Cost Management Exportを日次実行する
- 出力先をPhase 01で用意したStorageとする
- 必要時に再取得・再処理しやすい構造にする

### 2. 分析に必要な項目の保持

Power BIで少なくとも以下の軸を扱えるデータ構造にする。

- 日付
- Subscription
- Resource Group
- Service
- Resource
- Resource Type
- SKU
- Cost
- Currency
- Tag

管理対象タグ：

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

タグ未設定値は、後段で `Other` / `Untagged` として扱えるよう欠損状態を保持・正規化する。

### 3. 基本集計に必要な整形

以下を後続Semantic Modelで容易に計算できるデータ構造にする。

- 日別コスト
- 月別コスト
- Subscription別
- RG別
- Service別
- Resource別
- SKU別
- Tag別
- 前月比較

過剰な事前集計を作りすぎず、Power BI側で必要な粒度を失わないこと。

### 4. 予算・Forecastデータ

- Azure Cost Management標準の予算・Forecastを利用する方針を維持する
- RGを基本粒度とし、Department / Projectで分析可能な形にする
- 独自Forecastモデルは作成しない

取得方法がCost Management Exportと異なる場合は、その差異とデータ契約を記録する。

### 5. RG社内負担ルール用データ

RG単位・月単位の実コストを算出できるようにする。

ルール：

- RG月額コスト <= 設定閾値：実質負担額 0円
- RG月額コスト > 設定閾値：実コスト全額を実質負担額とする

閾値はPhase 01の設定値を参照し、具体的な金額をロジックへ直接ハードコードしない。

### 6. 更新情報

- データ取得・処理の成功日時を保持する
- Power BIで最終更新日時として利用できるようにする
- 再実行で二重計上しないよう冪等性を確保する

## 対象外

- Retail Prices API
- Azure Advisor
- Activity Log
- 独自異常検知
- Power BI画面
- 独自Forecast
- GitHub / Copilot / Azure DevOps費用

## 検証

代表的な期間・RGを選び、Azure Cost Analysisと以下を突合する。

- 日次合計
- 月次合計
- RG別合計
- Service別合計
- Resource別合計

通貨・税・クレジット・償却等のCost Management側の列定義に差異がある場合は、採用するコスト列を明示する。

## 完了条件

- [ ] Cost Management Exportが日次でStorageへ出力される
- [ ] 分析に必要な主要ディメンションを保持している
- [ ] 管理対象タグを後続分析で利用できる
- [ ] 予算・Forecastデータの取得経路が実装または明確化されている
- [ ] RG月額コストを正しく算出できる
- [ ] RG負担閾値を設定値から参照できる
- [ ] 再実行で二重計上しない
- [ ] 最終更新日時を保持している
- [ ] Cost Analysisとの主要集計値の突合が完了している

## Phase完了時に追記する内容

### 実装結果

未記入

### 変更ファイル

未記入

### データ契約

未記入

### 検証結果

未記入

### Phase 03への引き継ぎ

未記入
