---
title: "VercelのDeploy Hookがとても便利だった。" # 記事のタイトル
emoji: "💏" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["vercel", "webhook"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.07.07'
---

## 最初に

今回はNext.jsで作成した技術ブログをあるタイミングでビルドして欲しいので、その機能を実装していく。
そのタイミングはZennの記事を管理するリポジトリに更新があったらである。
そこで再ビルドをかけて記事を更新するようにしたい。
どうしてそんな事するのかというと下記の技術ブログはZennの記事を管理しているリポジトリからデータを取得して作成しているので、Zennに記事を投稿したら同様に再ビルドをかけて記事を追加して欲しいためである。

**Next.jsで作成した技術ブログ**
[大学生だった.](https://techblog-pink.vercel.app/)

それを実装するには Vercelにある Deploy Hook を使う事で簡単に実装できる。
これがとても便利でこれを使えば、コードを一行も書く事なくお望みの機能を実装出来るので紹介したいと思う。

[Deploy Hooks - Vercel Documentation](https://vercel.com/docs/more/deploy-hooks)

これを使うとVercelが生成したURLにPost通信を投げる事で再ビルドをかける事ができる。

## Deploy Hookを作成する。

![vercelProject](https://user-images.githubusercontent.com/23703281/124548943-6b6a6800-de69-11eb-99e6-62de2d711d01.png)


まずVercelでPost通信、再ビルドをかけたいプロジェクトに移動する。

settings > git ページに移動する。

自分はすでに作成したのがあるため上に表示されているが、作成してない場合は何も表示されていないと思う。

名前（任意のhookにあった名前）とビルドするブランチを指定する。プロジェクトと連携してある本番環境をビルドして欲しいのでmainと入力する。

![deployhooks](https://user-images.githubusercontent.com/23703281/124548987-791fed80-de69-11eb-80db-ab087104dcd1.png)

生成すると `https://api.vercel.com/v1/integrations/deploy/QmcwKGEbAyFtfybXBxvuSjFT54dc5dRLmAYNB5jxxXsbeZ/hUg65Lj4CV` こんな感じのURLが生成されるので、そこに curl等でPost通信を投げるとビルドを始めてくれる。そして `/deploy/` 以降のランダムな文字列が他の人に渡ると誰でもビルド出来てしまうのでパスワードのように扱う必要がある。

下記を実行すると再ビルドが走る。

```bash
curl -X POST 生成したURL
```

このURLをGitHubのZennを管理する、リポジトリのwebhooksに設定する事でリポジトリの更新が入るたびにVercelにPost通信を投げてビルドして記事を更新してくれる。

## GitHub webhooksの設定

[Vercel functionを使ってQiitaの記事をGitHubリポジトリから自動投稿・更新出来るAPIを作ってみた。](https://zenn.dev/unemployed/articles/nextjs-api-qiita-post#qiita%E3%81%AE%E8%A8%98%E4%BA%8B%E3%82%92%E7%AE%A1%E7%90%86%E3%81%99%E3%82%8B%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E3%81%AEwebhook%E8%A8%AD%E5%AE%9A)

上記のページでPayload URLを設定する箇所があるので、それを今回生成したURLにする事で設定する事ができる。

## 最後に

今回はコードを書かずに機能を追加する事が出来てとても簡単だった、webhookを使うとこのように外部サービスの機能を簡単に連携出来るのでとても便利だと思う。

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。