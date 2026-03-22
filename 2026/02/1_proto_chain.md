# Proto Chain

Proto Chain is a DSL which describes the encapsulation of traffic proxy.

# Proto Chain DSL Syntax

The syntax of Proto Chain DSL is defined using a variant of Extended Backus-Naur Form (EBNF), following conventions similar to Go's [grammar specification](https://go.dev/ref/spec) .

```ebnf
/* Proto Chain file struct */
ProtoChainFile = { Alias } , { Chain | ChainList } ;

/* Chain modifier: optional, applies to the entire chain */
ChainModifier = "@" , Identifier
              | "@" , "{" , Identifier , { "," , Identifier } , "}" ;

/* Identifier: matches Go-style identifiers, used for protocols, modifiers, and chain components */
Identifier = letter , { letter | digit | "_" } ;

/* Choice: represents a set of protocol options within a single chain component */
Choice = "{" , Identifier , { "," , Identifier } , "}" ;

/* Alias: defines a new identifier as equivalent to a choice of identifiers */
Alias = Identifier , "=" , Choice ;

/* ChainComponent: either a single identifier or a choice of identifiers */
ChainComponent = Identifier | Choice ;

/* Single chain: dot-separated protocol stack, optionally followed by a chain modifier */
Chain = ChainComponent , { "." , ChainComponent } , [ ChainModifier ] ;

/* Multiple chains: a list of chains in parallel, separated by commas */
ChainList = "[" , Chain , { "," , Chain } , "]" ;

/* Top-level Proto Chain expression: either a single chain or a list of chains */
ProtoChain = Chain | ChainList ;
```

# Predefined Identifier and Alias

```bash
# Transmission Control Protocol (TCP)
# https://datatracker.ietf.org/doc/html/rfc9293
tcp 

# User Datagram Protocol
# https://datatracker.ietf.org/doc/html/rfc768
# https://datatracker.ietf.org/doc/html/rfc8304
udp 

# ICMP（Internet Control Message Protocol）
# ICMPv4 https://datatracker.ietf.org/doc/html/rfc792
# ICMPv6 https://datatracker.ietf.org/doc/html/rfc4443
icmp

# The Transport Layer Security (TLS) Protocol
# tls1.3 https://datatracker.ietf.org/doc/html/rfc8446
tls 

# QUIC: A UDP-Based Multiplexed and Secure Transport
# https://datatracker.ietf.org/doc/html/rfc9000
quic

# The WebSocket Protocol
# https://datatracker.ietf.org/doc/html/rfc6455
websocket 

# A high performance, open source universal RPC framework
# https://grpc.io
gRPC 

# KCP - A Fast and Reliable ARQ Protocol
# https://github.com/skywind3000/kcp/blob/master/protocol.txt
kcp

# SOCKS Protocol Version 5
# https://datatracker.ietf.org/doc/html/rfc1928
socks 

# HTTP 流量（遵循 RFC 9110/9112）
http

# 通过 HTTP 代理的流量（任何方法）
# 遵循 RFC 9110/9112
http_proxy

# HTTP CONNECT 方法，用于建立 TCP 隧道
# RFC 9110 §9.3.6 https://www.rfc-editor.org/rfc/rfc9110.html#name-connect
http_connect

# The Secure Shell (SSH) Transport Layer Protocol
# https://datatracker.ietf.org/doc/html/rfc4253
ssh 

# WireGuard is an extremely simple yet fast and modern VPN that utilizes 
# state-of-the-art cryptography.
# 
# https://www.wireguard.com/protocol/
wireguard 

# Tor is a distributed overlay network designed to anonymize low-latency 
# TCP-based applications such as web browsing, secure shell, and instant messaging.
# 
# https://spec.torproject.org
tor 

# Security Architecture for the Internet Protocol
# https://datatracker.ietf.org/doc/html/rfc4301
ipspec 

# Multiplexed Application Substrate over QUIC Encryption (masque)
# https://datatracker.ietf.org/wg/masque/about/
masque

# https://github.com/xtaci/smux
smux 

# https://github.com/hashicorp/yamux
yamux 

# HTTP/2 Streams and Multiplexing
# https://datatracker.ietf.org/doc/html/rfc9113#name-streams-and-multiplexing
h2mux

all tunnel multiplex

vless vmess trojan anytls tuic 
shawdowsocks ss
shadowsocksR ssr
shadowsocks2022 ss2022
tuic
```

# Proto Naming Suggestions

When one proto is a variant of another proto, the raw proto should be used as a prefix.

```
quic_hysteria2 tls_reality
```

# Example

```
all := {tcp,udp,domain,icmp}
L4 := {tcp,udp}
tunnel := {websocket,http,gRPC,HTTPUpgrade}
multiplex := {smux, yamux, h2mux}

# socks
# REALITY is a TLS variant providing encryption and obfuscation.
# Hence, we use `tls_trojan` to represent it.
[tcp.socks.{tcp,imcp}, udp]
ip.tcp.{tls,tls_reality}.socks.all
ip.tcp.{tls,tls_reality}.tunnel.socks.all
ip.udp.{quic,kcp}.socks.all
ip.udp.kcp.{tls,tls_reality}.socks.all

# ssh
tcp.ssh.tcp

# vless
tcp.vless.all
tcp.{tls,tls_reality}.vless.all
tcp.{tls,tls_reality}.tunnel.vless.all
udp.{quic,kcp}.vless.all
udp.kcp.{tls,tls_reality}.vless.all

# Trojan is a TLS variant providing encryption and obfuscation.
# Hence, we use `tls_trojan` to represent it.
tcp.tls_trojan.all

# Hysteria2
# hysteria2 is a QUIC variant
# Hence, we use `quic_hysteria2` to represent it.
udp.quic_hysteria2.all

# ANYTLS
tcp.tls.anytls.all

# naive
tcp.tls.naive.all
udp.quic.naive.all

# xhttp
up.tcp.xhttp.all
down.tcp.xhttp.all

# @
tcp.xhttp.L4@{up, experimental}

ip4.tcp.tunnel.socks.all
ip6.tcp.tunnel.{tls,tls_reality}.socks.all
```
