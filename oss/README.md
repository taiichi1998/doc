# OSS 開発ツールガイド

このフォルダーは、現在の開発環境（Claude Code / React / Node.js / TypeScript / Azure / Azure SQL / CI/CD）で有力なOSSを、導入判断と実運用の観点で整理する。

## 上位候補

| OSS | 主用途 | 推奨度 | 主な導入先 |
|---|---|---:|---|
| OpenSpec | AI仕様駆動開発 | ★★★★★ | 開発フロー |
| Bruno | APIテスト | ★★★★★ | ローカル / CI |
| jq | JSON処理 | ★★★★★ | CLI / Pipeline |
| Gitleaks | Secret検出 | ★★★★★ | Pre-commit / CI |
| Semgrep | SAST | ★★★★★ | CI |
| Renovate | 依存関係更新 | ★★★★★ | Repository |
| DBeaver Community | DB管理 | ★★★★★ | 開発端末 |
| Playwright | E2Eテスト | ★★★★★ | ローカル / CI |
| Excalidraw | ラフ設計・図解 | ★★★★☆ | 設計 |
| Docusaurus | Docsサイト | ★★★★☆ | Docs基盤 |
| mise | 開発環境統一 | ★★★★☆ | 開発端末 / Repository |
| Schemathesis | OpenAPIベースAPIテスト | ★★★★☆ | CI |
| OWASP Dependency-Track | SBOM/SCA集中管理 | ★★★★☆ | DevSecOps基盤 |

## 導入方針

すべてを一度に導入しない。既存ツールとの重複を確認し、工程を自動化できるものから採用する。

優先候補は OpenSpec、Bruno、Gitleaks、Renovate。jq は日常CLI用途、PlaywrightはE2E標準として利用する。Semgrep、Schemathesis、Dependency-Trackは既存の SonarQube / Trivy / ZAP と役割を整理してから追加する。

各ファイルに「用途」「メリット」「このプロジェクトでの使い方」「導入ポイント」「既存ツールとの棲み分け」を記載する。
