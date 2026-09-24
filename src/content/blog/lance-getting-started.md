---
pubDate: '2025-10-25'
title: 'Lanceテーブルフォーマットを触ってみる'
tags: ['lancedb']
---
公式の[Getting Started with LanceDB](https://lancedb.com/docs/quickstart/)を元にLanceDBを触ってみます。

まずは必要なパッケージをインストールします。Python 3.8以上が必要となります。
```sh
pip install lancedb pandas
```

次に上記のライブラリをインポートします。`lancedb`がコアとなるベクターデータベースの機能を提供し、`pandas`はデータの操作に利用します。
```py
import lancedb
import pandas as pd
```

次にLanceDBに接続します。公式の例ではLanceDB Cloudを利用していますが、以下のように指定するとローカルで試すことができます。
```py
db = lancedb.connect(uri="data/lance.db")
```

pandasのデータフレームを3つのフィールド（id、vector、text）で作成します。
```py
data = pd.DataFrame([
    {"id": "1", "vector": [0.9, 0.4, 0.8], "text": "knight"},    
    {"id": "2", "vector": [0.8, 0.5, 0.3], "text": "ranger"},  
    {"id": "3", "vector": [0.5, 0.9, 0.6], "text": "cleric"},    
    {"id": "4", "vector": [0.3, 0.8, 0.7], "text": "rogue"},     
    {"id": "5", "vector": [0.2, 1.0, 0.5], "text": "thief"},     
])
```

データベース内にテーブルを作成します。スキーマはデータから自動で決定されます。
```py
table = db.create_table("adventurers", data)
```

次にベクトルの類似検索を実行します。クエリで指定するディメンションは実際のデータと一致する必要があります。例えば、前述のデータは`[0.9, 0.4, 0.8]`なので、ディメンションは3です。

この例を実行するとユークリッド距離を用いて、`[0.8, 0.3, 0.8]`にもっとも近い結果が3行返却されます。
```py
query_vector = [0.8, 0.3, 0.8] # warrior 
results = table.search(query_vector).limit(3).to_pandas()
print(results)
```

distanceカラムが低い値が距離が近い、つまり類似していることを表しています。

| id | vector          | text    | distance  |
|----|-----------------|---------|-----------|
| 1  | [0.9, 0.4, 0.8] | knight  | 0.02      |
| 2  | [0.8, 0.5, 0.3] | ranger  | 0.29      |
| 3  | [0.5, 0.9, 0.6] | cleric  | 0.49      |
