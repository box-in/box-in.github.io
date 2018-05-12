---
layout: post
title: Dionaeaのバイナリ選別
date: 2018-05-12
category: honeypot
tag: [Dionaea, T-Pot, Slack, VirusTotal]
permalink: Dionaea_2
---

### Dionaeaを運用する時は未知のバイナリに着目する

既知のバイナリを解析する必要は無い。
解析演習用のバイナリが欲しければHybrid-Analysisなどのサービスを利用すれば良い。
それらのサービスから入手できない未知のバイナリだけをDionaeaから採取すべきだと考える。

Dionaeaの別の運用目的として、既知の検体がどの程度着弾しているのか全体の傾向を掴むのは有益だと思う。
それはElasticStackに任せる事になるのでバイナリを保管する必要はない。

---
### 未知のバイナリを入手する

Dionaeaが採取したバイナリは全てbinariesディレクトリに保存される。
既知のバイナリも、ポリモーフィックなバイナリも、未知のバイナリも全て。
ここから未知のバイナリだけを解析する手順は次の通り。

1. dionaeaが取得したバイナリ一覧を確認する
2. ハッシュをVirusTotalに投げて種類を判別する
3. 未知の検体を解析環境にgetする
4. 解析する

---
### 楽をする

4以外はやりたくないので
VirusTotaに照合した結果に応じてバイナリを抽出する仕組みを作る。

#### 1. ポリモーフィックなバイナリを除外する

一例として[前回の記事](Dionaea_1)参照

#### 2. バイナリ一覧を取得する

既存のスクリプトの影響で`/data/dionaea/binaries/`の中身は逐一変化する。
1時間毎にwannacryを除去し、毎日3:27にバイナリをアーカイブしている。
バイナリ一覧を取るのにこれでは都合が悪いので、状態が固定されている`/data/dionaea/binaries.tgz.1.gz`を参照する。
`vtscan.py`はファイル一覧を受け取ってVirusTotalと照合するスクリプト。
`/home/tsec/artifact/`は未知のバイナリを抽出するディレクトリ。

```sh
$ cat sendto_slack.sh
#!/bin/sh
input=/data/dionaea/binaries.tgz.1.gz
output=/home/tsec/artifact/
if [ ! -d $output ]; then
    mkdir $output
fi
zcat $input | tar ztv | cut -d "/" -f 5 | xargs python vtscan.py
```

03:27に環境がリブートされてバイナリがアーカイブされるため、今回のスクリプトは04:27にセットしておく

```
$ tail /etc/crontab
# Daily reboot
27 3 * * * root reboot

# Check for updated packages every sunday, upgrade and reboot
27 16 * * 0 root apt-get autoclean -y && apt-get autoremove -y && apt-get update -y && apt-get upgrade -y && sleep 10 && reboot

# custom my script
10 * * * * root /bin/sh /home/tsec/.sh/clean_dionaea_log.sh
27 4 * * * root /bin/sh /home/tsec/.sh/sendto_slack.sh
```

#### 3. VirusTotalに照合する

VirusTotal APIを使ってファイル名（ハッシュ）をPOSTして、見るべきものが有った時だけSlackへ通知する。
該当のファイルは解析用に抽出しておく。

```python
$cat vtscan.py
import requests
import sys
import time
import json
import subprocess

apikey = Your VirusTotal API Key
slack_url = Your Slack WebHook Address
headers = {"Accept-Encoding": "gzip, deflate"}
vt_url = 'https://www.virustotal.com/vtapi/v2/file/report'
input_tgz = "/data/dionaea/binaries.tgz.1.gz"
output = "/home/tsec/artifact/"

def sendto_slack(message):
    parameters = {
        "channel": "#honeypot",
        "username": "HeneyBot",
        "icon_emoji": ":virustotal:",
        "text": message
        }
    requests.post(slack_url, data=json.dumps(parameters))

def get_message(json):
    result = ""
    if json["response_code"] == 0 or json["positives"] == 0:
        command = "gzip -dc " + input_tgz + "| tar -C " + output + " -zx data/dionaea/binaries/" + json["resource"]
        subprocess.call(command, shell=True)
        result = json["resource"] + " is unknown artifact"
    return result

if __name__ == "__main__":
    args = sys.argv
    result = ""
    if(len(args) < 2):
        print("please put suspicious hash's at least 1.")
        sys.exit(1)
    for hash in args[1:]:
        time.sleep(25)
        parameters = {"resource": hash, "apikey": apikey}
        response = requests.get(vt_url, params=parameters, headers=headers)
        if response.status_code == 200:
            result += get_message(response.json()) + "\n"
    if len(result) > 0:
        sendto_slack(result)
```

---
### まとめ

slackから通知が来るまでDionaeaのことは考えなくて良くなった。
通知が来たら使い慣れたツールでバイナリをダウンロードして解析作業に入れば良い。
解析端末に送りつけることも可能だと思うがやめておこう。
![slack](/assets/images/post/slack.png)
![winscp](/assets/images/post/winscp.png)

抽出しなかったバイナリは使わないので削除しても良いかもしれない。ハニーポットの容量を更に節約できる
