---
title: "JavaScriptのイテレータとジェネレータの使い方" # 記事のタイトル
emoji: "✅" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["javascript", "初心者", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## イテレータ

`for...of` 等で値を一つ一つ取り出せるオブジェクトを指す（反復可能なオブジェクト：配列、文字列等を指す）。イテレータの意味は2つあって「反復可能なオブジェクト」、「イテレータオブジェクト」通常は後者をイテレータと指す。

### イテレータオブジェクト

nextメソッドを持っており、このメソッドを呼び出す事で次々と要素を取り出せるオブジェクトの事を指す。

`next()` を呼び出すと返り値として 配列の値 `value` と `done` を返す。

`done` の `true, false` を見て配列にまだ値があるかを確認する事が出来る。

### 反復可能なオブジェクト

`[Symbol.iterator]` メソッドを実装する事でイテレータオブジェクトを返す。上記の事を実装する事で反復可能なオブジェクトを作る事が出来る。

## 配列をイテレータオブジェクトに変換する。

`配列.values()` とする事で配列をイテレータオブジェクトに変換出来る。しかし2016年12月時点ではchrome等には実装されていない。

```jsx
const book =[
	"hi",
	"jiro"
];

cosnt it = book.values();
console.log(it.next());
console.log(it.next());
console.log(it.next());

//実行結果
Object { value:"hi", done: false }
Object { value:"jiro", done: false }
Object { value:undefined, done: true }

//while文での実行
const book1 =[
	"hi",
	"taro"
];
cosnt it1 = book1.values();
let current = it1.next();
while(!current.done) {
	console.log(current.value);
	current = it.next();
}
Object { value:"hi", done: false }
Object { value:"taro", done: false }

```

## イテレータプロトコル

反復可能である配列はメソッドvaluesを使う事でイテレータに変換出来るが `values()` が何を行なっているのかイテレータであるために必要な「イテレータプロトコル」を実装してこれがどのようなものか見ていく。

```jsx
class Log {
	constructor() {
		this.messages = [];
	}
	add(messages) {
		const now = Data.now();
		console.log(`ログ追加: ${message} (${now})`);
		this.messages.push({ message, timestamp: now })
	}
	[Symbol.iterator]() {
		//イテレータオブジェクトに変換して返す。
		return this.messages[Symbol.iterator]();
	}
}

const log = new Log();
log.add("海の監視初日。勤務開始");
setTimeout(function() {log.add("クジラを見た");}, 3*1000);
setTimeout(function() {log.add("船を見た");}, 7*1000);
setTimeout(function() {log.add("監視終了");}, 9*1000);
setTimeout(function() {
	cosole.log(`本日の業務報告 (${new Date()})`);
	for(let entry of log) {
		const data new Data(entry.timestamp);
		console.log(`${entry.message} (${date})`);
	}
}, 10*1000);
```

## ジェネレータ

ジェネレータは普通の関数と違い呼び出された際はすぐに実行されず、まずはイテレータが戻される。そのあとイテレータのメソッドnextを呼び出すたびに実行が進む。

定義するには `function* 関数名` となる。それ以外は普通の関数と同じ構文が使用出来る。ただしアロー関数は使用できない。そして呼び出し側に値を供給する場合はキーワードyieldが使われる。 `return` も使われるが、ジェネレータ関数内では通常は値を返すために使用されない。

```jsx
function* colors() {
	yield 'あか';
	yield 'あお';
}

//呼び出し側
const it = colors();
console.log(it.next());// { value: 'あか', done: false }
console.log(it.next());// { value: 'あお', done: false }
console.log(it.next());// { value: undefined, done: true }

```

### yield式と双方向コミュニケーション

ジェネレータを使うと呼び出し側との間で双方向のコミュニケーションが可能になる。yieldは式なので、評価の結果何らかの値になる。どんな値になるかと言うと、next呼び出し時の引数になる。

上記のコードなら `yield = next()の引数が入る`  それを踏まえて下記のコードを見て欲しい。

```jsx
function* itterrogate() {
	const name = yield "お名前は?";//最初のnextではyield手前で止まる。次のnextの時にyieldに引数を返してnameに代入される。
	const color = yield "お好きな色は何ですか?";//2回目のnextはyield手前まで止まる。3回目のnextでcolorに引数が代入される。
	return `${name}さんの好きな色は${color}だそうです。`;
}

const it itterrogate();
console.log(it.next());//{ value: 'お名前は?', done: false } ここでは引数を渡しても無視される。
console.log(it.next(めぐみ));//{ value: 'お好きな色な何ですか', done: false }
console.log(it.next(あお));//{ value: 'めぐみさんの好きな色はあおだそうです。', done: true }
console.log(it.next());//{ value: undefined, done: true }
```

ジェネレータのどこかで `return` を呼び出すとdoneがtrueになり、valueプロパティはreturnに指定した値になる。これには注意が必要で `for...of` 構文で回す場合、trueになった際にループを抜けるのでvalueが出力されなくなる。なので `return` で意味のある値を渡すことはなるべく避けた方が良いと思われる。

### 参照

1)Ethan Brown. Learning JavaScript, 3rd Edition. O'Reilly. イーサン ブラウン ムシャ ヒロユキ ムシャ ルミ (訳) 2017. 「12章 イテレータとジェネレータ」.『初めてのJavascript』. 第3版. オライリージャパン. pp 197-209.

[JavaScript の イテレータ を極める！ - Qiita](https://qiita.com/kura07/items/cf168a7ea20e8c2554c6)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。