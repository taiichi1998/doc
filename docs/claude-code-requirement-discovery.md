# Claude Code を使用した実装前の要件決定プロセス

## 1. 目的

Claude Code を、実装を開始するためのツールとしてではなく、**実装前に重要な要件を洗い出し、整理し、意思決定を支援するための対話型 Requirement Discovery ツール**として使用する。

このフェーズではコード生成や実装計画の作成を目的としない。

目的は以下である。

- 要求を要件へ落とし込む
- 不明点・曖昧さ・矛盾を発見する
- 重要な意思決定事項を明確化する
- 選択肢とトレードオフを比較する
- 人間が最終決定する
- 決定内容を再利用可能なドキュメントとして残す
- 実装開始前に重大な未確定事項をなくす

---

## 2. 基本原則

Claude Code に要件を一方的に決定させない。

基本フローは以下とする。

```text
Claude Code
    ↓
既存資料・要求を整理
    ↓
不明点・矛盾・リスクを抽出
    ↓
重要度の高い事項から質問
    ↓
選択肢・メリット・デメリット・推奨案を提示
    ↓
人間が決定
    ↓
Claude Code が決定内容を記録
    ↓
次の重要事項へ
    ↓
重大な未確定事項がなくなるまで反復
    ↓
要件確定
    ↓
設計フェーズ
    ↓
実装計画
    ↓
実装
```

重要なのは、以下の責任分担である。

```text
Claude Code
- 調査
- 整理
- 質問
- 抜け漏れ検出
- 矛盾検出
- 選択肢提示
- トレードオフ整理
- 推奨
- 文書化

人間
- 業務判断
- リスク受容
- 優先順位決定
- 重要要件の最終決定
```

---

## 3. Requirement Discovery Mode

要件決定フェーズでは、Claude Code に明示的に Requirement Discovery の役割を与える。

このモードでは以下を禁止する。

- コード実装
- 詳細な実装方法への脱線
- 未確認の重要要件を推測で確定すること
- UIや内部実装など細部から議論を開始すること

### 推奨プロンプト

```text
このフェーズでは実装案やコードを作成しないでください。
目的は、実装前に要件を確定することです。

以下の順序で進めてください。

1. 現在分かっている要求・制約・前提を整理する
2. 不明点、曖昧な点、矛盾、リスクを抽出する
3. 要件に与える影響が大きい項目から質問する
4. 原則として1回の質問では1テーマを扱う
5. 選択肢がある場合は以下を提示する
   - 選択肢
   - メリット
   - デメリット
   - 推奨案
   - 推奨理由
   - 将来的な影響
6. 私の回答を決定事項として記録する
7. 新しい回答と既存要件に矛盾がないか確認する
8. 回答によって新たに発生した未確定事項を追加する
9. 重大な未確定事項がなくなるまで繰り返す
10. 実装方法・コード・詳細設計には進まない

最終的に以下を整理してください。

- Goals
- Users / Actors
- Functional Requirements
- Non-functional Requirements
- Business Rules
- Data Requirements
- Authorization Requirements
- Constraints
- Assumptions
- Out of Scope
- Acceptance Criteria
- Open Questions
- Decision Log
```

---

## 4. 質問する順序

Claude Code には細かい仕様から質問させず、影響範囲の大きい事項から確認させる。

### Level 1: 目的・スコープ

最初に確認する。

- なぜこの機能・システムが必要なのか
- 誰が利用するのか
- 何を解決するのか
- 成功条件は何か
- 今回含めないものは何か

### Level 2: 業務フロー・ビジネスルール

次に業務そのものを明確にする。

- 正常系の業務フロー
- 誰が何を行うか
- 状態遷移
- 承認・却下・差戻し
- 例外処理
- 業務ルール

### Level 3: データ・権限・責任境界

重大な設計判断につながるため、早めに確定する。

- 何を保存するか
- データの所有者
- 誰が閲覧できるか
- 誰が作成・変更・削除できるか
- データ保持期間
- 監査要件
- システム間の責任境界

### Level 4: 異常系・エッジケース

正常系だけで要件を確定しない。

- 処理途中で失敗した場合
- 二重操作
- 同時編集
- データ欠損
- 権限変更
- 担当者不在
- キャンセル
- 再実行
- 外部サービス停止

### Level 5: 非機能要件

