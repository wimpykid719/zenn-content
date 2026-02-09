---
title: "【大損中】最も美しいと言われるビットコインの技術論文を徹底解説してみた。Pythonでのサンプルコード付き！" # 記事のタイトル
emoji: "📉" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["bitcoin", "ブロックチェーン", "分散システム", "python"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
date: '2026.02.09'
---

## 最初に
※色んな意味で身を削って書いているので、もしかしたらそのうち非公開にして有料記事にするかもです。
この記事を読むとビットコインの技術論文が読めるようになります。
**また副次的な効果でパチンコ・カジノへ行く人はお金が貯まるようになります。**

最近ビットコインが安くなっているという事で購入したのですが、自分は3ヶ月ほど前から気になっていて、暴落後、1500万円超えてからしまったと思い、1450万円の時にエントリーして、現在（2026年2月7日）1070万円なので大損こいています...今年は700万円くらいまで下がると言われています。相場は完全に終わったという意見が聞こえてきます。

購入場所は手数料が安いのと、取引所・販売所での購入が出来るという事でGMOコインで購入しました。

 - 販売所
    - GMOコインが持っているビットコインを言い値で購入、なので少し高い。さらに購入手数料が0.06なので購入時点で損した所から始まります。ちりつもなので結構こちらも痛い出費です。
    - 自分はこっちで購入してしまった...エントリー値段といい全てミスっております。
 - 取引所
    - GMOコインユーザと直接やり取りして購入、ユーザが少なかったりすると買えなかったりする。

それでせっかく購入したのでエンジニアとしては裏側でどのような技術が使用されているのか気になり、ビットコインの何がすごいのか論文を読む事にしました。
何も知らずにただ買うより仕組みを知っている方が面白いはずです。

## ビットコイン用語
記事で解説する内容とビットコイン関連の記事を読む際に知っておくと理解が進む用語になっています。

 - 二重支払い問題：ビットコインが解決した事、ビットコインと聞くとブロックチェーン、マイニングによって改竄不可なシステムと言われていますが、ビットコインが解決した事は二重支払いの解決であり、同じ紙幣を2回使用する事を信頼のある第三者機関の監視なしで防ぐ仕組みを実現した。なのでこの解決方法を理解出来ればビットコインに関わる技術を理解したとしても過言でない。
 - ノード：ビットコインネットワークに参加しているコンピュータ。個人が所持しているパソコン1台をイメージ。
 - P2P：中央サーバーを介さず、参加者（ピア）同士が直接通信するネットワーク方式。ビットコインでは特定の管理者が存在せず、全ノードが対等に取引情報をやり取りする。個人のパソコン1台1台を繋いで巨大なネットワークにするイメージ。
 - ブロードキャスト：トランザクションやブロックをネットワーク全体に通知すること。今回の記事では触れていないのですが、ブロックが積み上がった際にお知らせを行うものになります。
 - ハッシュ：SHA-256等のハッシュ関数によって暗号化された文字列。改竄の難しい文字列を出力するんだなくらいで。
 - トランザクション：取引内容=コイン
 - PoW（proof of work）：コンセンサスアルゴリズムと呼ばれる物の1つでビットコインではPoWが用いられる。マイニングと呼ばれる作業が発生する仕組みになります。
 - マイニング：PoWの計算をノードが行う事で報酬としてビットコインがもらえる。
 - ブロック：複数のトランザクションを含む要素
 - ブロックチェーン：過去のブロックがハッシュで連結された履歴構造。1つのブロックを書き換えると、以降すべてのブロックが無効になるため改竄耐性が高い。
 - 署名：秘密鍵で生成される暗号学的な証明。このトランザクションを作ったのは確かに自分であることを示す。
 - 公開鍵：秘密鍵から数学的に導出される鍵。署名の正当性を他人が検証するために使われる。
 - UTXO（Unspent Transaction Output）：まだ使われていないトランザクションの出力。ビットコインでは残高という概念は存在せず、UTXOの集合＝所持金額。UTXO自体はネットワーク全体で共有されるものため、アドレスと個人が紐づけられると貯金額がバレてしまう。そのためサトシ・ナカモトは今いくら分のビットコインを所有しているのか等が知られてしまっている。UTXOの集合＝所持金となり、トランザクション=コインになります。論文のプライバシーの章でもあったが、アドレスと個人を紐づける事を推奨していない。また秘密鍵・公開鍵をトランザクション毎に生成するのも良しとしている。
 - ウォレット：署名に必要な秘密鍵を管理するソフトウェア／ハードウェア。実体としてビットコインを「保管」しているわけではなく、UTXOを使う権利（秘密鍵）を管理している。
 - アドレス：公開鍵をハッシュ化・エンコードした「受取先識別子」。銀行口座番号に近いが、残高はアドレスではなくUTXOに紐づく。
 - タイムスタンプサーバー：ある時点でこのデータが存在していた事を証明するもの、アプリケーション開発で記録するタイムスタンプをどうしたら正しいものと判断出来そうかみたいな考え方で読み進めると理解が進む。
 - 半減期：約4年ごと（210,000 ブロック積み上がる毎なため正確に4年とは限らないただ、10分毎にブロックが積み上がる仕様なのでそこまで大きくはずれない）にマイニング報酬が半分になるイベント。この記事では取り扱わない。
 - オフチェーン：ブロックチェーン外で取引を処理する仕組み。高速・低コストだが、最終的な確定はオンチェーンに依存。この記事では取り扱わない。
 - オンチェーン：ブロックチェーン上に直接記録される取引。確定まで時間と手数料がかかるが、最も高い安全性を持つ。この記事では取り扱わない。
 - ライトニングネットワーク：ビットコインには秒間取引がクレジットカード等と比較すると少なくスケーラビリティの問題があるため、それを解決する方法。ビットコインのレイヤー2。参加者間で支払いチャネルを開き、多数の取引をオフチェーンで高速処理する。最終結果のみをオンチェーンに記録することでスケーラビリティ問題を解決する。この記事では取り扱わない。
 - ハッシュレート（ハッシュパワー）：ネットワーク全体の計算能力（採掘の速さ）のこと。ハッシュレートが下がると、マイニングを停止したマシンが増えたことを意味し、ネットワークのセキュリティ（51%攻撃に対する耐性）が低下します。攻撃を防ぐために十分なハッシュレートが維持されることが、分散型システムの安全性の担保となります。この記事では取り扱わない。

## 読み進め方
ビットコインが解決した事、二重支払いの解決方法を理解するにはUTXO・分散型タイムスタンプサーバーの理解が必要です。
そして分散型タイムスタンプサーバーを理解するにはPoW、リンキング、ラウンド方式の理解が必要になります。
分散型タイムスタンプサーバーを構成する３つの技術解説では周辺知識としてブロック、ブロックチェーン、トランザクションを知っておくと良いと思い記事の冒頭で解説しています。

周辺知識として下記の用語を抑えつつ。もしすでにご存じなら下記リストから読み進めても良いです。
 - ブロック
 - ブロックチェーン
 - トランザクション

二重支払い問題を理解するには下記の用語を理解する。
 - UTXO
 - 分散型タイムスタンプサーバー
    - PoW
    - リンキング
    - ラウンド方式

さっそく次から仕組みの理解に入りたいと思います。

## ブロック
ビットコインの技術について話される際に登場する **ブロックチェーン** という言葉あります。一時期流行ったweb3という用語もブロックチェーンの技術を基盤とした分散型インターネットの構築を指しており、最新技術のように感じられますがサトシ・ナカモトの論文の引用を見ると1990年代のものが多く昔から存在する枯れた技術を組み合わせたもので構成されており、名前ほど最新でもない事に驚きました。
論文の公開自体が2008年のリーマンショックの時のため、すでに古いと言っても過言ではないかもですが...

> 1990年代後半に発表されたこのエッセイには、80年代からデイヴィッド・チョウム (David Chaum) らを中心に発展した電子マネー (electronic cash) の概要
> タイムスタンプ（タイムスタンプサーバー）の参考文献4本のうち3本は、米国 Bellcore の研究者だったスチュアート・ヘイバー (Stuart Haber) とスコット・ストルネッタ (Scott Stornetta) らによって書かれたものです。
> 残りの1本は、TIMESEC というタイムスタンプ研究プロジェクトの論文です。TIMESEC は、ベルギーの大学 UCLouvain と KU Leuven の研究者らによって進められた、同国の国家プロジェクトでした。
> 4本とも、発表されたのは1990年代です。

それではブロックの構成要素について見ていきます。
ビットコインのブロックには複数のトランザクションが含まれています。ブロックはブロックチェーンの基本的なデータ構造であり、下記の値が含まれます。

 - ヘッダー
    - バージョン
    - 前のブロックハッシュ値
    - マークルルート
    - タイムスタンプ
    - 難易度ターゲット（Bits）
    - Nonce
 - トランザクション（複数）

ヘッダー全体をハッシュ化した値をブロックの識別子、ブロックハッシュ値として利用して、次に積み上げられるブロックのヘッダーに `前のブロックハッシュ値` として記載される。


### バージョン
ブロックヘッダー先頭の **4バイト整数** で、もともとは「ブロックのルール/機能の世代」を表す目的で使われました。現在のビットコインでは主に、 **ソフトフォーク（後方互換な仕様変更）への賛同を示すシグナリング用のビット列** として使われます（例：BIP9のversionbits）。


### 前のブロックハッシュ値
前のブロックのブロックハッシュ値を指します。これにより、ブロック同士がチェーンのように連結され、ブロックチェーンが形成されます。この値により、ブロックの順序が保証され、過去のブロックを改竄しようとすると、その後のすべてのブロックのハッシュ値が変わってしまうため、改竄が困難になります。

### マークルルート

![マークルルート](https://firebasestorage.googleapis.com/v0/b/blog-bf60d.firebasestorage.app/o/blog-images%2Fbitcoin%2FCuGHSbghET.png?alt=media&token=b35c24a8-37b1-4b02-b6ee-1f185128a683)
[引用：4.4. マークルルートのブロックチェーンでの利用](https://qiita.com/haystacker/items/a7b0c2552e1fd937a789#44-%E3%83%9E%E3%83%BC%E3%82%AF%E3%83%AB%E3%83%AB%E3%83%BC%E3%83%88%E3%81%AE%E3%83%96%E3%83%AD%E3%83%83%E3%82%AF%E3%83%81%E3%82%A7%E3%83%BC%E3%83%B3%E3%81%A7%E3%81%AE%E5%88%A9%E7%94%A8)

トランザクションの改竄を検出するために使用されます。
ブロック内に含まれるすべてのトランザクションをマークルツリーというデータ構造でまとめ、そのルート（根）のハッシュ値を表します。マークルツリーは、複数のトランザクションを効率的にハッシュ化するための二分木構造で、トランザクションが1つでも変更されると、マークルルートの値が変わるため、ブロック内のトランザクションの整合性を保証します。
最小のトランザクション同士でハッシュ化しその後、再びハッシュ化されたトランザクション同士で再度ハッシュ化を繰り返す事で各トランザクションが改竄しにくい形にしている。

### タイムスタンプ
ブロックが作成された時刻をUnixタイムスタンプ（1970年1月1日からの経過秒数）で表します。この値により、ブロックが作成された時間が記録され、ブロックチェーン上での時系列が保証されます。ただし、マイナーが多少の時間を調整することは可能ですが、極端な調整は他のノードに拒否されるため、ある程度の正確性が保たれます。

### 難易度ターゲット（Bits）
マイニングの難易度を決める **ターゲット値（閾値）** を、コンパクト形式で表したものです（`nBits` / compact target）。ブロックハッシュは「このターゲット以下（数値として十分小さい）であること」が求められ、ターゲットが小さいほど難易度が上がります。

「先頭の0が増えるほど難しい」という説明は直感としては合っていますが、正確にはこの `Bits` で表されるターゲット値によって決まります。難易度は約2週間（2016ブロック）ごとに調整され、平均して約10分に1回ブロックが見つかるように自動調整されます。

### Nonce
「Number used once」の略（当座、臨時といった使い捨ての値に使用される）で、一度だけ使用される数値です。マイニング（採掘）の際に、ブロックヘッダーのハッシュ値の先頭に指定された数の0が続くように、このNonceの値を0から順番に増やしながら試行錯誤します。正しいNonce値を見つけることができれば、そのブロックがブロックチェーンに追加される権利を獲得できます。


### Pythonによるブロック・ブロックチェーンのサンプルコード
※補足：この後のサンプルコードには `index`（ブロック高/ブロック番号のようなもの）が登場しますが、 **ビットコインのブロックヘッダーには含まれません。** 実際のノード実装では「ブロック本体」とは別に、チェーン上での位置（高さ）などの **メタデータ** を保持します。ここでは表示や理解のために `index` を持たせていますが、 **ブロックのハッシュ（＝ヘッダー由来の識別子）には含めない** 形にしています。またマークルルートとトランザクションに関する処理も今回は省略しております。本来は1つのブロックに複数のトランザクションが含まれます。

```python
import hashlib
import json
import time
from dataclasses import dataclass


def sha256_hex(text: str) -> str:
    return hashlib.sha256(text.encode("utf-8")).hexdigest()


@dataclass
class Block:
    # 学習用メタデータ（ビットコインのブロックヘッダーには含まれない）
    index: int
    timestamp: int
    data: dict
    prev_hash: str
    nonce: int = 0

    def hash(self) -> str:
        # ブロックの内容が1bitでも変わると、hashが大きく変わる
        payload = {
            "timestamp": self.timestamp,
            "data": self.data,
            "prev_hash": self.prev_hash,
            "nonce": self.nonce,
        }
        return sha256_hex(json.dumps(payload, sort_keys=True, separators=(",", ":")))


class Blockchain:
    def __init__(self, difficulty: int = 3) -> None:
        self.difficulty = difficulty  # 先頭に必要な "0" の数（説明用に簡略化）
        self.chain: list[Block] = [self._create_genesis_block()]

    def _create_genesis_block(self) -> Block:
        # 最初のブロック（前のハッシュがないので固定値にする）
        return Block(
            index=0,
            timestamp=int(time.time()),
            data={"message": "genesis"},
            prev_hash="0" * 64,
        )

    def last_block(self) -> Block:
        return self.chain[-1]

    def mine(self, block: Block) -> Block:
        # Nonce を変えながら hash が条件（先頭0）を満たすまで探索する
        prefix = "0" * self.difficulty
        while True:
            if block.hash().startswith(prefix):
                return block
            block.nonce += 1

    def add_block(self, data: dict) -> Block:
        block = Block(
            index=self.last_block().index + 1,
            timestamp=int(time.time()),
            data=data,
            prev_hash=self.last_block().hash(),
        )
        mined = self.mine(block)
        self.chain.append(mined)
        return mined

    def is_valid(self) -> bool:
        prefix = "0" * self.difficulty
        for i in range(1, len(self.chain)):
            cur = self.chain[i]
            prev = self.chain[i - 1]
            if cur.prev_hash != prev.hash():
                return False
            if not cur.hash().startswith(prefix):
                return False
        return True


if __name__ == "__main__":
    bc = Blockchain(difficulty=3)  # 数字を上げるほど時間がかかる

    bc.add_block({"from": "Alice", "to": "Bob", "amount": 10})
    bc.add_block({"from": "Bob", "to": "Carol", "amount": 3})

    for b in bc.chain:
        print(
            f"index={b.index}, nonce={b.nonce}, hash={b.hash()[:16]}..., prev={b.prev_hash[:16]}..."
        )

    print("valid?", bc.is_valid())

    # 改竄例：3つのチェーンのうち真ん中のブロックのデータを変えると計算したNonceでは000が3つ続かず検証に失敗、hash とつながりが崩れる
    bc.chain[1].data["amount"] = 999
    print("valid after tamper?", bc.is_valid())
```

*実行結果*
```plaintext
index=0, nonce=0, hash=c7881137b58e11a2..., prev=0000000000000000...
index=1, nonce=271, hash=000889da2ba2c605..., prev=c7881137b58e11a2...
index=2, nonce=3100, hash=000b4185672fc452..., prev=000889da2ba2c605...
valid? True
valid after tamper? False
```


## トランザクション
仮想通貨でのやり取りを記録したものであり、下記の値が含まれる。

### 基本の構成要素
 - 利用者の公開鍵
 - ハッシュ
 - その前の取引利用者の署名

### 詳細
*1. 入力（Input）情報*
 - 使用するUTXO（未使用トランザクション出力）への参照
 - 前のトランザクションのハッシュ（TXID）
 - 前のトランザクションの出力インデックス
 - スクリプト署名（ScriptSig）—送信者の署名と公開鍵を含む

*2. 出力（Output）情報*
 - 送金額（Satoshi単位）
 - ロックスクリプト（ScriptPubKey）—受取人の公開鍵ハッシュ（アドレス）を含む
 - 複数の出力が可能（送金先とおつり用）

*3. その他のメタデータ*
 - バージョン番号 — トランザクションの形式バージョン
 - ロックタイム（Locktime） — トランザクションが有効になる時刻/ブロック高
 - 入力数と出力数 — 構造のサイズ情報

![トランザクション](https://firebasestorage.googleapis.com/v0/b/blog-bf60d.firebasestorage.app/o/blog-images%2Fbitcoin-whitepaper%2FkwjNz4D6WO.png?alt=media&token=65642d4c-ec33-4018-b55a-06c37bd3f4fa)

AさんがBさんとコインの取引を行う場合、Aさんの秘密鍵で署名をBさんへの取引に対して行います。それをBさんはAさんの以前の取引に記載されている公開鍵を使い検証する事でその取引がAさんによるものと保証されます。このやり取りを繰り返すと取引履歴がチェーンのように伸びます。この取引一つ一つを **電子コイン（ビットコイン）** と呼ぶ。
つまりこの複雑なやり取りを経てつながった取引履歴を我々はビットコインと呼んでいる。
※これはチェーンになっているのですが、ブロックチェーンとは別のチェーンになる。

ただ電子署名だけでは、同じ複数のトランザクションで使用する試みを防げません。これにより一度使用したコインを重複して使用する事を可能にします。これを **二重支払い問題** と言います。
通常のオンライン決済での対策としては支払いを受け入れる前に銀行側に使用済みのお金で決済を行なっていないか、問い合わせて決済者が支払いが可能か検証する事で防ぐ事が出来ます。口座の残高チェックなどもこれに該当するかと思います。ただこの問い合わせが毎回発生すると顧客を待たせる事になるため、支払を受け入れた後、しばらくしてから預入を行う支払方式があります。これで決済を行うと二重支払いが発覚した時点でお客に商品などが渡っているため売り手側が損害を受けます。

そして、これだと銀行という第三者に頼る必要性が出てくるため、ビットコインでは中央管理者（銀行など）を介さない方法でこの問題を解決しました。

## UTXO（Unspent Transaction Output：未使用トランザクションアウトプット）
ビットコインでは二重支払い問題をUTXO（未使用のトランザクション一覧）、分散型のタイムスタンプサーバーを導入する事で解決しています。

UTXOはトランザクション内のOutputに記載されて誰かに対して支払った記録をまとめた一覧になります。
取引履歴が ~~input~~・~~output~~ → ~~input~~・~~output~~ → ~~input~~・output このように続く場合、最後まで残っていたoutputをUTXOのリストに加えます。そして、トランザクションの署名検証の際にUTXO一覧に含まれているか確認し含まれていれば、未使用のコインとして検証完了とします。
使用したトランザクションはUTXOから取り除かれます。

### Pythonによるブロック・ブロックチェーンのサンプルコード2
先ほどのサンプルコードをさらに発展させトランザクション、UTXO、ウォレットの機能を追加しました。

```python
# トランザクション（UTXO / 署名 / 検証）まで含めた簡易サンプル
# ※`cryptography` 等の外部ライブラリは使わず、標準ライブラリだけで動く学習用です。
# ※署名アルゴリズムは「玩具版RSA署名」（pow(mod n) + SHA-256）で、実用上の安全性はありません。
import copy
import hashlib
import json
import secrets
import time
from dataclasses import dataclass
from typing import Dict, List, Optional, Tuple


def sha256_hex(text: str) -> str:
    return hashlib.sha256(text.encode("utf-8")).hexdigest()


def canonical_json(obj) -> str:
    # 署名/txid の元になる文字列を毎回同じ順序で作る（学習用）
    return json.dumps(obj, sort_keys=True, separators=(",", ":"), ensure_ascii=False)


# -----------------------------
# Toy RSA signature (NOT secure)
# -----------------------------
# ※ここでのRSAは学習用の超簡易実装です（安全性・堅牢性はありません）。
#   - 小さい桁の素数を自前生成（試し割り）
#   - 署名対象は SHA-256(message) を n で割った値
#
# 実際のRSAは桁数が大きく、素数生成、パディング（PSS等）、定数時間実装などが必須です。


def _egcd(a: int, b: int) -> Tuple[int, int, int]:
    if b == 0:
        return (a, 1, 0)
    g, x, y = _egcd(b, a % b)
    return (g, y, x - (a // b) * y)


def _modinv(a: int, m: int) -> int:
    g, x, _ = _egcd(a, m)
    if g != 1:
        raise ValueError("no modular inverse")
    return x % m


def _is_prime(n: int) -> bool:
    if n < 2:
        return False
    if n % 2 == 0:
        return n == 2
    i = 3
    while i * i <= n:
        if n % i == 0:
            return False
        i += 2
    return True


def _random_prime(low: int, high: int) -> int:
    # low..high からランダムに素数を探す（学習用の簡易実装）
    while True:
        n = secrets.randbelow(high - low + 1) + low
        if n % 2 == 0:
            n += 1
        while n <= high:
            if _is_prime(n):
                return n
            n += 2


def _hash_to_int(message: bytes, n: int) -> int:
    h = hashlib.sha256(message).digest()
    return int.from_bytes(h, "big") % n


def rsa_sign(message: bytes, d: int, n: int) -> int:
    m = _hash_to_int(message, n)
    return pow(m, d, n)


def rsa_verify(message: bytes, sig: int, e: int, n: int) -> bool:
    m = _hash_to_int(message, n)
    return pow(sig, e, n) == m


@dataclass
class TxIn:
    prev_txid: str
    prev_index: int
    # ScriptSig（署名と公開鍵）
    signature_hex: str = ""
    pubkey: str = ""  # ここでは公開鍵を "e:n" 形式の文字列として持つ（学習用）

    def outpoint(self) -> Tuple[str, int]:
        return (self.prev_txid, self.prev_index)

    def to_dict(self) -> dict:
        return {
            "prev_txid": self.prev_txid,
            "prev_index": self.prev_index,
            "signature_hex": self.signature_hex,
            "pubkey": self.pubkey,
        }


@dataclass
class TxOut:
    amount: int  # satoshi のつもり（整数）
    # ScriptPubKey（受取人の公開鍵ハッシュをロック条件として持つ、P2PKH風の簡略版）
    pubkey_hash: str

    def to_dict(self) -> dict:
        return {"amount": self.amount, "pubkey_hash": self.pubkey_hash}


@dataclass
class Transaction:
    version: int
    vin: List[TxIn]
    vout: List[TxOut]
    locktime: int = 0

    def signing_message(self, input_index: int) -> bytes:
        """
        各入力を「自分がこのUTXOの所有者である」と署名で証明する。
        ここでは説明簡略化のため、input_index を含めた tx 骨格をそのまま署名対象にする。
        """
        msg = {
            "version": self.version,
            "locktime": self.locktime,
            "vin": [{"prev_txid": i.prev_txid, "prev_index": i.prev_index} for i in self.vin],
            "vout": [o.to_dict() for o in self.vout],
            "signing_input_index": input_index,
        }
        return canonical_json(msg).encode("utf-8")

    def sign_input(self, input_index: int, wallet: "Wallet") -> None:
        message = self.signing_message(input_index)
        sig = rsa_sign(message, wallet.d, wallet.n)
        self.vin[input_index].signature_hex = f"{sig:x}"
        self.vin[input_index].pubkey = wallet.public_key()

    def txid(self) -> str:
        # ここでは簡単のため ScriptSig も含めて txid を作る（現実の仕様はもう少し複雑）
        payload = {
            "version": self.version,
            "locktime": self.locktime,
            "vin": [i.to_dict() for i in self.vin],
            "vout": [o.to_dict() for o in self.vout],
        }
        return sha256_hex(canonical_json(payload))

    def to_dict(self) -> dict:
        return {
            "txid": self.txid(),
            "version": self.version,
            "locktime": self.locktime,
            "vin": [i.to_dict() for i in self.vin],
            "vout": [o.to_dict() for o in self.vout],
        }


class Wallet:
    def __init__(self) -> None:
        # 小さい素数でRSA鍵を作る（学習用）
        p = _random_prime(100_000, 300_000)
        q = _random_prime(100_000, 300_000)
        while q == p:
            q = _random_prime(100_000, 300_000)

        self.n = p * q
        phi = (p - 1) * (q - 1)
        # 数学的に計算効率が良く、安全性が高い値になっている。
        self.e = 65537
        # まれに互いに素でないことがあるので、その場合は作り直す（学習用）
        while _egcd(self.e, phi)[0] != 1:
            p = _random_prime(100_000, 300_000)
            q = _random_prime(100_000, 300_000)
            while q == p:
                q = _random_prime(100_000, 300_000)
            self.n = p * q
            phi = (p - 1) * (q - 1)

        # 秘密鍵生成
        self.d = _modinv(self.e, phi)

    def public_key(self) -> str:
        return f"{self.e}:{self.n}"

    def pubkey_hash(self) -> str:
        # 本物のBitcoinは HASH160(pubkey) だが、ここでは SHA256 のみで説明簡略化
        return sha256_hex(self.public_key())

    def address(self) -> str:
        # 表示用（短く）
        return self.pubkey_hash()[:16]


class UTXOSet:
    def __init__(self) -> None:
        # (txid, vout_index) -> TxOut
        self._utxos: Dict[Tuple[str, int], TxOut] = {}

    def clone(self) -> "UTXOSet":
        u = UTXOSet()
        u._utxos = copy.deepcopy(self._utxos)
        return u

    def add_utxo(self, txid: str, index: int, txout: TxOut) -> None:
        self._utxos[(txid, index)] = txout

    def remove_utxo(self, txid: str, index: int) -> None:
        self._utxos.pop((txid, index), None)

    def find_spendable(self, owner_pubkey_hash: str) -> List[Tuple[Tuple[str, int], TxOut]]:
        return [(k, v) for (k, v) in self._utxos.items() if v.pubkey_hash == owner_pubkey_hash]

    # 残高出力
    # これを見ると未使用のトランザクション=使えるコインという認識で良さそう。
    def balance_of(self, owner_pubkey_hash: str) -> int:
        return sum(v.amount for v in self._utxos.values() if v.pubkey_hash == owner_pubkey_hash)

    def _verify_input(self, tx: Transaction, input_index: int, prev_out: TxOut) -> bool:
        txin = tx.vin[input_index]

        # coinbase（新規発行）等は署名検証しない（簡略化）
        if txin.prev_txid == "0" * 64 and txin.prev_index == -1:
            return True

        if not txin.signature_hex or not txin.pubkey:
            return False

        try:
            e_str, n_str = txin.pubkey.split(":")
            e = int(e_str)
            n = int(n_str)
            sig = int(txin.signature_hex, 16)
        except Exception:
            return False

        # 1) ScriptSig で提示された公開鍵が、ScriptPubKey のハッシュ条件に合致するか
        if sha256_hex(txin.pubkey) != prev_out.pubkey_hash:
            return False

        # 2) 署名検証（「この取引を作ったのは秘密鍵保持者」という証明）
        return rsa_verify(tx.signing_message(input_index), sig, e, n)

    def validate_and_apply(self, tx: Transaction) -> None:
        # 参照するUTXOが存在し、署名が正しく、in>=out であれば適用できる
        total_in = 0
        for idx, txin in enumerate(tx.vin):
            # coinbase（新規発行）を許可（簡略化）
            if txin.prev_txid == "0" * 64 and txin.prev_index == -1:
                continue

            prev = self._utxos.get(txin.outpoint())
            if prev is None:
                raise ValueError("invalid tx: referenced UTXO not found (double spend or unknown)")
            if not self._verify_input(tx, idx, prev):
                raise ValueError("invalid tx: signature/pubkey verification failed")
            total_in += prev.amount

        total_out = sum(o.amount for o in tx.vout)
        if tx.vin and not (tx.vin[0].prev_txid == "0" * 64 and tx.vin[0].prev_index == -1):
            if total_in < total_out:
                raise ValueError("invalid tx: outputs exceed inputs")

        # UTXO更新：入力UTXOを削除し、出力を追加する
        for txin in tx.vin:
            if txin.prev_txid == "0" * 64 and txin.prev_index == -1:
                continue
            self.remove_utxo(txin.prev_txid, txin.prev_index)

        txid = tx.txid()
        for i, out in enumerate(tx.vout):
            self.add_utxo(txid, i, out)


def make_coinbase(to_wallet: Wallet, amount: int, message: str = "coinbase") -> Transaction:
    # 入力なし（新規発行）扱い。実物のBitcoinでは coinbase の扱いはもっと厳密。
    tx = Transaction(
        version=1,
        vin=[TxIn(prev_txid="0" * 64, prev_index=-1)],
        vout=[TxOut(amount=amount, pubkey_hash=to_wallet.pubkey_hash())],
        locktime=0,
    )
    # message は本来 coinbase script に入るイメージだが、ここでは割愛
    return tx


def make_payment(from_wallet: Wallet, to_wallet: Wallet, amount: int, utxo: UTXOSet) -> Transaction:
    # 他のユーザから受け取ったコインを集計するために自分の公開鍵が含まれたトランザクション履歴を集計
    # なぜinではなくoutかと言うと他のユーザが出力したトランザクションが残高になるため。
    spendables = utxo.find_spendable(from_wallet.pubkey_hash())
    # なるべく多くのトランザクションを消費しないでコインを送金出来るようにコインを大きい準にならべる。
    spendables.sort(key=lambda kv: kv[1].amount, reverse=True)

    selected: List[Tuple[Tuple[str, int], TxOut]] = []
    total = 0
    # 送金額を超えるまでコインを集める。
    # 未使用のトランザクション=送金（使える）コインという認識で良さそう。
    for outpoint, txout in spendables:
        selected.append((outpoint, txout))
        total += txout.amount
        if total >= amount:
            break
    if total < amount:
        raise ValueError("insufficient funds")

    # 以前受け取ったトランザクション（コイン）を入力して、送金ユーザへ出力する。
    vin = [TxIn(prev_txid=txid, prev_index=idx) for (txid, idx), _ in selected]
    vout = [TxOut(amount=amount, pubkey_hash=to_wallet.pubkey_hash())]

    change = total - amount
    if change > 0:
        vout.append(TxOut(amount=change, pubkey_hash=from_wallet.pubkey_hash()))

    # 新たに作成したトランザクション
    tx = Transaction(version=1, vin=vin, vout=vout, locktime=0)
    for i in range(len(tx.vin)):
        # 使用するコインの支払いを承認するために持ち主が署名する
        tx.sign_input(i, from_wallet)
    return tx


@dataclass
class Block:
    index: int
    timestamp: int
    transactions: List[Transaction]
    prev_hash: str
    nonce: int = 0

    def hash(self) -> str:
        payload = {
            "timestamp": self.timestamp,
            "prev_hash": self.prev_hash,
            "nonce": self.nonce,
            "tx": [t.to_dict() for t in self.transactions],
        }
        return sha256_hex(canonical_json(payload))


class Blockchain:
    def __init__(self, difficulty: int = 3, genesis_owner: Optional[Wallet] = None) -> None:
        self.difficulty = difficulty
        self.utxo = UTXOSet()
        self.chain: List[Block] = []

        owner = genesis_owner or Wallet()
        # coin_baseはお金の出どころがないコインを指す。ビットコインではマイニングによってcoin_baseが発行される
        genesis_tx = make_coinbase(owner, amount=50, message="genesis")
        self._add_block_internal([genesis_tx], is_genesis=True)

    def last_block(self) -> Block:
        return self.chain[-1]

    def mine(self, block: Block) -> Block:
        prefix = "0" * self.difficulty
        while True:
            if block.hash().startswith(prefix):
                return block
            block.nonce += 1

    def add_block(self, txs: List[Transaction]) -> Block:
        return self._add_block_internal(txs, is_genesis=False)

    def _add_block_internal(self, txs: List[Transaction], is_genesis: bool) -> Block:
        # ブロックに入れる tx を、UTXO ルールで検証してから確定させる（説明用）
        tmp_utxo = self.utxo.clone()
        for tx in txs:
            tmp_utxo.validate_and_apply(tx)

        prev_hash = ("0" * 64) if is_genesis else self.last_block().hash()
        block = Block(
            index=len(self.chain),
            timestamp=int(time.time()),
            transactions=txs,
            prev_hash=prev_hash,
        )
        mined = self.mine(block)
        self.chain.append(mined)
        self.utxo = tmp_utxo
        return mined

    def is_valid(self) -> bool:
        prefix = "0" * self.difficulty
        replay_utxo = UTXOSet()

        for i, b in enumerate(self.chain):
            if i == 0:
                if b.prev_hash != "0" * 64:
                    return False
            else:
                if b.prev_hash != self.chain[i - 1].hash():
                    return False

            if not b.hash().startswith(prefix):
                return False

            # UTXO を先頭から作り直して、途中で矛盾が出ないか確認
            try:
                for tx in b.transactions:
                    replay_utxo.validate_and_apply(tx)
            except ValueError:
                return False

        return True


if __name__ == "__main__":
    alice = Wallet()
    bob = Wallet()
    carol = Wallet()

    bc = Blockchain(difficulty=3, genesis_owner=alice)

    print("alice addr", alice.address())
    print("bob   addr", bob.address())
    print("carol addr", carol.address())
    print("alice balance (after genesis)", bc.utxo.balance_of(alice.pubkey_hash()))

    # Alice -> Bob: 10
    tx1 = make_payment(alice, bob, amount=10, utxo=bc.utxo)
    bc.add_block([tx1])
    print("alice balance", bc.utxo.balance_of(alice.pubkey_hash()))
    print("bob   balance", bc.utxo.balance_of(bob.pubkey_hash()))

    # Bob -> Carol: 3
    tx2 = make_payment(bob, carol, amount=3, utxo=bc.utxo)
    bc.add_block([tx2])
    print("bob   balance", bc.utxo.balance_of(bob.pubkey_hash()))
    print("carol balance", bc.utxo.balance_of(carol.pubkey_hash()))

    print("chain valid?", bc.is_valid())

    # 二重支払い（例）：すでに使ったUTXOをもう一度参照する tx を作ると、UTXO検証で弾かれる
    try:
        # tx1 の 0番出力（Bobの10）を、Aliceが勝手に使おうとする（当然NG）
        bad = Transaction(
            version=1,
            vin=[TxIn(prev_txid=tx1.txid(), prev_index=0)],
            vout=[TxOut(amount=10, pubkey_hash=alice.pubkey_hash())],
            locktime=0,
        )
        bad.sign_input(0, alice)
        bc.add_block([bad])
    except ValueError as e:
        print("double spend blocked:", e)
```

## 分散型タイムスタンプサーバー
タイムスタンプの役割としては「ある時刻にその電子データが存在していたことと、それ以降改竄されていないことを証明する技術」になります。
サトシ・ナカモトは二重使用問題を解決するにはトランザクションの順番を知る必要があると考えました。タイムスタンプは、そのための基本的な枠組みとして、ビットコインに取り入れられています。
一般的なタイムスタンプサーバーは信頼出来る第三者機関TTPによる、タイムスタンプサービスになります。これをビットコインではTTPを介さずに実現しました。
その実現方法が下記3つの技術を構成されています。
 - PoW（その中核メカニズム）:履歴（ブロック列）に“コスト”を載せ、分岐が起きても 最も累積Workが大きい履歴に収束させるためのルール。
 - リンキング（ハッシュで鎖を作る）:過去の改竄を困難にする（過去を変えると後続全部を作り直し）。
 - ラウンド方式（ブロック化）:取引をまとめて周期的に確定することで、P2Pで扱いやすくする（スループット/伝播/合意の単位）。


### PoW（Proof-of-Work）
コンセンサスアルゴリズムと呼ばれるもの1つです。
マイニングと呼ばれる作業が発生する仕組みになります。サンプルコードの `def mine(self, block: Block) -> Block:` は、PoWの中核である「Nonce を探索して難易度条件を満たすハッシュを見つける」処理に相当します。
PoWをネットワークで運用する上では、この探索に加えて「見つかった Nonce を低コストで検証する処理」、一定間隔でブロックが見つかるようにする「難易度調整」、フォーク時に正史を決める「累積Workが最も大きいチェーンを採用するルール」などが関わります。

なぜこちらの処理も必要になるかと言うと、UTXOだけでは二重支払い問題が解決出来ず、「ほぼ同時に発生した2つの取引のどちらが本物か？」という矛盾を正す機能を持っていないからです。これがフォーク時に正史を決めるに該当します。
同時に下記の取引を発生させたとします。

取引A： その1BTCを「お店」に払う
取引B： その1BTCを「自分の別の財布」に送る

地理的に近いノード1は、先に「取引A」を受け取り、「このUTXOはまだ未使用だからOK！」と承認します。
遠くのノード2は、通信の遅延で「取引B」を先に受け取り、「このUTXOはまだ未使用だからOK！」と承認します。
このとき、ネットワーク内で **「正しい歴史」が2つに分裂（分岐）** してしまいます。UTXOの仕組みだけでは、この「どちらを優先すべきか」というルールが決まらないです。
そこでPoWの登場です。

PoWではブロックのヘッダーをSHA256（16進数を2進数にすると256文字になる）で任意のデータから64文字の16進数固定長ハッシュ値を生成した際にハッシュ値の先頭の0が続くようにブロックヘッダーに含まれるNonceの値を変更していく処理。
Nonceを1, 2, 3...と変更していき 0が続くハッシュ値を探す `000fab34...` 続く0が増える度に $16 ^ n$ 16のべき乗になり難易度が上がっていく。
これを現代のPCスペックで10分くらいかかるように調整しながら、スペックの高いPCが出現しても計算時間が一定になるよな仕組みになっている。
>Proof-of-Work の難易度は、1 時間あたりの平均ブロック数を基にした移動平均により定められる。各ブロックがあまりにも速く生成された場合、難易度が増加するのである。
0になるNonceを逐次検索処理を実行するのは大変だが0が指定された回数並んでいるか検証するのは楽な処理が出来上がります。


### リンキング
ハッシュ値同士をハッシュ化する事を繰り返す事で改竄しにくい形にしつつ前後関係を担保する。

![リンキング](https://firebasestorage.googleapis.com/v0/b/blog-bf60d.firebasestorage.app/o/blog-images%2Fbitcoin%2F8mLFm8YLNH.png?alt=media&token=2df39854-831e-41db-b776-81987c9f6623)
ブロックを生成する時にブロックヘッダーに前のブロックハッシュ値を含める事で前後関係がはっきりしてブロックの順番が担保されるようになります。
仮に途中の取引、5番目、6番目の間に偽の取引を含めたい含めたい場合、他の全てのブロックを変更する必要があり、それには再びPoWという膨大な計算が必要となり現実的とは言えなくなります。

### ラウンド方式
トランザクション事にタイムスタンプを発行するのではなく、ある程度のまとまりにして発行する事で取引出来る量を増やしています。
そのため、ブロックには複数のトランザクションが含まれていて、個々の順番は担保されていないのですが、マークルルートによって各トランザクションはハッシュ化により、リンキング（ブロックを生成する時にブロックヘッダーに前のブロックハッシュ値を含める事とは別になる。）されており、改竄は難しい状態になっている。

これら3つの技術を用いてサトシ・ナカモトはTTPを必要としない分散タイムスタンプサーバーを考案しました。
タイムスタンプサーバーと言いながら、正確な時刻を保証するというよりかはブロックの生成された順番を担保する事が出来るようになりました。
これにより、矛盾した重複する取引が存在出来ないようになり、**二重支払い問題の解決** が出来ました。

### 疑問
*PoWによって正史になれなかったブロック内のトランザクションの一部が正しいトランザクションの場合、消えて無くならないのか？*
回答：ならない。
負けたブロックに含まれていたトランザクションはUTXOに含まれているか判定され含まれていたら未使用のトランザクションとして、次のブロックでの承認を待つ事になります。
なぜ二重支払いの片方は「未記録の取引」として扱われないのでしょうか。
UTXOによって使用したコインが追跡されているため別々のトランザクションで同じコインが使用されている場合、どちらかしか有効にならないため。
未記録の取引としては扱われず、そのまま破棄されます。

*誠実なブロックチェーンと攻撃者のブロックチェーンを区別できるのか？*
回答：出来ない
攻撃者のチェーンが誠実な最長のチェーンを追い抜くと二重使用成功に誠実なノードも切り替えて、攻撃者のチェーンを伸ばそうとします。
この動きはブロックチェーンの仕様であるためルール通りです。
これにより誠実なノードが攻撃者のチェーンを伸ばす事もありえます。そのためノードを運営する側が不正なチェーンを作成して伸ばさない人が大多数である事を前提に成り立っているネットワークになります。多くの方が不正を働こうとする場合、誠実なチェーンを伸ばすCPUパワーが分散してしまい、攻撃者が有利になってしまいます。そのためビットコインのネットワークを運営する上で誠実な人間が、二重使用問題を解決するためには不可欠なのです。
先ほどの3つの技術に加え「誠実な人間」が必要不可欠になっています。

### 豆知識
PoWの元祖は迷惑メール対策に使用されていた。
大量のメールを一度に送信出来ないようにメール送信を行う前に時間のかかる問題を解かせて、その結果が正しければメールを受け取りそうでない場合は破棄するという仕組みを考えました。これにより一度に大量の送信は不可になりました。
ただこちらは当選メール等の正規配信者が大量のメールを送信する場合は不便だったため信頼出来る配信者のみにショートカットを公開して大量のメールを送信出来るようにしていました。
シンプルだけど面白い仕組みだなと思いました。
[シンシア・ドワーク (Cynthia Dwork) とモニ・ナオア (Moni Naor)による迷惑メール対策による論文](https://link.springer.com/content/pdf/10.1007/3-540-48071-4_10.pdf)

## 不正なブロックを積み上げるのが難しい数学的根拠

### ギャンブラーの破産問題

あるギャンブラーが、カジノ（または対戦相手）と次のようなゲームを繰り返すとします。

 - ルール： コインを投げ、表が出れば所持金が1円増え、裏が出れば1円減る。
 - 終了条件： 所持金が **0円（破産）になるか、目標金額の$N$円（勝利）** に到達するまで続ける。
 - 設定： ギャンブラーの最初の持ち金を $i$ 円とする。

このとき、 **「目標金額に達する前に、所持金が底をついて破産する確率はどれくらいか？」** を考えるのがこの問題です。

数学的な結論（破産確率の公式）勝つ確率を $p$ 、負ける確率を $q = 1-p$ としたとき、初期資金 $i$ から始めて、目標 $N$ に達する前に破産する確率 $P_i$ は以下のようになります。

① 公平なゲーム（$p = 1/2$）の場合勝つも負けるも五分五分の場合、破産確率は非常にシンプルです。

$$
P_i = 1 - \frac{i}{N}
$$

例： 持ち金が10万円、目標が100万円なら、破産確率は $1 - (10/100) = 90\%$ です。目標額10万円なら破産しないが、目標額が高くなるほど破産する確率が上がるみたいですね。

② 不公平なゲーム（$p \neq q$）の場合カジノ側が少し有利な場合（例：$p = 0.48$）、式は少し複雑になります。

$$
P_i = \frac{(q/p)^N - (q/p)^i}{(q/p)^N - 1}
$$

この式から、わずかな確率の差が、試行回数が増えるほど圧倒的な差となって現れることがわかります。
ここがポイント：なぜ「いつか破産」するのか資金力の差が勝敗を決める相手がカジノ（資金が無限に近い）の場合、こちらがどれだけ勝っていても、一度「運悪く連敗して0になる」とそこでゲームオーバーです。一方でカジノ側はいくら負けてもゲームを続行できるため、 **「無限に続ける＝いつか必ず破産する」** という結論（確率は1）に達します。試行回数が増えれば増えるほど計算結果の確率に結果は収束していくため、カジノが少しでも有利な場合、無限に試行する事で確率に結果は収束していきます。なので試行すればするほど破産に近づいていく、パチンコやギャンブル産業が儲かるわけですね。

### ランダムウォーク
この問題は、数直線上の点（所持金）が右か左に1歩ずつ動く「1次元ランダムウォーク」として捉えられます。左の壁（0円）と右の壁（$N$円）があり、どちらに先にぶつかるか？という **「吸収壁」** の問題として解かれます。

### 攻撃者が出来る事
攻撃者がビットコインに対して出来るのは自身のトランザクション記録を書き換える事で、最近の支払金額を取り返そうとする事のみ。
それには自身のトランザクションを含むブロックを承認されるまで伸ばし続ける必要がある。
善意のチェーンと攻撃者のチェーンの競争は 2 項ランダムウォーク（2パターンの確率）で説明が可能である。
 - 成功イベントは善意のチェーンが1ブロック延長して差が1つ広まる事
 - 失敗イベントは攻撃者のチェーンが1ブロック延長して差が1つ縮まる事

P = 善意のノードが次のブロックを見つける確率
q = 攻撃者が次のブロックを見つける確率
qz = 攻撃者が z ブロックの遅れから追いつく確率

$$
q_z = \begin{cases}
1 & \text{if } p \leq q \\
(q/p)^z & \text{if } p > q
\end{cases}
$$
この式の上は攻撃者が次のブロックを見つける確率の方 >= 善意のノードが次のブロックを見つける確率の場合、攻撃者が必ず勝つと言う意味です。攻撃者の方がハッシュレートが高いと言う事ですね。
下は善意のノードが次のブロックを見つける確率の方が高い場合、承認までのブロック数が多いほど攻撃者が勝てる可能性は低くなっていくと言う意味になります。

ではどれだけの時間待つ必要があるか。

### ポアソン分布
「めったに起こらないことが、ある期間内に何回起こるか」を計算するための確率分布です。
「1時間に平均2回メールが来る」という状況で、「次の1時間にちょうど5回メールが来る確率は？」といった疑問に答えてくれます。
ポアソン分布は、以下のような **「ランダムに発生する事象」** によく使われます。

 - 1日のうちに来店する客数
 - 1年間に特定の交差点で起きる事故の数
 - 1ページあたりの誤字脱字の数
 - 工場で生産された製品の中の不良品の数

ポイントは、**「いつ起こるか予測できないが、平均するとだいたいこれくらい」** という値（平均値）が分かっていることです。

$$
P(X=k) = \frac{\lambda^k e^{-\lambda}}{k!}
$$

 - $P(X=k)$：事象がちょうど
 - $k$ 回起こる確率$\lambda$：一定期間内の平均発生回数
 - $e$：ネイピア数（約2.718）
 - $k!$：$k$ の階乗

$$
P(X=k) = \underbrace{\frac{\lambda^k}{k!}}_{\text{回数の勢い}} \times \underbrace{e^{-\lambda}}_{\text{起こりにくさの調整}}
$$

① $\lambda^k$（ラムダの $k$ 乗）$\lambda$ は「普段の平均」です。もし平均（$\lambda$）が 5 回なのに、実際（$k$）に 10 回起こってほしいなら、$5^{10}$ となり、分子がぐんと大きくなります。つまり、 **「平均的にたくさん起こるなら、たくさん起こる確率も上がるよね」** という勢いを表します。

② $k!$（$k$ の階乗）回数 $k$ が増えるほど、分母の $k!$（$k \times (k-1) \times \dots \times 1$）が猛烈に大きくなります。これがあるおかげで、**「いくら平均が高くても、100回、1000回と極端に多く起こる確率はほぼゼロにする」** というブレーキの役割を果たしています。

kの期待する回数が大きくなるほど回数の勢いの値は小さくなっていく。

$$
{\frac{2^2}{2!}}
$$

この場合、回数の勢いが2=200%になってしまうため、それを調整するために ${e^{-\lambda}}$ を掛けて100%の範囲に調整している。

③ $e^{-\lambda}$（ネイピア数のマイナス $\lambda$ 乗）これがポアソン分布の肝です。$e$ は自然界の成長などに関わる定数（約2.718）です。このパーツは、$k$（実際に起こる回数）に関係なく、平均 $\lambda$ が大きいほど全体を小さくする役割があります。全てのパターンの確率（0回、1回、2回…無限回）を全部足したときに、合計がぴったり 100%（1）になるように調整する重りのようなものです。

$e^{-\lambda}$ は「全パターンの合計を1にする」ための係数ポアソン分布の世界では、起こりうるすべてのケース（0回、1回、2回、3回……無限回）の確率を全部足したときに、ちょうど 1（100%） になる必要があります。数学的に見ると、実は $\frac{\lambda^k}{k!}$ を $k=0$ から無限まで全部足すと、次のようになります。$$\sum_{k=0}^{\infty} \frac{\lambda^k}{k!} = 1 + \lambda + \frac{\lambda^2}{2!} + \frac{\lambda^3}{3!} + \dots = e^\lambda$$この合計値は $e^\lambda$ という値になります。これだと合計が 1 を超えてしまうので、 **あらかじめ全体を $e^\lambda$ で割っておく（＝ $e^{-\lambda}$ を掛ける）** ことで、合計を無理やり 1 に調整しているのです。

実際にさっきの平均2で計算してみると（$\lambda=2$）で、各パターンの $\frac{\lambda^k}{k!}$ を出してみる。

 - $k=0$（0回）: $\frac{2^0}{0!} = \frac{1}{1} = 1$
 - $k=1$（1回）: $\frac{2^1}{1!} = \frac{2}{1} = 2$
 - $k=2$（2回）: $\frac{2^2}{2!} = \frac{4}{2} = 2$
 - $k=3$（3回）: $\frac{2^3}{3!} = \frac{8}{6} \approx 1.33$
 - $k=4$（4回）: $\frac{2^4}{4!} = \frac{16}{24} \approx 0.66$

これらを全部足していくと、最終的に $e^2 \approx 7.389$ になります。
ここに調整役の $e^{-2} \approx 0.135$ を掛けると…2回の確率: $2 \times 0.135 = 0.27$ (27%)全パターンの合計: $7.389 \times 0.135 = 1$ (100%)見事に「確率」として成立する数字になりました。


### どれだけ待てば攻撃者の勝率を無視出来るのか
受け手に支払いを済ませたと暫くの間信じ込ませた後、自分に払い戻そうとしていると仮定する。その場合、受け手はアラートを受信するが、攻撃者はその時点で既に手遅れである事を期待する。
前提条件として、ビットコインでは攻撃者は自分が管理するUTXOについて、いつでも別の支払い先へ送金するトランザクション（いわゆる払い戻しトランザクション）を作成すること自体は可能である。
しかし、正規の支払いトランザクションがまだ存在しない限り、それは競合を生まない単なる通常のトランザクションに過ぎず、二重支払い攻撃において有利にはならない。
二重支払い攻撃が成立するためには、同一のUTXOを入力とする「正規の支払いトランザクション」と「攻撃者自身への支払いトランザクション」という、互いに競合する2つのトランザクションが同時に存在し、それぞれが異なるチェーン上で採掘される必要がある。
したがって、攻撃者が事前に払い戻しトランザクションを作成・採掘しようとしても、正規の支払いトランザクションがブロードキャストされる前であれば競争状態は発生せず、攻撃の成功確率を高めることはできない。
その結果、攻撃者は正規の支払いトランザクションがネットワークに公開された後に初めて、秘密裏に別チェーンを伸ばす形で攻撃を開始することになる。
この時点では、攻撃者の計算資源は常に正規チェーンより不利な状態からスタートするため、二重支払い攻撃の成功確率はナカモト論文で示されているように $q < p$ の条件下では指数的に低下する。

なので大前提として仕様上、攻撃者は常にカジノ・パチンコと同じで不利な状態からスタートする。（実際にお客が不利な状況で行われているか不明ですが、経営を続ける上で多少の確率操作はしていると考察する。）

そして、ブロックの生成は10分ほどと言われているが大体であり多少ランダムになる。
「攻撃者が今何ブロック掘れたか」は観測できない。
平均的な進捗しか分からない。

z = 正直側がzブロック掘る数
λ = zブロック掘る数時間のあいだに、攻撃者が平均で掘るブロック数

$$
λ=z\frac{q}{p}​
$$

正直なチェーンが $z$ ブロック掘る間に、攻撃者が掘り当てるブロック数の平均値になります。
※これはブロック生成の平均が10分に1回とは異なる平均値になります。

これにポアソン分布による各ケースの確率 × そのケースで追いつける確率 = そのケースでの成功確率を表すと以下の式になります。

$$
\sum_{k=0}^{\infty} \frac{\lambda^k e^{-\lambda}}{k!} \cdot \begin{cases}
(q/p)^{(z-k)} & \text{if } k \leq z \\
1 & \text{if } k > z
\end{cases}
$$

なぜ各ケースの確率とそのケースで追いつける確率を掛けるかと言うと、仮に各ケースの確率平均と期待値が同じ100%でも追いつけるかどうかは別の話になります。そのため追いつける確率も考慮しないといけないという事です。

上記の式を分解すると以下になります。

$$
\underbrace{\sum_{k=0}^{z} \frac{\lambda^k e^{-\lambda}}{k!} \cdot (q/p)^{(z-k)}}_{\text{有限個の計算}} + \underbrace{\sum_{k=z+1}^{\infty} \frac{\lambda^k e^{-\lambda}}{k!} \cdot 1}_{\text{無限個の計算が必要}}
$$

問題：k=z+1,z+2,z+3,...k = z+1, z+2, z+3, ...
k=z+1,z+2,z+3,... と

$$
\sum_{k=0}^{\infty} \frac{\lambda^k e^{-\lambda}}{k!} = 1
$$

無限に計算しないといけない。

無限部分を間接的に求める方法を試す。
 - Aが有限
 - Bが無限（無限の計算とは言っても、総和が1になる）

$$B=1−A$$

A：k=0からzまでの有限個の項を足す → 計算可能
k=0から無限までの無限個の項を足す → 計算不可能
B：1−Aで求める → 無限個の計算をせずに済む

これを実際の式に当てはめると以下になる。

$$
\sum_{k=z+1}^{\infty} \frac{\lambda^k e^{-\lambda}}{k!} = 1 - \sum_{k=0}^{z} \frac{\lambda^k e^{-\lambda}}{k!}
$$

こうする事で不可能と思われていた無限の計算部分を求める事が出来た。


追い越すケースも含めた計算式に変形して

$$
\sum_{k=0}^{\infty} \frac{\lambda^k e^{-\lambda}}{k!} \cdot \begin{cases} (q/p)^{(z-k)} & \text{if } k \leq z \\ 1 & \text{if } k > z \end{cases}
$$

$$
= \sum_{k=0}^{z} \frac{\lambda^k e^{-\lambda}}{k!} \cdot (q/p)^{(z-k)} + \sum_{k=z+1}^{\infty} \frac{\lambda^k e^{-\lambda}}{k!} \cdot 1
$$

無限部分の計算を置き換えて、共通項をまとめると以下に式変形が出来る。
$$
1 - \sum_{k=0}^{z} \frac{\lambda^k e^{-\lambda}}{k!} \left(1 - (q/p)^{(z-k)}\right)
$$

*実際のPythonコードにすると*
```python
import math

def attacker_success_probability(q, z):
    """
    q: 攻撃者が次のブロックを生成できる確率
    z: 正当なネットワークが先行しているブロック数
    """
    p = 1.0 - q
    # 期待値 lambda の計算
    lambd = z * (q / p)
    sum_val = 1.0

    for k in range(z + 1):
        # ポアソン分布の確率密度 P(k; lambda) = (lambda^k * exp(-lambda)) / k!
        poisson = math.exp(-lambd)
        for i in range(1, k + 1):
            poisson *= lambd / i

        # 成功確率の積算
        sum_val -= poisson * (1 - pow(q / p, z - k))

    return sum_val

# 実行例
q = 0.1  # 攻撃者のハッシュパワーが10%の場合
z = 2    # 2ブロック先行されている状態
prob = attacker_success_probability(q, z)
print(f"Attack Success Probability (q={q}, z={z}): {prob:.8f}")

q = 0.1  # 攻撃者のハッシュパワーが10%の場合
z = 6    # 6ブロック先行されている状態
prob = attacker_success_probability(q, z)

print(f"Attack Success Probability (q={q}, z={z}): {prob:.8f}")
```

実際に実行すると論文と近い値が出力され、生成済みのブロックとの差が大きくなるたびに攻撃成功率が下がっているのが確認出来る。

```plaintext
# 実行結果
Attack Success Probability (q=0.1, z=2): 0.05097789
```

勝率を0.1以下に抑えるなら2ブロック待つだけで、攻撃者の成功率は低くなる事が分かりました。
そしてサトシ・ナカモトは論文では記載していませんが、コミュニティの質問に対して6承認あれば大半の取引には十分と発言しているため、6ブロック待てば攻撃者の勝率を無視出来ると言え6ブロック分の承認を待つ習慣がビットコインでは生まれました。

実際に６ブロックでの勝率を見るとかなり低い事が証明されました。

```plaintext
# 実行結果
Attack Success Probability (q=0.1, z=6): 0.00024280
```

「攻撃者が裏でどれだけ進んでいるか分からない現実」をポアソン分布でモデル化し、それでもなお6ブロックの承認が十分に安全であることを数学的に示す事が出来ました。

## わかった事
 - ビットコインのネットワーク自体の脆弱性が突かれて、コインが取られるみたいな可能性は低く。今まで仮想通貨が流出するみたいなニュースは販売所・取引所側のセキュリティ問題によるものであった。
 - 思ったより難しくはない昨今のLLMと比較すると複雑なアルゴリズムはなく、基本情報技術者試験に合格出来るなら理解出来る内容になっている。
    - SHA-256と秘密鍵・公開鍵が理解出来れば理解出来る。
 - ビットコインのすごい所
    - 分散型のタイムスタンプサーバー考案。
    - 二重支払いの問題の解決。
    - 支払い取引を改竄しにくい仕組みにした。
    - 大量の電気代を信頼コストにする事、全員で取引履歴を監視する事で第三者の中央管理者不要にした。
 - 電気代とCPU稼働によって生成された取引履歴 = 通貨として機能している。
    - 最初はトランザクションがなくコインが存在しないがマイニングによってコインが発行されていった。
 - マイニングと呼ばれるのはNonceという値を求める処理で電力と時間を要する。
    - 取引履歴をハッシュ化した後にNonceと呼ばれる数値123みたいな値を追加してハッシュ値の000...が続く値をNonceをインクリメントしながら探す。単純な計算量を調整するための処理。
 - 秒間取引が7件しか行えない、Visaなどは4000～6000件行えるため、比較しても大量の取引を処理出来ない。
    - 小売店の電子決済等で利用するにはある程度の取引を別のネットワークで肩代わりして後からブロックチェーンに書き込む等の工夫した利用方法が必要。
    - 大きい金額を動かす取引には利用するのは良さそうだが、小売での使用を行う場合、ブロックチェーンに取引履歴を記載しないため、ビットコインとしての良さが消えてしまう。
 - 販売所で購入した自分のビットコインはサービスを提供した会社が構築したネットワーク内でのやり取りに留まっているため、おそらくブロックチェーン上の取引履歴には乗っておらず、ユーザ自身はブロックチェーンの恩恵を受けてないため、普通にサービスがハッキングされて口座の残高がなくなったりする可能性が多いにある。マウントゴックス事件など。
    - ブロックチェーンの分散性や改竄困難性の恩恵を直接受けるには、外部ウォレットに送金して自身でビットコインを管理する必要がある。
 - 長いブロックチェーンを正しいとする。
    - 多数決によって正史が決まる、そのため51%の悪意ある人が存在する場合、チェーンを書き換える事が出来る。ただそれには膨大なマシンパワーと電力を必要とするため割に合わないため発生する可能性は低い。
 - 仕様上、売り手の方が導入するメリットがある。
    - 取り消し操作などが出来ないため、支払いキャンセルの問い合わせなどのコスト・人件費を削減出来る。
    - 支払い側は秘密鍵の管理や支払い後の取り消しが不可能なためクレジットカード等と比較すると自己責任範囲が大きい。銀行などの第3者介入がないため、消費者を保護する仕組みがビットコインにはない。
 - 現在のビットコインを購入出来るサービス、GMOコイン、コインチェック等の現金を仮想通貨に交換できる販売所・取引所はサトシ・ナカモトの考えるビットコイン世界の外側であり、そこでのセキュリティに関して論文では言及されていない。第三者が勝手に行なっている事業なため、セキュリティ面で不安がある。
    - サトシ・ナカモトとしてはビットコインはコインの送金等に必要な秘密鍵を個人で保管し管理する事を想定している。
    - おそらく販売所・取引所サービスを提供している会社はユーザ1人1人のウォレットを管理しているのではなく会社で大きなウォレットがあって、そこにある通貨の金額内でユーザにDB上で売買した記録を残しており、これはブロックチェーンの外側の記録になっている。そのためビットコインを買ったようで実際にはサービス提供会社のサービスネットワーク内のみで残高が記載されただけになっている。
 - ビットコインを端的に言うと、信頼担保に第三者の存在を必要としない、二重支払いを防止するPayment（支払い）システム、でありそれ以上でもそれ以下でもない。

## 考察
 - サトシ・ナカモトはなぜビットコインを動かさないのか
    - 亡くなってしまったと言われている。
    - 本人は別のアドレスを持っていて、実際にはそちらで動かしていそう。
    - 本人がビットコインを手放そうものならおそらく価値の大暴落が起きるためそれはビットコインの普及にも害があり本人も行いたくないのではないかと思っている。
 - サトシ・ナカモトは日本人なのか
    - 論文のプライバシーに関する項目を見ると個人を特定出来ないように、全然国籍の異なる名前を使用しているのかなと思った。
 - ビットコインは今なぜ暴落しているのか
    - 論文を見る限りセキュリティ上は問題なさそうだが、スケーラビリティの問題、電力消費、取引時間等課題があり一般的な買い物等に普及する事がなさそう。そして証券会社等による実際の通貨の値段を紐づいたやり取りもビットコインとしては想定範囲外の使い方のため1ビット1000万円付くようなものなのかという疑問が残るため、暴落したりしているのではないかと思った。政治的な部分も大きそうですが、この辺りは分からないです。個人的には技術的にはすごいが、なぜここまでの値段が付くのか疑問があります。買っておいてですが...

## 宣伝
Web制作・業務向けWebアプリの開発をお手伝いしています 🤝

「既存の業務がExcel・手作業で限界を迎えている」
「簡単なWebシステムを作りたいが、受託会社に頼むほどではない」

こうしたケース向けに、小〜中規模のWeb制作・業務支援ツールを個人で開発しています。

*対応できる開発例*
 - アルバイト・正社員向けシフト管理システム
 - QRコードを利用した予約・順番待ち管理
 - 社内FAQ・マニュアル対応を自動化する社内問い合わせ削減ツール
 - 自社ブログ／コンテンツ管理用CMS
 - コーポレートサイト・LP制作

要件整理から実装まで一貫して対応できるため、
「何を作ればいいか分からない」状態からでもご相談可能です。

👉 実装例・使用可能な技術スタックはこちら
[ポートフォリオ | 技術ブログ](https://dev-spot-softwareengineer.vercel.app/)

ご相談はX（旧Twitter）のDMからお気軽にどうぞ。
[https://x.com/Unemployed_jp](https://x.com/Unemployed_jp)

※現在は【3月以降】の着手案件を受け付けています。

## 最後に
論文を読んでみて、一般人の自分には美しさはよく分からなかったです。ただ簡素にまとまっているかもと思いました。
そして引用論文の知識が事前にある方はそのまま理解が進んで美しいと感じるのかもと思いました。
一方で色々調べるとサトシ・ナカモトの論文では異常系の説明ばかりで正常系の説明が足りない。
数学的根拠の章でポアソン分布ではなく負の二項分布を使用する方がよいだとか意見があるみたいでした。この辺りを読み進められると一層理解が深まりそうでした。

昔は難しくてよく分からなかった、当時中学生だった自分が大人になってビットコインの仕組みについて理解を深め購入する所まで出来て嬉しかったです。
ただ当時マイニングしていたら今頃は...と思いました。
当時は理解出来ずやり方が分からなかったので余ったPCで参加出来ませんでしたが、あの時理解出来ていればと思い後悔しています。
そして内容は思っていたほど難しい技術やアルゴリズムは出てこなくて安心しました。数式の部分、分散タイムスタンプサーバーでなぜ二重支払い問題が解決出来るのかを直感的に理解出来る所まで噛み砕くのが難しくて挫折しそうでした。
生成AIが無ければ理解出来なかったでしょう。

末端のシステムエンジニアとしてこれからも勉強をして、質の良いソフトウェアを開発出来るように色んな事を学んでいきたいと思います。
色んなシステムの開発をして、いつか色んな会社から引っ張りだこな人になれたらと思います。

ここまで読んで頂きありがとうございました。

## 参照

ビットコインの論文に興味を持ったきっかになった動画、２つで2時間くらいあるけどとても面白かった。

https://www.youtube.com/watch?v=wZJ3TFTtlSg&t=1480s

https://www.youtube.com/watch?v=82YdOlNgQZ8&t=21s

[ビットコイン： P2P 電子通貨システム（日本語翻訳版）](https://bitcoin.org/files/bitcoin-paper/bitcoin_jp.pdf)

ビットコインにまつわる周辺情報などもまとまっていて、とても参考になった。
[ビットコイン： P2P 電子通貨システムの詳細解説した記事](https://www.ogis-ri.co.jp/otc/hiroba/technical/bitcoinpaper/part1.html)
分散型タイムスタンプサーバーへの理解が深まった。
[タイムスタンプの再発見と「いわゆるブロックチェーン」](https://shanematsuo.medium.com/%E3%82%BF%E3%82%A4%E3%83%A0%E3%82%B9%E3%82%BF%E3%83%B3%E3%83%97%E3%81%AE%E5%86%8D%E7%99%BA%E8%A6%8B%E3%81%A8-%E3%81%84%E3%82%8F%E3%82%86%E3%82%8B%E3%83%96%E3%83%AD%E3%83%83%E3%82%AF%E3%83%81%E3%82%A7%E3%83%BC%E3%83%B3-85abb13bdf0e)
[Satoshiが注意深く設定した世界の境界線](https://shanematsuo.medium.com/satoshi%E3%81%8C%E6%B3%A8%E6%84%8F%E6%B7%B1%E3%81%8F%E8%A8%AD%E5%AE%9A%E3%81%97%E3%81%9F%E4%B8%96%E7%95%8C%E3%81%AE%E5%A2%83%E7%95%8C%E7%B7%9A-cde3f8ffa0e4)
[秒間3000～4000取引の処理性能に到達したプライベートブロックチェーン](https://knowledge.sakura.ad.jp/7332)
[ビットコイン： リポジトリ](https://github.com/bitcoin/bitcoin)


記事に関するコメント等は

🕊：[X](https://x.com/Unemployed_jp)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/22565)

でも受け付けています。どこかにはいます。