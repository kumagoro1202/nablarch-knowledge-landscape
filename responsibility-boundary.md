# Nablarch フレームワーク vs 実装者 責任境界ガイド

本ドキュメントは、Nablarch フレームワーク（以下 FW）が自動処理する範囲と、実装者が Java コードや XML 設定で記述する必要がある範囲を明確に区分する。

レビュー観点を追加・修正する際は、必ずこのドキュメントを参照し、FW 責任範囲を「実装者が確認すべき項目」として記載しないようにすること。

---

## バッチ処理（nablarch-fw-batch）

**参照**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html

### FW が自動処理すること（実装者が書かなくてよいこと）

| 処理 | FW クラス/機構 | 注意点 |
|------|---------------|--------|
| DataReader.open() / close() の呼び出し | DataReadHandler | 実装者が open/close を呼ぶコードは書かない |
| トランザクション開始・コミット・ロールバック | トランザクション制御ハンドラ | 実装者はcommitIntervalの設定値のみ担当 |
| レコードの繰り返し読み込み | DataReadHandler | DataReader.read()の呼び出しをループするのはFW |
| 実行ID の採番 | DataReadHandler | ExecutionContext に自動セット |
| ExecutionListener の標準的な前後処理 | FW 標準実装 | 通常はimplementsしない。カスタム前後処理が必要な場合のみ追加 |
| ハンドラキューの実行制御 | BatchCommandLine / DispatchHandler | 実装者はアクションクラスの業務ロジックのみ実装 |
| リクエストパスに基づくアクション解決 | DispatchHandler | 実装者はコンポーネント名の正確性を確認 |

### 実装者が書くこと（レビュー観点の対象）

| 実装対象 | クラス/ファイル | レビューポイント |
|---------|---------------|----------------|
| 業務処理ロジック | BatchAction.execute() | 処理の正確性、例外ハンドリング |
| コミット間隔設定 | component-configuration.xml の commitInterval | 業務要件に基づく適切な値 |
| DataReader の read() 実装 | カスタム DataReader クラス | ストリーミング処理、カーソル管理 |
| カスタム DataReader の close() 業務後処理 | カスタム DataReader.close() | キャッシュクリア、一時ファイル削除等 |
| バリデーション呼び出し | BatchAction 内の ValidationUtil.validate() | 入力データの整合性確認 |
| ログ出力 | BatchAction 内のログ API 呼び出し | 処理件数・エラー件数の記録 |
| コンポーネント定義 XML | component-configuration.xml | クラス名・プロパティ名の正確性 |
| フォーマット定義 | .fmt ファイル | 項目数・バイト数・エンコーディング |
| リトライ/スキップ設計 | BatchAction の例外ハンドリング | RetryableException の使用 |
| チェックポイント設計 | SQL の WHERE 条件・処理済みフラグ | 冪等性の確保 |
| 排他制御 | ExclusiveControlManager の呼び出し | ロック取得・解放の対称性 |
| 終了コード | System.exit() または BatchCommandLine 設定 | ジョブスケジューラ連携 |
| テスト | BatchRequestTestSupport を使ったテストクラス | 正常系・異常系のカバレッジ |

---

## Web アプリケーション（nablarch-fw-web）

**参照**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/architecture.html

### FW が自動処理すること

| 処理 | FW クラス/機構 | 注意点 |
|------|---------------|--------|
| 文字エンコーディング変換 | HttpCharacterEncodingHandler | 実装者はハンドラの設定をするだけ |
| トランザクション開始・コミット・ロールバック | トランザクション制御ハンドラ | 実装者は業務処理のみ実装 |
| セッションストアの読み書き（往路・復路） | セッション変数保存ハンドラ | SessionUtil API 経由で操作 |
| CSRF トークン生成・検証 | CsrfTokenHandler | JSP に `<n:csrf>` タグを配置 |
| URI に基づくアクション解決 | DispatchHandler | アクションクラスの実装は実装者 |
| 二重サブミットトークン生成・検証 | DoubleSubmissionHandler | JSP に `<n:token>` タグを配置 |
| マルチパートのパース | MultipartHandler | サイズ上限設定は実装者 |
| HTML エスケープ処理 | `<n:write>` タグ内部 | 実装者は `<n:write>` を使う |
| メッセージID → 文言 解決 | MessageUtil | 実装者はメッセージ定義ファイルを作成 |
| ページングクエリ生成 | UniversalDao | 実装者はページング条件（件数・ページ番号）を設定 |

