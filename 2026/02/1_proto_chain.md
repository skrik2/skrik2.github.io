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

# Example

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
tcp.xhttp.L4@up
tcp.xhttp.all@down

# @
tcp.xhttp.L4@{up, experimental}
```
