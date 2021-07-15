# API実装1（おみくじ初級編）

## 実行用ファイルの作成

実行するための`app.js`ファイルを作成する．エディタから作成すれば OK．

ファイルを作成したら下記の内容を記述する．

```js
// app.js

import express from "express";

const app = express();
const port = 3001;

app.get("/", (req, res) => {
  res.send("Hello Node.js!");
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});

```

上記のコードは`app.js`が実行されると，`http://localhost:3001`でサーバが立ち上がることを示している．

また，`http://localhost:3001/`に`GET`でリクエストが来ると，`Hello Node.js!`というレスポンスを返すよう記述している．


## 動作確認

上記を記述したら，ターミナルで以下のコマンドを実行し，サーバを起動する．実行する際には作業ディレクトリに移動しておくこと．

```bash
$ node app.js
```

実行結果．下記のようにサーバが立ち上がれば成功．

```bash
Example app listening at http://localhost:3001
```

立ち上がったら，別のターミナルで下記コマンドを実行し，リクエストとレスポンスが適切に処理されることを確認する．

コマンドを実行してメッセージが返ってくれば成功．

```bash
$ curl localhost:3001
Hello Node.js!
```

`curl`はターミナルから http リクエストを送るコマンド．インストールされていない場合はインストールしておく．

サーバを終了する場合は`ctrl + c`で終了できる．


## URI の追加

`/`以外にも URI と作成してみる．

`app.js`を以下のように編集する．`/omikuji`，`/janken`の 2 つの URI を追加し，各レスポンスを JSON 形式で返すよう変更している．

```js
// app.js

import express from "express";

const app = express();
const port = 3001;

// 編集
app.get("/", (req, res) => {
  res.json({
    uri: "/",
    message: "Hello Node.js!",
  });
});

// 追加
app.get("/omikuji", (req, res) => {
  res.json({
    uri: "/omikuji",
    message: "This is Omikuji URI!",
  });
});

// 追加
app.get("/janken", (req, res) => {
  res.json({
    uri: "/janken",
    message: "This is Janken URI!",
  });
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});

```

## 動作確認

記述したら，再度下記コマンドでサーバを立ち上げる．

```bash
$ node app.js
```

別のターミナルから，`curl`コマンドで動作を確認する．

```bash
$ curl localhost:3001/
{"uri":"/","message":"Hello Node.js!"}

$ curl localhost:3001/omikuji
{"uri":"/omikuji","message":"This is Omikuji URI!"}

$ curl localhost:3001/janken
{"uri":"/janken","message":"This is Janken URI!"}

```

簡単であるがこれだけで API を実装することができた．


## 【演習】おみくじ処理の追加

実際におみくじの処理を追加してみよう．

`app.js`の以下の部分におみくじの処理を書いて動作を確認しよう．サーバを起動し，以下のような結果になるように実装する．何回か実行して異なる結果が返ってくれば OK！

```bash
$ curl localhost:3001/omikuji
{"uri":"/omikuji","message":"大吉"}

$ curl localhost:3001/omikuji
{"uri":"/omikuji","message":"凶"}

$ curl localhost:3001/omikuji
{"uri":"/omikuji","message":"中吉"}

```

実装例：

```js
// app.js

// 省略

app.get("/omikuji", (req, res) => {
  const omikuji = ["大吉", "中吉", "小吉", "凶", "大凶"];
  const min = 0;
  const max = omikuji.length - 1;
  const index = Math.floor(Math.random() * (max - min + 1)) + min;
  res.json({
    uri: "/omikuji",
    message: omikuji[index],
  });
});

// 省略

```
