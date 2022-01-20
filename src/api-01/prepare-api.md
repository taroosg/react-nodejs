# API 実装の準備

Node.js のプロジェクトを実装し，動かしてみる．

## プロジェクト作成

プロジェクトを作成するには，下記のコマンドを実行する．

```bash
$ mkdir express-project && cd express-project
$ npm init
```

`npm`は`node package module`の略であり，Node.js 上で動くパッケージを管理するツールである．Node.js で開発を行う場合はこれを用いることがほとんどである．

ダイアログが出てくるので，答えていく．全部そのまま Enter で OK．

```bash
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help init` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (express-project)
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
About to write to /home/taroosg/Desktop/express-project/package.json:

{
  "name": "express-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}


Is this OK? (yes)

```

完了したらエディタで開く．

エディタで開いたら`package.json`が作成されているので，中身を確認する．

```json
{
  "name": "express-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}

```

| 項目        | 意味合い                                                                   |
| ----------- | -------------------------------------------------------------------------- |
| name        | プロダクト名．                                                             |
| version     | プロダクトのバージョン．                                                   |
| description | プロダクトの説明．                                                         |
| main        | プロダクトをパッケージとして公開する場合に入り口となるファイルを指定する． |
| script      | 開発者が任意に作成するコマンド．                                           |
| auther      | 開発者情報．1 人のみ記述する．                                             |
| license     | ライセンス情報．                                                           |


## `package.json`に追記

下記の内容を追記する．

```json
"type": "module",
```

追記後は以下のような状態．

```json
{
  "name": "express-project",
  "version": "1.0.0",
  "type": "module",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}

```

>**💡 Key Point**
>
>`package.json`に`"type": "module"`を追記することで`import`形式で別ファイルを読みこむことができる．
>この追記を行わない場合，外部ファイルを読み込む際には`require()`関数を用いる．
>前回のReactなどでは`import`が基本となっており，こちらに揃えておいたほうがメリットが大きくオススメである．
