---
title: "解决 VSCode 无法安装 Golang 扩展问题"
date: 2019-05-13T19:18:42+08:00
publishdate: 2019-05-13
lastmod: 2020-01-19
draft: false
tags: ["editor", "vscode", "golang"]
---
## SS 方法
1 从 SS 复制 HTTP Proxy Shell Export Line

2 设置 Shell 代理
```bash
export http_proxy=http://127.0.0.1:1087;
export https_proxy=http://127.0.0.1:1087;
```

3 使用 go get 方式安装

## Goproxy 环境变量方法
go 1.11 版本开始新增了 goproxy 环境变量，用于下载源码时设置代理地址

```bash
export GOPROXY=https://goproxy.io
```

**该方法依赖 go module 功能**

## Go module replace 方法
在 *go.mod* 文件中将无法安装的包路径替换为其他能够访问的包路径，比如 github 下的包路径

```go
module hello

require (
    golang.org/x/text v0.3.0
)

# 将限制访问的 golang.org 下的地址替换为能够访问的 github 地址
replace (
    golang.org/x/text => github.com/golang/text v0.3.0
)
```

**由于包依赖通常比较多和杂，所以不推荐使用该方法**

## 参考 & 扩展
- [goproxy.io](https://goproxy.io/)
- [一键解决包失败](https://shockerli.net/post/go-get-golang-org-x-solution/)
- [跳出Go module的泥潭](https://colobu.com/2018/08/27/learn-go-module/)