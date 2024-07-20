---
title: "URL の結合を完全に理解する" # 記事のタイトル
emoji: "🔗" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["html", "javascript", "url"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
URL の結合をよく理解していなかったので、それに対する備忘録です。

例えば
```html
<a href="/top">Top</a>
```
みたいな HTML の anchor タグを使う時、現在の URL と`/top` を結合した URL に遷移するわけです。

これは、JavaScript の API にもあり、
```js
const merged = new URL('/top', 'https://example.com/profile') // 'https://example.com/top'
```
のようにして結合することもできます。

この結合は Deno などの JavaScript ランタイムでファイルを参照する時にも使うのですが、 `path.join` などとも挙動が違うのでまとめたいと思います。

## URL の構造
JavaScript の `URL` API によって、URL の各部分は以下のように分割できます。
<img width="634" alt="Untitled 16" src="https://github.com/user-attachments/assets/c01c8772-73c0-4860-aca1-36eef60a6ece">
これが URL の構造です。
この前提条件を踏まえて説明していきます。

また、`new URL(next, base)` のようにして、`next` を**結合先**、`base` を**結合元** と定義していきます。

## 結合先の最初が `/`
* 変化なし: `protocol`, `username`, `password`, `hostname`, `port`
* 変化あり: `pathname`, `search`

結合先が`/page`のような最初が`/`で始まる場合です。
そのとき、結合元の`pathname`以降が結合先で置き換わります。

例:
* `https://example.com` + `/page` = `https://example.com/page`
* `http://example.com` + `/page` = `http://example.com/page`
* `https://example.com/xxx` + `/page` = `https://example.com/page`
* `https://example.com/xxx?x=y` + `/page` = `https://example.com/page`
* `https://example.com` + `/page?x=y` = `https://example.com/page?x=y`

## 結合先を`/`で分割したものに`..`が含まれる場合
* 変化なし: `protocol`, `username`, `password`, `hostname`, `port`
* 変化あり: `pathname`, `search`

これは、パスを一層上に上がるイメージです。`pathname` 以降のみを変更します。

例えば、`https://example.com/aaa/bbb/ccc?x=y` + `../../../../../xxx`を考えてみます。

1. `ccc` と `..` が打ち消しあう。このとき、`?x=y`は消える、
(`https://example.com/aaa/bbb` + `../../../../xxx`)
2. `bbb` と `..` が打ち消しあう。
(`https://example.com/aaa` + `../../../xxx`)
3. `aaa` と `..` が打ち消しあう。
(`https://example.com/` + `../../xxx`)
4. `..` は 空の `pathname` について意味がないので、先頭の`..`が消える
(`https://example.com/` + `xxx`)
5. `https://example.com/xxx` になる

みたいなイメージです。

## 結合先を`/`で分割したものに`.`が含まれる場合
* 変化なし: `protocol`, `username`, `password`, `hostname`, `port`
* 変化あり: `pathname`, `search`

これは無視されます。
例えば、結合先が`./a/b/c`の場合は`a/b/c`と等価になります。

## 結合先が通常の文字列
* 変化なし: `protocol`, `username`, `password`, `hostname`, `port`
* 変化あり: `pathname`, `search`

これの挙動を完全に理解していませんでした。

結合元を`/`で分割した最後の部分以降を、結合先で置き換えます。

<img width="571" alt="Untitled 17" src="https://github.com/user-attachments/assets/23e22a74-0beb-498c-86ed-5b8dc10af911">

POSIX システムとは少し違うので注意です。

また、`pathname`が何もないか、`/`のみの場合、`https://example.com/` + `aaa/bbb` = `https://example.com/aaa/bbb` のようになります。

## 結合先が`//`で始まる
* 変化なし: `protocol` 
* 変化あり: `username`, `password`, `hostname`, `port`, `pathname`, `search`

これは、プロトコルはそのままでそれ以外を結合先で置き換えます。

例:
* `https://example.com/path` + `//example.net/` = `https://example.net/`
* `http://example.com/path` + `//example.net/` = `http://example.net/`

同じプロトコルで `<script>` を使いたい時などに使えます。

## まとめ
ファイルパスの操作などで URL API を使ったりする時は、`path.join` と挙動が違うので、気をつけたほうがいいです。
