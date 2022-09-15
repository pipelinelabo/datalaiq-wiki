# キットAPI

このAPIは、DatalaiQキットの作成、インストール、および削除を実装しています。キットには、特定の問題に対してすぐに実行可能なソリューションを提供するために、ローカルシステムにインストールされる他のコンポーネントが含まれています。キットには以下を含めることができます:

* Resources（リソース）
* Scheduled searches（スケジュール検索）
* Dashboards（ダッシュボード）
* Auto-extractor definitions（自動抽出定義）
* Templates（テンプレート）
* Actionables（アクショナブル）
* User files（ユーザーファイル）
* Macros（マクロ）
* Search library entries（クエリライブラリ）

あるキットは、ビルド時に指定された以下の属性も持っています:

* ID: このキットに固有の識別子。Androidの命名規則に従うことをお勧めします。例："com.example.my-kit"
* Name: My Kit "のようなキットの名前
* Description: キットのより詳細な説明
* Readme: キットに何が入っていて、何ができるのか、Markdown形式の長い説明
* Version: キットのバージョン

## キットを作成する

キットのビルドは、以下のように定義されたKitBuildRequest構造を含むPOSTリクエストを `/api/kits/build` に送信することで行われます:

```
type KitBuildRequest struct {
	ID                string
	Name              string
	Description       string
	Readme            string
	Version           uint
	MinVersion        CanonicalVersion 
	MaxVersion        CanonicalVersion 
	Dashboards        []uint64 
	Templates         []uuid.UUID 
	Pivots            []uuid.UUID 
	Resources         []string 
	ScheduledSearches []int32 
	Macros            []uint64 
	Extractors        []uuid.UUID 
	Files             []uuid.UUID 
	SearchLibraries   []uuid.UUID 
	Playbooks         []uuid.UUID 
	EmbeddedItems     []KitEmbeddedItem 
	Icon              string 
	Banner            string 
	Cover             string 
	Dependencies      []KitDependency 
	ConfigMacros      []KitConfigMacro
	ScriptDeployRules map[int32]ScriptDeployConfig
}
```

ID、Name、Description、Version フィールドは必須ですが、テンプレート/Actionable/Dashboards などの配列は任意であることに注意してください。例えば、2つのダッシュボード、アクション可能なもの、リソース、およびスケジュール検索を含むキットを構築するリクエストは次のとおりです:

```
{
    "ConfigMacros": [
        {
            "DefaultValue": "windows",
            "Description": "Tag or tags containing Windows event entries",
            "MacroName": "KIT_WINDOWS_TAG"
        }
    ],
    "Dashboards": [
        7,
        10
    ],
    "Description": "Test DatalaiQ kit",
    "ID": "io.datalaiq.test",
    "Name": "test-datalaiq",
    "Description":"testing\n\n## TESTING",
    "Pivots": [
        "ae9f2598-598f-4859-a3d4-832a512b6104"
    ],
    "Resources": [
        "84270dbd-1905-418e-b756-834c15661a54"
    ],
    "ScheduledSearches": [
        1439174790
    ],
    "EmbeddedItems":[
        {
           "Name":"TEST",
           "Type":"license",
           "Content":"VGVzdCBsaWNlbnNlIHRoYXQgYWxsb3dzIEdyYXZ3ZWxsIHRvIGdpdmUgeW91ciBmaXJzdCBib3JuIHNvbiBhIHN0ZXJuIHRhbGtpbmcgdG8h"
        }
    ],
    "Files":[
        "810a014d-1373-4d57-95b6-0638a7a01442",
        "09a26a2e-e449-4857-88d1-56cede1b8d95",
        "92bcfe5e-2c9a-4f39-9083-dd3f7a6f9738"
    ],
    "MinVersion":{"Major":4,"Minor":0,"Point":0},
    "MaxVersion":{"Major":4,"Minor":2,"Point":0},
    "Icon":"810a014d-1373-4d57-95b6-0638a7a01442",
    "Banner":"09a26a2e-e449-4857-88d1-56cede1b8d95",
    "Cover":"92bcfe5e-2c9a-4f39-9083-dd3f7a6f9738",
    "ScriptDeployRules": {
        "1439174790": {
            "Disabled": true,
            "RunImmediately": false
        }
    },
    "Version": 1
}
```

注意: テンプレート、アクショナブルファイル、ユーザーファイルに指定されるUUIDは、アイテムのリストでも報告される*ThingUUID*フィールドではなく、それらの構造に関連付けられた*GUID*であるべきです

