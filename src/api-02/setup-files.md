# ファイル作成と動作確認

## ルーティング表

これから実装していく機能は以下のとおり．基本的な CRUD 処理を実装する．

| URI           | method | name    | description          |
| ------------- | ------ | ------- | -------------------- |
| `/todo`       | GET    | findAll | データ全件取得       |
| `/todo/today` | GET    | find    | 本日締切のデータ取得 |
| `/todo`       | POST   | store   | データ作成処理       |
| `/todo/:id`   | PUT    | update  | データ更新処理       |
| `/todo/:id`   | DELETE | destroy | データ削除処理       |

## 役割分担

前回講義の役割分担表も参照．今回は repository に DB 関連の処理を実装する．

repository に DB 関連の処理を任せることで，DB が変更された場合でも他のコードの影響せずに運用することができる．このような役割分担をリポジトリパターンと呼ぶ．

| 項目         | 意味合い                                                            |
| ------------ | ------------------------------------------------------------------- |
| routes       | URI と実行する処理の対応．                                          |
| controllers  | リクエストパラメータの検証，レスポンス送信．                        |
| services     | リクエストに対する処理のメインロジックを記述．                      |
| repositories | DB 関連の処理を記述．今回は Supabase とやり取りする処理を記述する． |

## 既存ファイルへの追記と新規ファイルの準備

`app.js`にルーティングを追記する．

```js
// app.js

import express from "express";
import { omikujiRouter } from "./routes/omikuji.route.js";
import { jankenRouter } from "./routes/janken.route.js";
// 🔽 追加
import { todoRouter } from "./routes/todo.route.js";

const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
const port = 3000;

app.get("/", (req, res) => {
  res.json({
    uri: "/",
    message: "Hello Node.js!",
  });
});

app.use("/omikuji", (req, res) => omikujiRouter(req, res));
app.use("/janken", (req, res) => jankenRouter(req, res));

// 🔽 追加
app.use("/todo", (req, res) => todoRouter(req, res));

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

## 必要なファイルの作成

新しく以下のファイルを作成する．前回の役割に加えて，今回は DB 関連の処理が必要となるため `repositories` フォルダを作成してファイルを追加する．

- `routes/todo.route.js`
- `controllers/todo.controller.js`
- `services/todo.service.js`
- `repositories/todo.repository.js`

まずそれぞれのファイルが連携できることを確認するため，DB と接続せずに固定のデータを返す処理を実装する．

## ルーティングの作成

まずはルーティングを作成．

```js
// routes/todo.route.js

import express from "express";
import { readAllTodoData } from "../controllers/todo.controller.js";

export const todoRouter = express.Router();

todoRouter.get("/", (req, res) => readAllTodoData(req, res));
```

## コントローラの作成

コントローラではリクエストとレスポンスを定義．

```js
// controllers/todo.controller.js

import { getAllTodoData } from "../services/todo.service.js";

export const readAllTodoData = async (req, res, next) => {
  try {
    const result = await getAllTodoData();
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Successfully get All Todo Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};
```

## サービスの作成

サービスではリポジトリの関数（`findAll()`）を呼び出す．

```js
// services/todo.service.js

import { findAll } from "../repositories/todo.repository.js";

export const getAllTodoData = async () => {
  try {
    return findAll();
  } catch (e) {
    throw Error("Error while getting All Todo Data");
  }
};
```

## リポジトリの作成

リポジトリは一旦決まったメッセージを返す．

```js
// repositories/todo.repository.js

export const findAll = () => {
  return { message: "OK" };
};
```

## 動作確認

動作確認する．以下のコマンドでレスポンスが返ってくれば OK．

```bash
$ curl localhost:3000/todo

{
  "status": 200,
  "result": {
    "message": "OK"
  },
  "message": "Successfully get All Todo Data!"
}

```

一通りの動作が確認できたら，続いて Supabase の CRUD 処理を作成していく．
