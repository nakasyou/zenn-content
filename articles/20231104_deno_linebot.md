---
publication_name: liberluna
title: "DenoでLINE Botを作っちゃう" # 記事のタイトル
emoji: "💬" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["deno", "deno deploy", "line", "hono"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
こんにちは！Liberlunaのnakasyouです！

LiberlunaのLINE公式アカウントは、Denoで動いています。DenoでLINE公式アカウントを作る方法を紹介します！

## 対象読者
- Denoの基本的な使い方がわかっている人
- LINE BotをDenoで作ってみたい人
- Deno Deployの基本的な使い方がわかっている人

## 事前準備
### Denoのインストール
Denoがインストールされていない場合は、インストールしましょう。

https://docs.deno.com/runtime/manual/getting_started/installation
に方法が載っています。

### LINE Developer Consoleへのログイン
https://developers.line.biz/ja/
で、ログインまたは登録をしましょう。公式アカウントを作るときに使います。

## バックエンドを書く
### サーバーを作成する
LINE Botは、WebHookサーバーを使って作ることができます。そのバックエンドを書きましょう！
今回は、[Hono](https://hono.dev)というhttpサーバーを作成するライブラリを使います。

`main.ts`を以下のようにします。
```ts
import { Hono } from 'hono'

// アプリケーションの作成
const app = new Hono()

// `/webhook`にPOSTされたとき
app.post('/webbook', c => {
  // `Hello WebHook!'とテキストで返す
  return c.text('Hello WebHook!')
})

// サーバーを起動
Deno.serve(app.fetch)
```
こんな感じです。次に、`deno.json`を作成します。
```json
{
  "imports": {
    "hono": "https://deno.land/x/hono@v3.9.2/mod.ts"
  },
  "tasks": {
    "dev": "deno run --allow-net --watch main.ts",
    "start": "deno run --allow-net main.ts"
  }
}
```
`"imports"`を使い、簡単にHonoをインポートできるようにしています。バージョンの`v3.9.2`は、最新版に置き換えるなどしてください。

開発サーバーを起動しましょう。
```shell
deno task dev
```

URLが表示されるはずなので、`/webhook`にcURLしてみましょう。
![ss](https://github.com/nakasyou/zenn-content/assets/79000684/6baa7686-e7f6-4f36-9c3c-14c70796d717)
しっかりと、`Hello WebHook!`と返ってくることが確認できるはずです。

## LINEに対応する
次に、LINEのWebHookに対応させます。
`/webhook`にアクセスされたときの挙動を変更します。

```ts
app.post('/webhook', async c => {
  const data = await c.req.json() // WebHookデータ

  const replys: Promise<Response>[] = []
  for (const event of data.events) {
    // イベントでループ
    if (event.type !== 'message') return // メッセージでないイベントは無視

    const { message, replyToken } = event

    if (message.type !== 'text') return // テキストメッセージでないイベントは無視

    const textMessage: string = message.text // ユーザーの発言を取得

    const replyData = {
      replyToken,
      messages: [{
        type: "text",
        text: `あなたはさっき、${textMessage}と言った！`
      }],
    } // リプライするデータを作成
    replys.push(fetch("https://api.line.me/v2/bot/message/reply", {
      method: "POST",
      headers: {
        "Content-type": "application/json",
        "Authorization": "Bearer " + Deno.env.get("LINE_TOKEN"),
      },
      "body": JSON.stringify(replyData),
    })) // リプライ
  }
  await Promise.all(replys) // 全てのリプライ完了を待つ

  return c.text('OK')
})
```
「あ」と言われたら、「あなたはさっき、あと言った！」と返信してくるようにしました。
### Deno Deployにあげる
Deno Deployでデプロイします。
「Deno Deployの基本的な使い方がわかる」が、対象読者に含まれているので、詳細なやり方は割愛します。

## LINE公式アカウントの作成
公式アカウントを作っていきます。
まず、[LINE Developer Console](https://developers.line.biz/console)を開きます。

次に、ない場合はプロバイダを作ります。
![](https://github.com/nakasyou/zenn-content/assets/79000684/7caefc6e-fdd9-4c73-8796-ec9d4f2f7f64)
↑「作成」を押す

プロバイダの「チャネル設定」から「新規チャネル作成」、「Messaging API」をクリック
![](https://github.com/nakasyou/zenn-content/assets/79000684/a758f5de-e68e-41b2-812c-081434dd77f7)

必要事項を入力し、「作成」です。

すると、そのチャネルの設定画面が表示されます。

WebHook URLに、Deno DeployのWebHook URLを入力します。例えば、プロジェクト名がexampleの場合、`https://example.deno.dev/webhook`です。
![](https://github.com/nakasyou/zenn-content/assets/79000684/ccc0e1f5-38e1-4b06-ba85-81aba881b59d)
すると下に「WebHookを有効」というスイッチが出てくるので、オンにします。
「検証」を押すと、そのURLが正しいかが検証されます。

### アクセストークンの発行
「チャンネルアクセストークン(長期)」を発行して、コピーします。
![](https://github.com/nakasyou/zenn-content/assets/79000684/49416242-bc32-42c5-9215-eb16ec147621)

それを、Deno Deployのプロジェクトの環境変数に入れましょう！
![](https://github.com/nakasyou/zenn-content/assets/79000684/b699f89d-59ec-4918-b92b-5aa929984586)

これで完璧です！あとは、「Messaging API 設定」からQRコードを読み込みましょう！
![](https://github.com/nakasyou/zenn-content/assets/79000684/7c455c81-0cc5-4d1b-9f22-3c078cb11439)

## 試す！
対話してみます！すると、しっかりプログラムの通りになります！
![1699067525194](https://github.com/nakasyou/zenn-content/assets/79000684/91e801ae-e37a-44b3-89d9-e0ccfde9abba)

## 最後に
ソースコードは
https://github.com/nakasyou/deno-line-bot-example
にあります。

Deno Deployを用いて、簡単にLINE Botを作れます！
プログラムを変えれば、条件によって応答を変えたり、天気を表示したり... 活用の幅が広がります。
皆さんもぜひ、試してみたください！


