#DatalaiQアクセラレーター

DatalaiQは、フィールド抽出を行うために、エントリーがインジェストされた状態で処理することができます。 抽出されたフィールドは、各シャードに付随するアクセラレーション・ブロックに記録されます。 アクセラレーターを使用することで、ストレージのオーバーヘッドを最小限に抑えながら、スループットを劇的に向上させることができます。 アクセラレーターはウェルごとに指定され、できるだけ邪魔にならないように、かつ柔軟に設計されています。 もしデータが加速度指示と一致しない、あるいは指定されたフィールドが欠けているウェルに入ったとしても、DatalaiQは他のエントリーと同じようにそれを処理します。 アクセラレーションは、可能な限り実行されます。

私たちは、「インデクサ」や「インデックス」ではなく、「アクセラレータ」や「アクセラレーション」と呼んでいますが、これには2つの理由があります。第一に、DatalaiQはすでに "indexer "と呼ばれる非常に重要なコンポーネントを持っています。第二に、アクセラレーションはブルームフィルタによる直接的なインデックス付けによって行われるため、"インデックス "について説明することは必ずしも正確ではありません。

## アクセラレーションの基本

DatalaiQアクセラレータは、データが比較的一意である場合に最適なフィルタリング技術を使用しています。 あるフィールドの値が極めて一般的であったり、ほとんどすべてのエントリに存在する場合、それをアクセラレータの仕様に含めることはあまり意味のないことなのです。 複数のフィールドを指定してフィルタリングすることで、精度が向上し、クエリの速度も向上します。 アクセラレータの候補となるフィールドは、ユーザーが直接問い合わせるようなフィールドです。 例えば、プロセス名、ユーザー名、IPアドレス、モジュール名、その他、needle-in-the-haystackタイプのクエリで使用されるフィールドが挙げられます。

タグは、使用中の抽出モジュールに関係なく、常にアクセラレーションに含まれます。 クエリーがインラインフィルターを指定しない場合でも、1つのウェルに複数のタグがある場合、アクセラレーションシステムはクエリーの絞り込みと高速化を支援します。

ほとんどのアクセラレーション・モジュールは、ブルーム・エンジン使用時に約1～1.5%のストレージ・オーバーヘッドを生じますが、極めて低スループットのウェルでは、より多くのストレージを消費する可能性があります。 例えば、1秒間に1～10件のエントリーがある場合、アクセラレーションによって5～10%のストレージ・ペナルティが発生する可能性がありますが、1秒間に10～15万件のエントリーがあるウェルでは、ストレージ・オーバーヘッドは0.5%とごくわずかとなります。 DatalaiQアクセラレーターでは、ユーザーが指定した衝突率の調整も可能です。 ストレージに余裕がある場合、衝突率を下げると精度が上がり、クエリーが高速化しますが、ストレージのオーバーヘッドが増加します。 精度を下げると、ストレージのペナルティは軽減されますが、精度が低下し、アクセラレータの効果も低下します。 インデックスエンジンは、抽出されるフィールドの数や抽出されるデータの可変性によって、かなり多くのスペースを消費します。 たとえば、フルテキストインデックスでは、アクセラレータファイルが保存データファイルと同程度の容量を消費する可能性があります。

アクセラレータはエントリの直接データ部分で動作しなければならない（SRCフィールドで直接動作するsrcアクセラレータは例外）。

## アクセラレーションエンジン

エンジンは、抽出された加速度データを実際に保存するシステムです。 DatalaiQは2つのアクセラレーション・エンジンをサポートしています。それぞれのエンジンは、希望する取り込み速度、ディスクのオーバーヘッド、検索性能、データ量に応じて異なる利点を提供します。 アクセラレーション・エンジンは、アクセラレータ抽出器自体から完全に独立しています（アクセラレータ名設定オプションで指定します）。

デフォルトのエンジンは "index "エンジンです。 インデックス」エンジンは、あらゆる種類のクエリで高速に動作するように設計されたフルインデックスシステムです。 インデックスエンジンは通常、ブルームエンジンよりもかなり多くのディスクスペースを消費しますが、非常に大きなデータボリュームや、全データのかなりの部分に触れる可能性のあるクエリで動作する場合は、大幅に高速化されます。 大量のインデックスを持つシステムでは、インデックスエンジンが圧縮データと同程度の容量を消費することも珍しくありません。

ブルームエンジンは、ブルームフィルタを使用して、与えられたブロックにデータの一部が存在するかどうかを示します。 ブルームエンジンは通常、ディスクのオーバーヘッドをほとんど持たず、例えば特定のIPが現れたログを見つけるような、針小棒大なスタイルのクエリでうまく動作します。 ブルームエンジンは、フィルタリングされたエントリーが定期的に発生するようなフィルターではあまり効果がありません。 また、ブルームエンジンは、フルテキストアクセラレータと組み合わせた場合にもあまり適していません。


### インデックスエンジンの最適化

