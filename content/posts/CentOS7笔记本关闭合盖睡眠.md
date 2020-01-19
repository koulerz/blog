---
title: "CentOS7 笔记本关闭合盖睡眠"
date: 2019-07-11T12:14:42+08:00
publishdate: 2019-07-11
lastmod: 2020-01-19
draft: false
tags: ["os", "centos"]
---
## 具体流程
1. 修改配置文件`/etc/systemd/logind.conf`，去掉`HandleLidSwitch`行的注释，并将其值由`suspend`改为`ignore`
2. 执行`systemctl restart systemd-logind`命令，使刚才的修改生效

修改后的文件内容如下：
```
[Login]
#NAutoVTs=6
#ReserveVT=6
#KillUserProcesses=no
#KillOnlyUsers=
#KillExcludeUsers=root
#InhibitDelayMaxSec=5
#HandlePowerKey=poweroff
#HandleSuspendKey=suspend
#HandleHibernateKey=hibernate
HandleLidSwitch=ignore
#HandleLidSwitchDocked=ignore
#PowerKeyIgnoreInhibited=no
#SuspendKeyIgnoreInhibited=no
#HibernateKeyIgnoreInhibited=no
#LidSwitchIgnoreInhibited=yes
#IdleAction=ignore
#IdleActionSec=30min
#RuntimeDirectorySize=10%
#RemoveIPC=no
#UserTasksMax=
```

如需再次进入待机状态，可执行`systemctl suspend`命令。

## 通过修改配置文件自定义电源管理
```
# /etc/systemd

动作:
HandlePowerKey       # 按下电源键后的动作
HandleSleepKey       # 按下挂起键后的动作
HandleHibernateKey   # 按下休眠键后的动作
HandleLidSwitch      # 合上笔记本盖后待机

动作允许的值:
ignore      # 不执行任何操作
poweroff    # 关机
reboot      # 重新启动
halt        # 关机
suspend     # 待机挂起
hibernate   # 休眠
```

## 参考
- [CentOS 7.0 笔记本关闭合盖睡眠](https://blog.csdn.net/qq_34911465/article/details/70246193)
