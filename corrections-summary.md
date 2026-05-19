# レビュー観点是正サマリー（cmd_471）

## 是正の背景

cmd_464 で作成したレビュー観点が Nablarch の設計思想に反する記述を含んでいた。
Nablarch は「定型処理は FW が吸収し、実装者は業務ロジックに集中」する設計であるが、
作成した観点の一部がフレームワーク責任範囲を「実装者が確認すべき項目」として列挙していた。

---

## 判断基準

| 分類 | 基準 |
|------|------|
| **KEEP** | 実装者が書く Java コード・XML 設定・フォームクラス等で確認可能な事項 |
| **REWRITE** | 趣旨は妥当だが FW 責任箇所と実装者責任箇所が混在している |
| **REMOVE** | FW 責任範囲であるため実装者がレビューする必要がない |

---

## 変更一覧

### BATCH（バッチ処理）

| 観点ID | 旧タイトル | 新タイトル | 分類 | 変更理由 |
|--------|-----------|-----------|------|---------|
| BATCH-001 | ExecutionListener のライフサイクルメソッド実装確認 | カスタムExecutionListener実装時の確認（追加実装がある場合のみ） | **REWRITE** | 通常、実装者は ExecutionListener を implements しない。FW の標準実装が前後処理を担当。カスタム追加時のみ確認対象 |
| BATCH-002 | DataReader と DataWriter のオープン/クローズ対称性 | （削除） | **REMOVE** | DataReader.open()/close() の呼び出しは FW（DataReadHandler）が自動管理。実装者が open/close を呼ぶコードを書く必要がなく、レビュー観点として不適切 |
| BATCH-018（新） | （新規） | カスタムDataReader実装時のclose()業務後処理確認 | **新規追加** | BATCH-002 の代替。FW が close() を自動呼び出しする点を踏まえ、カスタム DataReader の close() 内の業務後処理（リソース解放・キャッシュクリア等）の確認に特化 |
| BATCH-003〜017 | （変更なし） | — | **KEEP** | 実装者が記述する XML 設定・Java コード・SQL を対象とした妥当な観点 |

### REST API（RESTful Web サービス）

| 観点ID | 旧タイトル | 新タイトル | 分類 | 変更理由 |
|--------|-----------|-----------|------|---------|
| REST-001 | JAX-RS アノテーションとリソースクラスの実装パターン確認 | RoutesMapping設定とアクションクラスの実装確認 | **REWRITE** | Nablarch の JAX-RS サポートでは @Path アノテーションはルーティングに使用しない。URL ルーティングは RoutesMapping（routes.rb 等）で行う。使用できるアノテーションは @Produces/@Consumes/@Valid の 3 種のみ（公式ドキュメント記載） |
| REST-002〜016 | （変更なし） | — | **KEEP** | 実装者が記述する Java コード・設定を対象とした妥当な観点 |

### Web アプリケーション

| 観点ID | 分類 | 備考 |
|--------|------|------|
| WEB-001〜016（全16観点） | **KEEP** | 全観点が実装者が記述するコード・設定を対象とした妥当な観点 |

---

## 新規追加メタデータフィールド

全49観点に以下のフィールドを追加:

| フィールド名 | 内容 |
|------------|------|
| `responsibility` | 責任区分（全観点 `developer`） |
| `review_target` | 実際にレビューするクラス・ファイル・設定の具体的な記述 |
| `framework_provided` | FW が自動処理する範囲（実装者が書かなくてよいこと）の明示 |

---

## 修正前後の観点総数

| 処理方式 | 修正前 | 修正後 | 差分 |
|---------|--------|--------|------|
| バッチ | 17 | 17 | ±0（REMOVE 1, 新規追加 1） |
| Web | 16 | 16 | ±0 |
| REST | 16 | 16 | ±0 |
| **合計** | **49** | **49** | **±0** |

---

## REMOVE観点の根拠（詳細）

### BATCH-002: DataReader と DataWriter のオープン/クローズ対称性

**旧観点の問題点**:
チェック方法に「`DataReader.open()` の呼び出しに対応する `close()` が try-finally または AutoCloseable で保護されているか」と記載していたが、これは FW が担当する処理を実装者に確認させる誤った観点。

**根拠（FW の実際の動作）**:
- DataReadHandler（FW 提供のハンドラ）がバッチのループ処理と DataReader の open/close を自動管理する
- 実装者が呼び出し側の open()/close() を try-finally で保護するコードを書く必要はない
- 参照: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html

**代替観点（BATCH-018）**:
カスタム DataReader を実装する場合に限り、close() メソッドの中身（業務的後処理）の実装確認に特化した観点を新規追加した。

---

## REWRITE観点の根拠（詳細）

### BATCH-001: ExecutionListener

**旧観点の問題点**:
チェック方法に「`ExecutionListener` インタフェースを `implements` しているか」と記載しており、全バッチで必須確認事項のように示していた。

**根拠（FW の実際の動作）**:
- Nablarch は標準の ExecutionListener 実装を提供しており、通常の前後処理（実行開始・終了ログ等）は FW の標準実装が担当する
- 通常、実装者は ExecutionListener を implements しない
- プロジェクト固有の前後処理が必要な場合のみ、カスタム実装を追加する

**是正後の観点**:
カスタム ExecutionListener を追加実装している場合のみ、その実装の妥当性を確認する観点に変更。優先度も MUST → Should に修正（条件付き確認のため）。

### REST-001: JAX-RS アノテーション

**旧観点の問題点**:
「リソースクラスに `@Path` アノテーションがあるか」「各メソッドに HTTP メソッドアノテーション（`@GET`, `@POST`）が付与されているか」と記載していたが、Nablarch の JAX-RS サポートでは @Path アノテーションをルーティング目的に使用しない。

**根拠（FW の実際の動作）**:
- Nablarch JAX-RS サポートで使用できるアノテーションは @Produces/@Consumes/@Valid の 3 種のみ（公式ドキュメント明記）
- URL ルーティングは RoutesMapping（routes.rb 等）で設定する
- 参照: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html

**是正後の観点**:
RoutesMapping 設定の確認と、@Produces/@Consumes の設定確認に焦点を変更。@Path 使用は誤りである旨を NG 例として明示。
