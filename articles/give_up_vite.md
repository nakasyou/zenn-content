---
title: "え？まだ \"vite\" 使ってるの？ バンドラー 2025 まとめ" # 記事のタイトル
emoji: "⚡" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["vite", "farm", "bun", "rolldown", "esbuild"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

未踏ジュニア Advent Calendar 2025 の記事です。
https://adventar.org/calendars/11697

## 対象読者

* JS のバンドラーやツールチェーンに興味がある人
* 今年の歴史を振り返りたい人
* ツールチェーンを選定したい人

## 背景

今年は未踏ジュニアという U-18 のクリエイター支援プログラムで Web ベースの画像編集アプリケーションを作成していました。

最初は Vite を使っていたのですが、開発サーバーの起動時間がこのくらいかかってしまいます。
![VITE v6.0.5  ready in 31012 ms という出力のスクショ](https://storage.googleapis.com/zenn-user-upload/aaa163e31526-20251208.png)
毎回の起動に 30 秒かかるという地獄でした。

画像編集ソフトは重いので、何回もプロジェクトで最新のバンドラに切り替えまくりました。
その履歴を振り返ることで 2025 年のバンドラ事情も振り返れたらと思います。バンドラだけでなくツールチェーンも含めて振り返ります。

俯瞰的な歴史とプロジェクトの移行履歴が混ざっているので若干わかりにくいかもしれません..

## 1 月

### Rspack v1.2 リリース

Rust 製の Webpack 互換バンドラである Rspack の v1.2 がリリースされました。
https://rspack.rs/blog/announcing-1-2#persistent-cache

ビルド過程のキャッシュをすることができるようになり、大幅に開発を高速化できるようになりました。

### 未踏ジュニア: Farm 発見

先ほどお見せしたスクショのように、Vite の起動速度が劇的に重かったので、代替のツールを探しました。

そして見つけたのが Farm です。Farm は Rust 製のバンドラ/開発ツールで、Vite と API の互換を持っています。

https://www.farmfe.org/

![](https://storage.googleapis.com/zenn-user-upload/f6e4eb577cfc-20251208.png)

公式サイトの図ですが、 Farm は Vite よりコールドスタートの速度が大幅に速いことが分かります。これにより超高速で超快適な開発生活が待っていると思っていました...

Farm はやはり不安定で、開発中のクラッシュやバージョンの不整合によりうまく動かなかったりすることが多発しました。

## 2 月

### Bun v1.2.3 リリース

Bun v1.2.3 がリリースされました！！

https://x.com/jarredsumner/status/1889222073983381975

Bun は JavaScript ランタイムなのですが、そこに Vite のような React 開発サーバーの機能が追加されたようです。

```ts
import App from './app.html'

Bun.serve({
  routes: {
    '/*': App
  }
})
```
のように html を import して `Bun.serve` でサーバーを起動するだけでだけで React アプリケーションの開発サーバーを立ち上げることができます。ホットリロード付きです。

### 未踏ジュニア: Bun React 見送り

この機能が気になってプロジェクトへの導入をチームメンバーと相談したのですが、この時点ではまだ実験的な機能であり、導入するには心配だったため見送りました。

## 3 月

### Rspack v1.3 リリース

https://rspack.rs/blog/announcing-1-3#performance-improvements

コード分割の速度が向上し、さらにバンドル時にターゲット環境に適した最小のコードを生成するようになりました。

### typescript-go

これはバンドラではないのですが、TypeScript を Go に移植することでパフォーマンスを向上させる typescript-go (=tsgo) が公開されました。
https://devblogs.microsoft.com/typescript/typescript-native-port

VS Code 自体のコンパイルが 77.8s から 7.5s に短縮され、 10.4 倍の高速化をこの時点で達成しています。


### rolldown-vite リリース

Vite はそのビルドのベースに esbuild/rollup を使用しています。
Rust 製の Rollup 互換バンドラの Rolldown がありますが、
https://rolldown.rs/
これを搭載した rolldown-vite がリリースされました。

Vite は開発サーバーで esbuild を使い、ビルドで rollup を使っていますが、これを両方 rolldown に置き換えることで開発時とビルド時の差を減らすことができるらしいです。

Vite は開発時にはバンドルしないことによって起動速度などの高速化を図っています。しかし、ブラウザの ESM 解決速度を Rolldown のようなネイティブバンドラが上回れば、バンドルしたほうが高速らしいです。これを実践しているのが Turborepo や Bun ですが、将来的には Rolldown の統合によってそれを実現するようです。

### tsdown リリース

ライブラリのバンドルにはよく Rollup ベースの tsup が使われますが、それに対して rolldown を使った tsdown が rolldown からリリースされました。

https://github.com/rolldown/tsdown

ライブラリ開発者が高速にライブラリをビルドできるようになりました。

### 未踏ジュニア: tsgo 見送り

未踏ジュニアでは、tsgo は自分でビルドする必要があり、また実験的すぎたのでプロジェクトでは見送りました。
また、rolldown-vite は存在を把握していませんでした。

## 4 月

### Next.js の Rspack サポート

https://rspack.rs/blog/rspack-next-partner

`next-rspack` というライブラリがリリースされ、Next.js で Rspack を使うことができるようになりました。

### 未踏ジュニア: 変化なし

Next.js をやめていたのでプロジェクトには直接関係ありませんでした。

## 5 月

### tsgo のプレビュー版公開

https://github.com/microsoft/typescript-go/pull/876

なんと tsgo のビルド成果物が配布され、VS Code 拡張も公開されるようになりました！！！

`bun add -D @typescript/native-preview` でインストールが可能になります。


### Bun v1.2.14 リリース

`bun init` としたときに React アプリケーションのテンプレートを選択できるようになりました。


https://bun.sh/blog/bun-v1.2.14#react-flag-for-bun-init

![](https://storage.googleapis.com/zenn-user-upload/ccee9e1e8e7f-20251208.png)

このような選択画面が出てきて、まさに Bun が React 開発ツールでもあることを前面に押し出している感じがあります。

### 未踏ジュニア: tsgo 見送りと Bun React 導入

未踏ジュニアでは、tsgo を プロジェクトに導入してみましたが、まだ不安定でした。

うまく VS Code で補完が効いてくれなかったり
![](https://storage.googleapis.com/zenn-user-upload/0dfe50313a1c-20251208.png)
このように tsc では通るのに tsgo ではエラーになることがありました。
![](https://storage.googleapis.com/zenn-user-upload/242c3e1c7f7d-20251208.png)
LSP は不安定ですが、CLI は不安定でもあまり気にならなかったことや、monorepo だったこともあって、

* tsc と tsgo の結果が一致している package -> tsgo
* うまくいかない package -> tsc

のようにうまく使い分けて開発を進めました。

また、Bun については、React 開発サーバーの安定化の兆しだと思い、プロジェクトに導入しました。
https://bun.sh/blog/bun-v1.2.14#react-flag-for-bun-init
Bun の React 開発サーバーはまだ不安定でしたが、 Farm よりはマシだったので Bun に乗り換えました。

## 6 月

### Vite v7 リリース

Vite v7.0 がリリースされました！！！
https://vite.dev/blog/announcing-vite7

ここで目玉なのが Environment API です。これは Vite v6 時代 Runtime API と呼ばれていたもので、JavaScript ランタイムを Environment という単位にまとめるものです。
Vite は当初ブラウザ向けに作られましたが、のちに SSR をサポートすることになり、その環境ごとの取り扱いが複雑でした。また Node.js を念頭に作られていたため、Cloudflare Workers などの開発に不便でした。
Environment API は、従来の Client/SSR の区別をやめて、プラグイン開発者が自由に Environment を定義できるようになります。
これにより、Vite を Cloudflare Workers/Wrangler 上で動かしたりすることが容易になります。Vite とランタイム間の通信を実装することによって Vite がランタイムを認識できるようになるため、Vite をより多くの環境で動かせるようになります。
Cloudflare の Vite Plugin はこの仕組みを利用しています。
https://developers.cloudflare.com/workers/vite-plugin/

### 未踏ジュニア: rolldown-vite 提案

未踏ジュニアではちょうど期間の始まりを示す「ブースト合宿」があり、メンバーと初めて対面しました。

プロジェクトでは Bun の開発サーバーを使っていましたが、HMR のときに頻繁にクラッシュするので、そのときに rolldown-vite を使わないかとメンバーに提案しましたが、今のままで安定しているからと跳ねられました...
どうやら macOS ではうまくいっていたらしくて、 WSL 固有の問題のようです。

## 7 月

### Deno 2.4: `deno bundle` 復活

Deno 1.x 時代に消えてしまった `deno bundle` コマンドが返ってきました！これは TypeScript ファイルをバンドルするコマンドです。

https://deno.com/blog/v2.4#deno-bundle

どうやら独自実装をやめて、単純に esbuild をラップする形で実装されたようです。

### SvelteKit: rolldown-vite サポート

https://svelte.dev/blog/whats-new-in-svelte-july-2025

SvelteKit が rolldown-vite をサポートしました。しかし、この時点では Tree-Shaking がうまく動かなかったそうです。

### 未踏ジュニア: rolldown-vite 導入

未踏ジュニアのプロジェクトで Bun の開発サーバーがクラッシュしまくっていて困ったので、こっそり rolldown-vite に変えてしまいました。こんなに話題になっているのに変えないのはもったいない。

そうするとあらびっくり、高速かつ超安定で、本当にベータ版なのかというレベルで動作しました。クラッシュもしません。

## 8 月

### tsgolint

oxc という JS のための統合ツールチェーンプロジェクトが、oxlint という高速な linter と tsgo を組み合わせた tsgolint をリリースしました。
https://github.com/oxc-project/tsgolint/releases/tag/v0.0.2

TypeScript の型情報を使った lint を ESLint で行う人も多いと思いますが、このプロセスには ESLint の遅さと TS の遅さの両方の負担がかかってしまいます。tsgolint は ESLint の代わりに oxlint、tsc の代わりに tsgo を使うことで大幅な高速化を実現しています。

### vite-plugin-svelte: rolldown-vite サポート

https://svelte.dev/blog/whats-new-in-svelte-august-2025

Svelte の Vite プラグインである vite-plugin-svelte が、rolldown-vite をサポートしました。これにより、SvelteKit でも安定して rolldown-vite を使うことができるようになりました。

### 未踏ジュニア: 変化なし

React を使っていたことや、Biome を使用していたのでツールチェーン関係はほとんど 8 月は変更しませんでした。

## 9 月

### create-vite で rolldown-vite テンプレートが利用可能に

https://github.com/vitejs/vite/pull/20820

![](https://storage.googleapis.com/zenn-user-upload/c568eb27d136-20251208.png)

こいつが実装されました。rolldown-vite の安定化のにおいがめっちゃ感じられる変化です。

また、Qwik が Playground の repl で rolldown を使い始めたり、date-fns が oxc を使い始めたりと、VoidZero 系のツールチェーンの採用が増えていった月でもあります。

### 未踏ジュニア: 変化なし

未踏ジュニアでは、rolldown-vite を使い続けました。

## 10 月

### Next 16 リリース

https://nextjs.org/blog/next-16#turbopack-stable

Next.js v16 では Turbopack がデフォルトで使われるようになりました。

### React Compiler v1.0

https://react.dev/blog/2025/10/07/react-compiler-1

React ルールに従ったコードを最適化する React Compiler が v1 をリリースしました。これは SWC, Vite などの既存ツールチェーンを意識して作られたため、統合することが可能です。

### Bun v1.3 リリース

https://bun.com/blog/bun-v1.3

Bun 1.3 では、React 開発サーバーの機能が超アップグレードされました。

`bun index.html` で開発サーバーをスタートすることができます。Bun はもともと Next.js の高速開発ツールだったので、若干伏線回収みたいな感じがあります。
また、Vite のような `import.meta.hot` API もサポートされました。

### Vite+ 発表

https://viteplus.dev/

Vite+ は、Web のための統合されたツールチェーンです。

* `vite build` -> ビルド、Rolldown
* `vite test` -> テスト、Oxfmt
* `vite lint` -> Lint、Oxlint
* `vite run` -> TurboRepo みたいなモノレポ管理
* `vite fmt` -> フォーマット、Oxfmt

みたいに VoidZero/Oxc 系のツールチェーンを Vite+ として統合したものです。Evan さんに質問する機会があってこれはただのエイリアスなのかと伺ったことがあるのですが、そうではなくより高度に統合されたツールチェーンだそうです。

コミュニティ向けは無料、スタートアップ企業向けは有料など、OSS ではなくビジネスモデルを意識したツールチェーンになっています。

Vite とは別のプロダクトとして、Vite は MIT ライセンスのまま維持されます。

### 未踏ジュニア: 変化なし

Bun の開発サーバーがまだ心配だったので、rolldown-vite の使用を継続しました。Vite+ は公開していないので、組み込みたかったのですが断念しました。

## 11 月

### 未踏ジュニア: 成果報告会

未踏ジュニアでは、11/3 に成果報告会としてプロジェクトの未踏ジュニアとしての期間の終了のイベントを行います。

![](https://storage.googleapis.com/zenn-user-upload/a0c228e700fb-20251208.png)

成果報告会では、以下のような技術でしめくくりました。

![](https://storage.googleapis.com/zenn-user-upload/4fc639fcd316-20251208.png)

技術スタックはこのようになり、最終的には rolldown-vite でフィニッシュできました。

## 12 月

### Vite v8 Beta リリース

Vite v8 Beta は rolldown をデフォルトにしたもので、基本的に rolldown-vite の後継だと思います。
`bun add -D vite@8.0.0-beta.0` でインストールできます。

また、tsconfig.json の paths オプションを自動的に解決する機能も追加されました。

### oxfmt

https://oxc.rs/docs/guide/usage/formatter

Oxc ベースのフォーマッターである oxfmt がリリースされました。

Vite+ にも組み込まれることが予想されます。VoidZero のスタックが着実に積みあがっている感じがします。

## まとめ

まだ "vite" を使っているのですか？？？？

紹介してきたように、安定しているので `bun add -D rolldown-vite` か `bun add -D vite@8.0.0-beta.0` を使いましょう！！

来年もどのようにバンドラーが進化していくのか楽しみですね
