![1647043658519-1d53f5f1-3ba3-4491-85b0-e466c4d6955d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103304256-904895942.png)

```plain
#知识点：
1、文件操作类安全问题
2、文件下载&删除&读取
3、白盒&黑盒&探针分析

#详细点：
文件读取：基本和文件下载利用类似
文件下载：利用下载获取源码或数据库配置文件（包括接口配置文件）及系统敏感文件为后续出思路（）
文件删除：除自身安全引发的文件删除外，可配合删除重装锁定文件进行重装（InstallLock）文件
```

- 审计分析-文件下载-XHCMS-功能点

------

```plain
白盒审计流程：
功能点抓包-寻代码文件-寻变量控制-构造测试

在页面中看到有下载的东西，然后寻找到相对应的下载地址：http://127.0.0.1:8087/?r=downloads&type=soft&line=pan&cid=1

来到index文件，
$file=addslashes($_GET['r']); //接收文件名
include('files/'.$action.'.php'); //载入相应文件
传入参数r=downloads，所以包含了downloads.php

在downloads.php看到传入参数type，line和cid
那么在下载的时候，会触发什么数据包呢？进行抓包
```

![1647048420915-391d38ca-6888-4842-ad48-73e651d15dda.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103323389-751652217.png)

```plain
结合代码分析，
switch ($outFile_extension) {  
    case "exe" :  
        $ctype = "application/octet-stream";  
        break;  
    case "zip" :  
        $ctype = "application/zip";  
        break;  
    case "mp3" :  
        $ctype = "audio/mpeg";  
        break;  
    case "mpg" :  
        $ctype = "video/mpeg";  
        break;  
    case "avi" :  
        $ctype = "video/x-msvideo";  
        break;  
    default :  
        $ctype = "application/force-download";  
}
```

![1647048540057-d1d121ad-5602-440b-a08b-e7c6f00351c9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103323722-460006758.png)

```plain
这个代码就是结合数据包Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8进行文件的筛选
为什么有些文件打开就提示下载，有些文件打开就可以直接访问，就是根据数据包解析不同来进行区分的。

继续查看代码，发现对传入的参数进行判断
```

![1647048806119-cb531439-f498-499c-bc3d-24d4baf25b0a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103324570-2038950991.png)

```plain
if ($type=='soft' AND ($line=="telcom" OR $line=="unicom"))
很明显，刚刚抓到的下载链接是http://127.0.0.1:8087/?r=downloads&type=soft&line=pan&cid=1
再次抓取下载文件链接：http://127.0.0.1:8087/?r=downloads&type=soft&line=telcom&cid=1和http://127.0.0.1:8087/?r=downloads&type=soft&line=unicom&cid=1
这样line就符合判断。进入循环

思路：
下载漏洞，任意文件下载
下载是谁，下载的文件能不能控制，能控制就有漏洞

继续跟踪代码，发现要下载的文件名$sourceFile，搜索变量最初的位置是$sourceFile = $fileadd; //要下载的临时文件名
```

![1647049377381-00a7ae79-f42d-414c-877a-7b7496155748.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103324569-1649848833.png)

```plain
整个代码：
if ($type=='soft' AND ($line=="telcom" OR $line=="unicom")){
$filename=$down['title'];
$filename2=$down['version'];
$filename=iconv("UTF-8", "GBK", $filename);
$houzhui=substr($fileadd,strrpos($fileadd,"."));
$sourceFile = $fileadd; //要下载的临时文件名  
$outFile = $filename." ".$filename2.$houzhui; //下载保存到客户端的文件名  
$file_extension = strtolower(substr(strrchr($sourceFile, "."), 1)); //获取文件扩展名  
//echo $sourceFile;  
//if (!ereg("[tmp|txt|rar|pdf|doc]", $file_extension))exit ("非法资源下载");  
//检测文件是否存在  
if (!is_file($sourceFile)) {  
    die("<script language=JavaScript>alert('抱歉，本地下载未发现文件，请选择网盘下载！');history.back();window.close();</script>");  
}  
$len = filesize($sourceFile); //获取文件大小  
$filename = basename($sourceFile); //获取文件名字  
$outFile_extension = strtolower(substr(strrchr($outFile, "."), 1)); //获取文件扩展名 

sourceFile是由fileadd控制，继续搜索fileadd变量哪里来

$fileadd=$down['softadd'];
```

![1647049539189-faef8112-ae7b-4490-9b7a-23fb823df6f9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325175-909791153.png)

```plain
这个变量由$down['softadd'];控制

继续看down是从哪里来：
$query = "SELECT * FROM download WHERE ( id='$fileid')";
$result = mysql_query($query) or die('SQL语句有误：'.mysql_error());
$down= mysql_fetch_array($result);
可以知道down是执行SQL语句过来。

整体流量：
数据库查询出来-->down-->fileadd-->sourceFile

打开数据库，查询的是download这个表的数据
fileid是由cid控制，查询1，然后获取softadd的值给到down
```

![1647049946800-6cdb2239-461c-470c-9c68-974879afbe59.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103322996-1696082158.png)

```plain
那么找出处，这个表的数据由谁控制他？谁能操作这个叫download的表。
如果要控制的话，要么更改这个值，要么添加这个值（insert，upload）不能通过删除或者查询去更改他。
全局搜索，看那个数据库调用download，找insert或者update的执行语句
```

![1647050577007-abed1584-cb65-4ff0-83d9-4996924eb654.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103324996-1743408707.png)

