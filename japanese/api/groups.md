# グループAPI

このページでは、ユーザーグループと対話するためのAPIについて説明します。

`api/groups` や `/api/groups/` パス下の他のURLに送られるリクエストは、ユーザーアカウントに対して操作されます。ウェブサーバーは正常なリクエストに対してはStatusOK (200) を送信し、エラーに対しては400-500のステータスを送信します (エラーに依存します)。

備考: グループからユーザーの追加/削除は、グループAPIではなく [ユーザーアカウントAPIs](account.md) で実行されます。グループAPIは、純粋にグループの作成、削除、記述のためのものです。

## データタイプ

このページのAPIは、主にグループ詳細構造を扱います。グループ詳細構造は以下の通りです:

```
{
  GID: int,		// Numeric group ID
  Name: string, // Group name
  Desc: string, // Group description
  Synced: bool	// Tracks if group is synchronized with the datastore (can usually be ignored)
}
```

## グループを追加する（管理者のみ）

新しいグループを追加するには、 `/api/groups` に POST リクエストを送信してください。リクエストの本文には、以下のようなグループの名前と説明を定義した構造体を含める必要があります:

```
{
	"Name": "newgroup",
	"Desc": "This is the new group"
}
```

サーバーはリクエストの解析とグループの作成を試みます。成功すれば、200のステータスコードと新しいグループのGIDをレスポンスの本文に含めて応答します。

## グループを削除する（管理者のみ）

グループを削除するには、`/api/groups/{gid}` に DELETE リクエストを送信してください。

## グループ情報を更新する（管理者のみ）

グループの情報を変更するには、 `/api/groups/{gid}` にPUTリクエストを送信してください。リクエストの本文には、以下のようなグループ詳細の構造が含まれる必要があります:

```
{
	"GID": 3,
	"Name": "newname",
	"Desc": "this is the new description",
}
```

NameまたはDescフィールドが除外されている場合、現在の値が保持されます。

## 全てのグループをリスト表示する

システム上のすべてのグループのリストを取得するには、 `/api/groups` に GET リクエストを送信してください。レスポンスには、グループの詳細な構造の配列が含まれます:

```
[
    {
        "Desc": "bar",
        "GID": 1,
        "Name": "foo",
        "Synced": true
    },
    {
        "Desc": "",
        "GID": 7,
        "Name": "datalaiq-users",
        "Synced": false
    },
    {
        "Desc": "",
        "GID": 8,
        "Name": "testgroup",
        "Synced": false
    }
]
```

## グループの情報を取得する

管理者または特定のグループのメンバーは、特定のグループに関する情報を取得することができます。GETリクエストを `/api/groups/{gid}` に発行してください。レスポンスにはグループの詳細が含まれます:

```
{
    "Desc": "",
    "GID": 7,
    "Name": "datalaiq-users",
    "Synced": false
}
```

## グループに含まれるユーザーをリスト表示する

特定のグループの管理者やメンバーは `/api/groups/{gid}/members` にGETリクエストを発行することで、そのグループに属するユーザーのリストを問い合わせることができます。レスポンスとして、ユーザーの詳細な構造の配列が返されます:

```
[
    {
        "Admin": false,
        "Email": "joe@datalaiq.io",
        "Groups": [
            {
                "Desc": "bar",
                "GID": 1,
                "Name": "foo",
                "Synced": false
            },
            {
                "Desc": "",
                "GID": 7,
                "Name": "datalaiq-users",
                "Synced": false
            },
            {
                "Desc": "",
                "GID": 8,
                "Name": "testgroup",
                "Synced": false
            }
        ],
        "Locked": false,
        "Name": "Joe User",
        "Synced": false,
        "TS": "2020-08-10T14:49:30.72782227-06:00",
        "UID": 16,
        "User": "joe"
    },
    {
        "Admin": false,
        "Email": "bkeaton@example.net",
        "Groups": [
            {
                "Desc": "",
                "GID": 7,
                "Name": "datalaiq-users",
                "Synced": false
            }
        ],
        "Locked": false,
        "Name": "Buster Keaton",
        "Synced": false,
        "TS": "0001-01-01T00:00:00Z",
        "UID": 17,
        "User": "buster"
    }
]
```
