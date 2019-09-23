---
title: "Nginx+uWSGIでFlaskを使用する"
date: 2019-09-23
categories: "Flask"
tags: [flask,nginx]
slug: flasknginx
adsenseTop: true
adsenseBottom: true
---

CentOS7でNginx+uWSGI+Flaskを使用したwebアプリのセットアップをしてみます。

## pythonインストール

```
sudo yum install -y python36 python36-libs python36-devel python36-pip
```

## FlaskとuWSGIをインストール

仮想環境下でflaskとuwsgiをpipでインストール。

```
python3.6 -m venv flask_app
cd flask_app
. bin/activate
pip3 install flask uwsgi
```

## Hello Worldを表示するまで

ファイルの構成はこんな感じになります

```
flask_app
    ❘-test.py
    ❘-uwsgi.ini
    ❘-venv
```

#### test.py

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
	return "Hello World"

if __name__ == "__main__":
	app.run()
```

#### nginxの設定

/etc/nginx/conf.d/flask.conf

```
server {
    listen 80;
    server_name example.com or IP;
    
    location / {
        include uwsgi_params;
        uwsgi_pass unix:///tmp/uwsgi.sock;
    }
}
```

UNIXドメインソケット/tmp/uwsgi.sockを介してnginxとuwsgiを接続します。

#### uWSGIの設定

以下のコマンドでuwsgiを起動
```
uwsgi --socket /tmp/uwsgi.sock --module test --callable app --chmod-socket=666
```

nginxとuwsgiのUNIXドメインソケットのpathは揃えておく。
これでexample.com(またはIPアドレス）にアクセスするとHello Worldが表示されます。  
でも毎回上のコマンドを打つのは面倒なので、`uwsgi.ini`を作ります。

```ini
#uwsgi.ini
[uwsgi]

uid = nginx #ユーザー名
gid = nginx　#グループ名

module = test  #WSGIモジュールをロードする　ここではtest.py
socket = /tmp/%n.sock　#デフォルトのプロトコルを使用して、指定されたUNIX / TCPソケットにバインドします
chmod-socket = 666　#パーミッション
callable = app　#デフォルトのWSGI呼び出し可能名を設定する

```

%nはuwsgiのマジック変数でファイル名の拡張子を除いた名前（uwsgi）に置き換えられます。  
`callable`はflaskの`app = Flask(__name__)`のappです。  
uwsgi.iniを作成したら次のコマンドで起動します。
```
uwsgi --ini uwsgi.ini
```

停止はctrl+c。

## デーモン化

デーモン化したuwsgiの起動、停止、リロードをするにはpidファイルが必要なので、`uwsgi.ini`に以下を追加

```ini
daemonize = /var/log/uwsgi/%n.log
pidfile = /tmp/flask_app.pid
vacuum = true
```
`daemonize = /var/log/uwsgi/%n.log`でデーモン化したときのログファイル置き場を指定。  
`vacuum = true` は起動した時、異常終了した時に残っているpidファイルやsocketをクリアする。

/var/log/にuwsgiディレクトリを作成
```
sudo mkdir /var/log/uwsgi
sudo chown -R xxx:xxx uwsgi
```

起動、停止、リロード
```
uwsgi --ini uwsgi.ini  # 起動
uwsgi --stop /tmp/flask_app.pid  # 停止
uwsgi --reload /tmp/flask_app.pid  # リロード
```

## The uWSGI Emperor

Emperorモードを使うと一つのサーバーで複数のアプリを利用する時に一括で管理できて便利。  
[The uWSGI Emperor – multi-app deployment](https://uwsgi-docs.readthedocs.io/en/latest/Emperor.html)

使い方は、/etc/uwsgi/vassalsディレクトリを作成してconfiguration file(uwsgi.ini)のシンボリックリンクを作成します。  
複数のアプリがある場合はそれぞれの.iniファイルのシンボリックリンクを作成。  


```
sudo mkdir /etc/uwsgi/vassals
sudo ln -s /var/www/html/flask_app/uwsgi.ini /etc/uwsgi/vassals
```

`uwsgi.ini`に以下を追加

```
chdir = /var/www/html/flask_app  #appをロードする前に指定したディレクトリに移動
```

このオプションがないと多分起動しない。

手動で起動する場合は以下のコマンドを打って起動

```
uwsgi --emperor /etc/uwsgi/vassals --logto /var/log/uwsgi/emperor.log
```

常に起動しておくためにサービスに登録します。  
/etc/systemd/system/にuwsgi.serviceを作成。

```
sudo mkdir /etc/systemd/system/uwsgi.service
```

```
#uwsgi.service

[Unit]
Description=uWSGI
After=syslog.target

[Service]
ExecStart=/var/www/html/flask_app/bin/uwsgi --emperor /etc/uwsgi/vassals --logto /var/log/uwsgi/emperor.log
Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

ExecStart=以下のパスはそれぞれの環境で違う場合があると思うので任意に変更してください。

起動、自動起動、リスタート、停止

```
sudo systemctl start uwsgi
sudo systemctl enable uwsgi
sudo systemctl restart uwsgi
sudo systemctl stop uwsgi
```

uwsgi.serviceに間違いがないのにサービスがスタートできない場合は  
systemdにユニットファイルを追加・更新したことを通知するコマンド

```
sudo systemctl daemon-reload
```

を打つとサービスが起動するかも。


## .iniファイルの例

```ini
[uwsgi]

uid = nginx
gid = nginx

chdir = /var/www/html/flask_app
module = run
socket = /tmp/%n.sock
chmod-socket = 666
callable = app


master = true
vacuum = true
processes = 2
threads = 1
pidfile = /tmp/flask_app.pid
thunder-lock = true　 #falseになっていると複数のuWSGIのプロセスで捌くリクエストに偏りが出てしまう。

#location of log files
logto = /var/log/uwsgi/%n.log

#daemon
#daemonize = /var/log/uwsgi/%n.log
log-reopen = true
log-maxsize = 8000000
```

エンペラーモードを使用。  
各オプションの詳細は[公式サイト](https://uwsgi-docs.readthedocs.io/en/latest/Options.html)を参照してください。
