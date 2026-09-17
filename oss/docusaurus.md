# Docusaurus

## 用途
MarkdownベースのドキュメントWebサイトを構築する。

## メリット
- Git管理中のMarkdownを読みやすいサイト化できる
- サイドバー等で大量の文書を整理できる
- バージョン管理されたDocsを作りやすい
- AIからMarkdownを更新しやすい

## このプロジェクトでの使い方
既存docリポジトリが大きくなり、GitHub上で文書を探す負担が増えた段階で導入する。Architecture、Development Guide、Security、CI/CD、OSS、Power Platform、ADRなどを対象にする。

## GitHub Markdownとの棲み分け
文書数が少ない間はGitHub Markdownで十分。サイト化によるビルド・ホスティング運用を増やすため先行導入しすぎない。

## 導入判断
優先度: 中。docリポジトリの規模拡大後に導入する。
