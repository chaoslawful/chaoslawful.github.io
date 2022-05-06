---
title: 在 Mele A2000 上运行 Archlinux ARM 版
tags:
    - Linux
    - ARM
    - hack
date: 2012-10-28 02:39:00
---

本来以为 archlinuxarm.org 上有 Mele A100 的安装指南装起来会方便些，但实际操作过程中仍然踩了一些坑，这里记录一下备查。
<!-- more -->

我选择 Mele A2000 的原因就是因为有 SATA 口，使用最新的 U-boot 后有可能直接把根分区放在 SATA 盘上，这样比起根分区在 SD 卡或 USB2.0 硬盘启动要快很多。遗憾的是，archlinuxarm.org 上最新的 sun4i 架构 rootfs 中带的 kernel 内置的启动代码有点儿小问题，在新版 Mele A2000 的板子上会将 SATA 控制器时钟设置错误，导致 kernel 启动挂载 sata 盘分区时循环显示 sata 接口 reset 错误无法继续：
```
[ 5.530000] ata1: SATA link down (SStatus 1 SControl 300)
[ 5.530000] ata1: EH complete
[ 5.540000] ata1: exception Emask 0x10 SAct 0x0 SErr 0x4000000 action 0xe frozen
[ 5.540000] ata1: irq_stat 0x00000040, connection status changed
[ 5.550000] ata1: SError: { DevExch }
[ 5.550000] ata1: limiting SATA link speed to 1.5 Gbps
[ 5.560000] ata1: hard resetting link
```

因此只有下载修正后的 kernel 代码自行编译内置了 sw_ahci_platform 模块的 uImage(以 Ubuntu 11.04 为例):
```bash
$ sudo apt-get install gcc-arm-linux-gnueabi g++-arm-linux-gnueabi u-boot-tools
$ git clone https://github.com/linux-sunxi/linux-sunxi.git
$ cd linux-sunxi
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- sun4i_defconfig
$ make ARCH=arm menuconfig ，将 Device Drivers/Serial ATA and Parallel ATA drivers/SoftWinner Platform AHCI SATA support 从默认的 module 改为 built-in
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- uImage modules
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- INSTALL_MOD_PATH=output modules_install
```
然后：
- 将 `arch/arm/boot/uImage` 复制到 SD 卡启动 FAT 分区下覆盖从原 rootfs 中复制过来的同名文件
- 将 `output/lib` 目录复制到 sata 盘 rootfs 中的 `/usr/` 目录下
- 重新启动应该就能正常从 sata 盘启动了

以上过程参考：
- http://linux-sunxi.org/Linux
- http://rhombus-tech.net/allwinner_a10/hacking_the_mele_a1000/Building_Debian_From_Source_Code_for_Mele/

启动之后的另一个问题就是无线网络，由于 archlinuxarm.org 上下载的 rootfs 是最小镜像，连配置网卡所需的 wireless_tools 包都没有装，需要自己编译一个放到 rootfs 里：
```bash
wget http://www.hpl.hp.com/personal/Jean_Tourrilhes/Linux/wireless_tools.29.tar.gz
tar xzf wireless_tools.29.tar.gz
cd wireless_tools.29
make CC=arm-linux-gnueabi-gcc LD=arm-linux-gnueabi-ld
make install PREFIX=<your root fs mount-point>/usr/local
```
在你的 rootfs 上新增文件 `/etc/ld.so.conf.d/local.conf` ，内容只有一行 `/usr/local/lib` ，保存后运行 `ldconfig` 刷新动态库缓存

通过 wireless_tools 和 rootfs 中带的 wpa_supplicant 可以手动连接 WPA2 验证的无线网络，过程为：
- 当前目录新增 wpa_supplicant 配置文件 mine.conf，内容为：
```
ctrl_interface=/var/run/wpa_supplicant
eapol_version=1
ap_scan=1
fast_reauth=1

network={
    ssid="无线接入点名称"
    psk="无线接入密码"
}
```
- `modprobe 8192cu` ，这步很重要！默认没有自动加载 rtl8192cu 无线驱动
- `ip link set wlan0 up`
- `wpa_supplicant -i wlan0 -c mine.conf -B`
- `iwconfig wlan0 essid <无线接入点名称>`
- `dhcpcd wlan0`
能上网后一切都好说了。

以上过程参考：https://wiki.archlinux.org/index.php/Wireless_Setup#Manual_setup

**更新：** 目前 archlinuxarm.org 上下载的最新镜像已经修正了 SATA 时钟错误问题并内置了 sw_ahci_platform 模块，无需自行编译内核了。另外 wireless_tools 也不用自己编译了，可以直接下载 [安装包](http://cn.mirror.archlinuxarm.org/armv7h/core/wireless_tools-29-8-armv7h.pkg.tar.xz) 用 pacman 安装
