# 必要な画面の準備

## ルーティング

クライアント側で必要な画面とパスは以下のとおり．フロント側のルーティングなのでサーバ側と混同しないように注意．

| パス          | 画面の概要                      |
| ------------- | ------------------------------- |
| `/todo/index` | todo 一覧画面（全件）           |
| `/todo/today` | todo 一覧画面（実行日以前締切） |
| `/todo/post`  | todo 入力 & 送信画面            |

## 必要なファイルの準備

1. `src`以下に`pages`フォルダを作成する．

2. `pages`フォルダの中に下記のファイルを作成する．

   - `TodoIndex.jsx`

   - `TodoToday.jsx`

   - `TodoPost.jsx`

ディレクトリは任意の構造にできるが，「ページを表すコンポーネント」と「部品としてのコンポーネント」を分けておくと管理しやすい．

それぞれ以下のように内容を記述する．

```js
// src/pages/TodoIndex.jsx

export const TodoIndex = () => {
  return <p>todo （全件）一覧画面</p>;
};
```

```js
// src/pages/TodoToday.jsx

export const TodoToday = () => {
  return <p>todo （本日）一覧画面</p>;
};
```

```js
// src/pages/TodoPost.jsx

export const TodoPost = () => {
  return <p>todo 入力送信画面</p>;
};
```

`App.jsx`を以下のように編集する．

（これまでのコードは適宜コメントアウトしておくこと）

```js
// src/App.jsx

import { BrowserRouter, Route, Link } from "react-router-dom";
import { TodoIndex } from "./pages/TodoIndex";
import { TodoToday } from "./pages/TodoToday";
import { TodoPost } from "./pages/TodoPost";

const NotFound = () => {
  return <h2>Not Found...</h2>;
};

const App = () => {
  return (
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
      </ul>
      <hr />
      <Routes>
        <Route path="/todo/index" element={<TodoIndex />} />
        <Route path="/todo/today" element={<TodoToday />} />
        <Route path="/todo/post" element={<TodoPost />} />
        <Route path="/*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
};
export default App;
```

## 動作確認

一通り記述したら画面の表示を確認する．

```bash
$ npm start
```

それぞれの画面が表示されれば OK．
