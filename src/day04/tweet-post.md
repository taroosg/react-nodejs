# tweet 送信画面の作成

ブラウザ画面から tweet を送信できるように処理を追加する．

## Form を扱うライブラリの準備

前回の API と連携させるため，画面に入力したデータを POST で送信する必要がある（前回はターミナルでリクエストを送信していた）．

form を扱うのは面倒なのでライブラリを使用するのが得策．ちょうど`useState`や`useEffect`などの「Hooks」と合わせて使用できる「React Hook Form」が提供されている．

[https://react-hook-form.com/jp/](https://react-hook-form.com/jp/)

下記コマンドでインストールする．

```bash
$ npm i react-hook-form
```

`TweetPost.jsx`を以下のように編集する．

まずは form の動きを確認する．

```js
import React, { useState } from "react";
import { useForm } from "react-hook-form";
import axios from "axios";

export const TweetPost = () => {
  const { register, handleSubmit } = useForm();

  const [formData, setFormData] = useState(null);

  const postFormData = async (postData) => {
    setFormData(JSON.stringify(postData));
  };

  return (
    <form onSubmit={handleSubmit(postFormData)}>
      <input {...register("tweet")} placeholder="Tweet" />
      <input {...register("user_id")} placeholder="User_id" type="number" />
      <p>{formData}</p>
      <button type="submit">送信</button>
    </form>
  )
};

```

## 動作確認

コードを記述したら動作を確認する．

入力欄に適当な文字列を入力し，入力した値が下記のように表示されればOK．

```txt
{"tweet":"test","user_id":"4"}
```

## POST 送信処理の追加

form の動きは確認できたので，サーバ側に入力内容を送信する．

POST で送信する場合は`axios.post()`を用いる．第 1 引数にリクエスト URL を，第 2 引数に送信するデータを入れる．

`postFormData`関数を以下のように編集する．

```js
const postFormData = async (postData) => {
  setFormData(JSON.stringify(postData));
  const result = await axios.post("http://localhost:3001/tweet", postData);
  console.log(result);
  return result;
};

```

## 動作確認

>必ずサーバ側のアプリケーションを動作させてくこと！！

先の動作確認と同様に，適当な文字列を入力欄に入力して送信する．

コンソールに以下のようなレスポンスが表示されればOK．

```js
{
  "data": {
    "status": 200,
      "result": {
      "id": "7IKHDJBdyUCwhVZ2wtsV",
        "data": {
        "tweet": "test",
          "user_id": "4"
      }
    },
    "message": "Succesfully post Tweet Data!"
  },
  "status": 200,
    "statusText": "OK",
      "headers": {
    "content-length": "132",
      "content-type": "application/json; charset=utf-8"
  },
  "config": {
    "url": "http://localhost:3001/tweet",
      "method": "post",
        "data": "{\"tweet\":\"test\",\"user_id\":\"4\"}",
          "headers": {
      "Accept": "application/json, text/plain, */*",
        "Content-Type": "application/json;charset=utf-8"
    },
    "transformRequest": [
      null
    ],
      "transformResponse": [
        null
      ],
        "timeout": 0,
          "xsrfCookieName": "XSRF-TOKEN",
            "xsrfHeaderName": "X-XSRF-TOKEN",
              "maxContentLength": -1,
                "maxBodyLength": -1
  },
  "request": { }
}

```

これでブラウザから tweet をサーバに送信する処理は完了．
