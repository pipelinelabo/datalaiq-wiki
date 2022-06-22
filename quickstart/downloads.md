# Downloads

Attention: The Debian and RHEL repositories are more easily maintained than these standalone installers and are the recommended methods of installation. See the [quickstart instructions](#!quickstart/quickstart.md).

## DatalaiQ Core

The DatalaiQ core installer contains the indexer and webserver frontend. You'll need a license; either get a Community Edition free license, or contact support@pipelinesecurity.net for commercial options.

[Download DatalaiQ Core Installer](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_pipeline-5.0.2-1.sh) (SHA256: 1b0112b713d09155a39025487faf9bf341b134c82a7c7f9ad6b98ccf7293cc94)

## Ingesters

The core suite of ingesters are available for download as installable packages.  Ingesters designed to operate on Linux machines are typically self contained, statically linked executables that are agnostic to the hosts package management system (with the exception of the NetworkCapture ingester).  Windows based ingesters are distributed as executable MSI packages.  Source code for many ingesters can be found at the [DatalaiQ Github](https://github.com/pipelinelabo/datalaiq-wiki) repository.

### Current Ingester Releases
| Ingester | SHA256 | More Info |
|:--------:|-------:|----------:|
| [Simple Relay](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_simple_relay_installer_5.0.2.sh) | ``329cc83d4798e7067794e82ec12fca67e05ced691457c6068578e49f36355ad2`` | [Documentation](#!ingesters/ingesters.md#Simple_Relay)|
| [File Follower](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_file_follow_installer_5.0.2.sh) | ``f8688d2500a3205483f6f32819b5d3d719f536a74a5c0457e918757c422a9633`` | [Documentation](#!ingesters/ingesters.md#File_Follower) |
| [HTTP Ingester](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_http_ingester_installer_5.0.2.sh) | ``1990b41b1a0d774682e856a609747a73947499a9c9e2e67e31fb17d6a3c1c28f`` | [Documentation](#!ingesters/ingesters.md#HTTP_POST) |
| [IPMI Ingester](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_ipmi_installer_5.0.2.sh) | ``d348e282b0ed3dd3d4af86117478f6d409aad9bc607832046b1d1724a4154850`` | [Documentation](#!ingesters/ingesters.md#IPMI_Ingester)|
| [Netflow Capture](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_netflow_capture_installer_5.0.2.sh) | ``2388ee6633f7f25b10a66de99261d9dd5386fb4165797e8a4373e31a41289421`` | [Documentation](#!ingesters/ingesters.md#Netflow_Ingester) |
| [Network Capture](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_network_capture_installer_5.0.2.sh) | ``49d380a312bdc88da489dcbebaa68cc6a36f67dd821b08cb05a52941a7cd6e9f`` | [Documentation](#!ingesters/ingesters.md#Network_Ingester) |
| [Collectd Collector](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_collectd_installer_5.0.2.sh) | ``fb1e31597323a3b612d94b45ba1d389d38778d9db8dfb2a23dc43c385cfb9c59`` | [Documentation](#!ingesters/ingesters.md#collectd) |
| [Ingest Federator](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_federator_installer_5.0.2.sh) | ``a6f89ce16eaed8cc3210fa30a9aaf03d7fc5ff001945ee25592494636aa16853`` | [Documentation](#!ingesters/ingesters.md#Federator_Ingester) |
| [Windows Events](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_win_events_5.0.2.msi) | ``57564845d010dd1756f5b706f31176bf12961375d0ea1d2c430411b6e74db3d4`` | [Documentation](#!ingesters/ingesters.md#Windows_Event_Service) |
| [Windows File Follower](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_file_follow_5.0.2.msi) | ``691db21112893e223398935d0d27c31f6fc76805798aa99bb24da109c3b18272`` | [Documentation](#!ingesters/ingesters.md#File_Follower) |
| [Apache Kafka](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_kafka_installer_5.0.2.sh) | ``a8c8f840a041223a378c0ca9931555f56b43a172257660f76b9357efa1dcb898`` | [Documentation](#!ingesters/ingesters.md#Kafka)|
| [Amazon Kinesis](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_kinesis_ingest_installer_5.0.2.sh) | ``ea3b925a12d343b74ef22455950936144dc203ee313590e49c2cf3506e51ebbc`` | [Documentation](#!ingesters/ingesters.md#Kinesis_Ingester)|
| [Google PubSub](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_pubsub_ingest_installer_5.0.2.sh) | ``248caac7a46fa822a61f2ff44030c2f6fe234f94810669e2118eed567b2a7e87`` | [Documentation](#!ingesters/ingesters.md#GCP_PubSub)|
| [Office 365 Logs](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_o365_installer_5.0.2.sh) | ``c7476daf29242cb0fdcf6761609b5cdfbab8e4d92b65d24464c53e549297b4cb`` | [Documentation](#!ingesters/ingesters.md#Office_365_Log_Ingester)|
| [Microsoft Graph API](http://repo.datalaiq.io/archive/5.0.2-1/installers/datalaiq_msgraph_installer_5.0.2.sh) | ``2bc6476373284752e1fe58f24c8731d0722afaab863eca372436146479f67dd2`` | [Documentation](#!ingesters/ingesters.md#Microsoft_Graph_API_Ingester)|

[//]: # (## Other downloads)

[//]: # (Some DatalaiQ components are distributed as optional additional installers, such as the search agent and the datastore.)

[//]: # (| Component | SHA256 | More Info |)
[//]: # (|:---------:|:------:|----------:|)
[//]: # (| [Datastore]&#40;https://update.gravwell.io/archive/5.0.2/installers/gravwell_datastore_installer_5.0.2.sh&#41; | ``cfbf16f99df8f50eebe6d5f02df0ecd29ef30fb3e927a3b0ed413bf8e32ff3ef`` | [Documentation]&#40;#!distributed/frontend.md&#41; |)
[//]: # (| [Offline Replicator]&#40;https://update.gravwell.io/archive/5.0.2/installers/gravwell_offline_replication_installer_5.0.2.sh&#41; | ``17d36a3c559663262db2baf4e7fe21bd9a4d811df2c6e6edfad0bcac0d8916b9`` | [Documentation]&#40;#!configuration/replication.md&#41; |)
[//]: # (| [Load Balancer]&#40;https://update.gravwell.io/archive/5.0.2/installers/gravwell_loadbalancer_installer_5.0.2.sh&#41; | ``1edc316a7635e75e702dc734b5387b43700a45c353418d15f22d1f22f555781d`` | |)
