---
pubDate: '2025-02-14'
title: 'IcebergにおけるDeletion Vectorの実装'
tags: ['iceberg']
---
最近Trino IcebergコネクタでV3に関する機能を書いているので、その中でもやや大きめな変更である[Deletion Vector](https://iceberg.apache.org/spec/#deletion-vectors)について解説します。

V2までは[Position Delete](https://iceberg.apache.org/spec/#position-delete-files)の削除ファイルのフォーマットとしてParquet、ORC、Avroを使用していました。V2の削除ファイルでは以下のようにファイルのパスとレコードの位置を保存する形式でした。以下の例ではdata-file.parquetの3行目と4行目（0始まりなのでインクリメントしています）が削除されたことを表しています。parquet-toolsで表示した内容を見やすいように少し加工しています。

```bash
[
 {"File_path":"..../data/data-file.parquet","Pos":2},
 {"File_path":".../data/data-file.parquet","Pos":3}
]
```

V3からはそれらは廃止されDeletion Vectorに置き換えられました。Deletion Vectorは元々Delta Lakeで先に開発された機能で、Merge-on-Readを利用したテーブルで削除されたレコードのファイル内の位置を保持しています。Delta LakeとIcebergどちらも内部的には[Roaring Bitmap](https://roaringbitmap.org/)というビットマップ圧縮アルゴリズムを使用しています。書き込み時はレコードの削除された位置をRoaring Bitmapに追加していき最後に削除ファイルを生成します。読み込み時はデータファイルのレコードの位置が削除ファイルに含まれているかをチェックし、含まれていたらその行はスキップという流れです。

どちらもRoaring Bitmapをそのまま使っているわけではなく、Roaring Bitmapを配列として保持したクラスを使っています。詳しく見てみたい方はIcebergであれば[RoaringPositionBitmap.java](https://github.com/apache/iceberg/blob/main/core/src/main/java/org/apache/iceberg/deletes/RoaringPositionBitmap.java)、Delta Lakeであれば[RoaringBitmapArray.scala](https://github.com/delta-io/delta/blob/master/spark/src/main/scala/org/apache/spark/sql/delta/deletionvectors/RoaringBitmapArray.scala)で実装されています。

2つのテーブルフォーマットの実装で異なる点をあげると、IcebergはRoaring Bitmapを保存する際に[Puffin](https://iceberg.apache.org/puffin-spec/)ファイルフォーマット内のBlobとして格納しているのに対し、Delta Lakeは単一のバイナルファイルを書き込む形式を採用しています（厳密にはファイルを生成せずにトランザクションログやチェックポイントに書き込むinlineモードもありますが）。IcebergのDeletion VectorでPuffinファイルを生成する処理は[BaseDVFileWriter.java](https://github.com/apache/iceberg/blob/main/core/src/main/java/org/apache/iceberg/deletes/BaseDVFileWriter.java)を確認すると分かります。Deletion Vectorを含むPuffinのイメージとしては以下のようなデータになります。

```bash
type: deletion-vector-v1
inputFields: [2147483645]
snapshotId: -1
sequenceNumber: -1
blobData: {2, 3}
offset: 4
length: 44
compressionCodec: null
properties: {referenced-data-file=.../data/data-file.parquet, cardinality=2}
```

開発者が注意点する点としてはV2までは1つのデータファイルに対して複数の削除ファイルを紐づけることができましたが、V3では1つのデータファイルに対して単一のDeletion Vectorのみ紐づけることができます。1つのデータファイルに対する削除が複数回発生する場合は過去のDeletion Vectorをマージしながら最新の削除ファイルを作ることになります。

また、バージョン1.7の時点でV3のテーブルは作成できますが、Deletion Vector周りのAPIは1.8でリリースされる予定です。1.8以降ではV3のテーブルに対して従来の削除ファイルは追加できないようになっていますが、1.7時点では追加できてしまうのでこのあたりも注意が必要です。

Position DeleteについてはIceberg Bay Area Meetupの「Evolution of Position Deletes」セッションでも詳しく説明がされています。

https://www.youtube.com/watch?v=vjgJridq8G0

参考リンク

- メーリングリスト: [Proposal: Introduce deletion vector file to reduce write amplification](https://lists.apache.org/thread/gr3g5rrr60fhvy0mrdj4s4w9x8c3v58g)
- Google Docs: [Proposal: Introduce deletion vector file to reduce write amplification](https://docs.google.com/document/d/1FtPI0TUzMrPAFfWX_CA9NL6m6O1uNSxlpDsR-7xpPL0/edit?tab=t.0#heading=h.f42to8zz3i0)
