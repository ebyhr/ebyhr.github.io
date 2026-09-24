---
pubDate: '2025-03-24'
title: 'Iceberg REST Catalog (iceberg-rest-fixture)の設定サンプル'
tags: ['iceberg']
---
IcebergのRESTカタログとして[tabulario/iceberg-rest](https://hub.docker.com/r/tabulario/iceberg-rest)を利用している方々も多いと思いますが、バージョン1.6.0で更新は止まっているので、他のRESTカタログを探す必要があります。

この記事では、移行先の候補の1つであるIceberg RESTカタログ（[apache/iceberg-rest-fixture](https://hub.docker.com/r/apache/iceberg-rest-fixture)）にTrinoから接続する例をご紹介します。DBはPostgreSQL、ストレージはMinIOを利用しています。

# Dockerコンテナの起動

iceberg-rest-fixtureイメージにPostgreSQLのJDBCドライバが含まれていないので、あらかじめローカルにダウンロードしておきます。
```sh
curl -o ./postgresql-42.7.5.jar https://jdbc.postgresql.org/download/postgresql-42.7.5.jar
```

次にdocker-compose.ymlファイルを用意します。
iceberg-rest-fixture部分の環境変数は、以下の流れでプロパティに置換されます。
1. `CATALOG_`プレフィックスを削除
2. アンダースコア2つ(`__`)を`-`に置換
3. アンダースコア1つ(`_`)を`.`に置換
4. 小文字に変換

例えば`CATALOG_S3_PATH__STYLE__ACCESS`は`s3.path-style-access`になります。

```yml
volumes:
  data: {}

services:
  postgresql:
    container_name: postgresql
    image: postgres:12
    environment:
      POSTGRES_DB: 'test'
      POSTGRES_USER: 'test'
      POSTGRES_PASSWORD: 'test'
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      retries: 3

  minio:
    container_name: minio
    image: quay.io/minio/minio
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - data:/data
    command: server /data --console-address ":9001"

  create-bucket:
    image: minio/mc
    depends_on:
      - minio
    volumes:
      - data:/data
    entrypoint: mc mb /data/bucket

  irc:
    hostname: irc
    image: apache/iceberg-rest-fixture:1.8.1
    depends_on:
      postgresql:
        condition: service_healthy
      minio:
        condition: service_started
    volumes:
      - ./postgresql-42.7.5.jar:/usr/lib/iceberg-rest/postgresql-42.7.5.jar
    ports:
      - "8181:8181"
    environment:
      CATALOG_URI: jdbc:postgresql://postgresql:5432/test
      CATALOG_JDBC_USER: test
      CATALOG_JDBC_PASSWORD: test
      AWS_REGION: us-east-1
      CATALOG_WAREHOUSE: s3://bucket/warehouse/
      CATALOG_IO__IMPL: org.apache.iceberg.aws.s3.S3FileIO
      CATALOG_S3_ENDPOINT: http://minio:9000
      CATALOG_S3_PATH__STYLE__ACCESS: true
      CATALOG_S3_ACCESS__KEY__ID: minioadmin
      CATALOG_S3_SECRET__ACCESS__KEY: minioadmin

    command: java -cp /usr/lib/iceberg-rest/*:iceberg-rest-adapter.jar org.apache.iceberg.rest.RESTCatalogServer

  trino:
    container_name: trino
    image: trinodb/trino:474
    environment:
      CATALOG_MANAGEMENT: 'dynamic'
    depends_on:
      - irc
      - minio
    ports:
      - "8080:8080"
```

docker-compose.ymlファイルの準備が完了したらコンテナを起動します。なお、IRCはIceberg Rest Catalogの略称です。海外のIcebergコミュニティで時々使われる略称なので、覚えておくと良いと思います。
```sh
docker compose up
```

# TrinoからRESTカタログへ接続

Trinoのコンテナ内にCLIがインストールされているので、以下のコマンドでCLIに接続します。
```sh
docker exec -it trino trino
```

Icebergコネクタを利用したカタログを作成します。カタログ名は何でも良いですが、分かりやすいようにコネクタ名と一致させています。
```sql
CREATE CATALOG iceberg 
USING iceberg
WITH (
 "iceberg.catalog.type" = 'REST',
 "iceberg.rest-catalog.uri" = 'http://irc:8181',
 "fs.native-s3.enabled" = 'true',
 "s3.endpoint" = 'http://minio:9000',
 "s3.aws-access-key" = 'minioadmin',
 "s3.aws-secret-key" = 'minioadmin',
 "s3.path-style-access" = 'true',
 "s3.region" = 'us-east-1'
);
```

スキーマやテーブルが無い状態なので、作成してみます。
```
CREATE SCHEMA iceberg.default;
CREATE TABLE iceberg.default.region AS SELECT * FROM tpch.tiny.region;
```

テーブルの中身を確認すると5件の結果が返却されます。
```sql
SELECT * FROM iceberg.default.region;
```

# JDBCカタログのテーブルを参照

せっかくなので、RESTカタログの裏側で利用されているPostgreSQLにも接続してみましょう。
```sql
CREATE CATALOG postgresql
USING postgresql
WITH (
 "connection-url" = 'jdbc:postgresql://postgresql:5432/test',
 "connection-user" = 'test',
 "connection-password" = 'test'
);
```

`postgresql`カタログに対して`SHOW TABLES`を実行すると`iceberg_namespace_properties`と`iceberg_tables`の2つのテーブルが返却されます。
```sql
SHOW TABLES IN postgresql.public;
            Table
------------------------------
 iceberg_namespace_properties
 iceberg_tables
```

`iceberg_namespace_properties`テーブルを参照すると、前述のステップで作成した`default`ネームスペースが存在します。
```sql
SELECT * FROM postgresql.public.iceberg_namespace_properties;
 catalog_name | namespace | property_key | property_value
--------------+-----------+--------------+----------------
 rest_backend | default   | exists       | true
```

`iceberg_tables`テーブルを参照すると、こちらも前述のステップで作成した`region`テーブルが存在します。
```sql
SELECT * FROM postgresql.public.iceberg_tables;
-[ RECORD 1 ]--------------+----------------------------------------------------------------------------------------------------------------------------------------
catalog_name               | rest_backend
table_namespace            | default
table_name                 | region
metadata_location          | s3://bucket/warehouse/default/region-d03afd016e9145638f8cd90da3547a9d/metadata/00001-8805ad43-32f8-4ea2-9feb-71282143daff.metadata.json
previous_metadata_location | s3://bucket/warehouse/default/region-d03afd016e9145638f8cd90da3547a9d/metadata/00000-a33d125e-ec4d-40ca-8370-461a08c64a87.metadata.json
iceberg_type               | TABLE
```
結果が横に長いので、見やすいように少し加工しています。

# MinIOからIcebergのコンテンツを確認

Icebergのコンテンツを確認したい場合は、Webブラウザで http://localhost:9001/ を開き、ユーザー名とパスワードに`minioadmin`を入力してログイン後に、Object Browserから参照できます。

![](https://storage.googleapis.com/zenn-user-upload/cbeed4915e05-20250324.png)
