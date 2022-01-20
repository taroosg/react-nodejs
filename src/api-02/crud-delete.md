# Delete の処理

## データ削除の処理

id を指定して該当するデータを DB から削除する．テーブルと id を指定すれば OK．

## ルーティングの作成

削除のルーティングを追加．更新の場合と同様にパラメータを受け取る．

```js
// routes/todo.route.js

import express from "express";
// 🔽 編集
import {
  readAllTodoData,
  readTodayTodoData,
  createTodoData,
  editTodoData,
  deleteTodoData,
} from "../controllers/todo.controller.js";

export const todoRouter = express.Router();

todoRouter.get("/", (req, res) => readAllTodoData(req, res));
todoRouter.get("/today", (req, res) => readTodayTodoData(req, res));
todoRouter.post("/", (req, res) => createTodoData(req, res));
todoRouter.put("/:id", (req, res) => editTodoData(req, res));
// 🔽 追加
todoRouter.delete("/:id", (req, res) => deleteTodoData(req, res));
```

## コントローラの作成

コントローラでは id を受け取り，サービスの処理を実行する．

```js
// controllers/todo.controller.js

// 🔽 編集
import {
  getAllTodoData,
  getTodayTodoData,
  insertTodoData,
  updateTodoData,
  destroyTodoData,
} from "../services/todo.service.js";

export const readAllTodoData = async (req, res, next) => {
  // 省略
};

export const readTodayTodoData = async (req, res, next) => {
  // 省略
};

export const createTodoData = async (req, res, next) => {
  // 省略
};

export const editTodoData = async (req, res, next) => {
  // 省略
};

// 🔽 追加
export const deleteTodoData = async (req, res, next) => {
  try {
    const { id } = req.params;
    if (!id) {
      throw new Error("something is blank");
    }
    const result = await destroyTodoData({
      id: id,
    });
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Successfully delete Todo Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};
```

## サービスの作成

サービスでは id 指定してリポジトリで定義した関数を実行する．

```js
// services/todo.service.js

// 🔽 編集
import {
  findAll,
  findToday,
  store,
  update,
  destroy,
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

export const updateTodoData = async ({ id, params }) => {
  // 省略
};

// 🔽 追加
export const destroyTodoData = async ({ id }) => {
  try {
    return await destroy({ id: id });
  } catch (e) {
    throw Error("Error while deleting Todo Data");
  }
};
```

## リポジトリの作成

リポジトリでは DB からデータを削除する．id があればデータを指定して削除することができる．

参考：[https://supabase.com/docs/reference/javascript/delete](https://supabase.com/docs/reference/javascript/delete)

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

export const update = async ({ id, params }) => {
  // 省略
};

// 🔽 追加
export const destroy = async ({ id }) => {
  try {
    const { data, error } = await supabase
      .from("todo_table")
      .delete()
      .match({ id: id });
    return data;
  } catch (e) {
    throw Error("Error while deleting Todo Data");
  }
};
```

## 動作確認（削除）

動作確認する．document 名 は既存のデータから適当に指定する．

コンソール画面 or 前項の Read 処理でデータを確認し，データが削除されていれば OK！

```bash
$ curl -X DELETE localhost:3000/todo/13

{
  "status": 200,
  "result": [
    {
      "id": 13,
      "user_id": 1,
      "todo": "node.js",
      "deadline": "2021-12-31",
      "is_done": false,
      "created_at": "2021-12-16T06:21:31.592284+00:00",
      "updated_at": "2021-12-16T06:21:31.592284+00:00"
    }
  ],
  "message": "Successfully delete Todo Data!"
}

```
