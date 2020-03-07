---
title: "通过启动参数设置Chrome禁止加载图片"
date: 2020-03-08T05:56:37+08:00
publishdate: 2020-03-08 05:56:37
lastmod: 2020-03-08 05:56:37
draft: false
tags: ["chrome", "web"]
---
试用 chromedp 时，希望启动的浏览器能够实现禁止加载图片的功能，以节省资源消耗。

随后在 chromedp Github 仓库 Issues 中找到了相关提问和答案。启动浏览器时通过设置启动参数 `blink-settings` 的值为 `imagesEnabled=false` 得以实现。

## 参考 & 扩展
- [chromedp](https://github.com/chromedp/chromedp)
- [how to disable images load](https://github.com/chromedp/chromedp/issues/260)
- [Chrome 命令行可配置参数](https://peter.sh/experiments/chromium-command-line-switches/)
- [Chrome 命令行参数 blink-settings 的所有配置项](https://cs.chromium.org/chromium/src/third_party/blink/renderer/core/frame/settings.json5?q=settings.json5&sq=package:chromium&dr&l=498)
