# Nablarch REST API レビュー観点プリセット
対象: Nablarch RESTful Web Services（nablarch-fw-jaxrs）

## 観点一覧

| ID | タイトル | 優先度 | 観点概要 | 根拠 |
|----|---------|:------:|---------|------|
| [REST-001](#rest-001) | RoutesMapping設定とアクションクラスの実装確認 | **MUST** | URLルーティングがRoutesMapping設定で行われ、`@Path`アノテーションを誤使用していないか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html) |
| [REST-002](#rest-002) | レスポンスオブジェクトの型とステータスコードの設計確認 | **MUST** | 適切なステータスコード（200/201/400/404/500等）を返し、常に200を返す実装になっていないか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html) |
| [REST-003](#rest-003) | 例外→HTTPレスポンスへのマッピング設計確認 | **MUST** | 業務例外・システム例外が統一されたエラーレスポンス形式に変換されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html) |
| [REST-006](#rest-006) | エラーレスポンスのJSON形式が全APIで統一されているか | **MUST** | バリデーション・認証・システムエラーのレスポンスJSON形式が全エンドポイントで統一されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html) |
| [REST-007](#rest-007) | REST APIリクエストボディのサーバーサイドバリデーション確認 | **MUST** | Bean ValidationまたはバリデーションアノテーションがJSONリクエストボディに適用されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html) |
| [REST-008](#rest-008) | リクエスト/レスポンスのメディアタイプ検証確認 | **Should** | `@Consumes`/`@Produces`でメディアタイプが宣言され、JSON以外のリクエストを拒否しているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html) |
| [REST-009](#rest-009) | コレクションリソースの件数制限とページング設計確認 | **Should** | 一覧系GETエンドポイントで全件返却していないか、ページングまたは件数上限が設定されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/universal_dao.html) |
| [REST-010](#rest-010) | PUT/DELETEの冪等性保証確認 | **Should** | 同じリクエストを複数回実行しても同じ結果になるか（冪等性）が確保されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html) |
| [REST-012](#rest-012) | APIリクエスト/レスポンスの適切なログ記録確認 | **Should** | リクエスト/レスポンスがログに記録され、機密情報がログに含まれていないか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/log.html) |
| [REST-014](#rest-014) | REST APIのURL設計がRESTful規約に従っているか確認 | **Should** | URLが名詞複数形でリソースを表現し、動詞をURLに含む設計（RPCスタイル）になっていないか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html) |
| [REST-016](#rest-016) | REST APIエンドポイントの自動テスト設計確認 | **Should** | Nablarch TestSupportを使ったテストが実装され、正常系・バリデーションエラー・認証エラーがカバーされているか | [公式](https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/index.html) |

---

## 各観点詳細

### REST-001
**RoutesMapping設定とアクションクラスの実装確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | `routes.rb`等のRoutesMapping設定ファイル（URLとアクションクラス・メソッドの対応）、アクションクラスの`@Produces`/`@Consumes`アノテーション |
| FW提供範囲 | URLルーティング処理（`RoutesMappingハンドラ`）、リクエストJSONのパース（`BodyConvertHandler`）、Bean Validation実行（`JaxRsBeanValidationHandler`）、HTTPレスポンス変換（`JaxRsResponseHandler`）はFWが自動処理 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html) |
| 優先度理由 | RoutesMapping設定の誤りはAPI全体のルーティング失敗・404を引き起こす。`@Path`アノテーションの誤使用はNablarchのアーキテクチャ原則に反し、意図通りに動作しない |
| チェック方法 | RoutesMapping設定（`routes.rb`等）にURLとアクションクラス・メソッドの対応が正しく定義されているか。アクションクラスに`@Produces(MediaType.APPLICATION_JSON)`または`@Consumes`が設定されているか。`@Path`アノテーションをルーティング目的で使用していないか（Nablarchでは非対応）。コンポーネント定義（XML）でRoutesMapping設定が正しく行われているか |

**NG例**
```java
// @Path をルーティング目的で使用（Nablarchでは動作しない）
@Path("/api/users")
public class UserAction {
    @GET
    public List<User> getUsers() { return dao.findAll(User.class); }
}
```

**OK例**
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

### REST-002
**レスポンスオブジェクトの型とステータスコードの設計確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | アクションクラスの`HttpResponse`/`Response`生成コード（ステータスコード設定箇所） |
| FW提供範囲 | HTTPレスポンスの送信処理はFW（`JaxRsResponseHandler`）が担当。実装者はステータスコードとレスポンスボディの内容を設定 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html) |
| 優先度理由 | 不正なステータスコードはクライアントのエラーハンドリングを破壊し、APIコントラクトの信頼性を損なう |
| チェック方法 | 作成成功時は201、正常取得は200、削除成功は204を返しているか。リソース未存在時は404、バリデーションエラーは400を返しているか。例外発生時に200を返していないか（try-catchで握りつぶしていないか） |

**NG例**
```java
@POST
@Path("/users")
public String createUser(UserForm form) {
    dao.insert(form);
    return "OK"; // ステータスコード指定なし → 常に 200
}
```

**OK例**
```java
@POST
@Path("/users")
public Response createUser(UserForm form) {
    dao.insert(form);
    return Response.status(Response.Status.CREATED).build(); // 201
}
```

---

### REST-003
**例外→HTTPレスポンスへのマッピング設計確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | `ExceptionMapper`実装クラス（業務例外→HTTPレスポンスのマッピング定義） |
| FW提供範囲 | `ExceptionMapper`自体の呼び出し機構はFWが提供。実装者はマッピング定義（業務例外→HTTPステータスコード・レスポンス形式）を実装 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html) |
| 優先度理由 | 例外マッピングが統一されていないと、エラー時のレスポンス形式がランダムになり、クライアントのエラーハンドリングが不可能になる |
| チェック方法 | `ExceptionMapper<T>`を実装した例外マッピングクラスが存在するか。業務例外→400、システム例外→500のマッピングが一貫しているか。エラーレスポンスのJSON形式が全APIで統一されているか |

**NG例**
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

**OK例**
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

### REST-006
**エラーレスポンスのJSON形式が全APIで統一されているか**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | `ExceptionMapper`のエラーレスポンスDTO定義、全APIのエラーレスポンスフィールド確認 |
| FW提供範囲 | JSONシリアライズはFW（`BodyConvertHandler`）が実行。実装者はエラーレスポンスの形式（フィールド構造・エラーコード体系）の統一を担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html) |
| 優先度理由 | エラーフォーマットの不統一はクライアントのエラーハンドリングコードを複雑にし、フロントエンド・連携システムとの統合を困難にする |
| チェック方法 | エラーレスポンスに`code`（エラーコード）と`message`（説明）フィールドが存在するか。バリデーションエラーで複数フィールドのエラーをまとめて返せるか。HTTPステータスコードとエラーコードが一貫した対応関係か |

**NG例**
```java
// エンドポイントによってフォーマットが異なる
// /api/users POST エラー
{"error": "バリデーションエラー"}
// /api/orders POST エラー
{"status": "NG", "detail": [{"field": "amount", "msg": "必須"}]}
```

**OK例**
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

### REST-007
**REST APIリクエストボディのサーバーサイドバリデーション確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | リクエストDTOクラスのBean Validationアノテーション定義（`@NotNull`・`@Size`等） |
| FW提供範囲 | `@Valid`アノテーション検出とバリデーション実行はFW（`JaxRsBeanValidationHandler`）が担当。実装者はDTOクラスへのアノテーション定義を担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html) |
| 優先度理由 | バリデーションのないREST APIは不正データの登録・処理エラーの原因となる。RESTではJSPのような入力制御がないため特に重要 |
| チェック方法 | リクエストDTOクラスにBean Validationアノテーションが定義されているか。`@Valid`アノテーションでバリデーションが実行されているか。バリデーションエラー時に400 Bad Requestと詳細メッセージが返されるか |

**NG例**
```java
@POST
public Response createUser(UserRequest request) {
    // バリデーションなし → null や不正値がそのまま DB へ
    dao.insert(request);
    return Response.ok().build();
}
```

**OK例**
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

### REST-008
**リクエスト/レスポンスのメディアタイプ検証確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | アクションクラスの`@Produces`と`@Consumes`アノテーション（Nablarchがサポートするアノテーション） |
| FW提供範囲 | `Content-Type`/`Accept`ヘッダのマッチング処理はFWが担当。実装者は`@Produces`/`@Consumes`で対応メディアタイプを宣言 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html) |
| 優先度理由 | `Content-Type`検証の欠如はメディアタイプインジェクション・予期しない解析エラーの原因となる |
| チェック方法 | `@Consumes(MediaType.APPLICATION_JSON)`がPOST/PUTメソッドに付与されているか。`@Produces(MediaType.APPLICATION_JSON)`が応答メソッドに付与されているか。JSON以外の`Content-Type`で415エラーが返されるか |

**NG例**
```java
@POST
// @Consumes 未指定 → 任意の Content-Type を受け入れる
public Response createUser(String body) { ... }
```

**OK例**
```java
@POST
@Consumes(MediaType.APPLICATION_JSON)
@Produces(MediaType.APPLICATION_JSON)
public Response createUser(UserRequest request) { ... }
```

---

### REST-009
**コレクションリソースの件数制限とページング設計確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | コレクション系アクションのページング条件実装コード（`page`パラメータ・`limit`パラメータ受け取り） |
| FW提供範囲 | ページングクエリの生成はFWが提供。実装者はページング条件の受け取り・設定とデフォルト上限値の実装を担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/universal_dao.html) |
| 優先度理由 | 件数無制限の全件返却は大量データ時のタイムアウト・メモリ枯渇・クライアント障害の原因となる |
| チェック方法 | 一覧系APIに`limit`パラメータ（またはデフォルト上限）が設定されているか。ページング情報（`totalCount`, `page`, `pageSize`）がレスポンスに含まれているか。最大返却件数の上限値（例: 1000件）が設定されているか |

**NG例**
```java
@GET
@Path("/api/users")
public List<User> getUsers() {
    return UniversalDao.findAll(User.class); // 全件返却
}
```

**OK例**
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

### REST-010
**PUT/DELETEの冪等性保証確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | PUT/DELETEアクションクラスの実装ロジック（存在チェック・冪等性確保） |
| FW提供範囲 | HTTPメソッドのルーティングはFWが担当。冪等性の実現（同じリクエストを複数回実行した場合の結果の一致）は実装者の設計が必要 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html) |
| 優先度理由 | 冪等でないPUT/DELETEはリトライ時に重複処理・データ破壊を引き起こす。RESTの設計原則として冪等性は重要 |
| チェック方法 | DELETEで既に削除済みのリソースへのリクエストが404または204を返すか。PUTが存在しないリソースに対して404を返し、勝手に作成しないか。ネットワーク障害時のリトライを考慮した設計か |

**NG例**
```java
@DELETE
@Path("/api/users/{id}")
public Response deleteUser(@PathParam("id") Long id) {
    dao.deleteById(id); // 存在しない場合に例外で 500 → 冪等でない
    return Response.ok().build();
}
```

**OK例**
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

### REST-012
**APIリクエスト/レスポンスの適切なログ記録確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | アクセスログ設定、アクションクラスのログ出力コード（機密情報マスキング含む） |
| FW提供範囲 | フレームワークレベルのアクセスログはFWが提供。業務的なリクエスト内容のログ出力（マスキング含む）は実装者が担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/log.html) |
| 優先度理由 | ログなしでは本番障害の原因究明が不可能になる。一方、パスワードや個人情報のログ記録はセキュリティインシデントの原因となる |
| チェック方法 | アクセスログ（メソッド・URL・ステータス・処理時間）がINFOレベルで出力されているか。リクエストボディにパスワード・カード番号等が含まれる場合、マスキングが実装されているか。エラー発生時にはスタックトレースがERRORレベルでログ出力されるか |

**NG例**
```
log.info("request: " + request.getBody()); // パスワード平文ログ
```

**OK例**
```
log.info("API: {} {} status={} time={}ms",
    request.getMethod(), request.getPath(),
    response.getStatus(), elapsedMs);
// パスワードフィールドはマスク済みオブジェクトをログ出力
```

---

### REST-014
**REST APIのURL設計がRESTful規約に従っているか確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | RoutesMapping定義のURL設計（`routes.rb`等）、アクションクラス名 |
| FW提供範囲 | URLルーティング解決はFWが担当。URL設計の妥当性（RESTful規約への準拠）の確認は実装者が担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html) |
| 優先度理由 | RESTfulでないURL設計はクライアント実装者の混乱を招き、APIの一貫性を損なう |
| チェック方法 | リソース名が名詞複数形か（`/users`, `/orders`, `/products`）。動詞がURLに含まれていないか（`/getUser`, `/createOrder`はNG）。リソースの階層関係がURLで表現されているか（`/users/{id}/orders`） |

**NG例**
```
GET  /getUsers         # 動詞が URL に入っている
POST /createUser       # 動詞が URL に入っている
POST /user/delete/{id} # HTTP メソッドと URL が不一致
```

**OK例**
```
GET    /api/users          # ユーザー一覧取得
POST   /api/users          # ユーザー作成
GET    /api/users/{id}     # ユーザー取得
PUT    /api/users/{id}     # ユーザー更新
DELETE /api/users/{id}     # ユーザー削除
```

---

### REST-016
**REST APIエンドポイントの自動テスト設計確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | NablarchのREST機能を使う（JaxRsアクションを実装する） |
| レビュー対象 | テストクラス（`RestTestSupport`等を継承したクラス） |
| FW提供範囲 | テストサポートクラス（`RestTestSupport`等）はFWが提供。テストケースの実装と正常系・バリデーションエラー・認証エラーのカバレッジは実装者が担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/index.html) |
| 優先度理由 | テストのないREST APIはリファクタリング・変更時のデグレを検知できず、本番障害のリスクが高まる |
| チェック方法 | `RestTestSupport`またはJUnit+RESTクライアントでのテストが存在するか。正常レスポンスのステータスコード・ボディ構造がテストで検証されているか。認証エラー（401）・バリデーションエラー（400）のテストケースがあるか |

**NG例**
```java
// リソースクラスのテストなし
```

**OK例**
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
