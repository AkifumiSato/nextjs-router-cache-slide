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

<div class="flex justify-center">
  <img src="/assets/prefetch_demo.png" class="h-100">
</div>

---

todo

- [ ] demoで色々確認

