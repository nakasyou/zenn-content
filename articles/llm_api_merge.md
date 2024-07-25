---
title: "事実、LLM の API は違いすぎる！ | CrossLM" # 記事のタイトル
emoji: "🤖" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["typescript", "llm"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
事実、LLM の API は違いすぎる！
どういうことかというと、LLM のプロバイダ間で API の互換性がないということです。
そう思いませんか？

## CrossLM
私は作っている PoC のライブラリです。
さまざまな LLM API のプロバイダの API の差異を吸収してくれるラッパーです。

```shell
deno add @pnsk-lab/crosslm # Deno
bunx jsr add @pnsk-lab/crosslm # Bun
pnpm dlx jsr add @pnsk-lab/crosslm # pnpm
```
でインストールし、
```ts
import { CrossLM } from '@pnsk-lab/crosslm'
import { CommandRPlus } from '@pnsk-lab/crosslm/cohere/command-r-plus'

const lm = new CrossLM([new CommandRPlus(process.env.COHERE_KEY)])

const { text } = lm.genetate([], {
  messages: [{ role: 'system', text: '1+1=?' }]
})
console.log(text)
```

普通に動きます。
また、これのプロバイダを変えても動くわけです:
```ts
import { CrossLM } from '@pnsk-lab/crosslm'
import { Gemini15Frash } from '@pnsk-lab/crosslm/gemini/15-flash'

const lm = new CrossLM([new Gemini15Frash(process.env.GOOGLE_AI_KEY)])
```
これにより、移植可能性が保たれるわけです。

同じ API を提供するラッパーです。

## ゼロ依存関係
このライブラリの依存関係は、ゼロです。

Cohere の SDK である [`cohere-ai`](https://npm.anvaka.com/#/view/2d/cohere-ai)[^cohere_ai_dep] や [`gloq-sdk`](https://npm.anvaka.com/#/view/2d/groq-sdk)[^groq_sdk_dep] は、少なくとも依存関係に `node-fetch` を持っています。

v22 から Node.js に `fetch` が実装されました。依存関係を追加して、node_modules をブラックホールに近づけるくらいなら Web Standard な API を使用した方が良いのではないか、その API を使用できないランタイムの場合、ユーザーが globalThis にポリフィルを導入すれば良いと考えています。Web Standard なランタイムを使用している人のインストール時間やディスクスペースの無駄になります。

[^cohere_ai_dep]: https://github.com/cohere-ai/cohere-typescript/blob/34c1985374d98ff68093722df205fc5225ed7517/package.json#L18
[^groq_sdk_dep]: https://github.com/groq/groq-typescript/blob/4ba6fe0c1865619fb607537620d80ef89d201077/package.json#L33

@google/generative-ai はゼロ依存関係で素敵ですね！

## Features という仕組み
LLM のプロバイダ間では、機能の提供の有無に差があります。例えば、画像を認識できるか否かや、システムプロンプトに対応しているかなどです。
それを解決するために、Features という仕組みを搭載しています。

例えば、Cohere API はストリーミングとドキュメントの読み込み、システムプロンプトをメッセージとして渡すことに対応しているため、`stream`, `system-as-role`, `input-document` などの Features が示されています。
同様に、マルチモーダルな Gemini 1.5 Flash には `input-images` というタグがついています。

```ts
const image: Blob = ...
lm.generate(['input-images'], {
  messages: [
    parts: [
      { text: 'この画像は何を示している？' },
      { image }
    ]
  ]
})
```
のように使用する Features を明示することで、それに沿ったモデルで生成することができます。

## 複数のモデルを使い分ける
CrossLM を使えば、複数のモデルの使い分けが可能です。
```ts
const lm = new CrossLM([
  new CommandRPlus(process.env.COHERE_KEY),
  new Gemini15Frash(process.env.GOOGLE_AI_KEY),
  new Groq('llama3-70b-8192', process.env.GROQ_KEY)
])
await lm.generate(...)
```
無料枠を使い果たしたなど、エラーが起きた場合は次のモデルに処理を回すので、いろいろな API Key を統合できます。
さらに、先ほどの Features 機能と組み合わせることで、たくさんあるモデルのなかから使いたい機能が備わっているモデルを引き出し、使えます。
これにより、LLM 機能の冗長性が保たれるかもしれません！

### ランダム機能
```ts
const lm = new CrossLM([
  new CommandRPlus(process.env.COHERE_KEY, { weight: 1 }),
  new Gemini15Frash(process.env.GOOGLE_AI_KEY, { weight: 0.5 }),
  new Groq('llama3-70b-8192', process.env.GROQ_KEY, { weight: 2 })
])
await lm.generate([], { ... }, { mode: 'random' })
```
のようにすると、重み付きでランダムに処理できます。
API Key の枠を分散して消費することができます。個人開発に優しくなるかもしれません！

## 欠点
欠点としては、LLM API の統合により失われる API がでてくるということです。

例えば、 Cohere API には独自の Web 検索機能があります。これは Gemini/Groq 等の API にないので、ライブラリ側が実装しなければ、その機能は使うことができません。
API の統合により、失われてしまう、カスタマイズ性が低下してしまいます。

## 伝えたいこと
メンテナンスできるかどうかすらわからないので、開発にこのライブラリを使ってみて！っていうことではないです。
このようなコンセプトを持ったライブラリができたり、プロバイダ側が共通した API を提供してほしいな、という考えです。

## リンク
https://jsr.io/@pnsk-lab/crosslm
https://github.com/pnsk-lab/CrossLM
