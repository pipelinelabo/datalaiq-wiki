## 既存の検索に再アタッチする
既に存在する検索へのアタッチは、接続時に「search」、「parse」、「PONG」とともにネゴシエートされなければならない「attach」という名前の別のサブプロトコルを使用して、既存のウェブソケットを通じて実行されます。

#### 権限
検索に参加するためには、ユーザーは検索に割り当てられたグループのメンバー、オーナー、または管理者である必要があります。 ユーザーが検索に添付することを許可されていない場合、permission deniedメッセージがソケットを介して返送されます。

### 検索にアタッチすることをリクエストする
基本的にはIDを渡すだけで、サーバーはErrorか、新しいSubProto、RenderMod、RenderCmd、SearchInfoで応答するようになります。

new Subproto は、新しくアタッチされた検索にサービスを提供するために作成される新しいサブプロトコルの名前です。 つまり、ウェブソケットを立ち上げて、同時にたくさんの検索にアタッチすることができるのです。

### 例
以下は、クライアントが検索リストを要求し、それにアタッチする例です。

検索のリストを要求する
```
WEB GET /api/searchctrl:
[
        {
                "ID": "004081950",
                "UID": 1,
                "GID": 0,
                "State": "DORMANT",
                "AttachedClients": 0
        },
        {
                "ID": "560752652",
                "UID": 1,
                "GID": 0,
                "State": "DORMANT",
                "AttachedClients": 0
        },
        {
                "ID": "608274427",
                "UID": 1,
                "GID": 0,
                "State": "DORMANT",
                "AttachedClients": 0
        }
]
```

"attach "サブプロトコルに関するトランザクション
```
SUBPROTO PUT attach:
{
        "ID": "560752652"
}
SUBPROTO GET attach:
{
        "Subproto": "attach5",
        "RendererMod": "text",
        "Info": {
                "ID": "560752652",
                "UID": 1,
                "UserQuery": "grep paco",
                "EffectiveQuery": "grep paco | text",
                "StartRange": "2017-01-14T06:08:32.024425042-07:00",
                "EndRange": "2017-01-14T16:08:32.024425042-07:00",
                "Started": "2017-01-14T16:08:32.025987218-07:00",
                "Finished": "2017-01-14T16:08:32.746482323-07:00",
                "StoreSize": 0,
                "IndexSize": 0
        }
}
```
新しいアタッチドサーチをネゴシエートした後は、通常レンダラーに取り込むときと同じように、古いAPIを使い続けてください。
```
SUBPROTO PUT attach5:
{
        "ID": 3
}
SUBPROTO GET attach5:
{
        "ID": 3,
        "EntryCount": 20000,
        "Finished": true
}
SUBPROTO PUT attach5:
{
        "ID": 16777218,
        "EntryRange": {
                "First": 0,
                "Last": 1024,
                "StartTS": "0001-01-01T00:00:00Z",
                "EndTS": "0001-01-01T00:00:00Z"
        }
}
```