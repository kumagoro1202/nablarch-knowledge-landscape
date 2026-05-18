# Nablarch 公式コンテンツ一覧

調査日: 2026-05-18

調査起点: https://fintan.jp/page/1868/

---

## Nablarch GitHub Organization

出典: https://github.com/nablarch

調査コマンド: `gh repo list nablarch --limit 200`

合計リポジトリ数: 120+（2026-05-18時点）

---

### nablarch（メイン情報リポジトリ）

| 項目 | 内容 |
|------|------|
| URL | https://github.com/nablarch/nablarch |
| 種別 | フレームワーク全体案内 / モジュール一覧 |
| 対象読者 | 全開発者 |
| 最終更新 | 2026-05（⭐44） |
| 役割サマリ | Nablarchフレームワーク全モジュールの一覧・説明・リンク集。IssueはJIRA連携。 |

---

### Nablarch Core モジュール群

| リポジトリ名 | URL | 最終更新 | 役割サマリ |
|------------|-----|---------|----------|
| nablarch-core | https://github.com/nablarch/nablarch-core | 2026-05 | フレームワーク中核実装。DI・ハンドラキュー基盤を提供 |
| nablarch-core-beans | https://github.com/nablarch/nablarch-core-beans | 2025-03 | JavaBeansユーティリティ（プロパティコピー等） |
| nablarch-core-repository | https://github.com/nablarch/nablarch-core-repository | 2024-09 | 設定管理（SystemRepository・ObjectLoader） |
| nablarch-core-transaction | https://github.com/nablarch/nablarch-core-transaction | 2024-09 | トランザクション管理抽象化 |
| nablarch-core-jdbc | https://github.com/nablarch/nablarch-core-jdbc | 2024-09 | JDBCユーティリティ・Universal DAO基盤 |
| nablarch-core-message | https://github.com/nablarch/nablarch-core-message | 2024-09 | メッセージ管理（エラーメッセージ・警告等） |
| nablarch-core-validation | https://github.com/nablarch/nablarch-core-validation | 2020-10 | 入力値バリデーション（独自アノテーション方式） |
| nablarch-core-validation-ee | https://github.com/nablarch/nablarch-core-validation-ee | 2023-09 | Jakarta Bean Validation準拠バリデーション |
| nablarch-core-dataformat | https://github.com/nablarch/nablarch-core-dataformat | 2025-03 | ファイル管理（固定長・CSV・JSON・XML等フォーマット処理） |
| nablarch-core-applog | https://github.com/nablarch/nablarch-core-applog | 2024-09 | ロギング機能 |

---

### Nablarch Framework（実行基盤）モジュール群

| リポジトリ名 | URL | 最終更新 | 役割サマリ |
|------------|-----|---------|----------|
| nablarch-fw | https://github.com/nablarch/nablarch-fw | 2021-03 | ハンドラキュー基盤実装 |
| nablarch-fw-web | https://github.com/nablarch/nablarch-fw-web | 2025-03 | Webアプリケーション基盤（サーブレット連携・ハンドラ群） |
| nablarch-fw-web-tag | https://github.com/nablarch/nablarch-fw-web-tag | 2024-09 | Jakarta Server Pages カスタムタグライブラリ |
| nablarch-fw-web-hotdeploy | https://github.com/nablarch/nablarch-fw-web-hotdeploy | 2020-10 | Webアプリホットデプロイ機能 |
| nablarch-fw-web-extension | https://github.com/nablarch/nablarch-fw-web-extension | 2020-10 | ファイルアップロード・ダウンロード機能 |
| nablarch-fw-web-dbstore | https://github.com/nablarch/nablarch-fw-web-dbstore | 2020-10 | DBセッションストア実装 |
| nablarch-fw-web-doublesubmit-jdbc | https://github.com/nablarch/nablarch-fw-web-doublesubmit-jdbc | 2020-04 | 二重サブミット防止トークン管理（JDBC実装） |
| nablarch-fw-batch | https://github.com/nablarch/nablarch-fw-batch | 2025-03 | バッチ処理基本ハンドラ・アクション群 |
| nablarch-fw-batch-ee | https://github.com/nablarch/nablarch-fw-batch-ee | 2020-10 | Jakarta Batch準拠バッチ処理実装 |
| nablarch-fw-jaxrs | https://github.com/nablarch/nablarch-fw-jaxrs | 2025-03 | RESTful Web Services（JAX-RS）対応基盤 |
| nablarch-fw-standalone | https://github.com/nablarch/nablarch-fw-standalone | 2022-03 | スタンドアロンアプリケーション起動基盤 |
| nablarch-fw-messaging | https://github.com/nablarch/nablarch-fw-messaging | 2024-09 | メッセージング基盤・プロトコル定義 |
| nablarch-fw-messaging-http | https://github.com/nablarch/nablarch-fw-messaging-http | 2020-10 | HTTPメッセージング実装 |
| nablarch-fw-messaging-mom | https://github.com/nablarch/nablarch-fw-messaging-mom | 2024-03 | MOM（Message Oriented Middleware）メッセージング |
| nablarch-fw-scoped-dicontainer | https://github.com/nablarch/nablarch-fw-scoped-dicontainer | 2020-09 | スコープ付きDIコンテナ |

