---
title: "macOS BigSur 开启显示器 HiDPI 方法"
date: 2020-12-24T13:15:58+08:00
publishdate: 2020-12-24T13:41:58+08:00
lastmod: 2020-12-24T13:15:58+08:00
draft: false
tags: ["mac", "os", "hidpi"]
---
过去为显示器设置 HiDPI 的方法仍然有效，可以查看参考和扩展中给出的链接。只是在最后遇到一个文件系统权限问题。

## 解决无法将文件写入只读文件系统问题
`macOS BigSur` 系统中加入了只读文件系统，无法再使用过去的方式将配置文件直接拷贝到 `/System/Library/Displays/Contents/Resources/Overrides` 目录中。即便开启了 SIP 仍然无法写入。

解决方法是将需要的文件写入 `/Library/Displays/Contents/Resources/Overrides` 目录中。写入该目录中同样有效。

## 使用脚本自动修改
发现一个自动适配 HiDPI 的脚本，但目前还未测试使用。

<https://github.com/mlch911/one-key-hidpi>

## 参考 & 扩展
- [macOS 下开启显示器 HiDPI 方法](https://kouler.com/posts/macos下开启显示器hidpi方法/)
- [macOS 无法获得权限问题](https://kouler.com/posts/macos无法获得权限问题/)
- [One key HiDPI](https://github.com/mlch911/one-key-hidpi)
