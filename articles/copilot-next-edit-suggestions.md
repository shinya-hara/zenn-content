---
title: "【VSCode】Copilot NES (Next Edit Suggestions) がすごいぞ！"
emoji: "➡️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", "githubcopilot", "ai"]
published: true
publication_name: "sonicmoov"
---

## 3 行で

- Copilot Next Edit Suggestions がすごい ✨
- `Tab` キーで Copilot の提案を次々に処理できる ➡️
- [`github.copilot.nextEditSuggestions.enabled`](vscode://settings/github.copilot.nextEditSuggestions.enabled) から有効にできるよ ⚙️

## Copilot Next Edit Suggestions

Copilot NES は、コードの「次の編集提案機能」です。従来の Copilot は主にインライン補完を行っていましたが、Copilot NES は **「一連の編集の後、どんな変更が続くとよいか？」** を提案してくれるのが大きな特徴です。

![](/images/copilot-next-edit-suggestions/copilot-nes.gif)

:::message
Cursor にも 似たような Tab キー 機能がありますね
https://www.cursor.com/ja/features
:::

## コード変更の提案に素早く移動 & 承認

`Tab` キーでコード変更の提案に素早く移動できるため、次の変更箇所を探す手間が減ります。
その後、再度 `Tab` キーを押すことで、提案を承認できます。

![](/images/copilot-next-edit-suggestions/gutter-menu-highlighted-updated.png)

## 使用例

### 間違いを見つけて修正

タイポの修正を提案してくれています。

![](/images/copilot-next-edit-suggestions/nes-1.png)

### 実装意図の変化に追従

実装者のコード変更の意図を汲み取って、変更を提案してくれます。

`Point` クラスを `Point3D` クラスに変更したら...
![](/images/copilot-next-edit-suggestions/nes-2-1.png)

`z: number` の追加を提案し...
![](/images/copilot-next-edit-suggestions/nes-2-2.png)

さらには `getDistance()` の実装修正を提案！
![](/images/copilot-next-edit-suggestions/nes-2-3.png)

### リファクタリング

ファイル内の変数名を一度変更すると、Copilot は他のすべてのコードでも更新を提案します。

![](/images/copilot-next-edit-suggestions/nes-3.png)

## 設定方法

エディタ右下の Copilot メニューから `Next Edit Suggestions` を有効にできます。

![](/images/copilot-next-edit-suggestions/copilot-menu-status-bar.png)

設定画面から有効にする場合はここからできます 👇
[`github.copilot.nextEditSuggestions.enabled`](vscode://settings/github.copilot.nextEditSuggestions.enabled)

デフォルトでは有効化されていない？ようなので、動作しない場合は設定を確認してみてください！

## 参考

https://code.visualstudio.com/docs/copilot/ai-powered-suggestions#_next-edit-suggestions
