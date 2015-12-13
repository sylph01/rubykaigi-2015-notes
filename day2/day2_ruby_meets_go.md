# Ruby Meets Go

Masaki Matsushita

----

Go 1.5 feature: buildmode "c-shared"
Using Go function from Ruby
  FFI and Fiddle without ruby.h
Writing extension library with Go and C
  Define functions equivalent to C Macros
  Avoid copy of strings
  ...

----

`go build -buildmode c-shared`
Cから読み込めるライブラリを作れる。`cgo`というものを使っている。

import "C" -> cgoを使います、という宣言
Goの文字列をC.CString("Hello, World")として変換する
Cのstringに変換したものを使って`C.puts(str)`みたくする

コメントのところにCの関数を書く

cstr := C.hello() : call C function from Go, get C string
fmt.Println(C.GoString(cstr)) : Convert into Go string

c-shared: Cから見ることのできる共有ライブラリを作ることができる

```
//export add
func add(a C.int, b C.int) C.int {
  return a + b
}
```

-> .soができる

Load c-shared libraries
ruby-ffi : gem install ffi
fiddle : standard ext library
これらを使うとruby.hすら使わずにGoの関数を呼び出すことができる

---


```
requiire "ffi"

module Int
  extend FFI::Library
  ffi_lib "int.so"
  attach_function :add, [:int, :int], :int
end

p Int.add(15, 27) #=> 42
```

文字列を受け渡すには少々テクニックが要る。

Cgo functions to Convert String
C.CString(goString string) *C.char
  ユーザーが責任をもってfreeしてあげなければならない
C.GoString(cString *C.char) string

"Writing extension libraries in Go" @ OedoRubyKaigi05

----

ruby.hにINT2NUM, NIL_P, RSTRING_PTRなど便利なマクロがあるがCgoから使うことはできない…
なのでGoでequivalentなC関数を作る

```
func LONG2NUM(n C.long) C.VALUE {
  return C.rb_long2num_inline(n)
}
```

----

Convert Go String into Ruby without Copy
普通の方法だと2回のコピーが発生する。自分でfreeしなくてはいけない。

Basic usage of C.CString()
Call C func and discard C str soon

Cの関数を1回呼ぶためだけにコピーがしなくてはいけない、ということがドキュメントに書かれているくらい

まっとうな使い方でGo StringをRuby Stringにしようとすると2回コピーが発生する(Go -> C, C -> Ruby)し、すぐに捨てちゃうようなもののためにコピーを発生させてしまう

怪しいコードを書かなくてはいけない

----

Ruby ReferenceをGoに伝搬させる

GoのGCはRubyから参照されていることを知らない
Go objがRubyで参照されている場合はGoの世界に知らせなければならない

→ objectsというmapを用意し、mapにオブジェクトへの参照をいれてやることでリファレンスカウンタ的なもの作る
allocation, freeの際に呼ばれるようにしてやる

----

GoのコードでGemを作る

ディレクトリ構造 bundle gem --ext foo
ext/foo/ に foo.go, wrapper.goなどを手で追加

Rakefile
.goをsource_patternに追加する必要がある

extconf.rb
いろんなダーティハックを使ってビルドを通るようにしている
goの実行ファイルがあるかどうかをcheck
通常のビルドプロセスが走らないようにハックをしている
Makefileをgo buildを呼び出すよう変更

----

拡張ライブラリの例

- gohttp
- memberlist(GH: mmasaki/memberlist)

今の所決定版といえるようなヘルパーライブラリは出ていない。GoをhackしてRubyの拡張ライブラリをGoで書けるようにしよう！