---

### Nablarch Common Component（共通部品）モジュール群

| リポジトリ名 | URL | 最終更新 | 役割サマリ |
|------------|-----|---------|----------|
| nablarch-common-dao | https://github.com/nablarch/nablarch-common-dao | 2025-03 | EntityクラスからSQL自動生成（Universal DAO） |
| nablarch-common-jdbc | https://github.com/nablarch/nablarch-common-jdbc | 2023-09 | DB接続ハンドラ |
| nablarch-common-idgenerator | https://github.com/nablarch/nablarch-common-idgenerator | 2020-10 | ID生成基盤 |
| nablarch-common-idgenerator-jdbc | https://github.com/nablarch/nablarch-common-idgenerator-jdbc | 2025-03 | JDBC実装ID生成 |
| nablarch-common-auth | https://github.com/nablarch/nablarch-common-auth | 2025-03 | 認証機能 |
| nablarch-common-auth-jdbc | https://github.com/nablarch/nablarch-common-auth-jdbc | 2020-10 | JDBC実装認証 |
| nablarch-common-auth-session | https://github.com/nablarch/nablarch-common-auth-session | 2023-03 | セッションベース認証実装 |
| nablarch-common-code | https://github.com/nablarch/nablarch-common-code | 2020-10 | コードマスタ管理機能 |
| nablarch-common-code-jdbc | https://github.com/nablarch/nablarch-common-code-jdbc | 2020-10 | JDBC実装コードマスタ |
| nablarch-common-date | https://github.com/nablarch/nablarch-common-date | 2020-10 | 日付管理機能 |
| nablarch-common-encryption | https://github.com/nablarch/nablarch-common-encryption | 2020-10 | 暗号化機能 |
| nablarch-common-exclusivecontrol | https://github.com/nablarch/nablarch-common-exclusivecontrol | 2020-10 | 排他制御機能 |
| nablarch-common-exclusivecontrol-jdbc | https://github.com/nablarch/nablarch-common-exclusivecontrol-jdbc | 2020-10 | JDBC実装排他制御 |
| nablarch-common-databind | https://github.com/nablarch/nablarch-common-databind | 2025-03 | データバインド機能（CSV・Fixed等） |

---

### Nablarch Adaptor（外部連携アダプタ）モジュール群

