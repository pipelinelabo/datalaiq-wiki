# プレイブックWebAPI

このAPIは、DatalaiQ内のプレイブックを操作するために使用されます。プレイブックは、メモと検索クエリをまとめて、ユーザがフォーマットしたドキュメントにするための、ユーザフレンドリーな方法です。

プレイブックの構造には、以下のコンポーネントが含まれます:

* `UUID`: インストール時に設定される、playbookの一意の識別子。
* `GUID`: プレイブックのグローバル名です。これはプレイブックのオリジナル作成者によって設定され、プレイブックがキットにバンドルされて別のシステムにインストールされても同じままです。各ユーザーは与えられたGUIDを持つ1つのプレイブックしか持つことができませんが、複数のユーザーが同じGUIDを持つプレイブックのコピーをそれぞれ持つことができます。
* `UID`: プレイブック所有者のユーザーID。
* `GIDs`: このプレイブックへのアクセスが許可されているグループIDのリストです。
* `Global`: このフラグが設定されている場合、システム上のすべてのユーザーがプレイブックを閲覧することができます。
* `Name`: プレイブックの名前。
* `Desc`: プレイブックの詳細説明。
* `Body`: プレイブックの内容を格納するバイト配列です。
* `Metadata`: クライアントが使用するための、プレイブックのメタデータを格納するバイト配列です。
* `Labels`: プレイブックに適用するオプションのラベルを含む文字列の配列。
* `LastUpdated`: プレイブックが最後に修正された日時を示すタイムスタンプです。
* `Author`: プレイブックの作者に関する情報を含む構造体（下記参照）。
* `Synced`: DatalaiQの内部で使用されています。

UUIDとGUIDフィールドは、すべてのAPIコールで互換的に使用することができることに注意してください。これは、キットに含まれるプレイブックが、キットのインストールを越えて持続するGUIDを含むリンクを使用して、互いにリンクすることができるようにするためです。

著者情報構造には以下のフィールドがあり、いずれも空白にしておくことができる:

* `Name`: 著者名
* `Email`: 著者のメールアドレス
* `Company`: 著者の会社
* `URL`: 著者についてのサイトURL

## プレイブックをリスト表示する

プレイブックを一覧表示するには、`/api/playbooks`にGETリクエストを送信してください。サーバーは、ユーザーが閲覧する権限を持つプレイブックの構造体の配列で応答します:

```
[
  {
    "UUID": "2cbc8500-5fc5-453f-b292-8386fe412f5b",
    "GUID": "c9da126b-1608-4740-a7cd-45495e8341a3",
    "UID": 1,
    "GIDs": [
      0
    ],
    "Global": false,
    "Name": "Netflow V5 Playbook",
    "Desc": "A top-level playbook for netflow, with background and starting points.",
    "Body": "",
    "Metadata": "eyJkYXNoYm9hcmRzIjpbXSwiYXR0YWNobWVudHMiOlt7ImNvbnRleHQiOiJjb3ZlciIsImZpbGVHVUlEIjoiNDhjNmIwZWYtNmU3Ni00MjA4LWJjYTctMGI5NWU0NzAwYmRkIiwidHlwZSI6ImltYWdlIn1dfQ==",
    "Labels": [
      "netflow",
      "netflow-v5",
      "kit/io.DatalaiQ.netflowv5"
    ],
    "LastUpdated": "2020-08-14T16:17:03.778971838-06:00",
    "Author": {
      "Name": "John Floren",
      "Email": "john@example.org",
      "Company": "DatalaiQ",
      "URL": "http://grawell.io"
    },
    "Synced": false
  },
  {
    "UUID": "973fcc22-1964-4efa-848c-7196ac67094e",
    "GUID": "dbd84b95-11b7-450d-9111-9bb33d63741b",
    "UID": 1,
    "GIDs": [
      0
    ],
    "Global": false,
    "Name": "Network Enrichment Kit Overview",
    "Desc": "",
    "Body": "",
    "Metadata": "eyJkYXNoYm9hcmRzIjpbXSwiYXR0YWNobWVudHMiOlt7ImNvbnRleHQiOiJjb3ZlciIsImZpbGVHVUlEIjoiOGIwZjQzMjItOTY1My00OTQyLWJkODctY2Y4ZWM5NjZmNmFmIiwidHlwZSI6ImltYWdlIn1dfQ==",
    "Labels": [
      "kit/io.datalaiq.networkenrichment"
    ],
    "LastUpdated": "2020-08-05T12:14:48.739069332-06:00",
    "Author": {
      "Name": "John Floren",
      "Email": "john@example.org",
      "Company": "DatalaiQ",
      "URL": "http://ppln.co"
    },
    "Synced": false
  }
]
```

プレイブックは非常に大きいため、すべてのプレイブックを一覧表示する際にはボディは省略されます。

URLに `?admin=true` パラメータを追加すると、ユーザーが管理者である場合に限り、システム上の *すべての* プレイブックのリストが返されます。

## プレイブックを取得する

Bodyを含む特定のプレイブックを取得するには、`/api/playbooks/<uuid>`にGETリクエストを送信してください。WebサーバーはUUIDフィールドが一致するplaybookを探そうとします。それが成功しない場合、以下の優先順位でユーザーが読むことができるplaybookを探します:

* Top precedence: ユーザーが所有しているプレイブック
* Next: グループに共有されているプレイブック
* Finally: グローバルフラグが設定されているプレイブック

## プレイブックを作成する

プレイブックは `/api/playbooks` に POST リクエストを送信することで作成されます。リクエストの本文には、ユーザーが設定したいフィールドを含める必要があります。UUID、UID、LastUpdated、Syncedフィールドが設定されている場合、サーバーはそれを無視することに注意してください。

```
{
    "Body": <contents of the playbook>,
	"Metadata": <any desired metadata>,
    "Name": "ssh syslog",
    "Desc": "A playbook for monitoring syslog entries for ssh sessions",
    "GIDs": null,
    "Global": true,
	"Author": {
		"Name": "Dean Martin"
	},
    "Labels": [
        "syslog"
    ]
}
```

サーバーは新しく作成されたプレイブックのUUIDで応答します。もし `GUID` フィールドがリクエストに設定されていれば、サーバーはそれを使用し、そうでなければ新しいものを生成します。

## プレイブックを編集する

プレイブックの内容を更新するには、`/api/playbooks/<uuid>` に PUT リクエストを送信してください。リクエストの本文には、更新するプレイブックの構造を含める必要があります。UUID、GUID、LastUpdated、およびSyncedフィールドへの変更は無視されることに注意してください。管理者はUIDフィールドを変更することができますが、一般ユーザーは変更することができません。

備考: フィールドの内容を更新するつもりがない場合は、リクエストで元の値を送信すべきです。サーバーは、例えば設定されていない "Desc" フィールドが、元の値を保持したいのか、それともそのフィールドを消去したいのかを知る術がないのです。

## プレイブックを削除する

プレイブックを削除するには、`/api/playbooks/<uuid>` に DELETE リクエストを送信します（UUID は希望のプレイブックと一致します）。