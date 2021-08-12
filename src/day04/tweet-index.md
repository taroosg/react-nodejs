# tweet 一覧画面の作成

続いて一覧画面を作成する．

## ダミーデータの表示

`TweetIndex.jsx`を以下のように編集する．

まずは適当なダミーデータを正しく画面に表示できることを確認する．ダミーデータを`useState`の変数に初期値として設定し，画面に表示する．

```js
// src/pages/TweetIndex.jsx

import React, { useState } from "react";

export const TweetIndex = () => {
  const dummyTweetList = [
    { id: "qwerty", data: { tweet: "test1", user_id: "1", created_at: JSON.stringify(new Date()), } },
    { id: "asdfgh", data: { tweet: "test2", user_id: "2", created_at: JSON.stringify(new Date()), } },
    { id: "zxcvbn", data: { tweet: "test3", user_id: "3", created_at: JSON.stringify(new Date()), } },
  ];

  const [tweetList, setTweetList] = useState(dummyTweetList);

  return (
    <ul>
      {tweetList.map((x, i) => <li key={i}>{x.data.tweet} by {x.data.user_id} at {x.data.created_at}</li>)}
    </ul>
  )
};

```

## 動作確認

一覧画面にアクセスし，dummy のデータが一覧で表示されればOK．

```txt
test1 by 1 at "2021-08-10T04:47:27.316Z"
test2 by 2 at "2021-08-10T04:47:27.316Z"
test3 by 3 at "2021-08-10T04:47:27.316Z"
```

## データ取得処理の追加

API サーバから tweet のデータを取得する処理を追加する．

1 回め講義の Google books API データ取得時と同様に`useState`と`useEffect`を用いた実装としている．

```js
// src/pages/TweetIndex.jsx

import React, { useState, useEffect } from "react";
import axios from "axios";

export const TweetIndex = () => {

  // 省略

  const [tweetList, setTweetList] = useState(null);

  useEffect(() => {
    const getAllTweet = async () => {
      const result = await axios.get("http://localhost:3001/tweet");
      setTweetList(result.data.result);
      return result;
    };
    getAllTweet();
  }, []);

  return (
    <ul>
      {tweetList?.map((x, i) => <li key={i}>{x.data.tweet} by {x.data.user_id} at {x.data.created_at}</li>)}
    </ul>
  )
};

```

## 動作確認

>必ずサーバ側のアプリケーションを動作させてくこと！！

ブラウザで動作を確認する．

一覧画面に Firestore に保存されているデータが表示されればOK．
