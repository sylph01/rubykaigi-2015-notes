# Request and Response

@tenderlove

manageiq/manageiq
Ruby Core Team
Rails Core Team
Rack Core Team

"責任を逃れる方法がわからない"

Railsのrequestとresponseの話。現在と未来。

softwareについての話なのでsoft talkです。

----

HTTP/2

Rack

Rails Req/Res

Rail / Rack + HTTP2

----

HTTP/2

利点

Binary protocol, プロトコルの管理が簡単
binary dataはtext dataより短い
(not always a benefit: 目で読んでもわからない)

多重化(multiplexed)

server push

chrome
chrome://net-internals/#spdy

HTTP/2ではヘッダーは小文字で表示される

Firefox
X-Firefox-Spdy

----

Rack

Rackが依存の爆発問題(explosion of dependencies)を解決した
Adapter Pattern

Unicorn/Passenger/Puma/Webrickはフレームワークのことを知らない。逆もしかり。
全部の組み合わせのgemを作らなくてはいけない

----

「Webを使っている魚は何という？ ウェブ鯖」

----

Rack::Sendfile

ActionDispatch::Static
  4つくらいの文字列を作る
  
ActionDispatch::LoadInterlock

ActiveSupport::Cache::Strategy::...
  あまり面白くない

Rack::Runtime
  実行時間をヘッダに書き込む

(nanika)
(shrug)

RequestID

Rack Logger

AD::ShowExceptions

WebConsole

RCE as a feature! (!?)

（疲れたからスキップ…あまり面白くない…）

callbackが増えれば増えるほどN◯de.jsになる

...

最後にrouterにたどりつく

１つのresource

全部のmiddlewareを実行したのに404になった！

Rack山
「たけし城みたいですね」

---

RackのAPIの問題に対して何をするべきか？

middlewareの種類を分類

event listener - doesn't care about content. load from DB etc
content filter - gzip
content producer - ERB

call(env) -> call(request, response)に変換

----

why support H2 in dev?

-> asset push in dev

