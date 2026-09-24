---
pubDate: '2023-06-14'
title: 'Trino Fest 2023 キーノート'
tags: ['trino']
---
2023年6月14、15日に開催された[Trino Fest](https://www.starburst.io/info/trinofest/)のキーノートTrino for lakehouses, data oceans, and beyondを見たので日本語訳サマリーです。動画は以下のリンクで見ることができます。

https://www.youtube.com/watch?v=SJ1h-I7HoII&feature=youtu.be&themeRefresh=1

### By the numbers

- 2022年11月から16回リリースされました
- 2023年に入ってからのコミット数は約2,250
- 合計のコントリビュータ数は660人以上
- Slackメンバーは現在9,900人以上
- 1,800社を超える2万人以上のコミュニティメンバー
- [db-enginesランキング](https://db-engines.com/en/system/Trino)は96位から69位へ

### New maintainers

新しいメンテナが2人追加されました。AWSのJames Pettyと、StarburstのManfred Moserです🎉全体のメンテナのリストは https://trino.io/development/roles.html#maintainers に載っています。

### Table function improvements

新たに以下のテーブル関数が追加されています。

- [exclude_columns](https://trino.io/docs/current/functions/table.html#exclude_columns)
    - SELECT文で特定のカラムだけ除いて結果を返却
- [sequence](https://trino.io/docs/current/functions/table.html#sequence-table-function)
    - 指定された範囲の値を生成して返却。従来のスカラー関数は最大で10,000のエントリまでという制限がありましたがテーブル関数ではその制限は撤廃されています。
- query/[raw_query](https://trino.io/docs/current/connector/elasticsearch.html#raw-query-varchar-table)
    - リモートで実行するクエリを文字列として受け取って実行結果を返却します。queryテーブル関数はJDBC系のコネクタやBigQuery、Cassandra、MongoDB、raw_queryテーブル関数はElasticsearchコネクタでサポートされています。
- [procedure](https://trino.io/docs/current/connector/sqlserver.html#procedure-varchar-table)
    - SQL Server内のストアドプロシージャを実行

### Fault-tolerant execution

性能の改善やストレージとしてHDFSも対象となりました。MongoDB, BigQuery, Redshift and Oracleコネクタへの対応が追加されました。

### Schema evolution, (meta)data, and tools

- `ALTER COLUMN … SET DATA TYPE`が新しいシンタックスとして追加
- `ALTER TABLE … RENAME COLUMN`の対応コネクタの追加
- `ALTER TABLE … DROP COLUMN`でROWタイプ内のフィールドを削除する機能が追加
- Hudiコネクタで[$timelineメタデータテーブル](https://trino.io/docs/current/connector/hudi.html#timeline-table)が追加
- Delta LakeコネクタでChange Data Feedを返却する[table_changesテーブル関数](https://trino.io/docs/current/connector/delta-lake.html#table-changes)が追加
- IcebergコネクタでREST、JDBCおよびNessieカタログが追加

### Lakehouse migration - table procedures

- [migrate](https://trino.io/docs/current/connector/iceberg.html#migrate-table)
    - HiveテーブルをIcebergテーブルへファイルの書き換えなしに変換するプロシージャ
- `register_table` / `unregister_table`
    - Iceberg, Delta Lakeテーブルをメタストアへ登録、もしくはメタストアから削除するプロシージャです。DROP TABLEはファイルを削除しますがunregister_tableではファイルは削除しません。

### Tons of performance improvements

数が多すぎるのでスクリーンショットを添付します🐰

![](https://storage.googleapis.com/zenn-user-upload/db15a47370c5-20250324.png)

![](https://storage.googleapis.com/zenn-user-upload/8195ea02d81a-20250324.png)


### Tracing with OpenTelemetry

OpenTelemetryを使ったオブザーバビリティの向上を進めています。添付はDatadogでフレームグラフを表示しています。

![](https://storage.googleapis.com/zenn-user-upload/4627bc023799-20250324.png)


### Client tool news

PythonクライアントではSQLAlchemy 2.0やEXECUTE IMMEDIATE等々、継続的に開発が進めめられています。[dbt Cloud](https://docs.getdbt.com/docs/cloud/connect-data-platform/connect-starburst-trino)のサポートも最近発表されました。

### Roadmap

今後のロードマップは以下のようなタスクが予定されています。

- [SQL 2023](http://peter.eisentraut.org/blog/2023/04/04/sql-2023-is-finished-here-is-whats-new)に関連したJSON周りやNumericリテラル(例 0xFFFF, 1_000_000)の対応
- `json_table`関数の追加
- [Snowflakeコネクタの追加](https://github.com/trinodb/trino/pull/17909)
- Java 21対応
- [Project Hummingbird](https://github.com/trinodb/trino/issues/14237)

### Trino: The Definitive Guide

2ndエディションが発売されました。Starburstから無料でPDFをダウンロードできます🐸 https://www.starburst.io/info/oreilly-trino-guide/

### Getting involved

- コミュニティへのSlackはhttps://trino.io/slack.htmlからご参加ください
- コントリビューションの際は[https://trino.io/development](https://trino.io/development/)にプロセスが載っています。GitHubのissueではgood first issueラベルが用意されています。
