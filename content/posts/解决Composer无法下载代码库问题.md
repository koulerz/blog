---
title: "解决 Composer 无法下载代码库问题"
date: 2019-05-13T19:20:42+08:00
publishdate: 2019-05-13
lastmod: 2020-01-19
draft: false
tags: ["composer"]
---
可以尝试为 composer 设置国内镜像，代码如下
```bash
$ composer config repo.packagist composer https://mirrors.aliyun.com/composer/
```

## 参考&扩展
- [Composer 中文镜像](https://learnku.com/php/wikis/30594)