# Tutorial 3 - House Keeping

[前のチュートリアルで][1]は、`boot`ビルドツールを使用し、コミュニティによって開発されたいくつかの追加タスクを配備することで、即時フィードバックの原則にアプローチました。

* `boot-cljs`: CLJSのソースコードをコンパイルする（[チュートリアル1][2]で紹介）。
* `boot-http`: CLJベースのWebサーバーを実行してページを提供する。
* `boot-reload`: 変更が保存されたときに静的リソースをリロードする。
* `boot-cljs-repl`: CLJS REPLをブラウザのJSエンジン（bREPL）に接続します。

このチュートリアルでは、ターミナルに送信される`boot`コマンドの長さを最小限に抑えることによって、`boot`ビルドツールとの開発者とのやりとりを改善し、プログラミングの即時フィードバックスタイルをサポートします。

## Preamble

[前のチュートリアル][1]の最後から作業を開始したい場合は、`git`がインストールされていると仮定して、次のようにします。

```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout se-tutorial-02
```

## Introduction

即座にフィードバックの原則にアプローチするために以前端末に投入した`boot`コマンドを見直してみましょう。

### CLJS compilation

まず、[チュートリアル1][2]では、いくつかの環境変数を設定しました。 すなわち、`:source-paths`と`:resource-paths`です。

次に、次の`boot`コマンドを使用してCLJSコンパイルを開始しました。

```bash
boot cljs target
```

利用可能ないくつかのデフォルトを利用して、コンパイルオプションをタスクに渡しませんでした。 すなわち：

* `none` コンパイラ最適化として;
* `source-map` `none`の最適化ではデフォルトは`true`
* `main.js` コンパイラによって生成されたJSファイルの名前

    > NOTE 1: ClojureScriptはHTMLソースマップをサポートしているため、設定オプション`source-map`を使用してブラウザでClojureScriptを直接デバッグできます。 最適化を`none`に設定すると、有効な値は`true`と`false`になり、デフォルトは`true`になります。 他の最適化設定では、ソースマップを書き込む場所（[ClojureScript Compiler Optimizations][3]から取得）へのパスを指定する必要があります。

### HTTP server

[チュートリアル2][1]では、`boot`コマンドに`serve`タスクを追加することから始めました。 `-d target`オプションを渡して、タスクにserveするディレクトリを指定します。これは、`target`タスクがデフォルトとして使用するのと同じディレクトリ名です。 覚えているように、Webサーバーを稼働させ続ける`wait`タスクを追加する必要もありました。

```bash
boot wait serve -d target cljs target
```

### CLJS recompilation

変更されたCLJSソースファイルを保存するたびにCLJSの再コンパイルをトリガするために、`wait`タスクを`cljs`タスクの前に配置する必要がある`watch`タスクに置き換えました。

```bash
boot serve -d target watch cljs target
```

### boot-reload

繰り返しますが、静的リソースのリロードをトリガするために、変更や変更がCLJSソースコードに保存されたときにリンクするように、`cljs`タスクの直前にパイプラインに`reload`タスクを追加しました。

```bash
boot serve -d target watch reload cljs target
```

### bREPL

最後に、ブラウザベースREPL（bREPL）を提供して即時フィードバックの原則にアプローチするという目的をほぼ完成させるために、`cljs-repl`タスクを追加する必要がありました。 `cljs-repl`タスクを`cljs`に追加するのを忘れないでください。

```bash
boot serve -d target watch reload cljs-repl cljs target
```

あなたがプロジェクトを進めるために覚えておかなければならないことに不平を言うならば、あなたは正しいです。

このため、`boot`コアの開発者は、タイピングと記憶を手助けする非常に快適な方法を提供します。

## Enter deftask

ユーザーの観点から見ると、`boot`の興味深い側面の1つは、`boot`によって事前に定義されているかどうかにかかわらず、タスクの構成可能な性質です。 そのアーキテクチャをより良く理解するために、`boot`ソースコードを勉強するのに時間を費やすことができますが、私たちは実用的になりたいです。 現時点では、ターミナルから`boot`コマンドを起動している間に、タスク名と順序を記憶する必要性を減らすことにのみ関心があります。

以下のように`build.boot`を開き、`deftask`マクロを使用して、他のタスクの順序付けられた構成として新しいタスクを定義するだけです。

