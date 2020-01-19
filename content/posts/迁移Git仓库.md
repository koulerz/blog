---
title: "迁移 Git 仓库"
date: 2017-05-26T13:21:15+08:00
publishdate: 2017-05-26
lastmod: 2020-01-19
draft: false
tags: ["git"]
---
```bash
// 切换到需要迁移的 Git 项目目录
$ cd project

// 添加 Git 远程仓库
$ git remote add gitlab git@gitlab.com:team/name.git

// 迁移代码到远程仓库
$ git push -u gitlab master
```

## 参考 & 扩展阅读
- [如何把已存在的git项目转移到Gitlab项目](https://segmentfault.com/q/1010000000385886)
- [Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)