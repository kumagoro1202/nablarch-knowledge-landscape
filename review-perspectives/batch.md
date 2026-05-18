# Nablarch バッチ処理 レビュー観点プリセット

対象: Nablarch バッチアーキテクチャ（nablarch-fw-batch）

---

## BATCH-001: ExecutionListener の正しい実装パターン

**観点タイトル**: ExecutionListener のライフサイクルメソッド実装確認

**観点詳細**: バッチ処理の前後処理（初期化・後処理）を担う `ExecutionListener` が、Nablarch の規約通りに実装されているか確認する。`beforeExecute` / `afterExecute` の両メソッドが定義され、例外発生時にリソースリークが起きない構造になっているか。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/getting_started/nablarch_batch/index.html

**優先度**: MUST

**優先度の理由**: ExecutionListener が不正実装だとバッチ開始・終了時の必須処理が実行されず、リソースリーク・データ不整合が発生する。

**チェック方法**:
- `ExecutionListener` インタフェースを `implements` しているか
- `beforeExecute` / `afterExecute` の両方が実装されているか
- `afterExecute` で例外が発生しても後続の後処理が実行される設計か（try-finally 利用等）

**NG例**:
```java
// afterExecute が空実装で後処理なし
public void afterExecute(String requestPath, ExecutionContext context) {
    // 何もしない
}
```

**OK例**:
```java
public void afterExecute(String requestPath, ExecutionContext context) {
    try {
        connection.close();
    } finally {
        LogUtil.log("batch finished: " + requestPath);
    }
}
```

---

## BATCH-002: DataReader / DataWriter の対称実装確認

**観点タイトル**: DataReader と DataWriter のオープン/クローズ対称性

**観点詳細**: `DataReader` の `open` に対して必ず `close` が呼ばれるよう設計されているか。`DataWriter`（ファイル出力等）も同様。リソース漏洩の原因となる非対称実装を検出する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html

**優先度**: MUST

**優先度の理由**: `close` が呼ばれない場合、ファイルハンドル・DB接続がリークし、長時間バッチやリトライ時に障害となる。

**チェック方法**:
- `DataReader.open()` の呼び出しに対応する `close()` が try-finally または AutoCloseable で保護されているか
- 例外パスでも `close` が保証されているか

**NG例**:
```java
reader.open(context);
List<Data> records = reader.readAll(); // 例外発生時 close されない
reader.close(context);
```

**OK例**:
```java
reader.open(context);
try {
    List<Data> records = reader.readAll();
} finally {
    reader.close(context);
}
```

---

## BATCH-003: トランザクション境界の適切な設定

**観点タイトル**: コミット単位・トランザクション境界の設計妥当性

**観点詳細**: バッチのコミット間隔（`commitInterval`）がコンポーネント定義で正しく設定されているか。大量データを1トランザクションで処理するとロールバック時の影響が甚大になる。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html

**優先度**: MUST

**優先度の理由**: トランザクション境界の誤設定はデータ整合性破壊・ロールバック時の全件再処理・メモリ枯渇を引き起こす。

**チェック方法**:
- `commitInterval` が業務要件（データ量・リカバリ要件）に基づいて設定されているか
- 1コミット間のレコード数が過大（10,000件超）になっていないか
- コミット失敗時のリトライ戦略が定義されているか

**NG例**:
```xml
<!-- commitInterval 未設定 → デフォルト値依存 -->
<component name="batchAction" class="com.example.SampleBatchAction"/>
```

**OK例**:
```xml
<component name="batchAction" class="com.example.SampleBatchAction">
    <property name="commitInterval" value="100"/>
</component>
```

---

## BATCH-004: エラーハンドリング・コミット前リトライ制御

**観点タイトル**: 一時エラー時のリトライ設計とスキップ制御

**観点詳細**: 一時的なエラー（デッドロック・タイムアウト等）に対するリトライ制御と、スキップ可能エラーの定義が適切か確認する。無限リトライや全件スキップが起きない設計であることを確認する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html

**優先度**: MUST

**優先度の理由**: リトライ設計が不適切だと、一時エラーで処理が停止するか、データ欠損を無視して処理が継続するかの二択になり、どちらも業務障害となる。

**チェック方法**:
- `RetryableException` と非リトライ例外が明確に区別されているか
- リトライ上限回数（`maxRetryCount`）が設定されているか
- スキップ対象の例外クラスが明示されているか

**NG例**:
```java
catch (Exception e) {
    // 全例外を無視してスキップ
    log.warn("skip record: " + e.getMessage());
}
```

