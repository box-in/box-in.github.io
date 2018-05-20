---
layout: post
title: T-Potに新しいログを追加する
date: 2018-05-21
category: "honeypot"
tag: [Honeypot, Logstash, Docker, T-Pot, Kibana]
permalink: Logstash_1
---

## 目的

T-PotのWebコンソールではDionaeaに届いたバイナリの分析結果が見れない。
Dionaeaの分析ではバイナリに着目したいのでダッシュボードにバイナリの情報を追加する。

## 前提

追加する情報はVirusTotalから取得する。
[Dionaeaのバイナリ選別](Dionaea_2)で書いたpythonスクリプトを少し改修して`/data/dionaea/log/virustotal.log`に未加工のjsonを出力する。
このjsonをT-Potで扱えるようにする。

改修箇所
```python
if response.status_code == 200:
  result += get_message(response.json()) + "\n"
  with open("/data/dionaea/log/virustotal.log", mode = "a") as log:
    log.write(response.text + "\n")
```

json
```json
{"scans": {"Bkav": {"detected": true, "version": "1.3.0.9466", "result": "W32.CoinMinerSimND.Worm", "update": "20180518"}, "MicroWorld-eScan": {"detected": true, "version": "14.0.297.0", "result": "Gen:Variant.Zusy.250227", "update": "20180520"}, "nProtect": {"detected": false, "version": "2018-05-20.02", "result": null, "update": "20180520"}, "CMC": {"detected": false, "version": "1.1.0.977", "result": null, "update": "20180520"}, "CAT-QuickHeal": {"detected": true, "version": "14.00", "result": "Trojan.IGENERIC", "update": "20180519"}, "ALYac": {"detected": true, "version": "1.1.1.5", "result": "Trojan.Downloader.83459R", "update": "20180520"}, "Cylance": {"detected": true, "version": "2.3.1.101", "result": "Unsafe", "update": "20180520"}, "Zillya": {"detected": true, "version": "2.0.0.3556", "result": "Backdoor.Agent.Win32.64934", "update": "20180519"}, "SUPERAntiSpyware": {"detected": false, "version": "5.6.0.1032", "result": null, "update": "20180520"}, "TheHacker": {"detected": true, "version": "6.8.0.5.2838", "result": "Trojan/Agent.snp", "update": "20180516"}, "K7GW": {"detected": true, "version": "10.47.27195", "result": "Trojan ( 0026ce8d1 )", "update": "20180520"}, "K7AntiVirus": {"detected": true, "version": "10.47.27196", "result": "Trojan ( 0026ce8d1 )", "update": "20180520"}, "Arcabit": {"detected": true, "version": "1.0.0.831", "result": "Trojan.Zusy.D3D173", "update": "20180520"}, "TrendMicro": {"detected": true, "version": "10.0.0.1040", "result": "BKDR_FORSHARE.C", "update": "20180520"}, "Baidu": {"detected": false, "version": "1.0.0.2", "result": null, "update": "20180518"}, "Babable": {"detected": false, "version": "9107201", "result": null, "update": "20180406"}, "Cyren": {"detected": true, "version": "6.0.0.4", "result": "W32/Trojan.ADTW-4111", "update": "20180520"}, "Symantec": {"detected": true, "version": "1.6.0.0", "result": "Trojan.Gen.2", "update": "20180519"}, "TotalDefense": {"detected": false, "version": "37.1.62.1", "result": null, "update": "20180520"}, "TrendMicro-HouseCall": {"detected": true, "version": "9.950.0.1006", "result": "BKDR_FORSHARE.C", "update": "20180520"}, "Paloalto": {"detected": true, "version": "1.0", "result": "generic.ml", "update": "20180520"}, "ClamAV": {"detected": true, "version": "0.99.2.0", "result": "Win.Trojan.Agent-6437050-0", "update": "20180520"}, "VBA32": {"detected": true, "version": "3.12.31.0", "result": "Backdoor.Agent", "update": "20180518"}, "Kaspersky": {"detected": true, "version": "15.0.1.13", "result": "Backdoor.Win32.Agent.texqw", "update": "20180520"}, "BitDefender": {"detected": true, "version": "7.2", "result": "Gen:Variant.Zusy.250227", "update": "20180520"}, "NANO-Antivirus": {"detected": false, "version": "1.0.106.22618", "result": null, "update": "20180520"}, "AegisLab": {"detected": true, "version": "4.2", "result": "Gen.Variant.Zusy!c", "update": "20180520"}, "Avast": {"detected": true, "version": "18.4.3895.0", "result": "Win32:Malware-gen", "update": "20180520"}, "Tencent": {"detected": true, "version": "1.0.0.1", "result": "Win32.Backdoor.Agent.Ebqk", "update": "20180520"}, "Endgame": {"detected": true, "version": "2.1.2", "result": "malicious (high confidence)", "update": "20180507"}, "Emsisoft": {"detected": true, "version": "4.0.2.899", "result": "Gen:Variant.Zusy.250227 (B)", "update": "20180520"}, "Comodo": {"detected": true, "version": "29043", "result": ".UnclassifiedMalware", "update": "20180520"}, "F-Secure": {"detected": true, "version": "11.0.19100.45", "result": "Gen:Variant.Zusy.250227", "update": "20180520"}, "DrWeb": {"detected": true, "version": "7.0.28.2020", "result": "Trojan.MulDrop7.29574", "update": "20180520"}, "VIPRE": {"detected": true, "version": "66806", "result": "Trojan.Win32.Generic!BT", "update": "20180520"}, "Invincea": {"detected": true, "version": "6.3.4.26036", "result": "heuristic", "update": "20180503"}, "McAfee-GW-Edition": {"detected": true, "version": "v2017.2786", "result": "Artemis!Trojan", "update": "20180520"}, "Sophos": {"detected": true, "version": "4.98.0", "result": "Troj/Agent-AYFU", "update": "20180520"}, "SentinelOne": {"detected": false, "version": "1.0.15.206", "result": null, "update": "20180225"}, "F-Prot": {"detected": false, "version": "4.7.1.166", "result": null, "update": "20180520"}, "Jiangmin": {"detected": true, "version": "16.0.100", "result": "Backdoor.Agent.amk", "update": "20180520"}, "Webroot": {"detected": true, "version": "1.0.0.403", "result": "W32.Trojan.Gen", "update": "20180520"}, "Avira": {"detected": true, "version": "8.3.3.6", "result": "TR/Downloader.gkqlp", "update": "20180520"}, "Fortinet": {"detected": true, "version": "5.4.247.0", "result": "W32/Agent.SNP!tr", "update": "20180520"}, "Antiy-AVL": {"detected": true, "version": "3.0.0.1", "result": "Trojan/Win32.Agent", "update": "20180520"}, "Kingsoft": {"detected": false, "version": "2013.8.14.323", "result": null, "update": "20180520"}, "Microsoft": {"detected": true, "version": "1.1.14800.3", "result": "Trojan:Win32/Tiggre!rfn", "update": "20180520"}, "ViRobot": {"detected": true, "version": "2014.3.20.0", "result": "Trojan.Win32.Z.Zusy.83459.N", "update": "20180519"}, "ZoneAlarm": {"detected": true, "version": "1.0", "result": "Backdoor.Win32.Agent.texqw", "update": "20180520"}, "Avast-Mobile": {"detected": false, "version": "180519-04", "result": null, "update": "20180519"}, "AhnLab-V3": {"detected": true, "version": "3.12.1.20782", "result": "Backdoor/Win32.Agent.C2068678", "update": "20180519"}, "McAfee": {"detected": true, "version": "6.0.6.653", "result": "Artemis!F63E34B172BC", "update": "20180520"}, "AVware": {"detected": true, "version": "1.5.0.42", "result": "Trojan.Win32.Generic!BT", "update": "20180520"}, "MAX": {"detected": true, "version": "2017.11.15.1", "result": "malware (ai score=100)", "update": "20180520"}, "Ad-Aware": {"detected": true, "version": "3.0.5.370", "result": "Gen:Variant.Zusy.250227", "update": "20180520"}, "Malwarebytes": {"detected": true, "version": "2.1.1.1115", "result": "Trojan.Downloader", "update": "20180520"}, "Zoner": {"detected": false, "version": "1.0", "result": null, "update": "20180519"}, "ESET-NOD32": {"detected": true, "version": "17414", "result": "a variant of Win32/Agent.SNP", "update": "20180520"}, "Rising": {"detected": false, "version": "25.0.0.1", "result": null, "update": "20180520"}, "Yandex": {"detected": true, "version": "5.5.1.3", "result": "Backdoor.Agent!vhGMM6/sJk0", "update": "20180518"}, "Ikarus": {"detected": true, "version": "0.1.5.2", "result": "Trojan.Win32.Agent", "update": "20180519"}, "eGambit": {"detected": false, "version": null, "result": null, "update": "20180520"}, "GData": {"detected": true, "version": "A:25.17117B:25.12296", "result": "Gen:Variant.Zusy.250227", "update": "20180520"}, "AVG": {"detected": true, "version": "18.4.3895.0", "result": "Win32:Malware-gen", "update": "20180520"}, "Panda": {"detected": true, "version": "4.6.4.2", "result": "Trj/GdSda.A", "update": "20180520"}, "Qihoo-360": {"detected": true, "version": "1.0.0.1120", "result": "Win32/Backdoor.6ab", "update": "20180520"}}, "scan_id": "5e15c97546a19759a8397e51e98a2d8168e6e27aff4dc518220459ed3184e4e2-1526806316", "sha1": "368ef0af957492ad0b55ce1351da1b44f67dbcb8", "resource": "f63e34b172bc6c88c002a2d25c738ea9", "response_code": 1, "scan_date": "2018-05-20 08:51:56", "permalink": "https://www.virustotal.com/file/5e15c97546a19759a8397e51e98a2d8168e6e27aff4dc518220459ed3184e4e2/analysis/1526806316/", "verbose_msg": "Scan finished, information embedded", "total": 66, "positives": 52, "sha256": "5e15c97546a19759a8397e51e98a2d8168e6e27aff4dc518220459ed3184e4e2", "md5": "f63e34b172bc6c88c002a2d25c738ea9"}
```


