---
title: "PHP 中的异常捕获"
date: 2016-05-12T19:17:55+08:00
publishdate: 2016-05-12
lastmod: 2020-01-17
draft: false
tags: ["php"]
---
PHP 中 `try` `catch` 可以帮助我们捕获程序代码的异常以便使我们很好的处理一些不必要的错误。

## Try 语句
`try` 是用来定义检测异常的代码块。    
需要进行异常处理的代码都必须放入 `try` 代码块内，以便捕获可能存在的异常。    
每一个 `try` 至少要有一个与之对应的 `catch` 。

```php
<?php

try {
    // 需要进行异常处理的代码
}
```

`try` 代码块有可能运行到最后一行，也有可能抛出异常，如果抛出了异常，代码的剩余部分就会被跳过，程序会跳到 `Catch` 语句块中执行。

## Catch 语句
定义处理发生异常时执行的代码块。使用多个 `catch` 可以捕获不同的类所产生的异常。

当 `try` 代码块不再抛出异常或者找不到 `catch` 能匹配所抛出的异常时，PHP 代码就会在跳转到最后一个 `catch` 的后面继续执行。    
PHP允许在 `catch` 代码块内再次抛出（throw）异常。

当一个异常被抛出时，其后（指抛出异常时所在的代码块）的代码将不会继续执行，而 PHP 就会尝试查找第一个能与之匹配的 `catch`。

如果一个异常没有被捕获，而且又没用使用 `set_exception_handler()` 作相应的处理的话，那么 PHP 将会产生一个严重的错误，并且输出 `Uncaught Exception ...` （未捕获异常）的提示信息。

```php
<?php

catch(Exception $e) {
    echo $e;
}
```

上面代码中，`$e` 是 `Exception` 类的一个实例。

`Exception` 是所有类型异常的祖先，所以捕捉 `Exception` 类会捕捉到任何类型的异常。    

为了处理不同类型的异常，也许会定义多个 `catch` 块，应该先定义最特定的类型，这是因为 `catch` 块是按照顺序来解析的。

## Throw 语句
`throw` 语句是用来触发并抛出异常的，并且会中断发生在这一点上的执行过程。必须给 `throw` 语句传递一个 `Exception` 类的实例。

```php
<?php

$a = new Exception('Err Msg');
throw $a;

// 等价于

throw new Exception('Err Msg');
```

## Finally 语句
无论是否检测到异常，程序都会执行 Finally 语句，可在代码块中插入出错后继续执行的代码,如关闭数据库连接，关闭已打开的文件或返回给客户端错误信息等。

```php
<?php

try {  
     // 需要进行异常处理的代码 
} catch (Exception $e) {  
     // 捕捉异常，记录日志或其他的操作  
     print $e->getMessage();    
} finally {  
     // 插入出错后继续执行的代码,如关闭数据库连接，返回给客户端错误信息等。  
     // ...  
}  
```

## Exception 类
`Exception` 类是所有异常类的基类，其构造函数可以接受一条错误信息或者错误代码作为参数。

构造好异常的实例，就会获得几个关键信息。    
如：构造异常的代码所处位置，在构造时执行的代码，错误信息和错误代码。

```php
<?php

getMessage()  // 返回异常信息    
getCode()     // 返回错误代码    
getFile()     // 返回发生错误所在的源文件    
getLine()     // 返回发生错误的行号，与getFile()一起使用    
getTrace()    // 返回包含场景回溯的一个数组    
```

数组中的每个键值都包含了文件、行号、函数名称以及调用函数的所有参数。    

`getTrace()` 与内置函数 `debug_backtrace()` 类似。    

`getTraceAscString()` 与 `getTrace()` 相同，不过返回的不是数组而是字符串。    

`__toString()` 返回用字符串表达的整个异常信息。

```php
<?php

function connectToDatabase()
{
    if(!$conn = pg_connect(..)) {
        throw new Exception('Could not connect to database!');
    }
}
    
try {
    connectDatabase();
} catch(Exception $e) {
    $e->getMessage();
}
```

## 扩展异常
`mysql_error()` 是提示mysql错误的函数,下面的这段代码就记录了`mysql_error()` 函数输出的错误信息。

```php
<?php

class databaseException extends Exception{
    public $databaseErrorMessage;
    
    public function __construct(
        $errorMessage=null,
        $code=0
    ) {
        $this->databaseErrorMessage = mysql_error();
        parent::__construct($errorMessage, $code);
    }
    
    protected function getDatabaseErrorMessage()
    {
        return $this->databaseErrorMessage;
    }
}
```

## PHP 中 Try Catch 捕获异常实例详解
PHP内置异常类的基本属性和方法

```php
<?php

try{
    // 需要进行异常处理的代码
} catch() {
    throw new Exception();
} catch() {
    // 这里可以捕获到前面一个块抛出的Exception
}
```

为了进一步处理异常，我们需要使用 PHP 中 `try{}` `catch{}`----包括 Try 语句和至少一个的 Catch 语句。

任何调用可能抛出异常的方法的代码都应该使用 Try 语句。 Catch 语句用来处理可能抛出的异常。

以下显示了我们处理 `getCommandObject()` 抛出的异常的方法。

```php
<?php

try {
    $mgr = new CommandManager();
    $cmd = $mgr->getCommandObject("realcommand");
    $cmd->execute();
} catch (Exception $e) {
    print $e->getMessage();
    exit();
}
```

可以看到，通过结合使用 `throw` 关键字和 PHP 中 `try{}` `catch{}` ，我们可以避免错误标记污染类方法中返回的值。

因为异常本身就是一种与其它任何对象不同的 PHP 内建的类型，不会产生混淆。

