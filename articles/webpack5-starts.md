---
title: "Webpack5 入門" # 記事のタイトル
emoji: "🎈" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["javascript", "webpack", "初心者", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.08.19'
---

## 最初に

`react-create-app` を使用しないでReactを使用するためにwebpackが必要だと知ったのでまず、通常のJavaScriptでwebpackがどのように使用されるのか入門してみようと思う。node.js環境が必要なのでdockerで作成しておく。

## インストール

```bash
# package.jsonを作成する。
npm init -y 
npm i -D webpack webpack-cli
```

## JSファイルをバンドルする

その後、 `src` フォルダを作成してそこにJavaScriptファイルを `main.js` , `sub.js` 2つ作成する。

index.js

```js
import { test } from './sub'

test()
```

sub.js

```js
export function test() {
  alert('テスト実行')
}
```

作成したらルートディレクトリで `npx webpack` を実行するとjsファイルがビルドされる。
ただ警告として `Set 'mode' option to 'development' or 'production'` と出る。開発用か本番用に出力するファイルを選べと言われる。

本番用はminファイルと言われ改行がなく容量を最小にしている。
その代わりビルドに時間がかかる。

開発用はビルド時間を短くできる。

指定方法は `--mode` オプションを使用する。

```bash
# 開発用のビルド
webpack --mode development

# 本番用のビルド
webpack --mode production

```

いちいち指定するのは面倒なのでpackage.jsonにコマンドを登録する。

packkage.json

```json
// package.jsonの一部
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    // 下記が追加したコマンド
    "build": "webpack --mode development",
    "prod": "webpack --mode production"
  },
```

実行方法は

```bash
npm run build
npm run prod
```

## webpack.config.jsでwebpackの設定をする

設定ファイルを作成してwebpackの設定をする。
ファイル名は `webpack.config.js` じゃなくても `webpack.dev.js` でも良い、しかしその場合はオプション `--config` でファイルを指定する必要がある。

```js
const path = require('path')

module.exports = {
  // 作業ディレクトリがある場合はentryの書き方が変わる
  context: `${__dirname}/src`,
  // entry: './index.js',
  // joinを使用するとOSの環境に合わせてパスをいい感じに設定してくれる。
  entry: path.join(__dirname, '/src', 'index.js'),

  // defaultの設定つまりあってもなくても一緒
  // ファイルの起点を設定する。
  // entry: './src/index.js',

  // ここは絶対パスで指定する必要がある。
  output: {
    // path: `${__dirname}/dist`,
    path: path.join(__dirname, '/dist'),
    filename: 'main.js'
  }
}
```

その他設定オプション

- mode： あらかじめビルドを `development`, `production` を決めれる。
- output: { filename: }: ビルドして出力されるファイル名を設定できる。

## scssファイルにも対応させる

```bash
npm i -D sass sass-loader
```

## postcssにも対応させる

ベンダープレフィックス（-web-kit-等ブラウザ毎の若干の差異）を付与してくれる。

```bash
npm i -D postcss-loader autoprefixer
```

新しく `postcss.config.js` ファイルを作成して

```js
// 古いブラウザにも対応出来るようにベンダープレフィックスを付ける。
module.exports = {
  plugins: (
    require)'autoprefixer')
  ]
}
```

webpack.config.jsには下記のように `postcss-loader` を追加する。

```
module: {
    rules: [
      {
        test:/\.scss$/,
        use:[
          'style-loader',
          'css-loader',
          'postcss-loader',
          'sass-loader'
        ]
      },
    ]
  }
```


## ファイルを読み込む

以前は `file-loader` をインストールして `webpack.config.js` に下記の追加設定が必要だったがwebpack5からインストール不要となったので設定せずにそのままバンドル可能。
画像のURLはデフォルトではハッシュ値になるので、buildして吐き出される画像の名前が元の奴とは異なるものになる。

```js
// webpackの設定
{
  test: /\.(jpe?g|gif|png|svg|woff2?|tff|eot)$/,
  use: [
    {
      loader: 'file-loader',
      options: {
        name: '[name].[ext]',
        outputPath: 'images',
        publicPath: 'images'
      }
    }
  ]
}

```

## main.jsにバンドルせずに複数のファイルに吐き出す

まず `entry` ポイントを複数設定する。そして `output` の `filename` に `'[name].main.js'` を追加すると `app.main.js, sub.main.js` という2つのファイルが出力されるようになる。

```js
// webpack.config.jsの一部
 entry: {
    app: path.join(__dirname, '/src', 'app.js'),
    sub: path.join(__dirname, '/src', 'sub.js')
  },

  output: {
    // path: `${__dirname}/dist`,
    path: path.join(__dirname, '/dist'),
    filename: '[name].main.js'
  },
```

## ハッシュ値について

![disk-cache](https://user-images.githubusercontent.com/23703281/129139957-ed0d6a71-38b6-4d5c-a6ec-6f52caa033b7.png)

chromeブラウザの検証 > ネットワークで画像がローカルにキャッシュとして残っているか確認できる。
disk cacheと書かれている場合はローカルのストレージにファイルが保存されており、高速化出来ている。

ブラウザにはキャッシュの機能があるので画像を更新してもファイル名が同じ場合キャッシュを読み込み変更が適用されないことがある。そのためwebpackではビルドするたびにハッシュ値を変更する設定がある。

その設定は複数あり上記の `'[name].main.js'` の `[name]` 部分を `[hash]` 等に変更する事で設定できる。

- `[hash]`： ビルドした時で同じハッシュ値がそれぞれのファイルに付けられる。しかしファイルに変更がない場合は同じハッシュ値のままになる。全て更新のため初回ロードが重くなる。そのためユーザが使用するサービスでの使用はオススメしない。
- `[contenthash]`: 生成されたファイル毎にハッシュ値が付く。画像ファイルによく用いられる。
- `[chunkhash]`: import等で繋がりがあるファイル同士が同じハッシュ値になる。

## Babelを連携する

JavaScript ES6構文（アロー関数等）をES5（IE11でも対応している）に変換する。

```bash
npm i -D babel-loader @babel/core @babel/preset-env
```

webpack.config.jsに下記のbabelを読み込む設定をrulesに追加する。

```js

{
  test: /\.js$/,
  // node_modulesは対象外とする。
  exclude: /node_modules/,
  loader: 'babel-loader'
}

```

ルートディレクトリに `babel.config.js` という設定ファイルを作成する。

```js
module.exports = api => {
  // キャッシュを追加する事でビルドの時間を短縮できる。
  api.cache(true)
  return {
    "presets": [
      ["@babel/preset-env", {
        // ブラウザのバージョンが将来的に上がっていっても対応できるように下記の設定方法にする。
        // browserlist が定期的に更新されるため捕手が容易になる。
        targets: [
          "last 1 version",
          // シェアが1%以上あるブラウザのバージョン
          "> 1%",
          "maintained node versions",
          "not dead"
        ],
        // ビルドする際に core-jsで変換に使った関数だけ取り込むようにしてくれるため
        // パフォーマンスが上がる。
        // これを使用するとcore-js, regenerator-runtimeをimportしなくてよくなる。
        useBuiltIns: "usage",
        corejs: 3
      }]
    ]
  }
}
```



ES7で追加された機能（ `配列.includes()`等 ）にも対応するために `core-js` というパッケージをインストールする。

ES8で追加されたasync, await構文を使用する際はさらに `regenerator-runtime` というパッケージのインストールが必要となる。そこにES5でもasync, awaitと似た機能が実装できるように関数等がまとめられていると思われる。

```bash
# core-jsはバージョンを指定してインストールする。
npm i -D babel-loader core-js@3, regenerator-runtime
```

usage使用前
なので `app.js` に下記のimport文が必要となる。

```js
import 'regenerator-runtime'
import 'core-js'

```

![usage使用前](https://user-images.githubusercontent.com/23703281/129836919-214eefb8-4d98-48d1-a7f4-c9c8474bd360.png)


usage使用後
utils.jsの容量が減っているのが確認できる。

![usage使用後](https://user-images.githubusercontent.com/23703281/129836598-b6be7282-dfbf-4a1f-8913-b36cba696784.png)



## ESLintを入れる

```bash
# eslint本体とwebpackで使用するローダとbebelで使用するためjqueryはこの後使用するためついでにインストールしておく。
npm i -D eslint eslint-loader babel-eslint 

# jqueryはこの後使用するためついでにインストールしておく。
npm i jquery
```

ルートディレクトリに `.eslintrc` というファイルを作成して下記の設定を加える。

```plaintext

{
  // この環境に設定するとどの環境でJSが実行されるのかESLintに伝えられる。
  // それに必要なグローバル変数をESLintが考慮してくれる。
  "env": {
    "node": true,
    "browser": true,
    "es2017": true
  },
  "extends": "eslint:recommended",
  "parser": "babel-eslint",
  "parserOptions": {
    "ecmaVersion": 2017,
    "sourceType": "module"
  },
  // jQueryをimportしなくてもESlintが認識してくれる。
  "globals": {
    "jQuery": "readonly", //or writable
    "$": "readonly",
    "utils": "readonly"
  },
  // たくさんのルールがあるのでまずは extendsでeslint:recommendedを設定してそこから
  // ルールを上書きして徐々に増やしていく。
  "rules": {
    "semi": ["error", "never"]
  }
}
```

- parser: 構文を解析するプラグインを設定する。
- extends: eslintの設定を拡張するために使用する。おすすめはデフォルトの eslint:recommended
- env: jsが実行される環境を指定する事で、その環境でのグローバルオブジェクトに対しての解析を行ってくれる。これが無いとブラウザでのグローバルオブジェクトwindowを使用した際などにエラーとして出力される。例：console.logを使用するだけでエラーになる。
- globals: eslintが、その値がグローバルに存在するかしないを判断できない際に追加してエラーが起きないようにする。
- paserOptions: `ecmaVersions`これはenvに `es2017: true` 設定した際に自動で設定されるのであえて設定しなくてもよい。 `sourceType` これはjsのファイルをどのように管理しているかeslintに知らせる設定で上記の場合はmoduleで管理していると伝える。デフォルトは `script` でその場合はes6から導入されたモジュールが使えなくなる。
- rules: `extends` のルールを上書きする際に使用する。 `"semi": ["error", "never"]` 文末のセミコロンがあるとエラーになる。


webpack.config.jsにeslintの設定をrulesに追記する。

```js
{
  enforce: 'pre',// これでjsファイルの場合は先にeslintの構文解析が入るようになる。
  test:/\.js$/,
  exclude: /node_modules/,
  loader: 'eslint-loader',
  // eslintのfixオプションをonにするルールに基づいてコードを整形してくれる。
  options: {
    fix: true,
  }
}
```

- fix: eslintでエラーになった箇所で直せる部分は自動で修正してくれる。

## JSファイルにバンドルされるcssを分離する

JSファイルとCSSを分離する。cssに変更がない際はキャッシュを用いる事のでパフォーマンス向上を狙える。そのために必要なMiniCssExtractPluginをインストールする。HTTP1の時代は複数ファイルを読み込むと速度が遅くなるということがあり、なんでも一つのファイルにする風潮があったが、現在のHTTPプロトコルはHTTP2が主流であり複数ファイルを読み込んでも速度低下する事がなくキャッシュの恩師を受けた方がパフォーマンス改善になるためJSとCSSを分離する。

```bash
npm i -D MiniCssExtractPlugin
```

webpack.config.jsの `plugins` に下記の設定を追加する。

```js
//scssを読み込む箇所に style-loaderの代わりに、MiniSccExtractを追加する。
{
  test:/\.scss$/,
  use:[
    // 'style-loader',
    MiniCssExtractPlugin.loader,
    'css-loader',
    'postcss-loader',
    'sass-loader'
  ]
},
// 省略
new MiniCssExtractPlugin({
      filename: `${outputFile}.css`, // 分割
      // chunkFilename: '[name].[hash].css'
    })
```

プラグインとローダの違い
ローダはファイル毎にコンパイルや変換処理をするもの、プラグインはこれよりも広義を意味してwebpackに機能追加したりする。

## srcにあるindex.htmlにjs, cssファイルタグを自動で追加する（インジェクトする）

ファイル名がハッシュ値とかになると手動でタグを追加するのは記述ミス等に繋がるため、それを回避するためにタグをプログラムから記述してもらう事にする。 それには `html-webpack-plugin` を使用する。

```bash
npm i -D html-webpack-plugin
```

webpack.config.js

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
// 一部省略

plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, '/src', 'index.html'),
      inject: 'body', // 分割
      // app.jsを代入したい場合は下記を記述する。
      // app.jsはscssとjqueryに依存している。
      chunks: ['app']
    })
  ]

