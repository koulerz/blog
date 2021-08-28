---
title: "使用 gitlab runner 自动部署代码"
date: 2021-08-28T18:08:32+08:00
publishdate: 2021-08-28T18:08:32+08:00
lastmod: 2021-08-28T18:08:32+08:00
draft: false
tags: ["gitlab", "cicd"]
---

## 下载 gitlab-runner 错误

下载链接：`sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64`

错误信息

```
root@desktop:~# sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
--2021-06-09 17:31:14--  https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
正在解析主机 gitlab-runner-downloads.s3.amazonaws.com (gitlab-runner-downloads.s3.amazonaws.com)... 52.217.16.228
正在连接 gitlab-runner-downloads.s3.amazonaws.com (gitlab-runner-downloads.s3.amazonaws.com)|52.217.16.228|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度： 46380315 (44M) [application/octet-stream]
正在保存至: “/usr/local/bin/gitlab-runner”

/usr/local/bin/gitlab-run 100%[====================================>]  44.23M  33.4KB/s    剩余 0s    s
段错误
```

最终使用官方链接下载成功 `sudo curl -L --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64"`

## 配置 gitlab-runner

[gitlab runner 安装与使用](https://cloud.tencent.com/developer/article/1620321)

## 解决任务执行失败问题

错误信息：
`This job is stuck, because the project doesn't have any runners online assigned to it. Go to Runners page `

出错原因：
是因为 gitlab-runner 中配置了 tag 标签，但 job 中没有设置 tag 标签

解决方法有两种：

1. 在 job 中指明标签
2. 在 gitlab-runner 中设置允许执行没有标签的 job

[This job is stuck, because the project doesn't have any runners online assigned to it](https://stackoverflow.com/questions/53370840/this-job-is-stuck-because-the-project-doesnt-have-any-runners-online-assigned)

## 对设备不适当的 ioctl 操作

错误信息：`mesg: ttyname 失败: 对设备不适当的 ioctl 操作`

解决方法：

1. 打开 /root/.profile 文件
2. 将 `mesg n || true` 行替换为 `tty -s && mesg n`

- [ttyname failed: Inappropriate ioctl for device` in Vagrant](https://superuser.com/questions/1160025/how-to-solve-ttyname-failed-inappropriate-ioctl-for-device-in-vagrant/1160074#1160074)

## Docker 权限错误

```
$ docker volume create z1-go
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.29/volumes/create: dial unix /var/run/docker.sock: connect: permission denied
```

解决方法：
将用户 `gitlab-runner` 加入 `docker` 用户组
`usermod -a -G docker gitlab-runner`

## 无法正确启动服务

使用 `gitlab-runner start` 无法启动服务，但使用 `gitlab-runner run` 能够启动服务。

可能是因为之前安装过 gitlab-runner 服务导致的冲突

### syslog 错误信息

```
Jun 11 17:34:46 desktop systemd[1]: gitlab-runner.service: Service hold-off time over, scheduling restart.
Jun 11 17:34:46 desktop systemd[1]: gitlab-runner.service: Scheduled restart job, restart counter is at 1035.
Jun 11 17:34:46 desktop systemd[1]: Stopped GitLab Runner.
Jun 11 17:34:46 desktop systemd[1]: Started GitLab Runner.
Jun 11 17:34:46 desktop gitlab-runner[31473]: Runtime platform                                  #033[0;m  arch#033[0;m=amd64 os#033[0;m=linux pid#033[0;m=31473 revision#033[0;m=7a6612da version#033[0;m=13.12.0
Jun 11 17:34:46 desktop gitlab-runner[31473]: Starting multi-runner from /etc/gitlab-runner/config.toml...#033[0;m  builds#033[0;m=0
Jun 11 17:34:46 desktop gitlab-runner[31473]: Running in system-mode.                           #033[0;m
Jun 11 17:34:46 desktop gitlab-runner[31473]:                                                   #033[0;m
Jun 11 17:34:46 desktop gitlab-runner[31473]: #033[31;1mFATAL: Service run failed                         #033[0;m  #033[31;1merror#033[0;m=chdir /home/gitlab-runner: no such file or directory
Jun 11 17:34:46 desktop systemd[1]: gitlab-runner.service: Main process exited, code=exited, status=1/FAILURE
Jun 11 17:34:46 desktop systemd[1]: gitlab-runner.service: Failed with result 'exit-code'.
```

### 解决方法

完全删除后，重新下载并安装解决了该问题

- Ubuntu 查看开机启动服务：`sudo systemctl list-unit-files --type=service|grep enabled`
- Ubuntu 禁用开启服务：`sudo systemctl disable apache2.service`

### 参考

- [gitlab-runner service not starting](https://gitlab.com/gitlab-org/gitlab-runner/-/issues/3344)
- [卸载 gitlab-runner](https://www.cnblogs.com/dhu121/p/13257256.html)

## Q&A

1. 服务器安装 gitlab-runner 并注册成功后，如何关联具体的项目地址？  
   看起来 gitlab-runner 仅仅是作为 gitlab 的扩展硬件资源。即当每次提交代码时，最简单的方式是在 gitlab 的服务器执行 CI/CD 流程，但由于这会消耗大量的资源，所以现在由用户各自配置的机器自主执行。
2. gitlab-runner 是否需要自动拉取代码  
   无需自动拉取代码。gitlab-runner 会自主拉取最新代码
3. 如何在 gitlab 页面删除执行失败的 pipeline  
   暂时没找到清除方法
4. gitlab-runner 注册成功后，部署在哪里，是否无需再拉取代码即可自动部署在本地服务器  
   gitlab-runner 会自主拉取最新代码在部署环境的指定目录。根据 gitlab-ci.yml 文件中的命令决定如何部署服务。
5. gitlab-runner 代码目录在哪里  
   本机具体路径为 `/home/my/builds/njBCKP-_/0/my/api-z1/.git/`，具体路径可能会因为系统环境有所不同
6. 同一项目如何部署在多个服务器
   1. 为统一项目配置多个 gitlab-runner，并设置不同标签
   2. 在 gitlab-ci.yml 配置文件中，根据标签配置不同的命令
7. 项目配置文件如何载入
   1. 在 gitlab 中可以自定义所需的环境变量
   2. 在 gitlab-ci.yml 中设置配置文件路径
   3. 配置文件从 Dockerfile 文件中载入
8. Github 是否有类似的功能  
   Github 提供了 Github Aciton 功能，与 Gitlab 中的 CI/CD 功能类似。同时，也提供了 Runner 程序用于将 Github Action 配置到自建主机。
9. gitlab-runner 配置文件在哪  
   linux 系统中，在 `/etc/gitlab-runner/`
10. 使用调试模式运行 gitlab-runner
    - `gitlab-runner -debug run`
    - 可以查看 [gitlab-runner faq](https://docs.gitlab.com/runner/faq/)
11. 如何设置手动执行 Job  
    在 `gitlab-ci.yml` 文件想要手动执行的 Job 中加入 `when: manual` 配置项
12. 如何使用 gitlab 设置中定义的变量  
    使用美元符号引用定义的变量，例如 `$VARIABLES_NAME`
13. 如何使用 `gitlab-ci.yml` 文件中的环境（environment）参数
14. 如何顺序执行 Job
    1. 可以通过设置多个 stage 间接为 job 设置执行顺序
    2. 可以将全部 job 设置为手动执行，然后手动按顺序执行
15. gitlab-runner 的 start 命令和 run 命令的区别
    - 通过 `start` 启动后，无需再运行 `run` 命令
    - `run` 命令可以用来测试

## 参考 & 扩展

- [gitlab-ci.yml 文件配置项](https://docs.gitlab.com/ee/ci/yaml/README.html)
- [Install GitLab Runner](https://docs.gitlab.com/runner/install/)
- [Install GitLab Runner In Linux](https://docs.gitlab.com/runner/install/linux-manually.html#install-1)
- [GitLab Runner](https://docs.gitlab.com/runner/)
- [Keyword reference for the .gitlab-ci.yml file](https://docs.gitlab.com/ee/ci/yaml/README.html)
- [Predefined variables reference](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)
- [gitlab-runner faq](https://docs.gitlab.com/runner/faq/)
- [shell executors](https://docs.gitlab.com/runner/executors/shell.html)
- [docker executors](https://docs.gitlab.com/runner/executors/docker.html)
- [gitlab-runner source code](https://gitlab.com/gitlab-org/gitlab-runner)
- [gitlab-runner service not starting](https://gitlab.com/gitlab-org/gitlab-runner/-/issues/3344)
- [gitlab runner 安装与使用](https://cloud.tencent.com/developer/article/1620321)
- [ttyname failed: Inappropriate ioctl for device` in Vagrant](https://superuser.com/questions/1160025/how-to-solve-ttyname-failed-inappropriate-ioctl-for-device-in-vagrant/1160074#1160074)
- [This job is stuck, because the project doesn't have any runners online assigned to it](https://stackoverflow.com/questions/53370840/this-job-is-stuck-because-the-project-doesnt-have-any-runners-online-assigned)
- [卸载 gitlab-runner](https://www.cnblogs.com/dhu121/p/13257256.html)
- [Runner uses Docker/Ruby instead of the Shell it's configured for](https://gitlab.com/gitlab-org/gitlab-runner/-/issues/2160)
