# FinOps

Azure中心のFinOps基盤に関する要件・設計ドキュメントです。

## ドキュメント構成

| No. | ドキュメント | ステータス | 内容 |
|---|---|---|---|
| 1 | [全体要件・前提](./01-overall-requirements.md) | ほぼ完了 | 対象範囲、基盤、データ取得、保持、タグ、RG負担ルール、アクセス |
| 2 | [機能要件](./02-functional-requirements.md) | ほぼ完了 | コスト分析、削減候補、配賦、Forecast、シミュレーション、タグ品質等 |
| 3 | [画面・運用要件](./03-screen-operation-requirements.md) | 未完了 | Power BIページ構成、各画面詳細、更新・運用 |

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

### データ更新

- Cost Management Export：日次
- Power BI：日次更新 + 必要時手動更新
- Azure価格データ：週1回
- データ保持：3年
- 直近6か月Hot、以降Cool、さらに古いデータはColdを検討

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

### 完了 / ほぼ完了

- [x] FinOps対象範囲
- [x] 基盤構成
- [x] RG中心 + Subscription対応方針
- [x] Cost Management Export日次更新
- [x] Power BI日次更新方針
- [x] データ保持期間3年
- [x] RG + Tagによる配賦方針
- [x] Ownerの扱い
- [x] タグ品質機能の大枠
- [x] 削減候補の大枠
- [x] Reservation / Savings Planを長期利用割引候補として統合
- [x] コストシミュレーション対象（SKU変更 + 新規リソース）
- [x] Retail Prices API → Storage → Power BI方針
- [x] 予算超過原因分析の粒度
- [x] 変更前後比較の粒度
- [x] Forecastの分析粒度
- [x] 経営向けサマリーの主要項目
- [x] RG月20万円社内負担ルール
- [x] 独自異常検知の初期要件からの削除
- [x] 独自異常通知の初期要件からの削除
- [x] Unit Costの初期対象外化
- [x] Invoice Power BI統合の将来機能化
- [x] Power BIページ構成
- [x] 部門内全RG閲覧方針
- [x] 更新失敗時は前回成功データを表示 + 最終更新日時表示

### 未完了

- [ ] 経営サマリー画面の詳細設計
- [ ] コスト分析画面の詳細設計
- [ ] 削減候補画面の詳細設計
- [ ] タグ品質画面の詳細設計
- [ ] コストシミュレーション画面の入力方式
- [ ] 変更前後比較画面の詳細設計
- [ ] 予算超過原因分析画面の詳細設計
- [ ] 未使用・低利用リソースの具体的判定条件
- [ ] Forecastの具体的計算方式
- [ ] 変更日時の取得・連携方式
- [ ] Storage Cold移行タイミングの最終決定
- [ ] GitHub / Copilot / Azure DevOps費用統合方式
- [ ] Invoice統合方式（将来）

## 次の作業

`03-screen-operation-requirements.md` をベースに、各Power BIページの詳細構成を決定する。

最初は **経営サマリー** から進める。
