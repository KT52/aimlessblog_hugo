---
title: FlaskとVueでCRUD操作（フロントエンド編）
date: 2019-12-27
categories: ["Flask"]
tags: ["flask","vue"]
slug: flaskvuefrontend
adsenseTop: true
adsenseBottom: true
draft: false
---

FlaskとVue.jsで作る簡易Webアプリフロントエンド編です。

## セッティング

バックエンド（API）とのやりとりに必要なaxiosをインストールします。

```
npm install --save axios
```

こちらは必須ではありませんがBootstrapとBootstrap-Vueをインストールします。

```
npm install --save bootstrap-vue bootstrap
```

## 画面の作成

ブックリストをテーブルで表示してテーブル上で更新と削除ができるようにします。  
新規登録はテーブルの下に作成します。

```html

<template>
  <div class="booklist">
    <div class="container">
      <div class="row">
        <div class="col-sm-12">
          <h1>ブックリスト</h1>
          <table class="table table-hover table-striped">
            <thead>
              <tr class="table-info">
                <th>id</th>
                <th>タイトル</th>
                <th>著者</th>
                <th>出版社</th>
                <th>更新</th>
                <th>削除</th>
              </tr>
            </thead>
            <tbody v-if="!edit">
              <tr v-for="(item,index) in items" :key="index">
                <td>{{item.id}}</td>
                <td>{{item.title}}</td>
                <td>{{item.author}}</td>
                <td>{{item.publisher}}</td>
                <td>
                  <b-button v-if="!edit" @click="Edit(item.id)" size="sm">Edit</b-button>
                </td>
                <td>
                  <b-button @click="del(item.id,index)" variant="danger" size="sm">Del</b-button>
                </td>
              </tr>
            </tbody>
            <!--------- 編集モード ----------->
            <tbody v-if="edit">
              <tr v-for="item in items" :key="item.id">
                <td>{{item.id}}</td>
                <td v-if="itemid == item.id">
                  <input v-model="item.title" size="10" />
                </td>
                <td v-else>{{item.title}}</td>
                <td v-if="itemid == item.id">
                  <input v-model="item.author" size="10" />
                </td>
                <td v-else>{{item.author}}</td>
                <td v-if="itemid == item.id">
                  <input v-model="item.publisher" size="10" />
                </td>
                <td v-else>{{item.publisher}}</td>
                <td v-if="itemid == item.id">
                  <b-button
                    @click="update(item.id,item.title,item.author,item.publisher)"
                    size="sm"
                  >update</b-button>
                </td>
                <td v-else></td>
                <td v-if="itemid == item.id">
                  <b-button @click="cancel" size="sm">Cancel</b-button>
                </td>
                <td v-else></td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      <!---------- 新規登録 ----------->
      <b-container>
        <b-row v-for="text in texts" :key="text">
          <b-col sm="3">
            <label>{{ text }}:</label>
          </b-col>
          <b-col sm="6">
            <b-form-input :id="text" type="text" class="mb-2"></b-form-input>
          </b-col>
        </b-row>
        <b-button @click="add">登録</b-button>
      </b-container>
    </div>
  </div>
</template>

```

scriptの一部

```js

<script>
const axios = require("axios").create()

export default {
  name: "booklist",
  data() {
    return {
      items: [],
      edit: false,
      itemid: "",
      texts: ["title", "author", "publisher"]
    };
  },
  mounted() {
    this.getItems()
  },
  methods: {
    getItems() {
      /* eslint-disable */
      const response = axios.get("http://localhost:5000/api/bookapp")
        .then(response => {
          console.log(response.data)
          this.items = response.data
        }).catch(error => {
          console.log(error.response.data)
        })
    },

// 省略

```

tbodyは通常モードと編集モードで分けるために`v-if="!edit"`と`v-if="edit"`で分けています。  
editはデフォルトでfalseになっていて、editボタンを押すとtrueになって編集モードになります。  
全部のテーブルが編集モードになっても意味がないのでeditボタンを押したところだけ開くようにしています。  
cancelでもとに戻す処理。

```js
Edit(id) {
      this.edit = true
      this.itemid = id
    },
cancel() {
      this.edit = false
    }
```

![booklist1](../../../images/booklist1.gif)

delボタンを押すとその行のリストを削除します。

```js

del(id,index) {
      const data = { id: id }
      axios.delete("http://localhost:5000/api/bookapp", { data: data })
        .then(response => {
          this.items.splice(index, 1)
          console.info(response.data.message)
        })
    }

```

axiosのdeleteは第2引数にdataをkey名でセットする必要があるので、`{ data: data }`  
としてバックエンドに送っています。  
idに紐付いたデータを削除するだけではなく、テーブルからも取り除く必要があるので

```js
this.items.splice(index, 1)
```

で取り除いています。

![booklist2](../../../images/booklist2.gif)

新規登録のメソッド

```js
add() {
      var title = document.getElementById("title").value
      var author = document.getElementById("author").value
      var publisher = document.getElementById("publisher").value
      const params = { title: title, author: author, publisher: publisher }
      axios.post("http://localhost:5000/api/bookapp", params)
        .then(response => {
          const id = response.data
          const newitem = {
            id: id,
            title: title,
            author: author,
            publisher: publisher
          }
          this.items.push(newitem)
          console.log(response)
        }).catch(error => {
          console.log(error.response.data)
        })
    }
```

登録後に`const id = response.data`でflaskから返されたid番号を取得して、`this.items.push(newitem)`でテーブルに追加しています。  

![booklist3](../../../images/booklist3.gif)  

これでFlaskとVue.jsで作った簡易アプリにCRUD機能を実装できました。  

ソースコードは[こちら](https://github.com/Squigly77/flask_vue_sample)から。

## 参考

[Vue.js 日本語ガイド](https://jp.vuejs.org/v2/guide/)  
[BootstrapVue](https://bootstrap-vue.js.org/)  
[axiosライブラリを使ってリクエストする](https://qiita.com/reflet/items/d5658d5d69e8e1ccd489)  
[配列の要素挿入、置き換えもsplice()で。Vue.jsでも大丈夫。（ 配列とかおれおれAdvent Calendar2018 – 07日目）](https://ginpen.com/2018/12/07/array-splice-to-insert-replace/)