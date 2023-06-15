---
title: "テーマをHugo Modulesベースで扱うHugoプロジェクトの作り方"
date: 2023-06-12T10:46:53+09:00
tags:
  - "hugo"
  - "hugo mod"
---

Hugo テーマを取り飲むのに以前からの`themes/`に配置する方法ではなく、Hugo Modulesを用いて外部参照して使用する方法。
<!--more-->

## 0. 事前準備

Hugo Modulesを使うには、`hugo`コマンドの他に、

- `git`コマンド
- `go`コマンド(`golang`パッケージ内)

が必須になるので、プラットフォームごとの手段でインストールしておくこと。

## 1. 空プロジェクトから作る方法

作る前に、**Hugoプロジェクト自体のMod ID**を決めておく必要がある。

ふつうは、(実際にpushする必要はなくても、)ソースであるHugoプロジェクトをgithubに置く前提で命名する。

- organization/repositoryが、それぞれexample-org/example-org.github.ioという名前である場合、Mod IDは`github.com/example-org/example-org.github.io`となる。

Hugoプロジェクト自体のディレクトリ名は適当な名前もよい。

```bash
$ mkdir example-org-site/
$ cd example-org-site/
$ hugo mod init github.com/example-org/example-org.github.io
$ mkdir content/
$ emacs hugo.toml
```


たとえば、Hugo Bear Blogテーマを使用するものとする場合を扱う。

- https://themes.gohugo.io/themes/hugo-bearblog/

テーマの説明の中に、`git submodule add https://github.com/janraasch/hugo-bearblog.git themes/hugo-bearblog`という記述がある。

この中のgitリポジトリのURL `https://github.com/janraasch/hugo-bearblog.git`から、
テーマのMod IDは`github.com/janraasch/hugo-bearblog`と決まる(必ず`https://`と`.git`は外す)。

最初の`hugo.toml`の内容は、テーマをインポートする記述だけをする:

{{<highlight toml>}}
[module]
  [[module.imports]]
    path = "github.com/janraasch/hugo-bearblog"
{{</highlight>}}

(注: hugo-bearblogは違うが、テーマによってはSCSSなどのビルドのための環境の用意(sassc等)が必要になる場合がある)

この時点で、ディレクトリの下には、以下のファイルができている:

- `hugo.toml`
- `go.mod`
- `content/`
- `resources/_gen/`

(`go.mod`と`resources/_gen/`以下は、`hugo mod init`で作られる)

ここで、サーバを立ち上げれば、テーマがダウンロードされ、`http://localhost:1313/`のWebページに適用されることをブラウザで確認する。

```bash
$ hugo server
```

つぎに記事を追加できることを確認する。

```bash
$ hugo new blog/first-post.md
$ emacs content/blog/first-post.md
```

その後は、これまでと同様に、`hugo.toml`でテーマの設定を変更などを行っていくことになるだろう。

## 2. Github ActionsでビルドしてGithub Pagesで公開する方法

続いて、hugo modベースのHugoプロジェクトを、gitリポジトリ化し、github上にpushし、Github ActionsでHTMLサイトをビルドさせてGithub Pagesで公開するまで行う。

### gitリポジトリ化

```bash
$ cd example-org-site/
$ git init
$ emacs .gitignore
$ git add .gitignore hugo.toml go.mod go.sum content/
$ git commit -m "[init]"
```

Hugoプロジェクトのための`.gitignore`ファイルのの内容は、以下のとおり:

```
/public/
/resources/_gen/
/assets/jsconfig.json
/.hugo_build.lock
hugo_stats.json
```

### Github Actionsの設定

以下の内容の`.github/workflows/gh-pages.yml`を追加する:

{{<highlight yml>}}
name: GitHub Pages

on:
  push:
    branches:
      - main  # Set a branch name to trigger deployment
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-22.04
    permissions:
      contents: write
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Go
        uses: actions/setup-go@v4.0.0
        with:
          go-version: '^1.20'

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.113.0'

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        # If you're changing the branch from main,
        # also change the `main` in `refs/heads/main`
        # below accordingly.
        if: ${{ github.ref == 'refs/heads/main' }}
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
{{</highlight>}}

