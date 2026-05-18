# ユーザーストーリー仮説: アーキテクト/テックリード

ロール定義: [roles.md](../roles.md#1-アーキテクトテックリード)

---

## シナリオ 1: ハンドラキューのカスタマイズ方法を調べる

**背景:**
新規プロジェクトでNablarchを採用することが決まった。認証・認可ハンドラを独自実装する必要があるため、ハンドラキューの仕組みとカスタムハンドラの作り方を把握したい。

**ニーズ発生:**
「ログインチェックを行う独自ハンドラをどう実装し、ハンドラキューに組み込むか」

**流入経路:**
- Google検索: "nablarch ハンドラ 自作" または "nablarch handler custom"
- → Fintan記事: [Nablarchのハンドラを作成してみよう](https://fintan.jp/page/14812/)

**探す情報:**
- ハンドラインターフェースの実装方法（前処理/後処理/例外処理の分類）
- ハンドラキューへの組み込み設定（component-configuration）
- 既存サンプルコード

**目的:**
独自ハンドラの実装パターンを理解し、開発チームへの設計指示書を作成する。

**困りごと・ギャップ:**
- Fintan記事はログインチェックの具体例のみ。「汎用的なハンドラ設計パターン」や「複数ハンドラの連携テスト方法」の解説が不足している
- 公式ドキュメント（[nablarch-document](https://nablarch.github.io/docs/LATEST/doc/)）はAPIリファレンスレベルで、「なぜこの順序でハンドラを並べるか」の設計根拠が得られない

---

## シナリオ 2: バッチとWebの共存アーキテクチャを設計する

**背景:**
既存のWebシステムに夜間バッチ処理を追加することになった。バッチモジュール（nablarch-fw-batch）をWebプロジェクトにどう組み込むかを検討している。

**ニーズ発生:**
「NablarchのWebプロジェクトとバッチプロジェクトを同一リポジトリで管理できるか」

**流入経路:**
- GitHub: [nablarch-system-development-guide](https://github.com/Fintan-contents/nablarch-system-development-guide)のREADMEやサンプルプロジェクト構成を参照
- 補助: [nablarch-example-batch](https://github.com/nablarch/nablarch-example-batch) のpom.xmlとディレクトリ構成を確認

**探す情報:**
- Web/バッチ共存プロジェクトのMaven構成例
- nablarch-fw-batchとnablarch-fw-webの依存関係
- 共通コンポーネント（DAOなど）のモジュール分割パターン

**目的:**
プロジェクトのMaven構成と各モジュールの責務分割方針を確定する。

**困りごと・ギャップ:**
- サンプルプロジェクトはWebのみ・バッチのみの単体構成。**Web+バッチ共存の公式サンプルが存在しない**
- nablarch-system-development-guideのサンプルも個別構成のみで、マルチモジュールpom.xml例がない

---

## シナリオ 3: Nablarch 6へのバージョンアップ影響を評価する

**背景:**
現行システムはNablarch 5u25で稼働中。Nablarch 6（Jakarta EE対応）への移行を検討しており、破壊的変更の影響範囲を把握したい。

**ニーズ発生:**
「Nablarch 5→6の移行で何が変わるか、影響範囲を調べたい」

**流入経路:**
- GitHub: [nablarch-document](https://github.com/nablarch/nablarch-document) → migration-guide を検索
- Qiita: [Nablarch 6のRESTfulウェブサービスのExampleをOpen Liberty＋PostgreSQLを使うように変更してみる](https://qiita.com/charon/items/2c14ab849d3edfd9fe61) を参照

**探す情報:**
- javax.* → jakarta.* パッケージ変更の影響
- 各モジュールのNablarch 6対応状況（nablarch-micrometer-adaptor等）
- 移行ステップと推奨順序

**目的:**
移行計画（スコープ・工数・リスク）を作成しプロジェクトオーナーに提案する。

**困りごと・ギャップ:**
- 移行ガイドは存在するが、「**実際の移行で何時間かかったか**」「**どのモジュールで問題が発生しやすいか**」といった実践的な移行体験レポートが極めて少ない
- Qiita記事は新規構築例であり、**既存コードベースの移行**に焦点を当てたコンテンツがない
