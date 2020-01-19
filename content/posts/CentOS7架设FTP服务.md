---
title: "CentOS7 架设 FTP 服务"
date: 2019-08-04T18:59:42+08:00
publishdate: 2019-08-04
lastmod: 2020-01-19
draft: false
tags: ["centos"]
---
## 大概操作流程
之前已经安装了 vsftpd 服务，并修改了配置

目前的状况是浏览器端可以登录，但无法显示文件列表

1. 开机并启动 vsfptd
    ```
    # 启动服务
    service vsftpd start
    
    # 停止服务
    service vsftpd stop
    
    # 重启服务
    service vsftpd restart
    
    # 查看状态
    service vsftpd status
    ```
2. Docker 中安装了 ftp 命令行工具，登录时提示错误 `vsftpd: refusing to run with writable root inside chroot()`
3. 更改挂载目录的拥有者，仍然提示错误
4. 挂载硬盘出错 `mount /dev/sdb is write-protected mounting read-only` `mount: unknown filesystem type '(null)'`
5. 硬盘格式为 ntfs，格式化为linux支持的格式可以解决该问题，但硬盘中有数据，放弃了该方法
6. 安装 `ntfs-3g`，使用该工具挂载硬盘
7. 命令行登录ftp，提示错误 `500 Illegal PORT command`
8. 将 vsftpd 设置为被动模式，增加 vsftpd 配置项
    ```
    # 开启被动模式
    pasv_enable=YES 
    
    # 被动模式最低端口
    pasv_min_port=30000 
    
    # 被动模式最高端口
    pasv_max_port=31000 
    ```
9. 仍然看不到文件列表
10. 下载 ftp 客户端，使用 filezilla 登录 ftp
11. 发现将 ntfs 挂载之后目录权限均为 root，使用 chown 更改所属用户没有效果，后来通过 ntfs-3g 挂载时设置用户和用户组为 vsftpd 用户
12. SELinux 在访问 ftp 时收到警告
13. 设置 SELinux 对 ftp 的控制，允许访问目录
    ```
    # 查看 SELinux 中 ftp 权限
    getsebool -a | grep ftp
    
    # 允许 ftp 访问目录
    setsebool -P tftp_home_dir 1
    setsebool -P allow_ftpd_full_access 1
    ```
14. 解决错误 `vsftpd: refusing to run with writable root inside chroot()`，在 vsftpd 配置文件中添加 `allow_writeable_chroot=YES`
15. ftp 仍然无法显示挂载硬盘的目录列表，原因是设置 ftp 为被动模式，防火墙阻止了之前设置的用于传输数据的被动模式端口 30000-31000
16. 批量开放防火墙端口
    ```
    firewall-cmd --permanent --zone=public --add-port=30000-31000/tcp
    firewall-cmd --reload
    ```
17. 设置开机启动 vsftpd 服务 `chkconfig vsftpd on`
18. 设置开机自动挂载 NTFS 硬盘
    ```
    # 编辑 /etc/fstab 文件
    UUID=831df52d-baad-4902-bcce-6128cec95411 /home/vsftpd/ftp ntfs-3g defaults 0 0
    ```

## 参考
- [vsftpd：500 OOPS: vsftpd: refusing to run with ](http://linux.it.net.cn/e/server/ftp/2015/0227/13554.html)
- [如何更改linux文件的拥有者及用户组(chown和chgrp)](https://blog.csdn.net/hudashi/article/details/7797393)
- [vsftpd 服务器报错：500 OOPS: vsftpd: refusing to run with writable root inside chroot()](https://blog.51cto.com/kisuntech/1308314)
- [vsftpd配置文件详解](https://blog.51cto.com/yuanbin/108262)
- [Centos/Linux 挂载移动ntfs硬盘](https://jingyan.baidu.com/article/fcb5aff76b59ededab4a7177.html)
- [Centos 7 挂载 NTFS 分区](https://blog.huihut.com/2017/03/29/Centos7NTFS/)
- [500 Illegal PORT command的问题（FTP主被动模式）](https://blog.csdn.net/enweitech/article/details/51330664)
- [CentOS下vsftp设置、匿名用户&本地用户设置、PORT、PASV模式设置](https://desert3.iteye.com/blog/1685734)
- [CentOS 6和CentOS 7防火墙的关闭](https://www.linuxidc.com/Linux/2016-12/138979.htm)
- [linux中ftp查看不到文件列表的问题](https://blog.csdn.net/qq_32786873/article/details/78873416)
- [如何更改linux文件的拥有者及用户组(chown和chgrp)](https://blog.csdn.net/hudashi/article/details/7797393)
- [Linux查看用户所属的组](https://blog.csdn.net/yasi_xi/article/details/8493574)
- [Linux 系统的 NTFS-3G 权限](https://havee.me/linux/2015-12/ntfs3g-permissions-on-linux.html)
- [查看 SELinux状态及关闭SELinux](https://blog.51cto.com/bguncle/957315?source=drt)
- [CentOS7 搭建vsftpd详细教程](https://www.linuxidc.com/Linux/2017-12/149909.htm)
- [vsftpd设置后无法登陆，解决OOPS: vsftpd: refusing to run with writable root inside chroot()](https://www.jianshu.com/p/763fdb8cc3f9)
- [vsftpd不能显示文件目录的解决方法](https://blog.csdn.net/imyyx125/article/details/77988841)
- [Centos防火墙设置与端口开放的方法](https://www.jianshu.com/p/3930b998ac2e)
- [CentOS 7 开放防火墙端口命令](https://blog.csdn.net/achang21/article/details/52538049)
- [CentOS7下firewall批量开放端口](https://blog.csdn.net/github_37128837/article/details/73356170)
- [CentOs 设置开机启动vsftpd服务](https://blog.csdn.net/itsmduan/article/details/80196948)
- [CentOS设置设备开机自动挂载](https://blog.csdn.net/u011563666/article/details/79191719)
- [centos7.2挂载NTFS格式移动硬盘](https://blog.csdn.net/Hu_wen/article/details/74805149)
- [通过UUID挂载磁盘](https://blog.csdn.net/zhang1990_2017/article/details/69857043)