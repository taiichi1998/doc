# 汎用Webアプリ開発テンプレート：Azure構成・環境設計

更新日: 2026-10-31  
状態: 基本方針確定（案件別設定・実装時検証は別途）

## 1. 目的と関連設計

特定の業務アプリに依存しない、安全で保守しやすいAzure構成を定める。すべてのSKU・保持日数・ネットワーク設定を固定せず、共通方針と案件別パラメーターを分ける。

- [認証・認可・BFF設計](azure-webapp-bff-auth-security-and-authorization-cache.md)
- [REST API共通仕様](rest-api-common-standards.md)
- [Tenant・User・Resourceスコープ管理](tenant-user-resource-scope-management.md)

認証・API・スコープの既存方針を再定義しない。本書はリソース配置、環境、運用基盤の基本方針を補完する。既存認証設計のPoC項目は、本書の作成によって検証済みにはならない。

## 2. 標準構成とオプション

| 区分 | リソース・機能 | 方針 |
|---|---|---|
| 標準 | App Service① | React SPA静的配信＋Node.js/TypeScript BFF。Easy AuthでEntra認証 |
| 標準 | App Service② | Node.js/TypeScript業務API。Easy Authを前提とせず、API自身でBearer JWT検証と毎回の認可 |
| 標準 | Azure SQL Database | 業務データの保存。アプリ別・環境別にDBを分離する基本構成 |
| 標準 | Managed Identity（MI） | 対応サービスへの接続に優先利用。接続先にも必要最小限の権限を設定 |
| 標準 | Application Insights＋Log Analytics Workspace | エラー・性能・依存先の監視、ログ保管・分析。既存Workspaceの利用も可能 |
| 任意 | UAT・stageスロット | 必要な案件だけ設ける。用途別の配置は次節で統一 |
| 任意 | Key Vault | 認証用シークレット・外部APIキー等、秘密情報が必要な場合に利用 |
| 任意 | Storage | ファイル保存等の要件がある場合のみ利用 |
| 任意 | Azure Managed Redis | 認可キャッシュ等に必要な場合のみ利用。認可判定自体は必須 |
| 任意 | Application Gateway＋WAF等 | 社内規程、共通入口、防御要件に応じて追加 |
| 任意 | Functions・キュー・通知等 | 非同期処理等の要件がある場合のみ追加 |

本番はBFF・APIともPrivateを標準とし、追加のGateway/WAFを必須にしない。開発・UATのPublic／Privateは固定せず、社内規程、扱うデータ、構築申請の負担に応じて選択する。公開方式にかかわらず認証・認可・アクセス制御は必要。

## 3. 環境・Plan・スロット

| 用途 | 必須／任意 | アプリの配置 | データ |
|---|---|---|---|
| 開発（dev） | 必須 | 部門共有の開発Planに、当該アプリ専用のBFF・API App Serviceを配置 | dev DB |
| ユーザー受入（uat） | 任意 | 開発側のBFF・APIそれぞれのuatスロット | uat専用DB |
| リリース直前確認（stage） | 任意 | 本番側のBFF・APIそれぞれのstageスロット | 接続先はリリース手順に応じて設定 |
| 本番（prod） | 必須 | 当該アプリ用の本番PlanにBFF・APIを配置 | prod DB |

- 開発Planと本番Planは分離する。
- 本番のBFF・APIは同じPlanで開始し、独立した増強・障害影響の分離が必要ならPlanも分ける。
- 共有PlanではCPU・メモリを共有する。App Serviceを分けるだけでは計算資源の独立を意味しない。
- スロット利用時はStandard以上など、必要機能に対応したPlanを使用する。
- UATは業務・受入検証、stageは短時間のリリース直前確認・本番スワップ用。どちらも標準必須にはしない。
- UAT合格後は同じビルド成果物を本番stageへ配置し、確認・承認後にスワップする。スロットを使わない案件は承認付きの通常デプロイ手順を定める。
- スワップでDBデータは入れ替わらない。DB接続等の環境固有設定はスロット設定として管理する。
- スワップ準備中は本番設定がソース側に適用され得る。初期化処理やバックグラウンド処理による意図しない本番更新・通知を防ぐ。
- Easy Auth有効時はSwap with Previewを利用できない。通常スワップとは区別して検証する。
- 開発・UATのDBは同じ開発用SQL論理サーバー配下に配置可能。本番のSQL論理サーバーは開発側と分離する。

