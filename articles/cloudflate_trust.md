---
title: "先生! Cloudflare は信用できます!" # 記事のタイトル
emoji: "🧑‍🏫" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["cloudflare", "cloudflare workers", "cloudflare pages"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
Cloudflare Pages 、使ってますか?超便利ですよね。
文化祭などでも、生徒がサイトを作るときに使用している場合があるそうです。

そんなときに問題が起こるかもしれません。「その Cloudflare って会社、信用できるの?? 」
根拠を集めましょう。

## どのような根拠?
顧客です。顧客が結構信用させるのにいいのではないでしょうか。

## 自治体を使う
これは強いカードです。自治体の Web サイトに Cloudflare が使われてたらいいのではないでしょうか。
[このサイト](https://uub.jp/opm/ml_homepage.html)が自治体の Web サイトを集めているので、それを使って確かめます。
DevTools で表からすべての URL を抽出して、 Deno で `cf-` ヘッダーが含まれているものを抽出しました。
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
