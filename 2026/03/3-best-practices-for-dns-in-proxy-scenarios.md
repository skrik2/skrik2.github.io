# 在代理场景下 DNS 的最佳实践 v2

> 这些经验仅适用于 **"所有流量由统一代理系统接管并可控的网络环境"。**
> 

**#1 大多数用户不需要手动管理 DNS，但仍依赖 DNS 系统作为基础设施**

**#2 尽量劫持所有的 DNS查询，方便管理**

- DNS 控制权必须集中在代理系统
- 不允许客户端使用绕过代理系统控制的 DNS（如 DoH/DoT），
- 比如，win11，浏览器可以设置 DOH，安卓可以设置 DOT
- 注意：有些 app 会内置 dns ，常规劫持就失效了，但这种情况，暂时还不常见

**#3 fakeip 不需要精细化管理 DNS**

- **几乎所有个人开发的代理工具根基还是 socks5 那一套东西**
- fakeip 在大部分语境下的代理方式是发起 connect(domain) 的代理请求
- DNS 解析由代理服务器管理
- 所以，只需要做一件，分清哪些 DNS 查询归 fakeip ，哪些不归，就足够了
- 另外，**理解 socks5 是理解 fakeip 的前置知识**
- **fakeip = “DNS 结果不重要，连接语义才重要”**
- socks5 参见 https://datatracker.ietf.org/doc/html/rfc1928

**#4 在代理场景下，fakeip 禁止 quic 流量，realip 启用 dns0_subnet**

- 在这里的语境下 fakeip 和 realip 是按 DNS 的处理方式对代理方式进行的一种分类
- 本质是对 proxy 连接语义的分类（如果不明白连接语义，先看看 socks5）
    - fakeip ⇒ connect(domain)
    - realip ⇒ connect(ip)
- 先说一条共识：**proxy tunnel 可以用 quic，ech 之类的东西，但是被代理流量最好用 TCP**
- fakeip 模式下，也就是 connect(domain) ，大部分实现是双 tcp 连接，没有 quic 的事，所以 ban 掉 HTTPS 查询和 udp 的 443 端口
- **注意：一般来说不是不能代理 quic 流量 ，而是最好不要，如果你想用 quic 就不要用 fakeip**
- realip 模式下，DNS 偏差会影响分流或增加延迟
- DNS 查询出来目标网站在美洲的 ip ，代理服务器在亚洲，亚洲连美洲延迟大大的有
- dns0_subnet 是矫正 DNS 偏差的一种手段

**#5 能用 fakeip 就不要用 realip**

- 大部分人，甚至即便是科班出身的人士，也未必能整明白如何获取正确的 DNS
- 直接交给代理服务器处理吧，fakeip的分流也很简单（很难搞出非预期的分流结果）

**#6 关于 IPv6 和 IPv4 考虑 client 到 proxy server 之间的连接**

- connect(ip)  连接语义中 ip 是 4 还是 6 不重要（一般默认 proxy server 连接 dst server 没有任何问题）
- 重要的是 client 和 proxy server 怎么连接