| リポジトリ名 | URL | 最終更新 | 役割サマリ |
|------------|-----|---------|----------|
| nablarch-doma-adaptor | https://github.com/nablarch/nablarch-doma-adaptor | 2020-10 | Doma ORマッパー連携アダプタ |
| nablarch-lettuce-adaptor | https://github.com/nablarch/nablarch-lettuce-adaptor | 2024-09 | Lettuce（Redis）連携アダプタ |
| nablarch-micrometer-adaptor | https://github.com/nablarch/nablarch-micrometer-adaptor | 2024-09 | Micrometerメトリクス連携アダプタ |
| nablarch-jaxrs-adaptor | https://github.com/nablarch/nablarch-jaxrs-adaptor | 2025-03 | JAX-RS連携アダプタ |
| nablarch-router-adaptor | https://github.com/nablarch/nablarch-router-adaptor | 2025-03 | URLルーティングアダプタ |
| nablarch-openapi-generator | https://github.com/nablarch/nablarch-openapi-generator | 2025-03 | OpenAPI仕様からコード生成ツール |
| nablarch-jsr310-adaptor | https://github.com/nablarch/nablarch-jsr310-adaptor | 2020-10 | JSR-310（Java Time API）連携アダプタ |
| nablarch-jboss-logging-adaptor | https://github.com/nablarch/nablarch-jboss-logging-adaptor | 2020-10 | JBossログ連携アダプタ |
| nablarch-slf4j-adaptor | https://github.com/nablarch/nablarch-slf4j-adaptor | 2020-10 | SLF4J連携アダプタ（Nablarch→SLF4J） |
| slf4j-nablarch-adaptor | https://github.com/nablarch/slf4j-nablarch-adaptor | 2021-09 | SLF4J連携アダプタ（SLF4J→Nablarch） |
| nablarch-log4j-adaptor | https://github.com/nablarch/nablarch-log4j-adaptor | 2023-01 | Log4j連携アダプタ |
| nablarch-wmq-adaptor | https://github.com/nablarch/nablarch-wmq-adaptor | 2024-03 | IBM MQ（WebSphere MQ）連携アダプタ |
| nablarch-web-thymeleaf-adaptor | https://github.com/nablarch/nablarch-web-thymeleaf-adaptor | 2018-04 | Thymeleafテンプレートエンジン連携アダプタ（Web） |
| nablarch-mail-sender-thymeleaf-adaptor | https://github.com/nablarch/nablarch-mail-sender-thymeleaf-adaptor | 2018-04 | Thymeleafメール送信アダプタ |
| nablarch-mail-sender-velocity-adaptor | https://github.com/nablarch/nablarch-mail-sender-velocity-adaptor | 2020-09 | Velocityメール送信アダプタ |
| nablarch-mail-sender-freemarker-adaptor | https://github.com/nablarch/nablarch-mail-sender-freemarker-adaptor | 2018-04 | FreeMarkerメール送信アダプタ |

---

### Nablarch Example Applications（実装例・サンプル）

| リポジトリ名 | URL | 最終更新 | スター | 役割サマリ |
|------------|-----|---------|------|----------|
| nablarch-example-web | https://github.com/nablarch/nablarch-example-web | 2026-03 | ⭐5 | Webアプリ（JSPカスタムタグ方式）のサンプル実装 |
| nablarch-example-thymeleaf-web | https://github.com/nablarch/nablarch-example-thymeleaf-web | 2025-03 | - | ThymeleafベースWebアプリのサンプル実装 |
| nablarch-example-rest | https://github.com/nablarch/nablarch-example-rest | 2025-03 | - | RESTful APIサンプル実装 |
| nablarch-example-batch | https://github.com/nablarch/nablarch-example-batch | 2025-10 | ⭐2 | バッチアプリサンプル実装 |
| nablarch-example-batch-ee | https://github.com/nablarch/nablarch-example-batch-ee | 2025-03 | - | Jakarta Batchサンプル実装 |
| nablarch-example-db-queue | https://github.com/nablarch/nablarch-example-db-queue | 2025-03 | - | DBキューを使った非同期処理サンプル |
| nablarch-example-http-messaging | https://github.com/nablarch/nablarch-example-http-messaging | 2025-03 | - | HTTPメッセージング受信サンプル |
| nablarch-example-http-messaging-send | https://github.com/nablarch/nablarch-example-http-messaging-send | 2025-03 | - | HTTPメッセージング送信サンプル |
| nablarch-example-mom-delayed-receive | https://github.com/nablarch/nablarch-example-mom-delayed-receive | 2025-03 | - | MOM遅延受信サンプル |
| nablarch-example-mom-delayed-send | https://github.com/nablarch/nablarch-example-mom-delayed-send | 2025-03 | - | MOM遅延送信サンプル |
| nablarch-example-mom-sync-receive | https://github.com/nablarch/nablarch-example-mom-sync-receive | 2025-03 | - | MOM同期受信サンプル |
| nablarch-example-mom-sync-send-batch | https://github.com/nablarch/nablarch-example-mom-sync-send-batch | 2025-03 | - | MOM同期送信バッチサンプル |
| nablarch-example-mom-testing-common | https://github.com/nablarch/nablarch-example-mom-testing-common | 2025-03 | - | MOMメッセージングテスト共通部品 |
| nablarch-example-workflow | https://github.com/nablarch/nablarch-example-workflow | 2025-05 | - | ワークフロー機能サンプル |
| nablarch-biz-sample-all | https://github.com/nablarch/nablarch-biz-sample-all | 2025-03 | ⭐1 | ビジネスサンプル実装集（画面・バッチ・REST等） |

