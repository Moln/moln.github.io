---
layout: page
title: 应用服务器DNS原因导致网络请求缓慢解决经验
---

## 经历项目

各三方 (Android 和 ios) SDK 账号认证中心服务.

要验证各种第三方 token 需要请求第三方的 http 接口.

## 架构

```
+--------------+--------------+----------------+---------------+----------------------+
| App Server 1 | App Server 2 | App Server ... |  App Server N | DNS server (dnsmasq) |
+--------------+--------------+----------------+---------------+----------------------+
```

## 问题

### 已考虑到的DNS问题:

- 可能会遇到三方 DNS服务器崩溃或者网络缓慢, 自行搭建1 DNS 服务 (DNSmasq).

### 遇到问题和现象

http 接口请求缓慢, 使用 `curl http://xxx.xxx.com` 能成功非常慢(至少5秒), `-v` 参数能看出DNS解析问题

使用 `curl http://ip` 就快了.

#### 问题定位:

- DNS 解析问题

#### 疑问:

- DNS能解析, 而且还有 dns 缓存服务, 为什么还这么慢

## 解决

### 分析

工具:

- tcpdump
- curl

操作:

在一应用服务器上, 分别执行

1. tcpdump 监听DNS 53 端口, `tcpdump -An -i eth0 port 53`
2. `curl http://xxx.com`

tcpdump 结果,会发现有个DNS响应时长正好跟curl时长吻合.

> 以下内容非原内容,已作修改

```
14:37:52.146823 IP 192.168.1.40.33744 > xxx.xxx.xxx.xxx.53: 11703+ A? xxx.com. (29)
E..9M.@.@......(dd.....5.%(.-............xxx.com.....
14:37:52.146953 IP 192.168.1.40.33744 > xxx.xxx.xxx.xxx.53: 3922+ AAAA? xxx.com. (29)
E..9M.@.@......(dd.....5.%(..R...........xxx.com.....
14:37:52.151338 IP xxx.xxx.xxx.xxx.53 > 192.168.1.40.33744: 11703 3/0/0 A 101.101.101.101, A 101.101.101.102, A 101.101.101.103 (77)
E..i$...@.-.dd.....(.5...U.J-............xxx.com..............X..e..;.........X..e..n.........X..e.g{
14:37:57.151412 IP xxx.xxx.xxx.xxx.53 > 192.168.1.40.33744: 3922 0/1/0 (83)
E..o$...@.-.dd.....??????????????????????????????????????????????????
```

从上记录看出, 最后一次  `14:37:57.151412 IP xxx.xxx.xxx.xxx.53 > 192.168.1.40.33744` 离首次请求时差5秒, 会是 AAAA (IPV6) 解析慢的问题?

分别试了下, 使用 `curl http://xxx.com -4` 和 `curl http://xxx.com -6`.

结果是的: 再次问题定位, DNS 解析 AAAA (IPV6)记录, 长时间未给响应问题. 

而 DNSmasq 服务对于未解析到域名不会缓存, 所以每次解析都会缓慢.

### 解决方向想法

1. Dnsmasq 有没有相关不解析IPV6 的设置?
   - 个人不熟悉该服务,放弃
   
2. Linux 系统层面. 
   - 尚未了解是否有相关参数禁用所有 ipv6的域名解析.
   - 有相关 dns 设置可以解决ipv6响应慢, `/etc/resolve.conf` 增加 `options single-request-reopen` (ipv6重新建个请求端口解析, 具体看官方解释) . 参考: ([CentOS 7 网页加载速度慢的解决办法](https://www.jianshu.com/p/d5b913783d4f))
   
3. 在应用服务代码请求客户端上解决
   - 应用服务代码使用PHP编写, 客户端使用 curl 扩展库, 最终通过 curl 选项设置 `CURLOPT_IPRESOLVE => CURL_IPRESOLVE_V4` 解决.

## 其它参考

- DNS缓存可以参考使用 nscd 或 dnsmasq
- [Linux 下开启缓存服务 NSCD](https://www.hi-linux.com/posts/9461.html)
- [利用 Dnsmasq 部署 DNS 服务](https://www.hi-linux.com/posts/30947.html)
- [CentOS 7 网页加载速度慢的解决办法](https://www.jianshu.com/p/d5b913783d4f)
- [Resolving hostname takes 5 seconds](https://unix.stackexchange.com/questions/290987/resolving-hostname-takes-5-seconds)