## Logstashの設定を書き換える

T-Potのダッシュボードに表示される情報は次のような段階を踏んでいる。
1. 各種ハニーポットがログを書き出す
2. Logstashがログを収集・整形してElasticSearchに格納する
3. KibanaがElasticSearchの情報を可視化する

前提で述べた通り今回のログはvirustotal.logに用意してある。
Logstashでそれを読むためにはlogstash.confの変更とdocker imageのリビルドが必要になる。

### logstash.confの変更

Logstashによる収集・整形・格納の動作設定は
`/opt/tpot/docker/elk/logstash/dist/logstash.conf`
に記述されている。
virustotal.logを収集し、json解析、ElasticSearchに格納するために
inputブロックとfilterブロックに以下の設定を追加する。

inputブロック
```yml
# VirusTotal
  file {
    path => ["/data/dionaea/log/virustotal.log"]
    type => "VirusTotal"
  }
```
filterブロック
```yml
# VirusTotal
  if [type] == "VirusTotal" {
    json {
      source => "message"
    }
  }
```

### logstashのDockerイメージをリビルドする

T-PotはLogstashを含む各サービスをDockerコンテナとして動かしている。
コンテナイメージは事前にビルドされているので今編集したlogstash.confは読み込まれていない。
`docker images`すればlogstashが2週間前にビルドされたイメージであることが分かる。

