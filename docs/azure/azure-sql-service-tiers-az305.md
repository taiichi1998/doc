# Azure SQL サービスティア選択ガイド（AZ-305）

> 更新基準: 2026-08-30 / Microsoft Learn の現行仕様を基準

AZ-305 で Azure SQL の選択肢を判断するためのチートシート。古い問題では `Standard / Premium` が中心だが、現在は **Azure SQL Database の vCore ベース**も重要。

## 1. 最初に製品を選ぶ

| 製品 | 選ぶ条件 | AZ-305での判断 |
|---|---|---|
| Azure SQL Database | クラウドネイティブな単一DB / Elastic Pool | 原則として最初の候補。管理負荷を最小化 |
| Azure SQL Managed Instance | SQL Serverとの高い互換性、インスタンスレベル機能が必要 | SQL Agent、インスタンス機能、移行互換性が重要なら候補 |
| SQL Server on Azure VM | OS / SQL Serverを最大限制御する必要がある | PaaSで満たせない機能・完全な制御が必要な場合 |

---

# 2. Azure SQL Database

## 現在の購入モデル

### vCore ベース（現行AZ-305で優先して理解）

| Service tier | 主用途 | 性能 / 可用性 | Zone redundancy | Serverless | 判断キーワード |
|---|---|---|---|---|---|
| General Purpose | 一般的な業務DB、コスト重視 | バランス型 | 対応 | 対応 | cost、general workload、serverless |
| Business Critical | 高トランザクション、低I/Oレイテンシ、重要OLTP | 複数の分離レプリカ、高可用性 | 対応 | 基本はProvisioned | low latency、high IOPS、HA、OLTP |
| Hyperscale | 大規模・急成長DB、高いスケーラビリティ | Compute/Storageを独立スケール、複数レプリカ | 対応 | 対応 | huge database、rapid growth、scale、read replicas |

### DTU ベース（古い問題でも頻出）

| Service tier | 主用途 | Zone redundancy | AZ-305での位置付け |
|---|---|---|---|
| Basic | 小規模・低負荷 | 非対応 | 最小コスト、非常に軽いワークロード |
| Standard | 一般用途 | 非対応 | コスト重視の旧来DTU問題 |
| Premium | 高性能OLTP | 対応 | 高IOPS・低レイテンシ・ゾーン冗長 |

**重要:** Basic / Standard はゾーン冗長をサポートしない。DTUモデルでゾーン冗長が必要なら Premium。

---

# 3. vCore と DTU の違い

## vCore

- CPU（vCore）、メモリ、ストレージをより明示的に選択
- Azure Hybrid Benefit を利用可能な構成がある
- General Purpose / Business Critical / Hyperscale
- 現在の設計問題ではこちらを中心に考える

## DTU

- CPU・メモリ・I/Oをまとめた DTU という単位
- Basic / Standard / Premium
- シンプルだが細かなリソース選択はしにくい
- 古いAZ-305問題では非常に多い

### 対応イメージ

```text
DTU                    vCore
Basic / Standard  ≒    General Purpose
Premium           ≒    Business Critical
                      Hyperscale（大規模向け）
```

※ 完全な1対1対応ではなく、試験で用途を整理するためのイメージ。

---

# 4. Provisioned と Serverless

## Provisioned compute

常時一定のComputeを確保する。

選ぶ条件:
- 継続的な負荷
- 予測可能な性能が必要
- 常時稼働する本番DB

## Serverless compute

負荷に応じてComputeを自動スケールし、使用量ベースで課金。

選ぶ条件:
- 負荷が断続的
- 開発 / テスト
- アイドル時間が長い
- コスト最小化

現行では Azure SQL Database の **General Purpose と Hyperscale** で serverless を利用可能。

---

# 5. 高可用性・Availability Zone

現在の Azure SQL Database では、vCoreモデルの以下がゾーン冗長に対応する。

- General Purpose
- Business Critical
- Hyperscale

DTUモデルでは:

- Premium: 対応
- Standard: 非対応
- Basic: 非対応

ゾーン冗長構成では、単一Availability Zone障害時のフェールオーバーについて **committed data の RPO = 0** を実現する。

### AZ-305注意点

古い問題で

> Zone failure + no data loss

という条件から「Premiumしかない」と暗記しない。

**現在は vCore の General Purpose / Business Critical / Hyperscale も候補になる。**

---

# 6. General Purpose vs Business Critical vs Hyperscale

## General Purpose

選ぶ:
- 一般的なWeb / 業務アプリ
- コストを抑えたい
- 特別な低レイテンシ要件がない
- Zone redundancy が必要だが Business Critical の性能までは不要

AZ-305:

