---
theme: default
highlighter: shiki
colorSchema: dark
fonts:
  mono: Fira Mono
lineNumbers: false
drawings:
  persist: true
title: Next.js App Router's Router Cache
remoteAssets: false
---

# Next.js App Router's<br> Router Cache

App Routerのクライアントサイドキャッシュの複雑さ

---

# About

- name: 佐藤 昭文（Akifumi Sato）
  - twitter: akfm_sato
  - github: AkifumiSato
  - zenn.dev: akfm
  - Web Engineer
- Next.js
  - 仕事でもNext.js（Pages Router）のプロジェクトを担当
  - 自身のサイトなどもNext.js（App Router）
  - App Routerに強い興味あり

---
layout: message
---

今日話すこと:

Router Cacheの複雑さ

---

<Title>Next.js(App Router)</Title>

---
layout: sub-section
breadcrumb: Next.js(App Router)
---

# App Routerとは

https://nextjs.org/docs/app

> Appルーターは、Reactの最新機能を使用してアプリケーションを構築するための新しいパラダイムです。すでにNext.jsに慣れ親しんでいる方であれば、App Routerが既存のファイルシステムベースのルーターであるPages Routerの自然な進化形であることがわかるでしょう。

- Next.jsの新しいRouter
  - 従来のRouterは**Pages Router**と呼称
  - フレームワークとしてはほとんど別物レベル
- Server first
- 積極的なキャッシュ
- Nesting Layout

---
layout: sub-section
breadcrumb: Next.js(App Router)
---

# App Routerのキャッシュ

