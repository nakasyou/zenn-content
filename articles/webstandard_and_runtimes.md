---
title: "Web Standard と バックエンド" # 記事のタイトル
emoji: "🟰" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["wintercg", "ecmascript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

JavaScript、たくさんバックエンドで使われてますよね、あなたも使ったことはありませんか？

そんな バックエンドの JavaScript と WebStandard についての私の考えです。

## Web Standard とは何か
Web を構成するための技術として、主に HTML/CSS/JavaScript があります。
どの**ブラウザ**でサイトを見ても同じ結果が得られるように、その HTML/CSS/JavaScript をまとめている仕様のことです。

例えば、 HTML の仕様は [HTML Living Standard](https://html.spec.whatwg.org/multipage/) が主流で、 WHATWG という団体が決めています。

JavaScript では、構文や基本的な機能 (`Array`などの言語使用) は ECMA という団体が ECMAScript を策定しています。この中には`fetch`は含まれていないので、先ほどの WHATWG が fetch の標準を作っています。

このように、 Web Standard はさまざまな団体やさまざまな仕様が混ざってできていて、**ブラウザ**を支えているのです。

(ここから、Web Standard は、 JavaScript についてを指します。)
## Node.js と Web Standard
Ryan Dahl は Chrome 内の V8 エンジンをサーバーサイドに持ち込み、 Node.js を作成しました。
これは JavaScript をサーバーサイドで書けるというものでした。
もちろん Web 標準にはファイル書き込み API などは存在しないので
```js
var fs = require('fs')
```
のように Node.js 独自のファイル書き込み API を搭載しました。
XMLHttpRequest もないです。

Node.js くらいしかサーバーサイド JS のランタイムはなかったので、この API を拡張していく形式に問題はなかったと思います。

## Node.js 以外のランタイムの登場
これが現在です。ECMAScript にのっとり作られたランタイムがたくさん登場しました。羅列しちゃいます:

* Deno
* Bun
* Google Apps Script
* Cloudflare Workers
* WinterJS
* And more...

めっちゃありますね。Web Standardという標準のおかげで、基本的なコードは同じように書けるわけです。

例えば、上記のランタイムのなかでこのコードは動きます:
```js
console.log(1 + 1)
// -> 2
```

以下のコードは Fetch という Web Standard にのっとっていれば動くはずです:
```js
await fetch('https://httpbin.org/get').then(res => res.json())
```

しかし、サーバーを書いてみるとどうでしょう:
```js
Deno.serve(() => new Response('hello world')) // Deno

Bun.serve({
  fetch: () => new Response('hello world')
}) // Bun

export default {
  fetch: () => new Response('hello world')
} // workers

require('http').createServer((req, res) => {
  res.end('hello world')
}).listen(3000) // Node.js
```
異なります。 Web Standard があるのに異なります。なぜでしょう。

それは、 Web Standard は**ブラウザ**のために作られたものだからです!!
ブラウザには、サーバーを起動させる機能は存在しません。サーバーにアクセスするためのものですから。

また、例えば WebSocket においても、クライアントは共通のコードだけれどのサーバーとなると違う API を使うことになります。

## Web Standard の限界と拡張
Web Standard は**ブラウザ**のために作られたものなので、サーバーは想定されていません。

そのため、各ランタイムが仕様を拡張しています。

IP アドレスを例に出してみます。Deno/Bun/Workers は、`Request`をクライアントからのリクエストとして使います。`Request`は本来受け取る用ではなく送信する用途のはずなので、クライアントの IP アドレスは `Request` に搭載されていません。なので、サーバーのハンドラーの第2引数や、ヘッダーを使って各ランタイムが拡張しています。
また、Deno や Bun は Web Standard にない API を提供するために、 `Deno`や`Bun`といったグローバル変数を提供しています。

さらに、Web Standard な API に対し、仕様にない実装をしたりすることもあります。

仕様では、`fetch`で`redirect: manual`をするとリダイレクト時にヘッダーを返しませんが、Denoだと`Location`ヘッダーなどをそのまま返したりします。
Bunでは、`fetch(url, { proxy: proxyUrl })`のようにプロキシを介して`fetch`できたりします。

このように、 Web Standard はブラウザのためのものなので各ランタイムが拡張していっているのです。
