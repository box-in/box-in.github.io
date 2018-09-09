---
layout: post
title: write-up CyberRebeatCTF
date: 2018-09-09
category: "CTF"
tag: [CTF, うさみみハリケーン, gdb, ILSpy]
permalink: CybetRebeatCTF
---

同人サークルE.N.Nachが主催する[CyberRebeat CTF](https://ennach.sakura.ne.jp/CyberRebeatCTF/index_jp.html)のwrite-upです.

## Binary
### SimpleBinary

与えられたバイナリファイルを一通り調べる.
```
# file SimpleBinary
SimpleBinary: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=98f51339e195ba048b503cf4e966d7351ed4b198, stripped

# binwalk SimpleBinary

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             ELF, 64-bit LSB executable, AMD x86-64, version 1 (SYSV)

# strings
特に意味のある文字列は見当たらない
```

気になる情報は見当たらないので動かしてみるしかなさそう.
```
# ./SimpleBinary AAAAAAAAAAAAAAA
何も表示されない
```

メモリ上にデータが乗っている可能性があるので解析していく.
```
# gdb SimpleBinary
gdb-peda$ start
[----------------------------------registers-----------------------------------]
RAX: 0x0
RBX: 0x0
RCX: 0x400740 (push   r15)
RDX: 0x1a
RSI: 0x1a
RDI: 0x7fffffffe590 ("{iCC .sith t'u Fn}dImTgaRh")
RBP: 0x7fffffffe560 --> 0x7fffffffe5c0 --> 0x0
RSP: 0x7fffffffe4d0 --> 0x1a00000000
RIP: 0x40056d (mov    DWORD PTR [rbp-0x70],0x3)
R8 : 0x7ffff7dd5e80 --> 0x0
R9 : 0x0
R10: 0x7fffffffe250 --> 0x0
R11: 0x7ffff7a30350 (<__libc_start_main>:       push   r14)
R12: 0x400450 (xor    ebp,ebp)
R13: 0x7fffffffe6a0 --> 0x1
R14: 0x0
R15: 0x0
EFLAGS: 0x246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x40055e:    mov    rax,QWORD PTR fs:0x28
   0x400567:    mov    QWORD PTR [rbp-0x8],rax
   0x40056b:    xor    eax,eax
=> 0x40056d:    mov    DWORD PTR [rbp-0x70],0x3
   0x400574:    mov    DWORD PTR [rbp-0x6c],0xb
   0x40057b:    mov    DWORD PTR [rbp-0x68],0x0
   0x400582:    mov    DWORD PTR [rbp-0x64],0x16
   0x400589:    mov    DWORD PTR [rbp-0x60],0xf
```
RDIに気になる文字列が展開されている.
トレースしていくと0x400630から0x40069aでループして文字列を操作しているようだ.
```
[-------------------------------------code-------------------------------------]
   0x40068b:    mov    BYTE PTR [rdx],al
   0x40068d:    add    DWORD PTR [rbp-0x74],0x1
   0x400691:    mov    eax,DWORD PTR [rbp-0x74]
=> 0x400694:    cmp    eax,DWORD PTR [rbp-0x8c]
   0x40069a:    jl     0x400630
   0x40069c:    nop
   0x40069d:    mov    rax,QWORD PTR [rbp-0x8]
   0x4006a1:    xor    rax,QWORD PTR fs:0x28
```
ループを抜けた0x40069cにブレークポイントを置いて一気に進める.
フラグを回収できた.
```
gdb-peda$ b *0x40069c
gdb-peda$ c
[-------------------------------------code-------------------------------------]
   0x400691:    mov    eax,DWORD PTR [rbp-0x74]
   0x400694:    cmp    eax,DWORD PTR [rbp-0x8c]
   0x40069a:    jl     0x400630
=> 0x40069c:    nop
   0x40069d:    mov    rax,QWORD PTR [rbp-0x8]
   0x4006a1:    xor    rax,QWORD PTR fs:0x28
   0x4006aa:    je     0x4006b1
   0x4006ac:    call   0x400420 <__stack_chk_fail@plt>
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe4d0 --> 0x1a00000000
0008| 0x7fffffffe4d8 --> 0x7fffffffe590 ("CRCTF{It's a humid night.}")
```
## Stegano
### Alpha

AlphaというタイトルのPNGが1枚渡される.
αチャンネルに何か仕込まれているだろうと予想してGIMPで開いてみる.
![](/assets/images/post/20180907/gimp2.png)
![](/assets/images/post/20180907/gimp1.png)
不透明度の大部分は253だが一部252の部分がある.
境界線を可視化できればフラグが入手できると予想されるが選択ツールで不透明度の境界を抽出できない（そもそも可能なのか？）
この手の問題解決でよく噂を聞く[うさみみハリケーン](https://www.vector.co.jp/soft/win95/prog/se375830.html)を使う
![](/assets/images/post/20180907/usamimi1.png)
![](/assets/images/post/20180907/usamimi2.png)
252と253の差はビット0の有無なのでαチャンネルビット0を抽出する.

## Misc
### Opening Movie

[@otameshi61](http://otameshi61.hatenablog.com/entry/2018/09/09/154341)さんのwriteupのトレースです

Windows用ILSpyを落としてきてMoviePlayer.dllを開く.
namespaceが4つあるので一つずつ見ていくとMoviePlayer.Pagesにそれらしいコードが見つかった.
C#知ってる人ならピンポイントで見にいけるのかな？
![](/assets/images/post/20180907/MoviePlayer.png)

countが300を超えるとtxtをiframeで読み込む処理になっている.
```
if (this.count >= 300)
{
  builder.AddContent(11, "\t");
  builder.OpenElement(12, "div");
  builder.AddAttribute(13, "class", "div");
  builder.AddContent(14, "FLAG:");
  builder.CloseElement();
  builder.AddContent(15, "\n\t");
  builder.OpenElement(16, "iframe");
  builder.AddAttribute(17, "src", this.txt);
  builder.CloseElement();
  builder.AddContent(18, "\n");
}
```

txtは`encrypt(str) + ".txt"`で作られている.
```
private string txt
{
  get
  {
    return this.encrypt("FLAG_IS_HERE") + ".txt";
  }
}
```

encryptメソッドはこれ.
```
private string encrypt(string str)
{
  MD5CryptoServiceProvider mD5CryptoServiceProvider = new MD5CryptoServiceProvider();
  return BitConverter.ToString(mD5CryptoServiceProvider.ComputeHash(Encoding.UTF8.GetBytes(str))).ToLower().Replace("-", "");
}
```

pythonで書き直す
```
import hashlib
from binascii import hexlify
import requests
md5 = hashlib.md5()
md5.update(bytes("FLAG_IS_HERE", "utf-8"))
r = requests.get("http://blazor.cyberrebeat.adctf.online/" + hexlify(md5.digest()).decode("utf-8") + ".txt")
print(r.text)
```
CRCTF{to the twilight of the internet}
