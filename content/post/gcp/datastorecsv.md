---
title: GCPのデータストアにcsvのデータを一括で登録する
date: 2018-08-08
categories: [GCP]
tags: [Python,GAE]
slug: datastorecsv
adsenseTop: true
adsenseBottom: true
---
## bulkloaderでアップロード
---

Google Cloud Platform (GCP)のデータストアにGoogle App Engine（GAE)経由でcsvファイルから一括でアップロードする場合、bulkloaderを使ってアップロードすることができます。

環境はPythonのスタンダードでデータとしてフリーの住所データを使用しています。
Kind名はaddress プロパティはbango,ken,shiku,chousonの４つでいずれもstring（文字列）

### まずはapp.yamlに以下を記述
---

```yaml
builtins:
- remote_api: on
```

### bulkloader.yamlを生成
---

```
appcfg.py create_bulkloader_config --filename=bulkloader.yaml --application=アプリ名 .
```

このコマンドでbulkloader.yamlがアプリのディレクトリに生成される。
生成されたbulkloader.yamlの`transformers:`以下を編集する。

```yaml

- kind: address #kind名
  connector: csv # csvを使用するのでcsvと記述.
  connector_options:
    # TODO: Add connector options here--these are specific to each connector.
  property_map:
    - property: bango
      external_name: bango
    - property: chouson
      external_name: chouson
    - property: shiku
      external_name: shiku
    - property: ken
      external_name: ken

```

### csvファイルを用意
---

csvファイルのヘッダがプロパティと同じか確認。

```txt
bango,ken,shiku,chouson
230-0033,神奈川県,横浜市鶴見区,朝日町
230-0035,神奈川県,横浜市鶴見区,安善町
```

### アップロード
---

以下のコマンドでデータストアにアップロード

```
appcfg.py upload_data --config_file=bulkloader.yaml --application=アプリ名 --filename=upload.csv --kind=address .
```

アプリ名とcsvファイルとkindは適宜変えてください。

これでアップロード出来るのですが、件数が多いとエラーが出てすべてを登録できなかった。

```
[ERROR   ] Error in WorkerThread-4: [Errno 10057] ソケットが接続されていないか、sendto 呼び出しを使ってデータグラム ソケットで送信するときにア
ドレスが指定されていないため、データの送受信を要求することは禁じられています。
[ERROR   ] Error in WorkerThread-5:
[ERROR   ] Error in WorkerThread-6: getaddrinfo returns an empty list
[ERROR   ] Error in WorkerThread-7: corrupted
[ERROR   ] Error in WorkerThread-8: corrupted
[ERROR   ] Error in WorkerThread-9: corrupted

[INFO    ] 1420 entities total, 0 previously transferred
[INFO    ] 310 entities (232988 bytes) transferred in 253.7 seconds
[INFO    ] Some entities not successfully transferred
```

[ここ](https://stackoverflow.com/questions/5466900/google-app-engine-bulkloader-unexpected-thread-death)を参考にして、`--batch_size=1000`と`--rps_limit=500`を追加

```
appcfg.py upload_data --config_file=bulkloader.yaml --application=アプリ名 --filename=upload.csv --kind=address --batch_size=1000 --rps_limit=500 .
```



```
[INFO    ] Connecting to xxxxxx.appspot.com/_ah/remote_api
[INFO    ] Starting import; maximum 1000 entities per post
.[INFO    ] [WorkerThread-0] Backing off due to errors: 1.0 seconds
.[INFO    ] [WorkerThread-0] Backing off due to errors: 2.0 seconds
.[INFO    ] [WorkerThread-0] Backing off due to errors: 4.0 seconds
.[INFO    ] [WorkerThread-0] Backing off due to errors: 4.0 seconds

[INFO    ] 3240 entities total, 0 previously transferred
[INFO    ] 3240 entities (2175312 bytes) transferred in 127.9 seconds
[INFO    ] All entities successfully transferred
```

今度は3240件全て登録できました。2分かかるのは僕の使用する低スペPCが原因なのかどうかは分かりません（笑）

ちなみに、データストアのデータをcsvファイルとしてダウンロードする場合は、

```
appcfg.py download_data --application=アプリ名 --config_file=bulkloader.yaml --filename=dl.csv --kind=address --batch_size=1000 --rps_limit=500 .
```

で取得できます。`appcfg.py upload_data`を`appcfg.py download_data`に変更して、csvファイル名はディレクトリに同じファイルがあるとエラーが出るので適当な名前を付けるとcsvファイルをダウンロードできます。

```
[INFO    ] Connecting to xxxxxx.appspot.com/_ah/remote_api
[INFO    ] Downloading kinds: ['address']
.[INFO    ] address: No descending index on __key__, performing serial download
...
[INFO    ] Have 3240 entities, 0 previously transferred
[INFO    ] 3240 entities (728547 bytes) transferred in 21.4 seconds
```

21秒で完了。