---
title: "WEB開発業務で絶対入れてるVSCode拡張7選" # 記事のタイトル
emoji: "🩻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["vscode", "環境構築", "初心者"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: "2023.01.21"
---

## 最初に

最近、業務用の開発環境をリセットすることが多くて、その度に VSCode の拡張機能を入れ直してる（クリーンインストール大好きなのでバックアップ復元はしない）ので毎回絶対インストールするのを紹介しようと思う。
みんなが業務で利用する拡張機能も知りたいのでコメント欄に使っている拡張とその用途を書いて頂けると、これから業務に就こうと考えている駆け出しエンジニアと呼ばれる方達にも参考になると思うのでお願いいたします。

上記は建前で主に自分が気になるので教えて下さい。

自分は rails アプリケーションの開発に関わっているので Ruby に関する拡張機能が多めになります。

## noctis

読み始めていきなり申し訳ないのですが、これは正直他の方は要らないと思います。
自分がプログラミングを始めた時からこのカラーテーマで、コードを書いていてるので愛着があります。
これじゃないと落ち着かない部分があるので利用してます。

テーマカラーの変更する拡張子で Azureus がお気に入りでずっと使っている。
https://marketplace.visualstudio.com/items?itemName=liviuschera.noctis

## Rails Run Specs

これは会社の先輩に教えて頂いて、めちゃくちゃ便利で使ってます。
RSpec でテストを実行する際に実行したい箇所にカーソルを合わせて下記のショートカットを押すとその箇所を実行するコマンドを VSCode のターミナルで実行してくれます。
毎回手打ちは大変なのでかなり重宝しています。

RSpec を `cmd+ctr+l` で 1 行単位でテストを実行できる。
https://marketplace.visualstudio.com/items?itemName=noku.rails-run-spec-vscode

## GitLens — Git supercharged

これもよく使ってます。チーム開発となるといろんな人が毎日コードを追加していきます。
中には追加された背景等を知らないと理解が難しい処理等があります。
そんな時これでこの部分は誰が書いたのか確認して聞きにいったりしています。
あとはコード追加時のプルリクを確認して追加された理由を確認したりします。
コード理解の手助けをしてくれるので必ず入れてます。

コード書いた人が分かるので、質問しに行く時やコミット時のコードを確認しやすくなる。
https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens

## Prettier - Code formatter

これは保存時にコードを整形してくれる。
インストール後に user の `settings.json` の `editor.formatOnSave` を `true` にする必要がある
ので今自分の VSCode の設定ファイルは下記のような感じになっている。

```json
// Code/User/settings.json
{
  "workbench.colorTheme": "Noctis Azureus",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "window.zoomLevel": -1
}
```

この[記事](https://zuma-lab.com/posts/eslint-prettier-settings)を参考に `vscode` の `settings` を `Prettier` の部分だけ変更した。

## Ruby

rails で定義元ジャンプ出来るようになる。
https://marketplace.visualstudio.com/items?itemName=rebornix.Ruby

## endwise

ruby 構文で end を自動で挿入してくれる。
地味に便利
https://marketplace.visualstudio.com/items?itemName=kaiwood.endwise

## Trailing Spaces

無駄なスペースに色を付けて表示してくれる。
rails アプリケーションの方には自動で空白を取り除く等のコード整形する処理が入っていないため、現状はこれでバックエンド処理に空白が入るのなるべく防いでいる。
最近環境リセット直後でこれが入ってなくて知らない間にめちゃくちゃ無駄な空白が入っていて別プルリクを作成して対応するはめになった（先輩に全部やってもらった）のでこれは絶対入れておきたい一つです。

https://marketplace.visualstudio.com/items?itemName=shardulm94.trailing-spaces

## 最後に

以上が自分が業務で使用する VSCode 拡張 7 選になります。記事を読んで頂きありがとうございます。
みなさんが業務で使う拡張もエピソード含め紹介があると、使い方をイメージできるのでコメントに残して頂けると幸いです。
去年は業務が少し忙しいを理由に個人開発・技術ブログを書くのが怠っていたので今年はたくさん書ければと思います。

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。
