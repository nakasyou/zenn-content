---
title: "ViteのHMRの仕組み？？" # 記事のタイトル
emoji: "🐣" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["vite", "hmr"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
HMR を実装する機会があったので、その時の知見をもとに Vite などの HMR の仕組みを共有します。

## HMRとは？

HMR とは、 Web 開発において、エディタ上でした変更を即座にブラウザーに反映させる仕組みのことです。

