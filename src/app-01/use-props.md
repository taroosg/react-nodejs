# props の活用

`props` は呼び出し元のコンポーネント（親コンポーネント）から呼び出されたコンポーネント（子コンポーネント）に渡されるデータの塊である．

> **💡 Key Point**
>
> React のコンポーネントは「関数」であり，入力と出力がある．「入力」に当たるのがこの `props` である．

## `props` でデータを渡す

実際に `App` コンポーネントから `Booklist` コンポーネントに文字列のデータを渡してみよう．

`App.jsx`を以下のように編集する．

```jsx
// App.jsx

import { Booklist } from "./components/Booklist";

const App = () => {
  const languages = ["React", "Vue", "Angular"];
  return (
    // 🔽 編集（コンポーネントに props を設定）
    <>
      <h1>react app</h1>
      <Booklist language={languages[0]} />
      <Booklist />
    </>
  );
};
export default App;
```

ここでは，「`App` コンポーネントから `Booklist` コンポーネントに」「`language` という名前で」「`languages[0]` の値」を渡している．

タグ内に変数を入れるときは `{}` を使用する．

## `props` でデータを受け取る

続いて，値を渡される `Booklist.jsx` を以下のように編集する．

```jsx
// Booklist.jsx

export const Booklist = ({ language }) => {
  return (
    <>
      <p>this is {language} list component</p>
    </>
  );
};
```

ブラウザで画面を確認すると以下のような状態になる．

1 つめのコンポーネントは `React` の文字列が追加されているが，2 つめのコンポーネントには追加されていない．なぜだろうか．

![メイン画面4](./img/mainview04.png)

1 つめのコンポーネントには `App.jsx` で `language={languages[0]}` が記述されているが，2 つめでは記述されていない．そのため，2 つめのコンポーネントでは `props` が空の状態となっており表示が追加されない．

## 追加！

2 つめのコンポーネントにもデータを渡すには次のように `App.jsx` を編集する．

（ついでにコンポーネントを追加！）

```jsx
// App.jsx

import { Booklist } from "./components/Booklist";

const App = () => {
  const languages = ["React", "Vue", "Angular"];
  return (
    // 🔽 編集（コンポーネントに props を設定）
    <>
      <h1>react app</h1>
      <Booklist language={languages[0]} />
      <Booklist language={languages[1]} />
      <Booklist language={languages[2]} />
    </>
  );
};
export default App;
```

こうなる！

![メイン画面5](./img/mainview05.png)

> **💡 Key Point**
>
> このように，関数コンポーネントは `props` を受け取り，要素を返す関数となる．実装するときは「何を入力して」「何を出力するのか」を意識すると（多分）混乱せずすすめることができる．
