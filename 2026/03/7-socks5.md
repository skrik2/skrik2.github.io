# socks5 代理过程

> 参考资料 [SOCKS Protocol Version 5](https://datatracker.ietf.org/doc/html/rfc1928)
> 
- socks5 代理的是 TCP, UDP 流量，无法代理 icmp 流量

https://github.com/openrfs/rfs/blob/main/rfc/1/1.v1.md 

# 版本协商和验证

## 第一阶段

- client 发送验证协商请求，server 会选择一个验证方式回复

```js
auth_nego_request {
	version(1)=0x05,
	// The auth_method_count field contains the number of method identifier octets that
  // appear in the auth_method_list field.
  auth_method_count(1),
	auth_method_list(auth_method_count),
}

auth_nego_repley {
	version(1)=0x05,
	method(1),
}
```

rfc 中定义的 method 方法：

| 值（Hex） | 名称 |
| --- | --- |
| `0x00` | NO AUTHENTICATION REQUIRED |
| `0x01` | GSSAPI |
| `0x02` | USERNAME/PASSWORD |
| `0x03–0x7F` | IANA ASSIGNED |
| `0x80–0xFE` | RESERVED FOR PRIVATE METHODS |
| `0xFF` | NO ACCEPTABLE METHODS |

如果 reply msg 中的 method 是 `0xFF` ，说明 socks5 server 无法支持 client 的 auth method，client 要立刻断开连接.

## 第二阶段

- 按照第一阶段的协商的方法，进行验证.
- 如果不需要协商则直接跳过.

# TCP 代理

## 第一阶段

- Socks5 Client 发送如下的代理请求报文
- Socks5 Server 接收到代理请求报文，就立刻开始尝试 tcp 连接.
- 如果 `dst_addr` 是 domain ，先进行 DNS 解析，再尝试 tcp 连接，无论连接是成功还是失败，都会回复.

```js
proxy_request {
    version(1)=0x05,
    cmd(1)=0x01, // CONNECT
    reserved(1)=0x00,
    atyp(1),
    [domain_length(1)],
    dst_addr(..)
    dst_port(2),
}

proxy_reply {
    version(1)=0x05,
    reply(1),
    reserved(1)=0x00,
    atyp(1),
    [domain_length(1)],
    bnd_addr(..),
    bnd_port(..)
}
```

atyp

- `0x01` : IPv4 address
- `0x03` : Domain
- `0x04` : IPv6 address

reply 的值：

| 值（Hex） | 含义 |
| --- | --- |
| `0x00` | succeeded |
| `0x01` | general SOCKS server failure |
| `0x02` | connection not allowed by ruleset |
| `0x03` | Network unreachable |
| `0x04` | Host unreachable |
| `0x05` | Connection refused |
| `0x06` | TTL expired |
| `0x07` | Command not supported |
| `0x08` | Address type not supported |
| `0x09–0xFF` | unassigned |

## 第二阶段

- 如果，没有错误， client 就可以发送数据了，socks5 server 会转发数据
- socks5 体系里，client 和 socks5 建立的一条 tcp ，只能代理一条 tcp 连接

```
1. client ↔ socks: 建立 TCP
2. client → socks: CONNECT
3. socks → target: 建立 TCP
4. socks ↔ target: TCP 建立成功
5. socks → client: 成功响应

6. 开始双向转发：

   client <-> socks <-> target

   (只是复制 TCP payload，不解析、不修改)
```

# UDP 代理

## 第一阶段

- Client 发送 UDP ASSOCIATE 代理请求
- Socks5 Server 收到 proxy_request 后回复报文

```js
proxy_request {
    version(1)=0x05,
    cmd(1)=0x03, // UDP ASSOCIATE
    reserved(1)=0x00,
    atyp(1),
    [domain_length(1)],
    dst_addr(..)
    dst_port(2),
}

proxy_reply {
    version(1)=0x05,
    reply(1),
    reserved(1)=0x00,
    atyp(1),
    [domain_length(1)],
    bnd_addr(..),
    bnd_port(..)
}
```

- `dst_addr/dst_port` 表示：客户端在该 UDP association 过程中预期用于发送 UDP 数据的地址
- 注意：不是后续发送 UDP 的目标地址，也不是后续客户端发送UDP的来源地址
- 服务器可以用于策略控制，添加防火墙规则之类的
- 如果，不需要服务器策略控制， `DST.ADDR/PORT` 可以用全零的端口号和地址
- `bnd_addr/port` 表示 socks5 server 在这个地址启动了一个 udp relay server ，后续 client 必须发送 udp 报文到 udp relay server.

### FQA 为什么保留 UDP ASSOCIATE 过程中 atyp 可以是 domain 的这种设计？

我感觉是没有必要的，可能是为了方便解析器的实现吧.

## 第二阶段

- Client 可以向从第一阶段获取的 `bnd_addr/port` 地址发送 UDP 报文
- UDP ASSOCIATE 必须要保持原有的 TCP 连接存活，如果原有的 TCP 连接端口，则立即停止 UDP ASSOCIATE
- relay server 只接受 **来自创建 UDP ASSOCIATE 的 TCP Client 所对应的 IP**
- 每个 UDP 报文，需要添加 以下格式的头部

```js
relay_udp_header {
    reserved(2)=0x00,
    frag(1),
    atyp(1),
    dst_addr(..),
    dst_port(..),
    payload(..)
}
```

- 发送 UDP 包结构

```js
[udp_header][relay_udp_header][raw_payload]
```

- udp relay server 不会向 client 通知自身的状态，从始至终保持静默.
- udp relay server 接收到来自 client 的 udp报文，处理后将其转发给 dst server.
- udp relay server 接收到来自 dst server 的响应 udp 报文，会添加相应的 relay_udp_headr 然后发送给客户端

### FRAG

- 如果 FRAG=0x00 则代表这个 UDP 报文，包含了一条完整的 UDP 报文.
- 如果 FRAG 的最高位为1，则代表这是最后一个 FRAG.
- 碎片化实现是可选的；不支持碎片化的实现必须丢弃任何 FRAG 字段不是 `0x00` 的数据包.

relay udp server 会维护两个东西

```js
// 用于存放待组装的 udp 报文
REASSEMBLY QUEUE
// 计时器
REASSEMBLY TIMER
```

- 同一个 UDP ASSOCIATE 在同一时间内只能有一个 REASSEMBLY QUEUE.
- 如果 REASSEMBLY QUEUE 接收到了一个 frag 小于等于当前处理过的最大 frag，则丢弃 REASSEMBLY QUEUE 中所有的报文.
- 每接收到一个合法的 FRAG 就重置一次 REASSEMBLY TIMER，当 TIMER 超时，则丢弃REASSEMBLY QUEUE 中所有的报文.

# BIND

这个东西已经被时代淘汰了

所以，没有介绍
