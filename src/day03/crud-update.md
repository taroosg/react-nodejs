# Update の処理

## データ更新の処理

既存のデータを上書きする処理を実装する．`PUT`形式でデータを送信する．

>**Key Point**💡
>
>`PUT`形式は`GET`と`POST`が合わさったような形式で，`req.params.id`で id を送信し，同時に`req.body`で上書きするデータを送信する．

## ルーティングの作成

update のルーティングを追加．

GETの場合と同様にid指定する．`/hoge`に`PUT`でリクエストを送信した場合，`req.params.id`は`hoge`になる．

```js
// routes/tweet.route.js

import express from 'express';
import { readAllTweetData, readOneTweetData, createTweetData, editTweetData } from '../controllers/tweet.controller.js';

export const tweetRouter = express.Router();

tweetRouter.get('/', (req, res) => readAllTweetData(req, res));
tweetRouter.get('/:id', (req, res) => readOneTweetData(req, res));
tweetRouter.post('/', (req, res) => createTweetData(req, res));
// ↓追加
tweetRouter.put('/:id', (req, res) => editTweetData(req, res));

```

## コントローラの作成

コントローラでは，リクエストから`更新対象のドキュメントのid`と`更新データ`の 2 つを受け取る．送信されたデータの中から，これら 2 つのデータを抽出し，サービスに渡す．

```js
// controllers/tweet.controller.js

import { getAllTweetData, getOneTweetData, insertTweetData, updateTweetData } from '../services/tweet.service.js';

export const readAllTweetData = async (req, res, next) => {
  // 省略
};

export const readOneTweetData = async (req, res, next) => {
  // 省略
};

export const createTweetData = async (req, res, next) => {
  // 省略
};

// ↓追加
export const editTweetData = async (req, res, next) => {
  try {
    const { id } = req.params;
    const { tweet, user_id } = req.body;
    if (!(id && tweet && user_id)) {
      throw new Error('something is blank');
    }
    const result = await updateTweetData({
      id: id,
      data: { tweet: tweet, user_id: user_id },
    });
    return res.status(200).json({
      status: 200,
      result: result,
      message: 'Succesfully edit Tweet Data!',
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

```

## サービスの作成

送信されたデータを渡すのみなのでこれまでの処理と同様．

```js
// services/tweet.service.js

import { findAll, find, store, update } from '../repositories/tweet.repository.js';

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
  try {
    return await update({ id, data });
  } catch (e) {
    throw Error('Error while updating Tweet Data');
  }
};

```

## リポジトリの作成

リポジトリでは，受け取ったデータで DB を更新する．`deadline`を Firestore の形式に変換し，同時に`updated_at`に実行日時を設定して送信する．

collection 名と document 名を指定して`update()`でデータを更新できる．実行完了後には，更新ドキュメントの id と更新データを返す．

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

// ↓追加
export const update = async ({ id, data }) => {
  const updateData = {
    ...data,
    updated_at: admin.firestore.Timestamp.now(),
  };
  const ref = await db.collection('tweet').doc(id).update(updateData);
  return {
    id: id,
    data: updateData,
  };
};

```

## 動作確認（更新）

動作確認する．document は既存のデータから適当に指定する．Read の処理結果などから存在する document 名 を確認しておこう．

コンソール画面 or 前項の Read 処理でデータを確認し，データが更新されていれば OK！

```bash
$ curl -X PUT -H "Content-Type: application/json" -d '{"tweet":"Nest.js","user_id":2}' localhost:3001/tweet/1JXLilqdOqU7rCrwjEpA

{
  "status": 200,
  "result": {
    "id": "1JXLilqdOqU7rCrwjEpA",
    "data": {
      "tweet": "Nest.js",
      "user_id": 2,
      "updated_at": {
        "_seconds": 1627610411,
        "_nanoseconds": 470000000
      }
    }
  },
  "message": "Succesfully edit Tweet Data!"
}


```
