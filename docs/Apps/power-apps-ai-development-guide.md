# Power Apps AI駆動開発ガイド

## 1. 目的

Power Apps Canvas App を Claude Code を中心に実装し、人間の作業を要件・接続・最終確認に寄せる。

目標は AI 80〜90%、人間 10〜20% 程度の実装比率とし、Power Apps Studio の Code View と YAML を利用して短いフィードバックループを作る。

## 2. 基本方針

- Canvas App を対象とする
- UI・Power Fx・CRUD・Navigation・Validation は原則 Claude Code が実装する
- SharePoint 接続、認証、権限など環境依存部分は人間が初期設定する
- AI が YAML 書式を推測しないよう、動作確認済み Golden Template を正とする
- Microsoft 公式 Power Apps YAML schema を静的検証に利用する
- 開発中はソース全体の pack/unpack を繰り返すより、Power Apps Studio の Code View への YAML 貼り付けを優先する
- 実装は画面または機能単位で行い、巨大なコンテキストを毎回読み込ませない
- Playwright は主に Preview / 実行版アプリの UI・操作確認に利用する

## 3. 人間が事前に用意するもの

### 3.1 要件・画面設計

最低限、以下を Markdown 等で定義する。

- 画面一覧
- 画面ごとの表示項目
- 操作
- 画面遷移
- CRUD 要件
- バリデーション
- 権限による表示・操作差分
- エラー時の挙動

### 3.2 SharePoint List

人間側で実環境の List を作成し、列定義を確定する。

AI には CSV だけではなく、可能であれば JSON / YAML で以下を渡す。

- List 名
- Column の表示名
- Internal Name
- データ型
- Required
- Choice 値
- Lookup
- Person
- 複数選択
- Default 値
- 必要な制約

### 3.3 接続済み Golden App

最初に人間が空の Canvas App を作成し、必要な SharePoint データソースを接続する。

このアプリを以後の実装ベースとする。

人間が担当する範囲は原則以下とする。

1. Canvas App 作成
2. SharePoint 接続
3. 必要なデータソース登録
4. Connection / 権限確認
5. 初回動作確認

## 4. Claude Code 側に用意するもの

推奨構成例：

```text
power-apps/
├─ requirements/
│  └─ requirements.md
├─ schema/
│  ├─ sharepoint-lists.yaml
│  └─ data-contract.yaml
├─ golden/
│  └─ base.pa.yaml
├─ templates/
│  ├─ screen-list.pa.yaml
│  ├─ screen-detail.pa.yaml
│  ├─ screen-edit.pa.yaml
│  ├─ gallery.pa.yaml
│  ├─ form.pa.yaml
│  └─ components/
├─ design/
│  ├─ design-tokens.yaml
│  └─ navigation.yaml
└─ .claude/
   └─ skills/
      └─ power-apps/
         ├─ SKILL.md
         └─ references/
            ├─ yaml-rules.md
            ├─ modern-controls.md
            ├─ classic-controls.md
            ├─ layout-patterns.md
            └─ examples/
```

## 5. Skill の責務

SKILL.md には大量の YAML サンプルそのものではなく、Claude が Power Apps を実装する際の手順・判断基準を記載する。

例：

- Modern Controls を原則優先する
- Golden Template を最優先の実装例とする
- 存在しない Property を推測して生成しない
- 新しい Control を利用する場合は reference の実績 YAML を確認する
- Power Fx の命名・エラー処理規約に従う
- SharePoint の Internal Name を利用する
- Delegation Warning が発生し得る式をレビューする
- 画面単位または機能単位で実装する
- YAML 生成後に schema / 構文チェックを行う
- UI 実装後に Playwright で主要シナリオを確認する

## 6. Golden Template / Reference の責務

AI の記憶や推測に依存しないことを重視する。

Power Apps Studio 上で実際に動作した以下の Control を一度作成し、その YAML を正解サンプルとして保存する。

- Screen
- Container
- Header
- Button
- Text / Label
- Text Input
- Dropdown / Combo Box
- Gallery
- Form
- Date Picker
- Toggle / Checkbox
- Dialog / Modal
- Notification
- Navigation

