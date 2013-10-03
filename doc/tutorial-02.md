# Tutorial 2 - ブラウザでClojureScript!! (bREPL)

このチュートリアルでは、外部のhttp-serverを利用した、ClojureScriptのREPLに
ブラウザで接続する方法(bRepl)を学ぶ。

## 始めに

ClojureのようなLISP方言を使う大きな目的の一つに、
対話的なプログラミングスタイルを可能にしてくれるREPL(読んで評価して表示
のループ(Read Eval Print Loop))の存在がある。ClojureScriptコミュニティーは
Clojureに存在する、REPLベースのプログラミング・エクスペリエンスを提供する
ために、熱心に作業を続けていて、ブラウザの中にあるJavaScriptエンジンに
ClojureScriptのREPLをコネクトするための方法を作り出した。
このプログラミングスタイルは、ClojureScriptをREPLの中で評価できるようにし、
REPLを接続したブラウザで、即時にフィードバックを得られる。

[cross site scripting][1]攻撃を予防するためのブラウザの読み込み制限の結果、
JavaScriptエンジンが組み込まれたブラウザでREPLに接続することは、
[Same Origin Policy][2]を尊重しなければならなくなった。どういうことかといえば、
もしClojureScriptのREPL(brepl)にブラウザが接続したければ、ローカルにhttp-serverを
設置する必要があるということだ。

どんなhttp-serverだって使うことはできる。ただ、このチュートリアルでは、Mac OS Xのために
用意された[MAMP][4]にある、[apache http-server][3]を使う。というのも、とても
簡単に設定が出来て、簡単に立ち上げることができるからだ。他のOSでは、同じような選択肢が必要だ。

> ノート1: とても手軽で、コンパクトなhtt-serverとして、Pythonのmoduleである
> `SImpleHttpServer`がある。もしOSにPythonをインストールしているならば、ただ
> 下のように立ち上げればよい。
> ```bash
> cd /path/to/modern-cljs/resources/public
> python -m SimpleHTTPServer 8888
> Serving HTTP on 0.0.0.0 port 8888 ...
> ```
>
> [Max Penet][5]の提案である。ありがとう！

> ノート2: 私たちは[次のチュートリアル][11]で、Clojurenのために、[Ring][17]と
> [Compojure][18]を利用したHTTPサーバーの立ち上げ方を見ていく。

## Preamble

前のチュートリアルを終わらせて、今回のチュートリアルを始めたいのならば、
お節介ながらに[git][16]をインストールして、下のことをやろう。

```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout tutorial-01
git checkout -b tutorial-02-step-1
```

これは、今回のチュートリアルを始めるために、Tutorial01ブランチを作り、
そこから新しいブランチを作る方法だ。

## MAMPをインストール、設定、そして立ち上げ！

MAMPをインストールするために、MAMPのドキュメントをみよう。そして、Admin GUI
の設定ボタンをクリックしよう。

![MAMP Admin Panel][6]

そして、Apache tabをクリックして、ローカルのApache http serverのルートディレクトリ
として、`/path/to/modern-cljs/resources/public`を選ぼう。で、OKをクリックだ。最終的に
`Start Servers`で全てやってくれる。これで、ローカルWebサーバーが、マシンの`8888`ポート上で
動くことになる。`http://localhost:8888/simple.html`を使って
[Tutorial 1 - The Basic][8]で作った[simple.html][7]を見に行って、
全てOKか確認しよう。

## ClojureScriptのREPL(brepl)に接続するために、ブラウザを設定しよう。

breplを設定するために、下のステップを実行する必要がある。

* ブラウザとbprelの間との接続をpredisposeするためのClojureScriptを作る
* ClojureScriptをコンパイルする
* breplサーバーをスターとさせる
* 接続できるようにする

### コネクションを作成しよう

`src/cljs/modern_cljs`の中に、下のような内容のClojureScriptを作ろう。

```clojure
(ns modern-cljs.connect
  (:require [clojure.browser.repl :as repl]))

(repl/connect "http://localhost:9000/repl")
```

このファイルを`connect.cljs`として保存しよう。

見ての通り、ブラウザからbreplに接続するため、
`clojure.browser.repl`という名前空間の中で定義された`connect`という
関数を呼び出してやる必要がある。breplに接続するためのポートとして`9000`を
使おう。というのも、これがbreplサーバーが通常使うポートだからだ。

