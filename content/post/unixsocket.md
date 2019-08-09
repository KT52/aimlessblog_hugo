---
title: NginxとPHP-FPMの通信をTCPからUNIX ドメインソケットに変更する
date: 2019-08-09
categories: Misc
tags: [php,vps,nginx]
slug: unixsocket
adsenseTop: true
adsenseBottom: true
---

NginxとPHP-FPMの通信をデフォルトのTCPからUNIX ドメインソケットに変更してみます。  
環境はCentOS７ PHP7.3。

## default.conf

まずはdefault.confから編集。

```
sudo vim /etc/nginx/conf.d/default.conf
```

```conf
 location ~ \.php$ {
        root           /var/www/html/;
        #fastcgi_pass   127.0.0.1:9000;
        fastcgi_pass unix:/run/php-fpm/php-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
```

fastcgi_passを変更します。

## www.conf

次にwww.confを編集します。

```
sudo vim /etc/php-fpm.d/www.conf
```

- 変更箇所その１

```
;listen = 127.0.0.1:9000　コメントアウトする
listen = /run/php-fpm/php-fpm.sock
```

    パスはfastcgi_passと同じ。

- 変更箇所その２

```
;listen.owner = nginx
;listen.group = nginx
;listen.mode = 0660

;上記の部分のコメントアウトを外す

listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```


変更したらphp-fpmとnginxを再起動します。

```
sudo systemctl restart php-fpm
```

```
sudo systemctl restart nginx
```

以上です。