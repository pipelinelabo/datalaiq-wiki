# DatalaiQメトリクスとクラッシュレポート

DatalaiQ ユーザーは、自分のネットワークで何が起こっているかを気にかけています。それが、人々が最初に私たちをチェックする大きな理由です。 すべてのユーザーが、DatalaiQ に組み込まれている自動クラッシュレポートとメトリックシステムを認識し、快適に使用できるようにしたいと考えています。 このドキュメントでは、両方のシステムについて説明し、DatalaiQ サーバーに送り返す内容の例を示します。

## クラッシュレポート

DatalaiQ コンポーネントがクラッシュすると、自動クラッシュレポートがDatalaiQに送信されます。 これは、問題のコンポーネントからのコンソール出力で構成され、通常、ライセンスに関する簡単な情報 (クラッシュしたシステムを特定するため) とスタックトレースが含まれます。 **すべて**のDatalaiQコンポーネント (ウェブサーバー、インデクサー、インジェスター、検索エージェント) は、クラッシュレポートを送信するように設定されています。

備考: クラッシュレポートは常にupdate.gravwell.ioにHTTPS通信を使用して送信されます。リモート証明書を完全に検証できない場合、レポートは送信されません。

DatalaiQのテストシステムからのクラッシュレポートの例を次に示します:


```
Component:      webserver
Version:        3.3.5
Host:           X.X.X.X
Domain: c-X-X-X-X.hsd1.nm.comcast.net
Full Log Location:      /opt/gravwellCustomers/uploads/crash/webserver_3.3.5_X.X.X.X_2020-01-31T14:39:42


Log Snippet:
Version         3.3.10
API Version     0.1
Build Date      2020-Apr-30
Build ID        745dc6ca
Cmdline         /opt/gravwell/bin/gravwell_webserver -stderr gravwell_webserver.service
Executing user  gravwell
Parent PID      1
Parent cmdline  /sbin/init
Parent user     root
Total memory    4147781632
Memory used     5.781707651865122%
Process heap allocation 2005776
Process sys reserved    72112017
CPU count       4
Host hash       4224be94ae35247ed32013d9021f64bc40986c9fbbafac97787ab58b400f1666
Virtualization role     guest
Virtualization system   kvm
max_map_count   65530
RLIMIT_AS (address space)       18446744073709551615 / 18446744073709551615
RLIMIT_DATA (data seg)  18446744073709551615 / 18446744073709551615
RLIMIT_FSIZE (file size)        18446744073709551615 / 18446744073709551615
RLIMIT_NOFILE (file count)      1048576 / 1048576
RLIMIT_STACK (stack size)       8388608 / 18446744073709551615
SKU             2UX
Customer NUM    00000000
Customer GUID   ffffffff-ffff-ffff-ffff-ffffffffffff

panic: send on closed channel

goroutine 90 [running]:
gravwell/pkg/search.(*SearchManager).clientSystemStatsRoutine(0xc01414edc0,
0xc00017fd20, 0x16, 0xc000c300c0, 0xc000c1dc80)
        gravwell@/pkg/search/manager_handlers.go:284 +0x106
created by gravwell/pkg/search.(*SearchManager).GetSystemStats.func1
        gravwell@/pkg/search/manager_handlers.go:301 +0x65
```

メッセージはクラッシュした特定のコンポーネントから始まります、この例ではWebサーバーです。次に、DatalaiQのバージョン、クラッシュしたシステムのIPとホスト名、およびDatalaiQ管理者が確認可能なクラッシュログのパスを一覧表示します。

上記以外のメッセージはは、クラッシュしたプログラムからのコンソール出力です。クラッシュレポーターは、これを `/dev/shm` にあるコンポーネントの出力ファイルから直接取得します。 システムが何を送信するかを見ることができます。 `/dev/shm/gravwell_webserver`. `/opt/gravwell/log/crash` で過去のクラッシュレポートを表示することもできますが、クラッシュレポーターを「無効化」すると、クラッシュログがそのディレクトリに保存されなくなることに注意してください。

