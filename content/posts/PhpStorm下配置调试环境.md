---
title: "PhpStorm 下配置调试环境"
date: 2016-04-06T14:42:30+08:00
publishdate: 2016-04-06
lastmod: 2020-01-19
draft: false
tags: ["phpstorm", "php", "xdebug"]
---
使用 brew 安装 php 和 xdebug

PHP xdebug 配置信息

```ini
[xdebug]
; xdebug.remote_autostart=1
zend_extension="/usr/local/Cellar/php71-xdebug/2.5.5/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_handler="dbgp"
xdebug.remote_host=localhost
xdebug.remote_port=9001
xdebug.idekey="PHPSTORM"
xdebug.profiler_enable_trigger=1
xdebug.profiler_output_dir="/Users/kouler/Codes/php/xdebug_profiler"
```

需要注意 xdebug 和 phpfpm 的默认端口都是 9000，端口冲突

## 参考 & 扩展
- [在Mac上的PHPSTORM配置XDebug来调试PHP程序](https://www.jianshu.com/p/3fe69df5d6de)
- [PHP:使用xdebug profiler 做性能分析](https://zhuanlan.zhihu.com/p/26615449)
