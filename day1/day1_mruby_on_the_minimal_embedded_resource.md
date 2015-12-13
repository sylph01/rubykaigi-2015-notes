# mruby on the minimal embedded resource

Shota Nakano @ Manycolors, Inc

----

developing:
mruby rapid prototyping platform (enzi)
  ARM Cortex-M4, non-OS device
  schematic, firmware, software, web simulator
consumer electronics using mruby
  Realtek, MIPS core, embedding Linux
  control and communication software using mruby
One of initial developer of mrdb (mruby debugger)
fork mruby-polarssl to Apache license
  mbed TLS

----

組み込みでmruby使うのの何がいいか？ -> インタプリタがある。

センサーを載せてもよい。センサーとかのプロトタイピングが容易にできる。

mruby codeを送る。コマンドではない。
クラスやメソッドを定義できる
PCは文字を送るだけで、ボードが解釈する

JSON, SimpleHTTP, RegExp, Time
SSL, MQTT is coming
Amazon JP is ready, US is coming

http://www.amazon.co.jp/enzi-EBB-01-basic-board%E3%80%90mruby%E3%83%A9%E3%83%94%E3%83%83%E3%83%89%E3%83%97%E3%83%AD%E3%83%88%E3%82%BF%E3%82%A4%E3%83%94%E3%83%B3%E3%82%B0%E3%83%97%E3%83%A9%E3%83%83%E3%83%88%E3%83%95%E3%82%A9%E3%83%BC%E3%83%A0%E3%80%91%E8%BB%BD%E9%87%8FRuby%E3%81%8C%E5%8B%95%E4%BD%9C%E5%8F%AF%E8%83%BD%E3%81%AA%E7%B5%84%E8%BE%BC%E5%9F%BA%E6%9D%BF/dp/B00PRO1JOU

spec
non-OS
STM32F4 - ARM Cortex-M4
Externa; SRAM 8M bits
GCC4 cross-compile

SRAM as mruby-specified memory
mrubyはメモリアロケーションをする関数(mrb_open_allocf())を変更することができる→外部のSRAMを使うようにした
Internal RAMは128KBが使用可能。

STM32F4シリーズにはFlexible Static Memory Controllerという機能でexternal memoryをMPUアドレスに割り振れる。

----

組み込み機器で何でmruby使うの？

Physical Computing
Cで開発するのは効率が悪い・開発に時間がかかる
  インタプリタならピンの番号を間違えてもすぐに直して試すことができる
  Cならファームウェア再コンパイルが必要
DIY/R&D→Hardware startup, M2M, consumer and other devices

mrubyのよいところ
小さい実装/app code size
OOP
言語レベルでFiberをサポート センサーからのデータを切り替えることで擬似マルチタスク
ライブラリが豊富。多少書き換えればGemで公開されているものも使える

デバイスからフロントエンドまで全部Rubyで作れる！

----

rapid prototyping platforms

- Python
    - ライブラリが少ない
- JS
    - いっぱい出てるけどCより短く書ける気がしない…
    - 2つくらいの製品ではインタプリタが動く
- Lua
    - 適材適所に見えるがライブラリがあまりない

（必ずしも全部試したわけではない）

----

mruby platforms

- enzi
- Wakayama.rb
    - RX core
- EAPL-Trainer mruby
    - LCD display included, RX core, libraries
- STM32F4 Discovery, PIC32, FPGA, ...
    - NO driver and libraries

Cortex-AシリーズとかAtomとかMIPSはPCと近いところがある（メモリ領域も十分ある）のでそれは今回のスコープ外

Real-Time OSとかは抜き

----

なんで外部SRAM必要なの？
mruby 1.2をコンパイルすると150KBのROMが必要
libc(newlib)はさらにROM/RAMを必要とする
  ROM 50KB+, RAM 100KB+

普通の組み込みMPUは128KBRAMあれば十分大きい ROMは1MBくらいのもある。ROMはあまり気にしなくても大丈夫かなあ。

何も手を入れずにmrubyを扱うことはできない。

容量対策
(1)DRAM - 基盤の層が4層くらいになる、高い
(2)SRAM - 中間的。キャパシティが少ない。
(2)NOR flash - 安くて大容量。しかし書き込み回数制限が…

SPIから繋いだやつはinternal addressを一般に持たないがSTMicro社のCPUはinternal addressを持てる

mrubyのRAM書き込み回数がそんなに多くない場合、cacheとしてflashが使えるのではないか？

memcpyで大量に書き込みが発生する
ループはそこまで関係ない。問題はstringを使うとメモリの書き換えが頻繁に発生する。
stringを使わないならflashをRAMとして使うのは有効

(1)stringを内部RAMにアロケートするように書き換える → 単純に書き換えるとGCが効かなくなる
(2)SRAM+Flash combination, FlashはSRAMからオーバーした分に限定する → mrubyの書き換えよりもドライバの書き換えが大変…

16bit mruby (mruby/c)を使う
PSoC5 (Cortex-M3 core)
19KB flash, 3KB SRAM
PSoC5 -> ROM 256kb, RAM 64kb
how to combine general mruby and mrbgems? -> 開発中、どうにかしたい

----

- non-OSでもmrubyいけるよ！強力！
- Flashを使うとlow-costだけど使い方を気をつけよう