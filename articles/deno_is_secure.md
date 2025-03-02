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

## 対象読者

- あのマルウェアが何をしているのか気になる人
- Web3/JavaScript 開発者

## Deno とは？

Deno を知らない人のために簡単に説明しましょう。Deno は、Node.js のような JavaScript ランタイムです。要は、JavaScript コードをブラウザ以外で動かすためのソフトウェアです。
Node.js と異なるポイントはいくつかありますが、その一つに「セキュア」という点があります。ブラウザにおいて、悪質なサイトにアクセスして、全ての権限を拒否されたとき、パソコンがハッキングされるとかいうことはありません。しかし、Node.js で実行する JavaScript の権限は大きく、勝手にファイルの読み書きしたりコマンド実行したりできてしまいます。Deno は、ブラウザのように、安全な状態で JavaScript コードの実行を可能にします。結構大きい利点ですよね。

## マルウェアの入手

https://github[.]com/arsantin/cometec[^get]

[^get]: https://zenn.dev/waki285/articles/web3-malware-deobfuscated#マルウェアの入手

## Node.js で実行する場合のリスク

### npm i

幸い、今回実行するプロジェクトではこの手法はないのですが、リスクとしては存在します。

まず、プロジェクトのセットアップ時に `npm i` をすると思います。その時点から任意のコードを実行できるチャンスは北朝鮮にあるのです。
例えば、`package.json` に `postinstall` スクリプトがある場合、
```json:package.json
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
```json:package.json
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
![IMG_4487](https://github.com/user-attachments/assets/6ac72061-ae41-4c24-8b0f-df592eb8cabf)

プロジェクトルートの package.json についている postinstall 等も実行されませんが、これは今後のアップデートで実行されるようになる可能性があるので、念のため精査しておきましょう。依存関係のものは大丈夫です。

では、deno i を実行しましょう。

```shell
deno i
```

### スクリプトを実行する

問題のファイルは `server/app.js` です。実行してみましょう。
```shell
deno \
  --allow-read=./  \
  --unstable-bare-node-builtins \
  --unstable-detect-cjs \
  --unstable-node-globals \
  --unstable-sloppy-imports \
  --unstable-unsafe-proto \
  server/app.js
```
![IMG_4491](https://github.com/user-attachments/assets/90d56305-dcca-4c2b-bad8-083ab5c29213)
こんな感じで環境変数、ファイルシステム、ネットワークなどのアクセスはユーザーの許可なしではできません。ブラウザはサンドボックスで、ユーザーの許可なしに通知を送信したり位置情報を送信できません。同様なことがサーバーサイド JavaScript でできるという強みを持つのが Deno です！

許可したいなら Y、拒否したいなら n を入力します。流石に Chrome のプロファイルへのアクセスは気持ち悪いので拒否してみましょう。
![IMG_4492](https://github.com/user-attachments/assets/d1f63a04-35c1-4060-82db-871b48d1be2a)
いいですね。

<details>
<summary>出力結果</summary>
  
```output
Warning Resolving "fs" as "node:fs" at file:///workspace/cometec/server/app.js:1:20. If you want to use a built-in Node module, add a "node:" prefix.
Warning Resolving "path" as "node:path" at file:///workspace/cometec/server/app.js:2:22. If you want to use a built-in Node module, add a "node:" prefix.
Warning Resolving "http" as "node:http" at file:///workspace/cometec/server/app.js:3:22. If you want to use a built-in Node module, add a "node:" prefix.
Warning Resolving "crypto" as "node:crypto" at file:///workspace/cometec/server/config/util.js:1:22. If you want to use a built-in Node module, add a "node:" prefix.

┏ ⚠️   Deno requests  env access to "FORCE_COLOR" .
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-env
┠─  Run again with --allow-env to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all env permissions) > y
✅  Granted env access to "FORCE_COLOR".

┏ ⚠️   Deno requests  env access to "TERM" .
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-env
┠─  Run again with --allow-env to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all env permissions) > y
✅  Granted env access to "TERM".

┏ ⚠️   Deno requests  env access to "CI" .
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-env
┠─  Run again with --allow-env to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all env permissions) > y
✅  Granted env access to "CI".

┏ ⚠️   Deno requests  env access to "TEAMCITY_VERSION" .
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-env
┠─  Run again with --allow-env to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all env permissions) > y
✅  Granted env access to "TEAMCITY_VERSION".

┏ ⚠️   Deno requests  env access to "COLORTERM" .
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-env
┠─  Run again with --allow-env to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all env permissions) > y
✅  Granted env access to "COLORTERM".

┏ ⚠️   Deno requests  env access to "TERM_PROGRAM" .
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-env
┠─  Run again with --allow-env to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all env permissions) > y
✅  Granted env access to "TERM_PROGRAM".

┏ ⚠️   Deno requests  env access  .
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-env
┠─  Run again with --allow-env to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all env permissions) > y
✅  Granted env access.

WARNING: No configurations found in configuration directory:/workspace/cometec/config
WARNING: To disable this warning set SUPPRESS_NO_CONFIG_WARNING in the environment.

┏ ⚠️   Deno requests  net access to "0.0.0.0:8510" .
┠─ Requested by ` Deno.listen() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-net
┠─  Run again with --allow-net to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all net permissions) > y
✅  Granted net access to "0.0.0.0:8510".

┏ ⚠️   Deno requests  net access to "0.0.0.0:8000" .
┠─ Requested by ` Deno.listen() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-net
┠─  Run again with --allow-net to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all net permissions) > y
✅  Granted net access to "0.0.0.0:8000".

server is listening on port 8000

┏ ⚠️   Deno requests  sys access to "hostname" .
┠─ Requested by ` Deno.hostname() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-sys
┠─  Run again with --allow-sys to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all sys permissions) > y
✅  Granted sys access to "hostname".

┏ ⚠️   Deno requests  sys access to "homedir" .
┠─ Requested by ` node:os.homedir() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-sys
┠─  Run again with --allow-sys to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all sys permissions) > y
✅  Granted sys access to "homedir".

┏ ⚠️   Deno requests  read access to "/home/gitpod/.config/google-chrome" .
┠─ Requested by ` Deno.lstatSync() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-read
┠─  Run again with --allow-read to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all read permissions) > y
✅  Granted read access to "/home/gitpod/.config/google-chrome".

