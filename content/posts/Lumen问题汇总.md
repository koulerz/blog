---
title: "Lumen 问题汇总"
date: 2016-10-19T11:42:41+08:00
publishdate: 2016-10-19
lastmod: 2020-01-19
draft: false
tags: ["web", "lumen"]
---
## lumen 安装在子目录下时访问首页出现404错误
public/index.php 中的
```php
<?php

$app->run();
```
改为   
```php
<?php

$app->run($app['request']);
```

## lumen 配置数据库
1. 在根目录添加 .env 配置文件并修改数据库信息
2. app.php 文件中去掉 `Dotenv::load(__DIR__ . '/../');`行的注释，只有去掉才可以使用env配置文件
3. 控制器文件中通过 app 函数来调用数据库，例：    
`$result = app('db')->select("select * from xay_waiter");`

[详细信息](http://laravelacademy.org/post/455.html)

## 使用 Session
1. 开启Facades，开启方式是去掉 bootstrap/app.php 中 `$app->withFacades();` 的注释。
2. 开启 Session，开启方式：去掉 bootstrap/app.php 中 `$app->middleware();` 的 `StartSession` 中间件的注释。
3. 使用时发生错误：*Class 'Memcached' not found* ，因为在 .env 文件中，Session 的默认驱动是：memcached。修改即可。    
目前支持的驱动有：file、cookie、database、memcached、redis、array。
4. 
    - `Session::put('key', 'value'); // 保存对象到 Session 中`    
    - `$value = Session::get('key'); // 从 Session 取回对象`     
    - `$value = Session::pull('key', 'default'); // 从 Session 取回对象并删除`    
    - `$data = Session::all(); // 从 Session 取出所有对象`    
    - `Session::has('users'); // 判断对象在 Session 中是否存在` 
    - `Session::forget('key'); // 从 Session 中移除对象`    
    - `Session::flush(); // 清空所有 Session`

## lumen 发送邮件
1. 在 composer.json 中，添加 mail 依赖，添加完成后在命令行使用 `composer update` 命令更新 composer.lock 文件安装依赖

```json
"require": {
    "illuminate/mail": "5.1.*"
},
```
2. 修改 env 配置文件中的邮件配置

```env
MAIL_DRIVER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=25    
MAIL_USERNAME=name@gmail.com
MAIL_PASSWORD=password
MAIL_FROM_ADDRESS=name@gmail.com
MAIL_FROM_NAME=name
```

3. 在控制器中使用 `Mail::send()` 方法发送邮件

```php
<?php

$email = 'name@gmail.com';
$name  = 'name';
$token = 'token';
$uid   = 1;
$data  = ['email' => $email, 'name' => $name, 'token' => $token, 'uid' => $uid];

// 发送简短字符而非完整视图
Mail::raw('Text to e-mail', function ($message) {
    $message->from('from@gmail.com', 'Laravel');
    $message->to('to@gmail.com')->cc('cc@gmail.com');
});

// 发送完整邮件
Mail::send('activemail', $data, function ($message) use ($data) {
    $message->from('from@gmail.com', 'from');
    $message->to($data['email'], $data['name']);
    $message->subject('欢迎注册，请激活您的账号！');
});
```
4. 在 /resources/views/ 目录下创建邮件视图文件 activemail.blade.php

```html
<!Doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
  </head>
<body>
  <a href="http://www.google.com" target="_blank">{{$email.'+'.$name.'+'.$token.'+'.$uid}}点击激活你的账号</a>
</body>
</html>

```
