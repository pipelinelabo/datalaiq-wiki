# データエクスプローラーAPI

データエクスプローラーはユーザーにクエリを書かせることなく自動的にデータを抽出・フィルタリングする機能を提供します。主にwebsocket APIのメッセージでアクセスしますが、RESTのエンドポイントも存在します。

データエクスプローラーは [自動抽出](extractors.md) の定義を使用してタグに対してどのようにデータを抽出するのかを定義します。タグの定義が存在しない場合、抽出生成 REST API を使用して自動抽出定義の候補を作成することができ、ユーザーはこれらの候補の 1 つを選択してインストールする必要があります。定義が作成されると、クライアントは検索用 Web ソケットで特別なコマンドを使用して、生の検索レンダラーから「強化」エントリを要求できるようになります。

## データ構造

### 要素

エントリから抽出されたデータは、エレメント構造体の配列として表現される。各要素はデータからの単一の「フィールド」、例えばネットフローレコードの送信元アドレスを表す。モジュールによっては、SubElementsフィールドを参照し、Elementを入れ子にして出力することがあることに注意。

* Name: Elementの名前
* Path: Elementの完全なパス指定。例えば、jsonモジュールの場合は `foo.bar.[0]` となります。
* Value: 文字列、数値、IPアドレスなど、データから抽出されたあらゆるものの値。
* SubElements: 自然なツリー構造を持つデータ型のために、この要素より "下" に位置する要素の (オプションの) 配列。
* Filters: 要素に適用可能なフィルタのリスト (例: "!=", "~", ">")。

```
interface Element {
	Name:        string;
	Path:        string;
	Value:       string | boolean | number;
	SubElements: Array<Element> | null;
	Filters:     Array<string> | null;
}
```

### Explore結果

ExploreResult 型は、特定のエントリから抽出された要素のセットを返すために使用されます。また、抽出を行ったモジュールの名前 (例: "json") や、エントリのタグの文字列名 (例: "syslog") も含まれます。

```
interface ExploreResult {
	Elements:	Array<Element>;
	Module:	string;
	Tag:		string;
}
```


### リクエスト/レスポンスのRESTエンドポイント
RESTエンドポイントでは、以下の構造体を使用します:

```
interface GenerateAXRequest {
	Tag:     string;
	Entries: Array<SearchEntry>;
}
```

```
interface GenerateAXResponse {
	Extractor:	AXDefinition;
	Entries:	Array<SearchEntry>;
	Explore:	Array<ExploreResult>;
}
```
AXタイプについては [自動抽出](autoextractors.md) のドキュメントに記載しています。

### フィルターリクエスト

FilterRequest 型は、検索クエリにフィルタを追加するために使用します。フィルタの配列を ParseSearchRequest や StartSearchRequest オブジェクトにアタッチすると、 クエリにフィルタが自動的に挿入されます。

* Tag: フィルタリングしたいタグ名(典型的にはExploreResultオブジェクトから取得されます)。
* Module: フィルタリングするモジュール名 (通常、ExploreResult オブジェクトから取得します)。
* Path: フィルタリングするElementのパス（ElementオブジェクトのPathフィールドにある）。
* Op: 使用するオプションのフィルタ操作 (Element オブジェクトの Filters 配列で指定します)。
* Value: フィルタリングするためのオプションの値（ElementオブジェクトのValueフィールドにある）。

OpとValueが指定されない場合、「フィルタ」は指定されたElementにフィルタをかけるのではなく、単に明示的にElementを抽出することに注意してください。

```
interface FilterRequest {
	Tag:    string;
	Module: string;
	Path:   string;
	Op:     string;
	Value:  string;
}
```

## 抽出生成RESTエンドポイント

データエクスプローラーでエントリーを解析する前に、そのタグに自動抽出定義をインストールする必要があります。抽出物生成エンドポイントは、タグ名と1つ以上のエントリーを受け取り、抽出可能なエントリーのコレクションを返します。これは、データエクスプローラーのモジュールごとに、可能な抽出物を 1 つずつ返します。ユーザーは最も適した抽出定義を選択するべきです（[対応する自動抽出定義をインストールする](autoextractors.md)）。

エンドポイントは `/api/explore/generate` で、GenerateAXRequest を含むボディで POST リクエストを実行する。サーバーは、(文字列)モジュール名とGenerateAXResponseオブジェクトのマッピングで応答し、それぞれがその特定のモジュールによって生成された抽出を表します。各 GenerateAXResponse オブジェクトは、Entries 配列の SearchEntry に対して Explore 配列に1つの ExploreResult オブジェクトを含む。

次のリクエストには、1つのエントリーが含まれています:

