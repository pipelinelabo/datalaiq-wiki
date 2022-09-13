# ダッシュボードAPI

ダッシュボードAPIは、基本的にダッシュボードをレンダリングするためにGUIによって使用されるjson blobを管理するための汎用CRUD apiです。ダッシュボード上に存在する検索はGUIによって起動され、バックエンド/フロントエンド/ウェブサーバはそれらが何であるかという概念を持っていません。

## データ構造

* ID: ダッシュボードで特定される64ビットの整数値
* GUID: ダッシュボードを指定する UUID です。キットのインストールをまたいで持続します（下記参照）。アクショナブルからダッシュボードを参照するときに使用します。
* Name: だっs部ボード名
* Description: ダッシュボードの詳細な説明
* UID: ダッシュボードの所有者のUID
* GIDs: ダッシュボードが共有されているグループのGID
* Global: 真偽値。trueにすると全てのユーザーがダッシュボードを閲覧可能になる (管理者のみ)
* Created: ダッシュボードが作成されたタイムスタンプ
* Updated: ダッシュボードが最後に更新されたタイムスタンプ
* Labels: ダッシュボードに含まれている[ラベル](#!gui/labels/labels.md)のリスト
* Data: ダッシュボードに含まれる実際のコンテンツ (以下を参照)

すべてのダッシュボードには `ID` フィールドと `GUID` フィールドの両方があることに注意してください。これは、ダッシュボードが、それらのダッシュボードを*参照するアクショナブルとともにキットに含まれる可能性があるためです。キットに含まれるダッシュボードは既存のGUIDを含み、そのGUIDはキットがインストールされたときにも保持されるため、アクショナブルがダッシュボードをそのGUIDで参照することは安全です。一方、IDフィールドは、ダッシュボードが作成またはインストールされるたびにランダムに生成されます。あるシステムには、実際には同じGUIDを持つ複数のダッシュボードがあるかもしれませんが（通常は異なるユーザーによってインストールされます）、各ダッシュボードは独自のIDを持ちます。

ウェブサーバは `Data` フィールドに何が入るかを気にしませんが(有効なJSONでなければならないことを除いて)、**GUI** が使用する特定のフォーマットが存在します。以下は、Dashboard の構造を完全に定義した Typescript で、GUI が使用する Data フィールドも含まれています:

```
interface RawDashboard {
    ID: RawNumericID;
    GUID: RawUUID;

    UID: RawNumericID;
    GIDs: Array<RawNumericID> | null;

    Name: string;
    Description: string; // empty string is null
    Labels: Array<string> | null;

    Created: string; // Timestamp
    Updated: string; // Timestamp

    Data: {
        liveUpdateInterval?: number; // 0 is undefined
        linkZooming?: boolean;

        grid?: {
            gutter?: string | number | null; // string is a number
            margin?: string | number | null; // string is a number
        };

        searches: Array<{
            alias: string | null;
            timeframe?: {} | RawTimeframe;
            query?: string;
            searchID?: RawNumericID;
            reference?: {
                id: RawUUID;
                type: 'template' | 'savedQuery' | 'scheduledSearch';
                extras?: {
                    defaultValue: string | null;
                };
            };
        }>;
        tiles: Array<{
            id: RawNumericID;
            title: string;
            renderer: string;
            span: { col: number; row: number; x: number; y: number };
            searchesIndex: number;
            rendererOptions: RendererOptions;
        }>;
        timeframe: RawTimeframe;
        version?: 1 | 2;
        lastDataUpdate?: string; // Timestamp
    };
}

interface RawTimeframe {
    durationString: string | null;
    timeframe: string;
    start: string | null; // Timestamp
    end: string | null; // Timestamp
}

interface RendererOptions {
    XAxisSplitLine?: 'no';
    YAxisSplitLine?: 'no';
    IncludeOther?: 'yes';
    Stack?: 'grouped' | 'stacked';
    Smoothing?: 'normal' | 'smooth';
    Orientation?: 'v' | 'h';
    ConnectNulls?: 'no' | 'yes';
    Precision?: 'no';
    LogScale?: 'no';
    Range?: 'no';
    Rotate?: 'yes';
    Labels?: 'no';
    Background?: 'no';
    values?: {
        Smoothing?: 'smooth';
        Orientation?: 'h';
        columns?: Array<string>;
    };
}
```

備考: このドキュメントでは、有効な `Data` 構造体を含みますが、簡潔さのために、タイルを含むものよりも空のダッシュボードを記述する構造体を使用する傾向があります。

## ダッシュボード作成

ダッシュボードを追加するには `/api/dashboards` に以下のフォーマットに沿ったデータと共にPOSTリクエストを送ります: 

```
{
        "Name": "test2",
        "Description": "test2 description",
		"UID": 2,
		"GIDs": [],
		"Global": false,
        "Data": {
			"tiles": []
        }
}
```

Data`プロパティは、GUIが実際のダッシュボードを作成するために使用するJSONです。ここでは、デモのために空の "tiles "フィールドを含めています。

もし `UID` パラメータがリクエストから省略された場合、デフォルトではリクエストしたユーザーのUIDが設定されます。管理者のみ、UIDを自分以外の任意のIDに設定することができます。

GIDs` 配列には、ダッシュボードを共有するグループ ID のリストを指定します。空のままだと、ダッシュボードは誰とも共有されません。

管理者ユーザーは、`Global`フィールドをtrueに設定することもできます。この設定により、システム上のすべてのユーザーがダッシュボードにアクセスできるようになります。

もし `GUID` フィールドが含まれていて、それが有効なUUIDである場合、ダッシュボードではランダムに生成されるのではなく、そのフィールドが使用されます。ほとんどの場合、GUIDフィールドは空白にしておくと、ウェブサーバーがランダムに生成できるようになります。

ウェブサーバーからの応答には、新しく作成されたダッシュボードの数値IDが含まれています。

## ダッシュボードリスト表示

### 現在のユーザーでダッシュボードのリストを表示する

ユーザーは自分がアクセスすることのできるダッシュボードのリストを `/api/dashboards` にGETリクエストを送信することで表示することができます。以下のレスポンスには2つのダッシュボードが含まれています:

```
[
  {
    "ID": 203486809563715,
    "Name": "test",
    "UID": 1,
    "GIDs": [],
    "Description": "test dashboard",
    "Created": "2020-09-22T09:16:51.66798721-06:00",
    "Updated": "2020-09-22T09:17:06.241311128-06:00",
    "Data": {
      "searches": [
        {
          "alias": "Search 1",
          "timeframe": {},
          "query": "tag=* count\n",
          "searchID": 4780372388
        }
      ],
      "tiles": [
        {
          "title": "count",
          "renderer": "text",
          "span": {
            "col": 4,
            "row": 4,
            "x": 0,
            "y": 0
          },
          "searchesIndex": 0,
          "id": 16007878262310,
          "rendererOptions": {}
        }
      ],
      "timeframe": {
        "durationString": "PT1H",
        "timeframe": "PT1H",
        "end": null,
        "start": null
      },
      "version": 2,
      "lastDataUpdate": "2020-09-22T09:17:06-06:00"
    },
    "Labels": null,
    "GUID": "9719d92a-df1f-4a05-885a-ad10915d8b42",
    "Synced": false
  },
  {
    "ID": 69148521436807,
    "Name": "Test 2",
    "UID": 1,
    "GIDs": [],
    "Description": "dashboard 2",
    "Created": "2020-09-22T09:17:13.809070187-06:00",
    "Updated": "2020-09-22T09:17:13.809070187-06:00",
    "Data": {
      "searches": [],
      "tiles": [],
      "timeframe": {
        "durationString": "PT1H",
        "timeframe": "PT1H",
        "end": null,
        "start": null
      }
    },
    "Labels": null,
    "GUID": "2c55cf84-bb63-40cf-bf54-3bff8c8d7fb6",
    "Synced": false
  }
]
```

### 特定のユーザーに所有されている全てのダッシュボードのリストを表示する
あるユーザーが明示的に所有しているダッシュボードを取得するには、`/api/users/{uid}/dashboards`に対して、{uid}を希望するユーザーIDに置き換えてGETリクエストを発行してください。ウェブサーバーはそのUIDが特に所有しているダッシュボードのみを返します。 これには、そのユーザーがグループ・メンバーシップを通じてアクセスできるダッシュボードは含まれません。


```
WEB GET /api/users/1/dashboards:
[
  {
    "ID": 4,
    "Name": "dashGroup2",
    "UID": 5,
    "GIDs": [
      3
    ],
    "Description": "dashGroup2",
    "Created": "2016-12-28T21:37:12.703358455Z",
    "GUID": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
    "Data": {
      "searches": [],
      "tiles": [],
      "timeframe": {
        "durationString": "PT1H",
        "timeframe": "PT1H",
        "end": null,
        "start": null
      }
    }
  }
]

```

### グループのダッシュボードをリスト表示する
特定のグループがアクセスできるすべてのダッシュボードを取得するには、`/api/groups/{gid}/dashboards`に対してGETリクエストを発行してください（{gid}は希望のグループIDに置き換えてください）。サーバーはそのグループと共有されているダッシュボードを返します。 これは、ダッシュボードを複数のグループで共有できるという点で、通常のUnixパーミッションから少し逸脱しています（したがって、グループは実際にダッシュボードを所有しているわけではなく、ダッシュボードはグループのメンバーのようなものです）。

```
WEB GET /api/groups/2/dashboards:
[
  {
    "ID": 3,
    "Name": "dashGroup1",
    "UID": 5,
    "GIDs": [
      2
    ],
    "Description": "dashGroup1",
    "Created": "2016-12-28T21:37:12.696460531Z",
    "GUID": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
    "Data": {
      "searches": [],
      "tiles": [],
      "timeframe": {
        "durationString": "PT1H",
        "timeframe": "PT1H",
        "end": null,
        "start": null
      }
    }
  },
  {
    "ID": 2,
    "Name": "test2",
    "UID": 3,
    "GIDs": [
      2
    ],
    "Description": "test2 description",
    "Created": "2016-12-18T23:28:08.250051418Z",
    "GUID": "d28b6887-ad55-479e-8af3-0cbcbd5084b1",
    "Data": {
      "searches": [],
      "tiles": [],
      "timeframe": {
        "durationString": "PT1H",
        "timeframe": "PT1H",
        "end": null,
        "start": null
      }
    }
  }
]

```


### 全てのユーザーのダッシュボードをリスト表示する（管理者のみ）
システム上の *すべての* ダッシュボードを取得するために、admin ユーザーは `/api/dashboards/all` に対して GET リクエストを発行することができます。このリクエストが管理者以外のユーザーによって発行された場合、そのユーザーがアクセスできるすべてのダッシュボードを返す必要があります (`/api/dashboards` に対する GET と同じです)。

```
WEB GET /api/dashboards/all:
[
  {
    "ID": 1,
    "Name": "test1",
    "UID": 1,
    "GIDs": [],
    "Description": "test1 description",
    "Created": "2016-12-18T23:28:07.679322121Z",
    "GUID": "d28b6887-ad55-479e-8af3-0cbcbd5084b1",
    "Data": {
      "searches": [],
      "tiles": [],
      "timeframe": {
        "durationString": "PT1H",
        "timeframe": "PT1H",
        "end": null,
        "start": null
      }
    }
  },
  {
    "ID": 2,
    "Name": "test2",
    "UID": 3,
    "GIDs": [],
    "Description": "test2 description",
    "Created": "2016-12-18T23:28:08.250051418Z",
    "GUID": "55bc7236-39e4-11e9-94e9-54e1ad7c66cf",
    "Data": {
      "searches": [],
      "tiles": [],
      "timeframe": {
        "durationString": "PT1H",
        "timeframe": "PT1H",
        "end": null,
        "start": null
      }
    }
  }
]

```

## 特定のダッシュボードの情報を取得する
特定のIDを取得するには、`/api/dashboards/{id}` にGETリクエストを発行します。`{id}` はダッシュボードのIDに置き換えてください。

また、IDではなくGUIDで特定のダッシュボードを取得することも可能です：`GET /api/dashboards/d28b6887-ad55-479e-8af3-0cbcbd5084b1`.


## ダッシュボードを更新する
ダッシュボードの更新（データの変更、名前、説明、GIDリストなどの変更）は `/api/dashboards/{id}` にPUTリクエストを発行することで行われます。

この例では、ユーザー (UID 3) がグループ 1 にダッシュボード (ID 2) へのアクセス許可を追加することを希望しています。ユーザーはまず、ダッシュボードを取得します:

```
GET /api/dashboards/2:
{
  "ID": 2,
  "Name": "test2",
  "UID": 3,
  "GIDs": [],
  "Description": "test2 description",
  "GUID": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
  "Created": "2016-12-18T23:28:08.250051418Z",
  "Data": {
    "searches": [
      {
        "alias": "Search 1",
        "timeframe": {},
        "query": "tag=* count\n",
        "searchID": 4780372388
      }
    ],
    "tiles": [
      {
        "title": "count",
        "renderer": "text",
        "span": {
          "col": 4,
          "row": 4,
          "x": 0,
          "y": 0
        },
        "searchesIndex": 0,
        "id": 16007878262310,
        "rendererOptions": {}
      }
    ],
    "timeframe": {
      "durationString": "PT1H",
      "timeframe": "PT1H",
      "end": null,
      "start": null
    },
    "version": 2,
    "lastDataUpdate": "2020-09-22T09:17:06-06:00"
  }
}
```

これで、ユーザーは対象のダッシュボードを取得し、変更を加えて、PUTリクエストを投稿しました:
```
WEB PUT /api/dashboards/2:
{
  "ID": 2,
  "Name": "marketoverview",
  "UID": 3,
  "GIDs": [
    3
  ],
  "Description": "marketing group dashboard",
  "GUID": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
  "Created": "2016-12-18T23:28:08.250051418Z",
  "Data": {
    "searches": [
      {
        "alias": "Search 1",
        "timeframe": {},
        "query": "tag=* count\n",
        "searchID": 4780372388
      }
    ],
    "tiles": [
      {
        "title": "count",
        "renderer": "text",
        "span": {
          "col": 4,
          "row": 4,
          "x": 0,
          "y": 0
        },
        "searchesIndex": 0,
        "id": 16007878262310,
        "rendererOptions": {}
      }
    ],
    "timeframe": {
      "durationString": "PT1H",
      "timeframe": "PT1H",
      "end": null,
      "start": null
    },
    "version": 2,
    "lastDataUpdate": "2020-09-22T09:17:06-06:00"
  }
}

```

サーバーは更新要求に対して、更新されたダッシュボード構造で応答します。

安全のために、元の取得時に存在したすべてのフィールドを、たとえ変更されていなくても送り返すように注意してください。例えば、この更新では `Description` フィールドは変更されていませんが、Web サーバーは *set* されていない* フィールドと *empty* フィールドを区別できないため、更新リクエストに含めています。

備考：必要に応じて、ダッシュボードIDの代わりにGUIDを使用することができます。

## ダッシュボードを削除する
ダッシュボードを削除するには、DELETEメソッドで`/api/dashboards/{id}`というURLにリクエストしてください（{id}はダッシュボードの数字ID）。
