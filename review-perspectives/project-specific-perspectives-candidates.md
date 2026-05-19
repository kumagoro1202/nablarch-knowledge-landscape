# プロジェクト固有観点候補リスト（cmd_472で除外）

本ファイルはNablarch観点プリセット（review-perspectives/batch.md, web.md, rest.md）から、
「プロジェクト固有事情に依存する」と判定して除外した観点のアーカイブである。

除外時の判定基準（REMOVE判定基準）:
- REMOVE-a: 「カスタム◯◯を実装する場合のみ」
- REMOVE-b: 「特定の拡張機能を使う場合のみ」
- REMOVE-c: 「◯◯ライブラリ/モジュールを採用する場合のみ」
- REMOVE-d: 「業務要件で◯◯が必要な場合のみ」
- REMOVE-e: responsibility = framework

これらの観点はプロジェクト固有のレビュー観点リスト作成時に再利用可能。
普遍観点プリセットからは除外したが、特定プロジェクトの状況に応じて選択的に追加することを想定。

除外件数: 18 件（batch:10, web:3, rest:5）

---

## BATCH-001: カスタムExecutionListener実装時の確認（追加実装がある場合のみ）

**カテゴリ**: batch

**観点詳細**: Nablarchバッチフレームワークは標準のExecutionListenerを提供しており、通常の前後処理はFWの標準実装が担当するため、実装者がExecutionListenerをimplementsする必要はない。プロジェクト固有の前後処理（外部APIへの通知・カスタムログ出力等）でカスタムExecutionListenerを追加実装した場合のみ、その実装の妥当性を確認する。

**除外理由コード**: REMOVE-a

**除外理由**: 「カスタムExecutionListernを追加実装した場合のみ」確認する観点であり、追加実装するか否かはプロジェクト判断

**将来の活用可能性**: プロジェクト固有の前後処理（外部API通知・カスタムログ等）を追加実装するプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/getting_started/nablarch_batch/index.html

---

## BATCH-004: 一時エラー時のリトライ設計とスキップ制御

**カテゴリ**: batch

**観点詳細**: 一時的なエラー（デッドロック・タイムアウト等）に対するリトライ制御と、スキップ可能エラーの定義が適切か確認する。無限リトライや全件スキップが起きない設計であることを確認する。

**除外理由コード**: REMOVE-d

**除外理由**: 一時エラーのリトライ要否・スキップ可能エラーの定義はプロジェクト業務要件に依存。リトライ不要のバッチ・全件停止方針のバッチでは対象外

**将来の活用可能性**: デッドロック/タイムアウトを想定する長時間バッチ・大量データバッチを持つプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html

---

## BATCH-005: チャンクサイズ・ストリーミング処理によるメモリ制御

**カテゴリ**: batch

**観点詳細**: バッチ処理で大量レコードをメモリ上に一括ロードしていないか確認する。DataReader がストリーミング（逐次読み込み）で実装されているか、または適切なチャンクサイズが設定されているか。

**除外理由コード**: REMOVE-a

**除外理由**: カスタムDataReader実装時の確認観点（read()メソッドのストリーミング実装）が中心であり、標準DataReaderのみ使用するプロジェクトでは対象外

**将来の活用可能性**: 大量データ処理でカスタムDataReaderを実装するプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html

---

## BATCH-007: バッチ中断時のチェックポイント・再開ポイント設計

**カテゴリ**: batch

**観点詳細**: バッチが途中で停止した場合に、どこから再開するかのチェックポイント設計が存在するか。無設計の場合、全件再処理でデータ二重登録が発生する可能性がある。

**除外理由コード**: REMOVE-d

**除外理由**: チェックポイント/再開ポイント設計は長時間バッチでの業務要件。短時間バッチでは不要

**将来の活用可能性**: 数時間〜数日の長時間バッチを持つプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html

---

## BATCH-009: バッチ入力レコードのバリデーション実装確認

**カテゴリ**: batch

**観点詳細**: バッチ入力データ（ファイル・DB）に対するバリデーションが実装されているか。バリデーションエラーレコードの扱い（スキップ・エラーファイル出力）が仕様通りか確認する。

