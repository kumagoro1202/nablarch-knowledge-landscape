# Nablarch バッチ処理 レビュー観点プリセット
対象: Nablarch バッチアーキテクチャ（nablarch-fw-batch）

## 観点一覧

| ID | タイトル | 優先度 | 観点概要 | 根拠 |
|----|---------|:------:|---------|------|
| [BATCH-003](#batch-003) | コミット単位・トランザクション境界の設計妥当性 | **MUST** | commitIntervalがコンポーネント定義で正しく設定されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html) |
| [BATCH-006](#batch-006) | XML/アノテーションによるコンポーネント定義の整合性確認 | **MUST** | コンポーネント定義が実装クラスと一致し、未使用定義・参照切れがないか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/repository.html) |
| [BATCH-008](#batch-008) | バッチ処理ログの詳細度・構造化確認 | **Should** | 開始/終了・処理件数・エラーをINFO/WARN/ERRORレベルで記録しているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/log.html) |
| [BATCH-012](#batch-012) | バッチ終了コードと監視ツール連携の確認 | **Should** | 正常/異常/警告で異なる終了コードを返しジョブスケジューラが正しく判定できるか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html) |
| [BATCH-013](#batch-013) | バッチ処理結果統計の出力確認 | **Should** | 処理件数・スキップ件数・エラー件数が記録・出力されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html) |
| [BATCH-014](#batch-014) | Nablarch TestSupport を使ったバッチ単体テスト設計 | **Should** | BatchRequestTestSupportを使ったデータ駆動テストが実装されているか | [公式](https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/index.html) |
| [BATCH-015](#batch-015) | リクエストパスとコンポーネント定義の対応確認 | **MUST** | 起動コマンドの`-requestPath`がコンポーネント定義のnameと完全一致するか | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/getting_started/nablarch_batch/index.html) |

---

## 各観点詳細

### BATCH-003
**コミット単位・トランザクション境界の設計妥当性**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのバッチ機能を使う（BatchActionを実装する） |
| レビュー対象 | `component-configuration.xml` の`commitInterval`設定値 |
| FW提供範囲 | トランザクションの開始・コミット・ロールバック自体はトランザクション制御ハンドラが自動実行。実装者は`commitInterval`の値設定のみ担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html) |
| 優先度理由 | トランザクション境界の誤設定はデータ整合性破壊・ロールバック時の全件再処理・メモリ枯渇を引き起こす |
| チェック方法 | `commitInterval`が業務要件（データ量・リカバリ要件）に基づいて設定されているか。1コミット間のレコード数が過大（10,000件超）になっていないか。コミット失敗時のリトライ戦略が定義されているか |

**NG例**
```xml
<!-- commitInterval 未設定 → デフォルト値依存 -->
<component name="batchAction" class="com.example.SampleBatchAction"/>
```

**OK例**
```xml
<component name="batchAction" class="com.example.SampleBatchAction">
    <property name="commitInterval" value="100"/>
</component>
```

---

### BATCH-006
**XML/アノテーションによるコンポーネント定義の整合性確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのバッチ機能を使う（BatchActionを実装する） |
| レビュー対象 | `component-configuration.xml`等のコンポーネント定義XMLファイル（classパス・propertyタグのname属性） |
| FW提供範囲 | コンポーネントのインスタンス化・DI注入はFWが自動実行。実装者はXML定義の正確性（クラス名・プロパティ名の一致）を担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/repository.html) |
| 優先度理由 | コンポーネント定義の誤りはアプリケーション起動失敗につながる。開発環境では動作しても本番環境の設定ファイル差異で障害が発生することもある |
| チェック方法 | `component`タグの`class`属性が実在するクラス名か。`property`タグの`name`属性がクラスのフィールド名と一致するか。定義されているが参照されていないコンポーネントがないか |

**NG例**
```xml
<component name="batchAction" class="com.example.SampleBatchActoin"/> <!-- タイポ -->
    <property name="commitIntervel" value="100"/>  <!-- タイポ -->
```

**OK例**
```xml
<component name="batchAction" class="com.example.SampleBatchAction">
    <property name="commitInterval" value="100"/>
</component>
```

---

### BATCH-008
**バッチ処理ログの詳細度・構造化確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | Nablarchのバッチ機能を使う（BatchActionを実装する） |
| レビュー対象 | 業務BatchActionクラスのログ出力コード |
| FW提供範囲 | ログ出力APIはNablarchが提供。FWレベルのアクセスログはFWが出力。業務的な処理件数ログ・統計情報の出力は実装者が担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/log.html) |
| 優先度理由 | ログ不足は障害時の原因究明を困難にする。過剰なDEBUGログは本番パフォーマンスに影響する |
| チェック方法 | バッチ開始・終了・処理件数がINFOレベルで記録されているか。業務エラーはWARN、システムエラーはERRORレベルか。`LogUtil`やNablarchのロギングAPIを使用しているか（`System.out.println`不使用） |

**NG例**
```
System.out.println("処理完了: " + count + "件"); // 本番ログに出ない
```

**OK例**
```java
Logger log = LoggerFactory.getLogger(getClass());
log.logInfo("バッチ完了", "処理件数: " + count);
```

---

### BATCH-012
**バッチ終了コードと監視ツール連携の確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | Nablarchのバッチ機能を使う（BatchActionを実装する） |
| レビュー対象 | バッチ終了処理の`System.exit()`呼び出し、またはBatchCommandLine終了コード設定 |
| FW提供範囲 | 終了コードの伝播機構はFWが提供。実装者は業務判断に基づく終了コード値の設定を担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html) |
| 優先度理由 | 終了コードが常に0（正常）だと、異常終了を監視システムが検知できず、障害が隠蔽される |
| チェック方法 | 正常終了:0、業務エラー:1〜9、システムエラー:10〜 の区分が設計されているか。`System.exit()`の代わりにBatchCommandLineの終了コード機構を使っているか。ジョブスケジューラとの終了コード連携が文書化されているか |

**NG例**
```java
catch (Exception e) {
    log.error("エラー", e);
    System.exit(0); // エラーなのに正常終了コード
}
```

**OK例**
```
catch (SystemException e) {
    log.error("システムエラー", e);
    System.exit(10);
}
```

---

### BATCH-013
**バッチ処理結果統計の出力確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | Nablarchのバッチ機能を使う（BatchActionを実装する） |
| レビュー対象 | 業務BatchActionクラスの件数カウント・統計ログ出力コード |
| FW提供範囲 | FWはフレームワークレベルの実行統計を提供するが、業務的な処理件数カウントと統計レポートの出力は実装者が担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html) |
| 優先度理由 | 統計情報がないと、バッチが実行されたか・何件処理されたかを後から確認できず、業務確認・監査対応が困難になる |
| チェック方法 | `ExecutionContext`から処理件数を取得して最終ログ出力しているか。エラーレコード数が明示的にカウントされているか。統計情報を専用テーブル・ファイルに永続化しているか |

**NG例**
```java
// 件数カウントなし
for (Record r : records) {
    process(r);
}
log.info("バッチ完了"); // 件数不明
```

**OK例**
```java
int processed = 0, skipped = 0;
for (Record r : records) {
    try { process(r); processed++; }
    catch (SkipException e) { skipped++; }
}
log.info("完了: 処理=" + processed + " スキップ=" + skipped);
```

---

### BATCH-014
**Nablarch TestSupport を使ったバッチ単体テスト設計**

| 項目 | 内容 |
|------|------|
| 優先度 | **Should** |
| 普遍性 | universal |
| 適用条件 | Nablarchのバッチ機能を使う（BatchActionを実装する） |
| レビュー対象 | テストクラス（`BatchRequestTestSupport`を継承したクラス） |
| FW提供範囲 | テストサポートクラス（`BatchRequestTestSupport`・`NablarchTestSupport`等）はFWが提供。テストケースの実装と正常系・異常系のカバレッジは実装者が担当 |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/development_tools/testing_framework/guide/development_guide/05_UnitTestGuide/02_RequestUnitTest/index.html) |
| 優先度理由 | テストのないバッチは変更時のデグレ検知ができず、本番障害リスクが高まる |
| チェック方法 | `BatchRequestTestSupport`を継承したテストクラスが存在するか。テストデータがExcelまたはYAMLで外部定義されているか。正常系・異常系の両テストケースが存在するか |

**NG例**
```java
// バッチアクションのテストなし（テストクラス自体が存在しない）
```

**OK例**
```java
public class SampleBatchActionTest extends BatchRequestTestSupport {
    @Test
    public void 正常終了() { execute("testCases/normal.xlsx"); }
}
```

---

### BATCH-015
**リクエストパスとコンポーネント定義の対応確認**

| 項目 | 内容 |
|------|------|
| 優先度 | **MUST** |
| 普遍性 | universal |
| 適用条件 | Nablarchのバッチ機能を使う（BatchActionを実装する） |
| レビュー対象 | 起動スクリプト・Jenkins設定の`-requestPath`パラメータ、`component-configuration.xml`のコンポーネント`name`属性 |
| FW提供範囲 | リクエストパスに基づくアクション解決（`DispatchHandler`）はFWが実行。実装者はコンポーネント名と起動パスの完全一致を確認する |
| 根拠URL | [公式](https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/getting_started/nablarch_batch/index.html) |
| 優先度理由 | リクエストパスの不一致はバッチ起動失敗の最も基本的な原因であり、リリース直前に発覚すると影響が大きい |
| チェック方法 | 起動スクリプト・Jenkinsジョブ等のリクエストパスとXMLのコンポーネント名が完全一致するか。ステージング環境でのバッチ起動テストが実施済みか |

**NG例**
```
# コンポーネント名が "sampleBatchAction" なのに
java ... -requestPath=sample_batch_action  # アンダースコア形式で不一致
```

**OK例**
```bash
java ... -requestPath=sampleBatchAction  # コンポーネント名と一致
```
