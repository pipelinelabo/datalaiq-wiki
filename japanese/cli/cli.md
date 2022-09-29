# DatalaiQ CLI

DatalaiQコマンドラインクライアントは、DatalaiQのリモート管理および検索の実行に使用できます（レンダラーのサポートは制限されています）。 管理者は、フル・ウェブ・ブラウザを使用せずにユーザーを管理し、クラスタの状態を監視することができます。 ユーザーは検索を実行し、結果をファイルにエクスポートして、他のツールでさらに分析することができます。

コマンドラインクライアントは、いくつかの検索結果をレンダリングできないという点で若干の制限があります（例えば、CLI はターミナルでチャートを描画できないので、チャートモジュールを使用する検索のレンダリングを拒否します）。 これは、上級ユーザがリモートでログインし、いくつかの非常に大きな検索を開始し、現地に到着したらフルブラウザで表示できるようにしたい場合に便利です。

典型的なインストールでは、CLIツールは `/usr/sbin/gravwell` としてインストールされます; `-h` フラグを渡すことで、どこから始めればよいかが分かります。 デフォルトでは、DatalaiQクライアントはローカルマシンのWebサーバーをリッスンすることを想定していますが、他のWebサーバーやリモートのDatalaiQインスタンスを指定する場合は `-s` フラグを指定してください。

```
DatalaiQ options
  -b	Background the search
  -debug string
    	Enable JSON output debugging to a file
  -f string
    	Output format: "simple" or "grid" (default "grid")
  -insecure
    	Do NOT enforce webserver certificates, TLS operates in insecure mode
  -insecure-no-https
    	Use insecure HTTP connection, passwords are shipped plaintext
  -o string
    	Output to file rather than stdio
  -query string
    	Query string
  -r	Raw output, no pretty print
  -s string
    	Address and port of DatalaiQ webserver
  -si
    	Enable additional search information output
  -t	Disable sessions, always require logins
  -time string
    	Query time range
  -v	Output client version number and exit
  -watch-interval int
    	Watch update interval
OPTIONS:
	shell: enter the interactive shell
	state: Show the state of configured indexers
	desc: Show the description of each connected indexer
	storage: Show the current state of each connected indexer
	systems: Show performance metrics of each addr
	indexes: Show size of each index
	ingesters: Show activity and performance metrics of each ingester
	sessions: Show your other active sessions
	notifications: Show your active notifications
	search: Perform search
	attach: Reattaches to existing search
	download: Download search results in a packaged format
	admin: Perform admin actions
	user: Perform user actions
	dashboards: Perform dashboard actions
	logout: Logout of the current session
	logoutall: Logout all sessions using your UID
	list_searches: list_searches
	search_ctrl: Issue search control command
	resource: Create and manage resources
	macro: Manage search macros
	kits: Manage and upload kits
	userfiles: Manage user files
	templates: Manage templates
	pivots: Manage actionables
	ingest: Ingest entries directly
	scheduled_search: Manage scheduled searches
	script: Run a script
	help: Display available commands
MODIFIERS:
	watch: Continually show results of stats commands

EXAMPLE: gravwell -s=localhost state
```

DatalaiQクライアントは、DatalaiQ上で検索を実行し、その出力を他のツールに供給するための優れた方法でもあります。 例えば、セキュリティ・データを処理するためのカスタム・プログラムを持っているが、ログ・エントリーをDatalaiQに保存したい場合、CLIクライアントを使用してバックグラウンド・クエリーを実行してエントリーを抽出し、その結果をファイルに保存してカスタム・プログラムに読み込ませることが可能です。

## CLIを対話的に使用する

DatalaiQ CLIクライアントは、商用スイッチに見られるものと同様の対話型シェルを提供します。このクライアントにはさまざまな「メニュー」レベルがあり、たとえばトップレベルのメニューから「dashboards」サブメニューを選択すると、ダッシュボードを管理するためのコマンドを含むことができます。このセクションでは、クライアントを対話的に使用するための基本的な方法を説明します。

### 接続とログイン

デフォルトでは、クライアントはDatalaiQのウェブサーバーが `localhost:443` でリスニングしていると仮定します。これが正しい場合は、単に `gravwell` というコマンドを実行することで接続できます。クライアントはユーザー名とパスワードの入力を促し、プロンプトを表示します:

```
$ gravwell
Username:  admin
Password:  changeme
#> 
```