```clj
(set-env!
  ...
  ...)

(require ...
         ...)

;; define dev task as composition of subtasks
(deftask dev
  "Launch Immediate Feedback Development Environment"
  []
  (comp
   (serve :dir "target")
   (watch)
   (reload)
   (cljs-repl) ;; before cljs task
   (cljs)
   (target :dir #{"target"})))
```

`deftask`マクロを使用して、新しく作成された`dev`の本体に`comp`関数を使用することに注意してください。 `comp`関数は、`boot`コマンドで見たのと同じ順序で同じタスクを作成します。 唯一の違いは、 `"target"`値が`serve`へ渡される方法と、コマンドラインで引数としてではなく`:dir`キーワード引数として`target`サブタスクに渡す方法です。 さらに、`target`スクに渡される値は、単なる文字列（すなわち、`"target"`）ではなく、文字列のセット（すなわち、`#{"target"}`）です。 現時点では、`boot` が複数のディレクトリに出力を生成する可能性があると思われる場合でも、この選択の理由を理解することには興味がありません。

新しく定義された`dev`タスクは、コマンドラインで使用できるようになりました。

```bash
boot -h
...
         dev                        Launch Immediate Feedback Development Environment
...
Do `boot <task> -h` to see usage info and TASK_OPTS for <task>.
```

コマンドラインから通常どおり`dev`タスクを起動して、以前にサブタスクを作成することで得られたのと同じ効果を得ることができます：

```bash
boot dev
Starting reload server on ws://localhost:60083
Writing boot_reload.cljs...
Writing boot_cljs_repl.cljs...
2015-11-04 08:58:55.742:INFO:oejs.Server:jetty-7.6.13.v20130916
2015-11-04 08:58:55.788:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:3000
<< started Jetty on http://localhost:3000 >>

Starting file watcher (CTRL-C to quit)...

nREPL server started on port 60086 on host 127.0.0.1 - nrepl://127.0.0.1:60086
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
Elapsed time: 19.929 sec
```

次に、nreplクライアントからのbREPL作成を含む、[チュートリアル2][1]ですでに示されているすべてのテストを繰り返すことができます。

悪くない。 即時フィードバックの原理にアプローチする開発環境を立ち上げるための非常に簡単なコマンド`boot dev`があります。

一通り試し終えたら、すべてを終了します。

## Defaults overwriting

当初から述べたように、私たちは、プロジェクトの構造レイアウトを簡素化する目的で、`boot`タスクのデフォルトに可能な限りこだわろうとしました。 つまり、たとえ初期設定のような非常にシンプルなプロジェクトにデフォルト設定が採用されていても、プロジェクトがより明瞭になるとすぐにそれらを上書きする必要があります。

ほとんどのWebアプリケーションは、HTML/CSS/JSリソースのそれぞれを分離しています。 例を挙げると、ほとんどのWebアプリケーションは、JSリソースを、Webサーバーが提供するディレクトリにある`js`サブディレクトリの下にグループ化し、対応する`<script>`htmlタグの`js`リソースをリンクします。

最初に`html/index.html`ファイルを編集し、CLJSのコンパイル・フェーズで生成された`main.js`ファイルへのリンクを変更して、この規約に準拠させてみましょう。

```html
<!doctype html>
<html>
  <head>
    <title>Hello, World!</title>
  </head>
  <body>
    <script src="js/main.js"></script>
  </body>
</html>
```

If you take a look at the [clojurescript compiler options][4] you'll
discover that you need two options to be passed to the CLJS compiler
to support this new files organization:

* [`:output-to`][5]: sets the path to the JavaScript file that will be
  output
* [`:asset-path`][6]: sets a path relative to the directory served by
  the web server.

How could we pass these options to the `boot-cljs` which launches the
CLJS compiler? You could easily go crazy trying to solve
this very simple problem by reading the `boot-cljs`
documentation. Some compiler options can be set at the task level,
others can be set by defining appropriate [`edn`][8] files located in
the right position in the resources directories (e.g. into the `html`
directory in our project).

Let's not go into too many details about the reasons that `boot-cljs` supports
the compiler options the way it does; instead, let's be pragmatic again and
just list what you need to do:

First you have to create the `js` subdirectory in the `html` directory
containing the html pages.

```bash
cd pathname/to/modern-cljs
mkdir html/js
```

Then create a `cljs.edn` file in the newly created `js` directory
reflecting the name of the JS file generated by the CLJS compiler.

```bash
touch html/js/main.cljs.edn
```

Now edit that file as follows:

```clj
{:require [modern-cljs.core]
 :compiler-options {:asset-path "js/main.out"}}
```

