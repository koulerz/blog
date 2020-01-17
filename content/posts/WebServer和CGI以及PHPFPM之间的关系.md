---
title: "WebServer 和 CGI 以及 PHPFPM 之间的关系"
date: 2016-06-15T23:28:36+08:00
publishdate: 2016-06-15
lastmod: 2020-01-17
draft: false
tags: ["webserver", "web", "php"]
---
## WebServer
最早的 Web服务器简单地响应浏览器发来的 HTTP 请求，并将存储在服务器上的 HTML 文件返回给浏览器，也就是静态HTML。

但事物总是不断发展的，网站也逐渐变得复杂，所以出现动态技术。但是 Web服务器并不能直接运行 php，asp 这样的文件，当浏览器请求 index.php?a=1&b=2 时，Web服务器就不知道该如何处理了，它不能理解 “php”、“?”、“&”、“a”、“b” 各是什么意思，这时它就需要调用相应的脚本解释程序（如PHP-CGI）来处理这样的请求。

## CGI
Web服务器不知道如何处理动态页面请求，需要移交其他程序来处理，这时需要与该程序做个约定，我给你什么，然后你给我什么，就是我把请求参数（如 a=1，b=2）发送给你，然后我接收你的处理结果给客户端。那这个约定就是 Common Gateway Interface，简称 CGI。这个协议可以用 VB，C，PHP，Python 等语言来实现。

CGI 只是接口协议，根本不是什么语言。

## WebServer 与 CGI 程序的交互
Web服务器将根据 CGI程序的类型决定数据向 CGI程序的传送方式，通常是通过标准输入/输出流和环境变量来传递。

CGI程序通过标准输入（STDIN）和标准输出（STDOUT）来进行输入输出。此外 CGI程序还通过环境变量来得到输入。

操作系统提供了许多环境变量，它们定义了程序的执行环境，应用程序可以存取它们。

Web服务器和 CGI接口又另外设置了一些环境变量，用来向 CGI程序传递一些重要的参数。

下面是一些常用的 CGI 环境变量：

|变量名|描述|
|:--|:--|
|CONTENT_TYPE|这个环境变量的值指示所传递来的信息的MIME类型|
|CONTENT_LENGTH |如果服务器与 CGI 程序信息的传递方式是 POST，这个环境变量即是从标准输入 STDIN中可以读到的有效数据的字节数。这个环境变量在读取所输入的数据时必须使用
|HTTP_COOKIE|客户机内的 COOKIE 内容|
|HTTP_USER_AGENT|提供包含了版本数或其他专有数据的客户浏览器信息|
|PATH_INFO|这个环境变量的值表示紧接在 CGI 程序名之后的其他路径信息。它常常作为 CGI 程序的参数出现|
|QUERY_STRING|如果服务器与 CGI 程序信息的传递方式是 GET，这个环境变量的值即使所传递的信息。这个信息经跟在CGI程序名的后面，两者中间用一个问号分隔|
|REMOTE_ADDR|这个环境变量的值是发送请求的客户机的 IP 地址，它是 WEB客户机需要提供给 WEB服务器的唯一标识，可以在 CGI 程序中用它来区分不同的 Web客户机|
|REMOTE_HOST|这个环境变量的值包含发送 CGI 请求的客户机的主机名。如果不支持你想查询，则无需定义此环境变量|
|REQUEST_METHOD|提供脚本被调用的方法。对于使用 HTTP/1.0 协议的脚本，仅 GET 和 POST 有意义|
|SCRIPT_FILENAME|CGI 脚本的完整路径|
|SCRIPT_NAME|CGI 脚本的的名称|
|SERVER_NAME|这是你的 WEB 服务器的主机名、别名或 IP 地址|
|SERVER_SOFTWARE|这个环境变量的值包含了调用 CGI 程序的 HTTP 服务器的名称和版本号。例如 Apache/2.2.14(Unix)|

## PHP-CGI
当 WEB服务器收到 index.php 这样的请求后，会启动对应的 CGI 程序，这里就是 PHP 的 CGI 程序，也就是 PHP-CGI。

接下来 PHP-CGI 会解析 php.ini 文件，初始化执行环境，然后处理请求，再以 CGI 规定的格式返回处理后的结果，退出进程。接着 Web服务器再将结果返回给浏览器。

## FastCGI
CGI 是个协议，跟进程什么的没关系。那 FastCGI 又是什么呢？

FastCGI 是用来提高 CGI程序性能的。PHP-GGI 会对每个请求都执行解析 php.ini 文件，初始化执行环境这些步骤，处理每个请求的时间会比较长，随之便出现了 FastCGI。

Web服务器启动时会载入一个 FastCGI进程管理器（IIS ISAPI 或 Apache Module)，解析配置文件，初始化执行环境，然后再启动多个 CGI解释器进程（PHP-CGI）等待 Web服务器的连接。

当请求到来时，FastCGI进程管理器会选择并连接到一个 PHP-CGI进程，Web服务器将 CGI环境变量和标准输入发送到该 PHP-CGI进程。

PHP-CGI进程完成处理后将标准输出和错误信息从同一连接返回给 Web服务器。当 PHP-CGI进程关闭连接时，请求便处理完成。PHP-CGI进程接着等待并处理来自 FastCGI进程管理器(运行在Web Server中)的下一个连接。 

这样就避免了重复的劳动，效率自然得以提升。而且当 PHP-CGI进程不够用时，FastCGI进程管理器可以根据配置预先启动几个 PHP-CGI进程等着；而当空闲 PHP-CGI进程太多时，也会停掉一些，这样即提高了性能，也节约了资源。这就是 FastCGI 对进程的管理。

## PHP-FPM
PHP-cgi 只是个 CGI 程序，他自己本身只能解析请求，返回结果，不会进程管理，所以就出现了一些能够调度 PHP-cgi 进程的程序，比如说由lighthttpd 分离出来的 spawn-fcgi。PHP-FPM 同样是调度 PHP-cgi 进程的程序，是 FastCGI 的一种实现，负责管理一个进程池。长时间的发展后，现在也逐渐得到了大家的认可，也越来越流行。

## 参考 & 扩展阅读
- [FastCgi 与 PHP-fpm 的关系](https://segmentfault.com/q/1010000000256516)
- [CGI 与 FastCGI](http://www.cnblogs.com/wanghetao/p/3934350.html)