┏ ⚠️   Deno requests  read access to "/home/gitpod/.config/BraveSoftware/Brave-Browser" .
┠─ Requested by ` Deno.lstatSync() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-read
┠─  Run again with --allow-read to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all read permissions) > y
✅  Granted read access to "/home/gitpod/.config/BraveSoftware/Brave-Browser".

┏ ⚠️   Deno requests  read access to "/home/gitpod/.config/opera" .
┠─ Requested by ` Deno.lstatSync() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-read
┠─  Run again with --allow-read to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all read permissions) > y
✅  Granted read access to "/home/gitpod/.config/opera".

┏ ⚠️   Deno requests  read access to "/home/gitpod/AppData/Roaming/Mozilla/Firefox/Profiles".
┠─ Requested by ` Deno.lstatSync() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-read
┠─  Run again with --allow-read to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all read permissions) > y
✅  Granted read access to "/home/gitpod/AppData/Roaming/Mozilla/Firefox/Profiles".

┏ ⚠️   Deno requests  read access to "/home/gitpod/.config/Exodus/exodus.wallet" .
┠─ Requested by ` Deno.lstatSync() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-read
┠─  Run again with --allow-read to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all read permissions) > y
✅  Granted read access to "/home/gitpod/.config/Exodus/exodus.wallet".

┏ ⚠️   Deno requests  net access to "94.131.97.195:1224" .
┠─ Requested by ` Deno.connect() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-net
┠─  Run again with --allow-net to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all net permissions) > y
✅  Granted net access to "94.131.97.195:1224".

┏ ⚠️   Deno requests  write access to "/home/gitpod/.npl" .
┠─ Requested by ` Deno.openSync() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-write
┠─  Run again with --allow-write to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all write permissions) > y
✅  Granted write access to "/home/gitpod/.npl".

┏ ⚠️   Deno requests  read access to <exec_path> .
┠─ Requested by ` Deno.execPath() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-read
┠─  Run again with --allow-read to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all read permissions) > y
✅  Granted read access to <exec_path>.

┏ ⚠️   Deno requests  run access to "/bin/sh" .
┠─ Requested by ` Deno.Command().spawn() ` API.
┠─ To see a stack trace for this prompt, set the DENO_TRACE_PERMISSIONS environmental variable.
┠─  Learn more at:  https://docs.deno.com/go/--allow-run
┠─  Run again with --allow-run to bypass this prompt.
┗  Allow? [y/n/A] (y = yes, allow; n = no, deny; A = allow all run permissions) > y
```

</details>

以上が全部「y」した場合の実行結果です。展開すると表示されます。

Google Chrome, Opera, Brave, Firefox のプロファイルを抜き、Exodus のウォレットも抜き、94.131.97.195:1224 とかいう怪しすぎるサーバーに送信してるように読み取れます。さらに、sh コマンドも実行しています。何をするかはわかりませんが。

Deno なら防げますね。

## まとめ

怪しいプロジェクトを動かすときのセキュリティ対策の一環として、北のミサイル開発を抑止するために、Deno でまず動かしてみましょう！！

