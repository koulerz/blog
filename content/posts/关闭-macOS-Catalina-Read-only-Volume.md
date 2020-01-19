---
title: "关闭 macOS Catalina Read-only Volume"
date: 2019-10-08T20:20:42+08:00
publishdate: 2019-10-08
lastmod: 2020-01-19
draft: false
tags: ["mac", "os"]
---
运行以下命令即可关闭 Read-only Volume，系统重启后命令会失效
```bash
$ sudo mount -uw /

# killall Finder
```

## 参考
- [macOS Catalina Read-only file system with SIP disabled](https://www.reddit.com/r/MacOS/comments/caiue5/macos_catalina_readonly_file_system_with_sip/et9a0pl/)