### 実装者が書くこと

| 実装対象 | クラス/ファイル | レビューポイント |
|---------|---------------|----------------|
| 業務アクションクラス | HttpRequest/ExecutionContext を受け取るメソッド | 業務ロジック、画面遷移、SessionUtil の使用 |
| バリデーション定義 | Form クラスのアノテーション、ValidationUtil.validate() | 入力値の制約定義 |
| ハンドラキュー設定 | web-component-configuration.xml の handlerQueue | ハンドラの種類・順序 |
| エラーページ設定 | web.xml の error-page | 全エラーコードのカバー |
| セキュリティヘッダ設定 | フィルタまたはアクション | X-Frame-Options 等の設定 |
| JSP の出力タグ選択 | JSP ファイル | `<n:write>` の使用、EL 式の直接使用回避 |
| SQL のバインドパラメータ | .sql ファイル | 文字列連結を使わない |
| ログアウト処理 | ログアウトアクション | SessionUtil.invalidate() の呼び出し |
| テスト | HttpRequestTestSupport を使ったテストクラス | 正常系・認証エラー系のカバレッジ |

---

## RESTful Web サービス（nablarch-fw-jaxrs）

**参照**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html

### 重要: Nablarch JAX-RS サポートの制約

Nablarch の Jakarta RESTful Web Services サポートで使用できるアノテーションは以下の 3 種のみ:
- **`@Produces`**: レスポンスのメディアタイプ指定
- **`@Consumes`**: リクエストのメディアタイプ指定
- **`@Valid`**: Bean Validation の実行

**`@Path` アノテーションはルーティングには使用しない**。URL とアクションクラスの紐付けは `RoutesMapping`（routes.rb 等）で行う。`@Context`（CDI）も非対応。

### FW が自動処理すること

| 処理 | FW クラス/機構 | 注意点 |
|------|---------------|--------|
| URL → アクション解決 | RoutesMapping / JaxRsMethodBinderFactory | 実装者は routes.rb または XML でルート定義 |
| リクエスト JSON → フォームクラス変換 | BodyConvertHandler | 実装者はフォームクラスを定義するだけ |
| Bean Validation 実行 | JaxRsBeanValidationHandler | @Valid アノテーション + FW が実行 |
| HttpResponse → HTTP レスポンス変換 | JaxRsResponseHandler | 実装者は HttpResponse を返すだけ |
| トランザクション管理 | トランザクション制御ハンドラ | バッチ・Web と同様 |

### 実装者が書くこと

| 実装対象 | クラス/ファイル | レビューポイント |
|---------|---------------|----------------|
| ルート定義 | routes.rb または RoutesMapping XML 設定 | URL・HTTPメソッド・アクションクラスの対応 |
| アクションクラス（業務ロジック） | HttpResponse を返すメソッド | @Produces/@Consumes アノテーション、ステータスコード |
| リクエスト DTO | フォームクラスの Bean Validation アノテーション | @NotNull/@Size 等の制約定義 |
| ExceptionMapper 実装 | 例外 → HTTP レスポンス マッピングクラス | エラーレスポンス形式の統一 |
| CORS ヘッダ設定 | フィルタまたはアクション | Access-Control-Allow-Origin 等 |
| 認証・認可チェック | JWT 検証ロジック、PermissionUtil 等 | 全保護エンドポイントへの適用 |
| API スキーマ定義 | openapi.yaml / swagger.json | 仕様とコードの一致 |
| テスト | RestTestSupport を使ったテストクラス | 正常系・バリデーションエラー・認証エラー |
