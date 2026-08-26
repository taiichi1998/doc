# FinOps Implementation Tasks

## 目的

`01-overall-requirements.md`、`02-functional-requirements.md`、`03-screen-operation-requirements.md` で確定したFinOps要件を、Claude Codeのセッションを分割しながら段階的に実装する。

各Phaseは原則として **1つのClaude Codeセッション** で扱い、不要な過去セッションを引き継がず、要件・現在Phase・コードだけを読ませることでトークン消費を抑える。

## 参照元

- `01-overall-requirements.md` — 全体要件・前提
- `02-functional-requirements.md` — 機能要件
- `03-screen-operation-requirements.md` — 画面・運用要件

上記3ファイルを要件の正とする。Phaseドキュメントと矛盾する場合は、上記3ファイルを優先する。

## 実装ルール

- 初期対象外機能を先回りして実装しない
- 要件にないAzureサービスや追加課金リソースを無断で導入しない
- 既存のプロジェクト標準、IaC、CI/CD、命名・タグ規約を優先する
- Azure設定は可能な限りIaCで再現可能にする
- 閾値や許可値など変更可能性がある値はハードコードせず設定値として管理する
- 各Phase終了時に、そのPhaseドキュメントのチェックリストと引き継ぎ欄を更新する
- Phase完了後に本ファイルのステータスも更新する
- 次Phaseでは原則として、要件3ファイル、本ファイル、対象Phaseファイル、実装コードのみを読み込む

## Phase一覧

| Phase | 内容 | 状態 | 詳細 |
|---|---|---|---|
| 1 | 基盤・実装骨格 | 未着手 | [phase-01-foundation.md](./implementation/phase-01-foundation.md) |
| 2 | Cost Managementデータ取得・整形 | 未着手 | [phase-02-cost-data.md](./implementation/phase-02-cost-data.md) |
| 3 | Prices / Advisor / Activity Log等のAzureデータ取得 | 未着手 | [phase-03-azure-data.md](./implementation/phase-03-azure-data.md) |
| 4 | Power BI Semantic Model / DAX | 未着手 | [phase-04-powerbi-model.md](./implementation/phase-04-powerbi-model.md) |
| 5 | 基本4画面 | 未着手 | [phase-05-basic-pages.md](./implementation/phase-05-basic-pages.md) |
| 6 | 高度3画面 | 未着手 | [phase-06-advanced-pages.md](./implementation/phase-06-advanced-pages.md) |
| 7 | 統合テスト・リリース仕上げ | 未着手 | [phase-07-integration-test.md](./implementation/phase-07-integration-test.md) |

## 依存関係

```text
Phase 1  基盤
   ↓
Phase 2  Cost Managementデータ
   ↓
Phase 3  Azure補助データ
   ↓
Phase 4  Power BI Semantic Model
   ↓
Phase 5  基本4画面
   ↓
Phase 6  高度3画面
   ↓
Phase 7  統合テスト
```

原則として順番に実施する。前Phaseの完了条件を満たしていない状態で次Phaseへ進まない。

## Claude Codeセッション運用

新しいセッションでは、対象Phase以外の実装詳細を大量に読み込ませない。

開始指示の基本形：

```text
finops/01-overall-requirements.md
finops/02-functional-requirements.md
finops/03-screen-operation-requirements.md
finops/TASKS.md
finops/implementation/phase-XX-*.md
を確認してください。

今回はPhase XXのみ実装してください。
初期対象外・将来機能は実装しないでください。
既存コードと既存規約を確認し、必要最小限の変更で実装してください。
完了後、対象Phaseドキュメントのチェックリスト・実装結果・変更ファイル・次Phaseへの引き継ぎを更新してください。
```

## 全体完了条件

- [ ] Cost Managementデータが日次でStorageへ保存される
- [ ] 価格データが週次更新される
- [ ] Storage Lifecycleが3年保持方針どおりIaC管理される
- [ ] Azure Advisor等から削減候補を取得できる
- [ ] Activity Logを変更前後比較に利用できる
- [ ] Power BI Semantic Modelと主要DAXが完成している
- [ ] 7つの初期Power BIページが利用できる
- [ ] RG月額負担ルールが設定値を用いて正しく計算される
- [ ] タグ品質を確認できる
- [ ] 最終更新日時が表示される
- [ ] 更新失敗時に前回正常データを維持できる
- [ ] Cost Analysis等の元データとの突合テストが完了している
- [ ] 初期対象外機能が混入していない
