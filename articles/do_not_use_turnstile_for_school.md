---
title: "Cloudflare Turnstile を使うと小中学生ユーザーを失う？" # 記事のタイトル
emoji: "🙇" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["cloudflare", "turnstile"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
Cloudflare Turnstile (Cloudflare の CAPTHCA のようなもの)、または Cloudflare の [Under Attack Mode](https://developers.cloudflare.com/fundamentals/reference/under-attack-mode/) を 学校向けアプリに使わない方がいい理由を紹介します。
使ってしまうと、もしかしたら大量のユーザーを失うかもしれません。

## 学校のフィルタリングとの相性

あなたが Cloudflare を用いてサイトを公開するとき、ユーザー A,B,C は以下のようにアクセスします。いたってシンプルですね。
![IMG_4242](https://github.com/user-attachments/assets/c5bc213c-c26b-400e-9295-b3c352fd0578)

学校のフィルタリングは、ほとんどの場合、プロキシを用いて一箇所にリクエストをあつめて、そこでブラックリストと照らし合わせてブロックします。
要するに、生徒のリクエストを仲介するのです。図に表すと以下のようになります:

![IMG_4243](https://github.com/user-attachments/assets/8c181623-156a-4947-a2c1-872ab7442b2f)

ユーザー A,B,C のリクエストを全て同じプロキシを通じて Cloudflare にリクエストを投げています。つまり、この場合では Cloudflare からは 1 人で 3 人分のリクエストを送っているように見えるのです。

実際のフィルタリングシステムは、大きな会社が担当しています。そのため、日本全国の学校を顧客にしていることが多いです。これによって、以下のようなことが起こります:
![IMG_4244](https://github.com/user-attachments/assets/17a6ddb3-4003-4809-a317-a93fc4bbbfb1)

1 つの IP アドレスで無数のアカウントを用いているように勘違いをして、Cloudflare は BOT または悪質なハッカーだと認識します。

Cloudflare Turnslite や Under Attack Mode は怪しい IP アドレスをブロックするので、そのフィルタリングサービスを使っている生徒は Cloudflare Turnstile を使うサイトにアクセスできなくなってしまうのです。

## 実例

Turnstile を試すと、こんな感じになります:

### Quizlet

Quizlet は、 Cloudflare を用いたオンライン暗記サービス/サイト/アプリです。 Web ブラウザを用いてアクセスすることができます。
私の学校では英語の毎授業でこれに取り組む時間がありましたが、最近は Cloudflare Turnstile のせいでアクセスできなくなっています。そのため、Quizlet の使用がなくなりました。
Quizlet は広告収入が 1 つの収益源なので、ユーザーが命なはずです。日本中に Quizlet にアクセスできなくなった学校があるはずで、その影響は少ないながらに存在するはずです。

### その他アクセスできなくなったサービス

 - Canva
 - ChatGPT
 - ChatBot Arena

## サイト作成者は何をすべきか

もしあなたが学校でつかったり、教育のために使ったりすることを目的とした Web サービスを作成していて、このことによるユーザー減少に対策したいのであれば:
- Cloudflare Turnstile をやめて、代わりに [reCAPTCHA](https://www.google.com/recaptcha/about/) や [hCAPTCHA](https://www.hcaptcha.com) にすることを検討してみてください。
- また、Cloudflare の [Under Attack Mode](https://developers.cloudflare.com/fundamentals/reference/under-attack-mode/) の使用を控えることを検討してください。

## Cloudflare へ思うこと

:::message alert
この節は私の個人的な感想です。
:::

まじいろんなサイト使えないの困るから！！！日本の学校のフィルタリング事情を考慮してほしい！！！！！！！！！！！！！！！！！！！！！！！
とても困ってます！
