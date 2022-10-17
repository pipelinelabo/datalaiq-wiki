# 設定パラメータ

設定ファイル `gravwell.conf` はインデクサ、ウェブサーバ、データストアの設定に使用されます。設定は *セクション* を含んでおり、角括弧の中で定義されます（例：`[Global]`）。各セクションはパラメータを含んでいます。次の例では、Globalセクション、Replicationセクション、Default-Wellセクション、およびStorage-Wellセクションを含んでいます:

```
[global]
### Authentication tokens
Ingest-Auth=IngestSecrets
Control-Auth=ControlSecrets
Search-Agent-Auth=SearchAgentSecrets

### Web server HTTP/HTTPS settings
Web-Port=80
Insecure-Disable-HTTPS=true

### Other web server settings
Remote-Indexers=net:127.0.0.1:9404
Persist-Web-Logins=True
Session-Timeout-Minutes=1440

### Ingester settings
Ingest-Port=4023
Control-Port=9404
Search-Pipeline-Buffer-Size=4

### Other settings
Log-Level=WARN

### Paths
Pipe-Ingest-Path=/opt/gravwell/comms/pipe
Log-Location=/opt/gravwell/log
Web-Log-Location=/opt/gravwell/log/web
Render-Store=/opt/gravwell/render
Saved-Store=/opt/gravwell/saved
Search-Scratch=/opt/gravwell/scratch
Web-Files-Path=/opt/gravwell/www
License-Location=/opt/gravwell/etc/license
User-DB-Path=/opt/gravwell/etc/users.db
Web-Store-Path=/opt/gravwell/etc/webstore.db

[Replication]
	Peer=10.0.0.1
	Storage-Location=/opt/gravwell/replication_storage
	Insecure-Skip-TLS-Verify=true
	Disable-TLS=true
	Connect-Wait-Timeout=20

[Default-Well]
	Location=/opt/gravwell/storage/default/
	Accelerator-Name=fulltext #fulltext is the most resilient to varying data types
	Accelerator-Engine-Override=bloom #The bloom engine is effective and fast with minimal disk overhead
	Disable-Compression=true

[Storage-Well "syslog"]
	Location=/opt/gravwell/storage/syslog
	Tags=syslog
	Tags=kernel
	Tags=dmesg
	Accelerator-Name=fulltext #fulltext is the most resilient to varying data types
	Accelerator-Args="-ignoreTS" #tell the fulltext accelerator to not index timestamps, syslog entries are easy to ID
```

各パラメーターにはデフォルト値があり、パラメーターが空であったり、設定ファイルで指定されていない場合に適用されます。

## Globalパラメータ

このセクションのパラメータは、設定ファイルの `[Global]` 見出しの下にあり、通常、ファイルの先頭にあります。

####**Indexer-UUID**
適用先			Indexer
デフォルト		[randomly generated if not set]
例:			`Indexer-UUID="ecdeeff8-8382-48f1-a24f-79f83af00e97"`
説明:		特定のインデクサに一意な識別子を設定します。ふたつのインデクサが同じ UUID を持つことはありません。このパラメータを設定しないと、インデクサはUUIDを生成してそれを設定ファイルに書き込みます。 これは、インデクサが壊滅的に失敗してレプリケーションから再構築される場合を除いて、通常は最良の選択です ([レプリケーションドキュメント](configuration/replication.md) を参照ください)。このパラメータを変更する前に、よく考えてください。

####**Webserver-UUID**
適用先:			Webserver
デフォルト値		[randomly generated if not set]
例			`Webserver-UUID="ecdeeff8-8382-48f1-a24f-79f83af00e97"`
説明		Sets a unique identifier for a particular webserver. No two webserver should have the same UUID. If this parameter is not set, the webserver will generate a UUID and *write it back into the config file*; this is usually the best choice. Think twice before modifying this parameter.

####**License-Location**
適用先:        Indexer and Webserver
デフォルト値:        `/opt/gravwell/etc/license`
例:        `License-Location=/opt/gravwell/etc/my_license`
説明:        datalaiqライセンスファイルのパスを設定します。パスはgravwellユーザーとグループによって読み取り可能である必要があります。

####**Config-Location**
適用先:        Indexer and Webserver
デフォルト値:        `/opt/gravwell/etc`
例:        `Config-Location=/tmp/path/to/etc`
説明:        config-locationでは、他のすべての構成パラメータを収容するための代替場所を指定することができます。 代替のConfig-Locationを指定することで、他のすべてのパラメータを代替パスで指定する必要なく、1つのパラメータを設定することができます。

####**Web-Port**
適用先:        Webserver
デフォルト値:        `443`
例:        `Web-Port=80`
説明:        Webサーバーのリスニングポートを指定します。Web-Port パラメータを 80 に設定しても、Web サーバは HTTP 専用モードに切り替わらないので、Insecure-Disable-HTTPS の設定が必要です。

####**Disable-HTTP-Redirector**
適用先:        Webserver
デフォルト値:        `False`
例:        `Disable-HTTP-Redirector=true`
説明:        デフォルトでは、DatalaiQは、平文のHTTPポータルを要求するクライアントを暗号化されたHTTPSポータルにリダイレクトするHTTPリダイレクタを開始します。

