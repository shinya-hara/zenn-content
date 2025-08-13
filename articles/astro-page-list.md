---
title: "Astroでサクッとページ一覧を実装する"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["astro", "vite"]
published: false
publication_name: "sonicmoov"
---

## はじめに

こんにちは、原です 🐐

Astro でコーディングしていると、「ページの一覧を表示したい」ということはないでしょうか？

手動でページリンクを管理するのは面倒だし、新しいページを追加するたびにいちいち更新するのも大変... 😅

ということで今回は Astro でページ一覧をサクッと実装する方法を紹介します！

## 前提

サンプルとして、以下のようなプロジェクト構成を想定しています

```
src/
└── pages/
    ├── features/
    │   ├── api.astro
    │   └── security.astro
    ├── about.astro
    ├── index.astro
    └── list.astro
```

`list.astro` にページ一覧を表示したいと思います

## 実装

コピペで動くはずです！🤣

ポイントは `import.meta.glob()` です！

```jsx:list.astro
---
function getPageInfo(path: string, module: { url: string }) {
  const url = module.url || "/";
  return { url, path };
}

const pages = Object.entries<{ url: string }>(
  import.meta.glob("../pages/**/*.astro", { eager: true })
).map(([path, module]) => getPageInfo(path, module));
---

<!doctype html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>ページ一覧 📚</title>
  </head>
  <body>
    <div>
      <h1>ページ一覧 📚</h1>
      <ul>
        {
          pages.map((page) => (
            <li>
              <a href={page.url}>{page.url}</a>
            </li>
          ))
        }
      </ul>
    </div>
  </body>
</html>
```

`/list` にアクセスすると以下のようなページが表示されるはずです！

![ページ一覧](/images/astro-page-list/list.png =400x)

## `import.meta.glob()` とは 🤔

`import.meta.glob()` は、Vite が提供する機能で、glob パターンでファイルを一括取得できる便利な関数です

https://ja.vite.dev/guide/features.html#glob-%E3%81%AE%E3%82%A4%E3%83%B3%E3%83%9B%E3%82%9A%E3%83%BC%E3%83%88

- `**` で任意の深さのディレクトリを指定
- `*`で任意のファイル名を指定
- 指定したパターンにマッチするファイルをまとめて取得

これを使えば、「pages ディレクトリ以下のすべての.astro ファイル」を簡単に取得できます！🎉

## ひと工夫 😉

ページのタイトルなどを表示したい場合、各ページ側で値を `export` しておけば参照できます！

```jsx:about.astro
---
export const title = "About"; // 👈 これを追加
---

<h1>About</h1>
```

:::details list.astro のコード

`title` のハンドリングを追加しています

```jsx:list.astro
---
function getPageInfo(path: string, module: { url: string; title?: string }) {
  const url = module.url || "/";
  return { url, path, title: module.title }; // 👈 title を追加
}

const pages = Object.entries<{ url: string; title?: string }>(
  import.meta.glob("../pages/**/*.astro", { eager: true })
).map(([path, module]) => getPageInfo(path, module));
---

<!doctype html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>ページ一覧 📚</title>
  </head>
  <body>
    <div>
      <h1>ページ一覧 📚</h1>
      <ul>
        {
          pages.map((page) => (
            <li>
              {/* 👇 title があればそれを表示、なければ url を表示 */}
              <a href={page.url}>{page.title || page.url}</a>
            </li>
          ))
        }
      </ul>
    </div>
  </body>
</html>
```

:::

以下のようなユースケースの実現に使えるでしょうか？🤔

- ページのタイトルを表示したい
- 画面 ID を表示したい
- 任意の順番でページを表示したい（ `order: number` などをもとにソートする）
- 特定のページは除外したい（ `exclude: boolean` などをもとにフィルターする）

## まとめ

`import.meta.glob()` を使えば、Astro でページ一覧を動的に生成することができました！

新しいページを追加しても自動で一覧に反映されるので、メンテナンスが楽になります ✨

ぜひ活用してみてください！
