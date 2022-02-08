# Slack 連携機能のテスト画面実装

サーバサイドで実装した Slack 投稿の処理を動かすためのテスト画面を作成する．

`src/components` 内に `TestSlack.jsx` を作成する．

ファイルを作成したらルーティングを追加する．`App.jsx` を以下のように編集する．

```js
// src/App.jsx

// 省略

// 🔽 追加
import { TestSlack } from "./pages/TestSlack";

// 省略

<BrowserRouter>
  <h1>Todoリストアプリケーション</h1>
  <ul>
    <li>
      <Link to="/todo/index">todo 一覧（全件）</Link>
    </li>
    <li>
      <Link to="/todo/today">todo 一覧（本日）</Link>
    </li>
    <li>
      <Link to="/todo/post">todo 入力</Link>
    </li>
    {/* 🔽 追加 */}
    <li>
      <Link to="/test-slack">test slack</Link>
    </li>
  </ul>
  <hr />
  <Routes>
    <Route path="/todo/index" element={<TodoIndex />} />
    <Route path="/todo/today" element={<TodoToday />} />
    <Route path="/todo/post" element={<TodoPost />} />
    {/* 🔽 追加 */}
    <Route path="/test-slack" element={<TestSlack />} />
    <Route path="/*" element={<NotFound />} />
  </Routes>
</BrowserRouter>;
```

続いて，`TestSlack.jsx` を以下のように編集する．内容は以下のとおり．

- `postAllTodo` 関数はデータ全件を Slack に投稿する URL にリクエストを送信する．

- `postTodayTodo` 関数は実行日以前のデータを Slack に投稿する URL にリクエストを送信する．

- `onClick` でクリック時に上記関数を実行するように設定する．

```js
// src/pages/TestSlack.jsx

import axios from "axios";

export const TestSlack = () => {
  const postAllTodo = async () => {
    const result = await axios.get("http://localhost:3001/slack");
    console.log(result);
    return result;
  };

  const postTodayTodo = async () => {
    const result = await axios.get("http://localhost:3001/slack/today");
    console.log(result);
    return result;
  };

  return (
    <>
      <button type="button" onClick={postAllTodo}>
        All
      </button>
      <button type="button" onClick={postTodayTodo}>
        Today
      </button>
    </>
  );
};
```

## 動作確認

各ボタンをクリックし，Slack に該当するデータが投稿されれば OK！！
