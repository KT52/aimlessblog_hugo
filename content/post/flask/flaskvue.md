---
title: FlaskとVue.js(Vue CLI)の連携
date: 2019-12-06
categories: ["Flask"]
tags: ["flask","vue"]
slug: flaskvue
adsenseTop: true
adsenseBottom: true
---

バックエンドにFlask、フロントエンドにVue.jsを使ったWebアプリを開発する際の備忘録。  
PythonやVue.js(Vue CLI)の導入については省略します。vueに関しては全くの初心者なので、vue CLIでプロジェクト作成時にRouterだけ追加しました。

## ディレクトリ

ディレクトリ構成はappディレクトリ内をフロントエンドとバックエンドに分けてアプリを作成していきます。  

```
app
 |--frontend(vue)
 |--backend(flask)
```

## Flaskのセッティング

```python
#run.py

from flask import Flask, render_template

app = Flask(__name__,
                static_folder="../frontend/dist/static",
                template_folder="../frontend/dist")

@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def index(path):
    return render_template('index.html')

if __name__ == "__main__":
    app.run()

```

flask単体でアプリを開発するときはflaskのstatic、templatesディレクトリを使用するのがデフォルト設定ですが、
今回はフロントエンドにvueを使用するので、vueでbuildしたあとに作成されるdistディレクトリを指定します。  
ルートも上のように書くとdist内のindex.htmlを参照するようになります。

## Vueのセッティング

`vue.config.js`をfrontendディレクトリに作成して以下を記入。

```js
module.exports = {
  assetsDir: 'static',
};
```

とりあえず今はvueのデフォルトのページが表示されればいいので、frontendディレクトリから
```
npm run build
```

でビルドします。ビルドが終わるとdistディレクトリが作成されます。

## Flaskでサーバーを起動

backendディレクトリに移動して`run.py`でサーバーを起動。  
ブラウザで`localhost:5000`にアクセス。

![flaskvue](../../../images/flaskvue.jpg)

問題なく表示されました。  

次回はsqliteを使用してRest APIを作成したいと思います。
