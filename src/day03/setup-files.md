# ファイル作成と動作確認

## ルーティング表

これから実装していく機能は以下のとおり．基本的なCRUD処理を実装する．

|URI|method|name|description|
|-|-|-|-|
|`/tweet`|GET|findAll|データ全件取得|
|`/tweet/:id`|GET|find|データ1件取得|
|`/tweet`|POST|store|データ作成処理|
|`/tweet/:id`|PUT|update|データ更新処理|
|`/tweet/:id`|DELETE|destroy|データ削除処理|

## 役割分担

前回講義の役割分担表も参照．今回は repository に DB 関連の処理を実装する．

repository に DB 関連の処理を任せることで，DB が変更された場合でも他のコードの影響せずに運用することができる．このような役割分担をリポジトリパターンと呼ぶ．


|項目|意味合い|
|-|-|
|routes|URI と実行する処理の対応．|
|controllers| リクエストパラメータの検証，レスポンス送信．|
|services|リクエストに対する処理のメインロジックを記述．|
|repositories|DB関連の処理を記述．今回は Firestore Database とやり取りする処理を記述する．|


## 既存ファイルへの追記と新規ファイルの準備

`app.js`にルーティングを追記する．

```js
// app.js

import express from 'express';
import { omikujiRouter } from './routes/omikuji.route.js';
import { jankenRouter } from './routes/janken.route.js';
// ↓追加
import { tweetRouter } from './routes/tweet.route.js';

const app = express();

const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
const port = 3001;

app.get('/', (req, res) => {
  res.json({
    uri: '/',
    message: 'Hello Node.js!',
  });
});

app.use('/omikuji', (req, res) => omikujiRouter(req, res));
app.use('/janken', (req, res) => jankenRouter(req, res));
// ↓追加
app.use('/tweet', (req, res) => tweetRouter(req, res));

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});

```

## 必要なファイルの作成

新しく以下のファイルを作成する．前回の役割に加えて，今回は DB 関連の処理が必要となるため `repositories` フォルダを作成してファイルを追加する．

- `routes/tweet.route.js`
- `controllers/tweet.controller.js`
- `services/tweet.service.js`
- `repositories/tweet.repository.js`

まずそれぞれのファイルが連携できることを確認するため，DB と接続せずに固定のデータを返す処理を実装する．

## ルーティングの作成

まずはルーティングを作成．

```js
// routes/tweet.route.js

import express from 'express';
import { readAllTweetData } from '../controllers/tweet.controller.js';

export const tweetRouter = express.Router();

tweetRouter.get('/', (req, res) => readAllTweetData(req, res));

```

## コントローラの作成

コントローラではリクエストとレスポンスを定義．

```js
// controllers/tweet.controller.js

import { getAllTweetData } from '../services/tweet.service.js';

export const readAllTweetData = async (req, res, next) => {
  try {
    const result = await getAllTweetData();
    return res.status(200).json({
      status: 200,
      result: result,
      message: 'Succesfully get All Tweet Data!',
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

```

## サービスの作成

サービスではリポジトリの関数（`findAll()`）を呼び出す．

```js
// services/tweet.service.js

import { findAll } from '../repositories/tweet.repository.js';

export const getAllTweetData = async () => {
  try {
    return findAll();
  } catch (e) {
    throw Error('Error while getting All Tweet Data');
  }
};

```

## リポジトリの作成

リポジトリは一旦決まったメッセージを返す．

```js
// repositories/tweet.repository.js

export const findAll = () => {
  return { message: 'OK' };
};

```

## 動作確認

動作確認する．以下のコマンドでレスポンスが返ってくれば OK．

```bash
$ curl localhost:3001/tweet

{
  "status": 200,
  "result": {
    "message": "OK"
  },
  "message": "Succesfully get All Tweet Data!"
}

```

一通りの動作が確認できたら，続いて Firestore の CRUD 処理を作成していく．
