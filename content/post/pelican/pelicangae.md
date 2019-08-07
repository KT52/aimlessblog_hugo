---
title: Google App EngineでPelicanを使ってブログをホスティングする方法
date: 2018-04-02
categories:	Pelican
tags: [Python,GAE,Pelican]
slug: pelicangae
adsenseTop: true
adsenseBottom: true
---

## PelicanをGAEで使う
---

Google App Engine（以下GAE）は静的サイトもホスティングできるので、Pythonの静的サイトジェネレーターPelicanを使ったブログサイトを作ってみた。<br>
なお、GAEの利用法（Google Cloud SDKとか)は省きます。Pelicanの使用法は[前に書いた記事](https://www.ravness.com/2018/03/pelicangithub/)を参照してください。

## GAEの新しいプロジェクトを作成する
---

ここではプロジェクト名を"myblog"とします。

## プロジェクトのディレクトリにひな型を作成
---

プロジェクト"myblog"のローカルディレクトリにpelicanの`pelican-quickstart`コマンドでひな型を作成。ディレクトリはこんな構造になっている。  

```
myblog/
    content/
    output/
    app.yaml
    pelicanconf.py
```  

  他にもファイルがプロジェクトのディレクトリにありますが、ブログ作成に最低限必要なのはこれだけ。<br>

## app.yamlの設定
---

```yaml
runtime: python27
api_version: 1
threadsafe: yes

handlers:
- url: /
  static_files: output/index.html
  upload: output/index.html

- url: /
  static_dir: output

```

## 記事を書いてジェネレイト
---

contentディレクトリに書いた記事を保存したら、`pelican content`で記事をジェネレイト。

## ローカルサーバーで確認
---

`dev_appserver.py .`でサーバーを立ち上げたら、ブラウザでlocalhost:8080にアクセス。ブログが表示されたら成功。<br>

## デプロイ
---

デプロイするのはoutputディレクトリだけでいいので、app.yamlでアップロードしないディレクトリを記述
```
skip_files:
- themes/
- content/
- plugins/
```

## サイトにアクセス
---

デプロイしたら`https://プロジェクト名.appspot.comにアクセス`  <br>

## 補足
---

`pelicanconf.py`にURLルールをセッティングをしている場合は`app.yaml`にindex.htmlの場所とアップロードを追記する必要があります。
```

ARTICLE_URL = '{date:%Y}/{date:%m}/{slug}/index.html'
ARTICLE_SAVE_AS = '{date:%Y}/{date:%m}/{slug}/index.html'
YEAR_ARCHIVE_SAVE_AS = '{date:%Y}/index.html'
MONTH_ARCHIVE_SAVE_AS = '{date:%Y}/{date:%m}/index.html'
```
僕の場合はurlをドメイン名/年/月/slug/index.htmlにして、年と月のアーカイブも表示する設定にしています。  
なので`app.yaml`にはこう記述しました。
```yaml
- url: /
  static_files: output/index.html
  upload: output/index.html

- url: /
  static_files: output/(.*)/(.*)/(.*)/index.html
  upload: output/(.*)/(.*)/(.*)/index.html

#　年アーカイブの場所
- url: /
  static_files: output/(.*)/index.html 
  upload: output/(.*)/index.html

#　月アーカイブの場所
- url: /
  static_files: output/(.*)/(.*)/index.html
  upload: output/(.*)/(.*)/index.html

- url: /
  static_dir: output
```

## GAEを無料で運用するための設定
---
<br>
GAEはスタンダード環境での無料枠が大きいので月間数万PVくらいまでなら無料枠で運営できる…はず。  
しかし何も設定しないとアクセス過多でインスタンスが複数立ち上がって無料で運用することができなくなる可能性があるので`app.yaml`でこれを制御します。  

```
instance_class: F1
automatic_scaling:
  min_idle_instances: automatic
  max_idle_instances: 1
  min_pending_latency: 3000ms
  max_pending_latency: automatic
```
インスタンスは一番性能が低いが28インスタンス時間が無料のf1-micro、`max_idle_instances: 1`でidle状態にあるインスタンスの最大値を1にして複数インスタンスが起動しないようにする。<br><br>

## 参考サイト
---
[Google App Engine で静的ウェブサイトをホストする](https://cloud.google.com/appengine/docs/standard/php/getting-started/hosting-a-static-website?hl=ja)<br>
[Pelican](http://docs.getpelican.com/)<br>
[Pelican Hosting on AppEngine](http://www.craigjperry.com/pelican-hosting-on-appengine.html)<br>
[Google App Engineを無料で運用する方法（2018年版）](http://koni.hateblo.jp/entry/2016/01/06/130613)

