---
title: Laravelでメール送信できない時の解決法
date: 2018-07-26
categories: [Laravel]
tags: [laravel,php]
slug: laravelmail
related_posts: laravelauth
adsenseTop: true
adsenseBottom: true
---

[前回](https://www.ravness.com/2018/07/laravelauth)
Laravelの認証機能のパスワードリセットでメール送信できなかった問題の解決法。

### .envの最初の設定
---

`.env`のメール設定はデフォルトで下のようになっている。

```ini

MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null

```

これをGmailから送信するので下のように設定したがエラーが出て送信できなかった。

```ini

MAIL_DRIVER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=xxxxxxxx@gmail.com
MAIL_PASSWORD=xxxxxxx
MAIL_ENCRYPTION=tls

```

![certificate verify failed](../../../images/certificateverifyfailed.png)

### 解決法
---

configディレクトリの`mail.php`を開き、下記を追加する。

```php

'stream' => [
    'ssl' => [
        'allow_self_signed' => true,
        'verify_peer' => false,
        'verify_peer_name' => false,
	],
],

```

そして、.envに以下の追加をする。

```ini

MAIL_FROM_ADDRESS=送信元のメールアドレスを記述
MAIL_FROM_NAME=送信元の名前

```

MAIL_FROM_NAMEは無くても大丈夫だけど一応入れてみた。

Gmailを使う場合は[Googleアカウント](https://myaccount.google.com/)から`ログインとセキュリティ`のページに移動して`安全性の低いアプリの許可: 無効`を有効に変更する。


.envとmail.phpを変更した後は`php artisan cache:clear`でキャッシュをクリアするのを忘れずに。

これでGmailとoutlookメールで試してみたところ無事送信できました。しかし、Gmailの2段階認証を使用している場合はまたエラーが出ることになる……

### Gmailで2段階認証を利用している場合
---

- [Googleアカウント](https://myaccount.google.com/)からログインとセキュリティ
- Googleへのログインのところにある`アプリ パスワード`をクリック。
- アプリを選択をクリックしてその他（名前を入力）を選択。適当に名前をつける。
- 生成を押すと16桁のパスワードが生成される。
- このパスワードを.envのMAIL_PASSWORD=に記入する

アプリパスワードを使用すると、2段階認証プロセスに対応していない端末上のアプリからGoogleアカウントにログインできるようになります。  
これでLaravelからGmailを利用して送信できるようになります。  

参考サイト：[stackoverflow Laravel SMTP driver with TLS encryption
](https://stackoverflow.com/questions/30714229/laravel-smtp-driver-with-tls-encryption)