---
title: "【無料で運用出来る】techBlogをNext.js, Tailwindcss, TypeScript, Vercelで作成した。" # 記事のタイトル
emoji: "📜" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["nextjs", "react", "typescript", "tailwindcss", "vercel"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.05.10'
---

## 最初に

Reactのチュートリアル、Next.jsのチュートリアル、TypeScriptの入門?を終えたので、そのアウトプットとして無料で運用出来るtechBlogを作成しようと思う。記事は新しくブログ用に書くとZennと分散してしまうのでZennに投稿している記事を使ってサイトをビルドしていく。その前、準備としてZennとGitHubを連携する必要があるのでまだ行っていない方は[こちらの記事](https://techblog-pink.vercel.app/posts/zenn-migrate-past-articles-github)を参考にすると出来ます。

### 実際に作成したtechBlog
モバイル版
![https://storage.googleapis.com/zenn-user-upload/0wwzsv9bsoyo8wicyl5j6fwe98yo](https://storage.googleapis.com/zenn-user-upload/0wwzsv9bsoyo8wicyl5j6fwe98yo)

デスクトップ版
![https://storage.googleapis.com/zenn-user-upload/j55aisuqu75pesx7rx66j6bhfnww](https://storage.googleapis.com/zenn-user-upload/j55aisuqu75pesx7rx66j6bhfnww)

[techBlog：大学生だった](https://techblog-pink.vercel.app)
[実際のコード](https://github.com/wimpykid719/blog)

### 参考にしたデザイン

![https://storage.googleapis.com/zenn-user-upload/iccv9zhlprbn9hxwb97f1ih50c6j](https://storage.googleapis.com/zenn-user-upload/iccv9zhlprbn9hxwb97f1ih50c6j)

[MNW- mobile](https://dribbble.com/shots/5489447-MNW-mobile)

## 構成

- Next.js
- index.tsx
- 記事をzennの投稿を管理しているリポジトリから取得する。
- TypeScriptでAPIで受け取るデータの型を指定する。
- [id].tsx
- コードのシンタックスハイライト、数式、テーブルを表示する。
- component各種
- Vercel環境変数
- Tailwindを導入する。
- google analyticsを導入する。

## Next.js

node.jsをインストールしたら開発を始めたいフォルダ内で `npx create-next-app` を実行すると必要なファイルやフォルダを生成してくれる。実行後はチュートリアルで作成したブログが出来上がっていると思う。基本的にそのブログを少カスタマイズしていく。

生成されたファイルは `index.js` 等なので `touch tsconfig.json` でTypeScriptの設定ファイルを生成するとNext.jsが自動で中身を書いてくれる。その後 `index.tsx` と変更してTypeScriptを扱えるようになる。 `npm run dev` でエラーなくビルド出来れるはず。

## 特徴

Next.jsはあらかじめWebサイト表示に必要なデータを集めて、レンダリングして静的ファイルにしたのをサーバに配置して返すようにしている。そのため表示速度がとても早い。これがよく言われている。SSG（Static Site Generation）である。ローカルで開発してる時はSSGでコードを書いても、 `npm run dev` ではSSR（Server-side Rendering）で表示されるのでリクエスト度にページがレンダリングされるので遅かったが、Vercelにデプロイして作成したページにアクセスした際にその速さに驚いた。

## 事前にレンダリング（Pre-rendering）とデータをフェッチする。

今回のGitHubのリポジトリから直接データをフェッチする前にNext.jsではどのようにデータを取得してHTMLを組み立てるのか見ていこうと思う。

### Pre-renderingとは

Next.jsでは全てのページをPre-renderingしている。つまり、Next.jsは前もって全てのページのHTMLを生成している。SPAのようにブラウザなどがJavaScriptを用いて、HTMLを生成しているのではない。

### Pre-renderingの方法は2つある。

- Static Generation（SSG）：Build時にHTMLファイルを生成する。
- Server-side Rendering（SSR）：RequestごとにHTMLファイルを生成する。

※開発モード( `npm run dev` or `yarn dev` )ではSSGの方法をとっていたとしても全てRequestごとにPre-renderingを行う(SSR)。

可能な限りSSGを使用する。理由はBuild時にHTMLを一括でCDNサーバ（Content Deivery Networkでキャッシュを分散させて配置する事でアクセスしてきた者から、一番近いサーバからキャッシュを渡して通信速度の改善を測る。）に作成・配置出来るため、毎回RequestによってHTMLを作成するServer-side Rendering（SSR）よりも表示速度が早くなるため。

ユーザのRequestよりも前にHTMLを作成出来るのであれば、SSGを使用する。ブログやECサイトは前もって表示出来るデータが決まっているので、これに当てはまる。

逆にユーザ動作がトリガーで表示が変わるものなどはSSRを利用する。

### SSGで外部データからコンテンツをbuildする。

HTMLを作成する際に、外部データの取得（DBから値を取得するなど）がある場合とない場合が想定される。

- 外部データが取得する必要がない場合

Build時に静的なHTMLファイルが生成される。

- 外部データの取得が必要な場合

Build時にNext.jsがDBにアクセスし必要な外部データを取得する。

その取得データを元にHTMLファイルを生成する。

この外部データが必要な場合、Next.jsでは `getStaticProps` を使用する。

### `getStaticProps` で外部からデータを取得する。

このメソッドを使用する事で、build時にNext.jsが外部データにアクセスする。

※ `getStaticProps` メソッドはサーバサイドでしか動作しない。中身の動作は自分で書く。

```jsx
// getStaticPropsでreturnしたpropsを引数として受け取れる。
export default function Home({ posts }) { ... }

// getStaticPropsの中身は自分で実装する。
export async function getStaticProps() {
	// DBやAPIから値を取得する。
	const data = await fetch('https://.../posts')
	const posts = await data.json()

	// 取得した値をHomeコンポーネントにprops経由で渡す
	return {
		props: posts
	}
}
```

ここで `export default function Home({ posts }) { ... }` がよく分からなかったので調べることにした。 `index.js` で使用されて、Next.jsの内部で `index.js` をbuildする際に実行されるんだと思う。

### APIにFetchを投げる、Datebaseにクエリを投げる。

- APIからデータ取得するには `fetch()` メソッドを使用する。

```jsx
export async function getSortedPostsData() {
	// Instead of the file system,
	// fetch post data from an external API endpont
	const res = await fetch('..')
	return res.json()
}
```

- DBからのデータ取得には `query()` メソッドを使用する。

## 拡張子のtsとtsxの違いは

`.ts` はTypeScriptだけを扱うファイルに付与して、TypeScriptの中でJSX構文（JSでhtmlを書くようにDOMを記述出来る）を扱うファイルは `.tsx` になる。

## Zennに投稿した記事を取得する。

Zennの記事を前回Githubレポジトリで管理出来るようにしたので、それをそのままブログの記事にする。

記事データの取得は `lib/posts.ts` に書かれた `getPostsData()` 関数が行っている。そして取得したデータを下記のようにトップページに並べている。

左がモバイル版、右がデスクトップ版のトップページ `/` のデザイン

![https://storage.googleapis.com/zenn-user-upload/32f6u9e1mc59hcwcwv4kklt49359](https://storage.googleapis.com/zenn-user-upload/32f6u9e1mc59hcwcwv4kklt49359)

実際に作成したtechBlogの `index.tsx` はこのような感じになっている。

`className=""` に書かれているのがtawilwindのクラス名でこれを使用する事で、クラス名を考える必要もcssを別に書き込む必要がないのでとても快適な開発を行える。導入方法は後半で解説している。

```tsx
import Head from 'next/head'
import { getPostsData } from '../lib/posts'
import { getSortedPostsData } from '../lib/posts'
import { getUserData } from '../lib/user'
import { GetStaticProps } from 'next'
import Link from 'next/link'
import { Article } from '../types/Article'
import { UserResponse } from '../types/Response'
import Layout from '../components/layout'
import Date from '../components/date'
import Topics from '../components/topics'
import { siteTitle } from '../components/layout'

//モバイル版の記事背景の色を4つ用意してそれを順番にclassNameに指定している。
const pattern = ["bg-blue", "bg-blue-light", "bg-gray", "bg-earth-light"]; 
//記事の最初はindex：0なので0/4のあまりは0でpattern配列の0番目が選択される。
function getColorClassFromIndex(index: number): string {
  return pattern[index % pattern.length];
}

export default function Home({ 
  sortedPostData, userData
}: {
  sortedPostData: Article[]
  userData: UserResponse
}) {
  return (
		{/*LayoutコンポーネントにGithubのプロフィール写真のURLを渡して取得するようにしている。*/}
    <Layout avatarUrl={userData.avatar_url}>
      <div className="lg:max-w-5xl lg:mx-auto">
        <Head>
          <title>{siteTitle}</title>
          <link rel="icon" href="/favicon.ico" />
        </Head>
        <ul className="mx-auto lg:flex lg:flex-wrap lg:justify-between lg:max-w-2xl xl:max-w-4xl">
					{/*Githubのリポジトリから取得したデータを整形したものをmap関数で並べる*/}
          {sortedPostData.map(({ id, title, date, topics, type }, index) => (
            <li className={getColorClassFromIndex(index) + " h-60 flex justify-center items-center lg:max-w-xs xl:max-w-sm w-full lg:mb-14 lg:bg-transparent" } key={id}>
              <div className="w-11/12 h-5/6 flex flex-col">
                <div className="font-bold text-xl">
                  <Link href={`/posts/${id}`}>
                    {title}
                  </Link>
                </div>
                <div className="text-xs text-gray-darker">
                  <Topics topicList={topics} />
                </div>
                <small className="border border-r-0 border-b-0 border-l-0 h-8 flex justify-between mt-auto items-end">
                  <span>{type}</span>
                  <Date dateString={date} />
                </small>
              </div>
            </li>
          ))}
        </ul>
      </div>
    </Layout>
  )
}

// サーバ側で実行される処理データをfetchする。
export const getStaticProps: GetStaticProps = async () => {
	// リポジトリ内にあるファイル情報を全て取得している。
  const allPostsData = await getPostsData()
	// mdファイルのmeta情報を元に日付順に並び替えたり、データを整形して渡す。
  const sortedPostData = await getSortedPostsData(allPostsData)
	// Githubのプロフィール画像を取得したいのでユーザ情報を取得する。
  const userData = await getUserData()
  return {
    props: {
      sortedPostData: sortedPostData,
      userData: userData, 
    }
  }
}
```

## 追加したい機能はlibフォルダに書いていく。

ルートフォルダに `lib/posts.tsx` という感じにファイルを作成してそこにAPIをFetchする関数を書いて、各々のページで `import` して使う。先ほどの `getStaticProps` 内で使用されている関数もここに記述されている。

```tsx
import matter from 'gray-matter'
import remark from 'remark'
import html from 'remark-html'
import prism from 'remark-prism'
import gfm from 'remark-gfm'
//マークダウンから数式を解析
import math from 'remark-math'
//解析された数式をkatexが読み込めるようにHTML変換する。
import htmlKatex from 'remark-html-katex'
import { ArticleResponse } from '../types/Response'
import { Article } from '../types/Article'

export async function getPostsData() {
	//Githubのzenn-cotent/articlesフォルダ内のデータを全件取得している。
  const zennArticles: ArticleResponse[] = await fetch("https://api.github.com/repos/wimpykid719/zenn-content/contents/articles", {
    headers: {"Authorization": `token ${process.env.GITHUB_TOKEN}`}
  })
    .then(res => {
        return res.json();
    })
    .catch(err => {
        console.log(err);
    });
	// 上記で取得したデータにはメタ情報のみでファイルデータはないのでそれを取得している。
  const datas = await (async (zennArticles) => {
    if (zennArticles) {
      return await Promise.all(zennArticles.map(async (article: ArticleResponse) => {
        const data = await fetch("https://api.github.com/repos/wimpykid719/zenn-content/contents/articles/" + article.name, {
          headers: {"Authorization": `token ${process.env.GITHUB_TOKEN}`}
        })
          .then(res => {
              return res.json();
          })
          .catch(err => {
              console.log(err);
          });
				// 取得したデータがbase64形式になっているのでそれをutf-8の文字列に変換する。
        const buffer = Buffer.from(data.content, 'base64');
        const fileContents = buffer.toString("utf-8");
				// mdファイルの構文を解析してメタ情報とコンテンツをオブジェクトに格納してくれる。
        const matterResult = matter(fileContents)
        if (!matterResult.data.published) {
          return
        }
        return {
          id: article.name.replace(/\.md$/, ''),
          ...(matterResult.data as { title: string; emoji: string; type: string; topics: string[]; published: boolean; date: string; }),
          content: matterResult.content
        }
      }));
    }
  })(zennArticles);
  const removeFalsyDatas = datas.filter(Boolean)
  return removeFalsyDatas;
}

export async function getSortedPostsData(articles: Article[]){
  return articles.sort((a, b) => {
    if (a.date === b.date){
      return 0
    }
    if (a.date < b.date) {
        return 1
    } else {
        return -1
    }
  })
}

export async function getHtmlContent(article: Article) {
  const processedContent = await remark()
    .use(math)
    .use(htmlKatex)
    .use(html)
    .use(prism)
    .use(gfm)
    .process(article.content)
  const contentHtml = processedContent.toString()
  return {
    ...article,
    content: contentHtml
  }
}

export function getAllPostIds(articles: Article[]) {
  return articles.map((article: Article) => {
    return {
      params: {
          id: article.id
      }
    }
  })
}

export function getPostData(articles: Article[], id: string) {
  return articles.filter((article: Article) => {
    if(article.id === id) {
      return article
    }
  })
} 

// title: "tsconfig.json オプション入門" # 記事のタイトル
// emoji: "🧐" # アイキャッチとして使われる絵文字（1文字だけ）
// type: "tech" # tech: 技術記事 / idea: アイデア記事
// topics: ["typescript", "初心者", "作業ログ"] # タグ。["markdown", "rust", "aws"]のように指定する
// published: true # 公開設定（falseにすると下書き）
// http://robin.hatenadiary.jp/entry/2017/01/08/225337 bugger.from推奨
```

## コードシンタックスハイライト、数式、tableをHTMLに変換する。

上記のコード一部、 `getHtmlContent()`

mdファイルをHTMLに変換するのに `remark()` を使用しているのでその拡張を提供している。

Next.jsのチュートリアルでもインストールする。mdファイル構文解析してhtmlに変換してくれる。

```bash
npm install remark remark-html
```

## シンタックスハイライトの導入

![https://storage.googleapis.com/zenn-user-upload/a534c3wohx90u39i1ymmpoxcur2w](https://storage.googleapis.com/zenn-user-upload/a534c3wohx90u39i1ymmpoxcur2w)

mdに書かれたコード部分をHTMLでタグ付けしてくれる。それをcssで装飾する事でシンタックスハイライトを作る。cssは自作で作成する事で好きな色でコードを装飾出来る。

[prism generator](http://k88hudson.github.io/syntax-highlighting-theme-generator/www/)を使うと配色の感じを確認しながら作成出来る。httpsに対応していないのが気になる。

```bash
npm i @sergioramos/remark-prism
```

生成したcssを `_app.tsx` で設定する。

```tsx
import '../styles/globals.css'
// 下記が自作のシンタックスハイライトのcss
import "../styles/prism-daigakusei.css"
// 同梱のprismのテーマを使いたいなら
// import "prismjs/themes/prism-funky.css"

// katexのcss
import "katex/dist/katex.min.css"
import usePageView from '../src/hooks/usePageView'
import { AppProps } from 'next/app'

function MyApp({ Component, pageProps }: AppProps) {
  usePageView()
  return <Component {...pageProps} />
}

export default MyApp
```

## 数式（Katex）の導入

上記にある通り `_app.tsx` でcssを当てておく。

![https://storage.googleapis.com/zenn-user-upload/gpa32uhhv2n62yhfso8wru296fcw](https://storage.googleapis.com/zenn-user-upload/gpa32uhhv2n62yhfso8wru296fcw)

```bash
npm install remark-math remark-html-katex
```

## tableの導入

![https://storage.googleapis.com/zenn-user-upload/g66ct9rp2mctgjy2c4z8mn7asxnf](https://storage.googleapis.com/zenn-user-upload/g66ct9rp2mctgjy2c4z8mn7asxnf)

```bash
npm install remark-gfm
```

このuseに記述する順番が違うと上手く変換してくれないので（数式の扱うmath, htmlkatexで起こった）、注意が必要多分ほんとはhtmlが一番最後にした方が良い気がするが動作してるのでこのままにしておく。

```tsx
const processedContent = await remark()
    .use(math)
    .use(htmlKatex)
    .use(html)
    .use(prism)
    .use(gfm)
    .process(article.content)
const contentHtml = processedContent.toString()
```

## TypeScriptでfetchしたデータに型を付ける

`types/Response.ts` を作成してGitHub APIから返ってくるデータに対して型を定義した。これを `import { ArticleResponse } from '../types/Response'` して使っている。

```tsx
export type ArticleResponse = {
  name: string;
  sha: string;
  size: number;
  url: string;
  html_url: string;
  git_url: string;
  download_url: string;
  type: string;
  _links: {
    self: string;
    git: string;
    html: string;
  }
}

export type UserResponse = {
  login: string;
  id: number;
  node_id: string;
  avatar_url: string;
  gravatar_id: string;
  url: string;
  html_url: string;
  followers_url: string;
  following_url: string;
  gists_url: string;
  starred_url: string;
  subscriptions_url: string;
  organizations_url: string;
  repos_url: string;
  events_url:  string;
  received_events_url: string;
  type: string;
  site_admin: boolean;
  name: null | string;
  company: null | string;
  blog: string;
  location: null | string;
  email: null | string;
  hireable: null | string;
  bio: null | string;
  twitter_username: null | string;
  public_repos: number;
  public_gists: number;
  followers: number;
  following: number;
  created_at: string;
  updated_at: string;
}
```

整形した記事データには `types/Article.ts` を使う。

```tsx
export type Article = {
  id: string;
  title: string;
  emoji: string;
  type: string;
  topics: string[];
  published: boolean;
  date: string;
  content: string;
}
```

## 取得したデータで記事ページを作成する。

記事ページは `[id].tsx` とする事で一つのファイルで複数のパスページを生成出来る。

パスの指定は `getStaticPaths()` で行う。必須のパラメータは `paths, fallback` になる。 `fallback` は事前にビルドしたパス以外にアクセスがあった場合の動作を `true, false`  で決める。 `false` の場合は404ページが表示される。

`True` であれば404にアクセスするとフォールバック版のページを表示するようになる。これはあらかじめ静的するページが多いECサイト等でビルドに膨大な時間をかけるのを改善するために全てページをビルドせず、一定のページはリクエストもらったら生成するように実装したい場合に使用される。

`paths` キーはどのパスがプリレンダリングされるか決定する。動的ルートを使用した `pages/posts/[id].js` というページがある。このページに `getStaticPaths` をエクスポートして `paths` に次の値を返すようにする。 `paths` には ページのパス名として使いたい値を入れておけば良い。それに合わせてページを作成してくれる。中身のデータ等は別で定義する。zennでいう所の `slug` が `id` に入る値になる。 

※ [https://zenn.dev/unemployed/articles/3c8a872a210ded](https://zenn.dev/unemployed/articles/3c8a872a210ded)

ここが [`3c8a872a210ded`](https://zenn.dev/unemployed/articles/3c8a872a210ded) slugに当たる。

```jsx
return {
  paths: [
    { params: { id: '1' } },
    { params: { id: '2' } }
  ],
  fallback: ...
}
```

## 簡単な例

`pages/posts/[id].js` というページごとに 1 件のブログ記事をプリレンダリングする例、ブログ記事の一覧はCMSから取得され、 `getStaticPaths` で返される。

```jsx
// pages/posts/[id].js

function Post({ post }) {
  // 記事をレンダリングします...
}

// この関数はビルド時に呼び出されます。
export async function getStaticPaths() {
  // 外部APIエンドポイントを呼び出して記事を取得します。
  const res = await fetch('https://.../posts');
  const posts = await res.json();

  // 記事に基づいてプリレンダリングしたいパスを取得します
  const paths = posts.map(post => ({
    params: { id: post.id }
  }));

  // ビルド時にこれらのパスだけをプリレンダリングします。
  // { fallback: false } は他のルートが404になることを意味します。
  return { paths, fallback: false };
}

// ビルド時にも呼び出されます。
export async function getStaticProps({ params }) {
  // paramsは記事の`id`を含みます。
  // ルートが/posts/1のような時、params.id は1です。
  const res = await fetch(`https://.../posts/${params.id}`);
  const post = await res.json();

  // 記事データをprops経由でページに渡します。
  return { props: { post } };
}

export default Post;
```

## techBlogで使用してる[id].tsx

`index.tsx` で一度取得したデータ渡したりする方法が無いみたいなので、再び一度取得したデータにfetchを投げている。 そこがなんか二度手間で気持ち悪い気がするけど、これしか手がない。もしいい方法があったら教えて下さい。

```tsx
import Layout from '../../components/layout'
import { getAllPostIds, getPostsData, getHtmlContent, getPostData } from '../../lib/posts'
import { getUserData } from '../../lib/user'
import Head from 'next/head'
import Date from '../../components/date'
import Social from '../../components/social'
import { GetStaticProps, GetStaticPaths } from 'next'
import { Article } from '../../types/Article'
import { UserResponse } from '../../types/Response'

export default function Post({ 
    postData,
    userData
}: { 
    postData: Article 
    userData: UserResponse
}) {
    return (
        <Layout avatarUrl={userData.avatar_url}>
            <Head>
                <title>{postData.title}</title>
            </Head>
            <article className="w-11/12 lg:max-w-4xl mx-auto pb-7">
                <h1>{postData.title}</h1>
                <div>
                    <Date dateString={postData.date} />
                </div>
                <div className="pt-7">
                    <p className="font-bold text-blue-darker">あとで読む</p>
                    <Social title={postData.title} id={postData.id} topics={postData.topics} /> 
                </div>
                <div dangerouslySetInnerHTML={{ __html: postData.content }} />
            </article>
        </Layout>
    )
}

export const getStaticPaths: GetStaticPaths = async () => {
    const allPostsData = await getPostsData()
		// https://techblog-pink.vercel.app/ファイル名みたいな形でページを生成してくれる。
    const paths = getAllPostIds(allPostsData)
    return {
        paths,
        fallback: false
    }
}

//ここで記事のデータを取得して上のPost()のprops引数で渡してレンダリングする。
export const getStaticProps: GetStaticProps = async ({ params }) => {
    const allPostsData = await getPostsData()
    const postData = getPostData(allPostsData, params.id as string).shift()
    const convertedPostData = await getHtmlContent(postData)
    const userData = await getUserData()
    return {
        props: {
            postData: convertedPostData,
            userData: userData
        }
    }
}
```

## ページ遷移するには

Next.jsは `pages` ディレクトリ（Next.jsに置いて特別な意味をもつ）配下に `js/ts` ファイルを配置することで、URLのマッピングを自動で行う。

```bash
--project
  --pages
    -- index.js -> "/"でアクセス可能
    -- posts
      -- first-post.js -> "/posts/first-post"でアクセス可能
```

## <Link>（Link Component）

通常ページを移動させたい際に `<a>` タグを使用するが、Next.jsでは `<link>` コンポーネントを使用する。これを使用しないとページを移動する際に通信が発生してページ更新が入る。そのため速度低下する。

### 使い方

- `<a>` タグの場合

```jsx
<h1 className="title">
  Learn <a href="https://nextjs.org">Next.js!</a>
</h1>
```

- `<Link>` コンポーネントの場合

```jsx
<h1 className="title">
  Read{' '}
  <Link href="/posts/first-post">
    <a>this page!</a>
  </Link>
</h1>
```

## 各種コンポーネント

## date.tsx

mdのメタ情報 `date` から日付フォーマットを変更する。

```tsx
import { parseISO, format } from 'date-fns'

/*
    cccc: Monday, Tuesday, ..., Sunday

    LLLL: January, February, ..., December

    d: 1, 2, ..., 31

    yyyy: 0044, 0001, 1900, 2017
*/

export default function Date({ dateString }: { dateString: string }) {
  const reDateString = dateString.replace(/\./g, '-')
  const date = parseISO(reDateString)
  return <time className="w-1/2 text-right" dateTime={dateString}>{format(date, 'LLLL d, yyyy')}</time>
}
```

### layout.tsx

各ページで共通のコンポーネントはここに記述する。 `Navbar` 等、 `children` は `index.tsx, [id].tsx` 内で挟まれたJSXがそれぞれのページで代入される。

```tsx
import Link from 'next/link'
import Head from 'next/head'
import { Navbar } from './menu';

export const siteTitle = '大学生だった'

//homeの型定義の?は必須の引数でない時に付ける。
export default function Layout({
    children,
    avatarUrl,
}: {
    children: React.ReactNode
    avatarUrl?: string
}) {
    return (
        <div>
            <Head>
                <link rel="icon" href="/favicon.ico" />
                <meta
                    name="description"
                    content="Zennに投稿した記事を使用して作成したオリジナルブログ、プログラミング技術に関する内容を投稿します。"
                />
                <meta
                    property="og:image"
                    content={`https://og-image.vercel.app/${encodeURI(siteTitle)}.png?theme=light&md=0&fontSize=75px&images=https%3A%2F%2Fassets.zeit.co%2Fimage%2Fupload%2Ffront%2Fassets%2Fdesign%2Fnextjs-black-logo.svg`}
                />
                <meta name="og:title" content={siteTitle} />
                <meta name="twitter:card" content="summary_large_image" />
            </Head>
            <div className="xl:flex">
              <Navbar avatarUrl={avatarUrl}/>
              <main className="lg:flex-1 bg-earth-lighter">{children}</main>
            </div>
        </div>
    )
}

//三項演算子の別の書き方らしい。JSX特有かは分からない。
//https://kei-s-lifehack.hatenablog.com/entry/2021/01/20/Next.js_Tutorial_--_header_%E9%83%A8%E5%88%86%E3%81%AE%E5%88%86%E5%B2%90
// {!home && (
//   <div>
//       <Link href="/">
//           <a>← Back to home</a>
//       </Link>
//   </div>
// )}
```

### menu.tsx

ハンバーガーメニューのコンポーネント（レスポンシブ対応）

home アイコンは[Heroicons](https://heroicons.com/)からJSXでコピーして貼り付ける。

Twitter, Githubのアイコンは[react-icons](https://react-icons.github.io/react-icons/) を使用する。

インストールは `npm install react-icons --save` を実行する。

```tsx
import Link from 'next/link';
import Image from 'next/image'
import { useState } from 'react';
import { aboutme } from '../aboutme'
import { FaTwitter } from "@react-icons/all-files/fa/FaTwitter";
import { FaGithub } from "@react-icons/all-files/fa/FaGithub";

export const Navbar = ({
  avatarUrl
  } : {
    avatarUrl: string
  }) => {
  const [active, setActive] = useState(false);

  const handleClick = () => {
    setActive(!active);
  };

  return (
    <>
      {/* 下記のコードは三項演算子を使用していてactiveがTrueなら空が適用される。それで要素が出てくる。最初はfalseで要素を隠す。 */}
      <nav className={`${
        active ? '' : '-translate-x-72'
      } xl:flex flex-wrap bg-earth-light p-1.5 fixed z-10 w-72 transform transition-transform xl:static xl:translate-x-0 xl:bg-blue-light overflow-y-auto disable-scrollbars inset-y-0 min-h-screen rounded-3xl rounded-tl-none rounded-bl-none xl:rounded-none`}
      >
        <div className="flex justify-end xl:hidden">
            <button
              className="inline-flex p-3 hover:bg-gray rounded outline-none text-blue-dark"
              onClick={handleClick}
            >
              <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5 lg:h-8 lg:w-8" viewBox="0 0 20 20" fill="currentColor">
                <path fillRule="evenodd" d="M7.707 14.707a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414l4-4a1 1 0 011.414 1.414L5.414 9H17a1 1 0 110 2H5.414l2.293 2.293a1 1 0 010 1.414z" clipRule="evenodd" />
              </svg>
            </button>
          </div>
        <div className="w-60 mx-auto">
          <div className="text-blue-dark flex justify-between mt-7">
            <div>
              <Image
                src={avatarUrl}
                alt="avatar"
                width={80}
                height={80}
                className="rounded-3xl"
              />
            </div>
            <div>
              <p className="text-xl font-bold">大学生だった.</p>
              <ul className="text-xs font-extralight mt-2">
                <li><a href={aboutme.twitterURL} target="_blank"><FaTwitter />　{aboutme.twitterID}</a></li>
                <li><a href={aboutme.githubURL} target="_blank"><FaGithub />　{aboutme.githubID}</a></li>
              </ul>
            </div>
          </div>
          <div className="text-sm text-blue-dark mt-2">
            <p className="text-base font-bold">自己紹介</p>
            <p>{aboutme.description}</p>
            <br/>
            <p className="text-base font-bold">使う技術</p>
            <p>{aboutme.tech}</p>
            <br/>
            <p className="text-base font-bold">今やってる事</p>
            <p>{aboutme.lately}</p>
            <br/>
            <p className="text-base font-bold">今後やりたい事</p>
            <p>{aboutme.future}</p>
          </div>
          <div className='flex flex-col text-blue-dark mt-10 mb-10'>
            <Link href='/'>
              <a className='inline-flex lg:w-auto w-full px-3 py-2 rounded font-bold items-center justify-center '>
                <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                  <path d="M10.707 2.293a1 1 0 00-1.414 0l-7 7a1 1 0 001.414 1.414L4 10.414V17a1 1 0 001 1h2a1 1 0 001-1v-2a1 1 0 011-1h2a1 1 0 011 1v2a1 1 0 001 1h2a1 1 0 001-1v-6.586l.293.293a1 1 0 001.414-1.414l-7-7z" />
                </svg>
                Home
              </a>
            </Link>
          </div>
        </div>
      </nav>
      <div className="bg-earth-lighter p-1.5 xl:hidden">
        <button
          className="inline-flex p-3 hover:bg-gray rounded outline-none lg:text-lg"
          onClick={handleClick}
        >
          <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6 lg:h-10 lg:w-10" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6h16M4 12h16M4 18h7" />
          </svg>
        </button>
      </div>
    </>
  );
};
```

### Social.tsx

各種リンクにページのリンクを設定する。

Twitter, facebook, はてな, Line, Pocket, feedly（RSSフィードのURLが必要みたいで設定してない）

```tsx
import { FaTwitter } from "@react-icons/all-files/fa/FaTwitter";
import { FaFacebookSquare } from "@react-icons/all-files/fa/FaFacebookSquare";
import { FaLine } from "@react-icons/all-files/fa/FaLine";
import { FaGetPocket } from "@react-icons/all-files/fa/FaGetPocket";
import { SiHatenabookmark } from "@react-icons/all-files/si/SiHatenabookmark";
import { SiFeedly } from "@react-icons/all-files/si/SiFeedly";

const websiteUrl = "https://techblog-pink.vercel.app/"

export default function Social({ title, id, topics }: { title:string; id: string; topics: string[] }) {
  return (
    <>
      <div className="flex max-w-sm flex-wrap justify-between mt-3">
        <button className="w-12 h-12 md:w-14 md:h-14 mr-1 mb-1 p-3 bg-gray-light rounded-2xl">
          <a href={`https://twitter.com/share?text=後で読む：${title}&hashtags=${topics}&url=${websiteUrl}posts/${id}&related=Unemployed_jp`}
        target='_blank' rel='noopener noreferrer'>
            <FaTwitter />
          </a>
        </button>
        <button className="w-12 h-12 md:w-14 md:h-14 mr-1 mb-3 p-3 bg-gray-light rounded-2xl">
          <a href={`http://www.facebook.com/share.php?u=${websiteUrl}posts/${id}`} >
            <FaFacebookSquare />
          </a>
        </button>
        <button className="w-12 h-12 md:w-14 md:h-14 mr-1 mb-3 p-3 bg-gray-light rounded-2xl">
          <a href={`https://b.hatena.ne.jp/entry/${websiteUrl}posts/${id}`} data-hatena-bookmark-layout='touch-counter'
            title={title} target='_blank' rel='noopener noreferrer'>
              <SiHatenabookmark />
          </a>
        </button>
        <button className="w-12 h-12 md:w-14 md:h-14 mr-1 mb-3 p-3 bg-gray-light rounded-2xl">
          <a href={`https://social-plugins.line.me/lineit/share?url=${websiteUrl}posts/${id}`} target='_blank'>
            <FaLine />
          </a>
        </button>
        <button className="w-12 h-12 md:w-14 md:h-14 mr-1 mb-3 p-3 bg-gray-light rounded-2xl">
          <a href={`http://getpocket.com/edit?url=${websiteUrl}posts/${id}`} target='_blank' rel="nofollow">
            <FaGetPocket />
          </a>
        </button>
        <button className="w-12 h-12 md:w-14 md:h-14 mr-1 mb-3 p-3 bg-gray-light rounded-2xl">
          <a href={`http://cloud.feedly.com/#subscription/feed/フィードURLたぶんRSS`} target='blank'>
            <SiFeedly />
          </a>
        </button>
      </div>
    </>
  );
}
```

### topics.tsx

記事に設定したトピックを取得して表示する。

```tsx
export default function Topics({ topicList }: { topicList: string[] }) {
  return (
    <>
      {topicList.map(topic => (<span className="mr-2.5" key={topic}>{topic}</span>))}
    </>
  );
}
```

## Vercelで環境変数を取り扱うには

Next.jsでは環境変数を設定するには `.env` ファイルを使用する。

```tsx
GITHUB_TOKEN=xxx
NEXT_PUBLIC_GOOGLE_ANALYTICS_ID=xxx
```

これを `.gitignore` に設定しておくのだがそうするとVercelにデプロイする際に読み込まれないので、あらかじめVercelに環境変数を設定しておく必要がある。デプロイした後にしか設定出来ないので設定して再びデプロイする必要がある。Project Settings > Enviroment Variables

NameとValueにそれぞれ値を設定する。例 NameにGitHub_Token, Value=xxx

![https://storage.googleapis.com/zenn-user-upload/nqxxyyh5rnu0c67p5kywzsw34pzh](https://storage.googleapis.com/zenn-user-upload/nqxxyyh5rnu0c67p5kywzsw34pzh)

## TailwindをNext.jsで使用する。

公式の沿ってインストールしていく。

### インストール方法

Next.jsのバージョンでインストール方法が変わるのでバージョンを確認する。

```bash
npm info next version

# 自分の環境
10.1.2
```

Next.jsのバージョンは10以上なのでこちらのコマンドを実行する。

```bash
npm install -D tailwindcss@latest postcss@latest autoprefixer@latest
```

次に `tailwind.config.js` と `postcss.config.js` を下記のコマンドを実行してNext.jsのルートフォルダに作成する。

```bash
npx tailwindcss init -p
```

※tailwindはpostcssから作られているので、これらの設定も必要とする。Next.jsで使用する場合はデフォルトで使用出来るので追加の編集はいらない。

postcssとは

[PostCSSとは何か - morishitter blog](https://morishitter.hatenablog.com/entry/2015/08/03/164424)

プロジェクトのルートに `tailwind.config.js` ファイルが生成される。

次に `tailwind.config.js` を編集する。

**tailwind.config.js**

```jsx
// tailwind.config.js
module.exports = {
	/*
		purgeにNext.jsで使用される予定だったcss moduleファイル
		を取り除くコードを追加する。
		purge: [],ここを下記のように変更
	*/
	purge: ['./pages/**/*.{js,ts,jsx,tsx}', './components/**/*.{js,ts,jsx,tsx}'],
  // purge: [],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
```

こちらのファイルも作成される。

**postcss.config.js**

```jsx
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### 使用方法

公式によると2つの方法があるらしい。

1：JSファイルにimportして使う。

```jsx
// pages/_app.js
// import '../styles/globals.css'
import 'tailwindcss/tailwind.css'

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}
export default MyApp
```

この場合はNext.jsで通常使用されるcss module、 `global.css` と `Home.module.css` 等を削除する事が出来る。

2： CSSにTailwindを含める。おそらく普通にcss moduleを使用してそこにTailwindを書き込む方法だと思う。

`./styles/globals.css` を開いてそこに `@tailwind` を書いていく。

個人的にはこちらの方が汎用性があるのでこちらを採用する。

```css
/* ./styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/*この続きに普通にcssを追加できる。*/
h1 {
	font-size: 1rem;
}

/*@applyを使えばtailwindの構文も使える。*/

h2 {
  @apply font-bold text-xl pt-4 pb-4;
}

/* 独自のタグも作成出来る。 */
.btn-blue {
  @apply bg-blue-500 text-white font-bold py-2 px-4 rounded;
}
```

このファイルを `_app.js` で `import` する。

```jsx
import '../styles/globals.css'

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}

export default MyApp
```

公式のサンプル

[vercel/next.js](https://github.com/vercel/next.js/tree/canary/examples/with-tailwindcss)

### 独自のカラーテーマを使用するには

`tailwind.config.js` の `theme` に下記のように追加する。使用する際は `bg-blue-light` のように使用する。 

※これを設定するとデフォルトの `bg-blue-100` みたいなのが使えなくなる。

```jsx
module.exports = {
  purge: ['./pages/**/*.{js,ts,jsx,tsx}', './components/**/*.{js,ts,jsx,tsx}'],
  darkMode: false, // or 'media' or 'class'
  theme: {
    colors: {
      blue: {
        light: '#b9d7ea',
        DEFAULT: '#769fcd',
        dark: '#112d4e',
      },
      earth: {
        light: '#f9f7f7',
        DEFAULT: '#BDBDBD',
        dark: '#757575',
      },
      gray: {
        dark: '#212121',
        DEFAULT: '#d6e6f2',
        light: '#f7fbfc'
      }
    },
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
```

## Favicon、画像等のファイルを置く場所

### Aseets （webで使用する素材の総称を指すと思われる）

Next.jsでは static assets（画像、アイコン、静的なhtmlファイル）などを `public` ディレクトリ配下に配置する。

```bash
--project
  --public
    -- hoge.jpg
    -- huga.svg
		-- favicon.ico
```

画像の表示は `<img>` タグを拡張した、 `<Image>` コンポーネントを使用する。

このコンポーネントを使用すれば、画像の最適化をNext.jsが自動的に行う。

最適化はCMSなどの他サーバで管理している画像に関しても最適化を行う。

【最適化の例】

- 画像のリサイズを行う。
- jpgファイルをwebPなどの軽量なフォーマットに変換する。

画像の最適化はビルド時に一括で行うのではなく、ユーザがRequestするたびに適宜行っていく。

画像が大量にあったとしてもBuild時間が大幅にかからない。

読み込みもviewportスクロールされた時に初めて画像が読み込まれる。

`width` と `height` を指定しておくと、そのサイズまであらかじめ画像を圧縮する。

また、レスポンス表示で幅が小さくなった場合も自動でそのサイズにトリミングした画像を生成する。

```jsx
//node_modulesのnextというフォルダにimage.jsがある。
import Image from 'next/image'

const YourComponent = () => (
	<Image
		src="/images/profile.jpg"
		height={144}
		width={144}
		alt="Your Name"
	/>
)
```

### メタデータ

Next.jsでは `<head>` タグではなく `<Head>` コンポーネントを使用する。

これにより、ページごとに動的にMetadataを変更する事が可能。

```jsx
import Head from 'next/head'

// メタタグ内に<div>要素が追加されるのは記法としておかしいため
// フラグメントが挟んである。
export default function FirstPost() {
	return (
		<> 
			<Head>
				<title>First Post</title>
				{/*こんな感じにfaviconは配置する*/}
				<link rel="icon" href="/favicon.ico" />
			</Head>
			<h1>First Post</h1>
			<h2>
				<Link href="/">
					<a>Back to home</a>
				</Link>
			</h2>
		</>
	)
}
```

※ `<>` 空のタグに見えるものはReactのフラグメントを省略記号で記法したものである。 `<React.Fragment>` と同じ意味になる。レンダーで子要素を返す際に要素が `div` タグで挟まれるの回避するためにある。

```html
<table>
  <tr>
		<!--
			tdをrender()で返す場合に<>を記法しないと
			divに挟まれる。
		-->
    <div> 
      <td>Hello</td>
      <td>World</td>
    </div>
  </tr>
</table>
```

詳しくは

[フラグメント - React](https://ja.reactjs.org/docs/fragments.html)

## google analyticsを導入する。

UAコードの取得は[この記事](https://t.co/kxYDlLaFGi?amp=1)を参考にすると取れる。今GAが主流みたいでどこにあるんだ〜って結構作成するのに時間かかった。

導入するには `_app.tsx, _document.tsx` と `lb/gtag.ts, src/hooks/usePageView.ts` が必要になる。

### TypeScriptに対応させる。（型を導入する。）

これはインストールするだけで特に `import` する必要はない。最初 `import` 必要だと思ってあちこち記事を探したけど何もしなくてよかった。

```bash
npm install --save @types/google.analytics
```

### _app.tsx

ここで `usePagaView()` を実行する。

```tsx
import '../styles/globals.css'
import "../styles/prism-daigakusei.css"
import "katex/dist/katex.min.css"
import usePageView from '../src/hooks/usePageView'
import { AppProps } from 'next/app'

function MyApp({ Component, pageProps }: AppProps) {
  usePageView()
  return <Component {...pageProps} />
}

export default MyApp
```

## usePageView.ts

これが何してるかと言うと、next.jsではページが切り替わる際、JavaScript（有効なら）で切り替えているのでURLもJavaScriptを使って変更している。そのため通信が発生していない。なので `gtag.ts` でユーザの行動が追跡出来ない。それを解消するためにURLの変更が行われたら `gtag.ts` が実行されるように関数をラップしている。

```tsx
import { useEffect } from 'react'
import Router from 'next/router'

import * as gtag from '../../lib/gtag'

export default function usePageView() {
  //関数型コンポーネントのライフサイクル
  useEffect(() => {
    if (!gtag.existsGaId) {
      return
    }

    const handleRouteChange = (path: string) => {
      gtag.pageview(path)
    }
    //componentDidMountの役割URLが変更されるたびにhandleRouteChangeが実行される。
    Router.events.on('routeChangeComplete', handleRouteChange)
    
    //componentWillUnmontの役割
    return () => {
      Router.events.off('routeChangeComplete', handleRouteChange)
    }
  }, [Router.events])
}
```

### gtag.ts

```tsx
import { Event } from '../types/GoogleAnalyticsEvent'
export const GA_ID = process.env.NEXT_PUBLIC_GOOGLE_ANALYTICS_ID

// IDが取得できない場合を想定する
export const existsGaId = GA_ID !== ''

// PVを測定する
export const pageview = (path: string) => {
  window.gtag('config', GA_ID, {
    page_path: path,
  })
}

// GAイベントを発火させる
export const event = ({action, category, label}: Event) => {
  if (!existsGaId) {
    return
  }

  window.gtag('event', action, {
    event_category: category,
    //JavaScriptのオブジェクトをJSONの文字列に変換している。
    event_label: JSON.stringify(label),
  })
}
```

## 最後に
Reactのチュートリアルから初めてとても長い道のりでしたが、自分のブログ + 新しいデザイン・アニメーション・JavaScriptを試したり出来る実験場を持つ事が出来て嬉しいです。
Next.jsは最初は訳が分からなくて「このフレームワークは比較的軽いよ」と言ってた人に嘘だろと思っていたのですが少し慣れてくるとそんな気もするようになりました。
GitHubのAPIは不安定な日があったりするのでビルドをする時間帯は気にした方が良いかもしれないです。そこだけが少しデメリットかもしれないです。
あとはリポジトリを新しく作成すればZennとは別で記事を書いたりも出来ます。
ここまで記事を読んで下さりありがとうございました。参考にした記事書いて下さった方、スタックオーバーフロー・Discordで質問に回答して下さった方々にとても感謝しています。おかげでここまで作りきることが出来ました。

## 今後欲しい機能

🔨：ダークモード

🔨：ページネーション

🔨：記事ページでのレコメンド（タグ名から）

🔨：RSS対応

🔨：zenn-contentリポジトリが更新されたらVercelで再ビルドが走るようにしたい。

### 参照

[Next.js 9.3新API getStaticProps と getStaticPaths と getServerSideProps の概要解説 - Qiita](https://qiita.com/matamatanot/items/1735984f40540b8bdf91)

[Next.jsのgetStaticPropsで外部APIからデータを取得する方法｜Playground発！アプリ開発会社の技術ブログ](https://tech.playground.style/javascript/fetch-api-data/)

[Next.js 9.3の変更点](https://the2g.com/post/nextjs-9-3)

[大幅にリニューアルされた Next.js のチュートリアルをどこよりも早く全編和訳しました - Qiita](https://qiita.com/thesugar/items/01896c1faa8241e6b1bc)

[Install Tailwind CSS with Next.js - Tailwind CSS](https://tailwindcss.com/docs/guides/nextjs)

[Next.js with Tailwind CSS 環境構築](https://zenn.dev/k_logic24/articles/next-with-tailwind)

[Customizing Colors - Tailwind CSS](https://tailwindcss.com/docs/customizing-colors)

[Next.jsにTailwind CSS 2.0を導入する](https://zenn.dev/akakuro/articles/d39e939e72c321)

[初めてでもわかるNext.jsの基礎(React) | アールエフェクト](https://reffect.co.jp/react/next-js)

[Basic Features: データ取得 | Next.js](https://nextjs-ja-translation-docs.vercel.app/docs/basic-features/data-fetching#fetching-data-on-the-client-side)

[Routing: はじめに | Next.js](https://nextjs-ja-translation-docs.vercel.app/docs/routing/introduction)

[Next.js 9.3新API getStaticProps と getStaticPaths と getServerSideProps の概要解説 - Qiita](https://qiita.com/matamatanot/items/1735984f40540b8bdf91)

[【Next.js】チュートリアルの実施+周辺知識のキャッチアップ](https://zenn.dev/yuki_yuki/scraps/5b28f6a278db3c)

[Next.js を使った JAMstack なブログの始め方](https://gotohayato.com/content/517/)

[Next.jsでGoogle Analyticsを使えるようにする](https://panda-program.com/posts/nextjs-google-analytics#typescript%E5%AF%BE%E5%BF%9C%E3%82%92%E3%81%99%E3%82%8B)

[](https://tailwindcss.jp/docs/responsive-design)

[Tailwind CSS Cheat Sheet](https://nerdcave.com/tailwind-cheat-sheet)

### 一応読んだcreate-react-appではなく、Next.jsを使う理由

[なぜNext.jsを採用するのか？ - mottox2 blog](https://mottox2.com/posts/429)

### サイトの配色決めで利用した。

[Palette List - Color Palette Generator - 10,000+ Palettes](https://www.palettelist.com/c29867/bccdde)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。