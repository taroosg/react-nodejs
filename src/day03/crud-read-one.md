# Read の処理（1件）

## ルーティングの作成

`:id`はユーザが付加したパラメータを指定する．コントローラで`req.params.id`で取得する．

```js
// routes/tweet.route.js

import express from "express";
import { readAllTweetData, readOneTweetData, createTweetData } from "../controllers/tweet.controller.js";

export const tweetRouter = express.Router();

tweetRouter.get("/", (req, res) => readAllTweetData(req, res));
// ↓追加
tweetRouter.get("/:id", (req, res) => readOneTweetData(req, res));
tweetRouter.post("/", (req, res) => createTweetData(req, res));

```

## コントローラの作成

URLに付加したidを`req.params.id`で取得する．

```js
// controllers/tweet.controller.js

import { getAllTweetData, getOneTweetData, insertTweetData } from "../services/tweet.service.js"

export const readAllTweetData = async (req, res, next) => {
  // 省略
};

// ↓追加
export const readOneTweetData = async (req, res, next) => {
  try {
    const { id } = req.params;
    const result = await getOneTweetData({ id: id, });
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully get One Tweet Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

export const createTweetData = async (req, res, next) => {
  // 省略
};

```

## サービスの作成

データを1件取得するにはidがあればOK．リポジトリの処理にidを渡す．

```js
// services/tweet.service.js

import { findAll, find, store } from '../repositories/tweet.repository.js';

export const getAllTweetData = async () => {
  // 省略
};

// ↓追加
export const getOneTweetData = async ({ id }) => {
  try {
    return find({ id: id });
  } catch (e) {
    throw Error("Error while getting One Tweet Data");
  }
};

export const insertTweetData = async ({ data }) => {
  // 省略
};

```

## リポジトリの作成

```js
// repositories/tweet.repository.js

import admin from "../model/firebase.js";
const db = admin.firestore();

export const findAll = async () => {
  // 省略
};

// ↓追加
export const find = async ({ id }) => {
  try {
    const tweetSnapshot = await db.collection("tweet").doc(id).get();
    return {
      id: tweetSnapshot.id,
      data: {
        ...tweetSnapshot.data(),
        created_at: tweetSnapshot.data().created_at.toDate(),
        updated_at: tweetSnapshot.data().updated_at.toDate(),
      }
    }
  } catch (e) {
    throw Error("Error while getting One tweet Data");
  }
};

export const store = async ({ data }) => {
  // 省略
};

```

## 動作確認

下記コマンドで動作をチェック．全件取得した中から適当なidを入力し，該当のデータが取得できればOK．

```bash
$ curl localhost:3002/tweet/1JXLilqdOqU7rCrwjEpA

{
  "status": 200,
  "result": {
    "id": "1JXLilqdOqU7rCrwjEpA",
    "data": {
      "updated_at": "2021-07-29T11:38:38.006Z",
      "user_id": 1,
      "created_at": "2021-07-29T11:38:38.005Z",
      "tweet": "node.js"
    }
  },
  "message": "Succesfully get One Tweet Data!"
}

```

