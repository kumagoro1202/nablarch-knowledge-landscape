# ユーザーストーリー仮説: 外部委託先エンジニア

ロール定義: [roles.md](../roles.md#7-外部委託先エンジニア)

---

## シナリオ 1: 初めてNablarchプロジェクトにアサインされて全体を把握する

**背景:**
フリーランスエンジニア。SIerから既存Nablarchシステムの機能追加を受注した。Spring Boot経験は豊富だがNablarch自体は初めて。発注元からソースコードと公式ドキュメントのURLだけ渡されて作業開始。

**ニーズ発生:**
「Nablarchとはどんなフレームワークか。Spring Bootとの違いを素早く把握したい」

**流入経路:**
- Google検索: "nablarch とは" または "nablarch spring 違い"
- → Qiita: [Nablarchを使ってみよう（@kirin1218）](https://qiita.com/kirin1218/items/242ee0f174f1cb12ef17)
- → Qiita: [Nablarchを学ぶ前に知っておくべき知識について（@kirin1218）](https://qiita.com/kirin1218/items/99a51d8ffed8b107b4f9)

**探す情報:**
- NablarchのDI・設定ファイル・アーキテクチャの概要
- Spring Bootとの主な違い（DIの方法・ハンドラキューの概念等）
- 最初に読むべきドキュメントの場所

**目的:**
1週間以内にNablarchの基礎を習得し、割り当てられた機能追加タスクを開始できる状態にする。

**困りごと・ギャップ:**
- 「Nablarch経験者向けに書かれたコンテンツ」が大多数で、**「他フレームワーク経験者がNablarchに入門するための橋渡しコンテンツ」が存在しない**
- Spring経験者目線での「Nablarchではこう書く」対比表がない
- 公式ドキュメントは機能別に整理されており、**「まずここを読め」という入門パス**が不明確

---

## シナリオ 2: 発注元のコーディング規約に準拠して実装する

**背景:**
発注元（SIer）から「nablarch-unpublished-api-checker でのチェックをCIで通すこと」と指示された。チェッカーが「非公開API使用」と警告を出しているが、何が問題なのかわからない。

**ニーズ発生:**
「nablarch-unpublished-api-checkerが検出するNablarch非公開APIの一覧と、公開APIによる代替方法を知りたい」

**流入経路:**
- GitHub: [nablarch-unpublished-api-checker](https://github.com/nablarch/nablarch-unpublished-api-checker) のREADMEと設定例を確認
- 公式ドキュメント: [nablarch-document](https://nablarch.github.io/docs/LATEST/doc/) → APIリファレンスで公開/非公開の区別を確認

**探す情報:**
- チェッカーが検出する「非公開API」の定義と具体例
- 警告が出たコードの修正方法
- チェッカーのMaven Plugin設定方法

**目的:**
CIチェックをpassし、発注元へ納品物のコーディング規約準拠を証明する。

**困りごと・ギャップ:**
- nablarch-unpublished-api-checkerのREADMEにセットアップ手順はあるが、**「具体的にどのAPIが非公開で何を使えばよいか」のリスト**が整理されていない
- チェッカーのエラーメッセージから修正方法を逆引きする手段がない

---

## シナリオ 3: MySQL環境でNablarchを使うための設定を調べる

**背景:**
発注元のシステムはMySQL + Nablarchで動いている。新規追加機能でMySQL固有のSQL構文（ON DUPLICATE KEY UPDATE等）を使いたいが、NablarchのDialectがどこまで対応しているか不明。

**ニーズ発生:**
「NablarchでMySQLを使うためのDialect設定と、MySQL固有機能の対応状況を知りたい」

**流入経路:**
- Qiita: [Nablarch:MySQL＆TiDB用のDialectを作成する（@KO_YAmajun）](https://qiita.com/KO_YAmajun/items/3433275e930be9a56270)
- GitHub: [nablarch-core-jdbc](https://github.com/nablarch/nablarch-core-jdbc) でDialectクラスを確認

**探す情報:**
- NablarchのMySQL対応状況（公式Dialectの存在有無）
- カスタムDialect実装の必要箇所と手順
- MySQL固有SQL構文のNablarch流の書き方

**目的:**
MySQL環境での機能を正常動作させ、発注元の動作確認を通過する。

**困りごと・ギャップ:**
- **NablarchのMySQLDialectは公式未提供**。@KO_YAmajunのQiita記事が唯一の情報源だが、2024年公開の1記事に依存するのは不安
- 「主要RDBMSごとのDialect対応表」が公式ドキュメントに存在しない
- MySQL以外（TiDB, Aurora MySQL等）での動作実績情報がほぼない
