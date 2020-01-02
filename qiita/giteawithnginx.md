Gitea (mysql , nginx ) + AD認証 でご自宅インストール
====================

* https://qiita.com/hinoshiba/items/302a79329fbf03ff1250


# Gitea(https://gitea.io/en-us/) とは
* Goで書かれたセルフホスト型git
* gogsからforkされたもの
* バイナリ１つで動作するので、既存環境を犯さない
* とても軽量（僕はmem使用量３４MBぐらい）
* ただ、SSL非対応？なので、nginx等でリバプロしてあげる必要がある

# 今回導入の構成
* ubuntu 16.04
* nginx
* mysql

# インストール手順
* サーバインストール手順などは割愛
* sqliteでも動くらしいが、今回はmysqlを利用する
* 以下手順に登場する```www.xxxxx.com```は、変数なので、各人置き換えてください。

## 1. 必要なものをインストール
```bash
$ export http_proxy=http://proxy.i.xxxx.com:8080/
$ apt update
$ apt install mysql-server nginx
```
* 初期パスワード確認されたら何かいれておく

---
## 2 mysql 設定
### 2.1 mysql_secure_installation
```bash
$ mysql_secure_installation

Change the password for root ? ((Press y|Y for Yes, any other key for No) : No

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.
```
### 2.2 gitea db create
```bash
$ mysql -u root -p 
DROP DATABASE IF EXISTS gitea;
CREATE DATABASE IF NOT EXISTS gitea CHARACTER SET utf8 COLLATE utf8_general_ci;
GRANT ALL ON gitea.* to 'gitea'@'localhost' identified by '<your password>';
quit
```
### 2.3 mysql起動
```bash
$ systemctl start mysql
$ systemctl enable mysql
```

----
## 3. gitea インストール
### 3.1 gitea のダウンロードと設置
```bash
$ adduser git
$ mkdir /var/lib/gitea
$ chown git. /var/lib/gitea/
$ cd /var/lib/gitea/
$ wget https://dl.gitea.io/gitea/1.5/gitea-1.5-linux-amd64
$ chown git. gitea-1.5-linux-amd64
$ chmod u+x gitea-1.5-linux-amd64
$ ln -s /var/lib/gitea/gitea-1.5-linux-amd64 /var/lib/gitea/gitea
```
### 3.2 systemd作成
```bash
$ vim /etc/systemd/system/gitea.service
```
```toml
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target
After=mysql.service
#After=postgresql.service
#After=memcached.service
#After=redis.service

[Service]
# Modify these two values and uncomment them if you have
# repos with lots of files and get an HTTP error 500 because
# of that
###
LimitMEMLOCK=infinity
LimitNOFILE=65535
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea
ExecStart=/var/lib/gitea/gitea web
Restart=always
Environment=USER=git HOME=/home/git

[Install]
WantedBy=multi-user.target
```
### 3.3 起動
```bash
$ systemctl daemon-reload
$ systemctl start gitea
$ systemctl enable gitea
```

---
## 4. nginxインストール
### 4.1　ssl鍵作成
```bash
$ mkdir /etc/nginx/crt
$ cd /etc/nginx/crt
$ openssl genrsa -aes256 -out server.key 4096
#適当なパスワード
$ openssl rsa -in server.key -out server.key
#先ほどの適当なパスワード
$ openssl req -new -days 3650 -key server.key -out server.csr
$ openssl x509 -in server.csr -out server.crt -req -signkey server.key -days 3650
$ chmod 400 server.*
```
### 4.2 sslconfigの作成
```bash
$ vim /etc/nginx/conf.d/ssl.conf
```
```vim
server {
        listen 80;
        server_name  www.xxxxx.com;
        return 301 https://$host;
}
server {
        listen 443 default ssl http2;
        ssl on;
        ssl_certificate     /etc/nginx/crt/server.crt;
        ssl_certificate_key /etc/nginx/crt/server.key;
        server_name  www.xxxxx.com;
        client_max_body_size 1G;
                location /gitea/ {
                client_max_body_size 0;
                proxy_pass http://localhost:3000/; #giteaがサブディレクトリの場合、末尾"/"が必須
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $http_host;
                proxy_set_header X-Forwarded-Ssl on;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_max_temp_file_size 0;
                proxy_redirect off;
                proxy_read_timeout 120;
        }
        error_page 500 502 503 504 /500.html;
        location / {
                root /usr/share/nginx/html;
                index index.html;
        }
}
```
### 4.3 nginx 強化
```bash
$ vim /etc/nginx/nginx.conf
```
```diff
- 21         #server_tokens off;
+ 21         server_tokens off;
- 33         ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
+ 33         #ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
+ 34         ssl_prefer_server_ciphers on;
+ 35         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
+ 36         ssl_ciphers ECDHE+RSAGCM:ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:!EXPORT:!DES:!3DES:!MD5:!DSS;
```
### 4.4 nginx起動
```bash
$ systemctl restart nginx
$ systemctl enable nginx
```

