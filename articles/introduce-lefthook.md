---
title: "Lefthookを使ってcommit前にコード品質をチェックしよう"
emoji: "🥊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["lefthook", "git", "環境構築"]
published: false
---

## こんなうっかりありませんか？

git commit したあとに「あ、コードフォーマットが漏れてた！」みたいな経験ありませんか？🙄

今回はそんなうっかりを回避するために、あるツールをつかって commit 前にチェックするようにしてみます！

## Lefthook

今回紹介するのはこちら

https://github.com/evilmartians/lefthook/tree/master

ロゴが思いっきり左フック 🥊

Git hooks を管理するツールで、類似ツールに husky や simple-git-hooks などがありますね

## セットアップ

### インストール

```sh
npm i -D lefthook
```

### 設定ファイル作成

プロジェクトルートに作成

```yml:lefthook.yml
pre-commit:
  commands:
    lint:
      # コマンドはいい感じに定義
      run: npm run lint {staged_files}
    # ほかにも実行したいコマンドあれば追記
```

`{staged_files}` はステージ上にあるファイルを表すシンタックスです

これを活用することで commit しようとするファイルだけを処理対象にできます 👏

husky を使う際はステージ上のファイルを検出するために lint-staged を使うのが主流でしたが、lefthook にはステージ上のファイルを検出する仕組みを内包しているので、追加のツールは必要ありません 😉

### 設定を反映

```sh
npx lefthook install
```

これでコミット前に指定したコマンドが実行されるようになりました 🎉

## ローカルの設定ファイルを使用する

`lefthook.yml` はプロジェクトの標準設定として定義しておきつつ、個人のローカル環境では設定を上書きしたいケースってありますよね？

そんなときは `lefthook-local.yml` を作成しましょう！

たとえば以下のような `lefthook.yml` が存在すると想定します

```yml:lefthook.yml
pre-commit:
  commands:
    format:
      run: npm run format {staged_files}
```

これはコミット前にフォーマットをチェックする処理ですが、エディタの機能などで自動フォーマットが効く環境であれば、pre-commit フックでの処理は不要かもしれません

そこでローカル設定でグローバル設定を上書きしてみます

```yml:lefthook-local.yml
pre-commit:
  commands:
    format:
      skip: true
```

`skip: true` を指定することで、特定のコマンドをスキップできます 😶‍🌫️

https://github.com/evilmartians/lefthook/blob/master/docs/configuration.md#skip

`lefthook-local.yml` は開発者別のローカル設定ファイルなので、 `.gitignore` に追加しておきましょう 🙉

```:.gitignore
lefthook-local.yml
```

## レシピ集

### 特定のファイルだけを検査対象にしたい

`glob` を使いましょう

https://github.com/evilmartians/lefthook/blob/master/docs/configuration.md#glob

```yml:lefthook.yml
pre-commit:
  commands:
    lint:
      glob: "*.{js,ts,jsx,tsx}"
      run: npm run lint {staged_files}
```

### コマンドで自動修正したファイルを含めてコミットしたい

`stage_fixed` を使いましょう

https://github.com/evilmartians/lefthook/blob/master/docs/configuration.md#stage_fixed

```yml:lefthook.yml
pre-commit:
  commands:
    format:
      glob: "*.{js,ts,jsx,tsx}"
      run: npm run format:fix {staged_files}
      stage_fiexed: true
```

## おわりに

今回は Lefthook を紹介しました！

設定が簡潔・簡単なので既存プロジェクトでも導入しやすいのでは無いでしょうか？

気になった方はぜひ導入してみてください 🥊