**OK例**:
```java
catch (DeadlockException e) {
    throw new RetryableException(e); // リトライ対象
} catch (ValidationException e) {
    errorWriter.write(record);       // スキップして記録
}
```

---

## BATCH-005: 大量データ処理時のメモリ管理

**観点タイトル**: チャンクサイズ・ストリーミング処理によるメモリ制御

**観点詳細**: バッチ処理で大量レコードをメモリ上に一括ロードしていないか確認する。`DataReader` がストリーミング（逐次読み込み）で実装されているか、または適切なチャンクサイズが設定されているか。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html

**優先度**: MUST

**優先度の理由**: 数百万件のデータを `List` で一括取得すると OutOfMemoryError が発生し、本番バッチが停止する。

**チェック方法**:
- `DataReader.readAll()` や `List` への一括格納を使っていないか
- DB から取得する場合、カーソル/フェッチサイズが適切に設定されているか
- ファイル読み込みの場合、`BufferedReader` 等で逐次処理しているか

**NG例**:
```java
// 全件をメモリにロード
List<Record> allRecords = dao.findAll(); // OOM リスク
for (Record r : allRecords) { process(r); }
```

**OK例**:
```java
// DataReader でストリーミング読み込み
public Record read(ExecutionContext ctx) {
    return cursor.hasNext() ? cursor.next() : null;
}
```

---

## BATCH-006: コンポーネント定義（component-configuration）の正確性

**観点タイトル**: XML/アノテーションによるコンポーネント定義の整合性確認

**観点詳細**: バッチのコンポーネント定義（`component-configuration.xml` 等）が実際の実装クラスと一致しているか。未使用定義・参照切れ・プロパティ名のタイポがないか確認する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/repository.html

**優先度**: MUST

**優先度の理由**: コンポーネント定義の誤りはアプリケーション起動失敗につながる。開発環境では動作しても本番環境の設定ファイル差異で障害が発生することもある。

**チェック方法**:
- `component` タグの `class` 属性が実在するクラス名か
- `property` タグの `name` 属性がクラスのフィールド名と一致するか
- 定義されているが参照されていないコンポーネントがないか

**NG例**:
```xml
<component name="batchAction" class="com.example.SampleBatchActoin"/> <!-- タイポ -->
    <property name="commitIntervel" value="100"/>  <!-- タイポ -->
```

**OK例**:
```xml
<component name="batchAction" class="com.example.SampleBatchAction">
    <property name="commitInterval" value="100"/>
</component>
```

---

## BATCH-007: ステップ実行の中断・再開設計

**観点タイトル**: バッチ中断時のチェックポイント・再開ポイント設計

**観点詳細**: バッチが途中で停止した場合に、どこから再開するかのチェックポイント設計が存在するか。無設計の場合、全件再処理でデータ二重登録が発生する可能性がある。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html

**優先度**: Should

**優先度の理由**: 大量データバッチで中断再開設計がないと、障害復旧時の全件再処理でダウンタイムが長期化し、データ品質も損なわれる。

**チェック方法**:
- 処理済みレコードのマーキング（フラグ更新・別テーブル記録）が実装されているか
- 再開時に未処理レコードのみを読み込む `DataReader` の WHERE 条件が正しいか
- 再実行時に二重処理が発生しない冪等性設計になっているか

**NG例**:
```sql
-- 全件取得（再実行時に重複処理）
SELECT * FROM batch_target
```

**OK例**:
```sql
-- 未処理のみ取得
SELECT * FROM batch_target WHERE status = 'UNPROCESSED'
```

---

## BATCH-008: ログ出力の適切なレベル・フォーマット

**観点タイトル**: バッチ処理ログの詳細度・構造化確認

**観点詳細**: バッチ開始/終了・処理件数・エラー発生をログに記録しているか。ログレベルが適切（INFO/WARN/ERROR）で、本番運用に耐えるフォーマットか確認する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/log.html

**優先度**: Should

**優先度の理由**: ログ不足は障害時の原因究明を困難にする。過剰なDEBUGログは本番パフォーマンスに影響する。

**チェック方法**:
- バッチ開始・終了・処理件数が INFO レベルで記録されているか
- 業務エラーは WARN、システムエラーは ERROR レベルか
- `LogUtil` や Nablarch のロギング API を使用しているか（`System.out.println` 不使用）

**NG例**:
```java
System.out.println("処理完了: " + count + "件"); // 本番ログに出ない
```

**OK例**:
```java
Logger log = LoggerFactory.getLogger(getClass());
log.logInfo("バッチ完了", "処理件数: " + count);
```

---

## BATCH-009: 入力データバリデーション設計

**観点タイトル**: バッチ入力レコードのバリデーション実装確認

