# 我所理解的 XUDP

XUDP 是工作在可靠有序双向 tunnel 之上的带地址的“多路复用 UDP 隧道”

```
client - xudp_client - proxy_server.xudp_server - dst_server
```

- xudp_client 接收到 client 发送的 udp，先将其重组成 XUDP 格式的包
	- xudp.addr = UDP 目标地址
- 再通过一条可靠有序双向 tunnel 发送给 proxy_server 
- proxy_server.xudp_server 接收到 XUDP 包，将其重组成 UDP 包，再将其发送目标服务器
- 对 dst_server 而言，所有通信都来自 proxy_server
- 如果 xudp_server 用于发送的端口收到 UDP 包，先执行 UDP 过滤策略（由具体实现决定），过滤后的 UDP 重组成 XUDP 发送给 xudp_client
	- xudp.addr = UDP 来源地址
- UDP 映射行为的实际表现由 proxy_server 底层网络 NAT 拓扑与 proxy_server.xudp_server 的映射策略共同决定
	- 其中 FullCone 等行为由映射策略对回流来源的过滤规则所决定的

XUDP Stream:

- 由 stream_id 标识
- stream_id 表示一个逻辑 UDP 会话，**其在 proxy_server 侧对应一组端口映射与地址状态。**
- client 和 server 各自独立在每个方向（上行下行）维护独立的 addr state（四个状态）
- 通过 ATYP 进行：
	- state update
	- state reference

# 报文格式

> [RFC 9000](https://datatracker.ietf.org/doc/html/rfc9000#name-notational-conventions) 的包结构风格

```
XUDP_0 {
	stream_id(16), // stream 标识
	atyp(8)=0x00, // 地址类型，复用该子隧道上一次使用的地址和端口
	length(16), // payload 长度
	payload(..) // payload
}

XUDP_1 {
	stream_id(16), // stream 标识
	atyp(8)=0x01, // 地址类型 IPv4+port
	addr(48), // IPv4
	port(16), // port
	length(16), // payload 长度
	payload(..) // payload
}

XUDP_2 {
	stream_id(16), // stream 标识
	atyp(8)=0x02, // 地址类型 Domain+port
	domain_length(8), 
	addr(..), // domain
	port(16), // port
	length(16), // payload 长度
	payload(..) // payload
}

XUDP_3 {
	stream_id(16), // stream 标识
	atyp(8)=0x03, // 地址类型 IPv6+port
	addr(128), // IPv6
	port(16), // port
	length(16), // payload 长度
	payload(..) // payload
}
```

# 原始材料

https://github.com/v2fly/v2ray-core/issues/112#issuecomment-688271288

https://github.com/XTLS/Xray-core/discussions/252

https://github.com/XTLS/Xray-core/discussions/237

https://xtls.github.io/about/news.html#_2023-4-18-v1-8-1

https://github.com/XTLS/Xray-core/tree/v26.2.6

https://github.com/SagerNet/sing-vmess/tree/v0.2.7
