# todo 一覧画面の作成

続いて一覧画面を作成する．

## ダミーデータの表示

`TodoIndex.jsx` を以下のように編集する．

まずは適当なダミーデータを正しく画面に表示できることを確認する．ダミーデータを `useState` の変数に初期値として設定し，画面に表示する．

```js
// src/pages/TodoIndex.jsx

import { useState } from "react";

export const TodoIndex = () => {
  const dummyTodoList = [
    {
      id: 1,
      todo: "test1",
      deadline: "2022-02-02",
      user_id: "1",
      id_done: false,
      created_at: JSON.stringify(new Date()),
      updated_at: JSON.stringify(new Date()),
    },
    {
      id: 2,
      todo: "test2",
      deadline: "2022-02-22",
      user_id: "2",
      id_done: false,
      created_at: JSON.stringify(new Date()),
      updated_at: JSON.stringify(new Date()),
    },
    {
      id: 3,
      todo: "test3",
      deadline: "2022-02-28",
      user_id: "3",
      id_done: false,
      created_at: JSON.stringify(new Date()),
      updated_at: JSON.stringify(new Date()),
    },
  ];

  const [todoList, setTodoList] = useState(dummyTodoList);

  return (
    <ul>
      {todoList.map((x, i) => (
        <li key={i}>
          {x.deadline} {x.todo} by {x.user_id} at
          {x.created_at} {x.updated_at}
        </li>
      ))}
    </ul>
  );
};
```

## 動作確認

一覧画面にアクセスし，dummy のデータが一覧で表示されれば OK．

```txt
2022-02-02 test1 by 1 at "2021-08-10T04:47:27.316Z" "2021-08-10T04:47:27.316Z"
2022-02-22 test2 by 2 at "2021-08-10T04:47:27.316Z" "2021-08-10T04:47:27.316Z"
2022-02-28 test3 by 3 at "2021-08-10T04:47:27.316Z" "2021-08-10T04:47:27.316Z"
```

## データ取得処理の追加

API サーバから todo のデータを取得する処理を追加する．

1 回め講義の Google books API データ取得時と同様に `useState` と `useEffect` を用いた実装としている．

```js
// src/pages/TodoIndex.jsx

import { useState, useEffect } from "react";
import axios from "axios";

export const TodoIndex = () => {
  // 省略

  // 🔽 編集
  const [todoList, setTodoList] = useState(null);

  // 🔽 追加
  const getAllTodo = async () => {
    const result = await axios.get("http://localhost:3001/todo");
    setTodoList(result.data.result);
    return result;
  };

  // 🔽 追加
  useEffect(() => {
    getAllTodo();
  }, []);

  return (
    <ul>
      {todoList?.map((x, i) => (
        <li key={i}>
          {x.deadline} {x.todo} by {x.user_id}
          {x.created_at} {x.updated_at}
        </li>
      ))}
    </ul>
  );
};
```

## 動作確認

> 必ずサーバ側のアプリケーションを動作させてくこと！！

ブラウザで動作を確認する．

一覧画面に Supabase に保存されているデータが表示されれば OK．
