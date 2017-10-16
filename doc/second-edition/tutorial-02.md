# Tutorial 2 - Immediate Feedback Principle

このチュートリアルでは、Bret Victorが彼の[トーク][1]で述べたように、即時フィードバックの原則に沿うようにClojureScriptプロジェクトを構成することを目的としています。

## Preamble (前文)

[前のチュートリアル][2]の最後から作業を開始する場合は、[git][3]がインストールされていると仮定して、次のようにします。

```bash
git clone https://github.com/magomimmo/modern-cljs.git
cd modern-cljs
git checkout se-tutorial-01
```

これは、チュートリアルのリポジトリをcloneし、第一回のチュートリアルの最後から始めます。

> NOTE 1: `se-`は「second edition」の意味です。

## Introduction

`boot`ビルドツールは、CLJ開発者のための標準の一種です。`leiningen`ビルドツールと比べて、新しくてまだ成熟していません。 しかし、`boot`コミュニティは、ギャップを埋め、そしておそらくは追い越すことを目的として、機能、`boot`の用語の *タスク* を徐々に充実させるために熱心に働いています。

コミュニティによって開発された[bootのtask][4]を見てみると、Bret Victorの即時フィードバックの原則に従うために必要なものがすべてあることがわかります。

* [`boot-http`][5]: 単純なCLJベースのHTTPサーバを提供する`boot`のtask。
* [`boot-reload`][6]: 静的リソース（CSS、画像など）のライブリロードを提供する`boot`のtask。
* [`boot-cljs-repl`][7]: CLJS開発のためのREPLを提供する`boot`のtask。
    > NOTE 2: 既に前のチュートリアルで`boot-cljs`タスクを使用しました。

## CLJ-based HTTP server

まず、`boot-http`サーバを、`modern-cljs`ホームディレクトリにある`build.boot`ファイルに追加してみましょう。

```clj
(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[adzerk/boot-cljs "1.7.228-2"]
                 [pandeiro/boot-http "0.7.6"]         ;; add http dependency
                 [org.clojure/tools.nrepl "0.2.12"]]) ;; required by boot-http

(require '[adzerk.boot-cljs :refer [cljs]]
         '[pandeiro.boot-http :refer [serve]]) ;; make serve task visible
```

> NOTE 3: 現在の `0.7.6`バージョンの` boot-http`のバグのために `tools.nrepl`の` 0.2.12`バージョンも追加しなければなりませんでした。

ご覧のとおり、最新の`boot-http`のリリースをプロジェクトの依存関係に追加し、`boot`コマンドを`require`フォームで参照することで、その`serve`タスクを利用することができます。

私たちは依然としていくつかの`boot`デフォルトを暗黙的に利用していることに注意してください。

* `boot.properties`ファイルで定義されているClojure 1.7.0の使用。
* 暗黙的に`boot-cljs`依存関係によってインポートされたClojureScript 1.7.228の使用。

いつものように、新しく追加された`serve`タスクのヘルプドキュメントを見てみましょう：

```bash
boot serve -h
Start a web server on localhost, serving resources and optionally a directory.
Listens on port 3000 by default.

Options:
  -h, --help                Print this help info.
  -d, --dir PATH            PATH sets the directory to serve; created if doesn't exist.
  -H, --handler SYM         SYM sets the ring handler to serve.
  -i, --init SYM            SYM sets a function to run prior to starting the server.
  -c, --cleanup SYM         SYM sets a function to run after the server stops.
  -r, --resource-root ROOT  ROOT sets the root prefix when serving resources from classpath.
  -p, --port PORT           PORT sets the port to listen on. (Default: 3000).
  -k, --httpkit             Use Http-kit server instead of Jetty
  -s, --silent              Silent-mode (don't output anything)
  -t, --ssl                 Serve via Jetty SSL connector on localhost on default port 3443 using cert from ./boot-http-keystore.jks
  -T, --ssl-props SSL       SSL sets override default SSL properties e.g. "{:port 3443, :keystore "boot-http-keystore.jks", :key-password "p@ssw0rd"}".
  -R, --reload              Reload modified namespaces on each request.
  -n, --nrepl REPL          REPL sets nREPL server parameters e.g. "{:port 3001, :bind "0.0.0.0"}".
  -N, --not-found SYM       SYM sets a ring handler for requested resources that aren't in your directory. Useful for pushState.
```

