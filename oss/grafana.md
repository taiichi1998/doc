# Grafana

## 概要

Grafana は、メトリクス・ログ・トレースなどの運用テレメトリを可視化する OSS ダッシュボード基盤。

## このプロジェクトでの主用途

- アプリ/API の稼働状況
- レスポンスタイム、エラー率、スループット
- VM / コンテナ / OSS 基盤の監視
- OpenTelemetry で収集したテレメトリの可視化
- セキュリティ/テスト基盤の運用メトリクス

## Power BI / Azure Workbooks との違い

| 項目 | Grafana | Power BI | Azure Workbooks |
|---|---|---|---|
| 主目的 | Observability / 運用監視 | BI / 経営・業務分析 | Azure 運用・調査 |
| 得意データ | Metrics / Logs / Traces / 時系列 | コスト・売上・業務データ | Azure Monitor / Log Analytics 等 |
| リアルタイム監視 | ◎ | △ | ○〜◎ |
| 長期の業務分析 | △ | ◎ | △ |
| Azure ネイティブ | ○ | ○ | ◎ |
| OSS | ◎ | × | × |
| アラート | ◎ | 主用途ではない | Azure Monitor と連携 |
| 複数データソース統合 | ◎ | ◎ | Azure 系中心 |

## 使い分け

```text
経営・FinOps・ライセンス・予算分析
    -> Power BI

Azure リソースの障害調査・Log Analytics の可視化
    -> Azure Workbooks

アプリ/API/OSS の継続的な Observability
    -> Grafana
```

Power BI の代替として Grafana を採用するのではなく、目的を分離する。

## 推奨構成

```text
React / Node.js API / VM / OSS
          |
     OpenTelemetry
          |
 Metrics / Logs / Traces
          |
   Observability Backend
          |
       Grafana
```

Azure Monitor / Application Insights / Log Analytics を中心にする場合は Azure Workbooks で十分なケースも多い。Azure 外を含む複数データソースや OSS ベースの統一 Observability が必要になった時に Grafana の価値が高い。

## 導入判断

推奨度: ★★★★☆

現時点で Power BI を置き換えない。アプリケーション監視・OSS 基盤監視が本格化した段階で導入候補とする。