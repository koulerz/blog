---
title: "安装 Hexo"
date: 2016-04-01T10:11:57+08:00
publishdate: 2016-04-01
lastmod: 2020-01-17
draft: false
tags: ["hexo", "tool"]
---
1. 安装 [Node.js](https://nodejs.org)
2. 安装 [Hexo](https://hexo.io/zh-cn/) 
```bash
$ sudo npm install -g hexo-cli
```

3. 初始化 Hexo
```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

4. 生成静态页面 
```bash
$ hexo generate
```

5. 在 Github 新建项目
6. 填写网站及Git配置信息 _config.yml
```yml
deploy:
    type: git
    repo: https://github.com/koulerz/kouler.git
```

7. 安装 hexo-deployer-git 
```bash
$ npm install hexo-deployer-git --save
```

8. 部署项目到 Github 
```bash
$ hexo deploy
```