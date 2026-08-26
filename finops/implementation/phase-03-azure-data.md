# Phase 03 — Azure補助データ取得

## ステータス

**未着手**

## 目的

Power BIの削減候補、コストシミュレーション、変更前後比較に必要なAzure補助データを取得し、Storageへ保存する。

このPhaseではPower BI画面は実装しない。

## 参照要件

- `../01-overall-requirements.md`
- `../02-functional-requirements.md`
- `../03-screen-operation-requirements.md`
- `../TASKS.md`
- `phase-01-foundation.md`
- `phase-02-cost-data.md`

## 前提

Phase 01〜02が完了し、Storageとコストデータが利用可能であること。

## 実装対象

### 1. Azure Retail Prices API

コストシミュレーション用価格データを取得する。

要件：

- Azure Retail Prices APIを利用
- Storageへ保存
- 更新頻度は週1回
- Power BIは保存済み価格データを参照する

保持項目は、シミュレーションで少なくとも以下を判別できるものとする。

- Service / Resource Type
- Region
- SKU / Meter
- Unit Price
- Currency
- Unit of Measure
- OS / License等の料金条件（存在する場合）
- 価格の有効性を判断するために必要な識別情報

APIレスポンスを無加工でPower BIへ依存させず、後続で安定して参照できるデータ契約を用意する。

### 2. シミュレーション対象Resource Type

Azure全サービスを固定一覧として実装しない。

対象：

- 管理対象RGで実際に利用中のResource Type
- Phase 01の設定で管理する将来採用候補Resource Type

Azure Resource Graph、ARM API等、既存環境に適した方法で利用中リソース情報を取得する。

### 3. Azure Advisor

削減候補の基本判定としてAdvisor推奨を取得する。

対象：

- 未使用リソース候補
- 低利用 / Right Sizing候補
- コスト削減推奨

可能な範囲で以下を保持する。

- 対象Resource
- RG
- Recommendation種別
- 推奨内容
- 想定削減額
- 削減率算出に必要な情報
- Recommendation ID等の追跡用識別子

CPU・メモリ・通信量を全面収集して独自Right Sizing判定を構築しない。

### 4. Reservation / Savings Plan

Microsoft側が提供する推奨値を取得可能な経路を利用し、後続Power BIで以下を比較できる形にする。

- 候補の有無
- 対象
- 想定削減額
- 1年契約時の効果
- 3年契約時の効果
- 削減率

独自の高度な予測モデルや損益分岐点モデルは作成しない。

### 5. Azure Activity Log

変更前後比較用として変更イベントを取得する。

CI/CDだけでなく、以下を対象にできること。

- Portal操作
- SKU変更
- ARM / Bicep等の変更
- その他Activity Logに記録される管理操作

後続で少なくとも以下を関連付けられるデータ構造にする。

- 変更日時
- Subscription
- RG
- Resource
- 操作内容
- 実行主体（取得可能な場合）
- Correlation ID等の追跡情報

### 6. 更新方式

- 価格データ：週1回
- Advisor / Resource情報 / Activity Log：用途に適した定期更新
- 既存の実行基盤・CI/CD・Automation手段を優先する
- 新規Azureサービスが必要な場合は追加コストと理由を記録する
- 再実行時に不整合・重複を生まない

## 対象外

- 独自Right Sizingエンジン
- 独自Reservation最適化モデル
- Power BI Semantic Model
- Power BI画面
- GitHub / Copilot / Azure DevOpsコスト
- Invoice統合

## 完了条件

- [ ] Retail Prices APIから価格を取得できる
- [ ] 価格データがStorageへ保存され週次更新できる
- [ ] 利用中Resource Typeを取得できる
- [ ] 将来候補Resource Typeを設定と統合できる
- [ ] Advisor推奨を取得・保存できる
- [ ] Reservation / Savings Plan比較に必要なMicrosoft推奨データの取得経路が実装または明確化されている
- [ ] Activity Logを取得・保存できる
- [ ] 各データの識別子・更新日時・粒度が文書化されている
- [ ] 再実行で重複や二重計上を起こさない

## Phase完了時に追記する内容

### 実装結果

未記入

### 変更ファイル

未記入

### データ契約

未記入

### 更新方式

未記入

### Phase 04への引き継ぎ

未記入
