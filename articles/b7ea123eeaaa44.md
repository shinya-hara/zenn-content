---
title: "【2022年】Next.js + TypeScript + ESLint + Prettier の構成でサクッと環境構築する"
emoji: "🛠️"
type: "tech"
topics:
  - "nextjs"
  - "typescript"
  - "eslint"
  - "環境構築"
  - "prettier"
published: true
published_at: "2022-02-20 16:05"
---

今回は Next.js の環境構築をまとめていこうと思います。ミニマムな構成だと思うので、どのプロジェクトでも使いやすいのではないでしょうか？

ではサクッといきましょう！

## 最終構成

今回の環境構築で導入する主要なパッケージ群です。執筆時点でインストールされたバージョンも併記しておくので参考にしてください。
| パッケージ | バージョン |
| ---- | ---- |
| next | 12.1.0 |
| react | 17.0.2 |
| eslint | 8.9.0 |
| prettier | 2.5.1 |
| typescript | 4.5.5 |

## 前提

`node` と `yarn` の環境が必要です。ない場合は先にそちらのセットアップをしてください。

## create-next-app

Next.js を TypeScript でセットアップします。下記コマンド実行後、プロジェクト名を聞かれるので入力して続行してください。

```sh
yarn create next-app --typescript
```

これだけで最低限のセットアップが完了し、開発サーバーを起動できるはずです。お手軽ですね！

```sh
cd project-name
yarn dev
# => http://localhost:3000
```

コマンド実行後は以下のようなディレクトリ構成になっているはずです。

```sh
.
├── pages
│   ├── api
│   │   └── hello.ts
│   ├── _app.tsx
│   └── index.tsx
├── public
│   ├── favicon.ico
│   └── vercel.svg
├── styles
│   ├── Home.module.css
│   └── globals.css
├── .eslintrc.json
├── .gitignore
├── README.md
├── next-env.d.ts
├── next.config.js
├── package.json
├── tsconfig.json
└── yarn.lock
```

設定ファイル等以外のソース周りはまとまっていた方が見通しが良いので、`src` ディレクトリを作って移動します。

```sh
.
├── public
│   ├── favicon.ico
│   └── vercel.svg
├── src
│   ├── pages
│   │   ├── api
│   │   │   └── hello.ts
│   │   ├── _app.tsx
│   │   └── index.tsx
│   └── styles
│       ├── Home.module.css
│       └── globals.css
├── README.md
├── next-env.d.ts
├── next.config.js
├── package.json
├── tsconfig.json
└── yarn.lock
```

## Path alias の設定

モジュールの import で相対パスしか使えないと辛いので、パスエイリアスを設定しておきます。

```diff json:tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
-    "incremental": true
+    "incremental": true,
+    "baseUrl": "./",
+    "paths": {
+      "@/*": ["./src/*"]
+    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

これで以下のような import が可能になります。

```ts
// 相対パスだけだと辿るのが辛い...
import moduleA from "../../../../modules/A";

// 絶対パスなら可読性も上がる！
import moduleA from "@/modules/A";
```

## ESLint

静的解析が行われることでコード品質を一定以上に保つことができると考えています。

`create-next-app` の時点で ESLint の設定もされています。追加で設定したい項目があれば `.eslintrc.json` を編集するすることで対応できます。

import の 順番が揃っていると見通しが良いのでそのルールを入れておこうと思います。

```diff json:.eslintrc.json
{
-  "extends": "next/core-web-vitals"
+  "extends": [
+    "next/core-web-vitals",
+    "plugin:import/recommended",
+    "plugin:import/warnings"
+  ],
+  "rules": {
+    "import/order": [
+      "error",
+      {
+        "alphabetize": {
+          "order": "asc"
+        }
+      }
+    ]
+  }
}
```

ESLint の対象ファイルは `src` ディレクトリ以下で十分かと思うでの、 `npm-scripts` を修正しておきましょう。加えて自動修正用のコマンドも追加しておきます。

```diff json:package.json
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start",
-  "lint": "next lint"
+  "lint": "next lint --dir src",
+  "lint:fix": "yarn lint --fix"
},
```

## Prettier

コードフォーマットを手動でやるのは辛い上、チームメンバーとフォーマットの仕方が揃わず、差分が発生してしまった経験は誰しもあるのではないでしょうか？ そんな問題をできるだけ減らすため、コードフォーマッターである Prettier を導入しましょう。

まずは必要なパッケージのインストールをします。

```sh
yarn add -D prettier eslint-config-prettier
```

次に、ESLint と Prettier が干渉しないように、ESLint 側に設定を追加します。

```diff json:.eslintrc.json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:import/recommended",
-    "plugin:import/warnings"
+    "plugin:import/warnings",
+    "prettier"
  ],
  "rules": {
    "import/order": [
      "error",
      {
        "alphabetize": {
          "order": "asc"
        }
      }
    ]
  }
}
```

Prettier 用の設定ファイルを追加します。設定可能なオプションは [Prettier の公式サイト](https://prettier.io/docs/en/options.html) を確認してください。

```json:.prettierrc
{
  "trailingComma": "all",
  "tabWidth": 2,
  "semi": false,
  "singleQuote": true,
  "jsxSingleQuote": true,
  "printWidth": 100
}
```

最後に、フォーマット用の `npm-scripts` を追加します。

```diff json:package.json
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start",
  "lint": "next lint --dir src",