「インデックス」は、ファイルバックされたデータ構造を使用して、キーデータを保存し、クエリを実行します。ファイルのバックアップはメモリマップを使用して行われ、カーネルがダーティページを書き戻すのに熱心な場合、かなり乱暴になることがあります。 カーネルがダーティページを書き戻す頻度を減らすために、カーネルのダーティページパラメータを調整することが強く推奨されます。 これは、"/proc" インターフェース経由で行われ、"/etc/sysctl.conf" 設定ファイルを使って恒久的にすることができます。 以下のスクリプトは、いくつかの効率的なパラメータを設定し、それらがリブートにわたって固定されるようにします。

```
#!/bin/bash
user=$(whoami)
if [ "$user" != "root" ]; then
	echo "must run as root"
fi

echo 70 > /proc/sys/vm/dirty_ratio
echo 60 > /proc/sys/vm/dirty_background_ratio
echo 2000 > /proc/sys/vm/dirty_writeback_centisecs
echo 3000 > /proc/sys/vm/dirty_expire_centisecs

echo "vm.dirty_ratio = 70" >> /etc/sysctl.conf
echo "vm.dirty_background_ratio = 60" >> /etc/sysctl.conf
echo "vm.dirty_writeback_centisecs = 2000" >> /etc/sysctl.conf
echo "vm.dirty_expire_centisecs = 3000" >> /etc/sysctl.conf

```

## アクセラレーションの設定

アクセラレーションはウェルごとに設定されます。 各ウェルは、アクセラレーションモジュール、抽出用フィールド、衝突率、およびエントリソースフィールドを含めるオプションを指定することができます。 特定のソースで一般的にフィルタリングする場合（例えば、特定のデバイスから来るsyslogエントリだけを見る）、ソースフィールドを含めると、抽出されるフィールドとは無関係にアクセラレータの精度を高める効果的な方法が提供されます。

| Acceleration Parameter | Description | Example |
|----------|------|-------------|
| Accelerator-Name  | インジェスト時に使用するフィールド抽出モジュールを指定する。 | Accelerator-Name="json" |
| Accelerator-Args  | アクセラレーションモジュールの引数（通常は抽出するフィールド）を指定する。 | Accelerator-Args="username hostname appname" |
| Collision-Rate | ブルームエンジンを使用したアクセラレーションモジュールの精度を制御する。 0.1 から 0.000001 の間である必要があります。デフォルトは0.001です。 | Collision-Rate=0.01
| Accelerate-On-Source | 各モジュールの SRC フィールドを含めるように指定する。 これにより、CEFのようなモジュールとSRCを組み合わせることができる。 | Accelerate-On-Source=true
| Accelerator-Engine-Override | インデックス作成に使用するエンジンを指定します。 デフォルトではインデックスエンジンが使用される。 | Accelerator-Engine-Override=index

### サポートされている抽出モジュール

