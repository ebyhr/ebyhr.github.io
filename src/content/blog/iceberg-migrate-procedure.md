---
pubDate: '2025-03-15'
title: 'HiveテーブルをIcebergテーブルに移行するプロシージャの実装'
tags: ['trino', 'iceberg']
---
# migrateプロシージャのシンタックス

Trinoバージョン411でIcebergコネクタに追加されたmigrateプロシージャについて解説します。このプロシージャは、既存のHiveテーブル（ORC、Parquet、Avroフォーマット）をIcebergテーブルに変換します。

```sql
CALL iceberg.system.migrate(
 schema_name => 'testdb', 
 table_name => 'customer_orders', 
 recursive_directory => 'true');
```

このプロシージャは3つの引数をサポートしています。
| 名前 | 必須 | 説明 |
| ---- | ---- | ---- |
| schema_name | Required | スキーマ名 |
| table_name | Required | テーブル名 |
| recursive_directory | Optional | ディレクトリを再帰的に走査するか |

recursive_directoryの設定可能な値はtrue、false、failです。
* true: ディレクトリを再帰的に検索する
* false: 直下のディレクトリのファイルのみを検索する
* fail: テーブルまたはパーティションのロケーション配下にネストされたディレクトリが存在する場合に例外をスローします。failがデフォルトです。

-----

# migrateプロシージャの実装

次に、このプロシージャの実装の詳細について説明します。
## 1. Hiveテーブルの定義に基づいてIcebergのスキーマオブジェクトを生成する
引数で指定されたテーブルをHive Metastore (HMS)やGlueから取得します。
Hiveテーブルの情報からIcebergを作成するのに必要な[Schema](https://github.com/apache/iceberg/blob/main/api/src/main/java/org/apache/iceberg/Schema.java)や[PartitionSpec](https://github.com/apache/iceberg/blob/main/api/src/main/java/org/apache/iceberg/PartitionSpec.java)のオブジェクトを生成します。


## 2. Hiveテーブルのファイルをリストする
非パーティションテーブルであればテーブルのパス、パーティションテーブルであれば各パーティションのディレクトリ配下を見ていきます。
ParquetやORCファイルはフッターから[Metrics](https://github.com/apache/iceberg/blob/main/api/src/main/java/org/apache/iceberg/Metrics.java)（行数、カラム毎のサイズ、非NULLの数、NULLの数、NaNの数、上限と下限）を算出します。Avroでは行数のみ算出します。

次に既存のHiveのデータファイルから[DataFile](https://github.com/apache/iceberg/blob/main/api/src/main/java/org/apache/iceberg/DataFile.java)のオブジェクトを生成します。Trinoでは以下のようなヘルパーメソッドを使っています。

```java
    public static DataFile buildDataFile(String path, long length, Optional<StructLike> partition, PartitionSpec spec, String format, Metrics metrics)
    {
        DataFiles.Builder dataFile = DataFiles.builder(spec)
                .withPath(path)
                .withFormat(format)
                .withFileSizeInBytes(length)
                .withMetrics(metrics);
        partition.ifPresent(dataFile::withPartition);
        return dataFile.build();
    }
```

## 3. トランザクションを開始する
Icebergテーブルを新規作成する[Transaction](https://github.com/apache/iceberg/blob/main/api/src/main/java/org/apache/iceberg/Transaction.java)を作成し、前のステップで生成したDataFileのリストを追加していきます。この時点で、Icebergのメタデータファイルは生成されていますが、メタストア上ではまだHiveテーブルのままです。
```java
Table table = transaction.table();
AppendFiles append = table.newAppend();
dataFiles.forEach(append::appendFile);
append.commit();
```

## 4. メタストアの情報を更新する
Trinoやその他のクエリエンジンはメタストア上の情報を見てそのテーブルフォーマットを判断しています。このステップで`table_type`テーブルプロパティに`ICEBERG`を設定し、メタストアの情報を更新します。

逆に言い換えるとIcebergテーブルを元に戻したい場合は`table_type`プロパティを削除してしまえば、Hiveテーブルとしてアクセス可能です。

## 5. トランザクションをコミットする
最後にトランザクションをコミットします。このコミットでIcebergのアーキテクチャでよく出てくる”最新のメタデータファイルへのポインタ”となる`metadata_location`テーブルプロパティが設定され、Icebergテーブルへのアクセスが可能となります。

