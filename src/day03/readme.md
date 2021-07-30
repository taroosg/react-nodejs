# Day03


## todo

- やることの説明
- リポジトリパターンの説明
- repositoryフォルダとファイルの作成を追記
- データ渡しの確認
- create
- read
- update
- delete


## 今回のゴール

- Cloud Firestore を用いたデータの永続化を実装する．
- 「twitter のようななにか」を実装し，CRUD 処理とデータの構造を把握する．
- 外部データ取得 -> データ保存の流れを実装し，複数の処理を連携させる感覚を掴む．

## Node.js におけるデータ永続化

データの永続化とは，アプリケーションの状態によらずデータを保持することである．言い換えると，DB などの外部にデータを保存すること．

データの永続化に使用する外部 DB は次のようなものがある．Node.js ではいずれの DB も利用できるが，RDB であれば MySQL か PostgreSQL，NoSQL では MongoDB，Firebase，Redis などがよく用いられる．

## RDB

SQL を用いてデータ管理を行う DB．柔軟な集計や結合ができ，情報も多い．

基本的には同じ SQL を記述すれば同じ動作が実行されるが，それぞれの DB で仕様が異なる部分もあるので使用前は要チェックすべし．

できることは非常に幅広く，データを扱う上でできないことはほぼないと考えて良い．

### 例

- MySQL
- PostgreSQL
- OracleDB

## NoSQL

Not Only SQL．SQL を前提としたデータ構造に縛られない DB．「値」およびそれを取得するための「キー」だけを格納できる Key-Value 型の DB などが代表的だ．

下記の例においてもそれぞれデータ構造が異なるため，プロジェクトの目的に合致した DB を選択する必要がある．

RDB と比較して後発のためにより直感的に扱えるものが多い一方で，データの格納および取得が高度に最適化されているが故に，機能性を最小限にしているものもある．

- MongoDB
- Firebase Realtime DB
- Firebase Cloud Firestore
- Redis
- DynamoDB
- Neo4j


## Cloud Firestore の準備

今回は NoSQL である Firebase Cloud Firestore を用いて CRUD 処理を実装してみる．

## DB の作成

- Firebase のコンソールにアクセスし，今回のプロジェクトのページを表示．
- 左側メニューの「Cloud Firestore」をクリックし，DB を作成する．必ず「テストモード」にチェックを入れて作成すること．リージョンは任意．

## JSON ファイルのダウンロード

Node.js で CloudFirestore を操作するには，設定ファイルを用意する必要がある．Firebase で適当なプロジェクトを作成したら，下記の手順で必要なファイルをダウンロードする．

- Firebase のコンソールにアクセスし，今回のプロジェクトのページを表示．
- ⚙ -> プロジェクトを設定 で設定画面を表示．
- 「サービスアカウント」タブ -> 「サービスアカウントを作成」ボタン -> 「新しい秘密鍵の作成」ボタン -> 「キーの生成」ボタンの順にクリック．
- 適当な場所に json ファイルを保存する．
- コンソール画面は開いたままにしておこう．

## JSON ファイルの配置と構成ファイルの作成

- プロジェクト直下に`model`ディレクトリを作成する．`model`ディレクトリの中に ↑ でダウンロードした json ファイルを移動する．
- `model`ディレクトリの中に`firebase.js`ファイルを作成する．
- 下記の内容を記述する．
- `const serviceAccount = require('...');`の部分の`require()`内を json ファイルのパスに書き換える．

`firebase.js`は以下のような状態．

```js
import admin from 'firebase-admin';

import { createRequire } from 'module'
const require = createRequire(import.meta.url)
const serviceAccount = require('./hoge-firebase-adminsdk-fuga-piyo.json')

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

export default admin;

```

## `.gitignore`に追記

下記を追記しておく．ダウンロードした JSON ファイルには Firebase プロジェクトの情報が含まれているためである．

サーバにデプロイする場合にも必要になるため，その場合は環境変数などを使用して工夫する必要がある．

```
/node_modules
/model/hoge-firebase-adminsdk-fuga-piyo.json
```