```
{
  "Tag": "foo",
  "Entries": [
    {
      "TS": "2020-11-02T16:58:56.717034109-07:00",
      "SRC": "",
      "Tag": 0,
      "Data": "ewogICJUUyI6ICIyMDIwLTEwLTE0VDEwOjM1OjQxLjEzNjUyMjg4NC0wNjowMCIsCiAgIlByb3RvIjogInVkcCIsCiAgIkxvY2FsIjogIls6Ol06NTMiLAogICJSZW1vdGUiOiAiNzMuNDIuMTA3LjE4MTo0Nzc0MiIsCiAgIlF1ZXN0aW9uIjogewogICAgIkhkciI6IHsKICAgICAgIk5hbWUiOiAicG9ydGVyLmdyYXZ3ZWxsLmlvLiIsCiAgICAgICJScnR5cGUiOiAxLAogICAgICAiQ2xhc3MiOiAxLAogICAgICAiVHRsIjogNjUsCiAgICAgICJSZGxlbmd0aCI6IDQKICAgIH0sCiAgICAiQSI6ICIyMDguNzEuMTQxLjM0IiwKCSJOb25zZW5zZSI6IFsgMTAwLCAyMDAsIDMwMCBdLAoJIk1vcmVOb25zZW5zZSI6IFsgeyJmb28iOiAiYmFyIn0gXQogIH0KfQ==",
      "Enumerated": null
    }
  ]
}

```

以下は、上記のリクエストに対するレスポンスの例です。簡潔にするため、1つのモジュール（"json"）の結果のみを掲載し、Elementsの配列は短くしています:

```
{
	"json": [
		{
			"Extractor": {
				"Name": "foo",
				"Desc": "Auto-generated JSON extraction for tag foo",
				"Module": "json",
				"Params": "TS Proto Local Remote Question",
				"Tag": "foo",
				"Labels": null,
				"UID": 0,
				"GIDs": null,
				"Global": false,
				"UUID": "00000000-0000-0000-0000-000000000000",
				"LastUpdated": "0001-01-01T00:00:00Z"
			},
			"Entries": [
				{
					"TS": "2020-11-02T16:58:56.717034109-07:00",
					"SRC": "",
					"Tag": 0,
					"Data": "ewogICJUUyI6ICIyMDIwLTEwLTE0VDEwOjM1OjQxLjEzNjUyMjg4NC0wNjowMCIsCiAgIlByb3RvIjogInVkcCIsCiAgIkxvY2FsIjogIls6Ol06NTMiLAogICJSZW1vdGUiOiAiNzMuNDIuMTA3LjE4MTo0Nzc0MiIsCiAgIlF1ZXN0aW9uIjogewogICAgIkhkciI6IHsKICAgICAgIk5hbWUiOiAicG9ydGVyLmdyYXZ3ZWxsLmlvLiIsCiAgICAgICJScnR5cGUiOiAxLAogICAgICAiQ2xhc3MiOiAxLAogICAgICAiVHRsIjogNjUsCiAgICAgICJSZGxlbmd0aCI6IDQKICAgIH0sCiAgICAiQSI6ICIyMDguNzEuMTQxLjM0IiwKCSJOb25zZW5zZSI6IFsgMTAwLCAyMDAsIDMwMCBdLAoJIk1vcmVOb25zZW5zZSI6IFsgeyJmb28iOiAiYmFyIn0gXQogIH0KfQ==",
					"Enumerated": null
				}
			],
			"Explore": [
				{
					"Elements": [
						{
							"Name": "TS",
							"Path": "TS",
							"Value": "2020-10-14T10:35:41.136522884-06:00",
							"Filters": [
								"==",
								"!="
							]
						},
						{
							"Name": "Proto",
							"Path": "Proto",
							"Value": "udp",
							"Filters": [
								"==",
								"!="
							]
						},
						{
							"Name": "Local",
							"Path": "Local",
							"Value": "[::]:53",
							"Filters": [
								"==",
								"!="
							]
						},
						{
							"Name": "Remote",
							"Path": "Remote",
							"Value": "73.42.107.181:47742",
							"Filters": [
								"==",
								"!="
							]
						},
						{
							"Name": "Question",
							"Path": "Question",
							"Value": "{\n    \"Hdr\": {\n      \"Name\": \"porter.datalaiq.io.\",\n      \"Rrtype\": 1,\n      \"Class\": 1,\n      \"Ttl\": 65,\n      \"Rdlength\": 4\n    },\n    \"A\": \"208.71.141.34\",\n\t\"Nonsense\": [ 100, 200, 300 ],\n\t\"MoreNonsense\": [ {\"foo\": \"bar\"} ]\n  }",
							"SubElements": [
								{
									"Name": "Question.A",
									"Path": "Question.A",
									"Value": "208.71.141.34",
									"Filters": [
										"==",
										"!="
									]
								},

							],
							"Filters": [
								"==",
								"!="
							]
						}
					],
					"Module": "json",
					"Tag": "foo"
				}
			]
		}
	]
}
```

ネストの例として、"Question" 要素の `SubElements` フィールドに注目してください。

## クエリソケットコマンド

