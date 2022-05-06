---
title: "Ubuntu 11.10 (Oneiric) 上如何编译带 utrace 补丁的内核"
tags:
    - Linux
    - kernel
    - utrace
date: 2012-03-25 22:48:00
---

首先准备 linux 内核编译环境：
1. `sudo apt-get install fakeroot build-essential crash kexec-tools makedumpfile kernel-wedge kernel-package`
2. `sudo apt-get build-dep linux`
3. `sudo apt-get install git-core libncurses5 libncurses5-dev libelf-dev asciidoc binutils-dev`
<!-- more -->

检出带有 utrace 补丁的官方内核代码，并生成对应 Ubuntu 当前版本内核（3.0）的补丁：
1. `git clone https://github.com/utrace/linux.git utrace-linux-git`
2. `cd utrace-linux-git/`
3. `git checkout -b utrace-3.0 origin/utrace-3.0`
4. `git diff v3.0 > /tmp/utrace.patch`

获得 Ubuntu 定制内核代码，并打上 utrace 补丁：
1. `sudo apt-get install linux-source`
2. `tar xjf /usr/src/linux-source-3.0.0.tar.bz2`
3. `cd linux-source-3.0.0/`
4. `patch -p1 < /tmp/utrace.patch`

输出为：
```
patching file Documentation/DocBook/Makefile
patching file Documentation/DocBook/utrace.tmpl
patching file arch/x86/kernel/ptrace.c
patching file fs/proc/array.c
patching file include/linux/ptrace.h
patching file include/linux/sched.h
Hunk #1 succeeded at 187 (offset 3 lines).
Hunk #2 succeeded at 206 (offset 3 lines).
Hunk #3 succeeded at 1415 with fuzz 2 (offset 7 lines).
patching file include/linux/signal.h
patching file include/linux/tracehook.h
patching file include/linux/utrace.h
patching file init/Kconfig
Hunk #1 succeeded at 388 (offset 16 lines).
patching file kernel/Makefile
patching file kernel/fork.c
Hunk #1 FAILED at 168.
Hunk #2 succeeded at 1098 (offset 3 lines).
1 out of 2 hunks FAILED -- saving rejects to file kernel/fork.c.rej
patching file kernel/ptrace.c
patching file kernel/sched.c
patching file kernel/signal.c
Hunk #4 succeeded at 1993 (offset -2 lines).
Hunk #5 succeeded at 2017 (offset -2 lines).
Hunk #6 succeeded at 2124 (offset -2 lines).
Hunk #7 succeeded at 2132 (offset -2 lines).
patching file kernel/utrace.c
```
注意 `kernel/fork.c` 的补丁失败了，看看 `kernel/fork.c.rej` ：
```patch
--- kernel/fork.c
+++ kernel/fork.c
@@ -168,6 +168,7 @@
    free_thread_info(tsk->stack);
    rt_mutex_debug_task_free(tsk);
    ftrace_graph_exit_task(tsk);
+   tracehook_free_task(tsk);
    free_task_struct(tsk);
 }
 EXPORT_SYMBOL(free_task);
```
修改 `kernel/fork.c` ，在 `free_task` 函数的 `ftrace_graph_exit_task(tsk);` 之后手工加上这行 `tracehook_free_task` 调用即可。

编译新的内核，使用当前系统内核的配置参数作为基准，开启 utrace 补丁提供的 CONFIG_UTRACE 功能即可：
```
cp /boot/config-$(uname -r) .config
make oldconfig
```
出现如下提示时回答 y： `Infrastructure for tracing and debugging user processes (UTRACE) [N/y/?] (NEW)`
```
make-kpkg clean
export CONCURRENCY_LEVEL=9
```
这里指定编译内核时的并发任务数，设置为核数+1即可
```
fakeroot make-kpkg --initrd --append-to-version=-utrace binary-arch
```

编译完成后，在 `linux-source-3.0.0/` 的上级目录会生成 linux-headers/image/debug-symbol 的 deb 安装包，直接用 `sudo dpkg -i *.deb` 安装即可。重启后选择新内核进入即可用 SystemTap 进行用户态程序跟踪。

## 参考资料

- https://help.ubuntu.com/community/Kernel/Compile#Alternate_Build_Method:_The_Old-Fashioned_Debian_Way

