# ステータスを取得する

Websocket located at /ws/stats

サーバはすぐにRoutingWebsocketプロトコルを介して統計情報を投げてくるようになります。 これは以下のタイプが登録されていることを想定しており、それぞれについてデータを生成します:

* ping
* idxStats
* sysStats
* sysDesc

接続すると、ウェブソケットは最初に(そして常に最初に)sysDesc型のJSONパッケージを1つ投げます。 これは、あなたが統計を取り続けるバックエンドシステムの数を列挙するために、確実に使用することができます。 この最初のパケットを使って、テーブルなどを構築し、 idxStats と sysStats タイプで継続的に埋められるようにします。 ping タイプは keep alive であり、30 秒後に ping タイプから更新がない場合、バックエンドは死んでおり、ユーザに通知します。

# タイプ例

## システム詳細

```
{
   "HostNameA": {
        CPUCount:      int
        CPUModel:      string
        CPUMhz:        string
        CPUCache:      string
        CPUMIPS:       string
        TotalMemoryMB: uint64
        SystemVersion: string
    },
    "HostNameB": {
        CPUCount:      int
        CPUModel:      string
        CPUMhz:        string
        CPUCache:      string
        CPUMIPS:       string
        TotalMemoryMB: uint64
        SystemVersion: string
    }
}
```

## ping JSON例
```
{
	"Error": "",
	"States": [
		{
			"Addr": "localhost:9404",
			"State": "OK"
		},
		{
			"Addr": "10.0.0.1:9404",
			"State": "OK"
		}
	]
}
```

## sysStatsのJSONフォーマットパケット例
```json
{
	"Error": "",
	"Stats": [
		{
			"Name": "webserver",
			"Error": "",
			"Host": {
				"Uptime": 1237164,
				"TotalMemory": 8219250688,
				"FreeMemory": 501071872,
				"CachedMemory": 5131911168,
				"Disks": [
					{
						"Mount": "/",
						"Partition": "/dev/mapper/mint--vg-root",
						"Total": 243347922944,
						"Used": 30278324224
					},
					{
						"Mount": "/boot",
						"Partition": "/dev/sda1",
						"Total": 246755328,
						"Used": 47550464
					}
				],
				"CPUUsage": 9.66123,
				"Net": {
					"Up": 0,
					"Down": 0
				}
			}
		},
		{
			"Name": "localhost:9404",
			"Error": "",
			"Host": {
				"Uptime": 3946,
				"TotalMemory": 8248610816,
				"FreeMemory": 7495507968,
				"CachedMemory": 317022208,
				"Disks": [
					{
						"Mount": "/",
						"Partition": "/dev/disk/by-uuid/b3b698db-b6c7-4490-a34f-e60e57c9b8e0",
						"Total": 9707950080,
						"Used": 3922296832
					},
					{
						"Mount": "/",
						"Partition": "/dev/disk/by-uuid/b3b698db-b6c7-4490-a34f-e60e57c9b8e0",
						"Total": 9707950080,
						"Used": 3922296832
					},
					{
						"Mount": "/home",
						"Partition": "/dev/sda3",
						"Total": 19549782016,
						"Used": 12020940800
					},
					{
						"Mount": "/mnt/storage1",
						"Partition": "/dev/sda6",
						"Total": 284401197056,
						"Used": 214626893824
					}
				],
				"CPUUsage": 2.5773196,
				"Net": {
					"Up": 1709294,
					"Down": 1710926
				}
			}
		},
		{
			"Name": "10.0.0.1:9404",
			"Error": "",
			"Host": {
				"Uptime": 3946,
				"TotalMemory": 8248610816,
				"FreeMemory": 7495507968,
				"CachedMemory": 317022208,
				"Disks": [
					{
						"Mount": "/",
						"Partition": "/dev/disk/by-uuid/b3b698db-b6c7-4490-a34f-e60e57c9b8e0",
						"Total": 9707950080,
						"Used": 3922296832
					},
					{
						"Mount": "/",
						"Partition": "/dev/disk/by-uuid/b3b698db-b6c7-4490-a34f-e60e57c9b8e0",
						"Total": 9707950080,
						"Used": 3922296832
					},
					{
						"Mount": "/home",
						"Partition": "/dev/sda3",
						"Total": 19549782016,
						"Used": 12020940800
					},
					{
						"Mount": "/mnt/storage1",
						"Partition": "/dev/sda6",
						"Total": 284401197056,
						"Used": 214626893824
					}
				],
				"CPUUsage": 2.5773196,
				"Net": {
					"Up": 1709294,
					"Down": 1710926
				}
			}
		}
	]
}
```


## インデクサー統計情報のJSONフォーマットパケット例

```json
{
	"type": "idxStats",
	"data": {
		"Stats": {
			"127.0.0.1:9404": {
				"IndexStats": [{
					"Name": "default",
					"Stats": [{
						"Data": 39859808,
						"Entries": 504549,
						"Path": "/opt/sinkhole/storage/default"
					}]
				}, {
					"Name": "testingB",
					"Stats": [{
						"Data": 24,
						"Entries": 0,
						"Path": "/opt/sinkhole/storage/testB"
					}]
				}, {
					"Name": "testingA",
					"Stats": [{
						"Data": 3100139,
						"Entries": 30000,
						"Path": "/opt/sinkhole/storage/testA"
					}]
				}, {
					"Name": "testingC",
					"Stats": [{
						"Data": 24,
						"Entries": 0,
						"Path": "/opt/sinkhole/storage/testC"
					}]
				}]
			}
		}
	}
}
```