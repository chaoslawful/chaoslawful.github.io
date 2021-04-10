---
title: "使用 Terratec Cinergy T Stick+ DVB 电视卡进行 RTL-SDR 实验时的几点注意事项"
tags:
    - Linux
    - SDR
date: 2013-06-18 00:12:00
---

- Terratec Cinergy T Stick+ 相关驱动仅在 kernel 3.7 及以上版本中并入了主干
  - 使用之前版本内核的同学需要自行从 `git://linuxtv.org/media_build.git` 检出代码编译安装驱动模块
  - 详情见 http://www.linuxtv.org/wiki/index.php/TerraTec_Cinergy_T_Stick%2B
- 安装 rtl-sdr 工具后，请确保 `/etc/udev/rules.d/` 中增加了其给出的 udev 规则以对设备结点权限进行修正，否则只有 root 才能访问 DVB 设备
- 使用 rtl-sdr 等 SDR 工具前，需要将自动加载的 dvb_usb_rtl28xxu 模块卸载，否则 SDR 相关工具会无法打开 DVB 设备并报错，建议检出最新版 rtl-sdr 代码并使用 `--enable-driver-detach` 选项配置后编译安装，这样 rtl-sdr 工具一旦发现设备被该模块占用便会将其自动卸载。
