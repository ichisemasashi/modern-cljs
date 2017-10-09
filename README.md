## Latest tutorials

### [Tutorial 22 - A reasoned porting of the Official React Tutorial to Reagent - Part II][47]

Part II of a step by step reasoned porting of the [Official React Tutorial][48] to [Reagent][49].

### [Tutorial 21 - A reasoned porting of the Official React Tutorial to Reagent - Part I][46]

Part I of a step by step reasoned porting of the [Official React Tutorial][48] to [Reagent][49].

### [Tutorial 20 - House Keeping][40]

Step by step guide for publishing a library to clojars repository by
using `boot`.

# Modern ClojureScript

> **ATTENTION NOTE**: [第2版][1]を公開する途中です。 [初版][2]との主な違いは、[Leiningen][4]の代わりに[Boot][3]ビルドツールを使用することです。 この第2版は依然としてドラフト版であり、エラー、誤植、またはコード内のバグが見つかった場合は、寛容でなければなりません。

> **WARNING NOTE FOR WINDOWS USERS**: 現時点では、bootは10未満のMS Windowsでは実行しません。これに当てはまる場合は、modern-cljsシリーズに従うために、[仮想マシン](https://www.virtualbox.org/)または[docker Linuxコンテナ](https://docs.docker.com/windows/)を使用してください。

モダンClojureScript（modern-cljs）は、[ClojureScript][5]（CLJS）プロジェクトの作成と実行をガイドする一連のチュートリアルです。

CLJSはJavaScriptをターゲットとするClojureプログラミング言語のコンパイラです。 これは、Webブラウザやその他のクライアントサイドまたはサーバーサイドのJavaScriptインタプリタ（[nodej][6]など）で動作するJavaScriptコードを生成します。

## Required background

このチュートリアルでは、事前のプログラミング経験が必要です。 チュートリアルはあなたがまだClojureに堪能ではない場合でも、少しはClojureを試していると仮定します。 また、HTML、JavaScript、ブラウザDOMを使用してWebをプログラミングする経験がある場合には、非常に役立ちます。

Clojure（またはLispについて）について何も知らなければ、これらのチュートリアルを始める前にちょっと勉強することをお勧めします。

Clojureには、インターネット上で自由に利用できる優れたリソースがたくさんあります。プログラマとしてあなたの価値のためにClojure（または他のLispの方言）の本を読むことの利点を過大評価することはできません。

いくつかの本をお薦めします。

* [Clojure Programming][7]:
  Clojureの3人のヒーローによって書かれた。Clojureとそのエコシステムについて知る必要があるすべてを含んでいます。
* [Programming Clojure][8]: 他の伝説のClojure開発者が書いた、Clojureを学ぶ最も簡単な方法です。
* [The Joy of Clojure][9]: タイトルが内容を語っている。 読む必要があります！
* [ClojureScript Rationale][41] and [ClojureScript Quick Start][42]
* [ClojureScript Up and Running][10]: 現時点では、ClojureScriptに関する唯一の出版された本です。 ClojureScriptは進化が速いので、この本は少し古くなっています。 短くて便利です。特に、外部のJavaScriptライブラリと統合したい場合は。
* [SICP - Structure and Interpretation of Computer Programs][11]: これは私が非常に長いキャリアで読んだ最高のプログラミング本です。 これは、Clojureではなく[Scheme/Racket][12]（Lispの方言）を使用しており、[オンライン版][13]、[印刷版][13]、または[講義シリーズ][14]で利用可能です。
* [On Lisp][15]: マクロについて知りたければ、これが始める場所です。 ClojureではなくCommon Lisp（Lisp方言）を使用します。

## Required tools

多くの人がClojureとClojureScriptで開発するのに最適なオペレーティングシステムとエディタ/IDEについて心配しています。 私は個人的にはMac OS X、Debian、Ubuntuを使います。 Emacsをエディタとして使用しています。

私は古い人間なので、[*nix][43]とEmacsは私が最もよく知っているOSとエディタです。 つまり、この一連のチュートリアルでは、オペレーティングシステムやエディタに関する提案や参照はありません。 すでに持っていて知っているツールを使用してください。 私はClojure/CLJSのためのIDE/プラグインを開発している人たちを尊敬しているので、あるものが他のものよりも優れていると言うことができませんし、あなたは新しいプログラミング言語の学習と新しいプログラミング環境の学習を組み合わせたくありません。

> NOTE: Emacsについてもっと学びたいと思っている人は、[あなたが始めるのを助けるためのリソース](https://github.com/magomimmo/modern-cljs/blob/master/doc/supplemental-material/emacs-cider-references.md).

[git][16]と[Java][34]がインストールされている必要があります。また、[gitの基本][17]に精通している必要があります。

## Clojure community documentation

コミュニティが作ったあなたの役に立つであろうClojureのドキュメント・サイトは、[clojuredocs](http://clojuredocs.org/) と [Grimoire](https://www.conj.io/).

## Libraries and tools

[Clojure Toolbox](http://www.clojure-toolbox.com/)は、CLJ / CLJS用のライブラリとツールのディレクトリです。

## Why the name Modern ClojureScript?

このチュートリアルシリーズがなぜmodern-cljsという名前になったのか、ClojureScriptが最近のものであるのか疑問に思うかもしれません。 私が2012年にこのシリーズを開始したとき、[モダンJavaScript：Develop and Design][18] bookからClojureScriptへ例題をいくつか移植しようとしていました。今は変更するのが遅すぎます。

# The Tutorials

言いましたように、これはシリーズの[第2版][1]で、[Boot][3]ビルドツールに基づいています。 私は[Leiningen][4]ビルドツールに基づいたシリーズの[初版][2]を更新したり、サポートしたりするつもりはありません。

## Introduction

この一連のチュートリアルでは、簡単なCLJSプロジェクトの作成と実行について説明します。 一連のシリーズは、1つのプロジェクトの段階的な強化をしてゆきます。

チュートリアルを通して作業しながら、チュートリアル1から始めて、各チュートリアルのすべてのコードを自分で入力することを*強く*お勧めします。 私の経験では、これはプログラミング言語に習熟していない方にとって最良のアプローチです。

## [Tutorial 1 - The Basics][19]

非常に基本的なCLJSプロジェクトを作成して設定します。

## [Tutorial 2 - Immediate Feedback Principle][20]

非常にインタラクティブな開発環境を構築するために、Bret Victorの即時フィードバックの原則に可能な限り近づく。

## [Tutorial 3 - House Keeping][21]

即時フィードバック開発環境（IFDE）にアプローチするため、bootコマンドの起動を自動化します。

## [Tutorial 4 - Modern ClojureScript][22]

[モダンJavaScript：Develop and Design][18]のJavaScriptログインフォームの例をCLJSへ移植することで、CLJSフォームの検証を楽しくしてください。

## [Tutorial 5 - Introducing Domina][23]

ログイン・フォームをよりClojureらしくするために[Domina library][24]を使う。

## [Tutorial 6 - The Easy Made Complex, and the Simple Made Easy][25]

最新のチュートリアルから問題を解決するための2つの異なる方法を調査して見つけます。

## [Tutorial 7 - Introducing Domina Events][26]

DOMイベントを処理を、よりClojureらしくアプローチするため、Dominaイベントを使用する。

## [Tutorial 8 - DOM Manipulation][27]

DOMイベントに応答するDOM要素をプログラム的に操作します。

## [Tutorial 9 - Introducing AJAX][28]

AJAXを使用して、CLJSクライアント側のコードをサーバーと通信させます。

## [Tutorial 10 - A Deeper Understanding of Domina Events][29]

[第4回チュートリアル][22]のログインフォームの例にDominaのイベントを適用します。

## [Tutorial 11 - HTML on Top, Clojure on the Bottom][30]

前のチュートリアルのログインフォームの例の高い（HTML5）層と深い（サーバー上のClojure）の層を調べます。

## [Tutorial 12 - Don't Repeat Yourself][31]

クライアント側のCLJSとサーバー側のClojureとの間でバリデータを共有することにより、[DRY][44]原則を守ります。

## [Tutorial 13 - Better Safe Than Sorry (Part 1)][32]

`Enlive`テンプレートシステムについて学習し、ショッピング電卓の例を開始することで、単体テストの段階を設定します。 [DRYの原則][44]を満たし、循環する名前空間の依存関係の問題を解決するために、コードのリファクタリングを使用します。

## [Tutorial 14 - Better Safe Than Sorry (Part 2)][33]

`shoppingForm`にバリデータを追加し、単体テストを行います。

## [Tutorial 15 - Better Safe Than Sorry (Part 3)][35]

Bret Victorよる即時フィードバックの原則と[テスト駆動開発環境（TDD）][45]を1つのJVMで同時に満たす開発環境を構成します。

## [Tutorial 16 - On pleasing TDD practitioners][36]

[テスト駆動開発環境][45]をよりカスタマイズ可能にする。

## [Tutorial 17 - REPLing with Enlive][37]

フォームに無効な値を入力すると、対応するヘルプメッセージがユーザーに通知されるように、バリデータをWebフォームに統合します。

## [Tutorial 18 - Augmented TDD session][38]

CLJ/CLJSのREPLで拡張された[TDD][45]環境を利用して、クライアント側フォームのバリデーションを完成させます。

## [Tutorial 19 - Livin' on the edge][39]

CLJ/CLJSコンパイラの新しいReader Conditionals拡張機能に準拠させる方法を説明します。

## [Tutorial 20 - House Keeping][40]

`boot`を使用してライブラリをclojarリポジトリに公開するためのステップバイステップガイド。

## [Tutorial 21 - A reasoned porting of the Official React Tutorial to Reagent - Part I][46]

パート1、ステップバイステップで[公式Reactチュートリアル][48]を[Reagent][49]に移植しました。

## [Tutorial 22 - A reasoned porting of the Official React Tutorial to Reagent - Part II][47]

パート2、ステップバイステップで[公式Reactチュートリアル][48]を[Reagent][49]に移植しました。

# License

Copyright © Mimmo Cosenza, 2012-2016. Released under the Eclipse Public
License, the same license as Clojure.


[1]: https://github.com/magomimmo/modern-cljs/tree/master/doc/second-edition
[2]: https://github.com/magomimmo/modern-cljs/tree/master/doc/first-edition
[3]: https://github.com/boot-clj/boot
[4]: http://leiningen.org/
[5]: https://github.com/clojure/clojurescript.git
[6]: https://github.com/clojure/clojurescript/wiki/Quick-Start#running-clojurescript-on-nodejs
[7]: http://www.clojurebook.com/
[8]: http://pragprog.com/book/shcloj2/programming-clojure
[9]: http://www.joyofclojure.com/
[10]: http://shop.oreilly.com/product/0636920025139.do
[11]: http://mitpress.mit.edu/sicp/
[12]: http://racket-lang.org/
[13]: http://mitpress.mit.edu/sicp/full-text/book/book.html
[14]: http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/index.htm
[15]: http://www.paulgraham.com/onlisp.html
[16]: http://git-scm.com/
[17]: http://git-scm.com/documentation
[18]: http://www.larryullman.com/books/modern-javascript-develop-and-design/
[19]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-01.md
[20]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-02.md
[21]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-03.md
[22]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-04.md
[23]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-05.md
[24]: https://github.com/levand/domina
[25]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-06.md
[26]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-07.md
[27]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-08.md
[28]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-09.md
[29]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-10.md
[30]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-11.md
[31]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-12.md
[32]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-13.md
[33]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-14.md
[34]: https://github.com/clojure/clojurescript.git
[35]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-15.md
[36]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-16.md
[37]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-17.md
[38]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-18.md
[39]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-19.md
[40]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-20.md
[41]: https://github.com/clojure/clojurescript/wiki/Rationale
[42]: https://github.com/clojure/clojurescript/wiki/Quick-Start
[43]: https://en.wikipedia.org/wiki/Unix-like
[44]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[45]: https://en.wikipedia.org/wiki/Test-driven_development
[46]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-21.md
[47]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-22.md
[48]: https://facebook.github.io/react/docs/tutorial.html
[49]: http://reagent-project.github.io/
