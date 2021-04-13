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
const a: object = {
	name: 'daigakuseidatta',
	age: 25
}
//a.name と実行してもそんなのないと言われる

//オブジェクトリテラル型で型定義する。
const person : {
	name: string;
	age: number;
} = {
	name: 'Jack',
	age: 21
}

//オプショナルプロパティを使う(?)
let person: {
	age: number
	lastName: string
	readonly first: string
	gender?: string//?で会ってもなくてもよくなる。
} = {
	age: 28,
	latsName: 'sato',
	firstName: 'tiaki'
	//genderがなくても怒られない。
}
person.gender = 'male' //後から追加できる。
person.lastName = 'kobayashi' //上書きできる。
person.firstName = 'kuniko' //上書き出来ない。

//インデックスシグネチャ
// オブジェクトが複数のプロパティを持つ可能性を示す。
// [key:T]: Uのように定義する。
// keyはstringかnumberのみ
const capitals: {
	[countryName: string]: string
} = {
	Japan: 'Tokyo',
	Korea: 'Seoul'
}
capitals.China = 'Beijing'
capitals.Canada = 'Ottawa'

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

//オブジェクト内に関数がある場合
const Falcon : {
    name: string;
    fly: () => void;
} = {
    name: '鷹',
    fly: function(): void{
        console.log(this.name + 'が大空を飛びました');
    }
};

Falcon.fly();  // '鷹が大空を飛びました

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

## union型（合併型）・Intersection型（交差型）

**合併型**
複数の型を指定したい時に使用する。
型Aか型Bのどちらかの型を持つ

**交差型**
型Aと型B両方の型を持つ
交差型は「AとBに共通する型」ではない。

```tsx
//合併型を簡素に使用した場合
let unionType: number | string = 10;


type Knight = {
	hp: number
	sp: number
	weapon: string
	swordSkill: string
}

type Wizard = {
	hp: number
	mp: number
	weapon: sting
	magicSkill: string
}

// 合併型...KnightまたはWizardの型を持つ。
type Adventurer = Knight | Wizard

// 交差型... KnightかつWizardの型を持つ
type Paladin = Knight & Wizard

//合併型の場合ここにmpが追加されてもエラーにならない。
//指定された型があってもなくても良い。
const adventure1: Adventure = {
	hp: 100,
	sp: 30,
	//mp: 30,
	weapon: '木の剣',
	swordSkill: '三連斬り',
}

//Wizard寄りの冒険者
const adventure2: Adventurer = {
	hp: 100,
	mp: 30,
	weapon: '木の枝',
	magicSkill: 'ファイアボール'
}

//交差型は合併したものが全て揃ってないとだめ。
const paladin: Paladin = {
	hp: 300,
	sp: 100,
	mp: 100,//一つでも型が欠けるとエラーになる。
	weapon: '銀の剣',
	swordSkill: '日輪',
	magicSkill: 'アルティメットキャノン'
}


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

## 型エイリアス
- typeを使って、型に名前を付けて宣言できる。
- 同じ型を何度も定義する必要がない。（再利用性が高い）
- 型に名前を付けることで変数の役割を明確化できる。
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

type Country = {
	capital: string
	language: string
	name: string
}

const japan: Country = {
	capital: 'Tokyo',
	language: 'Japanese',
	name: 'Japan'
}

```

## Interface・Type Alias 使い方
2021年時点でマイクロソフトはInterfaceの方が拡張しやすいという。理由から推奨している。

### Interface
- interface宣言子で定義する。
- Type Aliasと違って「=」は不要（一種のクラスなので=がいらない）
```tsx
interface Bread {
	calories: number
}
```
- 同名のinterfaceを宣言すると型が追加（マージ）される後から型を追加定義できる。
- 宣言のマージ：同じ名前を共有する複数の宣言を自動的に結合
```tsx
interface Bread {
	type: string
}

const francePan: Bread = {
	calories: 350,
	type: 'hard'
}

```
実際に色々、継承してみる。
```tsx
interface Bread {
	calories: number
}

nterface Bread {
	type: string
}

const francePan: Bread = {
	calories: 350,
	type: 'hard'
}

//上記のinterfaceを型エイリアスで表現
type MaboDofu = {
	calories: number
	spicyLevel: number
}
type Rice = {
	calories: number
	gram: number
}

type MaboDon = MaboDon & Rice // 交差型（Intersection）

const maboDon: MaboDon = {
	calories: 500,
	spicyLevel: 10,
	gram: 350
} 
```

### Interfaceの拡張
- extendsを使う事で継承したサブインターフェースを作れる。
- Type Aliasをextendsすることも出来る。
```tsx
interface Book {
	page: number
	title: string
}

interface Magazine extends Book {
	cycle: 'daily' | 'weekly' | 'monthly' | 'yearly'
}

const jump: Magazine = {
	cycle: 'weekly',
	oage: 300,
	title: '週刊少年ジャンプ'
}

//type Aliasを継承してみる。
type BookType = {
	page: number
	title: string
}

interface Handbook extends BookType {
	theme: string
}

//最後はInterfaceの型で変数を作成する。
const nijiiro: HandBook = {
	page: 138,
	title: 'にじいろ',
	theme: '旅行'
}
```

### Interfaceでclassに型を定義する。
- implements（実装する）を使ってclassに型を定義する。
```tsx
interface Book {
	page: number
	title: string
}
//Bookを実装するComicと読めば良い。
//Bookを実装するComicは必ずpage、titleというプロパティを持つ
class Comic implements Book {
	page: number;
	title: string;
	constructor(page: number, title: string, private publishYear: string) {
		this.page = page
		this.title = title
	}
	getPublishYear() {
		return this.title + "が発売されたのは" + this.publishYear + "年です。"
	}
}


const popularComic = new Comic(200, '鬼滅の刃', 2016)

```

### Type AliasとInterfaceの違い

**Type Alias**
用途：複数の場所で再利用する型に名前を付ける。
拡張性：同名のtypeを宣言するとエラー
継承：継承はできない交差型で新しい型エイリアスを作る
使用できる型：オブジェクト関数以外のプリミティブ、配列、タプルも宣言可能
考慮事項：拡張しにくい不便さがある。
いつ使う：アプリ開発ではType Alias

**Interface**
用途：オブジェクト・クラス・関数の構造を定義するため
拡張性：同名のinterfaceを宣言するとマージされる（宣言のマージ）
継承：extendsによる継承ができる
使用できる型：オブジェクトと関数の型のみ宣言できる
考慮事項：拡張できることによりバグを生む可能性
いつ使う：ライブラリ開発ではInterface


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

[【日本一わかりやすいTypeScript入門】型エイリアス(type)でオブジェクトの型定義](https://www.youtube.com/watch?v=2DoYdw-rvL0&list=PLX8Rsrpnn3IW0REXnTWQp79mxCvHkIrad&index=6)

[【日本一わかりやすいTypeScript入門】ハンズオンで理解するInterfaceとType Aliasの違い](https://www.youtube.com/watch?v=J2vox52T4W8&list=PLX8Rsrpnn3IW0REXnTWQp79mxCvHkIrad&index=10)