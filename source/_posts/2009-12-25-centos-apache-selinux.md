---
title: CentOS/RHEL 上 Apache 一个诡异问题的解决（都是浮云）
tags:
    - Linux
    - Apache
    - SELinux
date: 2009-12-25 15:27:00
---

最近帮同事检查一个 Apache 的问题，现象如下：
- 原本配置了多个 `VirtualHost` ，`DocumentRoot` 指向 `/var/www/` 下不同的子目录，都能正常工作
- 新加了一个 `VirtualHost` ，将 `DocumentRoot` 指向 `/home/aa/` ，重启 Apache 后无法访问该 `VirtualHost` 下的内容，提示 `403 Forbidden` ，而此时访问原先的几个 `VirtualHost` 还是没有问题
- Apache 错误日志里没有什么奇怪的输出
- 操作系统 **CentOS 4.x**
<!-- more -->

首先考虑是不是新加的 `VirtualHost` 配置有误导致文件路径映射错误，所以用 `strace` 跟了一下 Apache 看它在哪个阶段出问题的，发现 Apache 去访问的文件路径在 `/home/aa/` 下，该文件也确实存在，从 `/home/aa/` 直到该文件的各级目录都有 `a+rx` 权限， Apache 应该能直接访问。诡异的是， Apache 进程调用的 `lstat64` 系统调用对 `/home` 、 `/home/aa` 进行权限检查时都是 OK，但对 `/home/aa` 下的目标文件进行权限检查时却是 `EACCESS` ，让人百思不解。

后来才发现原来该系统上未关闭 SELinux 子系统，而 SELinux 默认会对 Apache 进程 httpd 进行若干保护，其中一种就是只允许 httpd 访问特定路径下的文件，系统默认的 `/var/www` 在允许之列，而其他目录都是禁止访问的。解决办法有二：
- 临时关闭 SELinux 对 httpd 的保护：
  1. `setsebool -P httpd_disable_trans 1`
  1. 重启 Apache
- 永久关闭 SELinux 子系统：
  1. 修改 `/etc/sysconfig/selinux` ，将其中的 `SELINUX=...` 改为 `SELINUX=disabled`
  1. 重启操作系统