注意: バナー、カバー、アイコンに指定されたUUIDは、ビルドリクエストのファイルリストに含まれている必要があります。 ビルドリクエストに、メインファイルリクエストに含まれていないファイルUUIDへの参照が含まれている場合、APIサーバーはそのリクエストを拒否します

システムは、新しく構築されたキットを説明する構造で応答します:

```
{
	"UUID": "2f5e485a-2739-475b-810d-de4f80ae5f52",
	"Size": 8268288,
	"UID": 1
}
```

このキットは `/api/kits/build/<uuid>` を GET してダウンロードすることができます。上記のレスポンスがあれば、 `/api/kits/build/2f5e485a-2739-475b-810d-de4f80ae5f52` からキットを取得することができるでしょう。

### 依存関係

あるキットは、他のキットに依存している場合があります。これらの依存関係を、以下の構造で Dependencies 配列にリストアップします:

```
{
	ID			string
	MinVersion	uint
}
```

IDフィールドは依存関係のIDを指定します（例：io.datalaiq.testresource）。MinVersionフィールドは、インストールする必要があるそのキットの最小バージョンを指定します (例: 3)。

### マクロ設定

キットのインストール時にDatalaiQが作成する特殊なマクロである "コンフィグマクロ "を定義することができます。コンフィグマクロは次のようなものです。:

```
{
	"MacroName": "KIT_WINDOWS_TAG",
	"Description": "Tag or tags containing Windows event entries",
	"DefaultValue": "windows",
	"Value": "",
	"Type": "TAG"
}
```

UI は、インストール時にマクロの希望する値の入力を促し、ユーザーの応答を KitConfig 構造に含める必要があります。

コンフィグマクロの定義には、マクロが期待する値の種類を示すヒントとなるタイプフィールドを含めることができます。現在、以下のオプションが定義されています:

	* "TAG": 値は有効なタグでなければなりません。このタグは必ずしも現在のシステム上に存在する必要はありませんが、存在しないタグを入力した場合にチェックして警告を出すと便利でしょう。
	* "STRING": 自由形式の文字列を指定することができます。

Type が指定されない場合は、"STRING"（自由形式入力）となります。

### スクリプト実装設定

デフォルトでは、キットに含まれるスクリプトは、インストール時に有効に設定されます。この動作は、スクリプトのデプロイ設定構造で制御することができます:

```
{
	"Disabled": false,
	"RunImmediately": true,
}
```

この構造は、"Disabled "と "RunImmediately "という2つのフィールドを含んでいます。Disabledをtrueに設定すると、スクリプトは無効な状態でインストールされます。RunImmediately を true に設定すると、スクリプトはインストール後できるだけ早く実行されます（無効に設定されている場合でも）。

スクリプトのデプロイオプションは、*キットのビルド時*に設定するか、*キットのデプロイ時*に設定して、キットの組み込みオプションを上書きすることが可能です。

キットをビルドするとき、`ScriptDeployRules`フィールドには、スケジュールされたスクリプトID番号（`ScheduledSearches`フィールドにリストされている）からスクリプトデプロイ設定構造へのマッピングが含まれているはずです。

キットをインストールするとき、`ScriptDeployRules`フィールドには、スケジュールされたスクリプトの *名前* から設定へのマッピングが含まれている必要があります。デプロイオプションは、デフォルトを上書きしたい場合にのみ、インストール時に指定する必要があることに注意してください。

## キットをアップロードする

キットをインストールする前に、まずウェブサーバにキットをアップロードする必要があります。キットは `/api/kits` への POST リクエストによってアップロードされます。リクエストにはマルチパートフォームが含まれていなければなりません。ローカルシステムからファイルをアップロードするには、フォームに `file` という名前のファイルフィールドを追加し、そこにキットのファイルを記述します。HTTPサーバのようなリモートシステムからファイルをアップロードするには、キットのURLを含む `remote` という名前のフィールドを追加してください。

また、`metadata`というフィールドをリクエストに追加することができます。このフィールドの内容はサーバによって解析されません。その代わりに、アップロードされたキットのメタデータフィールドに内容が追加されます。これにより、例えばキットの発信元URLやキットがアップロードされた日付などを追跡することができます。

サーバーは、アップロードされたキットの説明を応答します（例）:

```
{
    "AdminRequired": false,
    "ConfigMacros": [
        {
            "DefaultValue": "windows",
            "Description": "Tag or tags containing Windows event entries",
            "MacroName": "KIT_WINDOWS_TAG",
            "Type": "TAG",
            "Value": "winlog"
        }
    ],
    "ConflictingItems": [
        {
            "AdditionalInfo": {
                "Description": "ASN database",
                "ResourceName": "maxmind_asn",
                "Size": 6196221,
                "VersionNumber": 1
            },
            "Name": "84270dbd-1905-418e-b756-834c15661a54",
            "Type": "resource"
        }
    ],
    "Description": "Test DatalaiQ kit",
    "GID": 0,
    "ID": "io.datalaiq.test",
    "Installed": false,
    "Items": [
        {
            "AdditionalInfo": {
                "Description": "ASN database",
                "ResourceName": "maxmind_asn",
                "Size": 6196221,
                "VersionNumber": 1
            },
            "Name": "84270dbd-1905-418e-b756-834c15661a54",
            "Type": "resource"
        },
        {
            "AdditionalInfo": {
                "Description": "My dashboard",
                "Name": "Foo",
                "UUID": "5567707c-8508-4250-9121-0d1a9d5ebe32"
            },
            "Name": "a",
            "Type": "dashboard"
        },
        {
            "AdditionalInfo": {
                "DefaultDeploymentRules": {
                    "Disabled": false,
                    "RunImmediately": true
                },
                "Description": "A script",
                "Name": "myScript",
                "Schedule": "* * * * *",
                "Script": "println(\"hi\")"
            },
            "Name": "5aacd602-e6ed-11ea-94d9-c771bfc07a39",
            "Type": "scheduled search"
        }
    ],
    "ModifiedItems": [
        {
            "AdditionalInfo": {
                "Description": "My dashboard",
                "Name": "Foo",
                "UUID": "5567707c-8508-4250-9121-0d1a9d5ebe32"
            },
            "Name": "a",
            "Type": "dashboard"
        }
    ],
    "Name": "test-datalaiq",
    "RequiredDependencies": [
        {
            "AdminRequired": false,
            "Assets": [
                {
                    "Featured": true,
                    "Legend": "Littering AAAAAAND",
                    "Source": "cover.jpg",
                    "Type": "image"
                },
                {
                    "Featured": false,
                    "Legend": "",
                    "Source": "readme.md",
                    "Type": "readme"
                }
            ],
            "Created": "2020-03-23T15:36:00.294625802-06:00",
            "Dependencies": null,
            "Description": "A simple test kit that just provides a resource",
            "ID": "io.datalaiq.testresource",
            "Ingesters": [
                "simplerelay"
            ],
            "Items": [
                {
                    "AdditionalInfo": {
                        "Description": "hosts",
                        "ResourceName": "devlookup",
                        "Size": 610,
                        "VersionNumber": 1
                    },
                    "Name": "devlookup",
                    "Type": "resource"
                },
                {
                    "AdditionalInfo": "Testkit resource\n\nThis really has no restrictions, go nuts!\n",
                    "Name": "LICENSE",
                    "Type": "license"
                }
            ],
            "MaxVersion": {
                "Major": 0,
                "Minor": 0,
                "Point": 0
            },
            "MinVersion": {
                "Major": 0,
                "Minor": 0,
                "Point": 0
            },
            "Name": "Testing resource kit",
            "Signed": true,
            "Size": 10240,
            "Tags": [
                "syslog"
            ],
            "UUID": "d2a0cb10-ff25-4426-8b87-0dd0409cae48",
            "Version": 1
        }
    ],
    "Signed": false,
    "UID": 7,
    "UUID": "549c0805-a693-40bd-abb5-bfb29fc98ef1",
    "Version": 2
}
```

ModifiedItems "フィールドに注意してください。このキットの以前のバージョンが既にインストールされている場合、このフィールドには *ユーザが変更した* 項目のリストが含まれます。ステージングされたキットをインストールすると、これらの項目が上書きされるため、ユーザーに通知し、変更を保存する機会を与える必要があります。

"ConflictingItems"は、ユーザが作成したオブジェクトと衝突するように見えるアイテムをリストアップします。この例では、ユーザは以前に "maxmind_asn" という名前の独自のリソースを作成したようです。`OverwriteExisting` を true に設定してインストール要求を送信すると、そのリソースはキットに含まれるバージョンで上書きされ、false に設定すると、インストール処理はエラーを返します

