---
title: "Lumen 加载根目录配置文件"
date: 2018-10-11T22:57:42+08:00
publishdate: 2018-10-11
lastmod: 2020-01-19
draft: false
tags: ["lumen", "web"]
---
Lumen 版本: 5.3

1. 在根目录创建 config 目录
2. 在 config 目录中新建配置文件
3. 在 `bootstrap/app.php` 中加载配置文件

```php
<?php

// bootstrap/app.php
$app->configure('fileName');

// Example: $app->configure('mail');
```

## 参考&扩展
- [Lumen 加载根目录配置文件](https://my.oschina.net/u/2292141/blog/894710)