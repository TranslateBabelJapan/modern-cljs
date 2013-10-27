# Tutorial 5 - Domina入門

このチュートリアルでは、前回のチュートリアルで見せた、ログインフォームの
バリテーションを改善するために、[Domina][1]を紹介する。

## Preamble

もし、[前回のチュートリアル][2]を終わらせて、このチュートリアルを始めるのなら、
[git][13]をインストールして、下のことをやるといい。

```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout tutorial-04
git checkout -b tutorial-05-step-1
```

## 始めに

前回から、[Js interop][7]を使うことで、JavaScriptからClojureScriptに直接
書き換えることで、ClojureScriptのコーディング始めた。今回はもうちょっとマシな
方法を試してみようと思う。

> [Domina][1]は、ClojureScriptのための、jQueryに影響を受けたDom処理ライブラリです。
> これは、Googlo CLosure Libraryが提供しているDom処理のインターフェイスを、
> CLojureの関数型的な慣習へと合わせたものです。Domina自体はどんなイノベーションを
> しているわけではないけれども、ClojureScriptの中で自然に感じるような、Dom処理を、
> 基本的な関数の中で使うことができるでしょう。

ClojureScriptの中を見ていたとき、最初に[clojurescriptone][3]を見つけて、
そして[Design and templation][4]を読んで、下の文章に対してとても納得した
ことがある。

> 多くのClojureのウェブアプリケーションでは、HTMLテンプレートとして[Hiccup][5]
> を使っている。プログラマーが同様にデザイナーであるならば、これは理想的である。
> しかし、多くの開発者というのはデザインに関しては疎い。Clojureのことには特に
> 注意を払わない良きデザインを行う人と働かなければいけない。Hiccupのデータ構造
> などに注意を払わなくても、HTMLやCSSと共に働くデザイナーに対して、ClojureSciptの
> テンプレートの方法を提供する。

昔の友である`login.html`を純粋なHTML/CSSテンプレートにしよう。そしてClojureScript
及びClojure及びCLojureScriptの流儀にページのDomのインターフェイスを作ってくれるのが
`domina`だ。

ここに、[tutorial 4][2]使った`login.html`の内容がある。

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Login</title>
    <!--[if lt IE 9]>
    <script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
    <![endif]-->
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <!-- login.html -->
    <form action="login.php" method="post" id="loginForm" novalidate>
        <fieldset>
            <legend>Login</legend>

            <div>
              <label for="email">Email Address</label>
              <input type="email" name="email" id="email" required>
            </div>

            <div>
              <label for="password">Password</label>
              <input type="password" name="password" id="password" required>
            </div>

            <div>
              <label for="submit"></label>
              <input type="submit" value="Login &rarr;" id="submit">
            </div>

        </fieldset>
    </form>
    <script src="js/modern.js"></script>
</body>
</html>
```

## プロジェクトのdependeciesにdominaを追加しよう

Leiningenを普段使うとき、もしClojureやClojureScriptのライブラリを使うために、
ライブラリを`project.clj`に書き足してやる必要がある。`project.cls`を書き直した
バージョンは下の通りだ。

```clojure
(defproject modern-cljs "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}

  ;; clojure source code path
  :source-paths ["src/clj"]

  :dependencies [[org.clojure/clojure "1.5.1"]
	             [org.clojure/clojurescript "0.0-1847"]
                 [compojure "1.1.5"]
                 [domina "1.0.2-SNAPSHOT"]]

  :plugins [; cljsbuild plugin
            [lein-cljsbuild "0.3.3"]
            [lein-ring "0.8.7"]]

  ;; ring tasks configuration
  :ring {:handler modern-cljs.core/handler}

  ;; cljsbuild tasks configuration
  :cljsbuild {:builds
              [{;; clojurescript source code path
                :source-paths ["src/cljs"]

                ;; Google Closure Compiler options
                :compiler {;; the name of emitted JS script file
                           :output-to "resources/public/js/modern.js"

                           ;; minimum optimization
                           :optimizations :whitespace

                           ;; prettyfying emitted JS
                           :pretty-print true}}]})
