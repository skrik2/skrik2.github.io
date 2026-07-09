# Mihomo 心智模型

> 未完待续

# 路由系统

路由系统匹配代理请求的元数据，然后使用路由规则对应的 Outbound 出站

请求元数据

- 代理请求目标地址信息：IP, Domain, Port
- 代理请求来源地址信息：IP, Port
- Inbound 相关
  - 类型，也就是 listeners 类型：tun, vless, socks 等
  - port，也就是 listener 的端口
  - user，也就是 listener 中的用户
  - name，也就是 listener 中的 name
  - listener 参见 https://wiki.metacubex.one/config/inbound/listeners/
- UID： Linux 上的 UID
- 进程相关（name, path）
- 网络流量类型：TCP 或者 UDP
- DSCP

```mermaid
flowchart TD
  Rule[匹配规则]
  Domain[匹配到域名规则]
  IP[匹配到目标 IP 规则（注 1）]
  MatchOtherRule[匹配到其他规则]
  Resolve[解析域名]

  NameServer[使用 nameserver 查询]
  Policy[匹配 nameserver-policy]
  Concurrent[使用 nameserver 和 fallback 并发查询]
  Filter[匹配 fallback-filter]
  DirectNS[使用 direct-nameserver 重新解析]

  GetIP[查询得到IP]

  Proxy[Proxy Outbound]
  Direct[Direct Outbound]
  Outbound[Outbound]

  Rule -->  Domain
  Rule --> IP
  Rule --> MatchOtherRule

  Domain -- 域名匹配到直连 --> Resolve
  Domain -- 域名匹配到直连并配置了 direct-nameserver --> DirectNS

  IP --> Resolve

  MatchOtherRule -- 直接使用对应的出站 --> Outbound

  Resolve -- 配置了 nameserver-policy --> Policy
  Policy -- 未匹配到 --> NameServer
  Policy --> GetIP
  Resolve -- 未配置 nameserver-policy --> NameServer

  NameServer -- 配置了 fallback --> Concurrent
  Concurrent --> Filter
  Filter --> GetIP
  NameServer -- 未配置 fallback --> GetIP

  GetIP -- 匹配到直连并配置了 direct-nameserver --> DirectNS
  DirectNS --> Direct

  GetIP -- IP 匹配到代理（注 2） --> Proxy
  Domain -- 域名匹配到代理 --> Proxy

  GetIP -- IP 匹配到直连（注 2）--> Direct
```

注

- 1：如果匹配的是 IP 类型的路由规则，那么会进行域名解析，然后判断解析结果是否匹配该 IP 规则，如果添加上 no-resolve 则直接跳过该规则的匹配.
  - 什么是 IP 类型的路由规则？使用的代理请求目标地址是 IP 类型的相关规则
- 2：解析得到的 IP 会填充到元数据的 `DstIP` 中。用于本次匹配和后续遇到的 IP 规则的匹配。最终发往代理的请求地址会变为 IP 而非域名。如果是直连，则直接向 IP 发起连接 

# DNS 系统

本地电脑上发起一次 DNS 查询的过程.

> mihomo 的 DNS 系统把路由和 DNS 查询混在了一起.

```mermaid
flowchart TD

%% node
  Rule[域名匹配]
  hosts[匹配到 hosts 中的域名]
  not-hosts[非 hosts 中的域名]

  response-hosts[使用 hosts 相关信息响应]
  fake-ip-filter[匹配 fake-ip-filter]

  return-fakeip[返回 fakeip]

  resolve[解析域名]

  nameserver[使用 nameserver 查询]
  nameserver-policy[匹配 nameserver-policy]

  response[使用查询结果响应]

  fallback[使用 fallback 进行相同的查询]
  fallback-filter[匹配 fallback-filter（注 1）]

%% flow 

  Rule --> hosts
  Rule --> not-hosts

  hosts --> response-hosts
  not-hosts -- 配置了 fake-ip-filter --> fake-ip-filter
  not-hosts --> return-fakeip

  fake-ip-filter -- 匹配到 fake-ip-filter --> resolve
  fake-ip-filter --> return-fakeip

  resolve -- 配置了 nameserver-policy --> nameserver-policy
  resolve -- 未配置 nameserver-policy --> nameserver

  nameserver-policy -- 未匹配到 --> nameserver
  nameserver-policy --> response

  nameserver -- 配置了 fallback --> fallback
  nameserver -- 未配置 fallback --> response

  fallback --> fallback-filter

  fallback-filter --> response
```

注

- 1：此时，有两个查询结果 nameserver 和 fallback ，如果 nameserver 符合 fallback-filter ，则使用 fallback 的查询结果
- 2：`direct-nameserver` 和 `proxy-server-nameserver` **不参与 DNS 查询过程**。
- 3：该查询过程并没有考虑缓存
