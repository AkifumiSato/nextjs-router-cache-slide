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
layout: message
---

今日話したいこと:

Router Cacheは複雑

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
- Reactコアチームと協業して開発
  - Server first
  - より積極的なキャッシュ戦略
  - Nesting Layout

---
layout: sub-section
breadcrumb: Next.js(App Router)
---

# App Routerのキャッシュ

App Routerにはいくつかのキャッシュ層が存在する

https://github.com/vercel/next.js/blob/05b2730f55371a7fb16b9afff6d01ad7010a897d/docs/02-app/01-building-your-application/04-caching/index.mdx

---

## Overview

| Mechanism                             | What is cached?                | Where is it cached? |  Duration                        |
| ------------------------------------- | ------------------------------ | ------------------- |  ------------------------------- |
| React Cache           | Return values of functions     | Server              |  Per-request lifecycle           |
| Data Cache             | Return values of data requests | Server              |  Persistent (can be revalidated) |
| Full Route Cache | Rendered HTML and RSC payload  | Server              |  Persistent (can be revalidated) |
| Router Cache         | Route Segments (RSC Payload)   | Client              |  User session or time-based.     |

---
layout: sub-section
breadcrumb: Next.js(App Router)
---

# なぜRouter Cacheに着目したのか

App Routerの仕組みを研究してみようと思ったら、Router Cacheが複雑なことに気づいた

- [Next.js App Router 遷移の仕組みと実装
](https://zenn.dev/akfm/articles/next-app-router-navigation)
  - 遷移の仕組みを調べてた
  - ここでRouter Cacheの複雑さに気づくが複雑すぎて理解を諦める
- [Next.js App Router 知られざるClient-side Cacheの仕様
](https://zenn.dev/akfm/articles/next-app-router-client-cache)
  - 頑張って調べてまとめたた

---

<Title>App Router Navigation</Title>

---
layout: sub-section
breadcrumb: App Router Navigation
---

# App Routerの遷移を理解する

Router Cacheの話題に入る前に、App Routerの遷移を理解する必要がある

- App Routerでは、積極的にprefetchを行い、結果はcacheとして格納される(**Router Cache**)
- Router Cacheという名称ではあるが、内部的には遷移時に必ず必要となる
- 必要なcacheが見つからない場合、即座にfetchを行いRouter Cacheを更新する

---
layout: sub-section
breadcrumb: App Router Navigation
---

# Demo

<div class="flex justify-center">
  <img src="/assets/prefetch_demo.png" class="h-100">
</div>


---
layout: sub-section
breadcrumb: App Router Navigation
---

# Navigationのポイント

- 遷移にRouter Cacheは必ず必要で、なければ遷移時にfetchしてcacheに格納する
- App Routerは積極的に静的化(static rendering)とprefetchを行う
- prefetchは`Link`コンポーネントの`prefetch`propsで制御でき、`undefined`,`true`,`false`で挙動が異なる
- Router Cacheは一般的なcache同様、expireされるまで再利用される

---

<Title>Router Cacheの複雑な挙動</Title>

---
layout: sub-section
breadcrumb: Router Cacheの複雑な挙動
---

# cacheのexpireが複雑

Router Cacheのexpireは`prefetch={}`・取得時間・利用時間によって複雑に分岐する

| 時間判定 / cacheの種類               | `auto` | `full` | `temporary` |
| ----------------------- | ------- | ---- | ---- |
| prefetch/fetchから**30秒以内**                  | `fresh`    | `fresh` | `fresh` |
| lastUsedから**30秒以内**     | `reusable` | `reusable` | `reusable` |
| prefetch/fetchから**30秒~5分**                  | `stale`    | `reusable` | `expired` |
| prefetch/fetchから**5分以降**                  | `expired`    | `expired` | `expired` |

---
layout: sub-section
breadcrumb: Router Cacheの複雑な挙動
---

# Cacheの状態ごとの挙動

- `fresh`, `reusable`: prefetch/fetchを再発行せず、cacheを再利用する
- `stale`: Dynamic Rendering部分だけ遷移時に再fetchを行う
- `expired`: prefetchh/fetchを再発行する

---
layout: sub-section
breadcrumb: Router Cacheの複雑な挙動
---

# cacheのpurgeが複雑

cacheを無効化する手段が複雑

- `router.refresh()`で全てのcacheを無効化することは可能
- 安定版の機能では、個別のcacheをpurgeする手段はない
- 将来的にはalpha機能のServer Actions＋`revalidatePath`/`revalidateTag`で個別のcacheをpurgeできるようになりそう
  - 現状はこれらを利用すると全てのcacheがpurgeされる
  - ちなみに`cookies.set`/`cookies.delete`でも全てのcacheはpurgeされる

---
layout: sub-section
breadcrumb: Router Cacheの複雑な挙動
---

# cacheがあるときにIntercepting routesがバグってる

Router CacheとIntercepting routesの組み合わせが設計からして相性が悪い

https://github.com/vercel/next.js/issues/52748

- Intercepting routesは`Next-Url`ヘッダー（GETパラメータなどを省いたもの）に基づいて判定される
- Router Cacheは現状`Next-Url`の考慮がなされてないため、Intercepting routes時にも意図せずcache hitしてしまう
- 安易に治すと、遷移ごとにprefetchしないといけなくてこれまでの何倍ものprefetchが行われてしまう
  - しかし現状、積極的すぎたprefetchは減らす方向にある模様（？）

---

<Title>App Routerのいいところ</Title>

---
layout: sub-section
breadcrumb: App Routerのいいところ
---

# App Routerには夢がある

- Server Components
  - より直感的なデータ取得とレンダリング
  - 一部ながらも`typeof window`分岐とさよなら
  - ファイルサイズやrenderingコストが低減される
- Nested Layout
  - URL仕様次第ではあるが、うまく使えば重複コードを減らせる
  - 遷移時に失われてたstateも保持できる
- Server Actions
  - RPC的な体験をライブラリなしで得られる

---
layout: sub-section
breadcrumb: App Routerのいいところ
---

# Next.jsはApp Routerに全力

- Pages Routerは若干メンテナンスモード気味
- 新機能はApp Routerばかり
- Reactコアチームも連携してApp Routerの開発は進んでるため、頓挫する可能性はかなり低そう

---

<Title>App RouterとRouter Cacheまとめ</Title>

---
layout: sub-section
breadcrumb: App RouterとRouter Cacheまとめ
---

# App RouterとRouter Cache

- App Routerは今後の主軸なのは間違いない
- Router Cacheは複雑かつまだ不安定気味
- 悩んだら以下の記事を読むと参考になるかも

https://zenn.dev/akfm/
