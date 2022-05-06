---
title: org2blog 发布语法高亮源码块的注意事项
tags:
    - Emacs
    - org2blog
    - org-mode
date: 2013-08-09 23:26:00
---

Org2blog 要在 Wordpress 中发布语法高亮的源码区块，需要做以下准备：
- Wordpress 中要安装 SyntaxHighlighter Evolved 插件
- emacs 中添加设置以将 pre 区块转换为 sourcecode 区块以让语法高亮插件工作： `(setq org2blog/wp-use-sourcecode-shortcode t)`
<!-- more -->

最后，org2blog 依赖于 org-mode 的 HTML 导出功能，后者总是会转义所有 XML/HTML 特殊字符，但当前版本的 SyntaxHighlighter Evolved 插件也总是会转义这些符号，造成双重转义使高亮输出发生混乱。解决方式是修改 SyntaxHighlighter Evolved 插件，将 `syntaxhighlighter.php` 中的 `encode_shortcode_contents_callback()` 函数改为如下形式：
```php
function encode_shortcode_contents_callback( $atts, $code = '', $tag = false ) {
    $this->encoded = true;
    // XXX: modified by wxz
    if (false === strpos($code, '<') && false === strpos($code, '>')) {
        // content already escaped
    } else {
        // content not escaped yet
        $code = str_replace( array_keys($this->specialchars), array_values($this->specialchars), htmlspecialchars( $code ) );
    }
    // XXX: ends here
    return '[' . $tag . $this->atts2string( $atts ) . "]{$code}[/$tag]";
}
```
这样可以在源码区块内容已经转义的情况下不再进行二次转义，从而解决输出混乱问题。

下面是一个语法高亮的例子：
```cpp
#include <stdio.h>
int main(int argc, char *argv[])
{
    printf ("Hello, world!\n");
    return 0;
}
```

