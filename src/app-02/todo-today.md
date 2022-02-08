# todo 一覧画面（本日）の作成

締切が本日以前の todo のみ表示する画面を作成する．

全件表示とほぼ同様だが，データ取得関数のリクエスト先が異なる．

データ取得後は，個別 Todo コンポーネントを動かすだけなので新たな実装は必要ない．

```js
// src/pages/TodoToday.jsx

import { useState, useEffect } from "react";
import axios from "axios";
import { Todo } from "../components/Todo";

export const TodoToday = () => {
  const [todoList, setTodoList] = useState(null);

  const getTodayTodo = async () => {
    const result = await axios.get("http://localhost:3001/todo/today");
    setTodoList(result.data.result);
    return result;
  };

  useEffect(() => {
    getTodayTodo();
  }, []);

  return (
    <ul>
      {todoList?.map((x, i) => (
        <Todo
          hoge={i}
          id={x.id}
          todo={x.todo}
          deadline={x.deadline}
          is_done={x.is_done}
          user_id={x.user_id}
          created_at={x.created_at}
          updated_at={x.updated_at}
          getData={getTodayTodo}
        />
      ))}
    </ul>
  );
};
```

## 動作確認

一覧画面（本日）にアクセスし，実行日以前のデータのみ表示されれば OK！

合わせて，更新と削除の処理が動作することも確認しておく．
