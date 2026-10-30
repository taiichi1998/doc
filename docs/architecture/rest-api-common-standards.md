# 汎用Webアプリ開発テンプレート：REST API共通仕様

更新日: 2026-10-30  
状態: 基本方針合意（詳細実装・例外条件は今後設計）

## 目的・適用範囲

特定の業務アプリに依存しない、再利用可能なREST APIの設計規約。既定はNode.js + TypeScriptだが、URL・HTTP・レスポンス・日時・OpenAPI等の規約は他言語にも適用する。認証・認可の詳細は[共通認証設計](azure-webapp-bff-auth-security-and-authorization-cache.md)を参照。

## 決定した10項目

| # | 項目 | 標準方針 | 補足 |
|---|---|---|---|
| 1 | API方式 | REST | GraphQL/gRPCは必要に応じて別テンプレート |
| 2 | API契約 | OpenAPI 3.1系 | OAS 3.2系への移行はツール互換性と必要機能を確認後 |
| 3 | URL設計 | リソース指向（名詞 + HTTPメソッド） | 一意リソースはパス、検索条件はクエリ。深い階層の設計は別途 |
| 4 | バージョン管理 | URLに `/api/v1` を含める | 破壊的変更時にv2を検討。OpenAPI仕様書バージョンとは別 |
| 5 | 成功レスポンス | データ直接返却 | 一覧はページング用メタデータを持つ構造を許可 |
| 6 | エラー形式 | RFC 9457 Problem Details | HTTPステータスと整合、`application/problem+json` |
| 7 | ページング | Offsetを基本 | Page表現も許容。大規模・頻繁な更新時はCursorを検討 |
| 8 | 入力検証 | Zod（Node.js/TypeScript） | 他言語は同等のサーバーサイド検証を採用 |
| 9 | 日時表現 | ISO 8601 / RFC 3339、UTC `Z` | 日付のみは `YYYY-MM-DD`、表示時にタイムゾーン変換 |
| 10 | APIドキュメント | Scalar（OSS版） | OpenAPIを表示。開発環境限定または適切なアクセス制御 |

## URL設計の標準

```http
GET    /api/v1/projects
GET    /api/v1/projects/123
POST   /api/v1/projects
PATCH  /api/v1/projects/123
DELETE /api/v1/projects/123
GET    /api/v1/projects/123/documents
GET    /api/v1/documents?status=active
```

- `/getProject` 等のアクション名を原則URLに置かず、HTTPメソッドで操作を示す。
- `/projects/123` は特定リソース、`?status=active` は検索・絞り込み条件。
- パスが深い場合、全階層を機械的に並べず、適切な親リソースを起点にする。具体的な深さの上限は未決定。
- 認可はURLの形に依存させず、API側で毎回対象リソースを確認する。

## 複数アプリを同一ゲートウェイに収容するとき

各アプリの**内部URLは `/api/v1/{resource}` のまま**とし、ゲートウェイ側でアプリ識別子を付けた公開URLにする方式を採用できる。

```text
公開URL:    /api/v1/rep/projects
Gateway:    /api/v1/rep/* を REP APIにルーティング
Rewrite:    /api/v1/rep/projects → /api/v1/projects
アプリ内部: /api/v1/projects
```

- Azure Application Gateway v2のパスベースルーティングとURL Rewrite等で実現可能。書き換え・ルール評価順序は構成時に検証する。
- 個別アプリを直接公開する場合は `/api/v1/{resource}` のままでよい。
- フロントエンドの参照先、相対リンク、OpenAPI servers、リダイレクト、Cookie Path、旧URL利用者との互換性は別途確認する。
- これはゲートウェイ統合時の選択肢であり、**汎用テンプレートの各アプリに `rep` のような識別子を埋め込まない**。

## レスポンス例

成功（単体取得、HTTP 200、`application/json`）:

```json
{
  "id": 123,
  "name": "Project A"
}
```

エラー（HTTP 403、`application/problem+json`）:

```json
{
  "type": "about:blank",
  "title": "Forbidden",
  "status": 403,
  "detail": "You do not have permission.",
  "instance": "/api/v1/projects/123"
}
```

- `success: true` や `data` の一律ラッパーは不要。
- 一覧のページング用ラッパー詳細（`items`, `total`, `nextCursor`等）、HTTP 201/204、バリデーションエラー拡張フィールド、冪等性、ソート等は今後別途決定する。
- エラー詳細に内部スタックトレースや機密情報を含めない。

## 実装・運用上の注意

- OpenAPI定義をリポジトリで管理し、実装と一致させる。CIで整合性検証を検討する。
- Zodは実行時入力検証。TypeScriptの型定義だけでは入力検証にならない。
- Offsetページングでは並び順を安定させ、巨大データセットや更新頻度が高い一覧はCursorへ切替を検討する。
- UTCの時刻は `2026-10-30T10:00:00Z` のように送る。日付だけの項目に時刻・タイムゾーンを強制しない。
- Scalarの試験機能やAPI仕様はアクセス制限を設定し、公開環境では必要性を審査する。

## 継続検討

1. 深いリソース階層・関連リソースのURL設計
2. 一覧取得レスポンスの具体的JSONスキーマ、limit/offsetの上限と既定値
3. HTTPステータス・エラーコード・入力検証エラーの詳細
4. OpenAPI生成方法、Schemaの単一ソース、Scalarの組込み方法
5. Gateway Rewrite、BFF、各APIのパス境界および公開URLの検証

## 参考

- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457)
- [RFC 3339: Date and Time on the Internet](https://www.rfc-editor.org/rfc/rfc3339)
- [Scalar](https://github.com/scalar/scalar)
- [Azure Application Gateway URL Rewrite](https://learn.microsoft.com/en-us/azure/application-gateway/rewrite-http-headers-url)
