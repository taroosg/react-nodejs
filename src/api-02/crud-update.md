# Update の処理

## データ更新の処理

既存のデータを上書きする処理を実装する．`PUT` 形式でデータを送信する．

> **Key Point**💡
>
> `PUT`形式は`GET`と`POST`が合わさったような形式で，`req.params.id` で id を送信し，同時に`req.body`で上書きするデータを送信する．
>
> 今回は使用しないが，`GET` でも同様に `req.params.id` で id を送信して受け取ることができる．

## ルーティングの作成

update のルーティングを追加．

id 指定する．`/hoge`に`PUT`でリクエストを送信した場合，`req.params.id`は`hoge`になる．

```js
// routes/todo.route.js

import express from "express";
// 🔽 編集
import {
  readAllTodoData,
  readTodayTodoData,
  createTodoData,
  editTodoData,
} from "../controllers/todo.controller.js";

export const todoRouter = express.Router();

todoRouter.get("/", (req, res) => readAllTodoData(req, res));
todoRouter.get("/today", (req, res) => readTodayTodoData(req, res));
todoRouter.post("/", (req, res) => createTodoData(req, res));
// 🔽 追加
todoRouter.put("/:id", (req, res) => editTodoData(req, res));
```

## コントローラの作成

コントローラでは，リクエストから `更新対象のドキュメントのid` と `更新データ` の 2 つを受け取る．送信されたデータの中から，これら 2 つのデータを抽出し，サービスに渡す．

```js
// controllers/todo.controller.js

// 🔽 編集
import {
  getAllTodoData,
  getTodayTodoData,
  insertTodoData,
  updateTodoData,
} from "../services/todo.service.js";

export const readAllTodoData = async (req, res, next) => {
  //  省略
};

export const readTodayTodoData = async (req, res, next) => {
  // 省略
};

export const createTodoData = async (req, res, next) => {
  // 省略
};

// 🔽 追加
export const editTodoData = async (req, res, next) => {
  try {
    const { id } = req.params;
    const { user_id, todo, deadline, is_done } = req.body;
    if (!(id && user_id && todo && deadline && is_done)) {
      throw new Error("something is blank");
    }
    const result = await updateTodoData({
      id: id,
      params: {
        user_id: user_id,
        todo: todo,
        deadline: deadline,
        is_done: is_done,
      },
    });
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Successfully edit Todo Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};
```

## サービスの作成

送信されたデータを渡すのみなのでこれまでの処理と同様．

```js
// services/todo.service.js

import {
  findAll,
  findToday,
  store,
  update,
} from "../repositories/todo.repository.js";

export const getAllTodoData = async () => {
  // 省略
};

export const getTodayTodoData = async () => {
  // 省略
};

export const insertTodoData = async ({ params }) => {
  // 省略
};

// 🔽 追加
export const updateTodoData = async ({ id, params }) => {
  try {
    return await update({ id, params });
  } catch (e) {
    throw Error("Error while updating Todo Data");
  }
};
```

## リポジトリの作成

リポジトリでは，受け取ったデータで DB を更新する．同時に `updated_at` に実行日時を設定して送信する．

`update()` メソッドが用意されているのでこれを使用すれば OK．

参考：[https://supabase.com/docs/reference/javascript/update](https://supabase.com/docs/reference/javascript/update)

```js
// repositories/todo.repository.js

import dotenv from "dotenv";
import { createClient } from "@supabase/supabase-js";

dotenv.config();

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_API_KEY
);

export const findAll = async () => {
  // 省略
};

export const findToday = async () => {
  // 省略
};

export const store = async ({ params }) => {
  // 省略
};

// 🔽 追加
export const update = async ({ id, params }) => {
  try {
    const { data, error } = await supabase
      .from("todo_table")
      .update({ ...params, updated_at: new Date().toISOString() })
      .match({ id: id });
    return data;
  } catch (e) {
    throw Error("Error while updating Todo Data");
  }
};
```

## 動作確認（更新）

動作確認する．更新前に適当なデータを確認し，更新状態が確認できるようにしておこう．

コンソール画面 or 前項の Read 処理でデータを確認し，データが更新されていれば OK！

```bash
$ curl -X PUT -H "Content-Type: application/json" -d '{"todo":"Nest.js","user_id":2,"deadline":"2021-12-20","is_done":true}' localhost:3000/todo/2

{
  "status": 200,
  "result": [
    {
      "id": 2,
      "user_id": 2,
      "todo": "Nest.js",
      "deadline": "2021-12-20",
      "is_done": true,
      "created_at": "2021-12-06T06:27:33.388245+00:00",
      "updated_at": "2021-12-16T06:56:33.618+00:00"
    }
  ],
  "message": "Successfully edit Todo Data!"
}

```
