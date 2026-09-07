> ## Content Index
> Fetch the complete content index at: https://ebyhr.github.io/llms.txt
> Use this file to discover other available public pages before exploring further.

# Puffinファイルフォーマット
- URL: https://ebyhr.github.io/puffin-file-format/
- Published: 2025-09-13T00:00:00.000Z
- Updated: 2026-09-05T23:42:18.000Z
- Author: Yuya Ebihara
- Tags: Iceberg

Icebergで統計情報やDeletion Vectorに利用されている[Puffinファイルフォーマット](https://iceberg.apache.org/puffin-spec/)に関する解説です。  
Puffinファイルフォーマットは任意のBlobを保存できるように設計されており、将来的に様々な用途に使えるようになっています。

前半でフォーマットの内容について、後半に中身を確認できるツールの紹介をします。

## File structure

Puffinファイル以下のように、Magicバイト、Blobᵢ、Footerという3つの部分から構成されています。

| Magic  |
| ------ |
| Blob₁  |
| Blob₂  |
| ...    |
| Blobᵢ  |
| Footer |

- Magic: 先頭のMagic部分は4バイト（0x50, 0x46, 0x41, 0x31）で、これはPFA1 (Puffin Fratercula arctica, Version 1)を表します。「Puffin」は日本語では[ニシツノメドリ](https://ja.wikipedia.org/wiki/%E3%83%8B%E3%82%B7%E3%83%84%E3%83%8E%E3%83%A1%E3%83%89%E3%83%AA)を表します。「Fratercula arctica」はこの鳥の学名です。
- Blobᵢ: このファイルに含まれているBlobの集合です。Footerにしたがってアプリケーション側で処理を行います。
- Footer: 以下の内容で構成されています。

## Footer structure

| Magic             |
| ----------------- |
| FooterPayload     |
| FooterPayloadSize |
| Flags             |
| Magic             |

- Magic: 前述の先頭と同じ4バイトです。
- FooterPayload: ファイル内のBlobを記述するJSONオブジェクトで、詳しくは後述します。必要に応じて圧縮することもできます。
- FooterPayloadSize: FooterPayloadのバイトサイズを保持します。圧縮されている場合は圧縮後のサイズです。
- Flags: 真偽値フラグを格納するための4バイトです。  
  - byte 0 (first): 最下位のビットは、FooterPayloadが圧縮されているか否かを示します。
  - 残りのビットは将来の利用のために予約されており、書き込む際には0に設定しておく必要があります。

## FooterPayload

FooterPayloadのバイト列は、JSON形式で1つのFileMetadataオブジェクトを表しています。

### FileMetadata

| フィールド名     | 型                           | 必須  | 説明                                                                                      |
| ---------- | --------------------------- | --- | --------------------------------------------------------------------------------------- |
| blobs      | BlobMetadataのリスト            | Yes | 次のBlobMetadataで解説します。                                                                   |
| properties | プロパティの値がすべて文字列であるJSONオブジェクト | No  | 任意のメタ情報を格納するための領域です。created-byにクエリエンジンの名前やバージョンなどを書き込むことが推奨されています（例：Trino version 476）。 |

### BlobMetadata

| フィールド名            | 型                           | 必須  | 説明                                                                   |
| ----------------- | --------------------------- | --- | -------------------------------------------------------------------- |
| type              | JSON string                 | Yes |                                                                      |
| fields            | JSON int                    | Yes | Thetaスケッチであればフィールド名、Deletion Vectorであれば\_posフィールドを表す2147483645が入ります。 |
| snapshot-id       | JSON long                   | Yes | Blobが計算されたIcebergテーブルのスナップショットのIDを表します。                              |
| sequence-number   | JSON long                   | Yes | Blobが計算されたIcebergテーブルのシーケンスナンバーを表します。                                |
| offset            | JSON long                   | Yes | ファイル内でBlobの内容が始まる位置（オフセット）です。                                        |
| length            | JSON long                   | Yes | ファイル内に格納されているBlobのサイズ（圧縮されている場合は圧縮後のサイズ）です。                          |
| compression-codec | JSON long                   | No  | 圧縮方式を格納します。lz4、zstdがサポートされており、省略された場合は圧縮されていないことを表します。               |
| properties        | プロパティの値がすべて文字列であるJSONオブジェクト | No  | Blobに関する任意のメタ情報を格納する領域です。                                            |

---

## Blob types

現在、Icebergでは2種類のBlobをサポートしています。

### apache-datasketches-theta-v1

1つ目はNDV (Number of Distinct Values)を[Theta sketch](https://datasketches.apache.org/docs/Theta/ThetaSketches.html)  
フォーマットで保存する`apache-datasketches-theta-v1`です。

propertiesフィールドにndvというキーで値が保存されます。

### deletion-vector-v1

2つ目はIceberg v3で追加された[Deletion Vector](https://iceberg.apache.org/spec/#deletion-vectors)を保存するための`deletion-vector-v1`です。  
NDVとは違い、propertiesフィールドではなくBlobに[Roaring Bitmap](https://roaringbitmap.org/)が格納されます。

---

## puffin-tools

Iceberg関連の開発をしていると、メタデータファイルやデータファイルの中身を直接確認したくなることがあります。  
Parquet、ORC、Avroなどのツールにはそれぞれ`parquet-tools`、`orc-tools`、`avro-tools`があるのに対して、  
Puffinファイルを確認するツールは存在しませんでした。

以下のようにxxdコマンドなどで確認できる部分もありますが、Blob部分を直接読むのは大変です。

```
00000000: 5046 4131 0000 0026 d1d3 3964 0100 0000  PFA1...&..9d....
00000010: 0000 0000 0000 0000 3a30 0000 0100 0000  ........:0......
00000020: 0000 0200 1000 0000 0000 0200 0400 3065  ..............0e
00000030: d8b8 5046 4131 7b22 626c 6f62 7322 3a5b  ..PFA1{"blobs":[
00000040: 7b22 7479 7065 223a 2264 656c 6574 696f  {"type":"deletio
00000050: 6e2d 7665 6374 6f72 2d76 3122 2c22 6669  n-vector-v1","fi
00000060: 656c 6473 223a 5b32 3134 3734 3833 3634  elds":[214748364
00000070: 355d 2c22 736e 6170 7368 6f74 2d69 6422  5],"snapshot-id"
00000080: 3a2d 312c 2273 6571 7565 6e63 652d 6e75  :-1,"sequence-nu
00000090: 6d62 6572 223a 2d31 2c22 6f66 6673 6574  mber":-1,"offset
000000a0: 223a 342c 226c 656e 6774 6822 3a34 362c  ":4,"length":46,
000000b0: 2270 726f 7065 7274 6965 7322 3a7b 2272  "properties":{"r
000000c0: 6566 6572 656e 6365 642d 6461 7461 2d66  eferenced-data-f
000000d0: 696c 6522 3a22 6669 6c65 3a2f 7661 722f  ile":"file:/var/
000000e0: 666f 6c64 6572 732f 3973 2f5f 7a77 6e34  folders/9s/_zwn4
000000f0: 725f 6e32 5f39 6270 306b 726c 6c70 3170  r_n2_9bp0krllp1p
00000100: 6c33 6330 3030 3067 702f 542f 6963 6562  l3c0000gp/T/iceb
00000110: 6572 675f 7175 6572 795f 7275 6e6e 6572  erg_query_runner
00000120: 3338 3334 3635 3034 3632 3339 3736 3339  3834650462397639
00000130: 3733 362f 7470 6368 2f72 6567 696f 6e2d  736/tpch/region-
00000140: 3766 3766 3134 3036 3865 3032 3465 6338  7f7f14068e024ec8
00000150: 6137 3363 3037 3338 3861 3132 3331 6365  a73c07388a1231ce
00000160: 2f64 6174 612f 3230 3235 3039 3132 5f31  /data/20250912_1
00000170: 3132 3732 335f 3030 3032 335f 7836 7776  12723_00023_x6wv
00000180: 772d 3063 3062 6665 3261 2d38 3434 372d  w-0c0bfe2a-8447-
00000190: 3465 3463 2d62 3630 612d 6166 3162 3663  4e4c-b60a-af1b6c
000001a0: 3430 3931 3366 2e70 6172 7175 6574 222c  40913f.parquet",
000001b0: 2263 6172 6469 6e61 6c69 7479 223a 2233  "cardinality":"3
000001c0: 227d 7d5d 2c22 7072 6f70 6572 7469 6573  "}}],"properties
000001d0: 223a 7b22 6372 6561 7465 642d 6279 223a  ":{"created-by":
000001e0: 2254 7269 6e6f 2076 6572 7369 6f6e 2074  "Trino version t
000001f0: 6573 7476 6572 7369 6f6e 227d 7dc7 0100  estversion"}}...
00000200: 0000 0000 0050 4641 31                   .....PFA1

```

そこで、Puffinファイルの中身を確認できる[puffin-tools](https://github.com/ebyhr/puffin-tools)を作成しました。  
サンプルのPuffinファイルを[resourcesディレクトリ](https://github.com/ebyhr/puffin-tools/tree/master/src/test/resources)に置いてあります。

[バージョン0.1](https://github.com/ebyhr/puffin-tools/releases/tag/0.1)のタグにjarファイルがあるので、  
ダウンロードして実行権限を追加するとすぐに利用できます。

```
chmod +x puffin-tools.jar

```

サポートしている引数は`--path`のみで、絶対パスか相対パスでPuffinファイルを指定します。  
実行すると、まずpropertiesフィールドがJSON形式で出力され、続いてBlobごとに内容が出力されます。

Theta Sketchであればndvフィールドが最後に出力されます。  
テーブルのカラム名ではなく、フィールドIDで保存されている点に注意してください。  
Puffinファイル内にはフィールドIDしか保存されていないため、フィールド名を表示するにはIcebergの  
メタデータを別途参照する必要があります。

```sh
./puffin-tools.jar --path apache-datasketches-theta-v1.puffin

=== properties ===
{created-by=Trino version testversion}

=== blob ===
type: apache-datasketches-theta-v1
inputFields: [1]
snapshotId: 3906904963261072048
sequenceNumber: 1
offset: 4
length: 69
compressionCodec: zstd
properties: {ndv=5}
ndv: 5
...

```

Deletion Vectorであれば一番最後に削除された行の位置を示す情報が出力されます。

```sh
./puffin-tools.jar --path deletion-vector-v1.puffin

=== properties ===
{created-by=Trino version testversion}

=== blob ===
type: deletion-vector-v1
inputFields: [2147483645]
snapshotId: -1
sequenceNumber: -1
offset: 4
length: 46
compressionCodec: null
properties: {referenced-data-file=file:/var/folders/9s/_zwn4r_n2_9bp0krllp1pl3c0000gp/T/iceberg_query_runner3834650462397639736/tpch/region-7f7f14068e024ec8a73c07388a1231ce/data/20250912_112723_00023_x6wvw-0c0bfe2a-8447-4e4c-b60a-af1b6c40913f.parquet, cardinality=3}
deletedRows: [0, 2, 4]

```

## References

- <https://iceberg.apache.org/puffin-spec/#file-structure>
- <https://www.dremio.com/blog/puffins-and-icebergs-additional-stats-for-apache-iceberg-tables/>
- <https://speakerdeck.com/tomtanaka/apache-iceberg-meetup-in-japan-number-1-iceberg-v3-spec>