```

> NOTE 1: **注目** [domina][1]のリリースしている"1.0.2-SNAPSHOT"の不完全さとバグ、
> のため、私はClojureScriptのリリースを"0.0-18477"に戻している。
>
> 既に[domina][1]ライブラリの作者は、私が送った修正をマージしているのだが、
> 未だに公開レポジトリでは、このように開発している。これはたまに起きることだ。
> 次のチュートリアルに進む時も、この手の問題が現れたりするだろう。そのときは、
> どのように管理するかを説明する。

## Dominaのセレクター

[Domina][1]は、いくつかのセレクタ関数をもっている。`domina.xpath`という名前空間の
中には`xpath`が存在していて、`domina.css`という名前空間には`sel`が存在している。とはいえ、
`domina`のコアには`by-id`、`value`、そして`set-value`という関数が定義されているので、
これらを使おうと思う。

`domina`が実行するGoogle Clojure Libraryからの、Dominaの`(by-id id)`はステキなことに
文字列を渡されたとしても、上手いこと変換してくれる。そして予想でkりうように、`domina`の
コアの名前空間には、とても使える関数がいくつも用意されている。`(value el)`は、要素の値を
返してくれるものだし、`(set-value! el value)`は、要素の値をセットする。

> Note 2: 関数が渡された引数について変更を加える場合、Clojureでは関数の後に"!"という
> 名前をつけるようになっている。

> Note 3: :use や :require が名前空間で必要になったとき、ClojureScriptは、
> :useでは :only が使えるし、また:requireで:asが使える。機能の違いについては、
> [the ClojureScript Wiki][8]を見て欲しい。

## バリテーションフォームを変化させよう

このステップでは、名前空間での宣言を書き換えて、Js interopで使っていた
`.getElementById`と`.-value`を、dominaで対応する関数である`by-id`と`value`の
関数に書き換えてみよう。

プロジェクトのディレクトリである`src/cljs/modern_cljs`から`login.cljs`を開き、
`namespace`の宣言部分と、`validate-form`関数の定義を、下のように書き換えてみよう。

```clojure
(ns modern-cljs.login
  (:use [domina :only [by-id value]]))

(defn validate-form []
  ;; get email and password element using (by-id id)
  (let [email (by-id "email")
        password (by-id "password")]
    ;; get email and password value using (value el)
    (if (and (> (count (value email)) 0)
             (> (count (value password)) 0))
      true
      (do (js/alert "Please, complete the form!")
          false))))
```

見ての通り、`domina`を使ったコードは以前よりとてもスムーズだ。残りのファイル
についてもこのように書き換よう。そして、全て上手くいっているかどうか、下の
コマンドで確認しよう。

```bash
cd /path/to/modern-cljs
lein ring server
lein cljsbuild once # in a new terminal and after having cd in modern-cljs
lein trampoline cljsbuild repl-listen
```

> Note 4: 貴方の開いているターミナルで、プロジェクトのホームディレクトリにいるかどうか
> 確かめよう。
> Note 4: be sure to `cd` to the home directory of the project in each
> terminal you open.

<http://localhost:3000/login.html>を開いて、ブラウザとClojue ScriptのReplが接続されたら、
REPLから下のようなコマンドを叩いてみよう。

```bash
ClojureScript:cljs.user> (in-ns 'modern-cljs.login)

ClojureScript:modern-cljs.login> validate-form
#<function validate_form() {
  var email__6289 = domina.by_id.call(null, "email");
  var password__6290 = domina.by_id.call(null, "password");
  if(function() {
    var and__3822__auto____6291 = cljs.core.count.call(null, domina.value.call(null, email__6289)) > 0;
    if(and__3822__auto____6291) {
      return cljs.core.count.call(null, domina.value.call(null, password__6290)) > 0
    }else {
      return and__3822__auto____6291
    }
  }()) {
    return true
  }else {
    alert("Please, complete the form!");
    return false
  }
}>
ClojureScript:modern-cljs.login>
```

`validate-form`シンボルを評価すると、Clojure Scriptのコンパイラがそのシンボル自身に結びついた
JavaScriptの関数を返す。`(validate-form)`で、この関数を呼び出てみると、きっとブラウザに
全ての入力が完了したかどうか、訪ねるアラートウィンドウが、ブラウザ上で表示されるだろう。
さらに`ok`をクリックすると、`(validate-form)`は`false`を返してくる様子が見えるだろう。

```bash
ClojureScript:modern-cljs.login> (validate-form)
false
ClojureScript:modern-cljs.login>
```
ログインフォームの`Email Address`と`Password`を書き込んでみよう。そして、
ClojureScirptのReplプロンプトから、`(validate-form)`を再び呼び出してみよう。
すすと、`(validate-form)`は`true`を返し、次のチュートリアルで使うClojureの
[元になる][6]サーバーサイド・スクリプトに、そのコントロールを投げようとする
様子が見れる。

```bash
ClojureScript:modern-cljs.login> (validate-form)
true
ClojureScript:modern-cljs.login>
```

## ショッピング金額の計算をするサンプル

さて、Larry Ullmanの[Modern JavaScirpt][6]の本から二番目のサンプルを
ClojureScriptに書き直してみよう。これは、商品の全体と、税込、そして
割引率を計算する、E-コマーシャル用のツールである。

### 純粋にHTML/CSS

HTML/CSS/imagesをコードから分離しつつ、そのデザインをたもつために、[Lally Ullman][6]と
[clojurescriptoneのアプローチ][4]の`shopping.html`の内容が下の内容である。これを
`resources/public`ディレクトリにセーブしよう。

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Shopping Calculator</title>
    <!--[if lt IE 9]>
    <script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
    <![endif]-->
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
  <!-- shopping.html -->
  <form action="" method="post" id="shoppingForm" novalidate>
    <legend> Shopping Calculator</legend>
    <fieldset>
      <div>
        <label for="quantity">Quantity</label>
        <input type="number"
               name="quantity"
               id="quantity"
               value="1"
               min="1" required>
      </div>
      <div>
        <label for="price">Price Per Unit</label>
        <input type="text"
               name="price"
               id="price"
               value="1.00"
               required>
      </div>
      <div>
        <label for="tax">Tax Rate (%)</label>
        <input type="text"
               name="tax"
               id="tax"
               value="0.0"
               required>
      </div>
      <div>
        <label for="discount">Discount</label>
        <input type="text"
               name="discount"
               id="discount"
               value="0.00" required>
      </div>
      <div>
        <label for="total">Total</label>
        <input type="text"
               name="total"
               id="total"
               value="0.00">
      </div>
      <div>
        <input type="submit"
               value="Calculate"
               id="submit">
      </div>
    </fieldset>
  </form>
  <script src="js/modern.js"></script>
</body>
</html>
```
まず始めに、ClojureScriptのコンパイルで生成する、外部JavaScriptの`js/modern.js`の
リンクを入れておこう。フォームの`action`の値には、この時何も入れておかないことに注意しよう。
新しいサンプルには、サーバーサイドが内からだ。

