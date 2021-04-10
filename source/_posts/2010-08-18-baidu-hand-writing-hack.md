---
title: Baidu 首页手写输入法交互数据格式分析
tags:
    - hack
date: 2010-08-18 22:38:00
---

手写识别接口以 HTTP POST 方式访问 http://hw.baidu.com ， `Content-Type` 为 `application/x-www-form-urlencoded` ，POST body 中包含如下 2 个参数：
- **type** - 总是为 1；
- **wd** - 为手写笔划矢量数据，格式如下：
  - 每条矢量笔划数据都是形如 `x1,y1,x2,y2,...` 的坐标列表，其中 x 和 y 坐标的有效值范围在 4~209 之间（屏幕坐标系，左上角为原点，向右向下坐标递增），以字符 "a" 作为分隔符将 10 进制坐标数值序列化为字符串。例如一个笔划经过的坐标若为 `(15,15)-(15,30)` ，则该笔划序列化后的字符串为 "`15a15a15a30`" ；而另一条笔划经过的坐标若为 `(15,30)-(30,30)` ，则序列化后的字符串为"`15a30a30a30`"；
  - 多条矢量笔划数据字符串之间以 "aa" 分隔后作为最终数据串，例如上述 2 条笔划组成的最终数据串为 "`15a15a15a30aa15a30a30a30`"

对应上述笔划数据的完整手写识别请求为：
```http
POST http://hw.baidu.com HTTP/1.1
Content-type: application/x-www-form-urlencoded
Content-length: 34

wd=15a15a15a30aa15a30a30a30&type=1
```

响应体为 JSON 对象，包含如下 2 个字段：
- **t** - 总是为 1；
- **s** - 为候选字列表字符串，其中包含 10 个候选字，以 Unicode 转义序列表示

对应上述手写识别请求的响应为：
```http
HTTP/1.1 200 OK
Date: Wed, 18 Aug 2010 14:39:09 GMT
Server: hw.baidu.com
Content-Length: 74
Content-Type: text/html;charset=gb2312
Cache-Control: private
Expires: Wed, 18 Aug 2010 15:39:09 GMT
Connection: Keep-Alive

{"s":"\u004c\u4e5a\u0063\u0043\u5315\u4e04\u4e0a\u006c\u535c\u4e00","t":1}
```
经过简单的解码即可知候选字列表为 "L乚cC匕丄上l卜一"。

对此接口进行一些简单的包装即可实现各种在线手写识别应用，不过有没有可能用这个做 OCR？有兴趣的同学可以尝试尝试 :-)