```

## HTML内の画像がハッシュ値でも読み込めるように設定する

上記のhtmlWebpackPluginと併用して使う。

```bash
npm i -D html-webpack-plugin
```

下記の設定を追加する事でwebpackがhtmlファイルを読み込んでくれるようになる。その際に `file-loader` の条件に一致して画像名をハッシュ化してくれる。

webpack.config.js

```js
{
  // htmlWebpackPluginと併用しないと動作しない
  test: /\.html$/,
  use: ['html-loader']
}
```

## 共通モジュール（Jquery等）をプロバイダーに登録してimportの記述を省略する

`ProvidePlugin` は webpackに元々入ってるの下記のimport文を追加する事で使用できるようになる。
下記は `webpack.ProvidePlugin` と記述するのを省略するためにオブジェクトリテラルを使って分割代入をする。

webpack.config.js

```js
const { ProvidePlugin } = require('webpack')

//一部省略
// pluginsに下記の項目を設定する。

new ProvidePlugin({
    jQuery: 'jquery',
    $: 'jquery',
    // オリジナル関数をimportなしで使えるようにする。
    utils: [path.join(__dirname, 'src/js/utils'), 'default']
  })

```

## SplitChunksを使ってモジュールと自作ファイルを適切に分ける

`app.js` と `sub.js` それぞれで Jqueryを読み込んでいる時、今ままでの設定ではビルドした際に両方のファイルにjqueryがバンドルされているという無駄があったため、それを `vender.js` にまとめてそれぞれのjsファイルで読み込んでもらう事にする。 

webpack.config.jsに `module` に下記の設定を追加する。

```js
optimization: {
  splitChunks: {
    // chunksがasyncだとダイナックimportと呼ばれる非同期のimport方法のみに設定が適用される。
      // import('./app.scss')
      // allは普通のimportとダイナミックを分けずにimportする。
      chunks: 'all',
      // 最低限分割するサイズ
      minSize: 0,
      cacheGroups: {
        defaultVendors: {
          name: 'vendors',
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          // reuseExistingChunk: true,
        },
        default: false,
      }
    }
```

ビルドすると `venders.js` が生成される。

## SplitChunksを使って非同期読み込みのファイルをバンドルする

上記のコードにある `optimization: {splitChunks: {ここに}}` に下記のコードを追加する。
すると `app.js` に記述した `import("js/sub")` の `sub.js` と `utils/index.js` がバンドルされて `utils.js` になり非同期で読み込まれるようになる。index.htmlのheadタグに注入されて実行されたら消えるようになる。

```js
// オリジナルの関数をキャッシュに追加する。
utils: {
  name: 'utils',
  test: /src[\\/]js/,
  // これがasyncだとutilsファイルはindex.htmlに含まれなくなる。 utilsファイルも生成されなくなる。
  // app.js等にバンドルされるようになる。
  chunks: 'async' //ここを消すと非同期の読み込みでも同期的な読み込みに変わる。
},
```

## Rsolveを使用してファイルパスを短くする

ディレクトリの構造が複雑になってきた際にショートカットとして設定する。

webpack.config.jsのmodule内に下記の設定を追加する

```js

resolve: {
    alias: {
      '@scss': path.join(__dirname, 'src/scss'),
      '@images': path.join(__dirname, 'src/images') 
    },
    // importする際の拡張子を省略できる。
    // import '@scss/app.scss' → '@scss/app'
    extensions: ['.js', '.scss'],
    // これを追加すると import jQuery from './node_modules/jquery'
    // みたいな書き方をしなくてよい。
    // これに自作のコードも追加する。すると import 'js/sub' でコードを読み込めるようになる。
    // 画像ファイルの読み込みにも適用される。 
    modules: [path.join(__dirname, 'src'), 'node_modules']
  }

```

使用する際
下記の様にパス設定を行える。

app.scss

```scss
.bg {
  // エイリアスで登録したパスで呼び出す場合
  // background-image: url('~@imgs/doodle.png');
  // モジュールで登録した方法で呼び出す場合
  background-image: url('~images/doodle.png');
  background-size: cover;
  width: 100px;
  height: 100px;
}
```


app.js

```js
import("@scss/app")
// 一部省略
```

## VSCodeの補完機能に上記のパス設定を追加する

jsconfig.jsonというファイルのディレクトリに追加して下記の設定を追加する。

- baseUrl: pathsの親フォルダとなるベースのパスを指定する。
- path: 左の文字列を入力した際に右側のパスが自動補完される。 

```json
// vscodeに自動補完機能を持たせる。
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@scss": ["scss"],
      "scss": ["scss"],
      "js": ["js"],
      "images": ["images"],
    }
  }
}
```

## 複数のHTMLが存在する場合にそれぞれ異なるスクリプトを自動で挿入する

それには `HtmlWebpackPlugin` の設定を新たに1つ追加してそこに別のHTML情報を記述する。

webpack.config.jsのplugins内に下記の設定を追加する。
これで [http://localhost:8080/other.html](http://localhost:8080/other.html) にアクセスした際に `sub.js` が注入されていることが確認できる。

```js
plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, '/src', 'index.html'),
      inject: 'body', // 分割
      // app.jsを代入したい場合は下記を記述する。
      // app.jsはscssとjqueryに依存している。
      chunks: ['app']
    }),
    new HtmlWebpackPlugin({
      template: path.join(__dirname, '/src', 'other.html'),
      filename: 'other.html',
      inject: 'body', // 分割
      // sub.jsはjqueryに依存している。
      chunks: ['sub']
    })
  ]
