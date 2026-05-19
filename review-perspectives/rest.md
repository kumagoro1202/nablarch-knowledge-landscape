# Nablarch REST API レビュー観点プリセット
対象: Nablarch RESTful Web Services（nablarch-fw-jaxrs）
---
## REST-001: RoutesMapping設定とアクションクラスの実装確認
**観点タイトル**: RoutesMapping設定とアクションクラスの実装確認
**観点詳細**: NablarchのJakarta RESTful Web Servicesサポートでは、URLルーティングはRoutesMapping（routes.rb等のXML設定）で行う。実装者が使用できるアノテーションは@Produces/@Consumes/@Validの3種のみであり、@Pathアノテーションはルーティングには使用しない。アクションクラスにはHttpResponseを返す業務ロジックメソッドを実装し、RoutesMapping設定でURLとメソッドを紐付ける。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html
**優先度**: MUST
**優先度の理由**: RoutesMapping設定の誤りはAPI全体のルーティング失敗・404を引き起こす。@Pathアノテーションの誤使用はNablarchのアーキテクチャ原則に反し、意図通りに動作しない。
**責任区分**: developer
**FW提供範囲**: URLルーティング処理（RoutesMappingハンドラ）、リクエストJSONのパース（BodyConvertHandler）、Bean Validation実行（JaxRsBeanValidationHandler）、HTTPレスポンス変換（JaxRsResponseHandler）はFWが自動処理
**レビュー対象**: routes.rb等のRoutesMapping設定ファイル（URLとアクションクラス・メソッドの対応）、アクションクラスの@Produces/@Consumesアノテーション
**チェック方法**:
RoutesMapping設定（routes.rb等）にURLとアクションクラス・メソッドの対応が正しく定義されているか。アクションクラスに@Produces(MediaType.APPLICATION_JSON)または@Consumesが設定されているか（Nablarchがサポートするアノテーション）。@Pathアノテーションをルーティング目的で使用していないか（Nablarchでは非対応）。コンポーネント定義（XML）でRoutesMapping設定が正しく行われているか。
**NG例**:
```java
// @Path をルーティング目的で使用（Nablarchでは動作しない）
@Path("/api/users")
public class UserAction {
    @GET
    public List<User> getUsers() { return dao.findAll(User.class); }
}
```
**OK例**:
```java
// RoutesMapping（routes.rb等）でURLとアクションを紐付け
// routes.rb 例:
// get '/api/users', to: 'userAction#getUsers'

// アクションクラス側
@Produces(MediaType.APPLICATION_JSON)
public List<User> getUsers(HttpRequest req, ExecutionContext ctx) {
    return UniversalDao.findAll(User.class);
}
```
---
## REST-002: レスポンスオブジェクトの型とステータスコードの設計確認
**観点タイトル**: レスポンスオブジェクトの型とステータスコードの設計確認
**観点詳細**: REST API のレスポンスに HttpResponse または JAX-RS の Response を使って適切なステータスコード（200/201/400/404/500 等）を返しているか確認する。常に 200 を返す実装になっていないか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html
**優先度**: MUST
**優先度の理由**: 不正なステータスコードはクライアントのエラーハンドリングを破壊し、API コントラクトの信頼性を損なう。
**責任区分**: developer
**FW提供範囲**: HTTPレスポンスの送信処理はFW（JaxRsResponseHandler）が担当。実装者はステータスコードとレスポンスボディの内容を設定
**レビュー対象**: アクションクラスのHttpResponse/Response生成コード（ステータスコード設定箇所）
**チェック方法**:
作成成功時は 201、正常取得は 200、削除成功は 204 を返しているか。リソース未存在時は 404、バリデーションエラーは 400 を返しているか。例外発生時に 200 を返していないか（try-catch で握りつぶしていないか）。
**NG例**:
```java
@POST
@Path("/users")
public String createUser(UserForm form) {
    dao.insert(form);
    return "OK"; // ステータスコード指定なし → 常に 200
}
```
**OK例**:
```java
@POST
@Path("/users")
public Response createUser(UserForm form) {
    dao.insert(form);
    return Response.status(Response.Status.CREATED).build(); // 201
}
```
---
## REST-003: 例外 → HTTP レスポンスへのマッピング設計確認
**観点タイトル**: 例外 → HTTP レスポンスへのマッピング設計確認
**観点詳細**: 業務例外・システム例外が統一されたエラーレスポンス形式に変換されているか確認する。ExceptionMapper の実装または Nablarch の例外ハンドリングハンドラが設定されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html
**優先度**: MUST
**優先度の理由**: 例外マッピングが統一されていないと、エラー時のレスポンス形式がランダムになり、クライアントのエラーハンドリングが不可能になる。
**責任区分**: developer
**FW提供範囲**: ExceptionMapper自体の呼び出し機構はFWが提供。実装者はマッピング定義（業務例外→HTTPステータスコード・レスポンス形式）を実装
**レビュー対象**: ExceptionMapper実装クラス（業務例外→HTTPレスポンスのマッピング定義）
**チェック方法**:
ExceptionMapper<T> を実装した例外マッピングクラスが存在するか。業務例外 → 400、システム例外 → 500 のマッピングが一貫しているか。エラーレスポンスの JSON 形式が全 API で統一されているか。
**NG例**:
```java
@POST
public Response createUser(UserForm form) {
    try {
        service.create(form);
    } catch (BusinessException e) {
        // それぞれのメソッドで個別に例外処理 → 形式バラバラ
        return Response.status(400).entity("Error: " + e.getMessage()).build();
    }
}
```
**OK例**:
```java
// ExceptionMapper で一元管理
@Provider
public class BusinessExceptionMapper implements ExceptionMapper<BusinessException> {
    @Override
    public Response toResponse(BusinessException e) {
        ErrorResponse error = new ErrorResponse("VALIDATION_ERROR", e.getMessage());
        return Response.status(400).entity(error).build();
    }
}
```
---
## REST-004: API の入出力スキーマが明確に定義・文書化されているか
**観点タイトル**: API の入出力スキーマが明確に定義・文書化されているか
**観点詳細**: REST API のリクエスト/レスポンスの JSON スキーマが、OpenAPI 仕様または同等のドキュメントで定義されているか確認する。nablarch-openapi-generator を使った自動生成が活用されているか確認する。
**根拠URL**: https://github.com/nablarch/nablarch-openapi-generator
**優先度**: Should
**優先度の理由**: API スキーマ未定義はクライアント実装の誤りを招き、統合テスト段階で大量のバグが発覚する。
**責任区分**: developer
**FW提供範囲**: APIドキュメント自動生成はFWが標準提供しない。実装者がOpenAPI仕様に従ってドキュメントを作成・更新する
**レビュー対象**: openapi.yamlまたはswagger.json等のAPIスキーマ定義ファイル
**チェック方法**:
openapi.yaml または swagger.json が存在し、最新状態か。リクエスト・レスポンスのフィールド定義（型・必須/任意・サンプル値）が記述されているか。nablarch-openapi-generator からコード生成を行っている場合、仕様とコードが乖離していないか。
**NG例**:
```
# API ドキュメント: なし（実装コードだけ存在）
```
**OK例**:
```bash
# openapi.yaml
paths:
  /api/users:
    post:
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserRequest'
```
---
## REST-005: JWT / Bearer Token 認証と認可チェックの実装確認
**観点タイトル**: JWT / Bearer Token 認証と認可チェックの実装確認
**観点詳細**: REST API の認証が適切に実装されているか確認する。認証トークン（JWT 等）の検証、有効期限チェック、権限チェックが全保護エンドポイントに適用されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/permission_check.html
**優先度**: MUST
**優先度の理由**: REST API の認証欠如は全データの無制限アクセスを許す。特に /api/admin/* 等の管理 API での漏洩は致命的。
**責任区分**: developer
**FW提供範囲**: 認証・認可ハンドラの機構はFWが提供。実装者はトークン検証ロジックと権限定義を実装
**レビュー対象**: JWTトークン検証ロジック実装、PermissionUtil/@RolesAllowed等の権限チェック実装
**チェック方法**:
Authorization: Bearer <token> ヘッダの検証が実装されているか。JWT 署名検証・有効期限チェックが行われているか。保護リソースへの未認証アクセスが 401 を返すか（403 ではなく）。ロールによるアクセス制御が @RolesAllowed または同等で実装されているか。
**NG例**:
```java
@GET
@Path("/api/admin/users")
public List<User> getAdminUsers() {
    // 認証・認可チェックなし → 誰でもアクセス可能
    return dao.findAll(User.class);
}
```
**OK例**:
```java
@GET
@Path("/api/admin/users")
@RolesAllowed("ADMIN")
public List<User> getAdminUsers() {
    return dao.findAll(User.class);
}
```
---
## REST-006: エラーレスポンスの JSON 形式が全 API で統一されているか
**観点タイトル**: エラーレスポンスの JSON 形式が全 API で統一されているか
**観点詳細**: バリデーションエラー・認証エラー・システムエラーのレスポンス JSON 形式が全 API エンドポイントで統一されているか確認する。クライアントが一元的にエラー処理できるフォーマットか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html
**優先度**: MUST
**優先度の理由**: エラーフォーマットの不統一はクライアントのエラーハンドリングコードを複雑にし、フロントエンド・連携システムとの統合を困難にする。
**責任区分**: developer
**FW提供範囲**: JSONシリアライズはFW（BodyConvertHandler）が実行。実装者はエラーレスポンスの形式（フィールド構造・エラーコード体系）の統一を担当
**レビュー対象**: ExceptionMapperのエラーレスポンスDTO定義、全APIのエラーレスポンスフィールド確認
**チェック方法**:
エラーレスポンスに code（エラーコード）と message（説明）フィールドが存在するか。バリデーションエラーで複数フィールドのエラーをまとめて返せるか。HTTP ステータスコードとエラーコードが一貫した対応関係か。
**NG例**:
```java
// エンドポイントによってフォーマットが異なる
// /api/users POST エラー
{"error": "バリデーションエラー"}
// /api/orders POST エラー
{"status": "NG", "detail": [{"field": "amount", "msg": "必須"}]}
```
**OK例**:
```java
// 全 API 統一フォーマット
{
  "code": "VALIDATION_ERROR",
  "message": "入力値に誤りがあります",
  "details": [
    {"field": "name", "code": "REQUIRED", "message": "氏名は必須です"}
  ]
}
```
---
## REST-007: REST API リクエストボディのサーバーサイドバリデーション確認
**観点タイトル**: REST API リクエストボディのサーバーサイドバリデーション確認
**観点詳細**: JSON リクエストボディに対するバリデーションが Bean Validation (@NotNull, @Size 等) または Nablarch バリデーションで実装されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html
**優先度**: MUST
**優先度の理由**: バリデーションのない REST API は不正データの登録・処理エラーの原因となる。REST では JSP のような入力制御がないため特に重要。
**責任区分**: developer
**FW提供範囲**: @Validアノテーション検出とバリデーション実行はFW（JaxRsBeanValidationHandler）が担当。実装者はDTOクラスへのアノテーション定義を担当
**レビュー対象**: リクエストDTOクラスのBean Validationアノテーション定義（@NotNull・@Size等）
**チェック方法**:
リクエスト DTO クラスに Bean Validation アノテーションが定義されているか。@Valid アノテーションでバリデーションが実行されているか。バリデーションエラー時に 400 Bad Request と詳細メッセージが返されるか。
**NG例**:
```java
@POST
public Response createUser(UserRequest request) {
    // バリデーションなし → null や不正値がそのまま DB へ
    dao.insert(request);
    return Response.ok().build();
}
```
**OK例**:
```java
@POST
public Response createUser(@Valid UserRequest request) {
    dao.insert(request);
    return Response.status(201).build();
}

public class UserRequest {
    @NotNull @Size(max = 100)
    private String name;
}
```
---
## REST-008: リクエスト/レスポンスのメディアタイプ検証確認
**観点タイトル**: リクエスト/レスポンスのメディアタイプ検証確認
**観点詳細**: API が Content-Type: application/json 以外のリクエストを適切に拒否（415 Unsupported Media Type）しているか確認する。Accept ヘッダが考慮されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html
**優先度**: Should
**優先度の理由**: Content-Type 検証の欠如はメディアタイプインジェクション・予期しない解析エラーの原因となる。
**責任区分**: developer
**FW提供範囲**: Content-Type/Acceptヘッダのマッチング処理はFWが担当。実装者は@Produces/@Consumesで対応メディアタイプを宣言
**レビュー対象**: アクションクラスの@Producesと@Consumesアノテーション（Nablarchがサポートするアノテーション）
**チェック方法**:
@Consumes(MediaType.APPLICATION_JSON) が POST/PUT メソッドに付与されているか。@Produces(MediaType.APPLICATION_JSON) が応答メソッドに付与されているか。JSON 以外の Content-Type で 415 エラーが返されるか。
**NG例**:
```java
@POST
// @Consumes 未指定 → 任意の Content-Type を受け入れる
public Response createUser(String body) { ... }
```
**OK例**:
```java
@POST
@Consumes(MediaType.APPLICATION_JSON)
@Produces(MediaType.APPLICATION_JSON)
public Response createUser(UserRequest request) { ... }
```
---
## REST-009: コレクションリソースの件数制限とページング設計確認
**観点タイトル**: コレクションリソースの件数制限とページング設計確認
**観点詳細**: コレクションを返す GET エンドポイントで全件返却していないか確認する。page / limit クエリパラメータによるページング、または件数上限が設定されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/universal_dao.html
**優先度**: Should
**優先度の理由**: 件数無制限の全件返却は大量データ時のタイムアウト・メモリ枯渇・クライアント障害の原因となる。
**責任区分**: developer
**FW提供範囲**: ページングクエリの生成はFWが提供。実装者はページング条件の受け取り・設定とデフォルト上限値の実装を担当
**レビュー対象**: コレクション系アクションのページング条件実装コード（pageパラメータ・limitパラメータ受け取り）
**チェック方法**:
一覧系 API に limit パラメータ（またはデフォルト上限）が設定されているか。ページング情報（totalCount, page, pageSize）がレスポンスに含まれているか。最大返却件数の上限値（例: 1000件）が設定されているか。
**NG例**:
```java
@GET
@Path("/api/users")
public List<User> getUsers() {
    return UniversalDao.findAll(User.class); // 全件返却
}
```
**OK例**:
```java
@GET
@Path("/api/users")
public PagedResponse<User> getUsers(
        @QueryParam("page") @DefaultValue("1") int page,
        @QueryParam("limit") @DefaultValue("20") int limit) {
    Pagination pagination = new Pagination(page, Math.min(limit, 100));
    List<User> users = UniversalDao.findAllBySqlFile(User.class, "FIND_ALL", pagination);
    return new PagedResponse<>(users, pagination);
}
```
---
## REST-010: PUT / DELETE の冪等性保証確認
**観点タイトル**: PUT / DELETE の冪等性保証確認
**観点詳細**: PUT・DELETE エンドポイントが冪等（同じリクエストを複数回実行しても同じ結果になる）か確認する。特に DELETE で存在しないリソースへのリクエストが適切なステータスコードを返すか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html
**優先度**: Should
**優先度の理由**: 冪等でない PUT/DELETE はリトライ時に重複処理・データ破壊を引き起こす。REST の設計原則として冪等性は重要。
**責任区分**: developer
**FW提供範囲**: HTTPメソッドのルーティングはFWが担当。冪等性の実現（同じリクエストを複数回実行した場合の結果の一致）は実装者の設計が必要
**レビュー対象**: PUT/DELETEアクションクラスの実装ロジック（存在チェック・冪等性確保）
**チェック方法**:
DELETE で既に削除済みのリソースへのリクエストが 404 または 204 を返すか。PUT が存在しないリソースに対して 404 を返し、勝手に作成しないか。ネットワーク障害時のリトライを考慮した設計か。
**NG例**:
```java
@DELETE
@Path("/api/users/{id}")
public Response deleteUser(@PathParam("id") Long id) {
    dao.deleteById(id); // 存在しない場合に例外で 500 → 冪等でない
    return Response.ok().build();
}
```
**OK例**:
```java
@DELETE
@Path("/api/users/{id}")
public Response deleteUser(@PathParam("id") Long id) {
    int deleted = dao.deleteById(id);
    if (deleted == 0) return Response.status(404).build();
    return Response.noContent().build(); // 204
}
```
---
## REST-011: Cross-Origin Resource Sharing (CORS) の適切な設定確認
**観点タイトル**: Cross-Origin Resource Sharing (CORS) の適切な設定確認
**観点詳細**: ブラウザからの AJAX リクエストを受け付ける REST API で、CORS 設定（Access-Control-Allow-Origin 等）が適切か確認する。ワイルドカード * による全オリジン許可になっていないか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html
**優先度**: Should
**優先度の理由**: CORS の過剰な許可（*）は悪意あるサイトからの API 呼び出しを可能にし、CSRF と組み合わせた攻撃のリスクが高まる。
**責任区分**: developer
**FW提供範囲**: CORS自動設定機能はFWが標準提供しない。実装者がフィルタかアクション内でAccess-Control-Allow-Origin等を設定
**レビュー対象**: CORSヘッダ設定コード（フィルタまたはアクション内のAccess-Control-*ヘッダ設定）
**チェック方法**:
Access-Control-Allow-Origin がホワイトリスト形式で設定されているか（* でないか）。Access-Control-Allow-Methods が最小限の HTTP メソッドに限定されているか。認証トークンを含む場合、Access-Control-Allow-Credentials: true と特定オリジンの組み合わせか。
**NG例**:
```
response.addHeader("Access-Control-Allow-Origin", "*"); // 全オリジン許可
response.addHeader("Access-Control-Allow-Credentials", "true"); // 組み合わせ危険
```
**OK例**:
```
String origin = request.getHeader("Origin");
if (ALLOWED_ORIGINS.contains(origin)) {
    response.addHeader("Access-Control-Allow-Origin", origin);
}
```
---
## REST-012: API リクエスト/レスポンスの適切なログ記録確認
**観点タイトル**: API リクエスト/レスポンスの適切なログ記録確認
**観点詳細**: REST API のリクエスト（メソッド・パス・パラメータ）とレスポンス（ステータスコード・処理時間）がログに記録されているか確認する。機密情報がログに含まれていないか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/log.html
**優先度**: Should
**優先度の理由**: ログなしでは本番障害の原因究明が不可能になる。一方、パスワードや個人情報のログ記録はセキュリティインシデントの原因となる。
**責任区分**: developer
**FW提供範囲**: フレームワークレベルのアクセスログはFWが提供。業務的なリクエスト内容のログ出力（マスキング含む）は実装者が担当
**レビュー対象**: アクセスログ設定、アクションクラスのログ出力コード（機密情報マスキング含む）
**チェック方法**:
アクセスログ（メソッド・URL・ステータス・処理時間）が INFO レベルで出力されているか。リクエストボディにパスワード・カード番号等が含まれる場合、マスキングが実装されているか。エラー発生時にはスタックトレースが ERROR レベルでログ出力されるか。
**NG例**:
```
log.info("request: " + request.getBody()); // パスワード平文ログ
```
**OK例**:
```
log.info("API: {} {} status={} time={}ms",
    request.getMethod(), request.getPath(),
    response.getStatus(), elapsedMs);
// パスワードフィールドはマスク済みオブジェクトをログ出力
```
---
## REST-013: API エンドポイントへのリクエスト頻度制限の確認
**観点タイトル**: API エンドポイントへのリクエスト頻度制限の確認
**観点詳細**: 特定クライアントからの過剰なリクエストを制限する仕組みが設計されているか確認する。外部向け API では特に重要。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html
**優先度**: May
**優先度の理由**: レート制限は外部公開 API では必須に近いが、内部 API では不要な場合もある。ただし、設計段階での検討は推奨される。
**責任区分**: developer
**FW提供範囲**: レート制限機能はFW本体が標準提供しない。APIゲートウェイまたはカスタムハンドラで実装者が実装
**レビュー対象**: APIゲートウェイのレート制限設定（nginx/AWS API Gateway等）またはカスタムハンドラ実装
**チェック方法**:
API ゲートウェイ（nginx / AWS API Gateway 等）でのレート制限が設定されているか。アプリケーションレベルでのリクエスト制限ハンドラが実装されているか。429 Too Many Requests のレスポンスが定義されているか。
**NG例**:
```
# レート制限なし → DDoS で即時ダウン
```
**OK例**:
```nginx
# nginx でのレート制限
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req zone=api burst=20 nodelay;
```
---
## REST-014: REST API の URL 設計が RESTful 規約に従っているか確認
**観点タイトル**: REST API の URL 設計が RESTful 規約に従っているか確認
**観点詳細**: URL が名詞複数形でリソースを表現し、HTTP メソッドで操作を表現する設計になっているか確認する。動詞を URL に含める設計（RPC スタイル）になっていないか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html
**優先度**: Should
**優先度の理由**: RESTful でない URL 設計はクライアント実装者の混乱を招き、API の一貫性を損なう。
**責任区分**: developer
**FW提供範囲**: URLルーティング解決はFWが担当。URL設計の妥当性（RESTful規約への準拠）の確認は実装者が担当
**レビュー対象**: RoutesMapping定義のURL設計（routes.rb等）、アクションクラス名
**チェック方法**:
リソース名が名詞複数形か（/users, /orders, /products）。動詞が URL に含まれていないか（/getUser, /createOrder は NG）。リソースの階層関係が URL で表現されているか（/users/{id}/orders）。
**NG例**:
```
GET  /getUsers         # 動詞が URL に入っている
POST /createUser       # 動詞が URL に入っている
POST /user/delete/{id} # HTTP メソッドと URL が不一致
```
**OK例**:
```
GET    /api/users          # ユーザー一覧取得
POST   /api/users          # ユーザー作成
GET    /api/users/{id}     # ユーザー取得
PUT    /api/users/{id}     # ユーザー更新
DELETE /api/users/{id}     # ユーザー削除
```
---
## REST-015: API のバージョン管理方式が一貫しているか確認
**観点タイトル**: API のバージョン管理方式が一貫しているか確認
**観点詳細**: REST API のバージョン管理が URL パス（/v1/, /v2/）またはヘッダで統一された方式で実装されているか確認する。バージョン管理なしの変更が後方互換性を破壊していないか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html
**優先度**: Should
**優先度の理由**: バージョン管理のない API 変更は既存クライアントを突然壊す。複数クライアント（web/mobile/外部）が存在する場合に特に重要。
**責任区分**: developer
**FW提供範囲**: バージョン管理機能はFWが標準提供しない。実装者がURLまたはヘッダでバージョン管理を設計・実装
**レビュー対象**: RoutesMapping定義のURLバージョン設定（/v1/等）、旧バージョンAPIの維持状況
**チェック方法**:
URL に /v1/ 等のバージョン番号が含まれているか。バージョンアップ時に旧バージョンが一定期間維持されるか。API 廃止予定が Deprecation ヘッダまたはドキュメントで通知されているか。
**NG例**:
```
# バージョン管理なし → 変更が即時に全クライアントに影響
GET /api/users
```
**OK例**:
```bash
# バージョン付き URL
GET /api/v1/users   # 旧クライアント継続動作
GET /api/v2/users   # 新フォーマット
```
---
## REST-016: REST API エンドポイントの自動テスト設計確認
**観点タイトル**: REST API エンドポイントの自動テスト設計確認
**観点詳細**: REST API のエンドポイントに対して、Nablarch TestSupport や RestAssured 等を使ったテストが実装されているか確認する。正常系・バリデーションエラー・認証エラーのテストが存在するか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/index.html
**優先度**: Should
**優先度の理由**: テストのない REST API はリファクタリング・変更時のデグレを検知できず、本番障害のリスクが高まる。
**責任区分**: developer
**FW提供範囲**: テストサポートクラス（RestTestSupport等）はFWが提供。テストケースの実装と正常系・バリデーションエラー・認証エラーのカバレッジは実装者が担当
**レビュー対象**: テストクラス（RestTestSupport等を継承したクラス）
**チェック方法**:
RestTestSupport または JUnit + REST クライアントでのテストが存在するか。正常レスポンスのステータスコード・ボディ構造がテストで検証されているか。認証エラー（401）・バリデーションエラー（400）のテストケースがあるか。
**NG例**:
```java
// リソースクラスのテストなし
```
**OK例**:
```java
public class UserResourceTest extends RestTestSupport {
    @Test
    public void ユーザー作成_正常() {
        RestMockHttpRequest request = post("/api/v1/users")
            .setBody(Map.of("name", "テスト太郎"));
        HttpResponse response = sendRequest(request);
        assertStatusCode(201, response);
    }
}
```
