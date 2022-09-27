# スケジュール検索API

本APIは、スケジュール検索の作成と管理を可能にします。検索はランダムに生成されるIDで参照されます。

## スケジュール検索構造

スケジュール検索には、以下の項目が含まれます:

* ID: スケジュール検索のID
* GUID: この特定の検索のための一意のID。作成時に空白のままだと、ランダムなGUIDが割り当てられる（これが標準的な使用例となるはずである）。
* Owner: 検索の所有者ID
* Groups: 検索結果の閲覧を許可されているグループのGID
* Global: 検索結果をすべてのユーザに見せることを示す boolean (このフィールドを設定できるのは管理者のみです)
* Name: スケジュール検索の名前
* Description: スケジュール検索の詳細
* Schedule: 実行するタイミングを指定する cronベースの文字列
* Permissions: パーミッションを格納するために使用される64ビット整数
* Disabled: trueに設定すると、スケジュール検索の実行を無効にするブール値
* OneShot: trueに設定すると、無効にしない限り、スケジュールされた検索をできるだけ早く1回実行するようになります
* LastRun: スケジュール検索が最後に行われた日付
* LastRunDuration: 最後に実行した際にかかった時間
* LastSearchIDs: このスケジュール検索で最も最近実行された検索の検索IDを含む文字列の配列
* LastError: この検索の最後の実行に起因するすべてのエラー

検索が「標準的な」スケジュール検索である場合、以下のフィールドも設定されます:

* SearchString: 実行されるクエリ文字列
* Duration: どの程度遡って検索を行うかを指定する，秒単位の値。これは負の値でなければならない。
* SearchSinceLastRun: 設定された場合、Durationフィールドは無視され、代わりにLastRun時間から現在までの検索が実行されます。

一方、検索対象がスクリプトの場合、以下のフィールドが設定されます:

* Script: ankoスクリプトを含むスクリプト名

## ユーザーコマンド

このセクションのAPIコマンドは、どのユーザーでも実行することができます。

### スケジュール検索をリスト表示する

ユーザーが見ることのできるすべてのスケジュール検索のリストを取得するには、`/api/scheduledsearches`に対してGETを実行してください（ユーザーが所有しているか、ユーザーのグループのいずれかがアクセス可能であるとマークされています）。結果はこのようになります:

```
[
  {
    "ID": 1439174790,
    "GUID": "efd1813d-283f-447a-a056-729768326e7b",
    "Groups": null,
	"Global": false,
    "Name": "count",
    "Description": "count all entries",
    "Owner": 1,
    "Schedule": "* * * * *",
    "Permissions": 0,
    "Updated": "2019-05-21T16:01:01.036703243-06:00",
    "Disabled": false,
    "OneShot": false,
    "Synced": true,
    "SearchString": "tag=* count",
    "Duration": -3600,
    "SearchSinceLastRun": false,
    "Script": "",
    "PersistentMaps": {},
    "LastRun": "2019-05-21T16:01:00.013062447-06:00",
    "LastRunDuration": 1015958622,
    "LastSearchIDs": [
      "672586805"
    ],
    "LastError": ""
  }
]

```

この例では、UID 1 (admin)が所有する "count"という名前の一つのスケジュール検索を示します。これは1分ごとに実行され、過去1時間に渡って `tag=* count` という検索を実行します。

### スケジュール検索を作成する

新しいスケジュール検索を作成するには、スケジュール検索に関する情報を含むJSON構造体で `/api/scheduledsearches` に対してPOSTリクエストを実行します。標準的な検索を作成するには、毎日午前8時に過去24時間の検索を実行するこの例のように、SearchStringとDurationフィールドに必ず値を入力してください:

```
{
  "Name": "myscheduledsearch",
  "Description": "a scheduled search",
  "Groups": [
    2
  ],
  "Global": false,
  "Schedule": "0 8 * * *",
  "SearchString": "tag=default grep foo",
  "Duration": -86400,
  "SearchSinceLastRun": false
}

```

また、SearchSinceLastRunフィールドがtrueに設定されている場合、検索エージェントはDurationを無視し（この新しい検索の最初の実行を除く）、代わりに最後の実行の時間から現在の時間まで検索を実行します。

スクリプトを使用してスケジュール検索を作成するには、「SearchString」と「Duration」フィールドではなく、「Script」フィールドに入力します。両方が入力されている場合は、スクリプトが優先されます。

スケジュール検索は、Disabledフラグをtrueに設定して作成すると、ユーザーの準備が整うまで実行されないようにすることができます。また、OneShotフラグをtrueに設定して作成すると、作成後すぐに検索が実行されるようになります。

サーバーは、新しいスケジュール検索のIDを応答します。

### 特定のスケジュール検索を取得する

一つのスケジュール検索に関する情報は `/api/scheduledsearches/{id}` のGETでアクセスすることができます。例えば、スケジュール検索のIDが1439174790の場合、`/api/scheduledsearches/1439174790`に問い合わせると、次のようなものが得られます:

