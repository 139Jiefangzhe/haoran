![1645851776859-76ee7446-d6de-4690-8d89-e5932c2e5f94.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172207864-1362606910.png)

```plain
#知识点：
1、XSS跨站-原理&攻击&分类等
2、XSS跨站-MXSS&UXSS&FlashXss&PDFXSS等

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

3、危害
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

- UXSS-Edge&CVE-2021-34506

------

```plain
先介绍一下MXSS
#MXSS：https://www.fooying.com/the-art-of-xss-1-introduction/
简单来说，MXSS是放在浏览器上是没有伤害的payload（浏览器进行了过滤），然后把这个浏览器信息通过用户端发给QQ用户（QQ聊天窗口会有个预览功能，会把这个payload从新恢复，从而形成跨站），在绿色的网站（www.baidu.com等）就可以直接显示绿标，从而执行了MXSS代码。
这种漏洞是产生在浏览器的漏洞，并不是在网上的漏洞。所以比较难被挖掘。而且这个QQ还要符合版本，现在基本上不能安装这个版本了。所以这个漏洞基本上没有什么大的作用。
#UXSS全称Universal Cross-Site Scripting
UXSS是利用浏览器或者浏览器扩展漏洞来制造产生XSS并执行代码的一种攻击类型。
造成这个漏洞不是因为你的网站问题，而是浏览器自身的问题。
MICROSOFT EDGE uXSS CVE-2021-34506
Edge浏览器翻译功能导致JS语句被调用执行
https://www.bilibili.com/video/BV1fX4y1c7rX

这个漏洞攻击的不是网站，而是浏览器自身。（翻译）
这个是通过百度，bing，或者其他搜索引擎去搜索跨站语句的时候，当使用到了浏览器的翻译功能的时候，这个语句会被执行。不是针对网站的，是针对浏览器自己的。
正常浏览器	创建了一个本地地址http://127.0.0.1:8081/web/edge.html
访问的时候就进行了弹窗，这个是本身代码上就执行了这个语句。
```

![1645840970010-1a316aea-4502-4f35-bd26-9da0ca114147.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172230986-446731621.png)

```plain
进行翻译，发现没有任何效果。
原来的语句进行了弹窗，翻译过来的语句没有进行弹窗。
```

![1645841074481-e507a7e0-e315-45f0-b01f-c31b797c2f58.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172231083-1755041919.png)

```plain
源代码：
<b><u>行之在线测试 </u></b>
<br>

Políticas de Privacidade
Usaremos seus dados pessoais para resolver disputas, solucionar problemas e aplicar nossos Termos e Condições de Uso.

<br>

Para prevenir abusos no app/site, o Badoo usa decisões automáticas e moderadores para bloquear contas, como parte de seu procedimento de moderação. Para isso, nós conferimos contas e mensagens para encontrar conteúdo que indicam quebra dos nossos Termos e Condições de Uso. Isso é feito através de uma

<b><u>OUR PAYLOAD IN TEXT FORM </u></b>
<br>
<br>

"><img src=x onerror=alert(1)> 
//这里写Payload

<br>
<br>
<br>
Políticas de Privacidade
现在放在低版本的edge浏览器中。版本 91.0.864.37
访问http://127.0.0.1:8081/web/edge.html
很明显进行了弹窗
```

![1645841323974-0891141c-0820-4df7-b88f-7ee3d90fb063.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172230723-721355510.png)

```plain
点击翻译功能，也进行了弹窗。进行了弹窗了两次。
```

![1645841362693-6fa378ba-79ba-4101-b0bc-802b2b792f4e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172230549-589937679.png)

```plain
这个就是浏览器自身漏洞（翻译功能的漏洞）
只要搜索跨站语句，启动翻译的时候，就会触发跨站的执行。
搜索语法：<img src=x onerror=alert(1)>（只要是跨站语句就行）

但是我的测试是不成功的。可能是浏览器做了限制什么的把。
这些漏洞一般要隔几天才能爆一个，所以比较难挖掘。
```

- FlashXSS-PHPWind&SWF反编译

------

```plain
swf是小动画一些视频类的后缀

swf是可以播放视频，swf播放哪个视频也会给他传参数。也就是说这个文件格式，他是有自身的语言和编写的。而这个语言里面能接受JavaScript代码。
用js可以加载swf文件，可以进行控制swf的播放 swf调用js。

不在乎是什么语言的网站，只要调用了js。
白盒黑盒都是通过搜索swf后缀文件来进行测试。

白盒分析，找相关swf文件：D:\phpstudy_pro\WWW\phpwind9.0_5275\phpwind\www\res\js\dev\util_libs\swfupload\Flash\swfupload.swf这个文件，直接打开是乱码
```

![1645844202476-f586084b-37f9-4b40-a056-d44563b9439a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172230862-820786520.png)

```plain
利用反编译工具ffdec_15.1.0，可以对swf进行反编译处理。

