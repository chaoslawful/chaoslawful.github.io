---
title: Win10 家庭版上安装 Hyper-V 的方法
date: 2021-04-10 23:41:27
tags:
    - Win10
    - Virtualization
    - WSL
    - Hyper-V
---

WSL2 需要打开 Hyper-V 的支持，而 Win10 家庭版上没有 Hyper-V 完整组件，有 2 种方法可以开启 Hyper-V：
<!-- more -->

1. 在管理员权限的 PowerShell 中执行如下命令行即可开启 Hyper-V 服务：
```bash
bcdedit /set hypervisorlaunchtype auto
```
1. 将以下代码另存为 `hyper-v.bat`，以管理员权限执行即可安装完整的 Hyper-V 组件：
```bash
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hv.txt
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hv.txt
dism /online /enable-feature /featurename:Microsoft-Hyper-V -All /LimitAccess /ALL
pause
```

参考资料
===

- https://docs.microsoft.com/en-us/answers/questions/29175/installation-of-hyper-v-on-windows-10-home.html

