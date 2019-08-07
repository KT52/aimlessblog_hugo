---
title: "VPSにCodeIgniter3を入れてURLからindex.phpを除去する"
date: 2019-01-31
tags: ["vps","codeigniter","php"]
draft: false
slug: ci3index
adsenseTop: true
adsenseBottom: true
---

CodeIgniter利用者のほとんどが要らないと思っているURLにindex.phpがついてる問題。
.htaccessだけではダメだったのでメモ。  
環境はApache, CentOS 7 64Bit, PHP 7.1<br>

## .htaccess
---

.htaccessの設定は[CodeIgniterの日本語ドキュメント](https://codeigniter.jp/user_guide/3/general/urls.html)
と同じで

```.htaccess

RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php/$1 [L]

```

ドキュメントルートが`/var/www/html/（CodeIgniterの）application`ではなく`/var/www/html/CodeIgniter-3.1.0/application`  
みたいな構造の場合は`RewriteBase /CodeIgniter-3.1.0/`を追加しないとアクセスできないかも。<br>

## httpd.conf
---

CodeIgniterのindex.phpの消し方をググると出てくる.htaccessの色々な書き方を試したけどアクセスできずにいたら、
>.htaccessを有効にするためには、ドキュメントルート以下でのAllowOverrideを許可する必要があります。

と書いてあるサイトを発見。  
ということで、`/etc/httpd/conf/httpd.conf`の`<Directory /var/www/html/>～～</Directory>` 内の `AllowOverride None` を `AllowOverride All`に変更。  
`systemctl restart httpd`でApache再起動。  
これでindex.phpなしのURLでアクセスできました。