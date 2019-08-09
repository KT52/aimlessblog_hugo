---
title: GCPのデータストアにcsvのデータを一括で登録する2
date: 2018-08-15
categories: [GCP]
tags: [Python,GAE]
slug: datastorecsv2
adsenseTop: true
adsenseBottom: true
---

今回はGoogle Cloud Platform (GCP)のデータストアにbulkloaderを使わずにGoogle App Engine（GAE)経由でcsvファイルを一括でアップロードします。
言語はPython。フレームワークはFlaskを使用。

bulkloaderを使う方法はこちら→[GCPのデータストアにcsvのデータを一括で登録する](https://www.ravness.com/2018/08/datastorecsv)

### app.yaml
---

app.yamlに以下を記述

```yaml

- url: / #今回はcsvファイルをルートディレクトリに置く
  static_files: up.csv #データストアに登録するcsvファイル
  upload: up.csv #データストアに登録するcsvファイル
  mime_type: text/csv #必要ないかも
  application_readable: true

```

`application_readable: true`とすることで静的ファイルを直接読み込むことが出来るので、ファイルを読み込むページを直接作っちゃおうというのが今回のやり方。

### アプリケーションの作成
---

Kind名はtest プロパティはalpha,bravo,charlie,deltaの４つでいずれもstring

```python
#!  /usr/bin/env python
#-*- coding: utf-8-*- 
from flask import Flask, request, redirect, url_for, render_template, g
from google.appengine.ext import ndb
import csv

app = Flask(__name__)

class test(ndb.Model):
	alpha = ndb.StringProperty()
	bravo = ndb.StringProperty()
	charlie = ndb.StringProperty()
	delta = ndb.StringProperty()
	
@app.route('/')
def index():
	return render_template('index.html')


@app.route('/upload')
def csvup():
	with open('./up.csv') as f:
		readcsv = csv.reader(f)
		for row in readcsv:
			alpha, bravo, charlie, delta = row
			c = test(alpha=alpha, bravo=bravo, charlie=charlie,delta=delta)
			c.put()
	return redirect(url_for('index'))

```

これでデータストアに登録できます。

しかし、このやり方だと誤って/uploadにアクセスしたりすると再度データストアに登録してしまうので注意。  
`application_readable: true`を使うと、csvファイルを読み込んでtableに出力するとか、テキストファイルからランダムに一つ読み込んでtwitterbotを運用したり（実際にやってる）他のことに利用できるかもしれません。
