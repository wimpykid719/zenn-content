---
title: "自宅ネットワークにどんな機器がぶら下がってるか気になったのでNmapを使って調査してみた。Edit
" # 記事のタイトル
emoji: "⛑️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["bash", "tips", "network", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.03.04'
---
## 環境

vagrantでubuntu20.04の仮装環境を作成してそこにnmapをインストールした。

`sudo apt-get update` を実行する。（vagrant 環境なのでsudoを付ける必要がある。）

`sudo apt-get install -y nmap` でオプション `-y` を付けてインストール中のyesを飛ばす。

インストールが上手く行けば `nmap -V` でバージョン情報が表示される。

ローカルにぶら下がっている機器を検索するコマンドは
x = 1（自宅の環境ならたぶん多くは1になる。）

`nmap 192.168.x.0/24` と入力するしばらくすると下記の情報が出力される。

## 192.168.x.0/24とは

そもそも `192.168.x.0/24` は何を意味しているのか気になったので調べてみると

**ブロードキャストアドレス** と呼ばれるものでホスト部が全て1で表現されるアドレスを指す。

ブロードキャストアドレスはホスト部すべてを示すもので、同一のネットワークに存在する全ホストに同一のパケットを送信するときなどに利用される。

nmapでこのネットワークアドレスを指定するとそのネットワーク内に繋がれたIPアドレスを検出してくれる。上記のコマンドでは `192.168.x.0` ~ `192.168.x.255` までの256のホストを検索する事になる。

### IPアドレスについて少し調べた。

> ホスト部が全て1で表現される。

ホスト部がなんなのか分からないので調べてみた。

IPアドレスにはネットワーク部、ホスト部がある。

実際のIPアドレスを元に見ていく。

`192.168.0.2` を二進数に直すと `11000000.10101000.00000000.00000010` となる。

この時サブネットマスクは `255.255.255.0` だったとする。二進数にする。 `11111111.11111111.11111111.00000000` 

このサブネットマスクの二進数に変換した数値の

`1` ：ネットワーク部

`0` ：ホスト部

IP　　　　　　　　`11000000.10101000.00000000.00000010` 

サブネットマスク　`11111111.11111111.11111111.00000000`

これでIPの24桁まではネットワーク部、残り8桁がホスト部という事が分かる。

さらにこのIPアドレスのホスト部 `00000010` を全て0に変えたのが **ネットワークアドレス** と呼ばれるものになる。ここが全て1に変えたのが **ブロードキャストアドレス**になる。

ホスト部を全て0にされたIPアドレス

 `11000000.10101000.00000000.00000000` 

十進数に直すと

`192.168.0.0` になる。

サブネットマスク は両者の境界を表現するために存在する。

先ほどホスト部の数値を全て0にしたのがネットワークアドレスと言ったがもう一つネットワークアドレスと呼ばれる部分があってそれはネットワーク部=ネットワークアドレス使われている場合もあるので注意が必要である。

 `192.168.1.0/24` 

`/24` の部分はプレフィックス記法、CIDR(Classless Inter-Domain routing)などと呼びます。

二進数にした際、左から24桁はネットワーク部という事を示しています。 `/16` の場合は16桁になります。

## ぶら下がっていた機器たち

次にスキャン結果を見ていく。（一応、意味はあんまないと思うけどローカルIPアドレスは伏せておく）

7つの機器？がローカルネットワークにぶら下がっていた。意外に少なかった。