如果抛出了一个异常， Try 语句中的脚本将会停止执行，然后马上转向执行 Catch 语句中的脚本。

例子如下：包含文件错误抛出异常

```php
<?php

// 错误的演示
try {
    require('test_try_catch.php');
} catch (Exception $e) {
    echo $e->getMessage();
}

// 正确的抛出异常
try {
    if (file_exists('test_try_catch.php')) {
        require('test_try_catch.php');
    } else {
        throw new Exception('file is not exists');
    }
} catch (Exception $e) {
    echo $e->getMessage();
}
```

如果异常抛出了却没有被捕捉到，就会产生一个 `fatal error` 。

多个 Catch 语句捕获多个异常

PHP将查询一个匹配的 Catch 代码块。如果有多个 Catch代码块，传递给每一个 Catch 代码块的对象必须具有不同类型，这样 PHP 可以找到需要进入哪一个 Catch 代码块。

当 Try 代码块不再抛出异常或者找不到 Catch 能匹配所抛出的异常时， PHP 代码就会在跳转最后一个 Catch 的后面继续执行。

多个异常的捕获的示例如下

```php
<?php
class MyException extends Exception {
   // 重定义构造器使第一个参数 message 变为必须被指定的属性
   public function __construct($message, $code=0)
   {
       // 可以在这里定义一些自己的代码
       // 建议同时调用 parent::construct() 来检查所有的变量是否已被赋值
       parent::__construct($message, $code);
   }
   
   // 重写父类中继承过来的方法，自定义字符串输出的样式
   public function __toString()
   {
       return __CLASS__.":[".$this->code."]:".$this->message."<br>";
   }
   
   // 为这个异常自定义一个处理方法
   public function customFunction()
   {
       echo "按自定义的方法处理出现的这个类型的异常";
   }
}

// 创建一个用于测试自定义扩展的异常类 MyException
class TestException {
    // 用来判断对象是否创建成功的成员属性
    public $var;
    
    function __construct($value=0)
    {
        // 通过构造方法的传值决定抛出的异常
        // 对传入的值进行选择性的判断
        switch($value){
            // 掺入参数1，则抛出自定义的异常对象    
            case 1:
                throw new MyException("传入的值“1”是一个无效的参数",5);
                break;
                
            // 传入参数2，则抛出PHP内置的异常对象
            case 2:                              
                throw new MyException("传入的值“2”不允许作为一个参数",6);
                break;
            
            //传入参数合法，则不抛出异常
            default:                     
                //为对象中的成员属性赋值        
                $this->var=$value;break;          
        }
    }
}

// 示例1，在没有异常时，程序正常执行，try中的代码全部执行并不会执行任何catch区块
try {
    $testObj =new TestException();      //使用默认参数创建异常的擦拭类对象
    echo "********<br>";                //没有抛出异常这条语句就会正常执行
} catch(MyException $e) {               //捕获用户自定义的异常区块
    echo "捕获自定义的异常：$e<br>";       //按自定义的方式输出异常消息
    $e->customFunction();               //可以调用自定义的异常处理方法
} catch(Exception $e) {                 //捕获PHP内置的异常处理类的对象
    echo "捕获默认的异常：".$e->getMessage()."<br>";       //输出异常消息
}

// 判断对象是否创建成功，如果没有任何异常，则创建成功
var_dump($testObj);        
 
// 示例2，抛出自定义的异常，并通过自定义的异常处理类捕获这个异常并处理
try {
    $testObj1 =new TestException(1);   //传1时，抛出自定义异常
    echo "********<br>";               //这个语句不会被执行
} catch(MyException $e) {              //这个catch区块中的代码将被执行
    echo "捕获自定义的异常：$e<br>";          
    $e->customFunction();                    
} catch(Exception $e) {                //这个catch区块不会执行
    echo "捕获默认的异常：".$e->getMessage()."<br>";       
}

// 有异常产生，这个对象没有创建成功
var_dump($testObj1);        
 
// 示例3，抛出自内置的异常，并通过自定义的异常处理类捕获这个异常并处理
try {
    $testObj2 =new TestException(2);   //传入2时，抛出内置异常
    echo "********<br>";               //这个语句不会被执行
} catch(MyException $e) {              //这个catch区块中的代码将被执行
    echo "捕获自定义的异常：$e<br>";          
    $e->customFunction();                    
} catch(Exception $e) {                //这个catch区块不会执行
    echo "捕获默认的异常：".$e->getMessage()."<br>";       
}

// 有异常产生，这个对象没有创建成功
var_dump($testObj2);        
```

在上面的代码中，可以使用两个异常处理类：

一个是自定义的异常处理类 `MyException` ；另一个则是 PHP 中内置的异常处理类 `Exception` 。

分别在 Try 区块中创建测试类 `TestException` 的对象，并根据构造方法中提供的不同数字参数，抛出自定义异常类对象、内置的异常类对象和不抛出任何异常的情况，跳转到对应的 Catch 区块中执行。如果没有异常发生，则不会进入任何一个 Catch 块中执行，测试类 `TestException` 的对象创建成功。

## 参考 & 扩展阅读
- [php中try catch捕获异常实例详解](http://www.jb51.net/article/57688.htm)
- [调试PHP OOP应用中的异常：try, catch和throw语句和Exception类组成的处理机制](http://www.yunxiu.org/blog/article/5416.htm)
- [更好的PHP错误处理：常规错误处理介绍](http://feiyang.me/2014/05/handle-php-error-better-introduce-errors/)
- [PHP 异常处理](http://www.w3school.com.cn/php/php_exception.asp)