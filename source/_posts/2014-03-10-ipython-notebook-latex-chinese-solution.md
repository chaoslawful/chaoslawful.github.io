---
title: IPython Notebook 转成 LaTeX 时的中文问题解决方案
tags:
    - IPython
    - i18n
date: 2014-03-10 10:32:00
---

从 IPython 1.0 开始 IPython Notebook 就自带了 nbconvert 工具，可以将 notebook 转换为 HTML/LaTeX/Markdown/reStructure/Slides 等多种外部格式，以方便内容的快速重用。从内容展现的一致性上看，将 notebook 转成 LaTeX 再处理为 PDF 文件是表现最好的，因此使用频率也最高，然而当 notebook 中出现中文时，默认生成的 LaTeX 文件无法进行支持，会导致最终生成的 PDF 文件中中文部分都是空白。经过实验，对生成的 LaTeX 文件进行如下手工修改可以快速修正此问题：

+ 首先用 nbconvert 将 notebook 转为 LaTeX 文件：
```bash
ipython nbconvert --to latex example.ipynb
```
+ 在生成的 LaTeX 文件里 `\documentclass` 行之后增加如下内容：

<pre>
\usepackage{fontspec, xunicode, xltxtra}
\usepackage[slantfont, boldfont]{xeCJK} % 允许斜体和粗体
\setCJKmainfont{WenQuanYi Micro Hei} % 默认中文字体
\setCJKmonofont{WenQuanYi Micro Hei Mono} % 中文等宽字体
\setmainfont{TeX Gyre Pagella} % 英文衬线字体
\setmonofont{Monaco} % 英文等宽字体
\setsansfont{Trebuchet MS} % 英文无衬线字体
\punctstyle{kaiming} % 开明式标点格式: 句末点号用全角, 其他半角
</pre>

+ 注释掉生成的 LaTeX 文件里 `\usepackage{babel}` 这一行
+ (可选) 将生成的 LaTeX 文件里 `\date{...}` 和 `\author{...}` 中的内容改为需要的日期和作者信息
+ 最后用支持 UTF-8 的 `xelatex` 命令直接将修改后的 LaTeX 文件转为 PDF 即可：
```bash
xelatex example.tex
```

另外，最后转 PDF 时若出现类似这样的错误：

<pre>
> Runaway argument?
> ! File ended while scanning use of @writefile.
>
> par
> l.110 begin{document}
</pre>

可以直接删除 `*.aux` 文件再重新转换即可。