RequiredDependencies" フィールドには、このキットの現在インストールされていない依存関係のメタデータ構造のリストが含まれ、表示すべきライセンスが含まれる可能性のあるアイテムセットを含みます。

「ConfigMacros」フィールドには、このキットでインストールされるコンフィギュレーション・マクロ (前項参照) のリストが含まれます。このキットの以前のバージョン(または別のキット)が同じ名前のマクロを既にインストールしている場合、ウェブサーバは「Value」フィールドにマクロの現在の値を事前に入力します。ユーザー*が以前に同じ名前のマクロをインストールしたことがある場合、ウェブサーバーはエラーを返します。

"myScript" という名前のスケジュール検索、特に `DefaultDeploymentRules` フィールドに注目 してください。これはスクリプトがどのようにインストールされるかを記述しています：有効であるとマークされ、できるだけ早く実行されます。

## キットをリスト表示する

GET で `/api/kits` にリクエストすると、すべての既知のキットのリストが返されます。以下は、システムにアップロードされたがまだインストールされていないキットがある場合の結果を示す例です:

```
[
    {
        "AdminRequired": false,
        "Description": "Test DatalaiQ kit",
        "GID": 0,
        "ID": "io.datalaiq.test",
        "Installed": false,
        "Items": [
            {
                "AdditionalInfo": {
                    "Description": "ASN database",
                    "ResourceName": "maxmind_asn",
                    "Size": 6196221,
                    "VersionNumber": 1
                },
                "Name": "84270dbd-1905-418e-b756-834c15661a54",
                "Type": "resource"
            },
            {
                "AdditionalInfo": {
                    "DefaultDeploymentRules": {
                        "Disabled": false,
                        "RunImmediately": true
                    },
                    "Description": "count all entries",
                    "Duration": -3600,
                    "Name": "count",
                    "Schedule": "* * * * *",
                    "Script": "var time = import(\"time\")\n\naddSelfTargetedNotification(7, \"hello\", \"/#/search/486574780\", time.Now().Add(30 * time.Second))"
                },
                "Name": "55c81086",
                "Type": "scheduled search"
            },
            {
                "AdditionalInfo": {
                    "Description": "My dashboard",
                    "Name": "Foo",
                    "UUID": "5567707c-8508-4250-9121-0d1a9d5ebe32"
                },
                "Name": "a",
                "Type": "dashboard"
            },
            {
                "AdditionalInfo": {
                    "Description": "foobar",
                    "Name": "foo",
                    "UUID": "ae9f2598-598f-4859-a3d4-832a512b6104"
                },
                "Name": "ae9f2598-598f-4859-a3d4-832a512b6104",
                "Type": "pivot"
            }
        ],
        "Name": "test-datalaiq",
        "Signed": false,
        "UID": 7,
        "UUID": "549c0805-a693-40bd-abb5-bfb29fc98ef1",
        "Version": 1
    }
]
```

各タイプのキットアイテムで利用可能な "AdditionalInfo "フィールドについては、このページの最後にあるリストを参照してください。

## キットの情報を取得する

GETリクエストは `/api/kits/<GUID>` で、`<GUID>` はインストールまたはステージングされたキットの GUID で、その特定のキットに関する情報を提供します。

例えば、`/api/kits/549c0805-a693-40bd-abb5-bfb29fc98ef1`にGETリクエストすると、次のような結果が得られます:

```
{
    "AdminRequired": false,
    "Description": "Test DatalaiQ kit",
    "GID": 0,
    "ID": "io.datalaiq.test",
    "Installed": false,
    "Items": [
        {
            "AdditionalInfo": {
                "Description": "ASN database",
                "ResourceName": "maxmind_asn",
                "Size": 6196221,
                "VersionNumber": 1
            },
            "Name": "84270dbd-1905-418e-b756-834c15661a54",
            "Type": "resource"
        },
        {
            "AdditionalInfo": {
                "DefaultDeploymentRules": {
                    "Disabled": false,
                    "RunImmediately": true
                },
                "Description": "count all entries",
                "Duration": -3600,
                "Name": "count",
                "Schedule": "* * * * *",
                "Script": "var time = import(\"time\")\n\naddSelfTargetedNotification(7, \"hello\", \"/#/search/486574780\", time.Now().Add(30 * time.Second))"
            },
            "Name": "55c81086",
            "Type": "scheduled search"
        },
        {
            "AdditionalInfo": {
                "Description": "My dashboard",
                "Name": "Foo",
                "UUID": "5567707c-8508-4250-9121-0d1a9d5ebe32"
            },
            "Name": "a",
            "Type": "dashboard"
        },
        {
            "AdditionalInfo": {
                "Description": "foobar",
                "Name": "foo",
                "UUID": "ae9f2598-598f-4859-a3d4-832a512b6104"
            },
            "Name": "ae9f2598-598f-4859-a3d4-832a512b6104",
            "Type": "pivot"
        }
    ],
    "Name": "test-datalaiq",
    "Signed": false,
    "UID": 7,
    "UUID": "549c0805-a693-40bd-abb5-bfb29fc98ef1",
    "Version": 1
}

```

