---
title: "無職が丁寧にubuntu18.04にMongoDBをインストールしたら起動させるのに一日かかった。（コマンドの意味詳細まで調べた。）" # 記事のタイトル
emoji: "⚡" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["linux", "mongodb", "debian", "環境構築", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.03.09'
---
## 最初に

大学の卒業制作で作成した検索エンジンを今の自分のパソコンで動かして見たくなったので、vagrantで作成したubuntu18.04の環境にmongodbをインストールしていこうと思う。

## 環境

- ubuntu18.04 (vagrant)

## インストール手順

wegetでmongodbの公式サイトから認証キーを取得して、apt-key

- -q：コマンドの処理過程を出力しない。
- -O ファイル名：ダウンロードしたファイルの保存先を指定する ファイル名の場所が `-` とするとファイルを保存せず標準出力になる。今回の場合 `server-4.4.asc` という引数が渡される。
- apt-key add ファイル名でダウンロードした認証キーを信頼キーのリストに追加する。ファイル名の部分が `-` の場合は標準入力になる。今回の場合 `server-4.4.asc` が引数で受け取られる。これはもちろんただの文字列ではなくファイルとして生成されていないが、参照すると認証キーを受け取れる。
- `-` この標準入出力を使用する事でファイルを生成せずに、値を受け渡す事が出来る。
- `|` はパイプラインと呼ばれ別々のコマンドを繋げている。今回の場合は左側の `-O -`  で標準出力にして右側で `add -` で標準入力で出力された `server-4.4.asc` を受け取っている。

 

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
```

上手く行くと、出力結果にokと表示される。

次に下記のコマンドを実行する。

- echoで""囲った文字列を標準出力する。
- teeコマンドを使って標準出力をファイルに書き込むと同時にコンソール画面にも出力して確認出来るようにする。
- 実行するとechoの内容 `deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse` が `/etc/apt/sources.list.d/mongodb-org-4.4.list`  に書き込まれると同時にターミナルに出力される。

```bash
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
```

aptの更新をする。

```bash
sudo apt-get update
```

エラーが発生した。google-chromeの32bitバージョンのサポートが打ち切られて、リポジトリにアクセス出来なくなっているために起きていたので、 `google-chrome.list` のサーバ先をコメントアウトしてアクセス出来ないようにした。 `.list` の拡張形式のファイルは先頭に `#` を付ける事でコメントアウトになる。

```bash
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/google-chrome.list:3 and /etc/apt/sources.list.d/google.list:1
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/google-chrome.list:3 and /etc/apt/sources.list.d/google.list:1
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/google-chrome.list:3 and /etc/apt/sources.list.d/google.list:1
W: Target CNF (main/cnf/Commands-amd64) is configured multiple times in /etc/apt/sources.list.d/google-chrome.list:3 and /etc/apt/sources.list.d/google.list:1
W: Target CNF (main/cnf/Commands-all) is configured multiple times in /etc/apt/sources.list.d/google-chrome.list:3 and /etc/apt/sources.list.d/google.list:1
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/google-chrome.list:3 and /etc/apt/sources.list.d/google.list:1
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/google-chrome.list:3 and /etc/apt/sources.list.d/google.list:1
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/google-chrome.list:3 and /etc/apt/sources.list.d/google.list:1
W: Target CNF (main/cnf/Commands-amd64) is configured multiple times in /etc/apt/sources.list.d/google-chrome.list:3 and /etc/apt/sources.list.d/google.list:1
W: Target CNF (main/cnf/Commands-all) is configured multiple times in /etc/apt/sources.list.d/google-chrome.list:3 and /etc/apt/sources.list.d/google.list:1
```

やっとこれでmongodbをインストール出来ると思い下記のコードを実行する。

```bash
sudo apt-get install -y mongodb-org
```

エラーが起きる。

```bash
E: dpkg was interrupted, you must manually run 'sudo dpkg --configure -a' to correct the problem.
```

実行しろみたいに書かれているので実行した。

- Debian Packageの略が名前の由来で「deb」ファイルを取り扱うコマンド
- debはdebianのパッケージである。
- 展開済だが未設定のパッケージを設定する。package の代わりに -a もしくは --pending を指定した場合は、展開済だが未設定のパッケージすべてを設定する。

「展開済みだが未設定のパッケージ」がなぜ出来ていたのか不明である。上記の文の意味が今の知識量では何を言っているのか分からない。

だがとりあえず、これを実行する事でインストール出来るようになった。

※どなたかご存知でしたら教えて下さい。

```bash
sudo dpkg --configure -a
```

実行結果

```bash
Setting up linux-image-4.15.0-109-generic (4.15.0-109.110) ...
depmod: ERROR: failed to load symbols from /lib/modules/4.15.0-109-generic/misc/vboxvideo.ko: Invalid argument
Processing triggers for linux-image-4.15.0-109-generic (4.15.0-109.110) ...
/etc/kernel/postinst.d/initramfs-tools:
update-initramfs: Generating /boot/initrd.img-4.15.0-109-generic
modinfo: ERROR: could not get modinfo from 'vboxvideo': Invalid argument
depmod: ERROR: failed to load symbols from /var/tmp/mkinitramfs_AkT88d/lib/modules/4.15.0-109-generic/misc/vboxvideo.ko: Invalid argument
/etc/kernel/postinst.d/x-grub-legacy-ec2:
Searching for GRUB installation directory ... found: /boot/grub
Searching for default file ... found: /boot/grub/default
Testing for an existing GRUB menu.lst file ... found: /boot/grub/menu.lst
Searching for splash image ... none found, skipping ...
Found kernel: /boot/vmlinuz-4.15.0-108-generic
Found kernel: /boot/vmlinuz-4.15.0-106-generic
Found kernel: /boot/vmlinuz-4.15.0-109-generic
Found kernel: /boot/vmlinuz-4.15.0-108-generic
Found kernel: /boot/vmlinuz-4.15.0-106-generic
Replacing config file /run/grub/menu.lst with new version
Updating /boot/grub/menu.lst ... done

/etc/kernel/postinst.d/zz-update-grub:
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/50-cloudimg-settings.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-4.15.0-109-generic
Found initrd image: /boot/initrd.img-4.15.0-109-generic
Found linux image: /boot/vmlinuz-4.15.0-108-generic
Found initrd image: /boot/initrd.img-4.15.0-108-generic
Found linux image: /boot/vmlinuz-4.15.0-106-generic
Found initrd image: /boot/initrd.img-4.15.0-106-generic
done
```

その後再び

```bash
sudo apt-get install -y mongodb-org
```

実行結果

```bash
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  dh-python libexpat1-dev libllvm8 libpython-all-dev libpython-dev libpython2.7 libpython2.7-dev libpython3-dev libpython3.6-dev linux-headers-4.15.0-106
  linux-headers-4.15.0-106-generic linux-image-4.15.0-106-generic linux-modules-4.15.0-106-generic python-all python-all-dev python-asn1crypto python-cffi-backend python-crypto
  python-cryptography python-dbus python-dev python-enum34 python-gi python-idna python-ipaddress python-keyring python-keyrings.alt python-pip-whl python-pkg-resources
  python-secretstorage python-setuptools python-six python-wheel python-xdg python2.7-dev python3-dev python3-keyring python3-keyrings.alt python3-secretstorage python3-wheel
  python3-xdg python3.6-dev
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  mongodb-database-tools mongodb-org-database-tools-extra mongodb-org-mongos mongodb-org-server mongodb-org-shell mongodb-org-tools
The following NEW packages will be installed:
  mongodb-database-tools mongodb-org mongodb-org-database-tools-extra mongodb-org-mongos mongodb-org-server mongodb-org-shell mongodb-org-tools
0 upgraded, 7 newly installed, 0 to remove and 226 not upgraded.
Need to get 102 MB of archives.
After this operation, 200 MB of additional disk space will be used.
Get:1 https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4/multiverse amd64 mongodb-database-tools amd64 100.3.0-1-g8b223b0a [52.8 MB]
Get:2 https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4/multiverse amd64 mongodb-org-shell amd64 4.4.4 [13.2 MB]
Get:3 https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4/multiverse amd64 mongodb-org-server amd64 4.4.4 [20.4 MB]
Get:4 https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4/multiverse amd64 mongodb-org-mongos amd64 4.4.4 [15.8 MB]
Get:5 https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4/multiverse amd64 mongodb-org-database-tools-extra amd64 4.4.4 [5628 B]
Get:6 https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4/multiverse amd64 mongodb-org-tools amd64 4.4.4 [2888 B]
Get:7 https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4/multiverse amd64 mongodb-org amd64 4.4.4 [3516 B]
Fetched 102 MB in 6s (16.5 MB/s)                                                                                                                                                  
Selecting previously unselected package mongodb-database-tools.
(Reading database ... 189860 files and directories currently installed.)
Preparing to unpack .../0-mongodb-database-tools_100.3.0-1-g8b223b0a_amd64.deb ...
Unpacking mongodb-database-tools (100.3.0-1-g8b223b0a) ...
Selecting previously unselected package mongodb-org-shell.
Preparing to unpack .../1-mongodb-org-shell_4.4.4_amd64.deb ...
Unpacking mongodb-org-shell (4.4.4) ...
Selecting previously unselected package mongodb-org-server.
Preparing to unpack .../2-mongodb-org-server_4.4.4_amd64.deb ...
Unpacking mongodb-org-server (4.4.4) ...
Selecting previously unselected package mongodb-org-mongos.
Preparing to unpack .../3-mongodb-org-mongos_4.4.4_amd64.deb ...
Unpacking mongodb-org-mongos (4.4.4) ...
Selecting previously unselected package mongodb-org-database-tools-extra.
Preparing to unpack .../4-mongodb-org-database-tools-extra_4.4.4_amd64.deb ...
Unpacking mongodb-org-database-tools-extra (4.4.4) ...
Selecting previously unselected package mongodb-org-tools.
Preparing to unpack .../5-mongodb-org-tools_4.4.4_amd64.deb ...
Unpacking mongodb-org-tools (4.4.4) ...
Selecting previously unselected package mongodb-org.
Preparing to unpack .../6-mongodb-org_4.4.4_amd64.deb ...
Unpacking mongodb-org (4.4.4) ...
Setting up mongodb-org-shell (4.4.4) ...
Setting up mongodb-org-database-tools-extra (4.4.4) ...
Setting up mongodb-database-tools (100.3.0-1-g8b223b0a) ...
Setting up mongodb-org-mongos (4.4.4) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Setting up mongodb-org-tools (4.4.4) ...
Setting up mongodb-org-server (4.4.4) ...
Adding system user `mongodb' (UID 111) ...
Adding new user `mongodb' (UID 111) with group `nogroup' ...
Not creating home directory `/home/mongodb'.
Adding group `mongodb' (GID 115) ...
Done.
Adding user `mongodb' to group `mongodb' ...
Adding user mongodb to group mongodb
Done.
Setting up mongodb-org (4.4.4) ...
```

インストール後にバージョンを確認するコマンドを実行する。

無事に `db version v4.4.4` と出力されインストールされたと考える。

```bash
mongod --version
```

## 起動

過去の記憶によると

```bash
mongod
```

これで起動するはず。

出力結果

```bash
{"t":{"$date":"2021-03-09T07:50:33.535+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"main","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}
{"t":{"$date":"2021-03-09T07:50:33.540+00:00"},"s":"W",  "c":"ASIO",     "id":22601,   "ctx":"main","msg":"No TransportLayer configured during NetworkInterface startup"}
{"t":{"$date":"2021-03-09T07:50:33.541+00:00"},"s":"I",  "c":"NETWORK",  "id":4648601, "ctx":"main","msg":"Implicit TCP FastOpen unavailable. If TCP FastOpen is required, set tcpFastOpenServer, tcpFastOpenClient, and tcpFastOpenQueueSize."}
{"t":{"$date":"2021-03-09T07:50:33.542+00:00"},"s":"I",  "c":"STORAGE",  "id":4615611, "ctx":"initandlisten","msg":"MongoDB starting","attr":{"pid":2435,"port":27017,"dbPath":"/data/db","architecture":"64-bit","host":"ubuntu-bionic"}}
{"t":{"$date":"2021-03-09T07:50:33.542+00:00"},"s":"I",  "c":"CONTROL",  "id":23403,   "ctx":"initandlisten","msg":"Build Info","attr":{"buildInfo":{"version":"4.4.4","gitVersion":"8db30a63db1a9d84bdcad0c83369623f708e0397","openSSLVersion":"OpenSSL 1.1.0g  2 Nov 2017","modules":[],"allocator":"tcmalloc","environment":{"distmod":"ubuntu1804","distarch":"x86_64","target_arch":"x86_64"}}}}
{"t":{"$date":"2021-03-09T07:50:33.542+00:00"},"s":"I",  "c":"CONTROL",  "id":51765,   "ctx":"initandlisten","msg":"Operating System","attr":{"os":{"name":"Ubuntu","version":"18.04"}}}
{"t":{"$date":"2021-03-09T07:50:33.543+00:00"},"s":"I",  "c":"CONTROL",  "id":21951,   "ctx":"initandlisten","msg":"Options set by command line","attr":{"options":{}}}
{"t":{"$date":"2021-03-09T07:50:33.545+00:00"},"s":"E",  "c":"STORAGE",  "id":20557,   "ctx":"initandlisten","msg":"DBException in initAndListen, terminating","attr":{"error":"NonExistentPath: Data directory /data/db not found. Create the missing directory or specify another path using (1) the --dbpath command line option, or (2) by adding the 'storage.dbPath' option in the configuration file."}}
{"t":{"$date":"2021-03-09T07:50:33.545+00:00"},"s":"I",  "c":"REPL",     "id":4784900, "ctx":"initandlisten","msg":"Stepping down the ReplicationCoordinator for shutdown","attr":{"waitTimeMillis":10000}}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"COMMAND",  "id":4784901, "ctx":"initandlisten","msg":"Shutting down the MirrorMaestro"}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"SHARDING", "id":4784902, "ctx":"initandlisten","msg":"Shutting down the WaitForMajorityService"}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"NETWORK",  "id":20562,   "ctx":"initandlisten","msg":"Shutdown: going to close listening sockets"}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"NETWORK",  "id":4784905, "ctx":"initandlisten","msg":"Shutting down the global connection pool"}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"STORAGE",  "id":4784906, "ctx":"initandlisten","msg":"Shutting down the FlowControlTicketholder"}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"-",        "id":20520,   "ctx":"initandlisten","msg":"Stopping further Flow Control ticket acquisitions."}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"NETWORK",  "id":4784918, "ctx":"initandlisten","msg":"Shutting down the ReplicaSetMonitor"}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"SHARDING", "id":4784921, "ctx":"initandlisten","msg":"Shutting down the MigrationUtilExecutor"}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"CONTROL",  "id":4784925, "ctx":"initandlisten","msg":"Shutting down free monitoring"}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"STORAGE",  "id":4784927, "ctx":"initandlisten","msg":"Shutting down the HealthLog"}
{"t":{"$date":"2021-03-09T07:50:33.546+00:00"},"s":"I",  "c":"STORAGE",  "id":4784929, "ctx":"initandlisten","msg":"Acquiring the global lock for shutdown"}
{"t":{"$date":"2021-03-09T07:50:33.547+00:00"},"s":"I",  "c":"-",        "id":4784931, "ctx":"initandlisten","msg":"Dropping the scope cache for shutdown"}
{"t":{"$date":"2021-03-09T07:50:33.548+00:00"},"s":"I",  "c":"FTDC",     "id":4784926, "ctx":"initandlisten","msg":"Shutting down full-time data capture"}
{"t":{"$date":"2021-03-09T07:50:33.548+00:00"},"s":"I",  "c":"CONTROL",  "id":20565,   "ctx":"initandlisten","msg":"Now exiting"}
{"t":{"$date":"2021-03-09T07:50:33.548+00:00"},"s":"I",  "c":"CONTROL",  "id":23138,   "ctx":"initandlisten","msg":"Shutting down","attr":{"exitCode":100}}
```

何かおかしい。

確か起動出来たらサーバが起動してるみたいに通常のコマンド入力を受け付けない形になるはずなんだけど、コマンドが入力出来る。

何かがおかしい。

プロセスを確認する。

```bash
ps -ef | grep mongod
```

出力結果

```bash
vagrant   2449  1626  0 08:03 pts/0    00:00:00 grep --color=auto mongod
```

- grep --color=auto mongo は検索結果に関わらず表示される。
- 特にmongodのプロセスは起動してない。

## 解決策

原因はデフォルトでストーレージにする予定の `/data/db` フォルダがないという事だった。

なぜこんな事になっているかというとubuntuでは `apt-get` でインストールしたため、dataを保存する予定のパスが `/var/lib/mongodb` に作られてしまうためらしい。

```bash
"error":"NonExistentPath: Data directory /data/db not found. Create the missing directory or specify another path using (1) the --dbpath command line option, or (2) by adding the 'storage.dbPath' option in the configuration file."
```

### 解決策1

mongod.confに `dbpath=/data/db` の一行を追加する。

**mongod.conf**

```bash
# mongod.conf
  
# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true

# この辺にでも追加しておけば良いと思う。
# それか直接上のdbpathを上書きすれば良いと思う。
dbpath=/data/db

#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1

# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo

#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options:

#auditLog:

#snmp:
```

その後、 `/data/db` のフォルダを手動で作成する。

```bash
vi /etc/mongod.conf
```

### 解決策2

起動時のdbパスを指定する。

```bash
sudo mongod --dbpath /var/lib/mongodb
```

個人的にはファイルの生成とか書き換えがなくて良い気がする。

エイリアス等で指定しておけばいいと思うが、朝から格闘してここで力尽きた。

## 最後に

ただmongodbを起動したかっただけなのに、どうしてこんな面倒なんだ。

パスの問題はどうにかして欲しい。

そもそもubuntuに `apt-get` インストールして使用することを想定してないのか?（そんなバカな事ないよなドキュメント通りにやっての `apt-get` だし）

ドキュメントには `data/db` フォルダを手動で作成して下さいなんて書かれてなかった。

それに systemd (systemctl)： `sudo systemctl start mongod` と System V Init (service)：`sudo service mongod start` と書かれていて意味が分からなかった。最近は `mongod` でデータベースを起動しないのか??

`systemctl` を使用するとデータベースが起動したあとに、そのままコマンド入力出来るみたい。おそらく `mongod` と変わらないがログの出力がファイルにされるようだ。

`service` との違いは詳しくは分からないが `systemctl` が使えるなら標準になっているためそちらを使用した方が良いらしい。

※もしご存知の方がいたら教えて下さい。

最終的にはとりあえず、Mongodbを起動することが出来て良かった。まだ大学で制作した検索エンジンのクローラーを動かして、データベースに保存するまでの動作確認は出来ていないが、明日そこまで行おうと思う。

なぜSQL構文をまともに書けないのに、Nosqlでデータベースを始めたのか当時の自分に疑問が残る。
当時はまだNosqlという概念が珍しくてモダンな環境って言葉に惹かれたんだと思う。
エンジニア目指してるんだったら、SQL構文についてしっかり勉強しようと思う。

この一日格闘した記事が誰かのお役に立てればこれほど嬉しい事はない。



記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。

### 参照

[Install MongoDB Community Edition on Ubuntu - MongoDB Manual](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)

[Linuxにおけるハイフンの意味](https://www.nemotos.net/?p=996)

[apt-key(8) - apt - Debian unstable - Debian Manpages](https://manpages.debian.org/unstable/apt/apt-key.8.ja.html)

[標準入力、標準出力、標準エラー出力、パイプとは ?](https://www.creatology.jp/unix/outin.html)

[Debianのsources.listの編集方法](https://www.garunimo.com/program/linux/column-linux12.php)

[](https://qiita.com/on-vegetable/items/8f821be0641d0dbb7631)

[dpkg(1) - dpkg - Debian jessie - Debian Manpages](https://manpages.debian.org/jessie/dpkg/dpkg.1.ja.html)

[MongoDB の dbpath を変更した時のトラブルシューティング - Qiita](https://qiita.com/thirdpenguin/items/4711e28f46cd9c7b09e4)

[What is the default database path for MongoDB?](https://stackoverflow.com/questions/12738322/what-is-the-default-database-path-for-mongodb)

[Linuxのコマンド実行で使うserviceとsystemctlの違いとは何か？](https://www.toumasu-program.net/qfr8l41pigu2v05ztwbc)