# 圧縮

ストレージはデフォルトで圧縮されており、ストレージからCPUへの負荷の移行を支援します。 DatalaiQは、スケールワイドのパラダイムで最新のハードウェア向けに構築された高度な非同期システムです。 最新のシステムは通常、CPUに過剰なプロビジョニングが施されており、大容量ストレージはそれに遅れをとっています。 DatalaiQはデータを圧縮することで、ストレージ・リンクのストレスを軽減し、余分なCPUサイクルを使って非同期にデータの圧縮と解凍を行うことができます。 その結果、低速の大容量記憶装置で圧縮を行う場合、検索をより高速に行うことができます。 圧縮は、ホットデータとコールドデータに対して異なる圧縮設定を行うことで、各ウェルに対して個別に設定することができます。

例外は、あまり圧縮されないデータです（まったく圧縮されない場合もあります）。このような場合、データを圧縮しようとしても、CPU時間を消費するだけで、ストレージスペースや速度の実際の向上は見込めません。生のネットワークトラフィックは、暗号化と高エントロピーのために効果的な圧縮ができない良い例である。 ウェルの圧縮を無効にするには、"Disable-Compression=true "ディレクティブを追加します。

## 圧縮設定

DatalaiQは、デフォルト圧縮と透過的圧縮の2種類の圧縮をサポートしています。 デフォルト圧縮は、[snppy](https://en.wikipedia.org/wiki/Snappy_%28compression%29)圧縮システムを使用して、ユーザースペースで圧縮と解凍を実行します。 デフォルトの圧縮システムは、すべてのファイルシステムと互換性があります。 透過的圧縮は、ファイルシステムを利用してブロック単位の透過的な圧縮を行います。

透過型圧縮は、非圧縮のページキャッシュを維持しながら、圧縮/解凍作業をホストカーネルにオフロードすることができます。 透過圧縮は非常に高速で効率的な圧縮/解凍を可能にしますが、基盤となるファイルシステムが透過圧縮をサポートしていることが必要です。 現在、[BTRFS](https://btrfs.wiki.kernel.org/index.php/Main_Page) と [ZFS](https://wiki.archlinux.org/index.php/ZFS) ファイルシステムがサポートされています。

注意: 透過的圧縮は、総保存量に関わるageoutルールに対して重要な意味を持ちます。より詳細な情報は [ageout documentation](ageout.md) を参照してください。

**Disable-Compression**
Default Value: `false`
Example: `Disable-Compression=true`
ウェル全体の圧縮を無効にし、温蔵庫と冷蔵庫の両方で圧縮を使用しないようにする

**Disable-Hot-Compression**
Default Value: `false`
Example: `Disable-Hot-Compression=true`
ホットストレージの圧縮は無効です。

**Disable-Cold-Compression**
Default Value: `false`
Example: `Disable-Cold-Compression=true`
コールドストレージの圧縮は無効です。コールドストレージが指定されていない場合、この設定は何の効果もありません。

**Enable-Transparent-Compression**
Default Value: `false`
Example: `Enable-Transparent-Compression=true`
DatalaiQはストレージデータを圧縮可能なものとしてマークし、圧縮処理の実行をカーネルに依存することになります。

**Enable-Hot-Transparent-Compression**
Default Value: `false`
Example: `Enable-Hot-Transparent-Compression=true`
DatalaiQは、ホットストレージのデータを圧縮可能なものとしてマークし、カーネルに依存して圧縮処理を実行します。

**Enable-Cold-Transparent-Compression**
Default Value: `false`
Example: `Enable-Cold-Transparent-Compression=true`
DatalaiQは、Coldストレージのデータを圧縮可能なものとしてマークし、圧縮処理をカーネルに依存する。

備考: 透過圧縮が有効で、基礎となるファイルシステムが透過圧縮と互換性がないと検出された場合、データは事実上圧縮されず、DatalaiQはユーザーに通知を送信します。

注意: ホット・ストレージとコールド・ストレージの場所に圧縮に関する互換性がない場合、DatalaiQは、ホットからコールドへのデータのエージアウトに追加の作業を実行する必要があります。 アクセラレーションが有効な場合、DatalaiQはエージアウトを実行する際にデータのインデックスを再作成します。 互換性のない圧縮設定は、エージアウトの際に大きなオーバーヘッドを発生させる可能性があります。 非圧縮データは透過的に圧縮されたデータと互換性がありますが、デフォルトの圧縮は非圧縮データまたは透過的に圧縮されたデータと互換性がありません。 DatalaiQは、互換性のない圧縮でも全く問題なく機能しますが、インデクサはエージアウト時に非常に大きな作業を行うことになります。


## 圧縮とレプリケーション

[レプリケーションシステム][replication.md]は、通常のウェルストレージのルールと同じものを遵守します。 レプリケートされたデータは、透過圧縮、デフォルト圧縮、または全く圧縮しないように設定することができます。 ウェル内のホットストレージとコールドストレージの位置の互換性と圧縮に関する同じルールが、複製されたデータとレプリケーションピアにも適用されます。 レプリケーション・ピアで互換性のない圧縮形式が構成されている場合、障害発生後の復元時にインデクサーの作業量が大幅に増加します。 最高のパフォーマンスを得るために、DatalaiQは、ホット、コールド、およびレプリケーション・ストアが同じ圧縮方式を使用することを推奨しています。

レプリケーションの保存場所の圧縮は、 `Disable-Compression` ディレクティブと `Enable-Transparent-Compression` ディレクティブで制御します。 デフォルトの圧縮方式はSnappyである。

## 圧縮例

ウェル全体の圧縮を無効にした保存ウェルの例:

```
[Storage-Well "network"]
	Location=/opt/gravwell/storage/network
	Cold-Location=/mnt/storage/gravwell_cold/network
	Tags=pcap
	Max-Hot-Data-GB=100
	Max-Cold-Data-GB=1000
	Delete-Frozen-Data=true
	Disable-Compression=true
```

ホットストレージのロケーションで圧縮を無効にし、コールドロケーションで透過的圧縮を有効にしたストレージウェルの例です。 この構成は互換性があると考えられ、エージアウト時に追加作業を必要としません。

```
[Storage-Well "syslog"]
	Location=/opt/gravwell/storage/syslog
	Cold-Location=/mnt/storage/gravwell_cold/syslog
	Tags=syslog
	Max-Hot-Data-GB=100
	Max-Cold-Data-GB=1000
	Delete-Frozen-Data=true
	Disable-Hot-Compression=true
	Enable-Cold-Transparent-Compression=true
```

ホットストレージのロケーションで透過的圧縮を有効にし、コールドウェルでデフォルトのユーザースペース圧縮を有効にしたストレージウェルの例です。 この構成は互換性がないとみなされ、データのエージアウト時に追加のオーバーヘッドが発生します。

```
[Storage-Well "windows"]
	Location=/opt/gravwell/storage/windows
	Cold-Location=/mnt/storage/gravwell_cold/windows
	Tags=windows
	Max-Hot-Data-GB=100
	Max-Cold-Data-GB=1000
	Delete-Frozen-Data=true
	Enable-Hot-Transparent-Compression=true
	Disable-Cold-Compression=true
```

レプリケーションストレージに透過型圧縮を使用したレプリケーション構成例です。

```
[Replication]
	Peer=indexer1
	Peer=indexer2
	Peer=indexer3
	Peer=indexer4
	Storage-Location=/mnt/storage/replication
	Enable-Transparent-Compression=true
```
