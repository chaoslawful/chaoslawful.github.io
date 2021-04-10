---
title: "RPM 打包时报告 Package already exists: %package ... 错误的可能原因"
tags:
    - RPM
    - Linux
date: 2010-02-05 10:56:00
---

用 rpmbuild 编译一些老的 SRPM 或 tarball 时经常会报告 `Package already exists: %package ...` 错误而无法继续，通常其原因是这些包的 spec 文件中含有当前版本的 rpmbuild 无法识别或展开的宏定义，只要修改一下 spec 文件保证其中非标准的宏都有对应定义应该就能继续打包了。

