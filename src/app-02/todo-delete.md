# todo 削除処理の作成

削除の処理を追加する．手順は以下のとおり．

- axios を用いて，Node.js 側で実装した削除の URL にリクエストを送信する．

- 削除には「id」があれば良い．

- 更新と同様，削除処理完了後に最新データ取得の処理を実行する．

## 削除処理の追加

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

  // 🔽 追加
  const deleteTodoData = async (id) => {
    const requestUrl = "http://localhost:3001/todo";
    const removedData = await axios.delete(`${requestUrl}/${id}`);
    const result = await getData();
    return removedData;
  };

  return (
    <li key={id} id={id}>
      <input
        type="checkbox"
        checked={is_done}
        onChange={() =>
          updateTodoData({ id, todo, deadline, is_done, user_id })
        }
      />
      {/* 🔽 追加 */}
      <button type="button" onClick={() => deleteTodoData(id)}>
        delete
      </button>
      <p>
        {deadline} {todo} by {user_id} at {created_at} and {updated_at}
      </p>
    </li>
  );
};
```

## 動作確認

delete ボタンをクリックし，該当するデータが削除されれば OK！
