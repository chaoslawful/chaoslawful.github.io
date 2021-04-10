---
title: Erlang 中 DNS 解析顺序的问题
tags:
    - Erlang
    - DNS
date: 2010-02-04 12:03:00
---

Erlang 的 DNS 解析方法有包括 file（读取 `/etc/hosts` 文件）、dns（Erlang 自己的 DNS 客户端）、native（调用外部程序 `inet_gethost` 用 libc 的 `gethostbyname` 函数解析域名） 在内的好几种方式，可以在 kernel inetrc 文件中以 `{lookup, [...]}` 形式指定多种 DNS 解析方法的应用顺序。在 `inet:gethostbyname_tm/4` 函数中可以发现任意一个域名的解析顺序是：

1. 尝试按 lookup 属性中指定的顺序应用 file、dns、native（或 yp、nis、nisplus、wins 等）中的方法解析域名，并忽略不认识的 DNS 解析方法；
2. 若所有指定的 DNS 解析方法都试过以后也没能得到 IP 地址，就检查给定域名是不是已经是一个 IPv4/v6 地址了，若是则直接返回，否则报错。

由于 dns 方式比 native 方式效率高、并发能力好，所以配置 lookup 为 `[file, dns]` 是比较常见的。这一解析顺序通常情况下工作的都很好，但若 DNS 服务器会将所有无法正常解析的域名解析为一个统一的 IP 地址（例如大部分公司网络环境），而你尝试去解析一个已经是 IPv4/v6 地址的字符串，就会产生严重的问题。Why？因为在第 1 步中，dns 方法会将已经是 IP 地址的字符串当作域名发送给 DNS 服务器，由于前述的 DNS 服务器特性，解析结果就会变成另外一个驴头不对马嘴的 IP 地址，造成严重的混乱。

这一问题影响面很广，基础库中的 gen_tcp、gen_udp、http 等都会受波及。目前的临时解决方法有这么几种：
1. 若可能被当作域名解析的 IP 地址是有限的几个，最简单的办法是在 `/etc/hosts` 文件中为这些域名设置本地域名别名，这样在 file 解析阶段就能将其正确处理了；
2. 或者在 lookup 属性中用 native 替代 dns，由于 native 使用 libc 的 `gethostbyname` 函数解析域名，其会先识别给定域名是否已经是 IP 地址，故可以避免发生此问题。弊端是 native 调用外部进程解析域名，并发度和性能上都有不小的损失；
3. 或者自行用 `inet_parse:ipv4_address/1, ipv6_address/1` 预先检查待解析的域名字符串是否已经是 IP 地址，若是则以标识 IP 地址的 4 元组（IPv4）或 16 元组（IPv6）作为主机地址参数。但此法对 http 模块无效，因为即便 URL 中给出的主机地址是 IP，该模块内部还是会进行一次域名解析。

该问题我已经发到了 erlang-bugs@erlang.org 并得到了 Raimo Niskanen 的积极回应，有望在下个版本中得到改进：

> Raimo Niskanen 写道：
> This leaves us some other alternatives:
> 1) To extend the 'file' or the 'dns' lookup
> methods with IP string detection.
> 2) To do only do IP string detection when the 'native'
> lookup method is not used, but to do it first instead
> of last as today, within the inet:gethostbyname
> lookup method wrapper itself.
> 3) To introduce a new lookup method e.g 'ipstring' that the
> user can insert in the lookup chain wherever suitable.
>
> I do not like 1) since it would make either of those
> lookup methods impure, harder to test and harder to explain.
>
> I prefer 2) because it would behave kind of natural.
>
> However, 3) is the purest but would require manual
> configuration in more cases.
>
> But 2) wins...
> At least until someone convinces me otherwise.

