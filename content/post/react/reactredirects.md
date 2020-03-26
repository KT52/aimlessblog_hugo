---
title: ReactやVueのアプリをNetlifyやFirebase Hostingにデプロイしてリロードすると404になる問題
date: 2020-03-26
tags: ["react","netlify","firebase","vue"]
draft: false
slug: reactredirects
adsenseTop: true
adsenseBottom: true
---

Reactで作ったアプリやWebサイトをNetlifyやFirebaseなどの静的サイトホスティングサービスにデプロイして  
react-router-domやvue-routerで指定したルート、example.netlify.com/aboutのような
ページに移動した後にf5（リロード）すると404ページが表示されてしまう問題は、rewriteやredirectの設定をすれば解決します。

## netlify

reactとvueのpublicディレクトリに`_redirects`というファイルを作成して、以下のように記述してbuildする。

```
/*    /index.html   200
```

## firebase

`firebase.json`を下記のように"rewrites"以下を追加してデプロイ

```json
{
  "hosting": {
    "public": "build",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```

### 参考

[netlify docs](https://docs.netlify.com/routing/redirects/rewrites-proxies/#history-pushstate-and-single-page-apps)  
[firebase ドキュメント](https://firebase.google.com/docs/hosting/url-redirects-rewrites?hl=ja#section-rewrites)