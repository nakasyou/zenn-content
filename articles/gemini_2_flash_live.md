---
title: "Gemini 2.0 Flash Live API 超解説" # 記事のタイトル
emoji: "🗣️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["gemini", "websocket"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

Gemini 2.0 Flash やばいですよね。その目玉機能の一つに、リアルタイムの音声動画通信があります。動画と音声で LLM とリアルタイム会話できるのです！
その API である Multimodal Live API があるのですが、これがとても情報量が少ないので、解説していきたいと思います。

Python の SDK しか提供されていないので、JavaScript 大好き nakasyou としては JavaScript で使いたくなったので、コードなどから調べました。

TypeScript でサンプルコードを書いていきます。

## 概要

## Multimodal Live API ってどんなの

リアルタイムで音声と動画で通信できるやつです。テキストも対応しています。
従来の LLM は、1プロンプトに対して1レスポンスなのはわかりますよね。それに対して、Live Streaming では、通常の会話のようにユーザーとモデルが任意のタイミングでメッセージを送ることができます。
さらに、ユーザーはテキストだけではなく、音声や動画を送ることができます。例えばカメラを知らない花にかざして「この花の名前は？」のように口で聞くことができます。
また、モデルはテキストだけでなく音声で応答できます。「こんにちは」と音声で話すと「こんにちは」と音声で返してくれたりします。

## 通信方法

従来の LLM API は、通信のために HTTP Streaming を使用していました。これは一方向の情報ストリーミングを可能にする技術です。ChatGPT からのレスポンスはすらすらとしていますが、これは HTTP Streaming を使用しているからです。
API にプロンプトを送ると、ストリーミングが開始され、一方向に結果が出力されるという具合です。

一方、Gemini 2.0 Flash の Multimodal Live API は、WebSocket を使用しています。 WebSocket は、双方向の通信を可能にする技術です。
従来の LLM API は、最初にプロンプトを送るだけなので、HTTP Streaming で十分でしたが、Multimodal Live API は、それだけでは足りません。ユーザーが任意のタイミングで情報を送信する必要があるためです。
例えば、Gemini の説明を聞いているときに、「やっぱり説明をやめて」みたいなことができるわけです。これには双方向通信をしなければならないので、 WebSocket を使用しているというわけです。

## 接続

WebSocket に接続するコードです:
```ts
const ws = new WebSocket('wss://generativelanguage.googleapis.com/ws/google.ai.generativelanguage.v1alpha.GenerativeService.BidiGenerateContent?key=apiKey')
```
ポイントとしては、
- エンドポイントは `wss://generativelanguage.googleapis.com/ws/google.ai.generativelanguage.v1alpha.GenerativeService.BidiGenerateContent`
- API Key は `key` パラメーターで指定する

といったところです。

## セットアップ

generationConfig やシステムプロンプトはどうやって指定するんだと思ったそこのあなた。JSON 形式のメッセージを送ります。

```ts
ws.send(JSON.stringify({
  setup: {
    // セットアップ用のデータ
  }
}))
```
のようなイメージです。セットアップで何が指定できるかは型定義として以下に置いておきます。
```ts
import type { Tool, Content, GenerationConfig } from '@google/generative-ai' // もともとある型

// こいつを指定する
interface Setup {
  // models/gemini-2.0-flash-exp みたいに、必須プロパティ
  model: `models/${string}`

  // 普通とちょっと違う GenerationConfig です。
  generationConfig?: LiveGenerationConfig

  // システムプロンプト。
  systemInstruction?: Content

  // ツール。Function Calling とか使える
  tools?: Tool[]
}

interface LiveGenerationConfig extends GenerationConfig {
  // これらの型はサポートされてない
  responseLogprobs?: never
  responseMimeType?: never
  logprobs?: never
  responseSchema?: never
  stopSequences?: never

  // 重要なので後で説明
  responseModalities?: ("TEXT" | "IMAGE" | "AUDIO")[]

  // 声とか変更できる
  speechConfig?: LiveSpeechConfig
}

interface LiveSpeechConfig {
  voiceConfig?: {
    prebuiltVoiceConfig?: {
      voiceName?: string // 変更したい声の名前
    }
  }
}
```

この `responseModalities` というのが、モデルの出力形式です。デフォルトは、`['AUDIO']`です。AUDIO だと、モデルは音声を出力(喋る)します。TEXT だと、テキストを返します。IMAGE は多分将来的に解放される、画像生成だと思います。
配列形式ですが、2つ以上していするとエラー出ます。

