# Azure Storage — AZ-305 試験対策

> 最新の Microsoft Learn の仕様を基準に、AZ-305 でストレージを選択するための判断ポイントを整理する。

## 1. まず覚える全体像

Azure Storage の問題では、次の順番で考える。

1. **何を保存するか** → Blob / Files / Queue / Table
2. **性能要件** → Standard / Premium
3. **アクセス頻度** → Hot / Cool / Cold / Archive
4. **どこまでの障害に耐えるか** → LRS / ZRS / GRS / GZRS
5. **セカンダリから読みたいか** → RA-GRS / RA-GZRS

---

## 2. Storage サービスの違い

| サービス | 保存対象 | AZ-305 のキーワード |
|---|---|---|
| **Blob Storage** | 画像、動画、ログ、バックアップ、非構造化データ | オブジェクト、大容量、静的データ |
| **Azure Files** | ファイル共有 | SMB / NFS、共有フォルダー、オンプレ互換 |
| **Queue Storage** | メッセージ | 非同期処理、疎結合、大量メッセージ |
| **Table Storage** | NoSQL Key-Value | シンプルな NoSQL、低コスト |
| **ADLS Gen2** | 分析用データレイク | Blob + Hierarchical Namespace、ACL、分析 |

### Blob と Files の判断

- アプリが **オブジェクトとして画像・ログ等を保存** → Blob
- 複数サーバーから **共有フォルダーとしてマウント** → Azure Files

---

## 3. ストレージアカウント種類

### Standard general-purpose v2（StorageV2）

基本はこれ。

- Blob
- Files
- Queue
- Table
- ADLS Gen2

を利用できる。

**AZ-305:** 特別な性能要件がなければ GPv2 を第一候補にする。

### Premium Block Blob

- Block Blob / Append Blob 向け
- SSD ベース
- 高トランザクション
- 小さいオブジェクトを大量処理
- 一貫して低レイテンシが必要
- 冗長性は基本 **LRS / ZRS**

### Premium File Shares（FileStorage）

- Azure Files 専用
- SMB / NFS
- 高 IOPS・低レイテンシ
- エンタープライズ、高性能ファイル共有
- 冗長性は基本 **LRS / ZRS**

### Premium Page Blob

- Page Blob 専用
- 高性能なランダム読み書き用途
- AZ-305 では Block Blob / Files より優先度低め

### 試験での覚え方

**Standard = 汎用性・コスト・Geo 冗長性**

**Premium = 高性能・低レイテンシ。ただし冗長性の選択肢が狭い**

---

## 4. Blob のアクセス層

| Tier | 用途 | 最低保持期間の目安 | 読み出し | コスト傾向 |
|---|---|---:|---|---|
| **Hot** | 頻繁にアクセス | なし | 即時 | 保存高 / アクセス安 |
| **Cool** | 低頻度 | 30日 | 即時 | 保存↓ / アクセス↑ |
| **Cold** | かなり低頻度 | 90日 | 即時 | 保存さらに↓ / アクセスさらに↑ |
| **Archive** | 長期保管 | 180日 | **数時間** | 保存最安 / 取得最高 |

### 超重要

**Hot / Cool / Cold は Online。Archive だけ Offline。**

Archive はアクセス前に **rehydration（再水和）** が必要。

### Archive の重要制限

Archive がサポートする冗長性は基本：

- LRS
- GRS
- RA-GRS

**ZRS / GZRS / RA-GZRS では Archive を使用できない。**

### AZ-305 判断

- 頻繁な読み書き → Hot
- 月に数回程度 → Cool
- ほぼ読まないが即時取得必要 → Cold
- 法令保存・長期バックアップ・数時間待てる → Archive

---

## 5. 冗長性 — 最重要

### LRS（Locally Redundant Storage）

同一リージョン内の単一物理ロケーションで複製。

**守れる:** ディスク/ラック等の局所障害

**守れない:** AZ / リージョン規模の障害

→ **最安**

### ZRS（Zone-Redundant Storage）

同一リージョンの複数 Availability Zone に同期複製。

**守れる:** AZ 障害

**守れない:** リージョン全体の障害

→ 「データを別リージョンへ出したくない + AZ障害対策」で有力。

### GRS（Geo-Redundant Storage）

プライマリリージョンから、ペアとなるセカンダリリージョンにも複製。

プライマリ側は LRS ベース。

**守れる:** リージョン障害

→ AZ 耐障害性まで要求されるなら GZRS を検討。

### GZRS（Geo-Zone-Redundant Storage）

**ZRS + Geo replication** と覚える。

プライマリ：複数 AZ

＋

セカンダリ：別リージョン

**守れる:** AZ 障害 + リージョン障害

→ 高い回復性が必要な Standard Storage の有力候補。

---

## 6. RA が付く意味

- GRS → セカンダリへの複製はあるが、通常時に直接読み取る用途ではない
- **RA-GRS** → セカンダリを読み取り可能
- GZRS → Geo + Zone
- **RA-GZRS** → Geo + Zone + セカンダリ読み取り可能

