# API実装2（おみくじ中級編）

簡単な API を実装するだけならば，ここまでの内容で十分である．おみくじの処理部分に適当な記述をすれば問題ないであろう．

しかし，コードの記述量が増えてくると，見通しが悪くなってしまい，保守管理にも支障をきたす．

そこで，処理中の役割毎に別ファイルに記述できるようにディレクトリ構成を変更する（責務の分離，などと呼ばれる）．


## ディレクトリ構造と役割

大きく`routes`，`controllers`，`services`の 3 つに分離する．DB などと組み合わせてデータを扱う場合は他に`model`を用意するが今回は省略する．

各要素の役割は以下の通り．

|項目|意味合い|
|-|-|
|routes|URI と実行する処理の対応．|
|controllers| リクエストパラメータの検証，レスポンス送信．|
|services|リクエストに対する処理のメインロジックを記述．|
|repositories|DB関連の処理を記述．今回は出番なし．|

このような役割分担とするため，以下のようにディレクトリとファイルを作成する．

```bash
.
├── app.js
├── controllers
│   └── omikuji.controller.js
├── node_modules
├── package.json
├── package-lock.json
├── routes
│   └── omikuji.route.js
└── services
    └── omikuji.service.js

```


## 実行ファイルの実装

`app.js`を以下のように編集する．

ルーティングは routes 以下のファイルに任せ，指定したファイルを読み込むよう記述．

```js
// app.js

import express from "express";
// おみくじのrouterを読み込む
import { omikujiRouter } from "./routes/omikuji.route.js";

const app = express();
const port = 3001;

app.get("/", (req, res) => {
  res.json({
    uri: "/",
    message: "Hello Node.js!",
  });
});

// おみくじのルーティングを変更
app.use("/omikuji", (req, res) => omikujiRouter(req, res));

app.get("/janken", (req, res) => {
  res.json({ message: "This is Janken URI!" });
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});

```


## ルーティングの実装

`routes/omikuji.route.js`に以下の内容を記述する．

URI と対応するコントローラの処理を記述する．

```js
// routes/omikuji.route.js

import express from "express";
import { getResult } from "../controllers/omikuji.controller.js"

export const omikujiRouter = express.Router();

omikujiRouter.get("/", (req, res) => getResult(req, res));

```


## コントローラの実装

`controllers/omikuji.controller.js`に以下の内容を記述する．

サービスを呼び出して実行したいメソッドを指定し，結果によってレスポンスを指定する．

```js
// controllers/omikuji.controller.js

import { getOmikuji } from "../services/omikuji.service.js";

export const getResult = async (req, res, next) => {
  try {
    const result = await getOmikuji({});
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully get Omikuji!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};

```


## サービスの実装

`services/omikuji.service.js`に以下の内容を記述する．

サービスでは実行したいロジックを書く．ここでおみくじのロジックを実装している．

```js
// services/omikuji.service.js

export const getOmikuji = async (query) => {
  try {
    const omikuji = ["大吉", "中吉", "小吉", "凶", "大凶"];
    const min = 0;
    const max = omikuji.length - 1;
    const index = Math.floor(Math.random() * (max - min + 1)) + min;
    return { result: omikuji[index] };
  } catch (e) {
    throw Error("Error while getting Omikuji.");
  }
};

```

処理の順序としては以下のようになる．

```txt
リクエスト               レスポンス
   ↓                       ↑
routes/omikuji.route.js    |
   ↓                       |
controllers/omikuji.controller.js
   ↓           ↑
ervices/omikuji.service.js

```


## 動作確認

サーバを立ち上げ，以下のコマンドでリクエストを送信する．

```bash
$ curl localhost:3001/omikuji
```

おみくじの結果が返ってくれば成功．数回実行し，異なる結果が返ってくることを確認しておこう．

```bash
$ curl localhost:3001/omikuji
{"status":200,"result":{"result":"凶"},"message":"Succesfully get Omikuji!"}

$ curl localhost:3001/omikuji
{"status":200,"result":{"result":"大凶"},"message":"Succesfully get Omikuji!"}

$ curl localhost:3001/omikuji
{"status":200,"result":{"result":"中吉"},"message":"Succesfully get Omikuji!"}

$ curl localhost:3001/omikuji
{"status":200,"result":{"result":"大吉"},"message":"Succesfully get Omikuji!"}

```

ユーザに返すレスポンスを変更したい場合はコントローラを，処理の内容を更新したい場合はサービスを書き換えれば良い．このように，各責務を別の場所に置くことでメンテナンスしやすいプロダクトになる．

これでおみくじの実装は完了である．

