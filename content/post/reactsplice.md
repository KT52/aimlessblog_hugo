---
title: React Hooksで配列の要素を削除したあとの再描画について
date: 2020-02-19
tags: ["react"]
draft: false
slug: reactsplice
adsenseTop: true
adsenseBottom: true
---

削除ボタンを設置したToDoリストやテーブルでボタンを押しても再描画されないときの解決法。

## クラス型コンポーネントの場合

削除に関係する部分のコードです。

```react

class TableList extends Component {
  constructor(props) {
    super(props);
    this.state = {
      items: [],
    }

    handleDelete(id, index) {
      const data = { id: id };
      axios
        .delete("http://localhost:5000/api/mydata", { data: data })
        .then(response => {
          console.info(response.data.message);
          this.state.items.splice(index, 1);
          this.setState({ items: this.state.items });
        })
    }
    render() {
    return (
      <table>
        {this.state.items.map((item, index) => (
          <tr key={item.id}>
            <td>{item.name}</td>
              <button onClick={() => this.handleDelete(item.id, index)}>
              Delete
              </button>
            </tr>
        ))}
      </table>
      )
    }
  }

```

クラス型コンポーネントの場合は`splice(index, 1)`のあとに`this.setState`で配列を上書きするだけ。

## React Hooksの場合

Hooksも同じだろうと思ってこのようなコードを書く

```react

function TableList() {
  const [items, setItems] = useState([]);
  
  const handleDelete = (id, index) => {
    const data = { id: id };
    axios
      .delete("http://localhost:5000/api/mydata", { data: data })
      .then(response => {
        console.info(response.data.message);
        items.splice(index, 1);
        setItems(items);
      })
  }

      return (
        <Button　onClick={() => handleDelete(item.id, index)}>
          Delete
        </Button>
      )
  }

```

が、うまく行かない。データは削除されているけどリロードをしないと反映されない。
どうしたらいいのかは、[スタックオーバーフロー](https://ja.stackoverflow.com/questions/57786/react-functional-component-%E5%86%8D%E3%83%AC%E3%83%B3%E3%83%80%E3%83%BC%E3%81%95%E3%82%8C%E3%81%AA%E3%81%84)にありました。  

Reactドキュメントの[フック API リファレンス　state 更新の回避](https://ja.reactjs.org/docs/hooks-reference.html#bailing-out-of-a-state-update)によると  

> 現在値と同じ値で更新を行った場合、React は子のレンダーや副作用の実行を回避して処理を終了します。（React は Object.is に
> よる比較アルゴリズム を使用します）

と書いてあります。  

Object.isとはなんぞや？と調べてみると
>Object.is() メソッドは 2 つの値の同一性を判定します。

との事。`items.splice(index, 1);`の後に`setItems(items);`でstate更新をしてもObject.isの判定ではtrueとなってしまうので再描画されないみたい。  
なので、このようにして再描画されるようにします。

```react

const newItems = [...items];
newItems.splice(index, 1);
setItems(newItems);

```

`[...items]`の...って何？と調べるとスプレッド構文とかいうものでオブジェクトや配列を展開するものらしい。  
つまり、

1. itemsの中身をスプレッド構文で展開してnewItemsに入れる
2. newItemsをspliceして配列の要素を削除
3. itemsとnewItemsはfalseなのでstateが更新されて再描画される

[state 変数は 1 つにすべきですか、たくさん使うべきですか？](https://ja.reactjs.org/docs/hooks-faq.html#should-i-use-one-or-many-state-variables)で...が使われていてreact独自の何かだと思って調べずに使っていたけどちゃんと意味があるのですね（当たり前）  

[フック API リファレンス 関数型の更新](https://ja.reactjs.org/docs/hooks-reference.html#functional-updates)の補足に

>補足
>クラスコンポーネントの setState メソッドとは異なり、useState は自動的な更新オブジェクトのマージを行いません。この動作は
>関数型の更新形式をスプレッド構文と併用することで再現可能です：
> ```
> setState(prevState => {
>  // Object.assign would also work
>  return {...prevState, ...updatedValues};
>});
> ```

と書いてあったのでこれも行けるのでは？と思い、

```react

items.splice(index, 1);
setPage(items => ({ ...items, items }))

```

こう書いても上手くいきました。