---
title: 在 Ubuntu 下用 adb 连接京崎 Tooky T85 手机
tags:
    - Android
    - Tooky
date: 2013-10-09 22:35:00
---

## 获取手机的 vendor id
用 USB 线连接手机，执行 `lsusb` 命令后可以看到如下内容：
```bash
Bus 001 Device 010: ID 4bb0:30d2
```
那么 vendor id 就是 `0x4bb0`

## 增加识别手机设备的 udev 规则
编辑 `/etc/udev/rules.d/51-android.rules` ，加入如下内容：
```bash
SUBSYSTEM=="usb", ATTRS{idVendor}=="4bb0", MODE="0666", GROUP="plugdev"
```
这里 `ATTRS{idVendor}` 条件之后填的就是上面获得的 vendor id。

添加完毕后执行 `sudo reload udev` 重新加载新的 udev 规则，然后重新插拔一次手机

## 让 adb 识别手机设备
做完以上步骤后 `adb devices` 仍然不能识别手机，因为京崎的 vendor id 并没有放在 Android SDK 的默认厂商列表中。

这里需要修改 `~/.android/adb_usb.ini` ，在最末尾增加 16 进制格式的 vendor id，这里增加后的内容就是：
```bash
# ANDROID 3RD PARTY USB VENDOR ID LIST -- DO NOT EDIT.
# USE 'android update adb' TO GENERATE.
# 1 USB VENDOR ID PER LINE.

0x4bb0
```
注意运行 `android update adb` 会清空该文件，慎用此命令！

最后执行如下命令重启 adb server 即可：
```bash
adb kill-server
adb start-server
adb devices
```

看到 `adb devices` 显示如下字样就是成功了：
```bash
List of devices attached
0123456789ABCDEF	device
```

