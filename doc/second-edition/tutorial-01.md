# Tutorial 1 - The Basics

この最初のチュートリアルでは、[boot][2]ビルドツールを使用して最小限のClojureScript（[CLJS][1]）プロジェクトを作成および設定します。

## Requirements

このチュートリアルでは、コンピュータに`Java`と`boot`をインストールする必要があります。

`java`をインストールするには、お使いのオペレーティングシステムに沿った[手順][3]に従ってください。 `boot`をインストールするには、[README][4]の対応するセクションの簡単な手順に従ってください。

ターミナルで`boot -h`コマンドを実行して、インストールをテストします。 次に、`boot -u`コマンドを発行して、最新バージョンの`boot`を取得します。

> NOTE 1: 私は強くJava 8を使用することをお勧めします。あなたがJava 7を使用している場合、それは言及する価値があるかもしれません
 https://github.com/boot-clj/boot/wiki/JVM-Options#permgen-errors

## Create the project structure

CLJSの最低限のWebプロジェクトは、3つのファイルで構成されています。

* htmlページ x 1;
* CLJSソースコード x 1;
* CLJSソースコードをコンパイルするための`boot`ビルドファイル x 1

CLJSが特定のディレクトリ構造を指示していない場合でも、プロジェクト構成要素とその構造を理解できるように、数か月だけでも簡単にプロジェクトを構成することをお勧めします。 さらに、各構築ツールには固有の特質があり、デフォルトと呼ばれます。 手元にあるツールのデフォルトを守るほど、プロジェクトを管理する際に苦労することは少なくなります。

これらの前提を考慮して、できるだけ`boot`デフォルトに固執することによって、`modern-cljs`という新しいプロジェクトのディレクトリ構造を作成しましょう。

プロジェクトにおける推奨されるファイルレイアウトは次のとおりです。

```bash
modern-cljs/
├── build.boot
├── html
│   └── index.html
└── src
    └── cljs
        └── modern_cljs
            └── core.cljs
```

* `modern-cljs` はこのプロジェクトのホームディレクトリ。
* `src/cljs/` はCLJSソース・ファイルをホストします。
* `html` はhtmlリソースをホストします。

> NOTE 2: CLJ/CLJSでは、単一セグメントの名前空間は[推奨されません][5]。 ですから、`modern_cljs`ディレクトリを作成しました。 パッケージ名にハイフン " - "（または他の特殊文字）を管理することの[Javaの困難さ][6]のために、対応するディレクトリ名のハイフン（`-`）をアンダースコア（`-`）に置き換えました。

> NOTE 3: ClojureScriptソースのファイル拡張子は**clj**ではなく**cljs**であることに注意してください。

ターミナルで次のコマンドを発行します。

```bash
mkdir -p modern-cljs/{src/cljs/modern_cljs,html}
```

次のコマンドを発行して、必要な3つのファイルを作成しましょう。

```bash
cd modern-cljs
touch html/index.html src/cljs/modern_cljs/core.cljs build.boot
```

## Hello World in CLJS

ここで、最初のCLJSコードを記述します。`src/cljs/modern_cljs/core.cljs`ファイルを希望のエディタで開き、次のCLJSコードを入力します。

```clj
;; create the main project namespace
(ns modern-cljs.core)

;; enable cljs to print to the JS console of the browser
(enable-console-print!)

;; print to the console
(println "Hello, World!")
```

すべてのCLJ/CLJSファイルは、ディスク上のパスに一致する名前空間宣言で始まる必要があります。

`modern-cljs.core` <--> `modern_cljs/core.cljs`

`(enable-console-print!)`の式は、`(println "Hello, world!")`がHello、World！をコンソールに表示するような方法で、ブラウザのコンソールに表示出力をリダイレクトします。

### Minimal build.boot

`core.cljs`をJSにコンパイルし、`index.html`ページにリンクする方法が必要です。

