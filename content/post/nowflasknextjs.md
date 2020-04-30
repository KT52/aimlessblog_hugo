---
title: Vercel（旧 Zeit Now）へのデプロイ覚え書き（Flask + Next.js）
date: 2020-04-30
categories: ["Flask"]
tags: ["flask","next.js"]
slug: nowflasknextjs
adsenseTop: true
adsenseBottom: true
draft: false
---

Vercel（旧 Zeit Now）にFlaskとNext.jsをデプロイしたのでその覚え書き。  
FlaskはGitHubからNext.jsはVercel CLIからデプロイしています。  
nowコマンドが使える前提で進めるのでまだの人は公式とかqiitaとかを参考にしてください。

## Flask

アプリのメインファイルはindex.pyにしました。元々はmain.pyでしたがうまく行かなかったのと、  
ググったらみんなindex.pyだったのでそのようにします。  
アプリを作成したら、`pip freeze > requirements.txt`で必要なライブラリを書き出します。  
次に`Now.json`というファイルを作って、以下のように設定します。

```json
{
    "version": 2,
    "builds": [{ "src": "*.py", "use": "@now/python" }],
    "routes": [
        { "src": "/.*", "dest": "/"}
        ]
}
```

GitHubにPushしたらVercelのダッシュボードに移動して、`import Project`からGitHubのRepositoryを選択して、
後は特に何も設定をせずにデプロイボタンを押せば後は勝手にやってくれて終わりです。  
Flaskで作成したREST APIアプリは問題なく動いています。

## Next.js

Next.jsはVercel CLIからデプロイしました。とあるジャンルのカタログサイトで画像が4000枚くらいあって面倒なのでGitHubにPushするのはやめました。まあデプロイも面倒だけど……  

package.jsonのscriptsは以下のようになっています

```json
"scripts": {
    "dev": "next",
    "build": "next build"
  },
```

buildコマンドはVercelで自動で設定されるので要らないかもしれないです。  

`.next`と`out`ディレクトリがすでに存在するならpackage.jsonと同じ場所に、`.nowignore`ファイルを作成して  
除外設定をしておきます。

```
/.next
/out
```

デプロイは

```
now
```

の3文字です。

```
Now CLI 18.0.0
? Set up and deploy “D:\usr\app_nextjs\frontend2”? [Y/n] y
? Which scope do you want to deploy to? squigly77
? Link to existing project? [y/N] n
? What’s your project’s name? frontend2
? In which directory is your code located? ./
Auto-detected project settings (Next.js):
- Build Command: `npm run build` or `next build`
- Output Directory: Next.js default
- Development Command: next dev --port $PORT
? Want to override the settings? [y/N] n
```

next.jsの場合はAuto-detected以下のように自動でBuild Command等を設定してくれます。  
これは、GitHubからも同様で、

![build](../../../images/vercel.com.jpg)

画像のようになってればOK。もしなってないなら `override`をONにして手動で入力しましょう。  

deployとbuildがエラー無しで成功するとaaaaa-xxxxx.now.shというURLでアクセスできるようになります。  
nowコマンドはデプロイするたびにxxxxxの部分がユニークな英数字のURLを作成してしまうので、  
プロダクション環境で使う場合は

```
now --prod
```

でデプロイして固定のURLでデプロイしましょう。  

## カスタムドメイン

カスタムドメインの設定はsettings→domains。

![domain1](../../../images/vercel2.jpg)

inputエリアにドメインを入力してaddボタンを押します。

![domain1](../../../images/vercel3.jpg)

Apexドメイン（example.com)を使用する場合は

![domain1](../../../images/vercel4.jpg)

画像の青枠で囲まれたネームサーバーをお名前とかドメインを管理しているサイトで設定します。  
ネームサーバーを設定しない、もしくは出来ない場合はANAME（Aレコードじゃない）を設定します。  
ちなみにANAMEが使えるドメインレジストラって日本にあるんですか？  

サブドメインの場合もネームサーバーを変更するかCNAMEを設定します。  
僕はドメインをCloud Flareに移管していて（$8.03/ 1年）ネームサーバーを変更できないのでCloud FlareでCNAMEを設定しています。

![s_cloudflare](../../../images/s_cloudflare.jpg)

Apexドメインも同じCNAMEを設定してCNAME Flattening化してサブドメインにリダイレクト設定をしています。  
ドメインのリダイレクト設定はドメイン設定後、VercelのドメインページのEDITから変更できます。

## おわりに

JAMstackなサイトを作成して1ページあたり20枚くらいの画像を表示させるとNetlifyは結構もたついたけど、  
Vercelはサクッと表示されました。体感的にはfirebase hostingと同じくらい。  
Netlifyと同じでBandwidthが一ヶ月100GBあるので小規模サイトなら無料で色々なことができそうです。