ウェブサーバが別のホストにある場合は、`s`フラグを使用してホスト名とポートを指定します（例：`gravwell -s webserver.example.com:4443` ）。

もしあなたのウェブサーバーに自己署名のTLS証明書がインストールされている場合、TLS検証を無効にしてもHTTPSを使用するために `-insecure` フラグを追加する必要があります。

もしあなたのウェブサーバーにTLS証明書がインストールされていない場合は、`-insecure-no-https`フラグを追加してHTTPオンリーモードを使用してください。これは安全ではないことに注意してください。パスワードはプレーンテキストでサーバに送信されます。

### 使用可能なコマンドをリスト表示する

`help` コマンドは現在のメニューレベルで利用可能なコマンドをリストアップします。クライアントを起動した直後は、トップレベルである:

```
#>  help
shell                enter the interactive shell
state                Show the state of configured indexers
desc                 Show the description of each connected indexer
storage              Show the current state of each connected indexer
systems              Show performance metrics of each addr
indexes              Show size of each index
ingesters            Show activity and performance metrics of each ingester
sessions             Show your other active sessions
notifications        Show your active notifications
search               Perform search
attach               Reattaches to existing search
download             Download search results in a packaged format
admin                Perform admin actions
user                 Perform user actions
dashboards           Perform dashboard actions
logout               Logout of the current session
logoutall            Logout all sessions using your UID
list_searches        list_searches
search_ctrl          Issue search control command
resource             Create and manage resources
macro                Manage search macros
kits                 Manage and upload kits
userfiles            Manage user files
templates            Manage templates
pivots               Manage actionables
ingest               Ingest entries directly
scheduled_search     Manage scheduled searches
script               Run a script
help                 Display available commands
```

Some of the items listed are commands:

```
#>  state
+----------------------+----------+
|               System |    State |
+======================+==========+
|    192.168.2.60:9404 |       OK |
+----------------------+----------+
|            webserver |       OK |
+----------------------+----------+
```

その他は、それ自身のコマンドを含むメニューです。以下の例では、「dashboards」メニューを選択し、利用可能なコマンドをリストアップし、「list」コマンドを実行します:

```
#>  dashboards
dashboards>  help
list                	List available user dashboards
mine                	List dashboards owned by you
del                 	Delete a dashboard available user dashboards
clone               	Clone a dashboard to enable ownership and editing
dashboards>  list
+---------+-------+-----------------+------------------------------+----------+-----------+
|    Name |    ID |     Description |                      Created |    Owner |    Groups |
+=========+=======+=================+==============================+==========+===========+
|     Foo |    10 |    My dashboard |    2019-04-15T12:19:49-06:00 |    admin |           |
+---------+-------+-----------------+------------------------------+----------+-----------+
dashboards>  
```

## キット管理

キットの管理は、キットサブメニューから行うことができます:

```
#>  kits
kits>  help
list                	List installed kits
get                 	Get kit details
uninstall           	Remove an installed kit
install             	Install an uploaded kit
upload              	Upload a new kit
pull                	Pull a kit from a remote kitserver
build               	Build a kit from existing queries, resources, and dashboards
rebuild             	Rebuild a kit from a previous build request, incrementing the version
repack              	Repack a currently-installed kit, optionally changing attributes.
remote              	List available kits from the remote kitserver
```

### キットをインストールする

キットをインストールするには、kitserver からインストールする方法と、ローカルファイルをアップロードする方法の 2 通りがあります。どちらの場合も、インストール作業は、ウェブサーバ上でキットをステージングし、それをインストールするという2つのステップで構成されます。

キットサーバからキットをインストールするには、まず `remote` コマンドを使用してサーバ上のキットをリストアップします。目的のキットの UUID をコピーして、`pull`コマンドを実行し、プロンプトが表示されたら UUID を貼り付けてください。これでキットがダウンロードされ、ステージングされます。キットのステージングが完了したら、`install`コマンドを実行してステージングされたキットを選択すると、インストール処理が開始されます。

ローカルファイルからキットをインストールするには、`upload`コマンドを実行し、プロンプトが表示されたらファイルのパスを入力してください。これでキットがステージングされます。次に `install` コマンドを実行し、ステージングされたキットを選択すると、インストールが開始されます。

