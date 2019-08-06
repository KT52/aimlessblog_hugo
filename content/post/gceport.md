---
title: GCEでSSHのポート番号を変更する
date: 2019-03-19
categories: [GCP]
tags: [vps,gce]
slug: gceport
---

Google Compute Engine(GCE)でSSH接続のポート番号がデフォルトの22番だと不正アクセスを試みようとするログがこんな感じでいっぱい残っているので、ポート番号の変更をしてみます。<br>

![不正アクセス](../../../images/gce_access.jpg)

## ファイアウォールルールの作成
---

GCPのコンソールページのナビゲーションメニューから、`VPCネットワーク`→`ファイアウォール　ルール`に移動して`ファイアウォール　ルールを作成`をクリック。  

設定するのは、  

-  名前→任意の名前
-  ターゲットタグ→任意のタグ
-  ソース IP の範囲→0.0.0.0/0
-  指定したプロトコルとポート→tcpにチェックしてポート番号を適当に

他は初期設定のままでOK。<br>

![不正アクセス](../../../images/firewallrule.jpg)  

## VMインスタンスにネットワークタグを設定
---

GCPのコンソールページのナビゲーションメニューから、`Compute Engine`→`VMインスタンス`に移動。使用するインスタンス内の画面上部にある編集をクリックして、ネットワークタグにファイアウォールルールで作成したターゲットタグを入力してenterキー。画面下部の保存で設定の保存<br>

## SELinux と iptablesの無効化
---

CentOSのSELinux と iptablesを無効化します。

```
systemctl disable firewalld
```

<br>
#### /etc/selinux/configを編集

```
cd /etc/selinux/
```

念のためバックアップ

```
cp config config_old
```

```
vim configで
#SELINUX=enforcingをコメントアウトして
SELINUX=disabledを記入
```

GCEを再起動（停止→開始）    
```
systemctl is-enabled firewalld
getenforce
```

でどちらも`disable`になっていればOK。<br>

## ポート番号を変更
---

sshd_configに使用するポート番号を記入

```

cd /etc/ssh
cp sshd_config sshd_config_old 念のためsshd_configをバックアップ
vim sshd_config
port 44444
```

sshdを再起動
`systemctl restart sshd` <br>

## 実際に接続
---

まずはコンソールからSSHで`ブラウザウィンドウで開く`をクリック。  
![ssh_fail](../../../images/gce_ssh_fail.jpg)<br>

<br>ポート22で接続できないことを確認<br>
![port22_fail](../../../images/gce_port22_fail.jpg)  

次は`ブラウザ ウィンドウでカスタムポートを開く`をクリックしてポート番号を入力して接続できればポート番号の変更は成功です。

ターミナルソフトで接続する場合もポート番号を22から変更して接続。  
<br>Google Cloud SDK shellから接続する場合は
```
gcloud compute ssh インスタンス名 --ssh-flag="-P ポート番号"
```

-P は大文字じゃないと接続できなかったです。