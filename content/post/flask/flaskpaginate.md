---
title: FlaskのページネーションはFlask-paginateを使うと簡単に実装できる
date: 2019-07-03
categoies: [Python]
tags: [python,flask]
slug: flaskpaginate
adsenseTop: true
adsenseBottom: true
---

[Flask Paginate](https://flask-paginate.readthedocs.io/en/latest/)はシンプルなFlaskのページネーション拡張で、
cssフレームワークのBootstrapとfoundationに対応しています。

## インストール
---

```
pip install -U flask-paginate
```

## 使い方
---

例としてsqlite3を使用して、何らかのデータ一覧を表示する場合。

```py
#!  /usr/bin/env python
from flask import Flask, request,render_template, g
from flask_paginate import Pagination, get_page_parameter
import sqlite3

DATABASE = 'data.db3'

def connect_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
        db.row_factory = sqlite3.Row
    return db

app = Flask(__name__)

@app.route('/')
#省略

@app.route('/pagination', methods=['POST','GET'])
def pic_view():
    if request.method == 'GET':
        name = request.args.get('name')
        con = connect_db()
        cur = con.execute("SELECT * FROM table WHERE name=?",(name,))
        result = cur.fetchall()
        cur.close()
        page = request.args.get(get_page_parameter(), type=int, default=1)
        res = result[(page - 1)*10: page*10]
        pagination = Pagination(page=page, total=len(result),  per_page=10, css_framework='bootstrap4')
        return render_template('example.html', rows=res, pagination=pagination)

if __name__ == "__main__":
    app.run(debug=True)
```


page番号をgetして変数pageに代入。クエリーパラメータがない場合はデフォルトの1


```py
page = request.args.get(get_page_parameter(), type=int, default=1)
```


Pagination()でページネーションに必要なパラメータを記載。

```python
pagination = Pagination(page=page, total=len(result),  per_page=10, css_framework='bootstrap4')
```


- page: 現在のページ
- total:　全レコード数。len()でカウント。
- per_page: 1ページあたりのレコード数。今回は10件取得。
- css_framework: 使用するCSSフレームワーク


レコードの中の何件目から何件目まで取得するかをスライスで取得。

```python
  res = result[(page - 1)*10: page*10]
```

これを設定しないと全ページ全レコードが表示されてしまう。  
公式サイトにオフセットの設定は書いてなかったので少しハマったところ。  

参考: [どのように１ページごとに表示させるアイテム数とページネータをコントロールすればいいでしょうか？](https://ja.stackoverflow.com/questions/44885/%E3%81%A9%E3%81%AE%E3%82%88%E3%81%86%E3%81%AB%EF%BC%91%E3%83%9A%E3%83%BC%E3%82%B8%E3%81%94%E3%81%A8%E3%81%AB%E8%A1%A8%E7%A4%BA%E3%81%95%E3%81%9B%E3%82%8B%E3%82%A2%E3%82%A4%E3%83%86%E3%83%A0%E6%95%B0%E3%81%A8%E3%83%9A%E3%83%BC%E3%82%B8%E3%83%8D%E3%83%BC%E3%82%BF%E3%82%92%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8C%E3%81%B0%E3%81%84%E3%81%84%E3%81%8B)

## ビュー
---

```html
{{ pagination.info }}
{{ pagination.links }}
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
    </tr>
  </thead>
  <tbody>
    {% for row in rows %}
      <tr>
        <td>{{ row.name }}</td>
        <td>{{ row.email }}</td>
      </tr>
    {% endfor %}
  </tbody>
</table>
{{ pagination.links }}
```


`{{ pagination.links }}`でページネーションが作成される。  
`{{ pagination.info }}`は"displaying 21 - 30 records in total 163"みたいなインフォメーション、なくてもいい。

こんなかんじでよく見る？Bootstrapのページネーションが簡単に実装できました。

![flaskpage](../../../images/flask_page.gif)