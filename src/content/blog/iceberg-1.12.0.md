---
pubDate: '2026-09-29'
title: 'Iceberg 1.12.0リリースノート'
slug: iceberg-1.12.0
tags: ['iceberg']
---

[Apache Iceberg 1.12.0](https://github.com/apache/iceberg/releases/tag/apache-iceberg-1.12.0)がリリースされました。主な変更点は以下の通りです。

## Deprecation / End of Support
- Spark: Spark 3.4のサポートを削除 ([#14122](https://github.com/apache/iceberg/pull/14122))
- Spark: 非推奨の`SparkFilters`を削除 ([#17702](https://github.com/apache/iceberg/pull/17702))
- Spark 4.0, 4.1: 非推奨の`SparkTableUtil`のメソッドを削除 ([#17703](https://github.com/apache/iceberg/pull/17703))
- Spark: 非推奨の`SparkReadConf`、`SparkWriteConf`、`SparkSchemaUtil`のメソッドを削除 ([#17626](https://github.com/apache/iceberg/pull/17626))
- Flink: Flink 2.0のサポートを削除
- Flink: 非推奨の`RewriteDataFiles.Builder.filter(Expression)`を削除 ([#17624](https://github.com/apache/iceberg/pull/17624))
- Core: 非推奨のREST namespaceエンコーディングヘルパーを削除 ([#17697](https://github.com/apache/iceberg/pull/17697))
- Core: 非推奨の`HadoopFileIO(SerializableSupplier)`コンストラクタを削除 ([#17704](https://github.com/apache/iceberg/pull/17704))
- Core, ORC: 非推奨のpartition stats読み込み機能を削除 ([#14998](https://github.com/apache/iceberg/pull/14998))
- 非推奨の`DataReader`を削除し`PlannedDataReader`に統一 ([#17699](https://github.com/apache/iceberg/pull/17699))
- 1.12.0での削除が予定されていた非推奨のメソッドとフィールドを削除 ([#17700](https://github.com/apache/iceberg/pull/17700))
- Data: 非推奨の`GenericAppenderFactory`と`BaseFileWriterFactory`を削除 ([#17696](https://github.com/apache/iceberg/pull/17696))
- AWS: 非推奨のS3署名クラスとプロパティを削除 ([#17627](https://github.com/apache/iceberg/pull/17627))
- Kafka Connect: 非推奨の`TableReference`と`IcebergWriterResult`のメンバーを削除 ([#17623](https://github.com/apache/iceberg/pull/17623))
- BigQuery: 非推奨のカタログプロパティ定数を削除 ([#17625](https://github.com/apache/iceberg/pull/17625))
- Core, Data, Spark, Flink: 行データを持つposition delete fileを削除 ([#17706](https://github.com/apache/iceberg/pull/17706))

## Behavior change
- `GeometryType`と`GeographyType`の`toString()`で、解決済みのCRS（geographyの場合はedgeアルゴリズムも含む）を常に表示するよう変更。`geometry`は`geometry(OGC:CRS84)`、`geography`は`geography(OGC:CRS84, spherical)`と表示されます ([#16765](https://github.com/apache/iceberg/pull/16765))。
  以前はデフォルトインスタンスの場合、型名のみ（`geometry` / `geography`）が表示されていました。
- デフォルトのAWS SDK HTTPクライアントがApache HttpClient 5に移行しました。AWSの依存関係を個別に指定しているユーザーは、`software.amazon.awssdk:apache-client`から`software.amazon.awssdk:apache5-client`に切り替える必要があります ([#18195](https://github.com/apache/iceberg/pull/18195))。
- RESTクライアントが、`Idempotency-Key`付きのPOSTリクエストについて、リトライ可能なエラー（408、500、502、503、504）の場合は再試行するようになりました ([#17947](https://github.com/apache/iceberg/pull/17947))。

## Spec
- 式（expressions）に関する仕様を追加 ([#16652](https://github.com/apache/iceberg/pull/16652))
- loadTableにおけるより細かい読み取り制限を追加 ([#13879](https://github.com/apache/iceberg/pull/13879))
- v4仕様に相対パスを追加 ([#15630](https://github.com/apache/iceberg/pull/15630))
- 仕様にcontent statsを追加 ([#14234](https://github.com/apache/iceberg/pull/14234))
- UDF定義モデルにオプションのspecific-nameを追加 ([#16727](https://github.com/apache/iceberg/pull/16727))
- variant型の分類とprimitive型のスコープを明確化 ([#16836](https://github.com/apache/iceberg/pull/16836))
- decimal型のシリアライズを明確化 ([#16798](https://github.com/apache/iceberg/pull/16798))
- スナップショット内でのcontent fileの一意性を明確化 ([#17198](https://github.com/apache/iceberg/pull/17198))

## API
- geometryとgeographyの単一値バイナリシリアライズを追加 ([#16607](https://github.com/apache/iceberg/pull/16607))
- `RepairTable`アクションインターフェースを定義 ([#17399](https://github.com/apache/iceberg/pull/17399))
- variantクラスをシリアライズ可能に変更 ([#17260](https://github.com/apache/iceberg/pull/17260))
- 不正なデータが渡された場合でもvariantバイナリパースが堅牢に動作するよう改善 ([#16568](https://github.com/apache/iceberg/pull/16568))
- `CatalogObjectIdentifier`を追加 ([#16160](https://github.com/apache/iceberg/pull/16160))
- partition statistics scan APIに対してproject()を実装 ([#16569](https://github.com/apache/iceberg/pull/16569))
- partition statistics scan APIに対してfilter()を実装 ([#16582](https://github.com/apache/iceberg/pull/16582))
- NPEを避けるためIN/NOT_IN述語でnullに対するガードを追加 ([#17014](https://github.com/apache/iceberg/pull/17014))

## Core
- Avroでgeometryとgeographyの値を読み書き可能に ([#17119](https://github.com/apache/iceberg/pull/17119))
- 読み取り専用のMumbling bitmap実装を追加 ([#16747](https://github.com/apache/iceberg/pull/16747))
- v4 manifest readerを追加 ([#16958](https://github.com/apache/iceberg/pull/16958))
- v4 manifest readerで相対パスを解決 ([#17434](https://github.com/apache/iceberg/pull/17434))
- v4ロケーションの相対化ユーティリティを追加 ([#16174](https://github.com/apache/iceberg/pull/16174))
- Data FileとDelete Fileを橋渡しするv4 TrackedFileAdaptersを追加 ([#16100](https://github.com/apache/iceberg/pull/16100))
- TrackedFileに`format_version`フィールドを追加 ([#16952](https://github.com/apache/iceberg/pull/16952))
- V4 DeletionVectorをkey_metadataフィールドで拡張 ([#17438](https://github.com/apache/iceberg/pull/17438))
- `DataFile`経由でco-locatedなdeletion vectorを公開 ([#17928](https://github.com/apache/iceberg/pull/17928))
- V4レイアウトでのParquetおよびAvro manifestの書き込みを許可 ([#15634](https://github.com/apache/iceberg/pull/15634))
- v4 tracked file用のManifestFileアダプターを追加 ([#17932](https://github.com/apache/iceberg/pull/17932))
- サイズ閾値以下のファイルをバッファする`EagerInputFile`と`EagerInputStream`を追加 ([#16729](https://github.com/apache/iceberg/pull/16729))
- manifest content cacheでmanifest listファイルをキャッシュ ([#16762](https://github.com/apache/iceberg/pull/16762))
- Parquetの列ごとのdictionaryエンコーディング ([#16713](https://github.com/apache/iceberg/pull/16713))
- mainブランチのrefをタグに設定できないよう制限 ([#16753](https://github.com/apache/iceberg/pull/16753))
- `DelegateFileIO`として実装した暗号化IOを追加 ([#14876](https://github.com/apache/iceberg/pull/14876))
- マージ時にDV暗号化メタデータを保持 ([#15911](https://github.com/apache/iceberg/pull/15911))
- manifest list暗号化キーを、それを使用するスナップショットと共にコミット ([#17984](https://github.com/apache/iceberg/pull/17984))
- どのデータファイルからも参照されなくなったdelete fileを削除するscanベースのアクションを追加 ([#15727](https://github.com/apache/iceberg/pull/15727))
- manifest内の重複ファイル削除時のスレッド競合を修正 ([#16686](https://github.com/apache/iceberg/pull/16686))
- row lineageのlast updated sequence継承を修正 ([#17039](https://github.com/apache/iceberg/pull/17039))
- snapshot-logの順序を前提としないよう、time-travelのスナップショット検索を修正 ([#17360](https://github.com/apache/iceberg/pull/17360))
- 否定されたall_manifestsフィルタのプルーニングを修正 ([#17346](https://github.com/apache/iceberg/pull/17346))
- entries metadata tableでの誤ったdelete manifestプルーニングを修正 ([#17440](https://github.com/apache/iceberg/pull/17440))
- 同一Puffinファイル内のDVに対するdelete fileの参照を修正 ([#17497](https://github.com/apache/iceberg/pull/17497))
- データファイルが複数スナップショットにまたがる複数DVを持つ場合のコミット検証を修正 ([#17764](https://github.com/apache/iceberg/pull/17764))
- スナップショット失効時にdelete manifestを正しく読み込むよう修正 ([#17763](https://github.com/apache/iceberg/pull/17763))
- position deleteまたはDVがデータファイルのパーティションと一致しない場合にスキャンを失敗させるよう修正 ([#16957](https://github.com/apache/iceberg/pull/16957))
- 浮動小数点値に対するZ-orderバイトエンコーディングを修正 ([#17071](https://github.com/apache/iceberg/pull/17071))
- residualsを無視する場合でもmanifest contentのプルーニングが維持されるよう修正 ([#17443](https://github.com/apache/iceberg/pull/17443))
- フィールドが削除された過去のsort orderで`SerializableTable.sortOrders()`が例外を投げる問題を修正 ([#16521](https://github.com/apache/iceberg/pull/16521))
- `RESTMetricsReporter.report()`が呼び出し元スレッドをブロックする問題を修正 ([#16695](https://github.com/apache/iceberg/pull/16695))
- load tableおよびviewレスポンスでcatalog labelsを読み込み可能に ([#18045](https://github.com/apache/iceberg/pull/18045))
- `SupportsLabels`経由でロード済みテーブルのcatalog labelsを公開 ([#18046](https://github.com/apache/iceberg/pull/18046))
- 有効なrewriteオプションに`max-file-group-input-files`を追加 ([#17544](https://github.com/apache/iceberg/pull/17544))

## Arrow
- 直接ByteBuffer向けのdict-encodedなVARCHAR/VARBINARY読み込みを修正 ([#17055](https://github.com/apache/iceberg/pull/17055))
- row lineageのベクトル化readerにおける直接メモリリークを修正 ([#17296](https://github.com/apache/iceberg/pull/17296))
- int-to-long昇格時のベクトル化readerでの`ClassCastException`を修正 ([#16343](https://github.com/apache/iceberg/pull/16343))
- デフォルト値を持つdecimal列のベクトル化読み込みを修正 ([#16501](https://github.com/apache/iceberg/pull/16501))
- 精度が18を超えるdecimalの切り捨てを修正 ([#16627](https://github.com/apache/iceberg/pull/16627))
- 全てnullのDELTAエンコードされたParquetページのベクトル化読み込みを修正 ([#17017](https://github.com/apache/iceberg/pull/17017))
- Arrow dictionaryデコードにおけるint96タイムスタンプオフセットを修正 ([#16435](https://github.com/apache/iceberg/pull/16435))

## Parquet
- variant shreddingで、フィールドの型が揃っている場合のみshreddingするよう変更 ([#17424](https://github.com/apache/iceberg/pull/17424))
- Parquetでgeometryとgeography WKB値を読み書き可能に ([#16982](https://github.com/apache/iceberg/pull/16982))
- variant shredded boundsに対して列メトリクスの切り詰め長を考慮 ([#17342](https://github.com/apache/iceberg/pull/17342))
- variant STRINGメトリクスの境界がUTF-8バイト順を使用するよう修正 ([#17397](https://github.com/apache/iceberg/pull/17397))
- variant BINARYの上限値が切り上げで切り詰められるよう修正 ([#16880](https://github.com/apache/iceberg/pull/16880))
- value列に統計情報がない場合のvariantメトリクスのクラッシュを修正 ([#16585](https://github.com/apache/iceberg/pull/16585))
- 大きいdecimal（精度18超）のvariant shreddingを修正 ([#17002](https://github.com/apache/iceberg/pull/17002))
- 適応型bloom filterサイズ調整を追加 (PARQUET-2254) ([#16363](https://github.com/apache/iceberg/pull/16363))
- オプトインの非圧縮row groupサイズ追跡を追加 ([#16327](https://github.com/apache/iceberg/pull/16327))
- timestamp_nsおよびtimestamptz_nsの述語プッシュダウンを修正 ([#16619](https://github.com/apache/iceberg/pull/16619))
- 祖先のstructがnullの場合にネストした初期デフォルト値が適用される問題を修正 ([#17320](https://github.com/apache/iceberg/pull/17320))
- オプショナルなstruct内のネストフィールドの誤ったプルーニングを修正 ([#18069](https://github.com/apache/iceberg/pull/18069))
- `notStartsWith`がnull値を含むrow groupを誤ってスキップする問題を修正 ([#17656](https://github.com/apache/iceberg/pull/17656))
- null_count統計情報を持たないParquetファイルのnullカウントを修正 ([#17557](https://github.com/apache/iceberg/pull/17557))
- デフォルト値が設定された列でフィルタした際にinitial-defaultの行が欠落する問題を修正 ([#16692](https://github.com/apache/iceberg/pull/16692))

## ORC
- OrcMetricsでのtimestamp_ns列の上限/下限値を修正 ([#16922](https://github.com/apache/iceberg/pull/16922))
- 不正なtimestamp unit属性に対する例外メッセージの文字化けを修正 ([#17098](https://github.com/apache/iceberg/pull/17098))
- timestamp nanoの述語プッシュダウンを修正 ([#17750](https://github.com/apache/iceberg/pull/17750))
- variant列を持つテーブルでのフィルタプッシュダウンを修正 ([#17998](https://github.com/apache/iceberg/pull/17998))

## Spark
- Spark 4.0, 4.1: unshredded variant列に対するベクトル化Parquet読み込みを追加 ([#16292](https://github.com/apache/iceberg/pull/16292))
- Spark 4.1: `rewrite_data_files`にHilbert曲線クラスタリング戦略を追加 ([#16827](https://github.com/apache/iceberg/pull/16827))
- Spark 4.1: Parquetでgeometryとgeography値を読み書き可能に ([#17073](https://github.com/apache/iceberg/pull/17073))
- Spark 4.1: geometryとgeographyのSpark型をマッピング ([#16851](https://github.com/apache/iceberg/pull/16851))
- DROP TABLE PURGEをRESTカタログに委譲する`rest-catalog-purge`プロパティを追加 ([#15614](https://github.com/apache/iceberg/pull/15614))
- セッションレベルのsplitサイズ上書きを追加 ([#16154](https://github.com/apache/iceberg/pull/16154))
- migrateプロシージャにignore_missing_filesを追加 ([#16643](https://github.com/apache/iceberg/pull/16643))
- snapshotプロシージャにignore_missing_filesを追加 ([#16710](https://github.com/apache/iceberg/pull/16710))
- セッションカタログでviewを返せるよう変更 ([#16845](https://github.com/apache/iceberg/pull/16845))
- Spark 4.1: listTableSummariesを実装 ([#16891](https://github.com/apache/iceberg/pull/16891))
- Spark 3.5, 4.0, 4.1: streaming merge-append書き込み設定を追加 ([#17347](https://github.com/apache/iceberg/pull/17347), [#17403](https://github.com/apache/iceberg/pull/17403))
- Spark 3.5, 4.0, 4.1: delete fileのexecutorキャッシュを有効にするrewriteオプションを追加 ([#17868](https://github.com/apache/iceberg/pull/17868))
- null booleanに対するZ-order NPEと大文字小文字を区別しない列解決を修正 ([#17669](https://github.com/apache/iceberg/pull/17669))
- 分散プランニングモードでリネームされた列に対するtime-travelフィルタを修正 ([#16523](https://github.com/apache/iceberg/pull/16523))
- manifest rewrite時のfirst row IDの引き継ぎを修正 ([#16699](https://github.com/apache/iceberg/pull/16699))
- Spark 3.5, 4.0: `rewrite_manifests`プロシージャに`sort_by`パラメータを追加 ([#18065](https://github.com/apache/iceberg/pull/18065))
- `rewrite_manifests`で書き込まれるmanifestを暗号化 ([#17987](https://github.com/apache/iceberg/pull/17987))

## Flink
- Flink 2.2および2.3サポートを追加 ([#17849](https://github.com/apache/iceberg/pull/17849))
- equality deleteをdeletion vectorに変換するFlinkメンテナンスタスク（`ConvertEqualityDeletes`）を追加 ([#16831](https://github.com/apache/iceberg/pull/16831), [#16844](https://github.com/apache/iceberg/pull/16844), [#16858](https://github.com/apache/iceberg/pull/16858), [#16874](https://github.com/apache/iceberg/pull/16874), [#16889](https://github.com/apache/iceberg/pull/16889), [#16948](https://github.com/apache/iceberg/pull/16948))
- ConvertEqualityDeletesをIcebergSinkと統合 ([#17142](https://github.com/apache/iceberg/pull/17142))
- equality-delete変換の失敗サイクル後に削除済みの行が再出現する問題を修正 ([#17630](https://github.com/apache/iceberg/pull/17630))
- パーティション指定のないequality deleteを、全パーティションに適用されるものとして解決するよう変更 ([#17018](https://github.com/apache/iceberg/pull/17018))
- Flink 2.1, 2.2, 2.3: Avro reader/writerでvariantをサポート ([#17737](https://github.com/apache/iceberg/pull/17737))
- Flink 2.1, 2.2, 2.3: shredded variantの書き込みをサポート ([#15596](https://github.com/apache/iceberg/pull/15596))
- Flink 2.1, 2.2, 2.3: SQL variant Avro動的レコードジェネレーターを追加 ([#16450](https://github.com/apache/iceberg/pull/16450))
- SQLでのIceberg viewの読み込みをサポート ([#17859](https://github.com/apache/iceberg/pull/17859))
- FlinkCatalogでCREATE VIEW、DROP VIEW、ALTER VIEW RENAMEをサポート ([#17873](https://github.com/apache/iceberg/pull/17873))
- DynamicSinkできめ細かいリソース管理のためのslot sharing groupの設定を許可 ([#16065](https://github.com/apache/iceberg/pull/16065))
- ALTER TABLEで列を特定の位置に追加するよう修正 ([#16419](https://github.com/apache/iceberg/pull/16419))
- FlinkSQLでのテーブルコメントの扱いを修正 ([#16423](https://github.com/apache/iceberg/pull/16423))
- 再起動時にFlinkのjobIdが変わるとDynamicCommitterでコミットが重複する問題を修正 ([#16011](https://github.com/apache/iceberg/pull/16011))
- dynamic-sinkのレコードルーティングでスキーマのidentifierフィールドを考慮するよう修正 ([#16243](https://github.com/apache/iceberg/pull/16243))
- スレッド/メモリリークを修正するためwakeupメソッドを実装 ([#16545](https://github.com/apache/iceberg/pull/16545))
- TableMaintenanceオペレーターのuidがジョブ再起動のたびにランダムに変わり、savepointからの復元が壊れる問題を修正 ([#17210](https://github.com/apache/iceberg/pull/17210))
- RANGE distributionがユーザー指定のequalityフィールドを無視してしまう問題を修正 ([#17276](https://github.com/apache/iceberg/pull/17276))
- ファイルがスキップされた場合の`DataIterator.seek()`でのファイルオフセット不一致を修正 ([#16929](https://github.com/apache/iceberg/pull/16929))
- 配列およびマップ内のtimestampのマイクロ秒切り捨てを修正 ([#18001](https://github.com/apache/iceberg/pull/18001))
- DynamicIcebergSinkのDataConverterでRowKindを保持 ([#18101](https://github.com/apache/iceberg/pull/18101))
- AvroToRowDataConvertersでのtimestamp-micros変換を修正 ([#17194](https://github.com/apache/iceberg/pull/17194))
- read splitのテーブルプロパティが無視される問題を修正 ([#17445](https://github.com/apache/iceberg/pull/17445))
- retain-last未設定時のExpireSnapshots設定でのNPEを修正 ([#17277](https://github.com/apache/iceberg/pull/17277))

## Hive
- HMSのcreateTimeとlastAccessTimeにおける整数オーバーフローを修正 ([#16620](https://github.com/apache/iceberg/pull/16620))
- HiveCatalogでのIcebergテーブル一覧取得にサーバーサイドフィルタを使用 ([#17317](https://github.com/apache/iceberg/pull/17317))

## Kafka Connect
- Kafka Connectおよびgeneric Recordの書き込みに対してParquet variant shreddingを有効化 ([#17520](https://github.com/apache/iceberg/pull/17520))
- オフセットのコミットを、既存のオフセットより大きい場合のみ行うよう変更 ([#17552](https://github.com/apache/iceberg/pull/17552))
- コミット失敗を黙って無視せず、エラーとして検知できるよう変更 ([#16237](https://github.com/apache/iceberg/pull/16237))
- 一時的なコミット例外に対する有限リトライを追加 ([#16434](https://github.com/apache/iceberg/pull/16434))
- 一部のBigDecimal値で不正なdecimal型が推論される問題を修正 ([#16606](https://github.com/apache/iceberg/pull/16606))
- レコードスキーマが更新されても値がnullの場合にテーブルスキーマを進化させるよう修正 ([#16826](https://github.com/apache/iceberg/pull/16826))
- `IcebergSinkConfig`での`ConcurrentModificationException`を修正 ([#16438](https://github.com/apache/iceberg/pull/16438))
- UUIDのAvroスキーマ変換を修正 ([#16828](https://github.com/apache/iceberg/pull/16828))
- idColumnsがテーブル設定から正しく読み込まれない問題を修正 ([#17152](https://github.com/apache/iceberg/pull/17152))
- control topicのオフセットをhigh-water markとして追跡するよう変更 ([#17933](https://github.com/apache/iceberg/pull/17933))
- 部分的なコミット失敗のメトリクスを追加 ([#16433](https://github.com/apache/iceberg/pull/16433))
- 特定のリバランスシナリオでcoordinatorが前回コミットのファイルをコミットしてしまう問題を修正 ([#17713](https://github.com/apache/iceberg/pull/17713))

## Open API / REST
- REST catalog仕様にVariantTypeを追加 ([#17256](https://github.com/apache/iceberg/pull/17256))
- unregister tableエンドポイントを追加 ([#16400](https://github.com/apache/iceberg/pull/16400))
- OpenAPI仕様にlistおよびload functionエンドポイントを追加 ([#15180](https://github.com/apache/iceberg/pull/15180))
- リモート署名の設定をREST仕様として正式に定義 ([#16822](https://github.com/apache/iceberg/pull/16822))
- REST仕様の式（expressions）を、新しい式仕様に合わせて更新 ([#17138](https://github.com/apache/iceberg/pull/17138))
- RFC 3986のパーセントエンコーディングを使用するようパスセグメントのエンコーディングを修正 ([#15989](https://github.com/apache/iceberg/pull/15989))
- REST仕様のdata-accessオブジェクトのスキーマを修正 ([#16594](https://github.com/apache/iceberg/pull/16594))
- UDF定義にspecific-nameを追加 ([#17364](https://github.com/apache/iceberg/pull/17364))
- カタログメタデータ拡充のための`labels`フィールドを追加 ([#15750](https://github.com/apache/iceberg/pull/15750))

## Vendor integrations
- AWS: REST SigV4署名でassumed-role認証情報を使用できるよう変更 ([#16794](https://github.com/apache/iceberg/pull/16794))
- AWS: IcebergToGlueConverterのコメントマップで列名が重複する場合に対応 ([#16853](https://github.com/apache/iceberg/pull/16853))
- AWS, Core: RemoteSigningConfigを実装 ([#17709](https://github.com/apache/iceberg/pull/17709))
- Core, AWS, GCP, Dell, Hive: FileIOのリークを修正し、カタログ実装間でclose()を標準化 ([#16862](https://github.com/apache/iceberg/pull/16862))
- API, AWS, Azure, GCP: positioned、vectored、acceleratedな読み込みについても読み取りメトリクスを追跡するよう対応 ([#17236](https://github.com/apache/iceberg/pull/17236))
- Core, AWS: RESTレスポンスヘッダーフィールド名の大文字小文字を区別しない扱いを修正 ([#17979](https://github.com/apache/iceberg/pull/17979))
- GCP: 不正なGCSバッファ読み込み範囲を指定した場合に拒否するよう修正 ([#17961](https://github.com/apache/iceberg/pull/17961))
- GCS, S3, ADLS: 入力ストリームのEOFを正しく処理するよう修正 ([#16055](https://github.com/apache/iceberg/pull/16055))
- Dell: 既知のECS入力ファイル長を保持 ([#17870](https://github.com/apache/iceberg/pull/17870))
- Aliyun: 既知のファイル長を`OSSFileIO.newInputFile`に渡すよう変更 ([#16870](https://github.com/apache/iceberg/pull/16870))

## Dependencies
- Spark 3.4のサポートを削除
- Flink 2.2および2.3のサポートを追加
- Flink 2.0のサポートを削除
- CVEを修正するためJacksonのバージョンを統一 ([#16954](https://github.com/apache/iceberg/pull/16954))
- CVE GHSA-r7wm-3cxj-wff9を修正するためJacksonをバンプ ([#17336](https://github.com/apache/iceberg/pull/17336))
- ORC: 1.9.8 -> 1.9.9
- Jackson: 2.21.3 -> 2.22.2
- AWS SDK: 2.44.4 -> 2.54.17
- Azure SDK: 1.3.6 -> 1.3.8
- Nessie: 0.107.5 -> 0.108.8
- Netty: 4.2.13.Final -> 4.2.18.Final
- Guava: 33.6.0-jre -> 33.7.1-jre
- Caffeine: 2.9.3 -> 3.2.4
- Calcite: 1.41.0 -> 1.42.0
- Avro: 1.12.1 -> 1.12.2
- Bouncycastle: 1.84 -> 1.86
- Delta: 3.3.2 -> 3.3.3
- Jetty: 12.1.8 -> 12.1.13

## LICENSE / NOTICE
- shaded-jarのLICENSE/NOTICEを整理し、不足していたthird-partyの通知を追加 ([#16543](https://github.com/apache/iceberg/pull/16543))
