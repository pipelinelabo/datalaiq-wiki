# 検索ウェブソケット

Websocket URL: /api/ws/search

このページでは、検索用のウェブソケットプロトコルを説明します。grep foo" 検索を行った際にクライアントとサーバ間で転送される JSON の完全な例と、エントリデータの取得については [ウェブソケット検索例](websocket-search-example.md) ページで見ることができます。

## Ping/Pongキープアライブ

検索ウェブソケットは、クエリのチェック、検索の送信、検索結果や検索統計の受信に使用されます。 検索用ウェブソケットはメッセージの "タイプ "を扱うためにRoutingWebsocketシステムを使用することを想定しています。 `/api/ws/search` は起動時に以下のメッセージの "subtypes" または "types" が登録されることを期待しています。PONG, parse, search, attach.

備考: メッセージの「タイプ」は、レガシーな命名のため、「SubProto」と呼ばれることがあります。これは将来的に変更される可能性がありますが、この API に対して開発を行う場合、"SubProto" はメッセージと共に送信される "type" 値を指し、RFC Websocket subprotocol spec ではないことを認識しておいてください。

PONG型はキープアライブ方式で、クライアントは定期的にPING/PONGリクエストを送信する必要があります。

これは、ユーザーが検索プロンプトなどに座っているときに、バックへの接続が正常であるかどうかをユーザーに伝えるために使用できます。ウェブソケット自体が生きているかどうかを調べることができるため、これはまったく必要ないかもしれません。

## 検索をパースする

"parse" ウェブソケットタイプは、サーチバックエンドを呼び出すことなく、クエリの妥当性を迅速にテストするために使用されます。

有効なクエリーとレスポンスを持つリクエストの例では、以下のようなJSONになります:

フロントエンドからのリクエスト:
```json
{
        SearchString: "tag=apache grep firefox | regex "Firefox(<version>[0-9]+) .+" | count by version""
}
```

バックエンドからのレスポンス:
```
{
        GoodQuery: true,
        ParseQuery: "tag=apache grep firefox | regex "Firefox(<version>[0-9]+) .+" | count by version"",
        ModuleIndex: 0,
}
```

無効なクエリーとレスポンスを持つリクエストの例では、以下のようなJSONになります:

フロントエンドからのリクエスト:
```
{
        SearchString: "tag=apache grep firefox | MakeRainbows",
}
```

バックエンドからのレスポンス:
```
{
        GoodQuery: false,
        ParseError: "ModuleError: MakeRainbows is not a valid module",
        ModuleIndex: 1,
}
```

## 検索を開始する
すべての検索はウェブソケットを通じて開始され、開始時に「parse」、「PONG」、「search」、「attach」サブタイプが要求されることが必要です。 

これは、websocketの確立時に以下のJSONを送信することで行われます:
```
{"Subs":["PONG","parse","search","attach"]}
```


SearchString メンバは、検索を呼び出す実際のクエリを含む必要があります。

SearchStartとSearchEndは、クエリが動作する時間範囲である必要があります。 時間範囲はRFC3339Nano形式で、"2006-01-02T15:04:05.9999999Z07:00 "のような形式でなければなりません。

良いクエリーを持つ検索リクエストの例は、次のようなJSONを含んでいます:
```
{
       SearchString: "tag=apache grep firefox | nosort",
       SearchStart:  "2015-01-01T12:01:00.0Z07:00",
       SearchEnd:    "2015-01-01T12:01:30.0Z07:00",
       Background:   false,
}
```

//サーバーはYay/Nayを応答し、さらに検索がクールであれば新しいサブタイプを応答します。
//searchStart と searchEnd は RFC3339Nano 形式の文字列でなければなりません。

良いクエリに対する応答は、次のようなJSONを含んでいる:
```
{
        SearchString: "tag=apache grep firefox | nosort",
        RenderModule: "text",
        RenderCmd:    "text",
        OutputSearchSubproto:  "searchSDF8973",
        OutputStatsSubproto:   "statsSDF8973",
        SearchID:              "skdlfjs9098",
		SearchStartRange:      "2015-01-01T12:01:00.0Z07:00",
        SearchEndRange:        "2015-01-01T12:01:30.0Z07:00",
        Background:            false,
}
```

エラー時のJSONレスポンスは次のようになります:
```
{
        Error: "Search error: The parameter "ChuckTesta" is invalid",
}
```

