# 我所理解的 NAT 类型

```text
  Local Client           Router            DST Server
10.10.145.2:8848 - 172.69.217.206:2026  -  1.1.1.1:443
```

# Full Cone NAT

- Router(172.69.217.206:2026) 可以接收来自任意 IP 任意 Port 的 UDP
- 同一台机器是否复用 Router 的端口和其他机器是否能共用端口，取决于具体的 NAT 实现

# Restricted Cone NAT

- Router(172.69.217.206:2026) 只能接收来自 1.1.1.1 任意 Port 的 UDP

# Port Restricted Cone NAT

- Router(172.69.217.206:2026) 只能接收来自 1.1.1.1:443 的 UDP

# Symmetric NAT

- 每一条 (client:port+dst_ip:port) 都会映射到一个 router.port

```
- 10.10.145.2:8848 -> 172.69.217.206:2026  -> 1.1.1.1:443 
- 10.10.145.2:8848 -> 172.69.217.206:30555 -> 8.8.8.8:53  
- 10.10.145.2:8848 -> 172.69.217.206:31000 -> 1.1.1.1:444 
```
- 同时 UDP 返回只接受已访问过的 dst_ip:port
