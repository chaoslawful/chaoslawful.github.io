---
title: "[zz] Optimizing TCP Socket Across Data Centers"
tags:
    - Linux
    - Networking
date: 2011-01-20 11:20:00
---

原文链接：http://sna-projects.com/blog/2011/01/optimizing-tcp-socket-across-data-centers/

一点评注：
广域网中高延迟带宽积链路的基本优化方式就是增加 TCP 连接的确认窗口大小，确认窗口大小在 Linux 下直接对应于 socket send/recv buffer size 设置，大于 64 KB 的窗口大小是 TCP 扩展选项，需要在 3-way handshake 时进行双方协商，因此对 socket send/recv buffer size 的设置必须在 listen / connect 之前进行，否则在 TCP 连接建立以后就无法使用大于 64 KB 的窗口了。看 man 7 tcp 就能发现相关说明。

另外应用程序能自行设置的最大 send/recv buffer size 同时收到 kernel 配置项 `net.core.rmem_max` / `net.core.wmem_max` 和 `net.ipv4.tcp_rmem[2]` / `net.ipv4.tcp_wmem[2]` 的约束，设置很大的 buffer size 前先看看需不需要提升这些限制。

