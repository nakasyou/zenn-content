---
title: "中学生がLLMの選定プラットフォーム作ったので使って: LMSpecs" # 記事のタイトル
emoji: "🤖" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["llm", "deno", "vite", "solid.js"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
今日で中学生を名乗れるのが最後なので、名乗りたいと思います。

LLM を活用したサービスを使ったり作ったりするとき、どの LLM を使うか迷ったことはありませんか？
選定するには、各モデルの比較が必要です。その比較をするプラットフォームを作成したので、紹介します。

https://github.com/nakasyou/lmspecs

## LLM の情報収集、大変じゃない？

コンテキストウィンドウや速度など、各 LLM の特徴を示すもの、いくつ思い浮かびますか？
公開日、料金、ベンチマークでわかる性能、最大出力長、マルチモーダルかどうか、Reasoning できるかどうか... など、大量にありますよね。それらを総合的に比較してモデルを選んでいくわけですが、そもそもそれらの情報収集が大変です。

例えば、Gemini 2.0 Flash と Command A を比較したいとします。そのための情報源をまとめました:
| title | Gemini 2.0 Flash | Command A |
| --- | --- | --- |
| Context Window | Docs ([検索結果の一番上](https://ai.google.dev/gemini-api/docs/long-context?hl=ja)は 100 万なのに [Docs](https://ai.google.dev/gemini-api/docs/models?hl=ja#gemini-2.0-flash) では 1,048,576) | Docs (「Cohere Command A context window」で出てこない ) |
| 料金 | [Pricing ページ](https://ai.google.dev/gemini-api/docs/pricing?hl=ja) | [Docs 外の Pricing ページ](https://cohere.com/pricing) |

各社のモデル情報提供形式が異なり、さらに各社の中でも情報が分散しています。そのため、とてもわかりにくいです。データを一元管理することでこの問題を解決するのが LMSpecs です。

## LMSpecs as a database

LMSpecs では、各モデルの情報が GitHub 上の JSON で管理されています。これらの JSON は LMSpecs 中のフォーマットに統一されていて、プログラムで処理することも容易です。
これを用いて、モデルを比較するための Excel スプレッドシートを作ることだってできます。

![image](https://github.com/user-attachments/assets/0818f72b-a09a-4db7-9aa2-e4d0631e49ef)
![image](https://github.com/user-attachments/assets/b639b3eb-f166-4c90-bc17-7f795b06b1b8)

## LMSpecs as a viewer

JSON を人間が閲覧するのは不便です。そのため、Web ビューワーもついています。

https://lmspecs.pages.dev/

### なぜコスト対知性がLLMの評価に使われないのだろうか

https://x.com/goisaki/status/1895392596916400611

単一のベンチマークを使って、賢いモデルを選択するのは本当に賢明ですか? 実際は、予算と照らし合わせる必要があると思います。
LMSpecs なら、コスト対知性のようなグラフを描画してダウンロードできます。このグラフは任意の縦軸、横軸、モデルを選択できます。
![image](https://github.com/user-attachments/assets/a020c780-feae-43bd-8048-fef50a17cbf8)

### モデルの情報を閲覧する

![image](https://github.com/user-attachments/assets/5c694ff8-edc3-4868-b864-756bc68ffe0d)

いろいろなところからモデルの情報を収集する手間を省くことができます。

また、モデルを横並びで比較することも可能です。
![image](https://github.com/user-attachments/assets/912fec73-8536-4215-8e7c-43db46a6e8a2)

## 技術解説

[Deno](https://deno.com/) をパッケージマネージャーとしています。モデル情報は JSON で保存されていて、JSON Schema は Valibot という JavaScript Validator から動的生成されます。
Web ビューワーは Vite および Solid.js を使用した SSG によって構成されています。
Vite には `import.meta.glob` という関数が搭載されています。これは Glob を用いて動的 import が可能な関数です。
```ts
const models = import.meta.glob('../models/*/meta.json')
```
のようにモデルの一覧を取得することが可能で、これはビルド時にバンドルされるので SSG でモデルをリアルタイムアプリケーションのように閲覧できます。

## よくありそうな質問

### 手作業?3か月で更新止まりそうじゃない?

多分。しかし、MCP による Web 閲覧搭載 Cline の助けを借りられるので、更新コストは低いです。また、スターしていただけるとモチベが上がって更新できます。
Chatbot Arena のようなリアルタイムなベンチマークは自動で更新されるので、ベンチマークの更新の心配はありません。

### ベンチマークはどのくらい搭載している?

Chatbot Arena、MMLU-Pro、Humanity’s Last Exam の 3 つのみです。現在 MMMU-Pro の追加作業をしていて、今後もっと追加する予定です。
