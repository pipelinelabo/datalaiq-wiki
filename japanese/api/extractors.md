# 自動抽出API

抽出WebAPIは、自動抽出定義へのアクセス、変更、追加、および削除のためのメソッドを提供します。より詳細な情報と設定については [自動抽出](/#!configuration/autoextractors.md) セクションを見てください。

## データ構造

自動抽出は以下のフィールドを含んでいます:

* Tag: 自動抽出を適用するタグ
* Name: 自動抽出定義の名前
* Desc: 自動抽出定義の詳細な説明
* Module: 抽出に使用するモジュール
* Params: 抽出に使用するモジュールに渡すパラメーター
* Args: 抽出モジュールの引数
* Labels: [ラベル](#!gui/labels/labels.md) のリスト
* UID: ダッシュボードの所有者のUID
* GIDs: ダッシュボードが共有されているグループのGID
* Global: 真偽値。全てのユーザーが閲覧可能にしたい場合はtrueに設定する（管理者のみ）。
* UUID: 自動抽出定義の一意なUID
* LastUpdated: 自動抽出定義が最後にアップデートされた日時

これは、構造に関するTypescriptの記述です:

```
type RawAutoExtractorModule = 'csv' | 'fields' | 'regex' | 'slice';

interface RawAutoExtractor {
	UUID: RawUUID;

	UID: RawNumericID;
	GIDs: Array<RawNumericID> | null;

	Name: string;
	Desc: string;
	Labels: Array<string> | null;

	Global: boolean;
	LastUpdated: string; // Timestamp

	Tag: string;
	Module: RawAutoExtractorModule;
	Params: string;
	Args?: string;
}
```

## 自動抽出定義をリスト表示する

api/autoextractors` に対して GET を実行すると、現在のユーザーがアクセスできる抽出定義のセットを表す JSON 構造体のリストが返されます。レスポンスの例は以下の通りです:

```
[
  {
    "Name": "Apache Combined Access Log",
    "Desc": "Apache Combined access logs using regex module.",
    "Module": "regex",
    "Params": "^(?P<ip>\\S+) (?P<ident>\\S+) (?P<auth>\\S+) \\[(?P<date>[^\\]]+)\\] \\\"(?P<method>\\S+) (?P<url>.+) HTTP\\/(?P<version>\\S+)\\\" (?P<response>\\d+) (?P<bytes>\\d+) \\\"(?P<referrer>\\S+)\\\" \\\"(?P<useragent>.+)\\\"",
    "Tag": "apache",
    "Labels": [
      "apache"
    ],
    "UID": 1,
    "GIDs": null,
    "Global": false,
    "UUID": "0e105901-92a7-4131-87bb-a00287d46f96",
    "LastUpdated": "2020-06-24T13:49:39.013266326-06:00"
  },
  {
    "Name": "vpcflow",
    "Desc": "VPC flow logs (TSV format)",
    "Module": "fields",
    "Params": "version, account_id, interface_id, srcaddr, dstaddr, srcport, dstport, protocol, packets, bytes, start, end, action, log_status",
    "Args": "-d \" \"",
    "Tag": "vpcflowraw",
    "Labels": null,
    "UID": 1,
    "GIDs": null,
    "Global": false,
    "UUID": "7f80df6a-a2ce-42aa-b531-ac11c596f64a",
    "LastUpdated": "2020-05-29T15:00:41.883390284-06:00"
  }
]
```

adminフラグをセットしてGETリクエストを実行すると(`/api/autoextractors?admin=true`)、システム上の*すべての*抽出定義のリストが返されます。

## タグで抽出設定をフィルタリングする

与えられたタグに対してシステムがどの抽出定義を使用するかを確認するには、 `/api/autoextractors/find/{tag}` に対してGETリクエストを発行します。"tag}" の部分は問題のタグに置き換えられます。例えば、"syslog "というタグをチェックするには、`/api/autoextractors/find/syslog`にGETを発行します。サーバーは、そのタグのための1つの自動抽出定義で応答し、一致する定義が存在しない場合は404で応答します。

## 自動抽出定義を追加する

自動抽出定義の追加は、リクエストボディに有効な定義を記述して `/api/autoextractors` へ POST を発行することで行われます。 構造体は有効である必要があり、ユーザーは同じタグに対して既存の自動抽出を定義することはできません。 新しい自動抽出ツールを追加する POST JSON 構造体の例は以下になります:

```
{
  "Tag": "foo",
  "Name": "my extractor",
  "Desc": "an extractor using the fields module",
  "Module": "fields",
  "Params": "version, account_id, interface_id, srcaddr, dstaddr, srcport, dstport, protocol, packets, bytes, start, end, action, log_status",
  "Args": "-d \" \"",
  "Labels": [
    "foo"
  ],
  "Global": false
}
```

自動抽出定義の追加時にエラーが発生した場合、ウェブサーバはエラーのリストを返します。成功した場合、サーバーは新しい抽出定義の UUID で応答します。

備考: 抽出定義を作成する際に、 `UUID`、`UID`、`GIDs`、`LastUpdated` フィールドを設定する必要はありません。これらは自動的に入力されます。管理者だけが `Global` フラグを true に設定することができます。

## 抽出定義を更新する

自動抽出定義の更新は、リクエストボディに有効な定義JSON構造を持つ `/api/autoextractors` にPUTリクエストを発行することで行われます。 この構造体は有効でなければならず、同じ UUID を持つ既存の自動抽出定義がなければなりません。 修正されていないすべてのフィールドは、サーバーから最初に返されたとおりに含まれている必要があります。定義が無効な場合、ボディにエラーメッセージを含む非200応答が返されます。構造は有効だが、更新された定義を配布する際にエラーが発生した場合、エラーのリストがボディで返されます。

## 抽出定義のシンタックスをテストする

自動抽出定義を追加したり更新したりする前に、構文を検証しておくと便利でしょう。api/autoextractors/test` への POST リクエストを実行すると、リクエストの検証が行われます:

```
{"Error":"asdf is not a supported engine"}
```

新しい自動抽出定義を追加するときは、その抽出機能が同じタグ上の既存の抽出定義と衝突しないことが重要です。既存の抽出定義を更新する場合、これは問題ではありません。指定したタグに抽出定義がすでに存在する場合、テストAPIは返された構造体に 'TagExists' フィールドを設定します:

```
{"TagExists":true,"Error":""}
```

TagExists` が true の場合、新しい抽出定義を作成する場合はエラーとして扱われ、既存の抽出定義を更新する場合は無視されます。

## 抽出定義ファイルをアップロードする

自動抽出の定義は TOML 形式で表現することができます。このフォーマットは人間が読むことができ、抽出定義を配布するのに便利な方法です。以下に例を示します:

```
[[extraction]]
	tag="bro-conn"
	name="bro-conn"
	desc="Bro conn logs"
	module="fields"
	args='-d "\t"'
	params="ts, uid, orig, orig_port, resp, resp_port, proto, service, duration, orig_bytes, dest_bytes, conn_state, local_orig, local_resp, missed_bytes, history, orig_pkts, orig_ip_pkts, resp_pkts, resp_ip_bytes, tunnel_parents"
```

このファイルをパースしてJSON構造を生成するのではなく、このタイプの定義は `/api/autoextractors/upload` へのPOSTリクエストで送られるマルチパートフォームを介してウェブサーバーに直接アップロードすることができます。このフォームには `extraction` という名前のファイルフィールドが含まれていて、そこに抽出定義の内容が格納されている必要があります。定義が有効であり、インストールに成功した場合、サーバーは 200 レスポンスで応答します。

## 抽出定義ファイルをダウンロードする

`api/autoextractors/download` に GET リクエストを発行すると、TOML 形式の自動抽出定義をダウンロードすることができます。ダウンロードしたい各定義について、その UUID をURLのパラメータとして追加します。UUID が ad782c81-7a60-4d5f-acbf-83f70e68ecb0 と c7389f9b-ba52-4cbe-b883-621d577c6bcc である2つの抽出定義をダウンロードしたい場合は、GET リクエストを `/api/autoextractors/download? id=ad782c81-7a60-4d5f-acbf-83f70e68ecb0&id=c7389f9b-ba52-4cbe-b883-621d577c6bcc`にします。

現在のユーザーが指定されたすべての抽出定義にアクセスできる場合、サーバーは、TOML形式の定義を含むダウンロード可能なファイルを応答します。このファイルは、前述のファイル・アップロードAPIを使用して、別のDatalaiQシステムにアップロードすることができます。

## 抽出定義を削除する

既存の自動抽出定義を削除するには、`/api/autoextractors/{uuid}` に DELETE リクエストを発行する。ここで `uuid` は自動抽出定義に関連付けられた UUID である。自動抽出定義が存在しないか、削除に失敗した場合、ウェブサーバはレスポンスボディに200以外のレスポンスとエラーを格納して応答します。

## 自動抽出に使用できるモジュールをリスト表示する

自動抽出定義には、有効なモジュールを指定する必要があります。サポートされているモジュールのリストを取得するためのAPIは、 `/api/autoextractors/engines` に対して GET リクエストを発行することで実行されます。 結果として得られるのは、以下のような文字列のリストです。:

```
[
	"fields",
	"csv",
	"slice",
	"regex"
]
```
