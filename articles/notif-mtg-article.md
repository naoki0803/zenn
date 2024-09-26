---
title: "脱MTG忘れ ~Google APIとSlackWebhookで自動リマインダーを構築してみた~"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [OAuth, GoogleAPI, Slack, SlackWebHook, shell script]
published: false
---

## はじめに

毎朝仕事の始まりは、その日のスケジュールを確認して、MTG 開始時刻の数分前にアラームをかける作業を行っている方は多いと思います。

特別大変な作業でもないので、私も毎朝続けてきましたが、頭のどこかで`アラームのかけ忘れはないか？` `MTGまであと何分だ？`と心配事を常に**ぼんやり**抱えている状態でした。

今回は、そんな心配事を取り除き作業に集中できる環境を作る為に、`当日のスケジュール確認`と`開始2分前のリマインド`を自動化してみました。
もし、同じ様な思いがある方は、初期設定だけちょっと大変ですが、ぜひ実施してみてください。

<!-- github で公開しているので、よければご利用ください。
https://github.com/naoki0803/NotifMTG

:::message
もともと、Udemy のコースで ServerLessFramework の勉強で、Lambda で AWS の料金を毎朝通知するプログラムをハンズオンで実装したのですが、

これ、MTG の通知も Lambda で出来るんじゃ？
あれ、Lambda だと多少お金かかるから Mac だけでできそうじゃない？

という具合で作成したので、github には serverless.yml のファイルがあります。
:::
https://www.udemy.com/course/aws-lambda-serverless-framework/?couponCode=ST11MT91624A -->

## 利用している技術

-   GoogleCalendarAPI
-   OAuth2.0
-   SlackWebhook
-   shell script
-   Node.js
-   cron

## プログラムの全体像

### このプログラムで出来ること

このプログラムは、OAuth2.0 を用いて Google Calendar API にアクセスし、登録されたミーティングの予定を取得して、以下の内容を Slack に通知します。

1.  朝一、当日のスケジュール一覧通知
2.  MTG 開始 2 分前のリマインド通知

また、プログラムの実行は cron を利用して、shell script から自動的に実行しているので、作業漏れが発生しません。

### このプログラムで出来ないこと(注意点)

現状では、以下に該当した場合リマインドの通知がされません。

-   PC 再起動後すべてのリマインド通知
-   cron で./script.sh を実行した時間以降に`追加登録されたスケジュール`のリマインド通知

また、利用は Mac を想定しており、Windows では cron の設定ができないので、本記事をそのままでは実行出来ません。
タスクスケジューラーなどで同じことが実現できると思いますが、未検証です。

## フォルダ構成と機能説明

### 📁 フォルダ構成

```
app
├── .env
├── README.md
├── config
│   ├── credentials.json
│   └── token.json
├── eslint.config.mjs
├── logs
│   └── cron.log
├── package-lock.json
├── package.json
├── script.sh
└── src
    ├── googleAuth.js
    └── mtgNotif.js
```

### 🔐 認証機能 (googleAuth.js)

OAuth2.0 を使用して Google Calendar API にアクセスするための認証処理を実装しています。
保存された認証情報(token.json)を読み込み、必要に応じて credentials.json の値を利用して、新しい認証情報を取得し token.json として保存します。

:::message
git clone をした時点では、`token.json`は作成されていませんが、プログラムを初回起動した際に Google アカウントにログインして、OAuth 認証が完了すると、自動的に`token.json`が作成されます。
:::

:::message
googleAuth.js ファイルは、以下リンクの GoogleWorkspace の Node.js クイックスタートに記載されている、サンプルを参考にして作成しています。
:::
https://github.com/googleworkspace/node-samples/blob/main/calendar/quickstart/index.js