- セキュリティ
- 可用性
- 性能
- スケーラビリティ
- 監査
- ログ
- バックアップ
- リカバリ
- 保守性
- コスト

### Level 6: UX・詳細仕様

上位要件がある程度固まってから確認する。

- 画面構成
- 入力方法
- 検索
- 並び順
- 表示項目
- エラー表示
- 操作性

### Level 7: 技術・実装制約

要件と技術設計を混同しないよう最後に整理する。

例：

- Azure を使用する
- Entra ID を使用する
- Azure SQL を使用する
- Private Network を使用する
- 社内標準ライブラリを使用する

技術選定そのものを要件として扱う場合は、なぜ変更できないのかも記録する。

---

## 5. 質問の形式

単純な自由回答だけではなく、意思決定しやすい形で質問する。

### 推奨形式

```text
Question
承認途中で申請内容を変更できるようにしますか？

A. 変更不可
B. 変更すると承認を最初からやり直す
C. 変更箇所によって承認継続または再承認を切り替える

Recommendation
B

Reason
監査性と実装複雑度のバランスが良いため。

Impact
A:
- 実装は単純
- 利用者の操作性が低下

B:
- 業務ルールが明確
- 監査しやすい
- 再承認が発生する

C:
- 柔軟性が高い
- ルールと実装が複雑になる
```

Claude Code の推奨案は判断材料であり、最終決定ではない。

---

## 6. 要件の重要度

すべての項目を同じレベルで人間確認すると、要件決定が遅くなる。

重要度を分類する。

### Critical

人間による明示的な決定を必須とする。

例：

- 業務ルール
- 認証
- 認可
- データ所有権
- 外部公開範囲
- 個人情報・機密情報
- 削除ポリシー
- 監査
- 本番運用
- セキュリティ
- 大きなコストに影響する要件
- システム境界

### Major

Claude Code が推奨案を提示し、必要に応じて人間が決定する。

例：

- 画面構成
- 検索条件
- 通知仕様
- 詳細な入力ルール
- データ表示方法

### Minor

後工程で変更可能であり、大きな影響を与えない項目。

例：

- 文言
- 表示順の微調整
- 内部命名
- 軽微なUX調整

Requirement Discovery では Critical → Major の順に優先する。

---

## 7. 具体例: 承認機能

承認機能について、最初から画面やAPIを考えない。

先に以下を決める。

```text
誰が申請できるか？

誰が承認者を決めるか？

申請者自身が承認者になることは可能か？

承認者は固定か、申請内容によって変わるか？

承認段階はいくつあるか？

並列承認はあるか？

並列承認の場合、全員承認か1名承認か？

否認と差戻しは別の状態か？

承認途中で申請内容を変更できるか？

変更した場合、承認を最初からやり直すか？

承認者が不在の場合はどうするか？

代理承認を許可するか？

代理承認者は誰が設定するか？

承認を取り消せるか？

承認後にデータを変更できるか？

承認履歴をどの程度保持するか？

誰が承認履歴を閲覧できるか？
```

これらが確定して初めて、状態モデル、DB、API、UIなどの設計へ進む。

---

## 8. Decision Log

重要な意思決定は最終要件だけでなく、**なぜその決定をしたか**も残す。

例：

```markdown
## DEC-001 承認後の変更

Status: Accepted

Decision:
承認済み申請を直接変更することは禁止する。
変更が必要な場合は新しいRevisionを作成する。

Alternatives:
- 承認後も直接編集可能
- 特定フィールドのみ編集可能

Reason:
監査証跡を明確に維持するため。

Impact:
- Revision管理が必要
- 過去状態を保持できる
- 監査性が向上する
```

これは後工程で「なぜこの仕様なのか」が失われることを防ぐ。

---

## 9. Requirement Reviewer

重要なシステムでは、最初の要件整理だけで完了としない。

Claude Code に別視点でレビューさせる。

```text
Requirement Discovery
        ↓
要件ドラフト
        ↓
Requirement Review
        ↓
抜け漏れ
矛盾
曖昧な表現
異常系不足
権限漏れ
非機能不足
        ↓
追加質問
        ↓
人間が回答
        ↓
要件更新
        ↓
再レビュー
        ↓
要件確定
```

レビュー時には、実装案ではなく要件品質を確認する。

### Reviewer の観点

