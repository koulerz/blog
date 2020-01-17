---
title: "使用 Hugo"
date: 2020-01-17T22:42:14+08:00
publishdate: 2020-01-17
lastmod: 2020-01-17
draft: false
tags: ["hugo", "command", "tool"]
---
## Hugo 安装和配置
- [Hugo Documentation](https://gohugo.io/documentation/)
- [minimal-bootstrap-hugo-theme](https://github.com/zwbetz-gh/minimal-bootstrap-hugo-theme)

## Hugo 常用命令
```shell
$ hugo help                  // 帮助命令
$ hugo version               // 打印版本号
$ hugo new site sitename     // 创建新站点
$ hugo new posts/new-post.md // 创建新文章
$ hugo                       // 生成静态页面
$ hugo server                // 启动 Hugo Web Server
```

## 在 Github 托管 Hugo
- [Host on GitHub](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

## Github Pages site 配置个人域名
1. 在发布项目根目录添加 CNAME 文件，内容为个人域名
2. 配置个人域名 DNS，添加 CNAME 记录，将 www 子域指向个人 Github Pages site 地址，*yourname.github.io*
3. 配置个人域名 DNS，添加 A 记录，将域名指向 Github Pages 提供的 IP 地址
4. 在 Github Pages site 仓库设置中添加配置好的域名，开启 `Enforce HTTPS`

- [Host on GitHub](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
- [CNAME errors](https://help.github.com/en/github/working-with-github-pages/troubleshooting-custom-domains-and-github-pages#cname-errors)
- [Managing a custom domain for your GitHub Pages site](https://help.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site)
