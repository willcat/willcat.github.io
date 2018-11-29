---
title: php匿名函数/闭包(closure)
date: 2018-11-29 14:17:24
tags:[php]
categories: [编程语言]
---
# php之匿名函数(闭包)
匿名函数（Anonymous functions），也叫闭包函数（closures），允许 临时创建一个没有指定名称的函数。最经常用作回调函数（callback）参数的值。当然，也有其它应用的情况。
**匿名函数目前是通过 Closure 类来实现的。**
>*注意*： 理论上讲闭包和匿名函数是不同的概念，不过PHP将其视作相同的概念（匿名函数在PHP中也叫作闭包函数），所以下面提到闭包时指的也是匿名函数；反之亦然

# 用法
## 调用方式
### callback/匿名函数方式
```php
echo preg_replace_callback('~-([a-z])~', function ($match) {
    return strtoupper($match[1]);
}, 'hello-world');
// 输出 helloWorld
```

### 变量方式
闭包函数也可以作为*变量的值*来使用。PHP 会自动把此种表达式转换成内置类[`Closure`](http://php.net/manual/zh/class.closure.php)的对象实例。把一个 closure 对象赋值给一个变量的方式与普通变量赋值的语法是一样的，最后也要加上分号。

```php
$greet = function($name)
{
    printf("Hello %s\r\n", $name);
};

$greet('World');
$greet('PHP');
```

## 从父作用域继承变量
闭包可以从父作用域中继承变量。 任何此类变量都应该用 `use` 语言结构传递进去。 `PHP 7.1` 起，不能传入此类变量： `superglobals`、 `$this` 或者和参数重名。
```php
$message = 'hello';

// 没有 "use"
$example = function () {
    var_dump($message);
};
echo $example();

// 继承 $message
$example = function () use ($message) {
    var_dump($message);
};
echo $example();

// 继承的变量值来自函数定义时的赋值，而非调用的时候。
$message = 'world';
echo $example();

//重置 $message
$message = 'hello';

// 使用引用传递继承变量
$example = function () use (&$message) {
    var_dump($message);
};
echo $example();

// 父作用域中修改的值，在函数调用里被反应出来。
$message = 'world';
echo $example();

//闭包也可以接受常规参数，如下面的$arg
$example = function ($arg) use ($message) {
    var_dump($arg . ' ' . $message);
};
$example("hello");
```
>上述例程的输出
```
Notice: Undefined variable: message in /example.php on line 6
NULL
string(5) "hello"
string(5) "hello"
string(5) "hello"
string(5) "world"
string(11) "hello world"
```

## Closures 和作用域
从父作用域中继承变量与使用全局变量是不同的。全局变量存在于一个全局的范围，无论当前在执行的是哪个函数。而 闭包的父作用域是定义该闭包的函数（不一定是调用它的函数）。

## `$this`的自动绑定
As of PHP 5.4.0, when declared in the context of a class, the current class is automatically bound to it, making $this available inside of the function's scope. If this automatic binding of the current class is not wanted, then static anonymous functions may be used instead.
从php5.4.0开始，在一个类的上下文中声明一个闭包时，当前类会被自动绑定到闭包里， 从而使`$this`在函数作用域中可以使用。如果不需要自动绑定当前类，则可以改为用静态匿名函数。
```php
class Test
{
    public function testing()
    {
        return function() {
            var_dump($this);
        };
    }
}

$object = new Test;
$function = $object->testing();
$function();
/***输出****/
object(Test)#1 (0) {
}
```
## 静态匿名函数
从PHP 5.4开始，可以静态声明匿名函数。 这可以防止它们将当前类自动绑定到它们。 对象也可能在运行时也不会绑定到它们。
```php
class Foo
{
    function __construct()
    {
        $func = static function() {
            var_dump($this);
        };
        $func();
    }
};
new Foo();
/**输出
Notice: Undefined variable: this in %s on line %d
NULL
*/

/***尝试绑定对象到静态匿名函数***/
$func = static function() {
    // function body
};
$func = $func->bindTo(new StdClass);
$func();
```

# 参考
[php manual:匿名函数](http://php.net/manual/zh/functions.anonymous.php)
[现代 PHP 新特性系列（五） —— 闭包和匿名函数](https://laravelacademy.org/post/4341.html)
