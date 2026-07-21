# POST 遷移と URL 入力による遷移の違い

Web 画面を開く方法には、フォーム送信などによる `POST` 遷移と、ブラウザのアドレスバーへ URL を入力する遷移がある。

大きな違いは、ブラウザが送る HTTP リクエストのメソッドと、送信データを URL とリクエスト本文のどちらに載せるかである。

## 結論

- アドレスバーに URL を入力して移動すると、ブラウザは通常 `GET` リクエストを送る。
- `POST` 遷移では、HTML の `<form method="post">` などが `POST` リクエストを送り、入力値を主にリクエスト本文へ載せる。
- 同じ URL でも、サーバーは `GET` と `POST` を別の処理として扱える。
- URL を直接入力しても、元の `POST` リクエストをそのまま再現することはできない。

## リクエストの違い

### URL を入力した場合

たとえば、アドレスバーへ次の URL を入力する。

```text
https://example.com/orders/123
```

ブラウザは概ね次のようなリクエストを送る。

```http
GET /orders/123 HTTP/1.1
Host: example.com
```

検索条件などは query parameter として URL に含められる。

```text
https://example.com/products?category=book&page=2
```

```http
GET /products?category=book&page=2 HTTP/1.1
Host: example.com
```

### POST 遷移の場合

次のフォームで送信ボタンを押した場合を考える。

```html
<form action="/orders" method="post">
  <input name="productId" value="A001">
  <input name="quantity" value="2">
  <button type="submit">注文する</button>
</form>
```

ブラウザは概ね次のようなリクエストを送る。

```http
POST /orders HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

productId=A001&quantity=2
```

アドレスバーには `/orders` と表示されても、`productId` や `quantity` は URL ではなくリクエスト本文に入っている。そのため、アドレスバーへ `https://example.com/orders` と入力するだけでは、同じ注文処理にはならない。

## 比較

| 観点 | URL 入力による遷移 | POST 遷移 |
| --- | --- | --- |
| HTTP メソッド | 通常は `GET` | `POST` |
| 主な用途 | ページやデータの取得 | 登録、更新、注文確定など、サーバー状態を変える処理 |
| データの場所 | URL のパスや query parameter | 主にリクエスト本文 |
| アドレスバーでの再現 | URL を入力すれば再現しやすい | URL だけでは本文を再現できない |
| ブックマーク・共有 | しやすい | POST 本文はブックマークや共有の対象にならない |
| 再読み込み | 同じ `GET` を再送する | 再送信確認が出ることがある |
| ブラウザ履歴 | 戻る・進むで再取得しやすい | 戻る・進むでフォーム再送信が必要になることがある |
| キャッシュ | キャッシュされることがある | 通常の画面設計ではキャッシュを前提にしない |

`GET` と `POST` の用途は単なる慣習ではない。HTTP では `GET` は safe、つまり対象リソースの状態変更を要求しないメソッドとして定義されている。一方、`POST` は送信内容をサーバー側で処理してもらうために使用され、登録や更新など状態変更を伴うことがある。

## 同じ URL でも処理が異なる例

サーバー側では、同じ `/users` というパスに対してメソッド別の処理を定義できる。

| リクエスト | 処理例 |
| --- | --- |
| `GET /users` | ユーザー一覧画面を表示する |
| `POST /users` | 入力内容を使ってユーザーを登録する |

ブラウザへ `https://example.com/users` と入力すると送られるのは `GET /users` である。そのため、`POST /users` の登録処理は実行されない。

サーバーが `POST` だけを受け付ける URL に `GET` でアクセスした場合は、実装によって `405 Method Not Allowed` になったり、別の画面が表示されたりする。

## POST 後に URL が表示される理由

POST の処理後に別画面へ移動する設計では、Post/Redirect/Get（PRG）パターンがよく使われる。

```text
1. ブラウザ → POST /orders        注文を登録する
2. サーバー → 303 See Other       /orders/123 へ移動するよう指示する
3. ブラウザ → GET /orders/123     注文完了画面を取得する
```

最後に表示されている画面は `GET` で取得されているため、再読み込みしても注文の `POST` が再送されにくい。また、完了画面の URL を履歴やブックマークで扱いやすくなる。

## 誤解しやすい点

### POST なら入力値が安全に隠れるわけではない

POST の本文はアドレスバーには表示されないが、通信内容そのものが暗号化されるわけではない。盗聴を防ぐには `HTTPS` を使用する必要がある。

また、パスワードや個人情報を URL に含めると、ブラウザ履歴、アクセスログ、参照元情報などに残る可能性があるため、機密情報は query parameter に載せない。

### URL が同じなら同じ画面になるとは限らない

レスポンスは、HTTP メソッドだけでなく、リクエスト本文、Cookie、ログイン状態、Header、サーバー側のデータにも左右される。同じ URL が表示されていても、同じ処理結果になるとは限らない。

### JavaScript による送信は画面遷移しないこともある

`fetch()` などで `POST` を送る場合、通信しただけでは通常のページ遷移は発生しない。レスポンスを受け取ったあと、画面の一部を更新したり、JavaScript で別 URL へ移動したりするのはアプリケーション側の実装である。

## 開発時の確認方法

Chrome や Edge の開発者ツールで Network タブを開くと、実際の通信を確認できる。

| 確認項目 | 分かること |
| --- | --- |
| Request Method | `GET`、`POST` など、送信した HTTP メソッド |
| Request URL | リクエスト先の URL |
| Query String Parameters | URL に含まれる検索条件など |
| Form Data / Payload | POST のリクエスト本文 |
| Status Code | `200`、`303`、`405` などの処理結果 |
| Location | リダイレクト先 |

「画面上は同じ URL に見えるのに動作が違う」という場合は、まず Request Method と Payload を比較する。

## まとめ

- URL 入力による遷移は通常 `GET` であり、URL で指定したリソースの取得を要求する。
- POST 遷移は `POST` の本文にデータを載せ、登録や更新などの処理を要求できる。
- URL だけでは POST のメソッドや本文を再現できない。
- POST 後は PRG パターンで `GET` の完了画面へリダイレクトすると、二重送信を防ぎやすい。
- 実際の違いは、ブラウザの開発者ツールで Method、Payload、Status Code、Location を確認すると判断できる。

## 出典

- RFC 9110: `HTTP Semantics` 9.3.1 GET / 9.3.3 POST  
  https://www.rfc-editor.org/rfc/rfc9110.html#section-9.3.1  
  https://www.rfc-editor.org/rfc/rfc9110.html#section-9.3.3
- MDN Web Docs: `Sending form data`  
  https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_and_retrieving_form_data
- MDN Web Docs: `303 See Other`  
  https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/303
- 取得日: 2026-07-21