```


## webpack.config.jsの設定を開発用と商用で切り分ける。

`webpack-merge` を用いて webpackの設定を `webpack.common.js`（開発用・商用での共通設定）, `webpack.dev.js`（開発用）, `webpack.prod.js`（商用） に分割する。

webpack.common.js

```js
const path = require('path')
// jsファイルにバンドルされるはずのcssを分離できる。cssに変更がない場合はキャッシュを利用できるので商用環境では分離をオススメする。
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
// const webpack = require('webpack')
// こうする事で個別でimport出来る。分割代入
const { ProvidePlugin } = require('webpack')


module.exports = ({ outputFile, assetFile }) => ({
  // 作業ディレクトリがある場合はentryの書き方が変わる
  context: `${__dirname}/src/js`,
  // entry: './index.js',
  // joinを使用するとOSの環境に合わせてパスをいい感じに設定してくれる。
  entry: {
    app: path.join(__dirname, '/src', '/js', 'app.js'),
    // これがないとsub.jsは出力されない。これは同期的な読み込みになる。
    sub: path.join(__dirname, '/src', '/js', 'sub.js')
  },

  // defaultの設定つまりあってもなくても一緒
  // ファイルの起点を設定する。
  // entry: './src/index.js',

  // ここは絶対パスで指定する必要がある。
  output: {
    // path: `${__dirname}/dist`,
    path: path.join(__dirname, '/dist'),
    filename: `${outputFile}.js`,
    assetModuleFilename: `images/${assetFile}[ext]`, // 分割
    // これがsub_js.jsを出力してる。
    chunkFilename: `${outputFile}.js`
  },


  // useはしたから実行されていく。
  // sassをcssに変換 → cssをバンドル → htmlにstyleタグを使ってcssを記述する。順番で実行される。
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        // // eslint導入方法
        // use:[
        //   // 'style-loader',
        //   'babel-loader',
        //   'eslint-loader'
        // ],
        loader: 'babel-loader'
      },
      {
        enforce: 'pre',
        test:/\.js$/,
        exclude: /node_modules/,
        loader: 'eslint-loader',
        // eslintのfixオプションをonにするルールに基づいてコードを整形してくれる。
        options: {
          fix: true,
        }
      },
      {
        test:/\.scss$/,
        use:[
          // 'style-loader',
          MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader',
          'sass-loader'
        ]
      },
      {
        test: /\.(ico|svg|jpe?g|png|webp)$/,
        type: "asset/resource",   // <--- 'file-loader'
        // または
        // type: 'asset/inline',  // <--- 'url-loader'
      },
      {
        // htmlWebpackPluginと併用しないと動作しない
        test: /\.html$/,
        use: ['html-loader']
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: `${outputFile}.css`, // 分割
      // chunkFilename: '[name].[hash].css'
    }),
    // よく使用するモジュールをimportせずにグローバルにバンドルする事ができる。
    // これによってapp.js, sub.jsでjqueryが使用できる。
    new ProvidePlugin({
      jQuery: 'jquery',
      $: 'jquery',
      // オリジナル関数をimportなしで使えるようにする。
      utils: [path.join(__dirname, 'src/js/utils'), 'default']
    })
  ],
  optimization: {
    splitChunks: {
      // chunksがasyncだとダイナックimportと呼ばれる非同期のimport方法のみに設定が適用される。
      // import('./app.scss')
      // allは普通のimportとダイナミックを分けずにimportする。
      chunks: 'all',
      minSize: 0,
      cacheGroups: {
        defaultVendors: {
          name: 'vendors',
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          // reuseExistingChunk: true,
        },
        // オリジナルの関数をキャッシュに追加する。
        utils: {
          name: 'utils',
          test: /src[\\/]js/,
          // これがasyncだとutilsファイルはindex.htmlに含まれなくなる。 utilsファイルも生成されなくなる。
          // app.js等にバンドルされるようになる。
          chunks: 'async' //ここを消すと非同期の読み込みでも同期的な読み込みに変わる。
        },
        default: false,
      }
    }
  },
  // ファイル等をimportする際にPathを入力しやすくするためにシュートカットを作成する。
  // 階層が複雑な際に使用すると便利です。
  resolve: {
    alias: {
      '@scss': path.join(__dirname, 'src/scss'),
      '@imgs': path.join(__dirname, 'src/images') 
    },
    // importする際の拡張子を省略できる。
    // import '@scss/app.scss' → '@scss/app'
    extensions: ['.js', '.scss'],
    // これを追加すると import jQuery from './node_modules/jquery'
    // みたいな書き方をしなくてよい。
    // これに自作のコードも追加する。すると import 'js/sub' でコードを読み込めるようになる。
    // 画像ファイルの読み込みにも適用される。 @を使用しなくてよくなる。
    modules: [path.join(__dirname, 'src'), 'node_modules']
  }
})
```

webpack.dev.js

```js
const path = require('path')

