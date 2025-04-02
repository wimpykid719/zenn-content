---
title: "ライブラリ無しで実装するReact無限スクロール" # 記事のタイトル
emoji: "🌀" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["react", "html", "css", "javascript", "frontend"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: "2025.02.17"
---

## 最初に

![テーブルレイアウト無限スクロール](https://github.com/wimpykid719/react-component/assets/23703281/20575578-6a46-418c-8e10-9adde45e0bf8)

昔業務で大きめな新機能追加で、新規画面実装を行いました。

そこで table 要素を使って組んだレイアウトに無限スクロール機能 + ユーザが選択した要素を別テーブルで保持する機能実装を行いました。
文字にするとややこしいですが、テーブルレイアウトが2つ横並びになっている少し変わったUIです。

こちらを外部UIライブラリを使わず、Reactのみでどのように実装したか紹介しようと思います。
非機能要件もあり、パフォーマンスにもこだわりました。

## 仕様

まず実装前にデザイナーさんから Figma で作成された UI と共にこんな感じの要望がありました。

テーブルのレイアウトが 2 つ並んでいる。

1 つ目に追加したい要素の一覧があってクリックしたらチェックボックスにチェックが入り、2 つ目のテーブルにバツアイコンと共に表示される。

追加された値を消すにはバツアイコンを押すか、チェックボックスをオフにするかという少し複雑な操作方法が出来る内容でした。

2 つのテーブルは運命共同体だ！！みたいな感じでした。

それだけだと、実装するのは難しかったため、デザイナーさんと話し合いながら細かい仕様を決めていきました。

- API から一覧の情報が返るので、それを 1 つ目のテーブルに表示
- 一度に表示される値は 10 件
- テーブルはデータ追加に伴い、一定の高さを超えるとデータをスクロールして閲覧する形にする
- 検索結果が 10 件以上の場合は下までスクロールされた際に追加で 10 件取得
- 1 万件までは 1 つ目のテーブルに追加されることを想定
- 2 つ目のテーブルには多くても 10 件ほどしか追加されない
- 取得した要素がクリックされたらチェックボックスを ON する
- チェックボックスが ON になったら、2 つ目のテーブルにクリックした値をバツマークと共に追加
- チェックボックスを OFF にすると、2 つ目のテーブルから OFF にした値が消える
- バツマークをクリックすると、2 つ目のテーブルからクリックした要素が消え、1 つ目テーブルにある同じ値のチェックボックスを OFF にする

こんな感じにある程度、仕様を固めて実装に入りました。

## 実装

業務で書いたコードをそのまま載せる訳にはいかないので、ポケモン API を使って似たような機能を実装しました。

[リポジトリ - テーブルレイアウト無限スクロール](https://github.com/wimpykid719/react-component)

こちらの実装したコードを見ながら解説出来たらと思います。

```bash
# cloneして
git clone https://github.com/wimpykid719/react-component.git

# パッケージのインストール
npm install

# アプリ起動
npm start
```

### データ構造

無限スクロールによって、取得するデータがどんどん追加されていくため全て配列で保持して操作するとデータが追加されるたびにチェックボックスの状態変更の処理が重くなっていくと考えました。

なのでデータ保持を配列 x 2, 連想配列（オブジェクト） x 1 の 3 つに分けて保持するようにしました。

イメージとしては

```jsx
// 選択可能なポケモン一覧
// テーブル1に表示する選択肢一覧の並び順と連想配列でデータ操作する際のキーを配列で保持している
// この配列をmapしながらデータを表示
const pokemonNames = ['キー1', 'キー2', 'キー3', 'キー4'...]

// 連想配列にテーブル1で必要なデータを保持
// こうする事で選択可能なポケモンが増えていっても、データ操作はキーを使って行えるので、配列操作よりも早い
const pokemonObj = {
 キー１: {
         name: 'ポケモン名前',
         url: '詳細なURL',
         checked: false,　// チェックボックスの状態
        },
 キー2: {
         name: 'ポケモン名前',
         url: '詳細なURL',
         checked: false,
        },
 キー3: {
         name: 'ポケモン名前',
         url: '詳細なURL',
         checked: false,
        },
  キー4: {
         name: 'ポケモン名前',
         url: '詳細なURL',
         checked: false,
        },
}

// 選択中のポケモン一覧
// こちらはそこまでデータが追加されない仕様のため配列で保持している
const checkedPokemons = [
 {name: 'ポケモン名前', url: '詳細なURL'},
 {name: 'ポケモン名前', url: '詳細なURL'},
 {name: 'ポケモン名前', url: '詳細なURL'},...
]
```

こんな感じにすることでデータ量が増えていってもチェックボックスの状態を操作する処理はそこまで遅くならないと考えました。

### テーブルレイアウト - 無限スクロール

1 つ目のテーブルレイアウト

```tsx
<TableWrapper onScroll={onScroll}>
  <Table ref={tableRef}>
    <Thead>
      <Tr className="TrTh">
        <Th>ポケモン名</Th>
        <Th>詳細URL</Th>
      </Tr>
    </Thead>
    <Tbody>
      {pokemonNames.length === 0 ? (
        <Tr>
          <Td colSpan={2}>
            <NoPokemon>
              {isError
                ? "ネットワークエラー、ポケモンを取得できません"
                : "ポケモンが表示されます"}
            </NoPokemon>
          </Td>
        </Tr>
      ) : (
        pokemonNames.map((name) => (
          <SerachedTr
            className="TrTd"
            key={name}
            onClick={
              pokemonObj[name]?.checked
                ? () => removePokemon(pokemonObj[name]?.name)
                : () => addPokemon(pokemonObj[name])
            }
          >
            <Td>
              <Checkbox
                checked={!!pokemonObj[name]?.checked}
                label={pokemonObj[name]?.name || ""}
              ></Checkbox>
            </Td>
            <Td>{pokemonObj[name]?.url}</Td>
          </SerachedTr>
        ))
      )}
      {!isLoading && (
        <Tr>
          <TdLoading colSpan={2}>
            <LoaderWrapper>
              <LoadingCircle />
            </LoaderWrapper>
          </TdLoading>
        </Tr>
      )}
    </Tbody>
  </Table>
</TableWrapper>
```

table 要素を div 要素（`TableWrapper`）でラッパーとして囲っています。理由としては table 要素は `border-raidus` が指定出来ないのでラッパーを使ってテーブルの角丸を作ることにしました。

![table要素角丸](https://github.com/wimpykid719/react-component/assets/23703281/b5c99da1-9264-4f25-8a79-d6e7ec1d9755)

そして `onScroll` をラッパーに設定してテーブルがスクロールされるたびに設定したメソッドが発火するようになっています。

```tsx
const SCROLL_HEIGHT = 576;
...

const TableWrapper = styled.div`
  overflow-x: auto;
  max-width: 432px;
  width: 100%;
  border-radius: 4px;
  border: solid 1px #ccc;
  max-height: ${SCROLL_HEIGHT}px;
  overflow-y: scroll;
  margin-top: 20px;
  @media screen and (max-width: 1279px) {
    max-width: 600px;
  }
  @media screen and (max-width: 720px) {
    min-width: 290px;
  }
`;
```

TableWrapper は最大の高さが `576px` になっていて中の table 要素がそれ以上になる時にスクロールできるようになっています。
※要素がテーブルヘッダ + 10 件の時に `576px` 超えるようになっている。

![スクロール検知に使用される買う要素の高さ](https://github.com/wimpykid719/react-component/assets/23703281/9be1707f-b139-4517-93d2-15c777a93d98)

```tsx
const onScroll = async (e: React.UIEvent<HTMLDivElement>) => {
  if (!tableRef.current) return;
  const { scrollHeight, scrollTop, clientHeight } = e.currentTarget;
  if (
    clientHeight + scrollTop !== scrollHeight ||
    isLoading ||
    !url ||
    tableRef.current?.clientHeight < SCROLL_HEIGHT
  )
    return;
  await execFetchPokemon();
};
```

スクロールが検知されると上記のメソッドが実行されます。`TableWrapper` のそれぞれの要素高さを取得して判定に利用しています。

- scrollHeight：これは紫の範囲になる。隠れてスクロールできる要素全てを足した高さを変数に持っている。
- clientHeight：これは青の範囲になる。見えている要素の高さを変数に持っている。これは 10 件のデータが入っている時は `max-height: 576px;` を指定しているので　 `576` になる。
- scrollTop：これは黄色の範囲になる。スクロールして上に隠れた部分の高さを意味している。なのでスクロールすると値が増えていく。

なので `clientHeight + scrollTop` を足して `scrollHeight` と同じになる時に最下部まで要素がスクロールされたと判定することができます。この時新たにデータを取得して追加することで無限スクロールを実現しました。

**判定の内容**

- スクロールが下部になっていない
- スクロール中
- url が falsy な値
- table 要素の高さが `576px` 未満

上記の際は無限スクロールが実行されないようにしました。

```tsx
const execFetchPokemon = async () => {
  setIsLoading(true);
  await fetchPokemon(url);
  setIsLoading(false);
};
...

const fetchPokemon = async (url: string | undefined) => {
  try {
    if (!url) return;
    const response = await fetch(url);
    const data: PokemonFetchedData = await response.json();
    const nextUrl = data.next || undefined;
    setUrl(nextUrl);
    setPokemonObj((prePokemonObj) => {
      const newPokemonObj = pokemonSelectCheckedObj(
        data.results,
        checkedPokemons
      );
      return { ...prePokemonObj, ...newPokemonObj };
    });
    setPokemonNames((prePokemonNames) => {
      return [
        ...prePokemonNames,
        ...data.results.map((result) => result.name),
      ];
    });
  } catch {
    setIsError(true);
  }
};

```

フェッチ処理が走るとステートに保存された url を元に api にリクエストを投げる。受け取った値に次の url が含まれていたらステートにセットしてなければ `undefined` をセットする。

`pokemonSelectCheckedObj` で上で紹介したデータ構造にして、ポケモン名だけ配列も同時にステートにセットする。

ここまでが table 要素を使った無限スクロールになります。ここから先は運命共同体のチェックボックスの解説になります。
なので React での無限スクロールを知りたかった人はここまでで作る事ができると思います。

### テーブルレイアウト - チェックボックス

チェックボックス

```
<SerachedTr
  className="TrTd"
  key={name}
  onClick={
    pokemonObj[name]?.checked
      ? () => removePokemon(pokemonObj[name]?.name)
      : () => addPokemon(pokemonObj[name])
  }
>
  <Td>
    <Checkbox
      checked={!!pokemonObj[name]?.checked}
      label={pokemonObj[name]?.name || ""}
    ></Checkbox>
  </Td>
  <Td>{pokemonObj[name]?.url}</Td>
</SerachedTr>
```

クリック出来る範囲を広くしたいので、実際にはクリック出来る要素はテーブルローになる。
クリックするとチェックの状態によって 2 種類のメソッドが用意されている。

```tsx
// クリックされた要素のpokemonObjのcheckをチェックONの状態にする
// ２つ目のテーブルレイアウトに要素を追加する
const addPokemon = (pokemon: Pokemon | undefined) => {
  if (!pokemon) return;
  const pokemonName = pokemon.name;
  setPokemonObj((prePokemonObj) => {
    return {
      ...prePokemonObj,
      [pokemonName]: {
        name: prePokemonObj[pokemonName]?.name || "",
        url: prePokemonObj[pokemonName]?.url || "",
        checked: true,
      },
    };
  });
  setCheckedPokemons((preCheckedPokemons) => {
    const checkedPokemon = { name: pokemon.name, url: pokemon.url };
    if (!preCheckedPokemons) {
      return [checkedPokemon];
    } else {
      return [...preCheckedPokemons, checkedPokemon];
    }
  });
};

// クリックされた要素のpokemonObjのcheckをチェックOFFの状態にする
// ２つ目のテーブルレイアウトから要素を削除する
const removePokemon = (pokemonName: Pokemon["name"] | undefined) => {
  if (!pokemonName) return;
  setPokemonObj((prePokemonObj) => {
    return {
      ...prePokemonObj,
      [pokemonName]: {
        name: prePokemonObj[pokemonName]?.name || "",
        url: prePokemonObj[pokemonName]?.url || "",
        checked: false,
      },
    };
  });

  setCheckedPokemons((preCheckedPokemons) => {
    return preCheckedPokemons.filter(
      (checkedPokemon) => checkedPokemon.name !== pokemonName
    );
  });
};
```

2 つ目のテーブルレイアウト

```tsx
<TableWrapper>
  <Table>
    <Thead>
      <Tr className="TrTh">
        <Th>ポケモン名</Th>
        <Th>詳細URL</Th>
      </Tr>
    </Thead>
    <Tbody>
      {!checkedPokemons || checkedPokemons.length === 0 ? (
        <Tr>
          <Td colSpan={2}>
            <NoPokemon>選択中のポケモンが表示されます</NoPokemon>
          </Td>
        </Tr>
      ) : (
        checkedPokemons.map((checkedPokemon) => (
          <Tr className="TrTd" key={checkedPokemon.name}>
            <PokemonNameTd>
              <RemoveButton
                onClick={() => removePokemon(checkedPokemon.name)}
              />
              {checkedPokemon.name}
            </PokemonNameTd>
            <Td>{checkedPokemon.url}</Td>
          </Tr>
        ))
      )}
    </Tbody>
  </Table>
</TableWrapper>
```

ここに 1 つ目のテーブルローでクリックされた要素が表示されるようになっている。`RemoveButton` をクリックすると `removePokemon` が実行されて 1 つ目のテーブルレイアウトでのチェックボックスは OFF 状態になり、2 つ目のテーブルレイアウトからも削除されるようになっている。

これで今回の仕様に沿った実装が出来ました！

## 最後に

無限スクロールをどうやって実装するんだろうと最初わからなかったのでなんとか実装出来てよかったです。
テーブルレイアウトに関しては苦手意識があって、最初既存のテーブルレイアウト（div で作られた）で実装しました。
既存のものにはレイアウト崩れがあって table 要素に書き直す事になって納期もあって大変でした。（ほとんどデザイナーの人に書いてもらった）その節はとても助かりました。ありがとうございます。
その時は言われたままに書いていたので、自分の理解も含めて解説記事を書いてみました。
今後も工夫したコンポーネント等記事に出来たらと思います。
ここまで読んで頂きありがとうございました。

### 参考文献

[Documentation - PokéAPI](https://pokeapi.co/docs/v2)

[Border style do not work with sticky position element](https://stackoverflow.com/questions/50361698/border-style-do-not-work-with-sticky-position-element)

🕊：[Twitter](https://twitter.com/Unemployed_jp)
👨🏻‍💻：[Github](https://github.com/wimpykid719)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/22565/wataru)