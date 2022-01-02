---
title: "Mac 在登录屏隐藏账户"
date: 2018-11-07T09:27:42+08:00
publishdate: 2018-11-07
lastmod: 2020-01-19
draft: false
tags: ["mac"]
---
```bash
# 在登录屏隐藏 PostgreSQL 账户
$ sudo dscl . create /Users/postgres IsHidden 1

# 在登录屏隐藏 hiddenuser 账户
$ sudo dscl . create /Users/hiddenuser IsHidden 1

# 在登录屏隐藏通过 other... 登录
$ sudo defaults write /Library/Preferences/com.apple.loginwindow SHOWOTHERUSERS_MANAGED -bool FALSE

```

## 参考 & 扩展
- [Hide PostgreSQL user in OSX Yosemite](https://followryan.wordpress.com/2015/02/19/hide-postgresql-user-in-osx-yosemite/)
- [Hide a user account in macOS](https://support.apple.com/en-us/HT203998)
- [Remove “Other…” from Login screen](https://apple.stackexchange.com/questions/232449/remove-other-from-login-screen)