---
title: GCEでSSHのポート番号を変更する
date: 2019-03-19
categories: [GCP]
tags: [vps,gce]
slug: gceport
adsenseTop: true
adsenseBottom: true
---

Google Compute Engine(GCE)でSSH接続のポート番号がデフォルトの22番だと不正アクセスを試みようとするログがこんな感じでいっぱい残っているので、ポート番号の変更をしてみます。

![不正アクセス](../../../images/gce_access.jpg)

## ファイアウォールルールの作成
---

GCPのコンソールページのナビゲーションメニューから、`VPCネットワーク`→`ファイアウォール　ルール`に移動して`ファイアウォール　ルールを作成`をクリック。  

設定するのは、  

-  名前→任意の名前
-  ターゲットタグ→任意のタグ
-  ソース IP の範囲→0.0.0.0/0
-  指定したプロトコルとポート→tcpにチェックしてポート番号を適当に

他は初期設定のままでOK。

![不正アクセス](../../../images/firewallrule.jpg)  

## VMインスタンスにネットワークタグを設定
---

GCPのコンソールページのナビゲーションメニューから、`Compute Engine`→`VMインスタンス`に移動。使用するインスタンス内の画面上部にある編集をクリックして、ネットワークタグにファイアウォールルールで作成したターゲットタグを入力してenterキー。画面下部の保存で設定の保存。

## SELinux と iptablesの無効化
---

CentOSのSELinux と iptablesを無効化します。

```sh
systemctl disable firewalld
```


#### /etc/selinux/configを編集

```sh
cd /etc/selinux/
```

念のためバックアップ

```sh
cp config config_old
```

```sh
vim configで
#SELINUX=enforcingをコメントアウトして
SELINUX=disabledを記入
```

GCEを再起動（停止→開始）    
```sh
systemctl is-enabled firewalld
getenforce
```

でどちらも`disable`になっていればOK。

## ポート番号を変更
---

sshd_configに使用するポート番号を記入

```sh
cd /etc/ssh
```
```sh
cp sshd_config sshd_config_old  念のためsshd_configをバックアップ
```
```sh
vim sshd_config
```
```sh
port 44444
```

sshdを再起動
`systemctl restart sshd`

## 実際に接続
---

まずはコンソールからSSHで`ブラウザウィンドウで開く`をクリック。

![ssh_fail](../../../images/gce_ssh_fail.jpg)

ポート22で接続できないことを確認

![port22_fail](../../../images/gce_port22_fail.jpg)  

次は`ブラウザ ウィンドウでカスタムポートを開く`をクリックしてポート番号を入力して接続できればポート番号の変更は成功です。

ターミナルソフトで接続する場合もポート番号を22から変更して接続。  
Google Cloud SDK shellから接続する場合は

```sh
gcloud compute ssh インスタンス名 --ssh-flag="-P ポート番号"
```

-P は大文字じゃないと接続できなかったです。