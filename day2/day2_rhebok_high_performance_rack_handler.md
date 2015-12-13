# Rhebok, High performance Rack Handler

Masahiro Nagano @kazeburo
"Site Reliability Engineer" @ Mercari

----

- rack handler / web server
- 1.5x-2x performance compared to Unicorn
- Prefork architecture same as Unicorn
- Rhebok is suitable for running HTTP application servers behind a reverse proxy like nginx
- Ruby port of Perl's Gazelle
    - Gazelle: High performance Plack handler
    - 2x-3x faster than commonly used Starman, Starlet
    - Production-ready
        - 1-3% performance

Unicornはツノ1本、Gazelle/Rhebokはツノ2本

Highly-optimized high traffic websites
  Gaming, Ad-tec, Recipe site, Media, massive scale SNS
can be applied to any site

ある程度最適化をしてくるとRack handlerが結構長くなってくる。ので最適化されたサイトのほうがいい

Who should not use Rhebok
  WebSocket or Streaming
  reverse proxyを置かないような場合 (connectionがいっぱいになっちゃう)

HTTP/1.1 Web Server
  KeepAlive以外サポート（理由がある）
  TCP / Unix Domain Socket
  Hot deployment using start_server
  OobGC リクエストが終わったあとにいっぺんにGCを走らせる

rackupで起動する

```
rackup -s Rhebok \
  --port 8080 \
  -E production \
  -O MaxWorkers=20 \
  -O MaxRequestPerChild=1000
...
```

(device) - (ReverseProxy) - (Rhebok)

hot deploy
  start_server (perlがオリジナル, golangもある)
  start_serverがsocketを作ってlistenせずにsocketする
  環境変数としてRhebokにファイルディスクリプタを渡す
  SIGHUPを送ると新しいサーバーをforkして立ち上げる、ファイルディスクリプタからlisten
  古いワーカーにSIGTERMを送りgraceful shutdown

ベンチマーク
  wrkをunix domain socketサポートにしてベンチマーク
  ISUCONベンチマーク 予選の問題を使用
    実際にあるweb applicationに近い

----

## How to create a high performance Rack Handler and Rhebok internals

Rack / Rack Handler

- Rack
    - インターフェースの仕様
        - WebServerとWAFの間のインターフェース仕様
    - 実際のmodule

WebServer(unicorn, thin, puma) <- (rack web interface) -> (rails, sinatra, padrino)Framework

```
app = Proc.new do |env|
  [
    '200',
    {'Content-Type' => 'text/html'},
    ['Hello']
  ]
end
```

env : Hash object that contains Request Data
  CGI keys
  Rack specific keys

配列を返す
  1個目: status (文字列!)
  2個目: response header
    複数同じキーのもの入れたい場合は改行で区切って入れる
  3個目: body
    response body must respond to each
      array of strings
      application instance
      file-like object

Rack Handlerの役割
  クライアントからのリクエストを受け取り
  envを作り
  アプリケーションを呼んで
  HTTP responseを作ってクライアントに返す

実際にRack handlerを作ってみる
  ソケットを作り
  accept
  リクエストを読み取ってenvを作る
  appを実行
  レスポンスを作る
  名前空間としてはRack::Handler

サーバーの実行
  例 `rackup -r ./shika.rb -s Shika -E production config.ru`

問題がある
  パフォーマンス
    1つのリクエストしかさばけない
    1つのリクエストが詰まったら残りも詰まる
  タイムアウトがない
  HTTP request parser supportがない

並行性
  multi-process
  multi-thread
    lightweight context-switch
  IO multiplexing
    event-driven, can handle multiple connections
    
よく使われるもの
  Unicorn -> multi process
  Puma -> multi thread + limited event model (+ multi process)
  Thin -> event model (+ multi process)

Prefork architecture
  親となるprocessでbindしてlisten
  その後子をfork、親のソケットをaccept

prefork_engine
  Ruby port of Perl's Parallel::Prefork
  a simple perfork server framework

IO timeout
  Unicorn does not have IO timeout
    send SIGKILL to a long running process
    default timeout 30sec
  select(2)を使う例
  RhebokはIO timeoutをCで実装している
    Rubyだとnon-blockingで実装してもblockingになっちゃう
    poll(2) instead of select(2)

HTTP request parsing
  easy to cause security issue
  safer to choose an existing one that is widely used
  Mongrel-based - Unicorn, Puma
  Node.js based - Passenger5
  PicoHTTPParser - Rhebok, h2o
    pico_http_parser gem

TCP optimization
  TCP_NODELAY
    普通はNagle's algorithmでまとめて送る。TCP_NODELAYでoffにする。
    小さいパケットを送るとTCPはもらったことを返さないといけないのでネットワークを使いすぎる
    スループットは上がるけどレイテンシも増える
    writev(2)を使う
      1回のシステムコールで、複数のstringをポインタ・長さ・個数のセットでwriteできる
