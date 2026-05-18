# ユーザーストーリー仮説: 保守・運用担当エンジニア

ロール定義: [roles.md](../roles.md#4-保守運用担当エンジニア)

---

## シナリオ 1: 本番障害のログからNablarchエラーの原因を特定する

**背景:**
本番環境でユーザーから「特定の検索処理がエラーになる」と報告が入った。Nablarch固有のスタックトレースが出力されているが、フレームワーク内部のエラーで原因がわからない。

**ニーズ発生:**
「NablarchのエラーコードやスタックトレースからSQL実行エラーの原因を特定したい」

**流入経路:**
- 公式ドキュメント: [nablarch-document](https://nablarch.github.io/docs/LATEST/doc/) → ログ仕様・メッセージID一覧を検索
- GitHub: [nablarch-core-applog](https://github.com/nablarch/nablarch-core-applog) のREADMEでログ出力設定を確認

**探す情報:**
- Nablarchエラーコード（メッセージID）の一覧と意味
- SQLログ出力設定の方法
- ハンドラキュー上での例外伝播ルール

**目的:**
障害原因を特定し、修正または運用回避策を実施する。

**困りごと・ギャップ:**
- **メッセージIDの一覧と説明が公式ドキュメントで一元管理されていない**。エラーコードが出てもソースコード検索が必要
- ハンドラキューでの例外処理フローを解説したコンテンツが不足しており、「どのハンドラが例外を握りつぶしているか」が追いにくい

---

## シナリオ 2: マイナーバージョンアップ（例: 5u24→5u25）の影響を確認する

**背景:**
セキュリティ勧告を受けてNablarchのマイナーバージョンアップを実施することになった。既存機能への影響を事前に確認したい。

**ニーズ発生:**
「Nablarch 5u25のリリースノートと破壊的変更一覧を確認したい」

**流入経路:**
- GitHub: [nablarch-document](https://github.com/nablarch/nablarch-document) → リリースノート（RELEASES.md相当）を参照
- Qiita: [5u25リリース記念 nablarch-micrometer-otlpでexample-restアプリの性能測定](https://qiita.com/KO_YAmajun/items/191ba0b148a61d6e9525) でリリース内容を把握

**探す情報:**
- 各バージョンのリリースノート（変更点・非推奨API・破壊的変更）
- バージョンアップ時の対応手順（pom.xml修正・設定変更）
- 既知の問題・バグフィックス

**目的:**
バージョンアップの影響範囲を確認し、テスト計画を立案・実施する。

**困りごと・ギャップ:**
- リリースノートはGitHub上に存在するが、**「運用担当者向け」の要約（この機能に影響あり/なし）がなく**、詳細を全て読まなければならない
- 依存モジュール（nablarch-micrometer-adaptor等）のバージョン互換性表が整備されていない

---

## シナリオ 3: パフォーマンス劣化の原因調査と対策

**背景:**
リリース後から徐々にレスポンスタイムが悪化している。Nablarch側での性能計測・ボトルネック特定の方法を調べたい。

**ニーズ発生:**
「Nablarchアプリのパフォーマンス統計を収集し、ボトルネックを特定したい」

**流入経路:**
- GitHub: [nablarch-statistics-report](https://github.com/nablarch/nablarch-statistics-report) のREADMEで機能を確認
- Qiita: [5u25リリース記念 nablarch-micrometer-otlpでexample-restアプリの性能測定](https://qiita.com/KO_YAmajun/items/191ba0b148a61d6e9525) でMicrometerを使ったメトリクス収集を参照
- Fintan: [NablarchとAWSマネージドサービスを使ったマイクロサービスアーキテクチャの構築法紹介](https://fintan.jp/page/200/) でX-Ray/CloudWatch連携例を確認

**探す情報:**
- nablarch-statistics-reportの設定手順と出力フォーマット
- Micrometer/OpenTelemetry連携の設定方法
- SQLクエリレベルの実行時間計測方法

**目的:**
パフォーマンスボトルネック（SQLか、ハンドラ処理か、外部連携か）を特定して改善対策を実施する。

**困りごと・ギャップ:**
- nablarch-statistics-reportは存在するが**ドキュメントが薄く**、セットアップが試行錯誤になる
- MicrometerとOpenTelemetry対応は新しいため、**古いバージョン（5u20以前）での性能計測方法**に関するコンテンツがない
- 「実際のパフォーマンス改善事例」（Before/After）のコンテンツが皆無
