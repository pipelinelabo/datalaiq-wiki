# DatalaiQのネットワークに関する考察

DatalaiQは、分散コンポーネント間の通信にいくつかのネットワークポートを使用します。この記事では、どのポートがどのような目的で使用されるかを説明します。

## インデクサコントロールポート：TCP 9404

gravwell.conf の `Control-Port` オプションで設定されるこのポートは、ウェブサーバーがインデキサーと通信するために使用されます。インデックスサーバー*上のファイアウォールが、*ウェブサーバー*からこのポートへの着信接続を許可していること、また、ウェブサーバーとインデックスサーバーの間でこのポートをブロックするネットワーク基盤がないことを確認します。

## ウェブサーバーポート： TCP 80/443

このポートは、DatalaiQユーザーがDatalaiQウェブサーバーにアクセスするためのものです。デフォルトの設定では、gravwell.conf の `Web-Port` オプションで指定されたポート 80 で暗号化されていない HTTP を使用します。この値は、必要に応じて8080などの別の値に変更することができます。TLS証明書をインストールする](#!configuration/certificates.md)場合、ポートを443に変更することをお勧めします。

## クリアテキスト・インジェストポート：TCP 4023

このポートは、インジェストがインデクサーに接続し、暗号化されていない通信でエントリーをアップロードするために使用されます。デフォルトのポートはTCP 4023ですが、gravwell.confの`Ingest-Port`オプションで変更することができます。インジェスターとインデクサーは全く別のネットワーク上にあることが多いので、*インジェスター*が*インデクサー*のこのポートへの接続を許可されるように、ファイアウォールを設定することが重要です。

## TLSインジェストポート：TCP 4024

このポートはインジェスターがインデクサーに接続し、TLS暗号化通信でエントリーをアップロードするために使用されます。デフォルトのポートはTCP 4024ですが、gravwell.confの`TLS-Ingest-Port`オプションで変更することが可能です。インジェスターとインデクサーは全く別のネットワーク上にあることが多いので、*インジェスター*が*インデクサー*のこのポートへの接続を許可されるように、ファイアウォールを設定することが重要です。

## インデックスレプリケーションポート：TCP 9606

このポートは、インデクサが[レプリケーション](#!configuration/replication.md)のために互いに通信するために使用されます。gravwell.confのReplication部分の `Peer` と `Listen-Address` オプションで指定されていない場合、デフォルトのポートは9606になります。このポートを使用するのはインデクサーのみです。

## データストア・ポート：TCP 9405

このポートは、DatalaiQクラスタに[複数のウェブサーバ](#!distributed/frontend.md)が構成されている場合に使用されます。データストア*コンポーネントは、このポート(`Datastore-Port`オプションで指定)で*ウェブサーバ*からの着信接続を待ち受けます。

## RHEL (Redhat Enterprise Linux) および CentOS のファイアウォールコマンド

RHEL/CentOSでは、独自のファイアウォールコマンドを使用します。便宜上、ウェブサーバとインデクサコンポーネント、およびSimple Relayインジェスタのポートを開くために必要なコマンドを集めました。ネットワークポートをリッスンするすべてのインジェスタは、おそらくこの方法でポートを開く必要があることに注意してください。

備考: ここで示されたコマンドは、一時的にポートを開くだけです。システムを再起動すると、ルールがリセットされます。ルールの変更を恒久的にするには、`sudo firewall-cmd --runtime-to-permanent` を実行してください。


### インデクサーポート

```
sudo firewall-cmd --zone=public --add-port=9404/tcp 
sudo firewall-cmd --zone=public --add-port=9405/tcp
sudo firewall-cmd --zone=public --add-port=4023/tcp
sudo firewall-cmd --zone=public --add-port=4024/tcp
```

### ウェブサーバーポート

```
sudo firewall-cmd --zone=public --add-service=http
sudo firewall-cmd --zone=public --add-service=https
```

### Simple Relayポート

```
sudo firewall-cmd --zone=public --add-port=7777/tcp
sudo firewall-cmd --zone=public --add-port=601/tcp
sudo firewall-cmd --zone=public --add-port=514/udp
```
