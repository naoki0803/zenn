---
title: "Mac && Cursor で C# の環境構築をしてみた"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [csharp, 環境構築, mac, cursor]
published: false
---

## 背景

C#をベースに開発をしている企業へ転職することになったので、言語キャッチアップの為に、Mac かつ Cursor を用いた開発環境の構築を行いました。

C# は Visual Studio を用いると、機能モリモリで開発しやすいようなのですが、Mac 版の提供は終了しており、`Visual Studio Code(以下VS Code) + 拡張機能`で代替する必要があります。
https://github.com/dotnet/vscode-csharp/wiki/Microsoft-.NET-Core-Debugger-licensing-and-Microsoft-Visual-Studio-Code

私は Cursor をエディターに採用しているので、Cursor で C# の開発環境を構築したのですが、躓くポイントが多かったので、同じような境遇の方の参考になればと思い記事にしてみました。

:::message alert
C# のデバッグはライセンスの関係で VS Code のみ利用可能ですが、別途ツールを導入することで、Cursor でもデバッグができるようになります。
そちらに関しては別記事を作成予定ですが、すぐに知りたい方は最下部の`参考情報`をご確認ください。
:::

## 環境

-   MacBook Air M2 15.3.2（24D81）
-   Cursor 0.47.8
-   .Net 9.0.201

## 環境構築

1. Cursor のインストール
   以下いずれかで Cursor をインストールしてください

    - [Cursor のダウンロードページ](https://www.cursor.com/ja)
    - [Homebrew](https://formulae.brew.sh/cask/cursor#default)でも OK

    ```
    brew install --cask cursor
    ```

2. .Net SDK のインストール

    - [.Net SDK のダウンロードページ](https://dotnet.microsoft.com/ja-jp/download/dotnet/sdk-for-vs-code?utm_source=vs-code&utm_medium=referral&utm_campaign=sdk-install)

    :::message
    [Homebrew](https://formulae.brew.sh/cask/dotnet-sdk)でも OK ですが、手順 4 で Path を通す際に記述している Path と異なる点にご注意ください
    :::

3. Cursor の拡張機能 `C# Dev Kit` をインストール
   ![](/images/csharp-environment-construction/CsharpDevkit.png)
   C# Dev Kit をインストールすると、依存関係として、C#[(ms-dotnettools.csdevkit)](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) もインストールされます。

4. 環境変数とシンボリックの作成
   .Net SDK をインストールすると、`dotnet` コマンドを利用する事ができるようになりますので、PATH を通しておきます。
   また、なにやら issue が発生しているようで、シンボリックリンクを作成することで、回避する事ができます。
   https://zenn.dev/unsoluble_sugar/articles/982e38df5ffbd9

    ```
    $ export DOTNET_ROOT=/usr/local/share/dotnet
    $ export PATH=$PATH:$DOTNET_ROOT:$DOTNET_ROOT/tools
    $ sudo ln -s /usr/local/share/dotnet/dotnet /usr/local/bin/dotnet
    ```

    :::message
    Terminal で ` dotnet --version` で現在の Version が表示されれば OK です

    ```
    $ dotnet --version
    9.0.201
    ```

    :::

## C# プロジェクトの作成

1.  プロジェクトを作成したい Directory に移動して、`dotnet new` コマンドを実行

    ```
    dotnet new console -n CsharpFirstProject
    ```

    Project の雛形が作成されます。

    ```tree:tree
    CsharpFirstProject
    ├── Program.cs
    ├── obj
    │   ├── project.assets.json
    │   ├── project.nuget.cache
    │   ├── CsharpFirstProject.csproj.nuget.dgspec.json
    │   ├── CsharpFirstProject.csproj.nuget.g.props
    │   └── CsharpFirstProject.csproj.nuget.g.targets
    └── CsharpFirstProject.csproj
    ```

    :::message
    dotnet new に続くオプションは console 以外にも以下のようなテンプレートがあります。
    | テンプレート種別 | コマンド | 説明 |
    | ---------------- | ---------- | ----------------------------------------------- |
    | プロジェクト | classlib | クラスライブラリプロジェクトを作成します。 |
    | プロジェクト | mvc | ASP.NET Core MVC プロジェクトを作成します。 |
    | プロジェクト | webapi | ASP.NET Core Web API プロジェクトを作成します。 |
    | プロジェクト | xunit | xUnit テストプロジェクトを作成します。 |
    | その他 | globaljson | global.json ファイルを作成します。 |
    | その他 | sln | ソリューションファイルを作成します。 |
    | その他 | gitignore | .gitignore ファイルを作成します。 |
    :::

2.  Hello World
    `dotnet run`を実行して Hello World を表示します。

    ```
    $ dotnet run
    Hello, World!
    ```

    :::message
    Project を作成すると、`Program.cs`に以下記述がされた状態になり、`dotnet run`でプログラムが実行されます。

    ```cs:Program.cs
    Console.WriteLine("Hello, World!");
    ```

    :::

## 最後に

dotnet と入力しても `Command Not Found` と表示される問題に足を取られ、
Path を通すという発想に至るまでに時間がかかりました。
初歩的ではありますが、これもまた成長とポジティブに捉えようと思います。

また、冒頭でも触れましたが、C# のデバッグは、ライセンスの関係で VS Code のみ利用可能です。
実際に作成した環境で デバッグ を実行すると、デバッグの実行は VS Code の専用ライセンスだから Cursor 君はダメー!!と表示され仲間に入れてもらえません。
ただ、世界は広く、Cursor 君はこっちで一緒に遊ぼうよと言ってくれる、素晴らしいコミュニティが作成(?)してくれたデバッグツールを導入する事で無事デバッグも実行できました。

デバッグツールの導入も一筋縄ではいかなかったので、そちらも別途記事にしようと思います。
※現時点でお困りの方は、以下参考情報をご確認ください。

## 参考情報

> Cursor で C# のデバッグを実施する方法

https://guiferreira.me/archive/2025/how-to-debug-dotnet-code-in-cursor-ai/

> Cursor でデバッグ使えないの？使えるよ！っというスレッド

https://forum.cursor.com/t/cursor-dont-support-c-debugger/1936/4
