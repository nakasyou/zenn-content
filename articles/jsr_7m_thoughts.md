---
title: "JSR を 7 ヶ月使った好感と不満" # 記事のタイトル
emoji: "📦" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["jsr", "deno", "javascript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

7 ヶ月前にこんな記事を書きました。
https://zenn.dev/nakasyou/articles/20230301_jsr
公開当時は最高だと思い、この 7 ヶ月間新しいパッケージの公開に npm を使わずに、JSR のみを使用してきました。その際に、たくさんの好感、そして同時に不満も多く生まれたので、それを羅列していきたいです。

## 👍
npm と比べていいことが一定数あります。批判だけするのはタチが悪いのでたくさん紹介していきます。

### TypeScript サポート

これ最高。
TypeScript パッケージを公開するとき、npm では

- tsc や esbuild などでトランスパイル/バンドルする
- 型定義を出力する

必要があり、設定ファイルも
```json:package.json
{
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "default": "./dist/index.js"
    }
  }
}
```

みたいにまあまあ複雑です。ここに`module`とか`cjs`や`browser`みたいなフィールドが大量にあるとカオスです。
JSR なら
```json:jsr.json
{
  "exports": {
    ".": "./index.ts"
  },
  "exports": "./index.ts" // または
}
```
みたいに直接 TypeScript ファイルを指定できるのです！ ビルドは不要です！ 忌々しい package.json の exports フィールドとはおさらばです！

### TSDoc からのドキュメント自動生成

例えば、以下のようなランダムな整数を取得する関数を公開するとします。そのときに、TSDoc を以下のように指定すると:
```ts:mod.ts
/**
 * Get random integer
 * @param min A min integer
 * @param max A max integer
 * @param opts Options
 * @returns Random value, min <= x <= max
 */
export const randInt = (min: number, max: number, opts?: RandOpts): number =>
  Math.floor(random(opts) * (max - min + 1)) + min
```

JSR のページ上にドキュメントが自動で生成されます。
![IMG_3847](https://github.com/user-attachments/assets/b06f9c61-7830-4e5d-b50a-59ec6597e1fd)

これは Hono というフレームワークの一部分ですが、以下のようにクラスのメソッドやプロパティも表示できます。Markdown も使えます。

![IMG_3848](https://github.com/user-attachments/assets/f4e26847-0284-452f-9123-048401c80436)

ライブラリの型定義や使い方を直感的に見たいユーザーにもいいです。また、TSDoc を自前で HTML にしてサイトにして公開するという開発者の手間も少なくなるでしょう。

### スコープによる分離

これは好みの問題ですが。
npm では、`my-package` のような命名でパッケージを公開できます。それに対して JSR では、`@scope/my-package` のようなスコープの下でしかパッケージが公開できません。

これにより享受できるメリットとして、typo を狙ったパッケージ名の悪用のリスクが減ります。
例えば、npm には [`-`](https://www.npmjs.com/package/-) というパッケージが存在します。
```shell
npm i -D ...
```
を
```shell
npm i - D ...
```
と typo してしまった場合、 `-` というパッケージと `D` というパッケージもインストールされてしまいます。
`-` には悪意のあるパッケージではありませんが、週間 6 万も間違ったパッケージがインストールされていると考えると怖いですね。これを低減することができます。

さらに、スコープの名字で製作者を明確にしたり、名前の衝突を防げたりといったメリットがあると筆者は考えます。

### npm との互換性

JSR は、npm レジストリと互換性を持っています。これにより、Node.js プロジェクトで JSR のパッケージを使用することができます。
TypeScript コードは自動的に JavaScript と .d.ts になります。

### OGP

JSR にはパッケージごとの OGP が存在します。npm にはありません。
ユーザーのリンククリック率を上げることができるかもしれません？

https://jsr.io/@ns/personalities

![IMG_3850](https://github.com/user-attachments/assets/21fb3746-e508-4845-a29f-6001413d8437)

## 👎

7ヶ月使って、悪いこともたくさん出てきました。紹介します。

### スコープの作成数に上限がある

JSR では、スコープの下にパッケージを公開すると前述しましたが、作成数に上限があります。

![IMG_3849](https://github.com/user-attachments/assets/95a32519-21e3-403f-bf7e-ddd62ceee319)

3 つしか作成することができません。新しいフレームワークのアイデアが思いついたから取得！や、新しい任意団体を立ち上げたから取得！みたいな感じに気軽に作成することはできません...
JSR チームに直接連絡を取ることで上限を伸ばしてもらえるようですが、どのくらい気軽にできるのかは未知数です。

前まで 2 つだった気がするので、上限が伸びたり撤廃されたりするのに期待できそうです。

### `jsr publish`で Deno がインストールされる!!

JSR にパッケージを公開するときに、`deno publish` をして公開することができます。または、npm に公開されている [jsr](https://www.npmjs.com/package/jsr) コマンドを利用して、`jsr publish` のように公開することができます。

これが問題なのですが、公開時に Deno を自動でインストールして、サブプロセスとして `deno publish` を呼んでいます。[^deno_publish_code]
公開機能だけが必要なのに、JavaScript Runtime や Linter/Formatter を含む 40MB 以上の Deno を自動でインストールするのには個人的にあまり好印象をもたらしません。

日本の企業については詳しく知りませんが、 jsr publish で Deno が動的にインストールされることを許容しない厳しい会社もあるのではないでしょうか。

[^deno_publish_code]: https://github.com/jsr-io/jsr-npm/blob/52f93c31b00661a6cf2ba40a9663e02e5b199506/src/commands.ts#L166

### Slow Types という 「言い訳」

(「言い訳」は誇張しています)

JSR には **Slow Types** という概念があります。JSR は、レジストリ単位で TypeScript をサポートしていると前述しました。その過程で生まれた「言い訳」が Slow Types です。JSR では、内部的に TypeScript コードを型定義ファイルである `.d.ts` にトランスパイルします。その際、tsc を使うと、JSR サーバーに大きな負荷がかかります。そのため、JSR は内部的に tsc を使わず、**型推論を行いません**。それを解決するために、「Slow Types」という建前を作っているのです。

このコードを例にします。
```ts:mod.ts
export const add = (a: number, b: number): number => {
  return a + b
}
```
型推論を行わずに .d.ts を出力するために、単純な TypeScript AST の操作によって .d.ts を出力します。[^ast_dts]

このコードでは、`: number` と明示的に指定されているので、AST の操作で
```ts:mod.d.ts
export declare const add: (a: number, b: number) => number
```
みたいに型推論なしで .d.ts の生成を実現しています。

では、次のコードではどうでしょう。
```ts:mod.ts
export const add = (a: number, b: number) => {
  return a + b
}
```
明示的に返り値を指定していません。返り値がコード内にないので、AST の操作によって `add` の `.d.ts` を生成することができないのです。`a + b` が number だと推測するには tsc が必要です。
このような、エクスポートされる変数や関数に対して明示的に型を指定していない状態を JSR は 「Slow Types」と呼んでいます。

別にこれくらいの型を書くくらいいいじゃないか！って思う人もいるかもしれません。しかし、Zod などのスキーマをエクスポートする場合はどうでしょう？
自作のプロトコルを作ったと仮定します。そのプロトコルのスキーマを Zod で公開したいとあなたは思いました。

```ts:index.ts
import * as z from 'zod'

export const schema = z.object({
  type: z.enum(['a', 'b']),
  data: z.object({ value: z.string() })
})
```
こんな感じにしましょう。しかし、公開はできません。schema の型を明示的に指定していないからです。Slow Types だと公開時に怒られます。

解決するには
```ts
import * as z from 'zod'

export const schema = z.ZodObject<{
    type: z.ZodEnum<["a", "b"]>;
    data: z.ZodObject<{
        value: z.ZodString;
    }, "strip", z.ZodTypeAny, {
        value: string;
    }, {
        value: string;
    }>;
}, "strip", z.ZodTypeAny, {
    type: "a" | "b";
    data: {
        value: string;
    };
}, {
    type: "a" | "b";
    data: {
        value: string;
    };
}> = // ...
```
みたいにめっちゃ長い型を明示的に指定しないといけません。Zod/TS の強みは消えてしまいますね。

[^ast_dts]: https://github.com/jsr-io/jsr/blob/d4cd64366870bca2a8ad4c5e030b42b9b48dea19/api/src/npm/emit.rs

### ビルドと相性が悪い

JSR、ビルドしたコードと相性が悪いです。例えば、先ほどの Zod コードを `tsc` 等でコンパイルして、ローカルで .d.ts を出力させるとします。
しかし、JSR の設定ファイルには .d.ts を指定できるフィールドはありません。

```ts:dist/index.ts
// @ts-self-types="./index.d.ts"
```
みたいに独自のコメントで、トランスパイルされた JavaScript コードに対して .d.ts を指定することができます。ちょっと面倒ですね。

#### Deno での問題

Deno では独自のモジュール解決システムを用いているため、前述のような Zod コードを `tsc` 等で .d.ts にコンパイルできません。そのため、このような方法で Slow Types を回避できません。
私はスキーマを JSR に上げるために、Deno プロジェクトを仕方なく tsc と互換性のある Bun に移行しました。

### Deno First じゃね？

JSR は Node.js 以外の JavaScript ランタイムが存在する時代のために作られた、多様性のある(?)レジストリのはずです。
しかし、JSR は前述の通り公開に Deno に依存しています。

また、Deno std という Deno が提供する標準ライブラリが JSR `@std` というスコープに存在します。[`@std/fs`](https://jsr.io/@std/fs) なんかは Deno しか対応していません。

さらに、JSR にはコマンドラインスクリプトを公開することができますが、Deno でしか実行できません。
```ts
deno run -Ar jsr:@fresh/init
```
みたいに Deno から JSR のモジュールを呼び出せますが、他のランタイムではできません。`jsr dlx`みたいなものはないです。

### 中央集権型システム

Deno の HTTP Imports は分散型システムに近かったです。しかし、JSR は中央集権型システムです。JSR は OSS ですが、 JSR 以外の Self-Host レジストリの使い方が文書化されているのを見たことがありません。また、JSR から別の JSR レジストリのパッケージを参照するのも不可能だと思います。

一方、パッケージが 1 つのサーバーのみに保存される分散型システムより、1 つの信頼できる中央集権型システムに依存した方がよいのかもしれません。

### 雑多
少しだけ不満なところ一覧
#### ダークモードない

npm にもないですが。
暗いところで眩しいです。

#### 検索弱い

ためせばわかりますが、検索弱いです。
スコアとか考慮してくれればいいのにって思います。

## npm vs JSR

|  | npm | JSR |
| --- | --- | --- |
| 信頼性 | ✅ | △ |
| ネイティブ TS | ❌ | ✅ |
| セルフホストレジストリ | ✅ | △ |

### 開発する視点

ネイティブ TypeScript サポートや、設定ファイルの簡潔さは JSR が勝ると思います。一定の条件下での JSR の開発体験は最高です。Deno ではもちろん、Bun や Node.js の開発でも JSR に公開できます。Deno でなくても開発体験は上々です。

しかし、前述したような Slow Types の問題や、ビルドとの相性を考えると、npm が有利になることもあるかもしれません。

### 使ってもらう視点

**技術的に** JSR は npm と上位互換性があります。npm で JSR パッケージをインストールできるといった具合です。
しかし、ユーザー目線ではそうではないかもしれません。
npm から JSR パッケージをインストールするとき、
```shell
npx jsr add @luca/flag
```
のようにします。
この際、`jsr` というパッケージを使用します。企業によっては、有名度がまだ低い JSR というレジストリやコマンドを使うことを許可してくれないかもしれません。
そういったことを考慮するなら、npm に公開するのが良いかもしれません。

## 感想、まとめ

7ヶ月前は、完璧で究極のレジストリだと思っていますが、7 ヶ月たち、意外とそうでもない感じがします。
完璧なレジストリなどないはずなので、今後の成長に期待したいです。

