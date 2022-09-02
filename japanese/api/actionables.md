# アクショナブルAPI

アクショナブル（以前はピボットと呼んでいました。）は、ウェブGUIで検索した結果からデータをピボットするオブジェクトです。例えば、IPアドレスにマッチする正規表現を使ってIPアドレスを抽出するクエリを定義することができます。ユーザーが検索結果にIPアドレスが含まれるクエリを実行した時、結果に出てきたIPアドレスはクリックできるようになっており、クリックするとアクショナブルで事前定義されているクエリのメニューが表示されます。

## データ構造

アクショナブルには以下のフィールドが含まれています:

* GUID: 実行可能なグローバル リファレンス。 キットのインストール後も維持されます。（次のセクションを参照）
* ThingUUID: アクショナブルを特定する一意なID。 (次のセクションw御参照)
* UID: アクショナブルのオーナーユーザーのUID。
* GIDs: アクショナブルがシェアされているグループのGIDリスト。
* Global: 真偽値。trueに設定するとそのアクショナブルは全てのユーザーが閲覧することができます（管理者のみ設定可能）。
* Name: アクショナブルの名前。
* Description: アクショナブルの詳細説明。
* Updated: アクショナブルが最後に更新されたタイムスタンプ。
* Labels: [ラベル](#!gui/labels/labels.md)を含むリスト。
* Disabled: アクショナブルを無効にするかどうかの真偽値。
* Contents: アクショナブルの実際の設定内容 （以下を参照）。
  * Contents.menuLabel: 任意設定。指定されなければアクショナブルの名前から最初の20文字が使用されます。
  * Contents.actions: アクショナブルから実行可能なアクションのリスト。
    * Contents.actions\[n].name: アクション名。
    * Contents.actions\[n].description: アクションの説明。
    * Contents.actions\[n].placeholder: プレースホルダーは、トリガーまたはカーソルのハイライトの値に置き換えられます。デフォルトは"\_VALUE_"。
    * Contents.actions\[n].start: オプションの ActionableTimeVariable (以下のインターフェースを参照) と、開始日変数。
    * Contents.actions\[n].end: オプションの ActionableTimeVariable (以下のインターフェースを参照) と、終了日変数。
    * Contents.actions\[n].command: アクション実行の定義を含む ActionableCommand (以下のインターフェースを参照)。
  * Contents.triggers: Array of triggers for this actionable.
    * Contents.triggers\[n].pattern: 特定の値にアクショナブルをマッチさせるための正規表現。
    * Contents.triggers\[n].hyperlink: Trueに設定するとクリックと値の選択でアクショナブルが実行できるようになります。Falseの場合は値の選択のみで実行可能です。

Web サーバーは `Contents` フィールドに何が入るかを気にしませんが (有効な JSON であることを除いて)、**GUI** が使用する特定の形式があります。以下は、Contents フィールドと使用されるさまざまなタイプの説明を含む、アクション可能な構造の完全な Typescript 定義です。

```
interface Actionable {
    GUID: UUID;
    ThingUUID: UUID;
    UID: NumericID;
    GIDs: null | Array<NumericID>;
    Global: boolean;
    Name: string;
    Description: string; // 空でも可
    Updated: string; // タイムスタンプ
    Contents: {
        menuLabel: null | string;
        actions: Array<ActionableAction>;
        triggers: Array<ActionableTrigger>;
    };
    Labels: null | Array<string>;
    Disabled: boolean;
}

type UUID = string;
type NumericID = number;

interface ActionableTrigger {
    pattern: string;
    hyperlink: boolean;
}

interface ActionableAction {
    name: string;
    description: string | null;
    placeholder: string | null;
    start?: ActionableTimeVariable;
    end?: ActionableTimeVariable;
    command: ActionableCommand;
}

type ActionableTimeVariable =
    | { type: 'timestamp'; format: null | string; placeholder: null | string }
    | { type: 'string'; format: null | string; placeholder: null | string };

type ActionableCommand =
    | { type: 'query'; reference: string; options?: {} }
    | { type: 'template'; reference: UUID; options?: { variable?: string } }
    | { type: 'savedQuery'; reference: UUID; options?: {} }
    | { type: 'dashboard'; reference: UUID; options?: { variable?: string } }
    | { type: 'url'; reference: string; options: { modal?: boolean; modalWidth?: string } };
```

## GUIDs と ThingUUIDs について

アクショナブルはGUIDとThingUUIDという別々のIDが付与されます。どちらもUUIDですが、なぜ2つも使用しているのかを次のセクションで明確にします。

例: アクショナブルを最初から作成したので、ランダムな GUID である「e80293f0-5732-4c7e-a3d1-2fb779b91bf7」と、ランダムな ThingUUID である「c3b24e1e-5186-4828-82ee-82724a1d4c45」が割り当てられます。 次に、実用的なものをキットにバンドルします。 次に、同じシステム上の別のユーザーがこのキットを自分用にインストールすると**同じ** GUID (`e80293f0-5732-4c7e-a3d1-2fb779b91bf7`) と **ランダムな** ThingUUID (`f07373a8- ea85-415f-8dfd-61f7b9204ae0`)が割り当てられます。

このシステムは、[templates](templates.md) で使用されているものと同じです。 テンプレートは GUID と ThingUUID を使用するため、ダッシュボードは GUID でテンプレートを参照できますが、複数のユーザーが同じキット (サンプル テンプレートを使用) を競合することなく同時にインストールできます。 ダッシュボードがテンプレートを参照するのと同じ方法でアクショナブルを参照する DatalaiQ コンポーネントはありませんが、将来の保証として動作を含めました。

### GUID と ThingUUID を介したアクショナブルへのアクセス

通常ユーザーはGUIDによってアクショナブルにアクセスしなければなりません。管理者ユーザーは代わりにThingUUIDを使ってアクセスするかもしれませんが、リクエスト URL に「?admin=true」パラメーターを設定する必要があります。

## アクショナブルを作成

アクショナブルを作成するには `/api/pivots` にPOSTリクエストを送信します。bodyは 'Contents' フィールドと、オプションでGUID、ラベル、名前、詳細説明を含んだJSON形式のデータです。例えば:

```
{
  "Name": "IP actions",
  "Description": "Actions for an IP address",
  "Contents": {
    "actions": [
      {
        "name": "Whois",
        "description": null,
        "placeholder": null,
        "start": {
          "type": "string",
          "format": null,
          "placeholder": null
        },
        "end": {
          "type": "string",
          "format": null,
          "placeholder": null
        },
        "command": {
          "type": "url",
          "reference": "https://www.whois.com/whois/_VALUE_",
          "options": {}
        }
      }
    ],
    "menuLabel": null,
    "triggers": [
      {
        "pattern": "/\\b(?:[0-9]{1,3}\\.){3}[0-9]{1,3}\\b/g",
        "hyperlink": true
      }
    ]
  }
}
```

APIは新しく作成されたアクショナブルのGUIDを含むレスポンスを返します。リクエスト中でGUIDを指定すると指定されてGUIDが使用されます。GUIDが指定されなければランダムなGUIDが生成されます。

備考: 現時点では `UID`、`GID`、`Global`フィールドはアクショナブル作成時に指定することはできません。それらは更新時に指定されなければなりません（以下を参照）。

## アクショナブルのリスト表示

全てのアクショナブルをリスト表示するには `/api/pivots` にGETリクエストを送信します。結果はアクショナブルのリストになります:

```
[
  {
    "GUID": "afba4f9b-f66a-4f9f-9c58-f45b3db6e474",
    "ThingUUID": "196a3cc3-ec9e-11ea-bfde-7085c2d881ce",
    "UID": 1,
    "GIDs": null,
    "Global": false,
    "Name": "IP actions",
    "Description": "Actions for an IP address",
    "Updated": "2020-09-01T15:57:23.416537696-06:00",
    "Contents": {
      "actions": [
        {
          "name": "Whois",
          "description": null,
          "placeholder": null,
          "start": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "end": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "command": {
            "type": "url",
            "reference": "https://www.whois.com/whois/_VALUE_",
            "options": {}
          }
        }
      ],
      "menuLabel": null,
      "triggers": [
        {
          "pattern": "/\\b(?:[0-9]{1,3}\\.){3}[0-9]{1,3}\\b/g",
          "hyperlink": true
        }
      ]
    },
    "Labels": null,
    "Disabled": false
  },
  {
    "GUID": "34ba8372-0314-460a-9742-5a65c18d6241",
    "ThingUUID": "e1bdf35a-de7b-11ea-9709-7085c2d881ce",
    "UID": 1,
    "GIDs": [
      0
    ],
    "Global": false,
    "Name": "Network Port",
    "Description": "Actions to take on a network port, e.g. 22",
    "Updated": "2020-08-14T16:17:03.790048874-06:00",
    "Contents": {
      "actions": [
        {
          "name": "Netflow - Most active hosts on this port",
          "description": null,
          "placeholder": null,
          "start": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "end": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "command": {
            "type": "query",
            "reference": "tag=netflow netflow Src Dst SrcPort DstPort Port==_VALUE_ Protocol Bytes | stats sum(Bytes) as ByteTotal by Port Src Dst | lookup -r network_services Protocol proto_number proto_name as Proto Port service_port service_name as Service | table Src Dst Port Service Proto ByteTotal",
            "options": {}
          }
        },
        {
          "name": "Netflow - Chart traffic",
          "description": "Traffic on this port over time",
          "placeholder": null,
          "start": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "end": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "command": {
            "type": "query",
            "reference": "tag=netflow netflow Src Dst SrcPort DstPort Port==_VALUE_ Protocol Bytes | lookup -r network_services Protocol proto_number proto_name as Proto Port service_port service_name as Service | stats sum(Bytes) by Service Port | chart sum by Service Port",
            "options": {}
          }
        },
        {
          "name": "Netflow - Internal IPs serving this port",
          "description": null,
          "placeholder": null,
          "start": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "end": {
            "type": "string",
            "format": null,
            "placeholder": null
          },
          "command": {
            "type": "query",
            "reference": "tag=netflow netflow Dst ~ PRIVATE DstPort==_VALUE_ Bytes Protocol | lookup -r ip_protocols Protocol Number Name as ProtocolName | stats sum(Bytes) as TotalTraffic by Dst | table Dst DstPort Protocol ProtocolName TotalTraffic",
            "options": {}
          }
        }
      ],
      "menuLabel": null,
      "triggers": []
    },
    "Labels": [
      "kit/io.gravwell.netflowv5"
    ],
    "Disabled": false
  }
]
```

## 特定のアクショナブルを取得

特定のアクショナブルを取得するには `/api/pivots/<guid>` にGETリクエストを送信します。サーバーは指定されたアクショナブルの詳細を返します。例えば、`/api/pivots/afba4f9b-f66a-4f9f-9c58-f45b3db6e474` にGETリクエストを送信すると以下のようなレスポンスを得られます:

```
{
  "GUID": "afba4f9b-f66a-4f9f-9c58-f45b3db6e474",
  "ThingUUID": "196a3cc3-ec9e-11ea-bfde-7085c2d881ce",
  "UID": 1,
  "GIDs": null,
  "Global": false,
  "Name": "IP actions",
  "Description": "Actions for an IP address",
  "Updated": "2020-09-01T15:57:23.416537696-06:00",
  "Contents": {
    "actions": [
      {
        "name": "Whois",
        "description": null,
        "placeholder": null,
        "start": {
          "type": "string",
          "format": null,
          "placeholder": null
        },
        "end": {
          "type": "string",
          "format": null,
          "placeholder": null
        },
        "command": {
          "type": "url",
          "reference": "https://www.whois.com/whois/_VALUE_",
          "options": {}
        }
      }
    ],
    "menuLabel": null,
    "triggers": [
      {
        "pattern": "/\\b(?:[0-9]{1,3}\\.){3}[0-9]{1,3}\\b/g",
        "hyperlink": true
      }
    ]
  },
  "Labels": null,
  "Disabled": false
}

```


管理者は特定のアクショナブルを ThingUUID と adminパラメーター を使って取得することができます。例: `/api/pivots/196a3cc3-ec9e-11ea-bfde-7085c2d881ce?admin=true`

## アクショナブルを更新

アクショナブルを更新するには `/api/pivots/<guid>` にPUTリクエストを送信します。リクエストbodyは、同じパスのGETによって返されるものと同じである必要がありますが、必要な要素はすべて変更されています。GUIDとThingUUIDは変更することはできず、以下のフィールドを変更可能です:

* Contents: 実際のアクショナブルの内容
* Name: アクショナブルの名前
* Description: アクショナブルの詳細説明
* GIDs: GIDのリスト。例: `"GIDs":[1,4]`
* UID: (管理者のみ) UID
* Global: (管理者のみ) 真偽値を設定。trueに設定されると全てのメンバーが閲覧可能。

備考: 上記フィールドの中で空のまま送信されると、その項目はnull値で更新されます。

## アクショナブルを削除

アクショナブルを削除するには `/api/pivots/<guid>` にDELETEリクエストを送信します。

## 管理者操作

管理者ユーザーは、システム上のすべてのアクショナブルを表示したり、変更したり、削除したりする必要がある場合があります。GUID は必ずしも一意であるとは限らないため、管理 API は代わりに DatalaiQ がアイテムを保存するために内部で使用する一意の UUID を参照する必要があります。 上記のアクショナブルリストの例には、「ThingUUID」という名前のフィールドが含まれていることに注意してください。 これは、アクショナブルの内部の一意の識別子です。

管理者ユーザーは、`/api/pivots?admin=true` でGETリクエストを使用して、システム内のすべてのアクション可能アイテムのグローバル リストを取得できます。

その後、管理者は、PUT を使用して特定のアクショナブルを `/api/pivots/<ThingUUID>?admin=true` に更新し、特定のアクショナブルを ThingUUID 値に置き換えます。 同じパターンが削除に適用されます。

管理者は、`/api/pivots/<ThingUUID>?admin=true` で (それぞれ)GETまたはDELETEリクエストを使用して、特定のアクショナブルオブジェクトにアクセスまたは削除できます。