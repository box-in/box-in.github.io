---
layout: post
title: DionaeaのDisk容量節約
date: 2018-05-06
category: honeypot
tag: [Dionaea, WannaCry, T-Pot]
permalink: Dionaea_1
---

　Dionaeaは送り込まれたマルウェアを逐一保存してくれるが、
グローバルIPで運用していると分単位でマルウェアが送られてくるので、
気付くと1GBを超えるマルウェアの群れを前に途方に暮れる事になる。
そんな事にならないように既知のマルウェアを除外するスクリプトを設定する。

　まずは保存された検体の正体を特定する必要がある。
T-PotでDionaeaを動かしている場合`/data/Dionaea/binaries/`がマルウェアの保存先になる。
ファイル名がMD5になった検体が並んでいるのでファイル名から検体を特定していく。

```
$ sudo ls -la /data/Dionaea/binaries
total 10728
drwxrw---- 2 tpot tpot    4096 May  6 20:54 .
drwxrw---- 7 tpot tpot    4096 May  6 03:29 ..
-rw------- 1 tpot tpot 5267459 May  6 20:19 0ab2aeda90221832167e5127332dd702
-rw------- 1 tpot tpot   70659 May  6 06:13 24899e33d1f6f7d1dbb4ecb458c4f057
-rw------- 1 tpot tpot   79875 May  6 07:23 60351ac08de4e1a27457407f3883b083
-rw------- 1 tpot tpot 5267459 May  6 20:40 62186bebffffcfafb1c70a8ff03fa317
-rw------- 1 tpot tpot   82435 May  6 06:30 842133ddc2d57fd0f78491b7ba39a34d
-rw------- 1 tpot tpot    5696 May  6 20:54 844290834b6450425b146d4517cdf780
-rw------- 1 tpot tpot    8040 May  6 20:40 ab27f6c7634e9efc13fb2db29216a0a8
-rw------- 1 tpot tpot    3923 May  6 20:54 dcc00237655c2a08646cabbc3a12f739
-rw------- 1 tpot tpot   84483 May  6 16:14 e7e171cdde7e3f39bf86e1d96f5ce433
-rw------- 1 tpot tpot   83459 May  6 07:22 f63e34b172bc6c88c002a2d25c738ea9
```

　今回の目的は既知のマルウェアの除外に加えてDiskスペースを節約する事なので5267459byteのファイルを見ていく。
- `0ab2aeda90221832167e5127332dd702`をVirusTotalで[照合した結果][1]{:target="_blank"}
WannaCryであると判定された。
- `62186bebffffcfafb1c70a8ff03fa317`をVirusTotalで[照合した結果][2]{:target="_blank"}
も同様だった。

実はWannaCryのサイズが5267459byteであるというのは既知らしい。
例えばnProtectの検出名などはRansom/W32.WannaCry.5267459になっている。

　ファイルサイズを指定して定期的に掃除すれば良いことが分かったのでスクリプトを設定していく。
T-Potは自身が侵害された場合に備えて定期的に設定をリセットするが`/home/`や`/etc/`はその限りではない。
`/home/.sh/`にスクリプトを置いて`/etc/crontab`から呼び出せば良い。

```
$ cat /home/tsec/.sh/clean_Dionaea_log.sh 
#!/bin/sh

find /data/Dionaea/binaries -size 5267459c | xargs rm -v
```

```
$ sudo tail -3 /etc/crontab
# custom my script
10 * * * *      root    /bin/sh /home/tsec/.sh/clean_Dionaea_log.sh
```

　この設定で大幅にDiskスペースを節約できるようになった。
WannaCryを除去する前にDionaeaがログローテーションしてしまう事もあるが
2週間分の検体が150MB程度に収まっている。

```
F:\tool\Dionaea>dir
2018/04/27  03:31                 0 binaries.tgz
2018/04/27  03:30        14,442,481 binaries.tgz.1.gz
2018/04/19  03:30        10,934,264 binaries.tgz.10.gz
2018/04/18  03:30        12,459,089 binaries.tgz.11.gz
2018/04/17  03:30         6,419,932 binaries.tgz.12.gz
2018/04/16  03:30           389,259 binaries.tgz.13.gz
2018/04/15  03:28           282,529 binaries.tgz.14.gz
2018/04/14  16:29        10,872,222 binaries.tgz.15.gz
2018/04/14  03:28         7,441,821 binaries.tgz.16.gz
2018/04/26  03:30        15,832,041 binaries.tgz.2.gz
2018/04/25  03:30         4,131,606 binaries.tgz.3.gz
2018/04/24  03:30        17,797,322 binaries.tgz.4.gz
2018/04/23  03:30        13,446,152 binaries.tgz.5.gz
2018/04/22  03:28        16,243,866 binaries.tgz.6.gz
2018/04/21  16:29         7,990,460 binaries.tgz.7.gz
2018/04/21  03:30         7,563,262 binaries.tgz.8.gz
2018/04/20  03:30         8,647,291 binaries.tgz.9.gz
              17 個のファイル         154,893,597 バイト
```

[1]: https://www.virustotal.com/#/file/64bb708b31b4b043018457c1098465ea83da7d6408c7029b2f68c333fc25891c/detection
[2]: https://www.virustotal.com/#/file/1dd001ef5ae3fdc07d44feae5246b23f199e01ce4f7e2b7dd5a354f7aea227fa/detection