**観点詳細**: バッチ入力データ（ファイル・DB）に対するバリデーションが実装されているか。バリデーションエラーレコードの扱い（スキップ・エラーファイル出力）が仕様通りか確認する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/validation.html

**優先度**: Should

**優先度の理由**: 入力バリデーションが欠如すると、不正データがDBに登録され後続処理の障害となる。

**チェック方法**:
- `ValidationUtil.validate()` または Bean Validation アノテーションが使用されているか
- バリデーションエラー発生時の処理フロー（スキップ/停止）が業務仕様と一致するか
- エラーレコードはエラーファイルまたはログに記録されているか

**NG例**:
```java
// バリデーションなしで直接DB登録
dao.insert(record); // 不正データが混入する
```

**OK例**:
```java
ValidationUtil.validate(record);
dao.insert(record);
```

---

## BATCH-010: 排他制御・二重起動防止設計

**観点タイトル**: 同一バッチの同時実行防止機構の確認

**観点詳細**: 同一バッチジョブが同時に複数起動された場合に、二重処理が発生しない排他制御が実装されているか確認する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/exclusive_control.html

**優先度**: Should

**優先度の理由**: 二重起動によるデータ二重処理は金融・会計系バッチでは重大なデータ不整合を引き起こす。

**チェック方法**:
- DB ロック・ファイルロック・`nablarch-common-exclusivecontrol` による排他が実装されているか
- ロック取得失敗時のエラーメッセージ・終了コードが適切か
- バッチ終了時（正常・異常）にロックが確実に解放されるか

**NG例**:
```java
// 排他制御なしで直接処理開始
processBatch();
```

**OK例**:
```java
ExclusiveControlManager.lock("BATCH_JOB_A");
try {
    processBatch();
} finally {
    ExclusiveControlManager.unlock("BATCH_JOB_A");
}
```

---

## BATCH-011: ファイル入出力のエンコーディング指定

**観点タイトル**: 固定長・CSV ファイルの文字コード設定確認

**観点詳細**: ファイル形式の入出力で文字コード（UTF-8/Shift-JIS 等）が明示的に指定されているか。デフォルトエンコーディングに依存していると OS・JVM 設定によって文字化けが発生する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/data_io/data_format.html

**優先度**: MUST

**優先度の理由**: エンコーディング未指定は実行環境依存の文字化けを引き起こし、本番障害・データ破損の原因となる。

**チェック方法**:
- フォーマット定義ファイル（`format.fmt`）に `file-type` と `text-encoding` が指定されているか
- `DataBindConfig` 等で明示的にエンコーディングが設定されているか

**NG例**:
```java
// エンコーディング未指定
new FileReader("input.csv") // JVM デフォルト依存
```

**OK例**:
```
# format.fmt
file-type: CSV
text-encoding: UTF-8
```

---

## BATCH-012: 終了コードの適切な設定

**観点タイトル**: バッチ終了コードと監視ツール連携の確認

**観点詳細**: バッチの正常終了・異常終了・警告終了で異なる終了コードが返されているか。ジョブスケジューラ（JP1 等）が正しく終了判定できる設計か確認する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html

**優先度**: Should

**優先度の理由**: 終了コードが常に 0（正常）だと、異常終了を監視システムが検知できず、障害が隠蔽される。

**チェック方法**:
- 正常終了: 0、業務エラー: 1〜9、システムエラー: 10〜 の区分が設計されているか
- `System.exit()` の代わりに `BatchCommandLine` の終了コード機構を使っているか
- ジョブスケジューラとの終了コード連携が文書化されているか

**NG例**:
```java
catch (Exception e) {
    log.error("エラー", e);
    System.exit(0); // エラーなのに正常終了コード
}
```

**OK例**:
```java
catch (SystemException e) {
    log.error("システムエラー", e);
    System.exit(10);
}
```

---

## BATCH-013: 処理件数・統計情報のレポーティング

**観点タイトル**: バッチ処理結果統計の出力確認

**観点詳細**: バッチ完了後に処理件数・スキップ件数・エラー件数が記録・出力されているか。運用チームがバッチ結果を確認できるレポーティング設計か確認する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html

**優先度**: Should

**優先度の理由**: 統計情報がないと、バッチが実行されたか・何件処理されたかを後から確認できず、業務確認・監査対応が困難になる。

**チェック方法**:
- `ExecutionContext` から処理件数を取得して最終ログ出力しているか
- エラーレコード数が明示的にカウントされているか
- 統計情報を専用テーブル・ファイルに永続化しているか