キットが存在しない場合は404が返され、ユーザーが要求された特定のキットへのアクセス権を持っていない場合は400が返されます。

## キットをインストールする

アップロードされたキットをインストールするには、`/api/kits/<uuid>` に PUT リクエストを送信します。UUIDはキットのリストにあるUUIDフィールドです。サーバーはいくつかの予備チェックを行い、整数を返します。この整数は、インストールステータス API (下記参照) を使ってインストールの進捗を問い合わせるために使うことができます。

インストール中、すべての必要な依存関係 (ステージング応答の RequiredDepdencies フィールドに記載) は、キット自体の最終インストールの前にステージングされ、自動的にインストールされます。

追加のキットのインストールオプションは、リクエストのボディで設定構造を渡すことで指定することができます:

```
{
    "AllowUnsigned": false,
    "ConfigMacros": [
        {
            "DefaultValue": "windows",
            "Description": "Tag or tags containing Windows event entries",
            "MacroName": "KIT_WINDOWS_TAG",
            "Value": "winlog"
        }
    ],
    "Global": true,
    "InstallationGroup": 3,
    "Labels": [
        "foo",
        "bar"
    ],
    "OverwriteExisting": true,
    "ScriptDeployRules": {
        "myScript": {
            "Disabled": true,
            "RunImmediately": false
        }
    }
}
```

備考: 以下はすべてオプションです。デフォルトのオプションを使用するには、リクエストからボディを省略するだけです。

`OverwriteExisting` がセットされていると、インストーラは、キットのバージョンと同じ名前の一意な識別子を持つ既存のアイテムを単純に置き換えるようになります。

`Global` フラグは管理者のみが設定することができます。設定された場合、すべてのアイテムはグローバルとしてマークされ、すべてのユーザがアクセスできるようになります。