-  "lint:fix": "yarn lint --fix"
+  "lint:fix": "yarn lint --fix",
+  "format": "prettier --write --ignore-path .gitignore './**/*.{js,jsx,ts,tsx,json}'"
},
```

Prettier 周りの設定は以上です。

## VSCode で保存時に自動でフォーマットをかける

ファイルの保存時に自動でフォーマットが走るようにしておくと、フォーマット漏れがなく便利です。

VSCode の拡張機能が必要なので、追加してない方は追加してください。
https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint
https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode

次に、VSCode の設定ファイルを追加します。

```json:.vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

これでファイル保存時に ESLint と Prettier による修正が走ります。素晴らしいですね！ もうこの環境なしに開発できません 笑

:::message
自動フォーマットがうまく動作しない場合は、VSCode を再起動してください
:::

## 成果物

これで Next.js の環境構築は完了です。最終的に以下のようなディレクトリ構造になります。

```
.
├── .vscode
│   └── settings.json
├── public
│   ├── favicon.ico
│   └── vercel.svg
├── src
│   ├── pages
│   │   ├── api
│   │   │   └── hello.ts
│   │   ├── _app.tsx
│   │   └── index.tsx
│   └── styles
│       ├── Home.module.css
│       └── globals.css
├── .eslintrc.json
├── .gitignore
├── .prettierrc
├── README.md
├── next-env.d.ts
├── next.config.js
├── package.json
├── tsconfig.json
└── yarn.lock
```

:::details package.json

```json:package.json
{
  "name": "nextjs-starter-kit-ts",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint --dir src",
    "lint:fix": "yarn lint --fix",
    "format": "prettier --write --ignore-path .gitignore './**/*.{js,jsx,ts,tsx,json}'"
  },
  "dependencies": {
    "next": "12.1.0",
    "react": "17.0.2",
    "react-dom": "17.0.2"
  },
  "devDependencies": {
    "@types/node": "17.0.18",
    "@types/react": "17.0.39",
    "eslint": "8.9.0",
    "eslint-config-next": "12.1.0",
    "eslint-config-prettier": "^8.3.0",
    "prettier": "^2.5.1",
    "typescript": "4.5.5"
  }
}
```

:::

:::details tsconfig.json

```json:tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "baseUrl": "./",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

:::

:::details .eslintrc.json

```json:.eslintrc.json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:import/recommended",
    "plugin:import/warnings",
    "prettier"
  ],
  "rules": {
    "import/order": [
      "error",
      {
        "alphabetize": {
          "order": "asc"
        }
      }
    ]
  }
}
```

:::

:::details .prettierrc

```json:.prettierrc
{
  "trailingComma": "all",
  "tabWidth": 2,
  "semi": false,
  "singleQuote": true,
  "jsxSingleQuote": true,
  "printWidth": 100
}
```

:::

## おわりに

今回は Next.js のセットアップをまとめてみました。大体いつも似たような構成に落ち着くので、手順化しておくことで少しでも楽できたらと思います。

また、React は CSS の実装方法が多様ですが、今回はデフォルトで採用される CSS Modules をそのまま利用しています。CSS in JS を使う場合は加えて設定が必要になります。それはまた別記事にしたいと思います！
