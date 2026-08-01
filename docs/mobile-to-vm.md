スマホからClaude Codeを実行する構成

1. 目的

社用スマホまたはPCのブラウザから、Azure VM上のClaude Codeへタスクを投入する。

Claude Chatは使用せず、Azure AI Foundry経由でClaude Codeを非対話実行する。

主な用途は以下とする。

* 対象リポジトリを指定
* ブランチを指定
* 対象ディレクトリを指定
* 使用モデルを指定
* 実装・修正・レビューなどのタスクを入力
* 実行結果をブラウザから確認

⸻

2. 推奨アーキテクチャ

社用スマホ / PC
        │
        │ HTTPS
        ▼
Azure App Service
  ├─ Web UI
  ├─ 実行管理API
  ├─ Entra ID認証
  └─ IPアクセス制限
        │
        │ Azure Resource Manager API
        ▼
Azure VM Run Command
        │
        ▼
既存Azure VM
  ├─ Claude Code
  ├─ 対象リポジトリ
  ├─ Azure AI Foundry接続設定
  ├─ Azure DevOps認証
  └─ MCP設定

App ServiceからVMのIPアドレスへ直接接続するのではなく、Azure Resource Manager APIを介してVM Run Commandを実行する。

そのため、PoC段階では以下を不要とする。

* SSH接続
* VMのPublic IP
* App ServiceのVNet統合
* VMへの受信ポート追加
* 独自のVM常駐API
* Azure Pipelines Self-hosted Agent

⸻

3. 実行方式

Claude Codeは対話画面として開かず、非対話モードで1タスクずつ実行する。

タスク入力
    ↓
Claude Codeを非対話実行
    ↓
処理完了
    ↓
実行結果を保存
    ↓
App Serviceから結果を確認

実行例：

claude -p "$(cat task.txt)" \
  --model "$MODEL" \
  --allowedTools "Read,Edit,Write,Bash"

要件が不明確な場合は、推測して実装せず、確認事項をファイルに出力して終了させる。

不明点がある場合は実装を開始せず、
questions.mdへ確認事項を出力して終了してください。

⸻

4. App Service画面

最初は1画面構成とする。

────────────────────────────────
Repository
[ procurement-api            ▼ ]
Branch
[ feature/vendor-registration  ]
Directory
[ src/features/vendors         ]
Model
[ Claude Sonnet              ▼ ]
Task
┌────────────────────────────────┐
│ ベンダー登録機能を実装してください。 │
│ 入力検証とユニットテストも追加する。 │
└────────────────────────────────┘
Timeout
[ 60 minutes                 ▼ ]
[ 実行 ]
────────────────────────────────
Status
Running / Completed / Failed
Result
- 実行結果
- 変更ファイル
- エラー内容
- 確認事項
- Commit ID
- PR URL
────────────────────────────────

必須入力項目

項目	内容
Repository	対象リポジトリ
Branch	対象ブランチ
Directory	作業対象の相対ディレクトリ
Model	Claude Codeで使用するモデル
Task	Claude Codeへの指示
Timeout	最大実行時間

将来追加する項目

* Auto Commit
* Create PR
* Dry Run
* 実行履歴
* 再実行
* タスクテンプレート
* リアルタイムログ

⸻

5. ディレクトリ指定

ディレクトリは指定可能とする。

ただし、絶対パスを自由入力させず、リポジトリ配下の相対パスだけを受け付ける。

入力例：

Repository: procurement-api
Directory: src/features/vendors

VM上では以下のように解決する。

/repos/procurement-api/src/features/vendors

実行例：

cd /repos/procurement-api/src/features/vendors
claude -p "$(cat /work/jobs/$JOB_ID/task.txt)"

以下のような指定は拒否する。

../../
/etc
/home
C:\Windows

App Service側で正規化し、/repos配下から外れるパスは実行しない。

⸻

6. モデル指定

Claude Codeの--modelオプションを使用し、実行ごとにモデルを指定する。

claude -p "$(cat task.txt)" \
  --model "foundry-claude-sonnet"

画面上では表示名を使用し、内部でAzure AI Foundryのデプロイ名へ変換する。

{
  "Claude Sonnet": "foundry-claude-sonnet",
  "Claude Opus": "foundry-claude-opus",
  "Claude Haiku": "foundry-claude-haiku"
}

モデル名の自由入力は許可せず、許可済みモデルをプルダウン表示する。

⸻

7. VM内のディレクトリ構成

/repos/
├─ procurement-api/
├─ vendor-portal/
└─ approval-service/
/work/jobs/
└─ {jobId}/
   ├─ task.txt
   ├─ request.json
   ├─ status.json
   ├─ output.log
   ├─ result.json
   └─ questions.md

status.json例

{
  "jobId": "20260801-001",
  "status": "completed",
  "startedAt": "2026-08-01T10:00:00+09:00",
  "completedAt": "2026-08-01T10:22:00+09:00",
  "exitCode": 0
}

result.json例

{
  "summary": "ベンダー登録機能を実装しました。",
  "changedFiles": [
    "src/features/vendors/vendor.service.ts",
    "src/features/vendors/vendor.service.test.ts"
  ],
  "commitId": null,
  "pullRequestUrl": null,
  "hasQuestions": false
}

⸻

8. 認証・セキュリティ

App Service

* Entra ID認証
* 社内テナントのみ許可
* 対象Entra IDグループのみ許可
* MFA
* IPアクセス制限
* HTTPSのみ
* Managed Identityを使用
* タスク内容と実行者を監査ログへ記録

App ServiceからAzure VM

App ServiceのManaged Identityを使用してAzure Resource Manager APIを呼び出す。

必要な権限はVM全体の共同作成者ではなく、Run Command実行に必要な最小権限へ限定する。

