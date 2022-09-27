# 検索ライブラリ

REST API located at /api/library

検索ライブラリAPIは、保存された検索クエリを保存・取得するために使用されます。 検索ライブラリーは、名前、メモ、タグなど、日々の業務で価値のあるクエリーを構築するのに便利なシステムです。

検索ライブラリはパーミッション制のAPIで、各ユーザーが自分の検索ライブラリを所有し、オプションでグループと共有することができます。 各ライブラリエントリはグローバルフラグを持ち、どのユーザもそのライブラリエントリを読むことができます。 グローバルフラグの設定は、管理者のみ可能です。

ライブラリーのエントリーを削除できるのはオーナーだけで、他のユーザーがグループメンバーシップによってライブラリーのエントリーにアクセスできる場合でも、エントリーを削除することはできません。

## 管理者操作

管理者は他のすべてのユーザーと同じように検索ライブラリを操作します。 もし管理者が自分の所有外のライブラリエントリの一覧表示、削除、修正など、ライブラリAPIへの管理アクセスが必要な場合は、リクエストに `admin` フラグを付加する必要があります。

例えば、`/api/library`に対してGETリクエストを行うと、呼び出したユーザーのライブラリエントリのみが返されます（管理者かどうかは関係ありません）。 同じGETリクエストを `/api/library?admin=true` に実行すると、すべてのユーザーのすべてのライブラリエントリが返されます。 adminフラグは、adminでないユーザーには無視されます。

## 基本API概要

ライブラリAPIは `/api/library` にルートがあり、以下のリクエストメソッドに応答する:

| method | description | Supports Admin Calls |
| ------ | ----------- | -------------------- |
| GET    | Retrieve a list of available library entries | TRUE |
| POST   | Add a new library entry | FALSE |
| PUT    | Update an existing library entry | TRUE |
| DELETE | Delete an existing library entry | TRUE |

すべての検索ライブラリ・エントリには、"ThingUUID "と "GUID "の両方が関連付けられています。ThingUUID」は常に一意であり、システム上に与えられたThingUUIDを持つ検索ライブラリ・エントリは1つしか存在しません。一方、"GUID "は、ダッシュボードやactionableなどの他のものから検索ライブラリ・エントリを参照するために使用されるIDです。キットをインストールすると、すべての検索ライブラリ・エントリは、キット作成者のシステム上と同じGUIDを持ち、クロスリンクが可能になります。各ユーザーは同じGUIDを持つ検索ライブラリ・エントリを持つ可能性がありますが、それらのエントリはそれぞれ固有のThingUUIDを持つことになります。

`DELETE` と `PUT` メソッドでは、特定のライブラリエントリの "ThingUUID" または "GUID" を URL に付加することが必要です。APIではThingUUIDとGUIDを同じように使うことができますが、"admin mode "では同じGUIDでアクセスできるアイテムが複数存在する可能性があることに注意してください。例えば、`5f72d51e-d641-11e9-9f54-efea47f6014a`というGUIDのエントリーを更新するには、 `/api/library/5f72d51e-d641-11e9-9f54-efea47f6014a` に対して、リクエスト本文にエンコードした完全なエントリー構造で `PUT` リクエストが発行されるだろう。

GUIDが存在しない場合、`GET`メソッドは利用可能なすべてのエントリのリストを返します。 もしユーザーがGUIDで指定された特定のエントリにアクセスできない場合、ウェブサーバーは403のレスポンスを返します。

ライブラリエントリの構造は次のとおりです:

```
struct {
	ThingUUID   uuid.UUID
	GUID        uuid.UUID
	UID         int32
	GIDS        []int32
	Global      boolean
	Name        string
	Description string
	Query       string
	Labels      []string
	Metadata    RawObject
}
```

構造体のフィールドは以下の通りです:

| Member      | Description                     | Omitted if Empty |
| ----------- | ------------------------------- | ---------------- |
| ThingUUID   | 一位な識別子 |                  |
| GUID        | グローバルな "名前 "で、キットのインストールを越えて永続化します。複数のユーザーが同じGUIDを持つ検索ライブラリ・エントリを持つことができます。 | |
| UID         | 所有者ID           |                  |
| GIDs        | エントリが共有されているグループのGID | X |
| Global      | エントリーがグローバルに読み取り可能かどうかを示すブール値 | |
| Name        | クエリの名前 | |
| Description | クエリの詳細説明 | |
| Query       | クエリ文字列 | |
| Labels      | ラベル名 | X |
| Metadata    | 任意のパラメータを格納するために使用される不透明なJSONブロブで、有効なJSONであればよく、特定の構造を持つことはありません。 | X |

ここでは、UID 1のユーザーが所有し、3つのグループと共有しているエントリーの例を示します。 UIDとGIDの値は、ユーザーAPIを使用してユーザー名とグループ名にマップバックする必要があります:

```
{
	"ThingUUID": "69755a85-d5b1-11e9-89c2-0242ac130005",
	"GUID": "ae132ecc-88dd-11ea-a6aa-373f4c2439d4",
	"UID": 1,
	"GIDs": [1, 3, 5],
	"Global": false,
	"Name": "syslog counting query",
	"Description": "A simple chart that shows total syslog over time",
	"Query": "tag=syslog syslog Appname | stats count by Appname | chart count by Appname",
	"Labels": [
		"syslog",
		"chart",
		"trending"
	],
	"Metadata": {}
}

```


