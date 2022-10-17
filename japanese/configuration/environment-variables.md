# 環境変数

indexer、webserver、および ingester コンポーネントは、設定ファイルではなく、環境変数によるいくつかのパラメータの設定をサポートしています。これは、より大きな展開のために、より汎用的な設定ファイルを作成するのに役立ちます。複数のディレクティブを含むことができる設定変数は、 カンマ区切りのリストを使って環境変数に設定されます。たとえば、Federator を起動するときに ingest secret を指定するには、次のようにします:

```
GRAVWELL_INGEST_SECRET=MyIngestSecret /opt/gravwell/bin/gravwell_federator
```

環境変数名の最後に"_FILE "を追加すると、Gravwellはその変数に目的のデータを含むファイルへのパスが含まれていると見なします。Dockerの "secrets "機能](https://docs.docker.com/engine/swarm/secrets/)と組み合わせると特に効果的です。

```
GRAVWELL_INGEST_AUTH_FILE=/run/secrets/ingest_secret /opt/gravwell/bin/gravwell_indexer
```

備考: 環境変数の値は、対応するフィールドが適切な設定ファイル（gravwell.conf またはインジェスターの設定ファイル）で明示的に設定されていない場合に **のみ** 使用されます。

## インデクサーとウェブサーバー

以下の表は、インデクサとウェブサーバの環境変数で設定できる `gravwell.conf` パラメータを表しています。これらの変数は、そのパラメータが `gravwell.conf` で設定されていない場合にのみ使用されることに注意してください。

| gravwell.conf variable | Environment Variable | Example |
|:------|:----|:---|----:|
| Ingest-Auth | GRAVWELL_INGEST_AUTH | GRAVWELL_INGEST_AUTH=CE58DD3F22422C2E348FCE56FABA131A |
| Control-Auth | GRAVWELL_CONTROL_AUTH | GRAVWELL_CONTROL_AUTH=C2018569D613932A6BBD62A03A101E84 |
| Indexer-UUID | GRAVWELL_INDEXER_UUID | GRAVWELL_INDEXER_UUID=a6bb4386-3433-11e8-bc0b-b7a5a01a3120 |
| Webserver-UUID | GRAVWELL_WEBSERVER_UUID | GRAVWELL_WEBSERVER_UUID=b3191f54-3433-11e8-a0c2-afbff4695836 |
| Remote-Indexers | GRAVWELL_REMOTE_INDEXERS | GRAVWELL_REMOTE_INDEXERS=172.20.0.1:9404,172.20.0.2:9404|
| Replication-Peers | GRAVWELL_REPLICATION_PEERS | GRAVWELL_REPLICATION_PEERS=172.20.0.1:9406,172.20.0.2:9406 |
| Datastore | GRAVWELL_DATASTORE | GRAVWELL_DATASTORE=172.20.0.10:9405 |

## インジェスター

インジェスターは、いくつかのパラメータを設定ファイルに明示的に設定するのではなく、環境変数として受け入れることもできます。

| Config file variable | Environment Variable | Example |
|:------|:----|:---|
| Ingest-Secret | GRAVWELL_INGEST_SECRET | GRAVWELL_INGEST_SECRET=CE58DD3F22422C2E348FCE56FABA131A |
| Log-Level | GRAVWELL_LOG_LEVEL | GRAVWELL_LOG_LEVEL=DEBUG |
| Cleartext-Backend-target | GRAVWELL_CLEARTEXT_TARGETS | GRAVWELL_CLEARTEXT_TARGETS=172.20.0.1:4023,172.20.0.2:4023 |
| Encrypted-Backend-target | GRAVWELL_ENCRYPTED_TARGETS | GRAVWELL_ENCRYPTED_TARGETS=172.20.0.1:4024,172.20.0.2:4024 |
| Pipe-Backend-target | GRAVWELL_PIPE_TARGETS | GRAVWELL_PIPE_TARGETS=/opt/gravwell/comms/pipe |


### フェデレータ固有の変数

フェデレータは多くのリスナーを実行し、それぞれが異なる取り込み秘密を関連付ける可能性があるため、実行時にそれらのリスナー秘密を設定するための特別な環境変数のセットを認識します。

各リスナーには名前がついています。以下の例では、リスナーは "base" という名前になっています:

```
[IngestListener "base"]
	Cleartext-Bind = 0.0.0.0:4023
	Tags=syslog
```

実行時にそのリスナーのインジェストシークレットを指定するために、変数 `FEDERATOR_base_INGEST_SECRET` を使用します:

```
FEDERATOR_base_INGEST_SECRET=SuperSecret /opt/gravwell/bin/gravwell_federator
```

または、他の環境変数と同様にファイルを指定することもできます:

```
FEDERATOR_base_INGEST_SECRET_FILE=/run/secrets/federator_base_secret /opt/gravwell/bin/gravwell_federator
```

### データストア固有の変数

[データストア](#!distributed/frontend.md)は、実行時に環境変数で設定することが可能です:

| gravwell.conf variable | Environment variable | Example |
|------------------------|----------------------|---------|
| Datastore-Listen-Address | GRAVWELL_DATASTORE_LISTEN_ADDRESS | GRAVWELL_DATASTORE_LISTEN_ADDRESS=192.168.1.100 |
| Datastore-Port | GRAVWELL_DATASTORE_LISTEN_PORT | GRAVWELL_DATASTORE_LISTEN_PORT=9995 |
