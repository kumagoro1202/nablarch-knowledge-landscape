# Nablarch バッチ処理 レビュー観点プリセット
対象: Nablarch バッチアーキテクチャ（nablarch-fw-batch）
---
## BATCH-001: カスタムExecutionListener実装時の確認（追加実装がある場合のみ）
**観点タイトル**: カスタムExecutionListener実装時の確認（追加実装がある場合のみ）
**観点詳細**: Nablarchバッチフレームワークは標準のExecutionListenerを提供しており、通常の前後処理はFWの標準実装が担当するため、実装者がExecutionListenerをimplementsする必要はない。プロジェクト固有の前後処理（外部APIへの通知・カスタムログ出力等）でカスタムExecutionListenerを追加実装した場合のみ、その実装の妥当性を確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/getting_started/nablarch_batch/index.html
**優先度**: Should
**優先度の理由**: カスタムExecutionListenerが不正実装だとプロジェクト固有の前後処理が実行されず、リソースリーク・データ不整合が発生する。ただし、標準実装のみ使用している場合は本観点の対象外。
**責任区分**: developer
**FW提供範囲**: 標準的なバッチ前後処理はFWの標準ExecutionListener実装が提供。通常、実装者はimplementsしない。プロジェクト固有の前後処理（外部API通知・カスタムログ等）が必要な場合のみカスタム実装を追加する
**レビュー対象**: カスタムExecutionListenerクラス（プロジェクトで追加実装した場合のみ）。通常は標準実装を使用するため対象なし
**チェック方法**:
カスタムExecutionListenerをimplementsしているクラスがある場合のみ確認。beforeExecute/afterExecuteの実装が業務的に必要な処理を行っているか。afterExecuteで例外が発生しても後続の後処理が実行される設計か（try-finally等）。標準のExecutionListenerで対応できないプロジェクト固有の要件がある場合のみ追加実装されているか。
**NG例**:
```java
// カスタムExecutionListenerのafterExecuteが空実装
public void afterExecute(String requestPath, ExecutionContext context) {
    // 何もしない（追加実装したのに処理なし）
}
```
**OK例**:
```java
// プロジェクト固有の前後処理が必要な場合のカスタム実装
public void afterExecute(String requestPath, ExecutionContext context) {
    try {
        externalMonitoringApi.notifyBatchEnd(requestPath); // 外部監視API通知
    } finally {
        LogUtil.log("batch finished: " + requestPath);
    }
}
```
---
## BATCH-003: コミット単位・トランザクション境界の設計妥当性
**観点タイトル**: コミット単位・トランザクション境界の設計妥当性
**観点詳細**: バッチのコミット間隔（commitInterval）がコンポーネント定義で正しく設定されているか。大量データを1トランザクションで処理するとロールバック時の影響が甚大になる。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html
**優先度**: MUST
**優先度の理由**: トランザクション境界の誤設定はデータ整合性破壊・ロールバック時の全件再処理・メモリ枯渇を引き起こす。
**責任区分**: developer
**FW提供範囲**: トランザクションの開始・コミット・ロールバック自体はトランザクション制御ハンドラが自動実行。実装者はcommitIntervalの値設定のみ担当
**レビュー対象**: component-configuration.xml のcommitInterval設定値
**チェック方法**:
commitInterval が業務要件（データ量・リカバリ要件）に基づいて設定されているか。1コミット間のレコード数が過大（10,000件超）になっていないか。コミット失敗時のリトライ戦略が定義されているか。
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
## BATCH-004: 一時エラー時のリトライ設計とスキップ制御
**観点タイトル**: 一時エラー時のリトライ設計とスキップ制御
**観点詳細**: 一時的なエラー（デッドロック・タイムアウト等）に対するリトライ制御と、スキップ可能エラーの定義が適切か確認する。無限リトライや全件スキップが起きない設計であることを確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html
**優先度**: MUST
**優先度の理由**: リトライ設計が不適切だと、一時エラーで処理が停止するか、データ欠損を無視して処理が継続するかの二択になり、どちらも業務障害となる。
**責任区分**: developer
**FW提供範囲**: ハンドラキューの例外伝播制御はFWが管理。RetryableExceptionの機構はFWが提供。実装者はどの例外をリトライ/スキップ対象にするかのロジックを実装する
**レビュー対象**: 業務BatchActionクラスの例外ハンドリングコード
**チェック方法**:
RetryableException と非リトライ例外が明確に区別されているか。リトライ上限回数（maxRetryCount）が設定されているか。スキップ対象の例外クラスが明示されているか。
**NG例**:
```java
catch (Exception e) {
    // 全例外を無視してスキップ
    log.warn("skip record: " + e.getMessage());
}
```
**OK例**:
```
catch (DeadlockException e) {
    throw new RetryableException(e); // リトライ対象
} catch (ValidationException e) {
    errorWriter.write(record);       // スキップして記録
}
```
---
## BATCH-005: チャンクサイズ・ストリーミング処理によるメモリ制御
**観点タイトル**: チャンクサイズ・ストリーミング処理によるメモリ制御
**観点詳細**: バッチ処理で大量レコードをメモリ上に一括ロードしていないか確認する。DataReader がストリーミング（逐次読み込み）で実装されているか、または適切なチャンクサイズが設定されているか。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html
**優先度**: MUST
**優先度の理由**: 数百万件のデータを List で一括取得すると OutOfMemoryError が発生し、本番バッチが停止する。
**責任区分**: developer
**FW提供範囲**: DataReadHandlerがDataReader.read()を繰り返し呼び出す制御はFW担当。ストリーミング実現のためのread()メソッド内の実装は実装者が担当
**レビュー対象**: カスタムDataReaderクラスのread()メソッド実装
**チェック方法**:
DataReader.readAll() や List への一括格納を使っていないか。DB から取得する場合、カーソル/フェッチサイズが適切に設定されているか。ファイル読み込みの場合、BufferedReader 等で逐次処理しているか。
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
## BATCH-006: XML/アノテーションによるコンポーネント定義の整合性確認
**観点タイトル**: XML/アノテーションによるコンポーネント定義の整合性確認
**観点詳細**: バッチのコンポーネント定義（component-configuration.xml 等）が実際の実装クラスと一致しているか。未使用定義・参照切れ・プロパティ名のタイポがないか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/repository.html
**優先度**: MUST
**優先度の理由**: コンポーネント定義の誤りはアプリケーション起動失敗につながる。開発環境では動作しても本番環境の設定ファイル差異で障害が発生することもある。
**責任区分**: developer
**FW提供範囲**: コンポーネントのインスタンス化・DI注入はFWが自動実行。実装者はXML定義の正確性（クラス名・プロパティ名の一致）を担当
**レビュー対象**: component-configuration.xml等のコンポーネント定義XMLファイル（classパス・propertyタグのname属性）
**チェック方法**:
component タグの class 属性が実在するクラス名か。property タグの name 属性がクラスのフィールド名と一致するか。定義されているが参照されていないコンポーネントがないか。
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
## BATCH-007: バッチ中断時のチェックポイント・再開ポイント設計
**観点タイトル**: バッチ中断時のチェックポイント・再開ポイント設計
**観点詳細**: バッチが途中で停止した場合に、どこから再開するかのチェックポイント設計が存在するか。無設計の場合、全件再処理でデータ二重登録が発生する可能性がある。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html
**優先度**: Should
**優先度の理由**: 大量データバッチで中断再開設計がないと、障害復旧時の全件再処理でダウンタイムが長期化し、データ品質も損なわれる。
**責任区分**: developer
**FW提供範囲**: チェックポイント保存・再開機能はFWが自動提供しない。設計・実装は実装者が行う
**レビュー対象**: バッチのSQL定義（未処理レコード取得条件）、処理済みフラグ更新ロジック
**チェック方法**:
処理済みレコードのマーキング（フラグ更新・別テーブル記録）が実装されているか。再開時に未処理レコードのみを読み込む DataReader の WHERE 条件が正しいか。再実行時に二重処理が発生しない冪等性設計になっているか。
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
## BATCH-008: バッチ処理ログの詳細度・構造化確認
**観点タイトル**: バッチ処理ログの詳細度・構造化確認
**観点詳細**: バッチ開始/終了・処理件数・エラー発生をログに記録しているか。ログレベルが適切（INFO/WARN/ERROR）で、本番運用に耐えるフォーマットか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/log.html
**優先度**: Should
**優先度の理由**: ログ不足は障害時の原因究明を困難にする。過剰なDEBUGログは本番パフォーマンスに影響する。
**責任区分**: developer
**FW提供範囲**: ログ出力APIはNablarchが提供。FWレベルのアクセスログはFWが出力。業務的な処理件数ログ・統計情報の出力は実装者が担当
**レビュー対象**: 業務BatchActionクラスのログ出力コード
**チェック方法**:
バッチ開始・終了・処理件数が INFO レベルで記録されているか。業務エラーは WARN、システムエラーは ERROR レベルか。LogUtil や Nablarch のロギング API を使用しているか（System.out.println 不使用）。
**NG例**:
```
System.out.println("処理完了: " + count + "件"); // 本番ログに出ない
```
**OK例**:
```java
Logger log = LoggerFactory.getLogger(getClass());
log.logInfo("バッチ完了", "処理件数: " + count);
```
---
## BATCH-009: バッチ入力レコードのバリデーション実装確認
**観点タイトル**: バッチ入力レコードのバリデーション実装確認
**観点詳細**: バッチ入力データ（ファイル・DB）に対するバリデーションが実装されているか。バリデーションエラーレコードの扱い（スキップ・エラーファイル出力）が仕様通りか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/validation.html
**優先度**: Should
**優先度の理由**: 入力バリデーションが欠如すると、不正データがDBに登録され後続処理の障害となる。
**責任区分**: developer
**FW提供範囲**: ValidationUtil等のバリデーション実行エンジンはFWが提供。実装者はバリデーション定義（アノテーション）とvalidate()呼び出しを担当
**レビュー対象**: 業務BatchActionクラスのバリデーション呼び出しコード、入力FormクラスのValidationアノテーション
**チェック方法**:
ValidationUtil.validate() または Bean Validation アノテーションが使用されているか。バリデーションエラー発生時の処理フロー（スキップ/停止）が業務仕様と一致するか。エラーレコードはエラーファイルまたはログに記録されているか。
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
## BATCH-010: 同一バッチの同時実行防止機構の確認
**観点タイトル**: 同一バッチの同時実行防止機構の確認
**観点詳細**: 同一バッチジョブが同時に複数起動された場合に、二重処理が発生しない排他制御が実装されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/exclusive_control.html
**優先度**: Should
**優先度の理由**: 二重起動によるデータ二重処理は金融・会計系バッチでは重大なデータ不整合を引き起こす。
**責任区分**: developer
**FW提供範囲**: 排他制御機能の提供（nablarch-common-exclusivecontrol）はFW。実装者はどのリソースをいつロックするかの設計・実装を担当
**レビュー対象**: 業務BatchActionクラスの排他制御実装（ExclusiveControlManager呼び出し）
**チェック方法**:
DB ロック・ファイルロック・nablarch-common-exclusivecontrol による排他が実装されているか。ロック取得失敗時のエラーメッセージ・終了コードが適切か。バッチ終了時（正常・異常）にロックが確実に解放されるか。
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
## BATCH-011: 固定長・CSV ファイルの文字コード設定確認
**観点タイトル**: 固定長・CSV ファイルの文字コード設定確認
**観点詳細**: ファイル形式の入出力で文字コード（UTF-8/Shift-JIS 等）が明示的に指定されているか。デフォルトエンコーディングに依存していると OS・JVM 設定によって文字化けが発生する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/data_io/data_format.html
**優先度**: MUST
**優先度の理由**: エンコーディング未指定は実行環境依存の文字化けを引き起こし、本番障害・データ破損の原因となる。
**責任区分**: developer
**FW提供範囲**: フォーマット定義に基づくファイルI/Oの実際の読み書きはFWが実行。実装者はエンコーディング設定の正確性を担当
**レビュー対象**: フォーマット定義ファイル（.fmt）のtext-encoding設定
**チェック方法**:
フォーマット定義ファイル（format.fmt）に file-type と text-encoding が指定されているか。DataBindConfig 等で明示的にエンコーディングが設定されているか。
**NG例**:
```java
// エンコーディング未指定
new FileReader("input.csv") // JVM デフォルト依存
```
**OK例**:
```bash
# format.fmt
file-type: CSV
text-encoding: UTF-8
```
---
## BATCH-012: バッチ終了コードと監視ツール連携の確認
**観点タイトル**: バッチ終了コードと監視ツール連携の確認
**観点詳細**: バッチの正常終了・異常終了・警告終了で異なる終了コードが返されているか。ジョブスケジューラ（JP1 等）が正しく終了判定できる設計か確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html
**優先度**: Should
**優先度の理由**: 終了コードが常に 0（正常）だと、異常終了を監視システムが検知できず、障害が隠蔽される。
**責任区分**: developer
**FW提供範囲**: 終了コードの伝播機構はFWが提供。実装者は業務判断に基づく終了コード値の設定を担当
**レビュー対象**: バッチ終了処理のSystem.exit()呼び出し、またはBatchCommandLine終了コード設定
**チェック方法**:
正常終了: 0、業務エラー: 1〜9、システムエラー: 10〜 の区分が設計されているか。System.exit() の代わりに BatchCommandLine の終了コード機構を使っているか。ジョブスケジューラとの終了コード連携が文書化されているか。
**NG例**:
```java
catch (Exception e) {
    log.error("エラー", e);
    System.exit(0); // エラーなのに正常終了コード
}
```
**OK例**:
```
catch (SystemException e) {
    log.error("システムエラー", e);
    System.exit(10);
}
```
---
## BATCH-013: バッチ処理結果統計の出力確認
**観点タイトル**: バッチ処理結果統計の出力確認
**観点詳細**: バッチ完了後に処理件数・スキップ件数・エラー件数が記録・出力されているか。運用チームがバッチ結果を確認できるレポーティング設計か確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html
**優先度**: Should
**優先度の理由**: 統計情報がないと、バッチが実行されたか・何件処理されたかを後から確認できず、業務確認・監査対応が困難になる。
**責任区分**: developer
**FW提供範囲**: FWはフレームワークレベルの実行統計を提供するが、業務的な処理件数カウントと統計レポートの出力は実装者が担当
**レビュー対象**: 業務BatchActionクラスの件数カウント・統計ログ出力コード
**チェック方法**:
ExecutionContext から処理件数を取得して最終ログ出力しているか。エラーレコード数が明示的にカウントされているか。統計情報を専用テーブル・ファイルに永続化しているか。
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
## BATCH-014: Nablarch TestSupport を使ったバッチ単体テスト設計
**観点タイトル**: Nablarch TestSupport を使ったバッチ単体テスト設計
**観点詳細**: バッチアクションのユニットテストが NablarchTestSupport または BatchRequestTestSupport を使って作成されているか。DBUnit との組み合わせでデータ駆動テストが実現されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/index.html
**優先度**: Should
**優先度の理由**: テストのないバッチは変更時のデグレ検知ができず、本番障害リスクが高まる。
**責任区分**: developer
**FW提供範囲**: テストサポートクラス（BatchRequestTestSupport・NablarchTestSupport等）はFWが提供。テストケースの実装と正常系・異常系のカバレッジは実装者が担当
**レビュー対象**: テストクラス（BatchRequestTestSupportを継承したクラス）
**チェック方法**:
BatchRequestTestSupport を継承したテストクラスが存在するか。テストデータが Excel または YAML で外部定義されているか。正常系・異常系の両テストケースが存在するか。
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
## BATCH-015: リクエストパスとコンポーネント定義の対応確認
**観点タイトル**: リクエストパスとコンポーネント定義の対応確認
**観点詳細**: バッチ起動コマンドに渡すリクエストパス（-requestPath）がコンポーネント定義の name と一致しているか確認する。不一致だとバッチが起動しない。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/getting_started/nablarch_batch/index.html
**優先度**: MUST
**優先度の理由**: リクエストパスの不一致はバッチ起動失敗の最も基本的な原因であり、リリース直前に発覚すると影響が大きい。
**責任区分**: developer
**FW提供範囲**: リクエストパスに基づくアクション解決（DispatchHandler）はFWが実行。実装者はコンポーネント名と起動パスの完全一致を確認する
**レビュー対象**: 起動スクリプト・Jenkins設定の-requestPathパラメータ、component-configuration.xmlのコンポーネントname属性
**チェック方法**:
起動スクリプト・Jenkins ジョブ等のリクエストパスと XML のコンポーネント名が完全一致するか。ステージング環境でのバッチ起動テストが実施済みか。
**NG例**:
```
# コンポーネント名が "sampleBatchAction" なのに
java ... -requestPath=sample_batch_action  # アンダースコア形式で不一致
```
**OK例**:
```bash
java ... -requestPath=sampleBatchAction  # コンポーネント名と一致
```
---
## BATCH-016: ファイルフォーマット定義の項目網羅性・型定義確認
**観点タイトル**: ファイルフォーマット定義の項目網羅性・型定義確認
**観点詳細**: フォーマット定義ファイル（.fmt）の項目定義がファイル仕様書と完全に一致しているか。項目の型・桁数・必須/任意が正確に定義されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/data_io/data_format.html
**優先度**: MUST
**優先度の理由**: フォーマット定義の誤りはパース失敗・データ欠損・文字化けに直結し、バッチ全件失敗の原因となる。
**責任区分**: developer
**FW提供範囲**: フォーマット定義に基づくデータパース処理はFWが実行。実装者はフォーマット定義の正確性（仕様書との一致）を担当
**レビュー対象**: フォーマット定義ファイル（.fmt）の項目定義（型・桁数・必須設定）
**チェック方法**:
フォーマット定義の項目数・順序がファイル仕様書と一致するか。固定長ファイルの場合、各項目のバイト数の合計がレコード長と一致するか。必須項目に required 制約が設定されているか。
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
## BATCH-017: バッチ処理時間の設計目標値と実測値の妥当性確認
**観点タイトル**: バッチ処理時間の設計目標値と実測値の妥当性確認
**観点詳細**: バッチの処理時間が業務 SLA（処理ウィンドウ）に収まる設計になっているか。ボトルネック（DB I/O・外部連携）の分析と改善策が検討されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html
**優先度**: Should
**優先度の理由**: SLA 未達のバッチは翌業務日の開始に影響し、業務停止を引き起こす。設計段階での性能見積もりが重要。
**責任区分**: developer
**FW提供範囲**: バッチ実行制御自体はFWが担当。性能設計・SQLチューニング・処理ウィンドウ見積もりは実装者が担当
**レビュー対象**: 業務BatchActionクラス・SQLファイル・パフォーマンステスト結果
**チェック方法**:
処理件数 × 1件あたり処理時間の概算が処理ウィンドウ内に収まるか。DB の SELECT/INSERT に適切なインデックスが設定されているか。外部 API 呼び出しがある場合、タイムアウト・サーキットブレーカーが設定されているか。
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
---
## BATCH-018: カスタムDataReader実装時のclose()業務後処理確認
**観点タイトル**: カスタムDataReader実装時のclose()業務後処理確認
**観点詳細**: FWはDataReader.open()とclose()の呼び出しを自動管理（DataReadHandler）するため、実装者がopen/closeを呼ぶコードを書く必要はない。ただし、カスタムDataReaderを実装した場合、closeメソッド内に業務的後処理（外部リソース解放・キャッシュクリア・一時ファイル削除等）が正しく実装されているか確認する。
**根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html
**優先度**: Should
**優先度の理由**: カスタムDataReader内でFW管理外のリソース（外部API接続・独自キャッシュ等）を使用している場合、close()に後処理が抜けるとリソースリークが発生する。
**責任区分**: developer
**FW提供範囲**: DataReader.open()とclose()の呼び出し自体はFW（DataReadHandler）が自動管理。実装者がopen()/close()を呼ぶコードを外から書く必要はない。カスタムDataReaderを実装した場合、close()メソッドの中身（業務後処理）の実装は実装者が担当
**レビュー対象**: カスタムDataReaderクラスのclose()メソッド実装（プロジェクトでカスタムDataReaderを実装している場合のみ）
**チェック方法**:
カスタムDataReaderを実装している場合のみ確認（FW標準DataReaderのみ使用の場合は対象外）。close()メソッドに業務的後処理（外部リソースの解放・一時ファイル削除等）が実装されているか。FW管理外のリソースを開いた場合はclose()内で確実にリリースしているか。
**NG例**:
```java
// カスタムDataReaderでcloseの後処理を忘れた例
public void close(ExecutionContext context) {
    // 何もしない（外部APIの接続が残ったまま）
}
```
**OK例**:
```java
// カスタムDataReaderで適切な後処理を実装した例
public void close(ExecutionContext context) {
    if (externalApiClient != null) {
        externalApiClient.disconnect(); // FW管理外のリソースを解放
    }
    if (tempFile != null && tempFile.exists()) {
        tempFile.delete(); // 一時ファイルのクリーンアップ
    }
}
```
