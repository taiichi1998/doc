# MkDocs Material

## 概要

MkDocs は Markdown から静的ドキュメントサイトを生成する OSS。Material for MkDocs は MkDocs 向けの高機能テーマ・ドキュメント基盤。

## このプロジェクトでの用途

現在 GitHub の `doc` リポジトリに蓄積している Markdown を、人間が読みやすい社内ドキュメントサイトとして公開する用途に向く。

```text
doc/
├── architecture/
├── apps/
├── security/
├── oss/
└── ...
       |
   MkDocs Material
       |
社内 Documentation Site
```

## メリット

- Markdown をそのまま利用
- Git でレビュー・履歴管理
- 全文検索
- ナビゲーション
- コードブロック
- Mermaid 等の技術ドキュメントと相性が良い
- CI/CD で自動ビルド・デプロイ可能
- AI が Markdown を生成・更新しやすい

## Claude Code との相性

Claude Code に Markdown を直接更新させ、そのままサイトへ反映できる。

```text
要件変更
  ↓
Claude Code が docs/*.md 更新
  ↓
PR Review
  ↓
Merge
  ↓
MkDocs Build
  ↓
社内 Docs 更新
```

AI が扱いやすい Markdown を Source of Truth に維持できる点が大きい。

## Docusaurus との違い

| 項目 | MkDocs Material | Docusaurus |
|---|---|---|
| 主言語 | Python 系 | Node.js / React 系 |
| Markdown Docs | ◎ | ◎ |
| セットアップ | 比較的シンプル | やや高機能 |
| カスタム React UI | △ | ◎ |
| 技術ドキュメント中心 | ◎ | ◎ |
| Web サイト拡張性 | ○ | ◎ |
| 運用の軽さ | ◎ | ○ |

## このプロジェクトでの選択

「Markdown を綺麗な社内技術ドキュメントとして見せたい」が中心なら MkDocs Material を優先。

独自 React コンポーネント、ブログ、製品サイト的な機能まで必要なら Docusaurus を検討する。

現状の `doc` リポジトリ用途では MkDocs Material の方がシンプルで相性が良い可能性が高い。

## 導入判断

推奨度: ★★★★★

既存 Markdown 資産をそのまま活かせるため導入コストが低い。ドキュメント量が増えた段階で特に価値が高い。