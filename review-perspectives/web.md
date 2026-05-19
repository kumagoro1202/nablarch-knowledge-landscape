# Nablarch Web アプリケーション レビュー観点プリセット
対象: Nablarch Webアーキテクチャ（nablarch-fw-web）
---
## WEB-001: ハンドラキューの順序・必須ハンドラの確認
**観点タイトル**: ハンドラキューの順序・必須ハンドラの確認
**観点詳細**: Nablarch Web アプリケーションのハンドラキュー（web-component-configuration.xml 等）に、必須ハンドラが適切な順序で登録されているか確認する。ハンドラの順序が誤ると認証バイパスや例外ハンドリング漏れが発生する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html
**優先度**: MUST
**優先度の理由**: ハンドラキューはリクエスト処理の根幹であり、順序誤りは認証・認可の無効化やCSRF対策の欠落につながる。
**責任区分**: developer
**FW提供範囲**: 各ハンドラの実際の処理ロジックはFWが提供。実装者はハンドラの種類と順序の設定を担当
**レビュー対象**: web-component-configuration.xml等のhandlerQueue設定
**チェック方法**:
HttpCharacterEncodingHandler が最前列に配置されているか。HttpResponseHandler が後段に配置されているか。認証ハンドラが業務ハンドラより前に配置されているか。SessionConcurrentAccessHandler（セッション排他）が必要な場合に設定されているか。
**NG例**:
```xml
<!-- 認証ハンドラが業務アクションより後 → 認証バイパス可能 -->
<list name="handlerQueue">
    <component-ref name="businessActionDispatcher"/>
    <component-ref name="authenticationHandler"/> <!-- 順序誤り -->
</list>
```
**OK例**:
```xml
<list name="handlerQueue">
    <component-ref name="httpCharacterEncodingHandler"/>
    <component-ref name="authenticationHandler"/>
    <component-ref name="csrfTokenHandler"/>
    <component-ref name="businessActionDispatcher"/>
</list>
```
---
## WEB-002: セッションスコープの適切な使用とタイムアウト設定
**観点タイトル**: セッションスコープの適切な使用とタイムアウト設定
**観点詳細**: SessionUtil を使ったセッション操作が適切か確認する。セッションへの不要な大容量データの格納、タイムアウト未設定、セッション固定攻撃への対策が実装されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/session_store.html
**優先度**: MUST
**優先度の理由**: セッション管理の不備はセッションハイジャック・固定攻撃・メモリ圧迫の原因となる。
**責任区分**: developer
**FW提供範囲**: セッションストアへの実際の読み書き（往路・復路）はFW（セッション変数保存ハンドラ）が自動実行。実装者はSessionUtil APIを正しく使うことを担当
**レビュー対象**: アクションクラスのSessionUtil.put()/get()/invalidate()呼び出しコード
**チェック方法**:
SessionUtil.put() で格納するオブジェクトが最小限か（大容量オブジェクト不可）。ログイン成功後に SessionUtil.invalidate() → 新セッション発行しているか（セッション固定対策）。セッションタイムアウト値が web.xml または設定ファイルで明示されているか。DB セッションストア使用時、セッションデータのスキーマが正しく定義されているか。
**NG例**:
```java
// ログイン成功後にセッション固定攻撃が可能
SessionUtil.put(context, "loginUser", user); // セッション無効化なし
```
**OK例**:
```
SessionUtil.invalidate(context); // 旧セッション破棄
SessionUtil.put(context, "loginUser", user); // 新セッションに格納
```
---
## WEB-003: CSRF トークンの生成・検証が全更新系リクエストに適用されているか
**観点タイトル**: CSRF トークンの生成・検証が全更新系リクエストに適用されているか
**観点詳細**: POST/PUT/DELETE 等の更新系リクエストに対して CSRF トークンの検証が実装されているか。カスタムハンドラ等でバイパスされていないか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html#csrf
**優先度**: MUST
**優先度の理由**: CSRF 対策の欠落は、悪意あるサイトからの不正リクエストによるデータ改ざんを許す。金融・行政系システムでは致命的な脆弱性となる。
**責任区分**: developer
**FW提供範囲**: CSRFトークンの生成・検証ロジック自体はFW（CsrfTokenHandler）が実行。実装者はハンドラ設定とJSPへの<n:csrf>タグ配置を担当
**レビュー対象**: web-component-configuration.xmlのCsrfTokenHandler設定、JSPの<n:csrf>タグ配置
**チェック方法**:
CsrfTokenHandler がハンドラキューに登録されているか。JSP の更新フォームに <n:csrf> タグまたは相当のトークン hidden フィールドが含まれているか。AJAX リクエストで CSRF トークンがヘッダまたはボディに付与されているか。GET リクエスト専用の除外設定が適切か（CSRFは GET では不要）。
**NG例**:
```xml
<!-- CSRF トークン未設定 -->
<form action="/update" method="POST">
    <input type="submit" value="更新"/>
</form>
```
**OK例**:
```xml
<form action="/update" method="POST">
    <n:csrf/>  <%-- CSRF トークン自動付与 --%>
    <input type="submit" value="更新"/>
</form>
```
---
## WEB-004: フォーム入力値のサーバーサイドバリデーション確認
**観点タイトル**: フォーム入力値のサーバーサイドバリデーション確認
**観点詳細**: クライアントサイドバリデーションのみに依存せず、サーバーサイドで ValidationUtil.validate() または Bean Validation が実装されているか確認する。バリデーションエラー時のユーザー向けメッセージが適切か確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/validation.html
**優先度**: MUST
**優先度の理由**: クライアントサイドのみのバリデーションは Burp Suite 等のツールで容易にバイパス可能。不正入力のDB登録を防ぐためサーバーサイド検証は必須。
**責任区分**: developer
**FW提供範囲**: バリデーション実行エンジン（アノテーション評価）はFWが提供。実装者はバリデーションアノテーション定義とvalidate()呼び出しを担当
**レビュー対象**: アクションクラスのValidationUtil.validate()呼び出し、FormクラスのValidationアノテーション定義
**チェック方法**:
アクションクラスで ValidationUtil.validate() が呼ばれているか。バリデーションアノテーション（@Required, @Length, @Domain 等）が Form クラスに定義されているか。バリデーションエラー時に入力画面に戻り、エラーメッセージが表示されるか。
**NG例**:
```java
// バリデーションなしで直接 DB 操作
public HttpResponse doPost(HttpRequest req, ExecutionContext ctx) {
    UserForm form = SessionUtil.get(ctx, "form");
    dao.insert(form); // バリデーションなし
}
```
**OK例**:
```java
public HttpResponse doPost(HttpRequest req, ExecutionContext ctx) {
    UserForm form = SessionUtil.get(ctx, "form");
    ValidationUtil.validate(form);
    dao.insert(form);
}
```
---
## WEB-005: 正常系・異常系の全遷移先が定義されているか
**観点タイトル**: 正常系・異常系の全遷移先が定義されているか
**観点詳細**: 業務フローの正常遷移だけでなく、バリデーションエラー・システムエラー・認証エラー・404 等の異常系画面が全て定義され、適切なユーザーメッセージが表示されるか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html
**優先度**: Should
**優先度の理由**: エラー画面未定義は内部エラー情報のユーザー露出（スタックトレース表示）につながり、セキュリティリスクとなる。
**責任区分**: developer
**FW提供範囲**: 例外捕捉とエラーページ遷移の基盤機構はFWが提供。実装者はエラーページのURL設定と全エラーコードのカバレッジを担当
**レビュー対象**: web.xmlのerror-page設定、web-component-configuration.xmlのErrorHandler設定
**チェック方法**:
web-component-configuration.xml に ErrorHandler でエラーページが設定されているか。HTTP 404/500 の各エラーページが web.xml に定義されているか。スタックトレースが画面に表示されないか（本番設定確認）。
**NG例**:
```xml
<!-- エラーページ未設定 → デフォルトのスタックトレースが表示される -->
```
**OK例**:
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
## WEB-006: アクションクラスと業務ロジッククラスの責務分離確認
**観点タイトル**: アクションクラスと業務ロジッククラスの責務分離確認
**観点詳細**: アクションクラスが HTTP リクエスト/レスポンス処理のみを担当し、業務ロジックが別クラス（サービス層・ドメイン層）に委譲されているか確認する。アクションの肥大化（Fat Action）を検出する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/architecture.html
**優先度**: Should
**優先度の理由**: アクションに業務ロジックが集中すると、テスト困難・変更影響範囲の拡大・コードの可読性低下が発生する。
**責任区分**: developer
**FW提供範囲**: HTTPリクエスト/レスポンスの変換はFWが担当。業務ロジックの責務配置設計は実装者が担当
**レビュー対象**: アクションクラス（業務ロジック混入確認）、サービスクラス・ドメインクラスの分離状況
**チェック方法**:
アクションクラスが HttpRequest / ExecutionContext / HttpResponse の処理のみを行っているか。SQL や業務計算ロジックがアクションクラスに直接書かれていないか。業務ロジッククラスが SystemRepository.get() で取得されているか。
**NG例**:
```java
public HttpResponse doPost(HttpRequest req, ExecutionContext ctx) {
    // アクション内に直接 SQL
    SqlResultSet result = DbConnectionContext.getConnection()
        .query("SELECT * FROM ...");
    // 業務計算もアクション内
    int total = calcTotal(result);
}
```
**OK例**:
```java
public HttpResponse doPost(HttpRequest req, ExecutionContext ctx) {
    OrderService service = SystemRepository.get("orderService");
    OrderResult result = service.processOrder(form);
    SessionUtil.put(ctx, "result", result);
    return new HttpResponse("/WEB-INF/view/complete.jsp");
}
```
---
## WEB-007: JSP での動的データ出力時の XSS エスケープ確認
**観点タイトル**: JSP での動的データ出力時の XSS エスケープ確認
**観点詳細**: JSP でユーザー入力・DB データを HTML 出力する際に、Nablarch のカスタムタグ（<n:write> 等）または JSTL の <c:out> でエスケープされているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/tag.html
**優先度**: MUST
**優先度の理由**: XSS は OWASP Top 10 の常連脆弱性。ユーザー入力の未エスケープ出力はスクリプトインジェクションによるアカウント乗っ取りにつながる。
**責任区分**: developer
**FW提供範囲**: <n:write>タグ内部のHTMLエスケープ処理はFWが実行。実装者は適切なタグ（<n:write>または<c:out>）を選択することを担当
**レビュー対象**: JSPファイルの動的値出力箇所（<n:write>タグ使用状況、EL式の直接使用確認）
**チェック方法**:
JSP での動的値出力に <n:write> または <c:out> を使用しているか。EL 式 ${...} を直接 HTML 属性・本文に使用していないか（EL 式はエスケープされない場合がある）。<script> タグ内での動的値出力には JavaScript エスケープが適用されているか。
**NG例**:
```xml
<!-- 未エスケープ → XSS 脆弱性 -->
<%= request.getParameter("name") %>
<!-- または -->
${user.comment}
```
**OK例**:
```xml
<n:write name="user.comment"/>
<%-- または --%>
<c:out value="${user.comment}"/>
```
---
## WEB-008: SQL 構築に動的文字列連結を使用していないか
**観点タイトル**: SQL 構築に動的文字列連結を使用していないか
**観点詳細**: SQL 文を文字列連結で組み立てていないか確認する。Nablarch の UniversalDao や SQL ファイル（.sql）のバインドパラメータを使用しているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/database.html
**優先度**: MUST
**優先度の理由**: SQL インジェクションは全データ漏洩・削除・認証バイパスを引き起こす最も危険な脆弱性の一つ。
**責任区分**: developer
**FW提供範囲**: バインドパラメータによる安全なSQL実行はFWが提供。実装者はSQL文字列連結を使わないことを担当
**レビュー対象**: SQLファイル（.sql）のバインドパラメータ定義、UniversalDao呼び出し箇所
**チェック方法**:
SQL ファイル内でユーザー入力を直接文字列連結していないか。UniversalDao.findAllBySqlFile() のパラメータがバインド変数経由か。動的条件が $if 等の Nablarch テンプレート機能で組み立てられているか。
**NG例**:
```java
// SQL インジェクション脆弱性
String sql = "SELECT * FROM users WHERE name = '" + name + "'";
```
**OK例**:
```java
// バインドパラメータで安全
UserCondition cond = new UserCondition();
cond.setName(name);
List<User> users = UniversalDao.findAllBySqlFile(User.class, "FIND_BY_NAME", cond);
```
---
## WEB-009: 認証済みユーザーのみが保護リソースにアクセスできるか
**観点タイトル**: 認証済みユーザーのみが保護リソースにアクセスできるか
**観点詳細**: 認証が必要なページ・アクションに対して、未認証アクセスが適切にリダイレクトされるか確認する。認可（権限チェック）が必要な機能に対してロールチェックが実装されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/permission_check.html
**優先度**: MUST
**優先度の理由**: 認証・認可の欠落は不正アクセスによる情報漏洩・操作の直接的な原因となる。
**責任区分**: developer
**FW提供範囲**: 認証チェックのハンドラ機構はFWが提供。実装者はハンドラ設定と権限チェック（PermissionUtil.permit()等）の呼び出しを担当
**レビュー対象**: web-component-configuration.xmlのAuthenticationHandler設定、アクションクラスのPermissionUtil呼び出し
**チェック方法**:
AuthenticationHandler がハンドラキューに登録され、未認証時に login 画面へリダイレクトされるか。権限が必要な画面・API では PermissionUtil.permit() または相当の権限チェックが実装されているか。テスト：ログアウト後に保護 URL へ直接アクセスした場合の動作確認。
**NG例**:
```java
// 権限チェックなしで管理画面にアクセス可能
public HttpResponse doGet(HttpRequest req, ExecutionContext ctx) {
    return new HttpResponse("/WEB-INF/view/admin.jsp"); // 誰でもアクセス可
}
```
**OK例**:
```java
public HttpResponse doGet(HttpRequest req, ExecutionContext ctx) {
    PermissionUtil.permit(ctx, "ADMIN_SCREEN");
    return new HttpResponse("/WEB-INF/view/admin.jsp");
}
```
---
## WEB-010: フォーム二重送信防止トークンの確認
**観点タイトル**: フォーム二重送信防止トークンの確認
**観点詳細**: 更新フォームの二重サブミット（ダブルクリック・ブラウザ戻るボタン）を防ぐトークン機構が実装されているか確認する。Nablarch の DoubleSubmissionHandler 等の使用を確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html
**優先度**: Should
**優先度の理由**: 二重サブミットはデータの二重登録・注文の重複等の業務障害につながる。特に EC・申請系システムでは重要。
**責任区分**: developer
**FW提供範囲**: 二重サブミットトークンの生成・検証はFW（DoubleSubmissionHandler）が実行。実装者はハンドラ設定とJSPへの<n:token>タグ配置を担当
**レビュー対象**: web-component-configuration.xmlのDoubleSubmissionHandler設定、JSPの<n:token>タグ配置
**チェック方法**:
DoubleSubmissionHandler がハンドラキューに登録されているか。フォームに二重サブミット防止トークンが含まれているか（<n:token> タグ等）。JavaScript による送信ボタン二重クリック防止も実装されているか。
**NG例**:
```xml
<!-- 二重サブミット対策なし -->
<form action="/order/register" method="POST">
    <input type="submit" value="注文確定"/>
</form>
```
**OK例**:
```xml
<form action="/order/register" method="POST">
    <n:token/>  <%-- 二重サブミット防止トークン --%>
    <input type="submit" value="注文確定"/>
</form>
```
---
## WEB-011: ファイルアップロードのサイズ制限・拡張子検証確認
**観点タイトル**: ファイルアップロードのサイズ制限・拡張子検証確認
**観点詳細**: ファイルアップロード機能でサイズ上限・許可拡張子の検証が実装されているか確認する。悪意あるファイル（スクリプト等）のアップロードを防止する設計か確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html
**優先度**: MUST
**優先度の理由**: 制限なしのファイルアップロードはサーバーストレージの枯渇・悪意あるスクリプト実行・DoS 攻撃のベクタとなる。
**責任区分**: developer
**FW提供範囲**: マルチパートのパース処理はFW（MultipartHandler）が担当。実装者はサイズ上限設定と拡張子検証ロジックの実装を担当
**レビュー対象**: MultipartHandlerのサイズ上限設定、アクションクラスの拡張子・MIMEタイプ検証コード
**チェック方法**:
MultipartHandler のサイズ上限設定（maxFileSize, maxRequestSize）が適切か。アップロードファイルの拡張子・MIMEタイプのホワイトリスト検証が実装されているか。アップロードファイルの保存先がドキュメントルート外か（Web からの直接アクセス不可）。
**NG例**:
```java
// 拡張子チェックなし
MultipartFile file = req.getFile("uploadFile");
file.moveTo("/var/www/html/uploads/" + file.getOriginalFilename());
```
**OK例**:
```
MultipartFile file = req.getFile("uploadFile");
String ext = FilenameUtils.getExtension(file.getOriginalFilename());
if (!ALLOWED_EXTENSIONS.contains(ext)) throw new ValidationException("不正なファイル形式");
file.moveTo("/var/data/uploads/" + UUID.randomUUID() + "." + ext);
```
---
## WEB-012: 画面文言のハードコード排除とメッセージプロパティ管理
**観点タイトル**: 画面文言のハードコード排除とメッセージプロパティ管理
**観点詳細**: 画面表示文言・エラーメッセージが JSP にハードコードされておらず、メッセージプロパティファイルで管理されているか確認する。多言語対応や一括変更が可能な設計か確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/message.html
**優先度**: Should
**優先度の理由**: メッセージのハードコードは多言語対応を困難にし、文言変更時の修正漏れを招く。
**責任区分**: developer
**FW提供範囲**: メッセージIDから文言への解決はFW（MessageUtil）が実行。実装者はメッセージ定義ファイルの作成と<n:message>タグの使用を担当
**レビュー対象**: message.properties等のメッセージ定義ファイル、JSPの<n:message>タグ使用状況
**チェック方法**:
エラーメッセージが message.properties または同等ファイルで管理されているか。JSP で <n:message> タグまたは MessageUtil.createMessage() が使われているか。日本語文言が JSP に直書きされていないか。
**NG例**:
```xml
<!-- メッセージハードコード -->
<span class="error">入力値が不正です。必須項目を確認してください。</span>
```
**OK例**:
```xml
<n:message messageId="errors.required" arg0="氏名"/>
```
---
## WEB-013: 大量データ表示時のページング・データ制限の確認
**観点タイトル**: 大量データ表示時のページング・データ制限の確認
**観点詳細**: 検索結果を全件取得・表示しておらず、適切なページング（limit/offset 等）で件数を制限しているか確認する。Nablarch の UniversalDao のページング機能が使われているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/universal_dao.html
**優先度**: Should
**優先度の理由**: 全件取得はメモリ枯渇・レスポンス時間劣化・画面表示不能の原因となる。数万件のデータ返却でブラウザがフリーズすることも。
**責任区分**: developer
**FW提供範囲**: ページングクエリの生成・実行はFWが提供。実装者はページング条件（件数・ページ番号）の受け取りと設定を担当
**レビュー対象**: UniversalDao呼び出しのページング条件設定コード（pageNumber・max等）
**チェック方法**:
UniversalDao.findAllBySqlFile() にページング条件（pageNumber, max）が設定されているか。件数上限（例: 1000件）が設定されているか。検索条件なし検索が実行できない制約があるか。
**NG例**:
```java
// 全件取得（件数制限なし）
List<User> users = UniversalDao.findAll(User.class);
```
**OK例**:
```
Pagination pagination = new Pagination(pageNo, 20); // 20件/ページ
List<User> users = UniversalDao.findAllBySqlFile(
    User.class, "FIND_USERS", condition, pagination);
```
---
## WEB-014: Nablarch TestSupport を使ったウェブアクション単体テスト設計
**観点タイトル**: Nablarch TestSupport を使ったウェブアクション単体テスト設計
**観点詳細**: ウェブアクションのテストが HttpRequestTestSupport を使って実装されているか確認する。リクエスト送信・レスポンス検証・セッション状態の確認がテストで行われているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/index.html
**優先度**: Should
**優先度の理由**: テストのないウェブアクションはリグレッションに気づかず、セキュリティ修正の影響範囲把握も困難になる。
**責任区分**: developer
**FW提供範囲**: テストサポートクラス（HttpRequestTestSupport等）はFWが提供。テストケースの実装と正常系・認証エラー系のカバレッジは実装者が担当
**レビュー対象**: テストクラス（HttpRequestTestSupportを継承したクラス）
**チェック方法**:
HttpRequestTestSupport を継承したテストクラスが存在するか。テストデータが Excel または YAML で管理されているか。正常遷移・バリデーションエラー・認証エラーのテストケースが存在するか。
**NG例**:
```java
// アクションクラスのテストなし
```
**OK例**:
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
## WEB-015: X-Content-Type-Options・X-Frame-Options 等のセキュリティヘッダ設定
**観点タイトル**: X-Content-Type-Options・X-Frame-Options 等のセキュリティヘッダ設定
**観点詳細**: レスポンスに適切なセキュリティ HTTP ヘッダが設定されているか確認する。クリックジャッキング・MIME スニッフィングへの対策が実装されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html#csrf
**優先度**: Should
**優先度の理由**: セキュリティヘッダの欠如はクリックジャッキング・MIME スニッフィング等のブラウザレベルの攻撃に対して無防備になる。
**責任区分**: developer
**FW提供範囲**: セキュリティヘッダを自動設定するFW機能は標準では提供されない。実装者がフィルタまたはアクション内でX-Frame-Options等を設定する必要がある
**レビュー対象**: web.xmlのフィルタ設定、またはアクションクラスのresponse.setHeader()呼び出し
**チェック方法**:
X-Frame-Options: DENY または SAMEORIGIN が設定されているか。X-Content-Type-Options: nosniff が設定されているか。Content-Security-Policy が設定されているか（推奨）。これらを Nablarch ハンドラまたは web.xml の filter で一括設定しているか。
**NG例**:
```java
// セキュリティヘッダなし → クリックジャッキング可能
return new HttpResponse("/WEB-INF/view/top.jsp");
```
**OK例**:
```java
// ResponseHeader ハンドラまたはフィルタで一括設定
response.setHeader("X-Frame-Options", "DENY");
response.setHeader("X-Content-Type-Options", "nosniff");
```
---
## WEB-016: ログアウト時のセッション完全破棄とリダイレクト確認
**観点タイトル**: ログアウト時のセッション完全破棄とリダイレクト確認
**観点詳細**: ログアウト処理でセッションが完全に破棄され（SessionUtil.invalidate()）、ログアウト後にブラウザの戻るボタンで保護リソースにアクセスできないか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/session_store.html
**優先度**: MUST
**優先度の理由**: セッションの不完全な破棄は、ログアウト後に他ユーザーが同じブラウザで保護リソースにアクセスできてしまうリスクがある。
**責任区分**: developer
**FW提供範囲**: セッションストアのクリア機構はFWが提供。実装者はSessionUtil.invalidate()の呼び出しタイミングとブラウザキャッシュ制御を担当
**レビュー対象**: ログアウトアクションクラスのSessionUtil.invalidate()呼び出し、Cache-Controlヘッダ設定
**チェック方法**:
ログアウトアクションで SessionUtil.invalidate() が呼ばれているか。ログアウト後に Cache-Control ヘッダでブラウザキャッシュを無効化しているか。ログアウト後の URL 直接アクセスがログイン画面にリダイレクトされるか。
**NG例**:
```java
public HttpResponse doLogout(HttpRequest req, ExecutionContext ctx) {
    SessionUtil.delete(ctx, "loginUser"); // ユーザー情報だけ削除（不完全）
    return new HttpResponse(302, "redirect:///login");
}
```
**OK例**:
```java
public HttpResponse doLogout(HttpRequest req, ExecutionContext ctx) {
    SessionUtil.invalidate(ctx); // セッション全体を破棄
    return new HttpResponse(302, "redirect:///login");
}
```
