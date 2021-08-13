# データ取得タイミングの調整

SWR を使用すると，様々なタイミングでデータ取得の処理を実行することができる．

## SWR のオプション設定

今回は以下の 2 パターンでデータを取得する処理を実装してみる．

- ユーザの操作によらず 5 秒毎にデータ取得．
- ユーザがウインドウをアクティブにしたタイミングでデータ取得．

その他のオプションはドキュメント参照．

[https://swr.vercel.app/docs/options](https://swr.vercel.app/docs/options)

`TweetIndex.jsx`を以下のように編集する．

```js
// src/pages/TweetIndex.jsx

  // 省略

  const fetcher = async (url) => (await axios.get(url)).data.result;

  const options = {
    // 初期データ
    initialData: null,
    // pollingの期間（ミリ秒）
    refreshInterval: 5000,
    // windowのフォーカス時にRevalidateする
    revalidateOnFocus: true,
  }

  const { data, error } = useSWR("http://localhost:3001/tweet", fetcher, options)

  // 省略

```

## 動作確認

タブを 2 つ用意し，両方とも実装中のアプリケーションを開く．片方は「 tweet 送信画面」もう片方は「一覧画面」を表示しておく．

送信画面側で tweet を送信し，一覧画面を操作しなくても自動的に表示されればOK．

なにもしなくてもデータ取って表示できる．すごい．