一般ユーザは、DatalaiQから適切に署名されたキットのみをインストールすることができます。AllowUnsigned` が設定されている場合、*管理者* は署名されていないキットをインストールすることができます。

`InstallationGroup` は、インストールするユーザーが所属するグループの1つに、キットの内容を共有することを可能にします。

`Labels` は、インストール時にキット内のラベル付け可能な全てのアイテムに適用されるべき追加ラベルのリストです。DatalaiQは、キットにインストールされたアイテムに、自動的に "kit "とキットのID（例えば、"io.datalaiq.coredns"）をラベル付けすることに注意してください。

`ConfigMacros`は、キット情報構造体にあるConfigMacrosのリストで、Valueフィールドにはユーザーが望むものを任意に設定することができます。Value" フィールドが空白の場合、ウェブサーバーは "DefaultValue" を使用します。

`ScriptDeployRule`には、デプロイメントルールを上書きしたいキットのスケジュールされたスクリプトのオーバーライドが含まれている必要があります。この例では、"myScript" という名前のスクリプトが無効な状態でインストールされます。デフォルトのデプロイメント オプションで問題ない場合は、このフィールドを空にすることができます。

### インストールステータスAPI

インストール要求が送信されると、サーバはその要求を処理するためにキューに入れます。多くの依存関係を持つ大きなパッケージのインストールには、ある程度の時間がかかることがあるからです。サーバーはインストールリクエストに対して、例えば `2019727887` のような整数値で応答します。これをインストールステータス API と共に使用して、インストールの進捗を問い合わせるために `/api/kits/status/<id>` に GET リクエストを送ります。例えば、 `/api/kits/status/2019727887` は次のように返します。:

```
{
    "CurrentStep": "Done",
    "Done": true,
    "Error": "",
    "InstallID": 2019727887,
    "Log": "\nQueued installation of kit io.datalaiq.testresource, with 0 dependencies also to be installed\nBeginning installation of io.datalaiq.testresource (9b701e75-76ee-40fc-b9b5-4c7e1706339d) for user Admin John (1)\nInstalling requested kit io.datalaiq.testresource\nDone",
    "Owner": 1,
    "Percentage": 1,
    "Updated": "2020-03-25T15:39:37.184221203-06:00"
}
```

"Owner "は、インストール要求を送信したユーザーのUIDです。"Done "は、キットが完全にインストールされるとtrueに設定されます。"Percentage "は0から1の値で、インストールがどの程度完了したかを示します。"CurrentStep "はインストールの現在のステータスで、"Log "はインストール全体のステータスの完全な記録を保持しています。"Error "は、インストールプロセスで何か問題が発生しない限り、空白になります。"Updated "は、ステータスが最後に変更された時間です。

GET で `/api/kits/status` を実行すれば、すべてのキットのインストール状況の一覧を取得することもできます。管理者はURLに `?admin=true` を追加することで、システム上のすべてのステータスを取得することができます。

## キットをアンインストールする

キットを削除するには、`/api/kits/<uuid>` に対して DELETE リクエストを発行します。キットのアイテムがインストール後にユーザーによって変更された場合、レスポンスは400のステータスコードを持ち、何が変更されたかの詳細を示す構造が含まれます:

```
{
    "Error": "Kit items have been modified since installation, set ?force=true to override",
    "ModifiedItems": [
        {
            "AdditionalInfo": {
                "Description": "Network services (protocol + port) database",
                "ResourceName": "network_services",
                "Size": 531213,
                "VersionNumber": 1
            },
            "ID": "2e4c8f31-92a4-48b5-a040-d2c895caf0b2",
            "KitID": "io.datalaiq.networkenrichment",
            "KitName": "Network enrichment",
            "KitVersion": 1,
            "Name": "network_services",
            "Type": "resource"
        }
    ]
}
```

UIはこの時点でユーザーにプロンプトを表示します。キットの削除を強制するには、`?force=true`パラメータをリクエストに追加します。

## リモートキットサーバーにクエリする

DatalaiQ Kit Server からリモートキットのリストを取得するには、 `/api/kits/remote/list` に対して GET を実行します。 これは、利用可能なすべてのキットの最新バージョンを表す、キットのメタデータ構造をJSONでエンコードしたリストを返します。 APIパスの `/api/kits/remote/list/all` は、すべてのバージョンのすべてのキットを提供します。

メタデータの構造は以下の通りです:

```
type KitMetadata struct {
	ID            string
	Name          string
	GUID          string
	Version       uint
	Description   string
	Signed        bool
	AdminRequired bool
	MinVersion    CanonicalVersion
	MaxVersion    CanonicalVersion
	Size          int64
	Created       time.Time
	Ingesters     []string //ingesters associated with the kit
	Tags          []string //tags associated with the kit
	Assets        []KitMetadataAsset
}

type KitMetadataAsset struct {
	Type     string
	Source   string //URL
	Legend   string //some description about the asset
	Featured bool
}

