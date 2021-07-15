# API実装3（じゃんけん）

おみくじの実装ができたので，ユーザ側からデータを送信してじゃんけんの結果を返す API を実装してみる．

## アプリケーションの全体像

ユーザはリクエスト時に`post`メソッドで自分の手を送信する．サーバ側はユーザから送信された手に対してランダムで手を出し，「ユーザの手」「サーバの手」「勝敗」をレスポンスで返却する．

下図を参考に，必要なファイルを作成する．

```bash
.
├── app.js
├── controllers
│   ├── janken.controller.js
│   └── omikuji.controller.js
├── node_modules
├── package.json
├── package-lock.json
├── routes
│   ├── janken.route.js
│   └── omikuji.route.js
└── services
    ├── janken.service.js
    └── omikuji.service.js

```


## 各ファイルの実装

`app.js`を以下のように編集する．

じゃんけんのルーティングを読み込む．また，POST メソッドでデータを受け取るためには`express.urlencoded({ extended: true })`とJSONデータの扱いで`express.json()`が必要になるため読み込んでいる．

```js
// app.js

import express from "express";
import { omikujiRouter } from "./routes/omikuji.route.js";
// じゃんけんのルーティングを読み込む
import { jankenRouter } from "./routes/janken.route.js";

const app = express();
// ↓POSTでデータを受け取るために必要
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
// じゃんけんのルーティングを追加
app.use("/janken", (req, res) => jankenRouter(req, res));

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});

```


## ルーティングの実装

`routes/janken.route.js`に以下の内容を記述する．

URI と対応するコントローラの処理を記述する．今回は POST で受け取る点と送信されてきたデータが`req`に入っている点に注意．

```js
// routes/janken.route.js

import express from "express";
import { getResult } from "../controllers/janken.controller.js";

export const jankenRouter = express.Router();

jankenRouter.post("/", (req, res) => getResult(req, res));

```


## コントローラの実装

`controllers/janken.controller.js`に以下の内容を記述する．

おみくじの場合とほぼ同じ．

```js
// controllers/janken.controller.js

import { getJanken } from "../services/janken.service.js";

export const getResult = async (req, res, next) => {
  try {
    const result = await getJanken(req.body);
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully get Janken!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

```


## サービスの実装

`services/janken.service.js`に以下の内容を記述する．

とりあえず毎回同じ結果を返している．

```js
// services/janken.service.js

export const getJanken = async (query) => {
  try {
    return { yourHand: query.myhand, comHand: "グー", result: "テスト中" };
  } catch (e) {
    throw Error("Error while getting Janken");
  }
};

```


## 動作確認

サーバを立ち上げて，`curl`コマンドでリクエストを送信する．POST する場合は下記の書式で実行する．

結果が返ってくれば成功．

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"パー"}' localhost:3001/janken

{"status":200,"result":{"yourHand":"パー","comHand":"グー","result":"テスト中"},"message":"Succesfully get Janken!"}
```

サーバに対して`{"myhand":"パー"}`という JSON 形式のデータを POST メソッドで送信している．


## 【演習】じゃんけんの実装

上記で記述した`services/janken.service.js`にじゃんけんのロジックを実装してみよう．

サーバを立ち上げて，以下のようにじゃんけんができれば OK！

```js
$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"グー"}' localhost:3001/janken

{"status":200,"result":{"yourHand":"グー","comHand":"チョキ","result":"Win"},"message":"Succesfully get Janken!"}

$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"チョキ"}' localhost:3001/janken

{"status":200,"result":{"yourHand":"チョキ","comHand":"パー","result":"Win"},"message":"Succesfully get Janken!"}

$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"パー"}' localhost:3001/janken

{"status":200,"result":{"yourHand":"パー","comHand":"チョキ","result":"Lose"},"message":"Succesfully get Janken!"}

```

じゃんけんができたら，グーチョキパー以外の手を送信すると NG なメッセージが返す実装にもチャレンジ！

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"無敵のアレ"}' localhost:3001/janken

{"status":200,"result":{"message":"Invalid hand..."},"message":"Succesfully get Janken!"}

```

実装例：

```js
// services/janken.service.js

export const getJanken = async (query) => {
  try {
    const hand = ["グー", "チョキ", "パー"];
    const myIndex = hand.indexOf(query.myhand);
    if (myIndex === -1) return { message: "Invalid hand..." };
    const comIndex = Math.floor(Math.random() * 3);
    const resultSheet = [
      ["Draw", "Win", "Lose"],
      ["Lose", "Draw", "Win"],
      ["Win", "Lose", "Draw"],
    ];
    return {
      yourHand: query.myhand,
      comHand: hand[comIndex],
      result: resultSheet[myIndex][comIndex],
    };
  } catch (e) {
    throw Error("Error while getting Janken");
  }
};

```
