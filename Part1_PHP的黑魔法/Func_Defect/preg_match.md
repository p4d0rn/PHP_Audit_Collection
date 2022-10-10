# preg_match

> int **preg_match**    ( string `$pattern`   , string `$subject`)
>
> 搜索`subject`与`pattern`给定的正则表达式的一个匹配
>
> **preg_match()**在第一次匹配后  将会停止搜索，匹配成功返回1，否则返回0

💤正则表达式模式修饰符：

> /i ：ignore--不区分大小写
>
> /g ：global--全局匹配，查找所有匹配项
>
> /m ：multi line--多行匹配，使边界字符^、$匹配每一行的开头和结尾
>
> /s ：使特殊字符圆点`.`中包括换行符`\n`。默认情况下的圆点 **.** 是匹配除换行符 **\n** 之外的任何字符，加上 **s** 修饰符之后, **.** 中包含换行符 \n。

```php
<?php
$a = $_GET['test'];
if(preg_match('/^web$/im', $a)){
    if(preg_match('/^web$/i', $a)){
        die('hacker')
    }
    else{
        echo "Y0u_G0t_M3";
    }
}
```

这边`$a`传入`web%0aweb`即可绕过preg_match（`%0a为URL编码的换行符`）