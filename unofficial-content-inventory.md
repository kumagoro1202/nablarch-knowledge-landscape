# Nablarch 非公式コンテンツ一覧

調査日: 2026-05-18

調査範囲: Qiita・Zenn・Fintan（公式枠外）・ブログ/note/個人サイト・SpeakerDeck・YouTube・X(Twitter)

---

## Qiita

Qiitaのnablarchタグは31記事（2026-05-18時点）。大多数はTIS社員（@KO_YAmajun, @charon, @heiwa）による実装ノウハウ記事。第三者個人エンジニアによる記事は @kirin1218 が中心。

| タイトル | URL | 著者 | 公開日 | 要旨 | 想定読者 | 信頼性所見 |
|---------|-----|------|--------|------|---------|----------|
| Javaのフレームワーク Nablarch を使ってみる [ウェブアプリケーション編] | https://qiita.com/heiwa/items/0a09322e93cb3bfe32cb | @heiwa (TIS) | 2017-01-02 | ブランクプロジェクトからWebアプリを構築する入門チュートリアル。MavenでEntity設計・ActionクラスをJSPと組み合わせて実装。「サクサク作れる」と評価 | Nablarch入門者 | 高: TIS社員による実務ベース記事 |
| github上のnablarch-exampleプロジェクトをEclipseに取り込む | https://qiita.com/KO_YAmajun/items/1014d4ea899a1164463c | @KO_YAmajun (TIS) | 2017-03-12 | nablarch-example-webをGitHubからEclipseに取り込み、Mavenビルドするまでの手順 | Nablarch入門者・Eclipse利用者 | 高: TIS社員による実装解説 |
| Nablarchのコンテナ用アーキタイプを使って、RESTful Webサービスを試す | https://qiita.com/charon/items/91e04b713570a81ca108 | @charon (TIS) | 2020-11-11 | Nablarch 5u18のコンテナ用アーキタイプでRESTfulプロジェクトを構築。アノテーションDI、PostgreSQL切り替え、Jib/Dockerまで解説 | REST API・Docker利用者 | 高: TIS社員による実装解説 |
| Nablarch-example-restを起動した状態でKarate-Gatlingテストを実行する | https://qiita.com/KO_YAmajun/items/0ebf9f6a51222ecdc1ce | @KO_YAmajun (TIS) | 2021-02-16 | NablarchのREST APIにKarate＋Gatlingで負荷テストを実施する方法。依存関係・feature/Scala設定例あり | テスト担当者・パフォーマンステスト | 高: TIS社員、具体的な実装例付き |
| Nablarch（RESTfulウェブサービス）で、データベースを使わないプロジェクトを作る | https://qiita.com/charon/items/967f878bde5096693ffc | @charon (TIS) | 2021-01-24 | RESTfulプロジェクトからH2等のDB依存を除去して動作確認用の簡易プロジェクトを作る手順 | 検証・PoC目的の開発者 | 高: TIS社員、実装手順が明確 |
| Nablarchを使ってみよう | https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17 | @kirin1218 (きりラボ 室長) | 2022-03-23 | Nablarchの特徴と開発の癖を整理した入門シリーズ第1弾。「癖が強いが仲良くなると良い」と評価。利点・課題を両面から解説 | Nablarch未経験エンジニア | 高: 実務経験に基づく主観的評価含む |
| Nablarchを使ったバッチアプリケーションの作成 | https://qiita.com/kirin1218/items/eb3033ecb1520b497cc2 | @kirin1218 (きりラボ 室長) | 2022-04-03 | Nablarchバッチテンプレートをダウンロード・実行するまでの入門ガイド。Eclipse＋Mavenで単体テスト・パッケージ化・バッチ実行 | Nablarchバッチ入門者 | 高: 実務経験に基づく解説 |
| Nablarchを使ったバッチアプリケーションの作成-その２- | https://qiita.com/kirin1218/items/ba6df84e07e96759715f | @kirin1218 (きりラボ 室長) | 2022-04-03 | バッチ実行フローの解説。Mainクラスの引数(diConfig/requestPath/userId)・SampleBatch/SampleResiBatchの継承パターンを解説 | Nablarchバッチ入門者 | 高: 実務経験に基づく解説 |
| Nablarchを使ったWebアプリケーションの作成 | https://qiita.com/kirin1218/items/c537cb8a444ee59763cd | @kirin1218 (きりラボ 室長) | 2022-04-13 | テンプレートプロジェクトのダウンロードからEclipseでの動作確認まで。Java 17対応の設定変更も解説 | Nablarch Web入門者 | 高: 実務経験に基づく解説 |
| Nablarchを学ぶ前に知っておくべき知識について | https://qiita.com/kirin1218/items/99a51d8ffed8b107b4f9 | @kirin1218 (きりラボ 室長) | 2022-04-16 | Servlet・JSP・WAR・JSR・Java SE/EEなどNablarch学習に必要な前提知識を初心者向けに整理 | Nablarch未経験・Java初中級者 | 高: 実務経験に基づく教育コンテンツ |
| Nablarch: writeタグとrawWriteタグの違いは？brタグを使用できるのはどっち？ | https://qiita.com/KO_YAmajun/items/51773c6a42875dc51135 | @KO_YAmajun (TIS) | 2024-01-24 | JSPカスタムタグのwrite/rawWriteのXSSエスケープ動作の違いをコード例と実行結果で解説 | Nablarch Web開発者 | 高: TIS社員、コード実証あり |
| nablarch: nablarch-example-batchをdockerコンテナにする方法 | https://qiita.com/KO_YAmajun/items/30a26d3ce61c3e5209af | @KO_YAmajun (TIS) | 2024-05-06 | 公式未提供のDocker化手順。compose.yml・Dockerfileの具体例でnablarch-example-batchをコンテナ実行 | Docker・コンテナ利用者 | 高: TIS社員、実装例付き |
| Nablarch 6のRESTfulウェブサービスのExampleをOpen Liberty＋PostgreSQLを使うように変更してみる | https://qiita.com/charon/items/2c14ab849d3edfd9fe61 | @charon (TIS) | 2024-06-07 | Nablarch 6 RESTfulサンプルをOpen LibertyとPostgreSQLで動かす改修手順。Java 21・JNDIデータソース設定 | Nablarch 6移行者・Open Liberty利用者 | 高: TIS社員、実装手順が明確 |
| Nablarch v6のウェブプロジェクト用アーキタイプを使って、ウェブアプリケーションを試す | https://qiita.com/charon/items/83a0c7901bcf733fc108 | @charon (TIS) | 2024-08-19 | Nablarch 6u1でウェブアプリを構築。Java 21対応・H2→PostgreSQL切り替え・Docker環境構築まで | Nablarch 6入門者 | 高: TIS社員、環境構築手順が詳細 |
| Nablarch:MySQL＆TiDB用のDialectを作成する | https://qiita.com/KO_YAmajun/items/3433275e930be9a56270 | @KO_YAmajun (TIS) | 2024-09-16 | Nablarchに存在しないMySQLDialectを実装する方法。12メソッドのオーバーライド参照表付き。TiDB互換性も解説 | MySQL/TiDB利用者 | 高: TIS社員、実装例と詳細テーブルあり |
| [5u25リリース記念]nablarch-micrometer-otlpでexample-restアプリの性能測定 | https://qiita.com/KO_YAmajun/items/191ba0b148a61d6e9525 | @KO_YAmajun (TIS) | 2024-09-30 | nablarch-micrometer-otlpによるOpenTelemetry準拠メトリクス収集の実装。Prometheusでの可視化まで | クラウドネイティブ・可観測性担当 | 高: TIS社員、リリース直後の解説 |
| Nablarchのブランクプロジェクトを作成した後に、使用するデータベースを切り替えるスクリプトを書く | https://qiita.com/charon/items/43b986e9614aaecd0ac7 | @charon (TIS) | 2024-10-08 | H2→PostgreSQL切り替えシェルスクリプト。pom.xml・接続パラメータ・DialectをDockerでワンコマンド変更 | 環境セットアップ担当 | 高: TIS社員、スクリプト実装例付き |
| Fintan・Nablarchコンテンツリスト | https://qiita.com/KO_YAmajun/items/3f071e02444124f7ef07 | @KO_YAmajun (TIS) | 2025-01-07 | FintanとNablarch公開資料の包括的一覧。入門からトレーニング・実装ガイド・事例までリンクを網羅（2026-01-27更新） | 全Nablarchユーザー | 高: TIS社員によるコンテンツインデックス |
| Nablarch×S3(ファイルアップロード)を実施するサンプルコード | https://qiita.com/KO_YAmajun/items/32b9154dab6c3d1a573f | @KO_YAmajun (TIS) | 2026-03-15 | AWS SDK v2でS3互換ストレージにアップロード。TransferManagerとリアクティブ方式の比較（推奨はTransferManager） | AWS S3連携開発者 | 高: TIS社員、実装例と性能比較あり |
| Nablarchでカスタムロガーを実装し、全てのアプリケーションログに透過的な項目添加を行う方法 | https://qiita.com/KO_YAmajun/items/e78aeaa04350b4183e76 | @KO_YAmajun (TIS) | 2026-05-10 | OtelJsonLogFormatterを拡張してOpenTelemetry対応カスタムロガーを実装。トレースID等を透過的に付与 | 可観測性・ログ設計担当 | 高: TIS社員、最新版(2026年)の解説 |

