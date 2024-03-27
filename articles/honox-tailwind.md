---
title: "HonoXでTailwind" # 記事のタイトル
emoji: "🔥" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["tailwind", "hono", "honox"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
[HonoX](https://github.com/honojs/honox)というフレームワークでTailwindCSSを使う方法です。

デフォルトでは、`hono/css`というものを使っていますが、Tailwindの人もいるだろうということです。

## 結論
依存関係インストール
```shell
bun add -D tailwindcss postcss autoprefixer -p
```
Tailwindを初期化
```shell
bunx tailwindcss init -p
```

そして、`tailwind.config.js`を以下のようにします:
```diff js:tailwind.config.js
  /** @type {import('tailwindcss').Config} */
  export default {
+   content: ['./app/**/*.tsx'],
    theme: {
      extend: {},
    },
    plugins: [],
  }
```

`app/tailwind.css`を作成して、以下のようにします
```css:app/tailwind.css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

このファイルを`./app/client.ts`でimportします
```diff js:tailwind.config.js
  import { createClient } from 'honox/client'
+ import './tailwind.css'

  createClient()

```

これれ完成です！

## 仕組み
```shell
bun add -D tailwindcss postcss autoprefixer -p
bunx tailwindcss init -p
```
この2つで、TailwindCSSをインストールして、設定ファイルを追加します。
このままだとTailwindのクラス名が認識されないので、`content: ['./app/**/*.tsx']`と書き、`app`ディレクトリ内の`.tsx`ファイルを読み込ませるようにします。

それでも、スタイルは反映されないので、PostCSSを使ったTailwindのスタイルファイル`./app/tailwind.css`を作ります。

しかしそれでも反映はされません、スタイルを読み込ませるようにする必要があります。
HonoXの全てのHTMLに反映される(であろう)JavaScriptエントリーポイント`./app/client.ts`にimport処理を入れたら完成です。
