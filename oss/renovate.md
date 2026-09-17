# Renovate

## 用途

npm等の依存パッケージ、Dockerイメージ、各種バージョン更新を検出し、自動で更新PRを作成する。

## メリット

- 複数リポジトリの依存更新を自動化できる
- 更新を放置しにくくなる
- Minor/Patchをまとめるなど運用ルールを設定できる
- CIテストと組み合わせて安全に更新判断できる

## このプロジェクトでの使い方

```text
Renovate
  ↓
Dependency更新PR
  ↓
Lint / TypeCheck / Unit / Integration / Build
  ↓
Security Scan
  ↓
AI Review / Human Review
  ↓
Merge
```

10前後のアプリ/リポジトリへ拡大した場合に特に効果が高い。

## Dependabotとの棲み分け

既にDependabotを利用する場合は両方を同時運用しない。更新グループ、スケジュール、複数エコシステム管理など必要な機能を比較してどちらかへ統一する。

## 導入判断

優先度: 高。リポジトリ数が増える前に標準設定を作る価値がある。