最初の数行は（バージョン、APIバージョン、ビルド日時とビルドID）はどのバージョンのDatalaiQで動作しているのかを確認するのに役立ちます。「Cmdline」、「Executing user」、「Parent PID」、「Parent cmdline」、および「Parent user」はすべて、DatalaiQプロセスがどのように実行されているかを把握し、潜在的な問題を特定するのに役立ちます。 PID 1としてプロセスを実行し、「manager」という名前を付けた場合、DatalaiQ が Docker コンテナーで実行されていると推測できます。1 つの環境 (「manager」によって起動される Docker など) で問題が発生することがありますが、他の環境 (Ubuntu、systemd 経由で起動される) では問題が発生しない場合もあります。

また、システムのメモリ量と設定された rlimits に関する情報も含めます。これは、特定のクラスのクラッシュを追跡するのに役立つためです。たとえば、512MBのRAMを搭載したシステムのメモリ不足エラーは 、「ホストハッシュ」フィールドは、プロセスを実行しているホストの一意の識別子ですが、これはハッシュであるため、「これは他のクラッシュレポートと同じマシンです」と言うためにしか使用できないことに注意してください。 他の情報は含まれていません。

「SKU」、「Customer NUM」、および「Customer GUID」フィールドは、使用中のライセンスを記述します。 SKUは、ユーザーのライセンスによって許可される機能を記述します。 この場合、DatalaiQ の従業員は無制限 (「UX」) ライセンスを使用しています。 顧客番号と顧客GUIDフィールドを使用すると、顧客データベースを参照して、誰が問題を抱えているかを確認できます。

このすべての情報の下には、DatalaiQ プロセスからのバックトレースがあります。 この場合、アルファビルドのバグにより、WebサーバーがGUIのハードウェア統計ページで使用するインデクサーのCPU/メモリ情報をチェックするために使用するルーチンでクラッシュが発生したことがわかります。 これらのスタックトレースにはユーザーデータは含まれず、ソースコードの行番号のみが含まれることを明確にしたいと思います。

### クラッシュレポートの無効化

何らかの理由でクラッシュレポートを送信しない場合は、レポートシステムをいくつかのオプションをオプションを使用して無効化することができます。

* シェルスクリプトのインストーラを使用している場合は、`--no-crash-report` フラグを使用することにより無効化できます。
* Debianレポジトリからインストールした場合は、`systemctl disable gravwell_crash_report` を使用して無効化できます。
* Dockerイメージを使用している場合は、`-e DISABLE_ERROR_HANDLING=true` をdockerコマンドで使用することにより無効化できます。

ただし、クラッシュレポートを有効にしていただけると幸いです。 これらのクラッシュレポートのおかげで、ユーザーが気付かない問題を特定して修正できることがよくあります。 これは、ソフトウェアを改善するための最良のフィードバックメカニズムの1つです。

お使いのシステムが送信した過去のクラッシュレポートをすべて削除することをご希望の場合は、support@ppln.coにメールしてください。システムから完全に削除します。

## メトリクスレポート

DatalaiQ ウェブサーバーコンポーネント (ウェブサーバーのみ) は、HTTPS POSTリクエストを一般的な使用統計とともにDatalaiQ企業サーバーに送信することがあります。 この情報は、どの機能が最も使用され、どの機能がより多くの作業を必要とするかを判断するのに役立ちます。 DatalaiQが消費しているRAMの量に関する統計を生成できます。ガベージコレクションを最適化する必要がありますか、それともデフォルト構成をより保守的にする必要がありますか? また、有料ライセンスが不適切に展開されていないことを確認することもできます。

これらの指標を収集する際の最も重要な目標は、データの匿名性を保護することです。 これらのメトリクスレポートには、DatalaiQに保存されているデータの実際の内容が含まれることはありません。また、システム上の実際の検索クエリやタグのリストを送信することもありません。

備考: クラッシュレポートは常にupdate.gravwell.ioにHTTPS通信を使用して送信されます。リモート証明書を完全に検証できない場合、レポートは送信されません。

