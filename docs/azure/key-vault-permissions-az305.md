# Azure Key Vault 権限まとめ【AZ-305】

## 1. Key Vaultで管理する3種類

| 種類 | 用途 | 例 |
|---|---|---|
| Secrets | 秘密情報 | DBパスワード、API Key、接続文字列 |
| Keys | 暗号化用の鍵 | RSA Key、暗号化キー |
| Certificates | 証明書 | TLS/SSL証明書 |

> Secrets / Keys / Certificates は、それぞれ異なる権限を持つ。

---

## 2. Secrets の主要権限

| 権限 | 意味 |
|---|---|
| Get | 指定したSecretを取得 |
| List | Secretの一覧を取得 |
| Set | Secretを作成・更新 |
| Delete | Secretを削除 |
| Recover | 削除されたSecretを復元 |
| Backup | Secretをバックアップ |
| Restore | Secretを復元 |

### Get と List の違い

- **Get**：特定のSecretを読む
- **List**：Key Vaultに存在するSecretを列挙する

「すべてのSecretを処理する」場合は、一覧を取得するため `List` が必要。

---

## 3. Secretを別Key Vaultへコピー

KV1の全SecretをKV2へ移行する場合：

```text
KV1
 │
 ├─ List → Secret一覧を取得
 │
 └─ Get  → 各Secretの値を取得
              │
              ↓
             KV2
              │
              └─ Set → Secretを書き込む
```

| Key Vault | 必要な権限 |
|---|---|
| 移行元 KV1 | Get + List |
| 移行先 KV2 | Set |

### 覚え方

```text
Source                    Destination

List + Get      →            Set
一覧 + 読む                   書く
```

---

## 4. Keys の主要権限

| 権限 | 意味 |
|---|---|
| Get | Key情報を取得 |
| List | Key一覧を取得 |
| Create | Keyを作成 |
| Delete | Keyを削除 |
| Encrypt | Keyを使用してデータを暗号化 |
| Decrypt | Keyを使用してデータを復号 |
| Sign | デジタル署名を作成 |
| Verify | デジタル署名を検証 |
| Wrap Key | 別のKeyを暗号化 |
| Unwrap Key | 暗号化されたKeyを復号 |

---

## 5. Encrypt / Decrypt

データそのものを暗号化・復号する。

```text
平文データ
   │
   │ Encrypt
   ↓
暗号化データ
   │
   │ Decrypt
   ↓
平文データ
```

```text
Encrypt / Decrypt
        ↓
「データ」に対する暗号化・復号
```

---

## 6. Wrap Key / Unwrap Key

「別の暗号化キー」を暗号化・復号する。

```text
Data Encryption Key（DEK）
        │
        │ Wrap Key
        ↓
暗号化されたDEK
        │
        │ Unwrap Key
        ↓
元のDEK
```

```text
Encrypt / Decrypt
→ データを暗号化・復号

Wrap / Unwrap
→ 鍵を暗号化・復号
```

---

## 7. Certificates の主要権限

| 権限 | 意味 |
|---|---|
| Get | Certificateを取得 |
| List | Certificate一覧を取得 |
| Create | Certificateを作成 |
| Import | 既存Certificateをインポート |
| Delete | Certificateを削除 |

例：既存のPFX証明書をKey Vaultに格納する場合は `Import`。

AZ-305では、Keysほど細かい操作を覚える優先度は高くない。

---

## 8. Access Policy と Azure RBAC

Key Vaultのアクセス制御には大きく2方式ある。

### Key Vault Access Policy

従来方式。

```text
VM1
 ↓
Key Vault Access Policy
 ↓
Secrets
├─ Get
├─ List
└─ Set
```

古いAZ-305問題では、Get / List / Set / Wrap / Unwrap などの個別権限を直接選ばせる問題が多い。

### Azure RBAC

現在の新規設計では基本的にこちらを優先。

```text
VM1
 ↓
Managed Identity
 ↓
Azure RBAC
 ↓
Key Vault Secrets User
 ↓
Key Vault
```

代表的なKey Vault RBACロール：

| ロール | 用途 |
|---|---|
| Key Vault Reader | メタデータ参照 |
| Key Vault Secrets User | Secretの値を読む |
| Key Vault Secrets Officer | Secretを管理 |
| Key Vault Crypto User | Keyを使った暗号処理 |
| Key Vault Crypto Officer | Keyを管理 |
| Key Vault Certificates Officer | Certificateを管理 |
| Key Vault Administrator | Key Vaultデータを広範囲に管理 |

---

## 9. AZ-305 最重要暗記

```text
Secrets

Get  = 1個読む
List = 一覧を見る
Set  = 書く
```

```text
Keys

Encrypt = データを暗号化
Decrypt = データを復号

Wrap   = 鍵を暗号化
Unwrap = 鍵を復号
```

```text
全Secretを別Key Vaultへコピー

移行元                 移行先

Get + List     →        Set
```

```text
アクセス制御

Access Policy = 従来方式
Azure RBAC    = 現在の新規設計で基本的に推奨
```

### 試験での判断

- 「特定Secretを読む」→ Get
- 「全Secretを処理」→ List + Get
- 「Secretを書き込む」→ Set
- 「データを暗号化」→ Encrypt
- 「データを復号」→ Decrypt
- 「暗号化キーを包む」→ Wrap Key
- 「包まれたキーを取り出す」→ Unwrap Key
- 「新規Key Vaultのアクセス制御方式を設計」→ Azure RBACを優先
