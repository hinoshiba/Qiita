( gitlab + ActiveDirectory認証 ) + フロントはNginx
=============================

* https://qiita.com/hinoshiba/items/e2ee75faa79b2683343a


#はじめに
* Nginxをすでに利用しているサーバで構築したかった
* そしてローカルに立っているドメコンに認証させたかった

#ポイント
* php-fpmと共存させるnginxの設定を行う場合、gitlab php-fpmの順でコンフィグを書く
 * そうしないと.phpファイルのレビューが表示できない
* gitlabはreconfigureでパーミッションを書き換えてしまうので、設定変更の度、ソケットの権限変更が必要な場合がある
 * 再起動後、502エラーが継続して発生する場合は該当することが多かった

#実行環境
* ubuntu 16.04 （x64）
* root権限

#ドメイン情報
* ドメイン名：ds.xxxx
 * ds.xxxx
* 認証ユーザ
 * グループ：ds.xxxx/Hi-Users/System
 * 名前：gitlab
 * CN=gitlab,OU=System,OU=Hi-Users,DC=ds,DC=xxxx

#導入手順
##インストール
###インストールスクリプト入手
```
wget https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh
```
###リポジトリスクリプトの修正(proxy追加）
```
vim script.deb.sh
```
```diff
+ export http_proxy=http://proxy.i.xxxx:8080/
+ export https_proxy=http://proxy.i.xxxx:8080/
+ export ftp_proxy=http://proxy.i.xxxx:8080/
```
### リポジトリスクリプトの実行
```
bash script.deb.sh
```
### インストール
```
apt update
apt install gitlab-ce
```
## gitlab設定変更
### 設定ファイル変更
```
vim /etc/gitlab/gitlab.rb
```
```diff
+ 13 external_url 'https://www.xxxxx/gitlab/'
+ 111 gitlab_rails['trusted_proxies'] = ['127.0.0.1']
+ 223 gitlab_rails['ldap_enabled'] = true
+ 226 gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
+ 227    main:
+ 228      label: 'LDAP'
+ 229      host: 'ds.xxxx'
+ 230      port: 389
+ 231      uid: 'sAMAccountName'
+ 232      bind_dn: 'CN=gitlab,OU=System,OU=Hi-Users,DC=ds,DC=xxxx'
+ 233      password: '************'
+ 234      encryption: 'plain'
+ 235      active_directory: true
+ 236      allow_username_or_email_login: true
+ 237      block_auto_created_users: false
+ 238      base: 'DC=ds,DC=xxxx'
+ 239      user_filter: ''
+ 240      attributes:
+ 241       username: ['uid', 'userid', 'sAMAccountName']
+ 242       email:    ['mail', 'email', 'userPrincipalName']
+ 243       name:       'displayName'
+ 244       first_name: 'givenName'
+ 245       last_name:  'sn'
+ 246 EOS
+ 636 unicorn['socket'] = '/var/opt/gitlab/gitlab-rails/sockets/gitlab.socket'
+ 883 web_server['external_users'] = ['www-data']
+ 897 nginx['enable'] = false
```
### 設定ファイル読み込みと停止
```
gitlab-ctl reconfigure
gitlab-ctl stop
```
### nginx設定
* sslやnginxの設定については割愛

```
vim /etc/nginx/conf.d/ssl.conf
```
```diff
+ 1 upstream gitlab-workhorse {
+ 2   server unix:/var/opt/gitlab/gitlab-workhorse/socket;
+ 3 }
+         location ^~ /gitlab/ { 
+                root /opt/gitlab/embedded/service/gitlab-rails/public;
+                client_max_body_size 0;
+                gzip off;
+                proxy_read_timeout      300;
+                proxy_connect_timeout   300;
+                proxy_redirect          off;
+                proxy_http_version 1.1;
+                proxy_set_header    Host                $http_host;
+                proxy_set_header    X-Real-IP           $remote_addr;
+                proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
+                proxy_set_header    X-Forwarded-Proto   $scheme;
+                proxy_pass http://gitlab-workhorse;
+        }
```
## ユーザの追加
* gitユーザを作成する必要がある

```
usermod -a -G git www-data
mkdir /home/git
chown git:www-data /home/git
```
## 起動
* gitlabを起動してから、nginxを起動する

```
gitlab-ctl restart   
```
*  gitlab　pid権限変更

```
chown git:www-data /var/opt/gitlab/gitlab-workhorse
```
* nginx再起動

```
systemctl restart nginx
```