App Routerには[いくつかのキャッシュ層](https://nextjs.org/docs/app/building-your-application/caching)が存在する


<div class="flex justify-center">
  <img src="/assets/cache-layer.png" class="h-80 w-100">
</div>

---

## Overview

| Mechanism           | What                       | Where  | Purpose                                         | Duration                        |
|---------------------|----------------------------|--------|-------------------------------------------------|---------------------------------|
| Request Memoization | Return values of functions | Server | Re-use data in a React Component tree           | Per-request lifecycle           |
| Data Cache          | Data                       | Server | Store data across user requests and deployments | Persistent (can be revalidated) |
| Full Route Cache    | HTML and RSC payload       | Server | Reduce rendering cost and improve performance   | Persistent (can be revalidated) |
| Router Cache        | RSC Payload                | Client | Reduce server requests on navigation            | User session or time-based      |

---
layout: sub-section
breadcrumb: Next.js(App Router)
---

# なぜRouter Cacheに着目したのか

App Routerの仕組みを研究してみようと思ったら、Router Cacheが複雑なことに気づいた

1. App Routerの遷移の仕組みに詳しくなりたかった
2. [Next.js App Router 遷移の仕組みと実装
](https://zenn.dev/akfm/articles/next-app-router-navigation)
3. [Next.js App Router 知られざるClient-side Cacheの仕様
](https://zenn.dev/akfm/articles/next-app-router-client-cache)

---

<Title>App Router Navigation</Title>

---
layout: sub-section
breadcrumb: App Router Navigation
---

# App Routerの遷移を理解する

Router Cacheの話題に入る前に、App Routerの遷移を理解する必要がある

- App Routerでは、積極的にprefetchを行い、結果はcacheとして格納される
  - 内部的には`prefetchCache`と呼ばれているが、公には**Router Cache**と呼ばれている
  - cacheと呼ばれてるが、遷移時に必ず必要となる
- 必要なcacheが見つからない場合、即座にfetchを行いRouter Cacheを更新する

---
layout: sub-section
breadcrumb: App Router Navigation
---

# Demo

https://next-router-cache-demo.vercel.app/

<div class="flex justify-center">
  <img src="/assets/prefetch_demo.png" class="h-85">
</div>


---
layout: sub-section
breadcrumb: App Router Navigation
---

# Navigationのポイント

- App Routerのrenderingにはstatic rendering/dynamic renderingの2つがあり、基本はstatic rendering
- 積極的にprefetchを行い、static rendering部分は早期にcacheされる
- dynamic rendering部分やprefetchが無効なページについては遷移時にfetchされる

---

<Title>Router Cacheの複雑な挙動</Title>

---
layout: sub-section
breadcrumb: Router Cacheの複雑な挙動
---

# cacheの内部的な分類

Router Cacheは内部的な分類を持っており、挙動がそれぞれ異なる

| cacheの種類    | `Link`                 | `router`                                             |
|-------------|------------------------|------------------------------------------------------|
| `auto`      | `prefetch={undefined}` | `router.prefetch(path, { kind: PrefetchKind.AUTO })` | 
| `full`      | `prefetch={true}`      | `router.prefetch(path)`                              | 
| `temporary` | `prefetch={false}`     | -                                                    |

---
layout: sub-section
breadcrumb: Router Cacheの複雑な挙動
---

# cacheのexpireが複雑

Router Cacheのexpireは`prefetch`props・取得時間・利用時間によって分岐する

| 時間判定 / cacheの種類            | `auto`     | `full`     | `temporary` |
|----------------------------|------------|------------|-------------|
| prefetch/fetchから**30秒以内**  | `fresh`    | `fresh`    | `fresh`     |
| lastUsedから**30秒以内**        | `reusable` | `reusable` | `reusable`  |
| prefetch/fetchから**30秒~5分** | `stale`    | `reusable` | `expired`   |
| prefetch/fetchから**5分以降**   | `expired`  | `expired`  | `expired`   |

---
layout: sub-section
breadcrumb: Router Cacheの複雑な挙動
---

# Cacheの状態ごとの挙動

- `fresh`, `reusable`: prefetch/fetchを再発行せず、cacheを再利用する
- `stale`: Dynamic Rendering部分だけ遷移時に再fetchを行う
- `expired`: prefetch/fetchを再発行する

---
layout: sub-section
breadcrumb: Router Cacheの複雑な挙動
---

# Cacheのライフサイクルに影響する要素

- static rendering/dynamic rendering
- `Link`コンポーネントの`prefetch` props
- 取得からの時間
- 最後にcacheが利用された時間

---
layout: sub-section
breadcrumb: Router Cacheの複雑な挙動
---

# cacheのpurgeが複雑

cacheをpurgeする手段が複雑

- 以下の方法で全てのRouter Cacheのpurgeが可能
  - `router.refresh()`
  - Server Actions+`revalidatePath`/`revalidateTag`
  - Server Actions+`cookies.set`/`cookies.delete`
- 将来的にはServer Actions＋`revalidatePath`/`revalidateTag`は個別のcacheをpurgeできるようになりそう

https://github.com/vercel/next.js/discussions/54075

---
layout: sub-section
breadcrumb: Router Cacheの複雑な挙動
---

# cacheがあるとIntercepting routesが機能しない

- intercepting routesとは
  - https://next-intercepting-routes-demo.vercel.app/
- Router CacheとIntercepting routesの組み合わせが設計からして相性が悪い
  - https://github.com/vercel/next.js/issues/52748
  - Intercepting routesは`Next-Url`ヘッダー（GETパラメータなどを省いたもの）に基づいて判定される
  - Router Cacheは現状URLパスをkeyにしているため`Next-Url`が考慮されておらず、Intercepting routes時もcache hitしてしまう
  - `Next-Url`をキーに含めるようにすると、これまでの何倍ものprefetchが行われてしまうので難しい問題

---

<Title>App Routerのいいところ</Title>

---
layout: sub-section
breadcrumb: App Routerのいいところ
---

# App Routerでよりシンプルになったものもある

- Server Components
  - より直感的なデータ取得とレンダリング
  - 一部ながらも`typeof window`分岐とさよなら
- Nested Layout
  - URL仕様次第ではあるが、うまく使えば重複コードを減らせる
  - 遷移時に失われてたstateも保持できる
- Server Actions
  - RPC的な体験をライブラリなしで得られる

---
layout: sub-section
breadcrumb: App Routerのいいところ
---

# Pages Routerとの共存は暫定的な可能性

> We are committed to supporting pages/ development, including bug fixes, improvements, and security patches, for multiple major versions moving forward.

https://nextjs.org/blog/next-13-4#is-the-pages-router-going-away

- 今後複数majorバージョンにおいてPages Routerのサポートは継続される
- **その後のことは名言しておらず、Pages Routerがdeprecate・廃止される可能性はある**
- 現状すでにサポートするとはいいつつ、新機能はApp Routerばかり
  - Pages Routerはバグも放置されがち？
- 現時点ではApp RouterはNext.jsの将来のメインストリームになる可能性はとても高い

---

<Title>App RouterとRouter Cacheまとめ</Title>

---
layout: sub-section
breadcrumb: App RouterとRouter Cacheまとめ
---

# App RouterとRouter Cache

- App RouterはNext.jsの今後の主軸になる可能性はとても高い
- App Router、特にRouter Cacheは複雑かつまだ不安定気味

より詳しい話はzennにも記事で書いてるのでよければ読んでください。

https://zenn.dev/akfm/

---
layout: message
---

Thanks
