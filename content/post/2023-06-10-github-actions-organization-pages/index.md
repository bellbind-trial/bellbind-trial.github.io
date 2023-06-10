---
title: "HugoサイトをGithub ActionsでOrganization用Github Pagesで公開する"
date: 2023-06-10T00:39:49+09:00
---

ローカルで編集したHugoサイトを、Github Actionsを使用してgithub上でビルドし、Organization用Github Pagesで公開する手順。
<!--more-->

## 用語

- Hugoサイト: Hugoコマンドで扱うソース(mdファイル、theme等)一式が入っているディレクトリ
  - コマンド`hugo new site SITE-NAME`で初期内容が作られる
- Organization: Github上での、複数のユーザーで共同管理できるグループのこと
- Repository: ファイル群を含むgitリポジトリ。Githubでは、ユーザーやOrganizationは、複数のRepositoryが所属する。
  - Githubでは、User/RepositoryやOrganization/Repositoryでユニークに扱うので、ユーザーやOrganizationが違えば、Repository名が同じでも別物となる。
- Github Pages: Repositoryの内容をstaticなWebサイトとして公開できるサービス
- Github Actions: git pushしたときにgithub上で記述した自動処理(workflow)をgithubサーバ上のコンテナで起動させられるシステム
  - ソースコードの入っているRepositoryをpushしたら、自動でビルドして結果のファイルを公開する、などが可能。
- `emacs`: Repository内の各種テキストファイルを編集するためのテキストエディタのコマンド(ここでは、引数のファイルの **テキスト編集をするタイミング** を示すために使う)
- `git push origin HEAD`: ローカルのgitリポジトリの現ブランチのcommit内容を、github上のRepositoryの同名ブランチにコピーするコマンド(ブランチ名を意識しなくてよくなる)

## 1. Github上でOrganizationを作り、Github Pages用のRepositoryを作る

以降、[`bellbind-trial`](https://github.com/bellbind-trial/)という名前のGithub Organizationを作るものとする。

- https://github.com/organizations/plan (Freeでよい)

Organization `bellbind-trial`のためのGithub Pages用のRepository名は、[`bellbind-trial.github.io`](https://github.com/bellbind-trial/bellbind-trial.github.io)という名前のRepositoryにする必要がある(つまり、ユニーク名としては、`bellbind-trial/bellbind-trial.github.io`)。

- https://github.com/new (注: `Owner`は対象のOrganization切り替えること)

一般的に、ユーザー名やOrganization名が **`XXXX`の場合のGithub PagesのRepository名は`XXXX.github.io`** にしなくてはいけない。

- Hugoサイトのファイルを入れるため、README、.gitignore、LICENSEはNoneのまま、空っぽのRepository作る必要がある。

## 2. ローカルのHugoサイトにgitリポジトリを加え、github上のRepositoryへpushする

ローカルのHugoサイトは`bellbind-trial`として作ってあるものとする
(`hugo server`で http://localhost:1313/ で見れるようになっている状態)。

{{<highlight bash>}}
$ cd bellbind-trial/
$ git init
$ git add archetypes content static assets data layouts themes hugo.toml
$ git remote add origin git@github.com:bellbind-trial/bellbind-trial.github.io.git
$ git commit
$ git push origin HEAD
{{</highlight>}}

以下の内容のHugoサイト用の`.gitignore`を追加しても良い
(この`.gitignore`ファイルが有れば、オプションなしの`hugo`コマンドを実行したとき生成されるファイル群をRepositoryから除外することができる)

```
/public/
/resources/_gen/
/assets/jsconfig.json
/.hugo_build.lock
hugo_stats.json
```

{{<highlight bash>}}
$ cd bellbind-trial/
$ emacs .gitignore
$ git add .gitignore
$ git commit
$ git push origin HEAD
{{</highlight>}}

## 3. RepositoryにGithub Actionsを加える

Github Actions設定は、Repository内の`.github/workflows/`以下に、処理内容を書いたymlファイルを置きpushするだけで、自動で実行されるようになる。

Hugoのためのユーザー/Organization用Github Pages設定は、 https://github.com/peaceiris/actions-hugo の内容そのまま改変なしで使える。

{{<highlight yml>}}
# .github/workflows/gh-pages.yml
name: GitHub Pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-22.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.113.0'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/main' }}
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
{{</highlight>}}

{{<highlight bash>}}
$ mkdir -p .github/workflows/
$ emacs .github/workflows/gh-pages.yml
$ git add .github/workflows/gh-pages.yml
$ git commit
$ git push origin HEAD
{{</highlight>}}

pushしたあとは、Repositoryページの[Actionsタブ](https://github.com/bellbind-trial/bellbind-trial.github.io/actions)でその結果が確認できる。

このymlによって生成されたファイルは、[`gh-pages`ブランチ](https://github.com/bellbind-trial/bellbind-trial.github.io/tree/gh-pages)に加えられる。


## 4. Github Pagesの`Branch`を`gh-pages`に変更する

デフォルトのGithub Pages用のブランチは`main`なので、この状態では、404 Not Foundのままである。
そこで、`Settings`タブの`Pages`にある`Branch`を`gh-pages`に切り替える(`Save`)。

![Setting - Pages - Branchをgh-pagesへ変更しSave](./pages-branch-change.png)

しばらく待てば、Github Pagesにアクセス可能になる。

- https://bellbind-trial.github.io/

## 6. 記事を追加してgit pushで更新する

ここでは、`post/2023-06-10-github-actions-organization-pages/index.md`を追加する場合を例にする。

この`index.md`ファイルには、埋め込む画像`~/Desktop/pages-branch-change.png`も加える。

(注: `git push`する前に、mdファイルのヘッダに最初から含まれる **`draft: true`部分を消す**のを忘れないこと)

{{<highlight bash>}}
$ hugo new post/2023-06-10-github-actions-organization-pages/index.md
$ cp ~/Desktop/pages-branch-change.png content/post/2023-06-10-github-actions-organization-pages/
$ emacs content/post/2023-06-10-github-actions-organization-pages.md
$ git add content static
$ git commit
$ git push origin HEAD
{{</highlight>}}

この構造の場合、`index.md`内の画像リンクは以下のようになる。

{{<highlight bash>}}
![Setting - Pages - Branchをgh-pagesへ変更しSave](./pages-branch-change.png)
{{</highlight>}}

- 参考: [page bundles](https://gohugo.io/content-management/page-bundles/)

記事を追加編集して`git add` - `git commit` - `git push`するたびに、githubの[Actionsタブ](https://github.com/bellbind-trial/bellbind-trial.github.io/actions)に、
「pages build and deployment」という項目が追加され、hugoのページ生成とgh-pagesブランチへの反映が行われる。
ページ生成処理が終了成功すると、その左に「緑✓」がつくので、その後すこしたてばGithub Pagesサイトに反映される。
