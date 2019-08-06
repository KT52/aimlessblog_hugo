---
title: GCEでSSH認証鍵をPuTTYgenで作成してターミナルソフトで接続する
date: 2019-03-08
categories: [GCP]
tags: [vps,gce]
slug: gcessh
related_posts: gceport
---

## PuTTYgenを入手する

[公式サイト](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)の真ん中あたりにあるputtygen.exeをダウンロードして任意の場所に置く。<br>

## PuTTYgenで公開鍵と秘密鍵を作成

puttygen.exeをクリックして起動。

Generateをクリックして緑色のバーが満タンになるまでマウスを赤枠のところでグリグリ動かす。<br>

![puttygen1](../../../images/puttygen1.jpg)

<br>下の画像のkey_commentを任意の名前に変更。key_passphraseはより強固にしたい場合は入力。<br>
一旦PuTTYgenは閉じずに置いといて、<br>

![puttygen2-1](../../../images/puttygen2-1.jpg)

ブラウザでGCPのCompute Engine→メタデータに移動してssh認証鍵→編集→`認証鍵全体を入力`に上の画像の赤枠の全て（見えてない部分もスクロールして）をコピペする。今回はkey_commentをtestにしたのでtestと表示されていれば問題なし。保存をクリック。<br>

![puttygen3](../../../images/puttygen3.jpg)

<br>再びPuTTYgenに戻って`save private key`をクリック。任意の名前.ppkで保存。無くさないように注意。<br>
これで公開鍵と秘密鍵の作成は終了です。<br>

## ターミナルソフトRLoginでSSH接続

[RLogin](http://nanno.dip.jp/softlib/man/rlogin/)でssh接続してみます。

ホスト名にはインスタンスの外部IP、ログインユーザー名にPuTTYgenのkey_comment、TCPポートはSSH、key_passphraseを作成した場合はパスワードorパスフレーズにkey_passphraseを入力。`SSH認証鍵`をクリックして先ほど作成した秘密鍵(xxxxx.ppk)を指定。<br>
OKをクリックして接続されれば成功。

![rlogin](../../../images/rlogin.jpg)<br>

![rlogin2](../../../images/rlogin2.jpg)