Azureリソース名の例は `project-dev-sql-db`、`project-uat-sql-db`、`project-prod-sql-db`。stage用DBを作る場合は `project-stage-sql-db`。命名は各Azureサービスの制約に合わせる。DB内部は既存方針どおりテーブル名・カラム名をlower_snake_case、テーブル名を単数形とする。

UATデータは業務に即したダミー・加工済みデータを基本とする。部門名などのマスターは機密性と社内ルールに応じて実データを利用可能。未加工の実データの利用条件は案件ごとに判断し、本番DBの直接共用を前提にしない。

## 4. ネットワークと担当範囲

| 対象 | 方針 |
|---|---|
| 本番の入口・API | Private Endpointを標準とし、Publicアクセスを無効化。社内・VPN等からの経路と名前解決を整備 |
| 開発・UAT | Private／Publicを選択可能。Public時はEntra認証＋会社IP制限等を設定し、社内規程に従う |
| Privateリソースへの送信 | 必要なApp ServiceのVNet統合と到達性を確認。Private Endpointは入口、VNet統合は出口 |
| ネットワーク関連 | VNet、サブネット、Private Endpoint、DNS、社内接続・通信許可等はインフラチームの申請・担当範囲に従う |
| 既存基盤 | 部門共有Planや申請済みネットワークは、ID等を受け取って参照。テンプレートから無断変更しない |

Private EndpointはApp Service・スロットごとに設定が必要。共有Planがあるだけでは追加申請は不要にならない。SQL DatabaseのPrivate Endpointは論理サーバー単位、Storageはアカウント内のBlob・Files等のサービス単位であり、再利用可能な範囲が異なる。

本番Private化と、すべての送信通信を閉域化することは同義ではない。DB・Storageに加え、Entra、監視、CI/CDの到達性もインフラ側と確認する。監視の閉域化が必要な場合はAzure Monitor Private Link Scope等を別途調整する。

## 5. 接続認証・基本設定

| 分類 | 標準方針 |
|---|---|
| 通信 | HTTPSのみ、TLS 1.2以上 |
| ユーザー認証 | Entra ID。BFFのEasy AuthとAPIのJWT検証・認可は関連設計に従う |
| リソース接続 | MI優先。Azure SQLへのMI接続ではDBパスワード保管用Key Vaultは不要 |
| 秘密情報 | 必要時のみKey Vaultに保存し、MIで取得。Easy Auth等が要求する認証用シークレットも対象 |
| 通常設定 | サーバー名・DB名などはApp Service設定へ。秘密情報と区別 |
| 最小権限 | MIごと・環境ごとに接続先の権限を限定。MIを有効化するだけではアクセスを許可しない |
| App Service | Always On・Health Checkを有効化し、認証構成と両立するヘルスチェック経路を用意 |
| 不要機能 | 不要なFTP・基本認証・リモートデバッグを無効化 |
| BFF防御 | CSRF・Cookie保護・転送先制限。トークン、Cookie、シークレットをログへ出力しない |

Easy Authのトークン取得・更新と `/.auth/me` 等からのブラウザへの露出を検証する。Easy Authを有効化しただけで「トークンがブラウザから隔離される」とは扱わない。

## 6. 監視・監査

- Application Insightsでエラー・応答時間・依存先を監視し、本番は基本アラートまで標準とする。
- Log Analytics Workspaceをログの保存・分析基盤として明示する。新規作成／既存利用は案件ごとに選択し、環境別の識別と閲覧権限を設ける。
- 業務の重要操作・権限変更・認可拒否は共通監査イベントとして記録する。イベント項目は既存スコープ設計を参照。
- 業務監査、Azure SQL監査、Azureリソースの操作・診断ログは別の記録であり、必要な収集・送信設定を行う。Workspaceの作成だけで自動収集されるとは扱わない。
- 重要な監査ログはサンプリングで欠落させない。収集上限や送信失敗による欠落、アクセス制御・改ざん防止を考慮する。
- 通常の診断ログは収集量を調整して費用を管理する。監査の完全性が必要な場合は、別の保存経路・保管先も検討する。
- 保持日数、アラート閾値、通知先は社内規程・システム要件に応じて設定する。監査ログの保持期間をバックアップの保持期間と同一と決め付けない。

