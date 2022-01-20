# Read の処理（全件）

## データ全件取得の処理

Read の処理では，ルーティングとコントローラとサービスははじめにつくったものを使用する．

リポジトリに以下の内容を記述する．テーブルを指定してデータをすべて取得する．`order()` を使用してデータを並び替えることができる．

参考：[https://supabase.com/docs/reference/javascript/using-modifiers](https://supabase.com/docs/reference/javascript/using-modifiers)

```js
// repositories/todo.repository.js

import dotenv from "dotenv";
import { createClient } from "@supabase/supabase-js";

dotenv.config();

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_API_KEY
);

// 🔽 編集
export const findAll = async () => {
  try {
    const { data, error } = await supabase
      .from("todo_table")
      .select()
      .order("deadline", { ascending: true })
      .order("todo", { ascending: true });
    return data;
  } catch (e) {
    throw Error("Error while getting Todo Data");
  }
};

export const store = async ({ params }) => {
  // 省略
};
```

## 動作確認（全件取得）

記述したら動作確認する．下記コマンドを実行して，保存されているデータが全件取得できれば OK（下記はデータ 2 件登録時の例）．

```bash
$ curl localhost:3000/todo

{
  "status": 200,
  "result": [
    {
      "id": 2,
      "user_id": 1,
      "todo": "react",
      "deadline": "2021-12-21",
      "is_done": false,
      "created_at": "2021-12-16T06:23:58.614838+00:00",
      "updated_at": "2021-12-16T06:23:58.614838+00:00"
    },
    {
      "id": 2,
      "user_id": 1,
      "todo": "node.js",
      "deadline": "2021-12-31",
      "is_done": false,
      "created_at": "2021-12-16T06:21:31.592284+00:00",
      "updated_at": "2021-12-16T06:21:31.592284+00:00"
    }
  ],
  "message": "Successfully get All Todo Data!"
}

```
