# Slack 連携実装

最終的には定期実行になるが，まずは手動で動作させるため API を作成しておく．

（フロントにもテスト画面を実装して手動実行させられるようにしたい）

## 準備

Node.js から Slack へ投稿するためにライブラリをインストールする．

```bash
$ npm i @slack/web-api
```

また，Slack 連携の処理を実装するため，下記ファイルを作成しておく．

- `routes/slack.route.js`
- `controllers/slack.controller.js`
- `services/slack.service.js`

`app.js` を以下のように編集する．

```js
// app.js

import express from "express";
import { omikujiRouter } from "./routes/omikuji.route.js";
import { jankenRouter } from "./routes/janken.route.js";
import { todoRouter } from "./routes/todo.route.js";
// 🔽 追加
import { slackRouter } from "./routes/slack.route.js";

const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
const port = 3001;

app.get("/", (req, res) => {
  res.json({
    uri: "/",
    message: "Hello Node.js!",
  });
});

app.use("/omikuji", (req, res) => omikujiRouter(req, res));
app.use("/janken", (req, res) => jankenRouter(req, res));
app.use("/todo", (req, res) => todoRouter(req, res));
// 🔽 追加
app.use("/slack", (req, res) => slackRouter(req, res));

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

## ルーティングの作成

エンドポイントは 2 つ作成する．

- `slack/` で todo リスト全件を Slack 投稿．
- `slack/today` で本日以前締切のデータのみを Slack 投稿．

```js
// routes/slack.route.js

import express from "express";
import {
  sendAllTodoData,
  sendTodayTodoData,
} from "../controllers/slack.controller.js";

export const slackRouter = express.Router();

slackRouter.get("/", (req, res) => sendAllTodoData(req, res));
slackRouter.get("/today", (req, res) => sendTodayTodoData(req, res));
```

## コントローラの作成

サービスの処理を呼び出しつつ，レスポンスを設定．

```js
// controllers/slack.controller.js

import {
  postAllTodoData,
  postTodayTodoData,
} from "../services/slack.service.js";

