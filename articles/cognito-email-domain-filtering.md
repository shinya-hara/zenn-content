---
title: "【Amplify Gen2 Auth】特定のメールドメインだけユーザー登録を許可したい"
emoji: "📧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - amplifygen2
  - cognito
  - lambda
  - aws
published: true
published_at: 2025-01-15 07:00
publication_name: "sonicmoov"
---

## はじめに

ソニックムーブでフロントエンドエンジニアをしている原です！

最近、Amplify Gen2 を使った開発をしているのですが、Cognito 周りで学びがあったのでアウトプットします

## やりたいこと

社内システムを開発していて、「**特定のメールドメインの場合のみ、ユーザー登録を許可したい**」という要件が発生しました

要件としては割と あるある ではないでしょうか？ 😉

## 実現方法

早速結論ですが、「**サインアップ前の Lambda トリガー**」を使います！

ユーザー登録の処理に割り込んで、メールドメインに応じたフィルタリング処理を行うイメージです 💪

https://docs.aws.amazon.com/ja_jp/cognito/latest/developerguide/user-pool-lambda-pre-sign-up.html

## 実装例

:::message
実装は Amplify Gen2 を想定したものですが、Lambda トリガー自体はこの環境でなくとも使用可能なので、Cognito を利用されている方は参考にしてください 👏
:::

Amplify Gen2 をセットアップするとデフォルトで以下のような `auth` が設定されていると思います（今回 `data` は不要なので除外してあります）

```
/amplify
├─ auth
│  └─ resource.ts
└─ backend.ts
```

```ts:amplify/auth/resource.ts
import { defineAuth } from '@aws-amplify/backend';

export const auth = defineAuth({
  loginWith: {
    email: true,
  },
});
```

```ts:amplify/backend.ts
import { defineBackend } from '@aws-amplify/backend';
import { auth } from './auth/resource';

const backend = defineBackend({
  auth,
});
```

### Lambda 関数を実装

ユーザーが登録しようとしたメールアドレスは `event.request.userAttributes['email']` に格納されているので、こちらに対して任意のバリデーションロジックを実装すれば OK です！

今回は正規表現で判定していて、許可ドメインをホワイトリストで指定するような想定ですね 🙆‍♂️

許可ドメインの値自体は Lambda の環境変数で受け取る想定で実装しています

```ts:amplify/auth/preSignUp/handler.ts
import type { PreSignUpTriggerHandler } from 'aws-lambda';
import { env } from '$amplify/env/preSignUp';

export const handler: PreSignUpTriggerHandler = async (event) => {
  const email = event.request.userAttributes['email'];
  const regex = new RegExp(`(${env.ALLOW_DOMAIN})$`);

  if (!regex.test(email)) {
    throw new Error('Invalid email domain');
  }

  return event;
};
```

Amplify Gen2 では `defineFunction()` で手軽に Lambda 関数が定義できるのでとっても楽です 😊

ここで許可ドメインを環境変数として注入しましょう（Amplify の環境変数経由でやればもっといい感じ）

```ts:amplify/auth/preSignUp/resource.ts
import { defineFunction } from '@aws-amplify/backend';

export const preSignUp = defineFunction({
  runtime: 22,
  environment: {
    /** NOTE: 許可ドメインをパイプ `|` で連結して指定（正規表現に利用する） */
    ALLOW_DOMAIN: 'example.co.jp|example.com',
  },
});
```

### 実装した Lambda をトリガーとして設定

なんとこれだけ！！

```diff ts:amplify/auth/resource.ts
  import { defineAuth } from '@aws-amplify/backend';
+ import { preSignUp } from './preSignUp/resource';

  export const auth = defineAuth({
    loginWith: {
      email: true,
    },
+   triggers: [preSignUp],
  });
```

### ドメインフィルタリングの完成！

実際に Amplify サンドボックスなどを起動して、ユーザー登録を試してみてください

Lambda 関数を作って `preSignUp` トリガーに設定しただけですが、これで許可ドメイン以外からのユーザー登録はブロックされるようになったはずです

思ったより簡単で拍子抜けしちゃいますね 🤣

## おわりに

今回は Lambda トリガーを使ったドメインフィルタリングを紹介しました

メールアドレスのドメインフィルタリングは、組織やサービスのセキュリティを強化し、登録ユーザーを管理する上で非常に効果的な方法です

Amplify Gen2 の柔軟なカスタマイズ機能を活用することで、ニーズに応じたフィルタリングロジックを簡単に組み込むことができます

この記事がシステム構築の参考になれば幸いです 🙌
