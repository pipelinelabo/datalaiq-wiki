# ログインサブシステム

## ログイン

ログインするには、JSONを `/api/login` に以下のような構造でPOSTしてください:

```
{
    "User": "username",
    "Pass": "password"
}
```

と表示され、サーバーはログインが成功したかどうかを示すために次のように応答します:

```
{
  "LoginStatus":true,
  "JWT":"reallylongjsonwebtokenstringishere"
}

```

ログインに失敗した場合、サーバーは "reason "プロパティを持つ構造を返します:
```
{
  "LoginStatus":false,
  "Reason":"Invalid username or password"
}
```

JSONを送信する代わりに、ログインPOSTリクエストにフォームフィールド「User」「Pass」を設定することも可能です

## ログアウト

* PUT /api/logout - 現在のインスタンスをログアウト
* DELETE /api/logout - 全ユーザーのインスタンスをログアウト

## JWT保護は、ファイルダウンロード操作に使用されないすべてのリクエストに対して実施されます。
ログインAPIから受け取ったJWTは、他のすべてのAPIリクエストのAuthorization Bearerヘッダとして含める必要があります。

```Authorization: Bearer reallylongjsonwebtokenstringishere```

###  ウェブソケット認証

便宜上、ウェブソケット API のエンドポイントは `Sec-Websocket-Protocol` ヘッダの値でJWTトークンも探します。 多くのウェブソケット実装はヘッダ値の受け渡しを適切にサポートしていないので、ウェブソケットサブプロトコルのネゴシエーションヘッダをオーバーロードしています。 API のエンドポイントでは、標準の `Authentication` ヘッダーの値も同様に探します。

## アクティブなセッションを取得する
`api/users/{id}/sessions` に GET を送ると、JSON のチャンクが返されます。 管理者はすべてのユーザのセッションをリクエストできますが、ユーザは自分のセッションのみをリクエストすることができます。

```
{
    "Sessions": [
        {
            "LastHit": "2020-08-04T15:28:12.601899275-06:00",
            "Origin": "127.0.0.1",
            "Synced": false,
            "TempSession": false
        },
        {
            "LastHit": "2020-08-03T23:59:53.807610997-06:00",
            "Origin": "127.0.0.1",
            "Synced": false,
            "TempSession": false
        },
        {
            "LastHit": "2020-08-04T09:45:48.291770859-06:00",
            "Origin": "127.0.0.1",
            "Synced": false,
            "TempSession": false
        }
    ],
    "UID": 1,
    "User": "admin"
}
```