```sh
$ docker images | head -5
REPOSITORY                TAG     IMAGE ID        CREATED            SIZE
dtagdevsec/kibana         1710    81b21a130b5a    2 weeks ago        234 MB
dtagdevsec/elasticsearch  1710    010748e212b0    2 weeks ago        129 MB
dtagdevsec/logstash       1710    a3ed2cf9d889    2 weeks ago        384 MB
dtagdevsec/cowrie         1710    838d1a736a1d    4 months ago       123 MB
```

変更を反映するにはコンテナイメージをビルドする必要がある。
logstashのイメージをビルドするために必要な情報は
`/opt/tpot/docker/elk/logstash/Dockerfile`
に記載されているので以下の通りコマンドを実行する。

```sh
$ docker build -t dtagdevsec/logstash:1710 /opt/tpot/docker/elk/logstash/
```

コンテナイメージが更新された事を確認する。

```sh
$ docker images | head -5
REPOSITORY                TAG     IMAGE ID        CREATED            SIZE
dtagdevsec/logstash       1710    a3ed2cf9d889    2 hours ago        668 MB
dtagdevsec/kibana         1710    81b21a130b5a    2 weeks ago        234 MB
dtagdevsec/elasticsearch  1710    010748e212b0    2 weeks ago        129 MB
dtagdevsec/cowrie         1710    838d1a736a1d    4 months ago       123 MB
```

tpotを再起動すればLogstash側の作業は完了。
```sh
$ sudo systemctl restart tpot
```

## Kibanaの設定を変更する

この状態で`virustotal.log`が更新されるとLogstashを通じてElasticSearchに新たな情報が格納される。
Kibanaから情報を活用するためにインデックスを更新しておく。
![kibana1](/assets/images/post/kibana.png)

インデックスを更新したのでVirusTotalの検出数やマルウェア判定名をピックアップできるようになった。
自分が分析しやすい項目を組み合わせて表やグラフを作っておく。
![kibana2](/assets/images/post/kibana_discover.png)
![kibana3](/assets/images/post/kibana_dashboard.png)
