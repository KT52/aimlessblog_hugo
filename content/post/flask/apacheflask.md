---
title: "CentOS7・ApacheでFlaskを動かす"
date: 2019-02-25
tags: ["apache","flask"]
slug: "apacheflask"
adsenseTop: true
adsenseBottom: true
---

先月下旬にVPS環境でCentOS7にApache、PHP、MariaDBを入れて動かしてみたLinux初心者が今度はFlaskを入れて動かしてみました。

Google Compute Engine(GCE)の無料枠を使って環境を構築しました。

CentOS 7.6.1810, Apache 2.4.6、Python3.6.6を使用しています。

アプリは/var/www/html/myappに配置します。

## インストール
---

まずは`pip3 install flask`でFlaskをインストール。

次にApacheでFlaskを動かすにはmod_wsgiが必要なのでこれもインストール。
`pip3 install mod_wsgi`

下記のようなエラーが出てpip3 install mod_wsgiできないので

```cmd

Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-k3z23pb6/mod-wsgi/
You are using pip version 9.0.3, however version 18.0 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```

`pip3 install --upgrade pip`でアップグレード。

再びmod_wsgiをインストールしようとすると今度は、

```cmd

FileNotFoundError: [Errno 2] No such file or directory: 'apxs'
```

とエラーが……。

調べてみると`httpd-devel`というのが必要らしいので、
`yum install httpd-devel`

httpd-develをいれてもmod_wsgiがインストールできない場合はエラーメッセージを確認してください。
多分下の2つあたりを入れておけば大丈夫だと思います。

- gcc（GNU Compiler Collectionの略 GCCとは、GNUプロジェクトが開発および配布している、さまざまなプログラミング言語のコンパイラ集）

- python-devel（Python の開発に必要なライブラリとヘッダーファイル）

`yum install gcc python-devel`

これで準備が整ったので
`pip3 install mod_wsgi`でインストール。

### 設定ファイルの追加
---

設定ファイルにVirtualHostを追加する。
`/etc/httpd/conf/httpd.conf` に追記するか、`/etc/httpd/conf.d/にwsgi.conf`ファイルを作成して下記のように記述。  
僕はwsgi.confを新規作成するやり方なので、`vim /etc/httpd/conf.d/wsgi.conf`で

```conf

# wsgi.conf
<VirtualHost *:80>
serverName 11.111.11.111

WSGIDaemonProcess myapp user=apache  group=apache threads=5
WSGIScriptAlias / /var/www/html/myapp/flask.wsgi

<Directory /var/www/html/myapp/>

WSGIProcessGroup myapp
WSGIApplicationGroup %{GLOBAL}
WSGIScriptReloading On

Require all granted

</Directory>
</VirtualHost>

```


ポートは80番を使用。  
serverNameはvps等で与えられたIPアドレスやドメインを記述。myapp、user=xxx、group=xxxは各自書き換えてください。  
`systemctl restart httpd`でApacheを再起動。

## xxx.wsgiの作成
---

/var/www/html/myapp/にflask.wsgiファイルを作成

```python

# coding: utf-8
import sys

sys.path.insert(0, '/var/www/html/myapp/')

from myapp import app as application

```



## myapp.py
---

とりあえずHello, World!を表示させるmyapp.pyを/var/www/html/myapp/に作成

```python

#-*- coding: utf-8-*-
import sys 
#インストールしたモジュールが読み込めないので
sys.path.append('/usr/local/lib64/python3.6/site-packages/')
sys.path.append('/usr/local/lib/python3.6/site-packages/')

from flask import Flask, request, redirect, url_for, render_template, g

app = Flask(__name__)

@app.route('/')
def index():
    return 'Hello, World!'
    
if __name__ == "__main__":
    app.run()
```


アクセスしたらInternal Server Errorがでたので/var/log/httpd/error_logを確認したところ、
```
ImportError: No module named flask
```

まだエラーが出るので再確認

```
ImportError: No module named werkzeug.exceptions No module named werkzeug.exceptions
```

と表示されたので、
myapp.pyにモジュールが置いてある場所

```python
import sys
sys.path.append('/usr/local/lib64/python3.6/site-packages/')
sys.path.append('/usr/local/lib/python3.6/site-packages/')
```

を追加しました。

`systemctl restart httpd`でApacheを再起動してGCEで与えられたipアドレスにアクセスしてHello, World!が表示されました！


## 参考サイト
---

[mod_wsgi (Apache)](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)  
[AWS(EC2) で Python3 + Flask を Apache で動かす（mod_wsgi?）](https://qiita.com/yabekenzo/items/346142e29db36b77e42d)
