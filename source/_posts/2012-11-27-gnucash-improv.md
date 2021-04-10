---
title: 对 GNUCash 进行的一点儿改进
tags:
    - Linux
    - GNUCash
    - patch
date: 2012-11-27 00:40:00
---

当前版本的 GNUCash 在连接 MySQL 数据库时有个比较严重的字符编码问题：为了兼容 Unicode 字符，GNUCash 在 `gnc_dbi_mysql_session_begin()` 函数中主动执行 `SET NAMES 'utf8'` 语句设定 MySQL 连接字符编码为 UTF-8，但遗憾的是用来在连接故障时修复连接的 `gnc_dbi_verify_conn()` 函数并没有进行类似的操作。一旦数据库连接因网络问题意外断开，用户输入新交易时就会使用 `gnc_dbi_verify_conn()` 重新创建的新连接，而该连接的字符编码是默认的 latin1，直接后果就是输入中文等字符都变成了一堆乱码。

实际上 GNUCash 底层使用的 libdbi 库是考虑了连接字符编码问题的，其专门提供了一个 "encoding" 的设置选项用来对连接字符编码进行通用设置，底层的各个 driver 在创建连接时都会尊重该设置。所以只要简单地增加该选项，不管连接怎么断开重连，字符编码都能保持正常。

打上附件中的 fix_charset.patch 就可以修正此问题了。

另外，General Ledger 功能实际上非常好用，有了它就不需要在各个 Account 之间来回切换查帐了。但 GNUCash 目前默认在 General Ledger 中只显示最近一个月的交易，可苦了想对更老交易对账的用户。

我对其进行了小小的修改，打上附件中的 enh_general_ledger.patch 即可让 General Ledger 显示当前年度的所有交易，这样对账就非常方便了。

**更新：** 最近发现 General Ledger 实际上可以自定义显示日期范围，只要在进入 General Ledger 界面后选择菜单上的 View/Filter...，即可在弹出对话框的 Date 页选择显示起止日期，无需使用附件中的补丁。

- [修正数据库连接断开重连后字符编码错误的补丁](/files/fix_charset.patch.gz)
- [让 General Ledger 显示当前年度所有交易的补丁](/files/enh_general_ledger.patch.gz)

