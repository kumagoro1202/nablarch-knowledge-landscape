# Nablarch Knowledge Landscape

Nablarchエコシステムの公式・非公式コンテンツを俯瞰整理するリポジトリです。

## 目的

- Nablarch関連コンテンツの全量把握と役割整理
- 新規コンテンツの配置先判断ロジックの整備
- ステークホルダーロール別のユーザーストーリー仮説の構築
- AIレビュー基盤のプリセット観点リストの提供

## ディレクトリ構成

| ファイル/ディレクトリ | 内容 |
|----------------------|------|
| official-content-inventory.md | 公式Nablarchコンテンツ一覧（GitHub/Fintan） |
| unofficial-content-inventory.md | 非公式Nablarchコンテンツ一覧（Qiita/Zenn等） |
| content-placement-flowchart.md | 新規コンテンツ配置先判断フローチャート |
| questions-for-lord.md | 調査中に生じた判断保留事項の一覧 |
| roles.md | ステークホルダーロール定義（Phase 2） |
| user-stories/ | ロール別ユーザーストーリー仮説（Phase 2） |
| review-perspectives/ | Nablarch特化レビュー観点リスト（Phase 3） |

## 観点リストの読み方

`review-perspectives/` 配下の各ファイルは以下の構成になっています。

1. **一覧サマリー表で全体把握** — ファイル冒頭の「観点一覧」表で、全観点を横断比較・取捨選択できます
2. **IDリンクで詳細セクションへジャンプ** — 一覧表のID列（例: `[BATCH-003](#batch-003)`）をクリックすると対応する詳細セクションに移動します
3. **詳細は項目表+NG/OK例で確認** — 各観点の詳細セクションには優先度・適用条件・チェック方法をまとめた項目表と、NGパターン・OKパターンのコード例が記載されています
4. **機械処理は `perspectives.json` を利用** — プログラムで観点データを扱う場合は `review-perspectives/perspectives.json` を参照してください

## 出典URL明記ポリシー

本リポジトリの全記述には出典URLを必ず付記します。
URLのない記述は信頼性なしとみなし、随時修正します。

## GitHub Organization の確認手順

このリポジトリで扱う GitHub Organization は以下の3つで、
**それぞれ独立した Organization です（親子・傘下関係はありません）**。

| Organization | URL | 用途 |
|---|---|---|
| nablarch | https://github.com/nablarch | フレームワーク本体・ライブラリ |
| nablarch-development-standards | https://github.com/nablarch-development-standards | 開発標準・規約・コーディングルール |
| Fintan-contents | https://github.com/Fintan-contents | TIS Fintanの実装事例・拡張コンテンツ |

### 新しいリポジトリを発見したときの確認コマンド

```bash
gh api orgs/<org-name> | jq '.login, .description'
# 各 org に直接 API を叩いて独立組織として確認すること
```

## レビュー観点作成チェックリスト

観点を追加・修正する際は以下を必ず確認してください:

1. 実装者が Java コードを書く箇所か（FW が内部処理する箇所でないか）
2. FW が自動でやってくれることを「実装者が確認すべき項目」として記載していないか
3. 根拠となる公式ドキュメント URL は実在するか（curl で 200 確認）
4. NG 例・OK 例のコードは文法的に正しいか
5. 観点がプロジェクト固有事情に依存していないか（後述の普遍性基準を満たすか）

> **責任境界の詳細**: `responsibility-boundary.md` を参照してください。
> FW が自動処理する範囲（実装者が書かなくてよいこと）を処理方式別に整理しています。

## プリセット観点の普遍性基準

本リポジトリの `review-perspectives/` 配下に格納する観点プリセットは、
「プロジェクト固有色のないベース観点」を維持するため、以下の基準を満たすこと。

### KEEP 判定基準（以下を全て満たすこと）

- **a**: `responsibility = developer` であること
- **b**: Nablarch の該当処理方式（バッチ/Web/REST）を使う限り、実装者が必ず触れる/書くコード/設定であること
- **c**: 特定ライブラリ・拡張・カスタム実装に依存しないこと
- **d**: プロジェクト固有事情（業務要件・採用判断）に依存しないこと

### REMOVE 判定基準（いずれか該当）

- **REMOVE-a**: 「カスタム◯◯を実装する場合のみ」適用される観点
- **REMOVE-b**: 「特定の拡張機能を使う場合のみ」適用される観点
- **REMOVE-c**: 「◯◯ライブラリ/モジュールを採用する場合のみ」適用される観点
- **REMOVE-d**: 「業務要件で◯◯が必要な場合のみ」適用される観点
- **REMOVE-e**: `responsibility = framework` の観点

### 観点追加時のレビュー手順

1. `triggering_condition` フィールドを記述する
   - 許容される書き方:「Nablarch の `<バッチ|Web|REST>` 機能を使う」形式のみ
   - 禁止する書き方:「カスタム◯◯を実装する場合」「特定機能を使う場合」「業務要件で◯◯が必要な場合」等
2. 観点本文（`detail`, `check_method`, `review_target`）にプロジェクト固有性を示すキーワードが含まれていないか確認
   - 禁止キーワード例: 「カスタム」「拡張機能」「採用する場合」「業務要件で」「特定の◯◯を使う」
3. KEEP 判定基準 a〜d を全て満たすことを確認
4. `universality: universal` を付与
5. 上記を満たさない観点は `review-perspectives/project-specific-perspectives-candidates.md` に追加し、プリセット本体には含めない

### 観点数に関するスタンス

プリセット観点は「漏らせない普遍観点だけを残す」ことを目的としており、
**観点が少ないことは正しい状態**である。観点数が処理方式ごとに過去目標値
（例: 15以上）を下回ったとしても、それは普遍性フィルタが正しく機能した結果である。

「網羅性」ではなく「普遍性」を優先する設計方針を維持すること。

## 参照情報

- Nablarch公式: https://nablarch.github.io/docs/LATEST/doc/
- Nablarch GitHub Organization: https://github.com/nablarch
- nablarch-development-standards Organization: https://github.com/nablarch-development-standards
- Fintan: https://fintan.jp/page/1868/
- Fintan-contents Organization: https://github.com/Fintan-contents
