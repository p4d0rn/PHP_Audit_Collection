# call_user_func

`call_user_func` ：把第一个参数作为回调函数调用，其余参数是回调函数的参数。

常规用法：调用自定义或PHP自带的函数

```php
<?php
function hello($name){
    echo 'Hello '.$name.PHP_EOL;
}
call_user_func('hello', 'Taco');
call_user_func('phpinfo');
```

隐藏用法：调用类中的方法

```php
<?php
class fun {
    static function say_hello(){
        echo "Hello!\n";
    }
    function __call($method, $args){
        eval("system('calc');");
    }
}
$test = new fun();
// 调用类中的静态方法，有如下三种方式
call_user_func("fun"."::say_hello");    // 类名::方法名
call_user_func(array("fun", "say_hello"));  // array(类名, 方法名)
call_user_func(array($test, "say_hello"));  // array(对象，方法名)
// 若类中不存指定的方法，调用__call
call_user_func(array($test, '666'));
```

