# 検索履歴

REST API located at `/api/searchhistory`

検索履歴APIは、あるユーザ、グループ、またはそれらの組み合わせが行った検索の一覧を取得するために使用します。検索結果は、誰が検索を開始したか、どのグループがそれを所有しているか、いつ開始されたか、検索を表す2つの文字列（ユーザーが実際に入力したものとバックエンドが処理したもの）などの基本的な情報を提供します。

## 基本API概要

基本的な動作は `/api/searchhistory/{cmd}/{id}` に対して GET を実行することで、`cmd` は欲しい検索のセット、`id` はその検索に関連する ID を表しています。例えば、UID 1 のユーザーが所有するすべての検索をしたい場合は、 `/api/searchhistory/user/1` に対して GET を実行し、GID 4 のグループが所有するすべての検索をしたい場合は、 `/api/searchhistory/group/4` に対して GET を実行することになるでしょう。単純に `/api/searchhistory` に対して GET を実行すると、現在のユーザーの履歴が返されます。

特定のUIDがアクセスできるすべての検索を要求するには、"all "コマンドを使用します。これは、そのUIDが所有するすべての検索と、そのUIDが所属するグループが所有するすべての検索を私に与えることを意味します。例えば、`/api/searchhistory/all/1`をGETすると、UID 1のユーザーがそれらのグループのメンバーであれば、GID 1, 2, 3, 4のグループが所有する検索を返すかもしれません。返される結果は JSON フォーマットの SearchLog 構造体のリストである。

JSONの例:
```
[
        {
                "UID": 1,
                "GID": 2,
                "UserQuery": "grep stuff",
                "EffectiveQuery": "grep stuff | text",
                "Launched": "2015-12-30T23:30:23.298945825-07:00"
        },
        {
                "UID": 1,
                "GID": 2,
                "UserQuery": "grep stuff | grep things | grep that | grep this | sort by time",
                "EffectiveQuery": "grep stuff | grep things | grep that | grep this | sort by time | text",
                "Launched": "2015-12-30T23:31:08.237520376-07:00"
        }
]
```

### 管理者クエリ

管理者として `/api/searchhistory?admin=true` に GET リクエストすると、すべてのユーザーによるすべての検索結果が返されます。

### 絞り込み検索履歴

`api/searchhistory` の GET メソッドの特別なケースとして、"refine" 値を設定することでシンプルな部分文字列検索を提供することができます。例えば、 `/api/searchhistory?refine=foo` へのGETリクエストは、現在のユーザーに対して、検索語のどこかに "foo" を含むすべての検索を返します。
