# Supabase の準備

## Supabase の準備

今回は NoSQL である Supabase を用いて CRUD 処理を実装してみる．

DB がオンライン上にあるので，環境問わず使えるところが魅力．

## DB の作成

### プロジェクト作成

Supabase のコンソール（[https://app.supabase.io/](https://app.supabase.io/)）にアクセスし，GitHub アカウントでサインインする．

「New project」をクリックし，project を作成する．
![Supabaseプロジェクト作成](./img/supabase-01.png)

プロジェクト名と DB アクセス用パスワードを設定する．面倒なパスワードを作成しないといけないので注意．

![Supabaseプロジェクト詳細](./img/supabase-02.png)

### テーブルの作成

「Create a new table」をクリックし，テーブルを作成する．

テーブル名：`todo_table`

カラム：以下のとおり設定

| Name         | Type         | 備考                |
| ------------ | ------------ | ------------------- |
| `id`         | `int8`       | 初期設定のまま      |
| `user_id`    | `int8`       | 追加                |
| `todo`       | `varchar`    | 追加                |
| `deadline`   | `date`       | 追加                |
| `is_done`    | `bool`       | 初期値は `false`    |
| `created_at` | `timestampz` | 初期設定のまま      |
| `updated_at` | `timestampz` | `created-at` と同様 |

## Node.js と Supabase の連携

1. Supabase の管理画面から「`⚙`」→「`API`」の順にクリックする．
2. `Project API keys` と `Config` の `URL` が必要になるので確認する．

![Supabaseアクセス情報](./img/supabase-03.png)

3. プロジェクト直下に `.env` ファイルを作成し，以下の内容を記述する．これらの情報は機密情報なので環境変数を用いて管理する．

```txt
SUPABASE_URL=`2で確認したURL`
SUPABASE_API_KEY=`2で確認したAPI key`
```

4. 以下のコマンドを実行し，環境変数を使用するためのライブラリをインストールする．

```bash
$ npm i dotenv
```

5. `.gitignore` に以下の内容を追記し，`.env` ファイルを Git 管理外にする．

```
/node_modules

# 🔽 ここを追記
.env
```

6. 下記コマンドを実行し，Node.js から Supabase を扱うためのライブラリをインストールする．

```bash
$ npm i @supabase/supabase-js
```

ここまででアプリケーションと Supabase を連携させる準備は完了．