`shoppeing.html`は、下のように表示されているはずだ。

![Shopping Page][9]

### 金額計算するClojureScriptのコード

ショッピングした金額計算を行うコードを書くときがきた。この計算フォームからいくつかの
値を読み取る必要がある。

* 商品の量
* 1つ辺りの金額
* 税率
* 割引

全体の計算が終わった時、フォームの結果を書き直し、そして`false`を返す。というのは、
データを返すためのサーバーサイドが何もないからだ。

`shopping.cljs`ファイルを`src/cljs/modern_cljs`ディレクトリに作り、そして
下のコードをタイプしよう。

```clojure
(ns modern-cljs.shopping
  (:use [domina :only [by-id value set-value!]]))

(defn calculate []
  (let [quantity (value (by-id "quantity"))
        price (value (by-id "price"))
        tax (value (by-id "tax"))
        discount (value (by-id "discount"))]
    (set-value! (by-id "total") (-> (* quantity price)
                                    (* (+ 1 (/ tax 100)))
                                    (- discount)
                                    (.toFixed 2)))
    false))

(defn init []
  (if (and js/document
           (.-getElementById js/document))
    (let [theForm (.getElementById js/document "shoppingForm")]
      (set! (.-onsubmit theForm) calculate))))

(set! (.-onload js/window) init)
```

さっそく、いつもの通り、この小さなショッピングの計算機をコンパイルしよう。

```bash
lein ring server # in the modern-cljs home directory
lein cljsbuild auto # in the modern-cljs directory in a new terminal
lein trampoline cljsbuild repl-listen # in a the modern_cljs directly in a new terminal
```

### ちょっとしたトラブルーシューティング

`localhost:3000/shopping.html`に訪れて、`Calculate`ボタンを押すことで、
計算することができる。しかし"Page not found"というのを受けとる。何が
起こってるんだ？

受けとったエラーに何も手がかりはない。このようなときに、使えるデバッグツールを
また紹介していない。だから、手探りでトラブルシューティングしてみよう。
ClojureSCript REPLをブラウザにコネクトしよう(breplだね)。で、前のLogin formの
サンプルと共に、breplから`(validate-form)`関数を動かして、振る舞いをテストしてみよう。
そして、同じように`moderncljs.shopping`という名前空間で定義した`(calculate)`関数の
評価を行ってみよう。

