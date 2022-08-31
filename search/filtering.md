# Inline Filtering

Very frequently you will want to filter entries in a query based on some criteria--perhaps you want to remove all HTTPS flows from Netflow data, or you only want to look at traffic originating in a certain subnet, or you need to match against a particular username in Windows logs. *Inline filtering* is an efficient way to filter down many different types of DatalaiQ data.

DatalaiQ extraction modules will typically allow *extracted* items to be *filtered* at extraction time. Consider the query below, which extracts the IPv4 destination IP and the TCP destination port from packets:

```
tag=pcap packet ipv4.DstIP tcp.DstPort
```

We can add filters to, for example, only show packets destined for port 22 on IPs in the 10.0.0.0/8 subnet:

```
tag=pcap packet ipv4.DstIP ~ 10.0.0.0/8 tcp.DstPort == 22
```

Any entry whose DstIP and DstPort do not match the specified filters will be **dropped**.

## Filtering Operations & Data Types

Within the DatalaiQ search pipeline, enumerated values can be a variety of different *types*, for example strings, integers, or IP addresses. Some types cannot be filtered in certain ways--it is not particularly useful to ask if an IP address is "less than" another IP address! The filtering operations supported by DatalaiQ are below:

| Operator | Name |
|----------|------|
| == | Equal |
| != | Not equal |
| < | Less than |
| > | Greater than |
| <= | Less than or equal |
| >= | Greater than or equal |
| ~ | Subset |
| !~ | Not subset |

Most of these operations are self-explanatory, but the subset operations deserve special mention. The subset operation (~) applies to strings and IP addresses; for strings, it means "the enumerated value contains the argument", while for IP addresses it means "the IP address is within the specified subnet. Thus, `json domainName ~ "datalaiq.io"` would pass only those entries which have a JSON field named 'domainName' that contains the string "datalaiq.io". Similarly, `packet ipv4.DstIP ~ 10.0.0.0/8` would pass only those entries whose IPv4 destination IP address is in the 10.0.0.0/8 subnet.

Each enumerated value type is compatible with some filters but not others:

| Enumerated Value Type | Compatible Operators |
|-----------------------|----------------------|
| string | ==, !=, ~, !~
| byte slice | ==, !=, ~, !~
| MAC address | ==, !=
| IP address | ==, !=, ~, !~
| integer | ==, !=, <, >, <=, >=
| floating point | ==, !=, <, >, <=, >=
| boolean | ==, !=
| duration | ==, !=, <, >, <=, >=

## Filtering & Acceleration

If your data is in an [accelerated well](#!configuration/accelerators.md), inline filters can be used to speed up your queries. Only the equal operator (==) will engage acceleration; filtering for equality allows the acceleration engine to look up entries which match the desired field.

For acceleration to be engaged, your data needs to be configured to accelerate on the specified field. For instance, if you are indexing pcap data on the tcp.DstPort and tcp.SrcPort fields, the query `tag=pcap packet tcp.DstPort==22` will use the index, but `tag=pcap packet ipv4.SrcIP==10.0.0.1` will not.

## Built-in Keywords

DatalaiQ implements some special shortcuts for filtering. When filtering IP addresses by subnet, you can specify the keywords PRIVATE and MULTICAST instead of giving a single subnet, e.g. `packet ipv4.DstIP ~ PRIVATE`. The keywords map to the following subnets:

* PRIVATE: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.0/8, 224.0.0.0/24, 169.254.0.0/16, fd00::/8, fe80::/10
* MULTICAST: 224.0.0.0/4, ff00::/8

## Filtering Examples

Regex:

```
tag=syslog regex *shd.*Accepted (?P<method>\S*) for (?P<user>\S*) from (?P<ip>[0-9]+.[0-9]+.[0-9]+.[0-9]+)" user==root ip ~ "192.168"
```

AX:

```
tag=testregex,testcsv ax dstport==80 | table app src dst
```

CSV:

```
tag=default csv [5]!=stuff [4] [3]~"things" | table 3 4 5
```

IPFIX:

```
tag=ipfix ipfix destinationIPv4Address as Dst destinationTransportPort==443 | count by Dst | chart count by Dst
```

IP module:

```
tag=csv csv [2] as srcip | ip srcip ~ PRIVATE
```
