---
title: "You don't need Node.js" # 記事のタイトル
emoji: "🌪️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["javascript", "node.js", "deno", "bun"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
Node.jsはいらない場合がある、むしろいらない場合の方が多いかもしれない、という記事です。

## Post Node.js ランタイムの登場
Node.js のあとにできたランタイムがいくつも登場しています。
- [Deno](https://deno.com)
- [Bun](https://bun.sh)
- [WinterJS](https://wasmer.io/posts/winterjs-v1)
- [LLRT](https://github.com/awslabs/llrt)

この中でも、人気である**Deno**と**Bun**を中心に考えていきます。

## DenoやBunに変えるメリット
これがなければNode.jsから変える必要はないと思います。

私は、以下の2つが、2ランタイムに共通して言える大きなメリットだと思います:
- **ネイティブ**TypeScriptサポート
- 高速

### ネイティブTypeScriptサポート
現在、JavaScriptを記述するときは、
