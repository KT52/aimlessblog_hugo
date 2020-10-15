---
title: Reactのlightbox-reactを使ってみる
date: 2020-10-15
categories: ["React"]
tags: ["react", "nextjs", "lightbox"]
slug: lightboxreact
adsenseTop: true
adsenseBottom: true
draft: false
---

lightbox系のライブラリでは[react-medium-image-zoom](https://github.com/rpearce/image-zoom)が一押しですが、modalを開いてから次の画像にスライドできないので、
Reactのlightboxライブラリ[「lightbox-react」](https://github.com/treyhuffine/lightbox-react)を試してみました。  
オリジナルの[react-image-lightbox](https://github.com/frontend-collective/react-image-lightbox)はxボタン以外をクリックしても閉じることができなかったり、zoomした後の画像が動かせなかったりしたのでこちらを使用します。

## インストール

`npm install lightbox-react`

## example

公式のexampleはクラスコンポーネントで書かれてますが、hooksとnext.jsを使用します。  
まずはindex.jsにimport

```js
import React, { useState } from "react";
import Lightbox from "lightbox-react";
import "lightbox-react/style.css";
```

next.jsを使ってるなら`import "lightbox-react/style.css"`を`_app.js`に記述します。

```js
import React, { useState } from "react";
import Lightbox from "lightbox-react";
import Homecss from "../styles/Home.module.css"

const images = [
  "001.jpg","002.jpg","003.jpg","004.jpg"
]

const title = ["Azura", "Ingun Black-Briar", "The Blue Palace", "Cicero"];

export default function Home() {
  const [photoIndex, setIndex] = useState(0);
  const [isOpen, setisOpen] = useState(false);

  return (
    <div>
      <main className={Homecss.main}>
        <h3>lightbox-reactのDEMO</h3>
        <div>
          {images.map((image, index) => (
            <img
              src={`/img/${image}`}
              alt={image}
              width="200px"
              onClick={() => {
                setisOpen(true), setIndex(index);
              }}
              key={index}
              className={Homecss.img}
            />
          ))}
        </div>

        {isOpen && (
          <Lightbox
            mainSrc={`/img/${images[photoIndex]}`}
            nextSrc={images[(photoIndex + 1) % images.length]}
            prevSrc={images[(photoIndex + images.length - 1) % images.length]}
            onCloseRequest={() => setisOpen(false)}
            onMovePrevRequest={() =>
              setIndex((photoIndex + images.length - 1) % images.length)
            }
            onMoveNextRequest={() => setIndex((photoIndex + 1) % images.length)}
            imageTitle={title[photoIndex]}
            imageCaption={title[photoIndex]}
           />
        )}
      </main>
    </div>
  );
}
```

公式のサンプルはボタンを押してモーダルを開くようになっていますが、画像をクリックして表示させたいので  
`photoIndex`にセットする値は項目のインデックスを使用。

\<Lightbox>以下でlightboxの色々なオプションを設定。  
`mainSrc`と`CloseRequest`が必須項目。  
キャプションとタイトルはデフォルトでは何も表示されないので追加したり、デフォルトでズーム可なのを`enableZoom={false}`で不可にしたりできます。画像のダウンロードをできなくする`discourageDownloads`をtrueにすると不具合があるので設定するのは止めましょう。  

上のコードを実際に表示させたのがこれ→[DEMO](https://cocky-rosalind-bc79b3.netlify.app/)