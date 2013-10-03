# modern-cljs 日本語訳プロジェクト

## 翻訳完了したものたち


## [Tutorial 1 - 基本的なこと][5]

最初のチュートリアルでは、基本的なClojureScriptのプロジェクトを作り、
設定するところから始める。

## [Tutorial 2 - ブラウザでClojureScript!! (bREPL)][6]

このチュートリアルでは、ClojureScriptのREPLを、外部http-serverを立ち上げることで
ブラウザと接続する方法を学ぶ。

## 以下、もとのブランチから

A series of tutorials to guide you in creating and setting up
[ClojureScript][1] (CLJS) projects.

If you ask yourself why the series has been named `modern-cljs` even
if the ClojureScript language is only a couple of years old, you're
right. It just happens that I started this series while trying to port
on ClojureScript few examples from the
[Modern JavaScript: Develop and Design][34] book. Its correct title
should have been `current-cljs`. Now it's too late to change it.

## Required background

This series requires some programming experience. I assume you already
made your hands dirty with Clojure programming language, but you need
not to be proficient with it or even with the underlying Java and
JavaScript programming languages.

If you don't know anything about Clojure (or Lisp), I strongly
recommend to study it before to start reading this series.

Even if the net is plenty of outstanding resources on Clojure
programming language, you can't understimate the benefit of reading
few books on Clojure and on other Lisp dialects:

* [SICP - Structure and Interpretation of Computer Programs][27]: this
  is still the best book I have ever read about Computer Programming
  in my very long carrier, even if it uses the Scheme programming
  language (a Lisp dialect);
* [On Lisp][28]: if you want to make your hands dirty with macros,
  this is the place to start from even if it uses CommonLisp (a well
  known Lisp dialect);
* [Clojure Programming][29]: written by three of the most heroics
  clojureans, it contains everything you need to know about Clojure
  and its ecosystem;
* [Programming Clojure][30]: written by another legendary clojurean,
  it represents the easiest path to approach Clojure;
* [The Joy of Clojure][31]: the title speaks by itself. A must read
* [ClojureScript Up and Running][32]: at the moment, it's the only
  available book on ClojureScript. Due to the youth of the
  ClojureScript language, the book is now a little bit outdated. It's
  so brief that you can read it in a very short time with some profit,
  specially if you want to integrate external JavaScript libraries in
  your code.
* [**The Annotated Clojure Reference Manual** by Rich Hickey][33]: The
  most missed book on Clojure :).

## Tooling

More people asked me which operating system and editor/IDE are most
appropriated for developing Clojure/ClojureScript code. I personally
use Mac OS X and Ubuntu. On top of those I use Emacs with the standard
stuff to be productive with Clojure programming. Because I'm aged,
*nix and Emacs are the OS/Editor I know better. That said, in this
series you're not going to find any suggestion about any OS or
IDE/Editor. Use whatever tool fits better your habits. I have too much
respect for people developing IDE/plugins for Clojure/ClojureScript to
say that one is better than another.

# Introduction

This series of tutorials will guide you in creating, setting up and
running simple CLJS projects. The series follows a progressive
enhancement of the project itself.

> NOTE 1: I suggest to code yourself the content of the series. In my
> experience it is always the best choice if you are not already fluent
> with the programming language you have under your fingers.

That said, assuming you already have installed [leiningen 2][2], to
run the latest available tutorial without coding:

1. `git clone https://github.com/magomimmo/modern-cljs.git`
2. `cd modern-cljs`
3. `lein cljx once # used from tutorial-16 forward`
4. `lein ring server-headless`
5. open a new terminal and cd in the modern-cljs main directory
6. `lein cljsbuild once`
7. `lein trampoline cljsbuild repl-listen`
8. visit [login-dbg.html][3] and/or [shopping-dbg.html][4]
9. play with the repl connected to the browser

> NOTE 2: If you want to access the code of any single tutorial because
> you don't want to `copy&paste` it or you don't want to write it
> yourself, do as follows:
>
> * `git clone https://github.com/magomimmo/modern-cljs.git`
> * `cd modern-cljs`
> * `git checkout tutorial-01 # for tutorial 1, tutorial-02 for tutorial 2 etc `


## [Tutorial 3 - CLJ based http-server][7]

In this tutorial you are going to substitute the external http-server
with [ring][8], a CLJ based http-server.

## [Tutorial 4 - Modern ClojureScript][9]

In this tutorial we start having some fun with CLJS form validation, by
porting from JS to CLJS the login form example of
[Modern Javascript: Development and design][10] by [Larry Ullman][11].

## [Tutorial 5 - Introducing Domina][12]

In this tutorial we're going to use [domina library][13] to make our
login form validation more clojure-ish.

## [Tutorial 6 - Easy made Complex and Simple made Easy][14]

