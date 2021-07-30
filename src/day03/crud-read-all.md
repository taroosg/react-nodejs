# Read の処理（全件）

## データ全件取得の処理

Read の処理では，ルーティングとコントローラとサービスははじめにつくったものを使用する．

リポジトリに以下の内容を記述する．collection 名を指定してデータをすべて取得する．日付部分は独自形式なので，データ取得後に`Date`形式に変換している．

`tweetSnapshot`は取得したデータそのままで使いにくいので，必要な部分を取り出して`tweets`に入れている．

また，Firestore の`created_at`と`updated_at`はそのままだと使いづらいので`.toData()`で変換している．

```js
// repositories/tweet.repository.js

import admin from '../model/firebase.js';
const db = admin.firestore();

export const findAll = async () => {
  try {
    const tweetSnapshot = await db.collection('tweet').get();
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
    throw Error('Error while getting Tweet Data');
  }
};

export const store = async ({ data }) => {
  // 省略
};

```

## 動作確認（全件取得）

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

