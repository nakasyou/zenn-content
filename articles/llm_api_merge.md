---
title: "事実、LLM の API は違いすぎる！" # 記事のタイトル
emoji: "🤖" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["typescript", "llm"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
事実、LLM の API は違いすぎる！
そう思いませんか？

## CrossLM
私は作っている PoC のライブラリです。
さまざまな LLM API のプロバイダの API の差異を吸収してくれるラッパーです。

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
import { Gemini15Frash } from '@pnsk-lab/crosslm/gemini/1-5-flash'

const lm = new CrossLM([new Gemini15Frash(process.env.GOOGLE_AI_KEY)])
```
これにより、移植可能性が保たれるわけです。

同じ API を提供するラッパーです。

## Features という仕組み
LLM のプロバイダ間では、機能の提供の有無に差があります。例えば、画像を認識できるか否かや、システムプロンプトに対応しているかなどです。
それを解決するために、Features という仕組みを搭載しています。

例えば、Cohere API はストリーミングとドキュメントの読み込み、システムプロンプトをメッセージとして渡すことに対応しているため、`stream`, `system-prompt-as-message`, `document` などの Features が示されています。
同様に、マルチモーダルな Gemini 1.5 Flash には `input-images` というタグがついています。

```ts
lm.generate(['image'], { messages: [ ... ] })
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
API Key の枠を分散して消費することができます。


