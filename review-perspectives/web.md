# Nablarch Web アプリケーション レビュー観点プリセット
対象: Nablarch Webアーキテクチャ（nablarch-fw-web）

## 観点一覧

| ID | タイトル | 優先度 | 観点概要 | 根拠 |
|----|---------|:------:|---------|------|
| [WEB-001](#web-001) | ハンドラキューの順序・必須ハンドラの確認 | **MUST** | 必須ハンドラが適切な順序で登録されているか（認証バイパス・例外漏れを防ぐ） | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html) |
| [WEB-002](#web-002) | セッションスコープの適切な使用とタイムアウト設定 | **MUST** | SessionUtilの操作が適切で、セッション固定攻撃・タイムアウト未設定がないか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/session_store.html) |
| [WEB-003](#web-003) | CSRFトークンの生成・検証が全更新系リクエストに適用されているか | **MUST** | POST/PUT/DELETE等の更新系リクエストにCSRFトークン検証が実装されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html#csrf) |
| [WEB-004](#web-004) | フォーム入力値のサーバーサイドバリデーション確認 | **MUST** | `ValidationUtil.validate()`またはBean Validationがサーバーサイドで実装されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/validation.html) |
| [WEB-005](#web-005) | 正常系・異常系の全遷移先が定義されているか | **Should** | バリデーションエラー・システムエラー・404等の異常系画面が全て定義されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html) |
| [WEB-006](#web-006) | アクションクラスと業務ロジッククラスの責務分離確認 | **Should** | アクションがHTTPリクエスト/レスポンス処理のみを担当し、業務ロジックが委譲されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/architecture.html) |
| [WEB-007](#web-007) | JSPでの動的データ出力時のXSSエスケープ確認 | **MUST** | `<n:write>`または`<c:out>`でエスケープされているか（EL式の直接使用がないか） | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/tag.html) |
| [WEB-008](#web-008) | SQL構築に動的文字列連結を使用していないか | **MUST** | UniversalDaoやSQLファイルのバインドパラメータを使用し、文字列連結がないか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/database.html) |
| [WEB-009](#web-009) | 認証済みユーザーのみが保護リソースにアクセスできるか | **MUST** | 未認証アクセスが適切にリダイレクトされ、ロールチェックが実装されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/permission_check.html) |
| [WEB-012](#web-012) | 画面文言のハードコード排除とメッセージプロパティ管理 | **Should** | 画面文言・エラーメッセージがメッセージプロパティファイルで管理されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/message.html) |
| [WEB-014](#web-014) | Nablarch TestSupportを使ったウェブアクション単体テスト設計 | **Should** | `HttpRequestTestSupport`を使ったリクエスト/レスポンス/セッション検証が実装されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/index.html) |
| [WEB-015](#web-015) | X-Content-Type-Options・X-Frame-Options等のセキュリティヘッダ設定 | **Should** | レスポンスにセキュリティHTTPヘッダが設定されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html#csrf) |
| [WEB-016](#web-016) | ログアウト時のセッション完全破棄とリダイレクト確認 | **MUST** | `SessionUtil.invalidate()`でセッションが完全に破棄されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/session_store.html) |

---

## 各観点詳細

### WEB-001
**ハンドラキューの順序・必須ハンドラの確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | `web-component-configuration.xml`等の`handlerQueue`設定 |
| FW提供範囲 | 各ハンドラの実際の処理ロジックはFWが提供。実装者はハンドラの種類と順序の設定を担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html) |
| 優先度理由 | ハンドラキューはリクエスト処理の根幹であり、順序誤りは認証・認可の無効化やCSRF対策の欠落につながる |
| チェック方法 | `HttpCharacterEncodingHandler`が最前列に配置されているか。`HttpResponseHandler`が後段に配置されているか。認証ハンドラが業務ハンドラより前に配置されているか。`SessionConcurrentAccessHandler`（セッション排他）が必要な場合に設定されているか |

**NG例**
```xml
<!-- 認証ハンドラが業務アクションより後 → 認証バイパス可能 -->
<list name="handlerQueue">
    <component-ref name="businessActionDispatcher"/>
    <component-ref name="authenticationHandler"/> <!-- 順序誤り -->
</list>
```

**OK例**
```xml
<list name="handlerQueue">
    <component-ref name="httpCharacterEncodingHandler"/>
    <component-ref name="authenticationHandler"/>
    <component-ref name="csrfTokenHandler"/>
    <component-ref name="businessActionDispatcher"/>
</list>
```

---

### WEB-002
**セッションスコープの適切な使用とタイムアウト設定**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | アクションクラスの`SessionUtil.put()`/`get()`/`invalidate()`呼び出しコード |
| FW提供範囲 | セッションストアへの実際の読み書き（往路・復路）はFW（セッション変数保存ハンドラ）が自動実行。実装者は`SessionUtil` APIを正しく使うことを担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/session_store.html) |
| 優先度理由 | セッション管理の不備はセッションハイジャック・固定攻撃・メモリ圧迫の原因となる |
| チェック方法 | `SessionUtil.put()`で格納するオブジェクトが最小限か（大容量オブジェクト不可）。ログイン成功後に`SessionUtil.invalidate()`→新セッション発行しているか（セッション固定対策）。セッションタイムアウト値が`web.xml`または設定ファイルで明示されているか。DBセッションストア使用時、セッションデータのスキーマが正しく定義されているか |

**NG例**
```java
// ログイン成功後にセッション固定攻撃が可能
SessionUtil.put(context, "loginUser", user); // セッション無効化なし
```

**OK例**
```
SessionUtil.invalidate(context); // 旧セッション破棄
SessionUtil.put(context, "loginUser", user); // 新セッションに格納
```

---

### WEB-003
**CSRFトークンの生成・検証が全更新系リクエストに適用されているか**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | `web-component-configuration.xml`の`CsrfTokenHandler`設定、JSPの`<n:csrf>`タグ配置 |
| FW提供範囲 | CSRFトークンの生成・検証ロジック自体はFW（`CsrfTokenHandler`）が実行。実装者はハンドラ設定とJSPへの`<n:csrf>`タグ配置を担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html#csrf) |
| 優先度理由 | CSRF対策の欠落は、悪意あるサイトからの不正リクエストによるデータ改ざんを許す。金融・行政系システムでは致命的な脆弱性となる |
| チェック方法 | `CsrfTokenHandler`がハンドラキューに登録されているか。JSPの更新フォームに`<n:csrf>`タグまたは相当のトークンhiddenフィールドが含まれているか。AJAXリクエストでCSRFトークンがヘッダまたはボディに付与されているか。GETリクエスト専用の除外設定が適切か（CSRFはGETでは不要） |

**NG例**
```xml
<!-- CSRF トークン未設定 -->
<form action="/update" method="POST">
    <input type="submit" value="更新"/>
</form>
```

**OK例**
```xml
<form action="/update" method="POST">
    <n:csrf/>  <%-- CSRF トークン自動付与 --%>
    <input type="submit" value="更新"/>
</form>
```

---

### WEB-004
**フォーム入力値のサーバーサイドバリデーション確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | アクションクラスの`ValidationUtil.validate()`呼び出し、FormクラスのValidationアノテーション定義 |
| FW提供範囲 | バリデーション実行エンジン（アノテーション評価）はFWが提供。実装者はバリデーションアノテーション定義と`validate()`呼び出しを担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/validation.html) |
| 優先度理由 | クライアントサイドのみのバリデーションはBurp Suite等のツールで容易にバイパス可能。不正入力のDB登録を防ぐためサーバーサイド検証は必須 |
| チェック方法 | アクションクラスで`ValidationUtil.validate()`が呼ばれているか。バリデーションアノテーション（`@Required`, `@Length`, `@Domain`等）がFormクラスに定義されているか。バリデーションエラー時に入力画面に戻り、エラーメッセージが表示されるか |

**NG例**
```java
// バリデーションなしで直接 DB 操作
public HttpResponse doPost(HttpRequest req, ExecutionContext ctx) {
    UserForm form = SessionUtil.get(ctx, "form");
    dao.insert(form); // バリデーションなし
}
```

**OK例**
```java
public HttpResponse doPost(HttpRequest req, ExecutionContext ctx) {
    UserForm form = SessionUtil.get(ctx, "form");
    ValidationUtil.validate(form);
    dao.insert(form);
}
```

---

### WEB-005
**正常系・異常系の全遷移先が定義されているか**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | `web.xml`の`error-page`設定、`web-component-configuration.xml`の`ErrorHandler`設定 |
| FW提供範囲 | 例外捕捉とエラーページ遷移の基盤機構はFWが提供。実装者はエラーページのURL設定と全エラーコードのカバレッジを担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html) |
| 優先度理由 | エラー画面未定義は内部エラー情報のユーザー露出（スタックトレース表示）につながり、セキュリティリスクとなる |
| チェック方法 | `web-component-configuration.xml`に`ErrorHandler`でエラーページが設定されているか。HTTP 404/500の各エラーページが`web.xml`に定義されているか。スタックトレースが画面に表示されないか（本番設定確認） |

**NG例**
```xml
<!-- エラーページ未設定 → デフォルトのスタックトレースが表示される -->
```

**OK例**
```xml
<error-page>
    <error-code>404</error-code>
    <location>/WEB-INF/view/error/pageNotFound.jsp</location>
</error-page>
<error-page>
    <error-code>500</error-code>
    <location>/WEB-INF/view/error/systemError.jsp</location>
</error-page>
```

---

### WEB-006
**アクションクラスと業務ロジッククラスの責務分離確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | アクションクラス（業務ロジック混入確認）、サービスクラス・ドメインクラスの分離状況 |
| FW提供範囲 | HTTPリクエスト/レスポンスの変換はFWが担当。業務ロジックの責務配置設計は実装者が担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/architecture.html) |
| 優先度理由 | アクションに業務ロジックが集中すると、テスト困難・変更影響範囲の拡大・コードの可読性低下が発生する |
| チェック方法 | アクションクラスが`HttpRequest`/`ExecutionContext`/`HttpResponse`の処理のみを行っているか。SQLや業務計算ロジックがアクションクラスに直接書かれていないか。業務ロジッククラスが`SystemRepository.get()`で取得されているか |

**NG例**
```java
public HttpResponse doPost(HttpRequest req, ExecutionContext ctx) {
    // アクション内に直接 SQL
    SqlResultSet result = DbConnectionContext.getConnection()
        .query("SELECT * FROM ...");
    // 業務計算もアクション内
    int total = calcTotal(result);
}
```

**OK例**
```java
public HttpResponse doPost(HttpRequest req, ExecutionContext ctx) {
    OrderService service = SystemRepository.get("orderService");
    OrderResult result = service.processOrder(form);
    SessionUtil.put(ctx, "result", result);
    return new HttpResponse("/WEB-INF/view/complete.jsp");
}
```

---

### WEB-007
**JSPでの動的データ出力時のXSSエスケープ確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | JSPファイルの動的値出力箇所（`<n:write>`タグ使用状況、EL式の直接使用確認） |
| FW提供範囲 | `<n:write>`タグ内部のHTMLエスケープ処理はFWが実行。実装者は適切なタグ（`<n:write>`または`<c:out>`）を選択することを担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/tag.html) |
| 優先度理由 | XSSはOWASP Top 10の常連脆弱性。ユーザー入力の未エスケープ出力はスクリプトインジェクションによるアカウント乗っ取りにつながる |
| チェック方法 | JSPでの動的値出力に`<n:write>`または`<c:out>`を使用しているか。EL式`${...}`を直接HTML属性・本文に使用していないか（EL式はエスケープされない場合がある）。`<script>`タグ内での動的値出力にはJavaScriptエスケープが適用されているか |

**NG例**
```xml
<!-- 未エスケープ → XSS 脆弱性 -->
<%= request.getParameter("name") %>
<!-- または -->
${user.comment}
```

**OK例**
```xml
<n:write name="user.comment"/>
<%-- または --%>
<c:out value="${user.comment}"/>
```

---

### WEB-008
**SQL構築に動的文字列連結を使用していないか**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | SQLファイル（`.sql`）のバインドパラメータ定義、`UniversalDao`呼び出し箇所 |
| FW提供範囲 | バインドパラメータによる安全なSQL実行はFWが提供。実装者はSQL文字列連結を使わないことを担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/database.html) |
| 優先度理由 | SQLインジェクションは全データ漏洩・削除・認証バイパスを引き起こす最も危険な脆弱性の一つ |
| チェック方法 | SQLファイル内でユーザー入力を直接文字列連結していないか。`UniversalDao.findAllBySqlFile()`のパラメータがバインド変数経由か。動的条件が`$if`等のNablarchテンプレート機能で組み立てられているか |

**NG例**
```java
// SQL インジェクション脆弱性
String sql = "SELECT * FROM users WHERE name = '" + name + "'";
```

**OK例**
```java
// バインドパラメータで安全
UserCondition cond = new UserCondition();
cond.setName(name);
List<User> users = UniversalDao.findAllBySqlFile(User.class, "FIND_BY_NAME", cond);
```

---

### WEB-009
**認証済みユーザーのみが保護リソースにアクセスできるか**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | `web-component-configuration.xml`の`AuthenticationHandler`設定、アクションクラスの`PermissionUtil`呼び出し |
| FW提供範囲 | 認証チェックのハンドラ機構はFWが提供。実装者はハンドラ設定と権限チェック（`PermissionUtil.permit()`等）の呼び出しを担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/permission_check.html) |
| 優先度理由 | 認証・認可の欠落は不正アクセスによる情報漏洩・操作の直接的な原因となる |
| チェック方法 | `AuthenticationHandler`がハンドラキューに登録され、未認証時にlogin画面へリダイレクトされるか。権限が必要な画面・APIでは`PermissionUtil.permit()`または相当の権限チェックが実装されているか。テスト：ログアウト後に保護URLへ直接アクセスした場合の動作確認 |

**NG例**
```java
// 権限チェックなしで管理画面にアクセス可能
public HttpResponse doGet(HttpRequest req, ExecutionContext ctx) {
    return new HttpResponse("/WEB-INF/view/admin.jsp"); // 誰でもアクセス可
}
```

**OK例**
```java
public HttpResponse doGet(HttpRequest req, ExecutionContext ctx) {
    PermissionUtil.permit(ctx, "ADMIN_SCREEN");
    return new HttpResponse("/WEB-INF/view/admin.jsp");
}
```

---

### WEB-012
**画面文言のハードコード排除とメッセージプロパティ管理**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | `message.properties`等のメッセージ定義ファイル、JSPの`<n:message>`タグ使用状況 |
| FW提供範囲 | メッセージIDから文言への解決はFW（`MessageUtil`）が実行。実装者はメッセージ定義ファイルの作成と`<n:message>`タグの使用を担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/message.html) |
| 優先度理由 | メッセージのハードコードは多言語対応を困難にし、文言変更時の修正漏れを招く |
| チェック方法 | エラーメッセージが`message.properties`または同等ファイルで管理されているか。JSPで`<n:message>`タグまたは`MessageUtil.createMessage()`が使われているか。日本語文言がJSPに直書きされていないか |

**NG例**
```xml
<!-- メッセージハードコード -->
<span class="error">入力値が不正です。必須項目を確認してください。</span>
```

**OK例**
```xml
<n:message messageId="errors.required" arg0="氏名"/>
```

---

### WEB-014
**Nablarch TestSupportを使ったウェブアクション単体テスト設計**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | テストクラス（`HttpRequestTestSupport`を継承したクラス） |
| FW提供範囲 | テストサポートクラス（`HttpRequestTestSupport`等）はFWが提供。テストケースの実装と正常系・認証エラー系のカバレッジは実装者が担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/index.html) |
| 優先度理由 | テストのないウェブアクションはリグレッションに気づかず、セキュリティ修正の影響範囲把握も困難になる |
| チェック方法 | `HttpRequestTestSupport`を継承したテストクラスが存在するか。テストデータがExcelまたはYAMLで管理されているか。正常遷移・バリデーションエラー・認証エラーのテストケースが存在するか |

**NG例**
```java
// アクションクラスのテストなし
```

**OK例**
```java
public class UserActionTest extends HttpRequestTestSupport {
    @Test
    public void 正常登録() {
        HttpResponse response = execute("POST", "/user/register", "testCases/register.xlsx");
        assertThat(response.getStatusCode(), is(302));
    }
}
```

---

### WEB-015
**X-Content-Type-Options・X-Frame-Options等のセキュリティヘッダ設定**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | `web.xml`のフィルタ設定、またはアクションクラスの`response.setHeader()`呼び出し |
| FW提供範囲 | セキュリティヘッダを自動設定するFW機能は標準では提供されない。実装者がフィルタまたはアクション内で`X-Frame-Options`等を設定する必要がある |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html#csrf) |
| 優先度理由 | セキュリティヘッダの欠如はクリックジャッキング・MIMEスニッフィング等のブラウザレベルの攻撃に対して無防備になる |
| チェック方法 | `X-Frame-Options: DENY`または`SAMEORIGIN`が設定されているか。`X-Content-Type-Options: nosniff`が設定されているか。`Content-Security-Policy`が設定されているか（推奨）。これらをNablarchハンドラまたは`web.xml`のfilterで一括設定しているか |

**NG例**
```java
// セキュリティヘッダなし → クリックジャッキング可能
return new HttpResponse("/WEB-INF/view/top.jsp");
```

**OK例**
```java
// ResponseHeader ハンドラまたはフィルタで一括設定
response.setHeader("X-Frame-Options", "DENY");
response.setHeader("X-Content-Type-Options", "nosniff");
```

---

### WEB-016
**ログアウト時のセッション完全破棄とリダイレクト確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのウェブ機能を使う（アクションクラスを実装する） |
| レビュー対象 | ログアウトアクションクラスの`SessionUtil.invalidate()`呼び出し、`Cache-Control`ヘッダ設定 |
| FW提供範囲 | セッションストアのクリア機構はFWが提供。実装者は`SessionUtil.invalidate()`の呼び出しタイミングとブラウザキャッシュ制御を担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/session_store.html) |
| 優先度理由 | セッションの不完全な破棄は、ログアウト後に他ユーザーが同じブラウザで保護リソースにアクセスできてしまうリスクがある |
| チェック方法 | ログアウトアクションで`SessionUtil.invalidate()`が呼ばれているか。ログアウト後に`Cache-Control`ヘッダでブラウザキャッシュを無効化しているか。ログアウト後のURL直接アクセスがログイン画面にリダイレクトされるか |

**NG例**
```java
public HttpResponse doLogout(HttpRequest req, ExecutionContext ctx) {
    SessionUtil.delete(ctx, "loginUser"); // ユーザー情報だけ削除（不完全）
    return new HttpResponse(302, "redirect:///login");
}
```

**OK例**
```java
public HttpResponse doLogout(HttpRequest req, ExecutionContext ctx) {
    SessionUtil.invalidate(ctx); // セッション全体を破棄
    return new HttpResponse(302, "redirect:///login");
}
```
