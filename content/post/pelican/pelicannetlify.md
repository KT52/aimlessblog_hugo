---
title: NetlifyにPelicanを使用して静的コンテンツをホスティングする
date: 2018-06-07
categories: Pelican
tags: [netlify,python,pelican]
slug: pelicannetlify
adsenseTop: true
adsenseBottom: true
---

![Netlify](../../../images/netlify.com.jpg)

## Netlifyとは？

[Netlify](https://www.netlify.com/)とは静的コンテンツベースのウェブサイトに特化したWEBホスティングサービスです。  PelicanやHexoなどの静的サイトジェネレーターでウェブサイトを公開するのに便利なサービスです。他にはGitHub Pagesなどがありますね。  
GitHubやGitLabと連携してリポジトリからデプロイしたり、連携しなくてもhtmlファイルなどが入ったフォルダをzipで固めて直接ドラッグアンドドロップでアップロードするだけでサイトを公開できます。  

## Netlifyの特長

- ビルドコマンドが実行可能  
静的サイトジェネレーターでhtmlをジェネレイトするのをNetlifyが勝手にやってくれるので、記事を書いてpushするだけでい
い。

- http/2対応
- 独自ドメインが使用可能

- 無料のssl/https

- CDN  
cloudflareなどのサービスを使わずにキャッシュして高速化することができる。

- フォームの設置

他にもいろいろな機能があります。詳しくは→[Docs](https://www.netlify.com/docs/)参照。  
GitHub Pagesとの比較は[コチラ](https://www.netlify.com/github-pages-vs-netlify/)を参照してください。  
デメリットは日本語未対応なことでしょうか。

## Freeプラン

Freeプランでどれくらい利用できるのかというと、

- ネットワーク転送量 100GB/月
- ストレージ100GB
- APIリクエスト500リクエスト/分, 3デプロイ/分  

## サインイン

ログインはGitHub、GitLab、Bitbucket、Emailから可能

## GitHubと連携

使用するリポジトリを選ぶ。

## Deployセッティング

デプロイセッティングとビルドセッティングを登録したら`Deploy site`をクリックして完了。  
ビルドセッティングは後で設定可。

## コンソール画面

`Deploy site`をクリックしたらこんな風にコンソール画面が表示されます。"happy-hugle-5cae8e.netlify.com"というURLが割り当てられました。  
<a href="../../../images/netlify_overview.jpg" data-toggle="lightbox" data-max-width="100%"><img src="../../../images/netlify_overview.jpg"  class="img-thumbnail" alt="overview" width="300" ></a> 

## GitHubにpushする前にすること  

Pelicanの使用法は[コチラ](https://www.ravness.com/2018/03/pelicangithub/) に書いたので省略。

- .gitignoreファイルを作成してoutputディレクトリをgitの管理から除外する。  

```
echo "/output" >> .gitignore
```

- 使用しているライブラリの一覧を`requirements.txt`に書き出す。

```
pip freeze > requirements.txt
```  
このrequirements.txtを元にNetlifyがライブラリを勝手に読み込んでくれる。

- pythonのヴァージョンを`runtime.txt`に記述。  
デフォルトでは2.7を使用しているので違うヴァージョンを使用している場合は使用するヴァージョンを`runtime.txt`を作って記述する。

```
3.6.4
```

## GitHubにpush

```
git init
git add .
git commit -m "First commit"
git remote add origin https://github.com/ユーザー名/リポジトリ.git
git remote -v
git push origin master
```

全部GitHubにあげてみたけど、仮想環境（venv）のlibディレクトリとかScripitsディレクトリとか要らなくね？
ってことで、.gitignoreに"Lib/"、"Scripts/"等を追記した。  
  
pelicanのテーマとプラグインを導入している場合はそのディレクトリもGitHubにpushすること。

## Netlifyにデプロイ

NetlifyのコンソールページからDeploys→Deploy settingsに進んで、Build commandに`pelican content`、Publish directoryに`/output`を記入してsave。  
GitHubにpushしているとNetlifyが自動でデプロイしてくれる。

Deploysページに"published"が表示されていれば成功。  
<a href="../../../images/deploy2.jpg" data-toggle="lightbox" data-max-width="100%"><img src="../../../images/deploy2.jpg" width="200" alt="deploy" class="img-thumbnail"></a>  