---

## Zenn

Zennでは「nablarch」検索でヒット件数が非常に少ない（検索ページは「見つかりませんでした」表示）。唯一見つかった記事はNablarchを直接扱う内容ではなかった。

| タイトル | URL | 著者 | 公開日 | 要旨 | 想定読者 | 信頼性所見 |
|---------|-----|------|--------|------|---------|----------|
| Repositoryクラス、ServiceクラスのJUnitを使った単体テストについて | https://zenn.dev/monaka0309/articles/4cc135396a7ae2 | もなか | 2025-01-27 | Spring Boot JUnit入門記事。Nablarchへの言及は`@ExtendWith`説明のNablarch公式リンク1件のみ（Nablarch専記事ではない） | Spring Boot初心者 | 低: Nablarch内容は参照リンクのみ |

---

## Fintan（公式枠外の事例・体験記）

Fintanは主にTISによるNablarch公式コンテンツサイトだが、以下はトレーニング・APIリファレンス等の公式ドキュメントではなく、事例紹介・技術解説・ツール紹介に分類されるもの。

| タイトル | URL | 著者 | 公開日 | 要旨 | 想定読者 | 信頼性所見 |
|---------|-----|------|--------|------|---------|----------|
| NablarchアプリをAzureで動かした事例の紹介 | https://fintan.jp/?p=6634 | Nablarchチーム | 2021-03-31 | example-chatをAzure App Service/PostgreSQL/Redis/Blob Storageにデプロイ。Docker/Azure Container Registry連携まで | クラウド移行担当者 | 高: TIS公式チームによる検証済み事例 |
| NablarchとAWSマネージドサービスを使ったマイクロサービスアーキテクチャの構築法紹介 | https://fintan.jp/page/200/ | Nablarchチーム | 2021-03-31 | AWS Fargate+ECSでNablarchをマイクロサービス展開。X-Ray分散トレーシング・Micrometer/CloudWatchメトリクス収集 | MSA・AWSアーキテクト | 高: TIS公式チームによる構築事例 |
| Nablarchの開発がどのように行われているかのご紹介 | https://fintan.jp/page/1590/ | Nablarchチーム | 2020-10-28 | Nablarchの開発プロセス公開。後方互換性レビュー・GitHub/Concourse CI・reST+Sphinx文書・JUnit/性能テストの体制 | OSSコントリビューター候補・上流工程担当 | 高: TIS公式チームによる開発内部情報 |
| Nablarchのハンドラを作成してみよう | https://fintan.jp/page/14812/ | 西川 武 (TIS) | 2025-02-14 | ログインチェックハンドラを実例にした独自ハンドラ実装ガイド。前処理/後処理/例外処理の分類・ハンドラキュー組み込み手順 | Nablarchカスタマイズ担当 | 高: TIS著者、具体的実装例付き |
| Oscana: SAStruts/Struts1からNablarchへのマイグレーションツール | https://fintan.jp/page/1452/ | Nablarchチーム | 2021-03-31 | SAStruts/Struts1からNablarchへ自動変換。Java/JSP/SQL/設定ファイルに対応。互換ライブラリでレガシー動作を維持 | 既存システム移行担当 | 高: TIS公式ツールの解説 |