const HtmlWebpackPlugin = require('html-webpack-plugin')
const { merge } = require('webpack-merge')
const commonConf = require('./webpack.common')

const outputFile = '[name]'
const assetFile = '[name]'

module.exports = () => merge(commonConf({ outputFile, assetFile }),  {
  mode: 'development', //分割
  // これを使用するとchromeの検証でビルド前のファイルも確認できるのでデバックが取りやすくなる。
  devtool: 'source-map', // 分割
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    port: 8080,
    host: '0.0.0.0',
    watchOptions: {
      ignored: /node_modules/
    }
  },
 
  
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, '/src', 'index.html'),
      inject: 'body', // 分割
      // app.jsを代入したい場合は下記を記述する。
      // app.jsはscssとjqueryに依存している。
      chunks: ['app']
    }),
    new HtmlWebpackPlugin({
      template: path.join(__dirname, '/src', 'other.html'),
      filename: 'other.html',
      inject: 'body', // 分割
      // sub.jsはjqueryに依存している。
      chunks: ['sub']
    })
  ]
})
```

webpack.prod.js

```js
const path = require('path')

const HtmlWebpackPlugin = require('html-webpack-plugin')
const { merge } = require('webpack-merge')
const commonConf = require('./webpack.common')

const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')

const outputFile = '[name].[chunkhash]'
const assetFile = '[contenthash]'

