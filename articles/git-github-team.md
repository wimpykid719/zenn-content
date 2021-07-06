---
title: "すぐ業務に付けるようにGitHubで一人チーム開発を体験してみた。" # 記事のタイトル
emoji: "👨‍❤️‍💋‍👨" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["git", "github", "初心者"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.07.06'
---

## 最初に

最近になってようやくgithubでrepositorieを作成して、書いたコードをpushする事が習慣的に出来るようになって前よりも草が生やせるようになったので、今回はそれを記事にして残して置こうと思います。
それで業務にもすぐに対応出来るように今回は後半で、実際に自分が開発してる技術ブログに機能を追加する過程をチーム開発風にissueを立て、プルリクエストを出してマージするところまでやってみようと思う。

その前に基礎的なGit・GitHubの使い方を見ていこうと思う。

## Clone

今回は初めて大学の卒業制作で作成した[検索エンジン](https://github.com/wimpykid719/pythonengine)をGithubに上げていたのでそれをcloneしてみる。

ディレクトリを作成する。

そこに移動する。

```bash
mkdir searchEngine
cd searchEngine
```

cloneする。URLはGithubのレポジトリページにあるcodeをクリックするとclone用のページを表示してくれます。

```bash
git clone https://github.com/wimpykid719/pythonengine.git
```

cloneすると移動したディレクトリにファイルがダウンロードされる。そして開発を始めていく流れだと思う。

## コードをGithubにあげるまで

リモートリポジトリはあらかじめGithubのサイトで作成しておく。

Githubで管理したいコードが置いてあるフォルダ内で実行する。

```bash
git init
```

.gitというフォルダ作成される。

次にファイルをステージングエリアに移動させる。

```bash
git add .
git commit -m "メッセージ"
```

※直前の `git add` を取り消したい場合は `git reset HEAD` を使うと取り消す事ができる。

※コミットメッセージのみ修正したい場合は下記を実行する。

```bash
git commit --amend -m "修正後コメント"
```

※gitをインストールして、はじめて使用する場合はユーザを入力してないと `commit` 出来ない。

**ユーザ入力**

```bash
git config --global user.name “userName”
git config --global user.email "xxx@yyy.com"
```

リモートリポジトリを指定する。

はじめてpushする際はアップロード先を指定する必要がある。

アップロード先はレポジトリ作成時のhttpsをコピペする。

```bash
git remote add origin アップロード先
```

これでGithubのリポジトリにコードが追加される。

```bash
git push -u origin master

# **ブランチ名がmasterでなく長い場合
#** 下記のようにすると現在のブランチの先頭、コミットの一番新しいものを指すことが出来る。
****git push origin HEAD
```

pushのオプション `-u` は `--set-upstream` と同じ意味でこれを付けると次回から `git push` だけでGithubにコードをあげることが出来る。

ローカルブランチが上流ブランチorigin masterとして設定される。

イメージはGithubのレポジトリと直接繋がっているような感じ？そのため `git push` だけでGithubまでコードがアップされる。

**上流ブランチとは**

あるローカルブランチが履歴を追跡するように設定したリモートブランチの事を指す。

ローカルブランチが更新されたら、リモートブランチも更新される関係になる。

**現在のブランチが上流ブランチに設定されているか確認**

```bash
git branch -a

#実行結果
* master
  remotes/origin/master
```

`*` これは現在作業しているブランチを指している。その下に設定された上流ブランチが表示される。

**間違えてアップしたファイルを.gitignoreで削除する方法**

`.gitignore` というファイルを作成してそこにファイル名を書いて置けばいいんだけど、後からやる場合はキャッシュを削除する作業が必要みたい。

これを行なった後にadd とcommitしないと意味がない。

```bash
git rm --cached 無視したいファイル名

git rm -r --cached . これはファイル全体のキャッシュ削除
```

`--cached` オプションを使用しないと、ファイル事体が削除されるので注意が必要です。

## ブランチの作成

新規機能の作成等でmainブランチからブランチを切って開発を進める事がある。その方法を紹介する。

現在のブランチを確認する。 `*` が付いているのが現在いるブランチ、下記の場合は `main` ブランチにいる。

```bash
git branch

#実行結果
* main
  settings-feedly-button
```

ブランチを実際に作成する。

```bash
git branch 作成したいブランチ名
```

作成したブランチに切り替える。

```bash
git checkout 作成したブランチ名
```

git バージョン2.23.0以降では `switch` で `checkout` と同じ事ができる。

オブション `-c` を使えばブランチの作成と切り替えを同時に行える。

```bash
git switch -c 作成して切り替えたいブランチ名
```

## 作成したブランチをGitHubにPushする

現在のブランチを切り替えたら

```bash
git push origin HEAD
```

これで現在のブランチをGitHubにpushする事が出来ます。

もしくはブランチ名を直接指定することも出来ます。

```bash
git push origin ブランチ名
```

## 作成したブランチをmainブランチにマージする

機能追加が終了し本番環境にデプロイする際にブランチをmainブランチにマージする事があります。

まずはメインのブランチに移動する。

```bash
git checkout メインのブランチ大体mainかmasterになってる。
```

その後でマージする事が出来ます。ただチーム開発ではプルリクエストを出してコードをレビューしてもらった後にGitHub上でマージする事が多いと思われるので、この操作を行うことは少ないかもしれない、

```bash
git merge マージしたいブランチ名
```

## マージしたブランチを削除する

マージして不要になった、ブランチはなるべく消した方が良いので削除方法も載せておく。これでローカルのブランチは消せる。

### GitHub上でブランチを削除した後にローカルのブランチを削除する方法

実はこの状態でブランチを削除してGitHubにpushしてもGitHub側のブランチは削除されない。削除するには手動でGitHub上で削除する必要がる。

```bash
git branch -d ブランチ名
```

### リモートブランチを削除してから、それをローカルに反映させる方法

ローカルからコマンドを使用してGitHub上（リモートブランチ）を削除する。

```bash
git push --delete origin ブランチ名
```

リモートリポジトリのブランチ状態をローカルに同期する。

GitHub上でリモートブランチを先に削除した場合は下記のコマンドが使用で出来ます。これでローカルブランチも削除される事になる。

これは間違いで実際に削除するのは追跡ブランチでローカルブランチは削除されない。そのため `git branch -d ブランチ名` を実行する必要がある。

他の記事とかにはよくこれで削除するみたいに書いてあったが、違うみたいだ。

それとも `git push --delete origin ブランチ名` と実行したあとだとローカルにある追跡ブランチも同時に削除されるから、 `git fetch -p` リモートからローカルにデータを同期する際に追跡ブランチを通さないといけないが、それが存在しないため同期されないんだろうか? 試しにGitHubから手動でブランチを削除した後に `git pull --prune` をやってみたが、結果は同じでローカルブランチが削除される事はなかった。

詳しい方がいたら教えて下さい。

```bash
git fetch -p
```

## 実際に技術ブログに機能を追加しながら、チーム開発を行ってみる。

今回は技術ブログに機能追加を行う過程を、チーム開発風に行ってみたいと思う。機能としてはRSSを生成する機能を追加したらビルドに時間が掛かるようになってしまったため、本番環境以外ではRSSを生成する機能を実行しないようにする。

### GitHubリポジトリにissueを立てる

タスクを設定しておいて、終わったらチェックを打って行くようにすると進捗が分かる。 `- [ ] タスク` でチェックボックスを追加出来る。

![機能追加issue](https://user-images.githubusercontent.com/23703281/124534050-72d04800-de4e-11eb-9665-4212ac23f709.png)

### 機能追加用のブランチを切る

今回は `make-rss-do-only-production` というブランチを作成して、機能を実装していく。

```bash
git branch make-rss-do-only-production
```

ブランチを切り替える。

```bash
git checkout make-rss-do-only-production
```

機能開発を終えたらコミットする。

```bash
git add .
git commit -m "done rss do only production env"
```

リモートのリポジトリに作成したブランチをpushする。

```bash
git push origin HEAD
```

### Pull リクエスト作成する

![pullreq](https://user-images.githubusercontent.com/23703281/124534168-a6ab6d80-de4e-11eb-919d-f865bf061bd8.png)

GitHubに戻ると `Compare & pull request` とブランチが追加された等の表示がされるので、プルリクエストを作成する。

![createpullreq](https://user-images.githubusercontent.com/23703281/124534213-b9be3d80-de4e-11eb-97d9-7b9cb5fc8581.png)

`Closes #5` とする事でissueと結びつける事が出来、pull requestがマージされた際にissueを閉じる事ができる。 `#5` は先ほど作成した issueの番号になる。

### プルリクエストをマージする

ここからはチームリーダがプルリクエストされたコードをレビューして動作確認が取れたらマージする作業を行います。今回はいないので自分でマージしていきます。

`Merge request` をクリックするとマージされます。その前にコメントとか残しておくといいかもしれないです。

![merge](https://user-images.githubusercontent.com/23703281/124534290-e2463780-de4e-11eb-9b4d-df58b813326e.png)

マージが完了するとissueも自動でcloseされる。

![merged](https://user-images.githubusercontent.com/23703281/124534228-c5a9ff80-de4e-11eb-967e-bb2d5eda226e.png)

## マージをローカルのリポジトリにも適用する

mainブランチ切り替えて、まだマージされていないブランチを確認する。まだローカルでは `make-rss-do-only-production` をマージしていないので `make-rss-do-only-production` が表示される。

```bash
git checkout main
git branch --no-merged
```

リモートリポジトリから最新のmainブランチを同期させる。

※ここでブランチをmainに切り替えてないと、マージマージされないので注意が必要です。

```bash
git pull origin main
```

その後 `git branch --no-merged` しても `rss-do-only-production` が表示されないのでローカルのブランチがマージされた事が確認できる。

### リモートのブランチを削除する

```bash
git push --delete origin make-rss-do-only-production
```

### ローカルのブランチを削除する

ここではまだローカルブランチが残っているので、 `git fetch -p` する事でリモートブランチとローカルブランチを同期する。と思っていたが、どうやらこれで削除するのは追跡ブランチというものらしい。

追跡ブランチとは

ローカルブランチとリモートブランチの間に噛ませるように追跡ブランチと上流ブランチが実はあるみたい。どうしてデータを同期する中継するような構造になってるかは分からない。この追跡ブランチが上流ブランチを通して、リモートブランチの変更を監視している。なのでここが削除されればリモートブランチの状態を反映するのは難しくなる。

ローカルブランチ（ローカル） → 追跡ブランチ（ローカル） → 上流ブランチ（リモート）→ リモートブランチ（リモート）

[http://elsur.xyz/git-commands-pull-fetch](http://elsur.xyz/git-commands-pull-fetch)

ローカルブランチを削除するには手動での削除が必要になるので下記のコマンドで削除する。

```bash
git branch -d make-rss-do-only-production
```

これを繰り返しながらプロダクトを作っていく流れだと思う。

## 最後に

git・githubは最初よく分からなくて、コンフリクト起こしたり、あんまり触るのが好きではなかったです。

今はリポジトリを作成してコードを管理するくらいはそつなくこなせるようになったので、これからは積極的にGitHubにコード上げていこうと思います。コード書くのも大変なんですけど、、、

どの仕事もやることは多いですが、エンジニアは使えないといけないツールが多くてコード書くのも大変だし、ツールのコンセプトを理解して使い方を覚えるのも大変です。

それでもプログラムが完成した瞬間は何にも変えられないくらい楽しいですね。

そのためだけに難しい言語を覚えて、難しいツールを覚えて使っていこうと思います。

### 参照

[git pushのオプション -u とは - Qiita](https://qiita.com/shumpeism/items/1b8027c8905ca826416d)

[Git用語：上流ブランチとは？](http://www-creators.com/archives/4931)

[git push -u オプションで"上流ブランチ"を設定](https://www-creators.com/archives/5204#i)

[git push origin HEADを知らない人は時間を無駄にしています！ - Qiita](https://qiita.com/Takao_/items/44d2aa23d6b1ab22a53a)

[gitコマンド checkoutとswitchの違い　～これからはswitchを使おう～ | Snow System](https://snowsystem.net/git/git-command/git-switch/)

[git add を取り消す - Qiita](https://qiita.com/yukure/items/89562e5eb1d03995dc5b)

[gitを使用したブランチ作成からpushまでの簡単な流れ - Qiita](https://qiita.com/infratoweb/items/5021a36f69f26dc7f0b9)

[git mergeを使ってブランチをマージする方法 | TechAcademyマガジン](https://techacademy.jp/magazine/10264)

[https://stackoverflow.com/questions/37664226/git-fetch-origin-prune-doesnt-delete-local-branches](https://stackoverflow.com/questions/37664226/git-fetch-origin-prune-doesnt-delete-local-branches)

[https://stackoverflow.com/questions/48820631/git-remote-prune-origin-does-not-delete-the-local-branch-even-if-its-upstream-re](https://stackoverflow.com/questions/48820631/git-remote-prune-origin-does-not-delete-the-local-branch-even-if-its-upstream-re)

### 一応読んだ

HEADについて知りたくて読んだが、よく分からなかった。ブランチ、コミットについて詳しく書かれていた。

[GitのHEADとは何者なのか - Qiita](https://qiita.com/ymzk-jp/items/00ff664da60c37458aaa)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。