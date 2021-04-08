---
title: Ubuntu 18.04 LTS 中让桌面系统不占用 N 卡显存的方法
date: 2021-04-08 14:18:54
tags: [linux,ubuntu,cuda,nvidia]
---

### 背景

使用自己电脑的 N 卡进行 CUDA 加速时都希望能使用全部显存，但在 Ubuntu 18.04 LTS 中安装官方专有驱动后，如果用 nvidia-settings 将 PRIME profiles 设为 Nvidia，会让包括 X 桌面在内的所有图形加速功能都走 N 卡，会占用不少显存。若将 PRIME profiles 改为 Intel，N 卡驱动又不会被加载，无法使用 CUDA 加速。

### 解决方法

PRIME profiles 是通过 `/usr/bin/prime-select` 命令完成切换的，PRIME profiles 设为 Intel 时该命令会生成一个驱动黑名单 `/lib/modprobe.d/blacklist-nvidia.conf`，修改该文件内容将如下几行注释掉：

```
blacklist nvidia
blacklist nvidia-modeset
alias nvidia off
alias nvidia-modeset off
```

logout 后重新 login 即可实现 X 桌面系统图形加速走集成显卡且 N 卡驱动正常加载的效果，可使用全部显存进行 CUDA 计算。

