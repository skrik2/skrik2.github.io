# 我所理解的 UoT Protocol Version 2

- UoT2 工作在可靠有序的双向 tunnel 之上
- client 先发送一个 proxy request 给 proxy server
    - proxy_request.addr = sp.v2.udp-over-tcp.arpa
- proxy server 接收到 proxy_request； 检查地址，如果是 `sp.v2.udp-over-tcp.arpa`，则将 proxy tunnel 升级成 UoT tunnel
    - proxy server 不会通知 client 升级
- client 会在发送 proxy request 后，紧接着发送 uot_request ，其包含即将要发送的 UDP 目标地址
- client 发送了 uot_request 后，可立即发送后续的 uot_stream（connect_stream 或 no_connect_stream）
- proxy server 收到 uot_request 后，记录地址信息，准备处理后续 client 发送过来的 connect_stream 或 no_connect_stream
- 同时用 connect_stream 或 no_connect_stream 的格式转发来自目标地址或其他地址的 UDP
- 如果 is_connect = 0x01 则 后续使用 connect_stream，proxy server 只能转发来自目标地址的UDP，client 只能向 目标地址发送 UDP
- 如果 is_connect = 0x00 则 后续使用 no_connect_stream，proxy server 可以转发来自任意地址的UDP，client 可以向任意地址发送 UDP
    - uot_request 只需要发送一次
- 是否接收来自目标地址或任意地址的 UDP 则取决于 proxy server 的底层网络拓扑结构

# 报文结构

> [RFC 9000](https://datatracker.ietf.org/doc/html/rfc9000#name-notational-conventions) 的包结构风格

```js
uot_request {
    is_connect(8)=0x01,
    ATYPE(8),
    address(..),
    port(16),
}

// is_connect(8)=0x01
connect_stream {
    length(16),
    data(..)
}

// isConnect(8) = 0x00
no_connect_stream {
    ATYPE(8),
    address(..),
    port(16),
    length(16),
    data(..)
}

```
**ATYP / address / port**: Uses the SOCKS address format, but with different address types:

| ATYP   | Address type |
| :----- | :----------- |
| `0x00` | IPv4 Address |
| `0x01` | IPv6 Address |
| `0x02` | Domain Name  |

