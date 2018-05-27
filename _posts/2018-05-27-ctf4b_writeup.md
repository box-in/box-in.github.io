---
layout: post
title: write-up ctf4b [Warmup] Simple Auth
date: 2018-05-27
category: "CTF"
tag: [CTF, reversing, IDA]
permalink: CTF_1
---
### 目的

2018-05-26に開催された[SECCON Beginners CTF 2018]のReversingカテゴリである
[Warmup] Simple Authのwrite-upを通じてバイナリ解析に対する理解を定着させる

### 全容を把握する

バイナリの全容を把握するためにフリーウェア版の[IDA]を使う。
バイナリ種別を自動的に判別してくれるのでELF64として開く。

![img1](/assets/images/post/ida.png)

グラフビューによるとmain関数からシンプルな条件分岐を経て終了するフローが見える。
左の処理には`Auth Complite!!`の文字列と`flag is %s`の文字列が見えるので
そちらに分岐する条件を探るのが目標になる。

![img2](/assets/images/post/reversing1.png)

条件分岐の直前の処理を解読したものは以下の通り。
authサブルーチンの結果が1になるような文字列を標準入力に渡せばよい。
authサブルーチンを解読する前に、文字列のアドレスが`rdi`レジスタに入っている事と
戻り値が`eax`レジスタに入る事を覚えておく。
```
call  __isoc99_scanf    ; 標準入力から読み取る
lea   rax, [rbp+var_30] ; 読み取った値のアドレスをraxレジスタへ格納する
mov   rdi, rax          ; 引数用のrdiレジスタにアドレスを移し替える
call  auth              ; authサブルーチンを実行する
cmp   eax, 1            ; 戻り値用のeaxレジスタ内の値と1を比較する
jnz   short loc_40081   ; 直前のcmpの結果がnot equalならloc_40081へジャンプする
```

### authサブルーチンを解読する

最初の3行によると
このサブルーチンはローカル変数用に0x50byteのスタックを確保している。
引数として渡された文字列はローカル変数の`[rbp+s]`に代入されている。
`[rbp+var_30]`から`[rbp+var_18]`までmovが連続しており
何かの文字列を初期化していると予想出来る。
少し離れたアドレス`[rbp+var_40]`に格納されている値は
後の処理を読むと分かるがauthサブルーチンの戻り値である。

![img3](/assets/images/post/reversing2.png)

文字列と思われるhexはasciiコードとして読むとFLAGであることが分かる。

```
63746634627B726576337273696E675F70347373773072647D
↓
ctf4b{rev3rsing_p4ssw0rd}
```

FLAGは入手出来たが更に先を読んでいく。
ここでは文字列の内容には触れず長さだけを比較して
長さが異なる場合は`loc_400772`へジャンプしている。
長さが同じ場合はジャンプせずそのまま左の処理に移行する。

```
mov   rax, [rbp+s]       ; 入力した文字列のアドレス[rbp+s]をraxレジスタに格納する
mov   rdi, rax           ; 引数用のrdiレジスタに移し替える
call  _strlen            ; 文字列の長さを取得する
mov   [rbp+var_38], eax  ; 文字列長をeaxレジスタから[rbp+var_38]に格納する

lea   rax, [rbp+var_30]  ; FLAGのアドレス[rbp+var_30]をraxレジスタに格納する
mov   rdi, rax           ; 引数用のrdiレジスタに移し替える
call  _strlen            ; FLAGの長さを取得する
mov   [rbp+var_34], eax  ; 文字列長をeaxレジスタから[rbp+var_34]に格納する

mov   eax, [rbp+var_38]  ; 入力文字の長さ[rbp+var_38]をeaxレジスタに格納する
cmp   eax, [rbp+var_34]  ; 入力文字の長さとFLAGの長さ[rbp+var_34]を比較する
jnz   short loc_400772   ; cmpの結果がnot equalならloc_400772へ移動する
```
条件分岐の先の処理は以下の通り。
`loc_400772`では`[rbp+var_40]`に0が格納され、
`loc_400779`でその値が`eax`を経由して呼び出し元に戻される。
戻り値を1にするためには`[rbp+var_40]`に0を格納しない左端の処理を通らなければならない。

![img4](/assets/images/post/reversing3.png)

`loc_400768`を正解のルートで抜けるには
0で初期化された`[rbp+var_3C]`を入力された文字列の長さ`[rbp+var_38]`と同じ値にする必要がある。
この条件を満たすまでは`loc_400738`で文字の比較が行われ
失敗した時点で`loc_40075F`を経由して戻り値が0になる。
成功した場合は`[rbp+var_3C]`に1が加算されて`loc_400768`に戻るので最終的には正解ルートに抜けることが出来る。

### まとめ

- FLAGはctf4b{rev3rsing_p4ssw0rd}
- 前提としてhello world出来るぐらいのC言語力は必要
- スタック、レジスタ、命令10個程度が分かれば読み進められる
- バイナリ初心者でもソースコードが想像できる良い問題だと思う

### バイナリ初心者になる為に参考にした本

- [たのしいバイナリの歩き方]
- [リバースエンジアリングバイブル]

[たのしいバイナリの歩き方]:https://www.amazon.co.jp/dp/4774159182
[リバースエンジアリングバイブル]:https://www.amazon.co.jp/dp/4844334794
[SECCON Beginners CTF 2018]:https://2017.seccon.jp/news/seccon-beginners-ctf-2018.html
[IDA]:https://www.hex-rays.com/products/ida/support/download_freeware.shtml
