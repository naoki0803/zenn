---
title: "warpのHotkeyが日本語入力だと反応しない場合の対応"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [warp,btt]
published: true
---
## 背景

https://www.warp.dev/

UIのかっちょよさや、iTermよりも使い勝手が良さそうなので、Terminalアプリの**warp**を導入してみました。

っが、しかし。

iTermと同様にHotkeyを設定して、どこからでもTerminalを呼び出せるのですが、入力モードがmac純正の｢ABC｣でないと、反応しないバグがある事がありました。
https://github.com/warpdotdev/Warp/issues/2795

本記事では、[BTT(Better Touch Tool)](https://folivora.ai/)を利用して、入力ソースが｢ABC｣以外の状態でも、warpで設定したのhotkeyを~~無理やり~~動かす方法を記述しています。


## 環境
Mac Book Air M2(14.6.1)
BTT 4.664
warp v0.2024.09.03.08.02.stable_03

## 設定方法
1. キーボードショートカットで、hotkeyとして設定したキーを設定
![](/images/8edf1e7afed931/image1.png)
:::message
私は｢option + Space｣で設定しています。
:::

2. アクション設定の1つ目で入力ソース｢ABC｣を選択
![](/images/8edf1e7afed931/image2.png)

3. アクション設定の2つ目で改めてhotkeyとして設定したキーを設定
![](/images/8edf1e7afed931/image3.png)


:::message
内容としてはキーボードで｢option + Space｣が入力されたら、入力ソースを｢ABC｣に変更して、再度｢option + Space｣を実行する、という内容です。

なんだかカッコ悪い感はありますが、動きます！
warpは想定したhotkeyで動作します！！
:::

## 最後に
hotkey問題に関するその他のトラブル対応や、Raycastを利用した対処方法を以下記事で紹介してくださっていました。
https://zenn.dev/k_saito7/articles/warp-hotkey-window

また、warp自体の利用方法に関しては公式や、以下の記事を参考にさせていただいています。
https://qiita.com/miruon/items/2c7c2b3da40722f91fbc