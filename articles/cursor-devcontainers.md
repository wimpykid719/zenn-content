---
title: "超簡単に社内向け業務で使用するツールの開発環境を Cursor x DevContainersで作成した" # 記事のタイトル
emoji: "👨🏻‍💼" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["vscode", "環境構築", "cursor", "docker", ] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: "2024.10.23"
---

## 最初に
この記事を開いて下さりありがとうございます。
記事にいいね、コメントしてくれると作者がもっとたくさん技術記事書くかもです。

Zennでの記事公開は1年以上ぶりになります。
あれから色々状況が変わりなんやかんやあって、仕事を辞めて8ヶ月に及ぶ個人でのサービス開発、リリースまでして今は業務委託でフリーランスとしてシステム開発をしています。
個人開発に関してはQiitaにリリースしたサービスについて投稿したので、もしよかったらどうぞ。Zennでもその後の運用面について書けたらと思っています。

[退職して560万円相当の工数をかけてお金を稼ぐサービスを開発した
](https://qiita.com/Unemployed_jp/items/32b8612fd47ca4ec76b3)

状況と雇用形態は変わったのですが、環境は変わらず現在もフルリモートで自宅寝室から働いています。
働く時間がフレックスになった事でミーティングがなければ好きな時間に散歩に出かけられるようなったのが嬉しいです。
それで業務委託先は技術ブログを投稿する文化があるので刺激を受けて今回の記事を書いてみました。
内容としてはなんか難しそうで今まで手が出せてなかったDevContainersに挑戦して環境構築をしてみたというものです。



## DevContainersとは

![DevContainer-extension](https://github.com/user-attachments/assets/92367c69-3e5b-4918-96ca-d7a5555d93a0)

マイクロソフトが開発しているVSCode, Cursor用の拡張機能です。

Dockerファイル、docker-composeファイルを `.devcontainer` に用意する事でコンテナ内でVSCodeサーバを起動して、そこにリモートでVSCode, Cursorを接続して開発するというものです。

[公式 - DevContainersインストール先](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

### メリット

下記のようなメリットがあります。

- ローカル環境を汚さずに開発を行うことができます。
- 案件ごとにNode, Ruby, Python等のバージョン切り替え等を気にする事なく開発が出来ます。
- brewでのパッケージ管理が不要になる、xcodeのインストールも不要です。
- ローカル環境で開発環境を構築したようにVSCode, Cursorの拡張機能を利用する事ができます。
- 開発環境を容易に配布出来て、構築もしやすい。

以前はローカル環境を汚したくない一心でDockerコンテナにインストールされたBiome, Jest, RSpecをホスト側から実行するために難解なシェルスクリプトを書いてハック的な方法を用いてましたが、それらも不要になります。

そして、チーム開発においても事前に必要なソフトウェアをDocker、VScode・Cursor、拡張機能としてDev Containersのみに絞れるのでオンボーディングがスムーズになります。

これには参画者、担当者もニッコリです。

## 導入方法

リモートコンテナ内で開発する場合には `.devcontainer` というフォルダが必要になります。

こちらをプロジェクトフォルダのルート配下に作成しましょう。

- .devcontainer
    - .bashrc：bashシェル表示設定をカスタマイズする、現在のブランチ名を表示するための設定
    - devcontainer.json：こちらが本体の設定ファイル必須
    - docker-compose.yml
    - Dockerfile

自分は上記のようなファイルを用意しました。

devcontainer.jsonのみでもコンテナ起動は出来るみたいですが、カスタムして使用したかったのでファイルを用意しました

### .bashrc

現在のgit ブランチが表示されるようにシェルの見た目を変更する設定が書かれています。

作業する際、常に自分がどのブランチを操作しているのかは把握したいので追加してます。

```bash
source /etc/bash_completion.d/git-prompt
PS1='(\[\033[01;32m\]$(conda info --envs | grep "*" | cut -d" " -f1 | sed -e "s/*//g")\[\033[00m\]) [\u@\h \W$(__git_ps1 " (%s)")]\$ '
```

### devcontainer.json

ここではコンテナ起動はdocker-compose.ymlを参照するようにして起動後はnpm installを実行するようにしています。開発環境でパッケージをインストールする必要があったためここでは指定しております。自身がコンテナ起動後に実行したいコマンドを指定して下さい。

Dockerfileで指定しても良いと思います。

下記がサーバで起動したVSCodeの設定ファイル（Cursorの場合もここはVSCodeで大丈夫です）

- extensions：あらかじめインストールしておきたい拡張機能ここではPythonとJupyter NoteをVSCode上で実行できる拡張をインストールします。
- settings：拡張の各種設定になります。ここではJupyter Noteが実行するパスと実際にファイルが配置されている場所のパスを合わせるために `workspaceFolder` を指定しています。なので `./` がルートパスに指定されます。

```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/python
{
	"name": "Jupyter Notebook開発環境",
	// Or use a Dockerfile or Docker Compose file. More info: https://containers.dev/guide/dockerfile
	"dockerComposeFile": "docker-compose.yml",
  "service": "jupyternotebook",
  "workspaceFolder": "./",
  "postCreateCommand": "npm install",
	"customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-toolsai.jupyter"
      ],
      "settings": {
        "jupyter.notebookFileRoot": "${workspaceFolder}"
      }
    }
	}
}

```

### docker-compose.yml

起動したいコンテナ設定

```yaml
version: '3'

services:
  jupyternote:
    build:
      context: ./
      dockerfile: Dockerfile
    container_name: jupyter-note
    working_dir: ./
    stop_signal: SIGINT
    volumes:
      - .:/適選展開したいフォルダを指定して
    ports:
      - "8888:8888" # jupyter notebook用
      - "8000:8000" # node.js用
    stdin_open: true
    # -tを意味する
    tty: true

```

### Dockerfile

Jupyter Notebookの公式イメージをベースにgit vimをインストールします。
これがないと git commit 出来ないです。
その後シェルにブランチ名を表示させる設定を反映させています。
Jupyter Notebookの公式イメージで既に作成された `.bashrc` が存在するためそこの下部に設定を追加します。

```docker
FROM jupyter/base-notebook:latest

USER root
COPY .bashrc /

RUN apt-get update && \
apt-get install -y git vim && \
apt-get clean && \
rm -rf /var/lib/apt/lists/*

USER $NB_UID
RUN cat /.bashrc >> ~/.bashrc && source ~/.bashrc
```

## DevContainers起動方法

![DevContainer-extension](https://github.com/user-attachments/assets/ea0085c4-042a-49b2-982c-599bae6eba49)

左下にある「><」クリックして Reopen in Container をクリックすることでコンテナが起動してコンテナ内のVSCodeサーバに接続してVSCodeターミナルからコンテナを操作可能になるかと思います。

お疲れ様でした。

これでVSCodeの拡張機能の恩恵を受けながらコンテナ内で完結する開発を行う事が可能になりました。

## 補足

コンテナ内でブランチ名が表示されない、またはgit commitする際に `git config --global --add safe.directory *****` を実行しろと表示される際は `git config --global --add safe.directory /作業ディレクトリ` コマンドを実行してフォルダが安全である事を設定して下さい。

## 最後に

今年もあまり技術記事を投稿出来なかったので、なるべく学んだ内容をアウトプットしていけたらと思っています。
生活を安定させていきたいので来年はエンジニアとしてスキルアップを目指しつつ収入を増やしていけたらなと思います。
記事を最後まで読んで下さりありがとうございます。

次は下記の内容について書いていけたら良いなと思ってます。

 - アルゴリズム・データ構造入門
 - コーディングテストでボコボコにされた話
 - 仕事をどのように探した？
 - スキルシート・職務経歴書の書き方
 - 個人開発のその後
 - Webエンジニアのキャリアについて
 - Node.jsの組み込み関数のデバッグ

また新しくポートフォリオ作成と新規でサービス開発をまたしたいものです。

記事にいいね、コメントしてくれると作者が大喜びするかもです！

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。
