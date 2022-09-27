# リソースWebAPI

WebAPIは、リソースへのアクセスを提供する。リソースの参照は、曖昧さを防ぐため、GUIDで行う必要があります。

## リソースメタデータ構造
リソースシステムは、各リソースに対してメタデータ構造体を保持しています。Web APIは、この構造体をJSONでエンコードしたものを使って通信します。フィールドは概ね自明ですが、正確を期すため、ここで説明します:

* UID: リソース所有者のUID
* GUID: リソースの一意な識別子
* LastModified: リソースが最後に更新された時刻
* VersionNumber: リソースの内容が変更される度にインクリメントされる
* GroupACL: リソースへのアクセスを許可するグループ ID のリスト (整数)
* Global: trueの場合、そのリソースはシステム上のすべてのユーザーが読むことができます。管理者のみがグローバルリソースを作成可能
* ResourceName: リソース名
* Description: リソースの詳細
* Size: リソースのサイズ（バイト）
* Hash: リソースのSHA1ハッシュ
* Synced: (内部的に使用される)

## リソースをリスト表示する

すべてのリソースのリストを取得するには、`/api/resources`をGETしてください。結果はこのようになります:

```
[{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:19:10.945117816-07:00","VersionNumber":0,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"Description of the resource","Size":0,"Hash":"","Synced":true},{"UID":1,"GUID":"66f7be7d-893b-4dc4-b0ad-3609b348385d","LastModified":"2018-02-12T11:06:44.215431364-07:00","VersionNumber":1,"GroupACL":[1],"Global":false,"ResourceName":"test","Description":"test resource","Size":543,"Hash":"zkTmUEV+AR6JZdqhobIeYw==","Synced":true}]
```

この例では、"newresource" (GUID 2332866c-9b8d-469f-bf40-de9fad828362) と "test" (GUID 66f7be7d-893b-4dc4-b0ad-3609b348385d) の2つのリソースを表示します。

## リソースを作成する

リソースを作成するには、`/api/resources`に対してPOSTリクエストを行い、以下の形式のJSON構造体を送信します:

```
{
	"GroupACL": [3,7],
	"Global": false,
	"ResourceName": "newresource",
	"Description": "Description of the resource"
}
```

備考: 構造体は、メタデータ構造のサブセットであり、ユーザが設定可能なフィールドを含みます。

サーバーは、新しく作成されたリソースのリソースメタデータ構造で応答します:

```
{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:19:10.945117816-07:00","VersionNumber":0,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"Description of the resource","Size":0,"Hash":"","Synced":false}
```

## リソースコンテンツを設定する

