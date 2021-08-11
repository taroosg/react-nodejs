# tweet 個別表示画面の作成

各 tweet の個別画面を実装する．

## tweet コンポーネントの編集

`Tweet.jsx`コンポーネントを以下のように編集する．

日時表示部分にリンクを設定し，URL に id を追加している．

```js
// src/components/Tweet.jsx

import React from "react";
import { Link } from "react-router-dom";

export const Tweet = ({ key, id, tweet, user_id, created_at }) => {
  return (
    <li key={key} id={id}><p>{tweet} by {user_id} at <Link to={`/tweet/${id}`}>{created_at}</Link></p></li>
  )
};

```

## 個別 tweet コンポーネントの実装

個別表示用ページの`TweetFind.jsx`を以下のように編集する．

`useParams`を用いることで，URL に付加された id を取得することができる．

この id を用いてサーバ側にリクエストを送信する．

```js
// src/pages/TweetFind.jsx

import React, { useState, useEffect } from "react";
import { useParams } from "react-router-dom"
import axios from "axios";

export const TweetFind = () => {
  const { id } = useParams();
  const [tweet, setTweet] = useState(null);

  useEffect(() => {
    const getOneTweet = async (id) => {
      const result = await axios.get(`http://localhost:3001/tweet/${id}`);
      setTweet(result.data.result);
      return result;
    };
    getOneTweet(id);
  }, []);

  return (
    <table>
      <tbody>
        <tr><td>DocumentId</td><td>{tweet?.id}</td></tr>
        <tr><td>Tweet</td><td>{tweet?.data.tweet}</td></tr>
        <tr><td>User_id</td><td>{tweet?.data.user_id}</td></tr>
        <tr><td>Created_at</td><td>{tweet?.data.created_at}</td></tr>
      </tbody>
    </table>
  )
};

```

## 動作確認

一覧画面の作成日時部分のリンクから個別ページに遷移し，個別のデータが表示されればOK．
