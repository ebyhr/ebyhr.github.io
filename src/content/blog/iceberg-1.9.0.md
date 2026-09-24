---
pubDate: '2025-04-28'
title: 'Iceberg 1.9.0 リリースノート'
slug: iceberg-1.9.0
tags: ['iceberg']
---
Iceberg 1.9.0がリリースされました！全体の変更はこちらのタグから確認できます。
https://github.com/apache/iceberg/releases/tag/apache-iceberg-1.9.0

## 行レベルのリネージ

Row lineageは1.9.0未満のバージョンではV3の機能としてはオプショナルな扱いでしたが、1.9.0からは必須の機能となりました。[スペック](https://iceberg.apache.org/spec/#row-lineage)にエンジン側で`next-row-id`フィールドの値を管理する必要があると書いてありますが、Icebergのライブラリを使って書き込みをしていれば、ライブラリ側でも管理してくれます。
https://github.com/apache/iceberg/pull/12781

## カラムのデフォルト値のサポート

カラムのデフォルト値を管理するDefault valuesがリリースされました。スペックのページは https://iceberg.apache.org/spec/#default-values です。`initial-default`と`write-default`という2つのプロパティで構成されています。`initial-default`はカラムを追加した際に、過去のデータのデフォルト値として利用されます。`write-default`はカラムを追加した際に、それ以降の書き込み時に利用されるデフォルト値です。
https://github.com/apache/iceberg/pull/12211

## GEOMETRY型とGEOGRAPHY型の追加

* `geometry(C)`：線形または平面上の辺補間を用いたジオメトリです。
* `geography(C, A)`：非線形の辺補間を用いたジオメトリです。辺補間のアルゴリズムは引数Aにより定義されます。現在サポートされているアルゴリズムはSPHERICAL、[VINCENTY](https://en.wikipedia.org/wiki/Vincenty%27s_formulae)、THOMAS、ANDOYER、[KARNEY](https://link.springer.com/content/pdf/10.1007/s00190-012-0578-z.pdf)の5つです。

両者とも引数Cは[CRS](https://iceberg.apache.org/spec/#crs)を表しています。デフォルトは`OGC:CRS84`が利用されます。

https://github.com/apache/iceberg/pull/12346

## パーティションレベルの統計情報

テーブルレベルのPuffinファイルを用いたNDVの統計情報とは異なり、パーティション内のレコード数などの情報をParquetファイルやORCファイルで管理します。
https://github.com/apache/iceberg/pull/11216

## SigV4関連の修正

これまでRESTカタログでSigV4を有効化するには`rest.sigv4-enabled=true`と設定する形でしたが、それはレガシー扱いとなり、今後は`rest.auth.type=sigv4`で設定することが推奨されています。
https://github.com/apache/iceberg/pull/11995

## ナノ精度のタイムスタンプ型のサポート

ナノ精度のタイムスタンプ型が各ファイルフォーマットでサポートされました。関連するPRは以下の通りです。https://github.com/apache/iceberg/pull/11775 で報告したとおり、Literalsクラスの実装は一部微妙なところがあり、クエリエンジンの実装の仕方によっては結果の不整合やオーバーフローになる可能性があるので、注意が必要です。
* Parquet: https://github.com/apache/iceberg/pull/12463
* ORC https://github.com/apache/iceberg/pull/12567
* Avro https://github.com/apache/iceberg/pull/12455

## メタデータファイルのjsonファイルをun-prettyに変更

これまでメタデータファイルは改行や空白の入った見やすいpretty形式でしたが、ファイルサイズを削減するためにun-pretty形式に変更されました。
https://github.com/apache/iceberg/pull/12318

## Spark 3.3のサポートを停止

1.8.0で廃止予定になっていたSpark 3.3が削除されました。
https://github.com/apache/iceberg/pull/12279

