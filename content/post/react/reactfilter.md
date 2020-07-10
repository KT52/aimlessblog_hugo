---
title: React Hooksで絞り込み検索
date: 2020-07-10
tags: ["react"]
draft: false
slug: reactfilter
adsenseTop: true
adsenseBottom: true
---

Hooksで絞り込み検索を実装してみた。

## その1

```js
import React, { useState, useEffect } from "react";
import TextField from "@material-ui/core/TextField";

const data = [
  { product: "SILVERSTONE SST-ST30SF V2", price: "5280" },
  { product: "TOSHIBA MQ04ABF100", price: "4280" },
  { product: "Viewsonic VX2476-SMHD", price: "2980" },
  { product: "Versa H18 mATX", price: "3108" },
  { product: "ASRock B450M-HDV R4.0", price: "7480" },
  { product: "Crucial CT2K8G4DFS832A", price: "7980" },
  { product: "AMD Ryzen 5 3400G BOX", price: "20680" },
  { product: "W.D Blue SN550 WDS500G2B0C", price: "8770" },
];

function Table() {
  const [items, setItems] = useState(data);
  const [value, setValue] = useState("");

  const handleSearch = (e) => {
    setValue(e.target.value);
  };

  useEffect(() => {
    const newItems = items.filter((item) => {
      return (
        item.product.toLowerCase().indexOf(value) !== -1 ||
        item.price.toLowerCase().indexOf(value) !== -1
      );
    });
    newItems(newItems);
    if(!value){
            setItems(data)
        }
  }, [value]);

  return (
    <div>
      <form action="">
        <TextField label="search" onChange={handleSearch} />
      </form>
      {items.map((item, index) => {
        return (
          <li key={index}>
            {item.product} / {item.price}
          </li>
        );
      })}
    </div>
  );
}
export default Table;
```

絞り込み検索を実装するには、`indexOf(value) !== -1`で入力した文字列があるか探して、`filter`メソッドで新たな配列を生成します。  

これで成功かと思いきや、backspaceキーを押して1文字ずつ消したときに絞り込みが行われないし、全部消してもリストが再描画されない……。  
とりあえず`if(!value){setItems(data)}`で検索窓が空になったときに再描画されるようにした。

![gif1](../../../images/reactfilter1.gif)

## その2

```js
import React, { useState, useEffect } from "react";
import TextField from "@material-ui/core/TextField";

const data = [
  { product: "SILVERSTONE SST-ST30SF V2", price: "5280" },
  { product: "TOSHIBA MQ04ABF100", price: "4280" },
  { product: "Viewsonic VX2476-SMHD", price: "2980" },
  { product: "Versa H18 mATX", price: "3108" },
  { product: "ASRock B450M-HDV R4.0", price: "7480" },
  { product: "Crucial CT2K8G4DFS832A", price: "7980" },
  { product: "AMD Ryzen 5 3400G BOX", price: "20680" },
  { product: "W.D Blue SN550 WDS500G2B0C", price: "8770" },
];

function Table() {
  const [items, setItems] = useState(data);
  const [value, setValue] = useState("");
  const [filteredItems, setFiltered] = useState([]);　//追加

  const handleSearch = (e) => {
    setValue(e.target.value);
  };

  useEffect(() => {
    const newItems = items.filter((item) => {
      return (
        item.product.toLowerCase().indexOf(value) !== -1 ||
        item.price.toLowerCase().indexOf(value) !== -1
      );
    });
    setFiltered(newItems); //変更
  }, [value]);

  return (
    <div>
      <form action="">
        <TextField label="search" onChange={handleSearch} />
      </form>
      {!filteredItems
        ? items.map((item, index) => {
            return (
              <li key={index}>
                {item.product} / {item.price}
              </li>
            );
          })
        : filteredItems.map((item, index) => {
            return (
              <li key={index}>
                {item.product} / {item.price}
              </li>
            );
          })}
    </div>
  );
}
export default Table;
```

変更したのは絞り込み検索後の新たな配列を元の'items'ではなく新しいstate変数'filteredItems'にセットした所と、  
レンダリングを'filteredItems'がある時と無いときで分岐した所です。  

![gif2](../../../images/reactfilter2.gif)