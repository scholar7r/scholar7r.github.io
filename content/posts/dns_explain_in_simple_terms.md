+++
title = '深入浅出 DNS'
date = 2025-11-24T11:06:32+08:00
draft = false
author = "scholar7r"
authorTwitter = "scholar7r"
+++

域名系统 (Domain Name System) 用作将域名和 IP 地址相互映射的一个分布式数据库，能够使人们更方便地访问互联网。DNS 使用 TCP 和 UDP 端口 53。目前对于每个一级域名长度的限制是 63 个字符，域名总长度不得超过 253 个字符。

<!--more-->

## 记录类型

DNS 的记录称之为资源记录 (Resource record)，常见类型有：

- 主机记录 (A 记录)： `RFC 1035` 定义，将特定的主机名映射到对应主机的 IP 地址上。
- 别名记录 (CNAME 记录)：`RFC 1035` 定义，将某个别名指向到某个 A 记录上，这样就不用再次为某一个新名字指定一条新的 A 记录。
- IPv6 主机记录 (AAAA 记录)：`RFC 3596` 定义，与 A 记录对应，将特定的主机名映射到对应主机的 IPv6 地址。
- 服务位置记录 (SRV 记录)：`RFC 2782` 定义，用于提供特定服务的服务器的位置。
- 域名服务器记录 (NS 记录)：用来指定该域名由哪个DNS服务器来进行解析。
- NAPTR 记录：`RFC 3403` 定义，提供正则表达式方式去映射域名。

