---
title: "tsconfig.json オプション入門" # 記事のタイトル
emoji: "🧐" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["typescript", "初心者", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.04.09'
---

## tsconfig.jsonファイルを作成する。

```bash
../node_modules/.bin/tsc --init
```

これでルートディレクトリに作成される。

## Watchモード

tsファイルの変更を監視して即座にjsファイルに変換してくれる。 なので毎回 `tsc` コマンドを入力する必要がなくなる。

```bash
tsc index.ts -w　または --watch
```

終了は `ctr + c` 

## tsファイルをまとめてコンパイル

tsconfig.jsonを作成する。

```bash
tsc --init
```

これをすると `tsc` とコマンド入力するだけで、まとめてコンパイルできる。

そのまま、watchモードも使用できる。

```bash
tsc -w
```

## tsconfig.json

オプションや設定をみていく。

## exclude

特定のtsファイルを除く。こうすると `compiler.ts` ファイルはコンパイラされない。

```json
"exclude": [
	"compiler.ts"
]
```

条件を絞る事もできる。 `ファイル名.spec.ts`  と付くファイルをコンパイルから外す。 `*` ワイルドカードが使用できる。

```json
"exclude": [
	"compiler.ts"
	"*.spec.ts"
]
```

特定のフォルダ下のコンパイルを除く。

```json
"exclude": [
	"**/*.ts"
]
```

よく使用されるのは `node_modules` を取り除く。（excludeを記述しなければデフォルトで取り除く設定になっている。）

```json
"exclude": [
	"node_modules"
]
```

### include

逆の `include: []` もある。（記述しない場合はデフォルトでは全てになっている。）

これを設定すると `include` されてたい他のtsファイルは `exclude` に入ってなくてもコンパイルされないようになる。

```json
"include": [
	"index.ts"
]
```

`include` よりも `exclude` の方が優先される。この場合 `index.ts` はコンパイルされない。

```json
"include": [
	"index.ts"
],
"exclude": [
	"index.ts"
]

```

### files

`files` はファイルを絶対パスで指定する。ワイルドカードは使えない。 `include` みたいなもの。さらに `exclude` に入ってるファイルも指定するとコンパイルされるようになる。

`files > exclude > include` みたいな関係になる。

```json
"files": [
	"tmp/compiler.ts"
]
```

## コンパイラオプション

`tsconfig.json` の `compileOptions` と書かれた各設定をみていく。

### target

TypeScriptがコンパイルするJavaScriptのバージョンを指定する。デフォルトではES3になっている。

### lib

TypeScriptが用意した型の定義を指定して、それを元にコンパイルする。JavaScriptで組み込まれている。 `.toUpperCase()` などの型を定義したファイルがあり、それをコンパイル時に読み込ませているため元々、使用出来る関数等の型定義しなくてもコンパイル時にエラーが出なくなる。

`"lib": []` のように空で指定するとエラーになる。コメントアウトで何も指定しなくすると `target` の内容に合わせて自動で指定してくれる。

```json
{
	"compilerOptions": {
		"target": "es6",
		"module": "commonjs",
		"lib": [// コメントアウトでも内部で下記の型定義を読み込んでくれる。
			"ES6",
			"DOM",
			"DOM.Iterable",
			"ScriptHost"
		],//targetがes6なので指定しても指定しなくても同じ意味になる。
	//"lib": [
	//		"ES6",
	//		"DOM",
	//		"DOM.Iterable",
	//		"ScriptHost"
	//	]
	
	}
}
```

### allowJs

JavaScriptもコンパイルする対象とする。使い所はよく分からない。

### checkJs

allowJsと一緒に使ってJSファイルのエラーもチェックしてくれる。

### JSX

Reactで使用する。

### declaration・declarationMap

型定義ファイルを作成する。 `.d.ts` これを作成する。コンパイルされたJSファイルのドキュメントのような形で使用する。

### sourceMap

TypeScriptをChromeの検証に読み込ませる事ができる。 `ファイル名.js.map` が出来る。

### outDir

```json
"outDir": "./dist" //こうするとコンパイルしたJSファイルがdistフォルダに格納される。
```

例えばtsファイルがsrc等のフォルダにあったとしても、distフォルダにはコンパイルされたJSファイルのみが入る。しかし `./src/tmp/index.ts` と `./hello.ts` のように別々の箇所にファイルがある場合は `./dist/src/tmp/index.js` と `./dist/src/hello.js` になる。システム側で効率の良い構成で保存してくれる。

### rootDir

上記の場合にフォルダも一緒にして欲しい場合こちらを有効にする。ただし、tsファイル指定した階層よりも上にあるとエラーになる。

### removeComments

tsファイル書かれたコメントをコンパイル時にJSファイルから取り除く。

### noEmit

コンパイルはせずに型チェックのみを行う。ファイルを出力しない。

### importHelpers・downlevelIteration

`target` がES5とES3のみ使用出来る。for-of をコンパイルする際に出るエラーを防ぐ。

### noEmitOnError

エラーが起きたらJSファイルにコンパイルしないようにする。

### strict

これを `true` にすると下記のコメントアウトも自動で `true` になる。

```json
"strict": true,                                 /* Enable all strict type-checking options. */
// "noImplicitAny": true,                       /* Raise error on expressions and declarations with an implied 'any' type. */
// "strictNullChecks": true,                    /* Enable strict null checks. */
// "strictFunctionTypes": true,                 /* Enable strict checking of function types. */
// "strictBindCallApply": true,                 /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
// "strictPropertyInitialization": true,        /* Enable strict checking of property initialization in classes. */
// "noImplicitThis": true,                      /* Raise error on 'this' expressions with an implied 'any' type. */
// "alwaysStrict": true,
```

### noImplicitAny

暗黙的なanyを避ける。型を指定せずに型推論にanyになるとエラーになる。変数の場合は値が入れば型推論で型がわかるのでエラーにならない。

### strictNullChecks

null・undefinedを別の型に入れようとするとエラーになる。

### strictFunctionTypes

クラスの継承時のバグを減らす。

### strictBindCallApply

bind、call、apply

持ってない機能を取り込んで使用する事が出来る。

```tsx
// ペンギンくん
const Penguin : {
    name: string;
} = {
    name: 'ペンギン',
};

// 鷹
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

//第一引数にthisにしたい値を入れて使用する。
// this.nameがPenguin.nameを参照するようになる。
Falcon.fly.call(Penguin); //ペンギンが大空を飛びました。
const flyPenguin = Falcon.fly.bind(Penguin);
flyPenguin(); //ペンギンが大空を飛びました。

// 貰った引数を表示出来るように、flyメソッドを上書きする。
Falcon.fly = function(aComment){
    console.log(this.name + 'が' + aComment + '大空を飛びました');
}

// call
Falcon.fly.call(Penguin, '元気よく');
// ペンギンが元気よく大空を飛びました

// bind
var flyPenguin = Falcon.fly.bind(Penguin, '元気よく');
flyPenguin();
// ペンギンが元気よく大空を飛びました

//受け取れる。引数の数を増やすためさらにメソッドを上書きする。
Falcon.fly = function(comment1, comment2){
    console.log(this.name + 'が' + comment1 + comment2 + '大空を飛びました');
}

//call
Falcon.fly.call(Penguin, '思い切って', '元気よく');
// ペンギンが思い切って元気よく大空を飛びました

// applyの場合は配列で渡す。
Falcon.fly.apply(Penguin, ['思い切って', '元気よく']);
// ペンギンが思い切って元気よく大空を飛びました
```

bind, call, applyの引数を監視する。引数の数、型等に間違いがあればエラーを出す。

### strictPropertyInitialization

クラスを使用する際に使う。

### noImplicitThis

thisが暗黙的にanyを指したり、何を指定してるか分からない際にエラーを起こす。

### alwaysStrict

JSファイルにコンパイルした時に `"use strict"` を使用する。

### Additional Checks

主にこの4つでコードの品質を保つ。

```json
// "noUnusedLocals": true,                      /* Report errors on unused locals. */
// "noUnusedParameters": true,                  /* Report errors on unused parameters. */
// "noImplicitReturns": true,                   /* Report error when not all code paths in function return a value. */
// "noFallthroughCasesInSwitch": true,          /* Report errors for fallthrough cases in switch statement. */
```

### noUnusedLocals

使ってないローカル変数はダメだという。

### noUnsuedParameters

関数の引数で取る予定なのに使われない場合にエラーになる。

### noImplicitReturns

暗黙的な `return` はだめです。下記の場合 `false` だと `return` が実行されないのでそれはダメですよとエラーになる。

```tsx
function echo(message: string): string | undefined {
	if (message) {
		return mesage;
	}
}
```

こうする必要がある。

```tsx
function echo(message: string): string | undefined {
	if (message) {
		return mesage;
	}
	return;
}
```

### noFallthroughCasesInSwitch

コードを綺麗にする。

### Experimental Options

将来JavaScriptに追加されるかもしれない機能を実験的に使えるようにする。

### 参照

[](https://qiita.com/39_isao/items/c00a200b158ba057363f)

[超TypeScript入門完全パック- TypeScriptでアプリを作りたい方必見！](https://www.youtube.com/watch?v=F9vzRz6jyRk)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。