これと同じシステムを使用して、DatalaiQ の新しいリリースをユーザーに通知します。メトリクス レポートが送信されると、サーバーはDatalaiQの最新バージョンで応答します。 これにより、新しいバージョンが利用可能になったときにDatalaiQ UIに通知を表示できます (これらの通知は、DatalaiQ.conf の「Disable-Update-Notification」パラメーターで無効にできます)。

Here's an example that was sent by a DatalaiQ employee's home system:
以下が実際に送信されたメトリクスレポートの例です:

```
{
    "ApiVer": {
        "Major": 0,
        "Minor": 1
    },
    "AutomatedSearchCount": 1,
    "BuildVer": {
        "BuildDate": "2020-04-02T00:00:00Z",
        "BuildID": "e755ee13",
        "GUIBuildID": "87e5e523",
        "Major": 3,
        "Minor": 3,
        "Point": 8
    },
    "CustomerNumber": 000000000,
    "CustomerUUID": "ffffffff-ffff-ffff-ffff-ffffffffffff",
    "DashboardCount": 5,
    "DashboardLoadCount": 13,
    "DistributedFrontends": false,
    "ForeignDashboardLoadCount": 0,
    "ForeignSearchLoadCount": 0,
    "Groups": 2,
    "IndexerCount": 4,
    "IndexerNodeInfo": [
        {
            "CPUCount": 12,
            "HostHash": "90578d2dcc5bea54614528e1b2c5a25c261cdd7c945f763d2387f309bdd38816",
            "ProcessHeapAllocation": 47899944,
            "ProcessSysReserved": 282423040,
            "TotalMemory": 67479150592,
            "VirtRole": "guest",
            "VirtSystem": "docker"
        },
        {
            "CPUCount": 12,
            "HostHash": "90578d2dcc5bea54614528e1b2c5a25c261cdd7c945f763d2387f309bdd38816",
            "ProcessHeapAllocation": 66157568,
            "ProcessSysReserved": 282554112,
            "TotalMemory": 67479150592,
            "VirtRole": "guest",
            "VirtSystem": "docker"
        },
        {
            "CPUCount": 12,
            "HostHash": "90578d2dcc5bea54614528e1b2c5a25c261cdd7c945f763d2387f309bdd38816",
            "ProcessHeapAllocation": 58577296,
            "ProcessSysReserved": 351827712,
            "TotalMemory": 67479150592,
            "VirtRole": "guest",
            "VirtSystem": "docker"
        },
        {
            "CPUCount": 12,
            "HostHash": "90578d2dcc5bea54614528e1b2c5a25c261cdd7c945f763d2387f309bdd38816",
            "ProcessHeapAllocation": 58304584,
            "ProcessSysReserved": 282226432,
            "TotalMemory": 67479150592,
            "VirtRole": "guest",
            "VirtSystem": "docker"
        }
    ],
    "IndexerStats": [
        {
            "WellStats": [
                {
                    "Cold": false,
                    "Data": 658757162,
                    "Entries": 2770447
                },
                {
                    "Cold": false,
                    "Data": 12681258,
                    "Entries": 9882
                },
                {
                    "Cold": false,
                    "Data": 325462303,
                    "Entries": 1344586
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 45312907669,
                    "Entries": 119150365
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 50161444,
                    "Entries": 297743
                }
            ]
        },
        {
            "WellStats": [
                {
                    "Cold": false,
                    "Data": 669469662,
                    "Entries": 2931573
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 325986097,
                    "Entries": 1348645
                },
                {
                    "Cold": false,
                    "Data": 50301788,
                    "Entries": 298556
                },
                {
                    "Cold": false,
                    "Data": 45316008062,
                    "Entries": 119174395
                },
                {
                    "Cold": false,
                    "Data": 12341038,
                    "Entries": 9559
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                }
            ]
        },
        {
            "WellStats": [
                {
                    "Cold": false,
                    "Data": 663669955,
                    "Entries": 2782081
                },
                {
                    "Cold": false,
                    "Data": 326449600,
                    "Entries": 1350525
                },
                {
                    "Cold": false,
                    "Data": 50427080,
                    "Entries": 299538
                },
                {
                    "Cold": false,
                    "Data": 12552734,
                    "Entries": 9759
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 45445347364,
                    "Entries": 119473828
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                }
            ]
        },
        {
            "WellStats": [
                {
                    "Cold": false,
                    "Data": 660249138,
                    "Entries": 2794164
                },
                {
                    "Cold": false,
                    "Data": 45332590720,
                    "Entries": 119204014
                },
                {
                    "Cold": false,
                    "Data": 50572152,
                    "Entries": 300251
                },
                {
                    "Cold": false,
                    "Data": 12608944,
                    "Entries": 9730
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 0,
                    "Entries": 0
                },
                {
                    "Cold": false,
                    "Data": 325899751,
                    "Entries": 1347670
                }
            ]
        }
    ],
    "IngesterCount": 8,
    "LicenseHash": "kH3R+R4AdTCnXFYDi3L4nZ==",
    "LicenseTimeLeft": 23550517079782204,
    "ManualSearchCount": 330,
    "ResourceUpdates": 13356,
    "ResourcesCount": 4,
    "SKU": "2UX",
    "ScheduledSearchCount": 4,
    "SearchCount": 331,
    "Source": "X.X.X.X",
    "SystemMemory": 67479150592,
    "SystemProcs": 3,
    "SystemUptime": 1920449,
    "TimeStamp": "2020-04-02T22:11:23Z",
    "TotalData": 185614443921,
    "TotalEntries": 494907311,
    "Uptime": 300,
    "UserLoginCount": 27,
    "Users": 2,
    "WebserverNodeInfo": {
        "CPUCount": 12,
        "HostHash": "90578d2dcc5bea54614528e1b2c5a25c261cdd7c945f763d2387f309bdd38816",
        "ProcessHeapAllocation": 311618224,
        "ProcessSysReserved": 420052881,
        "TotalMemory": 67479150592,
        "VirtRole": "guest",
        "VirtSystem": "docker"
    },
    "WebserverUUID": "17405830-3ac4-4b75-a639-6a265e6718a4",
    "WellCount": 28
}
```

