# Firestore の準備

## Firestore Database の準備

今回は NoSQL である Firebase Firestore Database を用いて CRUD 処理を実装してみる．

DB がオンライン上にあるので，環境問わず使えるところが魅力．

## DB の作成

- Firebase のコンソールにアクセスし，今回のプロジェクトのページを表示．
- 左側メニューの「Firestore Database」をクリックし，DB を作成する．必ず「テストモード」にチェックを入れて作成すること．リージョンは任意．

## JSON ファイルのダウンロード

Node.js で Firestore を操作するには，設定ファイルを用意する必要がある．Firebase で適当なプロジェクトを作成したら，下記の手順で必要なファイルをダウンロードする．

1. Firebase のコンソールにアクセスし，今回のプロジェクトのページを表示．
2. ⚙ -> プロジェクトを設定 で設定画面を表示．
3. 「サービスアカウント」タブ -> 「サービスアカウントを作成」ボタン -> 「新しい秘密鍵の作成」ボタン -> 「キーの生成」ボタンの順にクリック．
4. 適当な場所（デスクトップなど）に json ファイルを保存する．
5. コンソール画面は開いたままにしておこう．

## JSON ファイルの配置と構成ファイルの作成

- プロジェクト直下に`model`ディレクトリを作成する．`model`ディレクトリの中に ↑ でダウンロードした json ファイルを移動する．
- `model`ディレクトリの中に`firebase.js`ファイルを作成する．
- 下記の内容を記述する．
- `const serviceAccount = require('...');`の部分の`require()`内を json ファイルのパスに書き換える．

`firebase.js`は以下のような状態．

```js
import admin from 'firebase-admin';
import { createRequire } from 'module';

const require = createRequire(import.meta.url)
const serviceAccount = require('./hoge-firebase-adminsdk-fuga-piyo.json')

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

export default admin;

```

## `.gitignore`に追記

ダウンロードした JSON ファイルには Firebase プロジェクトの情報が含まれているため， GitHub に push しないよう Git 管理から外しておく．

サーバにデプロイする場合にも必要になるため，その場合は環境変数などを使用して工夫する必要がある．

```
/node_modules
/model/hoge-firebase-adminsdk-fuga-piyo.json
```

## 必要なパッケージのインストール

下記コマンドでインストールする．必ずプロジェクトのディレクトリで行うこと．

エラーがでなければOK．

```bash
$ npm i firebase-admin

added 168 packages, and audited 334 packages in 8s

12 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities

```

