---
title: "Dockerで立てたNode.js, TypeScript環境にESLintとprettierを導入して、VSCode保存でコード整形も行う" # 記事のタイトル
emoji: "☂️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["eslint", "prettier", "typescript", "nodejs", "vscode"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: "2021.08.04"
---

## 最初に

綺麗なコードを書きたいという事で、Stripe の API を試す環境（TypeScript と Express）に ESLint と Prettier を導入してコードを静的解析、整形出来る環境を作ろうと思う。

## node.js の環境構築

今回も docker-compose を使ってサクッと作ろうと思う。docker じゃなく docker-compose を使う理由はポートをルーティングしたコンテナの起動等が `docker-compose.yml` に記述出来るので起動時のコマンドを省略出来るので使用している。単一のコンテナを利用する際でも便利だと思う。

軽量な node.js のイメージを使用する。

Dockerfile

```
FROM node:lts-stretch-slim
WORKDIR /src
```

ポートは 8080 でルーティングして `docker-compose.yml` が置かれたフォルダの `./` とコンテナ内の `/src` を同期する。

docker-compose.yml

```yaml
version: "3"

services:
  app:
    build: .
    container_name: stripe-practice
    ports:
      - "8080:8080"
    working_dir: /src
    volumes:
      - ./:/src
    # docker run -iを意味する
    stdin_open: true
    # -tを意味する
    tty: true
```

`docker-compose up -d` でビルド（イメージがなければ）とコンテナの実行をバックグラウンドで行う。 `docker-compose exec app bash` でコンテナ内にログインする。

`node -v` でバージョン情報が表示されて入れば node がインストールされている事が確認できる。

## TypeScript・Express をインストールする

まず npm の初期化を行う。この際に `package.json` が作成される。

```bash
npm init
```

TypeScript のインストール

```bash
npm install typescript
```

node.js の TypeScript 用の型をインストールする

```bash
npm i @types/node
```

`tsconfig.json` を作成する

```bash
npx tsc --init
```

コンパイルここでファイルを指定するとルートディレクトリに作成された `tsconfig.json` の設定ファイルを無視するようになるので注意が必要である。 `npx` はローカルにインストールしたパッケージのコマンドをパスの設定をしなくても探して実行してくれるコマンド

```bash
npx tsc
```

express のインストールと express の typeScript 用の型をインストールする。

```bash
npm install express
npm install -D @types/express
```

動作確認コード