まず、ファイル`html/index.html`を開き、次のように編集します。

```html
<!doctype html>
<html>
  <head>
    <title>Hello, World!</title>
  </head>
  <body>
    <script src="main.js"></script>
  </body>
</html>
```

CLJSファイルへの参照はないことに注意してください。 `<script>`タグを`index.html`ページに追加して、`main.js`ファイルにリンクしました。 このJSファイルは、上記のcore.cljsソースコードをコンパイルする際に`boot`ビルドツールによって生成されます。

`core.cljs`ファイルをコンパイルするには、別の拡張子を持つ通常のCLJファイルである`build.boot`ファイルを編集して、`boot`コマンドを設定する必要があります。

```clj
(set-env!
 :source-paths #{"src/cljs"}
 :resource-paths #{"html"}

 :dependencies '[[adzerk/boot-cljs "1.7.228-2"]])

(require '[adzerk.boot-cljs :refer [cljs]])
```

かなりミニマル！ `set-env!`関数は`:source-paths`と`:resource-paths`オプションを、上で作成したプロジェクト構造の対応する値に設定します。 次に、`:dependencies`キーワードを追加することによって、プロジェクトの唯一の明示的依存関係として`boot-cljs`コンパイルタスクを挿入します。

`clojure`と`clojurescript`の依存関係が含まれていなくても、`boot`は自動的にそれがうまく動作することがわかっている対応するリリースを自動的にダウンロードすることができます。

最後に、`require`フォームは、`cljs`タスクを`boot`コマンドで参照できるようにします。

ターミナルから`boot -h`コマンドを実行すると、`cljs`タスクが起動できるようになります。

```bash
boot -h

...

Tasks:   ...
         target                      Writes output files to the given dir...
         ...
         zip                         Build a zip file for the project.

         cljs                        Compile ClojureScript applications.

...

Do `boot <task> -h` to see usage info and TASK_OPTS for <task>.
```

次のコマンドを実行することによって、`cljs`タスクに関する詳細情報を取得できます。

```bash
boot cljs -h
Compile ClojureScript applications.
...
Available --optimization levels (default 'none'):
...
```

ご覧のとおり、CLJSコンパイラのデフォルトの最適化ディレクティブは`none`です。 別のチュートリアルでは、さまざまなCLJSのコンパイルの最適化について説明します（`none`、`whitespace`、`simple`、`advanced`）。 現時点では、開発サイクル中に一般的に使用されている`none`にしましょう。 作業中の`boot cljs`を見てみましょう：

```bash
boot cljs
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
```

`cljs`タスクは、`main.js` JSファイルを生成することによってCLJSコードをコンパイルしました。そして今、簡単なデフォルトに準拠するためだけに、`index.html`ページの`<script>`タグで`main.js`を含むJSファイルを呼び出すのかわかるでしょう。

さて、あなたがプロジェクトのディレクトリ構造を探すと、それが見つけられないと驚くでしょう：

```bash
tree
.
├── build.boot
├── html
│   └── index.html
└── src
    └── cljs
        └── modern_cljs
            └── core.cljs

4 directories, 3 files
```

実際には、明示的に指示しなければ、デフォルトでは`boot`は出力ファイルを作成しません。 `target`の定義済みタスクのコマンドラインヘルプを見てみましょう：

```bash
boot target -h
Writes output files to the given directory on the filesystem.

Options:
  -h, --help      Print this help info.
  -d, --dir PATH  Conj PATH onto the set of directories to write to (target).
  -m, --mode VAL  VAL sets the mode of written files in 'rwxrwxrwx' format.
  -L, --no-link   Don't create hard links.
  -C, --no-clean  Don't clean target before writing project files.
```

面白い。 `target`の事前定義された`boot`のタスクは、タスク（例えば、`cljs`）の出力を所与のディレクトリに書き込むことができる。 上記のヘルプはあなたには言いませんが、ディレクトリを指定しなければ、出力ファイルをプロジェクトのホームディレクトリのデフォルトの`target`サブディレクトリに書き込みます。これは、次のように確認できます。

