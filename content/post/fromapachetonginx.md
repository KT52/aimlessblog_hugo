---
title: CentOS7でApacheからNginxに移行するときにした事
date: 2019-08-05
categories: ["Misc"]
tags: ["nginx",vps]
slug: fromapachetonginx
---

ApacheからNginxに移行するときにした事のメモ。
Apache環境下でPHP7.3とMariaDBを使用していました。

## Apacheの停止
---

ApacheとNginxを併用したりはしないのでApacheを止めます。

```
sudo systemctl stop httpd
```

自動起動も停止

```
sudo systemctl disable httpd
```

## nginx用レポジトリファイルの作成
---

ngninxをインストールするためにリポジトリを追加します。

**/etc/yum.repos.d/** 内に **nginx.repo** を作成。

```
sudo vim /etc/yum.repos.d/nginx.repo
```

以下をnginx.repoにコピペ
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/
gpgcheck=0
enabled=1
```

## nginxのインストール
---

```
sudo yum -y --enablerepo=nginx install nginx
```

インストールが成功したら起動。

```
sudo systemctl start nginx
```

ステイタス確認

```
sudo systemctl status nginx
```

`Active: active (running)`と表示されているならちゃんと起動している。

自動起動設定

```
sudo systemctl enable nginx
```

ブラウザでアクセスしてnginxのトップ画面が出れば成功。

![nginxtop](../../../images/nginxtop.jpg)

## php-fpmのインストール
---

Apache（モジュール版）と違いNginxではPHPをFastCGIを通して実行するのでphp-fpmを導入する必要があります。  

```
sudo yum install --enablerepo=epel,remi,remi-php73 php-fpm
```

僕はPHPををeventMPM+PHP-FPMで使用していたのでPHP-FPMはすでにインストール済みでした。


## php-fpmの設定
---

nginxとphp-fpmどちらから設定しても構わないのですがとりあえずphp-fpmの設定から

```conf
sudo cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/www.conf.old #念の為オリジナルをバックアップ
sudo vim /etc/php-fpm.d/www.conf
```

25行目前後のuserとgroupをnginxにする。

```
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
; RPM: apache Choosed to be able to access some dir as httpd
;user = apache
user = nginx
; RPM: Keep a group allowed to write in log dir.
;group = apache
group = nginx

```

pm.max_children、pm.start_servers、pm.min_spare_servers、pm.max_spare_servers、pm.max_requestsの設定は各環境により異なるので省略します。  

設定が終わったらphp-fpmの起動と自動起動の設定をする。

```
sudo systemctl start php-fpm
```

エラーがなかったら自動起動の設定

```
sudo systemctl enable php-fpm
```


## default.confの設定
---

次にnginxのdefault.confの編集をします。

まずはバックアップ
```
sudo cp /etc/nginx/conf.d/default.conf default.conf.old
```
```
sudo vim /etc/nginx/conf.d/default.conf
```

でdefault.confの編集。  
6行目くらいの

```
 location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
```

を

```
 location / {
        root   /var/www/html;
        index  index.html index.htm index.php;
```

indexにindex.phpを追加。rootもapacheで使用していたディレクトリに変更。  
36行目付近の

```text

# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    # 
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}
```

ここのコメントアウトを外して、rootとfastcgi_paramの2箇所を変更します。

```text
 # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000

    location ~ \.php$ {
        root           /var/www/html; #変更箇所
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;　#変更箇所
        include        fastcgi_params;
    }
```

変更して保存したら、構文に間違いがないか確認してなかったらnginxを再起動します。

```
sudo nginx -t
sudo systemctl restart nginx
```

## ブラウザからアクセス
---

ルートディレクトリ(/var/www/html/)にtest.phpファイルを置いてphpがちゃんと動作してるか確認します。  
で

```php
sudo vim index.php
#中身は
<?php phpinfo(); ?>
```

ブラウザからhttp://xxxxxx/test.phpにアクセスして

![phpfpm](../../../images/php-fpm.jpg)

Server APIが`FPM/FastCGI`になっていればphp-fpmが正常に動作しています。
