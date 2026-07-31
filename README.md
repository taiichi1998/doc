# Claude Code 標準テンプレート

## 目的

Claude Codeを利用した開発において、

- コンテキストを最小限に抑える
- 品質を標準化する
- AIが迷わない開発フローを実現する

ことを目的とする。

---

# 1. CLAUDE.md（必須）

## 役割

プロジェクト全体で毎回必要となる情報のみ記載する。

### 記載内容

- プロジェクト概要
- 基本ルール
- ビルド・テストコマンド
- 開発フロー
- 禁止事項

### 方針

- 短く保つ
- 詳細な手順は記載しない
- 技術固有のルールは Rules へ分離する
- 作業手順は Skills へ分離する

---

# 2. Rules（必須）

```text
.claude/
└── rules/
    ├── frontend.md
    ├── backend.md
    ├── database.md
    ├── infrastructure.md
    ├── testing.md
    └── security.md
```

## 役割

編集対象に応じたルールのみ読み込む。

### 例

- React → frontend.md
- API → backend.md
- SQL → database.md
- Azure → infrastructure.md

---

# 3. Skills（必須）

```text
.claude/
└── skills/
    ├── implement-feature/
    ├── implement-api/
    ├── implement-database-change/
    ├── provision-azure-resource/
    ├── review-pull-request/
    └── verify-requirements/
```

## 役割

作業手順を標準化する。

### 主な Skill

- 機能実装
- API実装
- DB変更
- Azureリソース作成
- 要件確認
- PRレビュー

---

# 4. Plan Mode（必須）

コードを書く前に必ず実装計画を立てる。

## 標準フロー

```text
要件理解
    ↓
既存実装調査
    ↓
実装計画
    ↓
実装開始
```

---

# 5. Worktree（必要時のみ）

## 利用目的

並列開発を行う場合のみ使用する。

## 標準

```text
1タスク
    ↓
1ブランチ
    ↓
1 Worktree
    ↓
1 Claude Code セッション
```

## 利用例

- 機能開発を並列で進める
- Hotfix対応
- AIセッションの分離

> 通常の順次開発では利用しない。

---

# 6. Subagent（必須）

Subagentはレビュー専用とする。

## 利用する Subagent

```text
code-reviewer
test-reviewer
security-reviewer
architecture-reviewer
```

## レビュー結果

```text
Critical
High
Medium
Low
```

> 実装は担当しない。

---

# 7. Hooks（必須）

危険な操作を防止する。

## 例

- Force Push
- Hard Reset
- Azureリソース削除
- Secret参照
- 危険なSQL実行

> 品質チェックは CI/CD 側で実施する。

---

# 8. 機能単位（縦切り）実装（必須）

1機能を最後まで完成させる。

```text
要件
    ↓
DB
    ↓
API
    ↓
画面
    ↓
テスト
```

> 画面だけ、APIだけ、といった横割り実装は行わない。

---

# 9. 品質確認（必須）

```text
テスト
    ↓
実装
    ↓
ローカル検証
    ↓
AIレビュー
    ↓
人レビュー
```

## ローカル検証項目

- Format
- Lint
- Type Check
- Unit Test
- Integration Test
- Build

---

# 10. コンテキスト管理（必須）

## セッション開始時

必要な資料のみ読み込む。

## 開発中

現在の作業に関係する情報のみ利用する。

## 長時間作業

`/compact` を実行し、コンテキストを整理する。

## 新しいタスク

`/clear` を実行し、新しいセッションとして開始する。

---

# 標準開発フロー

```text
要件・ユーザーストーリー
        ↓
Plan Mode
        ↓
必要なドキュメントを読む
        ↓
必要な Rules を読む
        ↓
Skill 実行
        ↓
機能実装
        ↓
ローカル検証
        ↓
Subagent レビュー
        ↓
Critical・Highを修正
        ↓
人レビュー
        ↓
Pull Request
```

---

# リポジトリ構成

```text
repo/
├── CLAUDE.md
├── .claude/
│   ├── rules/
│   ├── skills/
│   ├── agents/
│   └── settings.json
├── docs/
├── apps/
├── services/
├── database/
├── infra/
└── tests/
```

---

# 基本原則

- CLAUDE.mdは最小限に保つ
- 必要な情報だけ読み込む
- 機能単位で最後まで実装する
- 実装とレビューを分離する
- 既存実装を優先して再利用する
- 品質はCI/CDで担保する
- Worktreeは並列開発時のみ利用する
