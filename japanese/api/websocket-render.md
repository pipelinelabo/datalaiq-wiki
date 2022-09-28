# レンダーモジュールAPI

レンダーモジュールは、検索に関する情報、検索の実際のエントリー、検索に関する統計情報を取得するためのAPIを提供します。APIコマンドは、検索が開始されたときに確立されたwebsocketサブプロトコル上でJSONとして送信されます。サブプロトコルや検索の開始に関する情報は、[クエリAPIドキュメント](#!api/websocket-search.md)をご覧ください。

これらの記事の内容を超えて、レンダーモジュールのコマンドがどのように動作するかを確認する最も簡単な方法は、ウェブブラウザのコンソール（Chrome では F12）を使用する方法です。Websocket トラフィックを表示するセクションを探し、送受信されるメッセージを閲覧します:

![](webconsole.png)

ウェブソケットのトラフィックを観察するもう一つの方法は、[DatalaiQ CLI](#!cli/cli.md)を `-debug` フラグ付きで実行する方法です。このフラグは引数としてファイル名を取ります。ウェブソケットに送信されたJSONメッセージやウェブソケットから読み込まれたJSONメッセージは、そのファイルに書き込まれます。

## すべてのレンダーモジュールに共通するオペレーションID

すべてのレンダーモジュールは、以下のリクエストに応答します:

| Request name | Hex value | Decimal value | Description |
|--------------|-----------|---------------|-------------|
| REQ_CLOSE				| 0x1			| 1			| チャンネルを閉じる |
| REQ_ENTRY_COUNT		| 0x3			| 3			| 見たことのあるエントリー数を取得する|
| REQ_SEARCH_DETAILS	| 0x4			| 4			| 検索に関する詳細情報を取得する |
| REQ_SEARCH_TAGS		| 0x5			| 5			| 検索に使用したタグマップを取得する |
| REQ_GET_ENTRIES		| 0x10			| 16		| インデックスによる検索エントリーのブロックを要求する |
| REQ_STREAMING			| 0x11			| 17		| 検索エントリーを順次送信するよう依頼する |
| REQ_TS_RANGE			| 0x12			| 18		| 時間帯別のブロック単位で要求する |
| REQ_GET_EXPLORE_ENTRIES	| 0xf010	| 61456		| [データエクスプローラー](explorer.md)を適用し、インデックスによるエントリーのブロックを要求する。 |
| REQ_EXPLORE_TS_RANGE		| 0xf012	| 61458		| [データエクスプローラー](explorer.md)を適用して、時間範囲ごとにブロック化したエントリーをリクエストする。 |
| REQ_STATS_SIZE		| 0x7F000001	| 2130706433| 統計データのサイズを要求する |
| REQ_STATS_RANGE		| 0x7F000002	| 2130706434| 利用可能な統計情報の時間範囲を要求する |
| REQ_STATS_GET			| 0x7F000003	| 2130706435| 統計情報を要求する |
| REQ_STATS_GET_RANGE	| 0x7F000004	| 2130706436| 特定の時間範囲から統計情報を要求する |
| REQ_STATS_GET_SUMMARY	| 0x7F000005	| 2130706437| 統計の要約を請求する |
| REQ_SEARCH_METADATA   | 0x10001 | 65537 | クエリの列挙値メタデータの統計情報を要求します |

レスポンス値はリクエストと同じであるが、特別なレスポンスコード `RESP_ERROR` (0xFFFFFF) が追加されます。

| Response name | Hex value | Decimal value | Description |
|--------------|-----------|---------------|-------------|
| RESP_ERROR				| 0xFFFFFFFF	| 4294967295| (error) |
| RESP_CLOSE				| 0x1			| 1			| ソケットが閉じられます |
| RESP_ENTRY_COUNT			| 0x3			| 3			| エントリー数表示 |
| RESP_SEARCH_DETAILS		| 0x4			| 4			| 検索に関する情報を返す |
| RESP_SEARCH_TAGS			| 0x5			| 5			| 検索用タグマップの返送 |
| RESP_GET_ENTRIES			| 0x10			| 16		| 検索項目を返す|
| RESP_STREAMING			| 0x11			| 17		| サーチエントリーをストリーミング配信します|
| RESP_TS_RANGE				| 0x12			| 18		| 時間範囲内のエントリーのブロックを返す |
| RESP_GET_EXPLORE_ENTRIES	| 0xf010	| 61456		| [データエクスプローラ-](explorer.md)を適用し、インデックスによるエントリーのブロックを返す。 |
| RESP_EXPLORE_TS_RANGE		| 0xf012	| 61458		| [データエクスプローラー](explorer.md)を適用して、時間範囲ごとにブロック化したエントリーを返す。 |
| RESP_STATS_SIZE			| 0x7F000001	| 2130706433| 統計データのサイズを返す |
| RESP_STATS_RANGE			| 0x7F000002	| 2130706434| 統計情報の時間範囲を返す |
| RESP_STATS_GET			| 0x7F000003	| 2130706435| 統計情報を返す |
| RESP_STATS_GET_RANGE		| 0x7F000004	| 2130706436| 時間範囲内の統計情報を返す |
| RESP_STATS_GET_SUMMARY	| 0x7F000005	| 2130706437| 統計情報の概要を返す |
| RESP_SEARCH_METADATA           | 0x10001 | 65537 | クエリの列挙値メタデータの統計情報を返す |

APIリクエストは、検索作成時に確立された検索サブプロトコル上でJSONとして送信する必要があります。

## レスポンスフォーマット

すべてのモジュールは同じコマンドに応答しますが、関係するデータ型の性質が異なるため、エントリーを返す形式は異なります。[レンダーAPIレスポンスフォーマット](websocket-render-responses.md) には、各レンダー モジュールが使用する応答形式が記載されています。この記事の残りの部分で示す例は、テキスト レンダラーとテーブル レンダラーからの応答です。

## チャンネルを閉じる

検索への現在の接続を閉じるには、次の構造体をウェブソケットに送信します:

```
{
        "ID": 1
}
```

以下のようなレスポンスになるでしょう:

```
{
        "ID": 1,
        "EntryCount": 0,
        "AdditionalEntries": false,
        "Finished": false,
        "Entries": []
}
```

## エントリ数を取得する

REQ_ENTRY_COUNTコマンドは、レンダーモジュールに到達したエントリーの総数を要求します:

```
{
        "ID": 3
}
```

この例では、モジュールは41のエントリーがあると回答している:

```
{
        "ID": 3,
        "EntryCount": 41,
        "AdditionalEntries": false,
        "Finished": true
}
```

注意: ここで報告される数は、**レンダラーが受信したエントリーの総数です**。REQ_GET_ENTRIESなどの他のコマンドを使用すると、返されるエントリが少なくなることがあります。これは、一部のレンダラー（表やグラフなど）が*凝縮*レンダラーであるためです。

## 検索の詳細を取得する

REQ_SEARCH_DETAILSコマンド（0x4）は、検索そのものに関する情報を要求するコマンドである:

```
{
        "ID": 4
}
```

レスポンスには、統計情報と検索そのものに関する情報が含まれる:

```
{
	"ID": 4,
	"Stats": {
		"Size": 0,
		"Set": [
			{
				"TS": "2018-04-02T07:56:47.422345249-06:00",
				"Stats": []
			},
<some entries elided for brevity>
			{
				"TS": "2018-04-02T10:17:24.922345249-06:00",
				"Stats": [
					{
						"Name": "packet",
						"Args": "packet ipv4.SrcIP",
						"InputCount": 1619,
						"OutputCount": 1619,
						"InputBytes": 2034920,
						"OutputBytes": 2052729,
						"Duration": 54936015
					},
					{
						"Name": "count",
						"Args": "count by SrcIP",
						"InputCount": 1619,
						"OutputCount": 88,
						"InputBytes": 2052729,
						"OutputBytes": 28931,
						"Duration": 129787584
					}
				]
			},
<some entries elided for brevity>
			{
				"TS": "2018-04-02T12:47:24.922345249-06:00",
				"Stats": []
			},
			{
				"TS": "2018-04-02T12:52:06.172345249-06:00",
				"Stats": [
					{
						"Name": "packet",
						"Args": "packet ipv4.SrcIP",
						"InputCount": 1,
						"OutputCount": 1,
						"InputBytes": 99,
						"OutputBytes": 110,
						"Duration": 14480829
					},
					{
						"Name": "count",
						"Args": "count by SrcIP",
						"InputCount": 1,
						"OutputCount": 1,
						"InputBytes": 110,
						"OutputBytes": 110,
						"Duration": 52235061
					}
				]
			}
		],
		"RangeStart": "2018-04-02T07:56:47.422345249-06:00",
		"RangeEnd": "2018-04-02T12:56:47.422345249-06:00",
		"Current": "2018-04-02T07:56:47.422345249-06:00"
	},
	"SearchInfo": {
		"ID": "677124412",
		"UID": 1,
		"UserQuery": "tag=pcap packet ipv4.SrcIP | count by SrcIP | table SrcIP count",
		"EffectiveQuery": "tag=pcap packet ipv4.SrcIP | count by SrcIP | table SrcIP count",
		"StartRange": "2018-04-02T07:56:47.422345249-06:00",
		"EndRange": "2018-04-02T12:56:47.422345249-06:00",
		"Descending": true,
		"Started": "2018-04-02T12:56:47.430572676-06:00",
		"LastUpdate": "0001-01-01T00:00:00Z",
		"StoreSize": 75960,
		"IndexSize": 32,
		"ItemCount": 1575,
		"TimeZoomDisabled": false,
		"RenderDownloadFormats": [
			"json",
			"csv",
			"lookupdata"
		],
		"Duration": "0s",
		"Tags": ["pcap"]
	},
	"EntryCount": 1575,
	"AdditionalEntries": false,
	"Finished": true
	"OverLimit":false,
	"LimitDroppedRange":{"StartTS":"0000-12-31T16:07:02-07:52","EndTS":"0000-12-31T16:07:02-07:52"},
}
```

## タグマッピングを取得する

タグマップを取得するには、標準的なウェブソケットサブプロトコルでリクエストID 0x5を送信します:

```
{
        "ID": 0x5,
}
```

タグの名前とタグのインデックスの数値の対応表は以下のようになります:

```
{
        "ID": 0x5,
        "Tags": {
                "default": 0,
				"tagmctaggy": 1,
				"apache": 2,
				"syslog: 3,
				"gravwell": 4
			},
}
```

## エントリを取得する

リクエスト0x10(10進数16)はレンダラーにエントリーのブロックを要求します。`First` と `Last` フィールドを使用して、希望するエントリーを指定します。レンダラーは、REQ_ENTRY_COUNT (0x3) リクエストに応答して、エントリーの数を報告します。

このコマンドは、最初の1024個のエントリーを要求する（`First`は指定されない場合、デフォルトで0になる）:

```
{
	"ID": 16,
	"EntryRange": {
		"Last": 1024
	}
}
```

サーバーは、エントリーの配列と追加情報を応答する。

```
{
	"ID": 16,
	"EntryCount": 1575,
	"AdditionalEntries": false,
	"Finished": true,
	"OverLimit":false,
	"LimitDroppedRange":{"StartTS":"0000-12-31T16:07:02-07:52","EndTS":"0000-12-31T16:07:02-07:52"},
	"Entries": {
		"Rows": [
			{
				"TS": "2018-04-02T10:30:29-06:00",
				"Row": [
					"10.144.162.236",
					"9410"
				]
			},
<67 similar entries elided>
			{
				"TS": "2018-04-02T10:28:51-06:00",
				"Row": [
					"192.168.1.1",
					"2"
				]
			}
		],
		"Columns": [
			"SrcIP",
			"count"
		]
	}
}
```

`AdditionalEntries": false` フィールドに注目してください。これは、これ以上読み込むべきエントリがないことを意味します。もしこのフィールドが true に設定されて戻ってきたら、`First` を 1024 に、`Last` を 2048 に設定してコマンドを再発行し、利用できるエントリがなくなるまで繰り返すことで、さらにエントリを読み込むことができます。

## ストリーム結果を取得する

要求 ID 0x11 を送信すると、クライアントはレンダラーに Web ソケット経由でできるだけ早くエントリを送信するように要求できます。これは、エントリーがすぐにディスクに書き込まれるか、その他の簡単な操作でない限り、一般に推奨されません。レンダー モジュールは、クライアントが処理できるよりも速く結果を出力することがよくあります。結果をブロックごとに取得するには、REQ_GET_ENTRIES コマンドを使用することをお勧めします。

以下のようにリクエストします:

```
{
	"ID": 17
}
```

レンダラーは、大きなエントリーのブロックを出来るだけ早く送信し始めます:

```
{
	"ID": 17,
	"EntryCount": 1000,
	"AdditionalEntries": false,
	"Finished": false,
	"OverLimit":false,
	"LimitDroppedRange":{"StartTS":"0000-12-31T16:07:02-07:52","EndTS":"0000-12-31T16:07:02-07:52"},
	"Entries": [
<1000 entries elided>
	]
}
<many other entry blocks elided>
{
	"ID": 17,
	"EntryCount": 861,
	"AdditionalEntries": false,
	"Finished": false,
	"OverLimit":false,
	"LimitDroppedRange":{"StartTS":"0000-12-31T16:07:02-07:52","EndTS":"0000-12-31T16:07:02-07:52"},
	"Entries": [
<861 entries elided>
	]
}
{
	"ID": 17,
	"EntryCount": 0,
	"AdditionalEntries": false,
	"Finished": true,
	"OverLimit":false,
	"LimitDroppedRange":{"StartTS":"0000-12-31T16:07:02-07:52","EndTS":"0000-12-31T16:07:02-07:52"},
	"Entries": []
}
```

この場合、レンダラーは1000エントリーのブロックを多数送信し、次に残りの861ブロックを含むブロックを送信し、最後に0エントリーを含むブロックを送信しました。この0エントリーの最終ブロックは、これ以上エントリーを送信しないことを示すものである。

注意: ストリーミング結果を有効にする場合は、十分な注意を払うようにしてください。

## 特定の時間範囲のエントリを取得する

検索範囲の特定の部分のエントリーを取得するには、リクエスト0x12 (REQ_TS_RANGE) を使用します。大量のエントリが存在する可能性があるので、 REQ_GET_ENTRIES と同様に `First` と `Last` フィールドを使用して、一度にひとつのブロック (この場合は最初の 100 エントリ) をフェッチします:

```
{
	"ID":18,
	"EntryRange": {
		"First":0,
		"Last":100,
		"StartTS":"2018-04-02T16:19:51.579Z",
		"EndTS":"2018-04-02T16:42:28.649Z"
	}
}
```

サーバーは、要求された時間内に収まるエントリで応答します:

```
{
	"ID":18,
	"EntryCount":1575,
	"AdditionalEntries":false,
	"Finished":true,
	"OverLimit":false,
	"LimitDroppedRange":{"StartTS":"0000-12-31T16:07:02-07:52","EndTS":"0000-12-31T16:07:02-07:52"},
	"Entries": {
		"Rows": [
			{
				"TS":"2018-04-02T10:30:29-06:00",
				"Row":["10.194.162.236","9410"]
			},
			{
				"TS":"2018-04-02T10:30:36-06:00",
				"Row":["192.168.1.101","8212"]
			},
<entries elided>
			{
				"TS":"2018-04-02T10:28:51-06:00",
				"Row":["192.168.1.1","2"]
			}
		],
		"Columns":["SrcIP","count"]
	}
}
```

この場合、エントリーは100件未満でした。もしそれ以上あったなら、 `AdditionalEntries` フィールドは `true` に設定されていたでしょう。同じタイムスタンプで別のリクエストを送ることで、より多くのエントリーを取得することができますが、 `First` フィールドを 100 に、 `Last` フィールドを 200 に変更して次のブロックの 100 エントリーを取得することができます。

## 現在の統計情報のカウントを取得する

検索が実行されると、統計エントリが生成される。エントリ数は、0x7F000001 (REQ_STATS_SIZE) リクエストを使用して取得することができます:

```
{
        "ID": 2130706433,
}
```

サーバーは、統計エントリーの数を応答します:

```
{
        "ID": 2130706433,
        "Stats": {
			"Size": 466
		}
}
```

## 統計情報によってカバーされる現在の時間範囲を取得する

このコマンドは、検索統計が利用可能な時間範囲を返します。コマンドを送信します:

```
{
        "ID": 2130706434,
}
```

The server responds:

```
{
        "ID": 2130706434,
        "Stats": {
                "RangeStart": "2016-09-02T08:59:37.943271552-06:00",
                "RangeEnd": "2016-09-02T08:59:37.943271552-06:00",
                "Size": 2
        },
        "EntryCount": 510000
}
```

返されたSizeパラメータは、利用可能な最大粒度を示します。 この例では、発行された検索は 2 秒間しかカバーしていないので、 クライアントは最大粒度を 2 (あるいは 1 秒分のデータに対して 1 つの stat エントリ) にするよう要求することができます。 ウェブサーバが保持する統計エントリの最大数は65kである。ただし、検索が「完全解像度統計」で発行された場合は、メモリとストレージの許容範囲内で無制限の粒度を使用することができる。

## 統計情報セットの依頼

統計セットは、必要な「チャンク」数を指定することで要求されます。 例えば、1ヶ月分のデータを検索した場合、65k個の統計セットがありますが、表示には、10、100、1000の単位で、異なる粒度の統計データを表示することができます。

異なる粒度を得るには、StatsRequest で SetCount を送信します。 この例では、サイズ 1 のセット (すべてのモジュールをまとめる) を要求しています:

```
{
        "ID": 2130706435,
        "Stats": {
                "SetCount": 1
        }
}
```

レスポンスには、Statsの項目が1つだけ含まれています:

```
{
        "ID": 2130706435,
        "Stats": {
                "RangeStart": "0001-01-01T00:00:00Z",
                "RangeEnd": "0001-01-01T00:00:00Z",
                "Set": [
                        {
                                "Stats": [
                                        {
                                                "Name": "grep",
                                                "Args": "grep HEE",
                                                "InputCount": 510000,
                                                "OutputCount": 510000,
                                                "InputBytes": 52363340,
                                                "OutputBytes": 52363340,
                                                "Duration": 0
                                        },
                                        {
                                                "Name": "sort",
                                                "Args": "sort by time",
                                                "InputCount": 510000,
                                                "OutputCount": 500000,
                                                "InputBytes": 52363340,
                                                "OutputBytes": 51344450,
                                                "Duration": 0
                                        }
                                ],
                                "TS": "2016-09-02T08:59:37.943271552-06:00"
                        }
                ],
                "Size": 2
        },
        "OverLimit":false,
        "LimitDroppedRange":{"StartTS":"0000-12-31T16:07:02-07:52","EndTS":"0000-12-31T16:07:02-07:52"},
        "EntryCount": 510000
}
```

## 特定の時間範囲での統計セットを要求する

この例では、2016-09-09T06:02:14Z から 2016-09-09T06:02:16Z の間のサイズ 1 のセット（すべてのモジュールをまとめる）を要求しています。 ここで重要なのは、SetCountの数値は、要求された範囲に渡って均一な「ChunkSize」を生成するために使用されるということです。 例えば、2016-01-09T06:02:16Z から 2016-12-09T06:02:16Z (11-ish months) までのデータがあるのに、範囲を 1901-01-01T06:02:16Z から 2016-01-01T06:02:16Z にして SetSize を 100 にすると、その範囲とサイズの "ChunkSize" は実際にデータがある期間よりはるかに大きいので、1つの統計サイズしか取得できないのです。

```
{
        "ID": 2130706436,
        "Stats": {
                "SetCount": 1,
				"SetStart": "2016-09-09T06:02:14Z",
				"SetEnd": "2016-09-09T06:02:16Z",
        }
}
```

Response:

```
{
        "ID": 2130706436,
        "Stats": {
                "RangeStart": "0001-01-01T00:00:00Z",
                "RangeEnd": "0001-01-01T00:00:00Z",
                "Set": [
                        {
                                "Stats": [
                                        {
                                                "Name": "grep",
                                                "Args": "grep HEE",
                                                "InputCount": 510000,
                                                "OutputCount": 510000,
                                                "InputBytes": 52363340,
                                                "OutputBytes": 52363340,
                                                "Duration": 0
                                        },
                                        {
                                                "Name": "sort",
                                                "Args": "sort by time",
                                                "InputCount": 510000,
                                                "OutputCount": 500000,
                                                "InputBytes": 52363340,
                                                "OutputBytes": 51344450,
                                                "Duration": 0
                                        }
                                ],
                                "TS": "2016-09-09T06:02:14.943271552-06:00"
                        }
                ],
                "Size": 100
        },
        "OverLimit":false,
        "LimitDroppedRange":{"StartTS":"0000-12-31T16:07:02-07:52","EndTS":"0000-12-31T16:07:02-07:52"},
        "EntryCount": 500000
}
```

## Asking for stats set summary (request 0x7F000005)

Requesting a stats set summary is equivalent to requesting a stats set of size 1:

```
{
	"ID":2130706437
}
```

Response:

```
{
    "AdditionalEntries": false,
    "EntryCount": 1575,
    "Finished": true,
    "OverLimit":false,
    "LimitDroppedRange":{"StartTS":"0000-12-31T16:07:02-07:52","EndTS":"0000-12-31T16:07:02-07:52"},
    "ID": 2130706437,
    "Stats": {
        "Set": [
            {
                "Stats": [
                    {
                        "Args": "packet ipv4.SrcIP",
                        "Duration": 61425123,
                        "InputBytes": 27364970,
                        "InputCount": 24886,
                        "Name": "packet",
                        "OutputBytes": 27636389,
                        "OutputCount": 24861
                    },
                    {
                        "Args": "count by SrcIP",
                        "Duration": 215555286,
                        "InputBytes": 27636389,
                        "InputCount": 24861,
                        "Name": "count",
                        "OutputBytes": 541861,
                        "OutputCount": 1575
                    }
                ],
                "TS": "2018-04-02T09:06:41.441-06:00"
            }
        ],
        "Size": 0
    }
}
```

## 検索のメタデータを取得する

このメッセージは、パイプラインを通過した列挙値のハイレベルな調査を要求します。以下は、クエリの例と生成されるメタデータの例です:

```
tag=syslog syslog Hostname Appname |
     length |
     stats sum(length) count by Hostname Appname |
     table Hostname Appname sum count
```


```
{
	"ID": 65537
}
```

Response:

```
{
	"ID": 65537,
	"EntryCount": 1575,
	"AdditionalEntries": false,
	"Finished": true,
    "OverLimit":false,
    "LimitDroppedRange":{"StartTS":"0000-12-31T16:07:02-07:52","EndTS":"0000-12-31T16:07:02-07:52"},
	"Metadata": {
		"ValueStats": [
			{
				"Name": "count",
				"Type": "number",
				"Number": {
					"Count": 963,
					"Min": 1,
					"Max": 3
				},
				"Raw": {
					"Map": null,
					"Other": 0
				}
			},
			{
				"Name": "Hostname",
				"Type": "raw",
				"Number": {
					"Count": 0,
					"Min": 0,
					"Max": 0
				},
				"Raw": {
					"Map": {
						"ant": 25,
						"tracker": 25,
						"voice": 22,
						"warrior": 31,
						"whale": 29
					},
					"Other": 0
				}
			},
			{
				"Name": "Appname",
				"Type": "raw",
				"Number": {
					"Count": 0,
					"Min": 0,
					"Max": 0
				},
				"Raw": {
					"Map": {
						"alpine": 4,
						"time": 2,
						"zenith": 5
					},
					"Other": 865
				}
			},
			{
				"Name": "length",
				"Type": "number",
				"Number": {
					"Count": 963,
					"Min": 314,
					"Max": 812
				},
				"Raw": {
					"Map": null,
					"Other": 0
				}
			},
			{
				"Name": "sum",
				"Type": "number",
				"Number": {
					"Count": 963,
					"Min": 314,
					"Max": 1397
				},
				"Raw": {
					"Map": null,
					"Other": 0
				}
			}
		],
		"SourceStats": [
			{
				"IP": "192.168.1.1",
				"Count": 963
			}
		],
		"TagStats": {
			"rawsyslog": 963
		}
	}
}
```
