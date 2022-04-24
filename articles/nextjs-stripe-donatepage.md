---
title: "Next.jsで作成したブログにStripeを使って決済ページを実装してみた（TypeScript対応済み）" # 記事のタイトル
emoji: "💳" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["stripe", "javascript", "react", "nextjs", "typescript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2022.04.24'
---

## 最初に

> ブログに stripeAPI を使って決済画面を作る

学習予定に入ってる課題を達成したので、どのように実装して今理解出来ているStripe APIについてまとめて行こうと思う。

![stripe-donate](https://user-images.githubusercontent.com/23703281/164968927-e3add193-0491-4c80-a38b-eefa6c36bf0e.png)

※ 今回実際にNext.jsのブログに追加した[寄付を募るページ](https://techblog-pink.vercel.app/donate/price)

[blog - リポジトリ](https://github.com/wimpykid719/blog) コード全体はここで確認できる。

## カスタム決済のサンプルのコードをTypeScript化した

stripe Docs - カスタムの支払いフロー

[https://stripe.com/docs/payments/quickstart](https://stripe.com/docs/payments/quickstart)

野良のブログ等を読んでいたが情報が古かったりしたので、公式のサンプルに型を当てながら機能を見て行く事にした。

処理の流れは下記のようになっている。

![Stripeシステム](https://user-images.githubusercontent.com/23703281/164968971-ad8a4323-23d8-4952-8522-5471cf448aca.png)

出来上がったのが下記のレポジトリdockerですぐに環境構築出来るようになっています。

[https://github.com/wimpykid719/stripe-sample-typescript](https://github.com/wimpykid719/stripe-sample-typescript)

Reactのフロントエンドを担う箇所のコード `useEffect` で初回ロード時に [localhost:4242/create-payment-intent](http://localhost:4242/create-payment-intent) にリクエストを投げる。

`App.tsx`

```tsx
import React, { useState, useEffect } from "react";
import { loadStripe, StripeElementsOptions, Appearance } from "@stripe/stripe-js";
import { Elements } from "@stripe/react-stripe-js";

import CheckoutForm from "./CheckOutForm";
import "./App.css";

// Make sure to call loadStripe outside of a component’s render to avoid
// recreating the Stripe object on every render.
// loadStripe is initialized with a fake API key.
// Sign in to see examples pre-filled with your key.
const stripePromise = loadStripe('pk_test_46zswMCbz39W2KAqKj43vDRu')

export default function App() {
  const [clientSecret, setClientSecret] = useState("");

  useEffect(() => {
    // Create PaymentIntent as soon as the page loads
    fetch("/create-payment-intent", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ items: [{ id: "xl-tshirt" }] }),
    })
      .then((res) => res.json())
      .then((data) => setClientSecret(data.clientSecret));
  }, []);

  const appearance: Appearance = {
    theme: 'stripe',
  };
  const options: StripeElementsOptions = {
    clientSecret,
    appearance,
  };

  return (
    <div className="App">
      {clientSecret && (
        <Elements stripe={stripePromise} options={options}>
          <CheckoutForm />
        </Elements>
      )}
    </div>
  )
}
```

`amount` に決済したい値段を設定する。

バックエンドでStripeにリクエストを投げて `PaymentIntent` が発行される。

受け取った `PaymentIntet` はフロントエンドの`setClientSecret(data.clientSecret)` に格納されStripeフォームに渡る。

`server.ts`

```tsx
// Set your secret key. Remember to switch to your live secret key in production.
// See your keys here: https://dashboard.stripe.com/apikeys

import Stripe from 'stripe'
import express from 'express'

const stripe = new Stripe('sk_test_09l3shTSTKHYCzzZZsiLl2vA', { apiVersion: '2020-08-27' })

const app = express()

app.use(express.static('public'))
app.use(express.json())

// オーダの金額を計算するコードを書く
const calculateOrderAmount = (items: Items) => {
  const amount = items
  // ここで計算する処理を書く
  // Replace this constant with a calculation of the order's amount
  // Calculate the order total on the server to prevent
  // people from directly manipulating the amount on the client
  return 1400
}

type Items = {
  id: string[]
}

type ItemsBody = {
  items: Items
}

interface CustomRequest<T> extends express.Request {
  body: T
}

app.post('/create-payment-intent', async (req: CustomRequest<ItemsBody>, res: express.Response) => {
  const { items }: ItemsBody = req.body

  // Create a PaymentIntent with the order amount and currency
  const paymentIntent: Stripe.PaymentIntent = await stripe.paymentIntents.create({
    amount: calculateOrderAmount(items),
    currency: 'usd',
  })

  res.send({
    clientSecret: paymentIntent.client_secret,
  })
})

app.listen(4242, () => {
  console.log('Running on port 4242')
})
```

StripeフォームでPaymentIntent（決済したい値段の情報）を受け取ってフォームを表示する。

表示されたフォームにクレジットカード情報を入力して Pay Now ボタンをクリックすると Stripeにクレジットカード情報が飛んで実際に決済が行われる。

`CheckOutForm.tsx`

```tsx
import React, { useState, useEffect } from "react";
// stripeのコンポーネントを読み込んでる
import {
  PaymentElement,
  useStripe,
  useElements
} from "@stripe/react-stripe-js";

import { PaymentIntentResult } from '@stripe/stripe-js'

// import { CardElementType } from './types/stripe'

export default function CheckoutForm() {
  // 第一配列がデフォルトの値, 第二配列がそれを変更する関数が入る

  const stripe = useStripe();
  const elements = useElements();

  const [message, setMessage] = useState<string | undefined | null>(null);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    if (!stripe) {
      return;
    }

    //URLのクエリから値を取得する
    const clientSecret = new URLSearchParams(window.location.search).get(
      "payment_intent_client_secret"
    );

    // リダイレクト時にここに値が入る
    if (!clientSecret) {
      return;
    }

    stripe.retrievePaymentIntent(clientSecret).then(({ paymentIntent }: PaymentIntentResult) => {
      switch (paymentIntent?.status) {
        case "succeeded":
          setMessage("Payment succeeded!");
          break;
        case "processing":
          setMessage("Your payment is processing.");
          break;
        case "requires_payment_method":
          setMessage("Your payment was not successful, please try again.");
          break;
        default:
          setMessage("Something went wrong.");
          break;
      }
    });
  }, [stripe]);

  const handleSubmit: React.FormEventHandler<HTMLFormElement> = async (e) => {
    e.preventDefault();

    if (!stripe || !elements) {
      // Stripe.js has not yet loaded.
      // Make sure to disable form submission until Stripe.js has loaded.
      return;
    }

    setIsLoading(true);

    const { error } = await stripe.confirmPayment({
      elements,
      confirmParams: {
        // Make sure to change this to your payment completion page
        return_url: "http://localhost:8080",
      },
    });

    // This point will only be reached if there is an immediate error when
    // confirming the payment. Otherwise, your customer will be redirected to
    // your `return_url`. For some payment methods like iDEAL, your customer will
    // be redirected to an intermediate site first to authorize the payment, then
    // redirected to the `return_url`.
    if (error.type === "card_error" || error.type === "validation_error") {
      setMessage(error.message);
    } else {
      setMessage("An unexpected error occured.");
    }

    setIsLoading(false);
  };

  return (
    <form id="payment-form" onSubmit={handleSubmit}>
      <PaymentElement id="payment-element" />
      <button disabled={isLoading || !stripe || !elements} id="submit">
        <span id="button-text">
          {isLoading ? <div className="spinner" id="spinner"></div> : "Pay now"}
        </span>
      </button>
      {/* Show any error or success messages */}
      {message && <div id="payment-message">{message}</div>}
    </form>
  );
}
```

一度、バックエンドを通して値段に関する情報発行してフロントエンドに渡しているので、値段を不正に変更してリクエストを遅れないようになっている。

## Next.jsのブログに寄付ページを作成

上記のサンプルを元にNext.jsで作成したブログに新たに決済ページを作成してみる。

Next.jsで新しいページを作成する際には `pages` フォルダにファイルを作成するとそのファイル名がそのままパスになる。 寄付関連のページでまとめたいので `donate` フォルダを作成してそこにファイルを追加していく。 最終的には `price.tsx` , `checkout.tsx` , `complete.tsx` の3つになる。パスは `/donate/price` , `/donate/checkout`, `/donate/complete` となる。

### 募金額を選択するページ

ここは最初に寄付金額を選択するページを担う場所になる。

ラジオボタンを用いて金額を選択出来るようになっている。デザインはグラデーションを使用した物を作ってみたくて下記のサイトで生成したcssを適応している。

[CSS Gradient Generator](https://www.joshwcomeau.com/gradient-generator/)

![price](https://user-images.githubusercontent.com/23703281/164969003-cd98eaa5-69ef-4b16-9355-c9c31be04de4.png)

決済へをクリックするとURLに選択された料金idをクエリとして持たせて `donate/checkout` に遷移させる。

`price.tsx`

```tsx
// next.js
import Head from 'next/head'
import { useRouter } from 'next/router';
import { prices } from '../../techBlogSettings/pricelist'

// React
import { useState } from "react";

import { donateTitle } from '../../components/layout'
import PriceCard from '../../components/stripe/atoms/pricecard'

export default function Price() {
  const router = useRouter()
  const [donateId, setDonateId] = useState('1');

  const handleSubmit: React.FormEventHandler<HTMLFormElement> = async (e) => {
    // formにURLが指定されてない時デフォルトの操作で現在のURLにpostを投げてページを更新させてしまうので
    // それを防ぐために下記のコードを実行している
    e.preventDefault()
    // e.currentTarget.elementsこれでformの値をまとめて取れるけど今回はステートに既に値があるので
    // そちらを使用する
    // donateIdを渡して次のページに遷移する
    router.push({
      pathname: '/donate/checkout',
      query: { donate: donateId }
    });
  }

  const changePrice = (event: React.ChangeEvent<HTMLInputElement>) => {
    setDonateId(event.target.value);
  }

  return (
    <main className="bg-gray-light">
      <div className="lg:mx-auto max-w-2xl min-h-screen mx-auto">
        <Head>
          <title>{donateTitle}</title>
          <link rel="icon" href="/favicon/favicon.ico" />
        </Head>
        <div>
          <h1 className="text-2xl font-normal pl-3">募金額を選択</h1>
        </div>
        <form className="py-8" onSubmit={handleSubmit}>
          <div className="flex flex-wrap justify-center md:justify-between">
            {prices.map((price, id) => (
              <PriceCard key={id} id={price.id} amount={price.amount} donateId={donateId} message={price.message} onChange={changePrice}/>
            ))}
          </div>
          <button
            className="w-36 h-12 rounded-full text-white bg-stripe custom-box-shadow mx-auto flex justify-center items-center"
            type="submit"
          >
            <span className="block mr-3">決済へ</span>
            <span className="block h-8 w-8 rounded-full custom-area-opacity">
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32" className="mt-1 h-4 w-4" fill="currentColor">
                <path d="M19.414 27.414l10-10c0.781-0.781 0.781-2.047 0-2.828l-10-10c-0.781-0.781-2.047-0.781-2.828 0s-0.781 2.047 0 2.828l6.586 6.586h-19.172c-1.105 0-2 0.895-2 2s0.895 2 2 2h19.172l-6.586 6.586c-0.39 0.39-0.586 0.902-0.586 1.414s0.195 1.024 0.586 1.414c0.781 0.781 2.047 0.781 2.828 0z"></path>
              </svg>
            </span>
          </button>
        </form>
      </div>
    </main>
  )
}
```

### 決済を行うページ

先ほどのページから遷移されて実際にクレジットを入力するページが下記に当たる。

PaymentIntentを取得してElementsコンポーネントを通して（おそらく裏でStripeが提供するコンポーネントがよしなにやってくれている）、フォームに渡すまでフォームがレンダリングされない時間が0.5秒くらいあってそれが少しストレスになる。ページを開いた瞬間に全てのDomがレンダリングされている状態であって欲しいので PaymentElementに *`onReady` を配置してフォームの準備が完了するまではローディングを表示するようにしている。完了次第 `displayStripeForm` に `false` をセットして表示を終了する。*

![PaymentIntentを取得](https://user-images.githubusercontent.com/23703281/164969016-6c03c10f-76fd-456f-a8c6-bfb21f921262.png)

下記のフォームコンポーネントが一番複雑になっている。実際のStripeに関する機能よりもモバイルに対応させたり細かいスタイルの調整をJSで行っているので複雑になっている。

![mobile_desktop_chekout](https://user-images.githubusercontent.com/23703281/164969033-1d66af6d-232f-4938-b0df-3686b5ff81c4.png)

モバイルでは `translateY` を使用してフォームを使用しない時は `フォームの高さpx - 50px` というように要素分画面の外に移動させている。

そこで端末幅ごとにフォームの高さに可変があるので要素の高さを `ResizeObserver` で監視して変更があればそれに合わせてフォームの高さも変更するようにしている。

フォームを表示する際も上向きのスワイプを検知した際にフォームが表示されるようになっている。下向きでしまう様な動作になっている。

この辺りでPC版に切り替わったりモバイルに切り替わる際に `addEventListner` を解除したりする処理等で苦労した。

Stripeに関する処理はほぼサンプルと変わらない。フォームでsubmitされると `handleSubmit` が動作して決済処理に問題がなければ `/donate/complete` にリダイレクトされる。

`checkoutform.tsx`

```tsx
import { useState, useEffect, useRef} from "react";
// stripeのコンポーネントを読み込んでる
import {
  PaymentElement,
  useStripe,
  useElements
} from "@stripe/react-stripe-js";
import { PaymentIntentResult } from '@stripe/stripe-js'

import { CheckOutFormProps } from '../../types/stripe/CheckOutForm'

import { prices } from '../../techBlogSettings/pricelist'

import { EventManager } from '../../lib/utility/eventManager'
import { sleep } from '../../lib/utility/sleep'

import { description } from "../../techBlogSettings/checkformdescription";
import { aboutblog } from "../../techBlogSettings/aboutblog";

// import { CardElementType } from './types/stripe'

export default function CheckOutForm({donate}: CheckOutFormProps) {
  const stripe = useStripe();
  const elements = useElements();

  const [message, setMessage] = useState<string | undefined | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [displayStripeForm, setDisplayStripeForm] = useState(true);
  const [startY, setStartY] = useState(0);
  const [endY, setEndY] = useState(0);
  const [toggle, setToggle] = useState(false);
  const [stripeFormWrapperHeight, setStripeFormWrapperHeight] = useState(0);
  const stripeFormWrapper = useRef<HTMLDivElement>(null);

  // フック化したいけどtailwindcssのコンパイル時にファイル内にクラス名がないとcssが出力されなくなる
  const donateInfo = ((donate) => {
    switch(donate) {
      case "1":
        return {
          color: "custom-orange",
          info: prices[0]
        }
      case "2":
        return {
          color: "custom-blue",
          info: prices[1]
        }
      case "3":
        return {
          color: "custom-yellow",
          info: prices[2]
        }
    }
  })(donate)

  const displayUpStripeForm = () => {
    const swipeUp = 0 < (endY - startY) && 50 < Math.abs((endY - startY))
      ? `translateY(${stripeFormWrapperHeight - 50}px)` : 'translateY(0px)';
    if (stripeFormWrapper.current) {
      stripeFormWrapper.current.style.transform = swipeUp
      const formStatus = swipeUp === 'translateY(0px)' ? true : false
      setToggle(formStatus)
    }
  }

  const displayForm = async () => {
    await sleep(3000)
    setDisplayStripeForm(false)
    const height = stripeFormWrapper.current.getBoundingClientRect().height
    console.log('初期のフォーム高さ', height)
    setStripeFormWrapperHeight(height)
    if (window.innerWidth < 1024) {
      stripeFormWrapper.current.style.transform = `translateY(${height - 50}px)`
    }
  }

  const observeStripeFormHeight = () => {
    // 登録した要素のサイズ変更を監視
    const observer = new ResizeObserver(() => {
      if (stripeFormWrapper.current) {
        const height = stripeFormWrapper.current.getBoundingClientRect().height
        setStripeFormWrapperHeight(height)
      }
    });

    if(stripeFormWrapper.current) {
      observer.observe(stripeFormWrapper.current);
    }
  }

  const observeStripeForm = (windowEventManager) => {
    windowEventManager.add('touchstart',(event) => {
      setStartY(event.touches[0].pageY)
    })
    windowEventManager.add('touchmove',(event) => {
      setEndY(event.touches[0].pageY)
    })
    observeStripeFormHeight()
  }

  const swipeDisplayUpStripeForm = (windowEventManager) => {
    windowEventManager.add('touchend',() => {
      displayUpStripeForm()
    })
  }

  useEffect(() => {
    const windowEventManager = new EventManager(window);
    const mqlPC = window.matchMedia('(min-width:1024px)')
    mqlPC.addEventListener('change', ev => {
      console.log('初期ローディング')
      // ここで一旦解除してレイアウトも戻す
      windowEventManager.removeAll('touchstart');
      windowEventManager.removeAll('touchmove');
      windowEventManager.removeAll('touchend');
      if (stripeFormWrapper.current) {
        stripeFormWrapper.current.style.transform = `translateY(0px)`
      }
      if (ev.matches) {
        // pc用何もしない
        setToggle(false)
        return
      }
      // pcサイズじゃない場合元に戻す
      observeStripeForm(windowEventManager);
      swipeDisplayUpStripeForm(windowEventManager);
      // if (stripeFormWrapper.current) {
      //   const height = stripeFormWrapper.current.getBoundingClientRect().height
      //   console.log('モバイル用', height)
      //   stripeFormWrapper.current.style.transform = `translateY(${height - 50}px)`
      // }
    })
    if (document.readyState === "complete") {
      if (window.innerWidth < 1024) {
        observeStripeForm(windowEventManager);
        swipeDisplayUpStripeForm(windowEventManager);
      }
    } else {
      if (window.innerWidth < 1024) {
        window.addEventListener('load', observeStripeForm);
        swipeDisplayUpStripeForm(windowEventManager);
      }
    }

    if (!stripe) {
      return;
    }

    //URLのクエリから値を取得する
    const clientSecret = new URLSearchParams(window.location.search).get(
      "payment_intent_client_secret"
    );

    // リダイレクト時にここに値が入る
    if (!clientSecret) {
      return;
    }

    stripe.retrievePaymentIntent(clientSecret).then(({ paymentIntent }: PaymentIntentResult) => {
      switch (paymentIntent?.status) {
        case "succeeded":
          setMessage("Payment succeeded!");
          break;
        case "processing":
          setMessage("Your payment is processing.");
          break;
        case "requires_payment_method":
          setMessage("Your payment was not successful, please try again.");
          break;
        default:
          setMessage("Something went wrong.");
          break;
      }
    });
  }, [,stripe, startY, endY]);

  const handleSubmit = async (e: { preventDefault: () => void; }) => {
    e.preventDefault();

    if (!stripe || !elements) {
      // Stripe.js has not yet loaded.
      // Make sure to disable form submission until Stripe.js has loaded.
      return;
    }

    setIsLoading(true);

    const { error } = await stripe.confirmPayment({
      elements,
      confirmParams: {
        // Make sure to change this to your payment completion page
        return_url: `${aboutblog.url}donate/complete`,
      },
    });

    // This point will only be reached if there is an immediate error when
    // confirming the payment. Otherwise, your customer will be redirected to
    // your `return_url`. For some payment methods like iDEAL, your customer will
    // be redirected to an intermediate site first to authorize the payment, then
    // redirected to the `return_url`.
    if (error.type === "card_error" || error.type === "validation_error") {
      setMessage(error.message);
    } else {
      setMessage("An unexpected error occured.");
    }

    setIsLoading(false);
  };

  return (
    <>
      {displayStripeForm && (
        <div className="fixed inset-0 w-full h-full bg-stripe z-30 flex justify-center items-center">
          <div className=" w-9/12 max-w-2xl bg-white rounded-3xl mx-auto p-8 cutom-box-shadow-black">
            <div className="spinner"></div>
            <span>決済情報を確認しています...</span>
          </div>
        </div>
      )}
      <div className={`${toggle ? '' : 'hidden'} w-full h-full bg-gray-darker opacity-50 fixed z-10 transition-opacity`}></div>
      <form id="payment-form" className="lg:flex lg:justify-between pt-7" onSubmit={handleSubmit}>
        <div className="lg:max-w-sm lg:m-0 w-11/12 mx-auto">
          <h1 className="text-2xl p-0 mb-4 font-normal">決済内容</h1>
          <div className="flex justify-between bg-white rounded-xl shadow-lg p-4">
            <div className={`w-12 h-12 ${donateInfo.color} rounded-md flex justify-center items-center`}></div>
            <div className="text-2xl pt-5">{`${donateInfo.info.amount}`}<small className="text-xs">円</small></div>
          </div>
          <div className="mt-8">
            <ul className="list-disc pl-5 mb-checkout-description">
              <li>{description.text1}</li>
              <li>{description.text2}{donateInfo.info.message}</li>
              <li>{description.text3}</li>
              <li>{description.text4}</li>
            </ul>
          </div>
        </div>
        <div className="bg-stripe lg:rounded-b-3xl rounded-3xl rounded-b-none px-5 pb-12 lg:max-w-sm z-20 lg:static fixed w-full bottom-0 transition-transform" ref={stripeFormWrapper}>
          <div className="h-14 w-11/12 mx-auto mb-6 mt-3 lg:invisible">
            <div className="h-1 w-4/12 bg-gray rounded-full mx-auto"></div>
          </div>
          <PaymentElement
            className="mb-16 text-white"
            onReady={() => {displayForm()}}
          />
          <button  className="bg-green w-full h-12 text-white font-medium rounded-md hover:shadow-lg" disabled={isLoading || !stripe || !elements} id="submit">
            <span id="button-text">
              {isLoading ? <div className="spinner-2" id="spinner"></div> : `${donateInfo.info.amount}円 支払う`}
            </span>
          </button>
          {/* Show any error or success messages */}
          {message && <div id="payment-message" className="text-red">{message}</div>}
        </div>
      </form>
    </>
  );
}
```

`options` でStripeフォームのレイアウトをカスタマイズする事が出来る。ここでNext.jsに立てたAPIへリクエストを投げてさらにそのAPIがStripeにリクエストを投げてPaymentIntent（決済に関する情報）を受け取って `clientSecret` としてElementsコンポーネントに渡している。これは裏でStripeのフォームを表示するのに関わっている。これが渡っていないとクレジット番号等を入力するフォームが表示されない。

`checkout.tsx`

```tsx
// next.js
import Head from 'next/head'
import { useRouter } from 'next/router';

// React
import { useState, useEffect} from "react";
// stripe
import { loadStripe, StripeElementsOptions} from "@stripe/stripe-js";
import { Elements } from "@stripe/react-stripe-js";

import CheckOutForm from '../../components/stripe/checkoutform'
import { donateTitle } from '../../components/layout'

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY as string)

export default function Donate() {
  const router = useRouter()
  const donate = router.query.donate
  const [clientSecret, setClientSecret] = useState('');

  useEffect(() => {
    console.log('値段', donate);
    (async () => {
      const res = await fetch('/api/create-payment-intent', {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ items: [{ id: donate }] })
      })
      if (!res.ok) {
        throw `payment-intentでエラー：${res.status}`
      }
      const data = await res.json()
      setClientSecret(data.clientSecret)
    })()
  }, [donate]);
  const options: StripeElementsOptions = {
    clientSecret,
    appearance: {
      theme: 'stripe',
      variables: {
        colorBackground: "#4864e8",
        colorText: '#fff',
        colorDanger: '#EC407A',
        spacingGridRow: "40px"
      },
      rules: {
        '.Input:focus': {
          border: '1px solid #2dd7e0',
        },
      }
    },
  };

  return (
    <main className="bg-gray-light min-h-screen" >
      <div className="lg:max-w-5xl lg:mx-auto">
        <Head>
          <title>{donateTitle}</title>
          <link rel="icon" href="/favicon/favicon.ico" />
        </Head>
        {clientSecret && (
          <Elements options={options} stripe={stripePromise} key={clientSecret}>
            <CheckOutForm donate={donate} />
          </Elements>
        )}
      </div>
    </main>
  )
}
```

### 決済完了ページ

決済処理に問題がないと完了ページにリダイレクトされ、紙吹雪の演出で感謝と喜びの舞を表現している。 紙吹雪には `react-rewards` というライブラリを使用している。

使用はとても簡単で紙吹雪を発車したい要素に `id="rewardId"` この様にidを振って `reward()` を実行すれば紙吹雪を降らしてくれる。実行する前に `reward()` はフック化されているので下記の様に変数として用意する必要がある。用意する際に第3引数に設定オブジェクト渡す事で紙吹雪の時間、広がり方、紙吹雪の枚数等の調整が可能になっている。

```tsx
const { reward, isAnimating } = useReward('rewardId', 'confetti', {
  lifetime: 300,
  spread: 90,
  elementCount: 100
});
```

個人的にはフォームでは角丸UIだったのに急にカクカクデザインで色合いもブログよりになってしまい、このデザインをあまり気に入っていない。かと言って考え直す元気もないので一旦はこれで行く。唯一色だけはめでたい雰囲気を作りたいのとお金に関わる部分なので黄色にした事だけ気に入っている。UIデザイナーがつくづくすごいなと思う。自分は3ページ考えるだけでヘトヘトなのに彼らはたくさんのページ、しかもたくさんの情報を綺麗に整列させているのでとても凄いと思う。

![決済完了](https://user-images.githubusercontent.com/23703281/164969069-773dd9a8-8788-431d-b997-fa72c5f12ce2.png)

`complete.tsx`

```tsx
// next.js
import Head from 'next/head'
import Link from 'next/link'
// React
import { useEffect } from "react";
// lib
import { useReward } from 'react-rewards';
import { donateTitle } from '../../components/layout'
import { sleep } from '../../lib/utility/sleep';

export default function Complete() {
  const { reward, isAnimating } = useReward('rewardId', 'confetti', {
    lifetime: 300,
    spread: 90,
    elementCount: 100
  });
  const wrapedSleep = async() => {
    await sleep(1500)
  }
  useEffect(() => {

    if (document.readyState === "complete") {
      wrapedSleep()
      reward()
    } else {
      window.addEventListener('load', () => {
        wrapedSleep()
        reward()
      });
    }

  }, [])

  return (
    <main className="bg-earth-lighter">
      <div className="lg:mx-auto max-w-2xl min-h-screen w-11/12 mx-auto pt-6">
        <Head>
          <title>{donateTitle}</title>
          <link rel="icon" href="/favicon/favicon.ico" />
        </Head>
        <div className="border-2">
          <h1 className="text-2xl font-normal pl-3">
            <button onClick={reward} disabled={isAnimating}>決済完了</button>
          </h1>
        </div>
        <div className='text-center'>
          <div className="text-2xl font-normal pt-16 leading-10">
            募金ありが
            <span id="rewardId">
              と
            </span>
            うございました。<br/>お金は<span className="border-yellow border-b-4 pb-1 ">開発のために</span>とても大切に使わせて頂きます。
          </div>
        </div>
        <div className="flex justify-end">
          <div className="bg-yellow w-36 h-12 mt-16 py-3 pl-3">
            <Link href='/'>Homeに戻る →</Link>
          </div>
        </div>
      </div>
    </main>
  )
}
```

最後に「開発者への寄付」リンクをメニューに追加して完成とする。

![開発者への寄付](https://user-images.githubusercontent.com/23703281/164969086-f0ebf98c-e506-42eb-a0ef-d4f8b488abe8.png)

`menu.tsx`

```tsx
// 一部抜粋

<div className='flex flex-col text-blue-dark mt-10 mb-10'>
  <Link href='/donate/price'>
    <a className='inline-block w-full py-2 font-bold'>
      <span className="tiny-pl"><FaRegCreditCard /></span>
      <span className="pl-1">開発者へ寄付</span>
    </a>
  </Link>
  <Link href='/'>
    <a className='inline-flex w-full py-2 font-bold items-center'>
      <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
        <path d="M10.707 2.293a1 1 0 00-1.414 0l-7 7a1 1 0 001.414 1.414L4 10.414V17a1 1 0 001 1h2a1 1 0 001-1v-2a1 1 0 011-1h2a1 1 0 011 1v2a1 1 0 001 1h2a1 1 0 001-1v-6.586l.293.293a1 1 0 001.414-1.414l-7-7z" />
      </svg>
      Home
    </a>
  </Link>
</div>
```

## 最後に

やろうやろうと思いながら結構、時間が掛かってしまった。特にTS化する辺りでライブラリに提供されている型の当て方が全然ぴんと来なかった。解説記事、GitHubのコード検索にもどのように型を当てられているかみたいなサンプルを見つけられなくてとても時間が掛かってしまった。

根本は自分で型ファイルを作成して、それを `import` して使うのと変わらなくて、誰かが作成した型を当てれば良い。しかしこの部分にどの型を当てればいい等具体的な説明がないからそれを探すのがとても大変だった。この辺り皆さんがどのようにしているか教えて頂けると幸いです。

次はいよいよ大きいフレームワークを使ってログイン機能を兼ね備えたサービス作りを行いたい。
データベース等の知識を必要になってくるのでとても楽しみである。

> 予定（Django 使用してデータベースを使った EC サイトを作成したい）

となっているが業務でRailsを使用しているのとやはり解説記事の多さもRailsが多いので一旦Rails（APIモード）・RSpecとNext.jsを使ってユーザ間で回答者に謝礼としてお金を渡せる質問サイトを構築できたら最高だと思う。あとDjangoみたいにDBを操作する管理画面が用意されてないのが個人的にいいなと思う。ブログを作成して約一年が経つのですが、その間にエンジニアとしての職を得てブログの開発を細々と行えているのでとても嬉しいなと思います。この調子で息をするようにコードが書けるエンジニアに少しでも近づけたらなと思います。
最後まで読んで頂きありがとうございました。またサービスを作る過程を次の記事として直ぐに投稿できたらと思います。

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。