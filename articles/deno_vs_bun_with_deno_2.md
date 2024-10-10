---
title: "Deno vs Bun - Deno 2.0 において" # 記事のタイトル
emoji: "📦" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["deno", "bun"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

Deno 2.0 が昨日リリースされた。Deno 1.x から多くの変更点が存在する。Deno 1.x と Bun の比較記事は多く存在するが、メジャーバージョンアップがなされたので改めて Deno 2.0 と Bun を評価したい。

## Node.js 互換性

Deno 2 を紹介する Deno Blog において、
> Deno 2 is backwards compatible with Node and npm.

と記述している。これが気になった。Node.js で動くなら Deno で動くのか。Node.js の互換性について試してみる。

| name | Deno 2 | Deno 1.46 | Bun |
| --- | --- | --- | --- |
| Playwright | ❌ | ❌ | ✅ |

:::details 使用したコード
#### Playwright
```js
import { firefox } from 'playwright'
import { writeFile } from 'node:fs/promises'

const browser = await firefox.launch({ headless: true })
const page = await browser.newPage()
await page.goto('https://example.com')
```
:::