```
{
  "ID": 1439174790,
  "GUID": "efd1813d-283f-447a-a056-729768326e7b",
  "Groups": null,
  "Global": false,
  "Name": "count",
  "Description": "count all entries",
  "Owner": 1,
  "Schedule": "* * * * *",
  "Permissions": 0,
  "Updated": "2019-05-21T16:01:01.036703243-06:00",
  "Disabled": false,
  "OneShot": false,
  "Synced": true,
  "SearchString": "tag=* count",
  "Duration": -3600,
  "SearchSinceLastRun": false,
  "Script": "",
  "PersistentMaps": {},
  "LastRun": "2019-05-21T16:01:00.013062447-06:00",
  "LastRunDuration": 1015958622,
  "LastSearchIDs": [
    "672586805"
  ],
  "LastError": ""
}
```

スケジュール検索は、GUIDで取得することも可能です。これはウェブサーバーにとってより多くの作業を必要とするので、必要な場合にのみ使用されるべきであることに注意してください。上記のスケジュール検索を取得するには、`/api/scheduledsearches/cdf011ae-7e60-46ec-827e-9d9fcb0ae66d`にGETしてください。

### 既存のスケジュール検索を更新する

スケジュール検索を修正するには、 `/api/scheduledsearches/{id}` にHTTP PUTを行い、希望する変更点を含む更新された構造体を送信してください。変更されていないフィールドもプッシュしないと、空の値で上書きされてしまうので注意してください。

以下のフィールドを更新できます:

* Name
* Description
* Schedule
* SearchString
* Duration
* SearchSinceLastRun
* Script
* Groups
* Global (admin only)
* Disabled
* OneShot

スクリプトスケジュール検索は、Scriptフィールドを空にしてSearchStringとDurationをプッシュすることにより、標準のスケジュール検索に変更することができます。同様に、標準のスケジュール検索は、Scriptフィールドをプッシュし、SearchStringを空に設定することによって、スクリプトスケジュール検索に変換することができる。

### スケジュール検索をクリアする

スケジュール検索構造体のLastErrorフィールドは、エラーが発生した場合に設定され、その後の実行が成功してもクリアされません。これは `/api/scheduledsearches/{id}/error` の DELETE によって手動でクリアすることができます。

### スケジュール検索のパーシステント状態をクリアする

`api/scheduledsearches/{id}/state`をDELETEすると、スケジュール検索のLastErrorフィールドと永続マップの両方がクリアされます。これにより、不正なスクリプトのために状態が破損した場合、スケジュール検索をリセットすることができます。

### スケジュール検索を削除する

既存のスケジュール検索は `/api/scheduledsearches/{id}` に対して DELETE を実行することで削除することができます。

## 管理者操作

以下のコマンドは、管理者のみ使用可能です。

### 全てのスケジュール検索を表示する

管理者ユーザーは、システム上のすべてのスケジュールされた検索を表示する必要がある場合があります。管理者ユーザーは `/api/scheduledsearches?admin=true` のGETリクエストで、システム内のすべてのスケジュール検索のグローバルリストを取得することができます。

スケジュール検索IDはシステム全体で一意であるため、管理者は`?admin=true`を指定しなくても、検索の修正/削除/検索を行うことができます（ただし、不必要にパラメータを追加してもエラーにはなりません）。

### 特定ユーザーのスケジュール検索を取得する

`api/scheduledsearches/user/{uid}` (`uid` は数値のユーザー ID) に対して GET を実行すると、そのユーザーに属するすべての検索の配列を取得することができます。

### 特定ユーザーの全てのスケジュール検索を取得する

`api/scheduledsearches/user/{uid}` に対して DELETE を実行すると、指定したユーザーに属するすべてのスケジュール検索が削除されます。

### スケジュール検索のテストパースを実施する

scheduledsearches APIは、スケジュールされた検索を保存する前にテストするためのAPIを提供します。 ParseAPIは `/api/scheduledsearches/parse` にあり、PUTリクエストでアクセスする。 認証されたユーザーは、既存のスケジュールスクリプトを保存したり修正したりすることなく、パースされチェックされるスケジュールスクリプトを送信することができます。

パースを実行するには、PUTリクエストのボディに以下のJSON構造を `/api/scheduledsearches/parse` として送信してください:

```
{
	Script string
}
```

APIは以下のようなJSON構造で応答します:

```
{
	OK bool
	Error string
	ErrorLine int
	ErrorColumn int
}
```

スクリプトがパーステストに成功すると、レスポンスのOKフィールドに `true` が含まれます。 Errorフィールドは省略され、ErrorLineとErrorColumnフィールドはともに `-1` となる。 もし、提供されたスクリプトが正しくパースできなかった場合は、OKフィールドは `false` となり、Errorフィールドは失敗の理由を、ErrorLineとErrorColumnはスクリプトのどこでエラーが発生したかを表します。

ErrorLineとErrorColumnフィールドは、常に入力されるとは限りません。 値 -1 は、スクリプト解析システムがスクリプトのどこにエラーがあるかを知らないことを示す。

以下は、リクエストとレスポンスの例です:

#### 有効なスクリプト
Request
```
{
	"Script":"fmt = import(\"fmt\")\nfmt.Println(\"Hello\")\nfmt.Sstuff(\"Goodbye\")\n"
}
```

Response
```
{
	"OK":true,
	"ErrorLine":-1,
	"ErrorColumn":-1
}
```

#### 無効なスクリプト
Request
```
{
	"Script":"fmt = import(\"fmt\")\nfmt.Println(\"Hello\")\nfmt.Sstuff(\"Goodbye)\n"
}
```

Response
```
{
	"OK":false,
	"Error":"syntax error",
	"ErrorLine":3,
	"ErrorColumn":21
}
```
