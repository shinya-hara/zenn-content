---
title: "【2025年3月】styled-componentsがメンテナンスモードになったので、改めてReactでのCSSの実装方法を整理してみる"
emoji: "🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["styledcomponents", "cssinjs", "tailwindcss", "react", "nextjs"]
published: false
---

## 📝 はじめに

2025 年 3 月、長年 React 界隈を支えてきた CSS-in-JS ライブラリ「[styled-components](https://styled-components.com/)」がメンテナンスモードに移行したというニュースが流れてきました。

https://opencollective.com/styled-components/updates/thank-you

自分も含め、多くの React 開発者にとっては当たり前のように使っていた存在だけに、ちょっとした時代の変化を感じますね…😢

ということで今回は、styled-components に感謝しつつ、改めて「React で CSS を書く方法」について整理してみようと思います 🧵  
これから CSS の構成を見直す人や、他の選択肢を探してる人の参考になればうれしいです！

:::details 余談：時代の移り変わりを感じるニュースたち

### 🚫 Create React App（CRA）が非推奨に

React の公式ドキュメントからも案内があり、Create React App は **新規プロジェクトに使うことが推奨されなくなりました**。

https://zenn.dev/sonicmoov/articles/deprecated-create-react-app

### 📦 Recoil がアーカイブ化

Facebook（Meta）製の状態管理ライブラリ「Recoil」も、2024 年 12 月に GitHub 上で**アーカイブされました**。

https://zenn.dev/mk668a/articles/88685dfa915474
:::

## 💅 styled-components とは何者だったのか

styled-components は 2016 年ごろに登場した、CSS-in-JS の先駆け的な存在です。

```tsx
const Title = styled.h1`
  font-size: 1.5em;
  text-align: center;
  color: #bf4f74;
`;
```

↑ この書き方、めちゃくちゃ見覚えありますよね？笑

React コンポーネントとスタイルを密結合できることで、次のようなメリットがありました。

- スタイルがコンポーネントに閉じ込められる（スコープの心配が減る）
- props でスタイルを切り替えられる
- ThemeProvider でテーマ管理がしやすい

「CSS ってもっと柔軟に書けるんだ！」と、多くの React エンジニアに衝撃を与えたライブラリでした ✨

## 💀 なぜメンテナンスモードに？

2025 年 3 月時点で、styled-components は「メンテナンスモード」に移行しています。  
記事によるとその背景にはいくつかの要因があるとのことです。

- **React の方向性とのミスマッチ**  
  React コアチームが、Context API など一部の旧来 API を非推奨の方向に動かしており（特に RSC ＝ React Server Components との互換性問題）、styled-components のような仕組みとの相性が悪くなっています。

- **CSS-in-JS という考え方の衰退**  
  エコシステム全体として、CSS-in-JS から距離を取る傾向が強まっており、代わりに Tailwind CSS のようなユーティリティファーストのアプローチが主流になってきました。

- **メンテナの利用機会の減少**  
  現在のコアメンテナである quantizor 氏自身が、styled-components を使った大規模アプリの開発から離れており、実運用の中でメンテナンスを続けることが難しくなっているとのことです。

styled-components 自体は今でも動きますし、すぐに使えなくなるわけではありませんが、「今後の新機能追加や積極的な改善は行われない」というスタンスになった、というのが今回のアナウンスの趣旨です 📢

## 🧩 React で CSS を書く方法いろいろ（2025 年版）

ここからは、styled-components 以外で React にスタイルを当てる方法をざっと紹介していきます 💡

### 1. 🧾 Inline Style

React の JSX に直接スタイルをオブジェクトで書く、最もシンプルな方法です。

```tsx
<div style={{ color: "red", padding: "10px" }}>Inline</div>
```

- サクッと書ける
- JavaScript の変数やロジックをそのまま使える
- ただし `:hover` や メディアクエリ などは使えない

開発初期や小さなコンポーネントではアリですが、拡張性は低めです。

### 2. 💼 CSS Modules

`.module.css` や `.module.scss` などでスタイルをファイル分離しつつ、クラス名のスコープをローカルに保てる仕組みです。

```tsx
import styles from "./App.module.css";

<div className={styles.container}>Hello</div>;
```

- グローバル汚染を防げる
- 特別なランタイム不要で軽量
- 素朴だけど安定感のある選択肢

### 3. 🎨 Tailwind CSS

https://tailwindcss.com/

ユーティリティファーストな CSS フレームワーク。クラス名でデザインを構築するスタイルです。

```tsx
<div className="bg-blue-500 text-white p-4 rounded">Hello</div>
```

- デザインルールを統一しやすい
- 開発速度が上がる
- JSX がクラス名だらけになることもあるが、慣れれば 🙆‍♂️
- 周辺ライブラリが充実している

一度ハマると戻れない中毒性があります 😇

### 4. 👩‍🎤 Emotion

https://emotion.sh/docs/introduction

styled-components に似た CSS-in-JS ライブラリで、柔軟性が高いのが特徴です。

```tsx
import { css } from "@emotion/react";

const style = css`
  color: hotpink;
`;

<div css={style}>Hello</div>;
```

- styled と css 両方の API が使える
- TypeScript との相性も良好
- 条件分岐やテーマなどの柔軟な制御が得意

### 5. 🐼 Panda CSS

https://panda-css.com/

CSS-in-JS の新時代を担うことを目指す、新進気鋭の CSS-in-JS フレームワークです。Chakra UI チームが開発しています。

- トークンベースでデザイン管理ができる
- クラス名ベースの記法で開発速度も 🙆‍♂️
- TypeScript との統合が深く、補完・バリデーションが心地よい
- RSC サポート 🙆‍♂️

Tailwind のようなユーティリティの良さと、デザイントークン管理の強さを両立したい人におすすめです 💡

### 6. 🍦 Vanilla Extract

https://vanilla-extract.style/

「TypeScript で CSS を書く」新しいタイプの Zero-runtime CSS-in-TS ライブラリです。

```tsx
import { style } from "@vanilla-extract/css";

export const heading = style({
  fontSize: "2rem",
  color: "blue",
});
```

- CSS をビルド時に抽出 → ランタイムコストなし
- 型安全、パフォーマンスも高い
- 若干学習コストはあるが、大規模プロダクトに強い

デザインシステムとの相性も抜群です 👀

## ✍️ おわりに

styled-components がメンテナンスモードになったのはちょっと寂しいですが、React と CSS は今も進化し続けています 🌀

どんな手法にも向き・不向きがあり、正解はプロジェクトやチームによって異なります（というか正解はないかも・・・？）。だからこそ、今このタイミングで自分たちに合った CSS の書き方を見直してみるのもアリかもです！

他のおすすめ CSS ライブラリがあれば、ぜひコメントで教えてください〜！
