---
title: "Next.jsのAPI機能を使ってQiitaの投稿記事をZennみたいにリポジトリから自動投稿・更新、管理出来るようにした。" # 記事のタイトル
emoji: "🦑" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["nextjs", "react", "typescript", "api", "vercel"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.06.21'
---

## 最初に

前回Next.jsとVercelを使用して、Zennに投稿した記事を取得して作る[techBlog](https://zenn.dev/unemployed/articles/nextjs-build-techblog)を作成した。今回はQiitaに投稿した記事も取得してブログの記事に加えるようにする機能追加とどうせならという事でQiitaの記事をGithubリポジトリで管理してリポジトリに更新があったら自動でQiitaに投稿されるようにZennみたいな機能追加を行いたいと思います。

## 追加機能

techBlogに新たに追加する機能をまとめてみた。

- Zennみたいに連携したリポジトリに記事を追加したらQiitaに投稿・更新を行う。
- Qiitaに投稿してるリポジトリデータを取得してブログの記事にする。

## 必要な物

- qiitaのアクセストークン
- リポジトリにwebhookの設定をする
- githubのアクセストークン（techBlog作成の際に取得した。）

## qiitaのアクセストークンを取得

[個人アクセストークンの発行](https://qiita.com/settings/tokens/new) Qiitaのアカウント作成後ここから作成することが出来ます。

設定 > アプリケーション > 新しくトークンを発行するでも上記のリンクページにいけます。

権限は `read_qiita` と `write_qiita` にチェックを入れる。

あとは発行するを押すと出てくる。表示されたアクセストークンは一度しか表示されないので、コピーしておく。

## Qiitaの記事を管理するリポジトリのwebhook設定

webhookとは何かイベントが起きた際に外部にHTTPリクエストを投げてくれる機能でこれのおかげで別のサービス同士を連携させたりすることが可能になる。

リポジトリにwebhook設定するにはGithubでwebhookを設定したいリポジトリページに行って > settings > webhooksで設定できる。

### イベントとは

このイベントはGithubの場合はpushによりリポジトリの更新等が該当する。

他にも `Send me everything.` や `Let me select individual events.` 等細かく設定できるようだが、webhookを投げて欲しいタイミングはpush時のみなので `Just the push event.` にチェックを入れる。

![webhook設定](https://user-images.githubusercontent.com/23703281/122698234-d91b6f00-d281-11eb-8cf3-222e237fbe91.png)

### Payload URL

これはリクエストを投げて欲しい外部URLである。

自分の場合は `https://techblog-pink.vercel.app/api/qiita-post` ここにAPIを立てるので、このアドレスを指定する。

### Content type

これはおそらくリクエストを投げる際に一緒に送信するデータのタイプになると思うのですが、Json形式で送って欲しいので `application/json` を選択する。

`application/x-www-form-urlencoded` はURLにデータを乗せて送信する。

POSTメソッドでもURLにデータを乗せれるんですね。 そういう事するのGET メソッドのイメージがある。

これでwebhookの設定は完了したのであとはコードを書いていく。

と言ってもすでに完成しているので、コードの説明をしていこうと思う。

## Next.jsにAPIを立てる。

`pages/api/qiita-post.ts` というようにファイルを作成する事でAPIのエンドポイントをNext.jsでは作成できる。

あとはここにレスポンスを受け取った際の処理を記述していく事になる。

記述形式は node.jsのフレームワークexpressと同じだと思う。個人的には使用した事ないので詳しくは分からないが、チュートリアルでAPIを作成した際に

> [Connect/Express middleware support](https://nextjs.org/docs/api-routes/api-middlewares#connectexpress-middleware-support)

と書かれていたので ConnectとExpressを使っているんだと思う。

使い方はとても簡単で `req` にwebhook等から受け取ったデータが格納され `req.body` で取り出せる。

 `res` はwebhookを出したサーバに返す値を設定できる。下記の例では ステータスコード200で `{ text: 'Helllo' }` オブジェクトを返す。

```tsx
import { NextApiRequest, NextApiResponse } from 'next'

export default (req: NextApiRequest, res: NextApiResponse) => {
  // ここに処理を追加していく。
  res.status(200).json({ text: 'Hello' })
}
```

実際に実際に処理を追加し作成したコード

`qiita-post.ts`

```tsx
import type { NextApiRequest, NextApiResponse } from 'next'
import { getUpdatedFiles } from '../../lib/api/qiita'
import { makeQiitaArticle } from '../../lib/api/qiita'
import { postQiita } from '../../lib/api/qiita'
import { writeQiitaId } from '../../lib/api/qiita'
import { PushRes } from '../../types/Response'

export default async(req: NextApiRequest, res: NextApiResponse) => {
  
  if (req.method === 'POST') {
    const files = await getUpdatedFiles(req.body)
    console.log(`filesフィルタ前の中身：${files[0]}`)
    console.log(`files長さ：${files.length}`)

    // udefinedが配列に含まれるので取り除く
    const filesRemovedUndefined = files.filter(Boolean)
    console.log(`filesフィルタ後の中身：${filesRemovedUndefined[0]}`)
    console.log(`filesフィルタ後の長さ：${filesRemovedUndefined.length}`)
    // filesRemovedUndefinedに値があれば処理を続ける。
    if (filesRemovedUndefined.length) {
      const statuses = await Promise.all(filesRemovedUndefined.map( async(file) => {
        //　ここじゃない
        const article = makeQiitaArticle(file)
        const qiitaPostRes = await postQiita(article, file.qiitaId)
        // falseの場合はwebhookの2回目の通信になるのでここで処理を止める。
        if (!qiitaPostRes) {
          // 多分これが実行された時点で処理止まる気がする。レスポンス返してるから
          return { status: 200, message: 'notting to upadate posts' }
        } else {
          // 上記の分岐で引っ掛からなければwriteQiitaIdを実行できる。
          const status: PushRes | string | undefined = await writeQiitaId(file, qiitaPostRes.id)
          // 書き換えが成功すればそれを伝える
          if ('object' === typeof status) {
            if(status.commit.message) {}
            console.log(status.commit.message);
            // res.status(201).json({ status: `succeeded ${status.commit.message}` })
            return { status: 201, message: `succeeded ${status.commit.message}` }
          } else if('string' === typeof status) {
            // もしrepositoryの書き換えが必要ない記事の更新の場合はリポジトリの更新が必要ないので
            // 処理を止めた事を伝える。
            return { status: 200, message: status }
          } else {
            // リポジトリの書き換えで何かエラーが発生した事を伝える。
            return { status: 502, message: 'failed to update repository' }
          }
        }
      }))
      res.status(200).json( { allstatus: statuses} )
    } else {
      res.status(200).json( { status: 'noting to post' } )
    }
    
    //通信が成功したらstatusコード201とJsonを返す。
    // res.status(201).json({ body: req.body })
  } else {
    res.status(200).json({ name: 'John Doe' })
  }
}
```

## 処理の順番

1. ユーザのpushによるリポジトリの更新。
2. webhookからHTTPリクエストを受け取る。
3. getUpdatedFiles() 関数が新規投稿・更新予定に必要なデータを `[object, object,...]` の形で返す。
4. 返された配列に `undefined` も含まれることがあるのでそれを取り除いて、map関数で一つずつにして makeQiitaAriticle() 関数に渡し、返り値としてqiitaに投稿する際のデータ形式にして返してくれる。
5. postQiita() 関数に先ほど作成したqiitaの投稿データ形式と更新の場合に必要になる、記事のIDを渡す。更新・投稿した場合はqiitaからのレスポンスを返す。記事自体に変更がなく更新する必要がない場合は `false` を返す。これは後述で出てくるwriteQiitaId() 関数からリポジトリにpushする際にwebhookが再び走る。再びwriteQiitaId()が実行されpushを投げるループを止めるためにfalseを返している。なので最低でも2回、webhook走る。
6. writeQiitaId() は新規投稿の際に生成されたqiitaIdをリポジトリにある記事の `.md` ファイルに書き込む。
7. そして最後に処理が上手く行ったどうかのレスポンスを返す。

## getUpdatedFiles()

qiita.tsに書かれた関数を一つずつ紹介指定こうと思う。あまり綺麗な設計とは言えない、ブサイクなコードになっているが今後リファクタリングしながら綺麗にしていきたいと思う。

```tsx
//1 githubから投稿・更新された記事を取得 webhookのデータが大事
export async function getUpdatedFiles(payload: Webhook) {
  const BASE_URL = 'https://api.github.com/repos/wimpykid719/qiita-content/commits/'
  const latestCommitsha: string = payload.head_commit.id
  
  const updatedFileContents: Commits | undefined = await fetch(BASE_URL + latestCommitsha, {
    headers: {"Authorization": `token ${process.env.GITHUB_TOKEN}`}
  })
  .then(res => {
    if (res.ok) {
      return res.json();
    }
    return
  })
  .catch(err => {
    console.log(err);
  });
  const files = await Promise.all(updatedFileContents.files.map( async(updatedFile) => {
    // statusが削除のファイルは無視する。
    if(updatedFile.status === 'removed') {
      console.log('removedのステータスなのでファイルを弾いた')
      return
    }
    // statusがremoved以外でも、拡張子がmdファイル以外の場合は取得しない
    if (!/[\s\S]*?\.md/.test(updatedFile.filename)) {
      return
    }
    const fileJson: Content | undefined = await fetch(updatedFile.contents_url, {
      headers: {"Authorization": `token ${process.env.GITHUB_TOKEN}`}
    })
    .then(res => {
      if (res.ok) {
        return res.json();
      }
      return
    })
    .catch(err => {
      console.log(err);
    });

    console.log(`fileJsonの中身：${fileJson.name}`)
    
    const buffer = Buffer.from(fileJson.content, 'base64');
    const markdownContents = buffer.toString("utf-8");
    const matterResult = matter(markdownContents)
    if (!matterResult.data.published) {
      return
    }
    return {
      id: fileJson.name.replace(/\.md$/, ''),
      ...(matterResult.data as { title: string; emoji: string; type: string; topics: string[]; published: boolean; date: string; qiitaId: string; }),
      content: matterResult.content,
      path: fileJson.path,
      sha: fileJson.sha,
      markdownContents: markdownContents,
    }
  }))
  return files
}
```

webhookから取得した最新のコミットshaを使う事で、最新でコミットされたファイル情報を取得する事ができる。そこにあるコミット情報が `status` にあり削除、修正、新規追加なのかわかる。

`contents_url` からコミットしたファイルの詳細情報を取得する事ができる。

`filename` は対象となってるファイル名、これらが `files` に `[object, object...]` のような感じで格納されている。

`res.ok` で判定しているのは fetchが404でもエラーを投げないためresponseが成功した際に `true` を返す `.ok` を使用している。 `.status` でステータスコードを取得して判定する事も出来る。

`contents_url` にfetchを投げると返って来たJsonにbase64でマークダウンファイルの中身があるのでそれを取り扱えるように文字列に変換する。そしてマークダウンに含まれるメタデータ等と整形してJsonオブジェクトを作成して返す。

## makeQiitaArticle()

これはqiita APIで投稿する際のデータ形式にオブジェクトを作成して返す。

ドキュメントを参考に作成した。

[Qiita API v2 documentation - Qiita:Developer](https://qiita.com/api/v2/docs#post-apiv2items)

```tsx
//2 投稿・更新された記事をqiitaのフォーマットにする。
export function makeQiitaArticle(file: QiitaRepository) {
  const tags = file.topics.map((topic: string) => {
    return {'name': topic}
  })
  const article = {
    'body': file.content,
    'private': false,
    'tags': tags,
    'title': file.title,
    'tweet': true
  }
  return article
}
```

## postQiita()

先ほど作成したオブジェクトをfetchで送る。

2回目のwebhookによる処理の場合、ここで処理を止める。

```tsx
//3 qiitaに投稿する。
export async function postQiita(qiitaArticle: QiitaArticle, idArticle: string) {
  const url = idArticle ? 
    'https://qiita.com/api/v2/items' + '/' +idArticle :
    'https://qiita.com/api/v2/items';
  console.log(`urlの確認：${url}`)

  const patchPostOk = await ( async(url, qiitaArticle, idArticle) => {
    // idがあるやつはすでに投稿されている記事なので、記事の更新かそれとも2回目のフックか判定する。
    if(idArticle) {
      // 記事が存在するのか取得する。記事があるならJsonが返る。
      const qiitaArticleGetRes: QiitaArticleGetRes | undefined = await fetch(url, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${process.env.QIITA_TOKEN}`,
        },
        method: 'GET',
      })
      .then(res => {
        if (res.ok) {
          return res.json();
        } else {
          return
        }
      })
      .catch(err => {
        console.log(err);
      })
      // もしなければidはあるが記事はないことになる。つまりidが間違っている。
      if (!qiitaArticleGetRes) {
        return false
      }
      // idがあり、アップ予定の記事タイトルと元々の記事タイトルが違う。これは更新になる。
      if(!(qiitaArticle.title === qiitaArticleGetRes.title)) {
        console.log('タイトル更新')
        return true
      }
      // idがあり、アップ予定の記事タグと元々の記事タグが違う。これは更新になる。
      const flags2 = qiitaArticle.tags.map((tag) => {
        const flags = qiitaArticleGetRes.tags.map((resTag)=> {
          if(tag.name.toLowerCase() === resTag.name.toLowerCase()) {
            return false
          } else {
            return true
          }
        })
        return flags.every(v => v)
      })
      const flag = flags2.some(v => v)
      if(flag) {
        console.log('タグ更新')
        return true
      }
      // idがあり、アップ予定の記事と元々の記事が違う。これは更新になる。
      if(!(qiitaArticle.body === qiitaArticleGetRes.body)) {
        console.log('記事更新')
        return true
      }
      // idがあって変更が確認されない場合は2回目のwebhookによるものだから処理を止める必要がある。
      return false
    }
    // idがないやつは新規投稿する。
    console.log('記事投稿')
    return true
  })(url, qiitaArticle, idArticle)
  
  console.log(`投稿できるか確認：${patchPostOk}`)
  if (!patchPostOk) {
    return false
  }
  const method = idArticle ? 'PATCH': 'POST';
  console.log((`methodの確認：${method}`))
  console.log(`記事のタイトル：${qiitaArticle.title}`)

  
  const jsonQiitaArticle: string = JSON.stringify(qiitaArticle)
  const qiitaPostRes: QiitaPostRes | undefined = await fetch(url, {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env.QIITA_TOKEN}`,
    },
    method: method,
    body: jsonQiitaArticle,
  })
  .then(res => {
    if(res.ok) {
      return res.json();
    }
    return
  })
  .catch(err => {
    console.log(err);
  });
  return qiitaPostRes
}
```

`patchPostOk` にPath・Postを行っても良い場合に `True` を返し、2回目のwebhookのため処理を止める場合に `False` が入る。

それを調べる方法としてqiitaIdを用いて行う。

qiitaIdがない物は初めての投稿となるため投稿する必要があるデータになる。

qiitaIdがある物は、記事の更新の可能性があるのでまず、そのqiitaIdを持つ記事にqiita APIを使って記事情報を取得するそしてそれらを比較して差異があれば、記事の更新だと判断する。差異が見られなければ2回目のwebhookによる処理と判断し以降にfetchを投げる処理を行わないようにする。

## writeQiitaId()

ここでは新規投稿時に受け取ったqiitaIdをGitHubリポジトリにあるマークダウンに書き込む。

書き込みは文字列になっているので正規表現で置き換えたコンテンツをGitHub APIでコミット・プッシュする。その際に更新したいファイルの最新のshaが必要となる。

```tsx
//4 githubのリポジトリにqiitaIdを追加する。
export async function writeQiitaId(file: QiitaRepository, qiitaId: string) {
  console.log(`qiitaからのID： ${qiitaId}`)
  console.log(`fileからのID： ${file.qiitaId}`)
  const BASE_URL = 'https://api.github.com/repos/wimpykid719/qiita-content/contents/'
  const contentBeforeAddId = file.markdownContents
  // fileのidは空か同じものが入っているので、一致しなければ新規投稿を意味する。
  if(!(file.qiitaId === qiitaId)) {
    console.log(`sha：${file.sha}`)
    //markdownの文字列に正規表現でqiitaIdを追加する。
    const contentAddId = contentBeforeAddId.replace(/(?<=---[\s\S]*?\nqiitaId:\s*').*?(?='[\s\S]*?---)/, `${qiitaId}`)
    const buffer = Buffer.from(contentAddId, 'utf-8');
    const content = buffer.toString("base64");

    const resRepo: PushRes | undefined = await fetch(BASE_URL + file.path, {
      headers: {
        'Accept': 'application/vnd.github.v3+json',
        'Authorization': `token ${process.env.GITHUB_TOKEN}`,
      },
      method: 'PUT',
      body: `{\
        "message":"write ${qiitaId}",\
        "content":"${content}",\
        "sha":"${file.sha}"\
      }`,
    })
    .then(res => {
      if (res.ok) {
        return res.json();
      }
      return
    })
    .catch(err => {
      console.log(err);
    });
    return resRepo
  }
  return 'stop to rewrite repository'
}
```

上述のコードだと、複数のファイルの場合、1ファイル1コミット見たいな感じになるので変更をまとめてコミットする事ができない。それを行うにはGit Date APIを用いてBlod、Treeのgitの根本的な仕組みを理解する必要があり少々難解だったため、次回Gitの仕組みに関する記事を書いてプログラムからGitHubのリポジトリを簡単に操作出来るライブラリ作成に挑戦しようと思う。

## GitHubリポジトリにあるQiita記事データを取得してブログに追加する。

今度はブログにQiita投稿している記事を追加する機能を `getPostsData()` 関数に追加する。

前回のZennの投稿記事を管理しているリポジトリデータを取得して記事の形式にするのと同じ事をQiitaリポジトリ にもするそして最後2つの配列を合体して一つの配列 `allDatas` にして返す。

lib/posts.tsx

```tsx
export async function getPostsData() {
  const zennArticles: ArticleResponse[] = await fetchGithubRepo('https://api.github.com/repos/wimpykid719/zenn-content/contents/articles')

  const datas = await (async (zennArticles) => {
    if (zennArticles) {
      return await Promise.all(zennArticles.map( async (article: ArticleResponse) => {
        return fetchGithubMakeArticle('https://api.github.com/repos/wimpykid719/zenn-content/contents/articles/', article.name)
      }));
    }
  })(zennArticles);

  const qiitaArticles: ArticleResponse[] = await fetchGithubRepo('https://api.github.com/repos/wimpykid719/qiita-content/contents/articles')

  const datas2 = await (async (qiitaArticles) => {
    if (qiitaArticles) {
      return await Promise.all(qiitaArticles.map(async (article: ArticleResponse) => {
        return fetchGithubMakeArticle('https://api.github.com/repos/wimpykid719/qiita-content/contents/articles/', article.name)
      }));
    }
  })(qiitaArticles)

  const allDatas = datas.concat(datas2)

  const removeFalsyDatas = allDatas.filter(Boolean)
  return removeFalsyDatas;
}
```

そこでGitHubのレポジトリにアクセスする部分とmdファイルを変換する部分はほぼ同じ処理になるので共通化する関数としてutilityフォルダに `fetchGithubRepo()` と `fetchGithubMakeArticle()` に分けた。

## fetchGithubRepo()

今までfetch処理がthenとawaitを混合して書いていたので、それをawaitのみに統一したこちらの方が可読性が高く主流な書き方となりつつあるので、他の関数に記述されたfetch処理もこのように書き換えた方が良いかもしれない。fetchはステータスコードエラーではエラー処理を排出しないのでif文を使って排出するようにしている。

```tsx
export async function fetchGithubRepo(url: string) {
  try {
    const res = await fetch(url, {
      headers: {"Authorization": `token ${process.env.GITHUB_TOKEN}`}
    })
    if (!res.ok) {
      throw `ステータスコードエラー：${res.status}`
    } else {
      return res.json()
    }
  } catch(err) {
    console.log(`repofetchデータの処理中にエラー：${err}`);
  }
}
```

## fetchGithubMakeArticle()

これはリポジトリから個別にmdファイルを読み込んでブログに投稿出来る形式のオブジェクト作る。

qiitaIdを持つ場合は `from: 'Qiita'` とどこからの記事なのか分かるようにしている。Zennからの場合は `from: 'Zenn'` となる。

![fromQiita](https://user-images.githubusercontent.com/23703281/122698265-ecc6d580-d281-11eb-84fa-696073a934b0.jpeg)


このような感じに左下部で表示する際に使用するになっている。

```tsx
export async function fetchGithubMakeArticle(url: string, fileName: string) {
  try {
    const res = await fetch(url + fileName, {
      headers: {"Authorization": `token ${process.env.GITHUB_TOKEN}`}
    })
    if (!res.ok) {
      throw `ステータスコードエラー：${res.status}`
    } else {
      const data = await res.json()
      const buffer = Buffer.from(data.content, 'base64');
      const fileContents = buffer.toString("utf-8");
      const matterResult = matter(fileContents)
      if (!matterResult.data.published) {
        return
      }
      if (matterResult.data.qiitaId) {
        return {
          id: fileName.replace(/\.md$/, ''),
          ...(matterResult.data as { title: string; emoji: string; type: string; topics: string[]; published: boolean; date: string; }),
          content: matterResult.content,
          from : 'Qiita'
        }
      }
      return {
        id: fileName.replace(/\.md$/, ''),
        ...(matterResult.data as { title: string; emoji: string; type: string; topics: string[]; published: boolean; date: string; }),
        content: matterResult.content,
        from: 'Zenn'
      }
    }
  } catch(err) {
    console.log(`contentfetchデータの処理中にエラー：${err}`);
  }
  
}
```

## 最後に

なんとかブログに機能を無事に追加する事ができた。前回の記事で掲げたやる事リストには全く載ってなかったが、Vercelのサーバレスファンクションを使う事が出来てよかった。これが無料で出来るのは本当にありがたい。

作り始めた時は小さいスクリプトを組んで各機能に必要な動作が出来るか一個ずつ確認して出来るなと思ったら関数を書き始めて、それを組み合わせて目的の機能にしようとした時にエラーが連発して、そこでとても時間を浪費した。

### 反省

テストを手動で行っていたが、とても時間を食った。Vercelはブランチを切ってpushするとテスト環境のみに変更を加えてをプロダクトは変更を加えてない状態にしてくれる（これはとてもありがたい）。ただ毎回ビルドするのに1分くらいかかるため時間が取られた。コードを少し直してビルドを100回くらいは繰り返したと思う。おかげでコミット数だけはとても稼げて、すごい開発してるやつ風の草を生やせた。そこからGitHubに実際に記事を追加してwebhookにリクエストを出してもらったり、記事を消したり実際の操作をとにかく繰り返して時間がかかった。あとVercelのコンソールログは変数の中身が長いと表示出来ないみたいで動作時点での値を全て見る事が出来なかった。そのため引数を間違えたりしてとても時間と精神を持ってかれた。webhookを用いたプログラムでも、ローカルで開発出来る環境を作るべきだった。今思いつくのはwebhookで貰うJsonオブジェクトをコピーしておきそれをローカル環境で使う事くらい。

テストコードを書いた方が良かったのではと思う。HTTPリクエストを使用するテストコードの書き方とかイメージ出来なかったので、手を出す事が出来なかった。それもwebhookで貰う値だけコピーして使えばローカルでのテストコードが書けそうな気はする。

業務で作業している方々はどのようにテストをしているのか気になる。もし自分だったらこんな感じでテストコード書くよ等あったら教えて頂けると嬉しいです。

最後まで記事を読んで頂きありがとうございました。

### 参照

[Qiita API v2を利用してcurlで投稿してみた - Qiita](https://qiita.com/kai_kou/items/663d3f7bbc4da4ccf62d)

[Qiita API で投稿を自動化する - Qiita](https://qiita.com/tomowarkar/items/11488231c6d22d960323)

[Qiita に投稿する技術記事を GitHub で管理する方法 - Qiita](https://qiita.com/noraworld/items/79100783ba95d8c48924#github-webhook-%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)

[webhook イベントとペイロード](https://docs.github.com/ja/developers/webhooks-and-events/webhooks/webhook-events-and-payloads)

[JSON.stringify() - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

[Fetch の使用 - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/Fetch_API/Using_Fetch)

[リポジトリ](https://docs.github.com/ja/rest/reference/repos#get-repository-content)

[GitHubのGit Data APIでコミットを作成する - GeekFactory](https://int128.hatenablog.com/entry/2017/09/05/161641)

[Gitのコミットハッシュ値は何を元にどうやって生成されているのか](https://engineering.mercari.com/blog/entry/2016-02-08-173000/)

[js 偽とみなされる値を配列から取り除く - Qiita](https://qiita.com/may88seiji/items/e89ecf9232dfbe79d84f)

Gitの仕組みに関してはこの動画がとても分かりやすかったので、次回からこれを元にGitの勉強をしようと思う。

[Git Internals - Git Objects](https://www.youtube.com/watch?v=MyvyqdQ3OjI&list=LL&index=6)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。