---
title: 复习 Linux ELF 共享库的版本概念
tags:
    - Linux
    - ELF
date: 2009-12-30 16:34:00
---

* soname for a shared library: `lib<library name>.so.<major ver>`
  * 例： *libev.so.4*
* fully-qualified soname for a shared library: `<path>/lib<library name>.so.<major ver>`
  * 例： */usr/lib/libev.so.4*
* real name for a shared library: `lib<library name>.so.<major ver>.<minor ver>.<release>`
  * 例： *libev.so.4.0.0*
* linker name for a shared library: `lib<library name>.so`
  * 例： *libev.so*
<!-- more -->

一般 soname 对应的是一个符号链接，是在运行 `ldconfig` 时由其根据共享库 header 中的 `SONAME` 域创建的。如果创建共享库时未通过 `-Wl,-soname,...` 指定其 `SONAME` ，则 `ldconfig` 不会为其创建对应的 soname 符号链接。linker name 对应的符号链接主要用于开发链接使用，一般是创建一个指向 soname 而不是 real name 的链接，以便减少版本更替时需要改变的链接数量（当然共享库数量较少时指向 soname 或 real name 均可，系统软件包也是两种方式都有采用的例子）。

`<major ver>` 主要表明接口 ABI 兼容性，一般如果共享库接口产生了非向前兼容的更改就要升级 `<major ver>` 。

