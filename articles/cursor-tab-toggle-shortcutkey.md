---
title: "Cursor Tab 機能をショートカットキーでトグルする方法"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [cursor, shortcut, cursortab]
published: false
---

## この記事でできること

-   Cursor の有料機能「Cursor Tab」を 1 つのショートカットキーでオン/オフできるようになります

## 背景

Cursor に課金すると利用可能な Cursor Tab 機能はとても便利な反面、言語キャッチアップのために、自分でコードを書いて身体で覚えたい時などは、機能をオフにして、適切なタイミングで機能を切り替えたくなることがあります。

以下の記事で`disable cursor tab` または `Enable cursor tab` コマンドを実行することで、オン/オフできることを紹介していただいていたので、しばらくこの方法を使っていました。
https://zenn.dev/take_tech/articles/77383d811797cc

しかし、毎回コマンドを入力するのは少し手間がかかります。
そこで、1 つのショートカットキーで簡単にトグルできる方法を見つけたので共有します。

:::message
↓ Cursor Tab はこれです。
![](/images/cursor-tab-toggle-shortcutkey/1.png =900x)
Cursor の有料機能の一つで、コードを書く際にタブキーを押すと AI が提案するコード補完を受け入れることができる機能です。

-   [Cursor 公式ドキュメント - Cursor Tab 機能](https://docs.cursor.com/tab)

:::

## 設定方法

### 1. キーボードショートカットの設定画面を開く

1. `⌘ + Shift + P` を押してコマンドパレットを開く
2. `Keyboard Shortcuts` を選択

### 2. ショートカットキーの設定

1. 検索バーに「Toggle Cursor Tab 」と入力します
2. ショートカットキーを設定します

    ![](/images/cursor-tab-toggle-shortcutkey/2.png =900x)
    例：`ctrl + ⌘ + T`

### 3. 動作確認

1. 設定したショートカットキーを押して、Tab 機能がオンになることを確認

    ステータスバーの表示が、on/off で切り替わります。
    ![](/images/cursor-tab-toggle-shortcutkey/3.png =500x)
    ![](/images/cursor-tab-toggle-shortcutkey/4.png =500x)

## まとめ

地味ですが、個人的にはとても便利です。
