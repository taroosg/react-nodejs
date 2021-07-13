
# コンポーネント

ここから，実際にコンポーネントを実装してみる．

まずは必要なファイルを準備しよう．

- `src`ディレクトリに`components`ディレクトリを作成する．
- `components`ディレクトリ内に`Booklist.jsx`を作成する．
- `Booklist.jsx`に以下の内容を記述する．

```jsx
// Booklist.jsx
import React from "react";

export const Booklist = (props) => {
  return (
    <>
      <p>this is book list component</p>
    </>
  );
};

```

>【解説】コンポーネントの作成
>
>- コンポーネントには「class コンポーネント」と「関数コンポーネント」の 2 種類が存在する．
>- class コンポーネントは「状態を持つことができる（ステートフル）」という特徴があるが，関数コンポーネントでは「状態を持つことができない（ステートレス）」．
>- しかし，関数コンポーネントでも「React hooks」という技術を使うことで同様の振る舞いを実現することができる．
>- 「state」というのはコンポーネント自体が持つ値であり，他にコンポーネントが外から受け取る「props」が存在する．
>- コンポーネント作成の際には，できるだけ state を持たないよう設計すると動作確認が楽になるだけでなく，バグの生まれる可能性も低減できる．
>- そのため，本記事ではすべて関数コンポーネントを使用してアプリケーションを構築し，必要に応じて「React hooks」を使用して状態を管理する．


## コンポーネントの構造

- `import ...`は必要なライブラリを読み込む．
- 関数`Booklist`は呼び出し元のコンポーネントから`props`を受け取り，html要素を出力する．
- `export const Booklist = ...`とすることで，他のコンポーネントから`import { Booklist } ...`のように記述して呼び出せるようにしている．


## コンポーネントの呼び出し（1）

`Booklist`コンポーネントを作成したので，`App`コンポーネントから呼び出してみる．

`App.jsx`を以下のように編集する．

```jsx
// App.jsx
import React from "react";
import { Booklist } from "./components/Booklist"; // 追加（コンポーネントのimport）

const App = () => {
  return (
    <>
      <h1>react app</h1>
      <Booklist />
    </>
  );
};
export default App;

```

ブラウザで表示を確認すると以下のようになっている．

![メイン画面2](./img/mainview02.png)

また，検証画面を確認すると，以下のように Booklist コンポーネントが読み込まれていることがわかる．

![検証画面1](./img/elements01.png)


## コンポーネントの呼び出し（2）

呼び出すコンポーネントを 2 つにしてみる．

`App.jsx`を以下のように編集する．

```jsx
// App.jsx
import React from "react";
import { Booklist } from "./components/Booklist";

const App = () => {
  return (
    <>
      <h1>react app</h1>
      <Booklist />
      <Booklist />
    </>
  );
};
export default App;
```

ブラウザ画面で，`Booklist`コンポーネントが 2 つ表示されていることを確認しよう．

このように，作成したコンポーネントは複数使用することもできる．

![メイン画面3](./img/mainview03.png)

