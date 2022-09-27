# その他のAPI

いくつかのAPIは、主なカテゴリーにうまく当てはまりません。それらはここにリストアップされています。

## 接続テスト

この API は、バックエンドが HTTP リクエストに応答しているかどうかを検証するためのものです。GET on `/api/test` はボディコンテンツなしで 200 ステータスを返す必要があります。認証は必要ありません。

## バージョンAPI

バージョン情報を取得するために `/api/version` に対して GET を実行します。 認証は必要ありません。

```
{
    "API": {
        "Major": 0,
        "Minor": 1
    },
    "Build": {
        "BuildDate": "2020-05-04T00:00:00Z",
        "BuildID": "6c48dd4c",
        "GUIBuildID": "b5c8cd58",
        "Major": 4,
        "Minor": 0,
        "Point": 0
    }
}
```

## タグのリストを取得する

ウェブサーバーは、インデクサに知られているすべてのタグのリストを保持しています。このリストは `/api/tags` に対する GET リクエストで取得できます。これはタグのリストを返します。:

```
["default", "DatalaiQ", "pcap", "windows"]
```

## 検索モジュールをリスト表示する

利用可能なすべての検索モジュールのリストと、それぞれのモジュールに関する情報を取得するには、 `/api/info/searchmodules` を GET してください。これは、モジュール情報の構造体のリストを返します:

```
[
    {
        "Collapsing": true,
        "Examples": [
            "min by src",
            "min by someKey"
        ],
        "FrontendOnly": false,
        "Info": "No information available",
        "Name": "min",
        "Sorting": true
    },
    {
        "Collapsing": true,
        "Examples": [
            "unique",
            "unique chuck",
            "unique chuck,testa"
        ],
        "FrontendOnly": false,
        "Info": "No information available",
        "Name": "unique",
        "Sorting": false
    },
[...]
    {
        "Collapsing": false,
        "Examples": [
            "alias src dst"
        ],
        "FrontendOnly": false,
        "Info": "Alias enumerated values",
        "Name": "alias",
        "Sorting": false
    },
    {
        "Collapsing": true,
        "Examples": [
            "count",
            "count by chuck",
            "count by src",
            "count by someKey"
        ],
        "FrontendOnly": false,
        "Info": "No information available",
        "Name": "count",
        "Sorting": true
    }
]
```

## レンダーモジュールをリスト表示する

利用可能なすべてのレンダーモジュールのリストと各モジュールの情報を取得するには、 `/api/info/rendermodules` で GET を実行します。これは、モジュール情報の構造体のリストを返します:

```
[
    {
        "Description": "A raw entry storage system, it can store and handle anything.",
        "Examples": [
            "raw"
        ],
        "Name": "raw",
        "SortRequired": false
    },
    {
        "Description": "A chart storage system system.\n\t       Chart looks for numeric types, storing them.\n\t       Requested entries will be a set of types with column names.",
        "Examples": [
            "chart"
        ],
        "Name": "chart",
        "SortRequired": false
    },
[...]
    {
        "Description": "A point mapping system that supports condensing and geofencing",
        "Examples": [],
        "Name": "point2point",
        "SortRequired": false
    }
]
```

## GUI設定

このAPIは、ユーザーインターフェースに関する基本的な情報を提供します。api/settings` を GET すると、以下のような構造体が返されます:

```
{
  "DisableMapTileProxy": false,
  "DistributedWebservers": false,
  "MapTileUrl": "http://localhost:8080/api/maps",
  "MaxFileSize": 8388608,
  "MaxResourceSize": 134217728,
  "ServerTime": "2020-11-30T11:50:29.478092519-08:00",
  "ServerTimezone": "PST",
  "ServerTimezoneOffset": -28800
}

```

* `DisableMapTileProxy`, trueの場合、DatalaiQプロキシを使用するのではなく、OpenStreetMapサーバーに直接マップリクエストを送信するようUIに指示します。
* `MapTileUrl` は、マップタイルを取得するためにUIが使用するURLです。
* データストアを介して複数のウェブサーバーが連携している場合、`DistributedWebservers`はtrueに設定されます。
* `MaxFileSize` は `/api/files` API にアップロードできるファイルの最大サイズ (バイト単位) である。
* `MaxResourceSize` は、許容される最大リソースサイズをバイト数で指定します。
* `ServerTime` はWebサーバーの現在の時間です。
* `ServerTimezone` はWebサーバーのタイムゾーンです。
* `ServerTimezoneOffset` は、ウェブサーバーのタイムゾーンのオフセットで、UTCからの秒数で指定します。

## スクリプトライブラリ

このAPIを使うと、自動化スクリプトが `require` 関数を使って GitHub リポジトリからライブラリをインポートすることができるようになります。また、ユーザーの全リポジトリに対して git pull を実行するエンドポイントも用意されています。

### ライブラリを取得する

このエンドポイントは、おそらくサーチエージェントがライブラリ関数経由で使用する場合にのみ有用ですが、念のため含まれています。指定されたリポジトリからファイルを取得するには、URL にパラメータを指定して GET してください:

```
/api/libs?repo=github.com/gravwell/libs&commit=40e98d216bb6e69642df392b255e8edc0f57eb06&path=utils/links.ank
```

"repo "と "commit "の値は省略可能です。repo "を省略した場合、デフォルトはgithub.com/gravwell/libsになります。commit "を省略した場合、デフォルトはmasterブランチの先端となります。

### ライブラリを更新する

各ユーザーのリポジトリセットが管理されます。ユーザーは `/api/libs/pull` に GET リクエストを送ることで、自分自身のリポジトリセットに対して `git pull` を強制することができます。これには時間がかかる可能性があることに注意してください。
