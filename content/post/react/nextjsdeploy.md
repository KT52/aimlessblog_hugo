---
title: Next.jsを色々なサービスにデプロイするときの設定メモ
date: 2020-05-06
categories: ["React"]
tags: ["next.js","netlify","gae","firebase","react"]
slug: nextjsdeploy
adsenseTop: true
adsenseBottom: true
draft: false
---

Next.jsをNetlify、Firebase Hosting、Google App Engineにデプロイするときの設定などのメモ。  
今後他のサービスにデプロイした場合は追記するかも。  
Vercel（旧 Zeit Now）は前回紹介したので省略。

## Netlify

package.jsonのscriptsをこのようにする。

```json
"scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start",
    "export": "next export",
    "deploy": "npm run build && npm run export"
  }
```

Netlifyのbuild settingを  
`Build command: npm run deploy`  
`Publish directory: out/`  
にする。

![netlifynext](../../../images/netlfynext.jpg)

リロードしたときの404対策が必要な場合はpackage.jsonと同じ場所に`netlify.toml`を作って  
下記を記述。

```toml
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

## Firebase Hosting

Firebaseはローカルで`npm run export`したあとのoutディレクトリをデプロイするだけなので、  
firebase.jsonの設定だけです。

```json
"hosting": {
    "public": "out",
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
```

あとは`firebase deploy`でデプロイするだけ

## Google App Engine

まずはpackage.jsonのscriptsをこのようにします

```json
"scripts": {
    "gcp-build": "next build",
    "start": "next start -p $PORT"
  }
```

GAEは`app.yaml`に

```yaml
runtime: nodejs10
```

最低限これだけ書けばOKです。  
その他の設定（staticファイルとか）は各自違うと思うので省略します。

あとは、`.gcloudignore`にデプロイしないファイルの設定などを記述してデプロイ。

.gcloudignoreの例
```
.git
.gitignore

node_modules/
.next/
out/
```

