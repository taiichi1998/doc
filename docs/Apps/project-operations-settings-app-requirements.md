# プロジェクト運用設定アプリ 要件

## 1. 目的

JDMSの設定Excelを基に、図書処理の運用に必要なプロジェクト・部門・配布先・利用者権限を参照・管理する。

本アプリは、図書処理状況アプリとは分離した管理者向けの Power Apps Canvas App とする。

## 2. 対象範囲

| 区分 | 設計 |
|---|---|
| 利用者向け画面 | Power Apps Canvas App |
| 状態の保存先 | SharePoint List |
| 基本データの正 | JDMS の設定Excel |
| JDMSとの連携 | 管理者による設定Excelの手動取込 |
| 対象 | プロジェクト、部門、配布先メール、利用者権限、JDMS設定Excel取込 |
| 対象外 | JDMS・Corretへの設定の自動書込み、PAD実行設定、メンテナンス設定、メール通知、個別の変更履歴台帳 |

## 3. 基本方針

- JDMSの設定Excelを、プロジェクト・部門・配布先メールに関する基本データの正とする。
- JDMSの設定Excelを更新できる権限者は複数いるため、特定のSharePointフォルダを更新起点にした自動同期は行わない。
- JDMS更新後、権限者がプロジェクト運用設定アプリから最新のJDMS設定Excelを手動取込する。
- 取込結果をSharePoint Listへ反映し、図書処理状況アプリおよび配布処理で利用する。
- SharePoint Listの情報をJDMSへ自動反映しない。二重の正を作らない。

## 4. 権限

| ロール | 権限 |
|---|---|
| SystemAdmin | 全プロジェクトの閲覧・管理、ProjectAdminの登録、JDMS設定Excelの取込 |
| ProjectAdmin | 自プロジェクトの配布先・仮部門・自プロジェクトのProjectAdminを管理 |
| SettingsImportOperator | JDMS設定Excelの取込と取込結果の確認 |
| User | 本アプリへのアクセス不可 |

### 初期設定

- システム側でSystemAdminを1名設定する。
- SystemAdminが各プロジェクトのProjectAdminを登録する。
- ProjectAdminは自プロジェクト内に限り、他のProjectAdminを追加できる。
- 権限は単純な管理者チェックではなく、ロールで管理する。

## 5. 管理対象データ

### 5.1 プロジェクト

JDMS・Corretに登録されているプロジェクトを本アプリへ紐付けて利用する。

| 項目 | 内容 |
|---|---|
| project_code | 共通のプロジェクトコード。業務上の識別子 |
| project_name | プロジェクト名称 |
| jdms_project_id | JDMS上のプロジェクト識別子 |
| corret_project_id | Corret上のプロジェクト識別子 |
| is_active | 有効・無効 |
| source_updated_at | JDMS設定Excel上の更新日時または取込日時 |

本アプリからプロジェクトを自由に新規作成しない。JDMS設定Excelの取込により登録・更新する。

### 5.2 プロジェクト部門

部門コード・名称はプロジェクトごとに差異があるため、プロジェクト別部門マスタとして管理する。

一意判定は `project_code + department_code` とする。

| 項目 | 内容 |
|---|---|
| project_code | プロジェクトコード |
| department_code | プロジェクト内の部門コード |
| department_name | 部門名称 |
| is_temporary | 仮登録か |
| is_active | 有効・無効 |
| source_updated_at | 取込日時 |

### 5.3 配布先メールアドレス

社内・社外を問わず登録可能とする。名前は管理対象外とする。

一意判定は `project_code + normalized_email` とする。

| 項目 | 内容 |
|---|---|
| project_code | プロジェクトコード |
| normalized_email | 小文字化・空白除去したメールアドレス |
| email | 表示用メールアドレス |
| department_code | 紐付けるプロジェクト別部門コード |
| is_distribution_target | 配布通知の対象か |
| is_external_cc | 社外メール時に常時CCへ入れるか |
| is_active | 有効・無効 |
| source | JDMSImport / ManualTemporary |
| last_updated_by_upn | 最終更新者UPN |
| last_updated_at | 最終更新日時 |