type CanonicalVersion struct {
	Major uint32
	Minor uint32
	Point uint32
}
```

例は以下の通りです:

```
WEB GET http://172.19.0.2:80/api/kits/remote/list:
[
	{
		"ID": "io.datalaiq.test",
		"Name": "testkit",
		"GUID": "c2870b48-ff31-4550-bd58-7b2c1c10eeb3",
		"Version": 1,
		"Description": "Testing a kit with a license in it",
		"Signed": true,
		"AdminRequired": false,
		"MinVersion": {
			"Major": 0,
			"Minor": 0,
			"Point": 0
		},
		"MaxVersion": {
			"Major": 0,
			"Minor": 0,
			"Point": 0
		},
		"Size": 0,
		"Created": "2020-02-10T16:31:23.03192303Z",
		"Ingesters": [
			"SimpleRelay",
			"FileFollower"
		],
		"Tags": [
			"syslog",
			"auth"
		],
		"Assets": [
			{
				"Type": "image",
				"Source": "cover.jpg",
				"Legend": "TEAM RAMROD!",
				"Featured": true
			},
			{
				"Type": "readme",
				"Source": "readme.md",
				"Legend": "",
				"Featured": false
			},
			{
				"Type": "image",
				"Source": "testkit.jpg",
				"Legend": "",
				"Featured": false
			}
		]
	}
]
```

## 特定のキットの情報を取得する

リモートキットAPIは、 `/api/kits/remote/<guid>` に対して `GET` を発行して、特定のキットに関する情報を取得することもサポートしており、その場合は単一の `KitMetadata` 構造を返します。

例えば、`/api/kits/remote/c2870b48-ff31-4550-bd58-7b2c1c10eeb3` に対して `GET` を実行すると、ウェブサーバーは次のような結果を返します:

```
{
	"ID": "io.datalaiq.test",
	"Name": "testkit",
	"GUID": "c2870b48-ff31-4550-bd58-7b2c1c10eeb3",
	"Version": 1,
	"Description": "Testing a kit with a license in it",
	"Signed": true,
	"AdminRequired": false,
	"MinVersion": {
		"Major": 0,
		"Minor": 0,
		"Point": 0
	},
	"MaxVersion": {
		"Major": 0,
		"Minor": 0,
		"Point": 0
	},
	"Size": 0,
	"Created": "2020-02-10T16:31:23.03192303Z",
	"Ingesters": [
		"SimpleRelay",
		"FileFollower"
	],
	"Tags": [
		"syslog",
		"auth"
	],
	"Assets": [
		{
			"Type": "image",
			"Source": "cover.jpg",
			"Legend": "TEAM RAMROD!",
			"Featured": true
		},
		{
			"Type": "readme",
			"Source": "readme.md",
			"Legend": "",
			"Featured": false
		},
		{
			"Type": "image",
			"Source": "testkit.jpg",
			"Legend": "",
			"Featured": false
		}
	]
}
```

### リモートキットサーバーからキットのアセットを取得する

キットはまた、画像、マークダウン、ライセンス、および実際にキットをダウンロード/インストールする前にキットの目的を探るのに役立つ追加ファイルを表示するために使用できるアセットを含んでいます。 これらのアセットは `api/kits/remote/<guid>/<asset>` で GET リクエストを実行することでリモートシステムから取得することができます。 例えば、GUID `c2870b48-ff31-4550-bd58-7b2c1c10eeb3` のキットの Type "image" と Legend "TEAM RAMROD!" のアセットを取得したい場合は、`/api/kits/remote/c2870b48-ff31-4550-bd58-7b2c1c10eeb3/cover.jpg`に対して GET を実行することになります。


## キットに関する追加のフィールド

キットをリストアップするとき（`/api/kits`でGET）、各キットはAddditionalInfoフィールドを含むアイテムのリストを含んでいます。これらのフィールドは、キット内のアイテムに関する詳細な情報を提供します。内容はアイテムの種類によって異なり、以下のように列挙されます:

```
Resources:
		VersionNumber int
		ResourceName  string
		Description   string
		Size          uint64

Scheduled Search:
		Name                    string
		Description             string
		Schedule                string
		SearchString            string 
		Duration                int64  
		Script                  string 
		DefaultDeploymentRules  ScriptDeployConfig

Dashboard:
		UUID        string
		Name        string
		Description string

Extractor:
		Name   string 
		Desc   string 
		Module string 
		Tag    string 

Template:
		UUID        string
		Name        string
		Description string

Pivot:
		UUID        string
		Name        string
		Description string

File:
		UUID        string
		Name        string
		Description string
		Size        int64
		ContentType string

Macro:
		Name      string
		Expansion string

Search Library:
		Name        string
		Description string
		Query       string

Playbook:
		UUID        string
		Name        string
		Description string

License:
		(contents of license file itself)
```

## キットビルドリクエストの履歴を確認する

成功したキットのビルドリクエストはウェブサーバによって保存されます。GET リクエストを `/api/kits/build/history` に送ると、現在のユーザーに対するビルドリクエストの一覧を取得できます。レスポンスとして、ビルドリクエストの配列が返されます:

```
[{"ID":"io.datalaiq.test","Name":"test","Description":"","Version":1,"MinVersion":{"Major":0,"Minor":0,"Point":0},"MaxVersion":{"Major":0,"Minor":0,"Point":0},"Macros":[4,41],"ConfigMacros":null}]
```

備考: このストアはUID + キットIDをキーにしています。"io.datalaiq.test "というキットを再度ビルドすると、ストア内のバージョンは上書きされます。

特定の項目を削除するには、`/api/kits/build/history/<id>` に DELETE リクエストを送ります。例えば `/api/kits/build/history/io.datalaiq.test` などです。
