---
title: 修复 GNUCash 数据库中的乱码
tags:
    - Linux
    - GNUCash
date: 2012-11-18 01:58:00
---

GNUCash 是一款很好的记账软件，但在使用 MySQL 作为存储后端时，由于其对连接字符集的设置有漏洞，容易出现记账备注中的中文变乱码的情况。此时，可以在 MySQL 中执行如下 SQL 语句完成修复工作：
```sql
UPDATE splits SET memo = CONVERT(BINARY(CONVERT(memo USING latin1)) USING utf8) WHERE CHAR_LENGTH(CONVERT(BINARY(CONVERT(memo USING latin1)) USING utf8)) != CHAR_LENGTH(memo);
```

