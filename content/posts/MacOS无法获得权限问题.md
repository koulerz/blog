---
title: "macOS 无法获得权限问题"
date: 2016-11-20T22:40:40+08:00
publishdate: 2016-11-20
lastmod: 2020-01-19
draft: false
tags: ["mac", "os"]
---
通常情况下，在命令行中使用 `sudo` 命令便可以以 Root 用户执行命令

在 MacOS 中，某些系统文件夹下，即使使用 `sudo` 命令也无法获得 Root 权限，这被称为 System Integrity Protection（SIP）

要想关闭 SIP，需要进入 Recover 模式下设置，具体方法如下：
1. 重启电脑，开机时按住 `Command + r` 键进入 Recover 模式
2. 在工具中找到 Terminal 命令行终端
3. 在命令行中输入命令 `csrutil disable`
4. 重新启动后 SIP 就被关闭了

如需重新打开 SIP 保护，操作步骤一样，执行命令为 `csrutil enable`