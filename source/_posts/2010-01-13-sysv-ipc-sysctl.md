---
title: Linux 中针对 SysV IPC 的内核参数调整
tags:
    - Linux
    - IPC
date: 2010-01-13 10:34:00
---

SysV IPC 包括 Semaphore、Shared Memory 和 Message Queue 这 3 类进程间通信手段，虽然 POSIX.1-2001 实时接口标准规定了另一套提供相同手段但更一致化的接口（POSIX IPC），但 SysV IPC 仍然有相当数量的用户。
<!-- more -->

通过调整一些内核参数，可以更改 SysV IPC 对数据的固有限制，相关参数对应的控制文件可在 `/proc/sys/kernel/` 目录下找到，也可以通过 `sysctl` 更改，现罗列如下：

| **控制文件路径** | **内核参数(通过 sysctl 更改时使用)** | **含义** |
| :--- | :--- | :--- |
| `/proc/sys/kernel/shmmax` | `kernel.shmmax` | 共享内存中最大内存块尺寸（byte），默认为 33554432，即 32MB |
| `/proc/sys/kernel/shmall` | `kernel.shmall` | 整个系统中共享内存的最大尺寸，以页（4KB）为单位，默认为 2097152，即 2M 个页，共计 8GB |
| `/proc/sys/kernel/shmmni` | `kernel.shmmni` | 整个系统中共享内存块的最大数量，默认为 4096 |
| `/proc/sys/kernel/msgmax` | `kernel.msgmax` | 消息队列中单条消息的最大尺寸（byte），默认为 8192 |
| `/proc/sys/kernel/msgmnb` | `kernel.msgmnb` | 默认的每个消息队列的最大尺寸（byte），默认为 16384 |
| `/proc/sys/kernel/msgmni` | `kernel.msgmni` | 整个系统中消息队列的最大数量，默认为 16 |
| `/proc/sys/kernel/sem`    | `kernel.sem` | 信号量的 4 个控制参数，从前向后分别是：单个信号量集合中容纳的最大信号量数量（默认为 250）、整个系统中信号量的最大数量（默认为 32000）、每个 semop 调用中允许的最大操作数量（默认为 32）、整个系统中信号量集合的最大数量（默认为 128） |

以上参数及含义都可以从 linux kernel 源码树的 `ipc/` 子目录下 SysV IPC 对应的实现文件中找到，其相关宏定义在 `include/linux/` 子目录下的相关头文件中。
