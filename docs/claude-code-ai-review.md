# Claude Code AI Review

## 1. 目的

Azure DevOps の PR 作成・更新時に Claude Code を利用して AI レビューを実行する。

目的は、通常の静的解析だけでは検出しにくい以下を確認すること。

- 実装方針の妥当性
- アーキテクチャ違反
- セキュリティ上の問題
- 認証・認可の不備
- 変更内容とテスト変更の不整合
- プロジェクト固有ルール違反
- 保守性・可読性上の問題

---

## 2. 実行フロー

```text
PR作成 / 更新
    ↓
Azure DevOps Pipeline
    ↓
Self-hosted Agent
    ↓
AI Reviewer Container
    ↓
Claude Code
    ↓
レビュー結果生成
    ↓
PRコメント + Pipeline判定
```

---

## 3. Claude Codeへの入力

Claude Codeは以下を参照する。

- PRタイトル
- PR説明
- Git diff
- 変更対象コード
- 必要に応じて周辺コード
- テストコードの変更
- `.claude/` 配下のプロジェクトルール

Secret、Credential、不要な環境変数などはレビュー入力から除外する。

---

## 4. `.claude` 構成

```text
.claude/
├─ project-rules.md
├─ architecture.md
├─ security-rules.md
└─ review-rules.md
```

### project-rules.md

- コーディング規約
- 命名規則
- プロジェクト共通ルール
- 禁止事項

### architecture.md

- アーキテクチャ方針
- レイヤー間の依存ルール
- API設計方針
- DB設計方針
- Azure利用方針

### security-rules.md

- 認証
- 認可
- Secret管理
- 入力検証
- ログ
- セキュアコーディング

### review-rules.md

- AIレビュー基準
- Severity定義
- Pipeline停止条件
- Human Review条件
- エスカレーション条件

---

## 5. レビュー対象

Claude Codeは主に以下を確認する。

### 実装

- 要件に沿っているか
- 不要な複雑化がないか
- 明らかなバグがないか
- 既存設計との整合性があるか

### アーキテクチャ

- レイヤー違反
- 不適切な依存
- 責務の混在
- 共通機能の重複実装

### セキュリティ

- 認証漏れ
- 認可漏れ
- 権限昇格
- IDOR / BOLA
- Secret露出
- 入力検証不足
- インジェクション
- ログへの機密情報出力

### テスト

テスト実行そのものは既存CIに任せる。

Claude Codeでは以下を確認する。

- 実装変更に対してテスト変更が不足していないか
- 不自然にテストが削除されていないか
- 重要ロジック変更にテストが追加されているか
- テストが実装内容と一致しているか

---

## 6. Severity

| Severity | 処理 |
|---|---|
| Critical | Pipeline FAIL + Human Review必須 |
| High | Pipeline FAIL + Human Review必須 |
| Medium | Pipeline PASS + 指摘 |
| Low | Pipeline PASS + 指摘 |
| None | Pipeline PASS |

Critical / High の具体的な条件は `review-rules.md` に定義する。

---

## 7. レビュー結果

### 人間向け

Azure DevOps PRへコメントする。

```text
AI Review: HIGH

Finding:
Authorization check may be bypassed.

File:
src/api/project.ts:84

Reason:
Project ownership is not validated.

Recommendation:
Validate project access before update.

Decision:
Human review required.
```

### Pipeline向け

構造化JSONを出力する。

```json
{
  "result": "escalate",
  "maxSeverity": "high",
  "findings": [
    {
      "severity": "high",
      "file": "src/api/project.ts",
      "line": 84,
      "category": "authorization",
      "message": "Project ownership is not validated."
    }
  ]
}
```

Pipelineは `maxSeverity` を利用して PASS / FAIL を決定する。

---

## 8. 基本方針

- Claude Codeは人間レビューの完全な代替にはしない
- Critical / High は必ず人へエスカレーションする
- AIの自由判断だけに依存せず `.claude/` のレビュー規約を基準にする
- テスト実行、Lint、SAST、SCAなどの既存CIとは役割を分ける
- AIはコードの意味・設計・セキュリティ・変更意図のレビューを担当する
- レビュー結果はPRコメントと構造化JSONの両方で出力する
