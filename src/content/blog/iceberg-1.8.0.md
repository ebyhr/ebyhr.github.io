---
pubDate: '2025-02-13'
title: 'Iceberg 1.8.0 リリースノート'
slug: iceberg-1.8.0
tags: ['iceberg']
---
Iceberg 1.8.0が2月13日にリリースされたので変更点をまとめてみます。すべての変更点はこちらの[リンク](https://github.com/apache/iceberg/releases/tag/apache-iceberg-1.8.0)から参照できます。

互換性周りの問題がいくつか見つかっており、特にクエリエンジンの開発者はアップグレードするかどうかは慎重に判断した方が良さそうです。Trinoでは1.8.0はスキップして1.8.1を待つことを検討中です。
* RESTカタログでエンドポイントに対する追加のチェックが入ったため、1.8.0から古いRESTカタログに接続しにいくといくつかのエンドポイントでエラーとなります [#11756](https://github.com/apache/iceberg/pull/11756)。有名なカタログを例にあげるとPolarisおよびUnityカタログは対象に含まれています。
* AWSのSDKが最近新しいintegrity protectionsを導入した([aws-sdk-java-v2#5848](https://github.com/aws/aws-sdk-java-v2/issues/5848#issuecomment-2627909526))影響でMinIO(2025年より前のバージョン)、Vast、Dell ECSなどのストレージを利用している場合、DeleteObjectsRequestなどいくつかのリクエストが失敗します。

### [Sparkにrewrite_table_pathプロシージャを追加](https://github.com/apache/iceberg/pull/11555)

テーブルのメタデータを指定したディレクトリにコピーするプロシージャです。Icebergのメタデータファイル内では絶対パスが利用されているので、コピー後のファイルはプレフィックスを書き換えられたものが作成されます。データファイル以外がコピーの対象となっており、メタデータファイル、マニフェストリスト、マニフェストファイル、削除ファイルが対象です。

以下の例では`my_table`のメタデータを`s3://bucket/backup`へ修正しています。
```sql
CALL catalog_name.system.rewrite_table_path('my_table', 's3://bucket/my_table', 's3://bucket/backup');
```

デフォルトでは指定したテーブルの`metadata`ディレクトリ配下に`copy-table-staging-{UUID}`というディレクトリが作成され、その配下にメタデータファイル群が生成されます。ステージングの場所を変更したい場合は`staging_location`引数を指定できます。
```
└── metadata
    ├── 00000-eb20ffb5-3a13-4666-97f3-a52f987f1368.metadata.json
    └── copy-table-staging-60508edb-0c6a-46d0-8b81-7151cb337094
        ├── 00000-eb20ffb5-3a13-4666-97f3-a52f987f1368.metadata.json
        └── file-list
            ├── _SUCCESS
            └── part-00000-6959629c-7cdd-4fd0-aa19-66706ee6a5bd-c000.csv
```

デフォルトでは全バージョンのメタデータがコピーされます。一部のバージョンのみをコピーしたい場合は`start_version`と`end_version`引数を指定します。
```sql
CALL catalog_name.system.rewrite_table_path(
 table => 'my_table',
 source_prefix => 's3://bucket/my_table',
 target_prefix => 's3://bucket/backup',
 start_version => 'v5.metadata.json',
 end_version => 'v10.metadata.json');
```

#### [Sparkにcompute_table_statsプロシージャを追加](https://iceberg.apache.org/docs/latest/spark-procedures/#table-statistics)

NDV(Number of Distinct Values)を収集して[Puffinファイル](https://iceberg.apache.org/puffin-spec/)を生成するプロシージャです。特定のスナップショットや一部のカラムのみの統計情報を収集することも可能です。シンタックスは以下の通りです。テーブル名は必須で、`snapshot_id`と`columns`引数はオプションです。

`my_table`の最新のスナップショットの統計情報を収集する。
```sql
CALL catalog_name.system.compute_table_stats('my_table');
```

`my_table`の`snap1`スナップショットの統計情報を収集する。
```sql
CALL catalog_name.system.compute_table_stats(table => 'my_table', snapshot_id => 'snap1' );
```

`my_table`の`snap1`スナップショット内の`col1`および`col2`カラムの統計情報を収集する。
```sql
CALL catalog_name.system.compute_table_stats(table => 'my_table', snapshot_id => 'snap1', columns => array('col1', 'col2'));
```

#### [デフォルトバリュー](https://iceberg.apache.org/spec/#default-values)
内部的には2つのフィールドで構成さています。どちらも既存のデータファイルに変更を加えることなく、デフォルトの値はユーザーに提供する機能です。
* `initial-default`: フィールドが追加された後にデータを読み込む際に利用されるデフォルトの値です。
* `write-default`: フィールドが追加された後にデータを書き込む際に利用されるデフォルトの値です。

#### [Deletion Vectors](https://iceberg.apache.org/spec/#deletion-vectors)
V3スペック向けにDeletion VectorsのAPIが追加されました。こちらについては別のポストで説明しています。
https://zenn.dev/ebyhr/articles/52f7e8db8cc229

#### [Hiveランタイムモジュールの削除](https://github.com/apache/iceberg/pull/11801)
Hiveバージョン4からはHiveのリポジトリ側でIcebergのサポートがビルトインで開発されています。Iceberg側ではHive 2, 3向けの機能が開発されていましたが、関連するコードが1.8.0で削除されました。

#### [ParallelIterableのデッドロックの修正](https://github.com/apache/iceberg/pull/11781)

Icebergを運用していると時々問題になる[ParallelIterable](https://github.com/apache/iceberg/blob/main/core/src/main/java/org/apache/iceberg/util/ParallelIterable.java)クラスの修正です。冒頭の互換性の問題がない環境であれば、重要な修正なのでバージョンアップを検討することをオススメします。

#### [統計情報を更新するメソッドのsnapshot-id引数を非推奨に変更](https://github.com/apache/iceberg/pull/12010)

Javaのライブラリを直接利用している開発者に影響のある変更です。
`UpdateStatistics#setStatistics(long snapshotId, StatisticsFile statisticsFile)`が非推奨となり、`UpdateStatistics#setStatistics(StatisticsFile statisticsFile)`が推奨されています。StatisticsFileクラスにsnapshotIdが含まれているので、新しいAPIではそちらが利用されます。
