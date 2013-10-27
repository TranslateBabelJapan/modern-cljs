# Tutorial 6 - 容易なことは複雑になるし、シンプルなことは簡単になる

このチュートリアルでは、前回に出会ったチュートリアルの問題を調べて、
そのことを解決しようと思う。

## Preamble

もし、[前回のチュートリアル][1]を終わらせて、このチュートリアルを始めるのなら、
[git][9]をインストールして、下のことをやるといい。

```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout tutorial-05
git checkout -b tutorial-06-step-1
```
## Introduction

前回のチュートリアルでは、あんまりいいとは言えない問題で終わった。問題は下のように
まとめることができる。

* `login.html`のページと`login.cljs`のソースファイルが対応している

* `shopping.html`のページと`shopping.cljs`のソースファイルが対応している

* 普段の方法でアプリを立ち上げる。 

```bash
lein ring server # from the project home directory
lein cljsbuild auto # from the project home directory in a new terminal
lein trampoline cljsbuild repl-listen # from the project home directory in a new terminal
```

ブラウザでショッピングページに訪れると、`shopping.cljs`で宣言した、`window`オブジェクトの`onload`という属性にセットした関数である`init`ではなく、`login.cljs`で宣言されたものであることがわかる。

前回のチュートリアルで見たとおり、この振る舞いは、`lein-cljsbuild`によって使われるGoogle Closure Compilerに依存するものだ。


## Google Closure Compiler 入門

[最初のチュートリアル][3]で、私たちは`:cljsbuild`というキーワードで、Google Closure Compilerの振舞いを下のように変更した。

```clojure
(defproject ....
  ...
  :cljsbuild {:builds
              [{:source-paths ["src/cljs"]
                :compiler {:output-to "resources/public/js/modern.js"
                           :optimizations :whitespace
                           :pretty-print true}}]})

```

この`:source-paths`というオプションは、Closure Compilerに対して、`src/cljs`にあるディレクトリ構造から、全てのClojureScriptのソースコードを探し出してくるように指示している。また、`:compiler`の`:output-to`というキーワードは、Closure Compilerに対して、`"resources/public/js/modern.js"`にコンパイルした結果を保存するように指定している。

　ClojureScript/Closure Compilerというコンパイラーについて一つずつ細かに説明しようとは望まない。問題を調査し、解決するために必要な詳細というのは、コンパイラが、`"src/cljs"`ディレクトリ、あるいはそのサブディレクトリにある**全ての**ClojureScriptファイル(つまり、`connect.cljs`, `login.cljs`, `shopping.cljs`のことだ)から**一つの**JavaScriptファイルを生成している(つまり `"resources/public/js/modern.js"` のことだ)ということだ。

## 変わりやすさは罪か?

`login.cljs`も、`shopping.cljs`も、最終的に`(set! (.-onload js/window) init)`を呼び出している。これらは`logic.cljs`から一回、そして`shopping.cljs`から一回、計2回呼び出されている。これらの呼び出しは問題はない。というのは、どちちが最初にくるとしても、前回の値を上書きしてしまうからだ。JavaScriptの変化可能なデータ構造に対してクリアになるケースだ。

## 安易さは複雑さを生む

上記の問題から、読者はClojureScriptというのは*単体のブラウザアプリケーション*のときのみ有効であると感じるだろう。実際、JavaScriptの`window`オブジェクトに対して同じ`onload`の属性に変数を加してうことによる、処理の衝突を避けるための方法はある。コードを重複させてしまうことだ！

つまり、ディレクトリ構造を重複させ、そして、単体のJavaScriptファイルをそれぞれのHTMLに対応させる形ビルドオプションを追加する必要があるということだ。

 下にあるbashのコマンドをターミナルで叩いてみよう。

```bash
mkdir -p src/cljs/{login/modern_cljs,shopping/modern_cljs}
mv src/cljs/modern_cljs/login.cljs src/cljs/login/modern_cljs/
mv src/cljs/modern_cljs/shopping.cljs src/cljs/shopping/modern_cljs/
cp src/cljs/modern_cljs/connect.cljs src/cljs/login/modern_cljs/
cp src/cljs/modern_cljs/connect.cljs src/cljs/shopping/modern_cljs/
rm -rf src/cljs/modern_cljs
```

　そして、下のように`project.clj`を変化させてみよう。