## API連携

このセクションは、検索ライブラリAPIエンドポイントとのインタラクションの例を含んでいます。 これらの例は、DatalaiQ CLI で `-debug` フラグを使用して生成されました。

### 新しいエントリを作成する

Request:
```
POST /api/library
{
	"GIDs": [1, 2],
	"Global": false,
	"Name": "netflow agg",
	"Description": "Total traffic using netflow data",
	"Query": "tag=netflow netflow Bytes | stats sum(Bytes) as TotalTraffic | chart TotalTraffic",
	"Labels": [
		"netflow",
		"traffic",
		"aggs"
	]
}
```

備考: ここでは「GUID」フィールドが設定されていないため、システムで割り当てられます。また、リクエストにGUIDを含めることもできます。

Response:
```
{
	"ThingUUID": "c9169d15-d643-11e9-99d3-0242ac130005",
	"GUID": "ae132ecc-88dd-11ea-a6aa-373f4c2439d4",
	"UID": 1,
	"GIDs": [1, 2],
	"Global": false,
	"Name": "netflow agg",
	"Description": "Total traffic using netflow data",
	"Query": "tag=netflow netflow Bytes | stats sum(Bytes) as TotalTraffic | chart TotalTraffic",
	"Labels": [
		"netflow",
		"traffic",
		"aggs"
	]
}

```
### エントリの取得

Request:

```
GET http://172.19.0.5:80/api/library
```

Response:
```
[
	{
		"ThingUUID": "0b5a66cb-d642-11e9-931c-0242ac130005",
		"GUID": "ae132ecc-88dd-11ea-a6aa-373f4c2439d4",
		"UID": 1,
		"Global": false,
		"Name": "netflow agg",
		"Description": "Total traffic using netflow data",
		"Query": "tag=netflow netflow Bytes | stats sum(Bytes) as TotalTraffic | chart TotalTraffic",
		"Labels": [
			"netflow",
			"traffic",
			"aggs"
		],
		"Metadata": {
			"value": 1,
			"extra": "some extra field value"
		}
	},
	{
		"ThingUUID": "69755a85-d5b1-11e9-89c2-0242ac130005",
		"GUID": "d57611be-88dd-11ea-a94d-df6bfb56a8a8",
		"UID": 1,
		"Global": false,
		"Name": "test2",
		"Description": "testing second",
		"Query": "tag=foo grep bar",
		"Labels": [
			"foo",
			"bar",
			"baz"
		],
	}
]
```

### 特定のエントリを取得する

Request:

```
GET http://172.19.0.5:80/api/library/ae132ecc-88dd-11ea-a6aa-373f4c2439d4
```

Response:
```
{
	"ThingUUID": "0b5a66cb-d642-11e9-931c-0242ac130005",
	"GUID": "ae132ecc-88dd-11ea-a6aa-373f4c2439d4",
	"UID": 1,
	"Global": false,
	"Name": "netflow agg",
	"Description": "Total traffic using netflow data",
	"Query": "tag=netflow netflow Bytes | stats sum(Bytes) as TotalTraffic | chart TotalTraffic",
	"Labels": [
		"netflow",
		"traffic",
		"aggs"
	],
	"Metadata": {
		"value": 1,
		"extra": "some extra field value"
	}
}
```

Note that you would get the same response from `api/library/0b5a66cb-d642-11e9-931c-0242ac130005` too.

### エントリを更新する

Request:
```
PUT /api/library/69755a85-d5b1-11e9-89c2-0242ac130005
{
	"ThingUUID": "69755a85-d5b1-11e9-89c2-0242ac130005",
	"GUID": "d57611be-88dd-11ea-a94d-df6bfb56a8a8",
	"Global": false,
	"Name": "SyslogAgg",
	"Description": "Updated Syslog aggregate",
	"Query": "tag=syslog length | stats sum(length) | chart sum",
	"Labels": [
		"syslog",
		"agg",
		"totaldata"
	],
	"Metadata": {}
}
```

Response:
```
{
	"ThingUUID": "69755a85-d5b1-11e9-89c2-0242ac130005",
	"GUID": "d57611be-88dd-11ea-a94d-df6bfb56a8a8",
	"UID": 1,
	"Global": false,
	"Name": "SyslogAgg",
	"Description": "Updated Syslog aggregate",
	"Query": "tag=syslog length | stats sum(length) | chart sum",
	"Labels": [
		"syslog",
		"agg",
		"totaldata"
	]
}
```

### エントリを削除する

Request:
```
DELETE /api/library/69755a85-d5b1-11e9-89c2-0242ac130005
```

### エントリの管理者権限削除

非管理者が管理者フラグを付加した場合、ウェブサーバはそのフラグを無視します。非管理者が指定されたエントリの所有者である場合、アクション (この場合は DELETE) はそのまま機能します。 非管理者がそのエントリを所有していない場合、ウェブサーバは 403 StatusForbidden で応答します。

管理者削除を行う場合、URLパラメータに必ずThingUUIDを使用しないと、間違ったアイテムが削除される可能性があります。

Request:
```
DELETE /api/library/69755a85-d5b1-11e9-89c2-0242ac130005?admin=true
```
