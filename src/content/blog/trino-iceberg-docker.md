---
pubDate:  '2025-03-14'
title:  'Trino IcebergコネクタをDockerで試す方法'
tags:  ['trino', 'iceberg']
---
Trino Icebergコネクタを簡単に起動して試す方法をご紹介します。Icebergテーブルの中身が簡単に覗けるような流れにしており、CLIで完結するステップにしています。

まずはTrinoをDockerで起動します。Icebergのmetadataやdataファイルがホストからも参照できるようにディレクトリをマウントさせています。また、Trino IcebergカタログをSQLで生成できるように `CATALOG_MANAGEMENT`に`dynamic`を指定しています。

```bash
docker run --rm --name trino -d -p 8080:8080 \
--volume /tmp/iceberg:/tmp/iceberg \
-e CATALOG_MANAGEMENT=dynamic trinodb/trino
```

次にコンテナ内のTrino CLIに接続します。

```bash
docker exec -it trino trino
```

デフォルトで存在するカタログを確認してみましょう。以下の5つのカタログが表示されるかと思います。

```sql
SHOW CATALOGS;
 Catalog
---------
 jmx
 memory
 system
 tpcds
 tpch
```

それではTrino Icebergカタログを作成してみましょう。 `TESTING_FILE_METASTORE`は名前からも分かるように、あくまでテスト向けのものなので本番環境では利用しないようご注意ください。メタストアのディレクトリにコンテナの起動時に指定したパスを設定します。

```sql
CREATE CATALOG iceberg 
USING iceberg
WITH (
 "iceberg.catalog.type" = 'TESTING_FILE_METASTORE',
 "hive.metastore.catalog.dir" = 'file:///tmp/iceberg',
 "fs.hadoop.enabled" = 'true'
);
```

カタログ作成直後はスキーマが`information_schema`しか存在しないので、`tpch`というスキーマを準備して、その配下にテーブルを作成します。

```sql
CREATE SCHEMA iceberg.tpch;
CREATE TABLE iceberg.tpch.region AS SELECT * FROM tpch.tiny.region WITH NO DATA;
```

テーブル作成直後にストレージがどうなっているか見てましょう。`SHOW CREATE TABLE`文を実行すると`location`テーブルプロパティにパスが記載されています。コンテナの起動時にマウントしてあるので、ホスト側からファイルの状況を確認できます。

```sql
SHOW CREATE TABLE iceberg.tpch.region;
                               Create Table
---------------------------------------------------------------------------
 CREATE TABLE iceberg.tpch.region (
    regionkey bigint,
    name varchar,
    comment varchar
 )
 WITH (
    format = 'PARQUET',
    format_version = 2,
    location = 'file:///tmp/iceberg/tpch/region-8a373547f2944e4f9a7cd92f02ab7dee'
 )
```

```bash
tree /tmp/iceberg/tpch/region-8a373547f2944e4f9a7cd92f02ab7dee
/tmp/iceberg/tpch/region-8a373547f2944e4f9a7cd92f02ab7dee
└── metadata
    ├── 00000-aadb2eff-2307-421b-a051-333bbb277284.metadata.json
    └── snap-2846050716169058185-1-ff69e86f-8825-4ec5-ac88-a632a8ce1b5a.avro
```

現在はデータが空ですが、`INSERT`文を実行するとParquetファイルが`data`ディレクトリ内に作成され、`metadata`ディレクトリも更新されていることが分かります。

```sql
INSERT INTO iceberg.tpch.region SELECT * FROM tpch.tiny.region;
```

```bash
tree /tmp/iceberg/tpch/region-8a373547f2944e4f9a7cd92f02ab7dee
/tmp/iceberg/tpch/region-8a373547f2944e4f9a7cd92f02ab7dee
├── data
│   └── 20240810_034632_00010_vdhvw-4fdb22b5-408b-436f-8f39-aad3c6aedbd4.parquet
└── metadata
    ├── 00000-aadb2eff-2307-421b-a051-333bbb277284.metadata.json
    ├── 00001-528adfc2-637f-4e81-9b70-f79ad7d3e7bf.metadata.json
    ├── 00002-c72245d5-6ef2-4e0c-b17b-639ce61673c1.metadata.json
    ├── 20240810_034632_00010_vdhvw-1c9997a5-fe87-44c7-99ca-90b684250a93.stats
    ├── 3cd57da6-1750-4173-b0ce-1729e3b95af7-m0.avro
    ├── snap-1285440851173177234-1-3cd57da6-1750-4173-b0ce-1729e3b95af7.avro
    └── snap-2846050716169058185-1-ff69e86f-8825-4ec5-ac88-a632a8ce1b5a.avro
```

他にも[タイムトラベル](https://trino.io/docs/current/connector/iceberg.html#time-travel-queries)や[メタデータテーブル](https://trino.io/docs/current/connector/iceberg.html#metadata-tables)などIcebergコネクタの機能を自由に試せます。詳しくは公式のドキュメントをご参照ください。

https://trino.io/docs/current/connector/iceberg.html

普段Trino Icebergコネクタを開発している時は前述の方法は使わず、[IcebergQueryRunner](https://github.com/trinodb/trino/blob/master/plugin/trino-iceberg/src/test/java/io/trino/plugin/iceberg/IcebergQueryRunner.java)というクラスを利用しています。これを使えば1クリックでIDE上でTrinoのサーバーが起動し、CLIから接続できるようになっています。Trinoは様々なカタログ（Hive Thrift metastore、Glue、JDBC、REST、Nessie、Snowflake、Polaris）やストレージ（S3、ABFS、GCS、MinIO）をサポートしており、この方法で日々デバッグしたり機能の追加を行っています。Trinoの開発に興味がある方はこちらの方法もぜひお試しください。