---

## ブログ・note・個人サイト

| タイトル | URL | 著者 | 公開日 | 要旨 | 想定読者 | 信頼性所見 |
|---------|-----|------|--------|------|---------|----------|
| 可変条件を持つSQL | https://note.com/aikofight7/n/n817b1d24bea0 | あいこん (note.com) | 2021-07-05 | Nablarchの`$if`記法を使った可変条件付きWHERE句のSQL記述方法。Beanオブジェクト入力時のOR条件パターン | Nablarch SQL実装者 | 中: 個人メモ的記事。内容は妥当だがNablarch 5u8時点 |
| Nablarch 所感 | https://himeji-cs.jp/blog/2018/02/02/nablarch/ | 不明 (himeji-cs.jp) | 2018-02-02 | コード再利用性・DI・XML設定・Struts比較への主観的所感。「わかりにくい」という批評を含む個人ブログ記事（現在接続不可） | Nablarch評価検討者 | 中: 実務経験ベースの主観評価だが2018年時点の古い情報・サイト接続不可 |
| Javaフレームワーク「Nablarch」に深刻な脆弱性 - 早急に対応を | https://www.security-next.com/102898 | Security NEXT編集部 | 2019-02-27 | Nablarch 5系のXXE脆弱性（CVE-2019-5918）を報告。JVN公表に基づく。情報漏洩・サービス停止リスクあり | Nablarch利用セキュリティ担当 | 高: JVN公表情報に基づくニュース記事 |
| Nablarchとは何か：エンタープライズシステム向けフレームワークの概要と特長 | https://www.issoh.co.jp/tech/details/3384/ | 株式会社一創 | 不明 | エンタープライズ向けJavaフレームワークとしてのNablarchの特長・開発方法を解説。TIS以外の企業による第三者解説（現在403） | Nablarch導入検討者 | 中: TIS外部の企業ブログ。内容は妥当だが現在アクセス不可 |

