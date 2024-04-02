---
title: "You don't need Node.js" # 記事のタイトル
emoji: "🌪️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["javascript", "node.js", "deno", "bun"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
Node.jsはいらない場合がある、むしろいらない場合の方が多いかもしれない、という記事です。

## Post Node.js ランタイムの登場
Node.js のあとにできたランタイムがいくつも登場しています。
- [Deno](https://deno.com)
- [Bun](https://bun.sh)
- [WinterJS](https://wasmer.io/posts/winterjs-v1)
- [LLRT](https://github.com/awslabs/llrt)

この中でも、人気である**Deno**と**Bun**を中心に考えていきます。

## DenoやBunに変えるメリット
これがなければNode.jsから変える必要はないと思います。

私は、以下の3つが、2ランタイムに共通して言える大きなメリットだと思います:
- **ネイティブ**TypeScriptサポート
- 高速
- Web標準

### ネイティブTypeScriptサポート
現在、JavaScriptを記述するときは、TypeScriptを利用することが多いと思います。

Node.jsでTypeScriptを使うとき。`tsc`でコンパイルしてから実行したり、`ts-node`や`tsx`で行っているのではないでしょうか？
これは不便だと思います。私はNode.jsよりDenoを先に使い始めていたので、Node.jsでTypeScriptを使ったとき、超不便だと思いました。

これがDenoやBunだと要らないのです。
```sh
deno run main.ts
bun main.ts
```
で動いちゃいます。
これは超便利です。

### 高速
DenoやBunは実行速度が速いです。

### Web標準
DenoやBunは、Node.jsよりもWeb標準に忠実です。

例えば`fetch`。JavaScriptでデータを取得するのに現在ではスタンダードですよね？
これ、Node.jsではExperimentalなAPIなのです。少し前までは、`node-fetch`などのPolyfillを使う必要がありました。
しかし、DenoやBunでは普通に使えちゃいます。

そのほかにも
- ⭕️ 使える
- 🔼 unstable/experimental
- 🙅‍♀️ 使えない

として、こんな感じです(Node.js 20, Deno 1.42, Bun 1.1):
| API | Node.js | Deno | Bun |
| --- | --- | ---| ---|
| Web Crypto API (`crypto`) | 🔼 | ⭕️ | ⭕️ |
| `Request`/`Response` | 🔼 | ⭕️ | ⭕️ |
| `FormData` | 🔼 | ⭕️ | ⭕️ |
| `WebSocket` | 🔼 | ⭕️ | ⭕️ |
| `ReadableStream`/`WriableStream` | 🔼 | ⭕️ | ⭕️ |
| `alert`/`prompt`/`confirm` | 🙅‍♀️ | ⭕️ | ⭕️ |
| Web GPU | 🙅‍♀️ | 🔼 | 🙅‍♀️ |

すごいですね。DenoやBunはNode.jsよりWeb標準に忠実です。

#### ESM
DenoおよびBun^[https://bun.sh/docs/runtime/modules#module-systems]は、デフォルトでESMを使用しています。
モジュールシステムとして、CommonJS ModulesよりESMを使うことが多い現在で、嬉しいですね。

***
Node.jsをやめたくなってきましたか？

## DenoとBun
DenoとBunのメリットについて説明しました。次は、この2つの使い分けについてです。
DenoとBunを使い分けることで、Node.jsをやめることができます。

私なりの使い分け方は、**1から作るならDeno**、**Node.jsのプロジェクトを使うならBun**です。



