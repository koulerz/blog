---
title: "Screen 命令配置和常用操作"
date: 2019-07-10T20:48:42+08:00
publishdate: 2019-07-10
lastmod: 2020-01-19
draft: false
tags: ["command", "screen", "terminal"]
---
## 配置文件
该配置在screen底部状态栏显示窗口列表的名称，更易于管理会话和窗口

```
## ~/.screenrc

## 屏幕缓冲区4096行
defscrollback 4096

## 下标签设置
hardstatus on
hardstatus alwayslastline
hardstatus string "%{= kw}%-w%{= kG}%{+b}[%n %t]%{-b}%{= kw}%+w %=%d %M %0c %{g}%H%{-}"
termcapinfo rxvt 'hs:ts=\E]2;:fs=\007:ds=\E]2;screen\007'
termcapinfo xterm ti@:te@
termcapinfo xterm 'hs:ts=\E]2;:fs=\007:ds=\E]2;screen\007'
```

## 常用操作
```bash
screen -S name  # 创建名称为 name 的会话
screen -t name  # 创建名称为 name 的窗口
screen -ls      # 显示存在的会话
screen -r name  # 唤起一个被放入后台的会话
```

```
Ctrl + a, ?        # 显示所有按键绑定信息
Ctrl + a, w        # 显示所有窗口列表
Ctrl + a, Ctrl + a # 切换到之前的窗口
Ctrl + a, d        # 暂离当前会话，将当前会话放入后台
Ctrl + a, c        # 在当前会话中创建一个新窗口
Ctrl + a, A        # 为当前窗口设置名称
Ctrl + a, p        # 上一个窗口
Ctrl + a, n        # 下一个窗口
Ctrl + a, 0-9      # 切换到 0-9 窗口
Ctrl + a, k        # 杀掉当前窗口
Ctrl + a, [        # 拷贝模式
```