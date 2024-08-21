---
title: "dotenvなしでNode.jsの環境変数を読み込む方法"
emoji: "👨‍🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [nodejs,env,dotenv]
published: true
---

## 結論
nodeを実行時のオプションに`--env-file=.env`をつける
```
node --env-file=.env.env hoge.js
```
:::message
nodemon のオプションとして指定もできました。
:::


## 最後に
オプションつけずに実行してエラーになったりもするので、
dotenvをインストールして`require('dotenv').config()`を1行追加した方がいいような気もしてはいます。

## 参考
https://qiita.com/n0bisuke/items/c9f8cc3b7ddd419fcf1e
https://qiita.com/ozaki25/items/3e2cf94f29bd0edc1979