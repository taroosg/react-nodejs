# コマンドの追加

ここまで，下記コマンドでサーバを実行していた．

```bash
$ node app.js
```

これとは別に，プロジェクトに対して自分でコマンドを作成することができる．

コマンドを追加する場合は`package.json`に記述する．下記の内容を追記してみよう．

```json
{
  "name": "express-project",
  "version": "1.0.0",
  "type": "module",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "__comment": "↓ここを追記（この行は書かなくてOK）",
    "start": "node app.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

このように記述すると，`npm start`というコマンドを実行すると`node app.js`が実行される．

プロジェクトが複雑になると，場面に応じて様々なコマンドを実行したい場合がある．そのようなときに，統一されたコマンドを準備しておくことで開発をスムーズに進めることができる．

下記を実行するとサーバが立ち上がる．これまでと同じ動作であることを確認しよう．

```bash
$ npm start
```
