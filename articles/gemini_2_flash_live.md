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
例えば、Gemini の説明を聞いているときに、「やっぱり説明をやめて」みたいなことができるわけです。これには双方向通信をしなければならないので、 WebSocket を使用しているというわけです

## API の使い方

TypeScript でサンプルコードを書いていきます。

### 接続

WebSocket に接続するコードです:
```ts
const ws = new WebSocket('wss://generativelanguage.googleapis.com/ws/google.ai.generativelanguage.v1alpha.GenerativeService.BidiGenerateContent?key=apiKey')
```
ポイントとしては、
- エンドポイントは `wss://generativelanguage.googleapis.com/ws/google.ai.generativelanguage.v1alpha.GenerativeService.BidiGenerateContent`
- API Key は `key` パラメーターで指定する

といったところです。

### セットアップ

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
export interface Setup {
  // models/gemini-2.0-flash-exp みたいに、必須プロパティ
  model: `models/${string}`

  // 普通とちょっと違う GenerationConfig です。
  generationConfig?: LiveGenerationConfig

  // システムプロンプト。
  systemInstruction?: Content

  // ツール。Function Calling とか使える
  tools?: Tool[]
}

export interface LiveGenerationConfig extends GenerationConfig {
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

 export interface LiveSpeechConfig {
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

### 