これを JS にコンパイルして実行する [`http://localhost:8080/users`](http://localhost:3000/users) でアクセスするとユーザ情報が戻り値として返る。

```tsx
import express from "express";
const app: express.Express = express();
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

//CROS対応（というか完全無防備：本番環境ではだめ絶対）
app.use(
  (req: express.Request, res: express.Response, next: express.NextFunction) => {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Methods", "*");
    res.header("Access-Control-Allow-Headers", "*");
    next();
  }
);

app.listen(8080, () => {
  console.log("Start on port 8080.");
});

type User = {
  id: number;
  name: string;
  email: string;
};

const users: User[] = [
  { id: 1, name: "User1", email: "user1@test.local" },
  { id: 2, name: "User2", email: "user2@test.local" },
  { id: 3, name: "User3", email: "user3@test.local" },
];

//一覧取得
app.get("/users", (req: express.Request, res: express.Response) => {
  res.send(JSON.stringify(users));
});
```

## Prettier と ESLint を導入していく

npm インストールする際にオプションで `--save` と `--save-dev` を使用する事で本番環境・開発環境両方にインストールされるパッケージと開発環境のみにインストールされるパッケージと分ける事ができる。

- `--save` の場合は `package.json` の `dependencies` にパッケージ名が追加される。これは本番環境にデプロイした際などの参照されこれを元に必要なパッケージがインストールされる。
- `--save-dev` の場合は `devDependencies` にパッケージ名が追加され、ここに追加されたパッケージは本番環境ではインストールされず、開発環境のみにインストールされる。なので ESLint 等は開発する際に必要なだけで、本番環境を運用するには必要ないためこちらのオプションを付けると良いと思う。何もオプションを付けない場合はパッケージごとでデフォルトのインストール方法は変わるが基本的に `--save` と同じインストールになる。ESLInt はオプション付けなくても `devDependencies` に追加される。

```bash
npm install prettier
npm install eslint

# eslintのprettierと共存する設定とTypeScriptに対応する設定をインストールする
npm install eslint-config-prettier
npm install @typescript-eslint/eslint-plugin

# eslintの初期化をする。色々聞かれるので選択していく。esrintrc.jsが作成される
npx eslint --init

# eslintで静的解析する エラーがたくさん出る。
npx eslint index.ts

# 上記のエラーをなくしてくれる
npx eslint --fix index.ts

# コードを整形してくれるインデントを揃えたりしてくれる。
prettier --write index.ts
```

### Prettier の設定

.prettierrc

```json
{
  "printWidth": 120,
  "singleQuote": true,
  "semi": false
}
```

esritrc.js（設定ファイルは Json より js の方が読み込まれる優先順位が高いので js の方が良いと個人的に思います。誰かコメントアウト出来るからいいと言ってたけど、Json 基本的にコメントアウト出来る気がするんだけどそれ自体がデータに含まれるのか...）

```jsx
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    // 環境構築した時点ではReactの導入はしてないのでコメントアウトした
    // 'plugin:react/recommended',
    "airbnb",
    "prettier",
    // 下記を追加している記事があるが2021/8/4の時点では必要なくなった。
    // prettierの方で統一された。
    // 'prettier/@typescript-eslint'
  ],
  parser: "@typescript-eslint/parser",
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 12,
    sourceType: "module",
    project: "./tsconfig.json",
  },
  plugins: [
    // 環境構築した時点ではReactの導入はしてないのでコメントアウトした
    // 'react',
    "@typescript-eslint",
  ],
  root: true,
  rules: {
    // "quotes": ["error", "double"]
    // console.logがコードに含まれるとエラーになるので無視するようにルールを追加
    "no-console": "off",
  },
};
```

.eslintignore（手動で作成する必要がある。）

ここで ESLint の解析対象から外したいファイルを書き込む。 `index.js` はすでに解析された `ts` ファイルから作成されるので無視する。 `.eslintrc.js` はどこからも `import` されていないと必ずエラーを吐かれるのであらかじめ除外しておく。

```jsx
/.eslintrc.js
/index.js
```

tsconfig.eslint.json

これは `tsconfig.json` でコンパイルする予定のファイルと ESLint で静的解析するファイルを分けたい際に使用する。

```json
// tsconfigを拡張してTypeScript用のESLintの設定を追加してる。
// なので下記の設定をtsconfigに設定すれば、このファイルを生成する必要はない。
// ただここに書いているのはtsにコンパイラするのではなく、あくまでESLintの静的解析の対象か対象外に
// なるファイルの事指していると思われる。
{
  "extends": "./tsconfig.json",
  // コンパイルする対象ファイルを設定する
  "include": ["*.ts"],
  // コンパイルを除外するファイルを設定する。
  "exclude": ["node_modules", "dist"]
}
```

## VScode で保存時にコードを整形する

拡張機能として下記を２つインストールする。

Name: Prettier - Code formatter
Id: esbenp.prettier-vscode
Description: Code formatter using prettier
Version: 8.1.0
Publisher: Prettier
VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

Name: ESLint
Id: dbaeumer.vscode-eslint
Description: Integrates ESLint JavaScript into VS Code.
Version: 2.1.23
Publisher: Dirk Baeumer
VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)

`.vscode` フォルダ内にある。 `settings.json` に下記の設定を追加する。

この設定はワークスペース呼ばれる場所に VSCode の設定を書いていてデフォルトの設定とは別になる。

この開発フォルダを開いた時だけ Prettier と ESLint の設定になる。

ワークスペースには Code > Preferences > settings で右上のアイコンを選択する事でもアクセスできる。

