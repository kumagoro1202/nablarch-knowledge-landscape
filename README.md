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

> **責任境界の詳細**: `responsibility-boundary.md` を参照してください。
> FW が自動処理する範囲（実装者が書かなくてよいこと）を処理方式別に整理しています。

## 参照情報

- Nablarch公式: https://nablarch.github.io/docs/LATEST/doc/
- Nablarch GitHub Organization: https://github.com/nablarch
- nablarch-development-standards Organization: https://github.com/nablarch-development-standards
- Fintan: https://fintan.jp/page/1868/
- Fintan-contents Organization: https://github.com/Fintan-contents
