---
title: Zappaを使用してFlaskのREST APIをAWS Lambda + API Gatewayにデプロイする
date: 2020-04-01
categories: ["Flask"]
tags: ["flask","netlify"]
slug: flaskzappa
adsenseTop: true
adsenseBottom: true
draft: false
---

Zappaを使ってデプロイするまでの紹介と、sqlite3のデータベースファイルごとデプロイすると、  
NetlifyとかFirebase Hostingでもデータの読み込みができるよって話。  

環境はwindows10、Pythonは3.7を使用しています。  
AWSのアカウントを持っていて、AWS CLIで'aws configure'が終わっている前提で進めます。

## Zappa

[Zappa](https://github.com/Miserlou/Zappa)はAWS Lambda + API Gatewayを使用したサーバーレスなFlaskアプリを簡単に構築してデプロイできるツールです。  
FlaskだけでなくDjango、Bottle、Pyramidでも使えるようです。  

## 仮想環境でインストール

最初にvenvで仮想環境を作ります。

```
py -3.7 -m venv flaskzappa
```

ディレクトリ"flaskzappa"に移動して仮想環境下でFlaskとZappaをインストールします。

```
.\Scripts\activate
pip install flask zappa
```

## flaskアプリ

`app.py`にhello worldの表示とJSONを返すapiを作成します。

```py
from flask import Flask, jsonify

app = Flask(__name__)
app.config["JSON_AS_ASCII"] = False

@app.route('/')
def hello_world():
    return 'Hello, World!'

@app.route('/json')
def return_json():
    return jsonify({"name": "Flask","名前": "フラスク"})

if __name__ == "__main__":
    app.run()
```

localhost:5000でちゃんと表示されることを確認。  
なお、jsonifyはデフォルトだと日本語が文字化けするので、
```py
app.config["JSON_AS_ASCII"] = False
```
で文字化けしないようにします。

## Zappaのセットアップ

Zappaでデプロイするための設定をします。

```
zappa init
```

上のコマンドを入れると、

```
$ zappa init

███████╗ █████╗ ██████╗ ██████╗  █████╗
╚══███╔╝██╔══██╗██╔══██╗██╔══██╗██╔══██╗
  ███╔╝ ███████║██████╔╝██████╔╝███████║
 ███╔╝  ██╔══██║██╔═══╝ ██╔═══╝ ██╔══██║
███████╗██║  ██║██║     ██║     ██║  ██║
╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝     ╚═╝  ╚═╝
Welcome to Zappa!

Zappa is a system for running server-less Python web applications on AWS Lambda and AWS API Gateway.
This `init` command will help you create and configure your new Zappa deployment.
Let's get started!

Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
What do you want to call this environment (default 'dev'):
```

と表示されます。  
最下行でこのアプリの開発環境名を聞かれるので入力。  
何も入力せずenterを押すとデフォルトのdevになります。

```
Your Zappa deployments will need to be uploaded to a private S3 bucket.
If you don't have a bucket yet, we'll create one for you too.
What do you want to call your bucket? (default 'zappa-glrc6wuvz'):
```

アップロードするのにS3 bucketが必要だけどバケット名は？  
ここもデフォルトのままなので何も入力せずenter。

```
It looks like this is a Flask application.
What's the modular path to your app's function?
This will likely be something like 'your_module.app'.
We discovered: app.app
Where is your app's function? (default 'app.app'):
```

アプリのモジュラーパスを聞かれるのけど、
ここもデフォルトのapp.appで良いので何も入力せずenter。

```
You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]: 
```

デプロイに関するオプション。ここもデフォルトのままなので何も入力せずenter。

```json
Okay, here's your zappa_settings.json:

{
    "dev": {
        "app_function": "app.app",
        "aws_region": "ap-northeast-1",
        "profile_name": "default",
        "project_name": "flaskzappa",
        "runtime": "python3.7",
        "s3_bucket": "zappa-glrc6wuvz"
    }
}

Does this look okay? (default 'y') [y/n]:
```

この設定で良いのでenter。  

これで設定が終わってzappa_settings.jsonファイルが生成されました。

## デプロイ

準備が整ったのでデプロイします。コマンドは、
```
zappa deploy
```

```
Calling deploy for stage dev..
Creating flaskzappa-dev-ZappaLambdaExecutionRole IAM Role..
Creating zappa-permissions policy on flaskzappa-dev-ZappaLambdaExecutionRole IAM Role.
Warning! Your project and virtualenv have the same name! You may want to re-create your venv with a new name, or explicitly define a 'project_name', as this may cause errors.
Packaging project as zip.
Uploading flaskzappa-dev-1585651110.zip (11.5MiB)..
100%|█████████████████████████████████████| 12.0M/12.0M [00:10<00:00, 1.17MB/s]
Scheduling..
Scheduled flaskzappa-dev-zappa-keep-warm-handler.keep_warm_callback with expression rate(4 minutes)!
Uploading flaskzappa-dev-template-1585651238.json (1.6KiB)..
100%|█████████████████████████████████████| 1.64k/1.64k [00:00<00:00, 15.0kB/s]
Waiting for stack flaskzappa-dev to create (this can take a bit)..
100%|███████████████████████████████████████████| 4/4 [00:12<00:00,  3.14s/res]
Deploying API Gateway..
Deployment complete!: https://tfg7g8rucg.execute-api.ap-northeast-1.amazonaws.com/dev
```

一番下の`https://tfg7g8rucg.execute-api.ap-northeast-1.amazonaws.com/dev`API GatewayのURLです。

URLが表示されずエラーが出たときは`zappa tail`でログを確認しましょう。  

curlで上のURLを入れてみると……  

![url1](../../../images/GIF2020-03-3120-34-49.gif)

HelloWorldが返ってきました。jsonも無問題。

![url2](../../../images/GIF2020-03-3120-25-15.gif)

2回目以降は`zappa update`で更新。  
`zappa undeploy`でAWS LambdaとAPI Gatewayから全削除します。  

AWS Lambda、API Gateway、S3 bucket各コンソールに行くと関数、API、バケットが作成されているのを確認できます。  
本来ならAWS CLIやコンソールから手作業でやることを勝手にしてくれるのでめっちゃ楽。

## sqliteのデータファイルを一緒にデプロイしたらどうなるのか？

結論から言うと読み取りはできました。  
CRUDのCUD（生成・更新・削除）は無理。  
CRUD全ての機能が必要ならDynamoDBやRDSを使わないとダメそう。

## NetlifyやFirebase Hostingと組み合わせる

データを読み取ることができるのがわかったので、Netlifyと組み合わせて使ってみました。  
1からファイルを作成するのは面倒なので過去記事（[FlaskとVue.js(Vue CLI)の連携](http://localhost:1313/2019/12/flaskvue/)、[バックエンド編](http://localhost:1313/2019/12/flaskvuebackend/)、[フロントエンド編](http://localhost:1313/2019/12/flaskvuefrontend/))のファイルを少し変えてデプロイしてみました。

変更点は
```py
app = Flask(__name__,
                static_folder="../frontend/dist/static",
                template_folder="../frontend/dist")
```
を
```py
app = Flask(__name__)
```
にして、ルーティングの

```py
@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def index(path):
    return render_template('index.html')
```
を
```py
@app.route('/', methods=['GET'])
def index():
    result = book.all_list()
    r = make_response(result)
    return r
```
にして、データの読み取り以外とFlaskRestfulに関する部分は全削除。  

ReactやVueはAPIのURLをAPI GatewayのURLに変更します。  
ビルドしたファイルをNetlifyへデプロイしたのが下の画像

![deploynetlify](../../../images/s_dsBuffer.jpg)

見にくいですがアドレスがflask-zappa.netlify.comになっていますね？  

あまりデータを更新しないサイトならAWS LambdaとAPI Gateway + Netlifyで格安で運用できそう。  
AWS Lambdaが百万リクエストまで無料なのでAPI Gatewayの費用（数円から数十円）だけ？
