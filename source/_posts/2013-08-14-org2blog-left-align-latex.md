---
title: org2blog 如何发布左对齐的 LaTeX 公式
tags:
    - Emacs
    - org2blog
date: 2013-08-14 09:44:00
---

通常情况下 Wordpress 上的基于 MathJax 显示的 LaTeX 插件总是将独立公式显示为居中对齐的，但当文章中的公式都比较短时居中对齐就显得不太美观。若想让独立公式在文章中总是左对齐，可以在编辑 org2blog 内容时在正文开始前加入如下片段：

```javascript
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  displayAlign: "left",
  displayIndent: "2em"
});
</script>
```

这里的 `displayIndent` 可以控制左对齐公式的缩进宽度，若设为 "0" 就表示同正文平齐。下面是具体的例子：

$$
\int_a^b x\,dx
$$

$$
f(x)=\int_{-\infty}^{+\infty}\mathrm{e}^s x\,\mathrm{d}s
$$

P.S. 虽然 MathJax 支持直接识别很多 LaTeX 的环境块，但通常 Wordpress 上的 LaTeX 插件仅在检测到自己识别的 LaTeX 标签时才会生成加载 MathJax 的代码，如果 blog 中只使用了 LaTeX 环境块就会导致无法加载 MathJax 而全部显示为普通文本。解决方法是在正文某处显式写一个空内容的可识别 LaTeX 标签欺骗 LaTeX 插件加载 MathJax。

参考：https://github.com/mathjax/mathjax-docs/wiki/MathML-alignment-Left-or-Right