---

## スライド資料（SpeakerDeck等）

専用SpeakerDeckスライドは検索では確認できなかった。JJUG CCCなどのJavaコミュニティイベントで言及される可能性があるが、Nablarch専用発表資料はインデックス未確認。SlideShareにも専用資料なし。

| タイトル | URL | 著者 | 公開日 | 要旨 | 想定読者 | 信頼性所見 |
|---------|-----|------|--------|------|---------|----------|
| （専用スライド資料なし） | - | - | - | SpeakerDeck/SlideShareでNablarch専用のスライドは発見できなかった。TISがJJUG等で登壇している可能性はあるが未確認 | - | - |

---

## YouTube

Nablarch専用の動画コンテンツは確認できなかった。チュートリアル動画・解説動画は存在しないとみられる。

| タイトル | URL | 著者 | 公開日 | 要旨 | 想定読者 | 信頼性所見 |
|---------|-----|------|--------|------|---------|----------|
| （専用動画なし） | - | - | - | YouTubeでNablarch専用動画は未発見。Nablarchというキーワードで検索してもヒットなし | - | - |

---

## X(Twitter)サンプリング

X(Twitter)への直接アクセスは困難なため、検索エンジン経由でのサンプリング結果を示す。

**コミュニティの雰囲気（間接収集）:**

- 「情報が少なくてつらい」「癖が強い」「SIer案件で使わされる」系の言及が散見される
- 「Nablarchで案件」「受託でNablarch」という実務文脈での言及がある
- TIS社員による機能リリース告知ツイートが定期的にある模様（Qiita記事へのリンク投稿）
- フレームワーク比較記事でNablarchが除外されることが多く、「ニッチすぎる」との認識がコミュニティにある
- 「Nablarchとは何か」という素朴な質問やGoogle検索ゼロヒット体験の共有が散見される

**注記:** X(Twitter)の直接スクレイピングは不可能なため、上記は検索エンジン経由で収集したコミュニティ言及のサマリーです。完全網羅ではありません。

---

## 調査まとめ

| プラットフォーム | 記事数 | 特記事項 |
|----------------|--------|---------|
| Qiita | 20件（タグ総数31件） | 約8割がTIS社員。@kirin1218が唯一の有力な非TISエンジニア |
| Zenn | 0件（実質） | Nablarch専用記事なし。マイナーフレームワーク特性 |
| Fintan（事例・体験記） | 5件 | クラウド事例(Azure/AWS)・OSS開発プロセス・移行ツール |
| ブログ/note/個人サイト | 3件 | 古い個人ブログが主。新規参入少ない |
| SpeakerDeck/SlideShare | 0件 | 専用スライドなし |
| YouTube | 0件 | 専用動画なし |
| X(Twitter) | サンプリングのみ | 「情報少ない」「SIer向け」の認識が支配的 |

**全体所感:** Nablarchの非公式コンテンツは著しく少なく、大部分がTIS社員による記事。独立したコミュニティコンテンツが育っていない状態。これ自体がNablarchのエコシステムの課題を示す重要な知見である。
