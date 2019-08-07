---
title: Flask-SQLAlchemyとFlask-WTF その2
date: 2019-07-17
categories: [Python]
tags: [python,flask]
slug: flasksqlalchemy_wtfoms2
adsenseTop: true
adsenseBottom: true
---

[前回](https://www.ravness.com/2019/07/flasksqlalchemy_wtfoms1/)のSQLAlchemyの基本を抑えたところで、Flask-WTFを使って
フォームからSQLAlchemyを動かしてみます。

## Flask-WTF
---

[Flask-WTF](https://flask-wtf.readthedocs.io/en/stable/)はValidationやCSRF対策を施したフォームを作成するためのFlask拡張。  
Flask-WTFを使ってデータの挿入と削除機能を実装してみる。

まずはインストール

```
pip install Flask-WTF
```

前回のtest.pyに

```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, HiddenField
from wtforms.validators import DataRequired,Length
```

と app = Flask(__name__)以下にCSRFトークン生成のための秘密鍵

```python
app.config['SECRET_KEY'] = os.urandom(32)
```

を追加。  
次にフォーム用のクラスを作る。

```python
class Delete(FlaskForm):
    id = HiddenField('hidden')
    submit = SubmitField('delete')

class Insert(FlaskForm):
    username = StringField('UserName', validators=[DataRequired()
                ,Length(max=10,message="名前は10文字以下で")])
    job = StringField('Job', validators=[DataRequired()])
    submit = SubmitField('追加')
```

StringFieldはテキストフィールドを表し、HiddenFieldは隠しフィールド。 *validators=[DataRequired()]* はデータ入力必須項目。Length(max=10)は最大文字数１０文字。 各フィールドの最初の引数がフォームをレンダリングする際のラベルの設定。  
各種設定はFlask-WTFではなく本家[WTForms](https://wtforms.readthedocs.io/en/stable/)のドキュメントをチェックしてください。

## テンプレートにフォームを生成
---

test.pyにルーティングの設定をしてhtmlで表示してみる。

```python
from flask import Flask, request, redirect, url_for, render_template
from flask_sqlalchemy import SQLAlchemy
import os
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, HiddenField
from wtforms.validators import DataRequired,Length

app = Flask(__name__)

app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///test.db"

# CSRFトークン生成のための秘密鍵
app.config['SECRET_KEY'] = os.urandom(32)

db = SQLAlchemy(app)


class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(20), unique=True, nullable=False)
    job = db.Column(db.String(30), nullable=False)

    def __repr__(self):
        return '<User %r,%r,%r>' % (self.id,self.username,self.job)

class Delete(FlaskForm):
    id = HiddenField('hidden')
    submit = SubmitField('delete')

class Insert(FlaskForm):
    username = StringField('UserName', validators=[DataRequired()
                    ,Length(max=10,message="名前は10文字以下で")])
    job = StringField('Job', validators=[DataRequired()])
    submit = SubmitField('追加')

@app.route('/')
def index():
    form_d = Delete()
    form_i = Insert()
    result = User.query.all()
    return render_template('test.html',result=result,form_d=form_d,form_i=form_i)

if __name__ == "__main__":
    app.run(debug=True)
```

インスタンスを生成して変数form_d,form_iに代入。User.query.all()でテーブルuserの全リストを取得。

#### test.html

```html
<table border="1">
    <thead>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Job</th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        {% for row in result %}
        <tr>
            <td>{{row.id}}</td>
            <td>{{row.username}}</td>
            <td>{{row.job}}</td>
            <td>
                <form method="POST" action="{{url_for('index')}}">
                    {{ form_d.csrf_token }}
                    {{form_d.id(value=row.id)}}
                    {{form_d.submit}}
                </form>
            </td>
        </tr>
        {% endfor %}
    </tbody>
</table>

<p>
    <form method="POST" action="{{url_for('index')}}">
        {{form_i.csrf_token}}
      {{form_i.username.label}}:{{form_i.username(size=20)}} {{form_i.job.label}}:{{form_i.job}} {{form_i.submit}}
    </form>
</p>
```

{{ 変数.csrf_token }}がCSRFトークンの隠しフィールドを生成。  
その他のフィールドも`変数.～～`で生成されるので楽ちん。

とりあえずなんの装飾も施してないページができました。

![form1](../../../images/wtf1.png)<br>

## レコード削除とレコード追加機能を実装
---

```python

@app.route('/', methods=['GET', 'POST'])
def index():
    form_d = Delete()
    form_i = Insert()
    # insert処理
    if form_i.validate_on_submit():
        t = User(username=form_i.username.data, job=form_i.job.data)
        db.session.add(t)
        db.session.commit()
        return redirect(url_for('index'))
    # delete処理
    elif form_d.id.data:
        user = User.query.get(form_d.id.data)
        db.session.delete(user)
        db.session.commit()
        return redirect(url_for('index'))
    # それ以外
    result = User.query.all()
    return render_template('test.html',result=result,form_d=form_d,form_i=form_i)

```

`validate_on_submit()`がPOSTリクエストであるかどうか、そしてそれが有効であるかどうかをチェックしているので、`if request.method == 'POST':`は書かなくていいみたい。  
フォームのデータは`オブジェクト.data`で受け取れます。

10文字以上のnameで登録するとエラーメッセージが出るようにテンプレートに以下を追加

```html
{% if form_i.username.errors %}
{% for message in form_i.username.errors %}
{{message}}
{% endfor %}
{% endif %}
```

<br>

## 実際に動かしてみる
---

**insert**<br>
![form2](../../../images/wtf2.gif)<br><br>
**空白でボタンを押すと**<br>
![form2-2](../../../images/wtf2-2.gif)<br><br>
**delete**<br><br>
![form3](../../../images/wtf3.gif)<br><br>
**10文字以上の名前を登録すると**<br>
![form4](../../../images/wtf4.gif)<br><br>

## 更新
---

```python
class Update(FlaskForm):
    id = HiddenField('hidden')
    username = StringField('username', validators=[
                           DataRequired(), Length(max=10, message="名前は10文字以下で")])
    job = StringField('job', validators=[DataRequired()])
    submit = SubmitField('update')

@app.route('/', methods=['GET', 'POST'])
def index():
    form = Update()
    if form.validate_on_submit():
        r = User.query.get(form.id.data)
        r.username = form.username.data
        r.job = form.job.data
        db.session.commit()
        return redirect(url_for('index'))
    result = User.query.all()
    return render_template('test.html', result=result, form=form)
```

```html
<table border="1">
    <thead>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Job</th>
            <th></th>
        </tr>
    </thead>
    <tbody>

        {% for row in result %}
        <form method="POST" action="{{url_for('index')}}">
            {{form.csrf_token}}
            <tr>
                <td>{{row.id}}</td>
                <td>{{form.username(value=row.username)}}</td>
                <td>{{form.job(value=row.job)}}</td>
                <td>
                    {{form.id(value=row.id)}}
                    {{form.submit}}
                </td>
            </tr>
        </form>
        {% endfor %}

    </tbody>
</table>
```

ゴチャゴチャしてきたので更新機能だけのページに変更。

![form5](../../../images/wtf5.gif)<br><br>
見た目変化ないけど更新成功しています。