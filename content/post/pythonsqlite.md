---
title: Python+SQLiteのlike句で％を使う場合のプレースホルダの書き方
date: 2018-07-06
categories: [Python]
tags: [Python,SQLite]
slug: pythonsqlite
---

Python（+Flask）+SQLiteでパターンマッチングしたものを抽出するときに次のようなクエリを実行すると、<br>
```python
@app.route('/', methods=['POST'])
def searchPage():
    if request.method == 'POST':
        word = request.form['word']
        g.db = connect_db()
        g.db.execute("SELECT * FROM school WHERE name like '%?%'",(word,))
以下略
```
```
sqlite3.ProgrammingError: Incorrect number of bindings supplied. The current statement uses 0, and there are 1 supplied.
```

のようなエラーが出てしまう。 <br>
<br>
####解決法

変数側に％を記述する。
```python
g.db.execute("SELECT * FROM school WHERE name like ?",('%'+word+'%',))
```

<br>あと、よくあるミスとして変数の項目が1個のときもexecuteの第2引数をタプルにしないといけないのを忘れるのも注意。   
<br>[pythonドキュメントより](https://docs.python.jp/3/library/sqlite3.html)<br>
> ? を変数の値を使いたいところに埋めておきます。その上で、値のタプルをカーソルの execute() メソッドの第2引数として引き渡します。

`('%'+word+'%')`ではなく`('%'+word+'%',)`
