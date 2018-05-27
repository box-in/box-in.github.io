---
layout: post
title: write-up ctf4b [Warmup] Veni, vidi, vici
date: 2018-05-27
category: "CTF"
tag: [CTF, crypto, python]
permalink: CTF_2
---
### 前置き

Cryptoカテゴリのwarmup課題では3つの暗号文を解く。

- part1: `Gur svefg cneg bs gur synt vf: pgs4o{a0zber`
- part2: `Lzw kwugfv hsjl gx lzw xdsy ak: _uDskk!usd_u`
- part3: `{ʎɥdɐɹɓ0ʇdʎᴚ :sı ɓɐlɟ ǝɥʇ ɟo ʇɹɐd pɹıɥʇ ǝɥ⊥`

最初はサクラエディタで作業していたのでpart3が文字化けして無駄に悩んでしまった。
windows標準notepadやatomエディタで作業すること。

- part3: `{ﾊ舎･dﾉ惜ｹﾉ・ﾊ㌧ﾊ若ｴ・:sﾄｱ ﾉ橡人ﾉ・ﾇ數･ﾊ・ﾉ殪 ﾊ・ｹﾉ薪 pﾉｹﾄｱﾉ･ﾊ・ﾇ數･⊥`

### シフト数を把握する

part1とpart2は見るからに[シーザー暗号]なのでシフト数を探す。
まずはシンプルな13文字右シフト。
```
- 換え字表
abcdefghijklmnopqrstuvwxyz
nopqrstuvwxyzabcdefghijklm
- 暗号文
Gur svefg cneg bs gur synt vf: pgs4o{a0zber
- 平文
The first part of the flag is: ctf4b{n0more
```

このシフト数だとpart2は解読できないが
単語の位置や文字数が同じなので平文の前半は共通だと予想出来る。
`The > Lzw`に変換するためにはアルファベットを8文字左シフトすれば良いので
以下の通り解読できる。
```
- 換え字表
abcdefghijklmnopqrstuvwxyz
stuvwxyzabcdefghijklmnopqr
- 暗号文
Lzw kwugfv hsjl gx lzw xdsy ak: _uDskk!usd_u
- 平文
the second part of the flag is: _class!cal_c
```

### 解読機を作る

以上のルールに沿って解読機を作る。
<script src="https://gist.github.com/box-in/5226ba90d86be79d6437f666ec0bad4b.js"></script>
part1はシンプルな13文字右シフトだったが
part2は8文字左シフトであることが手作業で判明している。
両シフトに対応できるように換え字テーブルを3枚並べて
中央のテーブルから左右にシフトできるようにした。
![table](/assets/images/post/rot_table.png)
> CTF中は気付かなかったが
26文字に対して13文字の換え字変換は右シフトも左シフトも同じ結果になる

### part3は回転させる

最初は文字化けしていたので
ASCIIコードを加工したり色々な文字コードを試していたが、
正しい文字コードで表示すれば180°回転させた文字列であることが分かる
![rotate](/assets/images/post/rotate.png)

### まとめ

- FLAGはctf4b{n0more_class!cal_cRypt0graphy}
- 文字コードを適切に処理できるエディタを使うことが重要

### 参考書籍

- [暗号技術入門]

[暗号技術入門]:https://www.amazon.co.jp/dp/B015643CPE
[シーザー暗号]:https://ja.wikipedia.org/wiki/シーザー暗号