**除外理由コード**: REMOVE-d

**除外理由**: 外部ファイル/外部DB入力に対するバリデーションは入力形態に依存する業務要件。社内DB完結のバッチでは不要な場合がある

**将来の活用可能性**: 外部システム連携・ファイル入力を持つバッチプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/validation.html

---

## BATCH-010: 同一バッチの同時実行防止機構の確認

**カテゴリ**: batch

**観点詳細**: 同一バッチジョブが同時に複数起動された場合に、二重処理が発生しない排他制御が実装されているか確認する。

**除外理由コード**: REMOVE-b

**除外理由**: nablarch-common-exclusivecontrol等の排他制御モジュールを採用する場合のみ確認。排他要件のないバッチでは対象外

**将来の活用可能性**: 金融・会計系で二重実行が致命的なプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/exclusive_control.html

---

## BATCH-011: 固定長・CSV ファイルの文字コード設定確認

**カテゴリ**: batch

**観点詳細**: ファイル形式の入出力で文字コード（UTF-8/Shift-JIS 等）が明示的に指定されているか。デフォルトエンコーディングに依存していると OS・JVM 設定によって文字化けが発生する。

**除外理由コード**: REMOVE-d

**除外理由**: 固定長/CSVファイル入出力を行うバッチ限定の観点。DB入出力のみのバッチでは不要

**将来の活用可能性**: ファイル入出力を行うバッチプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/data_io/data_format.html

---

## BATCH-016: ファイルフォーマット定義の項目網羅性・型定義確認

**カテゴリ**: batch

**観点詳細**: フォーマット定義ファイル（.fmt）の項目定義がファイル仕様書と完全に一致しているか。項目の型・桁数・必須/任意が正確に定義されているか確認する。

**除外理由コード**: REMOVE-d

**除外理由**: ファイルフォーマット定義（DataConvertor）はファイル入出力を行う場合のみ必要

**将来の活用可能性**: ファイル入出力を行うバッチプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/data_io/data_format.html

---

## BATCH-017: バッチ処理時間の設計目標値と実測値の妥当性確認

**カテゴリ**: batch

**観点詳細**: バッチの処理時間が業務 SLA（処理ウィンドウ）に収まる設計になっているか。ボトルネック（DB I/O・外部連携）の分析と改善策が検討されているか確認する。

**除外理由コード**: REMOVE-d

**除外理由**: 処理時間の設計目標値はプロジェクト業務要件（処理ウィンドウ・SLA）に依存

**将来の活用可能性**: SLA合意のあるバッチプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/feature_details.html

---

## BATCH-018: カスタムDataReader実装時のclose()業務後処理確認

**カテゴリ**: batch

**観点詳細**: FWはDataReader.open()とclose()の呼び出しを自動管理（DataReadHandler）するため、実装者がopen/closeを呼ぶコードを書く必要はない。ただし、カスタムDataReaderを実装した場合、closeメソッド内に業務的後処理（外部リソース解放・キャッシュクリア・一時ファイル削除等）が正しく実装されているか確認する。

**除外理由コード**: REMOVE-a

**除外理由**: カスタムDataReader実装時のclose()処理確認観点。標準DataReader使用プロジェクトでは対象外

**将来の活用可能性**: カスタムDataReaderを実装するプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/batch/nablarch_batch/architecture.html

---

## WEB-010: フォーム二重送信防止トークンの確認

**カテゴリ**: web

**観点詳細**: 更新フォームの二重サブミット（ダブルクリック・ブラウザ戻るボタン）を防ぐトークン機構が実装されているか確認する。Nablarch の DoubleSubmissionHandler 等の使用を確認する。

**除外理由コード**: REMOVE-b

**除外理由**: DoubleSubmissionHandler採用とJSP上の<n:token>タグ配置を前提とする観点。二重サブミット要件のないシステムでは対象外

**将来の活用可能性**: EC/申請系/注文系など二重実行が致命的なプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html

---

## WEB-011: ファイルアップロードのサイズ制限・拡張子検証確認

**カテゴリ**: web