新しく作成されたリソースはデータを持ちません。リソースの内容を変更するには、 `/api/resources/{guid}/raw` に対して、 `{guid}` をリソースの適切な GUID に置き換えて、マルチパートの PUT リクエストを発行してください。このリクエストには、リソースに格納されるデータを含む `file` という名前の1つのパートだけが必要です。したがって、上記で作成したリソースの内容を設定するには、 `/api/resources/2332866c-9b8d-469f-bf40-de9fad828362/raw` に対してマルチパートのPUTを実行する。サーバーは、新しい修正時刻、サイズ、ハッシュを示す更新されたメタ構造で応答します。リソースに "maxmind.db" というファイルをアップロードする際の curl 呼び出しの例を以下に示します (Bearer トークンはユーザセッションに応じて適切に設定する必要があることに注意してください。）:

```
curl 'http://datalaiq.example.com/api/resources/2332866c-9b8d-469f-bf40-de9fad828362/raw' -X PUT -H 'Authorization: Bearer 7b22616c676f223a35323733382c22747970223a226a3774227d.7b22756964223a312c2265787069726573223a22323031392d31302d30395431333a33333a32352e3231343632203131352d30363a3030222c22696174223a5b33392c32323c2c35382c36362c3231372c32362c3131392c33362c3234312c33352c39302c312c39312c3138312c3234322c33362c3137342c3139342c3130382c37342c3133382c32362c3133392c3234362c37362c3132352c3136342c38382c39322c39302c3231312c36365d7d.ef9ca1e0ac7f012adcd796d8cca0746a6fabecd7e787c025d754e54a072be5c89dc7bac5f648ae26b422f0bbe6b69a806e8de4a0fe2b7d06d3293ed4c1323daf' -H 'Content-Type: multipart/form-data' -H 'Accept: */*' --form file=@maxmind.db
```

## リソースコンテンツを読み込む

リソースの内容を読み込むには、`/api/resources/{guid}/raw`に対して、`{guid}` をリソースの適切なGUIDに置き換えて、GETリクエストを実行するだけでよい。

## リソースメタデータの読み込み/更新

一つのリソースのメタデータを読むには、 `/api/resources/{guid}` に対して GET リクエストを実行してください。例えば、 `/api/resources/2332866c-9b8d-469f-bf40-de9fad828362` に対して GET すると、以下のような結果が得られます:

```
{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:29:10.557490321-07:00","VersionNumber":1,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"Description of the resource","Size":6,"Hash":"QInZ92Blt3TopFBeBTD0Cw==","Synced":true}
```

メタデータは `/api/resources/{guid}` にPUTリクエストを行うことで変更することができます。リクエストの内容は、GETリクエストで読み込んだものと同じ構造で、必要なフィールドが変更されている必要があります。例えば、"newresource "の記述を変更するには、以下の内容でPUTを実行します:

```
{"UID":1,"GUID":"2332866c-9b8d-469f-bf40-de9fad828362","LastModified":"2018-03-07T15:29:10.557490321-07:00","VersionNumber":1,"GroupACL":[3,7],"Global":false,"ResourceName":"newresource","Description":"A new description for the resource!","Size":6,"Hash":"QInZ92Blt3TopFBeBTD0Cw==","Synced":true}
```

備考: 一般ユーザーが変更できるのは、GroupACL、ResourceName、Description の各フィールドのみです。管理者ユーザーはGlobalフィールドを変更することもできます。その他のフィールドの変更は、サーバーによって無視されます。

## リソースのコンテンツタイプを取得する

`api/resources/{guid}/contenttype` への GET リクエストは、検出されたリソースの content-type と、リソースボディの最初の 512 バイトを含む構造体を返します:

```
{"ContentType":"text/plain; charset=utf-8","Body":"IyBEdW1wcyB0aGUgcm93cyBvZiB0aGUgc3BlY2lmaWVkIENTViByZXNvdXJjZSBhcyBlbnRyaWVzLCB3aXRoCiMgZW51bWVyYXRlZCB2YWx1ZXMgY29udGFpbmluZyB0aGUgY29sdW1ucy4KIyBlLmcuIGdpdmVuIGEgcmVzb3VyY2UgbmFtZWQgImZvbyIgY29udGFpbmluZyB0aGUgZm9sbG93aW5nOgojCWhvc3RuYW1lLGRlcHQKIwl3czEsc2FsZXMKIwl3czIsbWFya2V0aW5nCiMJbWFpbHNlcnZlcjEsSVQKIyBydW5uaW5nIHRoZSBmb2xsb3dpbmcgcXVlcnk6CiMJdGFnPWRlZmF1bHQgYW5rbyBkdW1wIGZvbwojIHdpbGwgb3V0cHV0IDQgZW50cmllcyB3aXRoIHRoZSB0YWcgImRlZmF1bHQiLCBjb250YWluaW5nIGVudW1lcmF0ZWQKIyB2YWx1ZXMgbmFtZWQgImhvc3RuYW1lIiBhbmQgImRlcHQiIHdob3NlIGNvbnRlbnRzIG1hdGNoIHRoZSByb3dzCiMgb2YgdGhlIHJlc291cmNlLgojIEZsYWdzOgojICAtZDogc3BlY2lmaWVzIHRoYXQgaW5jb21pbmcgZW50cmllcyBzaG91bGQgYmUgZHJvcHBlZCA="}
```

bytesパラメータを追加すると、例えば `/api/resources/{guid}/contenttype?bytes=1024` のように、返されるバイト数が変更される。もし128バイト未満しか指定されなかった場合、APIはデフォルトで512バイトを読み込むことになります。

## リソースを削除する

リソースを削除するには、通常通り `/api/resources/{guid}` に DELETE リクエストを発行し、`{guid}` をリソースの適切な GUID に置き換えるだけでよい。

## リソースをクローンする

既存のリソースは `/api/resources/{guid}/clone` に対して、`{guid}` をオリジナルのリソースの GUID に置き換える POST リクエストを発行することでクローンを作成することができます。リクエストの本文は、新しく作成されるクローンの名前を含むJSON構造体である必要があります:

```
{
	"Name": "Copy of Foo"
}
```

サーバーは新しくクローンされたリソースのメタデータで応答します:

```
{
  "UID": 1,
  "GUID": "bbf682f8-363f-4245-8182-d7f6286022ff",
  "Domain": 0,
  "LastModified": "2020-08-19T20:31:11.258257937Z",
  "VersionNumber": 1,
  "GroupACL": null,
  "Global": false,
  "ResourceName": "Copy of Foo",
  "Description": "foobar",
  "Size": 207,
  "Hash": "xfV5dQG4eRe75ULzdb2e2A==",
  "Synced": false,
  "Labels": [
    "blah"
  ]
}
```

## 管理者操作

管理者ユーザは時々、システム上のすべてのリソースを表示する必要があるかもしれません。管理者ユーザーは `/api/resources?admin=true` の GET リクエストでシステム内のすべてのリソースのグローバルリストを取得することができます。

リソースGUIDはシステム全体で一意であるため、管理者は`?admin=true`を指定しなくてもリソースの変更・削除・取得を行うことができます（ただし、不必要にパラメータを追加してもエラーにはなりません）。