> ## Content Index
> Fetch the complete content index at: https://ebyhr.github.io/llms.txt
> Use this file to discover other available public pages before exploring further.

# 2024年のTrinoへのコントリビュートまとめ
- URL: https://ebyhr.github.io/2024-trino-contribution/
- Published: 2024-12-31T00:00:00.000Z
- Updated: 2026-09-06T00:18:05.000Z
- Author: Yuya Ebihara
- Tags: Trino

2024年にTrinoにコントリビュートした記録をまとめてみました。バグの修正などは除外して、ユーザーに関わる機能にしぼっています。

Delta Lakeコネクタはdelta-kernelを利用せずにトランザクションログやチェックポイントのパースを始め、読み書きを自前で実装していることもあり、ここ2年は作業量が多かったのですが、Delta側が[Table Features](https://github.com/delta-io/delta/blob/master/PROTOCOL.md#table-features)を導入してからはreaderとwriterのバージョンがあがらなくなり、落ち着いてきました。[プロトコル](https://github.com/delta-io/delta/blob/master/PROTOCOL.md)や[RFC](https://github.com/delta-io/delta/tree/master/protocol%5Frfcs)も以前ほど頻繁には変更されていない印象を受けます。

そういった背景もあり、Delta Lakeの[Variant型](https://github.com/delta-io/delta/blob/master/protocol%5Frfcs/variant-type.md)の対応がまだ進行中ですが、先月からは基本的にIcebergのみに完全に注力していくことになりました。現在はIcebergコネクタをWAP（Write-Audit-Publish）パターンを実現できるようにシンタックスの追加を進めています。あとは[V3スペック](https://iceberg.apache.org/spec/#version-3-extended-types-and-capabilities)についても対応中です。

## Iceberg

- `add_files`および`add_files_from_table`プロシージャの追加 [#22751](https://github.com/trinodb/trino/pull/22751)
  - Sparkのプロシージャと基本的には同じように動作しますが、1つ違いを挙げるとスキーマのバリデーションを追加しています。Sparkでは`NOT NULL`カラムに対しても`NULL`が入ったファイルを追加できるのですが、Trino側ではエラーを投げるようにしています。
- `$all_manifests`メタデータテーブルの追加 [#24330](https://github.com/trinodb/trino/pull/24330)
- `$all_entries`メタデータテーブルの追加 [#24543](https://github.com/trinodb/trino/pull/24543)
- `$entries`メタデータテーブルの追加 [#24172](https://github.com/trinodb/trino/pull/24172)
- `ALTER COLUMN ... DROP NOT NULL`ステートメントのサポート [#20448](https://github.com/trinodb/trino/issues/20448)

## Delta Lake

- Deletion Vectorの書き込みサポート [#22102](https://github.com/trinodb/trino/pull/22102)
- タイムトラベルクエリのサポート[#21052](https://github.com/trinodb/trino/pull/21052)
- メタデータをメタストアにバックグラウンドで保存 [#21463](https://github.com/trinodb/trino/pull/21463)
- Delta LakeやIcebergでよく問題になるのが `information_schema`等からカラム情報をリストするのが遅いというものがあります。これは毎回ストレージに対するアクセスが発生するのが原因なのですが、それを回避するためにメタストアのプロパティにカラム情報と最新のトランザクションのバージョンを追加しています。
- [Type Widening](https://github.com/delta-io/delta/blob/master/protocol%5Frfcs/type-widening.md)が有効化されているテーブルの読み込みサポート [#22142](https://github.com/trinodb/trino/pull/22142)
- Iceberg UniFormが有効になったテーブルの読み込みのサポート [#22311](https://github.com/trinodb/trino/pull/22311)
- `$transactions`メタデータテーブルの追加 [#24292](https://github.com/trinodb/trino/pull/24292)
- [V2チェックポイント](https://github.com/delta-io/delta/blob/master/PROTOCOL.md#v2-spec) が有効化されているテーブルの読み込みのサポート [#19345](https://github.com/trinodb/trino/issues/19345)
- `ALTER COLUMN ... DROP NOT NULL` ステートメントのサポート. [#20448](https://github.com/trinodb/trino/issues/20448)

## その他

- JDBC系のコネクタに `execute`プロシージャを追加 [#22556](https://github.com/trinodb/trino/pull/22556)
  - `query`テーブル関数では主にSELECT系のクエリをサポートしていますが、DDLやDMLをデータソース側で実行できるように新しく`execute`プロシージャを追加しました。
- Vector関連の機能追加
  - 類似性の検索機能に関するリクエストがお客さまから挙がっていたので、エンジンに関数を追加しつつPostgreSQLコネクタを改善していました。
    - PostgreSQLコネクタでvector型の読み込みをサポート [#22630](https://github.com/trinodb/trino/pull/22630)
      - `euclidean_distance`、`dot_product`、`cosine_distance` 関数をエンジンに追加 [#22397](https://github.com/trinodb/trino/pull/22397)

`euclidean_distance`、`-dot_product`、 `cosine_distance` 関数をpgvectorへプッシュダウン [#22618](https://github.com/trinodb/trino/pull/22618) [#23015](https://github.com/trinodb/trino/pull/23015)。pgvectorの表現に合わせて以下のようなルールで生成するSQLを内部的に書き換えながらプッシュダウンをしています。

| Trino               | pgvector |
| ------------------- | -------- |
| euclidean\_distance | <->      |
| cosine\_distance    | <=>      |
| \-dot\_product      | <#>      |