---
title: "macOS 桌面壁纸路径"
date: 2019-10-08T22:33:42+08:00
publishdate: 2019-10-08
lastmod: 2020-01-19
draft: false
tags: ["mac"]
---
## 默认路径
1. `/Library/Desktop Pictures`
2. `/System/Library/Desktop Pictures`

## 查看当前桌面壁纸路径
输入以下命令后会在桌面显示当前桌面壁纸所在路径
```bash
$ defaults write com.apple.dock desktop-picture-show-debug-text -bool TRUE
$ killall Dock
```

输入以下命令后不再显示当前壁纸路径
```bash
$ defaults delete com.apple.dock desktop-picture-show-debug-text
$ killall Dock
```

## 参考
- [找不到当前壁纸文件的路径](https://www.macx.cn/thread-2098621-1-1.html)