---
title: "Laravel 中的服务容器设计模式"
date: 2022-01-23T15:42:42+08:00
publishdate: 2022-01-23
lastmod: 2022-01-23
draft: false
tags: ["laravel", "设计模式"]
---
## 名词解释：
|术语|说明|
|:--|:--|
|IoC (Inversion of Control)|控制反转|
|DI (Dependence Injection)|依赖注入|
|Service Container|服务容器/IOC 容器|
|Service Provider|服务提供者|

## 目录
1. 什么是依赖
2. 什么是容器
3. 什么是服务提供者
4. 什么是Facades
5. 执行流程
6. 概述执行流程
7. 参考

## 什么是依赖
大多数面向对象编程语言，在调用一个类的时候，先要实例化这个类，生成一个对象。 如果在写一个类时，过程中要调用到很多其它类，甚至这里的其它类，也要“依赖”于更多其它的类，那么可以想象，就需要进行多少次实例化。这就是“依赖”的意思。

## 什么是容器
容器（IoC容器）是一个设计模式，是 Laravel 的核心机制，容器本身就是通过 Laravel 中的一个核心类实现的，这个类，叫做 Application , 程序启动的时候就实例化了这个类， 放在 $app 变量中, 这就是 IoC 容器。
如果把某个类（不管有多少依赖关系）放入这个容器中，可以“解析”出这个类的实例。
Laravel 框架中 vendor 文件夹下的内核文件和各种扩展服务大都已注册在容器中方便使用。

## 什么是服务提供者
IoC容器中存放着注册过的类，但调用某个类时容器可以直接解析出这个类的对象。那么如何将类注册到IoC容器呢？这个时候就需要服务提供者了。框架中的扩展服务和开发者写的第三方扩展都是通过服务提供者注册到 IoC 容器，服务提供者提供了连接扩展与内核的作用，也正是因为有服务提供者，所以注册代码才不至于写乱。

接下来扩展一个自己的类将它注册到 IoC 容器并调用它的实例：

需要的文件：
- `/app/Tools/Piece.php` 扩展类
- `/app/Providers/ToolServiceProvider.php` 通过服务提供者将类注册到容器
- `/config/app.php` 配置文件
- `/app/Controllers/TestController.php` 控制器，用来调用扩展类

1. 首先创建扩展类
    ```php
    <?php
    // FilePath: /app/Tools/Piece.php

    namespace App\Tools;

    class Piece
    {
        public function work()
        {
            echo "i am working";
        }
    }
    ```

2. 接下来将这个扩展类通过 Service Provider 注册到容器
    ```php
    <?php
    // FilePath: /app/Providers/ToolServiceProvider.php

    namespace App\Providers;

    use App\Tools\Piece;
    use Illuminate\Support\ServiceProvider;

    class ToolServiceProvider extends ServiceProvider
    {
        public function register()
        {
            $this->app->singleton('ptool', function(){
                return new Piece;
            });
        }
    }
    ```

    - 这就是服务提供者，这个服务提供者完成将扩展类注册到IoC容器中的工作。
    - 这个服务提供者(ServiceProvider)文件中，register() 这个方法只能用来注册服务，不能写其它不相关的东西。
    - 在匿名函数里 new 了一个类实例并返回，这也就是为什么通过 IoC 容器可以直接得到类实例的原因。
    - 这里的 ptool 可以理解成一个别名，名称可以随意取，以后的使用中将通过这个别名从容器中获取到这个类实例。
    - 注意这里的 singleton 方法，Laravel 提供了多种方式将类注册到容器，singleton 是其中一种，意为单例模式，意思是这个类不论使用多少次，只能生成一个对象（即匿名函数中的代码只执行一次）。可以查看官方文档了解其它方式。

3. 现在扩展类仍无法在容器中生效，还需要将写好的服务提供者(Service Provider)加载到启动序列。
    ```php
    <?php
    // FilePath: /config/app.php

    'providers' => [
        // Laravel Framework Service Providers ...
        // ...

        // Application Service Providers ...
        // ...
        App\Providers\ToolServiceProvider::class,
    ],
    ```

    App 配置文件中，这个 providers 数组中存放着所有服务提供者。

