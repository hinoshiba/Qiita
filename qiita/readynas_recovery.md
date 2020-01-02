readynas management service is offline
=======================

* https://qiita.com/hinoshiba/items/2d4b4b69e147af0ebf8d

日本語の解決記事が見つけられなかったので、メモ

# 何がおきたか。

* ReadyNas のWebUIがみれない。
 * mysql追加してfw設定がよくわらかないのでmysql.confをいじくりまくっていたら、起動時のmysqldの起動にコケたらしく、そのままreadynasdがお亡くなりになったぽい。の作業記録
* GUI接続すると、ネットワークおかしいんじゃない？接続確認した？readynascloudに聞いてみてよ！というメッセージがたくさん。

* 手順に添い、NetGeAR　RAIDarをダウンロードしていろいろ見てみろよって言われるので、 DLして接続。
 * **ステータス：management service is offline**

# 調査
DLしてログ確認作業

```logs:readynasd.log
tail -n 4 readynasd.log
Mar 26 20:36:29 nas01 readynasd[2518]: readynasd log started
Mar 26 20:36:29 nas01 readynasd[2518]: readynasd started. (restarted=0)
Mar 26 20:36:32 nas01 readynasd[2518]: Error rndb_import() ==> rc=3
Mar 26 20:36:32 nas01 readynasd[2518]: dba import failed
```

どうやら、dbがおかしいらしい。。。のでバックアップして削除してみる。

# 対応

1 dbのバックアップ

```bash
cd ~
mkdir bkup
cd bkup
cp -rfp /var/readynasd ./
```
2 db削除

```bash
cd /var/readynasd/
rm *.sq3
```
3 再起動とログの確認

```bash
systemctl restart readynasd
```

```bash
systemctl status readynasd
Mar 26 21:21:13 nas01 rn-expand[5532]: auto_extend: Checking disk sdb...
Mar 26 21:21:13 nas01 rn-expand[5532]: auto_extend: Checking disk sda...
Mar 26 21:21:13 nas01 rn-expand[5532]: Trying xraid-expand (tiered expansion)
Mar 26 21:21:13 nas01 rn-expand[5532]: Considering X-RAID auto-expansion for data
Mar 26 21:21:13 nas01 rn-expand[5532]: Checking if RAID disk sdb is expandable...
Mar 26 21:21:13 nas01 rn-expand[5532]: Checking if RAID disk sda is expandable...
Mar 26 21:21:13 nas01 rn-expand[5532]: 0 disks expandable in data
Mar 26 21:21:15 nas01 readynasd[5532]: dumping id mapping from /var/lib/samba/winbindd_idmap.tdb
Mar 26 21:21:15 nas01 readynasd[5532]: ReadyNASOS background service started.
```

# まとめ

* 何かあったときのために、SSH有効は必須
* systemctl status readynasdで本件の状況の確認が可能
* dbエラーぽかったら上記手順で復旧可能。
* dbファイルを消すことでの影響は調査していない。


# その他
* SSHででかい容量をコピーすると同一の事象が発生する。
* 2017/3/26のメモをQiitaへ転機。

