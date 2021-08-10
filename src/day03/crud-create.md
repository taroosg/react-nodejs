# Create の処理

## Create 処理の流れ

はじめは Create の処理を実装する．クライアントから送信されたデータを DB に保存する一連の流れを書く．

今回は`POST`メソッドでデータを送信し，新規レコードを作成する．

## ルーティングの作成

ルーティングでは，collection 名とデータを受け取り，コントローラにデータを渡す．

```js
// routes/tweet.route.js

import express from 'express';
import { readAllTweetData, createTweetData } from '../controllers/tweet.controller.js';

export const tweetRouter = express.Router();

tweetRouter.get('/', (req, res) => readAllTweetData(req, res));
// ↓追加
tweetRouter.post('/', (req, res) => createTweetData(req, res));

```

## コントローラの作成

コントローラでは，データを整理してサービスに渡す．また，サービスの処理結果を元にレスポンスを返す．

```js
// controllers/tweet.controller.js

import { getAllTweetData, insertTweetData } from '../services/tweet.service.js'

export const readAllTweetData = async (req, res, next) => {
  // 省略
};

// ↓追加
export const createTweetData = async (req, res, next) => {
  try {
    const { tweet, user_id } = req.body;
    if (!(tweet && user_id)) {
      throw new Error('something is blank');
    }
    const result = await insertTweetData({
      data: { tweet: tweet, user_id: Number(user_id) },
    });
    return res.status(200).json({
      status: 200,
      result: result,
      message: 'Succesfully post Tweet Data!',
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

```

## サービスの作成

サービスではロジックが必要な場合は記述するが，今回はデータをそのまま渡すだけ．実際に Firebase にデータを保存する処理は`repositories`レイヤーに分割する．

```js
// services/tweet.service.js

import { findAll, store } from '../repositories/tweet.repository.js';

export const getAllTweetData = async () => {
  // 省略
};

export const insertTweetData = async ({ data }) => {
  try {
    const ref = await store({ data: data });
    return {
      id: ref.id,
      data: data,
    };
  } catch (e) {
    throw Error('Error while posting Tweet Data');
  }
};

```

## リポジトリの作成

collection を指定してデータを保存する処理を実装する．ここで Firestore 関連のコードを記述する必要があるため，関連するコードを import する．

データ永続化に関するコードをリポジトリのレイヤーに閉じ込めることで，DB の種類が変更された場合にもコントローラやサービスのコードに影響ない状態にすることができる．

Firestore に関するポイントは以下のとおり．

- `created_at`と`updated_at`は Firestore の独自形式となるため，ここでユーザが送信してきたデータとマージしている．

- collection が存在しない場合は自動的に作成される．処理が実行されると，作成された Document の ID と追加データが返される．

```js
// repositories/tweet.repository.js

import admin from '../model/firebase.js';
const db = admin.firestore();

export const findAll = () => {
  return { message: 'OK' };
};

export const store = async ({ data }) => {
  try {
    const postData = {
      ...data,
      created_at: admin.firestore.Timestamp.now(),
      updated_at: admin.firestore.Timestamp.now(),
    };
    const ref = await db.collection('tweet').add(postData);
    return ref;
  } catch (e) {
    throw Error('Error while store Tweet Data');
  }
};

```

## 動作確認

処理を追加したら動作確認する．サーバを起動して下記コマンドでデータを送信し，成功のレスポンスが返ってくれば OK．

また，ブラウザで Firebase のコンソール画面から Firestore にアクセスし，送信したデータが保存されていることを確認しておく．

動作が確認できたら，2-3 件データを入れておこう．

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"tweet":"node.js","user_id":1}' localhost:3001/tweet

{
  "status": 200,
  "result": {
    "id": "XMr6sQ26x99QVvSaHewe",
    "data": {
      "tweet": "node.js",
      "user_id": 1
    }
  },
  "message": "Succesfully post Tweet Data!"
}

```

