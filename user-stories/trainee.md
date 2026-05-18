# ユーザーストーリー仮説: 新人研修生

ロール定義: [roles.md](../roles.md#6-新人研修生)

---

## シナリオ 1: 研修でNablarchブランクプロジェクトを立ち上げる

**背景:**
入社1年目のエンジニア。Nablarch研修の初日。「ブランクプロジェクトを作ってHello Worldを表示しなさい」という課題が出た。Javaは授業で学習済みだがフレームワークは初めて。

**ニーズ発生:**
「Nablarchでプロジェクトを作る最初の手順を知りたい（環境構築から起動まで）」

**流入経路:**
- 研修担当から渡されたURL: [Fintan-contents/nablarch-training](https://github.com/Fintan-contents/nablarch-training)
- 補助: Qiita: [Nablarchを使ったWebアプリケーションの作成（@kirin1218）](https://qiita.com/kirin1218/items/c537cb8a444ee59763cd)

**探す情報:**
- Mavenアーキタイプコマンドによるプロジェクト生成手順
- 必要な開発環境（JDK、Maven、IDE）のバージョン
- 初回ビルドと動作確認（H2データベース起動）の手順

**目的:**
ブランクプロジェクトを起動し、ブラウザでトップページが表示されることを確認する。

**困りごと・ギャップ:**
- nablarch-trainingのREADMEは研修コース全体を示しているが、**「今日最初に何をするか」の具体的なステップゼロが不明確**
- Java 17/21での動作設定変更（モジュールシステム対応等）がトレーニング資料に反映されていない場合がある（@kirin1218のQiita記事に補足あり）
- エラーが出た際の「よくあるエラーと解決策」集が存在しない

---

## シナリオ 2: Nablarch特有の概念（ハンドラキュー・コンポーネント設定）を理解する

**背景:**
ブランクプロジェクトは動いたが、コードを追うとXMLの設定ファイル（component-configuration）が大量にあり、どこで何を設定しているか全くわからない。

**ニーズ発生:**
「component-configuration.xmlとは何か。ハンドラキューとは何かを図解で理解したい」

**流入経路:**
- 公式ドキュメント: [nablarch-document](https://nablarch.github.io/docs/LATEST/doc/) → アーキテクチャ概説ページ
- Qiita: [Nablarchを学ぶ前に知っておくべき知識について（@kirin1218）](https://qiita.com/kirin1218/items/99a51d8ffed8b107b4f9)

**探す情報:**
- Nablarchのアーキテクチャ概念図（ハンドラキューの処理フロー）
- component-configuration.xmlの基本構造
- SystemRepositoryの役割とBeanの登録方法

**目的:**
「なぜこのXMLが必要か」を理解し、設定ファイルの読み方を習得する。

**困りごと・ギャップ:**
- 公式ドキュメントのアーキテクチャ解説は技術的に正確だが**初学者には難解**。図解・アニメーション等のビジュアル説明が皆無
- YouTubeにNablarch解説動画が存在しない（Java入門動画は多数あるが、Nablarch固有概念の動画ゼロ）
- @kirin1218のQiita記事が唯一の「やさしい入門解説」だが、シリーズが途中で終わっている

---

## シナリオ 3: 研修課題（CRUD実装）でつまずいたときに自力解決する

**背景:**
研修課題でTODOリストのCRUD（作成・読取・更新・削除）を実装中。「UPDATE後に一覧画面が更新されない」というバグが発生。エラーは出ておらずデバッグ方法がわからない。

**ニーズ発生:**
「NablarchのWebアプリでリダイレクト後のデータ反映をデバッグする方法が知りたい」

**流入経路:**
- Google検索: "nablarch リダイレクト データ反映" → 公式ドキュメントの「HTTPリクエストフロー」を確認
- GitHub: [nablarch-example-web](https://github.com/nablarch/nablarch-example-web) のソースコードを参照してパターンを確認

**探す情報:**
- PRG（Post/Redirect/Get）パターンの実装方法
- フラッシュスコープを使ったメッセージ受け渡しの設定
- ログ出力を使ったデバッグ方法（SQLログの有効化）

**目的:**
バグを自力で解決して研修課題を完成させる。

**困りごと・ギャップ:**
- nablarch-example-webは動作するサンプルだが、**「なぜこの実装になっているか」のコメントが少なく**、初学者が真似しても理解できない
- **「Nablarch トラブルシューティング」「Nablarch FAQ」**的なコンテンツが存在しない（公式・非公式ともにゼロ）
- Stack OverflowのNablarchタグも回答数が極めて少なく、詰まったときの逃げ場がない
