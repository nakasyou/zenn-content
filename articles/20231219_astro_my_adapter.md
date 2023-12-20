---
publication_name: trans
title: "Astro で自分だけのアダプターを作る" # 記事のタイトル
emoji: "🚀" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["astro", "bun"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
AstroというWebフレームワークは知っていますか？

Astroは、SSR/SSGができて、NodeはCloudFlare Workers、Denoなどの好きなランタイムで動かせて、ReactやVueなど好きなUIフレームワークを使えるWebフレームワークです。

その「好きなランタイムで動かせる」を実現しているのが、AstroのAdapterです。
この記事では、AstroのAdapterの仕組みを調べて、Adapterを作っていきます。

## Adapterについて詳しく
Adapterは、前述した通り、Astroのサーバーをいろいろなランタイムで動かすためのものです。
言い換えれば、Astroとランタイムの間に入る「クッション」のような存在です。

### Adapterの必要性
ランタイムによってある行動をしたい場合のコードやAPIが異なります。
例えば、サーバーを起動する処理の場合、Denoは`Deno.serve`、Nodeは`http`モジュールなど、Cloudflare Workersは`addEventListener('fetch')`というふうに、ランタイムごとに違います。
そのため、単純に同じプログラムだと異なるランタイムで動かせないことがあるのです。
![Image](https://github.com/nakasyou/zenn-content/assets/79000684/20b01868-cc49-4d40-8581-5ff5b72ddeb1)

そこで、Adapterの登場です。
Adapterは、下の図のようにランタイムの差を吸収してくれるのです。
![Image](https://github.com/nakasyou/zenn-content/assets/79000684/dd959393-4af9-4a9d-ae27-39514298a9be)

これがAstroのAdapterの役割です。

### Adapter
Adapterの構造を解説します。

Astroは、ランタイムごとのAPIに依存しないコードをビルド時に生成します。`Deno.xxx`などを使わない、ECMAScriptなコードです。
このコードの役割の一つとして、`manifest`よ呼ばれるオブジェクトを生成するというものがあります。

型情報は以下の`SSRManifest`のようです。

https://github.com/withastro/astro/blob/7ae4928f303720d3b2f611474fc08d3b96c2e4af/packages/astro/src/core/app/types.ta

manifestには、画像などの外部アセットデータ以外の情報が詰まっています。
manifestを利用して、各ランタイム用にAdapterを書けるわけです。

実際の`manifest`は以下のようになっています。
![1](https://github.com/nakasyou/zenn-content/assets/79000684/a2ddbbd5-c45a-464b-9eaa-176ec1a661c2)
![2](https://github.com/nakasyou/zenn-content/assets/79000684/309333ea-8135-4d97-b9dd-42b0cacaa8ef)
かなり複雑な構造をしていますね。

## Adapterを作ってみる
早速、Adapterを作ってみましょう！

## Bun用Adapterの作成
[Astro公式は多くのAdapter](https://docs.astro.build/ja/guides/integrations-guide/)を作っていて、そのおかげでDenoなどで動かせますが、Bun用のアダプターはありません！
[Bun](https://bun.sh)は、高速なJavaScriptランタイムです。

ないなら作ってしまいましょう！

### BunのランタイムAPIの理解
AdapterはAstroとランタイムAPIを繋げるクッションなので、まずBunのランタイムAPIを簡単に理解しましょう！

まず、Bunのサーバー起動は次のようになっています。
```ts
Bun.serve({
  fetch: async (req: Request): Promise<Response> => {
    return new Response(`Hello, ${new URL(req.url).pathname}`)
  }
})
```
これは、アクセスすると、`Hello, <pathname>`と返ってくるコードです。
`Bun.serve`に`Request`を受け取り`Response`(PromiseでもOK)を返す`fetch`関数を渡す簡単なAPIです。

また、Bunでのファイル読み込みは
```ts
const foo = Bun.file('foo.txt')

foo.size // サイズ
foo.type // MIME Type

await foo.arrayBuffer() // ArrayBuffer取得
```
のような簡単なAPIになっています。

### プロジェクトを作る
まず、`create-astro`を使用してAstroプロジェクトを作成します。
```shell
bun create astro
```

次に、`astro.config.mjs`をサーバーモードに対応させましょう。
```diff ts:astro.config.mjs
  import { defineConfig } from 'astro/config';

  // https://astro.build/config
  export default defineConfig({
+   output: 'server'
  });
```

### Adapterをつくる
次に、Adapterを作りましょう！

#### Integration
`src/integrations/bun-adapter/index.ts`とでもするファイルを作ってください。

```ts:bun-adapter/index.ts
import type { AstroIntegration } from 'astro'

export const bunAdapter = (): AstroIntegration => {
  return {
    name: 'bun-adapter',
    hooks: {
      'astro:config:done': ({ setAdapter }) => {
         setAdapter({
           name: 'bun-adapter',
           serverEntrypoint: './src/integrations/bun-adapter/index.ts',
           supportedAstroFeatures: { staticOutput: 'stable' }
         })
      }
    }
  }
}
```
解説します。このコードは、Astro Integrationを作成する関数です。Integrationは、Astroのプラグインシステムのようなものです。

`name`は、Integrationの名前、`hooks`は、Astroが呼ぶイベントを代入します。
`astro:config:done`は、configがdoneしたときに呼ばれると思います(そのまま)

このイベントのハンドラーは、`setAdapter`という関数をAstroから受け取り、それを呼び出しAdapterを登録しています。

`name`にはAdapterの名前、`serverEntrypoint`はAdapter本体のパス、`supportedAstroFeatures`はAdapterが対応している機能が代入されます。

#### Integrationの登録
次に、このIntegrationをAstroに認識させます。
```diff ts:astro.config.mjs
  import { defineConfig } from 'astro/config';
+ import { bunAdapter } from './src/integrations/bun-adapter`

  // https://astro.build/config
  export default defineConfig({
    output: 'server',
+   adapter: bunAdapter()
  });
```
完璧です。importして受け渡してるだけです。

#### Adapter本体
では、`serverEntrypoint`であるAdapter本体の`src/integrations/bun-adapter/index.ts`を書きましょう。

```ts:bun-adapter/server.ts
import type { SSRManifest } from 'astro'

export function start (manifest: SSRManifest) {
  // サーバ起動時のコードを書いていく...
}
```
出ました前述の`manifest`です。`manifest`を受け取る`start`関数をエクスポートしています。

この状態で
```ts:bun-adapter/server.ts
import type { SSRManifest } from 'astro'

export function start (manifest: SSRManifest) {
  console.log('Hello world!!')
}
```
とやって`astro build`(`bun run build`)でビルドします。
すると、`dist`ディレクトリが作成されます。

`dist/server/entry.mjs`はサーバー起動のエントリーポイントのコードなので、これを実行してみてください。
```shell
bun ./dist/server/entry.mjs
```
![IMG_2301](https://github.com/nakasyou/zenn-content/assets/79000684/83206782-680c-4089-9da6-d248829c74c9)
こんな感じになるはずです。そしたら成功です。

「あれ？サーバー起動しないの？」って思う方もいらっしゃると思いますが、Adapterにはサーバー起動のコードが含まれていないのでそうなります。

:::message
Adapterのコードを編集して実行するときは、毎回ビルドしなければいけません。
:::

では、実際にサーバーを起動するコードを書きましょう。

`manifest`から`Bun.serve`に繋げるのです。

しかし、`manifest`は前述の通り非常に複雑です。なので、`astro/app`というモジュールが存在します。
```ts
import { App } from 'astro/app'

const app = new App(manifest)
```
のようにして初期化させます。
このAppというのが便利なのです。

Appは、`manifest`をRequest/Responseのインターフェースにしてくれるのです。

```ts:bun-adapter/server.ts
import type { SSRManifest } from 'astro'
import { App } from 'astro/app'

export function start (manifest: SSRManifest) {
  const app = new App(manifest)

  app.render(new Request('https://example.com/')).then(console.log)
}
```
のように`app.render`を使うと、Responseが出力されるはずです。


では、これをBun.serveで使います。
```ts:bun-adapter/server.ts
import type { SSRManifest } from 'astro'
import { App } from 'astro/app'

export function start (manifest: SSRManifest) {
  const app = new App(manifest)

  Bun.serve({
    fetch: req => app.render(req)
  })
  console.log('Started server: http://localhost:3000')
}
```
サーバーも動いてるしこれで完成！
と思いきや、これだとしっかり動きません。

その証拠の一つとして、faviconが表示されないはずです。(されているとしたらキャッシュの影響)
静的Assetsをサポートしていないためです。

manifestの説明に書いた通り、manifestには外部Assetsが含まれていません。

そのため、静的ファイルにはアクセスできないのです。これを解決するために、静的ファイルについてのコードを書きます。
`manifest.assets`には静的ファイルのパスのSetが含まれます。
これを用いて静的ファイルを配信します。

```ts:bun-adapter/server.ts
import type { SSRManifest } from 'astro'
import { App } from 'astro/app'
import path from 'node:path'

export function start(manifest: SSRManifest) {
  const app = new App(manifest)

  const getAssetFuncs: Record<string, (() => Promise<Response>) | undefined> = {} // pathnameからファイルデータを出す関数たち

  for (const asset of manifest.assets) {
    const filePath = path.join(
      import.meta.dir,
      '../client',
      asset.replace(/^\//, '') // `/favicon.svg`を`favicon.svg`に
    ) // ファイルパスを求める

    getAssetFuncs[asset] = async () => {
      const file = Bun.file(filePath)

      return new Response(file, {
        headers: {
          "Content-Type": file.type // Mime Typeを指定
        }
      })
    }
  }
  Bun.serve({
    fetch: async req => {
      const pathname = new URL(req.url).pathname // pathnameを取得
      const getAsset = getAssetFuncs[pathname]
      if (getAsset) {
        // ファイルがある
        return getAsset()
      }
      return await app.render(req)
    },
    port: 3001
  })
  console.log('Started server: http://localhost:3000')
}
```
こんな感じにしたら大体完成です！！！
しっかりfaviconもあるはずです。

## Hono用Adapterを作る
次に、Hono用Adapterを作ってみましょう。
Honoは、いろんなランタイムで動く高速なWebフレームワークです。

https://hono.dev/

目標はこんな感じです。
```ts
import { Hono } from 'hono'
import { astroApp } from '../dist/server/entry.mjs'

const app = new Hono()

app.use('/*', astroApp())

Bun.serve(app)
```

実現するためには、astroAppという関数の**export**が必要なのですが、これは実現できるのでしょうか？
できます。Bunのアダプタでは`start`をつかっていましたが、今回は`createExports`を使います。
