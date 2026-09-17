# OpenSpec

## 用途

AIコーディングを仕様駆動にし、実装前に変更内容・要件・設計意図を明文化する。

## メリット

- Claude Codeへの毎回の説明を減らしやすい
- 要件と実装のズレを抑える
- 仕様をレビューしてからコード生成へ進める
- AIセッションを跨いでも意図を残しやすい

## このプロジェクトでの使い方

```text
要件 / User Story
  ↓
OpenSpecで変更仕様を定義
  ↓
人が仕様レビュー
  ↓
Claude Codeで実装
  ↓
Unit / Integration / E2E
  ↓
PRレビュー
```

特にDB、API、Frontend、IaCを縦切りで実装する機能変更に向く。

## CLAUDE.md / Rules / Skillsとの違い

- CLAUDE.md: プロジェクト全体の恒久ルール
- Rules: 技術領域ごとの制約
- Skills: 作業手順
- OpenSpec: 今回何を変更するかという仕様

## 導入判断

優先度: 高。まず1機能でPoCし、既存の要件定義との二重管理にならないか確認する。
