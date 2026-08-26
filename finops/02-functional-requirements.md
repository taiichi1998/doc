# FinOps 機能要件

## ステータス

**ほぼ完了**

初期機能の大枠は確定済み。画面上の見せ方や一部の判定ロジック詳細は未確定。

## 1. 標準コスト分析

Azure Cost Management / Cost Analysisで確認できる基本情報をPower BIでも分析可能とする。

### 表示・分析軸

- 日別コスト
- 月別コスト
- Subscription別
- Resource Group別
- Service別
- Resource別
- SKU別
- Tag別

### タグ分析

- Department
- Project
- CostCenter
- Owner
- Environment
- Service

タグ未設定は `Other` または `Untagged` として集約する。

## 2. 予算・実績・Forecast

以下を表示する。

- 予算
- 実績
- 差分
- Forecast

### Forecast粒度

- RG単位
- Department単位
- Project単位

RGを基本として、Department / Projectでも絞り込み・比較可能とする。

## 3. 削減候補

削減候補を以下の3分類で表示する。

### 3.1 未使用リソース

削除候補となる未使用リソースを特定する。

### 3.2 低利用リソース

利用率が低く、SKUダウンなどのRight Sizing候補となるリソースを特定する。

### 3.3 長期利用割引候補

Reservation / Savings Planをまとめて長期利用割引候補として扱う。

表示内容：

- 候補の有無
- 対象リソース
- 想定削減額
- 1年契約時の効果
- 3年契約時の効果
- 削減効果比較

### 判定元

初期は以下を利用する。

- Azure Advisor
- Azure Cost Management

CPU使用率等を使った独自高度判定は初期必須としない。

## 4. 配賦

RG + Tagを使用してコストを配賦する。

主要配賦軸：

- Department
- Project
- CostCenter
- Owner

Service / Environmentは主に分析・フィルタ用途として扱う。

## 5. コストシミュレーション

初期から実装する。

### 対象

- SKU変更
- 新規リソース追加

例：

- App Service P1v3 → P2v3
- Azure SQLのSKU変更
- 新規App Service追加
- 新規Storage Account追加

### 表示内容

- 現在価格
- 変更後価格
- 月額差分
- 年額差分

### 価格取得

- Azure Retail Prices APIを使用
- APIから取得した価格をStorageへ保存
- Power BIは保存済み価格データを参照
- 更新頻度：週1回

## 6. タグ / メタデータ品質

専用ページを用意する。

### チェック内容

- 必須タグの有無
- タグ値の妥当性

例：

- Ownerが未設定
- CostCenterが未設定
- Environmentが許可された値以外
- Serviceが未設定

### 表示方針

メイン分析では未設定タグを `Other / Untagged` として扱い、タグ品質ページで不足リソースを詳細確認する。

## 7. 予算超過の原因分析

残す機能。

RGの予算超過やコスト増加が発生した場合、増加要因を以下の粒度まで追跡可能にする。

- Service
- RG
- Resource
- SKU

Tag単位までの原因追跡は初期必須としない。

### 分析イメージ

前月・前日・基準期間との差分を比較し、増加額の大きいService / Resource / SKUを特定する。

## 8. 変更前後比較

SKU変更やデプロイ等の変更前後でコスト差を確認する。

### 表示内容

- 変更前総額
- 変更後総額
- 総額差分
- Resource別差分
- SKU別差分

Project / Tag別影響分析は初期必須としない。

## 9. 社内負担額分析

会社のRG月額20万円ルールを反映する。

### 表示内容

- RG実コスト
- 会社負担判定
- 部門実質負担額
- 20万円までの残額
- 閾値超過RG

### 判定

`RG月額コスト <= 設定閾値` の場合：実質負担額0円

`RG月額コスト > 設定閾値` の場合：実コスト全額を実質負担額として表示

## 10. 経営向けサマリー

1画面で以下を確認可能とする。

- 総コスト
- 予算差
- Forecast
- 実質負担額
- 主な増加要因
- 削減候補
- Department別ランキング
- Project別ランキング

## 11. 初期対象外機能

### 異常検知

独自実装しない。

Azure Cost Management側で既にコストアラートを設定しているため、Power BI側での異常検知は重複機能として削除する。

### コスト異常通知

独自のTeams / メール通知機能は初期実装しない。

Azure Cost Management側のコストアラートを利用する。

### Unit Cost

初期対象外。

ユーザー1人あたり、案件1件あたり等のUnit Cost算出にはEntra ID、アプリ利用ログ、業務DB等の追加データソースが必要になるため、将来拡張とする。

### Invoice / 請求情報

Power BIへの統合は将来機能とする。

初期はAzure Cost Management側で確認する。

## 未完了事項

- 未使用・低利用リソースの具体的判定条件
- Reservation / Savings Planの1年・3年比較ロジック詳細
- 予算超過原因分析で使用する比較期間の最終決定
- 変更前後比較における「変更日時」の取得方法
- Forecastの具体的計算方式（Azure標準値利用か独自計算か）
