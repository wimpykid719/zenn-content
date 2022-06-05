---
title: "たった2つのコマンドでNext.js、Rails環境を構築できるようにした。" # 記事のタイトル
emoji: "🎒" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["docker", "nextjs", "rails", "環境構築", "postgresql"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2022.06.05'
---

## 最初に

個人開発でサービスを作成したいので、その開発環境を用意するリポジトリを作成したので設定方法を記事にしました。

Dockerを用いてフロントエンドをNext.js、バックエンドをRailsをAPIモード、DBはPostgresqlを使用する。Railsにしたのは会社のサービスがRailsで作成されていて、勉強の一貫と個人開発の経験が会社のサービス開発する際、スムーズに機能開発出来る手助けになれば良いなと思い選択した。

## 環境構築

何はともあれこれからNext.js、Rails APIモードで開発をしたい方がすぐに開発環境を作れるようにリポジトリにした。

[https://github.com/wimpykid719/nextjs-rails-postgresql-docker](https://github.com/wimpykid719/nextjs-rails-postgresql-docker)

リポジトリを好きなフォルダにcloneする。

### バックエンド

バックエンドのコンテナをビルドして起動するには

```bash
# 初回起動時のコマンド
docker-compose -f docker-compose.backend.yml -p backend up --build
```

このコマンド一つでいい感じに環境を作ってサーバを起動してくれる。

あとは `[localhost:8080](http://localhost:8080)` にアクセスすればrailsにアクセス出来る。

ただしデータベースの接続設定をしていないのでエラーページが返る。

後述で設定する。

バックエンドのログとフロントエンドのログを別々で確認したいので、docker-composeファイルを分けて起動している。そのため `-p backend` でプロジェクト名を付けてないとコンテナが混合していると警告が出る。

環境構築後に使用するコマンド群

```yaml
# ビルド後こちらで起動する
docker-compose -f docker-compose.backend.yml -p backend up

# コンテナに入る際は
docker exec -it backend-rails-api /bin/bash
# そこからDBにアクセスする
# ここからSQL構文で自由にデータ操作出来る
rails dbconsole

# コンテナの削除
docker-compose -f docker-compose.backend.yml -p backend rm
```

データベースの設定

`rails new` によって `config/datebase.yml` が作成されていると思うので下記設定に置き換えるとPostgresqlに接続出来るようになる。再びコンテナを止めて `docker-compose -f docker-compose.backend.yml -p backend up` で起動して  `[localhost:8080](http://localhost:8080)` にアクセスするとrailsのページが今度はエラーなしで返ってくる。所どころ `neumann` と名前が出てくるがこれは個人的にサービス名に使いたい名前なので作りたいサービスに合わせて変更して貰えればと思う。その際は `docker-compose.backend.yml` 等にも記述されているので全てを変更する必要がある。

**config/datebase.yml**

```yaml
# PostgreSQL. Versions 9.3 and up are supported.
#
# Install the pg driver:
#   gem install pg
# On macOS with Homebrew:
#   gem install pg -- --with-pg-config=/usr/local/bin/pg_config
# On macOS with MacPorts:
#   gem install pg -- --with-pg-config=/opt/local/lib/postgresql84/bin/pg_config
# On Windows:
#   gem install pg
#       Choose the win32 build.
#       Install PostgreSQL and put its /bin directory on your path.
#
# Configure Using Gemfile
# gem "pg"
#
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: neumann
  password: password
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: neumann_development

  # The specified database role being used to connect to postgres.
  # To create additional roles in postgres see `$ createuser --help`.
  # When left blank, postgres will use the default role. This is
  # the same name as the operating system user running Rails.
  #username: backend

  # The password associated with the postgres role (username).
  #password:

  # Connect on a TCP socket. Omitted by default since the client uses a
  # domain socket that doesn't need configuration. Windows does not have
  # domain sockets, so uncomment these lines.
  #host: localhost

  # The TCP port the server listens on. Defaults to 5432.
  # If your server runs on a different port number, change accordingly.
  #port: 5432

  # Schema search path. The server defaults to $user,public
  #schema_search_path: myapp,sharedapp,public

  # Minimum log levels, in increasing order:
  #   debug5, debug4, debug3, debug2, debug1,
  #   log, notice, warning, error, fatal, and panic
  # Defaults to warning.
  #min_messages: notice

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: neumann_test

# As with config/credentials.yml, you never want to store sensitive information,
# like your database password, in your source code. If your source code is
# ever seen by anyone, they now have access to your database.
#
# Instead, provide the password or a full connection URL as an environment
# variable when you boot the app. For example:
#
#   DATABASE_URL="postgres://myuser:mypass@localhost/somedatabase"
#
# If the connection URL is provided in the special DATABASE_URL environment
# variable, Rails will automatically merge its configuration values on top of
# the values provided in this file. Alternatively, you can specify a connection
# URL environment variable explicitly:
#
#   production:
#     url: <%= ENV["MY_APP_DATABASE_URL"] %>
#
# Read https://guides.rubyonrails.org/configuring.html#configuring-a-database
# for a full overview on how database connection configuration can be specified.
#
production:
  <<: *default
  database: neumann_production
  username: backend
  password: <%= ENV["BACKEND_DATABASE_PASSWORD"] %>
```

### フロントエンド

Next.jsの環境を作るコンテナ

下記のコマンドを実行すると `[localhost:3000](http://localhost:3000)` でNext.jsに接続出来るようになる。

```bash
# 初回起動時
docker-compose -f docker-compose.frontend.yml -p frontend up --build
```

環境構築後に使用するコマンド群

```bash
# ビルド後こちらで起動する
docker-compose -f docker-compose.frontend.yml -p frontend up

# コンテナに入る際は
docker exec -it frontend-nextjs /bin/bash

# コンテナの削除
docker-compose -f docker-compose.frontend.yml -p frontend rm
```

これで環境構築が出来る。

たった2つのコマンドで環境構築が出来る。一個だけ気になるのが、ctr+cでコンテナを終了する際に `exit 137` でコンテナを終了して正常終了してくれない。調べるとメモリが足りない等の記事が出る。しかし8GBもあげているので別の問題だと思われる。

試しにサーバを起動しない状態でコンテナを起動して `ctr+c` したら正常に終了したので

おそらく rails, next.js等のサーバを起動したままコンテナを終了しているのが原因かと思われる。

会社の開発環境でも `exit 137` で終了していたのでこれは仕方なさそう。解決方法ご存知の方いらしたら教えて下さい。

何がともあれこれでバンバン開発ライフを送れる。

## 各ファイルの解説

ここからは各ファイル群のコマンドを見て行こうと思う。

### バックエンド

Dockerfile

Rubyのイメージを元に作成している。

`entrypoint.sh` で少し複雑な処理をしている。

```dockerfile
FROM ruby:3.1
WORKDIR /backend
RUN set -eux && \
    apt-get update -qq && \
    apt-get install -y \
      postgresql-client

COPY Gemfile Gemfile.lock* /backend

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

# イメージ実行時に起動させる主プロセスを設定
CMD ["rails", "server", "-p", "8080", "-b", "0.0.0.0"]
```

entrypoint.sh

`set -e` エラーが発生するとスクリプトを終了するオプション

次にbundleインストールで Gemfile に書かれたrails バージョン7をインストールする。

次に分岐を用いて `routes.rb` ファイルがなければ rails環境が作られていないと判断して `rails new` を実行する。その後 Gemfile に更新が掛かるので再び `bundle install` する。

 `server.pid` これがあると `rails server` コマンドが実行出来なくなるので事前に存在する場合は削除するようにしている。これが存在する場合はサーバが起動している事にrailsではなっている。

正常に終了が行われなかった際にこのファイルが残る事が多いのでこのような措置を取っている。

なければ無視されるので問題ない。

`exec "$@"` が一番よく分からなかったのですが、Dockerfileに書かれている CMDのコマンドをここで実行してくれるらしい。

二重に実行されるのではという気がしそうなのですが、そんなことはないみたいです。

```bash
#!/bin/bash
set -e

bundle install

if [ ! -e "/backend/config/routes.rb" ]; then
  echo 'rails new APIモード を実行する'
  # --skip入れないとpgのgemないってエラーが出る
  rails new . --force --api --database=postgresql --skip-git --skip-bundle
  bundle install
fi

# Remove a potentially pre-existing server.pid for Rails.
rm -f /backend/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```

docker-compose.backend.yml

ログの出力をフロントエンドと分けたいのでdocker-composeファイルを2つに分けている。

```yaml
# dockerエンジンの対象バージョンに合わせる
version: '3'

services:
  db:
    image: postgres:14.3
    environment:
      POSTGRES_USER: neumann
      POSTGRES_PASSWORD: password
      POSTGRES_DB: neumann_development
      TZ: Asia/Tokyo
    ports:
      - '5432:5432'
    # stopでコンテナを落とすならDBのデータは消えないそうなのであえて永続化しない

  backend:
    build:
      context: ./backend/
      dockerfile: Dockerfile
    container_name: backend-rails-api
    ports:
      - '8080:8080'
    working_dir: /backend
    # こいつのおかげでctr+cした際にrails serverを切ってからコンテナを終了してくれる
    # 137のエラーを解決してくれている
    # 初回起動時のみだった。
    stop_signal: SIGINT
    volumes:
      - ./backend:/backend
    # docker run -iを意味する
    stdin_open: true
    # -tを意味する
    tty: true
```

### フロントエンド

nodeのイメージを使って作成する。

COPYにDockerfileを含めているのは `pacakge.json*` がある場合はコピーない場合は無視するパスを書きたい際に一つは絶対に存在するファイルでないとエラーになるのでなくなくDockerfileをコンテナに渡しています。

`entrypoint.sh` はバックエンドのコンテナとそこまで変わらないですね。

Dockerfile

```dockerfile
FROM node:16.15.0
WORKDIR /frontend

# ワイルドカードで不確定の場合がある際は必ず一つは存在するファイルをCOPYする必要がある。
COPY Dockerfile /neumann-client/package.json* /neumann-client/package-lock.json* /frontend

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

# イメージ実行時に起動させる主プロセスを設定
CMD ["npm", "run", "dev"]
```

entrypoint.sh

Next.jsがインストールされていない環境なら新規インストールする。

既にある場合は `node_modules` をインストールするようにしてある。

そして `exec "$@"` で `npm run dev` を実行して開発環境用のサーバを起動している。

```
#!/bin/bash
set -e

if [ ! -e "/frontend/package.json" ]; then
  echo 'nextjsを新規インストール'
  npm init -y
  npm install create-next-app
  npx create-next-app@latest neumann-client --use-npm --typescript
  rm -rf neumann-client/.gitignore
fi

if [ ! -d "/frontend/neumann-client/node_modules" ]; then
  echo 'neumann-clientの環境構築'
  npm install
fi

cd neumann-client

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```

docker-compose.frontend.yml

```yaml
# dockerエンジンの対象バージョンに合わせる
version: '3'

services:
  frontend:
    build:
      context: ./frontend/
      dockerfile: Dockerfile
    container_name: frontend-nextjs
    working_dir: /frontend
    stop_signal: SIGINT
    volumes:
      - ./frontend:/frontend
    ports:
      - '3000:3000'
    # docker run -iを意味する
    stdin_open: true
    # -tを意味する
    tty: true
```

## 最後に

これで環境構築の区切りが良い所まで出来たので記事にしました。次回は実際にバックエンド側でAPIを建てフロントエンド側のNext.jsで値を受け取れるように出来たらと思います。

### 参照

[DockerでのRuby on Rails環境構築を一つずつ詳解する - Qiita](https://qiita.com/daichi41/items/dfea6195cbb7b24f3419#dockerfile)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。