####**Insecure-Disable-HTTPS**
適用先:		Webserver
デフォルト値:	`false`
例:		`Insecure-Disable-HTTPS=true`
説明:	デフォルトでは、DatalaiQ は HTTPS モードで動作します。Insecure-Disable-HTTPS=true`を設定すると、DatalaiQは代わりにプレーンテキストのHTTPを使用し、`Web-Port`で待ち受けるように指示します。

####**Webserver-Domain**
適用先:		Webserver
デフォルト値:	0
例:		`Webserver-Domain=17`
説明: Webserver-Domain` パラメータは、Webサーバー上の [resources](#!resources/resources.md) ドメインを制御します。2 つのウェブサーバーが同じドメインで構成され、異なるリソースセットを持ち、同じインデクサに接続され、データストアを介して同期されていない場合、インデクサは 2 つのリソースセットの間でスラッシュを発生させます。Webサーバーを異なるドメインに置くと、リソースの競合なしに、 両者が同じインデックスを使用することができます。

####**Control-Listen-Address**
適用先:        Indexer
デフォルト値:
例:        `Control-Listen-Address=”10.0.0.1”`
説明:        Control-Listen-Address "パラメーターは、インデクサーのコントロールリスナーを 特定のアドレスにバインドすることができます。 デュアルホームマシン、または高速データネットワークと低速コントロールネットワークを持つマシンにDatalaiQをインストールすると、トラフィックが適切にルーティングされるようにするために、インデクサを特定のアドレスにバインドすることが望ましい場合があります。

####**Control-Port**
適用先:        Indexer and Webserver
デフォルト値:        `9404`
例:        `Control-Port=12345`
説明:        Control-Port」パラメータは、ウェブサーバからの制御コマンドをインデクサがリッスンするためのポートを選択します。この設定は、 インデクサとウェブサーバが通信するために同じである必要があります。インストーラはデフォルトでインデクサにバインド機能を設定しないため、ポートを1024より大きい値に設定する必要があります。 1 台のマシンで複数のインデクサを実行している環境、 あるいは別のアプリケーションがポート 9404 にバインドしている環境では、 制御ポートの調整が必要になることがあります。

####**Datastore-Listen-Address**
適用先:			Datastore
デフォルト値:
例:			`Datastore-Listen-Address=10.0.0.1`
説明:		Datastore-Listen-Addressパラメータは、データストアが特定のアドレスのみをリッスンするように指示します。デフォルトでは、データストアはシステム上のすべてのアドレスをリッスンします。

####**Datastore-Port**
適用先:			Datastore
デフォルト値:		`9405`
例:			`Datastore-Port=7888`
説明:		Datastore-Portパラメータは、データストアが通信するためのポートを選択します。ポートは1024以上である必要があります。デフォルトの9405は、ほとんどのインストールに適しています。

####**Datastore**
適用先:			Webserver
デフォルト値:
例:			`Datastore=10.0.0.1:9405`
説明:		データストアパラメータは、ウェブサーバがデータストアに接続して、ダッシュボード、リソース、ユーザープリファレンス、検索履歴を同期させることを指定します。これにより、[distributed webservers](distributed/frontend.md) が可能になりますが、必要な場合のみ設定するようにしてください。デフォルトでは、ウェブサーバはデータストアに接続しません。

####**Datastore-Update-Interval**
適用先:			Webserver
デフォルト値:		`10`
例:			`Datastore-Update-Interval=5`
説明:		Datastore-Update-Intervalパラメータは、データストアの更新を確認するまでのWebサーバーの待機時間（秒）を決定します。一般的には、デフォルトの10秒が適しています。

####**Datastore-Insecure-Disable-TLS**
適用先:		Webserver and Datastore
デフォルト値:	`false`
例:		`Datastore-Insecure-Disable-TLS=true`
説明:	Datastore-Insecure-Disable-TLSパラメータは、Webサーバーとデータストアの両方で使用されます。デフォルトでは、データストアはWebサーバからのHTTPS接続をリッスンします。このパラメータをtrueに設定すると、データストアは平文のHTTPを予期し、WebサーバにHTTPを使用するように指示します。

####**Datastore-Insecure-Skip-TLS-Verify**
適用先:		Webserver
デフォルト値:	`false`
例:		`Datastore-Insecure-Skip-TLS-Verify=true`
説明:	Datastore-Insecure-Skip-TLS-Verifyパラメータは、データストアに接続するときに無効なTLS証明書を無視するようにWebサーバーに指示します。これは、自己署名証明書を使用する場合に必要ですが、可能であれば避けるべきです。

####**External-Addr**
適用先:		Webserver
デフォルト値:
例:		`External-Addr=10.0.0.1:443`
説明:	External-Addr パラメーターは、他の Web サーバーがこの Web サー バーに連絡するために使用するアドレスを指定します。データストアを使用する場合、このパラメータは **必須** です。これは、あるWebサーバーのユーザーが別のWebサーバーで実行した検索結果を読み込むことができるようにするためです。

####**Search-Forwarding-Insecure-Skip-TLS-Verify**
適用先:		Webserver
デフォルト値:	`false`
例:		`Search-Forwarding-Insecure-Skip-TLS-Verify=true`
説明:	このパラメータは、データストアを使用して複数のWebサーバーを分散モードで操作する場合にのみ有効です。ウェブサーバーに自己署名証明書がある場合、このパラメータをtrueに設定しない限り、ユーザーはリモートウェブサーバーから検索にアクセスすることができません。

####**Ingest-Port**
適用先:        Indexer
デフォルト値:        `4023`
例:        `Ingest-Port=14023`
説明:        Ingest-Port パラメータは、 インジェスター接続のためにインデクサがリッスンするポートを制御します。 Ingest-Port パラメータを変更すると、 ひとつのマシンで複数のインデクサを実行しているときや、 別のアプリケーションがすでにデフォルトのポート 4023 にバインドされているときに便利です。

####**TLS-Ingest-Port**
適用先:        Indexer
デフォルト値:        `4024`
例:        `TLS-Ingest-Port=14024`
説明:        TLS-Ingest-Port パラメータは、 インジェスター接続のためにインデクサがリッスンするポートを制御します。 TLS-Ingest-Port パラメータを変更すると、 ひとつのマシンで複数のインデクサを実行しているときや、 別のアプリケーションがすでにデフォルトのポート 4024 にバインドされているときに便利です。 デフォルトでは、TLSトランスポートを使用するすべてのインジェスターは、リモート証明書を検証します。 展開が自動生成された証明書を使用している場合、インジェスターは証明書を信頼できるものとしてインストールする必要があるか、証明書の検証を無効にする必要があります（これは、TLSトランスポートによって提供される保護を効果的に破壊します）。

####**Pipe-Ingest-Path**
適用先:			Indexer
デフォルト値:		`/opt/gravwell/comms/pipe`
例:			`Pipe-Ingest-Path=/tmp/path/to/pipe`
説明:		Pipe-Ingest-Path は、Unix の名前付きパイプへのフルパスを指定します。 インデクサは名前付きパイプを作成し、同居インジェスタはそのパイプに接続して、非常に高速で低レイテンシーのトランスポートとして使用することができます。 名前付きパイプは、1ギガビット以上で動作するネットワークパケットインジェスターのような、非常に高い性能を必要とするインジェスターに最適です。 また、名前付きパイプは、通常とは異なるネットワークトランスポートや非IPベースの超高速相互接続でのトランスポートを容易にするために使用することができます。

####**Log-Location**
適用先:        Indexer
デフォルト値:        `/opt/gravwell/log`
例:        `Log-Location=/tmp/path/to/logs`
説明:        Log-Locationパラメータは、DatalaiQインフラストラクチャが自身のログを配置する場所を制御します。 DatalaiQは、自身のログを直接インデクサーにフィードせず、ファイルに書き込みます（DatalaiQのログも取り込みたい場合は、ファイル・フォロワー・インジェスターを使用します）。 このパラメータは、これらのログがどこに行くかを指定します。

####**Web-Log-Location**
適用先:        Webserver
デフォルト値:        `/opt/gravwell/log/web`
例:        `Web-Log-Location=/tmp/path/to/logs/web`
説明:        Web-Log-Locationパラメータは、Webサーバーのログがどこに保存されるかを制御します。 DatalaiQは、自身のログを直接インデクサーにフィードせず、ファイルに書き込みます（DatalaiQのログも取り込みたい場合は、ファイルフォロワーインジェスターを使用します）。 このパラメータは、これらのログがどこへ行くかを指定します。

####**Datastore-Log-Location**
適用先:		Datastore
デフォルト値:	`/opt/gravwell/log/datastore`
例:		`Datastore-Log-Location=/tmp/path/to/logs/datastore`
説明:	Datastore-Log-Locationパラメータは、データストアのログが保存される場所を制御します。

####**Log-Level**
適用先:        Indexer, Datastore, and Webserver
デフォルト値:        `INFO`
例:        `Log-Level=ERROR`
説明:        Log-Level パラメータは、datalaiq インフラストラクチャからのログの冗長性を制御します。 Log-Level には3つの利用可能な引数があります。INFO、WARN、およびERRORです。 INFOは最も饒舌で、ERRORは最も饒舌ではありません。 ログ記録システムは、ログ記録レベルごとにファイルを生成し、syslogデーモンと同様の方法でそれらをローテーションします。

####**Disable-Access-Log**
適用先:        Webserver
デフォルト値:        `false`
例:        `Disable-Access-Log=true`
説明:        Disable-Access-Log パラメータは、Webサーバーによって生成されるアクセスログを無効にするために使用されます。 アクセスログのインフラストラクチャは、個々のページへのアクセスを記録します。通常、これらのアクセスログはDatalaiQへのアクセスを監査し、潜在的な問題をデバッグするために重要ですが、多くのユーザーがいる環境ではアクセスログが大きくなるため、無効にするのが望ましい場合があります。

####**Disable-Self-Ingest**
適用先:			Webserver, Indexer
デフォルト値:		false
例:			`Disable-Self-Ingest=true`
説明:		Disable-Self-Ingest パラメータは、ウェブサーバとインデクサーのログを `gravwell` タグに取り込まないようにします。

####**Persist-Web-Logins**
適用先:        Webserver
デフォルト値:        `true`
例:        `Persist-Web-Logins=false`
説明:        Persist-Web-Logins パラメーターは、シャットダウン時にユーザー セッションを不揮発性ストレージに保存するよう Web サーバーに通知するために使用されます。 デフォルトでは、Web サーバーがシャットダウンまたは再起動されると、クライアントセッションが保存されます。 Persist-Web-Logins を false に設定すると、 Web サーバーが再起動するたびにセッションが無効になります。

####**Session-Timeout-Minutes**
適用先:        Webserver
デフォルト値:        `60`
例:        `Session-Timeout-Minutes=1440`
説明:        Session-Timeout-Minutesパラメータは、Webサーバーがセッションを破壊する前に、クライアントがアイドル状態である時間を制御します。 たとえば、クライアントがログアウトせずにブラウザを閉じた場合、システムは指定された時間だけ待機してからセッションを無効化します。 デフォルト値は60分ですが、ほとんどのインストーラは、この設定値をデフォルトで1日に設定しています。

####**Key-File**
適用先:        Indexer, Datastore, and Webserver
デフォルト値:        `/opt/gravwell/etc/key.pem`
例:        `Key-File=/opt/gravwell/etc/privkey.pem`
説明:        Key-Fileパラメータは、Webサーバー、データストア、およびインデクサーの秘密鍵として使用するファイルを制御します。 秘密鍵と公開鍵は、PEM 形式でエンコードする必要があります。 秘密鍵は保護する必要があり、危なくなったら破棄して再発行する必要があります。 詳細については、http://www.tldp.org/HOWTO/SSL-Certificates-HOWTO/x64.html を参照してください。

####**Certificate-File**
適用先:        Indexer, Datastore, and Webserver
デフォルト値:        `/opt/gravwell/etc/cert.pem`
例:        `Certificate-File=/opt/gravwell/etc/cert.pem`
説明:        Certificate-Fileパラメータは、TLSトランスポートに使用される公開鍵/秘密鍵ペアの公開鍵コンポーネントを指定する。 この公開鍵は、すべてのインジェスターとウェブ・クライアントに配信されるため、機密事項とはみなされない。 DatalaiQは、公開鍵がPEM形式でエンコードされ、公開鍵部分のみを含んでいることを期待しています。

####**Ingest-Auth**
適用先:        Indexer
デフォルト値:        `IngestSecrets`
例:        `Ingest-Auth=abcdefghijklmnopqrstuvwxyzABCD`
説明:        Ingest-Auth パラメータは、 インジェスターとインデクサーを認証する際に使用する共有秘密トークンを指定します。 このトークンは任意の長さにすることができますが、 DatalaiQ では最低でも 24 文字の高エントロピー・トークンを推奨しています。 デフォルトでは、インストーラはランダムなトークンを生成します。

####**Control-Auth**
適用先:        Indexer and Webserver
デフォルト値:        `ControlSecrets`
例:        `Control-Auth=abcdefghijklmnopqrstuvwxyzABCD`
説明:        Control-Authパラメータは、インジェスターとウェブサーバーの認証に使用される共有秘密トークンを指定します。 このトークンは任意の長さを指定できますが、DatalaiQでは少なくとも24文字の高エントロピー・トークンを推奨しています。 デフォルトでは、インストーラはランダムなトークンを生成します。

####**Search-Agent-Auth**
適用先:		Webserver
デフォルト値:	
例:		`Search-Agent-Auth=abcdefghijklmnopqrstuvwxyzABCD`
説明:	Search-Agent-Authパラメータは、検索エージェントをWebサーバーに認証するために使用される共有秘密トークンを指定します。インストーラのデフォルトでは、ランダムな検索エージェントトークンが生成されます。

####**Web-Files-Path**
適用先:        Webserver
デフォルト値:        `/opt/gravwell/www`
例:        `Web-Files-Path=/tmp/path/to/www`
説明:        Web-Files-Path は、ウェブサーバーによって提供されるフロントエンド GUI ファイルを含むパスを指定します。 Web ファイルには、Web ページを表示し、Web ブラウザを介して DatalaiQ システムと対話するためのすべての DatalaiQ コードが含まれています。

####**Tag-DB-Path**
適用先:		Indexer
デフォルト値:	`/opt/gravwell/etc/tags.db`
例:		`Tag-DB-Path=/tmp/path/to/tags.db`
説明:	Tag-DB-Pathパラメータは、タグデータベースの場所を指定します。このファイルは、インデクサーの数値タグIDをタグ名文字列にマッピングします。

####**User-DB-Path**
適用先:        Webserver
デフォルト値:        `/opt/gravwell/etc/users.db`
例:        `User-DB-Path=/tmp/path/to/users.db`
説明:        User-DB-Pathパラメータは、ユーザーデータベースファイルの場所を指定します。 ユーザー データベース ファイルには、ユーザーとグループの構成が含まれます。 ユーザーデータベースは、パスワードの保存と検証にbcryptハッシュアルゴリズムを使用しており、非常に堅牢であると考えられていますが、users.dbファイルは依然として保護されている必要があります。 デフォルトでは、インストーラーはユーザー・データベース・ファイルのファイルシステム・パーミッションを、DatalaiQユーザーとグループによってのみ読み取り可能になるように設定します。

####**Datastore-User-DB-Path**
適用先:		Datastore
デフォルト値:	`/opt/gravwell/etc/datastore-users.db`
例:		`Datastore-User-DB-Path=/tmp/path/to/datastore-users.db`
説明:	Datastore-User-DB-Pathパラメータは、データストア・コンポーネントによって管理されるユーザー・データベース・ファイルの場所を指定します。これは、User-DB-Pathパラメータで指定されたパスと同じであってはなりません。

####**Web-Store-Path**
適用先:        Webserver
デフォルト値:        `/opt/gravwell/etc/webstore.db`
例:        `Web-Store-Path=/tmp/path/to/webstore.db`
説明:        Web-Store-Path は、検索履歴、ダッシュボード、ユーザー設定、ユーザーセッション、およびその他の雑多なユーザーデータを保存するために使用されるデータベースファイルを指します。 ウェブストアデータベースファイルにはユーザーの認証情報は含まれませんが、ユーザーセッションCookieとCSRFトークンは含まれます。 DatalaiQはクッキーとCSRFトークンをオリジンに関連付けるので、攻撃者が盗んだクッキーやトークンを再利用するリスクは低いものの、データストアは保護されるべきです。 インストーラーは、DatalaiQ ユーザーによる読み取り/書き込みのみを許可するよう、ファイルシステムのパーミッションを設定します。

####**Datastore-Web-Store-Path**
適用先:		Datastore
デフォルト値:	`/opt/gravwell/etc/datastore-webstore.db`
例:		`Datastore-Web-Store-Path=/tmp/path/to/datastore-webstore.db`
説明:	Datastore-Web-Store-Pathパラメータは、データストアが検索履歴、ダッシュボード、およびユーザ設定を保存するために使用するデータベース・ファイルを指します。これは、Web-Store-Pathパラメータで指定されたものと同じパスであってはなりません（**）。

####**Web-Listen-Address**
適用先:        Webserver
デフォルト値:
例:        `Web-Listen-Address=10.0.0.1`
説明:        Web-Listen-Address パラメーターは、Web サーバーがバインドし、サー ビスを提供するアドレスを指定します。 デフォルトでは、このパラメーターは空で、Web サーバーがすべてのインターフェイスとアドレスにバインドすることを意味します。

####**Remote-Indexers**
適用先:        Webserver
デフォルト値:        `net:10.0.0.1:9404`
例:        `Remote-Indexers=net:10.0.0.1:9404`
説明:        Remote-Indexersパラメータは、ウェブサーバが接続し、制御する リモートインデックスのアドレスとポートを指定します。 Remote-Indexers はリスト・パラメータで、 複数のリモート・インデックスを提供するために何度でも指定することができます。DatalaiQ Cluster 版では、クラスタ内の各インデクサを指定する必要があります。 net:" という接頭辞は、リモート・インデクサがネットワーク・トランスポートを介してアクセス可能であることを意味します。

####**Search-Scratch**
適用先:        Indexer and Webserver
デフォルト値:        `/opt/gravwell/scratch`
例:        `Search-Scratch=/tmp/path/to/scratch`
説明:        Search-Scratch パラメータは、アクティブな検索中に検索モジュールが一時ストレージとして使用できるストレージロケーションを指定します。 検索モジュールによっては、メモリの制約により一時的なストレージを使用する必要がある場合があります。 たとえば、ソートモジュールは 5GB のデータをソートする必要がありますが、物理マシンには 4GB の物理 RAM しかない可能性があります。 このモジュールは、ホストのスワップを呼び出すことなく、スクラッチ領域を賢く使って大きなデータセットをソートすることができます（これはソートだけでなく、すべてのモジュールにペナルティが課せられます）。 各検索が終了すると、スクラッチスペースは破棄されます。

####**Render-Store**
適用先:        Webserver
デフォルト値:        `/opt/gravwell/render`
例:        `Render-Store=/tmp/path/to/render`
説明:        Render-Storeパラメータは、レンダラー・モジュールが検索結果を保存する場所を指定します。 Render-Store は一時的に保存される場所であり、通常は適度に小さいデータセットを表します。 検索がアクティブに実行されているとき、または休止状態でクライアントとやり取りしているとき、Render-Store はレンダラーがそのデータセットを保存および取得する場所である。 Render-Store は、フラッシュベースや XPoint SSD などの高速ストレージに保存する必要があります。 検索が中止されると、Render-Storeは削除されます（検索が保存されていない場合）。

####**Saved-Store**
適用先:        Webserver
デフォルト値:        `/opt/gravwell/saved`
例:        `Saved-Store=/path/to/saved/searches`
説明:        Saved-Storeパラメータは、保存された検索結果をどこに保存するかを指定します。 保存された検索は、検索の出力状態を表し、監査や、ユーザーが検索を再開せずに検索結果を後で参照したい場合に役立ちます。 保存された検索は明示的に削除する必要があり、そのデータはシャードのエージアウトポリシーの対象にはなりません。 保存された検索は完全にアトミックです。つまり、保存された検索の基礎となるデータは完全にエージングされ、削除されても、ユーザーは保存された検索を再度開いて調べることができるのです。 保存された検索は共有することもできます。つまり、ユーザーは保存された検索をパックアップして、DatalaiQの他のインスタンスと共有することができるのです。

####**Search-Pipeline-Buffer-Size**
適用先:        Indexer and Webserver
デフォルト値:        `2`
例:        `Search-Pipeline-Buffer-Size=8`
説明:        Search-Pipeline-Buffer-Sizeは、検索中に各モジュール間で転送可能なブロック数を指定します。 サイズを大きくすると、バッファリングがうまくいき、常駐メモリ使用量を犠牲にして、より高いスループットの検索ができる可能性があります。 ウェブサーバは通常、パイプラインを通過する際にすべてのエントリを常駐させ、 モジュールを凝縮してメモリ負荷を減らします。 システムが回転ディスクのような遅延の大きいストレージシステムを使用している場合、このバッファーのサイズを大きくすることが有利になることがあります。
このパラメータを大きくすると、検索のパフォーマンスが向上しますが、システムが一度に処理できる検索数に直接影響します。 ビデオフレーム、PE 実行ファイル、オーディオファイルのような非常に大きなエントリを保存していることが分かっている場合、バッファサイズを小さくして常駐メモリ使用量を制限する必要があるかもしれません。ホストカーネルが Out Of Memory (OOM) を呼び出して DatalaiQ プロセスを強制終了するのを確認した場合、このノブを最初に回してください。

####**Search-Relay-Buffer-Size**
適用先:		Webserver
デフォルト値:	`4`
例:		`Search-Relay-Buffer-Size=8`
説明:	Search-Relay-Buffer-Size パラメータは、 別のインデクサからの未処理のブロックを待っている間に、 ウェブサーバが各インデクサから受け入れるエントリブロックの数を 制御します。検索エントリは時間的に流れてくるので、 あるインデクサが古いエントリをまだ処理している間に、 別のインデクサがより新しいエントリに移行していることもありえます。ウェブサーバはエントリを時間的な順序で処理しなければならないので、 「先行している」インデクサからのエントリをバッファリングして、 遅いインデクサが追いつくのを待つことになります。一般に、デフォルト値はメモリの問題を防ぐのに役立ち、かつ許容できるパフォーマンスを提供します。大量のメモリを搭載しているシステムでは、この値を増やすと便利でしょう。

####**Max-Search-History**
適用先:        Webserver
デフォルト値:        `100`
例:        `Max-Search-History=256`
説明:        Max-Search-Historyパラメータは、ユーザーの検索履歴を保持する数を制御します。 検索履歴は、過去にさかのぼって古い検索を調べたり、同じグループの他のユーザがどのような検索をしているかを確認するのに便利です。 履歴を大きくすると、古い検索文字列をより多く残すことができますが、履歴に残る検索文字数が多すぎると、GUIで操作するときに速度が低下することがあります。

####**Prebuff-Block-Hint**
適用先:        Indexer
デフォルト値:        `32`
例:        `Prebuff-Block-Hint=8`
説明:        Prebuff-Block-Hint は、データブロックを格納する際にインデクサが狙うべきソフトターゲットをメガバイト単位で指定します。 非常に高スループットのシステムではこの値をもう少し大きくしたいかもしれませんし、 メモリに制約のあるシステムではこの値を小さくしたいと思うかもしれません。 この値はソフトターゲットであり、インデクサは通常、インジェストが高レートで行われているときのみ、この値を使用します。

####**Prebuff-Max-Size**
適用先:        Indexer
デフォルト値:        `32`
例:        `Prebuff-Max-Size=128`
説明:        Prebuff-Max-Sizeパラメータは、エントリをディスクに強制する前にプリバッファが保持する最大データサイズをメガバイトで制御します。 プリバッファは、ソースクロックがあまり同期していない場合に、エントリの保存を最適化するために使用されます。 プリバッファが大きければ大きいほど、 インデクサは乱暴な値を提供するインジェスターをより最適化することができます。 各ウェルは独自のプリバッファを持つため、インストールに4つのウェルが定義され、Prebuff-Max-Sizeが256である場合、インデクサはデータを保持するために最大1GBのメモリを消費する可能性があります。 プリバッファは定期的にエントリを退避させ、常にストレージメディアにプッシュしているため、プリバッファの最大サイズは通常、高スループットシステムにおいてのみ使用されます。 これは、ホストシステムのOOMキラーがDatalaiQプロセスを終了させる場合、（Search-Pipeline-Buffer-Sizeの次に）回すべき2つ目のノブです。

####**Prebuff-Max-Set**
適用先:        Webserver
デフォルト値:        `256`
例:        `Prebuff-Max-Set=256`
説明:        Prebuff-Max-Setは、最適化のためにプリバッファに保持することを許可する1秒ブロックの数を指定します。 インジェスターから提供されるエントリーのタイムスタンプが同期していないほど、このセットを大きくする必要があります。 例えば、タイムスタンプが2時間程度変動するようなソースからデータを取得する場合は、この値を7200に設定する必要があります。 Prebuff-Max-Sizeコントロールはまだ有効で、プリバッファを強制的に退避させるので、この値を高く設定しても低く設定するよりも悪影響は少ないです。

####**Prebuff-Tick-Interval**
適用先:        Webserver
デフォルト値:        `3`
例:        `Prebuff-Tick-Interval=4`
説明:        Prebuff-Tick-Intervalパラメータは、プリバッファに位置するエントリを人工的に退避させる頻度を秒単位で指定します。 プリバッファは、アクティブなインジェストがあるときは常に値を永続ストレージに退避させますが、非常に低スループットのシステムでは、この値を使用して、エントリを永続ストレージに強制的にプッシュするようにすることができます。 DatalaiQは、可能な限りデータの消失を許しません。インデクサを緩やかにシャットダウンする際には、プリバッファによってすべてのエントリが永続ストレージに到達するようにします。 しかし、ホストの安定性にあまり信頼が置けない場合は、この間隔を2近くに設定して、システム障害や怒った管理者がインデクサーの下から敷物を引き抜くことができないようにするとよいでしょう。

####**Prebuff-Sort-On-Consume**
適用先:        Indexer
デフォルト値:        `false`
例:        `Prebuff-Sort-On-Consume=true`
説明:        Prebuff-Sort-On-Consume パラメータは、ディスクにプッシュする前にデータのロックをソートするようプリバッファに指示します。 ソート処理は個々のブロックにのみ適用され、パイプラインに入るときにデータがソートされていることを保証するものではありません。 ストレージに格納する前にブロックをソートすると、インジェスト時に大きなパフォーマンスのペナルティが発生します。 ほとんどすべてのインストールで、この値はfalseのままにしておくべきです。

####**Max-Block-Size**
適用先:        Indexer
デフォルト値:        `4`
例:        `Max-Block-Size=8`
説明:        Max-Block-Size はメガバイト単位で値を指定し、 インデクサがエントリをパイプラインにプッシュする際に生成できる最大ブロックサイズを伝えるヒントとして使用されます。 ブロックが大きくなると、パイプラインにかかる圧力は減りますが、メモリにかかる圧力は増えます。 大容量メモリや高スループットのシステムではこの値を大きくしてスループットを向上させ、小容量メモリシステムではこのサイズを小さくしてメモリへの負荷を軽減させることができます。 Prebuff-Block-HintとMax-Block-Sizeのパラメータが交差することで、インジェストとサーチのスループットを調整する2つのノブが提供されます。 DatalaiQでは、128GBのノードで、検索スループットは1GB/s、インジェストはMax-Block-Size 16で125万エントリ/秒、Prebuff-Block-Hint 8で達成されています。

####**Render-Store-Limit**
適用先:		Webserver
デフォルト値:	1024
例:		`Render-Store-Limit=512`
説明:	Render-Store-Limitパラメータは、検索レンダラーが保存できるメガバイト数を指定します。

####**Search-Control-Script**
適用先:		Webserver
デフォルト値:
例:		`Search-Control-Script=/opt/gravwell/etc/authscripts/limits.grv`
説明:	Search-Control-Script パラメータは、検索時に適用するスクリプトを指定するためのリストパラメータである。リストパラメータであるため、複数回指定して、複数のスクリプトを指定することができます。これらのスクリプトは、ユーザーが実行する検索に追加の制限を適用することができます。すべてのスクリプトは、検索ごとに実行されます。検索制御スクリプトの詳細については、DatalaiQにお問い合わせください。

####**Webserver-Resource-Store**
適用先:		Webserver
デフォルト値:	`/opt/gravwell/resources/webserver`
例:		`Webserver-Resource-Store=/tmp/path/to/resources/webserver`
説明:	Webserver-Resource-Storeパラメータは、Webサーバーがそのリソースを格納する場所を指定します。このディレクトリは他のプロセスで使用されていないことが必要で、 インデクサやデータストアのリソースの場所として指定することはできません。

####**Indexer-Resource-Store**
適用先:		Indexer
デフォルト値:	`/opt/gravwell/resources/indexer`
例:		`Indexer-Resource-Store=/tmp/path/to/resources/indexer`
説明:	Indexer-Resource-Store パラメータは、 インデクサがそのリソースを格納する場所を指定します。このディレクトリは他のプロセスで使われていないものでなければならず、 ウェブサーバやデータストアのリソースの場所として指定することはできません。

####**Datastore-Resource-Store**
適用先:		Datastore
デフォルト値:	`/opt/gravwell/resources/datastore`
例:		`Datastore-Resource-Store=/tmp/path/to/resources/datastore`
説明:	Datastore-Resource-Storeパラメータは、データストアがそのリソースを格納する場所を指定します。このディレクトリは他のプロセスで使用されていないことが必要で、 インデクサやウェブサーバのリソースの場所として指定することはできません。

####**Resource-Max-Size**
適用先:		Webserver, Datastore, and Indexer
デフォルト値:	`134217728`
例:		`Resource-Max-Size=1000000000`
説明:	Resource-Max-Sizeパラメータは、リソースの最大サイズをバイト単位で指定します。

####**Docker-Secrets**
適用先:		Webserver, Datastore, and Indexer
デフォルト値:	`false`
例:		`Docker-Secrets=true`
説明:	Docker-Secretsパラメータは、DatalaiQが[Docker secrets](https://docs.docker.com/engine/swarm/secrets/)からインジェスト、コントロール、サーチエージェントのシークレットを読み込もうとすることを指示します。DatalaiQはシークレットをそれぞれ `ingest_secret`, `control_secret`, `search_agent_secret` という名前にし、VM内の `/run/secrets/` ディレクトリからアクセスできるようにすることを想定しています。

####**HTTP-Proxy**
適用先:		Webserver
デフォルト値:
例:		`HTTP-Proxy=wwwproxy.example.com:8080`
説明:	HTTP-Proxyパラメータは、WebサーバーによるHTTPおよびHTTPSリクエストに使用されるプロキシを設定します。これは事実上、環境変数 $http_proxy を設定するのと同じで、同じ構文で設定することができます。 指定したプロキシの値は `HTTP` と `HTTPS` の両方のリクエストで使用されます。

####**Webserver-Ingest-Groups**
適用先:		Webserver
デフォルト値:
例:		`Webserver-Ingest-Groups=ingestUsers`
説明:	Webserver-Ingest-Groupsパラメータは、DatalaiQ Web API経由で直接エントリーを取り込むことを許可されたユーザーを持つグループを指定するリストパラメータである。リストパラメータとして、これは複数回指定することができ、複数のグループがWeb API経由で取り込むことを可能にします。

####**Disable-Update-Notification**
適用先:		Webserver
デフォルト値:	`false`
例:		`Disable-Update-Notification=false`
説明:	Disable-Update-Notification を true に設定すると、DatalaiQ の新バージョンが利用可能になったときに Web UI で通知を表示しません。

####**Disable-Feature-Popups**
適用先:		Webserver
デフォルト値:	`false`
例:		`Disable-Feature-Popups=true`
説明:	Disable-Feature-Popups を true に設定すると、DatalaiQ UI は新機能が追加されたときにポップアップ通知を表示しないようになります。

####**Disable-Stats-Report**
適用先: Webserver
デフォルト値: false
例: `Disable-Stats-Report=true`
説明:	このパラメータを true に設定すると、ウェブサーバの [メトリクス報告ルーチン] (#!metrics.md) に、ライセンスに関する最小限の情報のみを送信し、広範なシステム統計は省略するように指示します。

####**Temp-Dir**
適用先:		Webserver
デフォルト値:	`/opt/gravwell/tmp`
例:		`Temp-Dir=/tmp/gravtmp`
説明:	Temp-Dirパラメータは、他のプロセスからの干渉を受けずに一時的なDatalaiQファイル用に使用することができるディレクトリを指定します。インストール前にアップロードしたキットを保存するなどの用途に使用されます。

####**Ingest-Throttle-Threshold**
適用先:		Indexer
デフォルト値:	`90`
例: `Ingest-Throttle-Threshold=75`

Ingest-Throttle-Threshold パラメータは、インデクサがデータ取り込みを制限し始めるべき使用ディスク容量のパーセンテージを指定します。データエージアウトパラメータが設定されている場合、インデクサはエージアウト規則に従って削除可能なデータがあるかどうか強制的にチェックします。エージアウトパラメーターに関係なく、ディスクの空き容量が閾値を下回るまで、データの取り込みを停止します。

####**Insecure-User-Unsigned-Kits-Allowed**
適用先:		Webserver
デフォルト値:	`false`
例:		`Insecure-User-Unsigned-Kits-Allowed=true`
説明:	このパラメータを設定すると、すべてのユーザーが署名されていないキットをインストールできるようになります。このオプションを有効にしないことを強くお勧めします。

####**Disable-Search-Agent-Notifications**
適用先:		Webserver
デフォルト値:	`false`
例:		`Disable-Search-Agent-Notifications=true`
説明:	このパラメータをtrueに設定すると、検索エージェントがチェックインに失敗した場合に、Web UIに通知が表示されなくなります。これは、検索エージェントを無効にしていて、通知を表示したくない場合に便利です。

####**Indexer-Storage-Notification-Threshold**
適用先:		Indexer
デフォルト値:		`90`
例:		`Indexer-Storage-Notification-Threshold=98`
説明:		ストレージの使用について警告するタイミングを決定するパーセンテージ値です。 この値が0以上の場合、Indexerが使用するストレージデバイスが指定されたストレージ使用率より多く使用されると、通知が投げられます。 この値は 0 から 99 の間でなければなりません (MUST)。

####**Disable-Network-Script-Functions**
適用先:		Webserver
デフォルト値:	`false`
例:		`Disable-Network-Script-Functions=true`
説明:	デフォルトでは、パイプライン内の anko スクリプトは、net/http ライブラリや ssh/sftp ユーティリティなどのネットワーク機能の使用を許可されています。これを 'true' に設定すると、これらの機能が無効になります。

####**Webserver-Enable-Frame-Embedding**
適用先:		Webserver
デフォルト値:	`false`
例:		`Webserver-Enable-Frame-Embedding=true`
説明:	デフォルトでは、WebサーバーはX-Frame-Options: denyヘッダーを設定することにより、DatalaiQページがフレーム内に表示されることを禁止しています。この設定パラメータを「true」に設定すると、このヘッダーが削除され、ページをフレーム内に埋め込むことができるようになります。

####**Webserver-Content-Security-Policy**
適用先:		Webserver
デフォルト値:	``
例:		`Webserver-Content-Security-Policy="default-src https:"`
説明:	このパラメータを使用すると、管理者はすべての DatalaiQ ページで送信される Content-Security-Policy ヘッダを定義することができます。これは重要なセキュリティ・オプションであり、https のみを必要とするなどの展開要件に基づいて組織用に設定する必要があります。

####**Default-Language**
適用先:		Webserver
デフォルト値:		`en-US`
例:		`Default-Language=en-US`
説明:		Default-Languageパラメータを設定すると、/api/languageにある認証されていないAPIで提供されるものを制御し、GUIが複数の言語がある環境でどの言語をデフォルトとすべきかを決定するために使用されます。これは、ユーザが言語を選択しておらず、ブラウザが `window.navigator.language` を介して優先言語を提供していない場合に使用されます。

####**Disable-Map-Tile-Server-Proxy**
適用先:		Webserver
デフォルト値:	`false`
例:		`Disable-Map-Tile-Server-Proxy=true`
説明:	このパラメータは、DatalaiQ の組み込みマップ プロキシを制御します。地図サーバーに過度の負担をかけないようにするため、DatalaiQウェブサーバーは地図タイルをキャッシュしています。しかし、プロキシを使用すると、実際のマップサーバーに送られる要求は、ユーザーのウェブブラウザではなくDatalaiQウェブサーバーから発生します。DatalaiQがロックされたネットワークにインストールされている場合、発信するHTTPが無効になっているため失敗する可能性があります。 `Disable-Map-Tile-Server-Proxy` を true に設定すると、ビルトインプロキシを無効にしてGUIが直接マップリクエストを行うようになります。プロキシが無効で、かつ `Map-Tile-Server` パラメータが設定されている場合、GUIはそのサーバにリクエストを行います。

####**Map-Tile-Server**
適用先:		Webserver
デフォルト値:	``
例:		`Map-Tile-Server=https://maps.example.com/osm/`
説明:	Map-Tile-Serverパラメータは、管理者がマップ・タイルの異なるソースを定義することを可能にします。デフォルトでは、DatalaiQはDatalaiQマップ・サーバーからタイルをフェッチし、必要に応じてOpenStreetMapサーバーにフォールバックします。このパラメータを設定すると、DatalaiQは指定されたサーバーのみを使用するようになります。指定するURLは、[ここ](https://wiki.openstreetmap.org/wiki/Tile_servers)で定義されている標準的なOpenStreetMapタイルサーバーフォーマットのプレフィックスで、z/x/y座標パラメータは省かれているはずです。例えば、タイルが `https://maps.wikimedia.org/osm-intl/${z}/${x}/${y}.png` でアクセスできる場合、例えば https://maps.wikimedia.org/osm-intl/0/1/2.png では `Map-Tile-Server=https://maps.wikimedia.org/osm-intl/` を設定することができます。

####**Gravwell-Tile-Server-Cooldown-Minutes**
適用先:		Webserver
デフォルト値:	5
例:		`Gravwell-Tile-Server-Cooldown-Minutes=1`
説明:	DatalaiQタイルプロキシが通常モードで動作している場合（無効になっていない、`Map-Tile-Server`パラメータが設定されていない）、Gravwellが動作するサーバからマップタイルを取得しようとします。そのサーバへのリクエスト送信が失敗した場合、プロキシはクールダウンの間、代わりに openstreetmap.org のサーバにフォールバックします。このパラメータを0に設定すると、クールダウンは無効になります。

####**Gravwell-Tile-Server-Cache-MB**
適用先:		Webserver
デフォルト値:	4
例:		`Gravwell-Tile-Server-Cache-MB=32`
説明:	DatalaiQタイルプロキシは、最近アクセスされたタイルのキャッシュを維持し、マップレンダリングを高速化します。このパラメータは、キャッシュが使用するストレージのメガバイト数を制御します。

####**Gravwell-Tile-Server-Cache-Timeout-Days**
適用先:		Webserver
デフォルト値:	7
例:		`Gravwell-Tile-Server-Cache-Timeout-Days=2`
説明:	DatalaiQタイルプロキシは、最近アクセスされたタイルのキャッシュを維持し、マップレンダリングを高速化します。このパラメータは、キャッシュされたタイルが有効とみなされる最大日数を制御します。この日数が経過すると、タイルはパージされてアップストリームサーバから再取得されます。

####**Disable-Single-Indexer-Optimization**
適用先:		Webserver
デフォルト値:	false
例:		`Disable-Single-Indexer-Optimization=true`
説明:	DatalaiQが単一のインデクサで使用される場合、インデクサからウェブサーバへのデータ転送量を減らすために、デフォルトですべてのモジュール（レンダーモジュールを除く）を*インデクサ*上で実行します。このオプションはその最適化を無効にします。DatalaiQサポートからの指示がない限り、このオプションを`false`に設定しておくことを強くお勧めします。

####**Library-Dir**
適用先:		Webserver
デフォルト値:	`/opt/gravwell/libs`
例:		`Library-Dir=/scratch/libs`
説明:	スケジュールされたスクリプトは `include` 関数を使用して追加のライブラリをインポートすることができます。これらのライブラリは外部のリポジトリから取得され、ローカルにキャッシュされます。この設定オプションは、キャッシュされたライブラリが格納されるディレクトリを設定します。

####**Library-Repository**
適用先:		Webserver
デフォルト値:	`https://github.com/gravwell/libs`
例:		`Library-Repository=https://github.com/example/gravwell-libs`
説明:	スケジュールされたスクリプトは `include` 関数を使用して追加のライブラリをインポートすることができます。これらのライブラリは、このパラメータで指定されたリポジトリにあるファイルから読み込まれます。デフォルトでは、便利なライブラリのGravwell-maintained リポジトリを指します。もし、独自のライブラリセットを提供したい場合は、このパラメータを、あなたが管理するgitリポジトリを指すように設定してください。

####**Library-Commit**
適用先:		Webserver
デフォルト値:
例:		`Library-Commit=19b13a3a8eb877259a06760e1ee35fae2669db73`
説明:	スケジュールされたスクリプトは `include` 関数を使用して追加のライブラリをインポートすることができます。これらのライブラリは `Library-Repository` オプションで指定されたリポジトリにあるファイルから読み込まれます。デフォルトでは、DatalaiQは最新バージョンを使用します。git commit 文字列が指定された場合、DatalaiQ は指定されたバージョンのリポジトリを代わりに使用しようとします。

####**Disable-Library-Repository**
適用先:		Webserver
デフォルト値:	false
例:		`Disable-Library-Repository=true`
説明:	スケジュールされたスクリプトは `include` 関数を使用して追加のライブラリをインポートすることができます。Disable-Library-Repository` を true に設定すると、この機能を無効にすることができます。

####**Gravwell-Kit-Server**
適用先:	Webserver
デフォルト値:	https://kits.gravwell.io/kits
例:	`Gravwell-Kit-Server=http://internal.mycompany.io/gravwell/kits`
説明:	Gravwell キットサーバーのホストを上書きすることができます。これは DatalaiQ キットサーバーのミラーをホストしている Airgapped やセグメント化された展開で役に立ちます。 この値を空文字列に設定すると、リモートキットサーバへのアクセスが完全に無効になります。
例:
```
Gravwell-Kit-Server="" #disable remote access to gravwell kitserver
Gravwell-Kit-Server="http://gravwell.mycompany.com/kits" #override to use internal mirror
```

####**Kit-Verification-Key**
適用先: Webserver
デフォルト値:
例: `Kit-Verification-Key=/opt/gravwell/etc/kits-pub.pem`
説明:	キットサーバーからのキットを検証する際に使用する公開鍵を含むファイルを指定します。別の Gravwell-Kit-Server を指定した場合はこの値を設定します。DatalaiQ の公式キットサーバーを使用する場合はこの値は必要ありません。キットの署名に適した鍵は、[gencert](https://github.com/gravwell/gencert) ユーティリティで生成できます。

####**Disable-User-Ingester-Config-Reporting**
適用先: Webserver
デフォルト値: false
例: `Disable-User-Ingester-Config-Reporting=true`
説明:	一般ユーザー(非管理者)がインジェスト状態の更新のConfigurationフィールドを受信しないようにウェブサーバーに指示します。設定にはインジェストシークレットやその他の「機密」項目は含まれませんが、設定全体を一般ユーザーから秘密にしておきたい場合もあるでしょう; このオプションはそれを行います。

####**Disable-Ingester-Config-Reporting**
適用先: Webserver
デフォルト値: false
例: `Disable-Ingester-Config-Reporting=true`
説明:	Webサーバに、インジェスト状態の更新のConfigurationフィールドを受け取るべきユーザがいないことを伝えます。設定にはインジェストシークレットやその他の「機密」項目は含まれませんが、設定全体をすべてのユーザーから秘密にしておきたい場合があるでしょう；このオプションはそれを行います。

####**Disable-Indexer-Overload-Warning**
適用先:	Indexer
デフォルト値:	false
例:	`Disable-Indexer-Overload-Warning=true`
説明: このパラメータを設定すると、インデクサは自分自身が「過負荷」であると判断した場合、通知を送信しなくなります。

## パスワード操作

[Password-Control]`設定セクションは、ユーザーの作成やパスワードの変更時にパスワードの複雑さの規則を強制するために使用することができます。このブロックで設定されたオプションは、ウェブサーバーにのみ適用されます。シングルサインオンを使用する場合は、これらの複雑さの設定ルールは適用されません。

備考: Password-Control`セクションは複数回宣言してはいけません。

```
[Password-Control]
	Min-Length=8
	Require-Uppercase=true
	Require-Lowercase=true
	Require-Special=true
	Require-Special=true
```

####**MinLength**
デフォルト値:	0
例:		`MinLength=8`
説明:	`MinLength` は、パスワードの最小の文字数を指定する。

####**Require-Uppercase**
デフォルト値:	false
例:		`Require-Uppercase=true`
説明:	`Require-Uppercase` が設定されている場合、パスワードには少なくとも1文字の大文字が含まれていなければなりません。

####**Require-Lowercase**
デフォルト値:	false
例:		`Require-Lowercase=true`
説明:	`Require-Lowercase` が設定されている場合、パスワードは少なくとも1つの小文字を含まなければなりません。

####**Require-Number**
デフォルト値:	false
例:		`Require-Number=true`
説明:	`Require-Number`が設定されている場合、パスワードには少なくとも1桁の数字が含まれていなければなりません。

####**Require-Special**
デフォルト値:	false
例:		`Require-Special=true`
説明:	`Require-Special` が設定されている場合、パスワードは少なくとも1つの "特殊" 文字を含んでいなければなりません。特殊文字のセットには、数字や文字ではないすべてのユニコード文字が含まれます。

## ウェル設定

このセクションのパラメータは、 `Default-Well` 仕様を含む、ウェルの仕様に適用されます。ウェル構成は *インデックス* にのみ適用され、ウェブサーバーでは無視されます。以下は、`Default-Well`と "pcap "という名前の2つのウェル設定のサンプルである。

```
[Default-Well]
	Location=/opt/gravwell/storage/default/
	Cold-Location=/opt/gravwell/cold-storage/default
	Accelerator-Name=fulltext
	Accelerator-Engine-Override=bloom
	Max-Hot-Storage-GB=20

[Storage-Well "pcap"]
	Location=/opt/gravwell/storage/pcap
	Cold-Location=/opt/gravwell/cold-storage/pcap
	Hot-Duration=1D
	Cold-Duration=12W
	Delete-Frozen-Data=true
	Max-Hot-Storage-GB=20
	Disable-Compression=true
	Tags=pcap
```

コンフィギュレーションファイルには、正確に1つの `Default-Well` セクションを含める必要があります。また、1つ以上の `Storage-Well` セクションを含むことができます。

ウェルがホット、コールド、アーカイブの各ストレージ間でエントリーを移動する方法については、 [エージアウトドキュメント](ageout.md) を参照してください。

備考: デフォルトのウェルには、他のウェルに含まれないすべてのタグが含まれます。

####**Location**
デフォルト値:	`/opt/gravwell/storage/default` for `Default-Well`, none for `Storage-Well`
例:		`Location=/opt/gravwell/storage/foo`
説明:	このパラメータは、ウェルが "ホット "データを保存する場所を制御します。2つのウェルが同じディレクトリを指すことは許されません!

### エージアウトオプション

####**Hot-Duration**
デフォルト値:
例:		`Hot-Duration=1w`
説明:	このパラメータは、データが "ホット "ストレージに保存される期間を決定する。この値は数字の後に "d"（日）または "w"（週）というサフィックスをつけたもので、`Hot-Duration=30d`はデータを30日間保持することを意味します。ただし、 `Cold-Location` パラメータを指定するか、 `Delete-Cold-Data` を true に設定しない限り、実際にデータがホットストレージから移動されることはないことに注意してください。

####**Cold-Location**
デフォルト値:
例:		`Cold-Location=/opt/gravwell/cold_storage/foo`
説明:	このパラメータは、`Location`で指定されたホットストアから移動されたデータであるコールドデータの保存場所を設定します。

####**Cold-Duration**
デフォルト値:
例:		`Cold-Duration=365d`
説明:	このパラメータは、データが "cold "ストレージに保存される期間を決定します。Delete-Frozen-Data` が true に設定されていない限り、データはコールドストレージから実際に移動されることはありません。

####**Max-Hot-Storage-GB**
デフォルト値:
例:		`Max-Hot-Storage-GB=100`
説明:	このパラメータは、指定されたウェルのホットストレージの最大ディスク消費量をギガバイト単位で設定します。この数値を超えた場合、最も古いデータはコールドストレージに移行されるか（可能な場合）、クラウドストレージに送信されるか（設定されている場合）、削除されます（許可されている場合）。

####**Max-Cold-Storage-GB**
デフォルト値:
例:		`Max-Cold-Storage-GB=100`
説明:	このパラメータは、指定されたウェルのコールドストレージの最大ディスク消費量をギガバイト単位で設定します。この数値を超えた場合、最も古いデータはクラウドストレージに送られ（設定されている場合）、削除されます（許可されている場合）。

####**Hot-Storage-Reserve**
デフォルト値:
例:		`Hot-Storage-Reserve=10`
説明:	このパラメータは、ディスクに少なくとも特定の割合の空き容量を残しておくようにウェルに指示する。したがって、`Hot-Storage-Reserve=10` を設定すると、ディスクの使用率が 90% に達したときに、ウェルはホットストレージからデータをエージアウトしようとする。

####**Cold-Storage-Reserve**
デフォルト値:
例:		`Cold-Storage-Reserve=10`
説明:	このパラメータは、ディスクに少なくとも特定の割合の空き領域を残すようウェルに指示する。したがって、`Cold-Storage-Reserve=10` を設定すると、ディスクの使用率が90%に達したときに、コールドストレージからデータをエージアウトしようとする。

####**Delete-Cold-Data**
デフォルト値:	false
例:		`Delete-Cold-Data=true`
説明:	このパラメータをtrueに設定すると、エージアウト基準のいずれかが満たされたときに、ホットストレージからデータを削除できることを意味します。

####**Delete-Frozen-Data**
デフォルト値:	false
例:		`Delete-Frozen-Data=true`
説明:	このパラメータをtrueに設定すると、エージアウト基準のいずれかが満たされたときに、コールドストレージからデータを削除できるようになります。

####**Archive-Deleted-Shards**
デフォルト値:	false
例:		`Archive-Deleted-Shards=true`
説明:	このオプションが設定されている場合、ウェルはシャードを削除する前に外部のアーカイブサーバにアップロードしようとします。このオプションは `[Cloud-Archive]` セクションが設定されている場合のみ動作することに注意してください!

####**Disable-Compression, Disable-Hot-Compression, Disable-Cold-Compression**
デフォルト値:	false
例:		`Disable-Compression=true`
説明:	これらのパラメータは、ユーザーモードによるウェル内のデータ圧縮を制御します。デフォルトでは、DatalaiQはウェル内のデータを圧縮します。Disable-Hot-Compression` と `Disable-Cold-Compression` を設定すると、それぞれホットストレージとコールドストレージで圧縮が無効になり、 `Disable-Compression` を設定すると両方で圧縮が無効になります。

####**Enable-Transparent-Compression, Enable-Hot-Transparent-Compression, Enable-Cold-Transparent-Compression**
デフォルト値:	false
例:		`Enable-Transparent-Compression=true`
説明:	これらのパラメータは、カーネルレベルでウェル内のデータの透過的圧縮を制御します。有効にすると、DatalaiQ は `btrfs` ファイルシステムにデータを透過的に圧縮するように指示することができます。これはユーザーモードによる圧縮よりも効率的です。Enable-Transparent-Compression` を true に設定すると、自動的にユーザーモード圧縮をオフにします。Disable-Compression=true` を設定すると、透過圧縮は **disable** となることに注意してください。

####**Ageout-Time-Override**
デフォルト値:
例:		`Ageout-Time-Override="3:00AM"`
説明:	このパラメータでは、エージアウトルーチンが実行される特定の時間を指定することができます。これは通常必要ありません。

### アクセラレーションオプション

####**Accelerator-Name**
デフォルト値:	
例:		`Accelerator-Name=json`
説明:	`Accelerator-Name`パラメータ(と `Accelerator-Args`パラメータ)を設定すると、そのウェルでアクセラレータが有効になります。詳しくは、[アクセラレーションに関するドキュメント](#!configuration/accelerators.md)を参照してください。

####**Accelerator-Args**
デフォルト値:	
例:		`Accelerator-Args="username hostname \"strange-field.with.specials\".subfield"`
説明:	`Accelerator-Args` パラメータ (および `Accelerator-Name` パラメータ) を設定すると、そのウェルでアクセラレーションが有効になります。詳しくは、[アクセラレーションに関するドキュメント](#!configuration/accelerators.md)を参照してください。

####**Accelerate-On-Source**
デフォルト値:	false
例:		`Accelerate-On-Source=true`
説明:	各モジュールの SRC フィールドを含めるように指定する。これにより、CEFのようなモジュールとSRCを組み合わせることができる。

####**Accelerator-Engine-Override**
デフォルト値:	"index"
例:		`Accelerator-Engine-Override=bloom`
説明:	使用するアクセラレーションエンジンを選択します。デフォルトでは、インデックス作成アクセラレータが使用されます。このパラメータを "bloom" に設定すると、代わりにブルームフィルタが選択されます。

####**Collision-Rate**
デフォルト値:	0.001
例:		`Collision-Rate=0.01`
説明:	ブルームフィルターの加速度エンジンの精度を設定します。0.1 から 0.000001 の間の値である必要があります。

### Generalオプション

####**Disable-Replication**
デフォルト値:	false
例:		`Disable-Replication=true`
説明:	設定された場合、このウェルの内容は複製されません。

####**Enable-Quarantine-Corrupted-Shards**
デフォルト値:	false
例:		`Enable-Quarantine-Corrupted-Shards=true`
説明:	設定すると、回復できない破損したシャードは、後で分析するために隔離された場所にコピーされます。デフォルトでは、ひどく破損したシャードは削除されることがあります。

## レプリケーション設定

Replication]`セクションは、[DatalaiQのレプリケーション機能](#!configuration/replication.md)を設定するセクションです。設定例としては以下のようなものがあります:

```
[Replication]
	Disable-Server=true
	Peer=10.0.01
	Storage-Location=/opt/gravwell/replication_storage
```

レプリケーション設定ブロックは、インデクサにのみ適用されます。

####**Peer**
デフォルト値:
例:		`Peer=10.0.0.1:9406`
説明:	Peer` パラメータはレプリケーションピアを指定します。IP アドレスかホスト名を指定し、最後にオプションでポートを指定します。ポートが指定されない場合は、デフォルトのポート (9406) が使用されます。Peer` は複数回指定することができます。

####**Listen-Address**
デフォルト値:	":9406"
例:		`Listen-Address=192.168.1.1:9406`
説明:	このパラメータは、DatalaiQがレプリケーション接続の着信を待機するIPとポートを定義します。デフォルトでは、すべてのインターフェイスでポート9406をリッスンします。

####**Storage-Location**
デフォルト値:	
例:		`Storage-Location=/opt/gravwell/replication`
説明:	他の DatalaiQ インデクサから複製されたデータの保存場所を設定します。

####**Max-Replicated-Data-GB**
デフォルト値:
例:		`Max-Replicated-Data-GB=100`
説明:	複製されたデータを保存する最大量をギガバイト単位で設定します。これを超えると、インデクサは複製されたデータを整理するために歩き始めます。まず、元のインデクサで削除されたシャードを削除し、次に最も古いシャードの削除を開始します。ストレージのサイズが制限を下回ると、削除は停止します。

####**Replication-Secret-Override**
デフォルト値:
例:		`Replication-Secret-Override=MyReplicationSecret`
説明:	デフォルトでは、DatalaiQはレプリケーションの認証に `Control-Auth` トークンを使用します。このパラメータを設定すると、代わりにカスタムのレプリケーション認証トークンを定義することができます。

####**Disable-TLS**
デフォルト値:	false
例:		`Disable-TLS=true`
説明:	このパラメータを true に設定すると、レプリケーションのための TLS が無効になります。インデクサは暗号化されていない着信接続をリッスンし、暗号化されていない接続を使用してピアと通信します。

####**Key-File**
デフォルト値: (value of `Key-File` in `[Global]` section)
例:		`Key-File=/opt/gravwell/etc/replication-key.pem`
説明:	このパラメータを使用すると、TLS接続に、グローバルに定義された鍵ではなく、別の鍵を使用することができます。

####**Certificate-File**
デフォルト値: (value of `Certificate-File` in `[Global]` section)
例:		`Certificate-File=/opt/gravwell/etc/replication-cert.pem`
説明:	このパラメータを使用すると、TLS接続に、グローバルに定義された証明書ではなく、別の証明書を使用することができます。

####**Insecure-Skip-TLS-Verify**
デフォルト値:	false
例:		`Insecure-Skip-TLS-Verify=false`
説明:	このパラメータをtrueに設定すると、レプリケーションピアに接続する際にTLS証明書の検証を無効にします。

####**Connect-Wait-Timeout**
デフォルト値:	30
例:		`Connect-Wait-Timeout=60`
説明:	レプリケーション・ピアへの接続時に使用するタイムアウトを秒単位で設定します。

####**Disable-Server**
デフォルト値:	false
例:		`Disable-Server=true`
説明:	レプリケーションサーバ*機能を無効にします。設定された場合、インデクサはそれ自身のデータをレプリケーションピアにプッシュしますが、他のインデクサがそれに対してプッシュすることは許可されません。

####**Disable-Compression**
デフォルト値:	false
例:		`Disable-Compression=true`
説明:	複製されたデータの圧縮を制御します。デフォルトでは、複製されたデータはディスク上で圧縮されます。

####**Enable-Transparent-Compression**
デフォルト値:	false
例:		`Enable-Transparent-Compression=true`
説明:	このパラメータが true に設定されると、DatalaiQ は複製されたデータに対して btrfs 透過圧縮を使用しようとします。Disable-Compression=true` を設定すると、これを無効にすることができます。

## シングルサインオン設定

SSO]`設定セクションは、DatalaiQ Webサーバーのシングルサインオンオプションを制御します。セクションのサンプルは以下のようなシンプルなものです。:

```
[SSO]
	Gravwell-Server-URL=https://10.10.254.1:8080
	Provider-Metadata-URL=https://sso.gravwell.io/FederationMetadata/2007-06/FederationMetadata.xml
```

しかし、より頻繁に追加の設定が必要になります:

```
[SSO]
	Gravwell-Server-URL=https://10.10.254.1:8080
	Provider-Metadata-URL=https://sso.gravwell.io/FederationMetadata/2007-06/FederationMetadata.xml
	Groups-Attribute=http://schemas.xmlsoap.org/claims/Group
	Group-Mapping=Gravwell:gravwell-users
	Group-Mapping=TestGroup:testgroup
	Username-Attribute = "uid"
	Common-Name-Attribute = "cn"
	Given-Name-Attribute  = "givenName"
	Surname-Attribute = "sn"
	Email-Attribute = "mail"
```

詳しくは、[SSO設定資料](sso.md)を参照してください。

####**Gravwell-Server-URL**
デフォルト値:
例:		`Gravwell-Server-URL=https://gravwell.example.org/`
説明:	SSOサーバーがユーザーを認証した後にリダイレクトされるURLを指定します。これは、DatalaiQサーバーのユーザー向けホスト名またはIPアドレスである必要があります。このパラメータは必須です。

####**Provider-Metadata-URL**
デフォルト値:
例:		 `Provider-Metadata-URL=https://sso.example.org/FederationMetadata/2007-06/FederationMetadata.xml`
説明:	SSOサーバーのXMLメタデータのURLを指定します。上記のパス（`/FederationMetadata/2007-06/FederationMetadata.xml`）はAD FSサーバーでは機能しますが、他のSSOプロバイダーでは調整する必要があるかもしれません。このパラメータは必須です。

####**Insecure-Skip-TLS-Verify**
デフォルト値:	false
例:		`Insecure-Skip-TLS-Verify=true`
説明:	trueに設定すると、このパラメーターは、SSOサーバーと通信する際に無効なTLS証明書を無視するようにDatalaiQに指示します。このオプションの設定には注意が必要です。

####**Username-Attribute**
デフォルト値:	"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"
例: 		`Username-Attribute = "uid"`
説明:	ユーザ名を格納する SAML 属性を定義する。Shibbolethサーバでは、代わりに "uid "を設定する。

####**Common-Name-Attribute**
デフォルト値: "http://schemas.xmlsoap.org/claims/CommonName"
例:		`Common-Name-Attribute="cn"`
説明:	ユーザの "コモンネーム "を含むSAML属性を定義する。Shibbolethサーバでは、これは代わりに "cn "に設定されるべきである。

####**Given-Name-Attribute**
デフォルト値:	"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname"
例:		`Given-Name-Attribute="givenName"`
説明:	ユーザの名前を格納する SAML 属性を定義する。Shibbolethサーバでは、この代わりに "givenName "を設定する必要がある。

####**Surname-Attribute**
デフォルト値:	"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname"
例:		`Surname-Attribute=sn`
説明:	ユーザの姓を含む SAML 属性を定義する。Shibbolethサーバでは、この代わりに "sn "を設定する必要があります。

####**Email-Attribute**
デフォルト値:	"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress"
例:		`Email-Attribute="mail"`
説明:	ユーザのメールアドレスを格納する SAML 属性を定義する。Shibbolethサーバでは、この代わりに "mail "を設定する必要がある。

####**Groups-Attribute**
デフォルト値:	"http://schemas.microsoft.com/ws/2008/06/identity/claims/groups"
例:		`Groups-Attribute="groups"`
説明:	ユーザが所属するグループのリストを含む SAML 属性を定義する。通常、グループリストを送信するように SSO プロバイダを明示的に構成する必要があります。

####**Group-Mapping**
デフォルト値:	
例:		`Group-Mapping=Gravwell:gravwell-users`
説明:	ユーザーのグループメンバーとして登録されている場合、自動的に作成されるグループの一つを定義します。これは、複数のグループを許可するために複数回指定することができます。引数はコロンで区切られた2つの名前で構成される必要があります。1つ目はグループのSSOサーバー側の名前（通常、AD FSの名前、AzureのUUIDなど）、2つ目はDatalaiQが使用する名前です。したがって、`Group-Mapping=Gravwell Users:gravwell-users` を定義すると、グループ "DatalaiQ Users" のメンバーであるユーザーのログイントークンを受け取った場合、 "gravwell-users" という名前のローカルグループを作成し、そのユーザーをそこに追加することになるのです。

## クラウドアーカイブ設定

DatalaiQは、個々のウェルで`Archive-Deleted-Shards`パラメータを使用して、データシャードを削除する前にリモートのCloud Archiveサーバーにアーカイブするように設定することができます。Cloud-Archive]`設定セクションで、これを有効にするためのCloud Archiveサーバーの情報を定義します。

```
[Cloud-Archive]
	Archive-Server=10.0.0.2:443
	Archive-Shared-Secret=MyArchiveSecret
```

クラウドアーカイブの設定は、インデクサにのみ適用されます。

####**Archive-Server**
デフォルト値:
例:		`Archive-Server=cloudarchive.example.org:443`
説明:	このパラメータでは、クラウドアーカイブサーバーのIP/ホスト名と、オプションでポートを指定します。ポートを指定しない場合は、デフォルト（443）が使用されます。

####**Archive-Shared-Secret**
デフォルト値:
例:		`Archive-Shared-Secret=MyArchiveSecret`
説明:	クラウドアーカイブサーバーと認証する際に使用する共有シークレットを設定します。インデクサは、ライセンスの顧客ID番号を認証プロセスの残り半分として使用します。

####**Insecure-Skip-TLS-Verify**
デフォルト値:	false
例:		`Insecure-Skip-TLS-Verify=true`
説明:	trueを設定すると、インデクサは接続時にクラウドアーカイブサーバーのTLS証明書を検証しない。
