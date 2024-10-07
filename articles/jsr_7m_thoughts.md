---
title: "JSR を 7 ヶ月使った好感と不満" # 記事のタイトル
emoji: "📦" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["jsr", "deno", "javascript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

7 ヶ月前にこんな記事を書きました。
https://zenn.dev/nakasyou/articles/20230301_jsr
公開当時は最高だと思い、この 7 ヶ月間新しいパッケージの公開に npm を使わずに、JSR のみを使用してきました。その際に、たくさんの好感、そして同時に不満が大きく生まれたので、それを羅列していきたいです。

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

さらに、スコープの名字で製作者を明確にしたり、名前の衝突を防げたりとしたメリットがあると筆者は考えます。

### 