`-d`オプションは、提供されるディレクトリを設定するために使用されます。 それが存在しなければ作成されます。 ターミナルで次の`boot`コマンドを試してみましょう：

```bash
boot serve -d target
2015-10-26 21:38:48.489:INFO:oejs.Server:jetty-7.6.13.v20130916
2015-10-26 21:38:48.549:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:3000
<< started Jetty on http://localhost:3000 >>
```

httpサーバ（Jetty）の起動後にコマンドが終了することに注意してください。 ブラウザのURLバーに`http://localhost:3000`と入力すると、エラーが発生します。 これは、`serve`タスクがブロックされないためです。

この問題を解決するには、すでに`boot`に含まれている定義済みの`wait`タスクを追加する必要があります。

```bash
boot wait -h
Wait before calling the next handler.

Waits forever if the --time option is not specified.

Options:
  -h, --help       Print this help info.
  -t, --time MSEC  Set the interval in milliseconds to MSEC.
```

この解決方法を見てみましょう：

```bash
boot wait serve -d target
2015-10-26 21:40:54.695:INFO:oejs.Server:jetty-7.6.13.v20130916
2015-10-26 21:40:54.772:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:3000
<< started Jetty on http://localhost:3000 >>
```

`boot`コマンドはもう終了しないので、ブラウザから`http://localhost:3000`に接続できます。 さあ、サーバーを停止してください（`CTRL-C`）。

`boot`タスクは簡単に連鎖させることができます。

```bash
boot wait serve -d target cljs target
2016-01-03 11:14:09.949:INFO::clojure-agent-send-off-pool-0: Logging initialized @7356ms
Directory 'target' was not found. Creating it...2016-01-03 11:14:10.011:INFO:oejs.Server:clojure-agent-send-off-pool-0: jetty-9.2.10.v20150310
2016-01-03 11:14:10.048:INFO:oejs.ServerConnector:clojure-agent-send-off-pool-0: Started ServerConnector@31c694ca{HTTP/1.1}{0.0.0.0:3000}
2016-01-03 11:14:10.049:INFO:oejs.Server:clojure-agent-send-off-pool-0: Started @7455ms
Started Jetty on http://localhost:3000
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
```

URL`http://localhost:3000`にアクセスし、ブラウザの開発者ツールを開いて文字列`Hello, World!`がコンソールに表示されていることを確認します。 次の手順に進む前に、現在のブートプロセス(`CTRL-C`)を終了してください。

## CLJS source recompilation

Immediate Feeback原則に従いたい場合は、`.cljs`ファイルを変更して保存するとすぐに、変更されたCLJSソースコードは再コンパイルされる必要があります。

`watch`タスクは、既に`boot`に含まれている多数の定義済みタスクのうちの別のものです。

```bash
boot watch -h
Call the next handler when source files change.

Options:
  -h, --help           Print this help info.
  -q, --quiet          Suppress all output from running jobs.
  -v, --verbose        Print which files have changed.
  -M, --manual         Use a manual trigger instead of a file watcher.
  -d, --debounce MS    MS sets debounce time (how long to wait for filesystem events) in milliseconds.
  -i, --include REGEX  Conj REGEX onto the set of regexes the paths of changed files must match for watch to fire.
  -e, --exclude REGEX  Conj REGEX onto the set of regexes the paths of changed files must not match for watch to fire.
```

CLJSソースコードの変更が保存されるたびにCLJSの再コンパイルの実行をトリガーするのとは別に、`watch`タスクはブロックもしていないため`wait`タスクに置き換えることもできます。

`cljs`タスクを呼び出す前に`watch`タスクを挿入するだけで、ソースの再コンパイルをトリガーできるはずです。

```bash
boot serve -d target watch cljs target
2017-03-09 23:07:18.959:INFO::clojure-agent-send-off-pool-0: Logging initialized @7461ms
2017-03-09 23:07:19.024:INFO:oejs.Server:clojure-agent-send-off-pool-0: jetty-9.2.10.v20150310
2017-03-09 23:07:19.075:INFO:oejs.ServerConnector:clojure-agent-send-off-pool-0: Started ServerConnector@693339d8{HTTP/1.1}{0.0.0.0:3000}
2017-03-09 23:07:19.080:INFO:oejs.Server:clojure-agent-send-off-pool-0: Started @7582ms
Started Jetty on http://localhost:3000

Starting file watcher (CTRL-C to quit)...

Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
Elapsed time: 8.787 sec
```

