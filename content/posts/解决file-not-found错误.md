---
title: "解决 file not found 错误"
date: 2021-08-28T02:20:34+08:00
publishdate: 2021-08-28T02:20:34+08:00
lastmod: 2021-08-28T02:20:34+08:00
draft: false
tags: ["nginx", "php"]
---

新创建的 `php` 容器配置好后，访问首页显示 `File not found`。

`nginx` 错误日志中显示 `FastCGI sent in stderr: "Primary script unknown" while reading response header from upstream`

于是在本地搭建了简化版的环境：

1. 创建了 `php` Docker 容器，增加了 `index.php` 测试文件并启动 `php-fpm`
2. 宿主机增加 `nginx` 配置并重新启动

## 尝试解决

根据错误提示，可以了解到错误原因大概率是 `nginx` 没有找到 `php` 文件。

这一错误出现的原因大致分为两类，一是文件权限错误，二是文件路径错误。

权限错误很容易就能够排查，所以怀疑大概率是 `nginx` 配置中的路径错误导致没有找到 `index.php` 文件。

根据 `nginx` 文档修改 `nginx` 配置多次后仍然无法解决。

## 解决问题

能够使用的方法越来越少，不得已只能尝试使用终极办法。

由于 `nginx` 是通过 `fastCGI` 协议与 `php-fpm` 通信，将需要执行的文件路径发送给 `php-fpm`，所以可以通过抓包软件直接抓取 `fastCGI` 的协议内容来调试文件路径错误。

通过监听 `php` 容器的本地 `ip` 和端口号，可以看到 `nginx` 与 `php-fpm` 之间的通信内容。

### nginx 向 php-fpm 发送的数据（FastCGI 协议数据）

```
SCRIPT_FILENAME/var/www/html/docker/index.php
QUERY_STRING
REQUEST_METHODGET
CONTENT_TYPE
CONTENT_LENGTH
SCRIPT_NAME/docker/index.php
REQUEST_URI/docker/index.php
DOCUMENT_URI/docker/index.php

DOCUMENT_ROOT/var/www/html
SERVER_PROTOCOLHTTP/1.1
REQUEST_SCHEMEhttp
GATEWAY_INTERFACECGI/1.1
SERVER_SOFTWAREnginx/1.17.6
REMOTE_ADDR127.0.0.1
REMOTE_PORT59010
SERVER_ADDR127.0.0.1
SERVER_PORT8080
SERVER_NAMElocalhost
REDIRECT_STATUS200
HTTP_HOSTlocalhost:8080
HTTP_CONNECTIONkeep-alive
HTTP_CACHE_CONTROLmax-age=0@
HTTP_SEC_CH_UA"Chromium";v="92", " Not A;Brand";v="99", "Google Chrome";v="92"
HTTP_SEC_CH_UA_MOBILE?0
HTTP_DNT1
HTTP_UPGRADE_INSECURE_REQUESTS1y
HTTP_USER_AGENTMozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.131 Safari/537.36
HTTP_ACCEPTtext/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
HTTP_SEC_FETCH_SITEnone
HTTP_SEC_FETCH_MODEnavigate
HTTP_SEC_FETCH_USER?1
HTTP_SEC_FETCH_DESTdocument
HTTP_ACCEPT_ENCODINGgzip, deflate, br#
HTTP_ACCEPT_LANGUAGEzh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
```

### php-fpm 给 nginx 发送的响应内容

```
Primary script unknownk
Status: 404 Not Found
X-Powered-By: PHP/7.4.22
Content-type: text/html; charset=UTF-8

File not found.
```

`FastCGI` 协议中 `SCRIPT_FILENAME` 的值就是最终要执行的 `php` 文件路径。

将 `index.php` 测试文件放入正确位置后，终于能够正常访问。

## 修正 nginx 配置

在抓包工具的帮助下，更直观和详细的了解了 `nginx` 配置文件中的路径组成。很快将 `nginx` 中的项目路径配置正确。

```conf
# nginx 配置
location / {
    root /src/shop/public;

    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root/index.php$fastcgi_script_name;
    include        fastcgi_params;
}
```

访问 `http://localhost:8080/book/1` 时，`SCRIPT_FILENAME` 的值映射为 `/src/shop/public/index.php/book/`1

## 更优雅的 nginx 配置方式

```conf
# 指定根目录
root /src;

# 还原真实的 URL
location /shop/public {
    try_files $uri $uri/ /shop/public/index.php?$query_string;
    index index.php;
}

# 捕获还原后的 php 脚本地址，交由 php-fpm 执行
location ~ \.php$ {
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
}
```

## 参考 & 扩展

- [应该把 nginx 和 PHP 放在一个 image 里还是分开](https://sexywp.com/should-nginx-and-php-put-together-or-seperate.htm)
- [How to debug Primary script unknown](https://stackoverflow.com/questions/35261922/how-to-debug-fastcgi-sent-in-stderr-primary-script-unknown-while-reading-respo)
- [Does Nginx need to have access to php files](https://serverfault.com/questions/896271/does-nginx-need-to-have-access-to-php-files)
- [Nginx Primary script unknown](https://serverfault.com/questions/517190/nginx-1-fastcgi-sent-in-stderr-primary-script-unknown)
- [How does try_files work with fast_cgi](https://serverfault.com/questions/617245/how-does-try-files-work-with-fast-cgi)
- [FastCGI in Wireshark](https://wiki.wireshark.org/FastCGI)
- [Inspecting FastCGI Packets with Wireshark](https://maxchadwick.xyz/blog/inspecting-fastcgi-packets-with-wireshark)
- [Nginx 伪静态配置](https://www.wiyisun.com/blog/nginx-php-config-fastcgi)