raw` と `text` のレンダラーには、さらに 2 つのウェブソケットコマンド `REQ_GET_EXPLORE_ENTRIES` と `REQ_EXPLORE_TS_RANGE` が実装されています。これらのコマンドはそれぞれ REQ_GET_ENTRIES と REQ_TS_RANGE コマンドを反映していますが、データエクスプローラのコマンドに対する応答には `Explore` という名前のフィールドが含まれ、そこには ExploreResult オブジェクト (上で定義) の配列がエントリごとに 1 つずつ格納されます。

例えば、次のコマンドは、最初の 10 個のエントリーを要求します:

```
{
	"ID": 61456,
	"EntryRange": {
		"First": 0,
		"Last": 10
	}
}
```

以下は、検索で1件しかヒットしなかった場合の応答例です:

```
{
  "ID": 61456,
  "EntryCount": 1,
  "AdditionalEntries": false,
  "Finished": true,
  "Entries": [
    {
      "TS": "2020-11-02T16:58:56.717034109-07:00",
      "SRC": "",
      "Tag": 0,
      "Data": "ewogICJUUyI6ICIyMDIwLTEwLTE0VDEwOjM1OjQxLjEzNjUyMjg4NC0wNjowMCIsCiAgIlByb3RvIjogInVkcCIsCiAgIkxvY2FsIjogIls6Ol06NTMiLAogICJSZW1vdGUiOiAiNzMuNDIuMTA3LjE4MTo0Nzc0MiIsCiAgIlF1ZXN0aW9uIjogewogICAgIkhkciI6IHsKICAgICAgIk5hbWUiOiAicG9ydGVyLmdyYXZ3ZWxsLmlvLiIsCiAgICAgICJScnR5cGUiOiAxLAogICAgICAiQ2xhc3MiOiAxLAogICAgICAiVHRsIjogNjUsCiAgICAgICJSZGxlbmd0aCI6IDQKICAgIH0sCiAgICAiQSI6ICIyMDguNzEuMTQxLjM0IiwKCSJOb25zZW5zZSI6IFsgMTAwLCAyMDAsIDMwMCBdLAoJIk1vcmVOb25zZW5zZSI6IFsgeyJmb28iOiAiYmFyIn0gXQogIH0KfQ==",
      "Enumerated": null
    }
  ],
  "Explore": [
    {
      "Elements": [
        {
          "Name": "TS",
          "Path": "TS",
          "Value": "2020-10-14T10:35:41.136522884-06:00",
          "Filters": [
            "==",
            "!="
          ]
        },
        {
          "Name": "Proto",
          "Path": "Proto",
          "Value": "udp",
          "Filters": [
            "==",
            "!="
          ]
        },
        {
          "Name": "Local",
          "Path": "Local",
          "Value": "[::]:53",
          "Filters": [
            "==",
            "!="
          ]
        },
        {
          "Name": "Remote",
          "Path": "Remote",
          "Value": "73.42.107.181:47742",
          "Filters": [
            "==",
            "!="
          ]
        },
        {
          "Name": "Question",
          "Path": "Question",
          "Value": "{\n    \"Hdr\": {\n      \"Name\": \"porter.datalaiq.io.\",\n      \"Rrtype\": 1,\n      \"Class\": 1,\n      \"Ttl\": 65,\n      \"Rdlength\": 4\n    },\n    \"A\": \"208.71.141.34\",\n\t\"Nonsense\": [ 100, 200, 300 ],\n\t\"MoreNonsense\": [ {\"foo\": \"bar\"} ]\n  }",
          "SubElements": [
            {
              "Name": "Question.A",
              "Path": "Question.A",
              "Value": "208.71.141.34",
              "Filters": [
                "==",
                "!="
              ]
            }
          ],
          "Filters": [
            "==",
            "!="
          ]
        }
      ],
      "Module": "json",
      "Tag": "foo"
    }
  ]
}

```

## クエリにフィルターを追加する

データエクスプローラーレンダラーコマンドから返される情報は、フィルターリクエストを作成するために使用できます。フィルタ要求は、データ内の値に基づいてクエリの結果を絞り込みます。たとえば、上記の応答例では、「Proto」フィールドが「udp」であるすべてのエントリーを除外したい場合があります:

```
{
  "Tag": "foo",
  "Module": "json",
  "Path": "Proto",
  "Op": "!=",
  "Value": "udp"
}
```

以下は、フィルタを含むStartSearchRequestです:

```
{
  "SearchString": "tag=foo",
  "SearchStart": "2020-01-01T12:01:00.0Z07:00",
  "SearchEnd": "2020-01-01T12:01:30.0Z07:00",
  "Filters": [
    {
      "Tag": "foo",
      "Module": "json",
      "Path": "Proto",
      "Op": "!=",
      "Value": "udp"
    }
  ]
}

```

サーバーの応答には、書き直されたSearchStringフィールドが含まれ、必要に応じて検索ライブラリに保存するのに適しています:

```
{
  "SearchString": "tag=foo json Proto!=udp",
  "RawQuery": "tag=foo",
  "RenderModule": "text",
  "RenderCmd": "text",
  "OutputSearchSubproto": "search484",
  "OutputStatsSubproto": "stats484",
  "SearchID": "7947106109",
  "SearchStartRange": "2015-01-01T12:01:00.0Z07:00",
  "SearchEndRange": "2015-01-01T12:01:30.0Z07:00",
  "Background": false
}
```

サーバーのレスポンスには、書き直された SearchString フィールドが含まれ、必要に応じて検索ライブラリに保存することができます。