例：

Microsoft.Compute/virtualMachines/read
Microsoft.Compute/virtualMachines/runCommand/write
Microsoft.Compute/virtualMachines/runCommands/read
Microsoft.Compute/virtualMachines/runCommands/write

VM

* Public IPは不要
* SSHの外部公開は不要
* Claude Code実行専用ユーザーを作成
* /reposと/work/jobsのみ書き込み可能にする
* Azure AI Foundry認証情報を安全に管理
* Git・Azure DevOps・MCPの権限を最小化する

⸻

9. 実行ユーザー

Run Commandの実行ユーザーと、普段Claude Codeを利用しているユーザーが異なる可能性がある。

そのため、Claude Code専用ユーザーを作成する。

claude-runner

Linuxの場合の実行例：

sudo -u claude-runner bash -lc '
  cd /repos/procurement-api/src/features/vendors
  claude -p "$(cat /work/jobs/20260801-001/task.txt)" \
    --model "foundry-claude-sonnet"
'

専用ユーザーに以下を設定する。

* Claude Code
* Azure AI Foundry接続設定
* Git認証
* Azure DevOps認証
* MCP設定
* Claude Code設定ファイル
* 必要な環境変数

⸻

10. 実行フロー

1. ユーザーがApp Serviceへログイン
2. リポジトリ、ブランチ、ディレクトリを指定
3. モデルとタスクを指定
4. App Serviceが入力値を検証
5. ジョブIDを発行
6. App ServiceがRun Commandを実行
7. VM上でClaude Codeを非対話実行
8. 実行結果をVM上のジョブフォルダーへ保存
9. App Serviceが状態を取得
10. スマホへ結果を表示

⸻

11. ブロッカー候補

VM Run Commandが正常に動かない

whoami程度のコマンドが数分以上終了しない場合、以下の可能性がある。

* Azure VM Agentの不具合
* Azure基盤へのアウトバウンド通信制限
* Proxy設定
* FirewallまたはNSG
* 以前のRun Commandが停止していない

Run Commandが安定しない場合は、今回の構成における最大のブロッカーとなる。

長時間実行

Run Commandは、長時間ジョブやリアルタイム対話には向いていない。

初期運用では以下の制限を設ける。

* 同時実行数：1
* 最大実行時間：60分
* 大規模タスクは分割
* タイムアウト時は失敗として終了
* 途中結果をファイルへ保存

出力量

Run Commandのレスポンスだけに依存せず、結果はVM上のファイルまたはBlob Storageへ保存する。

Claude Codeの権限確認

非対話実行中に権限確認で停止しないよう、使用可能なツールを事前設定する。

ただし、Bashの無制限許可は避ける。

Git競合

同じリポジトリ・ブランチを複数ジョブで同時操作しない。

初期は1タスクずつ実行する。

⸻

12. Dockerの採用方針

初期段階

Dockerは使用しない。

理由：

* 既存VMとClaude Code環境をそのまま利用できる
* Foundry、Git、MCP、Proxy設定を再構築しなくてよい
* コンテナイメージの管理が不要
* PoCの実装範囲を小さくできる

Dockerを採用する条件

以下が必要になった段階で採用する。

* 複数ユーザーで利用する
* タスクを並列実行する
* タスクごとに環境を分離する
* Node.jsやClaude Codeのバージョンを固定する
* VM環境への影響を防止する
* 再現可能な実行環境が必要になる

将来構成：

Azure VM
└─ Docker
   ├─ Job Container 1
   │  ├─ Claude Code
   │  └─ Repository
   └─ Job Container 2
      ├─ Claude Code
      └─ Repository

⸻

13. 将来拡張

Run Command方式で不足が出た場合は、VM上に軽量な実行APIを配置する。

スマホ
→ App Service
→ VM上の実行API
→ Claude Code

この方式にすると、以下が実現しやすくなる。

* リアルタイムログ
* 長時間ジョブ
* キャンセル
* セッション継続
* Claude Codeからの確認質問
* 並列実行
* WebSocket通信
* Dockerによるジョブ分離

ただし、App ServiceからVMへのネットワーク接続が必要になるため、VNet統合やNSG変更などの申請が必要になる可能性がある。

⸻

14. 初期実装範囲

最初は以下だけを実装する。

スマホ
→ App Service
→ Azure VM Run Command
→ 既存VM
→ Claude Code
→ Azure AI Foundry

初期機能

* Entra ID認証
* IPアクセス制限
* リポジトリ選択
* ブランチ指定
* 相対ディレクトリ指定
* モデル選択
* タスク入力
* 最大60分の非対話実行
* 実行状態表示
* 実行結果表示
* 確認事項の表示

初期対象外

* チャット形式
* リアルタイムターミナル
* 複数ジョブの並列実行
* Docker
* 自動PR作成
* 自動マージ
* 複雑なジョブキュー
* VM上の独自常駐API

⸻

15. 最終判断

PoCとしては、以下の構成で進める。

社用スマホ / PC
        ↓
App Service
        ↓
Azure Resource Manager API
        ↓
Azure VM Run Command
        ↓
既存VM上のClaude Code
        ↓
Azure AI Foundry / Azure DevOps / MCP

この構成は、既存VMを利用し、追加リソースとネットワーク変更を最小化できる。

ただし、実装開始前に以下を最優先で検証する。

1. VM Run Commandが数十秒以内に正常終了するか
2. Run CommandからClaude Code専用ユーザーで実行できるか
3. Azure AI Foundryへ接続できるか
4. Azure DevOpsリポジトリを操作できるか
5. MCPへ接続できるか
6. 実行結果をファイルへ保存できるか

Run Commandが安定しない場合は、SSHまたはVM上の軽量実行API方式へ設計変更する。
