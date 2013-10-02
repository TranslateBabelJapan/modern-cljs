# Tutorial 1 - The basic

まず始めに、[leiningen 2][2]と[lein-cljsbuild][3]というプラグインを使って、
[CLJS][1]の基本的なプロジェクトを作ってみよう。

[Leiningen][2]は、Clojureのためのビルド管理システムだ。[lein-cljsbuild][3]は
ClojureScriptに特化したLeiningenのプラグインだ。

## ClojureのProjectを作ろう！

まだ何もやっていないのなら、まずは[leiningen][2]をインストールしよう。そして
新しいプロジェクトを作ろう。名前は`modern-cljs`だ。

```bash
lein new modern-cljs
```

お節介だけど、`lein`を環境変数として`$PATH`に入れることをお薦めしよう。

## プロジェクトの構成をかえてみよう！

ClojureとClojureScriptをホストするためのディレクトリを作ろう。そして、
Leiningenが作った`src/modern_cljs`のディレクトリを新しい構成のディレクトリに
反映させよう。

```bash
cd modern-cljs
mkdir -p src/{clj,cljs/modern_cljs}
mv src/modern_cljs/ src/clj/
```

> NOTE 2: due to [java difficulties][4] in managing hyphen "-" (or other
> special characters) in package names, substitute an underscore for any hyphen
> in corresponding directory names.


