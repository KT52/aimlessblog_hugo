---
title: Netlifyで独自ドメインを設定する
date: 2018-07-01
categories: Pelican
tags: ["netlify","python","pelican"]
slug: netlifydomain
adsenseTop: true
adsenseBottom: true
---

Netlifyで独自ドメインを設定してみた

### ドメインを取得する
---

![Xdomain](../../../images/xdomaintop.jpg)  

他サイトのドメインはムームードメインを利用していますが、今回はドメインをXdomain<a href="https://px.a8.net/svt/ejp?a8mat=356RAJ+GF7GHE+CO4+15PEXE" target="_blank" rel="nofollow">（アフィリンク）</a><img border="0" width="1" height="1" src="https://www10.a8.net/0.gif?a8mat=356RAJ+GF7GHE+CO4+15PEXE" alt="">で取得することにしました。  

`ドメインの検索`→`会員登録`→`お支払い情報の入力`→`内容の確認・規約への同意`→`申込み完了`と進んで10～20分で簡単にドメインを取得できます。（クレカ払いの場合）  

### Netlifyにカスタムドメインを登録
---

Domain Setting→Add custom domainと進んで取得したドメインを入力`(www).xxx.com` を入力して`Verify`をクリック  
Netlifyはwww付きのドメインをプライマリードメインとして推奨しているようです。なのでwww付きで登録しました。  


### DNS設定
---

`Verify`をクリックした後はこんな画面になるので`Check DNS Configuration`をクリック。

![dns](../../../images/checkdns.jpg)  

すると、CNAMEをDNSプロバイダー（エックスドメイン）で編集しなさいみたいに表示されますが、僕はNetlifyのDNSを使用するので、`Set up Netlify DNS`をクリック。  
Netlifyの[ドキュメント](https://docs.netlify.com/domains-https/custom-domains/multiple-domains/#apex-domains-and-www-subdomains)には

> Unless your DNS provider supports CNAME flattening, ANAME or ALIAS records for root domains, we strongly recommend setting the www subdomain as your primary domain.  
DNSプロバイダーがルートドメインのCNAMEフラット化、ANAMEまたはALIASレコードをサポートしていない限り、wwwサブドメインをプライマリドメインとして設定することを強くお勧めします。

と書いてあるのでプライマリードメインをwww付きのドメインにした僕の場合はCNAMEの編集をする必要はありません。  
下の画像の"Use Netlify DNS"のところにも、ベストパフォーマンスを得るにはNetlify DNSを必ず使ってねと書いてありますね。ALIASやCNAMEも気にしないでね、とも。  

![dnsconf](../../../images/dnsconf3.jpg)

  
AレコードをドメインプロバイダーのDNSレコードに記述するやり方だとNetlifyの強力なCDNを利用することができないので注意。
  

![dns](../../../images/dnsconf1.jpg)

「Netlify 独自ドメイン」でググるとAレコードを記述する方法ばかりがヒットしたので最初はその方法で設定していましたが、Aレコードを使用するとNetlifyの恩恵が得られなさそうなのでwwwの使用に抵抗がなければwwwありでNetlify DNSを利用しましょう。  

### ネームサーバーの変更
---
`Set up Netlify DNS~~`をクリックして進めていくとDNSレコードをNetlifyが勝手に設定してくれて、更に進むと4つのネームサーバーが表示されるのでコピーしてエックスドメインの「ネームサーバーの確認・変更」から４つのネームサーバーをペーストする。    


![nameserver](../../../images/nameservers.jpg)

  DNSが浸透すると独自ドメインでアクセスできるようになります。


![domainset](../../../images/domainset.jpg)  


### Let’s Encryptの証明書を取得
---
カスタムドメインを設定したらHTTPSの`Verify DNS configuration`→`Let's Encypt certificate`をクリック。しばらくすると証明書を取得できます。  
下の画面のような表示になれば成功。   


![https](../../../images/https.jpg)


### httpsを強制
---
Force HTTPSをクリックしてhttpでのアクセスをhttpsにリダイレクトさせる。

### DNS 設定 （2019年3月13日追記）
---

Netlify DNSを使用してDNSレコードを自動でセッティングした後にMXレコード等のレコードを追加する場合は、  `Domain Settings`からCustom Domainsの`Netlify DNS`をクリックして設定ページに移動、`Add new record`で追加できます。

### 感想
---
ドメインを設定するのは簡単なのですが、改めてドキュメンテーションを読んでもDNS関連は全然知識がないし、英語なので誤読していないか心配。時間があるときに辞書片手にしっかり読まないといけないかなあ～と思ったりしました。