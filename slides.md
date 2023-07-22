---
theme: default
highlighter: shiki
colorSchema: light
fonts:
  mono: Fira Mono
lineNumbers: false
drawings:
  persist: true
title: Next.js(App Router) Router Cache
remoteAssets: false
---

# App Router's Router Cache

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
  - Server 1st
  - より積極的なキャッシュ戦略
  - Nesting Layout

---
layout: sub-section
breadcrumb: Next.js(App Router)
---

# App Routerのキャッシュ

https://github.com/vercel/next.js/blob/05b2730f55371a7fb16b9afff6d01ad7010a897d/docs/02-app/01-building-your-application/04-caching/index.mdx

4種類のキャッシュが存在する

---

## Overview

| Mechanism                             | What is cached?                | Where is it cached? |  Duration                        |
| ------------------------------------- | ------------------------------ | ------------------- |  ------------------------------- |
| React Cache           | Return values of functions     | Server              |  Per-request lifecycle           |
| Data Cache             | Return values of data requests | Server              |  Persistent (can be revalidated) |
| Full Routeroute-cache) | Rendered HTML and RSC payload  | Server              |  Persistent (can be revalidated) |
| Router Cache         | Route Segments (RSC Payload)   | Client              |  User session or time-based.     |

---
layout: message
---

今回は特に、Router Cacheがどう実現されているのかみてみましょう

---

<Title>App Router Navigation</Title>

---

memo: zennの内容をかいつまんで話してく

- Router Cacheを理解するには遷移とprefetchを理解する必要がある
- App RouterはLinkタグ内で積極的にprefetchを行う
