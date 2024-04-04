---
title: "You don't need Node.js" # 記事のタイトル
emoji: "🌪️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["javascript", "node.js", "deno", "bun"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

Node.jsはいらない場合がある、むしろいらない場合の方が多いかもしれない、
そしてDenoとBunを使い分けて代替する方法を説明するという記事です。

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

#### Bunのインストール速度
Bunの依存関係インストール速度は爆速です。`npm i`の25倍早くなるようです^[https://bun.sh/docs/cli/install]！

#### 実行速度
次のファイル書き込みコードをNode/Deno/Bunそれぞれのランタイムで試してみます

:::message
Xで、[@faa0311](https://twitter.com/faa0311)さんから[指摘](https://x.com/faa0311/status/1775596574657388791?s=20)がありましたが、このベンチマークにはランタイムの起動速度が含まれている可能性があります。
:::

```ts
import { writeFile } from 'node:fs/promises'

await writeFile('./a.txt', Math.random().toString())
```
30回平均が、以下のようになりました:

![IMG_2855](https://github.com/nakasyou/zenn-content/assets/79000684/4e0e501b-ee8d-4f7e-91c0-88f32ab1c0f5)

速いですね。
[Bunのトップページ](https://bun.sh)には、Node/Deno/Bunのベンチマークが載っています。

Webサーバーの応答速度などの項目があり、NodeよりDeno/Bunが高速です。


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
| API | Node.js^[https://nodejs.org/docs/latest-v20.x/api/] | Deno | Bun^[https://bun.sh/docs/runtime/web-apis] |
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
この2つの使い分けで、Node.jsを代替することができます。

### Bunの方がNode.jsプロジェクトに適している理由
#### DenoとBunのnpmインストール速度
これは、Bunの方が速いです。実験してみました:

まず、DenoとBunのキャッシュを削除します。
```shell
rm -rf ~/.bun/install
rm -rf ~/.cache/deno
```
次に、Astroをインストールしてみます。
```shell
bun add astro
deno cache astro
```
の速度を比較するのです。

次のようなBunコードを使います。

```ts
await Bun.$`rm -rf node_modules`
// Bun
const bunStarted = performance.now()
await Bun.$`bun add astro`
const bunResult = performance.now() - bunStarted


await Bun.$`rm -rf node_modules`

// Deno
const denoStarted = performance.now()
await Bun.$`deno cache npm:astro`
const denoResult = performance.now() - denoStarted

console.log(`Bun: ${bunResult} ms`)
console.log(`Deno: ${denoResult} ms`)
```
こうなりました:
![IMG_2853](https://github.com/nakasyou/zenn-content/assets/79000684/2e6ead0e-4433-4213-a775-732f7b819d88)

npmインストール速度において、Bunは高速です。

#### Node.js の仕組みの問題
- Denoは、基本的に`package.json`を使いませんが^[使うことも可能]、Bunでは使います。
- Denoは、基本的に`node_modules`を使いませんが^[生成することも可能]、Bunでは使います。
- Denoは、基本的に`.ts`などの拡張子をつけてimportしますが^[unstableなフラグで省略可能]、Bunでは省略可能です。

### Node.jsプロジェクト以外ならDeno
Denoは、Node.jsの互換性は意識はしていますが、Nodeとは別のところを行くランタイム、というイメージです。

#### ゼロコンフィグ
Denoは、設定ファイル不要です。
`task.ts`を作成すると、`deno run task.ts`で動きます。プロジェクトという概念なしで動かせます。

設定ファイルを作れないわけではなくて、ある程度の大きさのことをするときは作成するといいかもかもしれません。

#### 手軽
Pythonは、気軽にプログラムを作成して実行できますが、Denoも同じ感じです。
シンプルに、CLIから計算処理、バックエンドも作れたりするランタイムです。

#### モジュールはグローバル
Denoのモジュールはグローバルにキャッシュされます。
また、DenoはURLでimportします。
```ts
import * as module from 'https://example.com/mod.ts'
```
のようなイメージです。これはグローバルにキャッシュされます。

また、npmモジュールを利用したいときは、`npm:package`というURLを使えます。これもグローバルにキャッシュされます。

Pythonに似ています。
***
このように、DenoはもともとNode.jsの反省を踏まえて作られた、新しいランタイムを目指していたのに対し、BunはNode.jsとの互換性を意識して作られています。

そのため、Node.js用に作られたViteやNext.jsのようなフレームワークを使うときは、高速な代替であるBunを使うことがおすすめです。

一方、バックエンドやコンピュータ中心などのものを作ったりするときは、Denoがおすすめです。

## Node.jsプロジェクトをBunにスイッチする
これは超簡単です。
Node.jsプロジェクト内で、`npm i`の代わりに`bun i`をして依存関係をインストールするだけです。
そして、`node main.js`を`bun --bun main.js`、`npm run dev`を`bun --bun run dev`にするだけです。

### 既存のプロジェクトでこっそりBunを使う
`bun i`は、`package-lock.json`および`yarn.lock`から自動で読み取ってインストールしてくれます。
```shell
bun i --no-save
```
をすれば、npmやyarnを使っているプロジェクトのうんざりする依存関係インストール時間を短縮できます。

## Node.jsをやめられないところ
例えば、BunでViteを動かすとき、デフォルトでは、Node.jsが裏で動いたりしています。
`bunx --bun vite`のように`--bun`を使い、裏もBunで動かすこともできますが、SSRでは不安定だったりします。

また、DenoやBunのライブラリをnpmにpublishしたいときは、やっぱり`npm`コマンドが必要なので、Node.jsは必要です。

## まとめ
もちろん一概に言えることではないですが、Node.jsのようにプロジェクトを作ったり、Node.jsのフレームワークを使うならBunに変えられます。単純にNode.jsをバックエンドのJavaScriptランタイムとして使うならDenoに変えることができます。

そして、変えることでそれぞれのランタイムのメリットを享受できます。
みなさんも、Node.jsを、代替してみてはどうでしょうか？
