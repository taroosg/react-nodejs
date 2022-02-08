# データ取得タイミングの調整

SWR を使用すると，様々なタイミングでデータ取得の処理を実行することができる．

## SWR のオプション設定

今回は以下の 2 パターンでデータを取得する処理を実装してみる．

- ユーザの操作によらず 3 秒毎にデータ取得．

- ユーザがウインドウをアクティブにしたタイミングでデータ取得．

その他のオプションはドキュメント参照．

[https://swr.vercel.app/docs/options](https://swr.vercel.app/docs/options)

`TodoIndex.jsx`を以下のように編集する．

```js
// src/pages/TodoIndex.jsx

// 省略

const fetcher = async (url) => (await axios.get(url)).data.result;

const options = {
  // 初期データ
  initialData: null,
  // pollingの期間（ミリ秒）
  refreshInterval: 3000,
  // windowのフォーカス時にRevalidateする
  revalidateOnFocus: true,
};

const { data, error } = useSWR("http://localhost:3001/todo", fetcher, options);

// 省略
```

## 動作確認

タブを 2 つ用意し，両方とも実装中のアプリケーションを開く．片方は「 todo 送信画面」もう片方は「一覧画面」を表示しておく．

送信画面側で todo を送信し，一覧画面を操作しなくても自動的に表示されれば OK．

なにもしなくてもデータ取って表示できる．すごい．

> 補足
>
> - `TodoToday.jsx` でも同様の処理でデータを取得してみよう．
>
> - 更新及び削除処理と組み合わせる場合など，任意のタイミングでデータを反映できる useEffect のほうが適している場合もある．
>
> - useSWR には「mutate」という機能があり，データを post しながら先取りして更新できる仕組みもあるので，興味がある人はチャレンジしてみよう！
