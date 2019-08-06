---
title: Laravelの認証機能(Auth)
date: 2018-07-25
categories: [Laravel]
tags: [laravel,php]
slug: laravelauth
related_posts: laravelmail
---
Laravelの認証機能をローカル環境で試してみた。環境はLaravel5.6 Windows10 Xampp php7.2.6

### 認証機能を生成
---
```
php artisan make:auth
```

このコマンドで認証機能に必要なファイルが実装される。ルートの設定やビューも勝手に作ってくれるのでめっちゃ楽。既にアプリを作成中の場合でも`routes/web.php`の一番下に

```php
Auth::routes();

Route::get('/home', 'HomeController@index')->name('home');
```
が記述されている。

### データベースの準備
---
database\migrations\にある`_create_users_table.php`と`_create_password_resets_table.php`を利用するデータベースにマイグレーションします。xamppのMySQLを起動するのを忘れずに。

```
php artisan migrate
```

マイグレーションすると使用するデータベースにusersテーブルとpassword_resetsテーブルが作成される。

### 表示してみる
---
`php artisan serve`でローカルサーバーを起動して`localhost:8000`にアクセスすると右上にloginとregisterの文字が追加されています。（ルート'/'が return view('welcome');の場合）

![laravel](../../../images/laraveltop2.png)  

Register画面

![laravel](../../../images/laravelreg.png)  

Login画面

![laravel](../../../images/laravellogin.png) 

### テスト登録
---
registerページでユーザー名、Emailとパスワードを入力するとMySQL/MariaDBに登録されるのでphpmyadminで確認してみると、登録したパスワードが暗号化？されている
![laravel](../../../images/password.png) 

ログインすると`localhost:8000/home`に飛ばされてログイン状態になる。

![laravel](../../../images/logincyu.png) 

ログアウトは右上のユーザー名をクリックしてログアウトを選択。

### パスワードのリセット
---
パスワードのリセットは送信されたメールから行うので、送信するメールサーバーの設定が必要。`.env`のこの部分を自分の環境に合わせて設定する。

```env

MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null

```

ログイン画面の'Forgot Your Password?'をクリックするとEmail入力画面が表示されるので登録したメールアドレスを入力して送信するとメールが届く。  

![laravel](../../../images/reset.png) 

Reset Passwordボタンをクリックした先のページで新しいパスワードを入力する。  
ローカルサーバーでテスト中の場合はアドレスがhttp://localhost/password/reset/英数字の羅列になってるのでlocalhost:8000に修正してアクセスしてください。

### ルートの保護
---
(2019/04/30追記）認証済みユーザーのみのアクセスを許したい場合は、ルートミドルウェアを使います。  
ルート定義でルートミドルウェアを指定するだけで済みます。  

```php
<?php

Route::get('profile', function() {
    // 認証済みのユーザーのみが入れる
})->middleware('auth');

```

コントローラを使っているなら、コントローラのコンストラクターでmiddlewareメソッドを呼び出すだけでいい

```php
<?php

public function __construct()
{
    $this->middleware('auth');
}

```

### おわりに
---
`php artisan make:auth`と`php artisan migrate`を実行すればこちらで何かを設定することなく「ログイン」「ユーザ登録」「パスワードリセット」機能を実装できるのですごく簡単。  
あとは英語表記を日本語に修正したり、実際に使用する場合のルート定義の変更等をすれば個人サイトレベルだったら十分使えそう。  

より詳しい内容は日本語ドキュメントを参照してください→[
Laravel 5.6 認証](https://readouble.com/laravel/5.6/ja/authentication.html)<br><br>
実はメール送信がどうしてもできなくてつまずいたんだけどその解決法は次回書きます。

