---
title: "Laravel 字段验证"
date: 2016-12-17T18:06:02+08:00
publishdate: 2016-12-17
lastmod: 2020-01-19
draft: false
tags: ["laravel"]
---
## 手动创建验证程序示例
```php
<?php
 
$data = [
    'email' = 'aaa@gmail.com',
    'name'='aaa',
    'password'='aaa'
];

$rules = array(  
    'email' => 'required|email',  
    'name' => 'required|between:1,20',  
    'password' => 'required|min:8',  
);

$message = array(  
    'required' => ':attribute 不能为空',  
    'between' => ':attribute 长度必须在 :min 和 :max 之间"  
);  

$attributes = array(  
    'email' => '电子邮件',  
    'name' => '用户名',  
    'password' => '用户密码',  
);  

$validate = Validator::make($data,$rules,$message,$attributes);

var_dump($validate->fails());
var_dump($validate->messages());
var_dump($validate->messages()->first())
```

## Validator 的验证扩展
```php
<?php

# 基于 Validator 的手机号验证扩展

Validator::extend('mobile', function($attribute, $value, $parameters)
{
    return preg_match('/^0?(13[0-9]|15[012356789]|18[0-9]|14[57])[0-9]{8}$/', $value);
});
```

## 输出信息统一提示
上面例子中，message 和 attribute 都需要在使用 Validator 的时候自己定义，比较麻烦。Validator 的提示设置是按照语言来进行设置的。语言配置是 config 目录下的 app.php 里面的 locale 配置项，默认为 en。
设置中文提示步骤：
1. 修改 config/app.php 里面的 locale（或 .env 文件内的 APP_LOCALE），改为 cn
2. 创建 api/resources/lang/ch/validation.php 文件
3. 将 validation.php 文件内的提示信息或属性改为中文