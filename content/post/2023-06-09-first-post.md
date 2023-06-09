---
title: "最初のポスト"
date: 2023-06-09T22:40:33+09:00
description: "Example article description"
---
Hugoでの最初のポスト
<!--more-->

## 投稿のコマンド

{{<highlight bash>}}
$ hugo new post/2023-06-09-first-post.md
$ emacs content/post/2023-06-09-first-post.md
$ hugo server -D 
{{</highlight>}}

(`hugo server`の`-D`オプションは、`--buildDrafts`のショートハンドで、ヘッダが`draft: true`なmdファイルもレンダリング対象にするもの)

## 一覧表示用部分の切り分け

mdファイル内に`<!--more-->` を入れると、それ以降の内容は、一覧ページからカットされる。

(入れなければ、テキストが改行なしで全部入ってしまう)

## コードブロック構文

コードブロックのシンタックスハイライトでは、以下のような拡張構文を用いる(ただし、`{`と`{`の間の空白入れない)。

{{<highlight markdown>}}
{ {<highlight js>}}
if (true) console.log(`Hello World!`);
{ {</highlight>}}
{{</highlight>}}

は、以下のように出力される。

{{<highlight js>}}
if (true) console.log(`Hello World!`);
{{</highlight>}}

対応言語は、hugoが使用しているchromaのものから選べる(JavaScriptは、`js`でも`JavaScript`でもよい)。

- https://github.com/alecthomas/chroma#supported-languages

## MathJax構文

数式コードブロックのmarkdownソース(要空行):

{{<highlight markdown>}}

$$ F(k) = \sum^{N-1}_{n=0} f(n) \exp({-2\pi i \over N} n k) $$

\\[ f(t) = {1 \over N} \sum^{N-1}_{n=0} F(n) \exp({2 \pi i \over N} n t) \\]

{{</highlight>}}

結果:

$$ F(k) = \sum^{N-1}_{n=0} f(n) \exp({-2\pi i \over N} n k) $$

\\[ f(t) = {1 \over N} \sum^{N-1}_{n=0} F(n) \exp({2 \pi i \over N} n t) \\]


文中の数式:

{{<highlight markdown>}}
「式中の \\( \exp(\theta i) \\) は、\\(\cos(\theta) + i \sin(\theta)\\)を意味する。」
{{</highlight>}}

結果:

「式中の \\( \exp(\theta i) \\) は、\\(\cos(\theta) + i \sin(\theta)\\)を意味する。」
