# Read の処理（本日以前）

## 本日締切のデータだけを取得する処理

前項でデータを全件取得する処理を実装したので，次は条件をつけてデータを取得する処理を実装する．

`deadline` カラムの値を参照し，「実行した日と同じかそれ以前」のデータのみ取得する．

## ルーティングの作成

```js
// routes/todo.route.js

import express from "express";
// 🔽 編集
import {
  readAllTodoData,
  readTodayTodoData,
  createTodoData,
} from "../controllers/todo.controller.js";

export const todoRouter = express.Router();

todoRouter.get("/", (req, res) => readAllTodoData(req, res));
// 🔽 追加
todoRouter.get("/today", (req, res) => readTodayTodoData(req, res));
todoRouter.post("/", (req, res) => createTodoData(req, res));
```

## コントローラの作成

URL に付加した id を`req.params.id`で取得する．サービスに id を渡す．

```js
// controllers/todo.controller.js

import {
  getAllTodoData,
  getTodayTodoData,
  insertTodoData,
} from "../services/todo.service.js";

export const readAllTodoData = async (req, res, next) => {
  // 省略
};

// 🔽 追加
export const readTodayTodoData = async (req, res, next) => {
  try {
    const result = await getTodayTodoData();
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Successfully get Today Todo Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

export const createTodoData = async (req, res, next) => {
  // 省略
};
```

## サービスの作成

今回は特にロジックなし．

```js
// services/todo.service.js

import { findAll, findToday, store } from "../repositories/todo.repository.js";

export const getAllTodoData = async () => {
  // 省略
};

// 🔽 追加
export const getTodayTodoData = async () => {
  try {
    return await findToday();
  } catch (e) {
    throw Error("Error while getting Today Todo Data");
  }
};

export const insertTodoData = async ({ params }) => {
  // 省略
};
```

## リポジトリの作成

`deadline` のカラムで条件を指定する．

参考：[https://supabase.com/docs/reference/javascript/using-filters](https://supabase.com/docs/reference/javascript/using-filters)

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

// 🔽 追加
export const findToday = async () => {
  try {
    const { data, error } = await supabase
      .from("todo_table")
      .select()
      .lte("deadline", new Date().toISOString())
      .order("deadline", { ascending: true })
      .order("todo", { ascending: true });
    return data;
  } catch (e) {
    throw Error("Error while getting Todo Data");
  }
};

export const store = async ({ params }) => {
  // 省略
};
```

## 動作確認（1 件取得）

下記コマンドで動作をチェック．`deadline` が実行した日以前のデータが取得できれば OK．

```bash
$ curl localhost:3000/todo/today

{
  "status": 200,
  "result": [
    {
      "id": 7,
      "user_id": 2,
      "todo": "react",
      "deadline": "2021-12-04",
      "is_done": false,
      "created_at": "2021-12-06T06:31:13.794412+00:00",
      "updated_at": "2021-12-06T06:31:13.794412+00:00"
    },
    {
      "id": 6,
      "user_id": 1,
      "todo": "node.js",
      "deadline": "2021-12-06",
      "is_done": false,
      "created_at": "2021-12-06T06:30:24.275881+00:00",
      "updated_at": "2021-12-06T06:30:24.275881+00:00"
    },
    {
      "id": 1,
      "user_id": 1,
      "todo": "test",
      "deadline": "2021-12-06",
      "is_done": false,
      "created_at": "2021-12-06T06:26:57.344976+00:00",
      "updated_at": "2021-12-06T06:26:57.344976+00:00"
    },
    {
      "id": 8,
      "user_id": 1,
      "todo": "zoom meeting",
      "deadline": "2021-12-07",
      "is_done": false,
      "created_at": "2021-12-07T04:16:41.726675+00:00",
      "updated_at": "2021-12-07T04:16:41.726675+00:00"
    },
    {
      "id": 2,
      "user_id": 2,
      "todo": "Nest.js",
      "deadline": "2021-12-15",
      "is_done": true,
      "created_at": "2021-12-06T06:27:33.388245+00:00",
      "updated_at": "2021-12-06T08:27:11.024+00:00"
    }
  ],
  "message": "Successfully get Today Todo Data!"
}

```
