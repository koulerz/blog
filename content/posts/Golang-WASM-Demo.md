---
title: "Golang WASM Demo"
date: 2018-08-26T16:14:42+08:00
publishdate: 2018-08-26
lastmod: 2020-01-19
draft: false
tags: ["golang", "wasm", "web"]
---
Golang 1.11 加入了对 WebAssembly 的支持，以下会测试并运行一个简单的 Demo。

- MacOS 10.13.6
- Golang 1.11
- Chrome 68 (64-bit)

1. 编写测试代码并编译为 WASM 二进制代码
2. 编写 WebServer 代码用于正确加载本地 WASM 文件
3. 使用 Golang 编写好的 HTML 和 JavaScript 加载 WASM
4. 使用浏览器测试代码

## 编译测试代码
```go
// 测试代码 main.go
package main

import "fmt"

func main() {
    fmt.Println("hello, Go/WASM!")
}
```
```bash
# 编译命令 修改编译好的文件名称为 test.wasm 以便被自带的 HTML 文件正确加载
$ GOOS=js GOARCH=wasm go build -o test.wasm main.go
```

## 编写 WebServer 代码
```go
// server.go
package main

import (
    "flag"
    "log"
    "net/http"
)

func main() {
    port := flag.String("port", "8000", "port")
    flag.Parse()
    log.Fatal(http.ListenAndServe(":"+*port, http.FileServer(http.Dir("."))))
}
```

## 拷贝 Golang 自带的测试文件
```bash
# /usr/local/go 是 Golang 根目录 (GOROOT)
$ cp /usr/local/go/misc/wasm/wasm_exec.{html,js} ./
```

## 在浏览器测试代码
```bash
# 启动 WebServer
$ go run server.go
```

打开测试网页 `localhost:8000/wasm_exec.html`，会看到一个 `run` 按钮

点击按钮，在浏览器控制台将会看到 `hello, Go/WASM!` 字符串，代表代码正确运行

# 参考 & 扩展
- [Quick Tutorial: Write Go, Run WASM!](https://dev.to/cia_rana/quick-tutorial-write-go-run-wasm-2ilf)
- [WebAssembly](https://webassembly.org/)