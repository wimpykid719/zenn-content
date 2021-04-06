---
title: "Zennの過去記事を含め、Githubで連携するようにした。" # 記事のタイトル
emoji: "🚚" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["zenn", "git", "github", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
## 最初に

唐突ですが、Next.jsのチュートリアルを終えて、自分のサイトが欲しいと思うようになった。
どうせなら記事をGithubのリポジトリで管理して、そこにアップしたらzennと自分のブログに記事が表示されるようになったら一石二鳥だなと思った。
その前手順でGithubでZennの投稿を管理するようにしたいと思う。
すでに16記事ほどzennのオンラインエディタから投稿しているので過去の記事も移行したいと思う。

### やる事

- リポジトリの作成
- Zennと作成したGithubリポジトリを連携
- 過去記事をリポジトリに追加する

## リポジトリの作成

リポジトリの構成は `articles` フォルダに記事を入れていく。
zennではこのフォルダ名から記事を検索するようになっているので、同じように作成する必要がある。
最初はとりあえず、空のフォルダを作成して、リポジトリを作成する。

```
.
└─ articles
   ├── example-article1.md
   └── example-article2.md
```

## ZennとGithubの連携

ここは公式の記事がとても分かりやすくなっているので、それにしたがって `Only select repositories` にチェックボックスを入れ、作成したリポジトリを選択する。

[GitHubリポジトリでZennのコンテンツを管理する](https://zenn.dev/zenn/articles/connect-to-github)

## 過去記事をリポジトリに追加する

最初からGithubで記事を管理している人は良いが、自分みたいにある程度記事を投稿している人は少し手間が掛かる。
最初は書いた記事を全て削除してからではないと、記事がダブったりするのではと思って `zenn github 連携 過去の記事` 等でgoogle先生に聞いていたのだが、結果的にその心配はなかった。
zennには `slug` と呼ばれる各記事に付けられる固有のIDがある。
そのIDは記事のURLに含まれている。

[https://zenn.dev/unemployed/articles/3c8a872a210ded](https://zenn.dev/unemployed/articles/3c8a872a210ded)

この [`3c8a872a210ded`](https://zenn.dev/unemployed/articles/3c8a872a210ded) が `slug` と呼ばれるIDになる。
それをファイル名に入れて `3c8a872a210ded.md` とする事でオンラインエディタで作成した記事と結び付けてくれる。
そのおかげでこれまでのいいね・コメントを保ったままに出来る。これはとてもありがたい。
これだけだと、ファイルの中身が空っぽなので過去に投稿した記事をコピー・ペーストする必要がある。これが16記事もあるとまぁまぁ大変だった。
ファイルには記事のメタ情報（タイトル・絵文字・タイプ・トピック・公開設定）を含める必要があるので、こちらの内容もあらかじめ記事と同じにする。
上記の記事だとこんな感じのメタ情報になる。

ちなみに [https://zenn.dev/api/articles/3c8a872a210ded/markdown](https://zenn.dev/api/articles/3c8a872a210ded/markdown) こうする事でマークダウンを取得出来る。

```markdown
---
title: "独学・未経験】Pythonデスクトップアプリを作成したから見て欲しい。 （ポートフォリオ ）" # 記事のタイトル
emoji: "🖍️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["python", "初心者", "ポートフォリオ", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
ここから本文が入る。
```

2021/4/7時点では、過去の記事を取得する方法は手作業しかないのでこうする他ない。
記事のデータを取得する方法は今後追加されるかもしれない。それまで待てない、または記事がそこまで多くない人は早めにやっておくと楽かもしれない。
実際に自分が作成したzennの記事を管理するリポジトリ

[wimpykid719/zenn-content](https://github.com/wimpykid719/zenn-content)

## 最後に

この記事がGithubに連携してから初めて書く記事で、連携させると今まで自動で生成されていた `slug` を個別に設定する事が出来る。
この記事のパスは `https://zenn.dev/unemployed/articles/zenn-migrate-past-articles-github` となっていて記事のファイル名に付けた `zenn-migrate-past-articles-github` がそのまま使用されている。

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。

### 参照

[Zenn CLIで記事・本を管理する方法](https://zenn.dev/zenn/articles/zenn-cli-guide)

[Zennのスラッグ（slug）とは](https://zenn.dev/zenn/articles/what-is-slug)

[GitHubリポジトリ連携を通して感じたZennの良さ](https://zenn.dev/unsoluble_sugar/articles/9c04a36a5decdb6d1b20)

[Zennの記事をexportする](https://zenn.dev/yajamon/articles/ebd678f0dd57936e7673)