squid経由でIPhoneのlineのアプリを使う！
===================

* https://qiita.com/hinoshiba/items/c4967433ba6a5b474a40

#何がおきたか。
* 自宅のアクセスをSquid経由にしたらLineのアプリでメッセージが受信できない！

#解決方法
Connectポートに10006を追加してあげる。

``` diff:squid.conf
- acl SSL_ports port 443 
+ acl SSL_ports port 443 10006
```
#その他
* PATができる状態ならば、PROXY経由を試行したのち、自分で通信に行く。