Hugo Modulesを使う場合は`go`コマンドも必要なため、`actions-hugo`のREADMEにあるymlの内容の`Build`ステップの前に、`setup-go`を加えたものになる。

- https://github.com/peaceiris/actions-hugo
- https://github.com/actions/setup-go

```bash
$ mkdir .github/workflows/
$ emacs .github/workflows/gh-pages.yml
$ git add .github/
$ git commit -m "[add] .github/workflows/gh-pages.yml"
$ git push origin HEAD
```

`Actions`で、`pages build and deployment`が終了したあと、
Github Pagesを有効化するために、github.com上のリポジトリページで、`Settings` - `Pages`の`Branch`を`gh-pages`に変えれば、Webページが表示できるようになる。

----

## A. 既存のHugoサイトをHugo ModulesベースのHugoプロジェクトに変更する方法

失敗しやすいので、作業をする前に **Hugoサイトをまずバックアップしておく** べきである。
Githubにおいてあるなら、別途`git clone`したもの(`--recursive`なし)で作業すべきである。

### A.1. テーマを消す

`hugo mod init`でも`hugo.toml`を解釈するので、後に不要になるものを先に外しておく。

テーマはbinarioを使っているとする。

- https://themes.gohugo.io/themes/binario/


#### A.1,1. `hugo.toml`から`theme`を消す

`hugo.toml`内から、

```toml
theme = "binario"
```

を削除する。

#### A.1.2. `themes/`ディレクトリのクリーンアップ

`themes/binario`を消す。

- zip展開やgit cloneでの展開であれば、`themes/`ディレクトリごと削除するだけ

ただし、`git submodule`でテーマを導入している場合は、以下の手順で行う:

```bash
$ git submodule deinit -f themes/binario
$ git rm -rf themes/
$ git rm .gitmodules
$ rm -rf .git/modules/
```

### A.2. `hugo mod init`

Github上のorganization/repositoryが`bellbind-trial/bellbind-trial.github.io`なので、

```bash
$ hugo mod init github.com/bellbind-trial/bellbind-trial.github.io
```

を実行する。

### A.3. テーマの追加

`hugo.toml`に以下のエントリーを加える(一番下でよい):

テーマのgitリポジトリのURLが`https://github.com/vimux/binario`であるため、Mod IDは`github.com/vimux/binario`になる。

{{<highlight toml>}}
[module]
  [[module.imports]]
    path = "github.com/vimux/binario"
{{</highlight>}}

### A.4. `.github/workflows/gh-pages.yml`の編集

Buildステップの前に、setup-goのエントリーを加える:

{{<highlight yml>}}
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod
# ここから
      - name: Setup Go
        uses: actions/setup-go@v4.0.0
        with:
          go-version: '^1.20'
# ここまで
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.113.0'

      - name: Build
        run: hugo --minify
{{</highlight>}}


### A.5. `hugo server`で確認

gitリポジトリへ反映する前に、`hugo server`で以前と同じように表示されることを確認する。

うまく表示されていない場合、git cloneから、途中で失敗しないようにしてやり直したらうまくいくかもしれない。

### A.6. git commitし、githubへpush

```bash
$ git add hugo.toml .github/
$ git commit -m "[switch] to hugo mod"
$ git push origin HEAD
```

Github Actionsが成功することを確認し、Webページが表示されることを確認する。

----

## B. 共有キャッシュを消すコマンド: `hugo mod clean`

現状、テーマのような外部のHugo Modulesは、Hugoプロジェクトごとに保持するのではなく、ローカルマシン上で別途、共有してキャッシュされている。

全キャッシュを消すコマンド:

```
$ hugo mod clean --all
```

たとえば`hugo server`を立ち上げて、テーマモジュールのセットアップ中などに強制終了させた場合、その後の`hugo server`のHTML生成がおかしくなる場合がある。
そういう場合には、一度キャッシュを全部消して、やり直すとなおるかもしれない。