## 必要なパッケージのインストール

下記コマンドでインストールする．必ずプロジェクトのディレクトリで行うこと．

```bash
$ npm i firebase-admin

+ firebase-admin@9.5.0
added 120 packages from 109 contributors and audited 230 packages in 21.873s
```


## 既存ファイルへの追記と新規ファイルの準備

`app.js`にルーティングを追記する．

```js
// importに書き換え
import express from "express";
import { omikujiRouter } from "./routes/omikuji.route.js";
import { jankenRouter } from "./routes/janken.route.js";
import { tweetRouter } from "./routes/tweet.route.js";

const app = express();

const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
const port = 3001;

const omikujiRouter = require("./routes/omikuji.route");
const jankenRouter = require("./routes/janken.route");
// ↓追加
const tweetRouter = require("./routes/tweet.route");

app.get("/", (req, res) => {
  res.json({
    uri: "/",
    message: "Hello Node.js!",
  });
});

app.use("/omikuji", (req, res) => omikujiRouter(req, res));
app.use("/janken", (req, res) => jankenRouter(req, res));
// ↓追加
app.use("/tweet", (req, res) => tweetRouter(req, res));

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});

```

新しく以下のファイルを作成する．

- `routes/tweet.route.js`
- `controllers/tweet.controller.js`
- `services/tweet.service.js`
- `repositories/tweet.repository.js`

作成したら以下の内容を記述する．

まずはルーティングを作成．

```js
// routes/tweet.route.js
import express from "express";
import { readAllTweetData } from "../controllers/tweet.controller.js";

export const tweetRouter = express.Router();

tweetRouter.get("/", (req, res) => readAllTweetData(req, res));

```

コントローラではリクエストとレスポンスを定義．

```js
// controllers/tweet.controller.js
import { getAllTweetData, insertTodoData } from "../services/tweet.service.js"

export const readAllTweetData = async (req, res, next) => {
  try {
    const result = await getAllTweetData();
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully get All Tweet Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

```

サービスではリポジトリの関数を呼び出す．

```js
// services/tweet.service.js
import { findAll } from '../repositories/tweet.repository.js';

export const getAllTweetData = async () => {
  try {
    return findAll();
  } catch (e) {
    throw Error("Error while getting All Tweet Data");
  }
};

```

リポジトリは一旦決まったメッセージを返す．

```js
// repositories/tweet.repository.js
export const findAll = () => {
  return { message: "OK" };
};

```

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


## CRUD 処理の作成

一通りの動作が確認できたら，Firestore の CRUD 処理を作成していく．

ルーティング表

|URI|method|name|description|
|-|-|-|-|
|`/tweet`|GET|findAll|データ全件取得|
|`/tweet/:id`|GET|find|データ1件取得|
|`/tweet`|POST|store|データ作成処理|
|`/tweet/:id`|PUT|update|データ更新処理|
|`/tweet/:id`|DELETE|delete|データ削除処理|

### Create の処理

今回は`POST`メソッドでデータを送信し，新規レコードを作成する．

ルーティングでは，collection 名とデータを受け取り，コントローラにデータを渡す．

```js
// routes/todo.route.js
import express from "express";
import { readAllTweetData, createTweetData } from "../controllers/tweet.controller.js";

export const tweetRouter = express.Router();

tweetRouter.get("/", (req, res) => readAllTweetData(req, res));
// ↓追加
tweetRouter.post("/", (req, res) => createTweetData(req, res));

```

コントローラでは，データを整理してサービスに渡す．また，サービスの処理結果を元にレスポンスを返す．

```js
// controllers/todo.controller.js
import { getAllTweetData, insertTweetData } from "../services/tweet.service.js"

export const readAllTweetData = async (req, res, next) => {
  // 省略
};

// ↓追加
export const createTweetData = async (req, res, next) => {
  try {
    const { tweet, user_id } = req.body;
    if (!(tweet && user_id)) {
      throw new Error("something is blank");
    }
    const result = await insertTweetData({
      data: { tweet: tweet, user_id: user_id },
    });
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully post Tweet Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

```