ブラウザーで`http://localhost:3000`にアクセスして、「Hello、World！」を確認してください。 JSコンソールに表示されています。 次に、好きなエディタで`src/cljs/modern_cljs/core.cljs`ソースファイルを開き、表示するメッセージを変更し、ファイルを保存します。 新しいCLJSコンパイルタスクがターミナルで起動されていることがわかります。

```bash
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
Elapsed time: 0.174 sec
```

最後に、htmlページをリロードして、リンクされた`main.js`ファイルがブラウザコンソールで表示されたメッセージを観察して更新されたことを確認します。 ここまでは順調ですね。

次の手順に進む前に、`boot`プロセスを終了してください(`CTRL-C`)。

## Resources reloading

Anytime you modify a CLJS source file you have to manually reload the
html page pointing to it to verify the effect of your coding and we
want to get much closer to the Immediate Feedback principle.

Luckily, there is a `boot` task developed by the community to automate
reloading of any static resource: [`boot-reload`][6]. Again we have
to add the new task to the dependencies of the project and make it
visible to `boot` by requiring its primary command:

```clj
(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[adzerk/boot-cljs "1.7.228-2"]
                 [pandeiro/boot-http "0.7.6"]
		             [org.clojure/tools.nrepl "0.2.12"]
                 [adzerk/boot-reload "0.5.1"]]) ;; add boot-reload

(require '[adzerk.boot-cljs :refer [cljs]]
         '[pandeiro.boot-http :refer [serve]]
         '[adzerk.boot-reload :refer [reload]]) ;; make reload visible
```

This task has to be inserted in the `boot` command immediately before
the `cljs` compilation. Give it a try:

```bash
boot serve -d target watch reload cljs target
Starting reload server on ws://localhost:58020
Writing boot_reload.cljs...
2016-01-03 11:22:21.323:INFO::clojure-agent-send-off-pool-0: Logging initialized @9526ms
2016-01-03 11:22:21.387:INFO:oejs.Server:clojure-agent-send-off-pool-0: jetty-9.2.10.v20150310
2016-01-03 11:22:21.410:INFO:oejs.ServerConnector:clojure-agent-send-off-pool-0: Started ServerConnector@3117f174{HTTP/1.1}{0.0.0.0:3000}
2016-01-03 11:22:21.411:INFO:oejs.Server:clojure-agent-send-off-pool-0: Started @9614ms
Started Jetty on http://localhost:3000

Starting file watcher (CTRL-C to quit)...

Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
Elapsed time: 8.281 sec
```

Now reload the usual URL in your browser and repeat the above
procedure by modifying the message to be printed in the browser
console. As before you'll see that as soon as you save the `core.cljs`
file the CLJS recompilation is triggered. This time, thanks to the
`boot-reload` task, the page is reloaded as well. You can confirm this
by seeing if the new message is printed in the browser's console.

You can even modify the html source file to obtain an almost immediate
feedback from the browser.

Nice stuff. Kill the `boot` command again (`CTRL-C`) before advancing to
the next level.

## Browser REPL (bREPL)

One of the main reasons to use a LISP dialect like CLJ is its REPL
(Read Eval Print Loop), which enables a very interactive style of
programming. CLJS communities worked very hard to bring the
same REPL-based programming experience to CLJS available in CLJ, and created a
way to connect a CLJS REPL to almost any JS engine, including browser-embedded
engines. This style of programming allows you to
evaluate CLJS forms in the REPL and receive an immediate feedback in
the browser to which the REPL is connected.

The `boot` community has a task to offer in this area, too. Its name is
`boot-cljs-repl`. As we have already done for the other tasks that `boot`
does not include, we need to add `boot-cljs-repl` to the dependencies of the
`build.boot` project file. Then, as usual, we have to require its primary tasks
(i.e. `cljs-repl` and `start-repl`) to make them visible to the `boot`
command at the terminal.

