---
title: "Web 標準と、その限界" # 記事のタイトル
emoji: "😱" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["wintercg", "ecmascript", "javascript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

JavaScript、たくさんバックエンドで使われてますよね、あなたも使ったことはありませんか？

そんな バックエンドでも使われている JavaScript。そこに標準で組み込まれている Web 標準な API 。それを考えていきます。

## Web 標準 とは何か
Web を構成するための技術として、主に HTML/CSS/JavaScript があります。
どの**ブラウザ**でサイトを見ても同じ結果が得られるように、その HTML/CSS/JavaScript をまとめている仕様のことです。

例えば、 HTML の仕様は [HTML Living Standard](https://html.spec.whatwg.org/multipage/) が主流で、 WHATWG という団体が決めています。

JavaScript では、構文や基本的な機能 (`Array`などの言語使用) は ECMA という団体が ECMAScript を策定しています。この中には`fetch`は含まれていないので、先ほどの WHATWG が fetch の標準を作っています。

このように、 Web 標準 はさまざまな団体やさまざまな仕様が混ざってできていて、**ブラウザ**を支えているのです。

(ここから、Web 標準 は、 JavaScript のものについてを指します。)

## Node.js と Web 標準

Ryan Dahl は Chrome 内の V8 エンジンをサーバーサイドに持ち込み、 Node.js を作成しました。
これは JavaScript をサーバーサイドで書けるというものでした。
もちろん Web 標準にはファイル書き込み API などは存在しないので
```js
var fs = require('fs')
```
のように Node.js 独自のファイル書き込み API を搭載しました。

リクエストには、`http` モジュールを使用していたそうで、 XMLHttpRequest もなかったようです。

Node.js くらいしかサーバーサイド JS のランタイムはなかったので、このように `node:` API を拡張していく形式に問題はなかったと思います。

## Node.js 以外のランタイムの登場

これが現在です。ECMAScript にのっとり作られたランタイムがたくさん登場しました。羅列しちゃいます:

* [Deno](https://deno.com)
* [Bun](https://bun.sh)
* [Google Apps Script](https://workspace.google.com/intl/ja/products/apps-script/)
* [Cloudflare Workers](https://www.cloudflare.com/ja-jp/developer-platform/workers/)
* [Vercel](https://vercel.com/)
* [WinterJS](https://winterjs.org)
* And more...

めっちゃありますね。Web 標準 のおかげで、基本的なコードは同じように書けるわけです。

例えば、上記のランタイムのなかでこのコードは動きます:
```js
console.log(1 + 1)
// -> 2
```
これは Web 標準として console.log が定義されているからです。

以下のコードは Fetch という Web Standard にのっとっていれば動くはずです。そのため、Node.js, Bun, Deno などで動きます。
```js
await fetch('https://httpbin.org/get').then(res => res.json())
```
ブラウザの API と全く同じです。

しかし、サーバーの起動を書いてみるとどうでしょう:
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
全然コードが違いますね。Web 標準というものがあるのに違います。なぜでしょう。

それは、 Web 標準 は**ブラウザ**のために作られたものだからです !!
ブラウザに搭載されている JavaScript には、サーバーを起動させる機能は存在しません。ブラウザはそもそもサーバーにアクセスするためのものですから。

そのほかにも、 WebSocket サーバーおいても、クライアントは共通のコードだけれどのサーバーとなると違う API を使うことになります。

## Web 標準の限界、拡張
Web Standard は**ブラウザ**のために作られたものなので、サーバー環境は想定されていません。

そのため、各ランタイムが仕様を拡張しています。

例えば、Deno/Bun/Workers は、サーバー時に`Request`をクライアントからのリクエストとして使います。
```ts
type Handler = (req: Request) => Promise<Response>
```
のようなインターフェースでサーバーを定義するイメージです。ここで、`Request`、`Response` は Web 標準な API ですね。
このインターフェースでは、`Request` をクライアントからの「受信」として用いています。ブラウザでの `Request` は受信用ではなく `fetch` を介して送信するのが主な用途です。(Service Worker ではそうではありませんが)。
送信用の API なので、例えばクライアント側の IP アドレスを取得したい場合、 `Request` には含まれません。ブラウザでの `Request` には IP アドレス取得処理は不要だからです。これは Web 標準の限界です。

でも、サーバーを構築するのにあたって、クライアントの IP アドレスは取得できた方がいいです。そのため、
```ts
Bun.serve({
  fetch(request, server) {
    const addr = server.requestIp(request) // Get IP Addr with Bun
  }
})
Deno.serve({
  fetch(request, connInfo) {
    const addr = connInfo.remoteAddr // Get IP Addr with Deno
  }
})
export default {
  fetch(request) {
    const addr = request.headers.get('CF-Connecting-IP') // Get IP Addr with Cloudflare Workers
  }
}
```
のような方法で、各ランタイムが Web 標準を用いて独自に拡張しているのです。

さらに、Web Standard な API に対し、仕様にない実装をしたりすることもあります。
仕様では、`fetch`で`redirect: manual`をするとリダイレクト時にヘッダーを返しませんが、Denoだと`Location`ヘッダーなどをそのまま返したりします。
また、Bunでは、`fetch(url, { proxy: proxyUrl })`のようにプロキシを介して`fetch`できたりします。

このように、 Web 標準 はブラウザのためのものなので、サーバーに足りない機能を付け足すために、各ランタイムが拡張しています。

## 拡張のデメリット

拡張することによって Web 標準でできないことを突破していますが、それにはデメリットもあります。
それは、ランタイム間で API が異なってしまうことです。ランタイムを変更するときにコードを変更する必要が出てきてしまうかもしれません。

以下の Bun のコード
```ts
// プロキシを介してハッシュパスワードを送信する
await fetch(url, {
  method: 'POST'
  proxy: proxyUrl,
  body: await Bun.password.hash(password, { algorithm: 'bcrypt' })
})
```
を Deno に変更するとき、
```ts
// プロキシを介してハッシュパスワードを送信する
import { hash } from 'npm:bcryptjs'

const client = Deno.createHttpClient({
  proxy: { url: proxyUrl }
})
await fetch(url, {
  method: 'POST'
  client,
  body: await hash(password, 8)
})
```
みたいに変更する必要があります。小規模なプロジェクトならまだいいですが、大規模なプロジェクトになると移行コストが増えてしまうかもしれません。
Web 標準にのっとった部分の置き換えならコストは低いはずです。

## 解決策
### 1: `node:` API を使う

Deno, Bun は Node.js との互換性があるので、`node:` API を用いることによって Web 標準に依存せずに移行コストを下げることが可能かもしれません。

しかし、`node:` は Cloudflare Workers では一部使えない API がありますし、ブラウザでも使えません。
また、仕様策定が WHATWG や W3C などの幅広く意見を聞き公平に Web 標準を策定する団体が決めるのではなく、 Node.js が決めていることに問題があるかもしれません。
個人的に、マルチランタイムの時代に `node:` というランタイムの固有名詞を含むものを共通にしようとしているのに違和感を少し感じます。

### 2: ラッパーライブラリを使う

API をラップしてくれるライブラリを使うのも一つの手だと思います。

例えば [Hono](https://hono.dev) という Web 標準ベースの Web フレームワークは、
```ts
import { Hono } from 'hono'

import { getConnInfo } from 'hono/vercel'
import { getConnInfo } from 'hono/cloudflare-workers'
import { getConnInfo } from 'hono/bun'
import { getConnInfo } from 'hono/deno'
import { getConnInfo } from '@hono/node-server'

const app = new Hono()

app.get('/', c => {
  const addr = getConnInfo(c)
})
```
のようにして、IP アドレスを取得できます。上の import を使用するランタイムによって変更することで、移行コストを下げることができます。

しかし、このような方法は、拡張をラップしているだけであり、根本的な解決にはならないと思います。

### 3: [WinterCG](https://wintercg.org) に期待する

[WinterCG](https://wintercg.org) というコミュニティーグループが存在します。これは、ブラウザ以外の JS ランタイムの API を標準化することを目的としています。
いわば、「サーバーサイドの Web 標準」を策定するみたいな感じです。

Cloudflare Workers, Deno, Bun, fastly, Vercel, netlify などが参加しています。(Bun はないです...)

これにより、サーバーサイドの JS API が統合され、Web 標準の限界を乗り越えることができるのではないかと思います。

`import.meta.*` を集める？試みをしていたり
https://github.com/wintercg/import-meta-registry

WinterCG による `fetch` 標準を策定しようとしていたり
https://github.com/wintercg/fetch

期待してみる選択肢はいいかもしれません。

## まとめ

Web 標準は美しいと思います。また、それに即したコードが書けるのはまた良いことだと考えます。
しかし、 Web 標準には限界があり、Web 標準が全てに通用するとは限らないので、それを考慮してサーバーサイド JavaScript を書いていくといいと思います。