在开源项目 [miekg/dns](https://codeberg.org/miekg/dns) 中有如下定义：

```go
const (
 TypeNone       uint16 = 0
 TypeA          uint16 = 1
 TypeNS         uint16 = 2
 TypeMD         uint16 = 3
 TypeMF         uint16 = 4
 TypeCNAME      uint16 = 5
 TypeSOA        uint16 = 6
 TypeMB         uint16 = 7
 TypeMG         uint16 = 8
 TypeMR         uint16 = 9
 TypeNULL       uint16 = 10
 TypePTR        uint16 = 12
 TypeHINFO      uint16 = 13
 TypeMINFO      uint16 = 14
 TypeMX         uint16 = 15
 TypeTXT        uint16 = 16
 TypeRP         uint16 = 17
 TypeAFSDB      uint16 = 18
 TypeX25        uint16 = 19
 TypeISDN       uint16 = 20
 TypeRT         uint16 = 21
 TypeNSAPPTR    uint16 = 23
 TypeSIG        uint16 = 24
 TypeKEY        uint16 = 25
 TypePX         uint16 = 26
 TypeGPOS       uint16 = 27
 TypeAAAA       uint16 = 28
 TypeLOC        uint16 = 29
 TypeNXT        uint16 = 30
 TypeEID        uint16 = 31
 TypeNIMLOC     uint16 = 32
 TypeSRV        uint16 = 33
 TypeATMA       uint16 = 34
 TypeNAPTR      uint16 = 35
 TypeKX         uint16 = 36
 TypeCERT       uint16 = 37
 TypeDNAME      uint16 = 39
 TypeOPT        uint16 = 41
 TypeAPL        uint16 = 42
 TypeDS         uint16 = 43
 TypeSSHFP      uint16 = 44
 TypeIPSECKEY   uint16 = 45
 TypeRRSIG      uint16 = 46
 TypeNSEC       uint16 = 47
 TypeDNSKEY     uint16 = 48
 TypeDHCID      uint16 = 49
 TypeNSEC3      uint16 = 50
 TypeNSEC3PARAM uint16 = 51
 TypeTLSA       uint16 = 52
 TypeSMIMEA     uint16 = 53
 TypeHIP        uint16 = 55
 TypeNINFO      uint16 = 56
 TypeRKEY       uint16 = 57
 TypeTALINK     uint16 = 58
 TypeCDS        uint16 = 59
 TypeCDNSKEY    uint16 = 60
 TypeOPENPGPKEY uint16 = 61
 TypeCSYNC      uint16 = 62
 TypeZONEMD     uint16 = 63
 TypeSVCB       uint16 = 64
 TypeHTTPS      uint16 = 65
 TypeDSYNC      uint16 = 66
 TypeSPF        uint16 = 99
 TypeUINFO      uint16 = 100
 TypeUID        uint16 = 101
 TypeGID        uint16 = 102
 TypeUNSPEC     uint16 = 103
 TypeNID        uint16 = 104
 TypeL32        uint16 = 105
 TypeL64        uint16 = 106
 TypeLP         uint16 = 107
 TypeEUI48      uint16 = 108
 TypeEUI64      uint16 = 109
 TypeNXNAME     uint16 = 128
 TypeURI        uint16 = 256
 TypeCAA        uint16 = 257
 TypeAVC        uint16 = 258
 TypeAMTRELAY   uint16 = 260
 TypeRESINFO    uint16 = 261
)
```

## DiG 查询

尝试使用 DiG 进行查询 `scholar7r.cn` 域名，结果剔除了一些 IPv6 请求的错误和 DNSSEC 返回的相关数据：

```
; <<>> DiG 9.20.15 <<>> +trace scholar7r.cn
;; global options: +cmd
.   2829 IN NS g.root-servers.net.
.   2829 IN NS e.root-servers.net.
.   2829 IN NS l.root-servers.net.
.   2829 IN NS j.root-servers.net.
.   2829 IN NS h.root-servers.net.
.   2829 IN NS a.root-servers.net.
.   2829 IN NS d.root-servers.net.
.   2829 IN NS m.root-servers.net.
.   2829 IN NS f.root-servers.net.
.   2829 IN NS c.root-servers.net.
.   2829 IN NS b.root-servers.net.
.   2829 IN NS i.root-servers.net.
.   2829 IN NS k.root-servers.net.
;; Received 444 bytes from 223.5.5.5#53(223.5.5.5) in 6 ms

cn.   172800 IN NS ns.cernet.net.
cn.   172800 IN NS c.dns.cn.
cn.   172800 IN NS a.dns.cn.
cn.   172800 IN NS b.dns.cn.
cn.   172800 IN NS d.dns.cn.
cn.   172800 IN NS e.dns.cn.
;; Received 725 bytes from 2001:dc3::35#53(m.root-servers.net) in 67 ms

scholar7r.cn.  86400 IN NS aria.ns.cloudflare.com.
scholar7r.cn.  86400 IN NS patryk.ns.cloudflare.com.
;; Received 672 bytes from 203.119.29.1#53(e.dns.cn) in 48 ms

scholar7r.cn.  300 IN A 104.21.51.46
scholar7r.cn.  300 IN A 172.67.221.101
;; Received 181 bytes from 108.162.192.68#53(aria.ns.cloudflare.com) in 83 ms

```

上面是一个递归的请求，从根域名服务器开始一步一步请求到 `scholar7r.cn` 解析的 IP 地址。

1. 客户端发起请求查询根服务器。
1. 根服务器返回 13 个（A - M）的根服务器记录
1. 选取一个根服务器查询 `cn.` 顶级域名
1. 返回权威域名的名称服务器记录
1. 选取一个权威域名服务器查询 `scholar7r.cn`
1. 权威域名服务器返回 Cloudflare 名称服务器记录
1. 选举一个名称服务器查询 `scholar7r.cn`
1. 返回结果 IP 地址

```goat
 ┌──────┐                 ┌───────┐┌───┐┌─────────────┐┌─────────────────────┐
 │Client│                 │Root(.)││TLD││Authoritative││Cloudflare Nameserver│
 └──┬───┘                 └───┬───┘└─┬─┘└──────┬──────┘└──────────┬──────────┘
    │                         │      │         │                  │           
    │            .            │      │         │                  │           
    │────────────────────────>│      │         │                  │           
    │                         │      │         │                  │           
    │[Root Nameserver Records]│      │         │                  │           
    │<────────────────────────│      │         │                  │           
    │                         │      │         │                  │           
    │              cn.        │      │         │                  │           
    │───────────────────────────────>│         │                  │           
    │                         │      │         │                  │           
    │    [TLD Nameserver Records]    │         │                  │           
    │<───────────────────────────────│         │                  │           
    │                         │      │         │                  │           
    │              scholar7r.cn.     │         │                  │           
    │─────────────────────────────────────────>│                  │           
    │                         │      │         │                  │           
    │   [Domain's Authoritative Nameservers]   │                  │           
    │<─────────────────────────────────────────│                  │           
    │                         │      │         │                  │           
    │                       [scholar7r.cn]     │                  │           
    │────────────────────────────────────────────────────────────>│           
    │                         │      │         │                  │           
    │                         │   IP │         │                  │           
    │<────────────────────────────────────────────────────────────│           
 ┌──┴───┐                 ┌───┴───┐┌─┴─┐┌──────┴──────┐┌──────────┴──────────┐
 │Client│                 │Root(.)││TLD││Authoritative││Cloudflare Nameserver│
 └──────┘                 └───────┘└───┘└─────────────┘└─────────────────────┘
```

`scholar7r.cn` 域名只是解析了 A 记录，没有其他类型的解析，如果请求查询其他类型的解析，会发生什么事情？

```
; <<>> DiG 9.20.15 <<>> +trace scholar7r.cn CNAME
;; global options: +cmd
.   36352 IN NS m.root-servers.net.
.   36352 IN NS a.root-servers.net.
.   36352 IN NS b.root-servers.net.
.   36352 IN NS c.root-servers.net.
.   36352 IN NS d.root-servers.net.
.   36352 IN NS e.root-servers.net.
.   36352 IN NS f.root-servers.net.
.   36352 IN NS g.root-servers.net.
.   36352 IN NS h.root-servers.net.
.   36352 IN NS i.root-servers.net.
.   36352 IN NS j.root-servers.net.
.   36352 IN NS k.root-servers.net.
.   36352 IN NS l.root-servers.net.
;; Received 239 bytes from 223.5.5.5#53(223.5.5.5) in 7 ms

cn.   172800 IN NS a.dns.cn.
cn.   172800 IN NS b.dns.cn.
cn.   172800 IN NS c.dns.cn.
cn.   172800 IN NS d.dns.cn.
cn.   172800 IN NS e.dns.cn.
cn.   172800 IN NS ns.cernet.net.
;; Received 723 bytes from 2001:500:a8::e#53(e.root-servers.net) in 206 ms

scholar7r.cn.  86400 IN NS aria.ns.cloudflare.com.
scholar7r.cn.  86400 IN NS patryk.ns.cloudflare.com.
;; Received 672 bytes from 2001:dc7:1000::1#53(d.dns.cn) in 18 ms

scholar7r.cn.  1800 IN SOA aria.ns.cloudflare.com. dns.cloudflare.com. 2389274792 10000 2400 604800 1800
;; Received 361 bytes from 2606:4700:50::adf5:3a44#53(aria.ns.cloudflare.com) in 88 ms

```

返回了一个 SOA 记录，记录对应一个权威名称服务器。这是域名不存在或者是查询失败的常见返回结果。

## 报文格式

左侧为 DNS 报文格式，右侧为 Header 部分的详细分解。

```goat
                                                 1  1  1  1  1  1
+------------+     0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
| Header     |   +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
+------------+   |                      ID                       |
| Question   |   +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
+------------+   |QR|   Opcode  |AA|TC|RD|RA|   Z    |   RCODE   |
| Answer     |   +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
+------------+   |                    QDCOUNT                    |
| Authority  |   +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
+------------+   |                    ANCOUNT                    |
| Additional |   +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
+------------+   |                    NSCOUNT                    |
                 +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
                 |                    ARCOUNT                    |
                 +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

| 名称 | 描述 |
| ---- | ---- |
| ID | 由客户端生成的 16 位标识符，请求和回复使用相同的标识符。 |
| QR | 表示该报文为查询 <sup>(0)</sup> 或回复 <sup>(1)</sup>，占 1 位。 |
| OPCODE | 表示该查询的类型，一般情况下使用 0 表示该查询是一个标准查询，占 4 位。 |
| AA | 表示该查询的服务器是否为所查询域名的权威域名服务器，占 1 位。 |
| TC | 表示报文是否截断，占 1 位。 |
| RD | 表示该查询是否要求递归查询。占 1 位。 |
| RA | 表示递归查询是否可用。 |
| Z | 未来使用保留。 |
| RCODE | 回复代码：<br> - 0 无错误<br> - 1 格式错误：名称服务器无法解释查询 <br>- 2 服务端错误：名称服务器由于内部错误无法处理查询<br> - 3 名称错误：表示所查询的域名不存在<br> - 4 未实现：名称服务器不支持查询的类型<br> - 5 拒绝：名称服务器由于政策原因等无法执行操作 |
| QDCOUNT | 无符号的16位整数，指定问题部分中的条目数。  |
| ANCOUNT | 无符号的16位整数，指定应答部分中的资源记录数。  |
| NSCOUNT | 无符号的16位整数，指定权威记录部分中名称服务器资源记录的数量。  |
| ARCOUNT | 无符号的16位整数，指定附加记录部分中的资源记录数。 |

Question 部分

```goat
                                1  1  1  1  1  1
  0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                                               |
/                     QNAME                     /
/                                               /
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     QTYPE                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     QCLASS                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

| 名称 | 描述 |
| ---- | ---- |
| QNAME | 所查询的域名。 |
| QTYPE | 查询记录的类型。|
| QCLASS | 使用 0x0001，表示网络地址。 |

Answer 部分

```goat
                                1  1  1  1  1  1
  0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                                               |
/                     NAME                      /
/                                               /
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     TYPE                      |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     CLASS                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     TTL                       |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     RDLENGTH                  |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
/                     RDATA                     /
/                                               /
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

| 名称 | 描述 |
| ---- | ---- |
| NAME | 所查询的域名。 |
| TYPE | 查询记录的类型。|
| CLASS | 使用 0x0001，表示网络地址。 |
| TTL | 结果可被缓存的秒数。 |
| RDLENGTH | RDATA 的长度。 |
| RDATA | 结果数据。|

---
{data-content = " 引用 "}

1. [Project 1: Simple DNS Client](https://mislove.org/teaching/cs4700/spring11/handouts/project1-primer.pdf)
1. [DNS 报文格式](https://fasionchan.com/network/dns/packet-format/)