- 要件同士に矛盾がないか
- 主語が不明な要件がないか
- 「適切に」「必要に応じて」など曖昧な表現がないか
- 正常系しか定義されていない機能がないか
- 権限が未定義の操作がないか
- 削除・取消・失敗・再実行が定義されているか
- データの所有者が明確か
- Acceptance Criteria が検証可能か
- 非機能要件が不足していないか
- Out of Scope が明確か

---

## 10. Claude Code Skill 化

毎回同じプロンプトを入力するのではなく、Requirement Discovery を Skill として標準化する。

例：

```text
.claude/
└── skills/
    └── requirement-discovery/
        └── SKILL.md
```

利用イメージ：

```text
/requirement-discovery 承認機能
```

Skill には以下を定義する。

- Requirement Discovery の目的
- 質問順序
- Critical / Major / Minor の分類
- 質問フォーマット
- 推奨案の出し方
- Decision Log の記録方法
- 完了条件

Skill は要件決定プロセスを標準化するためのものであり、特定システムの要件そのものを固定化するものではない。

---

## 11. Subagent の利用

必要に応じて役割を分離する。

```text
Business / Product
       ↓
Requirement Analyst
       ↓
人間との対話
       ↓
Requirement Reviewer
       ↓
人間との追加対話
       ↓
Requirements Approved
       ↓
Architecture
       ↓
Implementation Planning
       ↓
Implementation
```

### Requirement Analyst

- 要求整理
- 質問
- 選択肢提示
- 推奨
- Decision Log 更新

### Requirement Reviewer

- 抜け漏れ検出
- 矛盾検出
- エッジケース確認
- セキュリティ観点確認
- 非機能観点確認

### Architecture Agent

要件確定後に初めて使用する。

- 要件からアーキテクチャを作成
- 技術選択
- システム境界設計
- API / DB / Infrastructure 設計

Requirement Analyst と Architecture Agent の責務を分離することで、技術都合で業務要件を早期に歪めることを防ぐ。

---

## 12. 推奨ドキュメント

Requirement Discovery の成果物は以下を基本とする。

```text
docs/
└── requirements/
    ├── requirements.md
    ├── open-questions.md
    └── decision-log.md
```

大きなアーキテクチャ判断は別途 ADR として管理する。

```text
docs/
└── adr/
    ├── ADR-001-authentication.md
    ├── ADR-002-api-boundary.md
    └── ADR-003-data-ownership.md
```

役割は以下の通り。

| Document | Purpose |
|---|---|
| requirements.md | 確定した要件 |
| open-questions.md | 未確定事項 |
| decision-log.md | 意思決定と理由 |
| ADR | 重要なアーキテクチャ判断 |

---

## 13. 完了条件

Requirement Discovery の完了条件を明確にする。

最低限、以下を満たすまで設計・実装フェーズへ進まない。

- Critical Open Questions が 0
- システム・機能の目的が明確
- Actors が定義されている
- 主要な業務フローが定義されている
- 主要な異常系が定義されている
- 認証・認可要件が定義されている
- データ所有者が定義されている
- 主要な非機能要件が定義されている
- Out of Scope が定義されている
- Acceptance Criteria が検証可能
- Critical Decision が人間によって承認済み

Minor な項目まで完全に決定する必要はない。

重要なのは、後工程で大きな手戻りを発生させる項目を先に確定することである。

---

## 14. 推奨する全体フロー

```text
Idea / Business Request
        ↓
Requirement Discovery
        ↓
Claude Code が質問
        ↓
人間が回答・決定
        ↓
Decision Log
        ↓
Requirement Reviewer
        ↓
追加質問
        ↓
Critical Open Questions = 0
        ↓
Requirements Approved
================================
        ↓
Architecture Design
        ↓
Architecture Review
        ↓
Implementation Plan
        ↓
Implementation
        ↓
Test
        ↓
Release
```

`Requirements Approved` を、要件決定フェーズと設計・実装フェーズの明確な境界とする。

---

## 15. 推奨構成

最初から多数の Agent を構築する必要はない。

まずは以下の構成から開始する。

```text
1. requirement-discovery Skill
        ↓
2. requirements.md
        ↓
3. decision-log.md
        ↓
4. Requirement Review
        ↓
5. Requirements Approved
```

運用が成熟した段階で、Requirement Reviewer や Architecture Agent を Subagent として分離する。

重要なのは Agent の数ではなく、**「質問するAI」「決定する人間」「決定を記録する仕組み」を分けること**である。