---
## 5. gitea初期設定
### 5.1 config編集
```bash
$ vim /var/lib/gitea/custom/conf/app.ini
```
```toml
[security]
INTERNAL_TOKEN = xxxxx
[server]
PROTOCOL         = http
DOMAIN           = www.xxxxx.com
ROOT_URL         = https://www.xxxxxcom/gitea/ #サブドメインを記載
DISABLE_SSH      = false #sshが見れるようにしておく
SSH_PORT         = 10022 #openssh-serverと競合しないよう、10022でlistenするようにする
START_SSH_SERVER = true
```
### 5.2 gitea再起動
```bash
$ systemctl restart gitea
```
### 5.3 webアクセス確認と初期設定
```
https://www.xxxxx.com/gitea/install
```
* mysqlとパスワードのみ設定
* administrator accountを初回作っておけば、あとで聞かれないので、作っておく
* ページ下部のインストールを選択

![スクリーンショット 2018-07-06 23.02.37.png](https://qiita-image-store.s3.amazonaws.com/0/210170/b972bc9d-667b-c26f-16c7-c4ebf21817d9.png)

---
## 6. ActiveDirectory認証設定
### 6.１ 認証情報登録
* 以下へアクセス

```
https://www.xxxxx.com/gitea/admin/auths
```
* Add Authentication Source を選択
* パラメータを入力
    * 認証タイプ　: LDAP(simple auth)
    * 認証名 : わかりやすいなにか
    * セキュリティプロトコル : ご自宅に合わせて。（和が家はLDAPS）
    * ホスト : ドメインコントローラの名前
    * ポート : 636
    * UserDN : 下記：”UserDNの調べ方”を参照
    * Userフィルター : (sAMAccountName=%s)
    * Username Attribute : sAMAccountName
    * Email Attribute : mail
    * tls verify disable (webサーバが話せるなら検証ありでもよい)
* Add Authentication Source　を選択

### 6.2 ドメインユーザでログオン確認
* 特に手順は無し。ログオン成功すれば完了
```
https://www.xxxxx.com/gitea/user/login
```

---

# 気になった点

## 注意点
* ActiveDirectoryユーザの利用に関して
    * simple authを利用する場合、dsqueryとcnのstringが同じユーザ名でログオンできる必要がある？ので、姓名がCN名になっているユーザは、アカウント名変更しないと正常ログオンができなかった。

## giteaで気になった点
* 翻訳がいけているけど、途中で力尽きた感がある
    * それ和訳しなくていいよがカタカナで、和訳できるでしょーが英語のままだったりする。（pull reqすればよい？w）
* .wikiファイルのプレビュー表示に非対応
    * https://qiita.com/hinoshiba/items/b931fcd06c278be5b7ca は泣く泣くmd生活に乗り換え

---
# UserDNの調べ方
* windows で以下を入力（コマンドがない場合はサーバマネージャをインストールする）

```cmd
dsquery user -u <username>
```

* 下記のような結果が返ってくる

```
CN=<username>,OU=Admin,OU=Hi-Users,DC=ds,DC=i,DC=xxxxx,DC=com
```

* CNが入力されたユーザ名となり、検索が走るため、%sとなる。
* よって、UserDNの項目に入力すべきは、以下となる

```
CN=%s,OU=Admin,OU=Hi-Users,DC=ds,DC=i,DC=xxxxx,DC=com
```

