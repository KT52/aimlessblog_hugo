---
title: Flask-SQLAlchemyとFlask-WTF その１
date: 2019-07-15
categories: [Python]
tags: [python,flask]
slug: flasksqlalchemy_wtfoms1
---

Flaskはローカルで使用してたのでデータベースはSQLを直に書いていたのですが、LaravelのEloquent ORMを利用してSQLAlchemyにも興味を持ったので[Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/)を使ってみました。

## Flask-SQLAlchemyのインストール
---

```
pip install flask-sqlalchemy
```

SQLAlchemyがインストールされていない場合はSQLAlchemyも一緒にインストールされる。<br>

## アプリ作成
---

とりあえず触れてみるだけなのでtest.pyとtemplatesディレクトリを作成するだけの環境で作業します。  
実際にアプリを作成する場合はconfigやmodelを別ファイルに分けたほうがいいでしょう。<br>

### 環境設定とモデル定義


test.py

```python

from flask import Flask, request, redirect, url_for, render_template
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

#　sqlite3のdbファイルがある場所
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///test.db"

db = SQLAlchemy(app)

# モデル定義
class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(20), unique=True, nullable=False)
    job = db.Column(db.String(30), nullable=False)

    def __repr__(self):
        return '<User %r,%r,%r>' % (self.id,self.username,self.job)

```

データベースはSQLite3を使用します。id（プライマリーキー、整数型）,username,jobの3つのカラムを持ったuserテーブルを作成。  
nullable=Falseはnullを許可しない設定。<br>

##　テーブル作成
---

pythonシェル上でテーブル作成。仮想環境で、

```
>>> from test import db
>>> db.create_all()
```

でtest.dbとモデル定義したuserテーブルが作成される。<br>

## Insert
---

とりあえず二人ユーザーを追加してみる

```
>>> from test import User
>>> user1 = User(username = "Yamada",job = "Fighter")
>>> user2 = User(username = "Oda", job = "Priest")
>>> db.session.add(user1)
>>> db.session.add(user2)
>>> db.session.commit()
```

db.session.add()でセッションに追加してcommit()でセッションをコミットするという流れでデータベースに挿入される。<br>

## Query
---

query.all()で全レコードを呼び出す

```
>>> User.query.all()
[<User 1,'Yamada','Fighter'>, <User 2,'Oda','Priest'>]
```

filter_byがwhere句でfirst()が条件にマッチした最初のレコードを取得

```
>>> User.query.filter_by(username='Yamada').first()
<User 1,'Yamada','Fighter'>

>>> user = User.query.filter_by(username='Oda').first()
>>> user.job
'Priest'
```

get()でプライマリーキーを元に呼び出すことができる。

```
>>> User.query.get(1)
<User 1,'Yamada','Fighter'>
```

<br>

## Delete
---

```
>>> user = User.query.filter_by(username='Oda').first()
>>> db.session.delete(user)
>>> db.session.commit()
```
<br>

## Update
---

```
>>> from test import User,db
>>> r = User.query.get(2)
>>> r.username = 'Kubo'
>>> db.session.commit()
#　確認
>>> User.query.all()
[<User 1,'Yamada','Fighter'>, <User 2,'Kubo','Priest'>]
```

Flask-SQLAlchemyの基本がわかったので次回はFlask-WTFでフォームを作って、insert、update、delete機能をつけたページから操作してみようと思います。