**NG例**:
```java
// 件数カウントなし
for (Record r : records) {
    process(r);
}
log.info("バッチ完了"); // 件数不明
```

**OK例**:
```java
int processed = 0, skipped = 0;
for (Record r : records) {
    try { process(r); processed++; }
    catch (SkipException e) { skipped++; }
}
log.info("完了: 処理=" + processed + " スキップ=" + skipped);
```

---

## BATCH-014: テスト可能性（TestSupport の活用）

**観点タイトル**: Nablarch TestSupport を使ったバッチ単体テスト設計

**観点詳細**: バッチアクションのユニットテストが `NablarchTestSupport` または `BatchRequestTestSupport` を使って作成されているか。DBUnit との組み合わせでデータ駆動テストが実現されているか確認する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/index.html

**優先度**: Should

**優先度の理由**: テストのないバッチは変更時のデグレ検知ができず、本番障害リスクが高まる。

**チェック方法**:
- `BatchRequestTestSupport` を継承したテストクラスが存在するか
- テストデータが Excel または YAML で外部定義されているか
- 正常系・異常系の両テストケースが存在するか

**NG例**:
```java
// バッチアクションのテストなし（テストクラス自体が存在しない）
```

**OK例**:
```java
public class SampleBatchActionTest extends BatchRequestTestSupport {
    @Test
    public void 正常終了() { execute("testCases/normal.xlsx"); }
}
```

---

## BATCH-015: ジョブ定義とリクエストパスの整合性

**観点タイトル**: リクエストパスとコンポーネント定義の対応確認

**観点詳細**: バッチ起動コマンドに渡すリクエストパス（`-requestPath`）がコンポーネント定義の `name` と一致しているか確認する。不一致だとバッチが起動しない。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/getting_started/nablarch_batch/index.html

**優先度**: MUST

**優先度の理由**: リクエストパスの不一致はバッチ起動失敗の最も基本的な原因であり、リリース直前に発覚すると影響が大きい。

**チェック方法**:
- 起動スクリプト・Jenkins ジョブ等のリクエストパスと XML のコンポーネント名が完全一致するか
- ステージング環境でのバッチ起動テストが実施済みか

**NG例**:
```bash
# コンポーネント名が "sampleBatchAction" なのに
java ... -requestPath=sample_batch_action  # アンダースコア形式で不一致
```

**OK例**:
```bash
java ... -requestPath=sampleBatchAction  # コンポーネント名と一致
```

---

## BATCH-016: DataBindConfig によるファイルフォーマット定義の完全性

**観点タイトル**: ファイルフォーマット定義の項目網羅性・型定義確認

**観点詳細**: フォーマット定義ファイル（`.fmt`）の項目定義がファイル仕様書と完全に一致しているか。項目の型・桁数・必須/任意が正確に定義されているか確認する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/data_io/data_format.html

**優先度**: MUST

**優先度の理由**: フォーマット定義の誤りはパース失敗・データ欠損・文字化けに直結し、バッチ全件失敗の原因となる。

**チェック方法**:
- フォーマット定義の項目数・順序がファイル仕様書と一致するか
- 固定長ファイルの場合、各項目のバイト数の合計がレコード長と一致するか
- 必須項目に `required` 制約が設定されているか

**NG例**:
```
# 仕様: 氏名30バイト / 定義: 氏名20バイト → 切り捨て発生
[氏名]
dataType: X
length: 20
```

**OK例**:
```
[氏名]
dataType: X
length: 30
required: true
```

---

## BATCH-017: 非機能要件（性能・SLA）を考慮した設計確認

**観点タイトル**: バッチ処理時間の設計目標値と実測値の妥当性確認

**観点詳細**: バッチの処理時間が業務 SLA（処理ウィンドウ）に収まる設計になっているか。ボトルネック（DB I/O・外部連携）の分析と改善策が検討されているか確認する。

**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html

**優先度**: Should

**優先度の理由**: SLA 未達のバッチは翌業務日の開始に影響し、業務停止を引き起こす。設計段階での性能見積もりが重要。

**チェック方法**:
- 処理件数 × 1件あたり処理時間の概算が処理ウィンドウ内に収まるか
- DB の SELECT/INSERT に適切なインデックスが設定されているか
- 外部 API 呼び出しがある場合、タイムアウト・サーキットブレーカーが設定されているか

**NG例**:
```java
// 1件ごとに SELECT + INSERT（N+1 問題）
for (Record r : records) {
    dao.findByKey(r.getId()); // N回 SELECT
    dao.insert(r);            // N回 INSERT
}
```

**OK例**:
```java
// バルク INSERT で性能改善
dao.batchInsert(records); // 一括 INSERT
```