---

### ドキュメント

| リポジトリ名 | URL | 最終更新 | スター | 役割サマリ |
|------------|-----|---------|------|----------|
| nablarch-document | https://github.com/nablarch/nablarch-document | 2026-05 | ⭐2 | 公式ドキュメント（Sphinx形式）ソース。公開先: https://nablarch.github.io/docs/LATEST/doc/ |
| nablarch.github.io | https://github.com/nablarch/nablarch.github.io | 2025-05 | - | GitHub Pagesサイト（ドキュメント公開基盤） |

---

### テストフレームワーク

| リポジトリ名 | URL | 最終更新 | 役割サマリ |
|------------|-----|---------|----------|
| nablarch-testing | https://github.com/nablarch/nablarch-testing | 2024-09 | Nablarchテストフレームワーク本体 |
| nablarch-testing-junit5 | https://github.com/nablarch/nablarch-testing-junit5 | 2022-03 | JUnit 5対応テスト支援 |
| nablarch-testing-jetty6 | https://github.com/nablarch/nablarch-testing-jetty6 | 2024-10 | Jetty 6組み込みテスト対応 |
| nablarch-testing-jetty9 | https://github.com/nablarch/nablarch-testing-jetty9 | 2024-10 | Jetty 9組み込みテスト対応 |
| nablarch-testing-jetty12 | https://github.com/nablarch/nablarch-testing-jetty12 | 2024-09 | Jetty 12組み込みテスト対応 |
| nablarch-testing-rest | https://github.com/nablarch/nablarch-testing-rest | 2024-09 | RESTful APIテスト支援ライブラリ |
| nablarch-test-support | https://github.com/nablarch/nablarch-test-support | 2022-03 | テスト共通サポートライブラリ |
| nablarch-test-support-hereis | https://github.com/nablarch/nablarch-test-support-hereis | 2017-02 | ヒアドキュメント形式テストデータ記述支援 |
| nablarch-integration-test | https://github.com/nablarch/nablarch-integration-test | 2020-10 | 結合テスト支援ライブラリ |

---

### 開発ツール・プロジェクト基盤

