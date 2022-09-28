## ユーザー設定
User Preferences APIは、ログイン時やデバイス間で持続するGUIプリファレンスを保存するために使用されます。

/api/users/{id}/preferencesを取得すると、JSONのチャンクが返されます。 管理者はすべてのユーザのプリファレンスを要求できますが、ユーザは自分のセッションのみを要求できます。プリファレンスが存在しない場合は、nullを返します。

GET on /api/users/preferences は、すべてのユーザーのプリファレンスを返します。

ユーザー設定を更新するには、/api/users/{id}/preferencesにPUTしてください。プリファレンスが存在しない場合は、提供されたJSONを使用して更新します。このAPIでは、POSTは発生しません。PUTメソッドのペイロードは、JSON blobになります。

GETとPUTが唯一の関連するメソッドです。各ユーザーは本来、1つのプリファレンスJSONブロブを1つだけ持つべきです。

DELETE on /api/users/{id}/preferences は、プリファレンスを削除します（管理者または自分で作成した場合）。


GETで返されるJSONの例:
```json
{
     "foo": "bar",
	 "bar": "baz"
}
```

## クライアントからの例
### リクエストする設定
```
WEB GET /api/users/5/preferences:
{
        "Name": "TestPref2",
        "Value": 57005,
        "DataData": "bW9yZSBpbXBvcnRhbnQgZGF0YQ=="
}
WEB GET /api/users/1/preferences:
{
        "Name": "TestPref1",
        "Val": 1234567890,
        "Data": "some important data",
        "Things": 3.1415
}
```
### 全ての設定をリクエストする (管理者)
```
WEB GET /api/users/preferences:
[]
```
### プッシュする
```
WEB REQ PUT /api/users/1/preferences:
{
        "Name": "TestPref1",
        "Val": 1234567890,
        "Data": "some important data",
        "Things": 3.1415
}
```
```
WEB REQ PUT /api/users/5/preferences:
{
        "Name": "TestPref2",
        "Value": 57005,
        "DataData": "bW9yZSBpbXBvcnRhbnQgZGF0YQ=="
}
```
### 存在しないユーザーにプッシュする

404 not foundが表示されます。

### 非管理者ユーザーの設定をプッシュ/取得する

403 forbiddenが表示されます

### 設定を削除する
```
WEB REQ DELETE /api/users/5/preferences:
```
### 非管理者ユーザーの設定を削除する

403 forbiddenが表示されます
