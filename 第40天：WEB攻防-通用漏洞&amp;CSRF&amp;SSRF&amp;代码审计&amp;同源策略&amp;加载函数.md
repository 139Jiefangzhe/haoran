![1646447255289-e72c09dd-b1e6-47c5-8fee-16156abd4ac9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102233252-1027053856.png)

```plain
#知识点：
1、CSRF-审计-复现测试&同源策略
2、SSRF-审计-功能追踪&函数搜索

#详细点：
CSRF全称：Cross-site request forgery，即，跨站请求伪造，也被称为 “One Click Attack” 或 “Session Riding”，通常缩写为CSRF或者XSRF，是一种对网站的恶意利用。举个生活中的例子：就是某个人点了个奇怪的链接，自己什么也没输，但自己的qq号或其他的号就被盗了。即该攻击可以在受害者不知情的情况下以受害者名义伪造请求，执行恶意操作，具有很大的危害性。
CSRF的攻击过程两个条件：
1、目标用户已经登录了网站，能够执行网站的功能。
2、目标用户访问了攻击者构造的URL。
CSRF安全问题黑盒怎么判断：
1、看验证来源不-修复
2、看凭据有无token--修复
3、看关键操作有无验证-修复
-CSRF安全问题白盒怎么审计：
同黑盒思路一样，代码中分析上述三看

SSRF(Server-Side Request Forgery:服务器端请求伪造) 是一种由攻击者构造形成由服务端发起请求的一个安全漏洞。一般情况下，SSRF攻击的目标是从外网无法访问的内部系统。（正是因为它是由服务端发起的，所以它能够请求到与它相连而与外网隔离的内部系统）SSRF 形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能且没有对目标地址做过滤与限制。比如从指定URL地址获取网页文本内容，加载指定地址的图片，下载等等。
-SSRF黑盒可能出现的地方：
1.社交分享功能：获取超链接的标题等内容进行显示
2.转码服务：通过URL地址把原地址的网页内容调优使其适合手机屏幕浏览
3.在线翻译：给网址翻译对应网页的内容
4.图片加载/下载：例如富文本编辑器中的点击下载图片到本地；通过URL地址加载或下载图片
5.图片/文章收藏功能：主要其会取URL地址中title以及文本的内容作为显示以求一个好的用具体验
6.云服务厂商：它会远程执行一些命令来判断网站是否存活等，所以如果可以捕获相应的信息，就可以进行ssrf测试
7.网站采集，网站抓取的地方：一些网站会针对你输入的url进行一些信息采集工作
8.数据库内置功能：数据库的比如mongodb的copyDatabase函数
9.邮件系统：比如接收邮件服务器地址
10.编码处理, 属性信息处理，文件处理：比如ffpmg，ImageMagick，docx，pdf，xml处理器等
11.未公开的api实现以及其他扩展调用URL的功能：可以利用google 语法加上这些关键字去寻找SSRF漏洞
一些的url中的关键字：share、wap、url、link、src、source、target、u、3g、display、sourceURl、imageURL、domain……
12.从远程服务器请求资源（upload from url 如discuz！；import & expost rss feed 如web blog；使用了xml引擎对象的地方 如wordpress xmlrpc.php）
-SSRF白盒可能出现的地方：
1、功能点抓包指向代码块审计
2、功能点函数定位代码块审计
-SSRF常见安全修复防御方案：
1、禁用跳转
2、禁用不需要的协议
3、固定或限制资源地址
4、错误信息统一信息处理

#系列内容点：
1、CSRF&SSRF&原理&利用&协议等
2、CSRF&SSRF&黑盒&审计&修复等
这类型的漏洞应该怎么挖掘呢

CSRF
特性挖漏洞
思路：
1.直接复现有没有
失败-->代码-->完整过滤-->没有漏洞
失败-->代码-->缺陷过滤（绕过）-->有漏洞
成功-->有

其他漏洞：
有关键函数和应用功能
```

- 代码审计-CSRF-SCMSFH无验证

```plain
网站：http://127.0.0.1:8108/admin/admin.php
先测试抓包，看看能不能实现。

抓包测试，构造访问地址
```

![1646449474455-fdb78c39-13dc-485e-a369-f02973da2d17.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102239165-1785182757.png)

```plain
构造地址，127.0.0.1:8081/web/11.html
```

![1646450698982-5e11efda-c74a-42b4-854e-c28ca18e8854.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102239365-1181264233.png)

```plain
成功添加用户，那证明有这个漏洞，并且没有任何过滤。

那么我们从代码上分析这个漏洞的形成。

添加用户的URL地址：http://127.0.0.1:8108/admin/admin.php?action=add
找到admin/admin.php，并且action=add参数的代码。
```

![1646450847881-5ea5e091-02f6-4196-bb6e-36b02433825e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102239759-58218223.png)

```plain
这些代码只是判断admin不合法，并没有验证他的来源，或者验证他的token值等，并没有过滤CSRF措施的逻辑判断。	
```

- 代码审计-CSRF-ZBLOG同源策略

```plain
继续同样的操作，添加账号，然后访问构造地址。访问地址：127.0.0.1:8081/web/22.html
```

![1646451879362-7d86757a-824e-4332-b11e-d8945143f109.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102239789-184422386.png)

```plain
这个就提示了非法访问，然后在代码上分析这个访问的来源。

添加账号的地址：http://127.0.0.1:8110/zb_system/cmd.php?act=MemberPst&csrfToken=753c2775ebd6103da6b4dcd352ec8fce
很明显，后面自带了token值。
找到相对应的目录zb_system/cmd.php，传参act=MemberPst，后面的token值先不用理。
搜索：MemberPst
```

