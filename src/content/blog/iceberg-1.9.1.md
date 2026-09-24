---
pubDate: '2025-05-27'
title: 'Iceberg 1.9.1リリースノート'
slug: iceberg-1.9.1
tags: ['iceberg']
---
[Iceberg 1.9.1のリリースノート](https://iceberg.apache.org/releases/#191-release)日本語訳です。完全なリストは[GitHub](https://github.com/apache/iceberg/releases/tag/apache-iceberg-1.9.1)をご覧ください。

* API
    * Icebergのビルドバージョンの修正。これによって`git.build.version`が`unspecified`となっていた問題が修正されました。 [#12949](https://github.com/apache/iceberg/pull/12949)
* Core
    * `remove-snapshots`を高速化のためにバルク操作にするPR [#12670](https://github.com/apache/iceberg/pull/12670) がRESTカタログで後方互換性の問題があったためリバート。[#13100](https://github.com/apache/iceberg/pull/13100)
    * IcebergのViewはメタデータファイル内の`version-log`というフィールドで履歴を保持しているのですが、過去のView IDを現在のIDとして設定した際に、新しいエントリーが過去の古いタイムスタンプを再利用する挙動を修正。[#12821](https://github.com/apache/iceberg/pull/12821)
* Dependencies
    * Parquetライブラリのバージョンを[CVE-2025-46762](https://github.com/advisories/GHSA-53wx-pr6q-m3j5)を修正するために1.15.2へとアップグレード。
