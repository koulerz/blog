---
title: "Golang 交叉编译"
date: 2018-03-05T18:28:42+08:00
publishdate: 2018-03-05
lastmod: 2020-01-19
draft: false
tags: ["golang"]
---
## 什么是跨平台交叉编译
### 交叉编译
通俗地讲就是在一种平台上编译出能运行在体系结构不同的另一种平台上的程序，比如在 PC 平台（X86 CPU）上编译出能运行在以 ARM 为内核的 CPU 平台上的程序，编译得到的程序在 X86 CPU 平台上是不能运行的，必须放到ARM CPU 平台上才能运行，虽然两个平台用的都是 Linux 系统。

交叉编译这种方法在异平台移植和嵌入式开发时非常有用。

### 本地编译
相对与交叉编译，平常做的编译叫本地编译，也就是在当前平台编译，编译得到的程序也是在本地执行。

用来编译跨平台程序的编译器就叫交叉编译器，相对来说，用来做本地编译的工具就叫本地编译器。

所以要生成在目标机上运行的程序，必须要用交叉编译工具链来完成。在裁减和定制 Linux 内核用于嵌入式系统之前，由于一般嵌入式开发系统存储大小有限，通常都要在性能优越的 PC 上建立一个用于目标机的交叉编译工具链，用该交叉编译工具链在 PC 上编译目标机上要运行的程序。

交叉编译工具链是一个由编译器、连接器和解释器组成的综合开发环境，交叉编译工具链主要由 binutils、gcc 和 glibc 3个部分组成。

有时出于减小 libc 库大小的考虑，也可以用别的 c 库来代替 glibc，例如 uClibc、dietlibc 和 newlib。

## Golang 的跨平台交叉编译
Go 语言是编译型语言，可以将程序编译后在将其拿到其它操作系统中运行，此过程只需要在编译时增加对其它系统的支持。

### 交叉编译依赖下面几个环境变量
- $GOARCH 目标平台（编译后的目标平台）的处理器架构（386、amd64、arm）
- $GOOS 目标平台（编译后的目标平台）的操作系统（darwin、freebsd、linux、windows）

### 各平台的 GOOS 和 GOARCH 参考

OS|ARCH|OS version
:--:|:--:|:--:
linux|386 / amd64 / arm|>= Linux 2.6
darwin|386 / amd64|OS X (Snow Leopard + Lion)
freebsd|386 / amd64|>= FreeBSD 7
windows|386 / amd64|>= Windows 2000

### 执行编译
```bash
# 如果你想在Windows 32位系统下运行
$ GOOS=windows GOARCH=386 go build test.go

# 如果你想在Windows 64位系统下运行
$ GOOS=windows GOARCH=amd64 go build test.go

# 如果你想在Linux 32位系统下运行
$ GOOS=linux GOARCH=386 go build test.go

# 如果你想在Linux 64位系统下运行
$ GOOS=linux GOARCH=amd64 go build test.go

# 编译为 WEB 端 WASM 二进制代码
$ GOOS=js GOARCH=wasm go build test.go
```
命令中可以设置 CGO_ENABLED = 0 表示设置CGO工具不可用；

GOOS 表示程序构建环境的目标操作系统(Linux、Windows)；

GOARCH 表示程序构建环境的目标计算架构(32位、64位)；

现在你可以在相关目标操作系统上运行编译后的程序了。

**注意：该方式暂时不支持 CGO**

## 参考 & 扩展
- [关于于 Go 的跨平台交叉编译浅析](http://aurthurxlc.github.io/Aurthur-2015/Cross-platform-cross-compiler-with-go.html)
