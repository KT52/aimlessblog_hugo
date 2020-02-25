---
title: Herokuを使ってスクレイピングした情報をLINE Notifyで通知する
date: 2018-12-05
categories: [Python]
tags: [python, heroku]
slug: herokulinenotify
adsenseTop: true
adsenseBottom: true
---

スクレイピングした情報をLINE Notifyで通知するスクリプトは完成したので、  
今回はこれを定期実行するためにHerokuにデプロイしたいと思います。

## Heroku

HerokuとGitの使い方はここでは省略しますが、Herokuのアカウントを持ってない場合は作って  
Heroku CLIをダウンロード。

## Herokuにデプロイするまでに準備するもの

- jra.py スクリプト実行ファイル
- index.py
- Procfile
- requirements.txt
- runtime.txt

## requirement.txt

必要なライブラリをここに記入。最初にデプロイするときにpipからインストールしてくれます。  

```txt
beautifulsoup4==4.6.3
requests==2.20.1
bottle==0.12.15
```

仮想環境でライブラリをインストールしているなら`pip freeze > requirements.txt`でも可。  

## Procfile

Herokuで実行するファイルをここに記述。ファイル名はProcfileのみで拡張子はなし。  
`web: python index.py`

## index.py

index.pyというダミーファイルを置きます。そうしないとアプリを起動しただけ（デプロイしただけ？）でスクリプトを実行してしまい、Line Notifyに複数回同じ内容の通知が届いてしまいます。  
index.pyはフレームワークbottleを使用

```python

import os
from bottle import route, run


@route("/")
def hello():
    return "Hello"

run(host="0.0.0.0", port=int(os.environ.get("PORT", 5000)))

```

## runtime.txt

使用するpythonのバージョンを記述

```txt
python-3.6.5
```

## 実行ファイルを本番用に変更

いままではアクセストークンをそのまま記述していましたが、セキュリティを考えて環境変数を設定する形にします。
環境変数の設定は後述。

```python

import requests
import re
from bs4 import BeautifulSoup
import datetime
import os #osモジュールで環境変数を読み込む

TOKEN = os.environ["TOKEN"] #環境変数"TOKEN"を取得


def line(mes,TOKEN): #<------
    url = "https://notify-api.line.me/api/notify"
    token = TOKEN #アクセストークンはここに記述しない
    headers = {"Authorization": "Bearer " + token}
    message = '\n' + mes
    payload = {"message":  message}
    requests.post(url, headers=headers, params=payload)

def main():
    today = datetime.datetime.today().strftime("%m/%d")
    headers = {'User-Agent': 'Mozilla/5.0'}
    res = requests.get('https://world.jra-van.jp/news/', headers=headers)
    soup = BeautifulSoup(res.content, "html.parser")
    for news in soup.find_all("div", id="news-area"):
        for a in news.find_all("a"):
            link = a.get("href")
            if a.find_all(text=re.compile(today)):
                for title in a.find_all("p", class_="inline-item text-title photo"):
                    texts = 'https://world.jra-van.jp'+link, title.get_text()
                    text = "\n".join(texts)
                    line(text,TOKEN) #<-------


if __name__ == '__main__':
    main()
```

コメント部分が[前回](https://www.ravness.com/2018/12/linescraping/)のコードから変更したところ。

## Herokuにデプロイ

Herokuにアプリを作成してデプロイします。  
僕はHerokuのダッシュボードから`create new app`で新しいアプリを作成した（してしまった）ので、

```bash
heroku login
git init
heroku git:remote -a アプリ名
```

で新しいGitのレポジトリを作成。  
その後は

```bash

git add .
git commit -m "First commit"
git push heroku master

```

でデプロイ。デプロイの仕方はダッシュボードのDeployタブで確認・編集できます。

## トークンを環境変数に設定

```bash
heroku config:set TOKEN="アクセストークンをここに"
```

HerokuのsettingページのConfig Varsで設定した環境変数を確認することができます。

## Heroku Scheduler

Herokuで定期的にappを実行するにはスケジューラーが必要なのでHeroku Schedulerのアドオンを使用。  
アドオンを追加するコマンドは
```bash
heroku addons:create scheduler:standard
```
  
なお、アドオンを使用する場合は無料利用でもクレジットカード、もしくはデビットカード情報を入力しなければならないので注意。  
クレカ情報を登録したくないならAPSchedulerというライブラリを使用すると良いみたいです。  

Herokuのダッシュボードにアクセスすると  
![dashboard](../../../images/dashboardheroku.jpg)

Heroku Schedulerが追加されているのでconfigure add onsをクリックしてスケジューラーの設定をする。

![schedulerheroku](../../../images/schedulerheroku.jpg)

$に実行するファイルを設定して、時間の部分はUTCなので実行したい時間にマイナス９時間した時刻を設定。  
今回は16：00に実行したいので07:00にする。

## 結果

16：00に実行されました。

![helokunotfiy](../../../images/herokuline.jpg)