module.exports = () => merge(commonConf({ outputFile, assetFile }),  {
  mode: 'production', //分割
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    port: 8080,
    host: '0.0.0.0',
    watchOptions: {
      ignored: /node_modules/
    }
  },
  
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, '/src', 'index.html'),
      inject: 'body', // 分割
      // コード内のコメント等取り除いて圧縮する。
      minify: {
        collapseWhitespace: true,
        keepClosingSlash: true,
        removeComments: true,
        removeRedundantAttributes: true,
        removeScriptTypeAttributes: true,
        removeStyleLinkTypeAttributes: true,
        useShortDoctype: true
      }
    })
  ],
  optimization: {
    minimizer: [
      (compiler) => {
        const TerserPlugin = require('terser-webpack-plugin')
        new TerserPlugin({
          terserOptions: {
                compress: {},
                mangle: true
            }
        }).apply(compiler)
    },
      new CssMinimizerPlugin(),
    ]
  }
})
```

これが最終的なwebpackの設定になる。
ファイルのディレクトリ構成等を確認したい場合は [GitHub-webpack-practice](https://github.com/wimpykid719/webpack-practice)に揚げてあるので確認して下さい。

## 最後に

おそらくこれで `create-react-app` を使用しなくてもReactの環境を構築するのに必要なwebpackの知識は付いたと思うので次回からReactの環境を作成してStripeのサンプルコードを動かしていけたらと思う。

### 参照

[【Webpack5】file-loaderを使った画像の読み込みがうまくいかない](https://teratail.com/questions/327351)

[Conflict: Multiple assets emit to the same filename](https://stackoverflow.com/questions/42148632/conflict-multiple-assets-emit-to-the-same-filename)

[Configuring ESLint - 日本語訳](https://jtfal.com/docs/eslint/user-guide/configuring/)

ESLint 環境プロパティ

[Specifying Environments](https://eslint.org/docs/user-guide/configuring/language-options#specifying-environments)

webpack-mergeのimportの仕方がwebpack5から変更された。

[Getting Error from webpack-cli: “TypeError: merge is not a function” in webpack config](https://stackoverflow.com/questions/62846123/getting-error-from-webpack-cli-typeerror-merge-is-not-a-function-in-webpack)

webpack5 から optimize-css-assets-webpack-pluginは非推奨となりcss-minimizer-webpack-pluginを使用する。
下記の質問ではなんでcssしか圧縮されないそして `...` 使用した際jsが圧縮されるのなんでという投稿者に問いに回答が付いてる。
[Why is CssMinimizerWebpackPlugin preventing my main js file from being minified?](https://stackoverflow.com/questions/66630656/why-is-cssminimizerwebpackplugin-preventing-my-main-js-file-from-being-minified)

terser-webpack-pluginはwebpack5からはデフォルトでインストールされるので追加する必要がないというドキュメント、しかしサンプルコードはwebpack4のしかないためどうやって使ったらいいのという質問が投げかけられてる。これはGitHubのissueにもドキュメント分かりにくすぎと上がっていた。
[Use latest terser-webpack-plugin with Webpack5](https://stackoverflow.com/questions/66343602/use-latest-terser-webpack-plugin-with-webpack5)

dockerで建てた環境からのwebpack-dev-serverにアクセスする方法
[Dockerでwebpack-dev-serverを起動してホストからアクセスする](https://zukucode.com/2020/05/webpack-dev-server-docker.html)

[JavaScript ES5, ES6等の早見表](https://www.tohoho-web.com/js/what.htm)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。