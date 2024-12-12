---
title: "OrvalでAPI仕様書からコード自動生成！スキーマ駆動開発を加速させよう"
emoji: "🐟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["orval", "openapi", "msw", "スキーマ駆動開発"]
published: false
publication_name: "sonicmoov"
---

:::message
この記事は [クラウドワークス グループ Advent Calendar 2024](https://qiita.com/advent-calendar/2024/crowdworks) シリーズ2の15日目の記事です。
:::

## はじめに

ソニックムーブでフロントエンドエンジニアをしている原です！

最近は Amplify Gen2 周りのキャッチアップをしていて、バックエンド側もちょこちょこ触っています 😉

今回は API 仕様書からコードを自動生成できる Orval というツールをご紹介します！
（おーゔぁる？おるゔぁる？読み方はわかってません 😂）

:::message
記事中のコードは 👇 のリポジトリにあるのであわせてご確認ください
:::
https://github.com/shinya-hara/orval-example/tree/main

## Orval とは？

Orval は OpenAPI 仕様で記述された API 仕様書をもとに、型安全な TypeScript コードを生成することができるライブラリです

仕様書とにらめっこして定型の実装を繰り返す・・・といったことがある程度回避できる素晴らしいツールです！

https://orval.dev/

## 導入準備

### プロジェクト作成

適当なディレクトリで `npm init` します

```bash
npm init -y
```

`src` ディレクトリも作っちゃいましょう！

```bash
mkdir src
```

```:現時点でのディレクトリ構成
.
├── src
└── package.json
```

### Orval のインストール

```bash
npm i -D orval
```

```:現時点でのディレクトリ構成
.
├── node_modules
├── src
├── package-lock.json
└── package.json
```

### API 仕様書の準備

OpenAPI 仕様の API 仕様書を準備しましょう！

弊社の場合はバックエンドエンジニア or フロントエンドエンジニアが [Stoplight](https://stoplight.io/) や [Apidog](https://apidog.com/jp/) を使って作成しています 📝

今回は有名な [Petstore の API 仕様書](https://petstore3.swagger.io/)をお借りします

https://petstore3.swagger.io/

API 仕様書を `src/api/docs/openapi.json` に配置します！

```:現時点でのディレクトリ構成
.
├── node_modules
├── src
│   └── api
│       └── docs
│           └── openapi.json    👈 DLしたJSONを配置
├── package-lock.json
└── package.json
```

### 設定ファイルの準備

Orval の設定ファイルである `orval.config.ts` を作成して、 `src/api/orval.config.ts` に配置します！

```ts:orval.config.ts
import { defineConfig } from "orval";

/**
 * Orval Configuration
 * @see https://orval.dev/reference/configuration/overview
 */
export default defineConfig({
  petstore: {
    input: {
      target: "./docs/openapi.json",
    },
    output: {
      mode: "split",
      target: "./generated/api.ts",
      schemas: "./generated/model",
      client: "axios-functions",
      mock: true,
      clean: true,
    },
  },
});
```

設定内容は一例なので詳細は公式ドキュメントを確認してください 👀

https://orval.dev/reference/configuration/overview

```:現時点でのディレクトリ構成
.
├── node_modules
├── src
│   └── api
│       ├── docs
│       │   └── openapi.json
│       └── orval.config.ts    👈 作成したconfigを配置
├── package-lock.json
└── package.json
```

## いざ、コード生成！

Orval 実行用の npm scripts を設定しておきましょう

```json:package.json
{
  "scripts": {
    "gen:api": "orval --config ./src/api/orval.config.ts"
  }
}
```

コード生成を試してみます 🚀

```bash
npm run gen:api
```

たくさんのファイルが生成されましたか？ 🫣
`orval` コマンドがエラーになる場合は設定を見直してみてくださいね

```:現時点でのディレクトリ構成
.
├── src
│   └── api
│       ├── docs
│       │   └── openapi.json
│       ├── generated    👈 自動生成結果が格納されているはず・・・！
│       │   ├── model
│       │   │   ├── address.ts
│       │   │   ├── apiResponse.ts
│       │   │   ├── category.ts
│       │   │   ├── customer.ts
│       │   │   ├── findPetsByStatusParams.ts
│       │   │   ├── findPetsByStatusStatus.ts
│       │   │   ├── findPetsByTagsParams.ts
│       │   │   ├── getInventory200.ts
│       │   │   ├── index.ts
│       │   │   ├── loginUserParams.ts
│       │   │   ├── order.ts
│       │   │   ├── orderStatus.ts
│       │   │   ├── pet.ts
│       │   │   ├── petBody.ts
│       │   │   ├── petStatus.ts
│       │   │   ├── tag.ts
│       │   │   ├── updatePetWithFormParams.ts
│       │   │   ├── uploadFileParams.ts
│       │   │   ├── user.ts
│       │   │   └── userArrayBody.ts
│       │   ├── api.msw.ts
│       │   └── api.ts
│       └── orval.config.ts
├── package-lock.json
└── package.json
```

## 生成されたコードを見てみよう

### `api.ts`

https://github.com/shinya-hara/orval-example/blob/main/src/api/generated/api.ts

このファイルには、実際に API エンドポイントにリクエストするための関数が生成されています

例えば「ペット ID をもとにペット情報を取得する」リクエストは以下のような関数が生成されていますね 👏

```ts:api.ts
/**
 * Returns a single pet
 * @summary Find pet by ID
 */
export const getPetById = <TData = AxiosResponse<Pet>>(
  petId: number,
  options?: AxiosRequestConfig
): Promise<TData> => {
  return axios.get(`/pet/${petId}`, options);
};
```

Orval が生成する関数には API 仕様書をもとに適切な型が適用されている点も嬉しいです！

今回で言うと `getPetById()` の返り値には `Pet` の型が含まれていることがわかります

型安全に実装していく上で、このあたりが自動生成コードでカバーされているのはとてもありがたいです 🥰

### `model/pet.ts`

先ほどでてきた `Pet` モデルの型情報はこのファイルに自動生成されていますね 🐐

https://github.com/shinya-hara/orval-example/blob/main/src/api/generated/model/pet.ts

`model/*` にはリクエストボディやレスポンスなどの型情報がずらっと生成されています

### `api.msw.ts`

このファイルには、[Mock Service Worker](https://mswjs.io/) で使用できるハンドラーの定義が生成されています

こちらは `orval.config.ts` で `mock: true` にした場合のみ生成されます

弊社のフロントエンド開発は MSW を使用したローカル開発を行うことが多いので、ハンドラーの自動生成で実装効率 UP です 💪

https://github.com/shinya-hara/orval-example/blob/main/src/api/generated/api.msw.ts

`getPetById()` のモックレスポンスを生成する関数は次のようになっていて、内部では [Faker](https://fakerjs.dev/) が利用されています 🐼

一見すると難解ですが実際に読んでみると大したことはないので安心してください

```ts:api.msw.ts ※フォーマットしています
export const getGetPetByIdResponseMock = (
  overrideResponse: Partial<Pet> = {}
): Pet => ({
  category: faker.helpers.arrayElement([
    {
      id: faker.helpers.arrayElement([
        faker.number.int({ min: undefined, max: undefined }),
        undefined,
      ]),
      name: faker.helpers.arrayElement([faker.string.alpha(20), undefined]),
    },
    undefined,
  ]),
  id: faker.helpers.arrayElement([
    faker.number.int({ min: undefined, max: undefined }),
    undefined,
  ]),
  name: faker.string.alpha(20),
  photoUrls: Array.from(
    { length: faker.number.int({ min: 1, max: 10 }) },
    (_, i) => i + 1
  ).map(() => faker.string.alpha(20)),
  status: faker.helpers.arrayElement([
    faker.helpers.arrayElement(["available", "pending", "sold"] as const),
    undefined,
  ]),
  tags: faker.helpers.arrayElement([
    Array.from(
      { length: faker.number.int({ min: 1, max: 10 }) },
      (_, i) => i + 1
    ).map(() => ({
      id: faker.helpers.arrayElement([
        faker.number.int({ min: undefined, max: undefined }),
        undefined,
      ]),
      name: faker.helpers.arrayElement([faker.string.alpha(20), undefined]),
    })),
    undefined,
  ]),
  ...overrideResponse,
});
```

## Orval 導入のメリット

Orval 導入で得られるメリットはたくさんありますが、以下が大きいと感じています！

### 手動コーディングの削減で効率化

言うまでもありませんね 😉

単純作業は自動化して他のところに時間を使いましょう！

### 仕様変更への迅速な対応

API 仕様が変わっていたのにそれに気づかずフロント側で不具合が・・・みたいな経験はありませんか？

Orval を導入していれば API 周り全体に型が適用されているので、静的解析で検出できます！

TypeScript のメリットを最大限享受して、型安全な開発ライフを送りましょう 🎉

### チーム全体で統一された開発体験

コードは自動生成なので一貫性のあるコードになります

API クライアント周りは同じフォーマットのコードとなるので、チーム内でのコードスタイルや設計のばらつきを防げますね

また、基本的には Orval が生成したコードを使えば良いので、学習コストの低減も見込めます

## おわりに

Orval を活用することで、手作業を最小限に抑えながら、仕様書から型安全なコードを自動生成し、チーム全体の開発体験を向上させることができます

特に、仕様変更が頻繁に発生するプロジェクトではその真価を発揮します

API 仕様書を SSOT として活用することで、バックエンドとフロントエンドの連携がスムーズになり、チーム全体の生産性が劇的に向上するはずです ✨

「Orval いいね 👍」って思っていただけた方はぜひ導入してみてください！
