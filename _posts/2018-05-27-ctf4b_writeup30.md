---
layout: post
title: write-up ctf4b [Warmup] plain mail
date: 2018-05-27
category: "CTF"
tag: [CTF, wireshark, smtp]
permalink: CTF_4
---

### パケットを確認する

与えられた`packet.pcap`からフラグを探す問題。
[Wireshark]でpcapを開くとメールサーバにデータを送っているのが見える。

![ws1](/assets/images/post/wireshark.png)

Wiresharkのツールバーから`分析 > 追跡 > TCPストリーム`で確認すると3通のメールを送っている事が分かる

1通目
```
220 67289bb1f069 ESMTP Exim 4.84_2 Fri, 27 Apr 2018 11:00:38 +0000
ehlo [172.19.0.3]
250-67289bb1f069 Hello client.4b [172.19.0.3]
250-SIZE 52428800
250-8BITMIME
250-PIPELINING
250 HELP
mail FROM:<me@4b.local> size=103
250 OK
rcpt TO:<you@4b.local>
250 Accepted
data
354 Enter message, ending with "." on a line by itself
I will send secret information. First, I will send encrypted file. Second, I wll send you the password.
.
250 OK id=1fC17G-00005T-T0
421 67289bb1f069 lost input connection
```
2通目
```
220 67289bb1f069 ESMTP Exim 4.84_2 Fri, 27 Apr 2018 11:00:40 +0000
ehlo [172.19.0.3]
250-67289bb1f069 Hello client.4b [172.19.0.3]
250-SIZE 52428800
250-8BITMIME
250-PIPELINING
250 HELP
mail FROM:<me@4b.local> size=658
250 OK
rcpt TO:<you@4b.local>
250 Accepted
data
354 Enter message, ending with "." on a line by itself
Content-Type: multipart/mixed; boundary="===============0309142026791669022=="
MIME-Version: 1.0
Content-Disposition: attachment; filename="encrypted.zip"

--===============0309142026791669022==
Content-Type: application/octet-stream; Name="encrypted.zip"
MIME-Version: 1.0
Content-Transfer-Encoding: base64

UEsDBAoACQAAAOJVm0zEdBgeLQAAACEAAAAIABwAZmxhZy50eHRVVAkAA6f/4lqn/+JadXgLAAEE
AAAAAAQAAAAASsSD0p8jUFIaCtIY0yp4JcP9Nha32VYd2BSwNTG83tIdZyU4x2VJTGyLcFquUEsH
CMR0GB4tAAAAIQAAAFBLAQIeAwoACQAAAOJVm0zEdBgeLQAAACEAAAAIABgAAAAAAAEAAACkgQAA
AABmbGFnLnR4dFVUBQADp//iWnV4CwABBAAAAAAEAAAAAFBLBQYAAAAAAQABAE4AAAB/AAAAAAA=
--===============0309142026791669022==--
.
250 OK id=1fC17I-00005a-Fw
421 67289bb1f069 lost input connection
```
3通目
```
220 67289bb1f069 ESMTP Exim 4.84_2 Fri, 27 Apr 2018 11:00:42 +0000
ehlo [172.19.0.3]
250-67289bb1f069 Hello client.4b [172.19.0.3]
250-SIZE 52428800
250-8BITMIME
250-PIPELINING
250 HELP
mail FROM:<me@4b.local> size=13
250 OK
rcpt TO:<you@4b.local>
250 Accepted
data
354 Enter message, ending with "." on a line by itself
_you_are_pro_
.
250 OK id=1fC17K-00005h-AC
421 67289bb1f069 lost input connection
```

本文中にFLAGらしき文字列は見当たらないが
2通目にencrypted.zipが添付されているのでこれを解凍するのだと予想出来る。

### メールデータを抽出する

メールに限らずコンピュータ上のデータはただの文字列である。
メールサーバから受け取った文字列をメールクライアントが解釈して
差出人やメール本文や添付ファイルをユーザーに使いやすい形で提供してくれている。

添付ファイルをzipの形で扱うために今回もメールクライアントに頑張ってもらう。
TCPストリーム(tcp.stream eq 2)を開いて`Save as`で適当な名前のemlファイルとして保存する。

![ws2](/assets/images/post/saveas_eml.png)

[Becky! Internet Mail]などのemlを解釈できるアプリケーションでインポートすれば添付ファイルにアクセスできる。

![ws3](/assets/images/post/becky.png)

3通目に書かれていた`_you_are_pro_`でzipを解凍するとflag.txtが手に入る。

### まとめ

- FLAGはctf4b{email_with_encrypted_file}
- データが揃っていれば適切なアプリケーションを使って閲覧できる

> 企業のインフラサポートをやっていたころ
> トラブルシューティングの一環でpcapを貰って分析することが多々あった。
> 当時はこうした小技を知らなかったけど
> もしかしたら機密メールも含まれていたのかも。
> pcapを取得する時は目的外通信を取らないようにフィルタをしっかり。

[Becky! Internet Mail]: http://www.rimarts.co.jp/becky-j.htm
[Wireshark]: https://www.wireshark.org/#download
