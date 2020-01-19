---
title: "解决 Cron 不能正常执行 Laravel 计划任务"
date: 2018-08-14T11:13:42+08:00
publishdate: 2018-08-14
lastmod: 2020-01-19
draft: false
tags: ["laravel", "cron"]
---
在 Cron 文件中添加了以下 Laravel 任务调度命令后，发现没有正常执行，而直接在终端执行该命令是正常的
```bash
* * * * * php /path/artisan schedule:run >> /dev/null 2>&1
```

在 Cron 文件中添加以下输出测试命令，也可以正常执行
```bash
* * * * * echo 111 >> /path/test
```

安装 Rsyslog 后查看 Cron 日志，并没有发现任何异常，日志正常输出每一次的调度命令

然后尝试将 Laravel 任务调度命令执行结果输出到文件，修改 Cron 文件后并没有在输出文件中发现任何内容
```bash
* * * * * php /path/artisan schedule:run >> /path/log
```

查看关于 Linux 中重定向相关内容后怀疑可能于此有关

> `>>` 输出重定向到一个文件或设备 追加原来的文件
> 
> `2>>` 将一个标准错误输出重定向到一个文件或设备 追加到原来的文件

随即修改 Cron 文件，将标准错误重定向到文件
```bash
* * * * * php /path/artisan schedule:run 2>> /path/log
```

文件中打印出了错误信息
```bash
/bin/sh: 1: php: not found
```

修改 Cron 文件后，计划任务正确执行

```bash
* * * * * /usr/local/bin/php /path/artisan schedule:run >> /dev/null 2>&1
```

## 重定向符号
> `stdout` 是标准输出流，它显示来自命令的输出。它的文件描述符为 1。
>
> `stderr` 是标准错误流，它显示来自命令的错误输出。它的文件描述符为 2。
> 
> `stdin` 是标准输入流，它为命令提供输入。它的文件描述符为 0。
>
> `/dev/null` 完全忽略标准输出或标准错误，将选择的流重定向到空文件
>
> ##### 标准输出重定向
> `>` 输出重定向到一个文件或设备 覆盖原来的文件
>
> `>!` 输出重定向到一个文件或设备 强制覆盖原来的文件
>
> `>>` 输出重定向到一个文件或设备 追加原来的文件
>
> `<` 输入重定向到一个程序
>
> ##### 标准错误重定向
> `2>` 将一个标准错误输出重定向到一个文件或设备 覆盖原来的文件  b-shell
>
> `2>>` 将一个标准错误输出重定向到一个文件或设备 追加到原来的文件
>
> `2>&1` 将一个标准错误输出重定向到标准输出 注释:1 可能就是代表 标准输出
>
> `>&` 将一个标准错误输出重定向到一个文件或设备 覆盖原来的文件  c-shell
>
> ##### 示例
> `1 > list.txt   2 > list.err` 将标准输出重定向到 list.txt，标准错误重定向到 list.err
>
> `1 > list.txt 2 > &1` 将标准输出和标准错误均重定向到 list.txt
>
> `1 > list.txt 2 > /dev/null` 将标准输出重定向到 list.txt 标准错误重定向到空文件予以丢弃

## 参考 & 扩展
- [Laravel + Crontab not working](https://stackoverflow.com/questions/42483935/laravel-crontab-not-working)
- [Why is my crontab not running](https://superuser.com/questions/445347/why-is-my-crontab-not-running)
- [Linux中重定向及管道命令](https://blog.csdn.net/hellozpc/article/details/46721811)
- [流、管道和重定向](https://www.ibm.com/developerworks/cn/linux/l-lpic1-v3-103-4/index.html)
