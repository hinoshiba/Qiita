L2TP/IPsec Linuxルータ間接続 -サーバ辺-
==============

* https://qiita.com/hinoshiba/items/62cedc6dac344ca64451


# 手順について
* OSインストールやネットワーク周りの設定は割愛します
* 接続に関するまとまった記事が見つけきれなかったので、メモのサーバ側手順
 * [ローカル拠点]---l2tp/ipsec---[VPS]で接続を行い、ローカル拠点の特定の通信をVPSに伝搬するための手順
* クライアント、サーバ共に、xl2tpd及びstrongswanを用いて構築します
* 推奨値やよくある設定値には特に説明を付与していません。状況によって変化するであろうパラメータのみ説明を付け加えています

# 接続構成
* サーバ(centos7)
 * 状況：VPS

# サーバ（centos7）側構築手順
## 必要物のインストール
```bash
yum -y install strongswan xl2tpd
```
## 接続後情報の設定
### l2tpの設定

```bash
vim /etc/xl2tpd/xl2tpd.conf
```
```conf
[lns default]                               ; Our fallthrough LNS definition
  ip range = 172.30.255.2                   ; * Allocate from this IP range #クライアントに配布して良いIPアドレス。複数拠点の場合ハイフン(-)でレンジ指定が可能
  local ip = 172.30.255.1                   ; * Our local IP to use #local #トンネル上の自分自身に設定するIPアドレス
  length bit = yes                          ; * Use length bit in payload?
  refuse pap = no                           ; * Refuse PAP authentication
  refuse chap = yes                         ; * Refuse CHAP authentication
  require authentication = yes              ; * Require peer to authenticate
  name = l2tp                               ; * Report this as our hostname
  pppoptfile = /etc/ppp/options.l2tpd.lns   ; * ppp options file
```
```bash
vim /etc/ppp/options.l2tpd.lns
```
```conf
name l2tp
refuse-chap
refuse-mschap
require-mschap-v2
odefaultroute
lock
nobsdcomp
mtu 1280
mru 1280
logfile /var/log/xl2tpd.log
persist
ms-dns  8.8.8.8 #DNSの値
ms-wins 8.8.8.8 #DNSの値
```
* アカウントの設定

```bash
vim /etc/ppp/chap-secrets
```
```conf
"${auth_username}" l2tp "${passphrase}" 172.30.255.2 # ipアドレスは対象アカウントに対して付与して良いものを記載する
```
### ipsecの設定
```bash
vim /etc/ipsec.conf
```
```conf
config setup
         nat_traversal=no　#状況に応じて
conn %default
        auto=add
conn L2TP-noNAT
    forceencaps=yes
    authby=secret
    auto=add
    rekey=no
    ikelifetime=8h
    lifetime=1h
    type=transport
    leftauth=psk
    rightauth=psk
    left=xx.xx.xx.xx    # 自サーバの待ち受けIPアドレスを指定(nat配下の場合はnat前のIPアドレス）
    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any
    dpddelay=40
    dpdtimeout=130
    dpdaction=clear
```

* 事前共有キーの設定

```bash
vim /etc/ipsec.secrets
```
```config
: PSK "${IPSEC_PSK}"
```
### iptalbesの設定
```bash
vim /etc/sysconfig/iptables
```
```diff
+ -A INPUT -p udp -m udp --dport 500 -j ACCEPT
+ -A INPUT -p udp -m udp --dport 1701 -j ACCEPT
+ -A INPUT -p udp -m udp --dport 4500 -j ACCEPT
```
### サービス起動と自動起動の有効化
```bash
systemctl restart xl2tpd
systemctl restart strongswan
systemctl enable xl2tpd
systemctl enable strongswan
systemctl status strongswan
systemctl status xl2tpd
```
# リッスン確認
```bash
[root@server /]# netstat -anp | grep udp
udp        0      0 0.0.0.0:4500            0.0.0.0:*                           10919/charon
udp        0      0 0.0.0.0:1701            0.0.0.0:*                           10899/xl2tpd
udp        0      0 0.0.0.0:500             0.0.0.0:*                           10919/charon
```
# 接続確認パラメータ
アカウント、事前共有キーを手元の端末に設定し、接続確認が取れれば完了
* 本手順通りの場合はpapを無効

# 参考
 * ２年前ぐらいなのでどこかを参考にした気がするけど覚えていない