### 5.4 プロジェクト利用者・ロール

| 項目 | 内容 |
|---|---|
| project_code | 対象プロジェクト。SystemAdminは空欄または全件権限で管理 |
| user_upn | 利用者のUPN |
| role | SystemAdmin / ProjectAdmin / SettingsImportOperator |
| is_active | 有効・無効 |
| last_updated_by_upn | 最終更新者UPN |
| last_updated_at | 最終更新日時 |

## 6. 画面構成

### 6.1 ダッシュボード

- 取込対象プロジェクトの選択
- 最終取込日時
- プロジェクト・部門・配布先の件数
- 仮登録部門の未解決件数
- JDMS設定Excel取込画面への導線
- 権限に応じた管理画面への導線

### 6.2 JDMS設定Excel取込

- 最新のJDMS設定Excelをアップロード
- 取込前の形式・必須列チェック
- 追加・更新・無効化予定件数の表示
- エラー行と理由の表示
- 反映実行ボタン
- 取込日時・取込実行者の表示

取込データにエラーがある場合は、SharePoint Listを更新しない。

### 6.3 部門管理

- 選択プロジェクトの正式部門一覧
- 仮部門の登録、修正、確定、無効化
- 仮登録部門には `仮登録` を表示
- ProjectAdminは自プロジェクトのみ操作可能
- 正式部門はJDMS設定Excel取込で更新する

### 6.4 配布先管理

- プロジェクト・部門での絞込み
- メールアドレスの一覧表示
- 配布対象フラグ、常時CCフラグ、有効・無効の編集
- 社内・社外メールアドレスをともに登録可能
- 部門はプロジェクト別部門マスタから選択する

### 6.5 プロジェクト利用者・権限管理

- ProjectAdminの一覧表示
- ProjectAdminの追加・無効化
- SystemAdminは全プロジェクトを操作可能
- ProjectAdminは自プロジェクト内のProjectAdminだけを管理可能

## 7. JDMS設定Excel取込

### 運用手順

1. JDMS権限者が最新のJDMS設定Excelをダウンロードする。
2. 設定を変更し、JDMSへアップロードする。
3. 更新後の最新JDMS設定Excelをプロジェクト運用設定アプリで取込する。
4. 取込結果を確認し、SharePoint Listへ反映する。

### 取込ルール

- JDMS設定Excelのテンプレートをそのまま利用する。
- 本アプリ専用の別テンプレートは作成しない。
- 既存レコードは業務キーで更新し、存在しないレコードは追加する。
- JDMS設定Excelから消えたデータは、即時削除せず `is_active = false` として無効化する。
- 取込失敗時は、前回成功時点のSharePoint Listデータを継続利用する。

## 8. 仮部門登録

JDMS設定Excelが日次更新であり、当日に新しい部門を利用する必要がある場合に限り、仮登録を認める。

- 仮登録・確定・修正はProjectAdmin以上のみ可能
- UserおよびSettingsImportOperatorは仮登録できない
- 仮登録部門は `is_temporary = true` とする
- 次回JDMS設定Excel取込時に、正式部門との一致を確認する
- 一致した場合は正式部門として確定する
- 一致しない場合は仮登録状態を維持し、管理画面上で確認対象として表示する

## 9. 変更履歴

専用の監査ログは作成しない。

SharePoint Listの標準列および業務列で、最低限以下を確認可能とする。

- 作成日時
- 作成者
- 更新日時
- 更新者
- `last_updated_by_upn`

Power Automateによる更新でSharePoint標準の更新者がフロー用アカウントになる場合は、`last_updated_by_upn` に実行者を記録する。

## 10. 図書処理状況アプリとの連携

図書処理状況アプリは、プロジェクト運用設定アプリが管理するSharePoint Listを参照する。

主な利用目的は以下とする。

- ログインユーザーが選択可能なプロジェクトの判定
- プロジェクトごとの部門・配布先メールアドレスの参照
- 運用者・管理者の閲覧範囲の判定
- 最新のJDMS設定情報に基づく図書処理の運用
