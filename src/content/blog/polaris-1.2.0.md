---
pubDate: 2025-10-23
title: Polaris 1.2.0リリースノート
slug: polaris-1.2.0
tags:
  - iceberg
  - polaris
---
Apache Polaris (incubating) 1.2.0が10月23日にリリースされました！原文はこちらです。

https://polaris.apache.org/downloads/#120-release1

## アップグレードに関する注意点
- Amazon RDS プラグインが有効化されました。  
  これにより、Polaris が AWS Aurora PostgreSQLに対してIAM 認証を使用して接続できるようになりました。

## 互換性のない変更
- カスタムロケーションを持つネームスペースの作成・変更  
  親ロケーション外に独自のロケーションを指定する操作は、デフォルトで禁止されるようになりました。  
  以前の動作を維持したい場合は、`ALLOW_NAMESPACE_CUSTOM_LOCATION` を `true` に設定してください。

## 新機能
- UpdateTable リクエストに対する、より細かい権限管理を追加。  
  既存の `TABLE_WRITE_PROPERTIES` などの権限は引き続き利用できますが、  
  `TABLE_ADD_SNAPSHOT` のように、特定の操作単位で権限を付与できるようになりました。

- Management API に「プリンシパル認証情報リセット」エンドポイントを追加。
  機能フラグ `ENABLE_CREDENTIAL_RESET`（デフォルト: `true`）で有効化を制御できます。

- フェデレーテッドカタログに対するサブカタログ RBAC をサポート。  
  新たに `ENABLE_SUB_CATALOG_RBAC_FOR_FEDERATED_CATALOGS` フラグを追加しました。  
  現時点ではネームスペースとテーブルが対象です。  
  カタログ単位では `polaris.config.enable-sub-catalog-rbac-for-federated-catalogs` プロパティで設定できます。  
  また、realm レベルでの設定可否は  
  `ALLOW_SETTING_SUB_CATALOG_RBAC_FOR_FEDERATED_CATALOGS`（デフォルト: `true`）で制御されます。

- STSを持たない S3 互換ストレージのサポートを追加。
  カタログストレージ設定で `stsUnavailable: true` を指定してください。

- **イベント永続化（プレビュー）を追加。**  
  新しいイベントタイプを導入し、リレーショナル JDBC 永続化およびAWS CloudWatchへのイベント保存に対応しました。  
  **注意:** この機能はプレビュー段階です。  
  今後のリリースでスキーマが変更される可能性があり、以前のイベントデータがアップグレード後に読み込めなくなる（削除される）場合があります。

## 変更点
- 以下の API が、成功時（201レスポンス）に新しく作成されたオブジェクトを返すようになりました。
    - `createCatalog`
    - `createPrincipalRole`
    - `createCatalogRole`

## 非推奨事項
- `polaris.active-roles-provider.type` プロパティは非推奨となり、今後は無効です。
- EclipseLinkを用いた永続化の実装は 1.0.0 から非推奨となっており、1.3.0 または 2.0.0（先に到達する方）で完全に削除されます。
- 旧管理エンドポイント `/metrics` および `/healthcheck`は 1.2.0 で非推奨となり、1.3.0 または 2.0.0 で削除予定です。今後は標準の `/q/metrics` および `/q/health` エンドポイントを使用してください。