| リポジトリ名 | URL | 最終更新 | スター | 役割サマリ |
|------------|-----|---------|------|----------|
| nablarch-single-module-archetype | https://github.com/nablarch/nablarch-single-module-archetype | 2025-03 | - | シングルモジュールプロジェクト雛形（Mavenアーキタイプ） |
| nablarch-profiles | https://github.com/nablarch/nablarch-profiles | 2025-03 | - | Mavenプロファイル設定集 |
| nablarch-parent | https://github.com/nablarch/nablarch-parent | 2025-03 | - | Maven親POM |
| nablarch-default-configuration | https://github.com/nablarch/nablarch-default-configuration | 2025-03 | - | デフォルト設定ファイル集 |
| nablarch-plugins-bundle | https://github.com/nablarch/nablarch-plugins-bundle | 2025-05 | - | プラグインバンドル |
| nablarch-ui-development-template | https://github.com/nablarch/nablarch-ui-development-template | 2025-05 | - | UI開発テンプレート（JavaScript/CSS基盤） |
| nablarch-openapi-generator | https://github.com/nablarch/nablarch-openapi-generator | 2025-03 | - | OpenAPI仕様からNablarch準拠コード生成ツール |
| nablarch-unpublished-api-checker | https://github.com/nablarch/nablarch-unpublished-api-checker | 2025-03 | - | 非公開API使用チェッカー（Maven Plugin） |
| nablarch-unpublished-api-checker-findbugs | https://github.com/nablarch/nablarch-unpublished-api-checker-findbugs | 2024-10 | - | FindBugsベース非公開APIチェッカー |
| nablarch-intellij-plugin | https://github.com/nablarch/nablarch-intellij-plugin | 2020-10 | ⭐1 | IntelliJ IDEA向けNablarchサポートプラグイン |
| nablarch-gradle-plugin | https://github.com/nablarch/nablarch-gradle-plugin | 2022-03 | ⭐1 | Gradle向けNablarchプラグイン |
| sql-executor | https://github.com/nablarch/sql-executor | 2025-03 | ⭐1 | SQL実行ユーティリティ |
| nablarch-etl | https://github.com/nablarch/nablarch-etl | 2025-12 | ⭐2 | ETL（Extract-Transform-Load）処理フレームワーク |
| nablarch-etl-maven-plugin | https://github.com/nablarch/nablarch-etl-maven-plugin | 2025-05 | - | ETL用Mavenプラグイン |
| nablarch-etl-designer | https://github.com/nablarch/nablarch-etl-designer | 2024-10 | - | ETLジョブ設計ツール |
| nablarch-workflow | https://github.com/nablarch/nablarch-workflow | 2024-10 | - | ワークフローエンジン |
| nablarch-workflow-tool | https://github.com/nablarch/nablarch-workflow-tool | 2024-10 | - | ワークフロー定義ツール |
| nablarch-mail-sender | https://github.com/nablarch/nablarch-mail-sender | 2022-03 | - | メール送信機能 |
| nablarch-messaging-simulator | https://github.com/nablarch/nablarch-messaging-simulator | 2024-10 | - | メッセージングシミュレータ（テスト用） |
| nablarch-statistics-report | https://github.com/nablarch/nablarch-statistics-report | 2024-10 | - | パフォーマンス統計レポート生成ツール |
| nablarch-smime-integration | https://github.com/nablarch/nablarch-smime-integration | 2024-10 | - | S/MIME（署名付きメール）統合機能 |
| nablarch-report | https://github.com/nablarch/nablarch-report | 2023-01 | - | レポート生成機能 |
| nablarch-report-sample | https://github.com/nablarch/nablarch-report-sample | 2023-01 | - | レポート生成サンプル |
| nablarch-git-sync-script | https://github.com/nablarch/nablarch-git-sync-script | 2023-01 | ⭐3 | ローカル・リモートリポジトリ同期スクリプト |
| nablarch-ci-script | https://github.com/nablarch/nablarch-ci-script | 2023-01 | - | CI用スクリプト集 |
| nablarch-module-version | https://github.com/nablarch/nablarch-module-version | 2023-01 | - | モジュールバージョン管理ツール |
| nablarch-toolbox | https://github.com/nablarch/nablarch-toolbox | 2020-10 | - | 開発支援ツールボックス |
| nablarch-backward-compatibility | https://github.com/nablarch/nablarch-backward-compatibility | 2018-04 | - | 後方互換性維持ライブラリ |
| developer-environment | https://github.com/nablarch/developer-environment | 2023-07 | - | 開発環境セットアップ設定集 |

