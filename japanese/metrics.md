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

The structure is large, in part because this webserver is connected to 4 indexers which each get their own set of information. Here's a breakdown of the fields in detail:

* `ApiVer`: An internal DatalaiQ API versioning number.
* `AutomatedSearchCount`: The number of searches which have been executed "automatically" (by the search agent, or by loading a dashboard).
* `BuildVer`: A structure describing the particular build of DatalaiQ on this system.
* `CustomerNumber`: The customer number associated with the license on this system.
* `CustomerUUID`: The UUID of the license on this system.
* `DashboardCount`: The number of dashboards that exist.
* `DashboardLoadCount`: The number of types any dashboard has been opened by any user.
* `DistributedFrontends`: Set to true if [distributed webservers](#!distributed/frontend.md) are enabled.
* `ForeignDashboardLoadCount`: The number of times users have viewed dashboards owned by another user (helps us determine if our dashboard sharing options are sufficiently flexible)
* `ForeignSearchLoadCount`: The number of times users have viewed searches owned by another user (helps us determine if our search sharing options are sufficiently flexible)
* `Groups`: The number of user groups on the system.
* `IndexerCount`: The number of indexers to which this webserver is connected.
* `IndexerNodeInfo`: An array of structures, one per indexer, briefly describing the statistics of each indexer:
	- `CPUCount`: The number of CPU cores on the indexer.
	- `HostHash`: A non-reversible hash (see [github.com/denisbrodbeck/machineid](https://github.com/denisbrodbeck/machineid)) that uniquely identifies the host machine running the indexer. Note that in this example, the indexers are all running on a single Docker host, so they all have the same HostHash.
	- `ProcessHeapAllocation`: The amount of heap memory allocated by the indexer process.
	- `ProcessSysReserved`: The total amount of memory the indexer process has reserved from the OS.
	- `TotalMemory`: The size of the system's main memory.
	- `VirtRole`: "host" or "guest", depending on if the indexer is running in a virtual machine or not.
	- `VirtSystem`: The virtualization system, if any, e.g. "xen", "kvm", "vbox", "vmware", "docker", "openvz", "lxc".
* `IndexerStats`: An array of statistics structures for each indexer:
	* `WellStats`: An array of anonymized information about each well on the indexer:
		* `Cold`: Whether or not this is a "cold" well.
		* `Data`: The number of bytes of data in this well.
		* `Entries`: The number of entries in this well.
* `IngesterCount`: The number of unique ingesters attached to the system.
* `LicenseHash`: The MD5 sum of the license in use.
* `LicenseTimeLeft`: The number of seconds remaining in the license.
* `ManualSearchCount`: The number of searches executed manually.
* `ResourceUpdates`: The number of times any resource has been modified.
* `ResourcesCount`: The number of resources on the system.
* `SKU`: The SKU of the license in use.
* `ScheduledSearchCount`: The number of scheduled searches installed on the system.
* `SearchCount`: a deprecated field, the total of `ManualSearchCount` + `AutomatedSearchCount`.
* `Source`: The IP from which this report originated.
* `SystemMemory`: How many bytes of memory are installed on the webserver's host system.
* `SystemProcs`: The number of processes running on the host system.
* `SystemUptime`: Number of seconds the host system has been running.
* `TimeStamp`: The time at which this report was generated.
* `TotalData`: The number of bytes across all wells on all indexers.
* `TotalEntries`: The number of entries across all wells on all indexers.
* `Uptime`: The number of seconds since the webserver process started.
* `UserLoginCount`: The number of times users have logged in.
* `Users`: The number of users on the system.
* `WebserverNodeInfo`: A brief description of the system running the webserver process:
	- `CPUCount`: The number of CPU cores on the webserver.
	- `HostHash`: A non-reversible hash (see [github.com/denisbrodbeck/machineid](https://github.com/denisbrodbeck/machineid)) that uniquely identifies the host machine running the webserver.
	- `ProcessHeapAllocation`: The amount of heap memory allocated by the webserver process.
	- `ProcessSysReserved`: The total amount of memory the webserver process has reserved from the OS.
	- `TotalMemory`: The size of the system's main memory.
	- `VirtRole`: "host" or "guest", depending on if the webserver is running in a virtual machine or not.
	- `VirtSystem`: The virtualization system, if any, e.g. "xen", "kvm", "vbox", "vmware", "docker", "openvz", "lxc".
* `WebserverUUID`: Every DatalaiQ webserver generates a UUID when installed; this field contains that UUID.
* `WellCount`: The total number of wells across all indexers.

We carefully considered the information we report, taking pains to make it impossible to glean any intelligence regarding the type or content of the data you've got in DatalaiQ. We are always happy to discuss the reasoning behind any of the information we gather; please email support@ppln.co with any questions.

### Limiting Metric Reporting

Customers may set `Disable-Stats-Report=true` in gravwell.conf, which will strip down the metrics report to a bare minimum containing the CustomerUUID, CustomerNumber, BuildVer, ApiVer, LicenseTimeLeft, and LicenseHash fields--just enough information to verify that the correct license is installed and the system is running.

We'd really appreciate if you'd leave full stats reports enabled, though. As we said above, these stats reports help us figure out which features are getting used the most, what kind of systems DatalaiQ is running on, how much RAM it's using--information that, in aggregate, can help us decide where to prioritize future improvements.