サービスではロジックが必要な場合は記述するが，今回はデータをそのまま渡すだけ．実際に Firebase にデータを保存する処理は`repositories`レイヤーに分割する．

collection が存在しない場合は自動的に作成される．処理が実行されると，作成された Document の ID と追加データが返される．

```js
// services/todo.service.js

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
    throw Error("Error while posting Tweet Data");
  }
};

```

リポジトリにデータを保存する処理を作成する．





このへん説明追加する




```js
// repositories/tweet.repository.js

import admin from "../model/firebase.js";
const db = admin.firestore();

export const findAll = () => {
  return { message: "OK" };
};

export const store = async ({ data }) => {
  try {
    const postData = {
      ...data,
      created_at: admin.firestore.Timestamp.now(),
      updated_at: admin.firestore.Timestamp.now(),
    };
    const ref = await db.collection("tweet").add(postData);
    return ref;
  } catch (e) {
    throw Error("Error while store Tweet Data");
  }
};

```

処理を追加したら動作確認する．サーバを起動して下記コマンドでデータを送信し，成功のレスポンスが返ってくれば OK．

また，Firebase のコンソール画面から Cloud Firestore にアクセスし，送信したデータが保存されていることを確認しておく．

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

### Read の処理

### データ全件取得の処理

Read の処理では，ルーティングとコントローラとサービスははじめにつくったものを使用する．

リポジトリに以下の内容を記述する．collection 名を指定してデータをすべて取得する．日付部分は独自形式なので，データ取得後に`Date`形式に変換している．

`tweetSnapshot`は取得したデータそのままで使いにくいので，必要な部分を取り出して`tweets`に入れている．

```js
// repositories/todo.repository.js
import admin from "../model/firebase.js";
const db = admin.firestore();

export const findAll = async () => {
  try {
    const tweetSnapshot = await db.collection("tweet").get();
    const tweets = tweetSnapshot.docs.map((x) => {
      return {
        id: x.id,
        data: {
          ...x.data(),
          created_at: x.data().created_at.toDate(),
          updated_at: x.data().updated_at.toDate(),
        },
      };
    });
    return tweets;
  } catch (e) {
    throw Error("Error while getting Tweet Data");
  }
};

export const store = async ({ data }) => {
  // 省略
};

```

記述したら動作確認する．下記コマンドを実行して，保存されているデータが全件取得できれば OK（下記はデータ 2 件登録時の例）．

```bash
$ curl localhost:3001/tweet

{
  "status": 200,
  "result": [
    {
      "id": "1JXLilqdOqU7rCrwjEpA",
      "data": {
        "tweet": "node.js",
        "created_at": "2021-07-29T11:38:38.005Z",
        "user_id": 1,
        "updated_at": "2021-07-29T11:38:38.006Z"
      }
    },
    {
      "id": "CfTIWGVsuZGOa3nZippY",
      "data": {
        "created_at": "2021-07-29T11:39:09.948Z",
        "updated_at": "2021-07-29T11:39:09.949Z",
        "user_id": 1,
        "tweet": "React"
      }
    }
  ],
  "message": "Succesfully get All Tweet Data!"
}

```

### データ1件取得の処理

ルーティングの追加

`:id`はユーザが付加したパラメータを指定する．コントローラで`req.params.id`で取得する．

```js
import express from "express";
import { readAllTweetData, readOneTweetData, createTweetData } from "../controllers/tweet.controller.js";

export const tweetRouter = express.Router();

tweetRouter.get("/", (req, res) => readAllTweetData(req, res));
// ↓追加
tweetRouter.get("/:id", (req, res) => readOneTweetData(req, res));
tweetRouter.post("/", (req, res) => createTweetData(req, res));

```

コントローラの追加

```js
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

サービスの追加

```js
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

リポジトリの追加

```js
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

動作確認

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

### Update の処理

update のルーティングを追加．

GETの場合と同様にid指定する．`/hoge`に`PUT`でリクエストを送信した場合，`req.params.id`は`hoge`になる．

```js
// routes/todo.route.js
import express from "express";
import { readAllTweetData, readOneTweetData, createTweetData, editTweetData } from "../controllers/tweet.controller.js";

