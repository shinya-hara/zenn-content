---
title: "Figma公式の Dev Mode MCP サーバーが来た！"
emoji: "🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["figma", "mcp", "ai", "cursor"]
published: true
publication_name: "sonicmoov"
---

## はじめに

こんにちは、原です 🐐

少し前に Figma MCP（非公式）の [Figma-Context-MCP](https://github.com/GLips/Figma-Context-MCP) が話題になりましたね

そして 2025 年 6 月 4 日、ついに Figma 公式の MCP サーバー がベータリリースされました 🚀

早速試してみたので、セットアップ周りや使い方、使ってみた所感などをお伝えします！

## Dev Mode MCP サーバー

Figma 公式の MCP サーバーは **Dev Mode MCP サーバー** と呼ばれているようです

以下は Dev Mode MCP サーバー お披露目時のブログ記事です

https://www.figma.com/blog/introducing-figmas-dev-mode-mcp-server/

誤解を恐れずに言うと、Figma と AI を連携し、効率よく開発を進められるようになるツールですね ✨

## セットアップ

詳細は公式を確認していただくとして、軽くまとめておきます！

:::message

**この機能を使用できるユーザー**

- [プロフェッショナル、ビジネス、またはエンタープライズプラン](https://help.figma.com/hc/ja/articles/360040328273-Figma-plans-and-features)の [Dev またはフルシート](https://help.figma.com/hc/ja/articles/27468498501527-Updates-to-Figma-s-pricing-seats-and-billing-experience#h_01JCPBM8X2MBEXTABDM92HWZG4)で利用できます
- MCP サーバーをサポートするコードエディターまたはアプリケーションを使用する必要があります(VS Code、Cursor、Windsurf、Claude Code など)
- Dev Mode MCP サーバーは Figma デスクトップアプリからのみ使用可能です

:::

### 1. MCP サーバーを有効にする

Figma Design ファイルのメニューから [Enable Dev Mode MCP Server] を選択します

![Enable Dev Mode MCP Server](/images/figma-dev-mode-mcp/mcp_menu.png =400x)

### 2. MCP クライアントをセットアップする

ここでは Cursor の設定方法を紹介します

1. Cursor 設定の MCP Tools から [+ New MCP Server] をクリックします

![Cursor MCP設定](/images/figma-dev-mode-mcp/cursor-mcp.png)

2. 次の設定を入力して保存します

```json
{
  "mcpServers": {
    "Figma": {
      "url": "http://127.0.0.1:3845/sse"
    }
  }
}
```

:::message
[Figma-Context-MCP](https://github.com/GLips/Figma-Context-MCP) を使う場合、Figma の personal access token が必要でしたが、Dev Mode MCP サーバーを使う場合は不要です！
:::

3. 設定完了後、サーバーを起動すれば完了です ✨

https://help.figma.com/hc/ja/articles/32132100833559-Dev-Mode-MCP%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E5%88%A9%E7%94%A8%E3%82%AC%E3%82%A4%E3%83%89

## 使い方

Figma design のコンテキストを AI クライアントに提供する方法は「選択ベース」と「リンクベース」の 2 つがあります

選択ベースで指示を出す場合、Figma のデスクトップアプリ上で Figma 内のフレームまたはレイヤーを選択します

選択状態を維持したまま、以下のようなプロンプトで指示を出すことができます

```plaintext:プロンプト例
現在選択中の Figma デザインをもとに、このモーダルにボタンを追加して。
```

![プロンプト](/images/figma-dev-mode-mcp/prompt.png)

リンクベースで指示を出す場合、Figma のフレームまたはレイヤーへのリンクをコピーして、そのままプロンプトに使用すれば OK です

特に理由がなければ選択ベースで指示を出すのがお手軽で良いかと思います 🤖

## 所感 & コツ

Dev Mode MCP サーバーを活用して実際にコンポーネントを実装してみました

初期のモックアップとしては十分に活用できそうで、コンポーネント単体の実装コストも半減ぐらいにはなるかなと感じました 💪

全体的には「発展途上」という印象ですが、モデル性能の向上や MCP サーバー自体の強化で、今後さらに便利に使えるようになるとは思います

コード生成の質を上げるためのコツは以下のように感じています（とはいえ毎回生成結果が異なったりはするので、ここはチューニングしていきたいところです 😅）

### AI エージェント向けのルールファイルの定義

プロジェクト構成の概要やコーディング規約があると生成精度も高まります
（Figma デザインの再現性には直結しない範囲ですが、成果物自体の精度は向上します）

### Figma のフレーム・レイヤー選択は小さめに

一度に多くの要素を選択すると、生成精度が落ちる印象でした

まずは小さなコンポーネントから始めてみるのが良いと思います

### 実装済みのコンポーネントをプロンプトに伝える

例えば `Button` コンポーネントをすでに実装済みで、それを利用した `Card` コンポーネントを作成する場合、プロンプトのコンテキストに `Button` の存在を含めるとよいです

単に `Card` の生成だけを依頼すると、`Button` コンポーネントをうまく活用してくれなかったりするので、そのあたりの指示は丁寧に出したほうが結果的にコスパ・タイパよく実装できるかと思います

```plaintext:プロンプト例
選択中の Figma デザインをもとに、Card コンポーネントを実装して。
Button コンポーネントは @button.tsx に実装済みなので活用して。
```

## まとめ

公式からリリースされた Dev Mode MCP サーバーを活用して、コンポーネントを実装してみました

Dev Mode が使える環境であれば利用可能なのでぜひ試してみてください！

まだ Code Connect との併用が試せていないので、そちらも検証してみたいと思います〜

AI 活用で実装スピードどんどん上げていきましょう 🐐💨

## 参考

- https://www.figma.com/blog/introducing-figmas-dev-mode-mcp-server/
- https://help.figma.com/hc/ja/articles/32132100833559-Dev-Mode-MCP%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E5%88%A9%E7%94%A8%E3%82%AC%E3%82%A4%E3%83%89
