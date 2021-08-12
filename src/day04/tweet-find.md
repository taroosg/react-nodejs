# tweet 個別表示画面の作成

各 tweet の個別画面を実装する．一覧画面で各 tweet をクリックすると個別の画面を表示するようにしたい．

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

この id を用いてサーバ側にリクエストを送信する（サーバ側では，前回の内容で id 指定して 1 件データ取得する処理を実装した）．

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


## 補足

オブジェクトのプロパティにアクセスする場合，該当しないキーにアクセスすると`undefined`となる．そのため，更に下位のキーを指定するとエラーとなるため処理が止まってしまう．

このような場合，`?.`を使用するとエラーではなく`undefined`として処理することができる．

```js
const someObject = {
  hoge: 1,
  fuga: 2,
  piyo: 3,
};

console.log(someObject.foo);
// undefined

console.log(someObject.foo.bar);
// Uncaught TypeError: Cannot read property 'bar' of undefined

console.log(someObject.foo?.bar);
// undefined

```
