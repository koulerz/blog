---
title: "Bash 命令行快捷键"
date: 2016-05-23T22:45:49+08:00
publishdate: 2016-05-23
lastmod: 2020-01-17
draft: false
tags: ["bash", "command"]
---
## 编辑命令
```
Ctrl + a ：移到命令行首    
Ctrl + e ：移到命令行尾    
Ctrl + f ：按字符前移（右向）    
Ctrl + b ：按字符后移（左向）    
Ctrl + xx：在命令行首和光标之间移动    
Ctrl + u ：从光标处删除至命令行首    
Ctrl + k ：从光标处删除至命令行尾    
Ctrl + d ：删除光标后的字符    
Ctrl + h ：删除光标前的字符    
```

## Bang (!) 命令
```
!!：     执行上一条命令    
!blah：  执行最近的以 blah 开头的命令，如 !ls    
!blah:p：仅打印输出，而不执行    
!$：     上一条命令的最后一个参数，与 Alt + . 相同    
!$:p：   打印输出 !$ 的内容    
!*：     上一条命令的所有参数    
!*:p：   打印输出 !* 的内容   

^blah：  删除上一条命令中的 blah    
^blah^foo：  将上一条命令中的 blah 替换为 foo    
^blah^foo^： 将上一条命令中所有的 blah 都替换为 foo    
```

## 参考 & 扩展阅读
- [让你提升命令行效率的 Bash 快捷键](https://linuxtoy.org/archives/bash-shortcuts.html)
