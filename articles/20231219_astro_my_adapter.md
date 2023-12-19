---
publication_name: Dev-TRANs
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

