# ユーザーストーリー仮説: 横断品質保証（QA）担当

ロール定義: [roles.md](../roles.md#8-横断品質保証qa担当)

---

## シナリオ 1: Nablarchプロジェクトのテストフレームワーク構成を決定する

**背景:**
複数のNablarchプロジェクトを横断して品質管理をしているQA担当。新規プロジェクト立ち上げ時に「どのテストフレームワークをどう組み合わせるか」の標準を決める必要がある。

**ニーズ発生:**
「nablarch-testing, nablarch-testing-junit5, nablarch-testing-rest をどう組み合わせるのがベストプラクティスか」

**流入経路:**
- GitHub: [nablarch-testing-junit5](https://github.com/nablarch/nablarch-testing-junit5) のREADMEを確認
- GitHub: [nablarch-testing-rest](https://github.com/nablarch/nablarch-testing-rest) のREADMEを確認
- 公式ドキュメント: [nablarch-document](https://nablarch.github.io/docs/LATEST/doc/) → テスト フレームワーク セクション

**探す情報:**
- 各テストモジュールの対象（UT/結合テスト/REST APIテスト）と使い分け
- JUnit 5 + nablarch-testing の設定手順
- Excleベーステストデータ管理の有効範囲

**目的:**
プロジェクト標準テスト構成を文書化し、全プロジェクトに展開する。

**困りごと・ギャップ:**
- **各テストモジュールの役割と組み合わせパターンが公式ドキュメントで一元整理されていない**。それぞれのREADMEを読み合わせるしかない
- JUnit 5対応の nablarch-testing-junit5 は比較的新しく、**既存の nablarch-testing との機能差・移行ガイドが不明確**
- 「Nablarchプロジェクトの推奨テスト戦略」のような上位方針ドキュメントが存在しない

---

## シナリオ 2: REST APIの負荷テスト環境を整備する

**背景:**
Nablarch REST APIの性能要件（1秒あたり200リクエスト処理）を検証したい。KarateとGatlingを組み合わせた負荷テストの実施方法を調べている。

**ニーズ発生:**
「nablarch-example-restにKarate+Gatlingを適用して負荷テストを実施したい。設定手順が知りたい」

**流入経路:**
- Qiita: [Nablarch-example-restを起動した状態でKarate-Gatlingテストを実行する（@KO_YAmajun）](https://qiita.com/KO_YAmajun/items/0ebf9f6a51222ecdc1ce)
- GitHub: [nablarch-example-rest](https://github.com/nablarch/nablarch-example-rest) でサンプルの構成を確認

**探す情報:**
- pom.xmlへのKarate/Gatling依存関係追加方法
- feature記述によるシナリオ定義
- 結果レポートの見方と性能基準との突き合わせ方法

**目的:**
CI環境に負荷テストを組み込み、機能リリースごとに性能劣化を検知できる仕組みを作る。

**困りごと・ギャップ:**
- @KO_YAmajunのQiita記事はセットアップ手順が詳しいが、**「CI/CDパイプラインへの組み込み」（GitHub Actions等での自動化）**の解説がない
- Nablarch用の性能テスト結果の基準値（比較ベンチマーク）が公開されていない
- Gatling以外（JMeter等）との組み合わせ実績情報が皆無

---

## シナリオ 3: セキュリティ脆弱性の対応状況を確認する

**背景:**
定期セキュリティレビューでNablarchの過去CVEと現在のバージョンが対応済みかを確認する必要がある。

**ニーズ発生:**
「Nablarchの既知CVEとセキュリティパッチ適用状況を一覧で確認したい」

**流入経路:**
- Google検索: "nablarch CVE" または "nablarch 脆弱性"
- → セキュリティニュース: [Javaフレームワーク「Nablarch」に深刻な脆弱性（security-next.com）](https://www.security-next.com/102898) で過去CVEを把握
- → 公式リリースノート（GitHub nablarch-document）で修正バージョンを確認

**探す情報:**
- 既知CVE（CVE-2019-5918等）の影響範囲と修正バージョン
- セキュリティアドバイザリの公式発出方法（JVN/GitHub Advisory）
- 現在使用バージョンが対応済みかの確認手順

**目的:**
セキュリティレビューレポートに「Nablarchの既知脆弱性は対応済み」と根拠付きで記載する。

**困りごと・ギャップ:**
- **Nablarchのセキュリティアドバイザリが一元化されていない**。JVNとGitHub Advisoryと公式ブログが分散しており、全件把握が困難
- 「使用バージョンがどのCVEに対応済みか」のバージョン×CVE対応マトリクスが存在しない
- セキュリティ担当向けの「Nablarchセキュリティガイドライン」（安全な設定・CSRFトークン・XSSエスケープの使い方）がまとまっていない（JSPタグのwrite/rawWriteの違いはQiita記事が詳しいが、公式ドキュメントでの強調が弱い）
