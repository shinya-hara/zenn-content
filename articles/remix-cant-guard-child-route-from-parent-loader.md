---
title: "【Remix】親ルートのloaderでバリデーションして、すべての子ルートを保護する方法はありません😅"
emoji: "💿"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["remix", "react"]
published: false
---

## はじめに

最近 Remix（の主に SPA モード）をキャッチアップ中です！

仕様を勘違いしていて、学びになったことがあったのでシェアです〜

## やりたいこと

基本的にログインが必須のサービスで、ログインしているユーザーには特定のページへのアクセスを許可したい

## 前提

仮で以下のようなルーティングを想定します

```
/       ... トップページ（ログイン必須）
/mypage ... マイページ（ログイン必須）
/signin ... ログインページ（ログイン不要）
```

## やったこと（間違った対応です 😅）

ログイン必須のページが今後増えることも考慮して、 [Nested Layout](https://remix.run/docs/en/main/file-conventions/routes#nested-urls-without-layout-nesting) を活用してログイン後にアクセス可能なルーティングをまとめていました

```
 app/
├── routes/
│   ├── _protected._index.tsx
│   ├── _protected.mypage.tsx
│   ├── _protected.tsx
│   └── signin.tsx
└── root.tsx
```

`_protected.tsx` のローダー実装は以下のような感じで、ログインセッションの確認をしています

```tsx:_protected.tsx
// SPAモードで試したので `clientLoader` になってます
export const clientLoader = async () => {
  try {
    // NOTE: 何らかのログイン状態を確認する処理（今回は session があればログイン済みと想定）
    const session = await getSession();
    return json(session);
  } catch {
    // ログインセッションが確認できなかったらログイン画面にリダイレクト
    return redirect('/signin');
  }
};

export default function ProtectedLayout() {
  return (
    <div>
      {/* 適当なUIがある想定 */}
      <Outlet />
    </div>
  )
}
```

子ルート側はログインセッションがある前提の実装をしていました（👉 ローダーでログイン済みかどうかを確認していない 🥶）

```tsx:_protected.mypage.tsx
export default function MyPage() {
  return (
    <div>
      {/* ログイン済みのユーザーにだけ表示したい内容 */}
    </div>
  )
}
```

## 正しくは？

各ルートでバリデーションを行いましょう！

Remix はアプリの高速化を目的に **すべてのローダーを並列で実行する** ようです 💨
そのため、**親ルートのみで一括でログインセッションをバリデーションすることはできません** 😅

> During a client-side transition, to make your app as speedy as possible, Remix will call all of your loaders in parallel, in separate fetch requests. Each one of them needs to have its own authentication check.
> https://remix.run/docs/en/main/guides/faq

早速、ログインセッションを確認する関数を実装しましょう！
関数化しておくことで各ルートでの実装がスッキリしますね 😉

```ts:session.ts
export async function requireUserSession(request) {
  // NOTE: 何らかのログイン状態を確認する処理（今回は session があればログイン済みと想定）
  const session = await getSession();

  // ログインセッションが確認できなかったらログイン画面にリダイレクト
  if (!session) {
    throw redirect("/signin");
  }

  return session;
}
```

実装した関数を子ルートのローダーで実行することで、ログインセッションの確認ができます

```tsx:_protected.mypage.tsx
export const clientLoader = async () => {
  const session = await requireUserSession();

  return json(session);
}
```

## おわりに

各ルートの `loader` （および `clientLoader`）は並列で実行されるため、親ルートのみでのバリデーションでは不十分ということがわかりました

仕様をあまり確認しないまま実装していたため、事前に気づけて良かったです 😉

バリデーションは各ルートで確実に実行しましょう

Remix 楽しいので今後も発見があればアウトプットしていきます〜
