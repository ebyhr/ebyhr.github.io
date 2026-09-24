---
pubDate: '2025-02-21'
title: 'Apache Iceberg Japan Meetup #1 イベントレポート'
tags: ['iceberg']
---
https://iceberg.connpass.com/event/342709/  
https://posfie.com/@mikiT_T/p/lpmf4YT


2/21(金)にdocomo R&D OPEN LAB ODAIBAにてIcebergミートアップを開催しました！合計5つのセッションはどれもディープな内容で勉強になりました。運営をしてくださった髙田さん、松原さん、酒徳さん、北岡さん、ありがとうございました！

各セッションの簡単な振り返りです。

-----

https://speakerdeck.com/tomtanaka/apache-iceberg-meetup-in-japan-number-1-iceberg-v3-spec

[動画](https://www.youtube.com/watch?v=z_qRWeBnaxE&list=PL3IALGSANhzUIGG43Q1NCxRSFyUaKfj3L&index=5&pp=iAQB)
AWSの田中さんによるV3スペックに関するセッションです。最近TrinoでV3向けの機能を書いていることもあり個人的に聞きたかった内容でした。V3ではVariant、Geo、Timestamp nanoなど複数のデータ型が追加されます。Deletion VectorはV3でMoRを利用する上で必須な機能なので、MoRを利用する方は確認することをお勧めします。Row lineageは全く追えてなかったので勉強になりました。Row lineageに関する[プロポーザル](https://docs.google.com/document/d/146YuAnU17prnIhyuvbCtCtVSavyd5N7hKryyVRaFDTE/edit?tab=t.0#heading=h.f2e8ffw3fu7n)も参考になりそうです。

-----

https://speakerdeck.com/lycorptech_jp/apache-iceberg-case-study-in-ly-corporation

[動画](https://www.youtube.com/watch?v=BZvSlIHihvw&list=PL3IALGSANhzUIGG43Q1NCxRSFyUaKfj3L&index=3&pp=iAQB)
LINEヤフーの奥田さんによる同社におけるIcebergの利用事例に関するセッションです。国内でのIcebergの利用事例はまだまだ限られている印象を受けるので聞きたい方も多かった内容だと思います。Icebergを採用したメリットとして、部分的なデータ削除の容易さ、データが利用可能になるまでの時間の短縮、スキーマの柔軟性、メタストアのスケーラビリティ等が挙げられています。Icebergを利用するうえで遭遇した課題として移行やテーブルのメンテナンスが挙げられています。このあたりの話は下佐粉さんがホストしているOTF TALKの[第17、18回](https://www.otftalk.com/2024/11/ep8.html)でも解説されているのでそちらもぜひご視聴ください。

-----

https://speakerdeck.com/bering/apache-icebergtehurutong-shi-shu-kiip-mijing-he-jie-jue-noshi-zu-mitozhu-yi-dian

[動画](https://www.youtube.com/@IcebergMeetup)
AWSの疋田さんによるIcebergがどのように同時書き込みの競合を解決しているかのセッションです。更新トランザクションの流れや競合が発生するポイントについて説明されています。
疋田さんのブログポスト「[Apache Icebergにおける同時実行制御の仕組みと注意点](https://bering.hatenadiary.com/entry/2025/01/18/234339)」でも詳しく解説されているので、こちらもぜひ合わせてご覧ください。

-----

https://www.canva.com/design/DAGfj7ubhaY/I5n-ddsAcL72SXvLihJv1Q/view

[動画](https://www.youtube.com/watch?v=n2hh35QlTWI&list=PL3IALGSANhzUIGG43Q1NCxRSFyUaKfj3L&index=4&pp=iAQB)
酒徳さんによるSnowflakeで始めるIceberg入門のセッションです。Snowflakeには複数のカタログがあるということは聞いたことがありましたが、このセッションを聞いて違いがクリアになりました。DuckDBからDelta LakeだけでなくIcebergにもアクセスできるのも知らなかったので今度試してみたいと思います。セッション内で紹介されていたRAKUDEJI 前田さんのポストは「[Snowflake×Icebergを採用すべきか迷った時に読む記事](https://zenn.dev/dataheroes/articles/24009ab6970e48)」から参照できます。

-----

https://speakerdeck.com/databricksjapan/iceberg-meetup-japan-number-1-iceberg-and-databricks

[動画](https://www.youtube.com/watch?v=oLHEl-iJ6nw&list=PL3IALGSANhzUIGG43Q1NCxRSFyUaKfj3L&index=1&t=4s&pp=iAQB)
DatabricksのVictoriaさんによるカタログに関するセッションです。Icebergを利用するうえで悩ましいところがカタログの選択だと思います。Unityカタログがどのような特徴をもっているか理解するのに良いセッションです。なお、動画はDatabricksの北岡さんが編集してくださりました。ポップな動画で楽しいので公開されたぜひご覧ください。
