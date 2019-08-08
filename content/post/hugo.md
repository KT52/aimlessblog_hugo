---
title: PelicanからHugoに移行しました
date: 2019-08-08
categories: Misc
tags: [hugo,netlify]
slug: hugo
adsenseTop: true
adsenseBottom: true
---

静的サイトジェネレーターをPelicanからHugoに移行しました。  
Pelicanにはなんの不満もありませんが、[getpelican/pelican-themes](https://github.com/getpelican/pelican-themes)を見ればわかるように、ほとんどのテーマが数年前から更新されずあまり活気が無いような気がしたのでHugoを試してみました。

## 実際にHugoをつかってみて
---

### Go言語を知らなくても静的サイトをジェネレイトするくらいなら問題ない。

Pelicanだと必要なライブラリをpipで導入したりしなければいけないのでpythonの知識が必要になりますが、
HugoはGo言語を全く知らなくても静的サイトを公開できます。
あと、PelicanはPython、JekyllはRubyを導入しないと使用できませんがHugoはHugoをインストールするだけでGo言語をインストールしなくていい。  
いまもどうやって"Hello World"するのかわからないです。

### LiveReload

Pelicanだと下書きや設定を変更した時に、ジェネレイトしてoutputディレクトリに移動して"python -m http.server"でサーバーを起動して……という動作が必要になりますが、Hugoはジェネレイトする必要がなくサーバーを起動したまま設定を変更したり記事を書いて保存するとすぐ反映されます。


## Hugoで静的サイトを作成してみた系の記事であまり書いてないけど知っておいたほうが良いこと
---

- Netlifyに公開する場合はテーマのカスタマイズをテーマの"layouts"ディレクトリ以下のファイルを直接いじるのではなく、必要なファイルをテーマが入ってるディレクトリじゃないほうの"layouts"ディレクトリに同じ構成のままコピーして、そちらのファイルを修正する。  
cssの変更はstatic/css/custom.cssに書く。
- Hugoのサマリー機能は日本語だともの凄く長くなるのでconfig.tomlに`hasCJKLanguage = true`を追加する。
- Netlifyにデプロイするときは"Show Advance"→"new variable"から"key"に`HUGO_VERSION`、"value"にダウンロードしたHugoのバージョンを記入してデプロイする。書かないと多分デプロイは失敗します。


記事数も少ないのでダウンロードして三日目くらいでPelicanから移行できるくらい簡単でした。configファイルで設定することもPelicanとあまり大差ないので他の静的サイトジェネレーターを使用したことがあるなら移行は簡単だと思います。
