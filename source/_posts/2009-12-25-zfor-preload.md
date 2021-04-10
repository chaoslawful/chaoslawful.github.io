---
title: LD_PRELOAD 对 PHP extension 失效的原因
tags:
    - Linux
    - PHP
date: 2009-12-25 15:05:00
---

最近写了一个主机健康检测和软负载均衡用的软件 [ZFOR](http://github.com/chaoslawful/zfor) ，可以通过 `LD_PRELOAD` 预载入的方式拦截系统的域名解析调用（ `gethostbyname` 、 `getaddrinfo` 等）。但发现对某些发行版的 PHP CURL extension 光加载 zfor 动态库没有用，还必须同时加载 `libcurl.so` 才能生效。

经过一番搜索，发现原来是 PHP 编译时让 Zend 引擎使用了 `RTLD_DEEPBIND` 标志来加载扩展，该标志的作用是约束动态符号解析的查找范围为载入的动态库及其依赖项，而不是从所有已加载项的符号表中寻找。PHP 启用该选项的原意是避免同时加载多个具有共同依赖库的扩展时产生多版本符号解析冲突的问题，但在这里就影响了 zfor 的 preload 拦截过程。因为 PHP CURL 扩展只依赖 `libcurl.so` 而不依赖 `libzfor.so` ，故在不预加载 `libcurl.so` 时 `libzfor.so` 不会列入 PHP CURL 扩展符号解析的查找范围，这样在 `libzfor.so` 中定义的 `getaddrinfo` 别名自然无法生效。

在同时预加载 `libcurl.so` 和 `libzfor.so` 时，由于是系统的动态库加载器 `ld-linux.so` 按默认解析策略进行符号解析，故会将 `libcurl.so` 中引用的 `getaddrinfo` 符号解析到 `libzfor.so` 中提供的实现上，而由于动态库都是单实例加载的，这样当 PHP CURL 扩展被载入时，其依赖的 `libcurl.so` 就会用之前预加载的那个实例来解决，不需要再进行符号查找，也就不用受到 `RTLD_DEEPBIND` 标志的约束了。

