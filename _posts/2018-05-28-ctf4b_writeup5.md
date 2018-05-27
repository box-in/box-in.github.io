---
layout: post
title: write-up ctf4b [Warmup] condition
date: 2018-05-28
category: "CTF"
tag: [CTF, reversing, IDA, pwn]
permalink: CTF_5
---

CTF終了後に[@kusano_k]さんのwrite-upを参考にさせて頂いたものです。

### おまけ [Warmup] condition

競技中は`0xDEADBEEF`の渡し方が分からなかったが
バッファオーバーフローを使えば良いらしい。
その前提でアセンブリを改めて眺めてみる。

```
var_30= byte ptr -30h
var_4= dword ptr -4

push    rbp
mov     rbp, rsp
sub     rsp, 30h
mov     [rbp+var_4], 0
mov     edi, offset format ; "Please tell me your name..."
mov     eax, 0
call    _printf
lea     rax, [rbp+var_30]
mov     rdi, rax
mov     eax, 0
call    _gets
cmp     [rbp+var_4], 0DEADBEEFh
jnz     short loc_4007BF
```

1. `sub rsp, 30h`なのでスタック領域が0x30byte確保されている。
2. `cmp [rbp+var_4], 0DEADBEEFh`なので
0x04の位置に`0xDEADBEEF`が存在すればFLAGを入手できる。
3. `call _gets`に渡されるバッファアドレスは
`lea rax, [rbp+var_30]`で指定されている通り0x30なので
0x30-0x04分のオフセットを考慮する必要がある。

![deadbeef](/assets/images/post/deadbeef.png)

この条件を踏まえたコードでdeadbeefの受け渡しに成功した。
[リトルエンディアン]なのでDEADBEEFはEFBEADDEにする必要がある。
<script src="https://gist.github.com/box-in/2ae5dc91dd3a4cb942838114f91f4cbe.js"></script>
```
b'OK! You have permission to get flag!!'
b'\nctf4b{T4mp3r_4n07h3r_v4r14bl3_w17h_m3m0ry_c0rrup710n}\n'
```
### まとめ

- FLAGはnctf4b{T4mp3r_4n07h3r_v4r14bl3_w17h_m3m0ry_c0rrup710n}
- スタックの状態を想像するのが解決の糸口

[@kusano_k]:https://qiita.com/kusano_k/items/9b74398f5bed2792d736
[リトルエンディアン]:https://ja.wikipedia.org/wiki/エンディアン#リトルエンディアン
