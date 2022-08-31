# ユーザー定義オブジェクトの管理

DatalaiQのユーザーは以下のようなオブジェクトを独自に作成することが可能です:

* Resources
* Saved/backgrounded searches
* Scheduled searches/scripts
* Dashboards
* Templates
* Userfiles

現時点では、これらのオブジェクトを管理者として管理するためのGUI機能はありません。しかし、[DatalaiQ コマンドラインクライアント](#!cli/cli.md)では、**admin** サブメニューからこれらのオブジェクトタイプをリスト、削除、編集することができます。

これらの管理オプションにアクセスするためには、コマンドラインクライアントを実行し、管理者としてログイン後、管理者メニューに入る必要があります:

```
$ ./client -s gravwell.example.org
Username:  admin
Password:  
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
list_extractions    List installed autoextractors
add_extraction      Add a new autoextractor
delete_extraction   Delete an installed autoextractor
update_extraction   Update an installed autoextractor
sync_extractions    Force a sync of installed autoextractors to indexers
resource            Create and manage resources
scheduled_search    Manage scheduled searches
templates           Manage templates
pivots              Manage actionables
userfiles           Manage user files
kits                Manage and upload kits
admin>
```

残りのセクションでは各オブジェクトタイプ毎の管理オプションについて簡単に説明します。

## ダッシュボード管理

全てのダッシュボードをリスト表示したい場合には、**admin** メニューから `all_dashboards` コマンドを実行します。

ダッシュボードを削除するには、**admin** メニューから `del_dashboard` コマンドを実行します。

## 検索（クエリ）管理

システム上にある検索（クエリ）を全てリスト表示したい場合には、**admin** メニューから `list_queries` コマンドを実行します。

クエリを削除するには、**admin** メニューから `delete_queries` コマンドを実行します。

## Resources管理

admin サブメニューには、通常の resources メニューで使用できるコマンドを反映したコマンドで resources を管理するための独自のサブメニューが含まれています:

```
admin>  resource
resource>  help
list                	List available resources
create              	Create a new resource
update              	Upload new data to a resource
delete              	Delete a resource
updatemeta          	Update resource metadata
resource>  
```

このメニューから管理者はシステム上にある**全ての** resources をリスト表示し、resources のコンテンツを編集、権限の変更、削除を行うことができます。

## スケジュール検索管理

admin サブメニューにはスケジュール検索を管理するためのサブメニューが含まれています:

```
admin>  scheduled_search
scheduled search>  help
list                	List saved searches
listall             	List all saved searches
create              	Create a new scheduled search
createscript        	Create a new scheduled search w/ script
delete              	Delete a scheduled search
```

このメニューから、管理者はシステム上にある**全ての**スケジュール検索を管理することができます。

## テンプレート/アクショナブル管理

テンプレートとアクショナブル (ここでは「ピボット」と呼びます) には、管理者向けの同一のコマンド セットを含む admin メニュー内のサブメニュー (「テンプレート」と「ピボット」) があります:

```
admin>  templates
template>  help
list                	List templates
create              	Create a new template
update              	Upload new contents to a template
delete              	Delete a template
print               	Print template contents
updatemeta          	Update template metadata
template>  quit
admin>  pivots
pivot>  help
list                	List actionables
create              	Create a new actionable
update              	Upload new contents to an actionable
delete              	Delete an actionable
print               	Print actionable contents
updatemeta          	Update actionabl metadata
pivot>
```

これらのコマンドは、システム上にある任意のテンプレートとアクショナブルに変更を加えることができます。

## ユーザーファイル管理

テンプレート、リソースなどと同様に、ユーザーファイルにも、管理者管理用の admin メニュー内にサブメニューがあります。。コマンドを実行することで、システム上にある任意のユーザーファイルを操作することができます。

```
admin>  userfiles
userfile>  help
list                	List available userfiles
add                 	Add a new userfile
update              	Update an existing userfile
del                 	Delete a userfile
get                 	Download a userfile
userfile> 
```
