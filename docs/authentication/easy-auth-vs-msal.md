# App Service Easy Auth と MSAL の使い分け

## 結論

Easy Auth と MSAL は、どちらか一方だけを常に選ぶ関係ではない。

- **Easy Auth**：App Service の入口で認証を処理・検証するAzureの機能
- **MSAL**：React SPAなどのクライアントがMicrosoft Entra IDへサインインし、API用アクセストークンを取得するSDK

判断基準は次の1点である。

> **ブラウザ上のReact SPAが、別の保護されたAPIをBearerトークンで直接呼ぶか。**

呼ぶならMSALを使う。呼ばないなら、フロントはEasy Authだけで開始できる。

---

## 1. それぞれが担当すること

| 観点 | Easy Auth（App Service Authentication） | MSAL |
|---|---|---|
| 実行場所 | App Serviceのプラットフォーム側 | Reactなどのアプリコード |
| 主な目的 | リクエストの認証、トークン検証、セッション管理 | サインイン、アクセストークン取得・更新 |
| 未認証アクセス | Entra IDログインへリダイレクト、または401を返す | アプリコードからログイン画面を起動する |
| API呼び出し用トークン | 受信・検証する側 | 取得してBearerヘッダーへ付与する側 |
| アプリコード | 少なくできる | React側に実装する |
| 認可 | 業務権限の判定は代替しない | 業務権限の判定は代替しない |

重要なのは、**認証と認可は別**であること。

- 認証：そのユーザーは誰か
- 認可：そのユーザーは、そのReqを承認・編集できるか

Easy AuthまたはMSALで認証しても、API側ではロール・部署・案件・金額閾値などを基に業務認可を行う。

---

## 2. Easy Authだけでよいケース

次の条件なら、フロントでMSALを使わずEasy Authだけで開始できる。

- Reactの配信とAPIが同じApp Service、または同一オリジン配下にある
- ブラウザCookieで認証済みセッションを維持できる
- SPAが別App ServiceのAPI、Microsoft Graph、外部SaaS APIを直接呼ばない
- API用アクセストークンをReactから明示的に扱う必要がない

### 流れ

```text
ブラウザ
  ↓ 未ログインでアクセス
App Service Easy Auth
  ↓ Entra IDへリダイレクト
Entra ID
  ↓ ログイン後にCookieを返す
ブラウザ
  ↓ Cookie付きで同一アプリの画面・APIを呼ぶ
App Service
```

### 利点

- React側の認証実装が少ない
- ログイン制御をApp Service設定へ寄せられる
- MVPを早く立ち上げやすい

### 注意点

別App ServiceのAPIをReactから直接呼ぶ構成へ広げると、Cookieだけでは扱いづらくなる。そこでMSALへ移行、または最初からMSALを採用する。

---

## 3. MSALが必要になるケース

次のいずれかに当てはまるなら、React SPAでMSALを使う。

- フロントとAPIが別App Serviceで、APIをBearerトークンで保護する
- 共通認可API・承認APIなど、複数のAPIをReactから呼ぶ
- Microsoft Graph APIをユーザー権限で呼ぶ
- APIごとにScope（例：`api://.../access_as_user`）を明示して最小権限にしたい
- SPA側でトークンの取得・失効・再取得を制御したい

### 流れ

```text
React SPA
  ↓ MSALでEntra IDへサインイン
Entra ID
  ↓ APIのScopeを含むアクセストークンを返す
React SPA
  ↓ Authorization: Bearer <access token>
API
  ↓ トークンを検証して処理
```

APIのトークン検証は、次のどちらかで実装する。

1. **Easy Authで検証する**  
   App Serviceの入口で認証済みリクエストだけを通す。APIコードは業務認可へ集中する。

2. **Node.js APIのミドルウェアでJWTを検証する**  
   より細かな制御が必要な場合に採用する。

同じ検証を無目的に二重実装しない。標準はEasy Authによる入口検証を優先し、例外的な要件がある場合だけアプリ側検証を追加する。

---

## 4. 本プラットフォームでの推奨

本プラットフォームは、Portalと複数の業務アプリ、共通認可・承認APIへ拡張する前提である。そのため標準は次の構成とする。

| 層 | 標準 | 理由 |
|---|---|---|
| React SPA | **MSAL + Entra ID** | 複数の保護APIへScope付きトークンでアクセスするため |
| Node.js API（App Service） | **Easy Auth + Entra ID** | API到達前にトークンを検証するため |
| APIの認可 | 共通認可APIまたはアプリ側の認可ロジック | RBAC、案件・組織・金額・承認段階を判定するため |

### App Registrationの分離

SPAとAPIは別のApp Registrationとする。

- **SPA App Registration**：リダイレクトURIを持つ。APIのScopeを要求する
- **API App Registration**：Scopeを公開する。受信トークンのAudienceとなる

開発環境と本番環境も分離し、テスト用の設定変更が本番へ影響しないようにする。

---

## 5. 選定フローチャート

```text
React SPAが別の保護APIを直接呼ぶか？
  ├─ いいえ
  │   └─ Easy Authだけで開始可能
  │
  └─ はい
      └─ ReactはMSALでトークン取得
          APIはEasy AuthまたはAPIミドルウェアで検証
```

---

## 6. 設計・実装時のチェックリスト

- [ ] APIごとに必要なScopeを定義した
- [ ] SPAとAPIのApp Registrationを分離した
- [ ] 開発環境と本番環境のApp Registrationを分離した
- [ ] App ServiceのEasy Authで未認証リクエストの扱いを設定した
- [ ] APIでは認証後も業務認可を必ず実施する
- [ ] フロントからクライアントシークレットを扱わない
- [ ] アクセストークンをログへ出力しない
- [ ] APIを直接公開せず、Application Gateway / APIM / Private Endpoint等のネットワーク設計と整合させる

---

## 7. よくある誤解

### Easy Authを使えばMSALは不要？

**常に不要ではない。**  
同一App Service・同一オリジンのCookieベース構成なら不要にできる。一方、SPAが別保護APIを呼ぶなら、SPA側でトークンを取得するMSALが必要になる。

### MSALを使えばEasy Authは不要？

**不要とは限らない。**  
MSALはSPAがトークンを取得するSDKであり、API入口での検証・セッション処理は別の責務である。App Service APIではEasy Authに検証を任せる選択ができる。

### Easy Authがあれば認可もできる？

**できない。**  
Easy Authは主に「誰か」を確認する。業務上の「何をしてよいか」は、APIと共通認可基盤で判定する。

---

## 参考

- [Azure App Service Authentication and Authorization (Easy Auth)](https://learn.microsoft.com/azure/app-service/overview-authentication-authorization)
- [Microsoft Authentication Library (MSAL)](https://learn.microsoft.com/entra/msal/)
