---
title: "北のミサイル開発を Deno で抑止する" # 記事のタイトル
emoji: "🇰🇵" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["北朝鮮", "セキュリティ", "deno"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

最近の Web3 エンジニアをターゲットにしたマルウェアについての話題が流行していますよね。悪意のあるプロジェクトを実行させていろいろな悪いことをするという手口です。
https://zenn.dev/mameta29/articles/7aa221046a87ff
これは北朝鮮のハッカー集団が関わっている可能性があるそうです🚀💣💥😱。Cheena さんが言っているので間違いないでしょう！！！[^cheena_post]
[^cheena_post]: https://x.com/cheenanet/status/1894794342830874627

経済制裁をすり抜けて資金を入手する仮定の一部として、Web3 開発者に悪意のあるプロジェクトでお金を稼いでいるのでしょう。これらの資金はミサイル開発にも使われかねません。これは日本の安全保障において大きな問題です。そのため、Deno を用いて北朝鮮のミサイル開発を抑止しましょう！

## Deno とは？

Deno を知らない人のために簡単に説明しましょう。Deno は、Node.js のような JavaScript ランタイムです。要は、JavaScript コードをブラウザ以外で動かすためのソフトウェアです。
Node.js と異なるポイントはいくつかありますが、その一つに「セキュア」という点があります。ブラウザにおいて、悪質なサイトにアクセスして、全ての権限を拒否されたとき、パソコンがハッキングされるとかいうことはありません。しかし、Node.js で実行する JavaScript の権限は大きく、勝手にファイルの読み書きしたりコマンド実行したりできてしまいます。Deno は、ブラウザのように、安全な状態で JavaScript コードの実行を可能にします。結構大きい利点ですよね。

## マルウェアの入手

hxxps://github[.]com/abdibrokhim/web3-online-chess-game

## Node.js で実行する場合のリスク

### npm i

幸い、今回実行するプロジェクトではこの手法はないのですが、リスクとしては存在します。

まず、プロジェクトのセットアップ時に `npm i` をすると思います。その時点から任意のコードを実行できるチャンスは北朝鮮にあるのです。
例えば、`package.json` に `postinstall` スクリプトがある場合、
```json
{
  "scripts": {
    "postinstall": "rm -rf ~/Desktop"
  }
}
```
Linux で実行するとあなたのデスクトップを破壊できます。
このほかにも、npm i で実行される scripts としては、
- `preinstall`
- `install`
- `postinstall`

があるので注意が必要です。

仮に、preinstall/install/postinstall を削除したとして、リスクはまだあります。依存パッケージの postinstall です。

例えば
```json
{
  "dependencies": {
    "super-evil-package": "*"
  }
}
```
みたいに依存関係が設定されていて、`super-evil-package` の postinstall は `npm i` で実行されます。
`super-evil-package` 自体に postinstall がなくても、その依存関係にあったら実行されます。

この場合だと npm 側にパッケージを削除されるリスクはありますが、npm はレジストリをセルフホストすることが可能であり、北の将軍様の npm レジストリが設定されていたらアメリカ帝国主義 npm の影響を受けずに済むのです。

つまり、npm i だけで北のミサイル開発を支援してしまう可能性があるのです。

### プロジェクトの実行

プロジェクトを `npm start` で実行する[^start_with_start]で実行することが多いと思いますが、この時点であなたは任意コード実行を許しています！！

Node.js の権限を甘く見てはできません。具体的なリスクは、この記事がとてもわかりやすいです:
https://zenn.dev/waki285/articles/web3-malware-deobfuscated


[^start_with_start]: https://zenn.dev/mameta29/articles/7aa221046a87ff#ハッカーの手口の詳細解説

## Deno を使おう

Deno は、前述の通り、セキュアな JavaScript ランタイムです。

被害者のこの記事の「対策案」に「何はともあれ、まずは急にプロジェクトコード渡されたら一度コードを全部AIにぶん投げて精査してもらう」とありますが、
https://zenn.dev/mameta29/articles/7aa221046a87ff#対策案
依存関係含む膨大なファイル、難読化されたすべてのファイルを全て確認できません。Deno を使いましょう。

### 安全な npm i

Deno では、`deno i` の代わりに `npm i` を使えます。

Deno は、明示的に許可を与えない限り依存パッケージの postinstall 等を実行しません！
プロジェクトルートの package.json についている postinstall 等は、実行されません。しかし、今後のアップデートで実行されるようになる可能性があるので、念のため精査しておきましょう。
```ansi
\u001b[0;40m\u001b[1;32mThat's some cool formatted text right?\u001b[0m
or
\u001b[1;40;32mThat's some cool formatted text right?\u001b[0m
```

