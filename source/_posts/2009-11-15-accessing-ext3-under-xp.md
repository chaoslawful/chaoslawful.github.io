---
title: Accessing EXT2/3 partition under Windows XP
tags:
    - WinXP
    - Ext3
date: 2009-11-25 23:37:00
---

After some googling I found several ways to do the job:

1. [Explore2fs](http://www.chrysocome.net/explore2fs)
   * *Pros:* Open-source; No installing needed
   * *Cons:* Read only; Exporting file is too slow...
2. [Ext2 IFS](http://www.fs-driver.org/index.html)
   * *Pros:* Mount EXT2/3 partition to a real disk letter; High accessing speed
   * *Cons:* Installing needed; No source available
3. [Ext2Fsd](http://www.ext2fsd.com/)
   * *Pros:* Opensource; Mount EXT2/3 partition to a real disk letter
   * *Cons:* Installing needed; Complex operations
4. [Diskinternal Linux Reader](http://www.diskinternals.com/linux-reader/)
   * *Pros:* User-friendly interface with lots of whistles
   * *Cons:* Installing needed; No source available; Not support UTF-8 charset
5. [Virtual Volumes](http://www.chrysocome.net/virtualvolumes)
   * *Pros:* Opensource
   * *Cons:* Installing needed; Not completed yet...

Finally I chose Ext2 IFS :-)

