# Pragmatic Testing of Ruby Core

@hsbt

----

ペパボでのRuby

- yaocloud - OpenStack向けfog
- mod_mruby, ngx_mruby, rcon

ruby, rake, rubygems, rdoc, psych, ...

----

How to contribute to OSS

people say: "it's easy! write documentation!"

-> no, it's difficult. Only Matz knows true Ruby behavior.
-> Documentation is valuable work.

Testing and Running is easy

- Ruby has a lot of test ecosystem and libraries
- Bundler and Docker provide an encapsulated environment

If you get test failures, you can submit issue ticket to tracker.

If test coverage is missing for some ruby code, you can also write new tests and submit a patch to upstream

Code reading tips
  git clone ***
  cd gems
  less .travis.yml
  (pick out before_script and script code and invoke it)
  だいたいのコードはこれやればテストできる

Rubyのlanguageをtestするには:

```
git clone https://github.com/ruby/ruby
cd ruby
autoconf
./configure --disable-install-doc
make -j
make check
```

1つ1つは何やってるのかが難しい

(Makefileではなく)common.mkを読む

make test
  test-sample
    sample/test.rb を実行する !?!?
      「sampleフォルダの下にはあるけれどRubyの歴史上は思い出深いファイル」
  btest-ruby
    あとで
  test-knownbug
    invoke KNOWNBUGS.rb (たいていempty)

btest-ruby
  bootstrap testの略
  bootstraptest/runner.rbを実行
  bootstrap testはtargetのrubyを変えることができる
    ちょっと壊したrubyで実行できるかを試したい
  
make test-all
  testディレクトリにあるやつをすべて動かします
  いわゆるアプリケーションのテスト
  標準添付しているライブラリのテスト
    Webrick, Logger
  TESTSという環境変数に値をいれると挙動が変わる
    TESTS=logger : test/loggerのみ実行
    TESTS=-"j4" : 4並列で実行 4 processes
  testフォルダの下はみなさんがよく目にするいわゆるテストコード
  -ext-フォルダ
    C APIに対してテストが実行できる
  test/ruby
    Rubyの組み込みクラスのテストが入っている

make check
  だいたい全部を実行する
  test-testframework
    テストフレームワークに対するテストを実行する
  test-almost
    test-unitやminitestに依存しないものをテストする

rubygems, rdoc, net-smtp は minitest を使っている。net-smtpは minitest を使う理由がないので直した。
rubygemsとrdocは別のupstreamがあるので直接取り込んだからminitestを使う。

rubygems and rdoc still support Ruby 1.8 -> tf.close!のclose!は1.8には存在しない！

test::unitフォーク
  test/lib/envutil.rb
    Rubyコミッタの中でもやばいと評判

rubyspec
  "CRuby開発者はRubySpec使ってくれないからやめた"
  rubyspec is not a specification. it's actually a set of test. the only real ruby specification is inside of Matz :)

rubyspecは生きてます
  ruby/rubyspec
  2.3で検索するとCRubyの新機能がわかって超便利
  
rubyci
chkbuild

make/run
make/bisect

test coverage
  coverageのcoverageしようとするとバグる…
