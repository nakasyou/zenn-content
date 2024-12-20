---
title: "学校向けアプリに Cloudflare Turnstile を使わないで！" # 記事のタイトル
emoji: "🙇" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["cloudflare", "turnstile"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
Cloudflare Turnstile、または Cloudflare の Under Attack Mode を 学校向けアプリに使わない方がいい理由を紹介します。
使ってしまうと、もしかしたら大量のユーザーを失うかもしれません。

## 学校のフィルタリングとの相性

あなたが Cloudflare を用いてサイトを公開するとき、ユーザー A,B,C は以下のようにアクセスします。いたってシンプルですね。
<img width="581" alt="IMG_4242" src="https://github.com/user-attachments/assets/c5bc213c-c26b-400e-9295-b3c352fd0578" />

学校のフィルタリングは、ほとんどの場合、プロキシを用いて一箇所にリクエストをあつめて、そこでブラックリストと照らし合わせてブロックします。
要するに、生徒のリクエストを仲介するのです。図に表すと以下のようになります:

<img width="581" alt="IMG_4243" src="https://github.com/user-attachments/assets/8c181623-156a-4947-a2c1-872ab7442b2f" />

ユーザー A,B,C のリクエストを全て同じプロキシを通じて Cloudflare にリクエストを投げています。つまり、この場合では Cloudflare からは 1 人で 3 人分のリクエストを送っているように見えるのです。

実際のフィルタリングシステムは、大きな会社が担当しています。そのため、日本全国の学校を顧客にしていることが多いです。これによって、以下のようなことが起こります:
<img width="1032" alt="IMG_4244" src="https://github.com/user-attachments/assets/17a6ddb3-4003-4809-a317-a93fc4bbbfb1" />

1IP アドレスで無数のアカウントを用いているように勘違いをして、Cloudflare は BOT または悪質なハッカーだと認識します。

Cloudflare Turnslite や Under Attack Mode は怪しい IP アドレスをブロックするので、そのフィルタリングサービスを使っている生徒は Cloudflare Turnstile を使うサイトにアクセスできなくなってしまうのです。

## 実例

