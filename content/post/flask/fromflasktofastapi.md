---
title: FlaskアプリをFastAPIに置き換える
date: 2020-09-08
categories: ["Flask"]
tags: ["fastapi","flask"]
slug: fromflasktofastapi
adsenseTop: true
adsenseBottom: true
draft: false
---

過去に公開した[FlaskとVueでCRUD操作（バックエンド編）](https://www.ravness.com/2019/12/flaskvuebackend/)をFastAPIで書いてみました。  

FastAPIのドキュメントを読んだり（ほぼチュートリアルしか読んでない）、実際に書いてみた感想としては、

- Flaskからの移行は容易
- importを除けば元のFlaskで書いていたファイルの一部を書き換えたり削除したりするだけで済んだ（特にSQLAlchemy関連）
- Flaskよりかは外部ライブラリに頼らないで色々できそう
- 今回のような極小アプリではFlaskとFastAPIの処理や体感速度に違いはない
- モダンなフレームワークで名前の通りAPIとして使用するならFastAPI
- フロントエンドとバックエンドを分けないならFlaskや他のフレームワーク
- ドキュメントの自動生成がすごい（語彙力）

## FastAPIをインストール

仮想環境化で`pip install fastapi uvicorn`をインストール。  
uvicornは非同期処理が可能なasgiサーバーを立ち上げるためのライブラリです。  
`pip install sqlalchemy`でSQLAlchemyもインストールします。


## Hello World

```py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}

```

ほとんどFlaskと変わりません。`uvicorn index:app --reload`でサーバーを立ち上げて`http://localhost:8000`にアクセスすると{"message": "Hello World"}のJSONが返されます。  

`http://localhost:8000/docs`にアクセスすると

![fastapi1](../../../images/fastapi1.jpg)

のようなSwagger UI が自動生成されます。  
青い部分をクリックすると展開。

![fastapi2](../../../images/fastapi2.jpg)

`http://localhost:8000/redocs`でReDocのドキュメントを見ることができます。

## パスパラメータ

```py
@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

パスパラメータやクエリパラメータ、bodyは関数の引数で定義します。  
`http://localhost:8000/items/foo`にアクセスすると、
`{"item_id": foo}`と返されます。でこれを`http://localhost:8000/docs`で見ると、

![fastapi3](../../../images/fastapi3.jpg)

こんなドキュメントが出来ます。try it outをクリックしてitem_id欄にbarと打ってexcuteボタンを押すと、

<a href="../../../images/fastapi4.jpg"><img src="../../../images/fastapi4.jpg"></a>

response bodyに{"item_id": "bar"}と返されています。  

## 構成

FlaskからFastAPIへこんな感じの構成で置き換え

```
fastapi_app
|--database
|     |--__init.py__
|     |--crud.py.py
|     |--databes.py
|     |--model.py
|--index.py
|--schemas.py
|--book.db
```

## SQLAlchemy関連のファイル（database.py,model.py,crud.py）

SQLite3のbook.dbの中身は

|id|title|author|publisher|
|----|----|----|----|
|1|犬|猫|鳥出版|
|2|自転車|ドープ・ランス|フランス社|
|3|yo soi el hombre|Messi|Mundo Deportivo|



以下のファイルは公式の[SQL (Relational) Databases](https://fastapi.tiangolo.com/tutorial/sql-databases/)を参考にしています。

database.pyはsqliteのファイル名以外はコピペ

```py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./book.db"
engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()


def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

model.py

```py
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import relationship
from sqlalchemy.orm import Session
from .database import Base


class Book(Base):
    __tablename__ = 'book'
    id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String)
    author = Column(String)
    publisher = Column(String)
```

crud.py（crudの処理を行うファイル）

```py
from sqlalchemy.orm import Session
from . import model


def all_list(db: Session):
    return db.query(model.Book).all()

def update(db: Session, data):
    id = data.id
    title = data.title
    author = data.author
    publisher = data.publisher
    t = db.query(model.Book).get(id)
    t.id=id
    t.title=title
    t.author=author
    t.publisher=publisher
    db.commit()

def delete(db: Session, id):
    result = db.query(model.Book).get(id)
    db.delete(result)
    db.commit()

def add(db: Session, data):
    title = data.title
    author = data.author
    publisher = data.publisher
    t = model.Book(title=title,author=author,publisher=publisher)
    db.add(t)
    db.commit()
    result = db.query(model.Book).order_by(model.Book.id.desc()).first()
    id = str(result.id)
    return id
```

flaskだとbodyで受け取ったモデルオブジェクトのアクセスは
```py
json = request.get_json()
id = json['id']
title = json['title']
```
としていたところがFastAPIでは`data.title`みたいにアクセスする。[[参照]](https://fastapi.tiangolo.com/tutorial/body/#use-the-model)  
flaskのようなかたちで受け取りたい場合はFastAPIに内包しているStarletteのRequestで受け取ることが出来ます。[[参照]](https://fastapi.tiangolo.com/advanced/using-request-directly/)

## Pydantic models

Pydanticを利用してリクエストとレスポンスのパラメータのバリデーションを行います。  
これは必須ではないのでなくても構いません。  

schemas.py

```py
from pydantic import BaseModel
from typing import Optional

# requestのBodyの型チェック
class Update(BaseModel):
    title: str
    author: str
    publisher: Optional[str] = None
    id: int

# requestのBodyの型チェック
class Add(BaseModel):
    title: str
    author: str
    publisher: Optional[str] = None

# responseの型チェック
class Datasout(BaseModel):
    title: str
    author: str
    publisher: Optional[str] = None
    id: int

    class Config:
        orm_mode = True
```

データベースから取得したデータのresponse_modelはorm_modeをtrueにして、データがdictではなくORMモデルであってもデータを読み取るようにPydanticモデルに指示します。  
"class Config:～"以下を消去してbook.dbのデータを読み込もうとすると、
```
pydantic.error_wrappers.ValidationError: 3 validation errors for Datasout
response -> 0
  value is not a valid dict (type=type_error.dict)
```
dict型じゃないよ、とエラーが出ます。

## index.py

メインファイル

```py
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from database import crud, model　# crud.py,model.pyのインポート
from database.database import SessionLocal, engine, get_db # database.pyからインポート
from fastapi.middleware.cors import CORSMiddleware　# CORS
from schemas import Update, Add, Datasout # schemas.pyからインポート
from typing import List

app = FastAPI()

origins = [
    "http://localhost",
    "http://localhost:8080",
    "http://localhost:8000",
    "http://localhost:3000",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/", response_model=List[Datasout])
async def read_list(db: Session = Depends(get_db)):
        result = crud.all_list(db)
        return result

@app.put("/update/")
async def item_update(data: Update, db: Session = Depends(get_db)):
    crud.update(db, data=data)
    return {"update":"成功"}

@app.delete("/delete/{id}")
async def item_delete(id: int, db: Session = Depends(get_db)):
    crud.delete(db, id=id)
    return {"Delete": "成功"}

@app.post("/add/")
async def post(data: Add, db: Session = Depends(get_db)):
    id = crud.add(db, data=data)
    return {"Add": "成功", "ID":id}
```

Flaskでは`@app.route("/", methods=["POST"])`のようにしていたのがFastAPIでは`@app.post("/")`のように書きます。  

レスポンスは`response_model=～`でバリデーションを行います。  
リクエストのバリデーションは`xxx: Update` `xxx: Add`のようにしてバリデーションを行います。  

## ドキュメントで確認

localhost:8000/docsにアクセス。

![fastapi5](../../../images/fastapi5.jpg)

docs上でアップデートしてみる。  
"Try it out"をクリックするとテキストを打ち込めるようになるので  
"string"を消してid:1の犬、猫。鳥出版を馬、羊、猿出版にしてexcute。

![fastapi6](../../../images/fastapi6.jpg)

成功のレスポンスが返ってきたのでhttp://localhost:8000/で確認。

![fastapi7](../../../images/fastapi7.jpg)

![fastapi8](../../../images/fastapi8.jpg)

更新されてます。  
データの削除や追加も問題なく行われていましたがここでは省略します。

## おわりに

FastAPIのベーシックな部分はFlask2.0と言ってもいいくらい大きな違いがないので、Flaskからの移行は簡単でした。  
ドキュメント（Swagger UI）が便利だったのでこれだけでFastAPIを選ぶ価値ありです。