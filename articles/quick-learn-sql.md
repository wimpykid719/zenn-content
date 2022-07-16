---
title: "業務で必要になったので急いでSQL入門してみた" # 記事のタイトル
emoji: "📚" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["sql", "データベース", "初心者", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2022.07.16'
---

## 最初に

SQLを知ってる方が良いタスクを貰ったのでSQL・データベースについて勉強する。

主にSQLが読めるようになりたいので、こちらを中心に構文を勉強する。

今まで避けていたので、ほぼ分からない所から学習する。

[SQL超入門コース　合併版｜SQLの超基本的な部分をたった2時間半で学べます【SQL初心者向け入門講座】](https://youtu.be/CJQDh_mJ1as)

サブクエリを取り扱っている

[【SQL入門講座 合併版】SQLの基本をたった1時間で学べます【初心者向けデータベース入門】](https://youtu.be/v-Mb2voyTbc)

## データベース

### テーブル

エクセルみたいな表の事を指す。

データベースは複数のテーブルから構成されている。

### カラム

表で言う縦方向のものでカテゴリ（性別、年齢、名前）みたいなもの

### レコード

表で言う横方向のもので、ユーザそのものに該当する。基本的には一意のデータになる。

## SQL

出来ることは

- データ操作：登録・削除・更新・検索（条件を絞る事も出来る）
- データ定義：テーブル（エクセルみたいな表）このような表形式で管理するデータベースをRDBMSと呼ぶ
- データ制御

### select

表からIDのカラムを取り出す。

```sql
# * 全てのカラムを取得
SELECT * FROM demo;

# 取得するカラムを指定する
SELECT ID FROM demo;

# カンマで区切る事で複数のカラムが指定出来る
SELECT ID, Hint FROM demo;

# asを使用してカラム名を変更する事も可能
SELECT ID, Hint AS ヒント, Name AS 名前  FROM demo;
```

### where

```sql
# IDが2以下のレコードを取得
SELECT * FROM demo WHERE id <= 2;

# 2以外のレコードを取得
SELECT * FROM demo WHERE id != 2;

# idが2のレコードを取得
SELECT * FROM demo WHERE id <= 2; 

# nameがKirill N.を取得
SELECT * FROM demo WHERE name = 'Kirill N.';
```

他に使える演算子

AND演算子はOR演算子よりも優先される

```sql
# idが5以上、7以下
SELECT * FROM demo
WHERE id >= 5 AND ID <= 7

# idが5か売上金額6000以上
SELECT * FROM demo
WHERE id = 5 OR 売上金額 >= 6000
```

ワイルドカード

```sql
# nameカラムでaiのから始まるデータを出力する
SELECT * FROM demo
WHERE name LIKE 'ai%'

# aiの後に任意の6文字が続く
SELECT * FROM demo
WHERE name LIKE 'ai______'

# 最後がrで終わるもの
SELECT * FROM demo
WHERE name LIKE '%r'

# 最後がrで終わるもの以外
WHERE name NOT LIKE '%r'

# ドットをエスケープ文字にしている。対象にしたい文字の前にエスケープに指定した文字列を指定する
WHERE name LIKE 'ai?.%' ESCAPE '?'
```

### between

範囲を指定してデータを出力

```sql
SELECT * FROM demo
WHERE 売上金額 BETWEEN 3000 AND 5000
```

### order by

データの並び替えに使用する

```sql
# レコードを昇順（1, 2, 3...）と番号順に並べる。ASCは省略出来る。
SELECT * FROM demo ORDER BY ID ASC

# レコードを降順（21, 20, 19...）と後ろから並べる。
SELECT * FROM demo ORDER BY ID DESC

# 複数のカラムを指定するnameを降順で優先してその中でidを降順に並び替える。
SELECT * FROM demo ORDER BY name, id DESC;

# 各カラムに並び替えを適応する。
SELECT * FROM demo ORDER BY name DESC, id ASC
```

### group by

```sql
# 同じ名前の売上金額を合計してName, 売上金額カラムを出力する
SELECT Name, Sum(売上金額) FROM demo GROUP BY name

# 売上金額の高い順に出力される
SELECT Name, Sum(売上金額) FROM demo GROUP BY name ORDER BY sum(売上金額) DESC

# 売上金額のグループごとの合計値, 最小値, 最大値を金額の高い順に並び替える
SELECT Name, 
Sum(売上金額), 
min(売上金額), 
max(売上金額) 
FROM demo GROUP BY name ORDER BY sum(売上金額) DESC

# Nameが同じ値の数を数えて出力してくれる
SELECT Name, 
COUNT(売上金額)
FROM demo GROUP BY name ORDER BY sum(売上金額) DESC

# レコードの数を出力
SELECT COUNT(*) FROM demo
```

### having

グルーピングした中からさらに特定の条件を取り出す。

```sql
# グルーピングされた売上金額から5000以上の物を取り出す
SELECT Name, Sum(売上金額)
FROM demo
GROUP BY name
HAVING sum(売上金額) >= 5000
ORDER BY sum(売上金額) DESC
```

whereとの違い

それは実行される順番にある。

1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. ORDER BY

なのでWhereとHavingが同時に使用された場合はWhereで絞られた値でグルーピングされて、その中でHavingが適用される。より複雑な抽出条件を作る事が出来る。

### inner join 内部結合

二つのテーブルのデータを結合して出力出来る。

ONで指定した条件に一致するレコードのみ出力。

```sql
# demoとpersonで同じidのレコードをid, name, genderというカラムで出力する
SELECT A.id, A.name, B.gender
FROM demo as A
INNER JOIN person as B
ON 
A.id = B.id
```

### left join, right join, outer join 外部結合

ONで指定した条件に一致しないレコードも表示。

```sql
# idが同じでないA.idのレコードが表示される。その場合genderの部分はNullになる。
# Aのレコード数データが出力される。Aでマッチしない箇所はNULLになる。
SELECT A.id, A.name, B.gender
FROM demo as A
LEFT JOIN person as B
ON A.id = B.id

# Bのレコード数表示される
SELECT A.id, A.name, B.gender
FROM demo as A
RIGHT JOIN person as B
ON A.id = B.id

# A, Bでマッチしない分も含めて表示する。
SELECT A.id, A.name, B.gender
FROM demo as A
FULL JOIN person as B
ON A.id = B.id
```

### length

```sql
名前のデータ文字数をカラムとして隣に出力する
SELECT name, CHAR_LENGTH(name) 文字数 FROM demo
```

### distinct

重複を排除する

```sql
# nameカラムの重複データを取り除く
SELECT DISTINCT name FROM demo

# 名前の種類を出力する
SELECT COUNT(DISTINCT name) FROM demo
```

distinctとgroup byの違い

重複を排除する処理が異なり

- distinct：値として削除されるので重複のカラムデータの合計値等は求められない。
- group by：重複値は一つにまとめられるだけなので合計値を算出したり出来る。

```sql
# 重複する名前を削除、内部的に重複したレコードの情報を持ってない。（おそらく）
SELECT DISTINCT name FROM demo

# 重複する名前を出力する。見た目は同じだが内部的に重複したレコードの情報を持っている。（おそらく）
SELECT name FROM demo GROUP By name

# これをdistinctに置き換える事は出来ない
SELECT name, SUM(売上金額) FROM demo
GROUP By name
```

### substring

```sql
# hintカラムの最初から3文字目から2文字で00~40以内の文字列を持つレコードを取得する
SELECT * FROM demo WHERE substring("hint", 3, 2) BETWEEN '00' and '40'

# テストデータとして売上日、売上金額、商品分類のカラムを追加
ALTER TABLE demo ADD 売上日 date ;
ALTER TABLE demo ADD 売上金額 integer;
ALTER TABLE demo ADD 商品分類 varchar(20);

# データを投入
UPDATE demo SET (売上日) = ('2022-06-09');
UPDATE demo SET (売上日) = ('2022-06-10') WHERE ID <= 10;
UPDATE demo SET (売上日) = ('2022-07-13') WHERE ID <= 5;

UPDATE demo SET (売上金額) = (4000);
UPDATE demo SET (売上金額) = (8000) WHERE ID <= 10;
UPDATE demo SET (売上金額) = (6000) WHERE ID <= 5;

UPDATE demo SET (商品分類) = ('パソコン系');
UPDATE demo SET (商品分類) = ('ソフトウェア系') WHERE ID <= 10;
UPDATE demo SET (商品分類) = ('ファッション系') WHERE ID <= 5;

# 売上金額を年月でまとめて合計値を出力
SELECT SUBSTRING("売上日", 1, 7) 年月, 
SUM("売上金額") 売上合計 
FROM demo
GROUP BY 年月

# 日ごとに売上合計を出力
SELECT SUBSTRING("売上日", 8) 日, 
SUM("売上金額") 売上合計
FROM demo
GROUP BY 日
```

### 正規表現を使ったSQL

```sql
# sqliteだと~（ちるだが使えない）
# nameカラムでaまたはSから始まる値とマッチさせたい時
SELECT * FROM demo
where "Name" glob '[aS]*'

# nameカラムでaまたはSから始まる値以外とマッチさせたい時
SELECT * FROM demo
where "Name" glob '[^aS]*'

# postgresqlだとこれで正規表現が取り扱える。
# nameカラムで名前にlが使用されているものを出力
SELECT * FROM demo 
WHERE "name" ~ '[l]'

# nameカラムでtestかseverのいずれかを出力
SELECT * FROM demo  
WHERE "name" ~ 'test|server'

SELECT * FROM demo  
WHERE "name" ~ 'limit\s(db|time)'

# データ投入
INSERT INTO demo VALUES(7, 'limits page', 'pagenation', '2022-12-24', 8431)

# nameカラムで空白とそうじゃない場合
SELECT * FROM demo
WHERE "name" ~ 'limit\s?'

# データ投入
INSERT INTO demo VALUES(8, 'aaabc', 'hoge', '2022-12-24', 8431)
INSERT INTO demo VALUES(9, 'aaaabc', 'hoge', '2022-12-25', 2673)

# nameカラムでaを3~4回繰り返す物にマッチ
SELECT * FROM demo
WHERE "name" ~ 'a{3,4}'
```

### round

```sql
# 小数点を取扱えすカラムを追加
ALTER TABLE demo ADD 小数点 NUMERIC;

# データ投入
UPDATE demo SET 小数点 = 0.22234;
UPDATE demo SET 小数点 = 3.335 WHERE ID <= 5;

# 丸める
SELECT 小数点, round(小数点) FROM demo

# 少数第2までを丸める
SELECT 小数点, round(小数点, 2) FROM demo
```

### case

```sql
# 単純ケース式
SELECT *, 
	case round(小数点)
    	WHEN 0 THEN 'D'
        WHEN 3 THEN 'B'
        ELSE 'S'
    END
FROM demo

# 検索ケース式
SELECT *, 
	CASE
    	WHEN 小数点<=1 THEN 'D'
        WHEN 小数点>=3 THEN 'B'
        ELSE 'S'
    END 成績
FROM demo

# case式を入れ子にする
# 売上日が2022-06-01~2022-12-24で該当するのを抽出してそこから成績を付ける。
SELECT *, 
	CASE
    	when 売上日 BETWEEN '2022-06-01' AND '2022-12-24' THEN
        	CASE
          		WHEN 小数点<=1 THEN 'D'
          		WHEN 小数点>=3 THEN 'B'
          		ELSE 'S'
        	END
       ELSE '0'
    END 成績
FROM demo

# 都道府県のエリアごとの人口を算出する
# テストテーブル作成

CREATE TABLE Area (
  pref_name nchar(6),
  population int
)

# テストデータ投入

INSERT INTO Area VALUES('京都', 300);
INSERT INTO Area VALUES('大阪', 900);
INSERT INTO Area VALUES('福岡', 500);
INSERT INTO Area VALUES('佐賀', 100);

# まずはエリアを割り当てたカラムを作成

SELECT
	CASE WHEN pref_name IN ('京都', '大阪') THEN '関西'
    WHEN pref_name IN ('福岡', '佐賀') THEN '九州'
    ELSE NULL
    END AS district,
    population
FROM
	Area

# エリアごとの合計を計算

SELECT
	CASE WHEN pref_name IN ('京都', '大阪') THEN '関西'
    WHEN pref_name IN ('福岡', '佐賀') THEN '九州'
    ELSE NULL
    END AS district,
    sum(population)
FROM
	Area
GROUP BY
	CASE WHEN pref_name IN ('京都', '大阪') THEN '関西'
    WHEN pref_name IN ('福岡', '佐賀') THEN '九州'
    ELSE NULL
    END;
```

### IN句

OR句がたくさんある際にこちらを使用すると手短に可読性もあげて記述できる

```sql
# nameカラムのtest, server, aaabcのいずれかに当てはまるレコードを出力する
SELECT * FROM demo
WHERE name IN ('test', 'server', 'aaabc')

# 上記以外のレコードを出力する
SELECT * FROM demo
WHERE name Not IN ('test', 'server', 'aaabc')

# 別テーブルの条件を抽出する
# 別テーブルから該当するnameを取り出してそのnameを元にIN句を使用する
SELECT * FROM demo
WHERE name IN (SELECT name FROM cardinfo WHERE 年齢 <= 30)
```

### limit

```sql
# 売上日を昇順にして3レコード取得する
SELECT * FROM demo
ORDER By 売上日
LIMIT 3

# 商品分類名で絞って3件をレコードの2件目以降（3件目）から取得する。
SELECT 商品分類,
	sum(売上金額) 売上合計
FROM demo
GROUP BY 商品分類
ORDER By 売上合計 DESC
LIMIT 3 OFFSET 2
```

### サブクエリ

select文を複数記述する

```sql
# テストデータ

CREATE TABLE Items (
  name nchar(20),
  category nchar(20),
  price int
)

INSERT INTO Items VALUES('iPhone12', 'スマホ', 100000);
INSERT INTO Items VALUES('Pixel 5', 'スマホ', 80000);
INSERT INTO Items VALUES('Xperia 5', 'スマホ', 90000);
INSERT INTO Items VALUES('dyson V10', '掃除機', 40000);
INSERT INTO Items VALUES('バルミューダC', 'スマホ', 60000);

# 先にカッコ内のselectが実行され平均以上の価格の商品が選択される
SELECT * FROM Items
WHERE price >= (SELECT AVG(price) FROM Items);

# カテゴリ毎の平均価格
# これよく分からない
SELECT * FROM Items AS i1
# ここで平均価格とpriceを比較i1に現在のpriceのカテゴリが入る。
# そしてそれを現在のi2に入ってるカテゴリと比較してマッチした物の平均を返す。
# 二重ループと同じような処理になっている？
WHERE price >= (SELECT AVG(price) 
                					FROM Items AS i2
               					WHERE i1.category = i2.category
               					GROUP BY category);
```

### データ操作

```sql
# カラムを新たに追加
ALTER TABLE demo ADD 売上金額 integer;

# 追加したカラムでidが5以下の売上金額カラムに2000というデータを入れる
# postgresqlだと売上金額の前にあるカラムも指定しないとエラーになる。
UPDATE demo SET (売上金額) = (2000)
WHERE ID <= 5

# 新規レコードの追加
INSERT INTO person VALUES('女性', 2)

# カラムの型を確認する
PRAGMA table_info(demo); 
```

## 最後に

今回のSQLを必要とするタスクは検索条件の追加でした。

そこはrailsがよしなにやってくれるのですが、それを追加した際、パフォーマンス低下を考え検索速度の検証を行う必要が出ました。

そこで実行されるSQLを発行してその速度を計測したかったので、railsのActive recordのクエリをSQLに変換しました。そこからサービスがApartmentを使用しているので、それに対応したものに書き換える必要があって、SQLを読んでどこを変更するか考える必要があったので勉強しました。そしてその作成したクエリの先頭に`EXPLAIN ANALYZE SELECT...` を付けるとそのSQLが実行された速度等色々な情報が出力されるようになります。

速度も問題なくタスクを終える事が出来ました！

SQL入門の書籍とデータベース設計に関する書籍も買ったのでこの記事に追加で更新していけたらなと思います。

### 参照

オンラインでSQLを実行出来るサービス - 練習のお供に

[SQL Online Compiler - for Data Science](https://sqliteonline.com/)

発行されるSQLの速度計測

[PostgreSQLの実行計画について調べてみた | キャスレーコンサルティング株式会社](https://www.casleyconsulting.co.jp/blog/engineer/259/)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。