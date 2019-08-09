---
title: Laravelで自作関数を使う
date: 2019-05-10
categories: [Laravel]
tags: [laravel]
slug: laravelfunction
adsenseTop: true
adsenseBottom: true
---

Laravelでヘルパー関数以外に自作した関数を使う方法。  
Composerでオートロードしたりサービスプロバイダを使用しないやり方です。

## クラスの作成
---

ファイルの場所は多分どこでも良いのですがとりあえずapp/以下にLibディレクトリを作ってその中にMy_func.phpを作成。  
staticメソッドのクラスを作成します。

```php

<?php

namespace App\Lib;

class My_func
{
  public static function xxx()
  {
    //code...
  }
}

```


## エイリアスの登録
---

`config/app.php`のaliasesにクラスを追加。コントローラーでしか自作関数を使わないなら登録しなくてもいいです。

``` php
<?php

  'aliases' => [
    'My_func' => App\Lib\My_func::class,
  ],
```


## コントローラーで使う
---

コントローラーに`use My_func`で使用可能。呼び出すときは`My_func::xxx()`で。  
エイリアスの登録を行わなかった場合は`use App\Lib\My_func`

```php

<?php

namespace App\Http\Controllers;

use My_func;

class YourController extends Controller
{
    public function aaa()
    {
        $a = My_func::xxx();
        return view('',['a'=>$a]);
    }
}

```


## bladeテンプレートで使う
---

エイリアスに登録しているならbladeテンプレートで使う場合も`{{My_func::xxx()}}`で呼び出し可能。  
ifやforeachでも`@foreach (My_func::xxx() as $zzz) {{$zzz}} @endforeach`のように使用できます。