Modern / Classic で構造が異なる場合は分離して管理する。

## 7. AI駆動の開発フロー

```text
人間
  ↓
要件・画面設計を確定
  ↓
SharePoint List作成
  ↓
空Canvas App作成＋SharePoint接続
  ↓
Claude Code
  ├─ 要件を読む
  ├─ SharePoint Schemaを読む
  ├─ Golden Template / Referenceを選択
  ├─ YAML生成
  ├─ Power Fx生成
  ├─ CRUD実装
  ├─ Navigation実装
  └─ Validation実装
  ↓
静的チェック
  ↓
Power Apps Studio Code ViewへYAML反映
  ↓
Preview / 実行版
  ↓
Playwright
  ├─ 画面遷移
  ├─ 入力
  ├─ CRUD
  ├─ 表示確認
  └─ Screenshot取得
  ↓
Claude Codeが問題を修正
  ↓
必要回数だけ反復
  ↓
人間が最終確認・公開
```

## 8. Power Apps Studio と Playwright の役割

### 開発中

Code View に YAML を貼り付け、数秒単位で画面を確認できるフィードバックループを優先する。

`Claude Code → YAML生成 → Code Viewへ貼付 → Preview → 問題をClaudeへ返す`

Studio 自体を Playwright で完全自動操作することは可能性としてはあるが、UI変更、認証、MFA、Power Apps 固有 DOM 等により壊れやすいため、必須要件にはしない。

### UI / E2E確認

Playwright は Preview または実行版 Canvas App の確認を中心に利用する。

- ボタン操作
- Navigation
- 入力
- 保存
- エラー表示
- 一覧・詳細表示
- Screenshot
- 主要ユーザーシナリオ

Claude Code が Screenshot と操作結果を確認し、必要に応じて YAML / Power Fx を修正する。

## 9. 手作業として残す範囲

完全自動化を目的にせず、以下は原則として人間が担当する。

| 項目 | 担当 |
|---|---|
| 要件・設計の最終決定 | 人間 |
| SharePoint List の本番設計確認 | 人間 |
| 初回 Canvas App 作成 | 人間 |
| SharePoint / Connector 接続 | 人間 |
| Connection Reference | 人間 |
| 認証・権限 | 人間 |
| Power Apps 共有設定 | 人間 |
| YAML / UI / Power Fx 実装 | AI |
| CRUD / Navigation / Validation | AI |
| 静的チェック | AI |
| UI / E2E テスト | AI + Playwright |
| 最終実機確認 | 人間 |
| 公開判断 | 人間 |

## 10. AI使用量の考え方

Claude Code をエージェント的に利用し、要件読込、YAML生成、修正、Playwright、Screenshot レビューまで繰り返す場合、累積コンテキストは大きくなる。

中規模 Canvas App では、実装の試行錯誤次第で累積数百万 token 規模になることを前提とする。

ただし API 従量課金ではなく Claude Code の seat / 定額利用枠を利用できる場合、単純な token 単価より利用上限と実行効率を重視する。

### 使用量を抑えるルール

- 巨大な要件書を毎ターン全文読み込みしない
- Skill から必要な Reference のみ読む
- 1画面 / 1機能単位で実装する
- Golden Template を利用して書式エラーによる再生成を減らす
- schema / 静的検証を UI 確認より先に行う
- Playwright Screenshot レビューは細かい変更ごとではなく、機能実装後にまとめて行う
- 同じファイルを不必要に何度もコンテキストへ投入しない

## 11. 最終的に目指す操作

開発基盤を一度整備した後、新しい Canvas App では人間の初期作業を以下まで減らす。

1. 要件・画面設計を用意する
2. SharePoint List を用意する
3. Canvas App を作成して SharePoint を接続する
4. Claude Code に要件と Schema を渡す
5. Claude が画面・Power Fx・CRUDを実装する
6. Code View に YAML を反映する
7. Claude + Playwright で確認・修正する
8. 人間が最終確認して公開する

最終目標は、アプリごとに YAML の書き方を人間が考えるのではなく、**Golden Template + Skill + Schema + Reference + Playwright** を共通開発基盤として再利用し、Canvas App の実装部分をほぼ AI に寄せることである。