また、正常にセットアップできると
```json
{
  "setupComplete": {}
}
```
という JSON メッセージが送られてきます。そうしたら会話スタートです！

## メッセージの受信

メッセージは、JSON 形式で送られてくるので、JSON 形式でパースすればよいのです。
```ts
ws.onmessage = async (event) => {
  const data = await new Response(event.data).json()
}
```
ここで注意しなければならないポイントは、メッセージは文字列形式で送られてくるとは限らないということです。Blob 形式や ArrayBuffer 形式で送られてきてもパースできるようにハンドリングしましょう。この場合は、JavaScript の `Response` を使用して正規化しています。

モデルには、「ターン」という概念があります。あなたが面接官と話すとしましょう。会話のキャッチボールなので、面接官が話しているときは会話を遮らないはずです。それと同じように、モデルにターンという概念があります。モデルが話している途中かどうかの状態があるというわけです。
ただし、ユーザーにはターンという概念はありません。ユーザーは好きなときにモデルの話を遮ることはできます。

送られてくるメッセージの種類は 4 つで、それぞれ役割が違います。それぞれ紹介します。

### setupComplete
setupComplete メッセージは、先ほど紹介したセットアップの完了後にサーバーから送られてくるメッセージです。先ほど紹介したものと同様に、以下の形式をしています。
```json
{
  "setupComplete": {}
}
```
setupComplete の中身は空です。

### serverContent

serverContent メッセージは、モデルからのメッセージです。テキストや音声が含まれています。
TypeScript 型にすると以下のような形式です。ドキュメントに書かれているものと異なりますが、実際はこのような挙動をします。
```ts
import type { Content } from '@google/generative-ai' // もともとある型

// これが送られてくる
interface Message {
  serverContent: {
    // ターンが終了したことを示す
    turnComplete: true
  } | {
    modelTurn: Content
  }
}
```
Content はつまり、ロールとレスポンスを含むものです。

以下のようにハンドリングすることができます。このメッセージは、複数の Part を含みます。(複数含むことを見たことがありませんが)
```ts
// Part を処理する
async function handlePart(part: Part) {
  // responseModalities に AUDIO を指定した場合
  await playSound(part.inlineData.data)

  // responseModalities に TEXT を指定した場合
  console.log('モデルからのメッセージ', part.text)
}
function handleServerContentMessage (message: Message) {
  if ('turnComplete' in message && message.turnComplete) {
    // モデルのターンが終了した
    console.log('モデルは今話したいことを全部話した！')
  } else {
    // Content を処理する
    for (const part of content.parts) {
      // 複数の Part が含まれる
      handlePart(part)
    }
  }
}
```
ここで重要なのが、先ほど記述した responseModalities です。
responseModalities の指定によって、音声が返ってくるかテキストが返ってくるかが異なるため、それに沿って Part をハンドリングする必要があります。
例えば TEXT を指定した場合、
```json
{
  "serverContent": {
    "modelTurn": {
      "parts": [{ "text": "こんにちは！" }]
    }
  }
}
```

のような JSON が送られます。 AUDIO の場合、
```json
{
  "serverContent": {
    "modelTurn": {
      "parts": [{
        "inlineData": { "type": "audio/pcm;rate=24000", "data": "<base64 形式のデータ>" }
      }]
    }
  }
}
```
のようなものが送られてきます。この送られてきた音声は、pcm形式の音声で、24000Hzのサンプルレートです。適切な方法で処理する必要がありまし。

### toolCall

toolCall メッセージは、Gemini API の Function Calling のように、リアルタイムでモデルが Function Call を要求するためのメッセージです。
型は、以下の通りです。特殊なことはなく、通常の Function Calling と似ているので、これ以上の説明は割愛します。
```ts
import type { FunctionCall } from '@google/generative-ai' // もともとある型

interface ToolCallMessage {
  toolCall: {
    functionCalls: FunctionCall[]
  }
}
```

### toolCallCancellation

toolCallCancellation メッセージは、toolCall で要求した Function Calling を「やっぱやめた」するためのものです。
例えば、ユーザーが「今の天気教えて、東京。間違えた、大阪」のようにしたとき、最初の東京の天気を要求する Function Calling はキャンセルする必要があります。
私の経験では、話題がすぐに変わった(私が変えた)とき受信しました。

型は以下のようになります:
```ts
interface ToolCallCancellationMessage {
  toolCallCancellation: {
    ids: string[] // キャンセルする関数呼び出し ID の配列
  }
}
```

## メッセージの送信


