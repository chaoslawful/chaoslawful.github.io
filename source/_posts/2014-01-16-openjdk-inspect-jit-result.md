---
title: OpenJDK 中查看 JIT 编译结果的方法
tags:
    - Java
    - JIT
date: 2014-01-16 19:35:00
---

## 背景

大家都知道 JVM 有 HotSpot 引擎可以对热代码路径进行有效的 JIT 优化，大幅度提升计算密集代码的性能，但 HotSpot 对自己编写的 Java 代码时进行了哪些 JIT 优化？优化后生成的 native code 长什么样？知道了这些信息之后，才能有的放矢地改进自己的代码。
<!-- more -->

## 初步查看 JIT 工作情况

以如下 Java 代码为例：
```java
// Accumulator.java
public class Accumulator {
	public static void main(String[] args) {
		int max = Integer.parseInt(args[0]);
		System.out.println(addAllSqrts(max));
	}

	static double addAllSqrts(int max) {
		double accum = 0;
		for(int i = 0; i < max; i++) {
			accum = addSqrt(accum, i);
		}
		return accum;
	}

	static double addSqrt(double a, int b) {
		return a + sqrt(b);
	}

	static double sqrt(int a) {
		return Math.sqrt(a);
	}
}
```

一般情况下，我们只需要知道 JIT 内联函数是否满足自己的期望即可，可以用如下命令行达到这个目的：
```bash
javac Accumulator.java
java -XX:+PrintCompilation Accumulator 50000
```
注意默认情况下一个方法至少要被调用 10k 次以上才可能被 JIT 优化，故这里运行时给出的循环参数也要大于该值，输出结果如下：

<pre>
     61    1             Accumulator::addSqrt (7 bytes)
     62    2             Accumulator::sqrt (6 bytes)
     67    3 %           Accumulator::addAllSqrts @ 4 (23 bytes)
...
</pre>

各个数据列的含义分别为：
| **列1** | **列2** | **列3** | **列4** | **列5** |
|:---|:---|:---|:---|:---|
| 自 JVM 启动到尝试优化的时间(ms) | 尝试优化的顺序(从 1 开始) | 特殊方法标识[^fn:1] | 方法全名及生成代码长度 | 若有 "made not entrant" 或 "made zombie" 字样就表明没有进行优化 |

用如下命令行可以更多地了解内联优化的实际情况以及优化发生的级别：
```bash
java -XX:+PrintCompilation -XX:+UnlockDiagnosticVMOptions -XX:+PrintInlining -XX:+TieredCompilation Accumulator 50000
```
输出结果如下：

<pre>
     50    1       3       java.lang.String::equals (81 bytes)
     51    2       3       java.lang.Object::<init> (1 bytes)
     53    3     n 0       java.lang.System::arraycopy (native)   (static)
     53    4       3       java.lang.String::charAt (29 bytes)
                              @ 18  java/lang/StringIndexOutOfBoundsException::<init> (not loaded)   not inlineable
     54    6       3       java.lang.String::hashCode (55 bytes)
     55    5       3       java.lang.String::indexOf (70 bytes)
                              @ 66   java.lang.String::indexOfSupplementary (71 bytes)   callee is too large
     56    7       3       java.lang.String::length (6 bytes)
     59    8       3       java.lang.Math::min (11 bytes)
     61    9       3       Accumulator::addSqrt (7 bytes)
                              @ 2   Accumulator::sqrt (6 bytes)
                                @ 2   java.lang.Math::sqrt (5 bytes)   intrinsic
     62   10       3       Accumulator::sqrt (6 bytes)
                              @ 2   java.lang.Math::sqrt (5 bytes)   intrinsic
     64   11       4       Accumulator::addSqrt (7 bytes)
...
</pre>

这里新增的第 3 列代表了优化发生的级别（Tier 0~4，其中 Tier 0 为 Interpreter，Tier 1~3 为 client 模式的优化策略，Tier 4 为 server 模式的优化策略）

## 查看 JIT 生成的汇编代码