![1646455708877-361d68ed-ab75-4bcd-8faa-46ed2cf989de.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102239726-1097220105.png)

```plain
发现关键函数CheckIsRefererValid();，我们可以从字面意思理解是防护Referer，是属于验证Referer的（同源策略）
那么继续跟踪追踪CheckIsRefererValid()函数
```

![1646455875715-211df065-bc77-4938-a852-4d0411f90c4f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102239332-1758292856.png)

```plain
发现两个自定义的函数CheckHTTPRefererValid和CheckCSRFTokenValid，继续跟踪代码CheckHTTPRefererValid，转到声明地方
```

![1646455969336-6e6208b0-6696-4eaf-86ca-f0f334a69b92.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102239430-1593407455.png)

```plain
GetVars('HTTP_REFERER', 'SERVER');这个函数应该是自带的函数，获取HTTP_REFERER的值。其实也不一定要去弄懂，看到英文单词意思是说这个是检测来源的。

那么现在修改来源，看看能不能添加成功。
访问127.0.0.1:8081/web/22.html，进行抓包
```

![1646456159511-3f36e583-b147-46b5-a9af-38865381e834.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102240056-32452540.png)

```plain
发现来源Referer: http://127.0.0.1:8081/web/22.html
然后把他进行修改，改为Referer: http://127.0.0.1:8110/zb_system/cmd.php

但是这里还是不能成功，可能是数据包过久的问题。还是不行，那就不知道什么原因了。
那么这种来源能不能伪造呢？
简单来说是可以伪造的，在代码上作文章。绑定这个referer的值，强制性绑定来源。

来源检测绕过：
1.伪造，需要在这个代码数据包文件固定来源。
2.尝试在网站寻找可上传地方，上传数据包文件，取得当前同域名访问地址。上传一般不会限制html文件。
那么应该怎么伪造呢？

在http://127.0.0.1:8081/web/getrefer.php代码中接受来源地址
<?php 
$xx=$_SERVER['HTTP_REFERER'];
echo $xx;
?>
写python脚本
import requests
url='http://127.0.0.1:8081/web/getrefer.php'
headers={
    'Referer':'https://baidu.com/'
}

code=requests.get(url,headers=headers)
print(code.text)
```

- 代码审计-SSRF-Yzmcms功能&函数

```plain
这个在黑盒上测试是在后台上产生的漏洞，是采集管理模块。（所以有些鸡肋）
```

![1646460460490-f8db0d3a-49eb-4b4b-8dd2-111ac43e51af.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102240229-1106766285.png)

```plain
这个就是服务器去请求外网的信息，访问别人的地址（服务器自己访问）

在安装目录下创建一个ssrf.html
内容：
<test123><a href="file://d:/www.txt">123</a></test123>
```

![1646461572737-8ca3003d-81c6-45f4-843c-fa3c01458555.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102239727-1644951321.png)

```plain
填写好内容，点击采集，读取到了w.txt的内容
```

![1646461848309-f9d7d789-f38e-475a-8cb6-5989be6ab89f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102239832-368866843.png)

```plain
那么这个在白盒上应该怎么分析呢？

添加采集的时候，抓包分析，触发的地址为/collection/collection_content/add.html
```

![1646462916229-77e2b69c-18c7-40ae-8df8-82033cf9360f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102240199-1424917995.png)

```plain
攻击点是这个http://127.0.0.1:8109/ssrf.html地址，那么应该怎么处理这个地址是关键。
数据包接受这个参数是：urlpage=http%3A%2F%2F127.0.0.1%3A8109%2Fssrf.html

找到代码对应文件application/collection/controller/collection_content.class.php，参数为add
```

![1646463113125-8d894303-3607-4b04-bc39-88765ac18fa7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102240824-1069895375.png)

```plain
这种写法是框架写法。
添加到数据库：$res = D('collection_node')->insert($_POST);
点击测试采集的时候进行抓包，请求地址：/collection/collection_content/collection_test/id/1.html
这个是框架，那么他对应的目录应该是application/collection/controller/collection_content.class.php
id是参数名，1.html是值
找到collection_test方法代码段
```

![1646463218721-f9dc3aa4-1321-4a68-b64a-f3dd735a67e2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102241149-499224183.png)

```plain
取出数据
		$data = D('collection_node')->where(array('nodeid' => $id))->find();
		if(!$data || $data['urlpage'] == '') showmsg('网址配置不能为空！', 'stop');
 
 那用什么来处理这个urlpage呢
```

![1646463418810-0094cfce-ee0c-4daf-9916-231eb6b8ae3f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102240861-1391462610.png)

```plain
用到两个函数来请求这个地址get_content和get_filter_html
转到函数get_content声明处
```

![1646463529338-be53e029-558b-4df1-b86e-ec201da8c8e8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102241244-20529189.png)

```plain
发现关键函数file_get_contents，将整个文件读入一个字符串
参考https://www.php.net/manual/zh/function.file-get-contents.php
整个思路：

1.特定漏洞功能-->代码段分析审计
2.特定漏洞函数-->则是对应功能判断

功能点--采集
采集添加--测试--抓包--代码--远程请求资源操作（函数）

函数点--功能审计
file_get_contents函数可导致ssrf产生漏洞

也可以用来搜索（全局）
```

![1646463910064-51832b98-709b-4e49-a06e-5f7bf34af5ce.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913102245574-1612783313.png)