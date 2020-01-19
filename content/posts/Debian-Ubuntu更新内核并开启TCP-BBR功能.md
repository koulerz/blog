---
title: "Debian Ubuntu 更新内核并开启 TCP BBR 功能"
date: 2018-06-16T03:37:24+08:00
publishdate: 2018-06-16
lastmod: 2020-01-19
draft: false
tags: ["os", "debian", "ubuntu", "bbr", "network"]
---
[BBR (Bottleneck Bandwidth and RTT)](https://github.com/google/bbr) 是 Google 提供的 TCP 拥塞控制算法，适用于复杂网络环境下的 TCP 加速。

1. Debian 8.x 或者 Debian 9.x 系统，当然以下教程也适合 Ubuntu 14.04 或 Ubuntu 16.04
2. 如果是虚拟机，那么得使用 KVM 或 Xen 等可以修改内核的平台，OpenVZ 方法我们不做介绍

## 升级内核
BBR 只支持 4.9.x 以上的内核，所以我们需要更新升级以下

如果你使用的是 Debian 9.x，那么这一步可以直接跳过，其他三个内核版本较旧的系统，我们可以使用 Ubuntu 打包好的内核安装包

首先，找到 4.9.x 以上版本的稳定内核，这里我们推荐使用 LTS 版本，目前最新的是 4.9.40 下载安装即可

```bash
mkdir kernel-tmp && cd kernel-tmp
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.9.88/linux-headers-4.9.88-040988_4.9.88-040988.201803181131_all.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.9.88/linux-headers-4.9.88-040988-generic_4.9.88-040988.201803181131_amd64.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.9.88/linux-image-4.9.88-040988-generic_4.9.88-040988.201803181131_amd64.deb
sudo dpkg -i *.deb
```
安装完以后直接 `reboot` 重启，一切顺利的话请检查以下当前的内核版本
```bash
root@debian ~ # uname -r
4.9.0-3-amd64
```

## 配置 BBR
直接修改 `/etc/sysctl.conf` 文件即可

```bash
cat >> /etc/sysctl.conf << EOF
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
EOF
```
然后使用 `sysctl -p` 命令让内核配置生效，不出意外，应该会提示

```bash
root@debian ~ # sysctl -p
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```
此时可以使用 `lsmod | grep bbr` 命令检查 BBR 是否已正确开启

```
root@debian ~ # lsmod | grep bbr
tcp_bbr                16384  61
```
如果出现 tcp_bbr 字样则说明没有问题

## 参考 & 扩展
- [Debian / Ubuntu 更新内核并开启 TCP BBR 拥塞控制算法](https://sb.sb/blog/debian-ubuntu-tcp-bbr/)

