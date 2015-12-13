# Discussion on Thread between version 1.8.6 and 2.2.3

@emorima

----

jishin.net
会員制の地震情報サービス
1スレッド1センサー、1プロセス100スレッド、100プロセス 10000スレッド

Thread#status
正常な通信が行えなくなったときにわかればよかった

main thread starts child threads (Thread.new)
and observers child threads' status (Thread#status)

Thread#status returns "run" or "sleep", but child thread often got stuck.

main thread -> observe class -> child thread
child threads update observe class's checked time
main thread checks the time and confirm that child threads are run.

- 大江戸Ruby会議01 "mission critical"なシステムでも使えるThreadの使い方
- Critical mission system implemented in Ruby (RK2011)

子スレッドで詰まった例
forループの中で処理が詰まった
10000スレッドで同時にタイムアウト例外処理したら詰まった

----

検証の方法

同じコードを1.8.6と2.2.3で動かす
100 threads / process * 1 process
CPU usage
memory usage

2.2.3のほうがCPU usageが高い

Rubyは速くなったがCPU使用率は上がったのではないか…？
なお実行回数は同じだった。速度も速くなっていない…？

子スレッドで例外処理を同時に投げる

メモリ使用量はだいぶ減ってる

----


