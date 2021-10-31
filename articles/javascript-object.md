---
title: "JavaScriptでオブジェクト指向の概要を掴む" # 記事のタイトル
emoji: "🤝" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["javascript", "初心者", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.10.31'
---

## 最初に

関数を並べて作成するスクリプトを書きながら限界を感じたので、オブジェクト指向について勉強して複雑なプログラムも綺麗にかつ簡単に作成したいと思う。

個人的に思うのは難しい技術ほど最初は取っ付きにくいがプログラムが複雑になるにつれて、その恩恵が大きくなると考えている。

関数を並べるだけのプログラムは最初は簡単で取っ付きやすいがプログラムが大きくなるに連れて状態管理等が必要になり苦しくなってくると思う。最初に苦労して後で楽するか、最初に楽をして苦労するかだと思う。その辺を考慮しながら、技術選定をするのは結構重要な考え方な気がする。

## シンボルプロパティ

`symbol()` を使って作成されたデータは6種類ある（文字列、数値、BigInt、真偽値、undefined、シンボル）プリミティブデータ型と呼ばれるものの一つであり、シンボルと呼ばれている。

すべてのプリミティブ値は、イミュータブルimmutable（変更出来ない値を指す）で変更できません。

下記ではオブジェクトの列挙方法をいくつか紹介しながら、シンボルプロパティの動作を確認していく。

使い方

```jsx
// シンボルから返される値は一意である。
let sym1 = Symbol()
let sym2 = Symbol('foo')
let sym3 = Symbol('foo')

Symbol('foo') === Symbol('foo')  // false
```

```jsx
const SYM = Symbol()
const o = {a: 1, b:2, c:3, [SYM]: 4}
for (let prop in o) {
  // for inはfor ofと違い順番が担保されてないので最初にどれが来るか予測出来ない。
  if(!o.hasOwnProperty(prop)) continue
  console.log(`${prop}: ${o[prop]}`)
}

// 実行結果
a: 1
b: 2
c: 3
```

実際には `if(!o.hasOwnProperty(prop)) continue` は省略できる。

しかし `for...in` を使用する際は `if(!o.hasOwnProperty(prop)) continue` を使うことを習慣づけておいた方が良い。 `in` がプロトタイプチェーンを遡ってプロパティを列挙

オブジェクトは「継承」によってプロパティを持つ場合があるので

### for...in, for...of違い

for...inは主にオブジェクトのループ専用として使われる。変数に入るのはオブジェクトのキーで順番は通りではない。値を取り出したい場合は変数に入った値を使用する。

一応、配列にも使用出来て、その場合は変数にindex番号が入る。 `0, 1, 2, 3` のような値が入る。ただ順序を保証していないので配列のループには不向きである。

そして下記のような使いにくい場面もある。

これは `in` がプロトタイプチェーンを遡るためである。

個人的にはなぜこんな仕様になっているのか不思議だ、オブジェクトにないプロパティまで出力するせいで使いにくい。

[in と hasOwnProperty() の違い - Qiita](https://qiita.com/shuhei/items/dabf0ca097f05264baf9)

```jsx
Object.prototype.hoge = 'hogeValue';
Array.prototype.fuga = 'fugaValue';

const obj = {key1: 'value1', key2: 'value2', key3: 'value3'};

for(let o in obj) {
    console.log(o);
}

const array = ['value1', 'value2', 'value3'];

for(let a in array) {
    console.log(a);
}

// 実行結果
key1 
key2 
key3 
hoge // 拡張で追加したhogeプロパティまで出力されている
0 
1 
2 
fuga // arrayも同様に
hoge // そしてObjectで追加したhogeプロパティはarrayに継承されるので出力される
```

これを回避する方法がある。

`obj.hasOwnProperty(o)` を使用してobj自体がoプロパティを持っているか確認してあれば `True` を返す。 

※プロパティはkeyと値が対になったものを指す 

プロパティ
`{key: value}`
これが複数にあるのをオブジェクト

オブジェクト
`{key: value, key2: value2...}`

`obj.hasOwnProperty(o)` はプロトタイプチェーンまで遡ることはないので `hoge`, `fuga` で `True` になる事はない。

```jsx
Object.prototype.hoge = 'hogeValue';
Array.prototype.fuga = 'fugaValue';

const obj = {key1: 'value1', key2: 'value2', key3: 'value3'};

for(let o in obj) {
    if (obj.hasOwnProperty(o)) {
        console.log(o);
    }
}

const array = ['value1', 'value2', 'value3'];

for(let a in array) {
    if (array.hasOwnProperty(a)) {
        console.log(a);
    }
}

// 実行結果
key1
key2
key3
0
1
2
```

もう一つは `defineProperty` を使用すると `prototype` の汚染を回避する事ができる。

[prototype汚染しないようにObject.prototypeを拡張する - Qiita](https://qiita.com/kakusuke/items/d4f7f3d45f85eaef6fda)

### イミュータブル とconst違い

```jsx
const obj = {foo: 0}; // constである変数を作る
obj.foo = 42; // {foo: 0}はミュータブルなのでfooを変更できる
obj = {bar: 0}; // error! objへ再代入はできない！

let immutableObj = new Immutable({foo: 0}); // Immutableオブジェクトを生成してimmutableObjへ代入する。イミュータブルなオブジェクトはImmutable.jsなどのライブラリで生成できます
immutableObj.set('foo', 42); // immutableオブジェクトは変更できない
immutableObj.get('foo'); // fooは42ではなく0のまま
immutableObj = immutableObj.set('foo', 42); // 再代入することで変更できる
```

## Object.keys

オブジェクトのプロパティにあるkeyを配列で取得する事ができる。

これを使ってオブジェクトを列挙する事ができる。

```jsx
const SYM = Symbol()
const o = {a: 1, b: 2, c: 3, [SYM]: 4}
const propArray = Object.keys(o)

propArray.forEach(prop => console.log(`${prop}: ${o[prop]}`))

// 実行結果
a: 1
b: 2
c: 3

```

## クラスとインスタンス生成

```jsx
class Car {
  constructor() {
  }
}

const car1 = new Car()
const car2 = new Car()

console.log(car1 instanceof Car) // true
console.log(car2 instanceof Car) // true
console.log(car1 instanceof Array) // false
```

### メソッドを追加してみる

シフトを変更するメソッドを追加する。エラー文を挟む事で無効なギアが設定出来ないようにしている。

```jsx
class Car {
  constructor(make, model) {
    this.make = make
    this.model = model
    this.userGears = ['P', 'N', 'R', 'D']
    this.userGear = this.userGears[0]
  }
  shift(gear) {
    if(this.userGears.indexOf(gear) < 0)
      throw new Error(`ギヤ指定が正しくない: ${gear}`)
    this.userGear = gear
  }
}

const car1 = new Car("Tesla", "Model S")
console.log(car1)
console.log(car1.make)

const car２ = new Car("Mazda", "3i")
console.log(car2)
console.log(car2.make)
```

上記の状態ではメソッドを使用しなくても `car1.userGear = 'X'` で変更可能となっており、アクセス制御がなく予期しないデータの変更をされてしまう可能性があり良くない。それを解決する方法として「アクセスプロパティ」を利用する。 `get, set` をセットで使用する。後で詳しくみていく。今はこれらがメンバ変数を操作してるという理解で大丈夫だと思う。 `_` を変数の前に付けるだけでこれが `get, set` 以外で直接参照されているのはおかしい事を教えてくれるに過ぎず機能を制限出来るわけでない事に注意が必要である。

```jsx
class Car {
  constructor(make, model) {
    this.make = make
    this.model = model
    this._userGears = ['P', 'N', 'R', 'D']
    this._userGear = this._userGears[0]
  }
  get userGear() { return this._userGear; }
  set userGear(value) {
    if(this.userGears.indexOf(gear) < 0)
      throw new Error(`ギヤ指定が正しくない: ${gear}`)
    this.userGear = gear
      
  }
  
  shift(gear) { this.userGear = gear; }
}

const car1 = new Car("Tesla", "Model S")
console.log(car1)
console.log(car1.userGear)

// 実行結果
Car {
  make: 'Tesla',
  model: 'Model S',
  _userGears: [ 'P', 'N', 'R', 'D' ],
  _userGear: 'P'
}
P
```

これを防ぐ方法として WeekMapを使用した関数をクロージャに隠す事で外から関数内の変数にアクセスして直接値を変更する事を防ぐ事が出来る。

```jsx
const Car =  (function() {
  const carProps = new WeakMap();
  class Car {
	  constructor(make, model) {
      this.make = make
      this.model = model
      this._userGears = ['P', 'N', 'R', 'D']
      carProps.set(this, { userGear: this._userGears[0] })
    }
    get userGear() { return carProps.get(this).userGear; }
    set userGear(value) {
      if(this._userGears.indexOf(value) < 0)
        throw new Error(`ギヤ指定が正しくない: ${value}`)
      carProps.get(this).userGear = value;
    }

    shift(gear) { this.userGear = gear; }
  }
  return Car;
})();
const car1 = new Car("Tesla", "Model S")
console.log(car1)

// 実行結果
Car {make: 'Tesla', model: 'Model S', _userGears: Array(4)}make: "Tesla"model: "Model S"_userGears: (4) ['P', 'N', 'R', 'D']userGear: (...)[[Prototype]]: Object
```

外からは `car1.userGear = 'R'` で値を変更出来なくなる。

## プロトタイプ

JavaScriptはインスタンス（Classをnewして生成されたオブジェクト）に対して使えるメソッドを参照するとき、プロトタイプメソッドを参照している。 `Car.prototype.shift` のように書かれる。 同様にArrayの関数はforEachは `Array.prototype.forEach` と書かれる。普段はこのような書き方ではなく `array.forEach((a) => {a})` みたいな感じで間が省略されている。他にも `#` を使って表すことも出来る。 `Car#shift` と書く事も出来る。

プロトタイプはクラスメソッドのみでなく、通常の関数もprototypeと呼ばれる特別なプロパティを持っている。関数の場合はあまり意識しなくて良さそう。

プロトタイプで重要なのは「動的ディスパッチ」というメカニズムでオブジェクトのプロパティ（メソッド）にアクセスしてそれが存在しない場合、そこでエラーを直ぐに返すのではなく、そのオブジェクトのプロトタイプを見て同じプロパティがないか確認する。例クラスCarが生成したインスタンスは全て同じプロトタイプを共有してるから、同じメソッドが使用できるという当たり前の事を小難しく言っている。

ではなんのために動的ディスパッチを出したのかと言うと、インスタンスを生成した後にそのインスタンスに同じ名前の関数を定義する事でプロトタイプにあるメソッドをそのインスタンスのみだが上書きして変更する事が可能である説明のために少しだけ登場させた。

実際にインスタンスに直接メソッドを定義してプロトタイプチェーンを上書きしたのが下記のコードになる。

```jsx
const Car =  (function() {
  const carProps = new WeakMap();
  class Car {
    constructor(make, model) {
      this.make = make
      this.model = model
      this._userGears = ['P', 'N', 'R', 'D']
      carProps.set(this, { userGear: this._userGears[0] })
    }
    get userGear() { return carProps.get(this).userGear; }
    set userGear(value) {
      if(this._userGears.indexOf(value) < 0)
        throw new Error(`ギヤ指定が正しくない: ${value}`)
      carProps.get(this).userGear = value;
    }
    shift(gear) { this.userGear = gear; }
  }
  return Car;
})();

const car1 = new Car()
const car2 = new Car()
console.log(car1.shift === Car.prototype.shift)
console.log(car1.shift === car2.shift)
car1.shift('D')
console.log(car1.userGear)

// メソッドを小文字でも大文字に変換するように上書きした
car1.shift = function(gear) { this.userGear = gear.toUpperCase() }
// インスタンスに直接定義したものだから、prototypeメソッド
// ではないと出力される。
console.log(car1.shift === Car.prototype.shift)
console.log(car1.shift === car2.shift)
car1.shift('d')
console.log(car1.userGear)

// 実行結果
true
true
D
false
false
D
```

## 静的メソッド

インスタンスに関わらない処理を行う際に使用されるらしい。ただ普通のメソッドとと動作が違うのかよく分からない。試しに `static` を外して `this.nextVin++` とクラスメソッドの様に変更しても動作は一緒になる。ただ下の `areSimilar, areSame` はクラスから直接呼び出しているので通常のメソッド定義だと実行出来なくなるので、少なくとも静的メソッドを使用する意味はありそうだ。個人的に感心したのは `Car.nextVin = 0` この行でインスタンス間で共有する変数を持たせることができるのに驚いた。

これは状態を持たせるの使える気がする。関数のみでやってた際は状態を管理するのが大変だったのでこれはとても使えそうな気がする。

```jsx
class Car {
    static getNextVin() {
        return Car.nextVin++
    }
    constructor(make, model) {
        this.make = make
        this.model = model
        this.vin = Car.getNextVin()
    }
    static areSimilar(car1, car2) {
        return car1.make === car2.make && car1.model === car2.model
    }

    static areSame(car1, car2) {
        return car1.vin === car2.vin
    }
}

// オブジェクトとして持たせる事ができる。
// しかもこの値はクラス間で共有できる。
// 状態管理に使える。
Car.nextVin = 0
Car.nextVin1 = {"hi": 0}

const car1 = new Car("Tesla", "Model S")
const car2 = new Car("Mazda", "3i")
const car3 = new Car("Mazda", "3i")

console.log(car1.vin)
console.log(car2.vin)
console.log(car3.vin)

console.log(Car)
```

## 継承

クラスを継承して下位のクラスにメソッドを追加する。

```jsx
class Vehicle {
    constructor() {
        this.passengers = []
        console.log("Vehicleが生成された")
    }
    addPassenger(p) {
        this.passengers.push(p)
    }
}

class Car extends Vehicle {
    constructor() {
        super()
        console.log("Carが生成された")
    }
    depoyAirbags() {
        console.log("バーンッ!")
    }
}

const v = new Vehicle()
v.addPassenger("太郎")
v.addPassenger("花子")
console.log(v.passengers)
// v.depoyAirbags() エラーになる

const c = new Car()
c.addPassenger("恵子")
c.addPassenger("順子")

console.log(c.passengers)
c.depoyAirbags()

```

## ポリモーフィズム

あるインスタンスをそのインスタンスが属するクラスのメンバーとして扱うだけではなく、スーパクラスのメンバーとして扱う。おそらく下記のコードから推測するに継承先のクラスでもある事を言いたいだけだと思う。インスタンス `c` は `Car` のクラスメンバに属するし `Vehicle` のクラスメンバにも属する。それだけの事を小難しく言っている。

```jsx
class Vehicle {
    constructor() {
        this.passengers = []
        console.log("Vehicleが生成された")
    }
    addPassenger(p) {
        this.passengers.push(p)
    }
}

class Car extends Vehicle {
    constructor() {
        super()
        console.log("Carが生成された")
    }
    depoyAirbags() {
        console.log("バーンッ!")
    }
}

class Mortrcycle extends Vehicle {}

const v = new Vehicle()
v.addPassenger("太郎")
v.addPassenger("花子")

const c = new Car()
c.addPassenger("恵子")
c.addPassenger("順子")

console.log(c.passengers)
c.depoyAirbags()

const c2 = new Car()
const m = new Mortrcycle()

console.log(c instanceof Car)
console.log(c instanceof Vehicle)
console.log(m instanceof Car)
console.log(m instanceof Mortrcycle)
console.log(m instanceof Vehicle)

console.log(c instanceof Object)
// 文字列もObjectを継承してるのか気になった。
// ただの文字列の場合はStringオブジェクトを継承しない。
// const s = "hrl"
const s = new String("hrl");
console.log(s instanceof Object)
// Strring オブジェクトも継承してない...
console.log(s instanceof String)
console.log(typeof s)
console.log(s.valueOf())
console.log(s instanceof Object)
```

## プロパティの列挙再び

プロトタイプチェーン上にあるプロパティにも `for...in` がアクセスしてしまう事を確認する。

```jsx
class Super {
    constructor() {
        this.name = 'Super'
        this.isSuper = true
    }
}

Super.prototype.sneaky = '非推奨!'

class Sub extends Super {
    constructor() {
        super()
        this.name = 'Sub'
        this.isSub = true
    }
}

const obj = new Sub()

for(let p in obj) {
    console.log(`${p}: ${obj[p]}` + 
                (obj.hasOwnProperty(p)? '' : ' (継承)'))
}
```

## 多重継承、ミックスイン、インタフェース

オブジェクト指向では「多重継承」と呼ばれる機構をサポートしている。

あるクラスが複数のスーパクラスを継承している状態である。これはメソッド名の衝突などを起こす危険性があり、その場合どちらを優先するのかも不明である。そのため多くの言語では多重継承を許していない。

しかし世の中の問題を考えた場合、多重継承が自然な場面も見られる。車は乗り物で保険をかけられる。さらに家は乗り物ではないが、保険をかけられる。こうしたケースにも対応できるように「インタフェース」と言う機構が用意されている。 `Car` クラスはスーパクラスとして `Vehicle` しか継承出来ないけれど、複数のインタフェース（ `Insurable`, `Container` ）を持つことができる。

JavaScriptでは「ミックスイン」と言う概念が使われており、それを使用する事で多重継承の問題を解決することができる。

下記は上手く動作するが、インスタンスを作成するたびにそれを `makeInsurable` 関数に渡さなければならないのが冗長である。

```jsx
class Car {
    constructor() {
        
    }
}

class InsurancePolicy {}

function makeInsurable(o) {
    o.addInsurancePolicy = function(p) { this.insurancePolicy = p }
    o.getInsurancePolicy = function() { return this.insurancePolicy }
    o.isInsured = function() { return !!this.insurancePolicy }
}

// makeInsurable(Car) とCarクラスに保険機能を追加しようとするとエラーになる
// const car1 = new Car()
// car1.addInsurancePolicy(new InsurancePolicy())

const car1 = new Car()
makeInsurable(car1)
console.log(car1.isInsured())
car1.addInsurancePolicy(new InsurancePolicy())
console.log(car1.isInsured())
```

これを解決するのに Carクラスのプロトタイプを渡してそこに `makeInsurable` のメソッドを追加する。

```jsx
makeInsurable(Car.prototype)

const car1 = new Car()
console.log(car1.isInsured())
car1.addInsurancePolicy(new InsurancePolicy())
console.log(car1.isInsured())
```

これで新しく作成したメソッドがいつもクラスCarの一部であるかのごとく動作する。JavaScriptから見るとこのメソッドはクラスCarの一部になる。コード上ではCarと保険に関する機能は分割されているので個別で管理する事ができる。ただ保険担当グループが `shift` メソッドを作成したりし始めたりすると衝突を起こすので注意が必要である。そして保険を掛けられるオブジェクトのに抽出したい場合 `instanceOf` をしてもCarクラスとしてしか認識されないので不便になることがある。

シンボルを使用するとCar機能と保険機能の衝突を防ぐ事ができる。

```jsx
class Car {
    constructor() {
    }
}

class InsurancePolicy {}

const ADD_POLICY = Symbol()
const GET_POLICY = Symbol()
const IS_INSURED = Symbol()
const _POLICY = Symbol()

function makeInsurable(o) {
    o[ADD_POLICY] = function(p) { this.insurancePolicy = p }
    o[GET_POLICY] = function() { return this.insurancePolicy }
    o[IS_INSURED] = function() { return !!this.insurancePolicy }
}

// makeInsurable(Car) とCarクラスに保険機能を追加しようとするとエラーになる
// const car1 = new Car()
// car1.addInsurancePolicy(new InsurancePolicy())

const car1 = new Car()
makeInsurable(car1)
console.log(car1[IS_INSURED]())
car1[ADD_POLICY](new InsurancePolicy())
console.log(car1[IS_INSURED]())
```

## 最後に

JavaScriptでのオブジェクト指向について少し概要をつかむ事が出来た。関数のみで作成するスクリプトより状態を管理するのが楽な印象を受けた。今後はこれをきっかけにより具体的なデザインパターンや設計方法を学んでいけたら良いと思う。

### 参照

1)Ethan Brown. Learning JavaScript, 3rd Edition. O'Reilly. イーサン ブラウン ムシャ ヒロユキ ムシャ ルミ (訳) 2017. 「9章 オブジェクトとオブジェクト指向プログラミング」.『初めてのJavascript』. 第3版. オライリージャパン. pp 161-182.

[immutableとconstの違いがわかりやすいコードを考えた - Panda Noir](https://www.pandanoir.info/entry/2016/06/18/185302)

[【javaScript】for...in、for...of、forEachの違いと用途](https://web-begginer-log.com/js_for/)

[.prototype.hasOwnProperty() | JavaScript 日本語リファレンス | js STUDIO](https://js.studio-kingdom.com/javascript/object/has_own_property)

[for...inとfor...ofの違い - Qiita](https://qiita.com/a05kk/items/d6f49ca5bd15f045ea6c)

[jsでのプロパティの存在チェック方法をまとめてみる - Qiita](https://qiita.com/rymiyamoto/items/be91b04f70de2b621bb3)

[Symbol - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。