```js:config/googleAuth.js
const fs = require('fs').promises;
const path = require('path');
const process = require('process');
const { authenticate } = require('@google-cloud/local-auth');
const { google } = require('googleapis');

// スコープの設定
const SCOPES = ['https://www.googleapis.com/auth/calendar.readonly'];
// Pathの設定
const TOKEN_PATH = path.join(process.cwd(), 'config', 'token.json');
const CREDENTIALS_PATH = path.join(process.cwd(), 'config', 'credentials.json');

/**
* 保存された認証情報を読み込む関数
*
* @return {Promise<OAuth2Client|null>}
*/
async function loadSavedCredentialsIfExist() {
  try {
    // トークンファイルを読み込む
    const content = await fs.readFile(TOKEN_PATH);
    // 読み込んだ内容をJSONとして解析
    const credentials = JSON.parse(content);
    // Googleの認証オブジェクトを生成
    const res = google.auth.fromJSON(credentials);
    return res; // 認証オブジェクトを返す
  } catch {
    return null; // エラーが発生した場合はnullを返す
  }
}

/**
* 認証情報を保存する関数
*
* @param {OAuth2Client} client
* @return {Promise<void>}
*/
async function saveCredentials(client) {
  // credentials.jsonを読み込み
  const content = await fs.readFile(CREDENTIALS_PATH);
  // 読み込んだ内容をJSONとして解析
  const keys = JSON.parse(content);
  // 使用する認証情報を選択（インストールされたアプリの情報か、ウェブアプリの情報か）
  const key = keys.installed || keys.web;
  // 保存するためのペイロードを作成
  const payload = JSON.stringify({
    type: 'authorized_user',
    /* eslint-disable camelcase */
    client_id: key.client_id,
    client_secret: key.client_secret,
    refresh_token: client.credentials.refresh_token,
    /* eslint-enable camelcase */
  });
  // ペイロードをトークンファイルに保存
  await fs.writeFile(TOKEN_PATH, payload);
}

/**
* APIを呼び出すための認証を行う関数
*
* @return {Promise<OAuth2Client>}
*/
async function authorize() {
  let client = await loadSavedCredentialsIfExist();
  if (client) {
    return client;
  }
  client = await authenticate({
    scopes: SCOPES,
    keyfilePath: CREDENTIALS_PATH,
  });
  if (client.credentials) {
    await saveCredentials(client);
  }
  return client;
}

// authorize関数とgenerateAuthUrl関数をエクスポート
module.exports = {
  authorize
};
```

### 🔔 予定取得と通知機能 (mtgNotif.js)

mtgNotif.js ファイルでは、googleAuth.js で認証された OAuth Client を利用して、Google Calendar から当日のスケジュールを取得します。
取得したスケジュールを整形し、その日の MTG 一覧を Slack へ通知します。
また、各ミーティングの開始 2 分前にも、Slack でリマインダー通知を行います。

```js:src/mtgNotif.js

const { IncomingWebhook } = require('@slack/webhook');
const { authorize } = require('./googleAuth.js');
const { google } = require('googleapis');
const dayjs = require('dayjs');
const cron = require('node-cron');

require('dotenv').config();

// SlackのWebhook URLを環境変数から取得
const webhookUrl = process.env.SLACK_WEBHOOK_URL;
const webhook = new IncomingWebhook(webhookUrl);


// Google Calendar APIからイベント(Schedule)を取得する関数
async function fetchTodayMtgSchedules() {
  try {
    // googleAuth.jsからexportしたauthorize関数を使ってOAuth2クライアントを取得
    const auth = await authorize();

    // Google Calendar APIのセットアップ
    const calendar = google.calendar({ version: 'v3', auth });

    // 今日の日付を設定
    const today = new Date();
    const startOfDay = new Date(today.setHours(0, 0, 0, 0)).toISOString();
    const endOfDay = new Date(today.setHours(23, 59, 59, 999)).toISOString(); // eslint-disable-line no-magic-numbers

    // Google Calendar APIからイベント(Schedule)の取得
    const response = await calendar.events.list({
      calendarId: process.env.CALENDAR_ID,
      timeMin: startOfDay,
      timeMax: endOfDay,
      singleEvents: true,
      orderBy: 'startTime',
    });

    // イベント(Schedule)の整形
    const events = response.data.items;
    const result = events.map(event => ({
      summary: event.summary,
      dateTime: new Date(event.start.dateTime).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' }) // 日本時間に変換
    }));
    return result;
  } catch (error) {
    console.log(`スケジュールの取得に失敗しました: ${error}`);  // eslint-disable-line no-console
    return [];
  }
}

// Slackに通知する関数
async function sendMtgNotification(events) {
  try {
    // result(Scheduleの内容)を整形して、Slackに通知
    if (events.length) {
      const formattedEvents = events.map(event => `\t･ ${event.dateTime} - ${event.summary}`).join('\n');
      await webhook.send({
        text: `【本日のMTG予定】\n${formattedEvents}`
      });

      // Reminderの設定
      scheduleReminder(events);

    } else {
      await webhook.send({
        text: '【本日のMTG予定】\n 本日MTGの予定はありません'
      });
    }
  } catch (error) {
    await webhook.send({
      text: `通知送信中にエラーが発生しました: ${error}`
    });
  }
}

// 指定された会議のn分前に通知する関数
function scheduleReminder(events) {
  events.forEach(event => {
    const today = dayjs().format('YYYY-MM-DD'); // 今日の日付を取得
    const eventDateTime = `${today} ${event.dateTime}`; // 今日の日付に時間を結合
    const eventTime = dayjs(eventDateTime, 'YYYY-MM-DD HH:mm'); // 結果をdayjsオブジェクトに変換
    const reminderMinutes = 2;  // 会議の何分前に通知するかを変数に格納
    const notifyTime = eventTime.subtract(reminderMinutes, 'minute'); // 通知する時間を計算
    const cronTime = `${notifyTime.minute()} ${notifyTime.hour()} * * *`; // cron時間を設定

    cron.schedule(cronTime, () => {
      // 非同期即時実行関数(IIFE)を使って、非同期処理をその場で実行
      (async function () {
        try {
          // 非同期処理を行う（例: Slackへのメッセージ送信）
          await webhook.send({
            text: `"${event.summary}" が2分後に始まります`
          });
        } catch {
          await webhook.send({
            text: `Reminder送信中にエラーが発生しました: "${event.summary}"`
          });
        }
      })(); // ここで関数を定義すると同時に実行している
    });
  });
};

// toDayMtgNotif関数をリファクタリング
async function toDayMtgNotif() {
  try {
    const events = await fetchTodayMtgSchedules();
    await sendMtgNotification(events);
  } catch (error) {
    await webhook.send({
      text: `toDayMtgNotifの実行に失敗しました: ${error}`
    });
  }
}

toDayMtgNotif();
```

