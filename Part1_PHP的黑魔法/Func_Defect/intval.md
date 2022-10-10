# intval

> int  **intval**  ( [mixed] `$var`   [, int `$base` = 10  ] )
>
> 通过使用指定的进制 `base` 转换（默认十进制），返回变量 `var` 的 integer数值。       
>
> **intval()** 不能用于 object

```php
<?php
echo intval(42);                // 42
echo intval(4.2);               // 4 省略小数点
echo intval('42');              // 42
echo intval('+42');             // 42 省略+号
echo intval('-42');             // -42
echo intval(042);               // 34 0开头表示八进制
echo intval('042');             // 42
echo intval(1e10);              // PHP5输出1410065408, PHP7输出10000000000
echo intval('1e7');             // PHP5输出1, PHP7输出10000000
echo intval(0x1A);              // 26 0x开头表示十六进制
echo intval('0x1A');              // 0
echo intval(420000000000000000000);   // PHP5输出-1609564160
echo intval('420000000000000000000'); // PHP5输出2147483647
echo intval(42, 8);        // 42
echo intval('42', 8);      // 34
echo intval(array());                 // 0 空数组输出0
echo intval(array('foo', 'bar'));     // 1 非空数组输出1
```

```php
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==1234){
        die("hacker!");
    }
    if(preg_match("/[a-z]|\./i", $num)){
        die("hacker!");
    }
    if(!strpos($num, "0")){ // 0在第一位或没有0都会进入if
        die("hacker!");
    }
    if(intval($num,0)===1234){
        echo "Y0u_4re_r1ght";
    }
}
```

num=+02322   (2322是1234的八进制)