```bash
vagrant@ubuntu-focal:~$ nmap 192.168.x.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2021-03-03 00:09 UTC
Nmap scan report for ntt.setup (192.168.x.x)
Host is up (0.0042s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE
53/tcp   open  domain
80/tcp   open  http
8888/tcp open  sun-answerbook

Nmap scan report for 192.168.x.x
Host is up (0.0069s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap scan report for 192.168.x.x
Host is up (0.014s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap scan report for 192.168.x.x
Host is up (0.0057s latency).
Not shown: 999 closed ports
PORT     STATE SERVICE
8009/tcp open  ajp13

Nmap scan report for 192.168.x.x
Host is up (0.0055s latency).
Not shown: 999 closed ports
PORT     STATE SERVICE
9000/tcp open  cslistener

Nmap scan report for 192.168.x.x
Host is up (0.0038s latency).
Not shown: 994 closed ports
PORT      STATE    SERVICE
21/tcp    filtered ftp
22/tcp    open     ssh
80/tcp    open     http
1900/tcp  open     upnp
20005/tcp open     btx
49152/tcp open     unknown

Nmap scan report for 192.168.x.x
Host is up (0.0060s latency).
Not shown: 998 closed ports
PORT      STATE    SERVICE
14238/tcp filtered unknown
62078/tcp open     iphone-sync

Nmap done: 256 IP addresses (7 hosts up) scanned in 35.52 seconds
```

`Not shown: 997 filtered ports` は一つのローカルIPアドレスで1000のportsを検索してそのうち997はportsが開いてなかった？という意味だと思う。残り3つが検出したものでそのすぐ下に詳細が表示される。

```bash
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

httpでアクセス出来るみたいなので、アクセスしてみた。

![](https://storage.googleapis.com/zenn-user-upload/ynz7rqfv9bk2kegwvq5xhi0p50vf)

ルータのログイン画面が表示された。

他の3つもルータ関係だった。sshが開いてるのはセキュリティ的に良くないと聞く。これでサポートの際に遠隔操作したりするのに使用するのだろうか。

port 9000で開いている `cslistener` はおそらくVirtualBoxの仮装環境だと思われる。

`iphone-sync` は名前からしてiphoneの機能だと思う。詳しくは分からないがセキュリティ的に良くないと書かれた記事をみた。もしかしたら名前は油断させるためでほんとは変なアプリかも知れない。

全く分からないのが `ajp13` である。見覚えがない。調べるとapacheが関係しているみたいでインストールした覚えはないが、何かのアプリに組み込まれているのかも知れない。

**ご存知の方がいらしたら教えて下さると助かります。**

パッと見怪しい機器は見つからなかった。

## 最後に

ここまで記事を読んで頂きありがとうございます。

実際にnmapを使って自宅のローカル環境をスキャンする事で、間接的にIPアドレスとサブネットマスクの関係性について知る事が出来た。

思ってたより、繋がってる機器は少なかった。chromecast、amazonTV、テレビがあると思ったが電源が切れていたのかスキャン結果には出てこなかった。

次回は自宅のネットワーク外からどんなポートが開いているのか調べてみたい。

ハッカーと呼ばれる人達はローカルネットワークに侵入した、次に開いているポート対してエクスプロイトコードを実行して権限を奪うのが主な手法なのか気になる。そもそも外側からどうやって入ってくるのかも気になる。

Wifiならパスワードを見つけて接続すれば可能だと思う。

インターネットからだと、ブロードバンドルータに対してエクスプロイトコードを実行するとか勝手に考えている。

まだまだ知識不足なので、必要だと思ってマスタリングTCP/IPという専門書を読んでいるが、手を動かす事なく用語の説明が多く、内容も退屈なので悲しい事に読んでも頭に入らないのが現状である。

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。

### 参照

[ターゲットの指定 |](https://nmap.org/man/ja/man-target-specification.html)

[ネットワークアドレス (network address)とは｜「分かりそう」で「分からない」でも「分かった」気になれるIT用語辞典](https://wa3.i-3-i.info/word11993.html)

[【図解】IPアドレスの仕組み(クラス分類からサブネットまで) | Enjoy IT Life](https://nishinatoshiharu.com/fundamental-ipaddress/)

[nmap(zenmap)で調べた自宅内の怪しいサービスを何とかせねば！](https://pctips.jp/security/nmap-security202001/)