```clj
(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[adzerk/boot-cljs "1.7.228-2"]
                 [pandeiro/boot-http "0.7.6"]
                 [org.clojure/tools.nrepl "0.2.12"]
                 [adzerk/boot-reload "0.5.1"]
                 [adzerk/boot-cljs-repl "0.3.3"]])

(require '[adzerk.boot-cljs :refer [cljs]]
         '[pandeiro.boot-http :refer [serve]]
         '[adzerk.boot-reload :refer [reload]]
         '[adzerk.boot-cljs-repl :refer [cljs-repl start-repl]]) ;; make it visible
```

Again, issue the `boot cljs-repl -h` command if you want to read the
documentation on its advanced options.

```bash
boot cljs-repl -h
Retrieving boot-cljs-repl-0.3.3.pom from https://repo.clojars.org/ (1k)
Retrieving boot-cljs-repl-0.3.3.jar from https://repo.clojars.org/ (4k)
Start a ClojureScript REPL server.

The default configuration starts a websocket server on a random available
port on localhost.

Options:
  -h, --help                   Print this help info.
  -b, --ids BUILD_IDS          Conj [BUILD IDS] onto only inject reloading into these builds (= .cljs.edn files)
  -i, --ip ADDR                ADDR sets the IP address for the server to listen on.
  -n, --nrepl-opts NREPL_OPTS  NREPL_OPTS sets options passed to the `repl` task.
  -p, --port PORT              PORT sets the port the websocket server listens on.
  -w, --ws-host WSADDR         WSADDR sets the (optional) websocket host address to pass to clients.
  -s, --secure                 Flag to indicate whether the client should connect via wss. Defaults to false.
```

The `cljs-repl` task has to be positioned just before the `cljs` task.


The `cljs-repl` author also suggests being explicit about the `Clojure`
and `ClojureScript` releases to be added in the dependencies section of
the `build.boot` build file.

```clj
(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[org.clojure/clojure "1.8.0"]         ;; add CLJ
                 [org.clojure/clojurescript "1.9.473"] ;; add CLJS
                 [adzerk/boot-cljs "1.7.228-2"]
                 [pandeiro/boot-http "0.7.6"]
		             [org.clojure/tools.nrepl "0.2.12"]
                 [adzerk/boot-reload "0.5.1"]
                 [adzerk/boot-cljs-repl "0.3.3"]
                 ])

(require '[adzerk.boot-cljs :refer [cljs]]
         '[pandeiro.boot-http :refer [serve]]
         '[adzerk.boot-reload :refer [reload]]
         '[adzerk.boot-cljs-repl :refer [cljs-repl start-repl]])
```

Before to go on with experimenting the newly added `cljs-repl` task,
edit the version of clojure compiler used by `boot` itself in the `boot.properties`
file we introduced in the previous tutorial as follows:

```bash
#http://boot-clj.com
#Thu Mar 09 21:18:39 CET 2017
BOOT_CLOJURE_NAME=org.clojure/clojure
BOOT_CLOJURE_VERSION=1.8.0
BOOT_VERSION=2.7.1
```
If you now launch the following `boot` command you'll receive a
warning and an error:

```bash
boot serve -d target watch reload cljs-repl cljs target
Retrieving
...
Starting reload server on ws://localhost:53582
You are missing necessary dependencies for boot-cljs-repl.
Please add the following dependencies to your project:
[com.cemerick/piggieback "0.2.1" :scope "test"]
[weasel "0.7.0" :scope "test"]
...
java.io.FileNotFoundException: Could not locate cemerick/piggieback__init.class or cemerick/piggieback.clj on classpath.
Elapsed time: 0.734 sec
```

This is because `boot-cljs-repl` does not transitively include its
dependencies and you have to explicitly add them in the
`:dependencies` section of the `build.boot` file:

```clj

(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[org.clojure/clojure "1.8.0"]         ;; add CLJ
                 [org.clojure/clojurescript "1.9.473"] ;; add CLJS
                 [adzerk/boot-cljs "1.7.228-2"]
                 [pandeiro/boot-http "0.7.6"]
                 [org.clojure/tools.nrepl "0.2.12"]
                 [adzerk/boot-reload "0.5.1"]
                 [adzerk/boot-cljs-repl "0.3.3"]
                 [com.cemerick/piggieback "0.2.1"]     ;; needed by bREPL
                 [weasel "0.7.0"]])                    ;; needed by bREPL

(require '[adzerk.boot-cljs :refer [cljs]]
         '[pandeiro.boot-http :refer [serve]]
         '[adzerk.boot-reload :refer [reload]]
         '[adzerk.boot-cljs-repl :refer [cljs-repl start-repl]])

```

