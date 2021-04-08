---
title: "TypeScript触った事ないけど、型についてまとめてみた。" # 記事のタイトル
emoji: "🦄" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["typescript", "javascript", "初心者", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## 最初に

今の所 JavaScript → React → Next.js と来ているのでTypeScriptもやろうと思う。（JavaScript以外はまだ考えてアプリを作成できるレベルじゃない）

まだ良さが分からないTypeScriptを勉強して大好きになると思う。

巷では型を定義出来るのが良いらしい。それで型についてまとめてみた。

## 環境構築

- Node.jsインストール
- 作業ディレクトリを作成
- `npm install typescript --save-dev`
- 上記のコマンドはローカルインストールなので、tscコマンドを実行する場合はパス指定する必要がある。 `../node_modules/.bin/tsc -v`

## アンビエント宣言（declare）

型定義と呼ばれるもので `declare var x: number` 等の記述を `型定義ファイル（.d.ts）`と呼ばれる別ファイルで管理する。

下記のコードはTypeScriptでは変数の宣言がされてないとしてエラーになる。

```tsx
x = 30;          // error TS2304: Cannot find name 'x'.
console.log(x);  // error TS2304: Cannot find name 'x'.
```

なので

型定義ファイルか同じファイルに `declare var x: number;` と追加する事でエラー解決する事ができる。

基本的には別で作成した型定義ファイルに記述する事が推奨されている。

そして、JavaScriptのライブラリ等では型定義ファイルが用意されている事が多いのでそちらを使う。この型定義があるとIDE等でのコード補完機能が得られる。

## 型注釈

変数の隣に `:` と書いてその右側に型の注釈を入れる。

```tsx
let hasValue: boolean = true;
```

型が推論出来ない時に使用する。

例：初期値の入力で変数を代入しない時等。

```tsx
let hasValue: boolean;
```

## 型推論

TypeScriptがどの型か変数の値から自動で教えてくれる。

## オブジェクトの型書き方

```tsx
const person : {
	name: string;
	age: number;
} = {
	name: 'Jack',
	age: 21
}

//ネストする場合
const person : {
	name: {
		first: string;
		last: string;
};
	age: number;
} = {
	name: {
		first: 'Jack',
		last: 'Smith'
	},
	age: 21
}
```

## 配列の型書き方

```tsx
const fruits: string[] = ['Apple', 'Banna', 'Grrape']
```

## 列挙型

特定のまとまった型を受け入れる。

```tsx
enum CoffeeSize {
	SHORT = 'SHORT',
	TALL = 'TALL',
	GRANDE = 'GRANDE',
	VENTI = 'VENTI'
}

const coffee = {
	hot: true,
	//size: 'SHORT'この文字しか入らないようしたい。
	//こうすると (prorerty) size: CoffeeSizeという列挙型になる。
	size: CoffeeSize.SHORT
}
//入らない
coffee.size = true
coffee.size = 'SHOT'

//入る
coffee.size = CoffeeSize.SHORT
```

## union型

複数の型を指定したい時に使用する。

```tsx
let unionType: number | string = 10;
```

## リテラル型

それしか代入出来なくなる。

```tsx
//エラーになる。
const apple: 'apple' = 'hello'

//エラーなし。
const apple: 'apple' = 'apple'
```

## uinon型 + リテラル型

```tsx
let cloathSize: 'small' | 'medium' | 'large' = 'large';
const cloth = {
	color: 'white',
	size: clothSize
}
//上記の状態ではsizeはリテラル型になって'large'しか受け付けない。
cloth.size = 'small' 

//そこでオブジェクトの型定義でユニオンを使用する。
const cloth = {
	color: 'white';
	//この3つしか入らないくなる。
	size: 'small' | 'medium' | 'large'
} = {
	color: 'white',
	size: 'medium'
}

```

## エイリアス

先ほどの `'small' | 'medium' | 'large'` を変数みたいに格納したい。

```tsx
//独自の型を定義した感じになる。
type ClothSize = 'small' | 'medium' | 'large'
const cloth = {
	color: 'white';
	//この3つしか入らないくなる。
	size: ClothSize
} = {
	color: 'white',
	size: 'medium'
}

```

## 関数に型を付ける

引数と戻り値に型を付ける。

```tsx
function add(num1: number, nunm2: number): number {
	return num1 + num2
}
```

### Void型

関数が何も返さない（undefindが一応返ってくる）物だとどうなるのか。

TypeScriptには `undefined` 型は存在するが関数の型定義で使用するとエラーになるので登場するのは稀になる。ただし関数の最後に `return` が存在する場合は `undefined` 型を使用する事ができる。

そこで関数で `undefined` が返ってくる場合は型定義で `void` を使用する。

```tsx
function sayHello(): void {
	console.log('Hello');
}
// 実行するとundefindを一応返す。
// なのでundefined型も存在する。
let tmp: undefined;
```

### Null型

```tsx
let tmpNull: null = null; //ok
let tmpNull: null = undefined; //ok
//この逆でundefinedにnullをいれても良い。
```

## 無名関数に型書き方

```tsx
const anotherAdd: (n1: numebr, n2: number) => number = function (num1: number, num2: number): number {
	return num1 + num2
};

//片方の型を省略できる。
const anotherAdd: (n1: numebr, n2: number) => number = function (num1, num2): number {
	return num1 + num2
};
```

## アロー関数の型書き方

```tsx
const doubleNumber = (num: number): num => num * 2;
//こんな感じに型定義をずらす事もできる。    //戻り値の型
const doubleNumber2: (num: number) => number = num => num * 2;
```

## コールバック関数の型書き方

関数自体は戻り値を持たない。第一引数はnumberを型に持つ、第二引数は関数でその関数の引数はnumberを型に持ち戻り値もnumberを返す。

```jsx
function doubleAndHandle(num: number, cb: (num: number) => number): void {
	const doubleNum = cb(num * 2);
	console.log(num * 2);
}
// コールバック関数の引数にはdoubleAndHandleの第一引数が入る。
doubleAndHandle(21, doubleNum => {
	return doubleNum
});

//仮にコールバック関数の戻り値をvoidの型にするとエラーは起きないが戻り値は無効となり使用出来なくなる。
function doubleAndHandle(num: number, cb: (num: number) => number): void {
	const doubleNum = cb(num * 2);
	//if(doubleNum) //doubleNum.toString()とかにするとエラーになる。
	console.log(doubleNum);
}

doubleAndHandle('21', doubleNum => {
	return doubleNum
});

```

## Any型・Unknow型

- any型はなんでも入る型、便利だがTypeScriptの機能を放棄しているようにも思える。
- unknon型はany型より少し厳しい型

```tsx
let unknownInput: unknown;
let anyInput: any;
let text: string;
unknownInput = 'hello';
unknownInput = 21
unknownInput = true;
text = anyInput//エラーにならない。使うときもエラーにならない。
text = unknownInput//ここでエラーになる。何でも入れられるが使うときに注意がでる。
//unknownの使い方 stringの型が来たときのみ代入する。
if (typeof unknownInput === 'string') {
	text = unknownInput;
}

```

## Never型

エラーをキャッチする関数等の戻り値として使用される。

他にはずっとループで周り続ける関数の戻り値に使用されることもある。

```tsx
// もしneverが書かれていない場合、TypeScriptの型推論ではvoid型になる。
// 理由はnever自体が新しい型でバージョンから登場してまだ追いついていない。
function error(message: string): never {
	throw new Error(message);
}
console.log(error('This is an error'));
```

## 最後に

調べる前は String, Number等の引数型があらかじめ分かっても、そこまで読みやすくなるのかと思っていた。オブジェクトの型、配列の型で細かく型を設定できると知り確かにこれなら戻ってくるJsonファイルの型とかを書いておけば後でコードを読んでも実際に実行する機会とかを減らせして効率が上がるのかなと思った。まだ途中なので色々と勉強していきたいと思います。

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。

### 参照

[TypeScript｜アンビエント宣言(declare)と型定義ファイル(.d.ts) - わくわくBank](https://www.wakuwakubank.com/posts/501-typescript-declaration/)

[超TypeScript入門完全パック- TypeScriptでアプリを作りたい方必見！](https://www.youtube.com/watch?v=F9vzRz6jyRk)