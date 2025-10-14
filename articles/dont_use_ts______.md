---
title: "思想バランス: TypeScript をバックエンドに使うな" # 記事のタイトル
emoji: "🧩" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["typescript", "contest2025ts"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

nakasyou は TypeScript が好きです。

https://x.com/nakasyou0/status/1926457184516288681

なんとなく呟いたら燃えたので思想補正しましょう。

## 浮動小数点演算が信頼できない

これは JavaScript でも同じですが、TypeScript の
