---
title: "浏览器拦截第三方 Cookies 解决方法"
date: 2016-10-13T12:09:37+08:00
publishdate: 2016-10-13
lastmod: 2020-01-17
draft: false
tags: ["web", "http"]
---
高版本的 Safari 浏览器为了保护用户隐私会默认阻止第三方的 Cookies。Web 开发中，使用第三方资源非常常见，大多数情况下，这并不会带来问题。然而当我们希望能读写第三方域下的 Cookies 时，就会遇到 Cookies 被拦截的情况从而导致各种问题。

## 第三方指的是谁
当用户请求某一域名下的页面时，如果页面中引用了另一个域名下的资源，则另一个域名被算作第三方。

## 解决第三方 Cookies 被拦截的方法
**方法一**:    
客户端手动修改设置。例如 Safari 可以通过偏好设置中的隐私栏修改 Cookies 的安全策略。

**方法二**:    
将需要请求的第三方资源放在本域下。

前两种方法有各自的限制，如果无法解决问题，则需要考虑更复杂的解决方法。

**方法三**:    
Safari 会在第三方域下完全没有 Cookies 时阻止第三方 Cookies，而第三方域下只要有过任意一个 Cookie，即可顺利读写。
据此，我们可以考虑当第三方域下没有 Cookies 时，首先将页面先跳到这个域，写入任意 Cookies，再跳回来。或者弹出一个新窗口，写入 Cookies，再关闭弹窗。

**方法四**:    
Safari 只阻止了第三方 Cookies，并没有阻止第三方 LocalStorage，于是，我们便可以使用更为激进的方案，即放弃第三方 Cookies，使用 LocalStorage 来代替。

## 参考 & 扩展阅读
- [当浏览器默认禁用第三方cookie](http://oldj.net/article/third-party-cookie/)