このウェブサーバーは4つのインデクサーに接続されており、それぞれが独自の情報セットを取得していることもあり比較的大きいシステム構成です。以下は、フィールドの詳細です:

* `ApiVer`: DatalaiQのAPIバージョン
* `AutomatedSearchCount`: 自動的に（サーチエージェント、またはダッシュボードの読み込みによって）実行された検索の数
* `BuildVer`: このシステムにおけるDatalaiQの特定のビルドを説明する構造体
* `CustomerNumber`: このシステムのライセンスに紐づく顧客番号
* `CustomerUUID`: ライセンスのUUID
* `DashboardCount`: 存在するダッシュボードの数
* `DashboardLoadCount`: ユーザーが開いたダッシュボードの種類数
* `DistributedFrontends`: [分散ウェブサーバー](#!distributed/frontend.md) が有効であればtrueになる
* `ForeignDashboardLoadCount`: 他のユーザーが所有するダッシュボードをユーザーが閲覧した回数（ダッシュボードの共有オプションが十分に柔軟であるかどうかを判断するのに役立ちます。）
* `ForeignSearchLoadCount`: 他のユーザーが所有する検索をユーザーが閲覧した回数（検索共有オプションが十分に柔軟であるかどうかを判断するのに役立ちます。）
* `Groups`: システム上に定義されたグループの数
* `IndexerCount`: ウェブサーバーが接続しているインでクサーの数
* `IndexerNodeInfo`: 各インデクサーの統計情報を簡潔に記述した、インデックスごとに1つずつある構造体の配列:
	- `CPUCount`: インでクサーのCPUコア数
	- `HostHash`: インでクサーが動作しているサーバーを特定するための不可逆なハッシュ値([github.com/denisbrodbeck/machineid](https://github.com/denisbrodbeck/machineid)を参照) 。 この例では、インデクサはすべて1つのDockerホスト上で動作しているので、すべて同じHostHashを持っていることに注意してください。
	- `ProcessHeapAllocation`: インデクサープロセスに割り当てられたヒープメモリの数
	- `ProcessSysReserved`: OSからインデクサーに割り当てられたメモリの数
	- `TotalMemory`: システム上にあるメモリの合計
	- `VirtRole`: "host "または "guest "で、インデクサが仮想マシン内で動作しているかどうかによって異なる
	- `VirtSystem`: 仮想化システムがある場合は、"xen", "kvm", "vbox", "vmware", "docker", "openvz", "lxc "など。
* `IndexerStats`: インデクサーの統計情報:
	* `WellStats`: インデックスに登録された各Wellに関する匿名化された情報の配列:
		* `Cold`: コールドウェルかどうか
		* `Data`: ウェルに格納されているデータサイズ（バイト）
		* `Entries`: ウェルに格納されているエントリの数
* `IngesterCount`: システムに接続しているインジェスターの数
* `LicenseHash`: 使用されているライセンスのハッシュ値（MD5）
* `LicenseTimeLeft`: ライセンス有効期限の残り秒数
* `ManualSearchCount`: 手動で実行されたクエリの総数
* `ResourceUpdates`: リソースが変更された数
* `ResourcesCount`: システム上にあるリソースの数
* `SKU`: 使用されているライセンスのSKU
* `ScheduledSearchCount`: システム上に定義されているスケジュール検索の数
* `SearchCount`: 非推奨のフィールドの場合は、`ManualSearchCount` + `AutomatedSearchCount` の合計値となります。
* `Source`: この結果が作成されたマシンのIPアドレス
* `SystemMemory`: ウェブサーバーのホストに割り当てられているメモリのサイズ
* `SystemProcs`: システム上で動作しているプロセスの数
* `SystemUptime`: サーバーが起動している時間
* `TimeStamp`: この結果が作成された時間
* `TotalData`: 全てのインデクサーのウェル上にあるデータサイズの合計
* `TotalEntries`: 全てのインデクサーのウェル上にあるエントリの合計
* `Uptime`: ウェブサーバーが起動してからの時間
* `UserLoginCount`: ユーザーがログインした数
* `Users`: システム上に定義されているユーザーの数
* `WebserverNodeInfo`: ウェブサーバーが動作しているサーバーの簡易的な統計情報:
	- `CPUCount`: ウェブサーバーに割り当てられてCPUのコア数
	- `HostHash`: ウェブサーバーが動作しているマシンの不可逆的なハッシュ値([github.com/denisbrodbeck/machineid](https://github.com/denisbrodbeck/machineid)参照)
	- `ProcessHeapAllocation`: ウェブサーバーに割り当てられたヒープメモリのサイズ
	- `ProcessSysReserved`: OSからウェブサーバーに割り当てられたメモリサイズの合計
	- `TotalMemory`: システム上にあるメモリサイズの合計
	- `VirtRole`: ウェブサーバーが仮想マシンで実行されているかどうかによって、「ホスト」または「ゲスト」。
	- `VirtSystem`: 仮想化システムがある場合は、"xen", "kvm", "vbox", "vmware", "docker", "openvz", "lxc "など。
* `WebserverUUID`: ウェブサーバーインストール時に生成されるUUID
* `WellCount`: 各インデクサーに含まれるウェルの合計

私たちは、報告する情報を慎重に検討し、あなたがDatalaiQで入手したデータの種類や内容に関する情報を得ることができないように苦心しました。私たちが収集した情報の根拠について、いつでもご相談に応じます。ご質問は、support@ppln.co までお寄せください。

### メトリクスレポートの制限

gravwell.conf で `Disable-Stats-Report=true` を設定すると、メトリックレポートは、CustomerUUID、CustomerNumber、BuildVer、ApiVer、LicenseTimeLeft、および LicenseHash フィールドから最小限の情報（正しいライセンスがインストールされているか、システムが稼働しているかを確認するのに十分）に絞り込まれます。

しかし、完全な統計レポートを有効にしておいていただけると本当に助かります。上記で述べたように、これらの統計レポートは、どの機能が最も使用されているか、DatalaiQ がどのようなシステムで動作しているか、どの程度の RAM を使用しているか、といった情報を把握するのに役立つものです。