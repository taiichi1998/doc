# Claude Code 開発テンプレート構成

```text
repo/
├── .claude/
│   ├── CLAUDE.md                  # 常時読む最小ルール（200行以下推奨）
│   │
│   ├── rules/                     # 常時読まないルール集
│   │   ├── architecture.md
│   │   ├── coding-standards.md
│   │   ├── frontend.md
│   │   ├── backend.md
│   │   ├── database.md
│   │   ├── testing.md
│   │   ├── security.md
│   │   └── git-workflow.md
│   │
│   ├── skills/                    # 必要時のみロード
│   │   ├── create-feature/
│   │   │   └── SKILL.md
│   │   ├── implement-api/
│   │   │   └── SKILL.md
│   │   ├── implement-ui/
│   │   │   └── SKILL.md
│   │   ├── create-db/
│   │   │   └── SKILL.md
│   │   ├── review-pr/
│   │   │   └── SKILL.md
│   │   ├── fix-bug/
│   │   │   └── SKILL.md
│   │   ├── write-test/
│   │   │   └── SKILL.md
│   │   └── refactor/
│   │       └── SKILL.md
│   │
│   └── hooks/                     # 品質ゲート（任意）
│
├── docs/
│   ├── requirements.md
│   ├── architecture.md
│   ├── api-spec.md
│   ├── database-design.md
│   ├── adr/
│   └── tasks/
│
├── apps/
│   ├── web/
│   └── api/
│
├── packages/
│   ├── contracts/
│   ├── shared/
│   └── ui/
│
├── database/
│   ├── schema/
│   ├── migrations/
│   └── seed/
│
├── infra/
│   ├── bicep/
│   └── pipelines/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
└── README.md
```
