# ユーザーAPI

このページではAPIについて説明します。

`/api/users` および `/api/users/` パスの以下の他のURLに送信されるリクエストは、ユーザーアカウントで動作します。 ウェブサーバーは正常なリクエストに対して Status OK(200) を送信し、エラーに対しては  400-500 ステータスを送信します (エラーによって異なります)。

デフォルトの「admin」アカウントのユーザー名は変更または削除できません。また、アカウントを管理者ステータスから降格することもできません。 他のユーザーアカウントにも管理者権限が割り当てられている場合があります。

## データタイプ

このページのAPIは、主にユーザーとグループの詳細構造を扱います。 ユーザーの詳細な構造は次の通りです: 

```
{
  UID: int,			// Numeric user ID
  User: string,		// Username e.g. "jdoe"
  Name: string,		// User's real name e.g. "John Doe"
  Email: string,	// User's email address e.g. "john@example.org"
  Admin: bool,		// Set to true if the user has admin rights
  Locked: bool,		// Set to true if the user's account has been locked
  DefaultGID: int,	// The user's default search group
  TS: string,		// Contains a timestamp representing when the user was last active
  Synced: bool,		// Tracks if user is synchronized with the datastore (can usually be ignored)
  Groups: Array<GroupDetails>	// An array of groups to which the user belongs
}
```

グループの詳細な構造は以下の通りです:

```
{
  GID: int,		// Numeric group ID
  Name: string, // Group name
  Desc: string, // Group description
  Synced: bool	// Tracks if group is synchronized with the datastore (can usually be ignored)
}
```

## ログイン/ログアウト

Webサーバーで認証するためのAPIは [こちら](login.md)。

## 現在のユーザー情報を取得

`/api/info/whoami` にGETリクエストを送信すると、現在のユーザー情報が返ってきます。

```
{
        "UID": 1,
        "User": "admin",
        "Name": "Sir changeme of change my password the third",
        "Email": "admin@admin.admin",
        "Admin": true,
        "Locked": false,
        "TS": "2016-12-05T17:05:33.121180268-07:00",
        "DefaultGID": 0,
        "Groups": [
                {
                        "GID": 1,
                        "Name": "foo",
                        "Desc": "This is the foo group"
                }
        ]
}

```

## 全てのアカウントをリスト表示

`/api/users` にGETリクエストを送ると、それぞれのユーザー情報がJSON形式で返ってきます。JSONのレスポンスは以下のようになります:

```
[
    {
        "Admin": true,
        "Email": "admin@admin.admin",
        "Groups": [
            {
                "Desc": "bar",
                "GID": 1,
                "Name": "foo",
                "Synced": false
            }
        ],
        "Locked": false,
        "Name": "Admin John",
        "Synced": true,
        "TS": "2020-07-30T08:51:35.205998608-06:00",
        "UID": 1,
        "User": "admin"
    },
    {
        "Admin": false,
        "Email": "john@example.net",
        "Groups": [
            {
                "Desc": "bar",
                "GID": 1,
                "Name": "foo",
                "Synced": false
            }
        ],
        "Locked": false,
        "Name": "John",
        "Synced": false,
        "TS": "2020-08-10T14:47:44.58356179-06:00",
        "UID": 14,
        "User": "john"
    },
    {
        "Admin": false,
        "Email": "joe@datalaiq.io",
        "Groups": [
            {
                "Desc": "",
                "GID": 7,
                "Name": "datalaiq-users",
                "Synced": false
            }
        ],
        "Locked": false,
        "Name": "Joe User",
        "Synced": false,
        "TS": "2020-08-10T14:49:30.72782227-06:00",
        "UID": 16,
        "User": "joe"
    }
]
```

## 単一ユーザーの情報を取得

単一ユーザーの情報を取得するには `/api/users/{id}` にGETリクエストを送信します。管理者は任意のユーザー情報を取得できますが、それ以外のユーザーは自身のユーザー情報のみ取得できます。ステータス200であればJSONフォーマットで情報が、400-500であればエラーメッセージが返ってきます。バックエンドは、このページの上部で説明されているように、ユーザーの詳細を含むJSONフォーマットで応答します。 応答の例は次のとおりです:

```
{
    "Admin": true,
    "DefaultGID": 1,
    "Email": "admin@admin.admin",
    "Groups": [
        {
            "Desc": "bar",
            "GID": 1,
            "Name": "foo",
            "Synced": false
        }
    ],
    "Locked": false,
    "Name": "Admin",
    "Synced": false,
    "TS": "2020-07-30T08:51:35.205998608-06:00",
    "UID": 1,
    "User": "admin"
}
```

## ユーザーを追加

ユーザーを追加するには、管理者アカウントで `/api/users` に以下のフィールドデータを一緒にPOSTリクエストで送信します。

```
{
     User: "buster",
     Pass: "gr4vwellRulez",
     Name: "Buster Keaton",
     Email: "bkeaton@example.net"
     Admin: false,
}
```

すべてのフィールドに入力する必要があります。 バックエンドは、問題なければ新しいユーザーのUIDで応答します。 例: 17。

## ユーザーアカウントロック

ユーザーアカウントをロックするには `/api/users/{id}/lock` に空のPUTリクエストを送信します。ユーザーをロックすると、ユーザーは直ちに全てのセッションからログアウトされアンロックされるまでログインできなくなります。

## ユーザーアカウントアンロック

ユーザーをアンロックするには `/api/users/{id}/lock` に空のDELETEリクエストを送信します。アクションが許可されている場合、ユーザーが実際にロック解除されたかどうかに関係なく、Web サーバーは成功を返します。これは、ロックされたユーザーをロックすると、ユーザーがロックされた状態で終了するためです。アンロックも同様です。

## ユーザー情報編集

