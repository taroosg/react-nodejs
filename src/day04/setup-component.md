# コンポーネントの最適化

各 tweet の内容はシンプルなのでこのままでも問題ないが，より複雑な情報を扱う場合には個別のコンポーネントに分けることが望ましい．

## 必要なファイルの準備

`src/components`フォルダの中に各 tweet を表示する`Tweet.jsx`を作成する．

以下の内容を記載する．

```js
// src/components/Tweet.jsx

import React from "react";

export const Tweet = () => {
  return (
    <li>
      <p>tweet by user_id at created_at</p>
    </li>
  )
};

```

## 一覧画面の編集

`TweetIndex.jsx`を以下のように編集する．

```js
import React, { useState, useEffect } from "react";
import axios from "axios";
// ↓ 追加
import { Tweet } from "../components/Tweet";

export const TweetIndex = () => {

  // 省略

  // ↓ 編集
  return (
    <ul>
      {tweetList?.map((x, i) => <Tweet key={i} id={x.id} tweet={x.data.tweet} user_id={x.data.user_id} created_at={x.data.created_at} />)}
    </ul>
  )
};

```

## 個別ツイートコンポーネントの実装

`Tweet.jsx`コンポーネントを以下のように編集する．

このコンポーネントは`key`，`id`，`tweet`，`user_id`，`created_at`の 5 つのパラメータを入力することとする．

```js
// src/components/Tweet.jsx

import React from "react";

export const Tweet = ({ key, id, tweet, user_id, created_at }) => {
  return (
    <li key={key} id={id}><p>{tweet} by {user_id} at {created_at}</p></li>
  )
};

```

## 動作確認

これまでと同様に tweet の一覧が表示されればOK．
