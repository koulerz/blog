---
title: "SSH 动态端口转发"
date: 2017-05-26T00:35:46+08:00
publishdate: 2017-05-26
lastmod: 2020-01-19
draft: false
tags: ["ssh"]
---
当我们在一个不安全的 WiFi 环境下上网，用 SSH 动态转发来保护我们的网页浏览及 MSN 信息无疑是十分必要的。

动态转发的命令格式：
```bash
$ ssh -D <local port> <SSH Server>
```
这里 SSH 创建了一个 SOCKS 代理服务。我们可以直接使用 localhost:7001 来作为正常的 SOCKS 代理来使用，直接在浏览器或 MSN 上设置即可。在 SSH Client 端无法访问的网站现在也都可以正常浏览。而这里需要值得注意的是，此时 SSH 所包护的范围只包括从浏览器端（SSH Client 端）到 SSH Server 端的连接，并不包含从 SSH Server 端 到目标网站的连接。如果后半截连接的安全不能得到充分的保证的话，这种方式仍不是合适的解决方案。

## SSH 隧道的搭建
首先需要有一台支持 SSH 的墙外服务器，此服务器只要能 SSH 连接即可。

客户端 SSH 执行如下命令：
```bash
$ ssh -D 7001 username@remote-host
```
上述命令中 `-D` 表示动态绑定，7001 表示本地 SOCKS 代理的侦听端口，可以改成别的，后面的 username@remote-host 就是登录远程服务器的用户名和主机。当然，这个命令后会提示输入密码，就是 username 这个用户的密码（除非配置了SSH公钥认证，可以不输入密码）这样隧道就打通了！

最后在浏览器或者其他应用程序上设置 SOCKS 代理(设置 v4 的 SOCKS 就可以了，v5 的 SOCKS 增加了鉴权功能)，代理指向 127.0.0.1，端口 7001 即可。

## 参考 & 扩展阅读
- [科学上网的一些原理](http://hengyunabc.github.io/something-about-science-surf-the-internet/)
- [实战 SSH 端口转发](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/)
- [SSH隧道翻墙的原理和实现](http://www.pchou.info/linux/2015/11/01/ssh-tunnel.html)