In this tutorial we're going to investigate and solve in two different
ways the not so nice issue we met in the last tutorial.

##  [Tutorial 7 - On being doubly aggressive][15]

In this tutorial we're going to explore CLJS/CLS compilation modes by
using the usual `lein-cljsbuild` plugin of `leiningen`, but we'll
discover a trouble we'll solve by using a new feature of the `0.3.0` of
`lein-cljsbuild` plugin.

## [Tutorial 8 - Introducing Domina events][16]

In this Tutorial we're going to introduce domina events which, by
wrapping Google Closure Library event management, allows to follow a
more clojure-ish approach in handing DOM events.

## [Tutorial 9 - DOM Manipulation][17]

In this tutorial we're going to face the need to programmatically
manipulate DOM elements as a result of the occurrence of some DOM
events.

## [Tutorial 10 - Introducing Ajax][18]

In this tutorial we're going to extend our comprehension of CLJS by
introducing Ajax to let the CLJS client-side code to communicate with
the CLJ server-side code.

## [Tutorial 11 - A deeper understanding of Domina events][19]

In this tutorial we're going to enrich our understanding of Domina
events by applying them to the `login form` example we introduced in
the [4th Tutorial][9].

## [Tutorial 12 - The highest and the deepest layers][20]

In this tutorial we're going to cover the highest and the deepest
layers of the Login Form example we started to cover in the
[previous tutorial][20].

## [Tutorial 13 - Don't Repeat Yourself while crossing the border][21]

One of our long term objectives is to eliminate any code duplication
from our web applications.  That's like to say we want firmly stay as
compliant as possible with the Don't Repeat Yourself (DRY)
principle. In this tutorial we're going to respect the DRY principle
by sharing validators between the client (i.e. CLJS) and the server
(i.e. CLJ).

## [Tutorial 14 - It's better to be safe than sorry (Part 1)][22]

In this tutorial we are going to prepare the stage for affording the
unit testing topic. We'll also introduce the `Enlive` template sytem
to implement a server-side only version of the Shopping Calculator
aimed at adhering to the progressive enanchement implementation
strategy. We'll even see how to exercize code refactoring to satisfy
the DRY principle and to solve a cyclic namespaces dependency problem.

## [Tutorial 15 - It's better to be safe than sorry (Part 2)][23]

In this tutorial, after having added the validators for the
`shoppingForm`, we're going to introduce unit testing.

## [Tutorial 16 - It's better to be safe than sorry (Part 3)][24]

In this tutorial we make unit testing portable from CLJ to CLJS (and
vice-versa) by using the `clojurescript.test` lib and the `cljx` lein
plugin.

## [Tuturial 17 - Enlive by REPLing][25]

To be respectful with the progressive enhancement strategy, in this
tutorial we're going to integrate the form validators for the
server-side Shopping Calculator into the corresponding WUI (Web User
Interface) in such a way that the user will be notified with the
right error messages when the she/he types in invalid values
in the form.

## [Tutorial 18 - Housekeeping][26]

In this tutorial we're going to digress about two topics. The setup of
a more comfortable browser REPL based on nREPL. The setup of a more
comfortable project structure by using the `profiles` features of
[Leiningen][2]

# License

Copyright © Mimmo Cosenza, 2012-2013. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://github.com/clojure/clojurescript.git
[2]: https://github.com/technomancy/leiningen
[3]: http://localhost:3000/login-dbg.html
[4]: http://localhost:3000/shopping-dbg.html
[5]: https://github.com/TranslateBabelJapan/modern-cljs/blob/japanese-translate/doc/tutorial-01.md
[6]: https://github.com/TranslateBabelJapan/modern-cljs/blob/japanese-translate/doc/tutorial-02.md
[7]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-03.md
[8]: https://github.com/mmcgrana/ring.git
[9]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-04.md
[10]: http://www.larryullman.com/books/modern-javascript-develop-and-design/
[11]: http://www.larryullman.com/
[12]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-05.md
[13]: https://github.com/levand/domina
[14]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-06.md
[15]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-07.md
[16]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-08.md
[17]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-09.md
[18]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-10.md
[19]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-11.md
[20]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-12.md
[21]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-13.md
[22]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-14.md
[23]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-15.md
[24]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-16.md
[25]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-17.md
[26]: https://github.com/magomimmo/modern-cljs/blob/master/doc/tutorial-18.md
[27]: http://mitpress.mit.edu/sicp/full-text/book/book.html
[28]: http://www.paulgraham.com/onlisp.html
[29]: http://www.clojurebook.com/
[30]: http://pragprog.com/book/shcloj2/programming-clojure
[31]: http://joyofclojure.com/
[32]: http://shop.oreilly.com/product/0636920025139.do
[33]: https://twitter.com/richhickey
[34]: http://www.larryullman.com/books/modern-javascript-develop-and-design/
