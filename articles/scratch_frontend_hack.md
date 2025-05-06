---
title: 【React】 Scratch のフロントエンドをハックしよう" # 記事のタイトル
emoji: "🔨" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["scratch", "react"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

https://scratch.mit.edu/ は React で書かれています。これをハックしてみましょう。
外部から Scratch のフロントエンドの State にアクセスすることができるため、いろいろ遊べます。

## `_reactRootContainer`

Scratch は React 16 を使用していますが、React にマウントされたルートの Element には `_reactRootContainer` というプロパティがついています。これを使えば、React の内部状態にアクセスできます。
[Scratch のプロジェクトページ](https://scratch.mit.edu/projects/1170596036/) では、ルートの Element の ID は `app` です。これを使ってみましょう:
```js
const app = document.getElementById('app')
const rootContainer = app._reactRootContainer
console.log(rootContainer)
```
ブラウザのコンソールで実行してみると、以下のような結果が得られます。
![](https://storage.googleapis.com/zenn-user-upload/de0b41c6d6d2-20250506.png)

## Fiber ノードを取得する

React は内部的に Fiber というグラフ構造のデータを使用して UI を管理しています。先ほど取得した `rootContainer` から Fiber ノードを取得することができます。
```js
const rootFiberNode = rootContainer._internalRoot.current
console.log(rootFiberNode)
```
![](https://storage.googleapis.com/zenn-user-upload/e677909f2b78-20250506.png)

しかし、これはルートの Fiber ノードであり、実際のコンポーネントの Fiber ノードはその子供にあります。これを取得するためには、`rootFiberNode.child` を使います。
```js
const appFiberNode = rootFiberNode.child
```

`memoizedProps` というプロパティを使うことで、コンポーネントの Props にアクセスできます。
ルートのコンポーネントには、`store` という props を使って Redux のストアを渡しています。これを取得して、Scratch のストアにアクセスしてみましょう。
```js
const scratchState = appFiberNode.memoizedProps.store.getState()

console.log(scratchState)
```
![](https://storage.googleapis.com/zenn-user-upload/56fda19b6a79-20250506.png)
いくつかのストアが組み合わさっていますが、重要なのは `scratchGui` というストアです。これを使うことで、Scratch エディタにアクセスできます。

## vm を使って遊ぼう

Scratch は、Scratch VM という仮想マシンを利用してプロジェクトを実行しています。この VM は、`scratchGui` ストアの中に格納してあります。取り出してみましょう。
```js
const vm = scratchState.scratchGui.vm
console.log(vm)
```
![](https://storage.googleapis.com/zenn-user-upload/4acd8b820acd-20250506.png)

vm を取得すればいろいろなことができます。例えば、コンソールからターボモードを有効にすることができます。
```js
vm.runtime.turboMode = true
```
![画面録画 2025-05-06 134852](https://storage.googleapis.com/zenn-user-upload/0fbf0d9205bd-20250506.gif)

また、このように変数を外部から変更できます。
![画面録画 2025-05-06 135826](https://storage.googleapis.com/zenn-user-upload/3aa8d28d6b9d-20250506.gif)

## ストアからアクセスできない値を変更する

ストアからアクセスできない値もあります。例えば、Scratch では自分以外が作成したプロジェクトで「中を見る」ボタンを押すと、「クラウド変数」と呼ばれる WebSocket を使ったユーザ共通の変数の通信が接続されます。この切断処理はストアからアクセスすることができません。
しかし、Fiber ノードを使い、任意のコンポーネントを取得することが可能です。

```ts
export function getSpecifiedFiber(
	root: ReactFiber,
	cond: (fiber: ReactFiber) => boolean,
) {
	const stack = [root]
	while (true) {
		const fiber = stack.pop()
		if (!fiber) {
			return null
		}

		if (cond(fiber)) {
			return fiber
		}
		if (fiber.child) {
			stack.push(fiber.child)
		}
		if (fiber.sibling) {
			stack.push(fiber.sibling)
		}
	}
}
```
この関数は、Fiber ノードを深さ優先探索で探索し、条件を満たすノードを返す関数です。これを使います。

クラウド変数の接続処理は、`cloudManagerHOC` という HOC が `CloudManager` というコンポーネントを UI に追加することで行われています:
https://github.com/scratchfoundation/scratch-editor/blob/ddd6535b7e5eca56af1c7a2244e6d8c1e3ee81a7/packages/scratch-gui/src/lib/cloud-manager-hoc.jsx

この中に「中を見る」を押したときにの切断処理があります。
https://github.com/scratchfoundation/scratch-editor/blob/ddd6535b7e5eca56af1c7a2244e6d8c1e3ee81a7/packages/scratch-gui/src/lib/cloud-manager-hoc.jsx#L64-L74

これを空の関数に置き換えることで、クラウド変数の接続を切断を無効化することができます！

まず、`getSpecifiedFiber` を使って `CloudManager` がマウントされている Fiber ノードを取得しましょう。
https://github.com/scratchfoundation/scratch-editor/blob/ddd6535b7e5eca56af1c7a2244e6d8c1e3ee81a7/packages/scratch-gui/src/lib/cloud-manager-hoc.jsx#L136-L145
このように propTypes に `canModifyCloudData` というプロパティがあるので、これを使って判定します。

```ts
const cloudManagerFiber = getSpecifiedFiber(root, (fiber) => {
    if (typeof fiber.type === 'function') {
        const propTypes = fiber.elementType.propTypes
        if (propTypes && 'canModifyCloudData' in propTypes) {
            return true
        }
    }
    return false
});
```

こうして取得した Fiber ノードから、コンポーネントを取得しましょう。
```js
const cloudManagerHOC = cloudManagerHOCFiber.elementType.prototype
```

このコンポーネントの `disconnectFromCloud` メソッドを空の関数に置き換えます。
```js
cloudManagerHOC.disconnectFromCloud = () => {}
```
これで「中を見る」を押したとしても、クラウド変数の接続が切断されなくなります。
![](https://storage.googleapis.com/zenn-user-upload/f633177a1b7c-20250506.gif)

このように、React の Fiber ノードを使うことで、ストアからアクセスできない値も変更することができます。

## まとめ

* Scratch は React 16 を使用している
* Fiber ノードを使えば Scratch を外部からハックできる
* Redux ストアからアクセスできない値も Fiber ノードを使えば変更できる

みなさんも、Scratch のソースコードを読み、ハックしてみてください。Happy Hacking!