```plain
在D:\phpstudy_pro\WWW\xhcms\admin\files\newsoft.php
不能http://127.0.0.1:8087/admin/files/newsoft.php，这样是访问不到的。
访问http://127.0.0.1:8087/admin/?r=login登录后台
查看admin/index.php

$file=addslashes($_GET['r']); //接收文件名
$action=$file==''?'index':$file; //判断为空或者等于index
include('files/'.$action.'.php'); //载入相应文件

构造\admin\files\newsoft.php访问地址：http://127.0.0.1:8087/admin/?r=newsoft
```

![1647051148351-a1f8201b-adae-4b0a-bc2c-f473c43c9629.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325958-425772652.png)

```plain
再次观察哪一个是控制softadd的，通过post提交
```

![1647051207969-da009fdc-a302-4ef7-9fbb-11e994abc13e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325213-1804972376.png)

```plain
然后保存数据包，在前端访问，重新下载
```

![1647051647624-48669d62-aacb-40fd-945d-df3aa44cf094.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325644-1825023740.png)

- 审计分析-文件读取-MetInfo-函数搜索

------

```plain
流程：特定函数搜索-寻触发调用-构造Payload测试
文件读取一般在页面中很难看出，所以需要搜索php读取的函数（readfile）
https://www.cnblogs.com/gyrgyr/p/5774436.html

全局搜索readfile，在/app/system/include/module/old_thumb.class.php目录下

代码中readfile($dir);那么dir在哪里来的
dir是从$dir = str_replace(array('../','./'), '', $_GET['dir']);过来的
```

![1647055009648-0a1a828c-c97a-4033-88d2-edd999bdf333.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103324771-1535669469.png)

```plain
通过get传参，那么就要看这个文件怎么访问他，看明显这个代码是写在了class上面去了。对象名：old_thumb
直接搜索old_thumb，看看谁调用过
```

![1647055186542-b2de630a-6c83-4d7b-8dd1-5e627d20334f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325013-463483487.png)

```plain
/include/thumb.php首先看到define('M_CLASS', 'old_thumb');这里并没有调用
如果不懂，那就看包含文件/app/system/entrance.php
define ('PATH_SYS_MODULE', PATH_WEB."app/system/include/module/");
刚刚方法的路径是/app/system/include/module/old_thumb.class.php

还包含了require_once PATH_SYS_CLASS.'load.class.php';
这个变量是PATH_SYS_CLASS=app/system/include/class/

打开app/system/include/class/load.class.php发现也没什么
在/include/thumb.php有引用，相当于调用了。
那么构造payload：/include/thumb.php?dir=http\..\..\config\config_db.php
```

![1647056464807-6e1efd37-ca67-4de1-a008-defc4179ca3d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325043-1151705424.png)

- 审计分析-文件删除-74CMS-函数搜索

------

```plain
文件删除一般在会员中心或者后台来进行操作，基本上不会在页面中展示。

文件删除函数unlink，直接全局搜索
在/upload/admin/admin_article.php路径下
```

![1647057487648-be5cf103-d9eb-425b-998b-d7a53e3abfb8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103324948-1334209538.png)

```plain
@unlink($thumb_dir.$img);看thumb_dir和img能不能控制，发现img是通过get传参的，所以img是可以控制的。
直接访问这个文件http://127.0.0.1:8092/admin/admin_article.php

添加新闻的时候http://127.0.0.1:8092/admin/admin_article.php?act=news_add
然后在代码中搜索act=news_add
```

![1647057966547-33c8232e-08ef-45ee-b626-4054a71fb20f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325063-1627194000.png)

```plain
继续看删除代码，所以也有act=del_img
```

![1647058012898-dabcd795-242a-4544-8e68-014c78127079.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325141-1850489314.png)

```plain
构造触发代码：http://127.0.0.1:8092/admin/admin_article.php?act=del_img
@unlink($upfiles_dir.$img);
找upfiles_dir定义的地方
$upfiles_dir.=date("Y/m/d/");
可以知道upfiles_dir是年月日，然后在创建一个文件
```

![1647062024424-02f581e2-c03e-496f-98ec-11d3445cb00f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325312-902496524.png)

```plain
这个是拼接语句，继续寻找发现在/upload/admin/include/admin_common.inc.php文件中有定义
定义语句：$upfiles_dir="../data/".$_CFG['updir_images']."/";
```

![1647062469916-7efd0711-7768-4b2c-884c-372041064d1e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325524-2012016673.png)

```plain
在之前的文件也包含了require_once(dirname(__FILE__).'/include/admin_common.inc.php');

构造payload：http://127.0.0.1:8092/admin/admin_article.php?act=del_img&img=../../1.txt
```

![1647063397143-9052f3ab-f256-4c55-9c19-2b64145386a5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913103325119-1750766346.png)

- 黑盒分析-下载读取-下载资源URL参数

------

```plain
67.202.70.133 Joomla






files/readfile.php?file=../configuration.php



http://down.znds.com/getdownurl/?s=L2Rvd24vMjAyMjAxMjUvbWd0dkRCRUlfNi4wLjkwMl9kYW5nYmVpLmFwaw==

L2Rvd24vMjAyMjAxMjUvbWd0dkRCRUlfNi4wLjkwMl9kYW5nYmVpLmFwaw==是base64编码


#黑盒探针
1、URL参数名及参数值分析：
参数名：英文对应翻译
参数值：目录或文件名
2、功能点自行修改后分析：
文件下载，删除，读取等
```