最初に、`shopping.html`に再び戻って、Breplの中で、下のClojureScriptの式を
評価してみよう。

```bash
ClojureScript:cljs.user> (in-ns 'modern-cljs.shopping)

ClojureScript:modern-cljs.shopping> (calculate)
false
ClojureScript:modern-cljs.shopping>
```

`calculate`関数は、正しく`false`を返しており、そして`Total`も計算フォームに
正しく表示されている。

![Total][11]

これが意味することは、breplの中では`caluculate`関数は正しく呼び出されている
ということであり、しかし`Calculate`ボタンのフォームでは正しく動いていないということに
なる。では、breplの助けにより、問題に向かってみよう。

```bash
ClojureScript:modern-cljs.shopping> (.-onsubmit (.getElementById js/document "shoppingForm"))
nil
ClojureScript:modern-cljs.shopping>
```

おっと、`shoppingForm`のフォーム要素である`onsubmit`の値がないらしい。
`caluculate`は、`window`オブジェクトの属性である`on-load`が値として
設定する`init`という値がセットされているべきだ。`window`オブジェクトの
`onload`プロパティの値を見てみよう。

```bash
ClojureScript:modern-cljs.shopping> (.-onload js/window)
#<function init() {
  if(cljs.core.truth_(function() {
    var and__3822__auto____6439 = document;
    if(cljs.core.truth_(and__3822__auto____6439)) {
      return document.getElementById
    }else {
      return and__3822__auto____6439
    }
  }())) {
    var login_form__6440 = document.getElementById("loginForm");
    return login_form__6440.onsubmit = modern_cljs.login.validate_form
  }else {
    return null
  }
}>
ClojureScript:modern-cljs.shopping>
```

おや、`window` `onload`の値として定義した`init`関数は、一つだけに定義
されているわけではなく、前回の`loginForm`関数にも定義されている。

これはGoogle Closure コンパイラーと使っているときに起こることだ(
cljsbuildがそうだ)。`lein-cljsbuild`の`:output-to`オプションの値が
、最初のチュートリアルから設定した、`source-paths`キーワードに
ある全てのClojureScriptを取得し、そして"js/modern.js"にファイルに対して
同じファイルになるようにセットしたのだ。

緊急にこの問題を解決するなら、breplで`(init)`関数を呼び出せばよい。

```bash
ClojureScript:modern-cljs.shopping> (init)
#<function calculate() {
  var quantity__18253 = domina.value.call(null, domina.by_id.call(null, "quantity"));
  var price__18254 = domina.value.call(null, domina.by_id.call(null, "price"));
  var tax__18255 = domina.value.call(null, domina.by_id.call(null, "tax"));
  var discount__18256 = domina.value.call(null, domina.by_id.call(null, "discount"));
  domina.set_value_BANG_.call(null, domina.by_id.call(null, "total"), (quantity__18253 * price__18254 * (1 + tax__18255 / 100) - discount__18256).toFixed(2));
  return false
}>
ClojureScript:modern-cljs.shopping>
```

これで*Shipping Calculator*フォームの*Calculate*ボタンが使えるようになった。

If you created a new git branch as suggested in the preamble of this
tutorial, I suggest you to commit the changes as follows

```bash
git commit -am "introducing domina"
```

# Next Step [Tutorial 6: 容易なことは複雑になるし、シンプルなことは簡単になる][12]

In the [next tutorial][12] we're going to investigate and solve in two
different ways the problem we just met.

# License

Copyright © Mimmo Cosenza, 2012-2013. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://github.com/levand/domina
[2]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-04.md
[3]: https://github.com/brentonashworth/one
[4]: https://github.com/brentonashworth/one/wiki/Design-and-templating
[5]: https://github.com/weavejester/hiccup
[6]: http://www.larryullman.com/books/modern-javascript-develop-and-design/
[7]: https://github.com/clojure/clojurescript/wiki/Differences-from-Clojure
[8]: https://github.com/clojure/clojurescript/wiki/Differences-from-Clojure
[9]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/shopping.png
[10]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-01.md
[11]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/total.png
[12]: https://github.com/TranslateBabelJapan/modern-cljs/blob/japanese-translate/doc/tutorial-06.md
[13]: https://help.github.com/articles/set-up-git