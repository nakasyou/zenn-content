---
title: "Gemini 2.0 Multimodal Live API 超解説" # 記事のタイトル
emoji: "🗣️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["gemini", "websocket", "javascript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

Gemini 2.0 Flash やばいですよね。その目玉機能の一つに、リアルタイムの音声動画通信があります。動画と音声で LLM とリアルタイム会話できるのです！
その API である Multimodal Live API があるのですが、これがとても情報量が少ないので、解説していきたいと思います。

Python の SDK しか提供されていないので、JavaScript 大好き nakasyou としては JavaScript で使いたくなったので、コードなどから調べました。

TypeScript でサンプルコードを書いていきます。

## 対象読者

- Multimodal Live API について気になっている人
- Multimodal Live API を使いたいけど情報がなくて困っている人
- Multimodal Live API を JS で使いたい人

かつ
- Gemini API をいじったことがある人

## 概要

### Multimodal Live API ってどんなの

リアルタイムで音声と動画で通信できるやつです。テキストも対応しています。
従来の LLM は、1プロンプトに対して1レスポンスなのはわかりますよね。それに対して、Live Streaming では、通常の会話のようにユーザーとモデルが任意のタイミングでメッセージを送ることができます。
さらに、ユーザーはテキストだけではなく、音声や動画を送ることができます。例えばカメラを知らない花にかざして「この花の名前は？」のように口で聞くことができます。
また、モデルはテキストだけでなく音声で応答できます。「こんにちは」と音声で話すと「こんにちは」と音声で返してくれたりします。

おすすめ記事:
https://zenn.dev/yamato_snow/articles/0aada24da44bd4
https://www.youtube.com/watch?v=9hE5-98ZeCg

### 通信方法

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

### メッセージの分別

勘の良い方はお気づきかもしれませんが、「serverContent」メッセージなら`serverContent`キーがメッセージにあり、その値として内容があるように、キーによってメッセージを分別できます。これはドキュメントに書かれていません。
具体的には、以下のように処理できます:
```ts
function handleMessage (message) {
  if ('setupComplete' in message) {
    // セットアップ完了
  } else if ('serverContent' in message) {
    // モデルからのメッセージ
  } else if ('toolCall' in message) {
    // ツール呼び出し
  } else {
    // ツールキャンセル
  }
}
```

## メッセージの送信

送信メッセージも JSON 形式です。4種類のメッセージがあります。

### setup

セットアップするためのメッセージです。[セットアップ](#セットアップ)で説明したので割愛します。

### clientContent

clientContent は、テキストや画像などの形式を送信するためのメッセージです。通常の Gemini API へのプロンプトと同じです。

型は以下のようになります。単純ですね。
```ts
import type { Content } from '@google/generative-ai' // もともとある型

interface ClientContentMessage {
  clientContent: {
    turns?: Content[] // ユーザーが送信したい　Contents
    turnComplete?: boolean // ユーザーのターンが終了したかどうか
  }
}
```

### realtimeInput

realtimeInput は、動画や音声をリアルタイムで送信するための機能です！こいつが目玉です！
リアルタイムで送信するデータなので、ターンという概念は存在しません。好きなタイミングでモデルの話を遮れます。

みなさん大好き型定義です:
```ts
import type { GenerativeContentBlob } from '@google/generative-ai' // もともとある型

interface RealtimeInputMessage {
  realtimeInput: {
    mediaChunks?: GenerativeContentBlob[]
  }
}
```

この機能には「リアルタイム音声を送信する」と「リアルタイム動画を送信する」の2つがあります。

#### リアルタイム音声

音声は、pcm 形式でエンコードされたサンプルレートが 16000Hz のデータをリアルタイムで送信します。複数回送信できます。モデルの声は 24000Hz でしたが、ユーザーの入力は 16000 Hz です。ここに注意してください。

```ts
ws.send(JSON.strinfify({
  realtimeInput: {
    mediaChunks: [
      {
        type: 'audio/pcm;rate=16000',
        data: base64Pcm // base64 形式でエンコードされた pcm データ
      }
    ]
  }
}))
```
のようにして送信してください。

#### リアルタイム動画

「Gemini 2.0 は動画をリアルタイムで見ることができる」と聞いていると、mov や mp4 などを送信していると想像するかもしれませんが、実際は jpeg を大量に送っているだけです。リアルタイムで更新される jpeg データを送信しているのです。

```ts
ws.send(JSON.strinfify({
  realtimeInput: {
    mediaChunks: [
      {
        type: 'image/jpeg',
        data: base64Jpeg // base64 形式でエンコードされた jpeg データ
      }
    ]
  }
}))
```

のようにして、カメラなどからリアルタイムで送信できます。注意点としては、image/jpeg 以外はサポートされていないところです。

### toolResponse

toolResponse は、Function Calling の結果を送るためのメッセージです。

型定義:
```ts
interface ToolResponseMessage {
  toolResponse: {
    functionResponses?: FunctionResponse[]
  }
}
```

toolCall と組み合わせて、以下のように使用できます:
```ts
const functions = {
  getTime() {
    return Date.now()
  }
}
async function handleToolCallMessage(message: ToolCallMessage) {
  for (const functionCall of message.toolCall.functionCalls) {
    ws.send(JSON.stringify({
      toolResponse: {
        functionResponses: [{
          name: functionCall.name,
          response: functions[functionCall.name](functionCall.args)
        }]
      }
    }))
  }
}
```

## まとめ

これらの内容で、Gemini Multimodal Stream API をいじれるようになったら嬉しいです！

## 宣伝

Multimodal Live API を @google/generative-ai-js に入れるための PR です。
リアクションしていただけると嬉しいです！
https://github.com/google-gemini/generative-ai-js/pull/306


