---
title: LaravelのページネーションとGETパラメータ
date: 2018-09-02
categories: [Laravel]
tags: [laravel,php]
slug: laravelpagination
adsenseTop: true
adsenseBottom: true
---

## Laravelのページネーション
---

Laravelはページネーションを簡単に実装できます。
取得したデータを１ページ20アイテムずつ表示したいなら
```php

$datas = Data::all();

```

これを

```php

$datas = Data::paginate(20);

```

に変更。bladeテンプレートに以下を追記

```html

{{$datas->links()}}


```

これだけでページネーションを実装できます。  

## GETパラメータとページネーション
---

GETメソッドで送信した情報を元に何らかのデータの一覧を表示するページを作成すると、URLは`http://localhost:8000/catalog?genre=foo`のようになります。このページにLaravelのページネーションを実装すると、１ページ目は表示されても２ページ目以降が表示されない。


１ページ目のURLが`http://localhost:8000/catalog?genre=Nu+gaze`  
２ページ目以降は`http://localhost:8000/catalog?page=2`  
となってしまう。  
?genre=Nu+gazeが消えてしまっているのでこれを`http://localhost:8000/catalog?genre=Nu+gaze&page2`のようにしたい。

例えば、selectで選んだデータの一覧を表示するページを作成したい場合

```html

<form method="GET" action="/catalog">
	<select name="genre">
		<option value="" disabled selected>Choose your option</option>
		@foreach($result as $row)
		<option value="{{$row->genre}}">{{$row->genre}}</option>
		@endforeach
	</select>
	<button type="submit">ボタン</button>
</form>

```

routes\web.phpは

```php

Route::get('/catalog', 'DataController@catalog');

```

コントローラーは

```php

<?php

namespace App\Http\Controllers;

//use DB;
use App\Data;
use Illuminate\Http\Request;

class DataController extends Controller
{

 public function catalog(Request $request)
    {
        $genre = $request->genre;
        $lists = Data::where('genre','=',$genre)->paginate(20);
        return view('catalog',['lists'=>$lists,'genre'=>$genre]);
	}
}

```

bladeテンプレートのページネーションの部分は

```php

{{$lists->links()}}


```

しかしこれだと2ページ目以降が正しく表示されないので、

```php

{{$lists->appends(request()->input())->links()}}

```

に変更する。`{{$lists->appends(request()->query())->links()}}` でも可。

これで2ページ以降も`http://localhost:8000/catalog?genre=Nu+gaze&page2`のようになってきちんと表示されるようになります。<br><br>
参考：[stackoverflow](https://stackoverflow.com/questions/17159273/laravel-pagination-links-not-including-other-get-parameters)