**観点詳細**: ファイルアップロード機能でサイズ上限・許可拡張子の検証が実装されているか確認する。悪意あるファイル（スクリプト等）のアップロードを防止する設計か確認する。

**除外理由コード**: REMOVE-d

**除外理由**: ファイルアップロード機能を持つ画面でのみ必要な観点

**将来の活用可能性**: ファイルアップロード機能を持つプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web/feature_details.html

---

## WEB-013: 大量データ表示時のページング・データ制限の確認

**カテゴリ**: web

**観点詳細**: 検索結果を全件取得・表示しておらず、適切なページング（limit/offset 等）で件数を制限しているか確認する。Nablarch の UniversalDao のページング機能が使われているか確認する。

**除外理由コード**: REMOVE-d

**除外理由**: 一覧画面・検索結果画面でのページング設計が中心であり、当該画面を持たないシステムでは対象外

**将来の活用可能性**: 一覧/検索画面を持つプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/database/universal_dao.html

---

## REST-004: API の入出力スキーマが明確に定義・文書化されているか

**カテゴリ**: rest

**観点詳細**: REST API のリクエスト/レスポンスの JSON スキーマが、OpenAPI 仕様または同等のドキュメントで定義されているか確認する。nablarch-openapi-generator を使った自動生成が活用されているか確認する。

**除外理由コード**: REMOVE-c

**除外理由**: nablarch-openapi-generator等の特定ライブラリを採用してAPIドキュメント自動生成する場合のみ確認

**将来の活用可能性**: OpenAPIドキュメントを公開するプロジェクトでは観点として有用

**元の根拠URL**: https://github.com/nablarch/nablarch-openapi-generator

---

## REST-005: JWT / Bearer Token 認証と認可チェックの実装確認

**カテゴリ**: rest

**観点詳細**: REST API の認証が適切に実装されているか確認する。認証トークン（JWT 等）の検証、有効期限チェック、権限チェックが全保護エンドポイントに適用されているか確認する。

**除外理由コード**: REMOVE-d

**除外理由**: JWT/Bearer Token認証は採用判断に依存する認証方式。セッション認証・APIキー認証など別方式を採用するプロジェクトでは対象外

**将来の活用可能性**: JWT/OAuth系認証を採用するREST APIプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/libraries/permission_check.html

---

## REST-011: Cross-Origin Resource Sharing (CORS) の適切な設定確認

**カテゴリ**: rest

**観点詳細**: ブラウザからの AJAX リクエストを受け付ける REST API で、CORS 設定（Access-Control-Allow-Origin 等）が適切か確認する。ワイルドカード * による全オリジン許可になっていないか確認する。

**除外理由コード**: REMOVE-d

**除外理由**: ブラウザからのクロスオリジンAJAX呼び出しを受け付けるAPIでのみ必要。サーバ間連携専用APIでは対象外

**将来の活用可能性**: ブラウザクライアントを持つREST APIプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html

---

## REST-013: API エンドポイントへのリクエスト頻度制限の確認

**カテゴリ**: rest

**観点詳細**: 特定クライアントからの過剰なリクエストを制限する仕組みが設計されているか確認する。外部向け API では特に重要。

**除外理由コード**: REMOVE-d

**除外理由**: 外部公開API・パブリックAPIで必要な観点。社内専用API・内部システム間連携APIでは対象外

**将来の活用可能性**: パブリックAPI/外部公開APIを持つプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/feature_details.html

---

## REST-015: API のバージョン管理方式が一貫しているか確認

**カテゴリ**: rest

**観点詳細**: REST API のバージョン管理が URL パス（/v1/, /v2/）またはヘッダで統一された方式で実装されているか確認する。バージョン管理なしの変更が後方互換性を破壊していないか確認する。

**除外理由コード**: REMOVE-d

**除外理由**: 複数クライアント（web/mobile/外部）への提供を前提とした観点。単一クライアント専用APIでは不要

**将来の活用可能性**: 長期運用される外部公開API・複数バージョン併存を要するプロジェクトでは観点として有用

**元の根拠URL**: https://nablarch.github.io/docs/LATEST/doc/application_framework/application_framework/web_service/rest/architecture.html

---