インストール`コマンドは、キットをデフォルトのオプションでインストールするかどうかをユーザに尋ねます。"no "と答えた場合、ユーザはそれぞれのオプションを個別に選択しなければなりません:

* Global: (管理者のみ) trueにすると全てのユーザーが閲覧可能です (デフォルト: false)
* Overwrite existing: True の場合、インストール時にキットの内容と競合する既存のオブジェクトを上書きします (デフォルト: false)。
* Allow unsigned: 無署名キットをインストールするために設定する必要があります (デフォルト: false)
* Item labels: キット内のアイテムに貼付する追加ラベルのオプション一覧 (デフォルト: none)
* Kit labels: キット本体に貼付する追加ラベルのオプション一覧 (デフォルト: none)
* Group: キットの内容を見ることができるグループを選択します (デフォルト: none)

### キットを構築する

`build` コマンドは、キットを構築するためのプロセスをユーザーに指示します。キットのID、名前、説明、そしてバージョンを入力するよう求められます。ID は競合を避けるために "io.gravwell.networkenrichment" のような "namespaced" ID であるべきであることに注意してください。名前と説明の欄は何でもかまいません。バージョンは整数値でなければなりません。

基本的な情報を収集した後、CLIはキットに含めるべきオブジェクトを選択するようユーザーに促します。ダッシュボード、テンプレート、actionableなどのプロンプトが次々と表示されます。プロンプトでEnterキーを押すと、そのタイプのオブジェクトを一切含めたくないことを示します。

すべてのオブジェクトが選択されると、CLIは結果のキットをダウンロードする場所を尋ねるプロンプトを表示します。ファイル名またはディレクトリのいずれかを入力することができます。完了すると、新しいキットの場所が表示されます。

### キットを再構築する

`rebuild` コマンドは、以前にビルドしたキットのアップデート版をビルドするために使用します。実行すると、以前にユーザがビルドしたキットの一覧が表示されます。ユーザーはそれを選択し、必要に応じてキットに含まれるアイテムのリストを変更するオプションが表示されます。その後、キットのバージョン番号が自動的に増加し、新しいキットファイルが生成されます。

### キットを再パッケージ化する

`repack` コマンドは `rebuild` コマンドとよく似た動作をしますが、*previously built* キットを再構築するのではなく、*currently-installed* キットの一つを再パッケージ化する点が異なります。DatalaiQ や他のユーザから入手した既存のキットを変更したい場合に便利です。キットをインストールし、変更する必要がある項目をすべて変更し、そのキットに対して repack コマンドを実行します。

## CLI経由で検索する

`search` コマンドはフォアグラウンドで検索を実行します:

```
#>  search
query>  tag=* count
time range> -1h
count 100
1/1
Press q[Enter] to quit, [Entry] to continue