为了更好地了解 JIT 优化效果，查看生成的 native code 是很有必要的，不过 OpenJDK 并没有自带反汇编插件 `hsdis-*.so` ，需要手工安装一下该插件。一般来说有这样几种安装方法：
1. 对于 Ubuntu 系的发行版，可以直接用命令 `sudo apt-get install libopenjdk-hsdis` 安装该插件
2. 用别人从 OpenJDK 中扣出来的代码自行编译安装，具体见 [这里](https://github.com/drazzib/openjdk-hsdis)
3. 按照 [HotSpotInternals - PrintAssembly](https://wikis.oracle.com/display/HotSpotInternals/PrintAssembly) 中的描述自己从 OpenJDK 源码树中编译

安装完成后，即可用一下命令输出所有 JIT 后函数的 native code 汇编代码：
```bash
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly Accumulator 50000
```
输出结果如下：

<pre>
Java HotSpot(TM) 64-Bit Server VM warning: PrintAssembly is enabled; turning on DebugNonSafepoints to gain additional output
Loaded disassembler from hsdis-amd64.so
Decoding compiled method 0x00007ffd9905ee50:
Code:
[Disassembling for mach='i386:x86-64']
[Entry Point]
[Verified Entry Point]
[Constants]
  # {method} 'addSqrt' '(DI)D' in 'Accumulator'
  # parm0:    xmm0:xmm0   = double
  # parm1:    rsi       = int
  #           [sp+0x20]  (sp of caller)
  0x00007ffd9905ef80: sub    $0x18,%rsp
  0x00007ffd9905ef87: mov    %rbp,0x10(%rsp)    ;*synchronization entry
                                                ; - Accumulator::addSqrt@-1 (line 16)
  0x00007ffd9905ef8c: vcvtsi2sd %esi,%xmm1,%xmm1
  0x00007ffd9905ef90: vsqrtsd %xmm1,%xmm1,%xmm1
  0x00007ffd9905ef94: vaddsd %xmm1,%xmm0,%xmm0  ;*dadd
                                                ; - Accumulator::addSqrt@5 (line 16)
  0x00007ffd9905ef98: add    $0x10,%rsp
  0x00007ffd9905ef9c: pop    %rbp
  0x00007ffd9905ef9d: test   %eax,0xc5f905d(%rip)        # 0x00007ffda5658000
                                                ;   {poll_return}
  0x00007ffd9905efa3: retq
  0x00007ffd9905efa4: hlt
  0x00007ffd9905efa5: hlt
...
</pre>

还可以增加选项 `-XX:PrintAssemblyOptions=hsdis-print-bytes` 额外输出指令实际字节序列：
```bash
java -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly -XX:PrintAssemblyOptions=hsdis-print-bytes Accumulator 50000
```
输出结果为：

<pre>
Java HotSpot(TM) 64-Bit Server VM warning: PrintAssembly is enabled; turning on DebugNonSafepoints to gain additional output
Loaded disassembler from hsdis-amd64.so
Decoding compiled method 0x00007f482105ee50:
Code:
[Disassembling for mach='i386:x86-64']
[Entry Point]
[Verified Entry Point]
[Constants]
  # {method} 'sqrt' '(I)D' in 'Accumulator'
  # parm0:    rsi       = int
  #           [sp+0x20]  (sp of caller)
  0x00007f482105ef80: sub    $0x18,%rsp         ;...4881ec18 000000
  0x00007f482105ef87: mov    %rbp,0x10(%rsp)    ;...48896c24 10
                                                ;*synchronization entry
                                                ; - Accumulator::sqrt@-1 (line 20)
  0x00007f482105ef8c: vcvtsi2sd %esi,%xmm0,%xmm0  ;...c5fb2ac6
  0x00007f482105ef90: vsqrtsd %xmm0,%xmm0,%xmm0  ;...c5fb51c0
                                                ;*invokestatic sqrt
                                                ; - Accumulator::sqrt@2 (line 20)
  0x00007f482105ef94: add    $0x10,%rsp         ;...4883c410
  0x00007f482105ef98: pop    %rbp               ;...5d
  0x00007f482105ef99: test   %eax,0xc048061(%rip)        # 0x00007f482d0a7000
                                                ;...85056180 040c
                                                ;   {poll_return}
  0x00007f482105ef9f: retq                      ;...c3
[Exception Handler]
[Stub Code]
  0x00007f482105efa0: jmpq   0x00007f482105eda0  ;...e9fbfdff ff
                                                ;   {no_reloc}
[Deopt Handler Code]
  0x00007f482105efa5: callq  0x00007f482105efaa  ;...e8000000 00
  0x00007f482105efaa: subq   $0x5,(%rsp)        ;...48832c24 05
  0x00007f482105efaf: jmpq   0x00007f4821038f00  ;...e94c9ffd ff
                                                ;   {runtime_call}
  0x00007f482105efb4: hlt                       ;...f4
...
</pre>

上述命令总是输出所有方法的 JIT 结果，当结果很多时难以看出重点，可以将 `-XX:+PrintAssembly` 选项改为 `-XX:CompileCommand=print,*class.method` 来限制输出哪些方法，例如：
```bash
java -XX:+UnlockDiagnosticVMOptions -XX:CompileCommand='print,*Accumulator.sqrt' Accumulator 50000
```
这样输出结果就变得很少了：

<pre>
CompilerOracle: print *Accumulator.sqrt
Java HotSpot(TM) 64-Bit Server VM warning: printing of assembly code is enabled; turning on DebugNonSafepoints to gain additional output
Compiled method (c2)      62    2             Accumulator::sqrt (6 bytes)
 total in heap  [0x00007fcacc1d18d0,0x00007fcacc1d1a98] = 456
 relocation     [0x00007fcacc1d19f0,0x00007fcacc1d19f8] = 8
 main code      [0x00007fcacc1d1a00,0x00007fcacc1d1a20] = 32
 stub code      [0x00007fcacc1d1a20,0x00007fcacc1d1a38] = 24
 oops           [0x00007fcacc1d1a38,0x00007fcacc1d1a40] = 8
 scopes data    [0x00007fcacc1d1a40,0x00007fcacc1d1a50] = 16
 scopes pcs     [0x00007fcacc1d1a50,0x00007fcacc1d1a90] = 64
 dependencies   [0x00007fcacc1d1a90,0x00007fcacc1d1a98] = 8
Loaded disassembler from hsdis-amd64.so
Decoding compiled method 0x00007fcacc1d18d0:
Code:
[Disassembling for mach='i386:x86-64']
[Entry Point]
[Verified Entry Point]
[Constants]
  # {method} 'sqrt' '(I)D' in 'Accumulator'
  # parm0:    rsi       = int
  #           [sp+0x20]  (sp of caller)
  0x00007fcacc1d1a00: sub    $0x18,%rsp
  0x00007fcacc1d1a07: mov    %rbp,0x10(%rsp)    ;*synchronization entry
                                                ; - Accumulator::sqrt@-1 (line 20)
  0x00007fcacc1d1a0c: vcvtsi2sd %esi,%xmm0,%xmm0
  0x00007fcacc1d1a10: vsqrtsd %xmm0,%xmm0,%xmm0  ;*invokestatic sqrt
                                                ; - Accumulator::sqrt@2 (line 20)
  0x00007fcacc1d1a14: add    $0x10,%rsp
  0x00007fcacc1d1a18: pop    %rbp
  0x00007fcacc1d1a19: test   %eax,0x9ea55e1(%rip)        # 0x00007fcad6077000
                                                ;   {poll_return}
  0x00007fcacc1d1a1f: retq
[Exception Handler]
[Stub Code]
  0x00007fcacc1d1a20: jmpq   0x00007fcacc1ceda0  ;   {no_reloc}
[Deopt Handler Code]
  0x00007fcacc1d1a25: callq  0x00007fcacc1d1a2a
  0x00007fcacc1d1a2a: subq   $0x5,(%rsp)
  0x00007fcacc1d1a2f: jmpq   0x00007fcacc1a8f00  ;   {runtime_call}
  0x00007fcacc1d1a34: hlt
  0x00007fcacc1d1a35: hlt
  0x00007fcacc1d1a36: hlt
  0x00007fcacc1d1a37: hlt
OopMapSet contains 0 OopMaps
</pre>

**Tips：** 用如下命令能看到 JVM 支持的所有内部选项及其当前值，可以用于排查环境问题或进行细节调整：
```bash
java -XX:+AggressiveOpts -XX:+UnlockDiagnosticVMOptions -XX:+UnlockExperimentalVMOptions -XX:+PrintFlagsFinal -version
```

## 参考资料
+ [JVM JIT for Dummies](http://www.slideshare.net/CharlesNutter/javaone-2012-jvm-jit-for-dummies)
+ [HotSpotInternals - PrintAssembly](https://wikis.oracle.com/display/HotSpotInternals/PrintAssembly)
+ [How to use -XX:+UnlockDiagnosticVMOptions -XX:CompileCommand=print option with JVM HotSpot](http://stackoverflow.com/questions/9341083/how-to-use-xxunlockdiagnosticvmoptions-xxcompilecommand-print-option-with-j)

[^fn:1]: `!` 表示是异常处理函数， `n` 表示是 native 函数， `%` 表示进行了 OSR（On-Stack Replace） 优化
