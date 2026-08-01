# Claude Code 実行基盤（簡易アーキテクチャ）

## 全体構成

```text
┌─────────────────┐
│ Web画面 / スマホ │
└────────┬────────┘
         │
         │ タスク実行
         ▼
┌─────────────────┐
│ 実行管理API      │
│ (ASP.NET等)      │
└────────┬────────┘
         │
         │ Job作成
         ▼
┌─────────────────┐
│ Job Queue / DB  │
│ 状態管理         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Worker           │
│ (Azure VM常駐)   │
└────────┬────────┘
         │
         ├─ Git Worktree作成
         ├─ Claude Code実行
         ├─ Build / Test
         ├─ Git Commit / Push
         └─ Job状態更新
         │
         ▼
┌─────────────────┐
│ Azure DevOps    │
│ Repo / Boards   │
└─────────────────┘
```

---

## Job状態

```text
Pending
    │
    ▼
Running
    │
    ├── 確認事項あり
    ▼
WaitingForAnswer
    │
    │ 回答
    ▼
Running
    │
    ├── エラー
    ▼
Failed
    │
    └── 完了
    ▼
Completed
```

---

## API

| Method | Endpoint | 内容 |
|---------|----------|------|
| POST | `/jobs` | ジョブ作成・実行 |
| GET | `/jobs/{id}` | 状態取得 |
| POST | `/jobs/{id}/resume` | 回答後に再開 |
| POST | `/jobs/{id}/cancel` | キャンセル |

---

## Worker処理

```text
Job取得
    ↓
Git Worktree作成
    ↓
Claude Code実行
    ↓
Build
    ↓
Test
    ↓
Git Commit
    ↓
Git Push
    ↓
Job状態更新
```

---

## Claude Codeへの指示例

```text
docs/tasks/US-123.md を読み込み、

- 完了済み([x])のタスクはスキップ
- 未完了([ ])のみ実装
- 実装完了後は [x] に変更
- 要件が曖昧な場合は推測せず、
  ## Questions に質問を追加して終了
- Build/Testを実行
- CommitしてPush
```

---

## 特徴

- Pipeline不要
- Azure VM上で常駐実行
- ジョブ単位で状態管理
- スマホ・Webから実行可能
- Claude Code CLIをそのまま利用
- 確認事項は停止して回答待ち
- 回答後は途中から再開
- Worktreeにより複数ジョブを並列実行可能
