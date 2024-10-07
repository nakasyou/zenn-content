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
      "default: "./dist/index.js"
    }
  }
}
```

みたいにまあまあ複雑です。ここに`module`とか`cjs`や`browser`みたいなフィールドが大量にあるとカオスです。
JSR なら
```jsonc:jsr.json
{
  "exports": {
    ".": "./index.ts"
  },
  "exports": "./index.ts" // または
}
```
みたいにして直接 TypeScript ファイルを指定することができます。
  
