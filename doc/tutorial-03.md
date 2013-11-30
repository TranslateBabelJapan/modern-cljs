# Tutorial 3 - Clojureベースなhttp-server

このチュートリアルは、[tutorial 2][1]で行った設定を、
ClojureのスタンダートなWebベースのアプリケーションを作るための
方法である[ring][2]と[compojure][3]を利用した外部http-serverに
取り替える方法を学ぶ。

## Introduction

いままでClojureScriptコードは、JavaScriptに一回コンパイルし、それをブラウザ上で走らせる
ためであって、Clojureがやることのできるhttp-serverまでは必要としていなかった。とはいえ、
Clojureが好きなわけだら、Clojureでのhttp-serverの立て方をもっと知りたいと思うはずだ。

[Ring][2] は、Webベースのアプリケーションを開発するための、Clojureベースのライブラリで
あり、fundamentalなbuilding-blockの一つである。他のあらゆるhttp-serverを使うかわりに、
これを使おうと思う。

## Preamble

[前回のチュートリアル][1]を終わらせて、今回のチュートリアルをやろうとしているならば、
[git][8]をインストールして、下のように入力することをお薦めする。

```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout tutorial-02
git checkout -b tutorial-03-step-1
```

## lein-ringプラグインをproject.cljに追加しよう

ClojureScriptを設定して走らせるために、ビルドのなかで如何に`lein-cljsbuild`
が手助けをしてくれるかを見てきた。似たような方法として、[ring][2]の仕事を
自動的かつ管理してくれる[lein-ring][3]というプラグインを使おうと思う。

`lein-ring`をインストールするために、`project.clj`にプラグインとして
`lein-ring`を追加しよう。`lein-cljsbuild`は、もし全てのプロジェクトで
使うことをお望みなら、グローバルなprofileにこいつを追加することもできる。
(例えば、`~/.lein/profiles.clj`の中に書くとかね)

`lein-cljsbuild`のように、`lein-ring`というプラグインは、`:ring`キーワードを
`project.clj`の中に追加する設定を必要としてくるだろう。`:ring`の値は、
設定のオプションへのマップを含んでいるのだが、ここではただ`:handler`という、
これから定義するであろう関数へのリファラーが必要となる。

先ほど話した、必要とされる設定と共に`project.clj`を書き換えよう。

```clojure
(defproject modern-cljs "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  ;; clojure source code pathname
  :source-paths ["src/clj"]

  :dependencies [[org.clojure/clojure "1.5.1"]
	             [org.clojure/clojurescript "0.0-1878"]]

  :plugins [;; cljsbuild plugin
            [lein-cljsbuild "0.3.3"]

            ;; ring plugin
            [lein-ring "0.8.7"]]

  ;; ring tasks configuration
  :ring {:handler modern-cljs.core/handler}

  ;; cljsbuild tasks configuration
  :cljsbuild {:builds
              [{;; clojurescript source code path
                :source-paths ["src/cljs"]

                ;; Google Closure Compiler options
                :compiler {;; the name of the emitted JS file
                           :output-to "resources/public/js/modern.js"

                           ;; use minimal optimization CLS directive
                           :optimizations :whitespace

                           ;; prettyfying emitted JS
                           :pretty-print true}}]})
```

## handlerを作ろう

ringのhandlerは、リクエストを引数として受けり、そしてレスポンスを返す
だけの関数だ。リクエストもレスポンスも、通常のClojureのマップだ。低レベル
レイヤーの[Ring API][4]を使う代わりに、他の共通ライブラリを`project.clj`に
追加しよう。その名も、[compojure][5]だ。

Webアプリケーションを小さく、そして独立したパートにcomposedし、[Ring][2]
のhandlerを生成するためのDSL(ドメイン特化言語)を[Ring][2]の
Webアプリケーションで出来るようにするための、小さなルーティングライブラリだ。

このチュートリアルでのゴールは、`resources/public`の中にセーブされた
静的なHtmlページ(simple.htmlだね！)を配信することができるように、http-serverを
立ち上げることだ。

`src/clj/modern_cljs`から`core.clj`というファイルを開いて、下の内容に書き換えて
みよう。

```clojure
(ns modern-cljs.core
  (:use compojure.core)
  (:require [compojure.handler :as handler]
            [compojure.route :as route]))

;; defroutes macro defines a function that chains individual route
;; functions together. The request map is passed to each function in
;; turn, until a non-nil response is returned.
(defroutes app-routes
  ; to serve document root address
  (GET "/" [] "<p>Hello from compojure</p>")
  ; to serve static pages saved in resources/public directory
  (route/resources "/")
  ; if page is not found
  (route/not-found "Page not found"))

;; site function creates a handler suitable for a standard website,
;; adding a bunch of standard ring middleware to app-route:
(def handler
  (handler/site app-routes))
```

## Add compojure to project.clj

Clojureベースのhttp-serverを立ち上げる前に、`project.clj`の
依存関係を記述しているセクションに`compojure`を追加しておく必要がある。
書き換えた`project.clj`は下のようになる。

```clojure
(defproject modern-cljs "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  ;; clojure source code pathname
  :source-paths ["src/clj"]

  :dependencies [[org.clojure/clojure "1.5.1"]
	             [org.clojure/clojurescript "0.0-1878"]
                 [compojure "1.1.5"]]

  :plugins [;; cljsbuild plugin
            [lein-cljsbuild "0.3.3"]
            [lein-ring "0.8.7"]]

  ;; ring tasks configuration
  :ring {:handler modern-cljs.core/handler}

  ;; cljsbuild tasks configuration
  :cljsbuild {:builds
              [{;; clojurescript source code path
                :source-paths ["src/cljs"]

                ;; Google Closure Compiler options
                :compiler {;; the name of the emitted JS file
                           :output-to "resources/public/js/modern.js"

                           ;; minimum optimization
                           :optimizations :whitespace

                           ;; prettyfying emitted JS
                           :pretty-print true}}]})
```

## http-serverを立ち上げろ！
全てを設定したら、下のようにサーバーを立ち上げることが出来るようになる。

```bash
lein ring server
2012-11-03 19:06:33.178:INFO:oejs.Server:jetty-7.6.1.v20120215
Started server on port 3000
2012-11-03 19:06:33.222:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:3000
```

"Hello form compojure"という文が表示されているページが見えているはずだ。
これを見ることができるのは、サーバーがデフォルトで動くポートである`3000`で
ある。オプションとして、`lein ring server 8888`のように、違うポート番号を
指定することができる。

ブラウザコネクトベースであるReplを、`lein trampoline cljsbuild repl-listen`で
立ち上げることによって動いていることもチェックができる。これは新しいターミナル
(`/path/to/modern-cljs`に移動することを思い出そう)でコマンドを叩こう。そして、
[simple.html][6]ページに行こう。

このチュートリアルのpreambleで提案した、新しいブランチを作成しているなら、
下のように変更を加えることをお薦めする。

```bash
git commit -am "added ring and compojure"
```

## 次のステップ - [Tutorial 4: モダンなClojureScript][7]

In the [next tutorial 4][7] we're going to have some fun introducing form validation in CLJS.

# License

Copyright © Mimmo Cosenza, 2012-2013. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-02.md
[2]: https://github.com/ring-clojure/ring
[3]: https://github.com/weavejester/lein-ring
[4]: http://ring-clojure.github.com/ring/
[5]: https://github.com/weavejester/compojure.git
[6]: http://localhost:3000/simple.html
[7]: https://github.com/TranslateBabelJapan/modern-cljs/blob/japanese-translate/doc/tutorial-04.md
[8]: https://help.github.com/articles/set-up-git
