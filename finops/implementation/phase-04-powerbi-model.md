# Phase 04 — Power BI Semantic Model / DAX

## ステータス

**未着手**

## 目的

Phase 02〜03で取得したデータをPower BIで一貫して分析できるSemantic Modelへ統合し、7画面で共通利用するメジャー・期間・比較ロジックを実装する。

このPhaseでは画面の作り込みを主目的にしない。

## 参照要件

- `../01-overall-requirements.md`
- `../02-functional-requirements.md`
- `../03-screen-operation-requirements.md`
- `../TASKS.md`
- `phase-02-cost-data.md`
- `phase-03-azure-data.md`

## 前提

Phase 02〜03のデータ契約が確定し、Power BIから参照できること。

## 実装対象

### 1. Power BIプロジェクト構成

- 既存Power BI成果物・開発方式を確認する
- ソース管理しやすい形式を優先する
- PBIP / TMDL / PBIR等を利用する場合は、既存環境とPower BI Desktopの対応状況を確認して採用する
- データ接続情報や環境依存値をコードへ直書きしない

### 2. Semantic Model

少なくとも以下を分析できるモデルを構築する。

- Date
- Subscription
- Resource Group
- Service
- Resource
- Resource Type
- SKU
- Department
- ManagedBy
- Repository
- NetworkMode
- NetworkConfigurationStatus
- Owner
- Project
- CostCenter
- Environment
- Tag
- Cost fact
- Budget / Forecast
- Advisor recommendation
- Price
- Activity Log / Change event

不要な多対多関係や双方向フィルターを増やさず、モデルの意味と粒度を明確にする。

### 3. 日付・期間

日付ディメンションを用意し、以下を扱えるようにする。

- 今月
- 先月
- 直近7日
- 直近30日
- 直近3か月
- 直近12か月
- 任意期間
- 月次
- 日次

### 4. 共通メジャー

少なくとも以下を実装する。

- 総コスト
- 予算
- 予算差
- Forecast
- 前月コスト
- 前月差額
- 前月比
- 実質負担額
- 会社負担閾値
- 閾値までの残額
- 閾値超過有無
- 閾値超過RG件数
- 想定削減額
- 削減率
- 最終更新日時

RG負担ルール：

- RG月額コスト <= 設定閾値：実質負担額 0円
- RG月額コスト > 設定閾値：実コスト全額

設定閾値を参照し、DAXへ具体的な金額を直接固定しない。

### 5. 比較ロジック

#### コスト分析

- 基本：前月比
- 比較対象月を選択可能

#### 変更前後比較

- デフォルト：変更前7日平均 vs 変更後7日平均
- 切替：30日平均 vs 30日平均

#### 予算超過原因分析

- デフォルト：前月比
- 切替：前日比 / 前週比
- Service / RG / Resource / SKUごとの差分・増加率を算出可能にする

### 6. タグ品質ロジック

必須タグ：

- Owner
- Project
- CostCenter
- Environment
- Department
- Service

実装：

- タグ充足率
- タグ不足リソース数
- 不正値リソース数
- 不足タグ判定
- 許可値に対する不正タグ判定

メイン分析では未設定タグを `Other` / `Untagged` として扱えるようにする。

### 7. 共通フィルター

画面側から以下をフィルター可能なモデルにする。

- Subscription
- Resource Group
- Service
- Resource
- SKU
- Department
- ManagedBy
- Repository
- NetworkMode
- NetworkConfigurationStatus
- Owner
- Project
- CostCenter
- Environment
- Tag

## 対象外

- 7ページの最終レイアウト
- 独自Forecast
- 独自異常検知
- RLS（初期不要）
- Unit Cost
- Invoice統合

## 検証

代表データを使い、以下を手計算または元データと突合する。

- 総コスト
- 月次合計
- 前月差額 / 前月比
- 予算差
- Forecast
- RG負担額
- タグ充足率
- 削減額
- 7日 / 30日比較

## 完了条件

- [ ] Semantic Modelが構築されている
- [ ] 主要分析軸が正しく関連付けられている
- [ ] 共通期間選択を実現できる
- [ ] 主要DAXメジャーが実装されている
- [ ] 前月比較が正しく動く
- [ ] RG負担ルールが正しく動く
- [ ] タグ品質指標が計算できる
- [ ] 変更前後比較の7日 / 30日ロジックが動く
- [ ] 予算超過原因分析用差分が計算できる
- [ ] 最終更新日時を取得できる
- [ ] 代表ケースの数値検証が完了している

## Phase完了時に追記する内容

### 実装結果

未記入

### 変更ファイル

未記入

### モデル構成

未記入

### 主要メジャー

未記入

### 検証結果

未記入

### Phase 05への引き継ぎ

未記入
