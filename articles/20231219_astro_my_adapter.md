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

### Bun用Adapterの作成
[Astro公式は多くのAdapter](https://docs.astro.build/ja/guides/integrations-guide/)を作っていて、そのおかげでDenoなどで動かせますが、Bun用のアダプターはありません！
[Bun](https://bun.sh)は、高速なJavaScriptランタイムです。

ないなら作ってしまいましょう！

#### BunのランタイムAPIの理解
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

await foo.arrayBuffer(); // ArrayBuffer取得
```
のような簡単なAPIになっています。

#### 作ってみる！
まず、`create-astro`を使用してAstroプロジェクトを作成します。
```shell
bun create astro
```

次に、