export const tweetRouter = express.Router();

tweetRouter.get("/", (req, res) => readAllTweetData(req, res));
tweetRouter.get("/:id", (req, res) => readOneTweetData(req, res));
tweetRouter.post("/", (req, res) => createTweetData(req, res));
// ↓追加
tweetRouter.put("/:id", (req, res) => editTweetData(req, res));

```

コントローラでは，リクエストから`更新対象のドキュメントのid`と`更新データ`の 2 つを受け取る．送信されたデータの中から，これら 2 つのデータを抽出し，サービスに渡す．

```js
// controllers/todo.controller.js
import { getAllTweetData, getOneTweetData, insertTweetData, updateTweetData } from "../services/tweet.service.js"

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
      throw new Error("something is blank");
    }
    const result = await updateTweetData({
      id: id,
      data: { tweet: tweet, user_id: user_id },
    });
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully edit Tweet Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

```


```js
// services/todo.service.js
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
    throw Error("Error while updating Tweet Data");
  }
};

```
リポジトリでは，受け取ったデータで DB を更新する．`deadline`を Firestore の形式に変換し，同時に`updated_at`に実行日時を設定して送信する．

collection 名と document 名を指定して`update()`でデータを更新できる．実行完了後には，更新ドキュメントの id と更新データを返す．

```js
import admin from "../model/firebase.js";
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
  const ref = await db.collection("tweet").doc(id).update(updateData);
  return {
    id: id,
    data: updateData,
  };
};

```

動作確認する．document は既存のデータから適当に指定する．Read の処理結果などから存在する document 名 を確認しておこう．

コンソール画面 or 前項の Read 処理でデータを確認し，データが更新されていれば OK！

```bash
$ $ curl -X PUT -H "Content-Type: application/json" -d '{"tweet":"Nest.js","user_id":2}' localhost:3001/tweet/1JXLilqdOqU7rCrwjEpA

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

### Delete の処理

削除のルーティングを追加．更新の場合と同様にパラメータを受け取る．

```js
// routes/todo.route.js
import express from "express";
import { readAllTweetData, readOneTweetData, createTweetData, editTweetData, deleteTweetData } from "../controllers/tweet.controller.js";

export const tweetRouter = express.Router();

tweetRouter.get("/", (req, res) => readAllTweetData(req, res));
tweetRouter.get("/:id", (req, res) => readOneTweetData(req, res));
tweetRouter.post("/", (req, res) => createTweetData(req, res));
tweetRouter.put("/:id", (req, res) => editTweetData(req, res));
// ↓追加
tweetRouter.delete("/:id", (req, res) => deleteTweetData(req, res));

```

コントローラでは document 名を受け取り，collection 名と document 名を指定してサービスの処理を実行する．

```js
// controllers/todo.controller.js

import { getAllTweetData, getOneTweetData, insertTweetData, updateTweetData, destroyTodoData } from "../services/tweet.service.js"

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
      throw new Error("something is blank");
    }
    const result = await destroyTodoData({
      id: id,
    });
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully delete Tweet Data!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

```

サービス

```js
// services/todo.service.js
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
export const destroyTodoData = async ({ id }) => {
  try {
    return await destroy({ id: id, });
  } catch (e) {
    throw Error("Error while deleting Tweet Data");
  }
};

```

リポジトリでは DB からデータを削除する．collection 名と document 名があればデータを指定して削除することができる．

```js
import admin from "../model/firebase.js";
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
    throw Error("Error while deleting Todo Data");
  }
};
```

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


## まとめ

今回は Node.js におけるデータの永続化を扱った．DB は Firestore を用いたが，他の DB でも処理の手順は同様である．

本来は DB 関連の処理は別のレイヤーに切り出すことが望ましい．そうすることで，DB 変更時にもコードの修正を該当レイヤーのみに閉じ込めることができる．

今回は以上である( `･ω･´)b