---

### AI・ナレッジ関連

| リポジトリ名 | URL | 最終更新 | 役割サマリ |
|------------|-----|---------|----------|
| nabledge | https://github.com/nablarch/nabledge | 2026-03 | AIエージェント向けNablarchナレッジベース（コード解析・ドキュメント検索・影響調査機能） |
| nabledge-dev | https://github.com/nablarch/nabledge-dev | 2026-05 | Nabledge開発用リポジトリ |
| nabledge-demo | https://github.com/nablarch/nabledge-demo | 2026-02 | Nabledgeデモ環境 |
| nablarch-tools-for-ai | https://github.com/nablarch/nablarch-tools-for-ai | 2025-09 | AIエージェント向けNablarch開発支援ツール群 |

---

## Fintan-contents Organization（Nablarch関連リポジトリ）

出典: https://github.com/Fintan-contents

調査コマンド: `gh repo list Fintan-contents --limit 100`

Nablarch関連リポジトリを以下に整理する（Fintan-contentsは全般的な開発ノウハウも保有するため、Nablarch直接関連のもののみ抜粋）。

### nablarch-system-development-guide

| 項目 | 内容 |
|------|------|
| URL | https://github.com/Fintan-contents/nablarch-system-development-guide |
| 種別 | 実装ガイド / 開発プロセスガイド |
| 対象読者 | Nablarchを使うシステム開発者全般（アーキテクト・PG） |
| 最終更新 | 2026-04（⭐9） |
| 役割サマリ | 開発開始前・開発中に何をすべきか・何を参照すべきかを示すガイド。サンプルプロジェクト（Web/バッチ/REST）付き。設計書とソースコードの対応も確認可能。 |

### nablarch-training

| 項目 | 内容 |
|------|------|
| URL | https://github.com/Fintan-contents/nablarch-training |
| 種別 | トレーニング教材 |
| 対象読者 | Nablarch学習者・新規参画エンジニア |
| 最終更新 | 2026-03（⭐1） |
| 役割サマリ | Nablarchを使ったシステム開発を学ぶためのトレーニングコンテンツ。 |

### nablarch-contents-resources

| 項目 | 内容 |
|------|------|
| URL | https://github.com/Fintan-contents/nablarch-contents-resources |
| 種別 | コンテンツリソース（画像・補助素材等） |
| 対象読者 | Nablarchコンテンツ制作者 |
| 最終更新 | 2024-10 |
| 役割サマリ | Nablarchコンテンツ用の補助リソース（Fintan記事等で使用する画像素材等）。 |

---

## 補足: nablarch-development-standards について

タスク定義では `nablarch/nablarch-development-standards` リポジトリの調査が指定されていたが、**2026-05-18時点でこのリポジトリは nablarch GitHub Organization に存在しない**。

- `gh repo list nablarch --limit 200` で 120+ リポジトリを確認したが、development-standards という名称のリポジトリはなし
- `gh api repos/nablarch/nablarch-development-standards` も 404 応答

開発標準に相当するコンテンツは以下に分散して提供されていると推定される:
- **Fintan-contents/nablarch-system-development-guide** — システム開発ガイド（プロセス・成果物標準）
- **nablarch-document** — フレームワーク公式ドキュメント（実装ガイド）
- **nablarch-unpublished-api-checker** — コーディング標準準拠チェックツール

---

## 調査サマリ

| 区分 | 件数 |
|------|------|
| nablarch GitHub Org 総リポジトリ数 | 120+ |
| Nablarch Core モジュール | 10 |
| Nablarch Framework（実行基盤） | 15 |
| Nablarch Common Component | 14 |
| Nablarch Adaptor | 16 |
| Example Applications | 15 |
| ドキュメント | 2 |
| テストフレームワーク | 9 |
| 開発ツール・プロジェクト基盤 | 27 |
| AI・ナレッジ関連 | 4 |
| Fintan-contents Nablarch関連 | 3 |
