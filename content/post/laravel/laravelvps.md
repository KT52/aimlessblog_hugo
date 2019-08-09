---
title: VPSでLaravelを使用する
date: 2019-06-11
categories: [Laravel]
tags: [laravel,vps]
slug: laravelvps
adsenseTop: true
adsenseBottom: true
---

VPSにLaravelを導入したときのメモ。  
環境はCentOS 7、PHP7.3.6、WebサーバはApache、nginxどちらでもOK。  

VPSやApacheのインストール等は[ネコでもわかる！さくらのVPS講座](https://knowledge.sakura.ad.jp/7938/)を参考にしてください。  

## composerのインストール
---



```
curl https://getcomposer.org/installer | php
```

composerコマンドをどこでも使えるようにするために`composer.phar`を`/usr/local/bin/`に移動

```
mv composer.phar /usr/local/bin/composer
```

composerとコマンドを打って下記のように表示されれば成功。

```
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 1.8.5 2019-04-09 17:46:47

```


## Laravelをインストール
---

#### Laravelインストーラーのダウンロード

```
composer global require "laravel/installer"
```


#### Laravelのプロジェクト作成

/var/www/html/に移動して

```
composer create-project --prefer-dist laravel/laravel プロジェクト名
```

でインストール。

## ドキュメントルートの変更
---

Laravelを使用するにはpublicディレクトリをドキュメントルートにする必要があるのでその設定を行います。

#### httpd.confの設定（Apache)

`sudo vim /etc/httpd/conf/httpd.conf`でhttpd.confの一番下に下記を追加する。  

```conf

NameVirtualHost *:80
<VirtualHost *:80>
DocumentRoot /var/www/html/プロジェクト名/public
ServerName www.example.com
<Directory "/var/www/html/プロジェクト名/public">
    AllowOverride all
</Directory>
</VirtualHost>

```

念のため`httpd -t`で間違いがないか確認。問題がなければ
```
systemctl restart httpd
```

でApacheを再起動。


#### default.confの設定(nginx)

`sudo vim /etc/nginx/conf.d/default.conf`で6行目付近の

```

location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
```

を

```

location / {
        root   /var/www/html/プロジェクト名/public;　#ドキュメントルートの変更
        index  index.php index.html index.htm;　#index.phpを追加
        try_files $uri $uri/ /index.php?$query_string; #追加
    }

```

に変更。

36行目付近の

```html
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000

        location ~ \.php$ {
        root           html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        include        fastcgi_params;
    }
```

を

```html

# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000

    location ~ \.php$ {
        root           /var/www/html/プロジェクト名/public; #ドキュメントルートの変更
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name; #変更
        include        fastcgi_params;
    }

```

間違いがないか`nginx -t` で確認。問題がなければ`sudo systemctl restart nginx`でnginxを再起動。

## パーミッションの変更
---

laravelプロジェクトのstrageディレクトリとbootstrap/cacheディレクトリを書き込み可能にする必要があるので、  
cd /var/www/html/プロジェクト名に移動して

```
chmod 777 -R storage
chmod 777 -R bootstrap/cache/
```

一応Apacheやnginxを再起動。

## ブラウザでアクセス
---

welcomeページが表示されれば成功。  

![laraveltop](../../../images/laraveltop.png)
