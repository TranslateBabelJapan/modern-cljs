# Tutorial 4 - モダンなClojureScript

このチュートリアルでは、[Larry Ullman][2]によって書かれた、
[Modern JavaScript: Development and Desiin][1]という本から、
いくかのJavaScriptのサンプルを書き直すことをやってみようと思う。

本のコードは、[ここ][3]からダウンロードすることができる。

参照元として、この本を選んだ理由としては、
JavaScriptのコーディングのためにいいアプローチを維持しているからだ。

思うに、LarryのアプローチをJavaScriptからClojureScriptに書き換えることは、
ClojureScriptをすらすら書くことの手助けになると思う。

## 始めに

ご存じのとおり、1990年代、JavaScriptはHTMLのフォームをバリデートしたり、
あるいはフォーム自体の改善のために使われていた。

そして、2000年代の始まりごろ、JavaScriptはサーバーサイドに対して、
非同期的なリクエストを送るために使われ始めた。数ヶ月の間で、二つのバズワードを
見るようになる。AjaxとWeb 2.0だ。

言ったように、[Modern JavaScript][1]の書籍にあるな、
[Larry Ullman's][2]のアプローチを、モダンなClojureScriptの類に変換する
ことに挑戦しようと思う。

じゃあ、彼の最初のサンプルであるログインフォームを、ClojureScriptに書き換えてみよう。
というのも、ここ10年のJavaSciprtの使われ方の変容をよく表現しているし、ClojureScriptや
Clojureの両方、あるいは片方について良くしらなくても、ClojureScriptについて学べるからだ。

> NOTE 1: 私見になるが、ClojureScriptは、サーバーサイトの開発を作るのを簡単にする
> だけではなく、クライアントサイトの開発もスマートにすることができると思う。
> アプリケーション・ロジックは、サーバーサイトからクライアントサイドに素早く
> 移動するわけで、サーバーサイド開発のように、ブラウザの中で走るCを着飾ったLisp、
> その名もJavaScriptをそれほど愛せなくなるだろう。(訳者中: ここは自信無し)
>
> もう *ブラウザの中で動く* ベストなLISPを見つけることに希望を見出すことができるし、
> クライアントサイド・プログラマーと共にそれらを届けることに挑戦するべきだ。言い換えると、
> 私たちには、 *サーバーサイドで動いている* Cを着飾ったLispと呼ばれるものを
> 見るリスクがあるというわけだ。
>
> NOTE 1 訳者(esehara)註: 上記部分は、元著者の力が入っているせいか、少々言っていることを
> 掴むのが正直難しかったです……

## Preamble

もし、[前回のチュートリアル][16]を終わらせて、このチュートリアルを始めるのなら、
[git][20]をインストールして、下のことをやるといい。

```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout tutorial-03
git checkout -b tutorial-04-step-1
```

## 登録フォーム

[Modern JS][3]のサンプルコードをダウンロードしたなら、`ch02`のディレクトリの中に`js/login.js`、
`css/styles.css`、`login.html`を見つけるはずだ。

![Modern ch02 tree][5]

`login.html`をブラウザで開いたなら、次のような画面が見えているはずだ。

![Login Form][6]

ちょっとHTMLを見てみよう。

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
    <form action="login.php" method="post" id="loginForm">
        <fieldset>
            <legend>Login</legend>

            <div>
              <label for="email">Email Address</label>
              <input type="email" name="email" id="email" required>

            </div>

            <div>
              <label for="password">Password</label>
              <input type="password" name="password" id="password"
              required>
            </div>

            <div>
              <label for="submit"></label>
              <input type="submit" value="Login &rarr;" id="submit">
            </div>

        </fieldset>
    </form>

    <script src="js/login.js"></script>

</body>
</html>
```
### Progressive enhancement and unobtrusive JS

`name`属性と、`id`属性の両方が、それぞれの要素にあることに注意しよう。
`name`の値は、サーバーサイド対してデータが送信されたときに使われる。
`id`の属性は、JavaScriptが使うためにある。

`login.php`というスクリプトは、`フォームアクション`と協調する。そして、
`login.js`はHTMLページの中にリンクがはられている。一方、`login.html`の中にある
`login.js`のほうは、JavaScriptと`フォーム`の間を直接接続したりはしない。
`login.php` script is associated with the `form action`. 
これらの選択を、Larry Ullmanの本では明確に *プログレッシブ・エンハンスメント*(直訳すると、革新的に改善する)、そして*目立たないJavaScript*と呼んでいる。

このアクションの中におけるアプローチを、下の[シークエンス・ダイアグラム][4]で見てみよう。

#### サーバーサイドだけでバリテーション

![Login Form Seq DIA 1][8]

ユーザーがフォームデータを送信すると、`login.php`と呼ばれる
サーバーサイドの`PHP`がバリテーションする。もしバリテーションが通った
なら、ユーザーはログインできる。`でなければ`、サーバーはユーザーに
正しい入力をするように、エラーを返すだろう。

JavaScriptとAjaxのおかげで、このユーザー・エクスペリエンス(ユーザー体験)は
大きな改善をすることができる。よりよい解決は、JavaScriptを使うことで、
クライアントサイドの振る舞いを、次のシークエンス・ダイアログのように
変更することだ。

#### クライアントサイドでバリテーション

![Login Form Seq DIA 2][9]

(例えば、JavaScriptを使うとかで)クライアントサイトのバリテーションを
通過したとしても、セキュリティのために、サーバーサイドのバリテーションも
聞く必要がある。

しかし、クライアントサイドのバリテーションを通過しないときは、サーバーに
飛ぶ必要はないわけで、即座にユーザーにエラーを返せばいい。

だが問題がないわけではない。クライアントサイドのバリテーションは、
ユーザーネームが登録されているかどうかをチェックすることができない。
この方法を、三番目のシークエンス・ダイアグラムで確認しよう。

#### アクションの中にあるAjax

![login Form Seq DIA 3][10]

ユーザー・エクスペリエンスはかなり改善されている。
Ajaxは、もっと効率よく、かつ反応がよいプロセスで、
サーバーが解決するコミュニケーションをしようとする。
(例えば、Emailアドレスが既に存在しているかどうか、とか)

このチュートリアルでは、サーバーサイドのバリテーションであったり、
あるいはAjaxがサーバーを呼び出すということをせず、クライアントサイドの
バリテーションに限定しようと思う。
もっと先のチュートリアルで、これらの方法を使うことになる。

## JavaScript

というわけで、さっそく`login.js`のコードを読んでみよう。

```JavaScript
// Script 2.3 - login.js

// Function called when the form is submitted.
// Function validates the form data and returns a Boolean value.
function validateForm() {
    'use strict';

    // Get references to the form elements:
    var email = document.getElementById('email');
    var password = document.getElementById('password');

    // Validate!
    if ( (email.value.length > 0) && (password.value.length > 0) ) {
        return true;
    } else {
        alert('Please complete the form!');
        return false;
    }

} // End of validateForm() function.

// Function called when the window has been loaded.
// Function needs to add an event listener to the form.
function init() {
    'use strict';

    // Confirm that document.getElementById() can be used:
    if (document && document.getElementById) {
        var loginForm = document.getElementById('loginForm');
        loginForm.onsubmit = validateForm;
    }

} // End of init() function.

// Assign an event listener to the window's load event:
window.onload = init;
```

## Porting to ClojureScript

決して短い道のりではないが、ログインフォームのバリテーションを
JavaScriptからClojureScriptに書き直すことを始めよう。
下層レイアーにあるJavaScript Virtual Machine(JSVM)で、
[CLJS interop][12]を使って、JavaScriptからClojureScriptに直接
書き直すことをやってみよう。

JavaScriptの`validateForm()`は、フォームの入力から、
`email`と`password`を受けとり、そして両方の値を調べている。
この`validateForm()`関数は、もしバリデーションが通ったなら
`true`を返し、なんらかのエラーがあるなら`false`を返す。

というわけで、何らかのClojureScriptのコードを書いてみよう。
`login.cljs`というファイルを、`src/cljs/modern_cljs`のディレクトリの
下に作ろう。そして、次のように書いてみよう。

```clojure
(ns modern-cljs.login)

;; define the function to be attached to form submission event
(defn validate-form []
  ;; get email and password element from their ids in the HTML form
  (let [email (.getElementById js/document "email")
        password (.getElementById js/document "password")]
    (if (and (> (count (.-value email)) 0)
             (> (count (.-value password)) 0))
      true
      (do (js/alert "Please, complete the form!")
          false))))

;; define the function to attach validate-form to onsubmit event of
;; the form
(defn init []
  ;; verify that js/document exists and that it has a getElementById
  ;; property
  (if (and js/document
           (.-getElementById js/document))
    ;; get loginForm by element id and set its onsubmit property to
    ;; our validate-form function
    (let [login-form (.getElementById js/document "loginForm")]
      (set! (.-onsubmit login-form) validate-form))))

;; initialize the HTML page in unobtrusive way
(set! (.-onload js/window) init)
```

見ての通り、書き直されたコードには二つの関数が定義されている。
`validate-form`と`init`だ。

> NOTE 2: ClojureやClojureScriptの中で、キャメルケースは
> 慣用的な使われ方ではないことに注意しよう。だから、
> `validateForm`は`vaidate-form`に書き直されている。

`let`はJavaScriptの`var`のように、ローカル変数を定義することができる。

同様に、JavaScriptのネイティヴな関数を呼び出すためだったり、JavaScriptのオブジェクトのプロパティにセット、あるいはゲットをするために、"."とか
".-"が使える。また評価される前の関数を値のように扱える。(例えば、関数を呼び出すなら`.getElementById`、プロパティにセットするなら`.-value`、そして関数を値として扱いたいなら、`.-getElementById`だ。)
これは、[ClojureScriptとClojureの違い][12]の一つである。それは、下層レイヤーであるところの仮想マシンに依存しているわけだ。(JavaScript VMとJVMの違いみたいなもんだね)

[ch02のModern JavaScript Code][3]から、`login.html`ファイルをもってきて、`resources/public`ディレクトリに置こう。

> NOTE 3:
> HTML5対応ブラウザを使っているなら、フォームの最後の属性として`novalidate`を追加することで、
> フォームのバリテーションをアクティブにしないようにすることができる。`novalidate`のスペルに気をつけて。
> 他方で、HTML5対応ブラウザは、`required`属性のフィールドを自由にさえぎることもできるし、そしてJavaScript
> が評価する前にこれらをチェックすることで、ClojureScriptによるアラートウィンドウを見ることができなくなる。

最後に、scriptタグの`src`属性の値に`js/modern.js`をセットしよう。これで`login.html`は最後だ。

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

これでほとんど終わりである。`style.css`をModern JavaScriptのコードである[ch02/css][3]から
`resources/public/css`ディレクトリにコピーしよう。

## ClojureScriptをコンパイル

普段通り、`login.cljs`をコンパイルしよう。(cfr. [Tutorial 1][11])

```bash
lein cljsbuild once
Could not find metadata thneed:thneed:1.0.0-SNAPSHOT/maven-metadata.xml in central (http://repo1.maven.org/maven2)
Retrieving thneed/thneed/1.0.0-SNAPSHOT/maven-metadata.xml (1k)
    from https://clojars.org/repo/
Compiling ClojureScript.
Compiling "resources/public/js/modern.js" from "src/cljs"...
Successfully compiled "resources/public/js/modern.js" in 5.075248 seconds.
```

ClojureScriptのソースコードが変更されたら、自動的にJavaSriptにコンパイルしなおす
ように設定したければ、`once`というコマンドを`auto`に書き換えればよい。

```bash
lein cljsbuild auto
```
## 実行

全てが上手くいっているかどうか、`login.html`をブラウザで開いて確認しよう。

![Login Form][13]

[最初のチュートリアル][11]から"Hello, ClojureScript"が、ログインフォームと共に
ブラウザ上に見えているはずだ。その理由は、先で説明する[Google Clojure Compiler][17]に
よるものだ。

さっそくフォームで遊んでみよう。

* EmailとPasswordのフィールドを入力する前に、ログインボタンをおしたら、
  アラートウィンドウが出て、フォームの入力が終わったかどうか聞いてくるはずだ。
* もし、EmailとPasswordの両方のフィールドを入力したあとに、ログインボタンを押した
  なら、`/path/to/modern-cljs/resources/public/login.php`が見つからない、という
  普段のブラウザで良く見かけるエラーメッセージが表示されるはずだ。これは、HTMLのページの
  アクション属性が、サーバーサイドのバリテーションスクリプトとして、`login.php`を参照している
  からだ。Clojureによる、サーバーサイドのバリテーションについては、先のチュートリアルで行う。

![Please, complete the form][14]

![File not found][15]

## 楽しいパート

このチュートリアルの最後の節で、[tutorial 3][10]で紹介したClojure http-severと
breplを利用して遊んでみよう。

0. オートモードでコンパイルを立ち上げよう。 `lein cljsbuild auto`
1. breplを立ち上げよう。 `lein trampoline cljsbuild repl-listen`
2. ring serverを立ち上げよう。 `lein ring server`
3. ページを見てみよう。 `http://localhost:3000/login.html`
4. breplで `(in-ns 'modern-cljs.login)`を入力してみよう。
5. でもって`validate-form`を入力してみよう。Google Clojure Comilerによってサポートされた、ClojureScriptコンパイラがJavaScript関数を生成しているのが見えるだろう。
6. `(validate-form)`を評価してみよう。すると、アラートウィンドウがフォームの入力を完了したかどうか訪ねてくるのが見えるだろう。
7. breplからブラウザと対話できたのだ！

もし、emailとpasswordを満たしてログインボタンをおしたとき、[TUtorial 3][16]のCompojureをセットアップしたときの`Not found page`が見えるだろう。

このチュートリアルのpreambleのように、gitの新しいブランチを作っているのなら、下のようにコミットしておくことをお薦めする。

```bash
git commit -am "modern javascript"
```

# Next Step [Tutorial 5: Introducing Domina][18]

In the [next tutorial][18] we're going to use [domina library][19] to
make our login form validation more clojure-ish.

# License

Copyright © Mimmo Cosenza, 2012-13. Released under the Eclipse Public
License, the same as Clojure.

[1]: http://www.larryullman.com/books/modern-javascript-develop-and-design/
[2]: http://www.larryullman.com/
[3]: http://www.larryullman.com/downloads/modern_javascript_scripts.zip
[4]: http://en.wikipedia.org/wiki/Sequence_diagram
[5]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/ch02-tree.png
[6]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/login-form.png
[8]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/login-01-dia.png
[9]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/login-02-dia.png
[10]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/login-03-dia.png
[11]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-01.md
[12]: https://github.com/clojure/clojurescript/wiki/Differences-from-Clojure
[13]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/login-cljs-01.png
[14]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/please-complete.png
[15]: https://raw.github.com/magomimmo/modern-cljs/master/doc/images/file-not-found.png
[16]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-03.md
[17]: https://developers.google.com/closure/compiler/
[18]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-05.md
[19]: https://github.com/levand/domina
[20]: https://help.github.com/articles/set-up-git
