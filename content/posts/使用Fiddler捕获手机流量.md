---
title: "使用 Fiddler 捕获手机流量"
date: 2020-03-12T00:41:43+08:00
publishdate: 2020-03-12T00:41:43+08:00
lastmod: 2020-03-12T00:41:43+08:00
draft: false
tags: ["fiddler", "web"]
---
## Fiddler 设置
1. 打开 Fiddler 连接设置，`Tools`->`Fiddler Options`->`Connections`
2. 设置允许远程计算机连接，`Allow remote computers to connect`
3. 可以自定义端口，默认端口为 8888，`Fiddler listens on port`
4. 确认设置，`OK`
5. 重启 Fiddler

## 手机端设置
1. 确保手机和电脑在同一局域网内
2. 在当前无线网络高级设置中手动配置 HTTP 代理
    1. IP 地址为电脑 IP 地址。鼠标移至 Fiddler 主界面右上 `Online` 字样处可查看当前 IP 地址
    2. 端口为 Fiddler 监听端口。默认为 8888
3. 禁用移动数据并关闭 VPN，使得数据通过无线网络代理传输
4. 打开手机浏览器，访问代理地址。例如: `http://192.168.1.111:8888`。正常情况下会打开 Fiddler Echo Service 页面
5. 点击页面中的 `FiddlerRoot certificate` 链接下载证书
6. 安装证书