```bash
boot cljs target
Writing main.cljs.edn...
Compiling ClojureScript...
• main.js
Writing target dir(s)...
```

```bash
tree
.
├── boot.properties
├── build.boot
├── html
│   └── index.html
├── src
│   └── cljs
│       └── modern_cljs
│           └── core.cljs
└── target
    ├── index.html
    ├── main.js
    └── main.out
        ├── boot
        │   └── cljs
        │       ├── main312.cljs
        │       ├── main312.cljs.cache.edn
        │       ├── main312.js
        │       └── main312.js.map
        ├── cljs
        │   ├── core.cljs
        │   ├── core.js
        │   └── core.js.map
        ├── cljs_deps.js
        ├── goog
        │   ├── array
        │   │   └── array.js
        │   ├── asserts
        │   │   └── asserts.js
        │   ├── base.js
        │   ├── debug
        │   │   └── error.js
        │   ├── deps.js
        │   ├── dom
        │   │   └── nodetype.js
        │   ├── object
        │   │   └── object.js
        │   └── string
        │       ├── string.js
        │       └── stringbuffer.js
        └── modern_cljs
            ├── core.cljs
            ├── core.cljs.cache.edn
            ├── core.js
            └── core.js.map

17 directories, 27 files
```

たくさんあります。 今私たちはそれを掘り下げません。 現時点では、私たちはいくつかのことに気づくだけです。

* `html`と`src`にあるオリジナルのディレクトリにはノータッチ.
* index.html`リソースまですべてが新しい`target`ディレクトリに生成されています。

`boot`が継続的に進んでいることを考慮して、次のように新しい`boot.properties`ファイルを作成して、プロジェクト内の現在の安定版を固定することを強くお勧めします。

```bash
boot -V > boot.properties
```

これには次の内容が含まれるはずです：

```bash
cat boot.properties
#http://boot-clj.com
#Thu Mar 09 14:56:52 CET 2017
BOOT_CLOJURE_NAME=org.clojure/clojure
BOOT_CLOJURE_VERSION=1.7.0
BOOT_VERSION=2.7.1
```

## Visit index.html

`boot`は、タスクによって生成された対応する出力からプロジェクトの入力を分離する際に、きちんとしたアプローチを採用しています。 `:source-paths`および`:resource-path`元のディレクトリからの入力ファイルは、どのブート・タスクによっても変更されることはありません。 すべては、内部的に生成された一時ディレクトリとは別に、明示的に`target`ディレクトリ内で発生します。


ブラウザを開き、ローカルの`target/index.html`ファイルにアクセスしてください。 開発ツール（例：Chrome開発ツール）でコンソールを開きます。 すべてがうまくいけば、 "Hello, World!"がコンソールに表示されます。

## Next Step - [Tutorial 2: Immediate Feedback Principle][7]

次の[チュートリアル][7]では、非常に対話型の開発環境を構築するために、[Bret Victorの即時フィードバックの原則][8]に可能な限り厳密に従います。

# License

Copyright © Mimmo Cosenza, 2012-2015. Released under the Eclipse Public
License, the same as Clojure.

[1]: https://github.com/clojure/clojurescript.git
[2]: http://boot-clj.com/
[3]: https://java.com/en/download/help/index_installing.xml?os=All+Platforms&j=8&n=20
[4]: https://github.com/boot-clj/boot#install
[5]: http://stackoverflow.com/questions/13567078/whats-wrong-with-single-segment-namespaces
[6]: http://docs.oracle.com/javase/specs/jls/se8/html/jls-6.html
[7]: https://github.com/magomimmo/modern-cljs/blob/master/doc/second-edition/tutorial-02.md
[8]: https://vimeo.com/36579366
