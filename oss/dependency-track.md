# OWASP Dependency-Track

## 用途

SBOMを取り込み、複数アプリケーションのOSS依存関係と既知脆弱性を継続的に集中管理するプラットフォーム。

## メリット

- 複数アプリの依存リスクを一箇所で把握できる
- SBOMベースでコンポーネントを追跡できる
- 新しい脆弱性が公開された後も既存成果物への影響を追いやすい
- 10アプリ規模でセキュリティ状況を横断管理しやすい

## Trivyとの棲み分け

- Trivy: CI時点でコード、依存、コンテナ、IaC等をスキャン
- Dependency-Track: SBOMを蓄積し、アプリ横断で継続監視・可視化

まずTrivyをCIゲートとして維持し、集中管理が必要になったらDependency-Trackを追加する。

## 構成イメージ

```text
Application CI
  ↓
SBOM生成
  ↓
Dependency-Track
  ↓
複数アプリのDependency / Vulnerability管理
```

## 導入判断

優先度: 中。アプリ数とSBOM運用が増えた段階で価値が高くなる。VM/コンテナ等の運用基盤も必要になるため、現時点で必須ではない。