* [CSV](#!search/csv/csv.md)
* [Fields](#!search/fields/fields.md)
* [Syslog](#!search/syslog/syslog.md)
* [JSON](#!search/json/json.md)
* [CEF](#!search/cef/cef.md)
* [Regex](#!search/regex/regex.md)
* [Winlog](#!search/winlog/winlog.md)
* [Slice](#!search/slice/slice.md)
* [Netflow](#!search/netflow/netflow.md)
* [IPFIX](#!search/ipfix/ipfix.md)
* [Packet](#!search/packet/packet.md)
* Fulltext

### 設定例

以下は、タブ区切りのエントリ、たとえばブロログファイルの行から、2番目、4番目、5番目のフィールドを抽出する設定例です。 この例では、各Bro接続ログから送信元IP、送信先IP、送信先ポートを抽出し、高速化するものです。 bro」ウェル（この例では「bro」タグのみを含む）に入るすべてのエントリーは、取り込み中に抽出モジュールを通過します。 データの一部が高速化仕様に適合しない場合、それは保存されるが高速化されません。それはクエリーに含まれるが、適合しないエントリーがウェルに多数存在する場合、クエリーは非常に遅くなります。

```
[Storage-Well "bro"]
	Location=/opt/gravwell/storage/bro
	Tags=bro
	Accelerator-Name="fields"
	Accelerator-Args="-d \"\t\" [2] [4] [5]"
	Accelerate-On-Source=true
	Collision-Rate=0.0001
```

## アクセラレーション基礎

各アクセラレーション・モジュールは、基本的なフィールド抽出のために、コンパニオン検索モジュールと同じシンタックスを使用します。 アクセラレータは、名前の変更、フィルタリング、列挙された値に対する操作をサポートしません。 アクセラレータは第一レベルのフィルターです。 アクセラレーションモジュールは、対応する検索モジュールが動作し、等値性フィルタを実行するたびに透過的に呼び出されます。

例えば、JSONアクセラレータを使用する次のようなウェルの構成を考えてみましょう:

```
[Storage-Well "applogs"]
	Location=/opt/gravwell/storage/app
	Tags=app
	Accelerator-Name="json"
	Accelerator-Args="username hostname app.field1 app.field2"
```

もし、次のようなクエリーを発行するとしたら:

```
tag=app json username==admin app.field1=="login event" app.field2 != "failure" | count by hostname | table hostname count
```

jsonサーチモジュールは、透過的にアクセラレーションフレームワークを呼び出し、抽出された "username "と "app.field1 "の値に対して第一レベルのフィルターを提供する。 app.field2 "フィールドは、直接の等値性フィルタを使用しないため、このクエリでは高速化されません。 除外、比較、またはサブセットのチェックを行うフィルターは、高速化の対象とはなりません。

### 特定のタグにアクセラレーションを適用する

アクセラレーションシステムは、ウェルレベルまたはタグレベルでの高速化が可能です。このため、ウェルでの基本的な加速スキームを指定し、特定のタグまたはタグのグループに対して特定のアクセラレーション構成を指定することができます。

タグごとのアクセラレーションは `gravwell.conf` の `[global]` コンフィギュレーションブロックに一つ以上の `Tag-Accelerator-Definitions` を含めることで有効になります。 Tag-Accelerator-Definitions` 設定パラメータは `Tag-Accelerator` ブロックを含むファイルを指している必要があります。 Tag-Accelerator` ブロックは、タグのセットと、それらの特定のタグのためのアクセラレータの設定を指定することができます。

例えば、ウェルがデフォルトのアクセラレーションスキーマを持ち（あるいは全く持たず）、いくつかのタグが単 位指定されている場合の定義を見てみましょう。 この例では、デフォルトのウェルに加え、2つのウェルを定義する予定です。 次に、タグに特定のアクセラレータを指定するアクセラレータ定義ファイルを含めます。

備考 - 複数のファイルをインクルードするために、複数の `Tag-Accelerator-Definitions` を指定することができる。

##### gravwell.conf

```
[global]
	Ingest-Auth=foo
	Control-Auth=bar
	Ingest-Port=4023
	TLS-Ingest-Port=0
	Log-Level=ERROR

	Tag-Accelerator-Definitions="/opt/gravwell/etc/accel.defs"

[Default-Well]
	Location="/opt/gravwell/storage/default"

[Storage-Well "test"]
	Location="/opt/gravwell/storage/test"
	Tags=test*
	Accelerator-Name="fulltext"
	Accelerator-Args="-ignoreTS -ignoreUUID"

[Storage-Well "raw"]
	Location="/opt/gravwell/storage/raw"
	tags=raw*
	tags=pcap*
```

##### accel.defs

```
[Tag-Accelerator "csv things"]
	Accelerator-Name=csv
	Accelerator-Args="[0] [1] [2] [3] [4] [5] [6]"
	Tags=test1
	Tags=testspecial*
	Tags=firewall

[Tag-Accelerator "packet stuff"]
	Accelerator-Name=packet
	Accelerator-Args="ipv4.SrcIP ipv4.DstIP tcp.SrcPort tcp.DstPort"
	Tags=pcap
```

`タグアクセラレータ`の定義はウェルと同じようにタグのワイルドカードをサポートしますが、`タグアクセラレータ`の指定はウェルの指定と同じである必要はありません。

特定のタグが特定のウェルとアクセラレータにどのようにマッピングされるか、テーブルを作成してみましょう:

| Tag      | Well Assignment | Accelerator                   |
|----------|-----------------|-------------------------------|
| default  | `default`       | NONE                          |
| firewall | `default`       | `csv`                         |
| test1    | `test`          | `csv`                         |
| testfoo  | `test`          | `fulltext`                    |
| pcap     | `raw`           | `packet`                      |


#### アクセラレータのタグマッチルール

アクセラレータのタグの定義は、いくつかの特定のルールと重複することがあります。 これは、1つのタグが2つの異なる仕様にマッチすることがないウェルアサインメントルールとは対照的です。 タグはアクセラレーションの定義に対して、`hard`マッチか`soft`マッチのどちらかでマッチングされます。 ハードマッチはタグがワイルドカードなしで直接指定されたときに起こり、ソフトマッチはタグがワイルドカードグロビングパターンを使ってマッチングされたときに起こります。 例えば、`foobar` というタグは、 `foo*` というタグの指定にソフトマッチでマッチします。もし、`foobar` というタグを直接指定した場合は、ハードマッチになります。

タグのマッチングのルールは、1つのタグが複数の `soft` 仕様にマッチしたり、複数の `hard`` 仕様にマッチしてはいけないというものです。 これらのマッチングルールに違反し、未定義の動作を引き起こすような仕様を見てみましょう:

```
[Tag-Accelerator "csv things"]
	Accelerator-Name=csv
	Accelerator-Args="[0] [1] [2] [3] [4] [5] [6]"
	Tags=test1
	Tags=foobar*

[Tag-Accelerator "packet stuff"]
	Accelerator-Name=packet
	Accelerator-Args="ipv4.SrcIP ipv4.DstIP tcp.SrcPort tcp.DstPort"
	Tags=foo*
```

この2つのアクセラレータの定義には、`foobar*`と`foo*`の両方のタグマッチングパターンがあることに注意してください。これは、`foobarbaz`というタグを設定すると、両方の仕様にマッチする（2つの`ソフト`マッチ）可能性があるということです。 DatalaiQは、タグマッチが重複していることを示す通知を送信します。
次に、オーバーラップが許容される仕様について説明します
:

```
#zeekconnは圧倒的にノイズが多いので、専用のアクセラレータをセットしてください。
[Tag-Accelerator "zeek conn"]
	Accelerator-Name=fields
	Accelerator-Args=`-d "\t" [1] [2] [3] [4] [5] [6] [7] [11] [15]`
	Tags=zeekconn

# All other zeek logs can use a tuned fulltext accelerator
[Tag-Accelerator "zeek"]
	Accelerator-Name=fulltext
	Accelerator-Args=`-ignoreFloat -min 2`
	Tags=zeek* #apply to all zeek prefixed tags
```

`zeekconn` というタグは両方のアクセラレータにマッチしますが、アクセラレータの定義である `zeek conn` には特定の `hard` マッチがあり、より一般的な `zeek` の定義には `soft` マッチがあることに注意しましょう。 このタグにはハードマッチとソフトマッチがあるので、この仕様は合法であり、 `zeekconn` タグは適切なアクセラレータに割り当てられます。 ハードマッチとソフトマッチのルールは、タグのサブセットに対してアクセラレータを指定し、Zeekデータのように、特定の大量タグに対してアクセラレータを調整するのに便利です。


`タグアクセラレータ`の定義は外部ファイルとして、または直接 `gravwell.conf` ファイルに含めることができます。 以下は有効な `gravwell.conf` で、2つの外部定義ファイルと同様にいくつかの加速器の偏差を指定します:

```
[global]
	Ingest-Auth=foo
	Control-Auth=bar
	Ingest-Port=4023
	TLS-Ingest-Port=0
	Log-Level=ERROR

	Tag-Accelerator-Definitions="/opt/gravwell/etc/syslog.defs"
	Tag-Accelerator-Definitions="/opt/gravwell/etc/zeek.defs"

[Default-Well]
	Location="/opt/gravwell/storage/default"

[Storage-Well "test"]
	Location="/opt/gravwell/storage/test"
	Tags=test*
	Accelerator-Name="fulltext"
	Accelerator-Args="-ignoreTS -ignoreUUID"

[Tag-Accelerator "csv things"]
	Accelerator-Name=csv
	Accelerator-Args="[0] [1] [2] [3] [4] [5] [6]"
	Tags=test1
	Tags=foobar*

[Tag-Accelerator "packet stuff"]
	Accelerator-Name=packet
	Accelerator-Args="ipv4.SrcIP ipv4.DstIP tcp.SrcPort tcp.DstPort"
	Tags=foo*
```

## Fulltext

フルテキストアクセラレータは、テキストログ内の単語をインデックス化するために設計されており、最も柔軟なアクセラレータオプションと考えられています。 他の多くの検索モジュールは、クエリを実行する際にフルテキストアクセラレータを呼び出すことをサポートしています。 しかし、フルテキストアクセラレータを使用するための主要な検索モジュールは、`-w` フラグを指定した [grep](/search/grep/grep.md) モジュールです。 Unix の grep ユーティリティと同様に、 `grep -w` は指定されたフィルタがバイトのサブセットではなく、ワードに期待されることを指定します。 words foo bar baz` と検索をかけると、foo, bar, baz という単語を探し、フルテキストアクセラレータを使用します。

フルテキストアクセラレーターは最も柔軟性が高い反面、最もコストがかかります。 これは、フルテキスト・アクセラレーターがすべてのエントリーのほぼすべての要素に対してインデックスを作成しているためです。

### Fulltext引数

フルテキストアクセラレータは、インデックスを作成するデータの種類を絞り込んだり、ストレージのオーバーヘッドが大きいもののクエリ時にはあまり役に立たないフィールドを削除したりするための、いくつかのオプションをサポートしています。

| Argument | Description | Example | Default State |
|----------|-------------|---------|---------------|
| -acceptTS | デフォルトでは、フルテキストアクセラレータはデータ中のタイムスタンプを識別し、無視しようとします。このフラグはその動作を無効にし、タイムスタンプをインデックス化することを可能にします。 | `-acceptTS` | DISABLED |
| -acceptFloat | デフォルトでは、フルテキストアクセラレータは浮動小数点数を識別して無視しようとします。このフラグを設定すると、その動作が無効になり、浮動小数点数のインデックスが作成できるようになります。 | `-acceptFloat` | DISABLED |
| -min | 抽出されたトークンの長さが少なくともXバイトであることを要求する。 これは、"is" や "I" のような非常に小さい単語でのインデックス作成を防ぐのに役立ちます。 | `-min 3` | DISABLED |
| -max | 抽出されたトークンの長さがXバイト未満であることを要求する。 これは、決してクエリされることのない非常に大きな「ブロブ」に対してインデックスを作成するのを防ぐのに役立ちます。 | `-max 256` | DISABLED |
| -ignoreUUID | UUID/GUID 値を無視するフィルタを有効にする。 ログによっては、すべてのエントリに対して UUID を生成するものがあります。これは、かなりのインデックス作成のオーバーヘッドを発生させ、ほとんど価値を提供しません。 | `-ignoreUUID` | DISABLED |
| -ignoreTS | アクセラレーション時にタイムスタンプを識別し、無視する。タイムスタンプは頻繁に変更されるため、肥大化の重大な原因となる可能性があります。フルテキストアクセラレータは、デフォルトでタイムスタンプを無視します。 | `-ignoreTS` | ENABLED |
| -ignoreFloat | 浮動小数点数を無視する。浮動小数点数がフィルタに使用されるログでは、 `-accptFloat` を使用することができる。 | `-acceptFloat` | ENABLED |
| -maxInt | 特定のサイズ以下の整数のみをインデックス化するフィルタを有効にします。 これは、HTTP アクセスログのようなデータのインデックスを作成する際に有効です。 リターンコードはインデックス化したいが、レスポンスタイムやデータサイズはインデックス化したくない場合などに有効です。 | `-maxInt 1000` | DISABLED |

備考: インデックスエンジンを使用する際には、インデックスを劇的に肥大化させる可能性があるため、 `-acceptTS` と `-acceptFloat` フラグを有効にする前に、自分のデータをよく理解しておく必要があります。 ブルームエンジンは、タイムスタンプや浮動小数点数などの直交するデータにはあまり影響を受けません。

### ウェルの設定例

以下のウェル設定例は `index` エンジンを使ってfulltextアクセラレーションを行うものです。 タイムスタンプや UUID を識別・無視し、すべてのトークンの長さが最低でも 2 バイトであることを要求しています。

```
[Default-Well]
	Location=/opt/gravwell/storage/default
	Accelerator-Name="fulltext"
	Accelerator-Args="-ignoreTS -ignoreUUID -min 2"
```

## JSON

JSONアクセラレータモジュールは、アクセラレータ名「json」で指定し、JSONモジュールと全く同じ構文でフィールドの抽出を行います。 フィールドの抽出については、[JSON検索モジュール](#!search/json/json.md)のセクションを参照してください。

###  ウェルの設定例

```
[Storage-Well "applogs"]
	Location=/opt/gravwell/storage/app
	Tags=app
	Accelerator-Name="json"
	Accelerator-Args="username hostname \"strange-field.with.specials\".subfield"
```

## Syslog

syslog アクセラレータは、RFC5424 に準拠した syslog メッセージで動作するように設計されています。 フィールド抽出の詳細については [syslog検索モジュール](#!search/syslog/syslog.md) のセクションを参照してください。

### ウェルの設定例

```
[Storage-Well "syslog"]
	Location=/opt/gravwell/storage/syslog
	Tags=syslog
	Accelerator-Name="syslog"
	Accelerator-Args="Hostname Appname MsgID valueA valueB"
```

## CEF

CEF アクセラレータは CEF ログメッセージ上で動作するように設計されており、検索モジュールと同じように柔軟性があります。 フィールド抽出の詳細については [CEF検索モジュール](#!search/cef/cef.md) セクションを参照してください。

### ウェルの設定例

```
[Storage-Well "ceflogs"]
	Location=/opt/gravwell/storage/cef
	Tags=app1
	Accelerator-Name="cef"
	Accelerator-Args="DeviceVendor DeviceProduct Version Ext.Version"
```

## Fields

fields アクセラレータは、CSV や TSV など、あらゆる区切りデータ形式に対して操作することができます。 fields アクセラレータでは、search モジュールと同じようにデリミターを指定することができます。 フィールドの抽出については、[fields検索モジュール](#!search/fields/fields.md) のセクションを参照してください。

### ウェルの設定例

この設定では、カンマで区切られたエントリから4つのフィールドを抽出します。区切り文字を指定するために `-d` フラグを使用していることに注意してください。

```
[Storage-Well "security"]
	Location=/opt/gravwell/storage/seclogs
	Tags=secapp
	Accelerator-Name="fields"
	Accelerator-Args="-d \",\" [1] [2] [5] [3]"
```

## CSV

CSVアクセラレータは、カンマ区切り値のデータに対して、周囲のホワイトスペースや二重引用符を自動的に削除して動作するように設計されています。 カラム抽出の詳細については、[CSV検索モジュール](#!search/csv/csv.md)のセクションを参照してください。

### ウェルの設定例

```
[Storage-Well "security"]
	Location=/opt/gravwell/storage/seclogs
	Tags=secapp
	Accelerator-Name="csv"
	Accelerator-Args="[1] [2] [5] [3]"
```

## Regex

正規表現アクセラレータは、非標準のデータ形式を扱うために、インジェスト時に複雑な抽出を可能にします。 正規表現は低速な抽出形式の一つであるため、特定のフィールドで高速化することにより、クエリの性能を大幅に向上させることができます。

### ウェルの設定例

```
[Storage-Well "webapp"]
	Location=/opt/gravwell/storage/webapp
	Tags=webapp
	Accelerator-Name="regex"
	Accelerator-Args="^\\S+\\s\\[(?P<app>\\w+)\\]\\s<(?P<uuid>[\\dabcdef\\-]+)>\\s(?P<src>\\S+)\\s(?P<srcport>\\d+)\\s(?P<dst>\\S+)\\s(?P<dstport>\\d+)\\s(?P<path>\\S+)\\s"
```

備考: gravwell.confファイルに正規表現を指定する際は、バックスラッシュ「 \ 」をエスケープすることを忘れないようにしましょう。 正規表現の引数 'Ⓐ' は 'Ⓑ' となります。

## Winlog

winlogモジュールは、おそらく最も遅い検索モジュールです。 Windows ログスキーマと組み合わされた XML データの複雑さは、モジュールが非常に冗長でなければならないことを意味し、結果として、かなり悪いパフォーマンスになっています。 これは、Windowsログデータの高速化が、最も重要なパフォーマンスの最適化であることを意味し、winlogモジュールで加速されていない数百万または数十億のエントリーを処理することは、耐え難いほど遅くなります。 アクセラレーターは、すべてのデータに対してwinlog検索モジュールを呼び出すことなく、必要な特定のログ・エントリーを絞り込むのに役立ちます。 しかし、winlogデータの高速化は、単に検索時間から取り込み時間へと処理をシフトするだけです。つまり、高速化が有効になるとWindowsログの取り込みが遅くなるので、winlog-acceleratedウェルへの取り込み時にDatalaiQの典型的な取り込み速度、1秒間に数十万エントリーを期待しないでください。

### ウェルの設定例

```
[Storage-Well "windows"]
	Location=/opt/gravwell/storage/windows
	Tags=windows
	Accelerator-Name="winlog"
	Accelerator-Args="EventID Provider Computer TargetUserName SubjectUserName"
```

注意: winlogアクセラレータは寛容です('-or'フラグは暗示的です)。 したがって、検索をフィルタリングするために使用するフィールドを指定します。たとえ、2 つのフィールドが同じエントリに存在しないとしてもです。

## Netflow

[netflow](#!search/netflow/netflow.md) モジュールは、netflow V5 フィールドでの高速化と、大量の netflow データでのクエリーの高速化を可能にします。 netflowモジュールは非常に高速で、データは非常にコンパクトですが、非常に大きなnetflowデータボリュームがある場合は、高速化を行うことが有益な場合もあります。 netflowモジュールはnetflowの直接フィールドのいずれかを使用できますが、ピボット・ヘルパー・フィールドは使用できません。 つまり、`IP`ではなく `Src` または `Dst` を指定しなければなりません。 加速度引数には `IP` と `Port` フィールドを指定することはできません。

備考: ヘルパー抽出の `Timestamp` と `Duration` はアクセラレータで使用することはできません。

### ウェルの設定例

この設定例では、`bloom`エンジンを使用し、送信元と送信先のIP/Portのペアとプロトコルで高速化を行っています。

```
[Storage-Well "netflow"]
	Location=/opt/gravwell/storage/netflow
	Tags=netflow
	Accelerator-Name="netflow"
	Accelerator-Args="Src Dst SrcPort DstPort Protocol"
	Accelerator-Engine-Override="bloom"
```

## IPFIX

[ipfix](#!search/ipfix/ipfix.md) モジュールは、IPFIX フォーマットのレコードに対するクエリーを高速化することができます。このモジュールは '通常の' IPFIX フィールドのいずれでも高速化できますが、ピボット・ヘルパー・フィールドは高速化できません。つまり、 `port` ではなく `sourceTransportPort` や `destinationTransportPort` を、また `ip` ではなく `src`/`dst` を指定しなければなりません。

### ウェルの設定例

この設定例では、ネットフローのセクションで示した例と同様に、フローのIPプロトコルと同様にソース/デスティネーションIP/ポートのペアで加速するために `index` エンジンを使用します。

```
[Storage-Well "ipfix"]
	Location=/opt/gravwell/storage/ipfix
	Tags=ipfix
	Accelerator-Name="ipfix"
	Accelerator-Args="src dst sourceTransportPort destinationTransportPort protocolIdentifier"
	Accelerator-Engine-Override=index
```

## Packet

[packet](#!search/packet/packet.md) モジュールは、同名の検索モジュールと同じ構文で、生のパケットキャプチャを高速化することができます。 検索モジュールと比較して、パケットアクセラレータの適用方法に微妙な、しかし重要な違いがあります: アクセラレータは重複するレイヤーを使用できます。 つまり、UDPとTCPの両方の項目を指定し、処理するパケットに応じて適切なフィールドを抽出することができるのです。

IPv4、IPv6、TCP、UDP、ICMPなど、すべてを同時に高速化するようにうまく設定することが可能です。 パケットアクセラレータは、指定されたフィールドを暗黙のフィルタとして扱いません。

つまり、`IP` や `Port` といった便利なエキストラクタは使えないということです。 つまり、`IP` や `Port` のような便利な抽出器は使えないということです。何を高速化したいのかを正確に指定しなければなりません。

### ウェルの設定例

```
[Storage-Well "packets"]
	Location=/opt/gravwell/storage/pcap
	Tags=pcap
	Accelerator-Name="packet"
	Accelerator-Args="ipv4.SrcIP ipv4.DstIP ipv6.SrcIP ipv6.DstIP tcp.SrcPort tcp.DstPort udp.SrcPort udp.DstPort"
```

## SRC

src アクセラレータは、エントリのソースフィールドのみを高速化する場合に使用します。 しかし、基本的には "Accelerate-On-Source" フラグを有効にし、クエリで src 検索モジュールを使用すれば、src アクセラレータを他のアクセラレータと組み合わせることが可能です。 フィルタリングの詳細については [src検索モジュール](#!search/src/src.md) を参照してください。

### ウェルの設定例

```
[Storage-Well "applogs"]
	Location=/opt/gravwell/storage/app
	Tags=app
	Accelerator-Name="src"
```

### SRCを組み合わせたウェルの構成とクエリーの例

```
[Storage-Well "applogs"]
	Location=/opt/gravwell/storage/app
	Tags=app
	Accelerator-Name="fields"
	Accelerator-Args="-d \",\" [1] [2] [5] [3]"
	Accelerate-On-Source=true
```

次のクエリは、fields アクセラレータと src アクセラレータの両方を呼び出して、特定のソースから来る特定のログタイプを指定します。

```
tag=app src dead::beef | fields -d "," [1]=="security" [2]="process" [5]="domain" [3] as processname | count by processname | table processname count
```

## アクセラレーション性能とベンチマーク

高速化の利点と欠点を理解するためには、それがストレージの使用、インジェストパフォーマンス、クエリパフォーマンスにどのように影響するかを見るのが一番です。 ここでは、[github](https://github.com/kiritbasu/Fake-Apache-Log-Generator)で公開されているジェネレーターを使用して生成された、Apacheの複合アクセスログを使用します。 データセットは、約24時間に渡る1000万件のApache複合アクセスログで、データ総量は2.1GBです。 我々は、4つの異なる構成で4つのウェルを定義します。 このデータには、返されたバイト数のような、インデックスを作成する意味があまりないパラメータがたくさんあるため、かなり単純なアプローチでインデックスを作成することになります。



| Well | Extractor | Engine | Description |
|------|-----------|--------|-------------|
| raw  | None | None | アクセラレータが全くない完全な生ウェル |
| fulltext | fulltext | index | インデックスエンジンを使用し、すべての単語に対して全文高速化を行うfulltextアクセラレーションウェル |
| regexindex | regex | index | 正規表現抽出器とインデックスエンジンを使ってよく加速されたもの。各パラメータが抽出され、インデックスが作成される |
| regexbloom | regex | bloom | regexindexウェルと同じ抽出器を持つが、ブルームエンジンを持つウェル。 各パラメータは抽出され、ブルームフィルタに追加される。 |

ウェルの構成は:

```
[Storage-Well "raw"]
	Location=/opt/gravwell/storage/raw
	Tags=raw
	Enable-Transparent-Compression=true

[Storage-Well "fulltext"]
	Location=/opt/gravwell/storage/fulltext
	Tags=fulltext
	Enable-Transparent-Compression=true
	Accelerator-Name=fulltext
	Accelerator-Args="-ignoreTS -min 2"

[Storage-Well "regexindex"]
	Location=/opt/gravwell/storage/regexindex
	Tags=regexindex
	Enable-Transparent-Compression=true
	Accelerator-Name=regex
	Accelerator-Engine-Override=index
	Accelerator-Args="^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"

[Storage-Well "regexbloom"]
	Location=/opt/gravwell/storage/regexbloom
	Tags=regexbloom
	Enable-Transparent-Compression=true
	Accelerator-Name=regex
	Accelerator-Engine-Override=bloom
	Accelerator-Args="^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
```

### テストマシン

クエリー、インジェスト、ストレージのパフォーマンス特性は、各データセットやハードウェアプラットフォームによって異なりますが、このテストでは以下のハードウェアを使用しています。:

| Component | Description |
|-----------|-------------|
| CPU       |  AMD Ryzen 1700 |
| Memory    | 16GB DDR4-2400 |
| Disk      | Samsung 960 EVO NVME |
| Filesystem | BTRFS with zstd transparent compression |

このテストはDatalaiQのバージョン `3.1.5`を使用して行われました。

### インジェストパフォーマンス

インジェストには、singleFileインジェスターを使用します。 singleFileインジェスターは一つの改行区切りファイルをインジェストするように設計されており、その都度タイムスタンプを導出するようになっています。 このインジェスターはタイムスタンプを生成するため、ある程度のCPUリソースを必要とします。 singleFileインジェスターは、私たちの[GitHubページ](https://github.com/gravwell/ingesters/)で公開されています。singleFileインジェスターの正確な起動方法は以下の通りです:

```
./singleFile -i apache_fake_access_10M.log -clear-conns 172.17.0.3 -block-size 1024 -timeout=10 -tag-name=fulltext
```

|  Well      | Entries Per Second | Data Rate  |
|------------|--------------------|------------|
| raw        | 313.54 KE/s        | 65.94 MB/s |
| regexbloom | 112.91 KE/s        | 23.75 MB/s |
| regexindex | 57.58  KE/s        | 12.11 MB/s |
| fulltext   | 26.37  KE/s        |  5.55 MB/s |

インジェスト性能から、フルテキストアクセラレーションシステムがインジェスト性能を劇的に低下させていることがわかります。 5.55MB/sはインジェスト性能が悪いように見えますが、これでも約480GBのデータと1日あたり23億エントリーになることは特筆すべきことです。

### ストレージ使用率

インジェストパフォーマンスと追加メモリー要件以外では、アクセラレーションを有効にすることによる主なペナルティは使用量です。 各抽出方法のインデックス・エンジンは50%以上のストレージを消費し、ブルーム・エンジンはさらに4%消費していることがわかります。 ストレージの使用量は消費されるデータに大きく依存しますが、平均すると、インデックス作成システムはかなり多くのストレージを消費します。

|  Well      | Used Storage | Diff From Raw |
|------------|--------------|---------------|
| raw        | 2.49GB       | 0%            |
| fulltext   | 3.83GB       | 53%           |
| regexindex | 3.76GB       | 51%           |
| regexbloom | 2.60GB       | 4%            |

### クエリパフォーマンス

クエリパフォーマンスの違いを示すために、sparseとdenseに分類される2つのクエリを実行します。 疎なクエリは、データセット内の特定のIPを探し、ほんの一握りのエントリを返します。 密なクエリでは、データセット内で適度に一般的な特定のURLを探します。 クエリを単純化するために、アクセラレーションシステムと一致するregexindexおよびregexbloomタグ用のaxモジュールをインストールします。 密なクエリではデータセットの約12%のエントリが検索され、疎なクエリでは約0.01%が検索されます。

スパースクエリーとデンスクエリーは:

```
ax ip=="106.218.21.57"
ax url=="/list" method | count by method | chart count by method
```

各クエリーを実行する前に、rootで以下のコマンドを実行して、システムキャッシュを削除します:

```
echo 3 > /proc/sys/vm/drop_caches
```

|  Well      | Query Type | Query Time | Processed Entries Percentage | Speedup |
|------------|------------|------------|------------------------------|---------|
| raw        | sparse     | 71.5s      |  100%                        | 0X      |
| regexbloom | sparse     | 397ms      |  0.00389%                    | 180X    |
| regexindex | sparse     | 190ms      |  0.000001%                   | 386X    |
| fulltext   | sparse     | 195ms      |  0.000001%                   | 376X    |
| raw        | dense      | 73.5s      |  100%                        | 0X      |
| regexbloom | dense      | 71.5s      |  100%                        | 1.02X   |
| regexindex | dense      | 14.2s      |  13%                         | 5.17X   |
| fulltext   | dense      | 24.6s      |  30%                         | 2.98X   |

備考: 正規表現検索モジュール/autoextractorはフルテキストアクセラレータと完全に互換性があるわけではないので、アクセラレータを使用するためにクエリを少し変更する必要があります。 それらは ``grep -w "106.218.21.57"`` と ``grep -w list | ax url=="/list" method | count by method | chart count by method`` である。

#### Fulltext

上記のベンチマークを見ると、フルテキストアクセラレータには大きな取り込みと保存のペナルティがあり、例のクエリではその費用を正当化できないことがよくわかる。 データがタブ区切り、csv、json などの完全にトークンをベースとしたもので、すべてのトークンが完全に独立している場合（単一の単語、数値、IP、値など）、フルテキストアクセラレータはあまり意味をなさない。 しかし、データに複雑な要素がある場合、フルテキストアクセラレータは他のアクセラレータではできないことを行うことができる。 ここでは、Apache のアクセスログを組み合わせたクエリを使って、フルテキストアクセラレータが真価を発揮できるようなクエリを見てみましょう。

URL のサブコンポーネントを調べて、PowerPC Macintosh コンピュータを使用して `/apps` サブディレクトリをブラウズしているユーザのチャートを取得するつもりです。 上記の例の正規表現は Apache のログ中の完全なフィールドをインデックスします。 これらのフィールドの一部を掘り下げて加速することはできませんが、フルテキストは可能です。

公平を期すため、フルテキストインデクサとその他のクエリの両方に最適化しましたが、どちらのクエリもどちらのデータセットでも機能します。

フルテキストアクセラレータに最適化されたクエリ:
```
grep -s -w apps Macintosh PPC | ax url~"/apps" useragent~"Macintosh; U; PPC" | count | chart count
```

fulltext以外に最適化されたクエリ:
```
ax url~"/apps" useragent~"Macintosh; U; PPC" | count | chart count
```

この結果は、なぜフルテキストがストレージとインジェストのペナルティに見合うことが多いかを示しています。:

|  Well      | Query Time | Speedup |
|------------|------------|---------|
| raw        | 71.7s      | 0X      |
| regexbloom | 72.6s      | ~0X     |
| regexindex | 72.6       | ~0X     |
| fulltext   | 5.73s      | 12.49X  |


#### AXモジュール

4つのタグのAX定義ファイルは以下の通りです。詳細は[AX]()のドキュメントを参照してください:

```
[[extraction]]
  tag = 'regexindex'
  module = 'regex'
  params = "^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
  name = 'apacheindex'
  desc = 'apache index'

[[extraction]]
  tag = 'regexbloom'
  module = 'regex'
  params = "^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
  name = 'apachebloom'
  desc = 'apache bloom'

[[extraction]]
  tag = 'fulltext'
  module = 'regex'
  params = "^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
  name = 'apachefulltext'
  desc = 'apache fulltext'

[[extraction]]
  tag = 'raw'
  module = 'regex'
  params = "^(?P<ip>\\S+) (?P<ident>\\S+) (?P<username>\\S+) \\[([\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<method>\\S+)\\s?(?P<url>\\S+)?\\s?(?P<proto>\\S+)?\" (?P<resp>\\d{3}|-) (?P<bytes>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
  name = 'apacheraw'
  desc = 'apache raw'
```
