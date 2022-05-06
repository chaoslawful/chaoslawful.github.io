---
title: Java 正则库 bug 一例
tags:
    - Java
    - regexp
date: 2014-03-23 22:16
---

一日同事尝试用 Java 正则库匹配某模式总是抛出 `String index out of range` 异常，百思不得其解。经过一番研究，得到了这样一个最小复现用例：

```java
import java.util.regex.*;
public class RegexBug {
    public static void main(String[] args) throws Exception {
        String s = "x"+Character.highSurrogate(0x10000)+Character.lowSurrogate(0x10000);
        Pattern p = Pattern.compile("(.)*xx");
        Matcher m = p.matcher(s);
        if(m.find()) {
            System.out.println("matched");
        }
    }
}
```
<!-- more -->

跟踪后发现这是 Java 正则库中的一个实现 bug，用例模式中的 `(.)*` 会被 Pattern 类解析为 GroupCurly 算子，而该算子内对应贪婪匹配模式的 `match0` 方法实现有误，当某次匹配不成功需要进行回溯时，其假定 `.` 匹配的任意字符都是相同长度而每次都减去了最后一次匹配成功的字符长度；然而 Java 内部使用 UTF-16 编码保存字符串，仅有 Unicode BMP 内的字符（即码点小于等于 U+FFFF 的字符）才能用单个 char 表示，Unicode SMP 内的字符（即码点大于 U+FFFF 的字符）都需要用 2 个替代码元表示 1 个字符，当匹配目标字符串中有 BMP/SMP 区字符混合存在的情况且匹配不成功时，回溯逻辑的字符等长假定就会失效，导致字符串下标计算出错而出现访问越界异常。不过同在 Pattern 类中表达非捕获重复算子的 Curly 就没有这个问题，如果将以上用例中的 `(.)*` 改为 `.*` 就一切正常了。

经过一番搜索，发现该 bug 已经在 2013-02-01 被人提交给 JDK buglist ([见这里](http://bugs.java.com/bugdatabase/view_bug.do?bug_id=8007395)) ，JDK 5~7 都受其影响，不过该 bug 已经在 JDK 8 中修复了。考虑到 Java 正则库实现细节非常不堪，也许还会有更多的坑在实践中被发现，定期关注一下 `java.util.regex` 相关的 bug ([见这里](https://search.oracle.com/search/search?search_p_main_operator=all&start=1&group=bugs.sun.com&q=java.util.regex)) 应该是个好主意……
