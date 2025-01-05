---
title: "ソースコード公開したらレスポンス速度を大幅に改善出来た！" # 記事のタイトル
emoji: "🧧" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["rails", "個人開発", "googlecloud", "docker", "ruby", "api"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2025.01.06'
---

## 最初に

明けましておめでとうございます🎍

去年、個人開発したBizRankのソースコードを公開出来たおかげで、自分が一番困っていたインスタンスのコールドスタートが遅すぎてレスポンスがタイムアウトする問題を無事解決出来ました！

[【無料公開】560万円かけて開発したサービスのソースコード](https://zenn.dev/unemployed/articles/bizrank-source-code-free)投稿が460以上いいねと好評だったおかげ色々な方の目に触れる事が出来て、すぐに原因が分かり修正・対応する事が出来ました。

またZennで初めてバッジも頂く事ができありがとうございます。

*コールドスタートに対して言及されていたツイート*

とっても助かりましたCubbitさんありがとうございます。

https://x.com/cubbit2/status/1853781784481402948

https://x.com/yoshi_10_11/status/1853786027946897678

## 原因

コンテナ起動時に実行される `entrypoint.sh` でbundle installをしている事が原因でした。この処理によりコンテナイメージにbundle install後のファイルが含まれていませんでした。

そのためリクエストが飛ぶ度にライブラリのインストールから始まり、サーバを起動して処理してレスポンスを返却しているのでとんでもなく時間がかかっていました！

 `entrypoint.sh` を下記ような書き方にした理由としては開発環境と本番環境でインストールするGemが異なるため分岐処理を行いたかったのです。

しかし、Dockerfileでは分岐処理を書けないためこのような策を取っていました。

シェルの方が色々な書き方が出来るので環境構築等を作る際に便利でした。

```bash
#!/bin/bash
set -e

# 本番環境以外で実行された場合はテストDBを作成
if [ "$RAILS_ENV" != "production" ]; then
  echo '開発環境用のGemをインストール'
  bundle install

  echo 'RSpec用のテストDBを作成'
  bundle exec rails db:create RAILS_ENV=test

else
  echo '本番環境用のGemをインストール'
  bundle install --without development test
fi

bundle exec rails db:migrate

if [ ! -e "/backend/config/routes.rb" ]; then
  echo 'rails new APIモード を実行する'
  # --skip入れないとpgのgemないってエラーが出る
  rails new . --force --api --database=mysql --skip-git --skip-bundle
  bundle install
fi

# Remove a potentially pre-existing server.pid for Rails.
rm -f /backend/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```

そしてコンテナイメージに対しての理解が乏しくてなんとなくで使用していたのが、今回の最たる原因だと思います。イメージに含まれる物とそうでない物の区別が曖昧でした。

なので今回の事象を通してDocker, コンテナ, イメージがどのような技術によって作られているのかはもう少し深掘りして学びたいと思いました。

## 解決方法

実際にどのような対応を行なったのか。

bundle install後に作られたファイル群がイメージに含まれるように、Dockerfile 内で `bundle install` を実行するようにしました。

そして開発環境用・本番環境用でDockerfileを分ける事にしました。

### 変更前

```docker
FROM ruby:3.2.2
WORKDIR /backend

RUN set -eux && \
    apt-get update -qq && \
    apt-get install -y \
      default-mysql-client

ENV TZ=Asia/Tokyo
COPY Gemfile Gemfile.lock* /backend/
COPY . /backend/

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

EXPOSE 8080
# イメージ実行時に起動させる主プロセスを設定
CMD ["rails", "server", "-p", "8080", "-b", "0.0.0.0"]
```

### 変更後

*開発環境用 - Dcokerfile*

```docker
FROM ruby:3.2.2
WORKDIR /backend

RUN set -eux && \
    apt-get update -qq && \
    apt-get install -y \
      default-mysql-client

ENV TZ=Asia/Tokyo
COPY Gemfile Gemfile.lock* /backend/
COPY . /backend/

# 開発環境用のbundle installを追加
RUN bundle install

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

EXPOSE 8080
# イメージ実行時に起動させる主プロセスを設定
CMD ["rails", "server", "-p", "8080", "-b", "0.0.0.0"]
```

*本番環境用 - Dcokerfile*

```docker
FROM ruby:3.2.2
WORKDIR /backend

RUN set -eux && \
    apt-get update -qq && \
    apt-get install -y \
      default-mysql-client

ENV TZ=Asia/Tokyo
COPY Gemfile Gemfile.lock* /backend/
COPY . /backend/

# 本番環境用のbundle installを追加
RUN bundle install --without development test

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

EXPOSE 8080
# イメージ実行時に起動させる主プロセスを設定
CMD ["rails", "server", "-p", "8080", "-b", "0.0.0.0"]
```

## 結果

Cloud Runにデプロイしたコンテナ起動時 `bundle install` が実行されずコールドスタート時のレスポンス速度を改善する事が出来ました。以前はタイムアウトするほどでしたが、修正適応後はタイムアウトせずにレスポンスが返るようになりました。

業務の場合、正確にどれくらいか変更前・後でレスポンスタイムを計測する方が良いのですが、今回は省略させて頂きます。

変更適応前のログを確認すると毎回 `bundle install` が実行されている。

![毎回bundle install](https://github.com/user-attachments/assets/0d680f7a-3053-4f27-bddf-276b985764b4)

変更適用後のログでは `bundle install` が実行されていない事を確認 🎉

![bundle installされなくなった](https://github.com/user-attachments/assets/84f4f0d3-4041-42d5-b2d8-002e83f23db6)

今回の変更はローカル開発環境でコンテナが起動する所まで確認出来ず、ぶっつけ本番でデプロイしたので個人開発とはいえ怖かったです。

理由としては接続するDB先が異なり、設定が本番と開発環境では違うため確認する事が出来ず、このような形を取りました。

やっぱり本番環境と同じインフラ構成でstaging環境があると嬉しいなと思いました。そこまでこのサービスに費用は割けないですが、ユーザが増えて大きいサービスになったら必須だと思いました。

## 最後に

去年は退職して個人開発したり、フリーランスになったり色々あったんですけど、なんとか生き延びてエンジニアとして、お仕事出来ているので嬉しく思います。

周りの方々のおかげです。いつもありがとうございます。

今後はBizRankをたくさんのユーザに使ってもらえるように大型アップデート出来たらと思います。現状のコンテンツだと弱いのでユーザ投稿方とBotのハイブリットに出来たらと考えています。

その前にCloud SQLの料金が高いのでコスト削減のためにTiDBに置き換えられたら開発を長く続けていけるかなと考えています。

### 今年の抱負（おまけ）

- zenn記事の投稿を去年より多く
    - 年末にPC不調でMacのOSをクリーンインストールして開発環境構築したのでそれの手順共有、主に自分用次またクリーンインストールする際の手順書として。僕が考える軽量で最強のM2 Mac開発環境
    - 業務で決済周りの改修を行った事について
    - BizRankの開発内容詳細
    - Cloud SQLからTiDBへの移行、これ絶対面白いと思う
    - 過去に別媒体で書いた記事を再構成して載せる
- 仕事を頑張り会社に貢献してキャリアを積んでいく
    - 高トラフィックシステムのリプレイスがあるから頑張りたい
- Notion SDKのラッパーを開発してライブラリとして公開
    - これを社内システム開発で使いたい
- 副業したい
    - 開発業務する時間までは取れそうにないからプログラミング初学者へのQ&Aみたいな仕事があったら紹介して欲しいです。プログラミング講師や軽微なサイトの運用保守、エンジニアの知識を活かしたヘルプデスク等なら出来ます。その他にもすでに持っている知識を切り売りするみたいなお仕事あれば参加したいです。週に10~16時間程は取れます
- 海外旅行
    - 週の多くを自宅で過ごしていて景色が変わらな過ぎて良くないなと思っている
    - 色んな景色を見に行きたい
- テックカンファレンスに参加
- 社交的になって、なるべく多くの人に会いに行く
    - 前職で仲良くしてくれた同僚、友達に会いにいける時、会いにいく
- ポートフォリオ用のサイトをリニューアル
    - ここからお仕事の依頼とか貰えたりするようになると嬉しい
- バックエンド、インフラ周りに強くなる
    - データ構造、アルゴリズム、ネットワーク辺りを勉強
    - 上手くシステム設計出来るようになる、建築・製造業の哲学がシステム設計・ソフトウェア開発に応用出来るんじゃないかと思ってその辺りの書籍を読んで業務に取り入れてみたい。SmartHRさんではトヨタの生産方式の概念をソフトウェア開発に取り入れており、あえて性能を抑えたデータベースを使用することで、システムのボトルネックを早期に顕在化させ、継続的な改善を促す手法を用いていました。この「意図的な制約」によってシステム全体の効率を向上させるアプローチは、大規模システムを安定的に機能追加しながら運用するための開発哲学で面白いなと思いました。なのでこのような事が書かれている書籍があったらこっそり教えて欲しいです
- 安定して収入を増やしていく
- 筋トレを続ける
- 機械学習・深層学習で使う数学の習得、生成AI周りの仕組みを数学的観点から理解したい

ここまで読んで下さりありがとうございました。とても嬉しいです！

引き続きよろしくお願いします。皆さんの今年の抱負だったりコメント頂けるとても喜びます。

コメントは絶対全部読みます。

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
👨🏻‍💻：[Github](https://github.com/wimpykid719)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/22565/wataru)

でも受け付けています。どこかにはいます。