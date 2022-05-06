---
title: 完整的 chroot 过程
tags:
    - Linux
date: 2013-09-10 14:22:00
---

直接简单地运行 `chroot <new-root-dir>` 不足以正常运行一些程序，因为很多程序需要访问 `procfs` 、 `sysfs` 、 `devfs` 等特殊的子目录，直接 `chroot` 更改根目录后这些子目录没有被挂载，内容都是空，导致这些程序出现问题。
<!-- more -->

完整的 `chroot` 前准备过程可以通过下列命令完成：
```bash
cd <new-root-dir>
mount -t proc proc proc/
mount -t sysfs sys sys/
mount -o bind /dev dev/
mount -t devpts pts dev/pts/
cp -L /etc/resolv.conf etc/resolv.conf
```
上述命令将宿主系统的 `procfs` 、 `sysfs` 、 `devfs` 等特殊目录挂载到新的根目录下对应子目录上，并设置 `chroot` 系统使用宿主系统的域名解析配置，最后用以下命令即可完成 `chroot` 操作：
```bash
chroot <new-root-dir> /bin/bash
```

参考：
+ https://wiki.archlinux.org/index.php/Change_Root
