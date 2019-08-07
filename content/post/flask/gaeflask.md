---
title: Google App EngineでFlaskを使う
date: 2018-05-17
categories: [GCP]
tags: [Python,gae]
slug: gaeflask
adsenseTop: true
adsenseBottom: true
---

Google App EngineでFlaskを使う時のチュートリアル。  
環境はスタンダード、Python2系。

### Google App Engineでサードパーティ製ライブラリを使う

GAEでサードパーティ製ライブラリを使うにはルートディレクトリにlibディレクトリを作ってそこにライブラリをインストールします。  
最初にvirtualenvで仮想環境化します。  
Flaskをインストールする場合は
```

mkdir lib
pip install -t lib/ Flask
```

これでlibディレクトリにFlaskがインストールされました。  

他のライブラリも同様に入れることができます。requirements.txtを作成して一気に入れる場合は、

```python
pip install -t lib -r requirements.txt
```
でライブラリをインストールできます。


### appengine_config.py

appengine_config.pyというファイルをapp.yamlがある場所に作成して、以下の記述をする
```python

# appengine_config.py
# coding: utf-8
from google.appengine.ext import vendor
# Add any libraries install in the "lib" folder.
vendor.add('lib')
```

プロジェクトのディレクトリ構成はこんな感じになる

```
myproject/
  lib/
  app.yaml
  appengine_config.py
  main.py
```

[「組み込みサードパーティ ライブラリ」](https://cloud.google.com/appengine/docs/standard/python/tools/built-in-libraries-27?hl=ja)にあるライブラリははapp.yamlに記述するだけでいい

```yaml
libraries:
- name: jinja2
  version: 2.6
```

でも、ローカルサーバーで確認する場合は結局libディレクトリ内にインストールする必要があると思うのであまり意味がないような

### app.yaml

app.yamlにstaticファイルやテンプレートファイルの場所を記述します

```yaml

runtime: python27
api_version: 1
threadsafe: yes

handlers:
- url: /static
  static_dir: static

- url: /templates
  static_dir: .*\.html

- url: /.*
  script: main.app
```

### main.pyにFlaskをインポート

```python

#!  /usr/bin/env python
#-*- coding: utf-8-*-
from flask import Flask, request, redirect, url_for, render_template, g

app = Flask(__name__)

@app.route('/')
def test():
    return "Hello World!"

```

### ローカルにサーバーを立ち上げる

`dev_appserver.py .`でローカルにサーバーを立ち上げて、`localhost:8080`にブラウザからアクセスすると"Hello World!"が表示されると思います。

### デプロイ

`gcloud app deploy app.yaml --project プロジェクト名`でデプロイ。<br>
`https://プロジェクト名.appspot.com/`にアクセスして"Hello World!"が表示されていることを確認。

参考サイト<br>
[Getting Started with Flask on App Engine Standard Environment](https://cloud.google.com/appengine/docs/standard/python/getting-started/python-standard-env)<br>
サードパーティライブラリの使い方:[Using third-party libraries](https://cloud.google.com/appengine/docs/standard/python/tools/using-libraries-python-27?hl=ja)
