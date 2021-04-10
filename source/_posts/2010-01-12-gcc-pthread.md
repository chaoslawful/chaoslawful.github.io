---
title: GCC 中 -pthread 和 -lpthread 的区别
tags:
    - GCC
    - C
    - C++
date: 2010-01-12 17:49:00
---

用 GCC 编译使用了 POSIX thread 的程序时通常需要加额外的选项，以便使用 thread-safe 的库及头文件，一些老的书里说直接增加链接选项 `-lpthread` 就可以了，像这样：
```bash
$ gcc -c x.c
$ gcc x.o -ox -lpthread
```

而 GCC 手册里则指出应该在编译和链接时都增加 `-pthread` 选项，像这样：
```bash
$ gcc -pthread -c x.c
$ gcc x.o -ox -pthread
```

那么 `-pthread` 相比于 `-lpthread` 链接选项究竟多做了什么工作呢？我们可以在 verbose 模式下执行一下对应的 GCC 命令行看出来。下面是老式的直接加 `-lpthread` 链接选项的输出结果：
```bash
$ gcc -v -c x.c
...
/usr/lib/gcc/i486-linux-gnu/4.2.4/cc1 -quiet -v x.c -quiet -dumpbase x.c
-mtune=generic -auxbase x -version -fstack-protector -fstack-protector -o /tmp/cch4ASTF.s
...
as --traditional-format -V -Qy -o x.o /tmp/cch4ASTF.s
...
$ gcc -v x.o -ox -lpthread
...
 /usr/lib/gcc/i486-linux-gnu/4.2.4/collect2 --eh-frame-hdr -m elf_i386 --hash-style=both
-dynamic-linker /lib/ld-linux.so.2 -ox
/usr/lib/gcc/i486-linux-gnu/4.2.4/../../../../lib/crt1.o
/usr/lib/gcc/i486-linux-gnu/4.2.4/../../../../lib/crti.o
/usr/lib/gcc/i486-linux-gnu/4.2.4/crtbegin.o
-L/opt/intel/Compiler/11.1/046/tbb/ia32/cc4.1.0_libc2.4_kernel2.6.16.21/lib/../lib
-L/usr/lib/gcc/i486-linux-gnu/4.2.4
-L/usr/lib/gcc/i486-linux-gnu/4.2.4
-L/usr/lib/gcc/i486-linux-gnu/4.2.4/../../../../lib
-L/lib/../lib
-L/usr/lib/../lib
-L/opt/intel/Compiler/11.1/046/lib/ia32
-L/opt/intel/Compiler/11.1/046/tbb/ia32/cc4.1.0_libc2.4_kernel2.6.16.21/lib
-L/usr/lib/gcc/i486-linux-gnu/4.2.4/../../..
x.o -lpthread -lgcc --as-needed -lgcc_s --no-as-needed -lc -lgcc
--as-needed -lgcc_s --no-as-needed
/usr/lib/gcc/i486-linux-gnu/4.2.4/crtend.o /usr/lib/gcc/i486-linux-gnu/4.2.4/../../../../lib/crtn.o
```

下面是在编译和链接时分别指定 `-pthread` 选项的输出结果：
```bash
$ gcc -v -pthread -c x.c
...
/usr/lib/gcc/i486-linux-gnu/4.2.4/cc1 -quiet -v -D_REENTRANT
 x.c -quiet -dumpbase x.c
-mtune=generic -auxbase x -version -fstack-protector -fstack-protector -o /tmp/cc205IQf.s
...
as --traditional-format -V -Qy -o x.o /tmp/cc205IQf.s
...
$ gcc -v x.o -ox -pthread
/usr/lib/gcc/i486-linux-gnu/4.2.4/collect2 --eh-frame-hdr -m elf_i386 --hash-style=both
-dynamic-linker /lib/ld-linux.so.2 -ox
/usr/lib/gcc/i486-linux-gnu/4.2.4/../../../../lib/crt1.o
/usr/lib/gcc/i486-linux-gnu/4.2.4/../../../../lib/crti.o
/usr/lib/gcc/i486-linux-gnu/4.2.4/crtbegin.o
-L/opt/intel/Compiler/11.1/046/tbb/ia32/cc4.1.0_libc2.4_kernel2.6.16.21/lib/../lib
-L/usr/lib/gcc/i486-linux-gnu/4.2.4
-L/usr/lib/gcc/i486-linux-gnu/4.2.4
-L/usr/lib/gcc/i486-linux-gnu/4.2.4/../../../../lib
-L/lib/../lib
-L/usr/lib/../lib
-L/opt/intel/Compiler/11.1/046/lib/ia32
-L/opt/intel/Compiler/11.1/046/tbb/ia32/cc4.1.0_libc2.4_kernel2.6.16.21/lib
-L/usr/lib/gcc/i486-linux-gnu/4.2.4/../../..
x.o -lgcc --as-needed -lgcc_s --no-as-needed <strong>-lpthread</strong>
 -lc -lgcc
--as-needed -lgcc_s --no-as-needed
/usr/lib/gcc/i486-linux-gnu/4.2.4/crtend.o /usr/lib/gcc/i486-linux-gnu/4.2.4/../../../../lib/crtn.o
```

可见编译选项中指定 `-pthread` 会附加一个宏定义 **-D_REENTRANT** ，该宏会导致 libc 头文件选择那些 thread-safe 的实现；链接选项中指定 `-pthread` 则同 `-lpthread` 一样，只表示链接 POSIX thread 库。由于 libc 用于适应 thread-safe 的宏定义可能变化，因此在编译和链接时都使用 `-pthread` 选项而不是传统的 `-lpthread` 能够保持向后兼容，并提高命令行的一致性。

