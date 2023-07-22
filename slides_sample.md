---
theme: default
highlighter: shiki
colorSchema: light
fonts:
  mono: Fira Mono
lineNumbers: false
drawings:
  persist: true
title: Browser-back experience and Next.js
remoteAssets: false
---

# Browser-back experience <br>and Next.js

ブラウザバック体験で見るNext.js

---

# About

- name: Akifumi Sato
  - twitter: akfm_sato
  - github: AkifumiSato
  - zenn.dev: akfm
  - Front-end/Backend Engineer
- Next.js
  - 仕事でもNext.jsのプロジェクトを担当
  - 自身のサイトなどもNext.js

---
layout: message
---

今日話したいこと:

Next.jsとブラウザバックの話

---

<Title>SPAとブラウザバック</Title>

---
layout: sub-section
breadcrumb: SPAとブラウザバック
---

# ブラウザバック体験の重要性

ブラウザバックはユーザーにとって、最も重要なWebブラウジング体験の1つ。

- 元ネタ: [リッチなWebアプリケーションのための7つの原則](https://yosuke-furukawa.hatenablog.com/entry/2014/11/14/141415)
  - 原著: [7 Principles of Rich Web Applications](https://rauchg.com/2014/7-principles-of-rich-web-applications)
  - rauch氏による過去のブログ
- **Don't break history, enhance it**
  - 履歴は壊すべきじゃない
  - 「ユーザーは"戻る"事によってデータが変更されるような事は期待していない。」
  - スクロール位置や状態はhistoryに関連づけられ、復元されることが望ましい

---
layout: sub-section
breadcrumb: SPAとブラウザバック
---

# ブラウザバック体験の重要性

しかし、こういった原則を重視してるサイトは少ない...

- 多くのサイトではスクロール位置や状態が失われる
- ユーザーからの不平不満も存在する
  - https://rentwi.hyuki.net/?1576010373357965312

---

<Title>Next.jsとブラウザバック体験</Title>

---
layout: sub-section
breadcrumb: Next.jsとブラウザバック体験
---

# ブラウザバック体験における重要な機能

前述のブラウザバック体験における重要な2つの機能ついて、Next.jsがどうなってるか見てみる

- **スクロール位置の復元**
- **状態の復元**

---
layout: sub-section
breadcrumb: Next.jsとブラウザバック体験
---

# Next.jsのスクロール復元

experimental(実験的機能)で提供されてるappディレクトリはおそらく未対応？

```ts
// next.config.js
const nextConfig = {
  // ...
  experimental: {
    scrollRestoration: true,
  },
}

module.exports = nextConfig
```

---
layout: sub-section
breadcrumb: Next.jsとブラウザバック体験
---

# Next.jsのスクロール復元

余談: リロード時にNext.js内部の履歴がリセットされるバグを修正した

https://github.com/vercel/next.js/pull/36861

<div class="flex justify-center">
  <img src="/assets/nextjs-pr.png" class="h-60">
</div>

---
layout: sub-section
breadcrumb: Next.jsとブラウザバック体験
---

# Next.jsのスクロール復元

詳細な解説はzennにまとめてあるので、そちらをご参照ください。

https://zenn.dev/akfm/articles/next-js-scroll-restore

<div class="flex justify-center">
  <img src="/assets/zenn-scroll.png" class="h-60">
</div>

---
layout: sub-section
breadcrumb: Next.jsとブラウザバック体験
---

# Next.jsの状態復元

残念ながら、公式に履歴に紐づけて状態を保存する手段はまだない...

<v-click>

**なので、復元するライブラリを作った！（作るのを手伝った）**

[recoil-sync-next](https://github.com/recruit-tech/recoil-sync-next)

```tsx{|4-7}
export const counter = atom<number>({
  key: 'counterState',
  default: 0,
  effects: [
    syncEffect({
      storeKey: 'ui-state', // 任意のkeyに紐づけて保存できる
      refine: number(),
    }
  )],
})
```

</v-click>

---
layout: sub-section
breadcrumb: Next.jsとブラウザバック体験
---

# Next.jsの状態復元

これも、詳細な解説はzennにまとめてあるのでそちらをご参照ください。

https://zenn.dev/akfm/articles/recoi-sync-next

<div class="flex justify-center">
  <img src="/assets/zenn-history.png" class="h-60">
</div>

---

<Title>今後Next.jsに求めたいこと</Title>

---
layout: sub-section
breadcrumb: 今後Next.jsに求めたいこと
---

# ブラウザバック体験から、Next.jsにもとめたいこと

まず`scrollRestoration`があることに感謝

- Beta機能の`app`ディレクトリでも、`scrollRestoration`対応してくれると嬉しい
- 公式に履歴に紐づく状態管理（`useNextState`...?）ができるようになると嬉しい
