---
title: "CentOS7+NginxでLet's EncryptのSSL証明書を取得する"
date: 2019-10-28
categories: "Misc"
tags: [nginx]
slug: nginxssl
adsenseTop: true
adsenseBottom: true
---

CentOS7+Nginxの環境でLet's EncryptのSSL証明書を取得します。

## certbotのインストール

証明書を取得するのに必要な「certbot」をインストールします。

```sh
sudo yum install certbot
```

## 証明書を取得する

今回はサーバーを停止しないで取得できるwebrootオプションを使います。

```sh
sudo certbot certonly --webroot -w /var/www/html/ -d example.com
```

-w 以下にドキュメントルート、-d 以下にホスト名を入力します。
ワイルドカード証明書も取得したい場合は「-d example.com」のあとに`-d *.example.com` を追加します。

上のコマンドを打つと初回時のみメールアドレス登録や同意しますか？みたいな質問があるのでそれに答える。
質問が終わると証明書の発行の準備が始まって、`- Congratulations!` と表示されれば成功。  
`/etc/letsencrypt/live/example.com/`以下に証明書のシンボリックリンクが作成されます。  
実ファイルは`/etc/letsencrypt/archive/`以下に作成される。


## DH交換鍵の作成

最近のnginxは`ssl_dhparam`が必要（個人サイトレベルなら必要ない？）らしいのでDH交換鍵を作成します。  
以下のコマンドで作成します。

```sh
sudo openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

サーバーによって違いますが、多分1、2分で作成できます。

## nginxの設定

取得した証明書をnginxに設定

```nginx
server {
    listen 443 ssl;
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;
}
```

`ssl_certificate`にfullchain.pem（証明書と中間証明書を連結したファイル）  
`ssl_certificate_key`にprivkey.pem（秘密鍵）  
ssl_dhparamに先程作成したDH交換鍵のパスを設定します。  
listen 443からssl_certificate_keyまでが最低限必要な設定。  

`sudo systemctl restart nginx` で再起動して、ブラウザでhttps\://example.com にアクセスすると鍵マークがついて、SSL化されています。

## 証明書の更新テスト

証明書の有効期限は3ヶ月で更新は30日前からしかできないので、--dry-runのオプションで更新できるかテストをします。  
更新はcertbot renewコマンドで

```sh
sudo certbot renew --dry-run --webroot -w /var/www/html
```

`Congratulations!`と表示されれば更新（テスト）成功。

## cron

更新を自動化するためにcronを設定。  

```sh
sudo vim /etc/crontab
```

に

```sh
0 3 * * * root /usr/bin/certbot renew --webroot -w /var/www/html --post-hook "systemctl restart nginx"
```

これは、毎日午前３時にcronを実行、--post-hookオプションで更新された場合はnginxを再起動する設定。

## 有効期限の確認

`/etc/letsencrypt/live/example.com`に移動して以下のコマンド

```sh
openssl x509 -noout -dates -in cert.pem
```

で証明書の有効期限を確認することができます。

```sh
notBefore=Sep 30 02:05:55 2019 GMT
notAfter=Dec 29 02:05:55 2019 GMT
```

notBeforeが更新日でnotAfterが有効期限です。自動更新されているかここをみて確認してください。

