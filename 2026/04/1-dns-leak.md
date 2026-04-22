# 我所理解的 DNS Leak

DNS Leak 指的是，**查询域名使用了非预期的 DNS Server**

# DNS 泄露网站的工作原理

DNS Leak 网站开发者会注册一个域名，用于检测 DNS 泄露.

```
xxxx.com
```

并维护这个域名的 DNS 权威服务器，记录查询 DNS 的来源 IP.

你在浏览器访问 DNS Leak 网站，网站会触发唯一子域名的解析，比如

```
random-id.xxxx.com
```

开发者维护的 DNS 权威服务器会记录哪个 IP 查询了这个域名.

把记录到 IP 和 geo 数据库对比.

如果，预期是 cloudflare 的 IP，但是实际却是 Telecom 的 IP.

那就说明实际查询 DNS 用了 telecom 的 DNS 服务器，而不是 cloudflare.

也就是发生了 DNS Leak.