ユーザー情報を編集するには `/api/users/{id}` PUTリクエストを編集したいフィールド情報のデータと一緒に送信します: 

```
{
     User: "chuck",
     Name: "Chuck Testa",
     Email: "chuck@testa.net"
}
```

入力されていないフィールドに対する変更は無視されます。上記のように、バックエンドは標準の応答JSONフォーマットで応答します。もし現在のユーザーが管理者ではなく、また自分自身のユーザーでなかった場合はリクエストが拒否されます。管理者は任意のユーザー情報を変更できます。初期管理者ユーザー (UID 1) は、その管理者ステータスを変更できません。 バックエンドは、エラーの原因と理由に応じて、成功の場合は 200、エラーの場合は 400-500 で応答します。

## ユーザーパスワード変更

To change a password, issue a PUT to the url `/api/users/{id}/pwd`.
パスワードを変更するには `/api/users/{id}/pwd` にPUTリクエストを以下のフィールドデータと一緒に送信します。

```
{
     OrigPass: "my old password was bad",
     NewPass: "thisis mynewpassword",
}
```

備考: 管理者ユーザーが別のユーザーのパスワードを変更する場合は、 OrigPass フィールドは必要ありません。

## ユーザー削除

ユーザーを削除するには `/api/users/{id}/` にDELETEリクエストを送信します。管理者のみこの機能を使用することができ、各ユーザーは自身のユーザーを削除することはできません。初期管理者ユーザー（UID 1）は削除することはできません。

## ユーザーの管理者権限を設定/取得

管理者権限ステータスを変更するには `/api/users/{id}/admin` にbodyを空にしてリクエストを送信します。リクエストメソッドは行いたいアクションによって変わります。Webサーバーはエラーの原因と理由に応じて、成功の場合は 200、エラーの場合は 400-500 で応答します。

* GET - 現在の管理者権限ステータスを返す
* PUT - ユーザーに管理者権限を与え、現在のステータスを返す
* DELETE - ユーザーから管理者権限を解除する

成功時のJSONレスポンス例:

```
{
     UID: 1,
     Admin: true
}
```

## ユーザーのセッションを取得

ユーザーは必要に応じて `/api/users/{id}/sessions` にGETリクエストを送信することで、自身のセッション情報を取得することができます。管理者は任意のUIDに対してセッション情報を取得することができます。レスポンスは以下のようにセッションオブジェクトの配列の形で返ってきます:

```
{
    "Sessions": [
        {
            "LastHit": "2020-08-11T14:41:23.829366-06:00",
            "Origin": "::1",
            "Synced": false,
            "TempSession": false
        }
    ],
    "UID": 1,
    "User": "admin"
}
```

## デフォルトの検索（クエリ）グループを設定

各ユーザーはデフォルトの検索（クエリ）グループを設定できます。グループがセットされると、各ユーザーによって実行されたクエリはグループに共有されます。デフォルトのグループを設定するには以下のように `/api/users/{id}/searchgroup` にグループを指定するフィールドデータと一緒にPUTリクエストを送信します:

```
{
	"GID": 3
}
```

## デフォルトの検索（クエリ）グループを取得

ユーザーの検索（クエリ）グループを取得するには `/api/users/{id}searchgroup` にGETリクエストを送信します。サーバーはGIDをレスポストして返します。


## ユーザー設定

「ユーザー設定」は、ユーザーのフロントエンドによって設定された任意の JSON blob で構成されます。これにより、フロントエンドはユーザーの UI 設定 (配色など) を複数のコンピューターに保存できます。 ユーザー設定は、1) 自身のユーザー、または 2) 管理者ユーザーのみが設定できます。

### ユーザー設定

ユーザー設定を設定するには `/api/users/{id}/preferences` に設定したいフィールドデータと一緒にPUTリクエストを送信します。


### ユーザー設定取得

ユーザー設定情報を取得するには `/api/users/{id}preferences` にGETリクエストを送信します。応答のbodyには、以前に設定された設定が含まれます。

### ユーザー設定初期化

ユーザー設定を初期化するには `/api/users/{id}preferences` にDELETEリクエストを送信します。

### 全てのユーザー設定を取得（管理者のみ実行可）

管理者は `/api/users/preferences` にGETリクエストを送信することで、全てのユーザー設定情報を取得することができます。応答には、ユーザー設定とユーザー*email*構成の両方が含まれます。 電子メール設定を設定および変更するためのインターフェースは、別の場所で説明されています:

```
[
    {
        "Data": "ImJhciBiYXogcXV1eCBhc2Rmc2FkZmRzZiI=",
        "Name": "prefs",
        "Synced": false,
        "UID": 1,
        "Updated": "2020-07-13T13:50:08.286015047-06:00"
    },
    {
        "Data": "ImZvbyI=",
        "Name": "emailSettings",
        "Synced": false,
        "UID": 1,
        "Updated": "2019-12-18T09:45:38.683780752-07:00"
    }
]
```

## グループにユーザーを追加

管理者は `/api/users/{id}/group` にPOSTリクエストを送信することで、ユーザーをグループに追加できます。リクエストbodyには追加したいグループのGIDが含まれている必要があります。以下のように、GID 8のグループに追加したい場合は以下のようになります:

```
{
	"GIDs": [8]
}
```

## グループからユーザーを削除

管理者は `/api/users/{id}/group/{gid}` にDELETEリクエストを送信することで、グループからユーザーを削除できます。

## ユーザーのグループを取得

ユーザー（もしくは管理者）は `/api/users/{id}/group` にGETリクエストを送信することで以下のように所属するグループを取得することができます:

```
[
    {
        "Desc": "foo group",
        "GID": 1,
        "Name": "foo",
        "Synced": false
    }
]
```
