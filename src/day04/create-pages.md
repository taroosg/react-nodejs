# 必要な画面の準備

## ルーティング

クライアント側で必要な画面とパスは以下のとおり．フロント側のルーティングなのでサーバ側と混同しないように注意．

|パス|画面の概要|
|-|-|
|`/tweet/`|tweet 一覧画面|
|`/tweet/:id`|tweet 1 件表示画面|
|`/tweet/post`|tweet 入力 & 送信画面|

## 必要なファイルの準備

1. `src`以下に`pages`フォルダを作成する．
2. `pages`フォルダの中に下記のファイルを作成する．
    - `TweetIndex.jsx`
    - `TweetFind.jsx`
    - `TweetPost.jsx`

ディレクトリは任意の構造にできるが，「ページを表すコンポーネント」と「部品としてのコンポーネント」を分けておくと管理しやすい．

それぞれ以下のように内容を記述する．

```js
// src/pages/TweetIndex.jsx

import React from "react";

export const TweetIndex = () => {
  return (
    <p>tweet 一覧画面</p>
  )
};

```

```js
// src/pages/TweetFind.jsx

import React from "react";

export const TweetFind = () => {
  return (
    <p>tweet 1 件表示画面</p>
  )
};

```

```js
// src/pages/TweetPost.jsx

import React from "react";

export const TweetPost = () => {
  return (
    <p>tweet 入力送信画面</p>
  )
};

```

`App.jsx`を以下のように編集する．

（これまでのコードは適宜コメントアウトしておくこと）

```js
// src/App.jsx

import React from "react";
import { BrowserRouter, Route, Link, Switch } from "react-router-dom";
import { TweetIndex } from "./pages/TweetIndex";
import { TweetFind } from "./pages/TweetFind";
import { TweetPost } from "./pages/TweetPost";

const NotFound = () => {
  return (
    <h2>Not Found...</h2>
  );
};

const App = () => {
  return (
    <BrowserRouter>
      <h1>Twitterのようなもの</h1>
      <ul>
        <li>
          <Link to="/tweet/">tweet 一覧</Link>
        </li>
        <li>
          <Link to="/tweet/post">tweet 入力</Link>
        </li>
      </ul>
      <hr />
      <Switch>
        <Route exact path="/tweet/">
          <TweetIndex />
        </Route>
        <Route exact path="/tweet/post">
          <TweetPost />
        </Route>
        <Route path="/tweet/:id">
          <TweetFind />
        </Route>
        <Route>
          <NotFound />
        </Route>
      </Switch>
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

それぞれの画面が表示されればOK．
