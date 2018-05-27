---
layout: post
title: write-up ctf4b てけいさんえくすとりーむず
date: 2018-05-27
category: "CTF"
tag: [CTF, python]
permalink: CTF_3
---
### 環境確認

問題サーバから出題された問題に解答するとFLAGがもらえるタイプ。
curlでアクセスしてみると出題される式はランダムかつ連続で100問正解する必要がある事が分かる。
問題を先に入手して解答しておくことは出来ない。

```
$ curl tekeisan-ekusutoriim.chall.beginners.seccon.jp:8690

Welcome to TEKEISAN for Beginners -extreme edition-
---------------------------------------------------------------
Please calculate. You need to answered 100 times.
e.g.
(Stage.1)
4 + 5 = 9
...
(Stage.99)
4 * 4 = 869
[!!] Wrong, see you.
---------------------------------------------------------------
(Stage.1)
803 - 561 = [!!] Wrong, see you.
```

### クライアントを作る

必要な機能は以下の通り。
- サーバと会話出来るようにする
- レスポンス末尾を切り出して式として解釈する
- 解答をサーバに渡す
- 最終的に得られるFLAGを確認する

<script src="https://gist.github.com/box-in/a6e74d751f9ee7505ee5e8ff00c2902c.js"></script>

低レベルな通信をするために[socketモジュール]を利用した。
普段はスクレイピングでrequestsモジュールばかり使っていたので苦労した。
最終的にサーバから返されたメッセージは以下の通り。

`b'Congrats.\nFlag is: "ctf4b{ekusutori-mu>tekeisann>bigina-zu>2018}"\n'`

### まとめ

- FLAGはctf4b{ekusutori-mu>tekeisann>bigina-zu>2018}
- CTFするならソケットプログラミングの知識はあった方が良い

[socketモジュール]: https://docs.python.org/ja/3.5/library/socket.html