### ClojureScriptをコンパイルしよう

で、この新しいClojureScriptのファイルをコンパイルする必要がある。[Google Closure Compiler][9]
(CLS)は[Tutorial 1][8]で既に設定した`project.clj`ですでに設定したcompilation optionがある。
そして、既に設定してあるものとして、このオプションをはずそう。さあ、ClojureScriptのcompilationを
呼び出そう。

```bash
lein cljsbuild once
Compiling ClojureScript.
Compiling "resources/public/js/modern.js" from "src/cljs"...
Successfully compiled "resources/public/js/modern.js" in 4.904672 seconds.
```

### Breplを立ち上げよう

ReplとBrowserの接続を出来るようにするために、ブラウザからの接続を待機する、replとして
振る舞うサーバーを立ち上げる必要がある。このために使う`repl-listen`は、既に
`lein-cljsbuild`が設定されている。

```bash
lein trampoline cljsbuild repl-listen
Running ClojureScript REPL, listening on port 9000.
"Type: " :cljs/quit " to quit"
ClojureScript:cljs.user>
```
まだbreplプロンプトでは、何もタイプが出来ない。ブラウザが接続することを
待ってから、やっと反応を返すことができる。

### コネクションを可能にしよう

立ち上がったbreplとブラウザを接続するために、
前のチュートリアルで作った[simple.html][7]に訪れる必要がある。
`file:///.../simple.html`のかわりに、`http://localhost:8888/simple.html`
に行って確認してみよう。言い換えると、breplはまだ接続しない。
[Tutorial 1][8].  

Obciously, http-serverは走っている。[simple.html][7]のページにいくことで、
ClojusureScript Complitationで作られた`modern.js`スクリプトは、`lein-cljsbuild`
プラグインの`repl-listen`によってスタートした`9000`ポートのbreplリスニングに
コネクトしようとする。

今なら、ClojureScriptをbreplの中で評価することができる。

```clojure
ClojureScript:cljs.user> (+ 41 1)
42
ClojureScript:cljs.user>
```

上手くいっているのならば、ブラウザとともにCLojureSciriptを対話的に評価しはじめているだろう。
そして、ブラウザ自身が即座にフィードバックを返してくれるはずだ。

```clojure
ClojureScript:cljs.user> (js/alert "Hello from a browser connected repl")
```
![Alert Window][10]

このREPLは、コマンドの履歴であったり、あるいは編集能力があまりにもないことに
そのうち気がつくと思う。望むなら、[rlwrap][12]をインストールしたり、[この][13]インストラクション
を見たりすることで、これらの機能を追加することができる。いいことに、スタンダートなClojure REPLを
走らせるかどうかを、他のbreplオプションから表明する方法を、[Tutorial 18 - Housekeeping]で見ることが
出来る。


もし、このチュートリアルのpreableにある提案にあった、新しいブランチを作っているのなら、
さらにこれらの変更をコミットしておくことを薦めする。

```bash
git commit -am "brepl enabled"
```

## Next step - [Tutorial 3: CLJ based http-server][11]

In the next [tutorial][11] we're going to substitute the external
http-server with a CLJ based http-server in such a way that in others
tutorial we'll implement a direct communications between CLJS client
code and CLJ server code.

# License

Copyright © Mimmo Cosenza, 2012-2013. Released under the Eclipse Public
License, the same as Clojure.

[1]: http://en.wikipedia.org/wiki/Cross-site_scripting
[2]: http://en.wikipedia.org/wiki/Same_origin_policy
[3]: http://httpd.apache.org/
[4]: http://www.mamp.info/en/index.html
[5]: https://github.com/mpenet
[6]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/mamp-01.png
[7]: http://localhost:8888/simple.html
[8]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-01.md
[9]: https://developers.google.com/closure/compiler/
[10]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/alert.png
[11]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-03.md
[12]: http://utopia.knoware.nl/~hlub/rlwrap/#rlwrap
[13]: https://github.com/emezeske/lein-cljsbuild/wiki/Using-Readline-with-REPLs-for-Better-Editing
[14]: https://github.com/emezeske/lein-cljsbuild/issues/186
[15]: https://github.com/emezeske/lein-cljsbuild
[16]: https://help.github.com/articles/set-up-git
[17]: https://github.com/mmcgrana/ring
[18]: https://github.com/weavejester/compojure
[19]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-18.md
