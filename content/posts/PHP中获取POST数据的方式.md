---
title: "PHP 中获取 POST 数据的方式"
date: 2016-09-13T18:42:34+08:00
publishdate: 2016-09-13
lastmod: 2020-01-17
draft: false
tags: ["php", "web", "http"]
---
||form-data|x-www-form-urlencoded|raw|
|:--|:----:|:-------------------:|:-:|
|$_POST|**推荐**|**推荐**|无效|
|$GLOBALS['HTTP_RAW_POST_DATA']|无效|无效|有效|
|php://input|无效|有效|**推荐**|

### $_POST 方式
---
通过 HTTP POST 方法传递的变量组成的数组。是自动全局变量。

Coentent-Type 仅在取值为 application/x-www-data-urlencoded 或 multipart/form-data 两种情况下，PHP才会将 HTTP 请求数据包中相应的数据填入全局变量 $_POST。

### $GLOBALS['HTTP_RAW_POST_DATA'] 方式
---
此变量仅在碰到未识别 MIME 类型的数据时产生。

PHP 默认识别的数据类型是 application/x-www.form-urlencoded 标准的数据类型。如果 POST 过来的数据不是 PHP 能够识别的，比如 text/xml 或者 soap 等可以用 $GLOBALS['HTTP_RAW_POST_DATA']方式来接收。

不过，访问原始 POST 数据的更好方法是 php://input。

### php://input 方式
---
php://input 允许读取 POST 的原始数据。和 $HTTP_RAW_POST_DATA 比起来，它给内存带来的压力较小，并且不需要任何特殊的 php.ini 设置。

php://input 不能用于 enctype="multipart/form-data"。

### 参考 & 扩展阅读
---
[$GLOBALS['HTTP_RAW_POST_DATA'] 和$_POST的区别](http://blog.csdn.net/china_skag/article/details/7284227)
[深入剖析PHP输入流 php://input 与 POST/GET 的区别](http://www.nowamagic.net/academy/detail/12220520)