![https://user-images.githubusercontent.com/23703281/128129511-21bb9d11-9cbd-4965-8051-c25e67e8f23f.png](https://user-images.githubusercontent.com/23703281/128129511-21bb9d11-9cbd-4965-8051-c25e67e8f23f.png)

settings.json（ワークスペース）

```json
{
  // eslintとprettierrcの設定
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.format.enable": false,
  "editor.formatOnSave": true,
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "editor.lineNumbers": "on",
  "eslint.packageManager": "npm",
  "files.insertFinalNewline": true,
  "files.trimTrailingWhitespace": true,
  "npm.packageManager": "npm",
  "typescript.enablePromptUseWorkspaceTsdk": true
}
```

これでコードが保存するたびに整形するようになった。

## 最後に

これでなんとか TypeScript のプロジェクトに Prettier と ESLint は導入出来たので次回から Stripe の API を試して行こうと思う。

### 参照

[GitHub - TryGhost/express-hbs: Express handlebars template engine with inheritance, partials, i18n and async helpers.](https://github.com/TryGhost/express-hbs)

npm にインストールしたパッケージを確認する。

[パッケージの一覧を表示！npm list の使い方【初心者向け】 | TechAcademy マガジン](https://techacademy.jp/magazine/16403)

[知らないのは損！npm に同梱されている npx がすごい便利なコマンドだった | DevelopersIO](https://dev.classmethod.jp/articles/node-npm-npx-getting-started/)

[express の開発に TypeScript を利用する - Qiita](https://qiita.com/zaburo/items/69726cc42ef774990279)

[TypeScript で始める Node.js 入門](https://ics.media/entry/4682/)

[eslint のエラーまとめ(warning Unexpected console statement no-console)](https://ebookbrain.net/eslint-error/#vcodeconsolelogwarning)

[.eslintignore の配置場所は気をつけた方がいい - Qiita](https://qiita.com/satoruk/items/48627a86d3ac51eff43b)

[[VSCode] TypeScript に ESLint を入れると Parsing error: "parserOptions.project" has been set for @typescript-eslint/parser. が表示される](https://wonwon-eater.com/ts-eslint-import-error/#outline__2_1)

[prettier,eslint を導入する際にハマったこと 2021 新年](https://zenn.dev/ryusou/articles/nodejs-prettier-eslint2021)

ESLint が TypeScript を解析できるようにするパーサのドキュメント

[typescript-eslint/README.md at master · typescript-eslint/typescript-eslint](https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/parser/README.md)

[TypeScript + Node.js プロジェクトに ESLint + Prettier を導入する手順 2020 - Qiita](https://qiita.com/notakaos/items/85fd2f5c549f247585b1)

[【いまさらですが】package.json の dependencies と devDependencies - Qiita](https://qiita.com/chihiro/items/ca1529f9b3d016af53ec)

[【日本一わかりやすい TypeScript 入門】ESLint と Prettier でコードの品質を高めよう](https://www.youtube.com/watch?v=R35LJL6a-p0)

[GitHub - prettier/eslint-config-prettier: Turns off all rules that are unnecessary or might conflict with Prettier.](https://github.com/prettier/eslint-config-prettier)

`prettier/@typescript-eslint` は外されたよって話

[ESLint couldn't find the config "prettier/@typescript-eslint" after relocating .eslintrc to parent](https://stackoverflow.com/questions/65675771/eslint-couldnt-find-the-config-prettier-typescript-eslint-after-relocating)

`prettier/@typescript-eslint` 以外にもバージョン 8.0.0 から統一されたものを確認する

[eslint-config-prettier/CHANGELOG.md at main · prettier/eslint-config-prettier](https://github.com/prettier/eslint-config-prettier/blob/main/CHANGELOG.md#version-800-2021-02-21)

[Prettier 入門 ～ ESLint との違いを理解して併用する～ - Qiita](https://qiita.com/soarflat/items/06377f3b96964964a65d)

[【VisualStudioCode】ワークスペースとは？｜副業エンジニアの雑記ブログ](https://kukka.me/vsc-workspace/)

[VS Code でワークスペースごとに設定を変更する方法](https://webdrawer.net/tools/vscodeworkspacesettings.html)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。
