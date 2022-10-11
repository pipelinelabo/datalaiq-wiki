# カスタムDockerデプロイメント

ほとんどのGravwellコンポーネントは静的にコンパイルされた実行ファイルとして配備されており、最新のLinuxホストでの実行に適しており、Dockerでの配備も簡単に行えます。 Gravwellのエンジニアは、開発、テスト、内部展開のために広範囲に渡ってDockerを使用しています。 Dockerはまた、お客様が迅速に立ち上げ、撤収し、そうでなければ大規模なGravwellの展開を管理することを可能にします。 お客様がDocker展開で素早く実行できるように、我々はクラスタとシングル版のSKUの両方にサンプルDockerfileを提供しています。

Dockerfile一式は[こちら](https://update.gravwell.io/files/docker_buildfiles_ad05723a547d31ee57ed8880bb5ef4e9.tar.bz2)にあります。MD5のチェックサムはad05723a547d31ee57ed8880bb5ef4e9です。

## Dockerコンテナを構築する

提供されたDockerfileを使用したDockerコンテナの構築は非常に簡単です。 GravwellのDockerデプロイメントでは、非常に小さなbusyboxベースコンテナを利用し、非常に小さなコンテナを実現します。

### Dockerfile

特定の展開要件に合うように docker ファイルを修正するのは、ほとんど作業が必要ありません。 標準的な gravwell docker ファイルは、起動時に X509 証明書を再生成する必要があるかどうかをチェックするために、小さな起動スクリプトを使用しています。 あなたのデプロイメントに有効な証明書がある場合、Gravwell バイナリを直接起動して、インストーラによってデプロイされる /opt/gravwell/bin のユーティリティの一部 (すなわち gencert と crashreport) を削除することができます。

単一インスタンス用のベースDockerfile:
```
FROM busybox
MAINTAINER support@ppln.co
ARG INSTALLER=gravwell_installer.sh
COPY $INSTALLER /tmp/installer.sh
COPY start.sh /tmp/start.sh
RUN /bin/sh /tmp/installer.sh --no-questions
RUN rm -f /tmp/installer.sh
RUN mv /tmp/start.sh /opt/gravwell/bin/
CMD ["/bin/sh", "/opt/gravwell/bin/start.sh"]
```

基本的な起動スクリプト:
```
#!/bin/sh

# unless environment variable says no, generate new SSL certs
if [ "$NO_SSL_GEN" == "true" ]; then
	echo "Skipping SSL certificate generation"
else
	/opt/gravwell/bin/gencert -key-file /opt/gravwell/etc/key.pem -cert-file /opt/gravwell/etc/cert.pem -host 127.0.0.1
	if [ "$?" != "0" ]; then
		echo "Failed to generate certificates"
		exit -1
	fi
fi

#fire up the indexer and webserver processes and wait
/opt/gravwell/bin/gravwell_indexer -config-override /opt/gravwell/etc/ &
/opt/gravwell/bin/gravwell_webserver -config-override /opt/gravwell/etc/ &
wait
```

## 環境変数を使用する

標準の docker 起動スクリプトは環境変数 `NO_SSL_GEN` を探し、"true" に設定すると X509 証明書の生成をスキップします。 デプロイメントで有効な証明書を注入する場合は、コンテナを起動する際に `-e NO_SSL_GEN=true` という引数を必ず含めてください。

インデクサ、ウェブサーバ、インジェスタは、必要であれば、設定ファイルではなく、環境変数でいくつかの設定パラメータを設定することも可能です。詳しくは [環境変数](environment-variables.md) のドキュメントを参照してください。

## インジェスター用サンプルDockerfile

Gravwellは継続的に新しいインジェストやコンポーネントをリリースしていますが、必ずしもすべてのインジェストのDockerfileを用意しているとは限りません。 しかし、Dockerfileは非常にわかりやすく、簡単に変更することができます。 以下はSimpleRelayインジェスターを経由してDockerコンテナを構築するDockerfileの例です。

```
FROM busybox
MAINTAINER support@ppln.co
ARG INSTALLER=gravwell_installer.sh
COPY $INSTALLER /tmp/installer.sh
RUN /bin/sh /tmp/installer.sh --no-questions
RUN rm -f /tmp/installer.sh
CMD ["/opt/gravwell/bin/gravwell_simple_relay"]
```

コンテナを構築するには、simple relay installer を Docker ファイルと同じ作業ディレクトリにコピーし、以下のコマンドを実行します。:
```
docker build --ulimit nofile=32000:32000 --compress --build-arg INSTALLER=gravwell_simple_relay_installer_2.0.sh --no-cache --tag gravwell:simple_relay_2.0.0 .
```
