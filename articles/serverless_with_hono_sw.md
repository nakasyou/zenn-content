---
title: "Hono + Service Worker で、「サーバーレス」" # 記事のタイトル
emoji: "🌫️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["hono", "service-worker"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
Hono と Service Worker で、「サーバーレス」に Web アプリを作る記事です。

## 注意点

通常 SaaS/FaaS タイプのサーバーレス度が 10% のところ、この方法はサーバーレス度を 50% にしているだけです。真のサーバーレスは難しいです。

## どうやるの

Service Worker を用いて、ローカルファースト、サーバーレスの Web アプリを作ります。
GitHub Pages などの静的サイトホスティングで動かすことが目標です。

Service Worker はリクエスト内容を変更することができるので、それを使います。

## 下準備
`404.html`を作りましょう。Service Worker を登録します。
```html:404.html
<!doctype HTML>
<html>
  <head>
    <meta charset="UTF-8" />
  </head>
  <body>
    <div>Loading...</div>
    <script>
      if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register('/sw.js').then(() => location.reload()).catch(() => alert('エラーが発生しました'))
      } else {
        alert('Service Worker を有効にしてください')
      }
    </script>
  </body>
</html>
```

次に、Hono を用いたコードを書いてみます。
```ts:sw.ts
declare const self: ServiceWorkerGlobalScope

import { Hono } from 'hono'
import { handle } from 'hono/service-worker'

const app = new Hono().basePath('/sw')
app.get('/', (c) => c.html('Hello World'))

self.addEventListener('fetch', handle(app))
```

esbuild で バンドルしましょう。
```shell
esbuild sw.ts --bundle --outfile=sw.js
```

適当なサーバーを開いてアクセスしてみると、Hono アプリとして動きます。ブラウザで動いているのです。
なぜこんなことができるかというと、Hono は Web 標準の上に構成されていて、コアはブラウザ上で動いてしまうからです。

