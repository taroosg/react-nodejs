# サーバ側の準備

サーバ側の処理は前回までに実装した内容そのままで問題ないが，クライアントからアクセスする場合に一部追加が必要になる．

## Cross-Origin Resource Sharing

サーバ側の規定の動作として「異なるオリジンからのアクセスをブロックする」というものが存在する．

```txt
https://www.emperor-crimson.com
  ^       ^
  |       |
scheme hostname


http://localhost:5000
  ^       ^       ^
  |       |       |
scheme hostname  port

```

>**Key Point**💡
>
>実際の運用でサーバとクライアントを分ける場合には「サーバ側でクライアントのオリジンからのアクセスを許可する実装」が必要となる．
>
>API へのリクエストを開放する場合にはリクエスト元のホスト問わず許可し，自分のアプリケーションからのみリクエストする場合はクライアント側のホスト名を指定しておく，など．

## 実装

Node.js では Cross-Origin を設定するためのライブラリが存在するので，これを利用するのが楽．

以下のコマンドでインストールする．

```bash
$ npm i cors
```

`app.js`に以下のように追記する．

今回はとりあえずリクエスト元を問わず受け付ける指定にしている．

より詳しい設定はドキュメントを参照．API 毎にアクセスを許可したり拒否したりできる．

[https://github.com/expressjs/cors](https://github.com/expressjs/cors)

```js
import express from 'express';
// ↓ 追加
import cors from 'cors';

// 省略

const app = express();
// ↓ 1 行追加
app.use(cors());
app.use(express.urlencoded({ extended: true }));
app.use(express.json());

// 省略

```

クライアントからリクエストを受ける必要があるので，サーバ側のアプリケーションを立ち上げておこう．

```bash
$ npm start
```
