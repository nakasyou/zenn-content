---
title: "JS ランタイムの標準への意見と妄想" # 記事のタイトル
emoji: "😴" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["ecmascript", "deno", "bun", "node.js", "wintercg"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

私がバックエンド JavaScript に求めている妄想を、**お前何様目線**で綴っていきたいです。

## 目的

* 現在の JavaScript ランタイムについて知ってもらう
* 共感してほしい

## 「バックエンド JS ランタイム」を定義

ブラウザでない JavaScript 実行環境 (Node.js, Bun, Deno, Workerd, WinterJS...) とします。

## 標準？

Web 標準は、ブラウザのために作られました。そのため、バックエンド JS ランタイムが提供するサーバー機能などは Web 標準の範囲外で、各ランタイムが独自に拡張しています。これについては、
https://zenn.dev/nakasyou/articles/webstandard_and_runtimes
を読んでいただくとわかるかと思います。

### 各ランタイム独自拡張の問題点

これによって起こる問題として、ランタイム間の移行可能性が低くなってしまうということです。
例えば、以下のように、 Deno で書かれたコードを Bun に移行するには一定のコストが必要です。

```ts
// Deno
const text = await Deno.readTextFile('./example.txt')

// Bun
const text = await Bun.file('./example.txt').text()
```

これはファイルの読み取りの例ですが、IP アドレスを取得するときも同様です。
```ts
// Deno
Deno.serve((_req, addr) => new Response(
  addr.hostname // 取得
))

// Bun
Bun.serve({ fetch: (req, server) => new Response(
  server.requestIP(req) // 取得
)})
```

Web 標準にこのような処理はないので、各ランタイムが独自の API を使用しています。
これにより、移行可能性が損なわれてしまっていますね。統合されていないことによる問題です。

### 「事実上の標準」としての Node.js

Deno, Bun, Cloudflare Workers などは、**互換性** のために、Node.js の API を実装しています。

例えば、
```js
import { readFile } from 'node:fs/promises'

const text = await readFile('./example.txt', { encoding: 'utf-8' })
```
のような Node.js のコードが存在するとします。Deno や Bun は従来の Node.js のポジションを狙っているので、Node.js の `node:*` API が使われているコードが動くようになっています。Cloudflare Workers でも同様に、既存の Node.js 向けライブラリが動くように Node.js の API との互換性を作っています。

つまり、Deno や Bun などの色々なランタイムが独自の API を作っている中でも、Node.js だけが「互換性」の名目で共通言語として君臨しているのです。まさに、「事実上の標準」です。
`node:fs` や `node:assert` など、`node:*` の API を使えば、さまざまなランタイムで動かせるわけです。

## 本当に `node:*` API は正解なのか

ここから私の個人的な意見です。

前述したように、`node:*` API は各バックエンド JS ランタイム間の「事実上の標準」なっています。私はそれに対して、本当にそれでいいのかと思います。

### 策定者の問題

まず、`node:*` の API は Node.js のコミュニティが作っています。W3C や ECMA、WHATWG などの団体ではなく、コントリビュータなどが作成しています。Bun や Deno が互換性のために Node.js に従っています。互換性のために Node.js のアップデートのたびに実装していくのはいいかもしれませんが、事実上の標準を作りだすために Node.js にしたがうのは違和感を感じます。Deno や Bun はただひたすらに Node.js に従っていくのでしょうか。

Node.js コミュニティがバックエンド JS ランタイムの標準を作っているという状況です。

### Web 標準との剥離

`node:*` API は古いものも含まれています。

例えば、ファイル操作の `node:fs` は Promise より前に作られたので、 Promise 非対応です。
```js
import { readFile } from 'node:fs'

readFile('./example.txt', 'utf-8', (err, data) => console.log(data))
```
もちろん Promise による `node:fs/promises` もあります。
```js
import { readFile } from 'node:fs/promises'

const text = await readFile('./example.txt', { encoding: 'utf-8' })
```
個人的に疑問に思うのが、最後の`/promises` です。現代の非同期処理の多くがデフォルトで Promise のなか、デフォルトで Promise ではないことを意味するのかにように `/promises` がついています。別に Node.js だけの API ならいいかもしれませんが、標準として使うには違和感があります。

Stream も同様です。現代は ReadableStream/WritableStream というブラウザでも使える Web 標準 API が存在するのに、Node.js は独自の Stream を提供しています。[process.stdout](https://nodejs.org/api/process.html#process_process_stdout) なども非 Web 標準なストリームです。

これらより、Web 標準で済むものが Web 標準ではない Node.js API になってしまっています。Web 標準でできることが多い現代なのに、Web 標準ではない API がバックエンド JS ランタイムの事実上の標準になってしまっています。これは問題ではないでしょうか。

### 固有名詞

これは根拠として弱いですが、マルチランタイム時代に `node:` というランタイム固有名詞の prefix がついていることは違和感があります。

## ほしいもの

ここまで、事実 → 意見 と述べてきましたが、**私の妄想**のフェーズです。

私がバックエンド JS ランタイムへ求めるものを綴っていきたいです。

`node:*` API ではないバックエンド JS ランタイム間の標準がほしいです！！

例えば、一例ですが、
```ts
import { open } from 'hyojun:fs'

const file = await open('./a.txt')

file.readable // ReadableStream
file.blob() // Blob
```
みたいなの良くないですか？ Web 標準にのっとったバックエンド JS ランタイム間の標準がほしいです！

Request => Response なサーバーもあったらいいな
```ts
import { serve } from 'hyojun:serve'

serve({
  fetch(req: Request, info) {
    console.log('IP Addr', info.remote.hostname)
  }
})
```
現在 Deno/Bun/Cloudflare Workers における Request => Response API のハンドラーでは、それぞれ形が違います。fetch の第二引数は Deno ではクライアントの情報、Bun ではサーバーのインスタンス、Workers では環境変数及び Bindings です。これも統合してほしいという妄想。

### メリット

* Web 標準にバックエンド JS ランタイム API を使える！
* Bun -> Deno, Deno -> Bun など、ランタイムの移行がスムーズにできる！
  ランタイム制作者はよりスピードなど、独自のユニークな機能で競争に集中できるようになるかもしれません。
* Node.js の「事実上の標準」から脱却できる！

### 予想される反論とそれに対する反論

#### [WinterCG](https://wintercg.org) があるじゃないか！

WinterCG は、各ランタイムの標準を作ることを促進する団体です。
WinterCG は、仕様を作らず、既存の仕様を使います。[^wintercg_does_not_create]
これは例えば、Node.js の API を「事実上の標準」から「標準」にするということです。

WinterCG が Web 標準に順序した仕様を選択するのであれば、私は非常に happy になるでしょう。

[^wintercg_does_not_create]: https://wintercg.org/faq#what-we-are-not-trying-to-do

#### Web 標準にできることが増えたら

Web 標準でできることが増えて、バックエンド JS ランタイムの標準が独自に作成する範囲が狭くなった場合、バージョンアップを必要とします。破壊的変更です。

これに対して、バージョンを明記できるようにしたらいいかもしれません:
```ts
import { open } from 'hyojun5:fs'
```
パッケージごとの設定ファイルに書き込み、より import をスマートにするという選択もあるかもしれません。

## まとめ

共感してほしい。
