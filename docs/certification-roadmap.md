# 資格取得ロードマップ（2026年10月版）

更新日: 2026-10-29

## 目的と評価基準
- 目標: Azure基盤・DevSecOps・エンタープライズAIの実務実績を軸に、AIソリューションアーキテクト／プラットフォームアーキテクトとして市場価値を高める。
- 優先順位: 現業への適合度、転職時の説明力、学習費用対効果、受験可能性、既存資格との重複度を総合評価した**個人向け推奨順**。客観的な資格難易度ランキングではない。
- **資格より成果**: ProcureOne、AIレビュワー、AIエージェント評価・運用、CI/CD、ゼロトラスト等のアーキテクチャ判断と成果指標をポートフォリオ化する。

## 取得済み・履歴要確認（ランキング対象外）
- AZ-305（試験）／Azure Solutions Architect Expert（認定）: 試験合格と認定付与は区別。認定には別途条件があるためMicrosoft Learnで個人ステータス確認。
- AZ-400（試験）／DevOps Engineer Expert（認定）: 同上。**取得済みか、受験済み・学習済みかの確定はMicrosoft Learnで確認**。
- AI-102: 旧候補。Microsoftの現行資格体系で後継のAI-103との関係、廃止日を確認してから受験判断。

## 推奨取得順位（取得済みのものはスキップ）
| 順位 | 資格・試験 | 分野 | 推奨理由 / 注意点 |
| ---: | --- | --- | --- |
| 1 | **SC-100 / Cybersecurity Architect Expert** | セキュリティ設計 | Azure・Entra・Zero Trust・運用ガバナンスに直結。Expert認定の前提資格条件を確認。 |
| 2 | **Microsoft AB-100 / Agentic AI Business Solutions Architect Expert** | AIアーキテクト | Azure・Power Platform・Copilot Studio・Foundry・マルチエージェント。**Expert認定にはAB-100合格に加え指定Associate認定1件が必要**。 |
| 3 | **AI-103 / Azure AI Apps and Agents Developer Associate** | AI開発 | Azure AIエージェント・RAG・評価の実務習得。AB-100の前提認定候補にもなる。 |
| 4 | **Claude Architect – Professional（正式名称・提供状況要再確認）** | Claudeアーキテクチャ | Claude Code・MCP・安全設計の経験と高い親和性。ただし一般受験可否、名称、公式公開内容の確認が必須。受験制限がある場合は学習テーマとして継続。 |
| 5 | **HashiCorp Terraform Associate** | IaC | Bicep経験をマルチクラウドに拡張。 |
| 6 | **AWS Solutions Architect – Professional (SAP-C02)** | マルチクラウド設計 | Azure以外への設計力の証明。AWS実務・基礎学習を先行。 |
| 7 | **CKA (Certified Kubernetes Administrator)** | コンテナ基盤 | ACA/AKSや基盤運用を深掘り。 |
| 8 | **PMP (Project Management Professional)** | PM | PL/PO/PM兼務の経験を対外的に説明。受験に必要な実務年数・教育要件を確認。 |
| 9 | **CISSP** | セキュリティ統括 | セキュリティアーキテクト・管理職志向。正式認定の実務経験要件に注意。 |
| 10 | **CKS (Certified Kubernetes Security Specialist)** | Kubernetesセキュリティ | CKAを前提にランタイム・サプライチェーン防御を強化。 |
| 11 | **FinOps Certified Practitioner** | クラウドコスト | 既存Power BI FinOps可視化・配賦・予算管理と相性がよい。 |
| 12 | **GitHub Agentic AI Developer / GH-600（名称・提供状況要再確認）** | AI開発生産性 | GitHubとAIエージェントの開発プロセスに関係。正式な試験体系は確認してから登録。 |
| 13 | **Google Cloud Professional Agentic Architect（正式名称・提供状況要再確認）** | AIアーキテクチャ | Azure以外のAgentic AI設計に拡張。公式の現行認定一覧で確認。 |
| 14 | **AWS Generative AI Developer – Professional** | 生成AI開発 | Bedrockを含めたAWS側のAI実装を体系化。 |
| 15 | **NVIDIA Agentic AI Professional（正式名称・提供状況要再確認）** | AIエージェント | モデル・推論・エージェント技術を広げる場合に。 |
| 16 | **TOGAF Enterprise Architecture** | EA | 部門・企業横断の基準・標準化が主業務になった場合。 |
| 17 | **RHCE** | Linux/自動化 | VM、コンテナ、運用自動化の深掘り。現時点では優先度低め。 |
| 18 | **GitHub Copilot Certification（現行名称・体系要確認）** | 開発支援AI | 社内教育・AI開発標準化向け。Claude関連資格と学習重複あり。 |
| 19 | **Google Cloud Professional Cloud Architect** | マルチクラウド | 第二クラウドをGoogle Cloudに定める場合に順位を引き上げる。 |
| 20 | **AWS DevOps Engineer – Professional** | DevOps | AZ-400の成果をAWSで横展開する場合に。 |
| 21 | **AWS Advanced Networking – Specialty / Security – Specialty** | NW/セキュリティ | AWSの本格的な構築運用が担当になった時に検討。 |
| 22 | **AWS Machine Learning Engineer – Associate** | ML運用 | MLパイプラインを業務で実装する場合に。 |
| 23 | **AWS Solutions Architect – Associate** | AWS基礎 | AWS SAPに直接挑戦しない場合の足場。 |
| 24 | **AWS Developer – Associate / CloudOps Engineer – Associate** | AWS実装/運用 | AWSの開発・運用を担当する場合のみ。 |
| 25 | **AWS AI Practitioner** | AI基礎 | 現状の実務経験では取得優先度は低い。 |

