---
title: Pythonの静的サイトジェネレーターPelicanでGitHub Pagesにブログを開設する
date: 2018-03-30 12:00
category: Pelican
tags: [Python,GitHub,Pelican]
slug: pelicangithub
modified: 2019-08-04
adsenseTop: true
adsenseBottom: true
---

今更ながら静的サイトジェネレーターというものの存在を知って、GitHub Pagesに[Pelican](http://docs.getpelican.com/en/stable/)を使ってブログを実際に作ってみたので、その手順を備忘録的に書いてみる。環境はWindows10、Python3.6.4。<br><br>

## GitHubに登録
---

Gitが使えるという前提で進めていきます。  
まずは[GitHub](https://github.com/)でアカウントを作成。<br><br>

## レポジトリを作成
---

`Repository name`に`ユーザーネーム.github.io`と打ち込んで`Create Repository`をクリックして新規作成。  
`Initialize this repository with a README`にチェックを入れるとREADME.mdが作られる。 <br><br> 

##  ローカルに環境を構築
---

ブログを作成するディレクトリにvenvなどで仮想環境を作り、pipでPelicanをインストール。  
Markdown形式で書く場合はMarkdownもインストール。  
`pip install pelican Markdown`  
<br>

##  pelican-quickstartでブログのひな型を作成
---

`pelican-quickstart`とコマンドを打つと、以下の質問が順番に表示されてブログのひな型を簡単に作成できる。  
```
This script will help you create a new Pelican-based website.
Please answer the following questions so this script can generate the files needed by Pelican.
>Where do you want to create your new web site? [.]
>What will be the title of this web site? Squigly77 Blog,
> Who will be the author of this web site? Squigly77
> What will be the default language of this web site? [Japanese]
> Do you want to specify a URL prefix? e.g., http://example, com (Y/n) y
> What is your URL prefix? (see above example; no trailing slash) https://squigly77.github.io
> Do you want to enable article pagination? (Y/n) y
> How many articles per page do you want? [10],
>What is your time zone? [Europe/Paris] Asia/Tokyo
> Do you want to generate a Fabfile/Makefile to automate generation and publishing? (Y/n) y
> Do you want an auto-reload & simpleHTTP Script to assist with theme and sited evelopment? (Y/n) y
> Do you want to upload your website using FTP? (Y/N) n
> Do you want to upload your website using SSH? (Y/N) n
> Do you want to upload your website using Dropbox? (Y/N) n
>Do you want to upload your website using S3? (y/N) n
> Do you want to upload your website using Rackspace Cloud Files? (Y/N) n
> Do you want to upload your website using GitHub Pages? (Y/N) y
> Is this your personal page_(username.github.io)? (Y/N) y Done. Your new project is available at D:\usr¥github-blog\squigly77.github.io
```
</font>

雛形の作成が終わると、ディレクトリ内はこんな構成になっていると思います。
```
username.github.io/
|---content
|---output
|---develop_server.sh
|---fabfile.py
|---Makefile
|---pelicanconf.py
|---publishconf.py
```
</font>
これでディレクトリ"username.github.io"内でpelicanコマンドが使えるようになります。<br><br>

## ブログの記事を書く
---

contentディレクトリ内にMarkdown形式で記事を書く。  
最初にメタデータを入れる。TitleとDateだけでもOK   

Title: 記事のタイトル  
Date: 日付  
Category:　カテゴリー  
Tags: タグ  
Author: 著者  

ファイル名.mdで保存する。   <br><br>

### 記事をジェネレイト

pelicanconf.pyがあるディレクトリで`pelican`コマンドでcontent内に作った記事をoutputディレクトリにジェネレイト。 
outputディレクトリにhtmlファイルが生成されます。  <br><br>

## ローカルで確認
---

outputディレクトリに移動`cd output`して、`python -m pelican.server`でローカルサーバーを立ち上げる。  
追記：現在のバージョンでは`python -m http.server`と打つのが正しい。  
ブラウザで`localhost:8000`にアクセスすると、作成された記事が出力されます。<br><br>

## staticファイル
---

staticファイルはcontentディレクトリ内にstaticディレクトリとかimagesディレクトリを作ってそこに入れる。  
pelicanconf.pyに`STATIC_PATHS = ['images']`のように記述。  
ディレクトリが複数の場合はコンマ区切りで記述  <br><br>

## テーマを導入
---

デフォルトのテーマを変えたい場合、自作するか、[Pelican Themes](http://pelicanthemes.com/)からお気に入りのものをダウンロードして適用することも出来ます。
themesディレクトリを作ってそこにダウンロードしたテーマを入れたら、`pelican-themes --i [テーマの入ったディレクトリ]`で導入することが出来ます。  
僕は"pelican-bootstrap3"というテーマを選んだので、`pelican-themes --i themes/pelican-bootstrap3`で入れました。  
ちゃんと入ったかを確認するには`pelican-themes -l`  
入ってたら`pelicanconf.py`にTHEME = 'テーマ名'を追加する。<br><br>

## GitHub PagesにPush
---

pipでghp-importを入れておくと、GitHub pagesへの公開が楽に行える。  
入れたらGitHubにアップロード  
`ghp-import output`  
`git commit -m "first post"`  
`git push -f origin gh-pages:master`

少し時間が経ったら、ブラウザで https://ユーザーネーム.github.io にアクセス。
<br><br>
実際に作ったのが[こちら](https://squigly77.github.io/)。テーマはFlexを使用。

## 参考サイト
[Pelican](http://docs.getpelican.com/en/stable/)  
[Pythonの静的サイトジェネレータ"Pelican"でお手軽にブログをはじめる手順](https://qiita.com/ogrew/items/ecef0a4700d5bd4d875d)  
[PELICAN + GITHUB PAGES でブログを作った話](http://daikishimada.github.io/pelican-start.html)