As you see we just required the `modern-cljs.core` namespace of the
project and set the `asset-path` option to the CLJS compiler. If the `output-to` option is not specified default value is created based on [relative path of the .cljs.edn file][9]. This way we have completed configuration for CLJS compiler. That's it.

You can now run the CLJS compilation. To see the very verbose output
of the CLJS compilation task, use the `-vv` option of the `boot`
command as follows:

```bash
boot -vv cljs target
...
Compiling ClojureScript...
...
• js/main.js
CLJS options:
{:asset-path "js/main.out",
 :output-dir
 "/Users/mimmo/.boot/cache/tmp/Users/mimmo/tmp/modern-cljs/3c5/-w7zxdu/js/main.out",
 :output-to
 "/Users/mimmo/.boot/cache/tmp/Users/mimmo/tmp/modern-cljs/3c5/-w7zxdu/js/main.js",
 :main boot.cljs.main583}
...
```

This way you can verify that the CLJS compiler correctly generated the
`main.js` file in the `js` subdirectory relative to the directory
served by the web server and that it sets the `js/main.out`
subdirectory as the value of the `asset-path` compiler option.

Now take a look at the layout of the `target` directory to confirm
that the compiler options worked as expected

```bash
target
├── index.html
└── js
    ├── main.cljs.edn
    ├── main.js
    └── main.out
        ├── boot
        ...
        └── modern_cljs
            ├── core.cljs
            ├── core.cljs.cache.edn
            ├── core.js
            └── core.js.map
```

Finally, repeat all the manual tests we introduced in the
[Tutorial 2][1] to verify that the Immediate Feedback Development
Environment is still working.

Launch the `boot dev` command:

```bash
boot dev
Starting reload server on ws://localhost:52434
Writing boot_reload.cljs...
Writing boot_cljs_repl.cljs...
2015-11-04 17:26:11.291:INFO:oejs.Server:jetty-7.6.13.v20130916
2015-11-04 17:26:11.445:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:3000
<< started Jetty on http://localhost:3000 >>

Starting file watcher (CTRL-C to quit)...

Adding :require adzerk.boot-reload to main.cljs.edn...
nREPL server started on port 52437 on host 127.0.0.1 - nrepl://127.0.0.1:52437
Adding :require adzerk.boot-cljs-repl to main.cljs.edn...
Compiling ClojureScript...
• js/main.js
Writing target dir(s)...
Elapsed time: 25.922 sec
```

Now launch the `nrepl` client in a new terminal:

```clj
boot repl -c
REPL-y 0.3.7, nREPL 0.2.12
Clojure 1.7.0
Java HotSpot(TM) 64-Bit Server VM 1.8.0_66-b17
        Exit: Control+D or (exit) or (quit)
    Commands: (user/help)
        Docs: (doc function-name-here)
              (find-doc "part-of-name-here")
Find by Name: (find-name "part-of-name-here")
      Source: (source function-name-here)
     Javadoc: (javadoc java-object-or-class-here)
    Examples from clojuredocs.org: [clojuredocs or cdoc]
              (user/clojuredocs name-here)
              (user/clojuredocs "ns-here" "name-here")
boot.user=>
```

Then launch the `bREPL`:

```clj
boot.user=> (start-repl)
<< started Weasel server on ws://127.0.0.1:52439 >>
<< waiting for client to connect ... Connection is ws://localhost:52439
Writing boot_cljs_repl.cljs...
```

An finally visit the `http://localhost:3000` URL to activate the bREPL

```
 connected! >>
To quit, type: :cljs/quit
nil
cljs.user=>
```

Play around as you like. When finished, quit everything and reset your
git branch.

```bash
git reset --hard
```

## Next step - [Tutorial 4: Modern ClojureScript][7]

In the [next tutorial 4][7] we're going to have some fun introducing
form validation in CLJS.

# License

Copyright © Mimmo Cosenza, 2012-2017. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-02.md
[2]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-01.md
[3]: https://github.com/clojure/clojurescript/wiki/Compiler-Options#source-map
[4]: https://github.com/clojure/clojurescript/wiki/Compiler-Options
[5]: https://github.com/clojure/clojurescript/wiki/Compiler-Options#output-to
[6]: https://github.com/clojure/clojurescript/wiki/Compiler-Options#asset-path
[7]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-04.md
[8]: https://github.com/edn-format/edn
[9]: https://github.com/boot-clj/boot-cljs/blob/master/docs/compiler-options.md