プロジェクトに静的ファイルをおいておくためのディレクトリも作っておこう。
(例えば、htmlのページとか、JavaScriptとか、CSSとか、ね）

```bash
mkdir -p resources/public/{js,css}
```

最後に、下のようなディレクトリ構成になればいい。

```bash
├── LICENSE
├── README.md
├── doc
│   └── intro.md
├── project.clj
├── resources
│   └── public
│       ├── css
│       └── js
├── src
│   ├── clj
│   │   └── modern_cljs
│   │       └── core.clj
│   └── cljs
│       └── modern_cljs
└── test
    └── modern_cljs
        └── core_test.clj

12 directories, 6 files
```

## Edit project.clj

でだ。`project.clj`を編集する必要が出てくる。というのも:


* プロジェクトにある`source-paths`のClojureのソースコードをアップデートするため。
* [lein-cljsbuild][3]のプラグインを`project.clj`の設定に追加するため。

Leiningenが作成した`project.clj`がそこにあるはずだ。

```clojure
(defproject modern-cljs "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.5.1"]])
```

そこに`lein-cljsbuild`のアップデートされたバージョンを置いて、`:cljsbuild`と
`:source-paths`の設定をするんだ。

```clojure
(defproject modern-cljs "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}

  ;; CLJ source code path
  :source-paths ["src/clj"]
  :dependencies [[org.clojure/clojure "1.5.1"]]

  ;; lein-cljsbuild plugin to build a CLJS project
  :plugins [[lein-cljsbuild "0.3.3"]]

  ;; cljsbuild options configuration
  :cljsbuild {:builds
              [{;; CLJS source code path
                :source-paths ["src/cljs"]

                ;; Google Closure (CLS) options configuration
                :compiler {;; CLS generated JS script filename
                           :output-to "resources/public/js/modern.js"

                           ;; minimal JS optimization directive
                           :optimizations :whitespace

                           ;; generated JS code prettyfication
                           :pretty-print true}}]})
```

## ClojureScriptのソースファイルを作ろう

`project.clj`を設定したあと、ClojureScriptのファイルを、`src/cljs/modern_cljs`ディレクトリに
作ろう。下のようなファイルだ。

```clojure
(ns modern-cljs.modern)

(.write js/document "Hello, ClojureScript!")
```

これを`modern.cljs`というファイル名で保存しよう。
Save the file as `modern.cljs`.

> ノート3: ClojureScriptのソースコードは*.cljsであって、*.cljではないことに
> 注意しよう。

## Htmlファイルもう

シンプルなHtmlを作って、そこに`project.clj`にあった`:output-to`キーワードの値である
ポイントの`script`タグを入れよう。それを`resources/public/`ディレクトリの中に、
`simple.html`の中にセーブしよう。

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Simple CLJS</title>
    <!--[if lt IE 9]>
    <script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
    <![endif]-->
</head>
<body>
    <!-- pointing to cljsbuild generated js file -->
    <script src="js/modern.js"></script>
</body>
</html>
```

## ClojureSciptをコンパイルしよう

ClojureからJavaScriptにするために、下のように`cljsbuild`のサブタスクである
`once`を使おう。

```bash
lein cljsbuild once
Retrieving lein-cljsbuild/lein-cljsbuild/0.3.3/lein-cljsbuild-0.3.3.pom from clojars
Retrieving lein-cljsbuild/lein-cljsbuild/0.3.3/lein-cljsbuild-0.3.3.jar from clojars
Compiling ClojureScript.
WARNING: It appears your project does not contain a ClojureScript dependency. One will be provided for you by lein-cljsbuild, but it is strongly recommended that you add your own.  You can find a list of all ClojureScript releases here:
http://search.maven.org/#search|ga|1|g%3A%22org.clojure%22%20AND%20a%3A%22clojurescript%22
Retrieving cljsbuild/cljsbuild/0.3.3-SNAPSHOT/cljsbuild-0.3.3-20130913.125809-2.pom from clojars
Retrieving cljsbuild/cljsbuild/0.3.3-SNAPSHOT/cljsbuild-0.3.3-20130913.125809-2.jar from clojars
Compiling "resources/public/js/modern.js" from ["src/cljs"]...
Successfully compiled "resources/public/js/modern.js" in 7.131854 seconds.
```

見ての通り、*WARNING*といっている。これは、`cljsbuild`プラグインは、ClojureScriptがどんなにリリース
されていたとしても、`project.clj`ファイルの`:dependencies`の部分に、特定のClojureScriptを使うように
強く薦めているのだ。

というわけで、ClojureScriptを`:depedencies`に入れることで、`cljsbuild`は幸せな気持ちになれるでしょう！

```clj
(defproject modern-cljs "0.1.0-SNAPSHOT"
  ...
  :dependencies [[org.clojure/clojure "1.5.1"]
                 [org.clojure/clojurescript "0.0-1878"]]
  ...)
```

> NOTE 4: we added the latest CLJS release (i.e. "0.0-1878") available
> at the time of this writing.

`lein cljbuild clean` コマンドは、以前の問題を綺麗にしてくれる
そして`lein cljsbuild once`をもう一度たたいてみよう。

```clj
lein cljsbuild clean
Deleting files generated by lein-cljsbuild.
```

```clj
lein cljsbuild once
Compiling ClojureScript.
Retrieving org/clojure/clojurescript/0.0-1878/clojurescript-0.0-1878.pom from central
Retrieving org/clojure/clojurescript/0.0-1878/clojurescript-0.0-1878.jar from central
Compiling ClojureScript.
Compiling "resources/public/js/modern.js" from ["src/cljs"]...
Successfully compiled "resources/public/js/modern.js" in 6.894774 seconds.
```

## simple.htmlを見てみよう

ブラウザを開いて、ローカルファイルの`simple.html`を見てみよう。
全て上手くいっているなら、"Hello, ClojureScipt!"の文字が見えるだろう。

![Hello ClojureScript][5]

## Next Step - [Tutorial 2: Browser CLJS REPL (bREPL)][6]

In the next [tutorial][6] we're introduce the so called *brepl*, a browser
connected CLJS REPL.

# License

Copyright © Mimmo Cosenza, 2012-2013. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://github.com/clojure/clojurescript.git
[2]: https://github.com/technomancy/leiningen
[3]: https://github.com/emezeske/lein-cljsbuild.git
[4]: http://docs.oracle.com/javase/specs/jls/se7/html/jls-6.html
[5]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/hellocljs.png
[6]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-02.md
