---
title: "PHP 中的二进制安全"
date: 2016-05-14T21:47:46+08:00
publishdate: 2016-05-14
lastmod: 2020-01-17
draft: false
tags: ["php"]
---
## 二进制安全是什么
先看一段代码：

```php
<?php

$string1 = "Hello"; 
$string2 = "Hello\0Hello";

// 返回0, 由于是非二进制安全，误判为相等
echo strcoll($string1, $string2);

// 返回<0,不相等
echo strcmp($string1, $string2);  
```

这是为什么呢？PHP 是基于 C 实现的，PHP 代码都会被 Zend引擎编译成 opcode，最终作为 C语言去执行。而对于 C语言中 "\0" 是字符串的结束符，它读到 "\0" 就会默认字符读取已经结束，从而抛掉后面的字符串。

```c
main(){  
    char ab[] = "Hello";  
    char ac[] = "Hello\0Hello";
    
    // 返回0, 由于是非二进制安全，误判为相等
    strcmp(ab, ac); 
}
```

有一个二进制安全的定义:

> 程序不会对其中的数据做任何限制、过滤、或者假设 —— 数据在写入时是什么样的，它被读取时就是什么样。

## PHP是如何实现二进制安全的
既然 PHP 是基于 C 实现的，C 字符串类型不是二进制安全的，PHP 又是如何实现的呢？这就是数据结构的功劳了。 PHP 的内核中，是如此定义字符串类型的

```c
struct{
    char *val;
    int len;
} str;
```

`val` 是指向字符串内存的指针，`len` 表示该字符串的长度，无论是否遇到 "\0" 字符，C 都按照 `len` 长度读取该字符串。

## 参考 & 扩展阅读
- [PHP有趣的细节：二进制安全](http://wuxinjie.github.io/php-02/)


