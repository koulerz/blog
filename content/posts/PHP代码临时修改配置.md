---
title: "PHP 代码临时修改配置"
date: 2018-11-06T19:01:42+08:00
publishdate: 2018-11-06
lastmod: 2020-01-19
draft: false
tags: ["php"]
---
```php
<?php

// 修改程序最大执行时间 为 300s
ini_set("max_execution_time", 300);

// 修改程序内存限制为 128M
ini_set("memory_limit", "128M");
```