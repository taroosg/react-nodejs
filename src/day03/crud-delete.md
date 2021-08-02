# Delete の処理

## データ削除の処理

id を指定して該当するデータを DB から削除する．コレクション名とドキュメント id がわかっていればOK．

## ルーティングの作成

削除のルーティングを追加．更新の場合と同様にパラメータを受け取る．

```js
// routes/tweet.route.js

import express from 'express';
import { readAllTweetData, readOneTweetData, createTweetData, editTweetData, deleteTweetData } from '../controllers/tweet.controller.js';

export const tweetRouter = express.Router();

tweetRouter.get('/', (req, res) => readAllTweetData(req, res));
tweetRouter.get('/:id', (req, res) => readOneTweetData(req, res));
tweetRouter.post('/', (req, res) => createTweetData(req, res));
tweetRouter.put('/:id', (req, res) => editTweetData(req, res));
// ↓追加
tweetRouter.delete('/:id', (req, res) => deleteTweetData(req, res));

```

## コントローラの作成

コントローラでは document 名（id）を受け取り，サービスの処理を実行する．

```js
// controllers/tweet.controller.js

import { getAllTweetData, getOneTweetData, insertTweetData, updateTweetData, destroyTweetData } from '../services/tweet.service.js';

export const readAllTweetData = async (req, res, next) => {
  // 省略
};

export const readOneTweetData = async (req, res, next) => {
  // 省略
};

export const createTweetData = async (req, res, next) => {
  // 省略
};

export const editTweetData = async (req, res, next) => {
  // 省略
};

// ↓追加
export const deleteTweetData = async (req, res, next) => {
  try {
    const { id } = req.params;
    if (!id) {
      throw new Error('something is blank');
    }
    const result = await destroyTweetData({
      id: id,
    });
    return res.status(200).json({
      status: 200,
      result: result,
      message: 'Succesfully delete Tweet Data!',
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

```

## サービスの作成

サービスではid指定してリポジトリで定義した関数を実行する．

```js
// services/tweet.service.js

import { findAll, find, store, update, destroy } from '../repositories/tweet.repository.js';

export const getAllTweetData = async () => {
  // 省略
};

export const getOneTweetData = async ({ id }) => {
  // 省略
};

export const insertTweetData = async ({ data }) => {
  // 省略
};

export const updateTweetData = async ({ id, data }) => {
  // 省略
};

// ↓追加
export const destroyTweetData = async ({ id }) => {
  try {
    return await destroy({ id: id, });
  } catch (e) {
    throw Error('Error while deleting Tweet Data');
  }
};

```

## リポジトリの作成

リポジトリでは DB からデータを削除する．collection 名と document 名があればデータを指定して削除することができる．

```js
// repositories/tweet.repository.js

import admin from '../model/firebase.js';
const db = admin.firestore();

export const findAll = async () => {
  // 省略
};

export const find = async ({ id }) => {
  // 省略
};

export const store = async ({ data }) => {
  // 省略
}

export const update = async ({ id, data }) => {
  // 省略
};

// ↓追加
export const destroy = async ({ id }) => {
  try {
    const ref = await db.collection('tweet').doc(id).delete();
    return { id: id, };
  } catch (e) {
    throw Error('Error while deleting Tweet Data');
  }
};
```

## 動作確認（削除）

動作確認する．document 名 は既存のデータから適当に指定する．

コンソール画面 or 前項の Read 処理でデータを確認し，データが削除されていれば OK！

```bash
$ curl -X DELETE localhost:3001/tweet/TPev9ejBQd7WkxWRuNKk

{
  "status": 200,
  "result": {
    "id": "TPev9ejBQd7WkxWRuNKk"
  },
  "message": "Succesfully delete Tweet Data!"
}

```
