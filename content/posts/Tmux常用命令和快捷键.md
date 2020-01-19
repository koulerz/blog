---
title: "Tmux 常用命令和快捷键"
date: 2019-08-07T17:43:42+08:00
publishdate: 2019-08-07
lastmod: 2020-01-19
draft: false
tags: ["command", "tmux", "terminal"]
---
## Tmux 命令

```
# 开启一个 Tmux Session
$ tmux new -s name 

# 断开当前会话，会话在后台运行，等同于 `Ctrl + B` + `D`
$ tmux detach

# 查看当前 Tmux 中有哪些 Session
$ tmux ls 

# 回到 Tmux Session 中
$ tmux a -t name（or at，or attach） 

# 关闭demo会话
tmux kill-session -t demo 

# 关闭服务器，所有的会话都将关闭
tmux kill-server 
```

## Tmux 快捷键
Tmux Session 中的命令需要先按下 Tmux 前缀键（默认是 `Ctrl + B`）
```
# 会话（Session） 指令

?        # 快捷键帮助列表
d        # detach，退出 Tmux Session，回到父级 Shell
r        # 强制重载当前会话
:        # 进入命令行模式，此时可直接输入ls等命令
s        # 列出所有 Session，可通过 j, k, 方向键选择，回车切换
$        # 为当前 Session 命名
```

```
# 窗口（window）指令

c        # 新建 Window
&        # 关闭当前窗口
p        # 切换到上一窗口
n        # 切换到下一窗口
0~9      # 切换到指定 Window
w        # 打开窗口列表，用于且切换窗口
,        # 为当前 Window 命名
.        # 修改当前窗口编号（适用于窗口重新排序）
f        # 快速定位到窗口（输入关键字匹配窗口名称）
```

```
# 面板（pane）指令

%        # 垂直切分 Pane
"        # 水平切分 Pane
x        # 关闭当前面板
空格键	 # 切换 Pane 布局
方向键   # 移动光标切换面板
t        # 显示时钟
```

## 参考
- [Tmux使用手册](http://louiszhai.github.io/2017/09/30/tmux/)