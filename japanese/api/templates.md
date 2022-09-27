# テンプレートAPI

テンプレートは、*変数*を含むDatalaiQクエリーを定義する特別なオブジェクトです。例えば、IPアドレスを変数とするテンプレートは、IPアドレス調査ダッシュボードを作成するために使用することができます。

## データ構造

テンプレート構造には、以下のフィールドが含まれます:

* GUID: テンプレートのグローバルリファレンス。キットのインストールをまたいで永続化します。(次のセクションを参照)
* ThingUUID: この特定のテンプレートインスタンスに固有のID。(次のセクションを参照)
* UID: テンプレート所有者のUID
* GIDs: テンプレートを共有されているグループのGIDリスト
* Global: 真偽値で、テンプレートがすべてのユーザに見えるようにする場合はtrueに設定します（管理者のみ）。
* Name: テンプレート名。
* Description: テンプレートの詳細。
* Updated: テンプレートが最後に更新されたタイムスタンプ
* Labels: [ラベル](#!gui/labels/labels.md)を含む配列リスト。
* Contents: テンプレート自体の実際の定義（下記参照）。
  * Contents.query: テンプレートのクエリ
  * Contents.variable: クエリ中に含まれる変数名
  * Contents.variableLabel: ユーザーに表示されるラベル
  * Contents.variableDescription: テンプレートを識別するヒント/追加の説明
  * Contents.required: テンプレート実行時にユーザーから値を要求された場合は真。
  * Contents.testValue: プレビューとバリデーションのためのデフォルト値。

ウェブサーバは `Contents` フィールドに何が入るかを気にしませんが (有効な JSON でなければならないことを除いて)、 **GUI** が使用する特定のフォーマットがあります。GUIで使用するためには、Contentsフィールドはこの構造に従わなければなりません:

```
Contents: {
  query: string;
  variable: string;
  variableLabel: string;
  variableDescription: string | null;
  required: boolean;
  testValue: string | null;
};
```

以下は、Typescript によるテンプレートデータ型の完全な定義です:

```
interface RawTemplate {
    GUID: RawUUID;
    ThingUUID: RawUUID;
    UID: RawNumericID;
    GIDs: null | Array<RawNumericID>;
    Global: boolean;
    Name: string;
    Description: string; // Empty string is null
    Updated: string; // Timestamp
    Contents: {
        query: string;
        variable: string;
        variableLabel: string;
        variableDescription: string | null;
        required: boolean;
        testValue: string | null;
    };
    Labels: null | Array<string>;
}
```

## GUIDsとThingUUIDs

テンプレートには、GUIDとThingUUIDという2つの異なるIDが付与されています。これらは両方ともUUIDで、混乱することがあります。なぜ1つのオブジェクトに2つの識別子があるのでしょうか？このセクションでは、その理由を明らかにします。

DatalaiQでは、ダッシュボードは特定のテンプレートを指す場合があります。このダッシュボードと対応するテンプレートは、他のユーザーに配布するためにキットにパックされることもあります。そのため、テンプレートの「グローバル」な名前としてGUIDを導入しました。しかし、複数のユーザーが同じキットをインストールすることが許可されているので、テンプレートの個々のインスタンス*のための異なる識別子も必要です。この役割を果たすのがThingUUIDフィールドです。

例を考えてみましょう。私はダッシュボードとテンプレートを含むキットを作成します。テンプレートはゼロから作成するので、ランダムなGUIDである `e80293f0-5732-4c7e-a3d1-2fb779b91bf7` とランダムなThingUUIDである `c3b24e1e-5186-4828-82ee-82724a1d4c45` が割り当てられます。そして、ダッシュボードにGUID (`e80293f0-5732-4c7e-a3d1-2fb779b91bf7`) でテンプレートを参照するタイルを作り、テンプレートとダッシュボード両方をキットにバンドルします。同じシステムの別のユーザーがこのキットを自分用にインストールすると、**同じ** GUID (`e80293f0-5732-4c7e-a3d1-2fb779b91bf7`) で **random** ThingUUID (`f07373a8-ea85-415f-8dfd-61f7b9204ae0`) のテンプレートにインスタンス化されるようになりました。ユーザーがダッシュボードを開くと、ダッシュボードは GUID == `e80293f0-5732-4c7e-a3d1-2fb779b91bf7` を持つテンプレートを要求します。ウェブサーバーは、ThingUUID == `f07373a8-ea85-415f-8dfd-61f7b9204ae0` のテンプレートのそのユーザーのインスタンスを返します。

ユーザーがグローバルテンプレートと同じGUIDを持つテンプレートをインストールした場合、そのテンプレートは透過的にグローバルテンプレートを上書きしますが、自分自身のためだけであることに注意してください。同じGUIDを持つテンプレートが複数存在する場合、以下の順序で優先されます:

* ユーザーに所有されているテンプレート
* ユーザーが所属しているグループに共有されているテンプレート
* グローバルのテンプレート

これは、ユーザーがグローバルダッシュボードにアクセスしている場合、同じGUIDを持つテンプレートの独自のコピーを作成することによって、ダッシュボードによって参照される特定のテンプレートをオーバーライドできることを意味します。実際には、このようなことはほとんどないはずです。

### GUIDとThingUUID経由でのテンプレートアクセス

一般ユーザは常にGUIDでテンプレートにアクセスしなければなりません。管理者ユーザーは代わりにThingUUIDでテンプレートを参照することができますが、`?admin=true`パラメータをリクエストURLに設定する必要があります。

## テンプレートを作成する

テンプレートを作成するには、`/api/templates`にPOSTを送信してください。ボディはJSON構造で、'Contents'フィールドに任意の有効なJSON、オプションでGUID、Labels、Name、Descriptionを指定します。以下はすべて有効です:

```
{
  "Contents": {
    "query": "tag=json json ip==%%IP%% | table",
    "required": true,
    "testValue": "\"10.0.0.1\"",
    "variable": "%%IP%%",
    "variableDescription": "the IP to investigate",
    "variableLabel": "IP address"
  }
}
```

```
{
  "GUID": "ce95b152-d47f-443f-884b-e0b506a215be",
  "Contents": {
    "query": "tag=json json ip==%%IP%% | table",
    "required": true,
    "testValue": "\"10.0.0.1\"",
    "variable": "%%IP%%",
    "variableDescription": "the IP to investigate",
    "variableLabel": "IP address"
  }
}
```

```
{
  "Contents": {
    "query": "tag=json json ip==%%IP%% | table",
    "required": true,
    "testValue": "\"10.0.0.1\"",
    "variable": "%%IP%%",
    "variableDescription": "the IP to investigate",
    "variableLabel": "IP address"
  },
  "Name": "mytemplate"
}
```

```
{
  "GUID": "ce95b152-d47f-443f-884b-e0b506a215be",
  "Contents": {
    "query": "tag=json json ip==%%IP%% | table",
    "required": true,
    "testValue": "\"10.0.0.1\"",
    "variable": "%%IP%%",
    "variableDescription": "the IP to investigate",
    "variableLabel": "IP address"
  },
  "Name": "mytemplate"
}
```

```
{
  "Contents": {
    "query": "tag=json json ip==%%IP%% | table",
    "required": true,
    "testValue": "\"10.0.0.1\"",
    "variable": "%%IP%%",
    "variableDescription": "the IP to investigate",
    "variableLabel": "IP address"
  },
  "Name": "mytemplate",
  "Labels": [
    "suits",
    "ladders"
  ]
}
```

API は新しく作成されたテンプレートの GUID で応答します。リクエストでGUIDが指定された場合は、そのGUIDが使用されます。GUIDが指定されない場合は、ランダムなGUIDが生成されます。

備考: 現時点では、`UID`、`GIDs`、`Global` フィールドをテンプレート作成時に設定することはできません。代わりに、updateコールで設定する必要があります(下記参照)。

## テンプレートをリスト表示する

ユーザーが利用できるすべてのテンプレートをリストアップするには、`/api/templates`をGETしてください。結果はテンプレートの配列になります:

```
[
  {
    "ThingUUID": "1b36a1d7-a5ac-11ea-b07e-7085c2d881ce",
    "UID": 1,
    "GIDs": [
      6,
      8
    ],
    "Global": false,
    "GUID": "780b1d31-e46b-4460-ad83-2fc11c34a162",
    "Name": "json ip",
    "Description": "JSON tag, filter by IP",
    "Contents": {
      "variable": "%%IP%%",
      "query": "tag=json* json ip==%%IP%% | table",
      "variableLabel": "IP address",
      "variableDescription": "the IP to investigate!",
      "required": true,
      "testValue": "\"10.0.0.1\""
    },
    "Updated": "2020-09-01T15:01:18.354750806-06:00",
    "Labels": [
      "test"
    ]
  }
]

```

## 特定のテンプレートを取得する

一つのテンプレートを取得するには、 `/api/templates/<guid>` に GET リクエストを発行してください。例えば、 `/api/templates/780b1d31-e46b-4460-ad83-2fc11c34a162` に GET すると、次のような結果が返されます:

```
{
  "ThingUUID": "1b36a1d7-a5ac-11ea-b07e-7085c2d881ce",
  "UID": 1,
  "GIDs": [
    6,
    8
  ],
  "Global": false,
  "GUID": "780b1d31-e46b-4460-ad83-2fc11c34a162",
  "Name": "json ip",
  "Description": "JSON tag, filter by IP",
  "Contents": {
    "variable": "%%IP%%",
    "query": "tag=json* json ip==%%IP%% | table",
    "variableLabel": "IP address",
    "variableDescription": "the IP to investigate!",
    "required": true,
    "testValue": "\"10.0.0.1\""
  },
  "Updated": "2020-09-01T15:01:18.354750806-06:00",
  "Labels": [
    "test"
  ]
}
```

管理者は、ThingUUIDとadminパラメータを使用して、明示的にこの特定のテンプレートを取得できることに注意してください、例えば `/api/templates/1b36a1d7-a5ac-11ea-b07e-7085c2d881ce?admin=true` です。

## テンプレートを更新する

テンプレートを更新するには、 `/api/templates/<guid>` に PUT リクエストを発行してください。リクエストのボディは、同じパスのGETによって返されるものと同じでなければならず、必要な要素は変更されます。GUIDとThingUUIDは変更できないことに注意してください; 以下のフィールドのみが変更可能です:

* Contents: テンプレートの実際のコンテンツ
* Name: テンプレート名を変更する
* Description: テンプレートの詳細を変更する
* GIDs: 32ビット整数のグループIDの配列（例：`"GIDs":[1,4]`）を設定することができる
* UID: (管理者のみ) 32ビット整数に設定
* Global: (管理者のみ) 真偽を設定します。グローバルテンプレートは、すべてのユーザーに表示されます

備考: これらのフィールドのいずれかを空白にすると、そのフィールドのNULL値でテンプレートが更新されます!

## テンプレートを削除する

テンプレートを削除するには、`/api/templates/<guid>` に DELETE リクエストを発行してください。

## 管理者操作

管理者ユーザーは、システム上のすべてのテンプレートを表示したり、変更したり、削除したりする必要がある場合があります。GUIDは必ずしも一意ではないため、管理者APIは代わりにDatalaiQがアイテムの保存に内部的に使用する一意のUUIDを参照する必要があります。上記のテンプレート・リストの例では、"ThingUUID "という名前のフィールドが含まれていることに注意してください。これは、そのテンプレートに対する内部的な一意の識別子です。

管理者ユーザーは `/api/templates?admin=true` の GET リクエストでシステム内のすべてのテンプレートのグローバルリストを取得することができます。

管理者は、`/api/templates/<ThingUUID>?admin=true`へのPUTで、希望するテンプレートのThingUUID値を代入して、特定のテンプレートを更新することができます。同じパターンが削除にも適用されます。

管理者は `/api/templates/<ThingUUID>?admin=true` の GET と DELETE リクエストで、それぞれ特定のテンプレートにアクセスしたり削除したりすることができます。