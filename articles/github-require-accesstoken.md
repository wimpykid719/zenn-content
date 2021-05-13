---
title: "GitHubパスワード認証（もうすぐ使えなくなる）をアクセストークン認証に変更する。" # 記事のタイトル
emoji: "🔐" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["github", "git"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.05.13'
---
## 最初に

2021年8月13日からGitHubでパスワードによる認証が廃止される（2021年6月30日、2021年７月28日にも対応出来ていない人に知らせるようになっているらしい。）ので、それに対応出来るようにアクセストークンによる認証方法に変更する。

[公式](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/)によると今回影響を受けるのは

- コマンドラインgitからのアクセス
- gitを使ったデスクトップアプリ（GitHub Desktopを除く）
- パスワード認証を使うサービス・アプリ全般

自分は普段コマンドラインからGitHubにアクセスしているので、一番最初のに当てはまる。

## パーソナルアクセストークンを取得する。

GitHubのsettingページに行く → Developer Setting → [Personal access token](https://github.com/settings/tokens)

このページでGenerate new tokenをクリックする。 名前はお好みで、アクセススコープは コマンドラインでリポジトリの操作全般を行いたいので、repoとadmin：repo_hookとdelete_repoにチェックを入れる。そしてGenerate tokenでトークンを生成する。生成されたトークンをパスワードの変わりに使用する。

[Creating a personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token)

## gitにトークンを設定して行く。

macの場合キーチェーンにgitのパスワード情報があると思うので`git-credential-osxkeychain` それを削除する。 

そして、  `git add .` をすると Username、passwordを聞かれるのでパスワードの部分に今回取得したアクセストークンを設定します。これで対応完了です。

そしてアクセストークンを使用する場合はリモートリポジトリ等のURL設定はSSHではなくHTTPSにする必要があるそうです。

[Updating credentials from the macOS Keychain](https://docs.github.com/ja/github/getting-started-with-github/updating-credentials-from-the-macos-keychain)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。