主要看里面的脚本代码
```

![1645844477504-0b43d031-bf04-406e-acaa-7bf452c9ab11.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172231394-166397314.png)

```plain
如果看不懂，那么就进行函数搜索：swf调用js代码的函数：ExternalInterface.call 执行JS代码
在swf中，ExternalInterface.call函数调用JavaScript代码。
参考：https://www.cnblogs.com/mfrbuaa/p/3991385.html

代码：
flex代码
------------------
<mx:Button id="btn" label="Button" click="hello()"/>
<mx:Script>
 <![CDATA[
  private function hello():void{
         ExternalInterface.call("sayHello");
        }
 ]]>
</mx:Script>

JavaScript代码
---------------------
function sayHello(){
 alert("Hello from JavaScript!");
}

使用sayhello来调用弹出Hello from JavaScript!
使用sayhello来调用JavaScript代码。

所以，在反编译过程中搜索ExternalInterface.call，这个函数就会跟JavaScript代码进行调用。
```

![1645845806189-45fabfb6-9f01-4917-a50d-825d5ce5f1ab.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172230651-918011494.png)

```plain
代码ExternalInterface.call(param1,EscapeMessage(param3),EscapeMessage(param2),EscapeMessage(param4));
括号里面的，都可以通过JavaScript来实现，所以看括号里面有没有变量。
在D:\phpstudy_pro\WWW\phpwind9.0_5275\phpwind\www\res\js\dev\util_libs\jPlayer\Jplayer.swf，这个文件也有漏洞。
在D:\phpstudy_pro\WWW\phpwind9.0_5275\phpwind\www\res\js\dev\util_libs\jPlayer\Jplayer.swf发现关键代码 ExternalInterface.call(this.jQuery,"jPlayerFlashEvent",JplayerEvent.JPLAYER_READY,this.extractStatusData(this.commonStatus)); 
this.jQuery这个变量进行传参，
```

![1645846114370-0038093e-b631-4b94-998e-5e2a3084fd6c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172230577-1965528430.png)

```plain
追踪变量jQuery，关键性代码：this.jQuery = loaderInfo.parameters.jQuery + "(\'#" + loaderInfo.parameters.id + "\').jPlayer";
loaderInfo.parameters.jQuery这个代码可以在
```

![1645846625302-785db086-8458-4171-a4ba-93b35789c62b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172230682-2066176697.png)

```plain
参考https://blog.csdn.net/wohuazhen/article/details/20639521
其实loaderInfo.parameters是在调用传参给swf，参数就是jQuery

那么就可以构造payload：
127.0.0.1:8104/res/js/dev/util_libs/jPlayer/Jplayer.swf?jQuery=alert(1))}catch(e){}//

只有在火狐中弹窗，在其他的地方就显示下载了。
```

![1645847778423-3502370b-eb65-43b8-92d7-6fcc99c010e1.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172232030-2109644030.png)

```plain
黑盒怎么测试呢，可以寻找网站有没有swf格式文件，然后下载下来进行反编译，看他是否有安全问题。

地址mickeymouse24.com/games/
```

- PDFXSS-PDF动作添加&文件上传

------

```plain
用到迅雷PDF 新建页面，调出浓缩图
```

![1645850106470-30180670-9e35-4b50-99cb-c8a29856a47b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172231032-881220800.png)

```plain
点击浓缩图，右键属性，属性里面有个动作
```

![1645850159720-d22c3632-3c56-4ff3-97e1-2e00b31788e1.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172230909-10972960.png)

```plain
选中，新添加JavaScript代码
```

![1645850204876-edebbbea-646c-40c1-a5a5-c41b847df28b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172232137-1223342403.png)

```plain
新增代码app.alert(1);
```

![1645850277509-a1cdc41b-4f40-437d-a691-79cda5e1a631.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172230920-1458461444.png)

```plain
然后保存，制作完成。

正常来说，访问这个pdf没有任何问题。把他放在浏览器进行访问，就会进行弹窗。

只有火狐不进行弹窗
```

![1645850431833-03e6eb67-9a4d-4eb0-a0db-3ca42d5320b3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912172231039-221324938.png)

```plain
那么这个漏洞有什么作用呢。很多网站有上传功能，把PDF进行上传上去。然后得到完整地址，可以复制地址，给别人访问，别人以为是PDF就会进行下载，然后弹窗。
上传平台：https://ftpod.cn/

地址：
https://a.y8j5.top/s/LyEvQhK
https://a.e9ts.top/s/LyEvQhK
https://a.7xsh.top/s/LyEvQhK
https://a.3kcn.top/s/LyEvQhK
https://a.7bne.top/s/LyEvQhK
https://a.vij7.top/s/LyEvQhK
https://a.tg9g.top/s/LyEvQhK
https://a.zye3.top/s/LyEvQhK
https://a.ppq4.top/s/LyEvQhK
```