4. 到目前为止，扩展类已经注册到 IoC容器中了，接下来看如何使用它
    ```php
    <?php
    // FilePath: /app/Http/Controllers/TestController.php

    namespace App\Http\Controllers;

    use App;
    use App\Http\Controllers\Controller;

    class TestController extends Controller
    {
        public function index()
        {
            App::make('ptool')->work();
        }
    }
    ```
    使用 App 中的 make() 方法，通过传入服务提供者中定义的别名就可以得到扩展类的实例，然后访问扩展类中的 work() 方法。

5. 访问 `localhost:8888/laravel/public` 地址，页面正确输出 `i am working`

## 什么是Facades
在 TestController.php 文件中，使用了 App::make('ptool')->work() 调用 work() 方法，其实还有更简单和优雅的写法。类似于这句代码中调用 App 类下的 make() 方法，App::make()，
那么从 App::make('ptool')->work() 简化到 Piece::work() 就是 Facades 所要做的事情。

接下来将实现它。需要的文件：
- `/app/Facades/Piece.php` Piece类的 Facade 文件
- `/config/app.php` 配置文件
- `/app/Controllers/TestController.php` 控制器，用来调用扩展类

1. 首先创建 Piece 类的 Facade 文件
    ```php
    <?php
    // FilePath: /app/Facades/Piece.php

    namespace App\Facades;

    use Illuminate\Support\Facades\Facade;

    class Piece extends Facade
    {
        protected static function getFacadeAccessor()
        {
            return 'ptool';
        }
    }
    ```
    通过这个文件 Facades 会自动完成 App::make('ptool') 这步操作并返回类实例。

2. 类似于服务提供者(Service Provider)使用之前要注册，要想使用 Facades，也需要在配置文件中的 aliases 数组中先设置一个别名。
    ```php
    <?php
    // FilePath: /config/app.php

    'aliases' => [
        'App' => Illuminate\Support\Facades\App::class;
        // ...
        'Piece' => App\Facades\Piece::class;
    ],
    ```
    这个数组中是所有设置过 Facades 的类，可以看到刚刚提及的 App::make()。

3. 现在就可以使用更简单的方式调用扩展类中的 work() 方法了，其实简单的说 Facades 只是一个别名而已，为了使代码看起来更优雅。
    ```php
    <?php
    // FilePath: /app/Http/Controllers/TestController.php

    namespace App\Http\Controllers;

    use App;
    use App\Facades\Piece;
    use App\Http\Controllers\Controller;

    class TestController extends Controller
    {
        public function index()
        {
            Piece::work();
        }
    }
    ```

4. 访问 `localhost:8888/laravel/public` 地址，页面正确输出 `i am working`

## 执行流程
接下来通过自己写的扩展透过 Laravel 源代码来理解一下 Laravel 框架中这部分的执行流程。

1. 首先请求路由：
    /app/Http/routes.php
    ```php
    <?php

    Router::get('/', 'TestController@index');  // <--
    ```

2. 通过路由，找到 TestController 中的 index() 方法：
    /app/Http/Controllers/TestController.php
    ```php
    <?php

    namespace App\Http\Controllers;

    use App;
    use App\Facades\Piece;

    class TestController extends Controller
    {
        public function index()
        {
            Piece::work(); // <--
        }
    }
    ```

3. 从现在开始，Facades 开始发挥作用，通过在App配置文件中设置的别名，自动加载 App\Facades\Piece 类
    /config/app.php
    ```php
    <?php

    'aliases' => [
        'App' => Illuminate\Support\Facades\App::class;
        // ...
        'Piece' => App\Facades\Piece::class; // <--
    ]
    ```

4. 现在便可通过别名将 Piece::work() 还原成  App\Facades\Piece::work()。接下来就是 Facades 的关键部分，这里的两个冒号表示去访问 Piece 类中的静态方法，而这个类中又没有 work() 静态方法，Facades 便是使用了 PHP 中的 __callStatic() 魔术方法使得程序跳转到这个方法中来
    /vendor/laravel/framework/src/Illuminate/Support/Facades/Facade.php
    ```php
    <?php
    public static function __callStatic($method, $args)
    {
        $instance = static::getFacadeRoot(); // <--

        switch (count($args)) {
            case 0:
                return $instance->$method();

            case 1:
                return $instance->$method();

            case 2:
                return $instance->$method($args[0], $args[1]);

        // ...
    }
    ```

    下图中的 static::getFacadeAccessor() 方法便是在 /app/Facades/Piece.php 文件中的那个方法
    /vendor/laravel/framework/src/Illuminate/Support/Facades/Facade.php
    ```php
    <?php
    public static function getFacadeRoot()
    {
        return static::resolveFacadeInstance(static::getFacadeAccessor()); // <--
    }
    ```

