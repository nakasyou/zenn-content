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

## Adapterの仕組み
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

### Adapterの構造
Adapterの構造、仕組みを解説します。

Astroは、ランタイムごとのAPIに依存しないコードをビルド時に生成します。`Deno.xxx`などを使わない、ECMAScriptなコードです。
このコードの役割の一つとして、`manifest`よ呼ばれるオブジェクトを生成するというものがあります。

型情報は以下の`SSRManifest`のようです。

https://github.com/withastro/astro/blob/7ae4928f303720d3b2f611474fc08d3b96c2e4af/packages/astro/src/core/app/types.ta

このオブジェクトには、画像などの外部アセットデータ以外の情報が詰まっています。
このオブジェクトを利用して、各ランタイム用にAdapterを書けるわけです。
