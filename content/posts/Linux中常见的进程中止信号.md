---
title: "Linux 中常见的进程中止信号"
date: 2021-08-28T16:05:38+08:00
publishdate: 2021-08-28T16:05:38+08:00
lastmod: 2021-08-28T16:05:38+08:00
draft: false
tags: ["linux", "signal"]
---

## SIGINT

- `Ctrl + C` 会发送 SIGINT 信号
- 信号会被当前进程树接收到，不仅当前进程会收到信号，它的子进程也会收到。
- 只能结束前台进程

## SIGTERM

- `kill -15` 会发送 SIGTERM 信号
- 没有参数的 kill 命令也会默认发送 SIGTERM 信号
- 只有当前进程会收到信号，子进程不会收到。如果当前进程被 kill 掉，那么它的子进程将会被 init 进程接管，也就是 pid 为 1 的进程
- 用来要求程序自己正常退出，让它自己清理文件和关闭。系统关机的时候会触发这个信号
- 这个信号不保证进程一定会被终止

## SIGKILL

- `kill -9` 会发送 SIGKILL 信号
- 用来强制使进程立即结束
- SIGKILL 不能被捕获，程序收到这个信号后，一定会退出