```clojure
(defproject ...
  ...

  :cljsbuild
  {:builds

   ;; login.js build
   {:login
    {:source-paths ["src/cljs/login"]
     :compiler
     {:output-to "resources/public/js/login.js"
      :optimizations :whitespace
      :pretty-print true}}
    ;; shopping.js build
    :shopping
    {:source-paths ["src/cljs/shopping"]
     :compiler
     {:output-to "resources/public/js/shopping.js"
      :optimizations :whitespace
      :pretty-print true}}}})
```
> NOTE 1: `:cljsbuild`の設定をより深く知るために、強く薦めたいのが、
> [lein-cljsbuild][5]のプラグインにある[advanced project.cljのサンプル][4]を読むことだ。

最終的に、それぞれのHtmlページのスクリプトのタグに正しいJavaScriptファイルを与えてあげる必要がある(要するに`"js/login.js"`は`login.html`に、`"js/shopping.js"`や`shopping.html`に、だ)。

上記のような解決のことを、**複雑さが追加された**というようによく言われる。事実悪いことに、それぞれのJavaScriptのファイルが出力されると、Closure Compilerが全体のサイズをスマートにするとしても、他の難しさを呼び出してしまう。ブラウザが、キャッシュから他の全てを保存するために、最初のキャッシュを保存する手立てが無くなってしまうのだ。

## シンプルさは簡単にする

今こそ、シンプルで簡単な方法を試してみよう。

* `login.cljs`と`shopping.cljs`ファイルの両方から、`(set! (.-onload js/window) init)`をはずそう。

* そして、`:export` タグを、`login.cljs`と`shopping.cljs`の`init`関数に追加しよう。

* `script`タグで、対応する`init`関数を、`login.html`と`shopping.html`の両方で呼び出そう。

* 完成！

> NOTE 2: でも、もしClojureScriptの関数として、`^:export`を使いたくなければ、
> Google Closure Compilerに対して`:optimizations`という戦略を使わせることもできる。
> `:simple`という最適化オプションを指定したら、Closure CompilerはJavaScriptを小さくしてくれるし、
> 他のローカル変数やローカル関数の名前も小さくなり、難読かされて、外部のJavaScriptコードとして存在しなくなる。
> もし、関数や変数の名前に`:export`というメタデータがあれば、その名前は標準的なJavaScriptのコードとして呼び出すことができる。
> 二つの関数は、次のような形で存在することになる。`modern_cljs.login.init()`と、`modern_cljs.shopping.init()`という形である。

さっそく`login.cljs`のコードの断片を見てみよう。

```clojure
;; the rest as before
(defn ^:export init []
  (if (and js/document
           (.-getElementById js/document))
    ;; get loginForm by element id and set its onsubmit property to
    ;; validate-form function
    (let [login-form (.getElementById js/document "loginForm")]
      (set! (.-onsubmit login-form) validate-form))))

;; (set! (.-onload js/window) init)
```

`shopping.cljs`のコードの断片も見てみよう。

```clojure
;; the rest as before
(defn ^:export init []
  (if (and js/document
           (.-getElementById js/document))
    (let [theForm (.getElementById js/document "shoppingForm")]
      (set! (.-onsubmit theForm) calculate))))

;; (set! (.-onload js/window) init)

```

これは、`login.html`のコード断片だ。

```html
    <script src="js/modern.js"></script>
    <script>modern_cljs.login.init();</script>
```

これは、`shopping.html`のコード断片だ。

```html
  <script src="js/modern.js"></script>
  <script>modern_cljs.shopping.init();</script>
```

これで普通に走らせればいい。

```bash
lein ring server # from the project home directory
lein cljsbuild auto # from the project home directory in a new terminal
lein trampoline cljsbuild repl-listen # from the project home directory in a new terminal
```

If you created a new git branch as suggested in the preamble of this
tutorial, I suggest you to commit the changes as follows

```bash
git commit -am "Introducing domina"
```

今後のチュートリアルで、[Modern JavaScriptのサンプル][7]を、ClojureScriptにする、関数型スタイルを改善するための[dominaのイベント管理方法][6]を紹介する。

# 次のステップ - [Tutorial 7: 二倍攻撃的にやろうぜ][8]

In the [next tutorial][8] we're going to explore CLJS/CLS compilation modes by
using the usual `lein-cljsbuild` plugin of `leiningen`.

# License

Copyright © Mimmo Cosenza, 2012-2013. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-05.md
[2]: http://localhost:3000/shopping.html
[3]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-01.md
[4]: https://github.com/emezeske/lein-cljsbuild/blob/master/example-projects/advanced/project.clj
[5]: https://github.com/emezeske/lein-cljsbuild
[6]: https://github.com/levand/domina#event-handling
[7]: http://www.larryullman.com/books/modern-javascript-develop-and-design/
[8]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-07.md
[9]: https://help.github.com/articles/set-up-git