良い検索要求の応答には、クライアントは検索ACKで応答しなければならない。Ack は true か false のどちらかで応答しなければなりません。 false レスポンスは、フロントエンドが理解できないレンダーモジュールをバックエンドが要求したときに使われるかもしれません（フロントエンドとバックエンドの間にバージョンの不一致があるときに起こるかもしれません）。

以下のJSONは、先ほどのレスポンス例に対する肯定的なACKを表しています:
```
{
       Ok: True,
       OutputSearchSubproto: "searchSDF8973"
}
```

ACK が送信されると、バックエンドは検索を開始し、新しいサブタイプでの検索結果を提供しはじめます。 元の search、parse、PONG サブタイプはアクティブなままで、フロントエンドが新しいクエリをチェックしたり、追加の検索を開始したりするために使用することができます。 アクティブなクエリとのやりとりはすべて、新しく交渉された検索専用のサブタイプで行われる必要があります。

## 備考
すべての検索は完全に非同期ですが、検索をバックグラウンド状態にすることを要求せずにクライアントが切断したり接続がクラッシュした場合、アクティブな検索は終了し、データはガベージコレクションされます。 これは、リソースの枯渇を防ぐためです。 ユーザーはバックグラウンド検索を明示的に要求する必要があります。

検索は複数の消費者を持つことができます。 例えば、Bobが検索を開始し、Janetがそれに接続して結果を見ることができます。 バックグラウンドでない検索は、すべてのコンシューマが切断された場合のみ終了し、クリーンアップされます。 ですから、ボブが検索を開始し、ジャネットが接続した後、ボブがナビゲートしたり、ブラウザを閉じたりしても、検索は終了しません。 Janetは検索を続けることができます。 しかし、Janetがナビゲートしたり、ブラウザを閉じたりすると、検索は終了し、ガベージコレクションが行われます。

## アクティブに検索中の統計情報を出力する

統計情報は統計IDで要求される

## Request/ResponseIDiの参照

リクエストIDコードとレスポンスIDコードの一覧です:
```
{
    req: {
        REQ_CLOSE: 0x1,
        REQ_ENTRY_COUNT: 0x3,
        REQ_DETAILS: 0x4,
        REQ_TAGS: 0x5,
        REQ_STATS_SIZE: 0x7F000001, //gets backend "size" value of stats chunks. never used
        REQ_STATS_RANGE: 0x7F000002, //gets current time range covered by stats. rarely used
        REQ_STATS_GET: 0x7F000003, //gets stats sets over all time. may be used initially
        REQ_STATS_GET_RANGE: 0x7F000004, //gets stats in a specific range
        REQ_STATS_GET_SUMMARY: 0x7F000005, //gets stats summary for entire results
        REQ_STATS_GET_LOCATION: 0x7F000006, //get current timestamp for search progress
        REQ_GET_ENTRIES: 0x10, //1048578
        REQ_STREAMING: 0x11,
        REQ_TS_RANGE: 0x12,
		REQ_GET_EXPLORE_ENTRIES: 0xf010,
		REQ_EXPLORE_TS_RANGE: 0xf012,
        SEARCH_CTRL_CMD_DELETE: 'delete',
        SEARCH_CTRL_CMD_ARCHIVE: 'archive',
        SEARCH_CTRL_CMD_BACKGROUND: 'background',
        SEARCH_CTRL_CMD_STATUS: 'status'
    },
    rep: {
        RESP_CLOSE: 0x1,
        RESP_ENTRY_COUNT: 0x3,
        RESP_DETAILS: 0x4,
        RESP_TAGS: 0x5,
        RESP_STATS_SIZE: 0x7F000001, //2130706433
        RESP_STATS_RANGE: 0x7F000002, //2130706434
        RESP_STATS_GET: 0x7F000003, //2130706435
        RESP_STATS_GET_RANGE: 0x7F000004, //2130706436
        RESP_STATS_GET_SUMMARY: 0x7F000005,
        RESP_STATS_GET_LOCATION: 0x7F000006, //2130706438
        RESP_GET_ENTRIES: 0x10,
        RESP_STREAMING: 0x11,
        RESP_TS_RANGE: 0x12,
		RESP_GET_EXPLORE_ENTRIES: 0xf010,
		RESP_EXPLORE_TS_RANGE: 0xf012,
        RESP_ERROR: 0xFFFFFFFF,
        SEARCH_CTRL_CMD_DELETE: 'delete',
        SEARCH_CTRL_CMD_ARCHIVE: 'archive',
        SEARCH_CTRL_CMD_BACKGROUND: 'background',
        SEARCH_CTRL_CMD_STATUS: 'status'
    }
}
```
