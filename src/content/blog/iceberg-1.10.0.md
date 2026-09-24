---
pubDate: '2025-09-12'
title: 'Iceberg 1.10.0リリースノート'
slug: iceberg-1.10.0
tags: ['iceberg']
---

[Apache Iceberg 1.10.0](https://github.com/apache/iceberg/releases/tag/apache-iceberg-1.10.0)がリリースされました。変更点は以下の通りです。

## Deprecation / End of Support
- Flink: バージョン1.18のサポートを削除 ([#12527](https://github.com/apache/iceberg/pull/12527))
- Common: `DynConstructors`および`DynMethods`クラスの内で非推奨となっていたメソッドを`public`からパッケージプライベートへ変更 ([#13053](https://github.com/apache/iceberg/pull/13053))
- AWS / Core / Flink / Parquet: 1.10.0対象の非推奨のコードを削除 ([#12909](https://github.com/apache/iceberg/pull/12909))

## Behavior change
- Hive: 存在しないネームスペースをリストすると`NoSuchNamespaceException`をスローするよう変更 ([#13130](https://github.com/apache/iceberg/pull/13130))  
  ※以前は空リストが返却されていた
- Core: パーティション統計ファイルのフィールドIDがスペックに準拠していたなかったため修正 ([#13329](https://github.com/apache/iceberg/pull/13329))

## Spec
- Table: 孤立したDeletion Vectorファイルを防ぐための書き込み要件を明確化 ([#13042](https://github.com/apache/iceberg/pull/13042))
- Table: `geometry`/`geography`オブジェクトの最大値・最小値の挙動を明確化 ([#12956](https://github.com/apache/iceberg/pull/12956))
- Table: 暗号化キーを追加 ([#12162](https://github.com/apache/iceberg/pull/12162))
- Table: `struct`型のフィールドでカラムのデフォルト値が競合する問題を回避 ([#12841](https://github.com/apache/iceberg/pull/12841))
- REST: Row Lineageに関するフィールド（`first-row-id`, `next-row-id`）を追加 ([#13010](https://github.com/apache/iceberg/pull/13010))
- REST: 暗号化キーを追加 ([#12987](https://github.com/apache/iceberg/pull/12987))
- REST: V3テーブルではRow Lineageが常に有効であるため、Row Lineageを有効化するエンドポイントを削除 ([#12986](https://github.com/apache/iceberg/pull/12986))
- REST: 503を再試行不可としてマーク ([#13619](https://github.com/apache/iceberg/pull/13619))

## API
- 暗号化用テーブルメタデータキーを追加 ([#12927](https://github.com/apache/iceberg/pull/12927))
- `RowDelta.deleteFile`メソッドを追加 ([#12861](https://github.com/apache/iceberg/pull/12861))
- `ExpireSnapshots.cleanExpiredMetadata`メソッドを追加 ([#13509](https://github.com/apache/iceberg/pull/13509))
- メトリクスの最大値・最小値で元の型を保持 ([#13695](https://github.com/apache/iceberg/pull/13695))
- identityパーティションでの`timestamp(9)`を修正 ([#13746](https://github.com/apache/iceberg/pull/13746))
- timestampリテラル用のExpressionを作成するためのメソッドを追加 ([#13747](https://github.com/apache/iceberg/pull/13747))

## Core
- JDBCカタログで起動中のテーブル作成時の競合を修正 ([#13345](https://github.com/apache/iceberg/pull/13345))
- RESTカタログ初期化失敗時にリソースをクローズ ([#13384](https://github.com/apache/iceberg/pull/13384))
- Core: パーティション統計ファイルのフィールドIDがスペックに準拠していたなかったため修正 ([#13329](https://github.com/apache/iceberg/pull/13329))
- パーティション進化やパーティションにNULLが含まれている際に、`PartitionsTable`が不完全なリストを返す問題を修正 ([#12528](https://github.com/apache/iceberg/pull/12528))
- パーティション統計情報のインクリメンタル更新をサポート ([#12629](https://github.com/apache/iceberg/pull/12629))
- ファイル書き換え時の最大数を設定するオプションを追加 ([#12824](https://github.com/apache/iceberg/pull/12824))
- Parquetファイルのカラム統計を有効化する`write.parquet.stats-enabled.column.`テーブルプロパティを追加 ([#12770](https://github.com/apache/iceberg/pull/12770))
- 現在のスキーマで削除されたパーティションフィールドを無視 ([#11868](https://github.com/apache/iceberg/pull/11868))
- データファイル書き換え時に孤立したDeletion Vectorを削除 ([#13245](https://github.com/apache/iceberg/pull/13245))
- パーティション統計でDeletion Vectorをサポート ([#13425](https://github.com/apache/iceberg/pull/13425))
- 孤立したDeletion Vectorをクリーンアップするため削除対象データファイルを追跡 ([#13222](https://github.com/apache/iceberg/pull/13222))
- `BaseTransaction`でコミットがされなかったファイル削除に`SupportsBulkOperations`を使用 ([#13653](https://github.com/apache/iceberg/pull/13653))
- `BaseFile`の`equalityFieldIds`にゼロコピーラッパーを使用 ([#13212](https://github.com/apache/iceberg/pull/13212))
- `BaseDVFileWriter`が空のPuffinファイルを作成しないよう修正 ([#13666](https://github.com/apache/iceberg/pull/13666))
- `ExpireSnapshots`でまだ参照されているデータファイルが誤って削除される問題を修正 ([#13614](https://github.com/apache/iceberg/pull/13614))
- `write.metadata.metrics.max-inferred-column-defaults`プロパティがネストカラムを考慮するよう修正 ([#13039](https://github.com/apache/iceberg/pull/13039))
- `timestamp(9)`を`SingleValueParser`でサポート ([#13487](https://github.com/apache/iceberg/pull/13487))
- `FileURI`および`FileSystemWalker`クラスを追加 ([#13429](https://github.com/apache/iceberg/pull/13429))
- Cherry-PickもしくはFast-Forwardでパーティションを検証する際に`ParallelIterable`で新規ファイルをバッチロードするよう修正 ([#13556](https://github.com/apache/iceberg/pull/13556))
- `FindFiles`クラスで`ParallelIterable`を利用できるよう`planWith`メソッドを追加 ([#13836](https://github.com/apache/iceberg/pull/13836))
- REST: `ParserContext`クラスを追加 ([#13191](https://github.com/apache/iceberg/pull/13191))
- REST: HTTPクライアントでユーザーエージェントを設定できるプロパティを追加 ([#13234](https://github.com/apache/iceberg/pull/13234))
- REST: RESTクライアントで TLS 設定を構成できるオプションを追加 ([#13190](https://github.com/apache/iceberg/pull/13190))
- REST: テーブルが破損しないよう502や504での再試行を停止 ([#13352](https://github.com/apache/iceberg/pull/13352))
- REST: RESTクライアントでHTTPプロキシをサポート ([#12406](https://github.com/apache/iceberg/pull/12406))
- REST: `RESTMetricsReporter`クラスにおけるメトリクスのアナウンスを非同期化 ([#13507](https://github.com/apache/iceberg/pull/13507))
- REST: 一部のステータスコードに対して冪等なリクエストの再試行を許可 ([#13449](https://github.com/apache/iceberg/pull/13449))
- REST: 認証情報を更新する際の`ScheduledExecutorService`を追加 ([#12563](https://github.com/apache/iceberg/pull/12563))
- REST: OAuth 2.0のToken Exchangeを無効化できるよう`token-exchange-enabled`プロパティを追加 ([#13809](https://github.com/apache/iceberg/pull/13809))
- REST: スキャンプランニング用のリクエスト/レスポンスモデルとパーサーを追加 ([#13004](https://github.com/apache/iceberg/pull/13004))

## Arrow
- Parquet V2をサポートするようリファクタリング ([#13290](https://github.com/apache/iceberg/pull/13290))
- `DELTA_BINARY_PACKED` Parquetエンコーディングをサポート ([#13391](https://github.com/apache/iceberg/pull/13391))
- `timestamp(9)`型をサポート ([#13562](https://github.com/apache/iceberg/pull/13562))
- 異なるエンコードが混在しているParquetファイルをベクトル化されたリーダーで読み込んだ際にネイティブメモリがリークする問題を修正 ([#13935](https://github.com/apache/iceberg/pull/13935))

## Parquet
- Parquet 1.16.0で追加された`LogicalTypeAnnotation`をVariant型で使用 ([#13941](https://github.com/apache/iceberg/pull/13941))

## Spark
- Spark 4.0をサポート ([#12494](https://github.com/apache/iceberg/pull/12494))
- V3: AvroリーダーでRow Lineageをサポート ([#13070](https://github.com/apache/iceberg/pull/13070))
- V3: Parquetベクトル化されたリーダーでRow Lineageをサポート ([#12928](https://github.com/apache/iceberg/pull/12928))
- `streaming-max-rows-per-micro-batch`をソフトリミットに変更 ([#12988](https://github.com/apache/iceberg/pull/12988))
- V3: 分散プランニングにおけるRow Lineageを修正 ([#13061](https://github.com/apache/iceberg/pull/13061))
- Storage Partition Join (SPJ)にバケットおよび時間単位のリデューサーを追加 ([#13166](https://github.com/apache/iceberg/pull/13166), [#13167](https://github.com/apache/iceberg/pull/13167))
- `RewriteTablePath`のインクリメンタルモードでスナップショットIDによるファイルをフィルタリングするよう修正 ([#12885](https://github.com/apache/iceberg/pull/12885))
- identifierフィールドを使用しているテーブルでDMLが失敗する問題を修正 ([#13535](https://github.com/apache/iceberg/pull/13435))
- パーティション統計情報を取得する`ComputePartitionStatsSparkAction`アクションと`compute_partition_stats`プロシージャを追加 ([#12450](https://github.com/apache/iceberg/pull/12450), [#13480](https://github.com/apache/iceberg/pull/13480))
- `ADD COLUMN`でカラムのデフォルト値が指定されると例外をスローするよう修正 ([#13464](https://github.com/apache/iceberg/pull/13464))
- 4.0: IcebergのストアドプロシージャをSparkのビルトインの実装へと以降 (引数がCase Sensitiveになり、Type Coercionがより厳格になります) ([#13106](https://github.com/apache/iceberg/pull/13106))
- `RewriteTablePathSparkAction`でファイルリスト書き込みにIcebergの`FileIO`を使用 ([#13459](https://github.com/apache/iceberg/pull/13459))
- ParquetファイルでDictionary EncodingされたUUID型でをサポート ([#13324](https://github.com/apache/iceberg/pull/13324))
- 4.0: Row Lineageをサポート ([#13310](https://github.com/apache/iceberg/pull/13310))
- パーティションからインポートするマニフェストの削除に`SupportsBulkOperations`を使用 ([#13620](https://github.com/apache/iceberg/pull/13620))
- コンパクション時にRow Lineageを保持するよう修正 ([#13555](https://github.com/apache/iceberg/pull/13555))
- Variant型の読み取りをサポート ([#13219](https://github.com/apache/iceberg/pull/13219))
- ファイルを削除する際のExecutorキャッシュを無効化する`spark.sql.iceberg.executor-cache.delete-files.enabled`プロパティを追加 ([#12893](https://github.com/apache/iceberg/pull/12893))
- `RewriteManifest`でパーティションのソート順を指定可能に ([#12840](https://github.com/apache/iceberg/pull/12840))
- 4.0: `Unknown`型の読み書きをサポート ([#13445](https://github.com/apache/iceberg/pull/13445))

## Flink
- Flink 2.0をサポート ([#12527](https://github.com/apache/iceberg/pull/12527))
- 動的シンク: 動的スキーマ・パーティション進化・テーブル作成対応 ([#12424](https://github.com/apache/iceberg/pull/12424))
- `TableSchema`から`ResolvedSchema`への移行 ([#13072](https://github.com/apache/iceberg/pull/13072))
- IcebergSink v2: デフォルトのライタータスク並列度を入力ストリームと同じ値になるよう設定 ([#13260](https://github.com/apache/iceberg/pull/13260))
- v2シンクでコンパクションをサポート ([#12979](https://github.com/apache/iceberg/pull/12979))
- v2シンクでデータファイルの書き換えをサポート ([#11497](https://github.com/apache/iceberg/pull/11497))
- RANGEディストリビューションを v2シンクに移植 ([#12071](https://github.com/apache/iceberg/pull/12071))
- Zookeeperロックをテーブルメンテナンスでサポート ([#12810](https://github.com/apache/iceberg/pull/12810))
- `RewriteDataFiles`で書き換え対象を指定するためのフィルターを追加 ([#13669](https://github.com/apache/iceberg/pull/13669))
- `JdbcLockFactory`の`ResultSet`リソースがリークする問題を修正 ([#13821](https://github.com/apache/iceberg/pull/13821))
- テーブルメンテナンスで孤立ファイル削除をサポート ([#13302](https://github.com/apache/iceberg/pull/13302))

## Hive
- 存在しないネームスペースをリストすると`NoSuchNamespaceException`をスローするよう変更 ([#13130](https://github.com/apache/iceberg/pull/13130))

## Kafka Connect
- [CVE-2025-48734](https://nvd.nist.gov/vuln/detail/CVE-2025-48734)を修正 ([#13561](https://github.com/apache/iceberg/pull/13561))

## Vendor Integrations
- AWS/GCP: プレフィックス付きクライアントを初期化する際のダブルチェックロックによる競合を修正 ([#13276](https://github.com/apache/iceberg/pull/13276))
- AWS/GCP: 空のイミュータブルコレクションによって`FileIO`がシリアライズできない問題を修正 ([#13216](https://github.com/apache/iceberg/pull/13216))
- AWS: 複数ストレージ認証プレフィックスをサポート ([#12799](https://github.com/apache/iceberg/pull/12799))
- AWS: S3クライアントに`LegacyMd5Plugin`を追加 ([#12264](https://github.com/apache/iceberg/pull/12264))
- AWS: `S3V4RestSignerClient`が認証セッションを過剰に作成する問題を修正 ([#13215](https://github.com/apache/iceberg/pull/13215))
- AWS: `KeyManagementClient`実装を追加 ([#13136](https://github.com/apache/iceberg/pull/13136))
- AWS: `deleteOnExit`メソッドによるメモリリークの修正 ([#13749](https://github.com/apache/iceberg/pull/13749))
- AWS/Azure: `S3InputStream.readFully`メソッドによるコネクションリークの修正 ([#13899](https://github.com/apache/iceberg/pull/13899), [#13905](https://github.com/apache/iceberg/pull/13905))
- Azure: 認証情報が同時に更新された際に発生するエラーを修正 ([#13730](https://github.com/apache/iceberg/pull/13730))
- Azure: `adls.token`プロパティを利用したアクセストークン認証をサポート ([#13825](https://github.com/apache/iceberg/pull/13825))
- GCP: BigQueryメタストアカタログを追加 ([#12808](https://github.com/apache/iceberg/pull/12808))
- GCP: 複数ストレージ認証プレフィックスをサポート ([#12881](https://github.com/apache/iceberg/pull/12881))
- GCP: Google認証サポートの追加 ([#13212](https://github.com/apache/iceberg/pull/13212))
- GCP: `KeyManagementClient`実装 ([#13334](https://github.com/apache/iceberg/pull/13334))

## Dependencies
- Parquet: 1.15.1 → 1.16.0
- Jackson: 2.19.0 → 2.19.1
- AWS SDK: 2.31.30 → 2.31.63
- Netty: 4.2.1.Final → 4.2.2.Final
- Comet: 0.5.0 → 0.8.1
- Apache httpclient: 5.4.3 → 5.4.4  
