---
title: FlaskとVueでCRUD操作（バックエンド編）
date: 2019-12-25
categories: ["Flask"]
tags: ["flask","vue"]
slug: flaskvuebackend
adsenseTop: true
adsenseBottom: true
---

FlaskとVue.jsでCRUD機能のついた簡単なWebアプリ（ブックリスト）を作成してみました。  
実際には同時進行で作成していますがバックエンドとフロントエンドに分けて記事を書いていきます。

## 構成

vue_apiディレクトリの下にfrontend(vue.js)とbackend(flask)ディレクトリを作成。
flaskは一応モデルだけ分離して作成しています。  
flaskとvueの連携については[前回の記事](https://ravness.com/2019/12/flaskvue/)を参照してください。

```
vue_api
   |--frontend
   |--backend
         |--model
         |   |--model.py
         |   |--book.db
         |--app.py
         |--config.py
```

## ライブラリのインストール

アプリを作成するにあたって必要なライブラリを仮想環境下でインストールします。  
インストールしたライブラリは下の5つです

- Flask-RESTful ……REST API構築用ライブラリ
- Flask-SQLAlchemy ……ORM
- Flask-CORS ……オリジン間リソース共有ライブラリ
- Flask-Marshmallow ……ORM等から取得したデータをjsonに変換できるようにするライブラリ
- marshmallow-sqlalchemy ……Flask-Marshmallowだけではエラーが出たので追加

## app.py

```python

from flask import Flask, request, render_template, jsonify, make_response
import json
from model.model import init_db,Bookmodel
from flask_restful import Api, Resource
from flask_cors import CORS

app = Flask(__name__,
                static_folder="../frontend/dist/static",
                template_folder="../frontend/dist")

app.config.from_object('config.BaseConfig')

cors = CORS(app)

@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def index(path):
    return render_template('index.html')

#app.pyでflaskアプリを構築してinit_db関数経由でmodel.pyに渡して初期化
init_db(app)
api = Api(app)
book = Bookmodel()

###### flask_restful ######
class Bookapp(Resource):
    def get(self):
        result = book.all_list()
        r = make_response(result)
        return r

    def put(self):
        book.update()
        r = make_response(jsonify({"message": "Update Succeeded"}))
        return r

    def delete(self):
        book.delete()
        r = make_response(jsonify({"message": "Delete Succeeded"}))
        return r

    def post(self):
        id = book.add()
        r = make_response(id)
        return r

api.add_resource(Bookapp, '/api/bookapp')

if __name__ == "__main__":
    app.run()
```

`Api(app)`でインスタンスを生成してapiに代入。`add_resource()`でapiにリソースを追加して、第2引数でルーティングの設定。  
具体的なリソースはクラス`class Bookapp(Resource):`以下で作成。  
クラスの中でhttpメソッドに対応した関数（GET/POST/PUT/DELETE）を定義して具体的な処理を書きます。  

putとdeleteでは処理後に`make_response()`と`jsonify()`を使って`{"message": "Update/Delete Succeeded"}`のレスポンスを返すようにしています。

CORSを利用すると、flaskのサーバーを立てた上でvue側のサーバー「localhost:8080」からもapp.pyにアクセスできるようになります。  
フロントエンド側のファイルを修正するごとに「npm run build」するのが面倒でないならCORSは必要ありません。


## model.py

```py

from flask import request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_marshmallow import Marshmallow
from marshmallow_sqlalchemy import ModelSchema

db = SQLAlchemy()

def init_db(app):
    app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///model/book.db"
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
    db.init_app(app)

class Book(db.Model):
    __tablename__ = 'book'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    title = db.Column(db.String)
    author = db.Column(db.String)
    publisher = db.Column(db.String)

ma = Marshmallow()
class BookSchema(ma.ModelSchema):
    class Meta:
        model = Book

class Bookmodel:
    def all_list(self):
        result = Book.query.all()
        bookschema = BookSchema(many=True)
        res = bookschema.dump(result)
        return jsonify(res)

    def update(self):
        json = request.get_json()
        id = json['id']
        title = json['title']
        author = json['author']
        publisher = json['publisher']
        t = Book.query.get(id)
        t.id=id
        t.title=title
        t.author=author
        t.publisher=publisher
        db.session.commit()

    def delete(self):
        json = request.get_json()
        id = json['id']
        result = Book.query.get(id)
        db.session.delete(result)
        db.session.commit()

    def add(self):
        json = request.get_json()
        title = json['title']
        author = json['author']
        publisher = json['publisher']
        t = Book(title=title,author=author,publisher=publisher)
        db.session.add(t)
        db.session.commit()
        result = Book.query.order_by(Book.id.desc()).first()　#今登録したデータのidを取得
        id = str(result.id)
        return id

```

CRUDの処理はmodel.pyに記述。  
チュートリアル的なものなのでデータベースのカラムはid、タイトル、著者、出版社だけです。  
SQLAlchemyで取得したデータはjsonifyを使ってもjsonに変換できないのでFlask-Marshmallowを介してjsonに変換しています。  
vueから送られたデータはjson形式なので`request.get_json()`で受け取ります。  

これでバックエンド側のAPIセッティングは終了したので次回はフロントエンドを作成していきます。

ソースコードは[こちら](https://github.com/Squigly77/flask_vue_sample)から。

## 参考

[Flask-RESTful公式](https://flask-restful.readthedocs.io/en/latest/index.html)  
[Flask-Marshmallow公式](https://flask-marshmallow.readthedocs.io/en/latest/)  
[Python SQLAlchemyとmarshmallowを使ってjson出力できるAPIを実装する](https://qiita.com/voygerrr/items/4c78d156fc91111798d5)
