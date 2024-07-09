---
title: "先生! Cloudflare は信用できます!" # 記事のタイトル
emoji: "🧑‍🏫" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["cloudflare", "cloudflareworkers", "cloudflarepages"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
Cloudflare Pages 、使ってますか?超便利ですよね。
文化祭などでも、生徒がサイトを作るときに使用している場合があるそうです。

そんなときに問題が起こるかもしれません。「その Cloudflare って会社、信用できるの?? 」
根拠を集めましょう。

## どのような根拠?
顧客です。顧客が結構信用させるのにいいのではないでしょうか。
先生方が信用している団体が多く使用しているサービスだったなら、信用度が上がるはずです。
特に公共性の高い企業や団体は有効なはずです。Pages の使用に一歩近づけます。

## 大手日本企業を使う
私が国内の大手企業などの顧客を調べました:

* GMO[^gmo]
* 日本航空(JAL)[^jal]
* 早稲田大学(企業でない)[^waseda]
* ライオン株式会社[^lion]

GMO は知らないにしても、日本航空はわかるのではないでしょうか。
日本企業は強いカードかもしれません。

[^gmo]: https://www.cloudflare.com/ja-jp/case-studies/gmo-internet-group/
[^jal]: https://www.cloudflare.com/ja-jp/case-studies/jal-internet-group/
[^waseda]: https://www.cloudflare.com/ja-jp/case-studies/waseda-university/
[^lion]: https://www.mki.co.jp/news/casestudy/20220215_1.html

## Web アプリを使う
先生方にも使っていらっしゃる人はいるのではないでしょうか。

* Discord
* Canva
* Udemy

### ChatGPT
「今話題の ChatGPT でも使われているんですよ。
ChatGPT の開発元の OpenAI という会社があり、その 49% の資本を Microsoft がもっているんです!
`${学校で使ってる Microsoft 製品}`」

## 公共性の高い事業を使う

### Cloudflare Waiting Room と コロナワクチン
全国53自治体(人口規模871万人)のコロナウイルスのワクチン受付に「Cloudflare Waiting Room」というシステムが使われているようです。[^cf_waiting_room]

[@kombumori](https://zenn.dev/kombumori)さんからの[情報](https://zenn.dev/link/comments/29c99388caa4cb)です。ありがとうございます🙇‍♀️

[^cf_waiting_room]: https://classmethod.jp/news/202106-cloudflare/
## 統計を使う
W3Techs によれば、2024年7月現在、すべての Web サイトの 19.1 % が Cloudflare を使っているようです。[^w3techs]

これは驚きですね。ウェブサイトの2割が Cloudflare を使っているというのはパワーワードです。
信用度上がっちゃうかもしれません。

[^w3techs]: https://w3techs.com/technologies/details/cn-cloudflare

## 自治体を使う
これは強いカードです。自治体の Web サイトに Cloudflare が使われてたらいいのではないでしょうか、と思い収集しました。

[このサイト](https://uub.jp/opm/ml_homepage.html)が自治体の Web サイトを集めているので、それを使って確かめます。
DevTools で表からすべての URL を抽出して、 Deno で `cf-` ヘッダーが含まれているものを抽出しました。
参考にどうぞ、少なくとも 57 の自治体の Web サイトが Cloudflare を使っています：
:::details 一覧表
```
http://www.city.higashiomi.shiga.jp/
https://www.city.kusatsu.shiga.jp/
https://www.city.ritto.lg.jp/
http://www.town.kunitomi.miyazaki.jp/
http://www.town.hinokage.lg.jp/
https://www.city.fukuchiyama.lg.jp/
https://www.town.toyosato.shiga.jp/
https://www.town.ryuoh.shiga.jp/
https://www.city.maibara.lg.jp/
https://www.city.joyo.kyoto.jp/
https://www.town.aisho.shiga.jp/
https://www.city.yasu.lg.jp/
https://www.hyugacity.jp/
https://www.vill.nishimera.lg.jp/
https://www.vill.shiiba.miyazaki.jp/
https://www.town.takanabe.lg.jp/
https://www.himeshima.jp/
https://www.city.nantan.kyoto.jp/www/
https://www.city.kushima.lg.jp/
https://www.city.kyoto.lg.jp/
https://www.city.ebino.lg.jp/
https://www.city.nichinan.lg.jp/
https://www.town.kyotamba.kyoto.jp/
https://www.city.kameoka.kyoto.jp/
https://www.city.otsu.lg.jp/
https://www.city.omihachiman.lg.jp/
https://www.city.miyazu.kyoto.jp/
https://www.town.yosano.lg.jp/
https://www.kouratown.jp/
https://www.city.yawata.kyoto.jp/
https://www.city.nagahama.lg.jp/
https://www.city.kyotango.lg.jp/
https://www.city.kyotanabe.lg.jp/
https://www.town-takachiho.jp/
https://www.town.shintomi.lg.jp/
https://www.town.takaharu.lg.jp/
https://www.town.shiga-hino.lg.jp/
https://www.town.kadogawa.lg.jp/
https://www.city.kizugawa.lg.jp/
https://www.city.nobeoka.miyazaki.jp/
https://www.town.mimata.lg.jp/
https://www.town.taga.lg.jp/
https://www.city.kobayashi.lg.jp/
https://www.city.saito.lg.jp/
https://www.town.aya.miyazaki.jp/
https://www.city.moriyama.lg.jp/
https://www.city.nagaokakyo.lg.jp/
https://www.city.hikone.lg.jp/
https://www.city.miyazaki.miyazaki.jp/
https://www.city.miyakonojo.miyazaki.jp/
https://www.town.kijo.lg.jp/
https://www.town.kawaminami.miyazaki.jp/
https://www.town.tsuno.lg.jp/
https://www.town.miyazaki-misato.lg.jp/
https://www.vill.morotsuka.miyazaki.jp/
https://www.town.gokase.miyazaki.jp/
https://www.city.uji.kyoto.jp/
https://www.city.shiga-konan.lg.jp/
```
:::

強すぎますね。日本の自治体なので、信用度爆上がりかもしれません。
国レベルでは発見することができませんでしたので、`*.go.jp`で Cloudflare を使用しているサイトなどあればコメントで教えていただきたいです。

## 最後に
どうだったでしょうか、これらの根拠を使えば、先生方の検討も加速するのではないでしょうか。

この記事が役に立ったのなら幸いです。
根拠もっとあるよ！って方はコメントで教えていただきたいです。
