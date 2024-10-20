---
title: "JS ランタイムへ求める妄想" # 記事のタイトル
emoji: "😴" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["ecmascript", "deno", "bun", "node.js", "wintercg"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

私がバックエンド JavaScript に求めている妄想を、何様目線で綴っていきたいです。

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

## My Opinion

私の個人的な意見。

### 本当に `node:*` API は正解なのか

前述したように、`node:*` API は各バックエンド JS ランタイム間の「事実上の標準」なっています。私はそれに対して、本当にそれでいいのかと思います。