## 7. バックアップ・復旧

基本方針は「DBの自動バックアップ＋復元確認、アプリはコード・成果物・IaCから再構築」とする。

取得頻度・保持期間・冗長性・保管地域・消去方法は、社内規程とシステム要件に従って設定し、汎用テンプレートでは具体値を固定しない。会話で確認したWeb区分の月次・1年保持は適用確認用の参考条件であり、全データ共通の強制値にはしない。

- Azure SQLでは直近の復旧用PITRと、必要時の長期保管LTRを使い分ける。Storage利用時は保存データに応じた保護を追加。
- 3-2-1、同時被災を避ける保管先、ランサムウェア対策、アクセス制御、区分別消去の社内条件を案件で適用する。
- 地理冗長性だけで3-2-1への適合やランサムウェア耐性を満たしたと断定しない。保持と削除不能設定の整合も確認する。
- アプリの別リージョン常設・フェイルオーバー構成は任意。バックアップの遠隔地保管要件とは分けて判断する。
- 本番稼働前および必要な変更時に復元可能性を確認する。復旧目標・定期確認頻度は案件ごとに定める。

## 8. SKU・IaC・実装時確認

### 案件別パラメーター

SKU・台数・保持期間は固定しない。以下は小規模な社内アプリの検討開始値であり、採用決定・性能保証・Microsoftの一律推奨値ではない。

| 対象 | 検討開始値・選択基準 |
|---|---|
| 開発Plan | 既存部門共有Plan。容量とスロット対応を確認 |
| 本番Plan | Premium v3／P1v3等から負荷・費用を検証。可用性重視は複数台、停止許容用途は1台も選択可能 |
| Azure SQL Database | 費用重視の小規模用途は開発S0・本番S1等から検証。負荷に応じて上位SKU・別購入モデルを選択 |
| Storage | StorageV2。開発LRS・本番ZRSを候補とし、バックアップの遠隔保管要件とは別に判断 |
| Key Vault | 秘密情報が必要な場合のみStandard等を選択 |

ゾーン冗長性等はSKU・リージョン・台数の対応を確認して採用する。標準構成が特定のSLAを保証するとは扱わない。

### IaCの範囲

Bicepで自チームが作成・変更できるリソースと設定を管理する。ネットワーク等、申請が必要な部分はインフラチームが構築・設定する。自チームに権限のない変更をテンプレートから実施しない。

既存リソースID、環境、公開方式、SKU・台数、保持設定、UAT・stage・Storage・Key Vault等の有効／無効をパラメーター化する。部門共通のPrivate基盤やFunctions等を一律作成する構成にはしない。

### 実装時の確認

設計書を追加して細分化せず、実装チェックリストとして次を確認する。

- API向けトークンの取得・更新、ブラウザ露出、JWT検証・認可。
- Private DNS、接続経路、デプロイ用SCMへの到達性、MIの権限。
- スロット別の認証・接続設定とスワップ挙動。BFF/APIのリリース順序とDB変更の前後互換性。
- ヘルスチェック、監視・監査収集、機密情報の除外。
- バックアップ設定と復元手順。

## 参考資料

- [App Serviceのアーキテクチャ推奨事項](https://learn.microsoft.com/en-us/azure/well-architected/service-guides/app-service-web-apps)
- [App Serviceのデプロイ推奨事項](https://learn.microsoft.com/en-us/azure/app-service/deploy-best-practices)
- [デプロイスロット](https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots)
- [App Service Private Endpoint](https://learn.microsoft.com/en-us/azure/app-service/overview-private-endpoint)
- [Managed Identity](https://learn.microsoft.com/en-us/azure/app-service/overview-managed-identity)
- [Application Insightsのリソース構成](https://learn.microsoft.com/en-us/azure/azure-monitor/app/create-workspace-resource)
- [Azure Monitorの収集上限](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/daily-cap)
- [Azure SQLの自動バックアップ](https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview)
