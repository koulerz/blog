---
title: "ZSH 环境变量失效问题"
date: 2019-11-06T13:52:42+08:00
publishdate: 2019-11-06
lastmod: 2020-01-19
draft: false
tags: ["terminal"]
---
## 环境变量失效
默认 shell 修改为 zsh 后，`.bash_profile` 中的环境变量全部失效

### 解决方法
1. 用户目录下创建 .zshrc 文件
2. 将以下命令写入文件 `source ~/.bash_profile`

重启终端后可解决


## 终端用户名配置未生效
.bash_profile 中的终端用户名配置（PS1）仍未生效 `export PS1='\[\033[01;36m\]\w \[\033[00m\]\$ '`

### 解决方法
解决 zsh 环境变量失效的问题后，在 `.bash_profile` 文件中修改 `PS1` 的值
```
## 修改前
export PS1='\[\033[01;36m\]\w \[\033[00m\]\$ ' # for bash

## 修改后
export PS1='%~ %(!.#.$) ' # for zsh
```

## 参考
- [环境变量怎么配置都不起作用？已经解决！一切源于 zsh](https://learnku.com/laravel/t/1308/how-does-the-configuration-of-environment-variable-do-not-work-already-solved-everything-from-zsh)
- [zsh prompt expansion](http://zsh.sourceforge.net/Doc/Release/Prompt-Expansion.html)