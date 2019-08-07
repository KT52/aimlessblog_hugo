---
title: Netlify+PelicanのブログにNetlify CMSを追加する
date: 2019-05-17
modified: 2019-05-22
categories: [Pelican]
tags: [pelican,netlify,python]
slug: netlifycms
adsenseTop: true
adsenseBottom: true
---
## はじめに
- - -
[Netlify CMS](https://www.netlifycms.org/)は静的サイトジェネレーターにWordpressのような管理画面を追加するオープンソースのコンテンツ管理システムです。  
Wordpressのデータベースの代わりにGitHubやGitLabのリポジトリを利用して管理しているようです。  
このブログは静的サイトジェネレーターのPelicanを利用していますがここにNetlify CMSを追加してみようと思います。  
そしてこの記事はNetlify CMSから投稿しています。
  

管理画面を表示するまでにする事は少なく、  

1. GitHub認証
2. Netlifyの設定
3. content内にadminディレクトリを作って2つのファイルを置く
4. pelicanconf.pyの編集
5. GitHubにpush
  

これだけです。

## GitHub認証
- - -
最初にGitHubの右上にある自分のアイコンから`setting→Developer settings→OAth Apps`を選択して`Register a new application`をクリック。  
すると下記の画面になるのでこんな感じに入力していきます。    
![gitoauth](/../../../images/gitoauth.jpg)

Application nameは適当にHomepage URLはブログのURL、`Authorization call back URL`は`https://api.netlify.com/auth/done`と入力してRegister applicationをクリック。  
するとClient IDと Client Secretが発行されるのでページはそのままでNetlifyのsite settingsに移動します。  

## Netlifyの設定
- - -
Netlifyのsite settingsの一番下にある`Access control`→`OAuth`→`Install provider`をクリック。すると下記の画面になるので先程のClient IDと Client Secretを入力して`install`を押します。    これでNetlifyの設定は終了です。  
![installprovider](/../../../images/installprovider.jpg)

## Netlify CMS用のファイルを作成
- - -
一旦ブラウザから離れてローカルのcontentディレクトリにadminディレクトリを作成して`index.html`、`config.yml`を作成します。

### index.htmlの中身

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Content Manager</title>
</head>
<body>
  <!-- Include the script that builds the page and powers Netlify CMS -->
  <script src="https://unpkg.com/netlify-cms@^2.0.0/dist/netlify-cms.js"></script>
</body>
</html>

```

公式サイトからのコピペでOKです。

### config.ymlの中身

```python

backend:
  name: github
  repo: Squigly77/myblog
  branch: master

publish_mode: editorial_workflow

media_folder: "content/images"
public_folder: "../../../images"

collections:
  - name: "blog"
    label: "Blog"
    folder: "content"
    create: true
    slug: "{{fields.slug}}"
    identifier_field: title
    fields:
      - {label: "Title", name: "title", widget: "string"}
      - {label: "Date", name: "date", widget: "date",format: "YYYY-MM-DD"}
      - {label: "Modified", name: "modified", widget: "string", required: false}
      - {label: "Category", name: "category", widget: "string"}
      - {label: "Tags", name: "tags", widget: "string"}
      - {label: "Slug", name: "slug", widget: "string"}
      - {label: "Related", name: "related_post", widget: "string", required: false}
      - {label: "Body", name: "body", widget: "markdown"}


```

#### **backend**

GitHubを使用している場合はrepoに使用するリポジトリとbranchを指定。

#### **publish_mode: editorial_workflow**

editorial_workflowというモードを設定することで下書き保存が可能になります。     これがないと記事を作成してセーブすると即座に公開されてしまう。

#### **media/public_folder**

- **media_folder**  
画像ファイル置き場の場所。

- **public_folder**  
記事をアップロードした後に実際にアクセスされる画像ファイルのパス。      記事を書いてるとき画像のパスは、({static}/images/xxx.jpg)とか(../../../images/xxx.jpg)こんな感じに書きますよね。その部分をpublic_folderに指定します。ただ、({static}/images/xxx.jpg)だと上手くいかないっぽい。

#### **collections**

- **name**（必須）  
コレクションのユニークキー。
- **label**  
管理画面上のラベル。指定しない場合は上のnameが使用される。
- **folder**（必須）  
markdownファイルを配置しているフォルダーの場所を指定。通常は`content`。      Netlify CMS上で作成された記事はここに保存されます。カテゴリー等で分けてcontent以下にフォルダーを作成している場合は`content/folder名`。`content`にもcontent/folderにも.mdファイルが置いてある場合はそれぞれのコレクションを作成しなければならない。
- **create**  
これよくわからないのでグーグル翻訳をそのまま載せときます。    フォルダコレクション専用です。 trueの場合、ユーザーはコレクション内に新しい項目を作成できます。デフォルトはfalse
- **slug**  
urlの最後の部分の設定と生成される.mdファイル名の設定。  
デフォルトの{{slug}}と指定すると、Netlify CMSでは後述するfieldのtitleを参照します。  なので日本語ブログだとurlが日本語タイトルをローマ字化したようなurlになり、ファイルも日本語.mdになってしまいます。  
`{{fields.slug}}`とすることでfieldsのslugを参照するようになります。
- **identifier_field**  
Netlify CMSの管理画面上の識別子。fieldのstring値の中から指定する。      僕はタイトルを指定しているので各記事のタイトルが表示される。
- **fields**  
.mdファイルにおけるメタデータの設定。使用するメタデータは人それぞれ違うと思うのでそのまんまコピペしないように。  
bodyは記事を書くフィールド。
    - **label**  
管理画面上で表示される名前とメタデータの`Title:`などの部分。
    - **name**（必須）  
フィールドのユニークな識別子。
    - **widget**  
フィールドのタイプ。stringなら文字、dateなら日付を
    - **required: false**  
必須項目ではないフィールドにする。  

folderのところでも書きましたが、markdownファイルをcontentとcontent/categoryみたいに分けて保存していて、Netlify CMSで作成された記事を任意の場所に置きたい場合はcollectionsのnameからfieldまでをそれぞれ記述しなければならないので注意。  

```yaml

collections:  
  - name: "blog"
    label: "Blog"
    folder: "content" #contentフォルダ
    create: true
    slug: "{{fields.slug}}"
    identifier_field: title
    fields:      
      - {label: "Title", name: "title", widget: "string"}
      #省略
  - name: "python"
    label: "python"
    folder: "content/python" #content内のpythonフォルダー
    create: true
    slug: "{{fields.slug}}"
    identifier_field: title
    fields:      
      - {label: "Title", name: "title", widget: "string"}
　　　#省略

```

以上でconfig.ymlの説明終わりです。

## pelicanconf.pyの編集
- - -

pelicanconf.pyに

```python

TEMPLATE_PAGES = {'admin/index.html': 'admin/index.html'}
STATIC_PATHS = ['images','static','admin'] #adminを追加

```

を追加。  これでサイトurl/adminにアクセスできるようになる。  markdownファイルをcontentフォルダのさらに下に置いてある場合はSTATIC_PATHSにそのフォルダ名を追記する必要があるかも（未確認）。

## GitHubにpush
- - -
準備が整ったのでGitHubにpush。

## 管理画面にアクセス
- - -

サイトurl/adminにアクセスすると、ログイン画面が表示されるのでログイン。

![login](/images/login.png)

Netlify CMSの管理画面が表示されました！

![netlifycmstop](/images/netlifycmstop.png)

ただ、すでに投稿済みのpelican→netlifyで作成した記事のタイトルは表示されず(泣)  
確認するとフィールドが空なのでCMS上でタイトルを表示させるにはフィールドにメタデータを記入して再投稿する必要がありそう。  
このスクショには写ってないですがNetlify CMSで作ったテスト投稿の記事はちゃんとタイトルが表示されています。  

（2019-05-22追記）CMSで作られた.mdファイルのメタデータと同じ"---"でメタデータを囲み、Titleをtitleみたいにfieldsのnameと同じ小文字にしたらCMSでもタイトルが表示されました。  

下書きを保存するWorkflowのページはこんな感じ。

![netlifycmsflow](/images/cmsflow.png)

new postをクリックして編集画面

![cmsedit1](/images/cmsedit1.png)
![cmsedit2](/images/cmsedit2.png)

画像のアップロードはトップページのMediaからか記事作成時markdownからリッチテキスト形式に変えてアップロードして記事に貼り付けることができます。  
記事を公開するには編集画面右上の`Set Status`をDraftからReadyにしてPublish。

![setstatus](/images/setstatus.gif)

または、workflow画面で記事をReadyにドラッグアンドドロップしてpublish。


![drafttoready](/images/drafttoready.png)

実際のブログでのプレビューを見るには編集画面の右上check for previewボタンを押してView Preview
に切り替わったらそこからプレビューに飛びます。ただしView Preview
に切り替わるまで少し時間がかかります。

## おわりに

することは認証してadminディレクトリにファイル作成してpelicanconf.pyを少し修正してGitHubにpushするだけで簡単なのですが、PelicanでNetlify CMSを利用する記事が全く無く、pelicanの理解が浅いとpelicanconf.pyの編集をしないとサイトurl/adminが"page not found"でアクセスできないという壁にぶち当たります。  
あと、config.ymlファイルを修正してpushして、を何度も繰り返してやっと記事を投稿できるようになったので、[config.ymlのドキュメント](https://www.netlifycms.org/docs/configuration-options/)をグーグル翻訳を使ってしっかり読んだほうが良いです。

