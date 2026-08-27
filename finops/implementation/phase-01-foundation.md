# Phase 01 — 基盤・実装骨格

## ステータス

**未着手**

## 目的

後続Phaseでデータ取得・Power BI実装を安全に進められるよう、FinOps実装の骨格とAzure基盤を整える。

このPhaseではPower BI画面や分析ロジックは実装しない。

## 参照要件

- `../01-overall-requirements.md`
- `../02-functional-requirements.md`
- `../03-screen-operation-requirements.md`
- `../TASKS.md`

## 実装対象

### 1. 実装構成の整理

- 既存リポジトリ構成・IaC・CI/CD・命名規則を確認する
- FinOps関連コードの配置先を決定する
- 設定、データ取得処理、Power BI成果物を責務ごとに分離する
- 後続Phaseが独立して作業できる構成にする

### 2. Storage基盤

Cost Management Export、価格データ、Azure補助データを保存できるStorage構成を用意する。

要件：

- 保持期間3年
- 0〜6か月：Hot
- 6か月超〜1年：Cool
- 1年超〜3年：Cold
- 3年経過後：削除
- Lifecycle ManagementはBicepで管理する

保存先はデータ種別が判別でき、後続処理から安定して参照できる構成とする。

### 3. 設定値管理

少なくとも以下をコードから分離して管理できるようにする。

- RG会社負担閾値（設定値として管理）
- 必須タグ
  - Owner
  - Project
  - CostCenter
  - Environment
  - Department
  - Service
- タグ許可値を必要に応じて保持できる構造
- 将来採用候補Resource Type
- データ更新に必要な対象スコープ等

### 4. IaC

- StorageおよびLifecycle設定をBicepで再現可能にする
- 既存の命名・タグ・デプロイ方式を利用する
- 追加リソースが必要な場合は、要件上必要であることを確認してから追加する

## このPhaseで固定しないもの

要件で決まっていない実行基盤を、このPhaseだけの判断で固定しない。

例：

- Azure Functions
- Logic Apps
- Data Factory
- その他の追加課金サービス

後続のデータ取得に実行基盤が必要な場合は、既存基盤を優先し、追加コストと運用負荷が最小になる方式を選択して記録する。

## 対象外

- Cost Management Exportの本処理
- Retail Prices API取得
- Advisor取得
- Activity Log取得
- Power BI Semantic Model
- Power BI画面
- GitHub / Copilot / Azure DevOpsコスト統合
- Invoice統合
- Unit Cost
- 独自異常検知・通知

## 完了条件

- [ ] FinOps実装コードの配置が決定している
- [ ] Storage基盤がIaCで定義されている
- [ ] Lifecycle Managementが3年保持方針どおり定義されている
- [ ] RG負担閾値を設定値として管理できる
- [ ] 必須タグ・タグ許可値を管理できる
- [ ] 将来候補Resource Typeを設定できる
- [ ] 既存CI/CDまたはデプロイ手順で基盤を再現できる
- [ ] 不要な追加Azureサービスを導入していない

## Phase完了時に追記する内容

### 実装結果

未記入

### 変更ファイル

未記入

### 設計判断

未記入

### Phase 02への引き継ぎ

未記入
