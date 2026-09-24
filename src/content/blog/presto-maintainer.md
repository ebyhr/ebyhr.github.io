---
pubDate: '2019-10-11'
title: 'Prestoコミッターに就任しました'
tags: ['trino', 'presto']
---

7/11に開催された[Presto Conference Tokyo 2019](https://techplay.jp/event/733772)について書こう書こうと思いつつ放置していたところ、
ちょうど3ヶ月後の10/11にコミッターになったので、ご報告もかねて下書きを開きました。
この記事では当日話そうと思ってスライドから削った部分や最近のコミュニティについて書きたいと思います。

現在[Presto Software Foundation](https://prestosql.io/foundation.html) (PSF) とPresto Foundationという2つの組織があり、
前者はPrestoを最初に作り始めたクリエイター達および[Starburst](https://www.starburstdata.com/)のメンバーを中心に、[Arm Treasure Data](https://www.treasuredata.com/)、[Varada](https://varada.io/)、[Qubole](https://www.qubole.com/)
などその他にも多くの企業・開発者から支持されながら運営されています。
後者はFacebookを中心にTwitter, Uber, Alibabaが支持していて、Linux Foundationにホストされることが先日発表されました。
こう書くとどららを選ぶべきか悩むかもしれませんが、前者の方が開発の速度は早くコミュニティが非常に活発に動いてるので、
特別な理由がなければPSF側のPrestoを使用することやコミュニティへの参加をお勧めします。
Facebook側のリポジトリやSlackも見るようにしているのですが、対応が遅く残念な気持ちになります。
メーリングリストは両者で同じアドレスが使用されているのですが、回答者の多くはPSFのコミュニティメンバーなのでSlackで直接質問するとすぐ回答を得られます。

PSFのSlackにはこちらのページにあるリンクから参加できます。

[https://prestosql.io/slack.html](https://prestosql.io/slack.html)

チャンネルは結構多くて戸惑いそうですが、個人的にお勧めするチャンネルは以下の通りです。

- `#beginner` 気軽に質問できるチャンネルです。
- `#troubleshooting` バグと思われる現象に遭遇した際に質問するチャンネルです
- `#dev` 開発に興味がある方はぜひ！
- `#general-jp` 日本語で気軽に話せるチャンネルです

実際に開発に参加しなくても、もっと日本からコミュニティに参加してくれる方が増えてくれるととても嬉しいです。
コードは書かずにSlackで議論に参加しているだけのメンバーもいますが、多数のコネクタを1リポジトリで管理していることもあり、そういったフィードバックはとても貴重に扱われます。

その他のリンクとしては以下のようなものがあります。

- GitHub[https://github.com/prestosql/presto](https://github.com/prestosql/presto)
    - CLA[https://github.com/prestosql/cla](https://github.com/prestosql/cla)
    - Code
      Style[https://github.com/airlift/codestyle](https://github.com/airlift/codestyle)
- Commit Message
  Guideline[https://chris.beams.io/posts/git-commit/](https://chris.beams.io/posts/git-commit/)
- User Manual[https://prestosql.io/docs/current/](https://prestosql.io/docs/current/)
- Blog[https://prestosql.io/blog/](https://prestosql.io/blog/)

今更になりますがPresto Conference Tokyo 2019を企画してくださったArm Treasure Dataの方々、
参加者のみなさん、HadoopのIssueを進めてくださったお二方、どうも有り難うございました。また来年も開催できると良いですね。