5. 这个方法返回 ptool ，正是在服务提供者(Service Provider)中注册的类别名
    /app/Facades/Piece.php
    ```php
    <?php
    namespace App\Facades;

    use Illuminate\Support\Facades\Facade;

    class Piece extends Facade
    {
        protected static function getFacadeAccessor()
        {
            return 'ptool'; // <--
        }
    }
    ```

6. 到这里，程序自动执行了 App::make('ptool') 这段代码，还记得使用 Facades 时将 App::make('ptool')->work() 简化成了 Piece::work() 吗，所以在这里，Facades 的工作就算结束了。
    /vendor/laravel/framework/src/Illuminate/Foundation/Application.php
    ```php
    <?php
    public function make($abstract, array $parameters=[])
    {
        $abstract = $this->getAlias($abstract);

        if (isset($this->deferredServices[$abstract])) {
            $this->loadDeferredProvider($abstract);
        }

        return parent::make($abstract, $parameters); // <--
    }
    ```

7. 通过Container 文件中的 make 方法，可以找到 ptool 对应的服务提供者
    /vendor/laravel/framework/src/Illuminate/Container/Container.php
    ```php
    <?php
    public function make($abstract, array $parameters=[])
    {
        $abstract = $this->getAlias($abstract);

        if (isset($this->instances[$abstract])) {
            return $this->instances[$abstract];
        }

        $concrete = $this->getConcrete($abstract);

        if ($this->isBuildable($concrete, $abstract)) {
            $object = $this->build($concrete, $parameters); // <--
        } else {
            $object = $this->make($concrete, $parameters);
        }
    }
    ```

    /vendor/laravel/framework/src/Illuminate/Container/Container.php
    ```php
    <?php
    public function build($concrete, array $parameters = [])
    {
        if ($concrete instanceof Closure) {
            return $concrete($this, $parameters); // <--
        }
    }
    ```

8. 找到创建的服务提供者 ( Service Provider ) 文件后，执行方法中的代码，这里返回一个实例，也就是扩展类 ( App\Tools\Piece ) 实例
    /app/Providers/ToolServiceProvider.php
    ```php
    <?php
    public function register() 
    {
        $this->app->singleton('ptool', function() {
            return new Piece; // <--
        });
    }
    ```

9. 最后，程序跳回 Facades ，执行扩展类中的 work() 方法
    /vendor/laravel/framework/src/Illuminate/Support/Facades/Facade.php
    ```php
    <?php
    public static function __callStatic($method, $args)
    {
        $instance = static::getFacadeRoot();

        switch (count($args)) {
            case 0:
                return $instance->$method(); // <--
            
            case 1:
                return $instance->$method($args[0]);

            // ...
        }
    }
    ```

10. 访问 `localhost:8888/laravel/public` 地址，页面正确输出 `i am working`。整个流程结束

## 概述文件执行流程
1. `/app/Http/routes.php` 路由文件
2. `/app/Controllers/TestController.php` 控制器，用来调用扩展类
3. `/config/app.php` 配置文件
4. `/app/Facades/Piece.php` Piece类的 Facade 文件
5. `/app/Providers/ToolServiceProvider.php` 通过服务提供者将类注册到容器
6. `/app/Tools/Piece.php` 扩展类

简单的说，请求通过路由文件 `routes.php` 执行控制器 `TestController.php`，控制器中的代码通过 App 配置文件 `app.php` 找到 Facades 文件 `\Facades\Piece.php`，Facades 文件找到 Service Provider 文件 `ToolServiceProvider.php`，通过 Service Provider 获取到扩展类 `Piece.php` 的实例 ，执行实例中的 work() 方法。

## 参考
- [Laravel 官方文档](https://laravel.com/docs/5.2)
- [Service Provider](http://laravelbase.com/collections/1/46)
- [Service Container IoC 容器](http://laravelbase.com/collections/1/47)
- [Laravel 核心：控制反转和门面模式](http://yansu.org/2014/12/06/ioc-and-facade-in-laravel.html)
- [Laravel5的Façade、Service Provider之一知半解](https://www.youtube.com/watch?v=sXvTKlZuW4M)
- [Laravel 4 - IoC Container Basics](https://www.youtube.com/watch?v=dkCW_uCMFPw)