### 🤖 自動実行機能 (script.sh)

crontab で登録した実行日時に、src/mtgNotif.js を実行する為のスクリプトです。
Slack の起動もこのスクリプトの中で実行しています。

:::message
このスクリプトを手動実行する事で、再起動後や、追加された MTG のリマインド通知を実行する事ができます。
※再起動後や、スケジュールが追加された場合は、ターミナルでプロジェクトに移動して`./script.sh`を実行してください。
:::

```sh:script.sh
#!/bin/bash
# Slackが起動していない場合、Slackを起動
if ! pgrep -x "Slack" > /dev/null; then
  echo "Starting Slack..."
  open -a "Slack"  # Slackを起動
  sleep 5 # 完全に起動するまで少し待つ
fi

# プロセスを停止（前日のプロセスを終了）
pid=$(ps aux | grep 'src/mtgNotif.js' | grep -v grep | awk '{print $2}')

if [ -n "$pid" ]; then
  kill "$pid"
  echo "Stopped process: $pid"
fi

cd ~/projects/NotifMTG

# mtgNotifをバックグラウンドで実行
/opt/homebrew/bin/node ~/projects/NotifMTG/src/mtgNotif.js &
```

#### cron の設定

平日 8:45 に実行する場合の crontab のサンプル

```cron:crontab -l
45 8 * * 1-5 /bin/zsh -c 'source ~/projects/NotifMTG/script.sh' >> ~/projects/NotifMTG/logs/cron.log 2>&1
```

## 利用前の環境設定

事前に以下作業を実施する必要があります。

1. git clone と npm install
2. Google API の利用準備
3. Google Calendar の ID を確認
4. SlackWebhookURL の取得
5. cron の設定

### 1. git clone と npm install

Terminal で任意のディレクトリに移動して以下コマンドを一行ずつ実行

```
git clone https://github.com/naoki0803/NotifMTG.git
npm install
```

:::message
git？npm？それ美味しいの？という方は以下を実施してください。

1. nodejs をインストール
   https://qiita.com/sefoo0104/items/0653c935ea4a4db9dc2b

2. 以下 URL にアクセスして、緑色の`<> Code ▼`をクリックして`Downloads ZIP`をクリックしてください。
   https://github.com/naoki0803/NotifMTG.git
   ![image](/images/notif-mtg-article/Slack_12.png =500x)

:::

### 2. Google API の利用準備

https://zenn.dev/yusuke49/articles/6c147bd6308912
こちらの記事の内容をすべて実施いただき、最終的に JSON ファイルをダウンロードします。
ダウンロードした json ファイルの名前を、`credentials.json`に変更して、config フォルダ内に保存します。

:::message alert
2 点だけ記事と異なる部分がありますので、以下も合わせてご確認ください。

#### 1 点目: OauthClient のスコープ

記事内の`3. OAuth同意を構成`の手順の手順 5 の画面で、スコープを選択しますが、
今回のプログラムの場合`auth/calendar.readonly`だけ選択いただければ動きます。
![image](/images/notif-mtg-article/Slack_9.png =500x)

