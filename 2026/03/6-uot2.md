# 我所理解的 UoT Protocol Version 2

- UoT2 建立在 socks5 体系之上
- client 先构造一个 socks5 proxy request 报文发送给 socks5 server
    - socks5.addr = sp.v2.udp-over-tcp.arpa
- socks5 proxy request 接收到 proxy request； 检查地址，如果是 sp.v2.udp-over-tcp.arpa，则将 socks5 tcp tunnel 升级成 UoT tunnel
    - socks5 server 不会通知 client 升级
- client 会在发送 socks5 proxy request 后紧着发送 uot_request 包含后续将要发送的 UDP 目标地址
- socks5 server 收到 uot_request 后，记录地址信息，准备处理后续 client 发送过来的 uot_connect_stream
    - uot_connect_stream 包含每个 UDP 的 payload
- 同时用 uot_connect_stream 的格式转发来自目标地址的 UDP
    - 严格执行过滤策略，只接受来自对应地址对应端口的 UDP

# 报文结构

> [RFC 9000](https://datatracker.ietf.org/doc/html/rfc9000#name-notational-conventions) 的包结构风格

```js
uot_request {
    isConnect(8)=0x01,
    ATYPE(8),
    address(..),
    port(16),
}

uot_connect_stream {
    length(16),
    data(..)
}
```

- ATYP 采用 socks5 的报文格式
