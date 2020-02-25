---
title: "NginxでphpMyAdminを使用する"
date: 2019-08-26
categories: "Misc"
tags: [nginx]
slug: nginxphpmyadmin
adsenseTop: true
adsenseBottom: true
---

apacheからnginxに移行してphpmyadminを使用するにはどうすれば良いのかな？といろいろ調べて  
phpmyadminにアクセスできるようになったのでメモ。  
apache使用時に既にインストール済みなのでphpmyadminのインストールは省略します。

## phpmyadmin.conf

"/etc/nginx/conf.d"以下に"phpmyadmin.conf"を作成。  

ipアドレスorドメインのみでアクセスする場合

```nginx
server {
       listen 80;
       server_name ipアドレスorドメイン;

       location / {
                root /usr/share/phpMyAdmin;
                index index.php;
                }

        location ~ \.php$ {
                fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME /usr/share/phpMyAdmin/$fastcgi_script_name;
                include fastcgi_params;
        }
```

ipアドレスorドメイン/phpmyadminでアクセスする場合の設定。

```nginx
server {
       listen 80;
       server_name  ipアドレスorドメイン;

       location /phpmyadmin {
                alias /usr/share/phpMyAdmin;
                index index.php;
                }

        location ~ ^/phpmyadmin/(.+\.php)$ {
                fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME /usr/share/phpMyAdmin/$1;
                include fastcgi_params;
        }
}
```

nginxを再起動してphpmyadminにアクセス。

## アクセスはできたけどphpmyadmin errorが表示される場合

1. /usr/share/phpMyAdminディレクトリの所有者とグループを変更する。

```sh
chown -R nginx:nginx /usr/share/phpMyAdmin
```

2. php.iniにセッションパスの定義をする。

```sh
sudo vim /etc/php.ini
```

```sh
session.save_path = "/var/lib/php/session"
```

```sh
chown -R nginx:nginx /var/lib/php/session
```

## rootディレクトリにシンボリックリンクを貼る

confファイルにどう書いてもうまく行かないときはrootディレクトリにシンボリックリンクを貼りましょう。

```sh
cd /var/www/html
ln -s /usr/share/phpMyAdmin phpmyadmin
```

これでipアドレスorドメイン/phpmyadminにアクセスできます。  
Laravelならpublicディレクトリにシンボリックリンクを貼ってください。  

最後に、  
/phpmyadminでアクセスできるようにするとセキュリティ的に良くないと思うので/xxxxxxphpmyadminとか適当に名前を変えましょう。