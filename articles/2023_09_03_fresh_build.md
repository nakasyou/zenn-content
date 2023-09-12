---
title: "Fresh 1.4の新機能！事前ビルドで高速化！" # 記事のタイトル
emoji: "🍋" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["deno", "fresh"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
DenoによるWebフレームワーク、Freshで、事前ビルド機能が追加されたので紹介します。
## 以前のFreshの欠点
以前のFreshでは、islandが含まれるページの初回ロードが非常に遅かったです。
これだと、ユーザー体験を悪くさせる危険性がありました。(私はこれに気づいてからあまりFreshを使用しなくなりました)

この理由は、1ファイルごとにhttpリクエストを送っていたからです。Viteの開発モードのようなイメージです(ローカルでは問題ない)。

Viteは事前ビルドをしていますが、以前のFreshはしていないので非常に低速でした。

Freshが事前ビルドを使用しなかった理由として、
- No buildを実現したかった
- Deno Deployへ高速にデプロイしたかった

などが挙げられると思います。
## 事前ビルドの使い方
事前ビルドは、簡単に使うことができます。
### Freshのアップデート
Fresh 1.4以前に作成されたプロジェクトでは、Freshを最新版にアップデートする必要があります。
```shell
deno task update
```
によりアップデートできます。簡単ですね。
### ビルド
ビルドは、
```shell
deno task build
```
だけでできます。
実行すると、`_fresh`というディレクトリがプロジェクトルートに作成されます。ここにバンドルされたファイルが入ります。

特別な操作は必要とせず、そのまま
```
deno run -A main.ts
```
をすればビルドされたファイルを使い、サーバーが起動します。簡単ですね。
### unstableなAPIに対応
このままだと、`Deno.openKV`などのunstableなAPIに対応していません。ビルド時にエラーを吐きます。

`deno.json(c)`を以下のように編集すれば良いのです。
```diff json:deno.json
  "tasks": {
    "check": "deno fmt --check && deno lint && deno check **/*.ts && deno check **/*.tsx",
    "start": "deno run -A --watch=static/,routes/ dev.ts",
-     "build": "deno run -A  --unstable dev.ts build",
+     "build": "deno run -A  --unstable dev.ts build",
    "preview": "deno run -A main.ts",
    "update": "deno run -A -r https://fresh.deno.dev/update ."
  }
```
## Actionsから使う
ビルドしないFreshでは、そのままDeno Deployにデプロイしていると思いますが、ビルドする場合、GitHub Actionsを通すのが簡単だと思います。
```yaml
name: Deploy
on:
  push:
  pull_request:

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest

    permissions:
      id-token: write
      contents: read

    steps:
      - name: Clone repository
        uses: actions/checkout@v3

      - name: Install Deno
        uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x

      - name: Build step
        run: "deno task build" # ここでビルド

      - name: Upload to Deno Deploy
        uses: denoland/deployctl@v1 # デプロイ
        with:
          project: "my-project" # プロジェクトの名前
          entrypoint: "./main.ts" # 普通にmain.tsで大丈夫
```
これでいけます！
## どんくらい高速化されるん？
[この記事](https://deno.com/blog/fresh-1.4)によると、約40-60倍高速になるそうです。神！
実験動画もあるので見てみてください。
## まとめ
Freshの事前ビルドを使えば、高速化ができます！
ぜひ、使ってみてください！
## 参考
https://deno.com/blog/fresh-1.4
https://fresh.deno.dev/docs/concepts/ahead-of-time-builds
