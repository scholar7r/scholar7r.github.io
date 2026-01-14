+++
title = '如何将字节序列反序列化成结构'
date = 2025-11-28T11:06:03+08:00
draft = false
author = "scholar7r"
authorTwitter = "scholar7r"
+++

> 反序列化在计算机领域通常指的是将序列化的数据，例如 JSON、XML 或者二进制格式重新转换为程序可操作的对象或数据结构的过程。

<!--more-->

我最近正在尝试自己写一个简单的 DNS 实现。总所周知，DNS 的报文是以字节序列进行传输的，传输层采用 UDP 协议。一个 DNS 报文的标准格式是下面这样的：

```goat
12 34 01 00 00 01 00 00 00 00 00 00
```

按照 RFC 1035 格式进行解析：

```
0-1    -   ID         0x1234 ->  4660

2-3    -   Flags      01 00
           +--+-------+--+--+--+--+---+-----+
           |QR|OPCODE |AA|TC|RD|RA|Z  |RCODE|
           +--+-------+--+--+--+--+---+-----+
           | 0|0000   |0 |0 |1 |0 |000|0000 |
           +--+-------+--+--+--+--+---+-----+
           => QR     = 0    (query)
           => OPCODE = 0000 (standart query)
           => AA     = 0
           => TC     = 0
           => RD     = 1    (recursion desired)
           => RA     = 0
           => Z      = 000
           => QCODE  = 0000

4-5    -   QDCOUNT    00 01  ->  1

6-7    -   ANCOUNT    00 00  ->  0

8-9    -   NSCOUNT    00 00  ->  0

10-11  -   ARCOUNT    00 00  ->  0
```

{{< details summary="RFC 1035" >}}

```

                                    1  1  1  1  1  1
      0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                      ID                       |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |QR|   Opcode  |AA|TC|RD|RA|   Z    |   RCODE   |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    QDCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    ANCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    NSCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    ARCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

where:

ID              A 16 bit identifier assigned by the program that
                generates any kind of query.  This identifier is copied
                the corresponding reply and can be used by the requester
                to match up replies to outstanding queries.

QR              A one bit field that specifies whether this message is a
                query (0), or a response (1).

OPCODE          A four bit field that specifies kind of query in this
                message.  This value is set by the originator of a query
                and copied into the response.  The values are:

                0               a standard query (QUERY)

                1               an inverse query (IQUERY)

                2               a server status request (STATUS)

                3-15            reserved for future use

AA              Authoritative Answer - this bit is valid in responses,
                and specifies that the responding name server is an
                authority for the domain name in question section.

                Note that the contents of the answer section may have
                multiple owner names because of aliases.  The AA bit
                corresponds to the name which matches the query name, or
                the first owner name in the answer section.

TC              TrunCation - specifies that this message was truncated
                due to length greater than that permitted on the
                transmission channel.

RD              Recursion Desired - this bit may be set in a query and
                is copied into the response.  If RD is set, it directs
                the name server to pursue the query recursively.
                Recursive query support is optional.

RA              Recursion Available - this be is set or cleared in a
                response, and denotes whether recursive query support is
                available in the name server.

Z               Reserved for future use.  Must be zero in all queries
                and responses.

RCODE           Response code - this 4 bit field is set as part of
                responses.  The values have the following
                interpretation:

                0               No error condition

                1               Format error - The name server was
                                unable to interpret the query.

                2               Server failure - The name server was
                                unable to process this query due to a
                                problem with the name server.

                3               Name Error - Meaningful only for
                                responses from an authoritative name
                                server, this code signifies that the
                                domain name referenced in the query does
                                not exist.

                4               Not Implemented - The name server does
                                not support the requested kind of query.

                5               Refused - The name server refuses to
                                perform the specified operation for
                                policy reasons.  For example, a name
                                server may not wish to provide the
                                information to the particular requester,
                                or a name server may not wish to perform
                                a particular operation (e.g., zone
                                transfer) for particular data.
                6-15            Reserved for future use.

QDCOUNT         an unsigned 16 bit integer specifying the number of
                entries in the question section.

ANCOUNT         an unsigned 16 bit integer specifying the number of
                resource records in the answer section.

NSCOUNT         an unsigned 16 bit integer specifying the number of name
                server resource records in the authority records
                section.

ARCOUNT         an unsigned 16 bit integer specifying the number of
                resource records in the additional records section.
```

{{< /details >}}

RFC 1035 中对报文的结构进行了规定，基于这个规定我们可以在 Golang 中进行反序列化。

下文中的代码使用 ChatGPT 生成。

```go
type Header struct {
    ID              uint16
    Flags           uint16 // better way is define a Flags structure
    QuestionCount   uint16
    AnswerCount     uint16
    authorityCount  uint16
    additionalCount uint16
}


func ParseDNSHeader(data []byte) (*DNSHeader, error) {
    if len(data) < 12 {
        return nil, fmt.Errorf("dns header must be 12 bytes")
    }

    h := &DNSHeader{
        ID:      binary.BigEndian.Uint16(data[0:2]),
        Flags:   binary.BigEndian.Uint16(data[2:4]),
        QDCount: binary.BigEndian.Uint16(data[4:6]),
        ANCount: binary.BigEndian.Uint16(data[6:8]),
        NSCount: binary.BigEndian.Uint16(data[8:10]),
        ARCount: binary.BigEndian.Uint16(data[10:12]),
    }
    return h, nil
}

// ===> 用例 <===

raw := []byte{
    0x12, 0x34, 0x01, 0x00,
    0x00, 0x01, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00,
}

h, _ := ParseDNSHeader(raw)
fmt.Printf("%+v\n", h)
```

总而言之，字节序列中的数据是按照顺序排列的，Golang 可以通过 `[]byte` 保存字节序列，约定好结构和成员所占据的字节长度之后就可以将序列切割成多个片，结构成员取字节序列中对应位置的数据即可。