#### 2 点目: 承認済みリダイレクト URI の記述

記事内の`4. アクセス認証情報を作成`の手順 3 の画面にある
承認済みのリダイレクト URI に`http://localhost:3000/callback`と入力して作成をクリックしてください。
![image](/images/notif-mtg-article/Slack_11.png =500x)

#### JSON ファイルのダウンロードと保存

上記画像の作成を押すと json のダウンロードが出来ますので、ファイルの名前を、`credentials.json`に変更して、config フォルダ内に保存します。

```
app
├── .env
├── config
│   └── credentials.json  #ここに保存します
└── src
    ├── googleAuth.js
    └── mtgNotif.js
```

![image](/images/notif-mtg-article/Slack_10.png =300x)

:::

### 3. Google Calendar の ID 確認

https://qiita.com/mikeneko_t98/items/60e264941492d0b44fe5

1. MTG のスケジュールなどを入力している Google Calendar の ID を確認して、`.env`ファイルに入力します。

```env:.env
CALENDAR_ID=カレンダーIDを入力する
```

```
app
├── .env
├── config
│   └── credentials.json
└── src
    ├── googleAuth.js
    └── mtgNotif.js
```

### 4. SlackWebhookURL の取得

1. 以下 URL に接続して`Go to Your Apps`をクリック
   https://api.slack.com/quickstart
   ![image](/images/notif-mtg-article/Slack_1.png =500x)

2. `Create New App`をクリック
   ![image](/images/notif-mtg-article/Slack_2.png =500x)

3. 下段の`From scratch`をクリック
   ![image](/images/notif-mtg-article/Slack_3.png =500x)

4. 任意の`AppName`の入力と、通知を送信する Slack の `Workspace` を選択して Create App をクリック
   ![image](/images/notif-mtg-article/Slack_4.png =500x)

5. 作成された App のサイドバーから`Incoming Webhooks`を選択して、Activate Incoming Webhooks を`On`に変更する。
   ![image](/images/notif-mtg-article/Slack_5.png =500x)

6. 同じ画面下部にある`Add New Webhook to Workspace`をクリック
   ![image](/images/notif-mtg-article/Slack_6.png =500x)

7. 通知を送信する`チャンネル`を選択して、許可をクリック
   ![image](/images/notif-mtg-article/Slack_7.png =500x)
   :::message
   私は自分自身に送っていますが、任意のチャンネルを選択していただいて大丈夫です。
   :::

8. 作成された Webhook URL を`.env`ファイルに入力します。
   ![image](/images/notif-mtg-article/Slack_8.png =500x)

```env:.env
SLACK_WEBHOOK_URL=Webhook URLを入力する
```

```
app
├── .env
├── config
│   └── credentials.json
└── src
    ├── googleAuth.js
    └── mtgNotif.js
```

### 5. cron の設定

1.Terminal で`crontab -e`を実行して以下 cron を登録

```cron
45 8 * * 1-5 /bin/zsh -c 'source ~<ご自身のprojectPath>/NotifMTG/script.sh' >> ~/<ご自身のprojectPath>/NotifMTG/logs/cron.log 2>&1
```

:::message
上記は平日 8:45 に通知する場合の cron です。
:::

### 6. 初回起動と認証

1. ターミナルで`node src/mtgNotif.js`を実行します。

2. ブラウザが開き、Google アカウントのログイン画面が表示されますので、ログインして、アプリのアクセスを許可してください。
3. 認証が成功したら、ターミナルに`Authorization complete`と表示され、Slack に本日の MTG の通知が来ます。

:::message
プログラムに Google Calendar の API を利用する権限を許可する作業なので初回のみ表示されます。

また、認証が完了すると`config`フォルダ内に`token.json`が保存されます。

```
app
├── .env
├── config
│   └── credentials.json
│   └── token.json
└── src
    ├── googleAuth.js
    └── mtgNotif.js
```

:::

## まとめ

実際に利用してみて、私にはもう`なくてはならない物`になっています。
朝一の手動アラーム設定作業と、どこかでスケジュールのことを気にかけて、うすーく頭のリソースを使っている状態から開放されました。
作業に集中できますし、自動化したことによってアラームのかけ忘れも発生しないので本当に快適です。

初期設定は色々と大変ですが、やってみる価値はあると思いますのでぜひ。
再起動後の自動実行や、追加登録されたスケジュールの通知も順次対応しようと思います.

また、内容に不備があれば、忌憚のないご意見をお願いします。
