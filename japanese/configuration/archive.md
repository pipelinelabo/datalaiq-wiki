# クラウドアーカイブ

DatalaiQは、クラウドアーカイブというエージアウトの仕組みをサポートしています。 クラウドアーカイブは、データを削除する前に遠隔地からアーカイブすることができるサービスです。 DatalaiQ Cloud Archiveは、アクティブに検索する必要はないものの、保持する必要があるデータの長期アーカイブ保存に最適な方法です。 Cloud Archiveサービスは様々なストレージプラットフォーム上でホスティングすることができ、リモートで低コストのストレージプラットフォームを提供するように設計されています。 クラウドアーカイブの設定はウェル単位で可能で、どのデータセットが長期保存に適しているかを判断することができます。

アーカイブシステムは、通常のエージアウトで削除される前に、データがアーカイブサーバーに正常にアップロードされることを確認します。

注意: インデックス作成者は、アーカイブサーバーへのアップロードに成功するまで、データを削除しません。 接続の問題、設定の誤り、ネットワークのスループットの低下などの理由でインデクサがアップロードできない場合、データを削除することはありません。 データを削除できない場合、インデクサーのストレージが不足し、新しいデータの取り込みを停止することがあります。 クラウドアーカイブのアップロードが完了しなかった場合、ユーザーインターフェースに失敗の通知が表示されます。

注意: クラウドアーカイブシステムは、転送中にデータを圧縮するため、アップロード時にCPUリソースを必要とします。 データをリモートシステムにプッシュする場合も、利用可能な帯域幅とCPUに依存しますが、時間がかかります。 クラウドアーカイブで消費される追加時間を考慮し、エージアウトパラメータを定義する際には少し余裕を持たせてください。

## インデクサー設定

すべてのインデクサは、リモートアーカイブサーバと認証トークンを指定するグローバルなクラウドアーカイブコンフィギュレーションブロックを有しています。この設定ブロックは、グローバルセクションの "[cloud-archive]" ヘッダを使用して指定します。 ウェル上でCloud Archiveを有効にするには、ウェル内に "Archive-Deleted-Shards=true" ディレクティブを追加してください。

以下は、3つの井戸を持つ構成例です:

```
[global]
Web-Port=443
Control-Port=9404
Ingest-Port=4023

[Cloud-Archive]
	Archive-Server=test.archive.gravwell.io
	Archive-Shared-Secret="password"

[Default-Well]
	Location=/opt/gravwell/storage/default/
	Cold-Location=/opt/gravwell/cold_storage/default/
	Hot-Duration=1d
	Cold-Duration=30d
	Delete-Frozen-Data=true
	Archive-Deleted-Shards=true

[Storage-Well "netflow"]
	Location=/opt/gravwell/storage/netflow/
	Hot-Duration=7d
	Delete-Cold-Data=true
	Archive-Deleted-Shards=true
	Tags=netflow

[Storage-Well "raw"]
	Location=/opt/gravwell/storage/raw/
	Hot-Duration=7d
	Delete-Cold-Data=true
	Tags=pcap
```

上記の例では、3つのウェルが設定されています（デフォルト、ネットフロー、ロー）。 デフォルトのウェルは、ホットストレージ層とコールドストレージ層の両方を使用しており、コールドストレージ層から通常ロールアウトするタイミングでデータがアーカイブされることを意味します。 netflowウェルにはホットストレージ層しかなく、そのデータは通常7日後に削除されるところ、アップロードされます。 rawウェルはクラウドアーカイブを有効にしていない（Archive-Deleted-Shards=false）ため、そのデータはアップロードされません。

## クラウドアーカイブホスティング

クラウドアーカイブサービスは、セルフホスティングや他の大規模インフラへの統合を想定して設計されたモジュールサービスです。 クラウドアーカイブサービスのホスティングやリモートでのデータアーカイブにご興味のある方は、support@ppln.co までご連絡ください。

備考: インデクサは、インデクサ上の顧客ライセンス番号を使用して、クラウドアーカイブサービスに認証されます。[overwatch](#!distributed/overwatch.md) 構成では、この番号は *webserver* に展開されたライセンス番号と異なる可能性があります。