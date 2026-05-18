# ユーザーストーリー仮説: PG/UT担当エンジニア（新人〜中堅）

ロール定義: [roles.md](../roles.md#3-pgut担当エンジニア新人中堅)

---

## シナリオ 1: Universal DAOを使ったDB検索の実装方法を調べる

**背景:**
Nablarch初経験の中堅エンジニア。アサインされたタスクで複数テーブルを結合した検索処理を実装しなければならない。JPA/MyBatisは経験あるが、NablarchのUniversal DAOは初めて。

**ニーズ発生:**
「Universal DAOでJOINを含む検索をどう書くか、カスタムSQLを使う場合の書き方を知りたい」

**流入経路:**
- 公式ドキュメント: [nablarch-document](https://nablarch.github.io/docs/LATEST/doc/) → Universal DAO ページに直接アクセス
- 補助: GitHub [nablarch-common-dao](https://github.com/nablarch/nablarch-common-dao) のREADMEとサンプルコード

**探す情報:**
- Universal DAOの基本使い方（findAll / findBySqlFile 等）
- SQLファイルを使った複雑なクエリの書き方（$if条件付きSQL）
- Entityクラスへのマッピング方法

**目的:**
担当機能のDB検索処理を実装し、UT（単体テスト）まで完了させる。

**困りごと・ギャップ:**
- 公式ドキュメントにサンプルコードはあるが「**複数テーブルJOINのEntityマッピング**」の具体例が薄い
- SQLファイルの`$if`記法はnote.comの個人記事（[可変条件を持つSQL](https://note.com/aikofight7/n/n817b1d24bea0)）に詳しいが、公式ドキュメントでの説明が見つけにくい
- エラーメッセージからデバッグする手順の解説がない

---

## シナリオ 2: バリデーションアノテーションの使い方を調べる

**背景:**
画面からの入力値チェック実装を担当。独自の入力バリデーションルール（文字種制限・桁数・必須チェック）を設定する必要がある。

**ニーズ発生:**
「Nablarchの入力バリデーションでカスタムアノテーションを作るにはどうするか」

**流入経路:**
- 公式ドキュメント: [nablarch-document](https://nablarch.github.io/docs/LATEST/doc/) → バリデーション セクション
- GitHub: [nablarch-core-validation-ee](https://github.com/nablarch/nablarch-core-validation-ee) のREADME（Jakarta Bean Validation対応版を使う場合）

**探す情報:**
- 標準バリデーションアノテーション一覧（@Required, @Length 等）
- カスタムアノテーション（ConstraintValidator）の作り方
- バリデーションエラーメッセージの設定方法

**目的:**
画面の入力チェック機能を実装し、テストデータを使ったUTで正常/異常系を確認する。

**困りごと・ギャップ:**
- Nablarch独自バリデーション（nablarch-core-validation）とJakarta Bean Validation（nablarch-core-validation-ee）の**2系統が存在し、どちらを使うべきか公式ガイダンスが不明確**
- カスタムバリデーターの実装例がドキュメントに不足しており、GitHubのソースコード直読みが必要になる場面がある

---

## シナリオ 3: nablarch-testingを使った単体テストの書き方を調べる

**背景:**
Nablarchのテストフレームワーク（nablarch-testing）を使って、DBアクセスを伴う処理のUTを書く必要がある。JUnit 5を使っているため対応状況を確認したい。

**ニーズ発生:**
「JUnit 5でnablarch-testingを使うにはどうするか。テストデータをExcelで管理する方法が知りたい」

**流入経路:**
- GitHub: [nablarch-testing-junit5](https://github.com/nablarch/nablarch-testing-junit5) のREADMEと設定例を確認
- Qiita: [Nablarch-example-restを起動した状態でKarate-Gatlingテストを実行する](https://qiita.com/KO_YAmajun/items/0ebf9f6a51222ecdc1ce) を参照（テスト手法の全体像把握）

**探す情報:**
- nablarch-testing-junit5のセットアップ手順（pom.xml依存関係）
- Excelテストデータの書き方と配置場所
- DBUnitとの連携・テーブル初期化方法

**目的:**
担当モジュールのUTを自動化し、CIパイプラインで実行できるようにする。

**困りごと・ギャップ:**
- nablarch-testing-junit5のドキュメントが少なく、**Excelテストデータのフォーマット仕様**が公式ドキュメントを深掘りしないと見つからない
- JUnit 5対応のサンプルプロジェクト（nablarch-example相当）が存在しない
- テストデータ管理の「ベストプラクティス」に関するコンテンツがほぼない
