# Engineering Knowledge Base

Azure、AIエージェント、DevSecOps、セキュリティ、業務自動化に関する設計書・学習ノート・ツール調査を管理するリポジトリです。

## 分野別ドキュメント

| カテゴリ | 内容 |
| --- | --- |
| [AI / Claude Code](docs/ai/README.md) | Claude Code、MCP、エージェント設計、AIレビュー |
| [クラウド / アーキテクチャ](docs/platform/README.md) | Azure、Webアプリ、インフラ・基盤設計 |
| [DevOps / CI・CD](docs/devops/README.md) | Azure DevOps、テスト、リリース、開発フロー |
| [セキュリティ / ガバナンス](docs/security/README.md) | Zero Trust、セキュリティ試験、AI統制 |
| [Power Platform / RPA](docs/power-platform/README.md) | Power Apps、RPA、業務効率化 |
| [FinOps](docs/finops/README.md) | Azureコスト、Power BI、要件・実装 |
| [OSS / ツール](docs/tools/README.md) | 開発支援、文書作成、監視、テストOSS |
| [学習 / 資格](docs/learning/README.md) | 資格ロードマップ、AZ-305学習ノート |

**[ドキュメント総合目次](docs/README.md)**

## よく使う資料
- [資格取得ロードマップ（優先順位付き）](docs/certification-roadmap.md)
- [Claude Code標準テンプレート](docs/ai/claude-code-standard-template.md)
- [AI/MCPセキュリティ・ガバナンス](docs/ai-mcp-platform-team-security-governance.md)
- [Azure Webアプリ標準テンプレート](docs/architecture/azure-webapp-standard-template.md)
- [Azure DevOpsワークフロー設計](azure-devops-workflow-design.md)
- [FinOpsプロジェクト](finops/README.md)

## 整理方針
1. **カテゴリ → テーマ → プロジェクト / 設計・実装・運用** の順に深い階層を許容する。
2. ナビゲーションはカテゴリ別READMEを入口とし、複数分野に関係する資料はリンクで相互参照する。
3. 既存ファイルの移動は参照箇所を確認して段階的に実施する。リンク切れ防止のため、現段階では元の資料を残す。
4. 新規文書は基本的に `docs/<category>/<topic>/...` に配置する。プロジェクト固有の連続した仕様・実装手順はプロジェクト単位で管理する。
