---
title: "Mac と Cursor で C# の環境構築"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [csharp, 環境構築, mac]
published: false
---

## 背景

TODO: なんか背景がすんなり入ってこない。伝えたいのは何か集中できてない Cursor？vsc?mac?debug?
転職先が C#環境のため、Mac で Cursor を用いた環境を構築しました。

C#開発は、Visual Studio `Code` ではなく、 Visual Studio(以下 VS Code) を用いて開発すると、
機能モリモリで開発しやすいようなのですが、Mac 版の提供はなくなっているようでした。
TODO: エビデンス確認

ただ、私は VS Code ではなく、Cursor をエディターに採用しているので、
Cursor 環境でも C# の開発環境を構築できたので、その方法についてまとめました。

:::message
C#の Debug は VS Code のみで利用できるライセンスになっているようで、
拡張機能を入れただけだと、Cursor ではデバッグすることができません。
そちらに関しては別途記事を作成しようと思います。TODO: 本当にあとでいい？
:::

## 環境

-   MacBook Air M2 15.3.2（24D81）
-   Cursor 0.47.8
-   .Net 9.0.201

## 環境構築

1. Cursor のインストール
   [Cursor](https://www.cursor.com/ja)
   or
   [Homebrew](https://formulae.brew.sh/cask/cursor#default)でも OK

    ```
    brew install --cask cursor
    ```

2. .Net SDK のインストール
   [.Net SDK のダウンロードページ](https://dotnet.microsoft.com/ja-jp/download/dotnet/sdk-for-vs-code?utm_source=vs-code&utm_medium=referral&utm_campaign=sdk-install)

    私は M2 Mac なので Arm 64 をインストールしています。
    Homebrew もありますが、バージョンが

    :::message
    homebrew でも OK ですが、手順 4 で Path を通す際に記述している Path と異なる点にご注意ください

    ```
    $ brew install --cask dotnet-sdk
    ```

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
    dotnet new に続くオプションは console 意外にも以下のようなテンプレートがあります。
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

Path を通すっという発想に至るまでに時間がかかりました。
dotnet と入力しても `Command Not Found` と表示される時点で、これは Path だ！っと直結できなかった自分とサヨナラすることができました。

また、実際に作成した環境で デバッグ を実行すると、
デバッグの実行は VS Code の専用ライセンスだから Cursor 君はダメー!!っと仲間に入れてもらえないのですが、世界は広い物で、Cursor 君はこっちで一緒に遊ぼうよと言ってくれる、素晴らしいコミュニティーが作成(?)した、Debug のツールが存在していました。

そのあたりはまた、別の記事で記載しようと思います.
※そんなん待ってられん！という方は以下参考にしてください。

> こちらの方が、方法を紹介してくださっています。

https://guiferreira.me/archive/2025/how-to-debug-dotnet-code-in-cursor-ai/

> Debug は VS Code のライセンスやでーっという説明

https://github.com/dotnet/vscode-csharp/wiki/Microsoft-.NET-Core-Debugger-licensing-and-Microsoft-Visual-Studio-Code

> Cursor でデバッグ使えないの？使えるよ！っという板

https://forum.cursor.com/t/cursor-dont-support-c-debugger/1936/4