Total Items: 1
101 stats records from Apr 15 12:39:37.000 <-> Apr 15 13:39:38.000
count 100.00/1.00 61.66 KB/616 B 8.109585ms
```

検索結果を保存したい場合は、 '-b' フラグを指定してクライアントを実行し、バックグラウンドで検索を実行するように指定します。そして、 `search` と `download` コマンドを使って検索を実行し、結果を保存します:

```
$ gravwell -b
#>  search
query>  tag=* json state=="NM"
time range> -1h
Background search with ID 065015787 launched
#>  download
+--------------+---------+----------+------------+---------------------+------------+
|    Search ID |    User |    Group |      State |    Attached Clients |    Storage |
+==============+=========+==========+============+=====================+============+
|    065015787 |       1 |        0 |    DORMANT |                   0 |    1.56 KB |
+--------------+---------+----------+------------+---------------------+------------+
search ID>  065015787
Available formats:
json
text
csv
format>  text
Save Location (default: /tmp)>  /tmp/nm.txt
Saving to  /tmp/nm.txt
#>  
```


## 管理者コマンド

DatalaiQクライアントには、システムを管理するための多くのコマンドがadminサブメニューに実装されています:

```
#>  admin
admin>  help
add_user            Add a new user
impersonate_user    Impersonate an existing users
del_user            Delete an existing user
get_user            Get an existing users details
update_user         Update an existing user
list_users          List all users
lock_user           Lock a user account
user_activity       Show a specific users activity
user_sessions       Show all open sessions
change_user_pwd     Change a users password
change_admin        Set a users admin status
add_group           Create a new group
del_group           Delete an existing group
list_groups         Lists all existing groups
list_group_users    Lists all members of an existing group
update_group        Update an existing group
add_users_group     Add users to an existing group
del_users_group     Delete users from an existing group
add_user_groups     Add user to existing groups
del_user_groups     Delete a user from groups
get_log_level       Get the webservers current logging level
set_log_level       Set the webservers current logging level
all_dashboards      Get all dashboards for all users
del_dashboard       Delete a dashboard owned by another user
license_info        Display license information
license_sku         Display license SKU
license_serial      Display license Serial Number
license_update      Upload a new license
list_queries        List all queries (active and saved) for all users
delete_queries      Delete any query (active or saved) for any user
list_users_storage  List all users current storage usage
add_indexer         Add another indexer to the configuration
list_kits           List all kits across all users
uninstall_kit       Uninstall a kit owned by any user
list_extractions    List installed autoextractors
add_extraction      Add a new autoextractor
delete_extraction   Delete an installed autoextractor
update_extraction   Update an installed autoextractor
sync_extractions    Force a sync of installed autoextractors to indexers
```

管理者メニューには、ユーザー/グループ管理の他に、システム上の他のユーザーが所有するダッシュボード、キット、その他のオブジェクトを管理するためのツールも用意されています。


## CLI例

### インデクサーの状態を確認する

```
$ gravwell state
+----------------------+----------+
|               System |    State |
+======================+==========+
|    10.0.0.2:9404     |       OK |
+----------------------+----------+
|    10.0.0.3:9404     |       OK |
+----------------------+----------+
|    webserver         |       OK |
+----------------------+----------+
```

すべてのコマンドの出力は、テーブルやフォーマットのない「生」に設定することができます。 DatalaiQのデータを他のツールやスクリプトに渡す場合、生の出力は消化しやすくなります。

```
$ gravwell -r state
10.0.0.3:9404 OK
webserver     OK
10.0.0.2:9404 OK
```

### ウェルとストレージサイズを確認する

```
$ gravwell -r indexes
10.0.0.2:9404 default /mnt/storage/gravwell/default 14.8 MB 93.76 K
10.0.0.2:9404 pcap /mnt/storage/gravwell/pcap 3.6 MB 29.63 K
10.0.0.2:9404 bench /mnt/storage/gravwell/bench 142.5 GB 686.66 M
10.0.0.2:9404 reddit /mnt/storage/gravwell/reddit 34.3 GB 73.72 M
10.0.0.2:9404 fcc /mnt/storage/gravwell/fcc 21.7 GB 11.09 M
10.0.0.2:9404 raw /mnt/storage/gravwell/raw 76.3 KB 0
10.0.0.2:9404 syslog /mnt/storage/gravwell/syslog 60.2 MB 406.62 K
10.0.0.3:9404 default /mnt/storage/gravwell/default 12.2 MB 77.56 K
10.0.0.3:9404 reddit /mnt/storage/gravwell/reddit 34.3 GB 73.66 M
10.0.0.3:9404 fcc /mnt/storage/gravwell/fcc 21.6 GB 11.06 M
10.0.0.3:9404 pcap /mnt/storage/gravwell/pcap 5.1 MB 44.85 K
10.0.0.3:9404 syslog /mnt/storage/gravwell/syslog 79.6 MB 536.86 K
10.0.0.3:9404 raw /mnt/storage/gravwell/raw 76.3 KB 0
10.0.0.3:9404 bench /mnt/storage/gravwell/bench 136.5 GB 658.69 M
```

### リモートインジェスターを確認する

```
$ gravwell -r ingesters
10.0.0.2:9404
        tcp://10.0.0.1:49544 111h27m33.8s [reddit] 5.16 M 2.68 GB
        tcp://192.210.192.202:45578 34m52.1s [pcap] 62.00 3.69 KB
        tcp://192.210.192.202:43369 34m51.9s [kernel] 1.1 K 121.43 KB
10.0.0.3:9404
        tcp://10.0.0.1:49770 111h27m0.01s [reddit] 5.24 M 2.72 GB
        tcp://192.210.192.202:43364 34m52.6s [pcap] 119.00 6.93 KB
        tcp://192.210.192.202:43368 34m51.9s [kernel] 1.33 K 141.57 KB
```

### スクリプトを実行する

```
$ gravwell script
script file path>  /tmp/myscript.ank
```