## 受験順の推奨ルート
1. **短期**: SC-100の学習を継続し、現在のMicrosoft認定ステータスを確認する。
2. **AI集中ルート**: AI-103 → AB-100（資格取得条件を満たす）→ Claude Architect – Professional（提供・受験資格が確認できる場合）。
3. **基盤拡張ルート**: Terraform Associate → AWS SAP（必要ならSAAから）→ CKA → CKS。
4. **中長期**: PMP・CISSP・TOGAF、必要に応じてFinOps・マルチクラウド資格を追加。
5. **並走する実績**: Claude Codeの権限・安全対策、評価/回帰テスト、トレース、コスト、SLO、障害復旧、監査証跡をコードと設計文書で残す。

## 学習範囲メモ
- **Claude / Agentic AI**: MCP認証認可、プロンプトインジェクション対策、ツール実行の副作用・冪等性、エージェントの状態管理、評価データの代表性、重大事故率、シャドー・カナリア、トレーシング。
- **Azure AI / Microsoft**: Foundry、Copilot Studio、Power Platform、Entra ID、RBAC、ネットワーク分離、データ所在、監査、AIガバナンス。
- **プラットフォーム**: IaC、GitOps/CI/CD、コンテナ、SAST/SCA、シークレット保護、FinOps。

## 確認事項・公式リンク
- Microsoft AB-100: https://learn.microsoft.com/en-us/credentials/certifications/exams/ab-100/
- Expert認定と前提資格: https://learn.microsoft.com/en-us/credentials/certifications/agentic-ai-business-solutions-architect/
- Microsoft認定一覧: https://learn.microsoft.com/en-us/credentials/certifications/
- AWS認定一覧: https://aws.amazon.com/certification/
- Google Cloud認定一覧: https://cloud.google.com/learn/certification
- CNCF認定: https://www.cncf.io/training/certification/
- HashiCorp認定: https://www.hashicorp.com/certification

> 注: 新設AI資格は名称、試験公開、言語、受験資格、認定要件が変わり得る。未確認事項は確定事実として扱わない。ランキングは状況に応じて更新する。
