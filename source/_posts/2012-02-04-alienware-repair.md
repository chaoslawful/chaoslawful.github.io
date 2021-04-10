---
title: Alienware 惊魂记
tags:
    - Alienware
date: 2012-02-04 01:43:00
---

Alienware M17x R3 已经到手很久了，本来配置的 AMD Radeon HD 6990M 是准备拿来玩 GPGPU 的。但由于 Alienware 坚持使用 Muxed Graphics Switch（人工切换集/独显比较可靠），而 Linux 下的新版本 AMD Catalyst 驱动只支持 Muxless Graphics Switch，使得这块卡在 Linux 下一直得不到很好的应用。

如果能禁用集成显卡，应该就可以正常使用 Catalyst 驱动，杯具的是官方 BIOS 从 A04 版之后就去掉了显卡切换选项，Linux 下也没发现有软件方式解决该问题。最近在这里找到人家破解的 M17x R3 A08 BIOS 解开了显卡切换选项，于是刷上试试。在新 BIOS 的第 2 个 Advanced/Video Configuration/Internal Graphics Device 组中，把 Internal Graphics Device 对应选项从原来的 Auto 改为 Disable 禁用集显，然后 Catalyst 果然好用了，只是屏幕亮度在禁用集显之后略有下降，不影响使用。

但是……我手很欠的想试试别的选项，结果把 Video Configuration/Primary Display 选项从原来的 SG 改成了 Auto，又把 Internal Graphics Device/Internal Graphics Device 从 Disable 改成了 Enable，重启以后就杯具鸟：主显示器黑屏，外接显示器无信号，BIOS 不进入 POST 过程，键盘无反应，电源灯间隔几秒闪一下，同时伴有很小的 beep 声。Alienware 就这样成了板砖……

## 恢复尝试1：CMOS放电
- 拔掉电池和电源适配器
- 长按电源键30s
- 插上电池或电源适配器开机
- 失败…… :-(

## 恢复尝试2：联系 DELL 售后
售后人员表示刷非官方 BIOS 导致的故障不保修，我擦…… :-(

## 恢复尝试3：BIOS 灾难恢复（需要另一台可用电脑）
- 打开 Alienware 背板，拆掉独立显卡（具体过程参考这里：http://dell.benyouhui.it168.com/thread-1711488-1-1.html）
- 用 7-zip 打开官方 A08 BIOS 可执行文件，提取其中的 `PAR00MEC.fd` 文件（用破解 BIOS 的同名文件也可以，我就是继续刷破解 BIOS ;-)），改名为 `PAR00X64.fd` （灾难恢复特殊文件，注意大小写，仅适用于 M17 系列，用附件中的 phoenixtool 打开 PAR00MEC.fd 即可获知该特殊文件名）
- 将 `PAR00X64.fd` 复制到仅有一个小于 2GB 的 FAT32 分区的 U 盘根目录下，再将 U 盘插到 Alienware 的 eSATA 口上（你没看错……实际是个 eSATA/USB 通用口）
- Alienware 拔掉电池和电源，按住小键盘区的 END 键不放，插上电源，自动开机后松开即可（注意：如果之前不拆掉独立显卡，还是会维持原来的板砖状态的，无法继续）
- 等几秒后，Alienware 会自动读取 U 盘上的灾难恢复文件并刷新 BIOS，同时伴有 10 几声较大的 beep 声
- beep 声结束后会自动重启 2 次，不要断电！
- 最后就看到正常的开机画面了！ :-)

破解的 A08 BIOS、phoenixtools 工具和拆机照见附件，也许对你有帮助哈。
- [破解后的 M17R3 A08 BIOS](/files/ALIENWARE_17XR3_UNLOCKED_A08_BIOS.rar)
- [BIOS 工具 phoenixtool v1.90](/files/phoenixtool190.zip)

<center>拆下来的 6990M 显卡及散热管</center>

![](/images/alienware_dgd.jpg)

<center>顺便拆开键盘看看</center>

![](/images/alienware_kbd.jpg)

<center>键盘下面有还没用上的 2 个内存槽</center>

![](/images/alienware_mem_slot.jpg)

<center>近看未用的内存槽，全插上 8GB 内存条就 16GB 内存咯</center>

![](/images/alienware_mem_slot_near.jpg)

<center>留个默认 BIOS 设置照，改 Primary Display 选项要小心啊</center>

![](/images/alienware_bios_default_setting.jpg)

<center>要禁用集显从这里进</center>

![](/images/alienware_igd_set1.jpg)
![](/images/alienware_igd_set2.jpg)

<center>这里的 Primary Display 选项别乱改哈</center>

![](/images/alienware_prim_disp.jpg)

<center>这里的 Integrated Graphics Device 从 Auto 改成 Disable 就能禁用集显了</center>

![](/images/alienware_disable_igd.jpg)

