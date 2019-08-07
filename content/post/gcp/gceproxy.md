---
title: GCEでBasic認証付きのプロキシサーバーを立てる
date: 2019-04-21
categories: [GCP]
tags: [vps,gce]
slug: gceproxy
adsenseTop: true
adsenseBottom: true
---

Google Compute Engine(GCE)でBasic認証付きのプロキシサーバーを立ててみた。

環境はCentOS 7 Apache 2.4.6

## Squidをインストール
---

最初にプロキシサーバソフトのSquid（スクイッド）をインストール

```
yum install squid

```

## squid.confの編集
---

```
vim /etc/squid/squid.conf
```

26行目付近の`acl CONNECT method CONNECT`の下に

```conf

auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic realm Squid proxy-caching web server
acl password proxy_auth REQUIRED

```

を追加。<br>`/etc/squid/passwd`が後で設定するベーシック認証のパスワードを書き込むファイルの場所。<br>

63行目付近の

```conf
# And finally deny all other access to this proxy
http_access deny all

```

に1行追加して
```conf
# And finally deny all other access to this proxy
http_access allow password
http_access deny all
```

`http_access allow password`でパスワード認証を許可する

<br>

デフォルトのポート番号3128を他の番号に変更するので67行目付近の`http_port 3128`をコメントアウトして新しいポート番号を追加。

```conf
# Squid normally listens to port 3128
#http_port 3128
http_port 35488
```

後はsquid.confの最下行に以下を追加。

```conf
# プロキシサーバーを使用している端末のローカルIPアドレスを隠蔽化
forwarded_for off
visible_hostname unknown
# プロキシ経由でアクセスしていることをアクセス先に知られないようにする
request_header_access X-Forwarded-For deny all
request_header_access Via deny all
request_header_access Cache-Control deny all
```

以上でsquid.confの編集は終わり。

```
systemctl enable squid
```
で自動起動設定。

```
systemctl start squid
```
でSquidを起動。

## パスワード設定
---

htpasswdコマンドを使ってbasic認証をかけるため`httpd-tools`をインストールする。
```
yum install httpd-tools
```

インストールしたらユーザーネームとパスワードを設定するので
```
htpasswd -c /etc/squid/passwd ユーザーネーム
```

enterキーを押すと設定したいパスワードを聞かれるので入力してenter、再入力を求められるのでもう一度パスワードを入力してenter。  
これでパスワードの設定は終わり。

## GCEのファイアウォールルールの作成
---

GCEのファイアウォールルールの作成は[前回記事](https://www.ravness.com/2019/03/gceport/)の`ファイアウォールルールの作成`と`VMインスタンスにネットワークタグを設定`を参照してください。

## ブラウザでプロキシサーバーの設定
---

すべての準備が整ったので実際にブラウザでアクセスしてみる。<br>
Chrome拡張のSwitchyOmegaは特定のタブやサイトだけプロキシ接続できるのでオススメです。<br>
通常のブラウザからの設定の場合はアドレスとポート番号を設定した後にユーザー名とパスワードの入力を求められるのでhtpasswdで設定した情報を入力するとプロキシ経由でネットに接続できます。

[確認くん](http://www.ugtop.com/spill.shtml)で確認してみると、ポート番号はGCEの外部IP、ゲートウェイの名前は～～googleusercontent.comに変わっています。<br><br>
![確認くん](../../../images/gceproxy.jpg)

<br>

プロキシサーバーを経由して長時間動画を見たりするとGCEを無料で運用することができなくなるので注意。<br>
念のためGCPのお支払い→予算とアラートの設定をしておくと良いかも。