> NOTE 3: At the moment we don't take care of the `:scope` of the
> dependencies. We'll come back to this directive in a later tutorial.

After having quit the previous process, you can safely run the
`boot` command in the terminal as follows:

```bash
boot serve -d target watch reload cljs-repl cljs target
Starting reload server on ws://localhost:58051
Writing boot_reload.cljs...
Writing boot_cljs_repl.cljs...
2016-01-03 11:32:46.553:INFO::clojure-agent-send-off-pool-0: Logging initialized @9256ms
2016-01-03 11:32:46.613:INFO:oejs.Server:clojure-agent-send-off-pool-0: jetty-9.2.10.v20150310
2016-01-03 11:32:46.636:INFO:oejs.ServerConnector:clojure-agent-send-off-pool-0: Started ServerConnector@4a94a06{HTTP/1.1}{0.0.0.0:3000}
2016-01-03 11:32:46.637:INFO:oejs.Server:clojure-agent-send-off-pool-0: Started @9340ms
Started Jetty on http://localhost:3000

Starting file watcher (CTRL-C to quit)...

nREPL server started on port 58053 on host 127.0.0.1 - nrepl://127.0.0.1:58053
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
Elapsed time: 18.949 sec
```

The command output informs you that [`nrepl server`][8] has been
started on the local host at a port number (your port number will be
different). If your editor supports `nrepl` you are going to use that
information to connect to the now running `nrepl server` with an
`nrepl client`.

> NOTE: Emacs and CIDER support this. You can learn more about them with these
[resources](https://github.com/magomimmo/modern-cljs/blob/master/doc/supplemental-material/emacs-cider-references.md).

At the moment we're happy enough to be able to run `cljs-repl` from a
second terminal by first launching the predefined `repl` task included
with `boot` and passing it the `-c` (i.e. client) option:

```bash
# in a new terminal
cd /path/to/modern-cljs
boot repl -c
REPL-y 0.3.7, nREPL 0.2.12
Clojure 1.8.0
Java HotSpot(TM) 64-Bit Server VM 1.8.0_112-b16
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

This is a standard CLJ REPL defaulted to the `boot.user`
namespace. From here we can launch a browser based CLJS REPL (bREPL) as
follows:

```cljs
boot.user=> (start-repl)
<< started Weasel server on ws://127.0.0.1:49358 >>
<< waiting for client to connect ... Connection is ws://localhost:49358
Writing boot_cljs_repl.cljs...
```

The terminal is now waiting for a client connection from the
browser. Visit the usual http://localhost:3000 URL to activate the
bREPL connection.

```cljs
 connected! >>
To quit, type: :cljs/quit
nil
cljs.user=>
```

To confirm that you can evaluate CLJS forms from the bREPL,
submit the alert function to the browser:

```cljs
(js/alert "Hello, ClojureScript")
nil
```

To stop the bREPL, submit the `:cljs/quit` expression. Then stop the
CLJ REPL (CTRL-D or `(exit)` or `(quit)`). Finally stop `boot`
(CTRL-C).

Before stepping to the next tutorial reset your git repository:

```bash
git reset --hard
```

## Next step - [Tutorial 3: House Keeping][9]

In the next [tutorial][9] we're going to automate the launching of the
`boot` command to approach the Immediate Feedback Development
Environment (IFDE).

# License

Copyright © Mimmo Cosenza, 2012-2015. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://vimeo.com/36579366
[2]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-01.md
[3]: https://git-scm.com/
[4]: https://github.com/boot-clj/boot/wiki/Community-Tasks
[5]: https://github.com/pandeiro/boot-http
[6]: https://github.com/adzerk-oss/boot-reload
[7]: https://github.com/adzerk-oss/boot-cljs-repl
[8]: https://github.com/clojure/tools.nrepl
[9]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-03.md
