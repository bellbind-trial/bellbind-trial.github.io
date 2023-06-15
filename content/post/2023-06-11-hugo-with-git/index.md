---
title: "git管理前提のhugoプロジェクトの作り方"
date: 2023-06-11T09:06:19+09:00
tags:
  - "hugo"
  - "git"
---
Github ActionsでGithub Pagesを更新させるために、そのWebページのソースとなるHugoプロジェクトを作ったあと、gitリポジトリ化する手順。
<!--more-->

## 1. gitリポジトリを前提としたHugoプロジェクトの作成

雛形の時点でgitリポジトリ化する。

コマンド列:

{{<highlight bash>}}
$ hugo new site MY-SITE
$ cd MY-SITE/
$ git init
$ git add .
$ git commit -m "[init]"
{{</highlight>}}

## 2. `.gitignore`を加える

Hugo用`.gitignore`ファイル内容:

{{<highlight config>}}
/public/
/resources/_gen/
/assets/jsconfig.json
/.hugo_build.lock
hugo_stats.json
{{</highlight>}}

コマンド列(emacsはテキストファイルの編集):

{{<highlight bash>}}
$ emacs .gitignore
$ git add .gitignore
$ git commit -m "[add] .gitignore"
{{</highlight>}}

## 3. テーマインストール

テーマは以下から選ぶ:

- https://themes.gohugo.io/

このにあるテーマはgitリポジトリで公開されているので、git submoduleでインストールする:

- 使用するテーマ: https://themes.gohugo.io/themes/risotto/

{{<highlight bash>}}
$ git submodule add https://github.com/joeroe/risotto.git themes/risotto
$ git commit -m "[add] theme"
{{</highlight>}}

## A. hugoについて

hugoは、Markdown形式で書いた文書ファイル(拡張子: .md)群を、HTMLファイル化して、一覧ページももつWebサイトを構成するファイル群に変換するシステム。

- https://gohugo.io/

生成するWebサイトのHTMLのソースとなるディレクトリは、「Hugoプロジェクト」と呼ぶ。

### hugoコマンドのインストール

Hugoのコマンドは、各プラットフォームのパッケージマネージャー経由で導入する。

- https://gohugo.io/installation/

### よく使う`hugo`コマンド

hugoでは、すべてコマンドライン上で`hugo`コマンドを使用して、Webサイトを構築する処理を行う。

- `hugo new site SITE-NAME`: 空っぽのHugoプロジェクトのファイル群を、`SITE-NAME/`ディレクトリ以下に生成するコマンド
- `hugo new ARTICLE-FILE.md`: 記事の雛形となる`content/ARTICLE-FILE.md`ファイルを生成するコマンド
- `hugo server`: プレビューとなるWebサーバを、`http://localhost:1313/`に立ち上げるコマンド
- `hugo`: `public/`ディレクトリ以下に、HTMLファイルのWebサイト一式を生成するコマンド

### Hugoプロジェクトのファイル構成

- サイト設定用ファイル:
  - `hugo.toml`: hugoコマンドが解釈する設定ファイル
  - `themes/`: Webサイトのテーマ(となるHugoプロジェクト)を置く場所
- 文書用ファイル:
  - `content/`: 個別のWebページのソースとなる文書データのmdファイルを置く場所
  - `static/`: 全記事から参照できるファイル(画像など)を置く場所
- サイトデザイン用ファイル:
  - `layouts/`: 各Webページの枠部分や、トップページなどの一覧ページを生成するためのテンプレートファイル群
  - `archetypes/`: `hugo new`で作るファイルの雛形となるmdファイルがある場所
  - `data/`: layoutのテンプレートが参照する設定ファイルを置く場所
  - `assets/`: layoutのテンプレートで使用するファイル(スクリプトやスタイルシートなど)のソースを置く場所

### hugoのテーマシステム

hugoプロジェクトのテーマというのは、既存のHugoサイトの`layouts/`等のサイトデザインファイル群を継承する仕組み。

hugoは、Webページを生成する処理で必要なファイルを参照するとき、Hugoサイト自身に存在しなければ、テーマファイルの中の同名ファイルを使用するようになっている。
(テーマの一部を上書きするときは、該当ファイルをHugoサイト直下の同名の位置へコピーして編集する)

このため、Hugoプロジェクトの構成はは、継承元となるテーマのテンプレートコードに依存したものとなり、テーマによって大きく違うものになる
(扱う`content/`下のファイル名を特定したテーマもある)。

- 現在は、テーマに限らず任意のファイルで、他プロジェクトのファイルを参照可能になるHugo Moduleも備わっている



## B. Hugo Modules

テーマに限らず、`content/`や`static/`なども他のHugoプロジェクトから継承可能にするのがHugo Modules。
`hugo mod`コマンドで操作する。

- `hugo mod`を使う場合は、テーマでも`git submodule`は使わなくなる

共通の記事や画像のデータがあるサイトを、ファイルコピーせずに管理できるようになる。
ただし、`hugo mod`を使うには、`hugo`単体ではなく、`git`と`golang`のランタイムもインストールされている必要がある。

- https://gohugo.io/commands/hugo_mod/

`hugo mod init`で有効化したあとは、`hugo.toml`に以下のように書くだけで、テーマが適用されるようになる。

(`path`の値はgitリポジトリのURLだが、`https://`は外す)

{{<highlight toml>}}
[module]
  [[module.imports]]
    path = 'github.com/joeroe/risotto'
{{</highlight>}}


注: Hugo Moduleを使う場合、`layouts/`等が直接継承されるので、`hugo.toml`中の`theme = "risotto"`の行は消さなくてはいけない。