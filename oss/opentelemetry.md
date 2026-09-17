# OpenTelemetry

## 概要

OpenTelemetry (OTel) は、アプリケーションから Metrics / Logs / Traces を収集・送信するためのベンダーニュートラルな Observability 標準・OSS。

Grafana のような可視化ツールではなく、「テレメトリを生成・収集・転送する層」。

## 何ができるか

- API の分散トレーシング
- HTTP リクエストの処理時間計測
- DB 呼び出しの追跡
- エラー情報の関連付け
- Metrics / Logs / Traces の共通化
- OpenTelemetry Collector による収集・加工・転送

## このプロジェクトでのメリット

複数の React / Node.js アプリ・API が増えても Observability の実装方式を共通化できる。

```text
Portal
  |
Approval API
  |
Application API
  |
Azure SQL
```

例えば 1 リクエストに Trace ID を付け、どの API / DB 処理で遅延・エラーが発生したか追跡しやすくする。

## Azure との関係

```text
Node.js API
   |
OpenTelemetry SDK
   |
OTel Collector（必要に応じて）
   |
Application Insights / Azure Monitor
または
Grafana 系 Backend
```

OTel を計装標準にすると、可視化・保存先への依存を減らしやすい。

## Grafana との違い

| OpenTelemetry | Grafana |
|---|---|
| データを生成・収集・転送 | データを可視化・探索 |
| SDK / Collector | Dashboard / Explore / Alert |
| Observability の入口 | Observability の表示・分析側 |

競合ではなく組み合わせて利用できる。

## 導入方針

Node.js/TypeScript API の共通テンプレートに OTel の計装方針を持たせる。サービス名、Environment、Trace ID、Correlation ID 等の属性命名も標準化する。

## 導入判断

推奨度: ★★★★★

アプリが複数に増えるプラットフォームでは、後付けより早い段階で計装標準を決める価値が高い。