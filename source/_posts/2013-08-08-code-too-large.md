---
title: 编译 Java 程序时出现 code too large 错误的分析
tags:
    - Java
    - Eclipse
    - ANTLR
date: 2013-08-08 17:48:00
---

我参与的一个实际项目里用到了 ANTLR 3.x 进行 parser 代码自动生成，其中含有大量的 static field 用作状态转移查找表。由于一个 class 中所有 static field 都是放在一个匿名 method 中统一进行初始化，而 Java VM 规范规定一个 class 中单个 method 中 bytecode 长度最多不能超过 65535 bytes，这个自动生成的 parser class 始终无法在 javac 上编译，总是会提示 `error: code too large` 。但问题是在 Eclipse 中编译使用该文件完全没有问题，由于 65535 bytes 的限制是 Java VM 规范所规定，同生成 class 的编译器没有关系，因此原因只可能是 ecj 生成的 bytecode 比 javac 少，这就有点儿诡异了。

要查看 class 中的字节码可以用 `javap` 程序完成，通过下面的命令可以将有问题的 class （这里假设是 Test）的静态初始化匿名 method 中所有 bytecode 输出：
```bash
javap -c Test | sed -ne '/static {}/,/return/ p'
```
将 Test class 的 static field 用对分法逐渐裁剪到 javac 刚好能编译过去，然后用上述命令导出 javac 编译结果和 ecj 编译结果进行对比分析，果然发现了一些差异（测试代码可从[此处](/files/Test.java.gz)下载）。

例如如下的这段 Java 代码：
```java
public static final long[][] FOLLOW_DEFAULT_in_table_option26937 =
    new long[][]{new long[]{0x0000000000000000L,0x0000000002000000L}};
```
由 ecj 编译产出的 bytecode 为：
|   **offset** | **bytecode**      | **stack** [before]->[after]  |
|-----------:|:----------------|:--------------------------|
|          0 | iconst_1        | -> 1                      |
|          1 | anewarray #idx  | count -> arrayref         |
|          4 | dup             | value -> value, value     |
|          5 | iconst_0        | -> 0                      |
|          6 | iconst_2        | -> 2                      |
|          7 | newarray long   | count -> arrayref         |
|          9 | dup             | value -> value, value     |
|         10 | iconst_1        | -> 1                      |
|         11 | ldc2_w #idx     | -> value                  |
|         14 | lastore         | arrayref, index, value -> |
|         15 | aastore         | arrayref, index, value -> |
|         16 | pustatic #idx   | value ->                  |

由 javac 编译产出的 bytecode 为：
|   **offset** | **bytecode**      | **stack** [before]->[after] |
|-----------:|:----------------|:--------------------------|
|          0 | iconst_1        | -> 1                      |
|          1 | anewarray #idx  | count -> arrayref         |
|          4 | dup             | value -> value, value     |
|          5 | iconst_0        | -> 0                      |
|          6 | iconst_2        | -> 2                      |
|          7 | newarray long   | count -> arrayref         |
|          9 | dup             | value -> value, value     |
|         10 | iconst_0        | -> 0                      |
|         11 | iconst_0        | -> 0                      |
|         12 | lastore         | arrayref, index, value -> |
|         13 | dup             | value -> value, value     |
|         14 | iconst_1        | -> 1                      |
|         15 | ldc2_w #idx     | -> value                  |
|         18 | lastore         | arrayref, index, value -> |
|         19 | aastore         | arrayref, index, value -> |
|         20 | pustatic #idx   | value ->                  |

可以看到 javac 多生成了 offset 10~13 的 bytecode，分析可知这段代码的功能是将内部 long[] 数组第一个元素置为 0。也就是说，ecj 利用了 Java VM 规范中的默认初始化约定，不会为显式初始化为 0 的元素生成赋值 bytecode，但 javac 却会傻傻地按照初始化列表逐个生成 bytecode。这样当非 0 元素初始化很少时，ecj 就比 javac 生成的静态初始化 bytecode 少得多了，自然会发生 javac 编译报 code too large 错误但 ecj 正常的情况。

不过从另一方面看，ANTLR 3.x 生成的 parser 代码也确实没有考虑 Java VM 的 bytecode 长度限制，一旦语法复杂到一定程度，生成的代码很可能连 ecj 都无法编译通过。这是一个比较严重的设计疏忽，希望 ANTLR 后续版本使用变通方法对这一点进行改进。

## 参考资料

- http://en.wikipedia.org/wiki/Java_bytecode_instruction_listings
- http://marxsoftware.blogspot.com/2010/01/reproducing-code-too-large-problem-in.html
- http://stackoverflow.com/questions/243097/javac-error-code-too-large
- http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.6
- http://www.certpal.com/blogs/2009/08/code-too-large-for-try-statement/
