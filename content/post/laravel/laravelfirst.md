---
title: 【自分用メモ】Laravel入門
date: 2018-07-21
categories: [Laravel]
tags: [laravel,php]
slug: laravelfirst
---
## はじめに
---

XdomainでPHP/mysqlサーバーが無料で使えるので久々にPHPを使ってみようと思い、今一番人気のPHPフレームワークLaravelを使ってみた。PHPフレームワークはCodeigniterを以前使用したことがあります。  
公式サイト及び日本語翻訳サイトからベーシックな部分を備忘録的にまとめてみた。

[公式サイト](https://laravel.com){:target="_blank"}  
[日本語訳サイト](https://readouble.com/){:target="_blank"}

<br>**環境**:Laravel5.6 Windows10 Xampp php7.2.6 

### Composerをインストール
---

[Composer](https://getcomposer.org/){:target="_blank"}のダウンロードページから`Composer-Setup.exe`をダウンロードしてインストール。
使用するPHPを指定するときは、XAMPPの中のPHP`C:\xampp\php\php.exe`を指定する。
<br>

### Laravelをインストール
---

<br>
`composer global require "laravel/installer"`でLaravelインストーラーをインストール。<br><br>

### プロジェクトの作成
---

<br>
`laravel new プロジェクト名（ここではmyapp）`と打つとmyappというディレクトリが作成されて、そこに必要とするパッケージが全部揃った、Laravelがインストールされる。  
myappディレクトリに移動して`php artisan serve`でローカルサーバーを起動してブラウザで`http://localhost:8000`にアクセスすると下の画面が表示される。  
![laravel](../../../images/laraveltop.png)  <br><br>

### データベースの設定
---

<br>
`.env`ファイルのデータベース名・ユーザー名・パスワードを変える。xamppはMariaDBが使用されてるけどmysqlのままでOK。

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=hoge
DB_USERNAME=taro
DB_PASSWORD=1234

```
<br><br>

### Eloquent ORM
---

Eloquent ORMというO/Rマッパーを使用するにはモデルを作成する必要がある。<br><br>


### モデルの作成
---

<br>
`php artisan make:model クラス名（ここではBooklist）`と打つとappディレクトリにBooklist.phpが作成される。
マイグレーションに必要なファイルも一緒に生成する場合は`php artisan make:model Booklist -m`。

```php

<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Booklist extends Model
{
    //
}

```

Booklistクラスは空ですが、この場合はクラス名を複数形(booklists)の「スネークケース」にしたものがテーブル名として使用される。<br><br>

### マイグレーション
---

<br>
`php artisan make:migration create_booklists_table --create=booklists`で`database/migrations`ディレクトリにファイルが生成される。createオプションでテーブル名を指定。
上記のモデルの作成時に-mオプションで作成した場合は既にマイグレーションファイルが生成されている。  
ファイル名にはマイグレーションの実行順をフレームワークに知らせるため、名前にタイムスタンプが含まれている。

create_booklists_table.phpを開いてテーブルの定義をする。

```php

<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateBooklistsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('booklists', function (Blueprint $table) {
			$table->increments('id');
			$table->string('title'); /* 追加 */
			$table->string('author'); /* 追加 */
			$table->string('published'); /* 追加 */
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('booklists');
    }
}

```

`php artisan migrate`でマイグレーションを実行。MySQLにBooklistsテーブルが作成される。<br><br>

### マイグレーションを実行して、SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes のエラーが出る場合
---

<br>
Laravelは絵文字の保存をサポートしているのでutf8mb4を使用しています。なので、バージョン5.7.7より古いMySQLや、バージョン10.2.2より古いMariaDBを使用している場合、デフォルトの文字列長255文字だと767バイトを超えてしまうので、   `app\Providers\AppServiceProvider.php`を開き、`use Illuminate\Support\Facades\Schema;`と。` Schema::defaultStringLength(191);`を追加する。


```php

<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Schema; /*追加*/

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191); /* 追加 */
    }

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}

```
<br>

### ルーティング
---

<br>
ルートは`routes/web.php`で定義する。

```php

Route::get('/', function () {
    return 'Hello World';
});
```

`php artisan serve`でローカルサーバーを起動して、`http://localhost:8000` にアクセスするとHello Worldが表示される。<br>

### ビュー

<br>
ビューは`resources/views`ディレクトリに作成する。Bladeテンプレートを使用しているので、***.blade.phpで保存。  
hello.blade.phpで保存しているなら`view('hello')`。  
`resources/views/about`のようなサブディレクトリを作成した場合は、`view(about.テンプレートファイル)`

```php

Route::get('/hello', function () {
    return view('hello');
});

```

```php

Route::get('/hello', function () {
    return view('about.hello');
});

```

`http://localhost:8000/hello`で表示される。<br>


### コントローラー

<br>
ルートファイルに全リクエストの処理を書いても問題ないけど、分離して管理したいときはコントローラクラスを使用する。  
コントローラーの作成は`php artisan make:controller コントローラ名`で、app\Http\Controllers\に作成される。

```php

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TestController extends Controller
{
    //
}
```

例としてテーブルの全データを表示するページを作りたいときは、

```php

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use app/Booklist; /* <---- */

class TestController extends Controller
{
    public function alllist()
    {
        $results = Booklist::all();
        return view('alllist',['results'=>$results]);
    }
}

```

使用するモデルを定義`use app/Booklist;`して、  
ファンクション'alllist'にクエリとビューを記述。    
ルートファイルは

```php

Route::get('/alllist', 'TestController@alllist');

```

コントローラ名＠ファンクション名でメソッドが実行される。<br><br>

### おわりに

とりあえず必要最低限の情報をまとめてみた。  
他に必要そうなEloquent ORMのクエリとかフォームのPOSTされたデータの取得とかBladeテンプレートとかも書くと長くなるので今回は省略。