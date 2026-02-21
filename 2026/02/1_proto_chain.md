# Proto Chain

Proto Chain is a DSL that describes the encapsulation of proxied traffic.

```
all := {tcp,udp,icmp}
L4 := {tcp,udp}
tunnel := {websocket,http,gRPC,HTTPUpgrade}
multiplex := {smux, yamux, h2mux}

# socks
# REALITY is a TLS variant providing encryption and obfuscation.
# Hence, we use `tls_trojan` to represent it.

[tcp.socks.{tcp,imcp},udp.socks.udp]
tcp.tunnel.socks.all
tcp.tunnel.{tls,tls_reality}.socks.all
tcp.{tls,tls_reality}.socks.all
udp.{quic,kcp}.socks.all
udp.kcp.{tls,tls_reality}.socks.all

# ssh
tcp.ssh.tcp

# vless
tcp.vless.all
tcp.tunnel.vless.all
tcp.{tls,tls_reality}.vless.all
tcp.tunnel.{tls,tls_reality}.vless.all
udp.{quic,kcp}.vless.all
udp.kcp.{tls,tls_reality}.vless.all

# Trojan is a TLS variant providing encryption and obfuscation.
# Hence, we use `tls_trojan` to represent it.
tcp.tls_trojan.all

# Hysteria2
# hysteria2 is a QUIC variant
# Hence, we use `quic_hysteria2` to represent it.
udp.quic_hysteria2.all

# shawsocks
[tcp.shadowsocks.{tcp, icmp}, udp.shadowsocks.udp]

# ANYTLS
tcp.tls.anytls.all

# naive
tcp.tls.naive.all
udp.quic.naive.all

# wireguard
udp.wireguard.all

# xhttp
up.tcp.xhttp.all
down.tcp.xhttp.all
```
