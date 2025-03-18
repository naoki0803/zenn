---
title: "git便利コマンド集"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [git, tips]
published: true
---

# 背景

何かしら git で困ったことや、実際に使用した git のコマンドを集めた備忘録です。

適宜内容を更新していきます。

# git diff

## 特定のコミットと現状のフォルダ or ファイルの差分を確認する

```
git diff <差分確認したい特定のcommit ID> <差分確認をしたい作業ディレクトリのフォルダ名 or ファイル名>
```

# git commit

## 直前のコミットメッセージを修正する

1. コミットメッセージをローカルで修正する

```
git commit --amend -m "新しいコミットメッセージ"
```

--amend を使うことで、最新のコミットメッセージを修正できます。

:::message
git push でリモートリポジトリに上げる前であれば、これだけで修正 OK
:::

2. リモートリポジトリに強制プッシュする

```
git push --force
```

push 済みのコミットメッセージを上書きするには、--force オプションをつけて強制的にプッシュする必要があります。

:::message alert
チームでの使用時に注意: リモートリポジトリの履歴を書き換えることになるため、他の開発者が同じリポジトリを利用している場合には、コンフリクトが発生する可能性があります。チームメンバーと調整してから行うことが推奨されます。
:::

## push 済みのリポジトリのコミットメッセージを修正する

1. 変更したいコミットの親コミットを指定

```
git rebase -i HEAD~5
```

:::message
（HEAD~5 は、変更したいコミットが 5 つ前のコミットである場合の例です。実際のコミット数に合わせて調整してください。）
git log で commit ID を確認して、変更したい commit の親コミット を指定しても問題ありません。
:::

2. エディタで表示されるリストで、変更したいコミットの pick を edit に変更:

```
pick 88b18c4 Type Union
edit a0c2d35 hoge Literal //'hoge'を'Type'に変更する
pick 569f1b3 Type enum
```

3. git commit --amend を実行して、コミットメッセージを「Type Union」に修正して保存します。

```
git commit --amend
```

4. rebase を続行:

```
git rebase --continue
```

5.リモートリポジトリへの反映: コミットメッセージを変更した後、git push -f コマンドで強制的にリモートリポジトリに反映できます。
:::message alert
-f オプションは慎重に使用する必要があります。
あくまで個人開発でリモートリポジトリに変更が無いことを前提に作業してください
また、過去の編集前の履歴は残ら無い点にも十分注意してください。
:::

# git rm

## .gitignore に記述する前に、不要なファイルを push してしまった場合の対処方法

1. .gitignore に不要なファイルやフォルダを記述する
2. git rm -r --cached . を実行し、再度 add と commit を実施する

```
git rm -r --cached .
git add .
git commit -m "既に追跡されている無視対象ファイルを削除"
```

::: message
この方法は履歴は残るので、セキュアな情報を push している場合は効果が無い
:::

# git stash

## 作業内容をスタッシュに一時避難する

### 利用背景

-   branch を切り替える前に作業をしてしまった場合
-   作業途中に、別ブランチに切り替える必要がある場合
-   色々やったけどうまくいかない、消すのはアレだけど、元に戻したい

### 作業手順

1. git stash コマンドで作業内容を一時退避させる

```
git stash
```

:::message

```
git stash save
```

と記載する事もあるが、save は省略可能です
:::

2. git stash apply コマンドで退避させた作業内容を戻す

:::message
branch を切り替え前に作業をしてしまった場合などは`git stash` で一時退避した後に、任意の branch に切り替えてから`git stash apply`を実行します。
:::

:::message
apply はスタックとして残り続ける為、保存する 必要がなければ、 `pop` を利用する

```
git stash pop
```

:::

## 新規作成した未追跡ファイルを stash する

1. 以下いずれかのコマンド（同じ意味）を実行する

```
git stash -u
git stash --include-untracked
```

2. 任意の場所に stash を pop する

# git checkout

## 直前のブランチに移動する

1. -（ハイフン）を指定することで、直前にチェックアウトしていたブランチに移動できます。

コマンド例: `git checkout -`

現在のブランチを確認

```
$ git branch --show-current
develop
```

ブランチを feature/hoge に切替

```
$ git checkout feature/hoge
```

現在のブランチを確認

```
$ git branch --show-current
feature/hoge
```

ハイフンを指定して、一つ前のブランチに切替

```
$ git checkout -
```

現在のブランチを確認

```
$ git branch --show-current
develop
```

# gh

## cli でリモートリポジトリを作成する方法

```
$ gh repo create --public --source=.
```

:::message
private で作成する場合は

```
$ gh repo create --private --source=.
```

:::
