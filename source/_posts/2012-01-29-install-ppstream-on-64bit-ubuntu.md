---
title: 64 位 Ubuntu 11.10 下安装 PPStream
tags:
    - Linux
date: 2012-01-29 11:15:00
---

PPStream 的 Linux 版本只出了 i386 架构的安装包，但由于 64 位 Ubuntu 11.10 的包划分有所变化，直接安装 PPStream 官方 deb 包会提示依赖问题。在我的机器上通过以下步骤可以成功安装 PPStream：
- 安装 32 位支持库：
```bash
sudo apt-get install ia32-libs
```
- 下载 PPStream deb 包：
```bash
wget http://download.ppstream.com/linux/PPStream.deb
```
- 去除无效的依赖关系再安装：
```bash
# 提取deb包文件
dpkg-deb -x PPStream.deb ppstream
# 提取deb包元信息
dpkg-deb -e PPStream.deb ppstream/DEBIAN
# 修改 ppstream/DEBIAN/control，删除Depends:...这行并保存
# 重新打包
dpkg-deb -b ppstream PPStream-hacked.deb
# 安装新包
sudo dpkg -i PPStream-hacked.deb
```
- 此时 PPStream 自带的 mplayer 因为缺乏 32 位的 libgif4 库而无法启动（表现为启动 PPStream 后视频无法播放，不停地跳到下一个视频），但 32 位版本的 libgif4 和 64 位版本文件位置相同不能直接安装，这应该是 apt 源中的打包问题，需要我们自己来修正：
```bash
# 下载32位libgif4包
apt-get download libgif4:i386
# 提取deb包文件
dpkg-deb -x libgif4*.deb libgif4
# 提取deb包元信息
dpkg-deb -e libgif4*.deb libgif4/DEBIAN
# 修改安装位置
mv libgif4/usr/lib{,32}
mv libgif4/usr/share/doc/libgif4{,-i386}
# 修改libgif4/DEBIAN/md5sums，将其中的usr/lib改为usr/lib32，usr/share/doc/libgif4改为usr/share/doc/libgif4-i386，保存
# 重新打包
dpkg-deb -b libgif4 libgif4-hacked.deb
# 安装新包
sudo dpkg -i libgif4-hacked.deb
```
- 现在应该可以启动 PPStream 了，若无声音可调整选项中的音频设备为 alsa，若无图像可调整选项中的视频设备为 xv （可根据自己系统环境找一个性能更好的选项）

Enjoy it!

**注意：上述对 32 位 libgif4 包的处理仅对 Ubuntu 11.10 及之前的版本有效，11.10 之后的版本动态库目录发生了变化，且该包的问题可能已经被修正，笔者没有新环境进行验证，请自行寻找相关解决方案！**

**更新：Linux Mint 14 (Nadia) / Ubuntu 12.10 (Quantal) 无需进行上面 libgif4 包的修正，直接 sudo apt-get install libgif4:i386 libjpeg62:i386 安装相应包即可正常启动 PPStream**