**RA = Read Access**

### 覚え方

`RA が付いたら「裏側のセカンダリコピーを通常時にも読める」`

---

## 7. 一発判断表

| 問題文 | 選択候補 |
|---|---|
| コスト最小、局所障害だけ対策 | **LRS** |
| AZ障害に耐える | **ZRS** |
| リージョン障害に耐える | **GRS / GZRS** |
| AZ + リージョン障害 | **GZRS** |
| 別リージョンのコピーを通常時にも読む | **RA-GRS / RA-GZRS** |
| 高トランザクション Blob、低レイテンシ | **Premium Block Blob** |
| 高性能 SMB/NFS ファイル共有 | **Premium File Shares** |
| 一般的なストレージ | **Standard GPv2** |
| 頻繁に読む Blob | **Hot** |
| 低頻度だが即時アクセス | **Cool / Cold** |
| 長期保存、取得に数時間かかってよい | **Archive** |
| データレイク、フォルダー ACL | **ADLS Gen2 / HNS** |

---

## 8. ADLS Gen2 と Hierarchical Namespace

ADLS Gen2 は独立した全く別のストレージサービスと考えるより、**Blob Storage にデータ分析向け機能を追加したもの**と理解する。

重要なのが **Hierarchical Namespace（階層型名前空間 / HNS）**。

有効化すると、ディレクトリ/ファイル階層を効率的に扱え、**POSIX ライクな ACL** を利用できる。

### AZ-305

問題文に以下があれば ADLS Gen2 を疑う。

- Data Lake
- Big Data analytics
- Hadoop / Spark / Databricks
- フォルダー/ファイル単位 ACL
- Hierarchical Namespace

---

## 9. Standard と Premium の冗長性

| Account type | 主な冗長性 |
|---|---|
| **Standard GPv2** | LRS / ZRS / GRS / RA-GRS / GZRS / RA-GZRS |
| **Premium Block Blob** | LRS / ZRS |
| **Premium File Shares** | LRS / ZRS |
| **Premium Page Blob** | LRS（利用可能な構成は最新リージョン仕様も確認） |

> ZRS 等の利用可否はリージョンにも依存する。試験問題でリージョンが明示される場合は注意。

---

## 10. Blob の種類

### Block Blob

最も一般的。

- 画像
- 動画
- ドキュメント
- バックアップ
- 一般的なオブジェクト

### Append Blob

末尾にデータを追加する用途。

→ **ログ**を連想。

### Page Blob

ランダム読み書き向け。

→ **VHD** を連想。

### 覚え方

- Block = 普通のファイル
- Append = ログ
- Page = VHD

---

## 11. よくある引っかけ

### 「Premiumなら最強」は間違い

Premium は性能面では強いが、GRS/GZRS 等を利用できないアカウント種類がある。

問題が **リージョン障害対策** を要求するなら、性能だけで Premium を選ばない。

### 「Archiveは安いCool」ではない

Archive は **Offline**。

即時アクセス要件があるなら選べない。

### 「GRSならいつでもセカンダリを読める」は間違い

通常時にセカンダリを読みたいなら **RA-GRS**。

### 「ZRSならリージョン障害にも耐える」は間違い

ZRS は **同一リージョン内の複数AZ**。

リージョン障害なら **GRS/GZRS**。

### 「Storage Account = Blob Storage」ではない

Storage Account は入れ物。Standard GPv2 の中で Blob / Files / Queue / Table 等を利用できる。

---

## 12. 30秒暗記

```text
何を保存？
├─ Object → Blob
├─ File share → Azure Files
├─ Message → Queue
├─ NoSQL Key-Value → Table
└─ Analytics + ACL → ADLS Gen2

性能？
├─ 普通 → Standard GPv2
└─ 高IOPS/低Latency → Premium

アクセス頻度？
Hot → Cool → Cold → Archive
頻繁 --------------------→ ほぼ読まない
即時 --------------------→ Archiveだけ数時間

障害範囲？
LRS   = Local
ZRS   = Zone
GRS   = Geo
GZRS  = Zone + Geo
RA-*  = Secondary Read Access
```

---

## 13. 現行 AZ-305 での優先度

**最優先で覚える:**

- Blob vs Files
- Standard vs Premium
- Hot / Cool / Cold / Archive
- LRS / ZRS / GRS / GZRS / RA系
- ADLS Gen2 + HNS + ACL
- コスト・可用性・リージョン障害からの選定

**細かすぎる SKU 数値や IOPS 上限値:**

AZ-305 では変更されやすい数値の丸暗記より、**要件から適切なストレージ設計を選ぶ判断基準**を優先する。

---

## 参考（Microsoft Learn）

- Storage Account overview
- Azure Storage redundancy
- Blob access tiers
- Azure Data Lake Storage Gen2 documentation

最終確認: 2026-08-30。仕様変更があり得るため、試験直前は Microsoft Learn の最新情報を優先する。
