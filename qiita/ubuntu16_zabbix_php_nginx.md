ubuntu16.04 + ZABBIX + nginx & php-fpm
===============================

* https://qiita.com/hinoshiba/items/c9a89472245b53cf2394


#はじめに
* 個人的にnginxが流行っているのでフロントをnginxにしています。
 * よって、nginxの方が良いかは未確認です。

#実行環境
* ubuntu 16.04 （x64）
* root権限

## 説明内の変数
* DB zabbixユーザのパスワード:```password:*****<YourDBPassword>*****```

#導入作業
## ZABBIXインストール
### リポジトリ追加と更新
```bash
wget https://repo.zabbix.com/zabbix/3.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.4-1+xenial_all.deb
dpkg -i zabbix-release_3.4-1+xenial_all.deb
apt-get update
```
### 必要パッケージインストール　
```bash
apt-get install zabbix-server-mysql zabbix-frontend-php fonts-vlgothic -y
```
## DB準備
### SQLユーザとDBの作成
```bash
mysql -u root -p 
create user zabbix@'localhost' identified by '*****<YourDBPassword>*****';
create database zabbix character set utf8 collate utf8_bin;
grant all privileges on zabbix.* to zabbix@'localhost'
```
* db作成時のcharasetを誤るとZABBIX初期DBのコピー時にしくじるので注意

### DB初期テンプレートの書き込み
```bash
zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -u zabbix -p zabbix
```
 * パスワード入力を求められます。

```bash
password:*****<YourDBPassword>*****
```

### ZABBIXコンフィグファイルの設定
```bash
vi /etc/zabbix/zabbix_server.conf
```
```diff:/etc/zabbix/zabbix_server.conf
- 91  # DBHost=localhost
+ 92  DBHost=localhost
  101 DBName=zabbix
  117 DBUser=zabbix
- 125 # DBPassword=
+ 126 DBPassword=*****<YourDBPassword>*****
```
* パスワードは""で囲まなくて良い

以上でZABBIXに限った設定が完了

## nginxとphp-fpmのインストールと設定
### apacheの停止と起動無効化
* インストールスクリプトでポートが被ってこけるので、ZABBIXインストール時に入ったapache2を止めてあげる

```bash
systemctl stop apache2
systemctl disable apache2
```
### インストール
```bash
apt-get install nginx php-fpm
```
### nginxの設定強化
```bash
vim /etc/nginx/nginx.conf
```
```diff:/etc/nginx/nginx.conf
- 21         #server_tokens off;
+ 21         server_tokens off;
- 33         ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
+ 33         #ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
+ 34         ssl_prefer_server_ciphers on;
+ 35         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
+ 36         ssl_ciphers ECDHE+RSAGCM:ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:!EXPORT:!DES:!3DES:!MD5:!DSS;
```
### default siteの削除
```bash 
rm /etc/nginx/sites-enabled/default
```
### オレオレ証明書の作成
```bash
pwd
/etc/nginx/ssl
openssl genrsa -aes128 -out server.key 2048
# パスワードを適当に決めて入力する
openssl rsa -in server.key -out server.key
# 先ほど決めたパスワードを入力
openssl req -new -days 3650 -key server.key -out server.csr
# 全て空白でEnter
openssl x509 -in server.csr -out server.crt -req -signkey server.key -days 3650
chmod 400 server.*
```
### ZABBIX用nginxの設定
```bash
vim /etc/nginx/conf.d/zabbix.conf
# 新規作成。。。のはず
```
```conf:/etc/nginx/conf.d/zabbix.conf
server {
       listen 443;
       ssl on;
        server_name  localhost;
        root   /usr/share/zabbix;


        ssl_prefer_server_ciphers  on;
        ssl_ciphers  'ECDH !aNULL !eNULL !SSLv2 !SSLv3';
        ssl_certificate  /etc/nginx/ssl/server.crt;
        ssl_certificate_key  /etc/nginx/ssl/server.key;
        location / {
                index  index.html index.htm index.php;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
                root   /usr/share/nginx/html;
        }
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.0-fpm.sock;
                include        fastcgi_params;
        }
}
```
### php.iniの設定
```bash
vim /etc/php/7.0/fpm/php.ini
```
```diff:/etc/php/7.0/fpm/php.ini
- 368 max_execution_time = 30
+ 368 ;max_execution_time = 30
+ 369 max_execution_time = 300

- 379 max_input_time = 60
+ 379 ;max_input_time = 60
+ 380 max_input_time = 600

- 658 post_max_size = 8M
+ 658 ;post_max_size = 8M
+ 659 post_max_size = 32M

- 927 ;date.timezone =
+ 928 date.timezone = Asia/Tokyo
```
## 各種起動と有効化
```bash
systemctl enable mysql
systemctl enable zabbix-server
systemctl enable php7.0-fpm
systemctl enable nginx
systemctl restart mysql
systemctl restart zabbix-server
systemctl restart php7.0-fpm
systemctl restart nginx
```
## 日本語対応
```bash
localedef -f UTF-8 -i ja_JP ja_JP
systemctl restart zabbix-server
systemctl restart php7.0-fpm
```

以上で全ての設定が完了

#その他
* webUIでアクセスを行い、ZABBIXの初期設定を行う

#追記
## グラフの日本語化
```bash
dpkg-reconfigure zabbix-frontend-php
```

#参考
* http://blog.serverworks.co.jp/tech/2016/12/10/zabbixinstall/
* https://qiita.com/tomoyk/items/6b007b04ae137c31d589