export const sendAllTodoData = async (req, res, next) => {
  try {
    const result = await postAllTodoData();
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Successfully send All Todo Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

export const sendTodayTodoData = async (req, res, next) => {
  try {
    const result = await postTodayTodoData();
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Successfully send Today Todo Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};
```

## サービスの作成

メインのロジックを記述する．

Slack 連携のライブラリを読み込み，投稿の処理を実装する．

- `postToSlack()` では `client.chat.postMessage` を使用し，任意の channel に任意のテキストを投稿している．

- `postAllTodoData()` では前項で実装した `findAll()` メソッドで全件データを取得し，`postToSlack()` メソッドで投稿を行う．

- `postTodayTodoData()` では前項で実装した `findToday()` メソッドで全件データを取得し，`postToSlack()` メソッドで投稿を行う．

- （投稿のテキストはいい感じに作りましょう w）

```js
// services/slack.service.js

import { findAll, findToday } from "../repositories/todo.repository.js";
import { WebClient } from "@slack/web-api";
import dotenv from "dotenv";

dotenv.config();

const postToSlack = async (token, channel, text) => {
  const client = new WebClient(token);
  const response = await client.chat.postMessage({ channel, text });
  console.log(response.ok);
  return response;
};

export const postAllTodoData = async () => {
  try {
    const token = process.env.SLACK_API_TOKEN;
    const todoData = await findAll();
    const text = todoData.map((x) => `${x.deadline}\t${x.todo}`).join("\n");
    return await postToSlack(token, "#test", `現在のTodo！\n${text}`);
  } catch (e) {
    throw Error("Error while posting All Todo Data");
  }
};

export const postTodayTodoData = async () => {
  try {
    const token = process.env.SLACK_API_TOKEN;
    const todoData = await findToday();
    const text = todoData.map((x) => `${x.deadline}\t${x.todo}`).join("\n");
    return await postToSlack(token, "#test", `本日締切！！\n${text}`);
  } catch (e) {
    throw Error("Error while posting Today Todo Data");
  }
};
```

## 動作確認（手動）

設定したエンドポイントにリクエストを送って Slack に投稿されることを確認！

### 全件投稿

```bash
$ curl http://localhost:3000/slack

{
  "status": 200,
  "result": {
    "ok": true,
    "channel": "C01EF7W0YF3",
    "ts": "1639641376.000100",
    "message": {
      "bot_id": "B01EF806CV7",
      "type": "message",
      "text": "現在のTodo！\n2021-12-04\treact\n2021-12-06\tnode.js\n2021-12-06\ttest\n2021-12-07\tzoom meeting\n2021-12-20\tNext.js\n2021-12-31\tnode.js\n2021-12-31\tnode.js\n2021-12-31\tnode.js\n2021-12-31\tnode.js\n2021-12-31\ttest",
      "user": "U01ESE8S9NU",
      "ts": "1639641376.000100",
      "team": "T4GCXGMGQ",
      "bot_profile": {
        "id": "B01EF806CV7",
        "app_id": "A01E8PJTS04",
        "name": "cat-commander",
        "icons": {
          "image_36": "https://avatars.slack-edge.com/2021-12-07/2833480487664_be12a9ce689e9e465952_36.png",
          "image_48": "https://avatars.slack-edge.com/2021-12-07/2833480487664_be12a9ce689e9e465952_48.png",
          "image_72": "https://avatars.slack-edge.com/2021-12-07/2833480487664_be12a9ce689e9e465952_72.png"
        },
        "deleted": false,
        "updated": 1638887573,
        "team_id": "T4GCXGMGQ"
      }
    },
    "response_metadata": {
      "scopes": [
        "chat:write"
      ],
      "acceptedScopes": [
        "chat:write"
      ]
    }
  },
  "message": "Successfully send All Todo Data!"
}
```

### 実行日以前の締切のみ投稿

```bash
$ curl http://localhost:3000/slack/today

{
  "status": 200,
  "result": {
    "ok": true,
    "channel": "C01EF7W0YF3",
    "ts": "1639641433.000200",
    "message": {
      "bot_id": "B01EF806CV7",
      "type": "message",
      "text": "本日締切！！\n2021-12-04\treact\n2021-12-06\tnode.js\n2021-12-06\ttest\n2021-12-07\tzoom meeting",
      "user": "U01ESE8S9NU",
      "ts": "1639641433.000200",
      "team": "T4GCXGMGQ",
      "bot_profile": {
        "id": "B01EF806CV7",
        "app_id": "A01E8PJTS04",
        "name": "cat-commander",
        "icons": {
          "image_36": "https://avatars.slack-edge.com/2021-12-07/2833480487664_be12a9ce689e9e465952_36.png",
          "image_48": "https://avatars.slack-edge.com/2021-12-07/2833480487664_be12a9ce689e9e465952_48.png",
          "image_72": "https://avatars.slack-edge.com/2021-12-07/2833480487664_be12a9ce689e9e465952_72.png"
        },
        "deleted": false,
        "updated": 1638887573,
        "team_id": "T4GCXGMGQ"
      }
    },
    "response_metadata": {
      "scopes": [
        "chat:write"
      ],
      "acceptedScopes": [
        "chat:write"
      ]
    }
  },
  "message": "Successfully send Today Todo Data!"
}

```

## 定期実行の設定

手動で投稿する実装が完了したら，時間を指定して定期実行する処理を実装する．

定期実行には `node-cron` ライブラリを用いる．

- 時間を指定しておくと自動で任意の処理を実行してくれる．

- 時間指定には 6 つの数値を用いる．順番が決まっており，以下の通りである．

```txt
 # ┌────────────── second (optional)
 # │ ┌──────────── minute
 # │ │ ┌────────── hour
 # │ │ │ ┌──────── day of month
 # │ │ │ │ ┌────── month
 # │ │ │ │ │ ┌──── day of week
 # │ │ │ │ │ │
 # │ │ │ │ │ │
 # * * * * * *

```

参考：[https://github.com/node-cron/node-cron](https://github.com/node-cron/node-cron)

### ライブラリのインストール

下記コマンドを実行してライブラリをインストールする．

```bash
$ npm i node-cron
```

### 定期実行の処理を追記

サービスの処理でライブラリを読み込み，定期実行の処理を追記する．

```js
// services/slack.service.js

import { findAll, findToday } from "../repositories/todo.repository.js";
import { WebClient } from "@slack/web-api";
import dotenv from "dotenv";
// 🔽 追加
import cron from "node-cron";

dotenv.config();

const postToSlack = async (token, channel, text) => {
  // 省略
};

export const postAllTodoData = async () => {
  // 省略
};

export const postTodayTodoData = async () => {
  // 省略
};

// 例：日時設定せずに 3 秒毎に実行
// cron.schedule('*/3 * * * * *', () => console.log('3秒ごとに実行'));

// 例：毎日 19 時 30 分 30 秒と 19 時 30 分 40 秒に実行
cron.schedule("30 30 19 * * *", () => postAllTodoData());
cron.schedule("40 30 19 * * *", () => postTodayTodoData());
```

## 動作確認（定期実行）

指定した時間に slack に投稿が行われれば OK！