> `minimize cost` + `zone redundancy` だけなら、現在は General Purpose を必ず候補に入れる。

## Business Critical

選ぶ:
- 高トランザクションOLTP
- 非常に低いI/O latency
- 高IOPS
- 高い可用性
- Read scale-out
- In-Memory OLTPなどBC/Premium固有機能が必要

AZ-305キーワード:

`low latency` / `high transaction rate` / `high IOPS` / `mission critical OLTP`

## Hyperscale

選ぶ:
- DBサイズが非常に大きい
- データが急速に増える
- ComputeとStorageを独立してスケールしたい
- 多数の読み取りレプリカが必要
- 大規模DBで高速なスケールが必要

現行仕様では最大 **128 TB** のデータをサポート。

AZ-305:

> 「高可用性だからHyperscale」ではない。

大規模性・スケーラビリティ要件がなければ、通常は General Purpose / Business Critical を先に検討。

---

# 7. Azure SQL Managed Instance

Managed Instance は Azure SQL Database の上位版というより、**SQL Server互換性を重視したPaaS**。

現在の主なサービスティア:

| Tier | 用途 | 判断 |
|---|---|---|
| General Purpose | 一般的なMIワークロード | コスト重視 |
| Next-gen General Purpose | GPより高性能・高い柔軟性が必要 | GPのアーキテクチャアップグレード |
| Business Critical | 高性能・低レイテンシ・高可用性 | ミッションクリティカル |

## Next-gen General Purpose

2026年時点で特に注意。

- 従来 General Purpose のアーキテクチャアップグレード
- Elastic SAN ベース
- 従来GPより高いI/O性能・柔軟性
- 最大500 DB / instance
- 最大32 TB storage
- ベースラインコストはGeneral Purposeと同等という位置付け
- Zone-redundant Next-gen General Purpose は 2026-08 時点で Preview

AZ-305では Preview 機能を前提とした正解には慎重になる。

---

# 8. 試験での即決フロー

```text
SQL Serverとの高い互換性 / Instance機能？
 ├─ Yes → Managed Instance
 │          ├─ 一般用途 → General Purpose / Next-gen GP
 │          └─ 高性能・低Latency → Business Critical
 │
 └─ No → Azure SQL Database
            |
            ├─ 超大規模 / 急成長 / Scale-out → Hyperscale
            |
            ├─ 高IOPS / 低Latency / Mission Critical OLTP
            |      → Business Critical
            |
            └─ 一般用途 / Cost重視
                   → General Purpose
```

DTU形式の選択肢しかない場合:

```text
低負荷       → Basic
一般用途     → Standard
高性能 / HA → Premium
```

---

# 9. AZ-305で間違えやすいポイント

### 「minimize cost」だけで Standard を選ばない

必須要件を満たす候補を残した後、その中で最も低コストなものを選ぶ。

### Zone redundancy = Premium ではない

これは古い問題で起こりやすい判断。

現行vCoreでは General Purpose / Business Critical / Hyperscale が対応。

### Hyperscale = Business Criticalの上位互換ではない

目的が違う。

- Business Critical → latency / IOPS / OLTP / HA
- Hyperscale → database size / scalability / read scale

### Managed Instance = SQL Databaseの高性能版ではない

MIを選ぶ主理由は SQL Serverとの互換性・インスタンスレベル機能。

---

# 10. 最短暗記

```text
General Purpose
→ 普通・安い・まず候補

Business Critical
→ 高IOPS・低Latency・重要OLTP

Hyperscale
→ 超巨大・急成長・Scale

Serverless
→ 断続負荷・Idle多い・Cost削減

Managed Instance
→ SQL Server互換性・Instance機能

DTU Standard
→ 一般用途だがZone redundancyなし

DTU Premium
→ 高性能 + Zone redundancy
```

---

## Microsoft公式参照

- Azure SQL Database overview: https://learn.microsoft.com/azure/azure-sql/database/sql-database-paas-overview
- Purchasing models: https://learn.microsoft.com/azure/azure-sql/database/purchasing-models
- Zone redundancy: https://learn.microsoft.com/azure/azure-sql/database/enable-zone-redundancy
- SQL Database reliability: https://learn.microsoft.com/azure/reliability/reliability-sql-database
- SQL Managed Instance service tiers: https://learn.microsoft.com/azure/azure-sql/managed-instance/service-tiers-managed-instance-vcore
- Next-gen General Purpose: https://learn.microsoft.com/azure/azure-sql/managed-instance/service-tiers-next-gen-general-purpose-use

> Azure SQL の機能・対応リージョン・Preview/GA状態は更新されるため、試験直前には最新の Microsoft Learn / AZ-305 Study Guide を再確認すること。
