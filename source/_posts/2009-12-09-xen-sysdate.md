---
title: 在 Xen 虚拟机下修改系统当前时间
tags:
    - XEN
    - Virtualization
date: 2009-12-09 17:24:00
---

Xen 虚拟机默认不允许不同的虚拟机使用不同的系统时间，因此所有虚拟机的系统时间都会同宿主机的系统时间严格同步，用 `date` 命令修改虚拟机系统时间时虽然提示成功但其实系统时间还是没变。若有独立修改 Xen 虚拟机的特殊需要，可以通过如下方法进行：
<!-- more -->

- 在 Xen 虚拟机的 root 提示符下输入如下命令之一以启用虚拟机独立的系统时间：
    ```bash
    echo 1 > /proc/sys/xen/independent_wallclock
    ```
    或
    ```bash
    sysctl xen.independent_wallclock=1
    ```
- 然后用 `date -s <target date>` 应该就可以设置系统时间了

这样的设置在虚拟机重启以后就会失效，若要使该设置永久生效，可以进行如下改动之一：

- 修改 `/etc/sysctl.conf` 文件，增加如下内容：
```bash
# Set independent wall clock time
xen.independent_wallclock=1
```
- 在虚拟机启动时的 kernel 命令行中加入选项 `independent_wallclock=1`

## 参考资料

- http://docs.vmd.citrix.com/XenServer/4.0.1/guest/ch04s06.html

