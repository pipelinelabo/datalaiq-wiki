# Controlling System Resource Usage

This section describes how you can tune the DatalaiQ system's consumption of system resources.

## Webserver Rendererストレージ

chartやtableなどのrenderモジュールは、結果をwebserverのディスク上に保持します。その結果、クエリによっては膨大なディスク容量を消費する可能性があります。これを避けるには、gravwell.confの `Render-Store-Limit` パラメーターの調整が必要です。 `Render-Store-Limit=64` に設定すると、クエリ毎のディスク消費を64MBまでに制限します。

## DatalaiQのresourcesサイズ制限

ユーザーが作成する [resources](#!resources/resources.md) はwebserverとindexerのディスクを大きく消費する可能性があります。gravwell.confの `Resource-Max-Size` パラメーターでは、このresourcesのサイズをバイト単位で制限することができます。 `Resource-Max-Size=20971520` に設定すると、resourcesのディスク消費を20MB未満に制限することができます。
