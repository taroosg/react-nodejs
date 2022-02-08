# todo 更新処理の作成

更新の処理を追加する．今回はチェックボックスで完了未完了の変更をできるようにする．

流れは以下のとおり．

- `onChange` を用いて，チェックボックスの切り替えを検出する．

- 切り替えのタイミングで更新処理の関数を実行する．`is_done` の「true」「false」を入れ替えてサーバに送信する．

- 更新処理終了後は，更新後の最新データを反映させるため，`TodoIndex.jsx` から入力されたデータ取得関数を実行している．

> 補足
>
> サーバへ送信するデータについてはサーバ側で定めている．実装の際に確認しておくこと．

## 更新処理の追加

`Todo.jsx` を以下のように編集する．

```js
// src/components/Todo.jsx

import axios from "axios";

export const Todo = ({
  id,
  todo,
  deadline,
  is_done,
  user_id,
  created_at,
  updated_at,
  getData,
}) => {
  const updateTodoData = async (params) => {
    const newData = { ...params, ...{ is_done: is_done ? false : true } };
    const requestUrl = "http://localhost:3001/todo";
    const updatedData = await axios.put(`${requestUrl}/${params.id}`, newData);
    const result = await getData();
    return updatedData;
  };

  return (
    <li key={id} id={id}>
      {/* 🔽 編集 */}
      <input
        type="checkbox"
        checked={is_done}
        onChange={() =>
          updateTodoData({ id, todo, deadline, is_done, user_id })
        }
      />
      <p>
        {deadline} {todo} by {user_id} at {created_at} and {updated_at}
      </p>
    </li>
  );
};
```

更新処理終了後には最新のデータを取得する必要があるので，一覧画面表示から関数を入力する．

```js
// src/pages/TodoIndex.jsx

import { useState, useEffect } from "react";
import axios from "axios";
import { Todo } from "../components/Todo";

export const TodoIndex = () => {
  // 省略

  return (
    <ul>
      {todoList?.map((x, i) => (
        <Todo
          id={x.id}
          todo={x.todo}
          deadline={x.deadline}
          is_done={x.is_done}
          user_id={x.user_id}
          created_at={x.created_at}
          updated_at={x.updated_at}
          // 🔽 追加
          getData={getAllTodo}
        />
      ))}
    </ul>
  );
};
```

## 動作確認

チェックボックスをクリックし，切り替われば OK！

Supabase の管理画面でもデータ（is_done）が更新されていることを確認する．
