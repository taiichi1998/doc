# Bruno

## 用途

APIリクエスト、認証、期待結果をファイルとして管理できるAPIクライアント / APIテストツール。

## メリット

- APIテスト資産をGit管理できる
- GUIで開発者が手動確認できる
- CLIからCIで実行できる
- APIごとの正常系・異常系を共有できる

## このプロジェクトでの使い方

```text
bruno/
├── environments/
├── auth/
├── approvals/
│   ├── create
│   ├── approve
│   ├── reject
│   └── get
└── users/
```

ローカルではAPIデバッグ、CIでは主要APIの回帰テストとして利用する。

## 既存テストとの棲み分け

- Unit: 関数・クラス
- Integration: DBや内部コンポーネント連携
- Bruno: HTTP API契約・エンドポイント
- Playwright: ブラウザからのE2E
- ZAP: Web/APIの動的セキュリティ検査

## 導入判断

優先度: 高。APIが増えるほど価値が上がる。HTTPieやHoppscotchを追加する前にBrunoへ寄せる。
