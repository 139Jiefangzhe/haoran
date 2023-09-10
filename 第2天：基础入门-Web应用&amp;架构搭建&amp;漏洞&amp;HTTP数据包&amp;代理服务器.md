![1645069065152-ad786db8-3781-4136-9e7c-56e03f8947d2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909211232600-2104367572.png)

```plain
#知识点：
1、网站搭建前置知识
2、WEB应用环境架构类
3、WEB应用安全漏洞分类
4、WEB请求返回过程数据包
#WEB应用安全漏洞分类
SQL注入，文件安全，RCE执行，XSS跨站，CSRF/SSRF/CRLF，反序列化，逻辑越权，未授权访问，XXE/XML，弱口令安全等
产生层面要区分出来


#WEB请求返回过程数据包参考：
https://www.jianshu.com/p/558455228c43
https://www.cnblogs.com/cherrycui/p/10815465.html
请求数据包，请求方法，请求体，响应包，响应头，状态码，代理服务器等
Request,Response,User-Agent,Cookie,Server,Content-Length等

Accept：指浏览器或其他客户可以接爱的MIME文件格式。Servlet可以根据它判断并返回适当的文件格式。

User-Agent：是客户浏览器名称

Host：对应网址URL中的Web名称和端口号。

Accept-Langeuage：指出浏览器可以接受的语言种类，如en或en-us，指英语。

connection：用来告诉服务器是否可以维持固定的HTTP连接。http是无连接的，HTTP/1.1使用Keep-Alive为默认值，这样，当浏览器需要多个文件时(比如一个HTML文件和相关的图形文件)，不需要每次都建立连接

Cookie：浏览器用这个属性向服务器发送Cookie。Cookie是在浏览器中寄存的小型数据体，它可以记载和服务器相关的用户信息，也可以用来实现会话功能。

Referer：表明产生请求的网页URL。如比从网页/icconcept/index.jsp中点击一个链接到网页/icwork/search，在向服务器发送的GET/icwork/search中的请求中，Referer是http://hostname:8080/icconcept/index.jsp。这个属性可以用来跟踪Web请求是从什么网站来的。

Content-Type：用来表名request的内容类型。可以用HttpServletRequest的getContentType()方法取得。

Accept-Charset：指出浏览器可以接受的字符编码。英文浏览器的默认值是ISO-8859-1.

Accept-Encoding：指出浏览器可以接受的编码方式。编码方式不同于文件格式，它是为了压缩文件并加速文件传递速度。浏览器在接收到Web响应之后先解码，然后再检查文件格式。
```

![1645069176173-2b9e274b-ace9-4128-abf8-b6acbda60aa3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212338898-1472264943.png)

```plain
POST /cmnt/vote HTTP/1.1
Host: comment.sina.com.cn
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: https://sports.sina.com.cn/others/volleyball/2022-01-01/doc-ikyakumx7803364.shtml
Cookie: SUB=_2AkMW7WMxf8NxqwFRmP0Qzmvka4h3ygDEieKgsZLqJRMyHRl-yD92qnEBtRB6PW1N3mpNlSc6uR7VjBSJygF1765GdUKX; SUBP=0033WrSXqPxfM72-Ws9jqgMF55529P9D9WW3n48F2PpVgV_avH7niNaF; 
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 139
```

- #### 架构-Web应用搭建-域名源码解析

```plain
购买服务器，进行宝塔下载安装。宝塔：集成化工具
安装Apache+PHP+mysql+PHPmyadmin
一键部署可以自己下载cms，比如zblog
有手就行。
```

- #### 请求包-新闻回帖点赞-重放数据包

```plain
设置代理，能进行包重发
新浪新闻：news.sina.com.cn
```

![1645069330722-62cbdff9-8100-4558-8604-cf2140ffc8a3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212352562-651243504.png)

```plain
点赞进行抓包，然后把抓到的包重放，就可以进行无限次点赞操作。
点赞数据包：发送到repeater模块，就可以进行重放，实验成功。
```

![1645069356528-2653efe4-72f7-4486-a69f-b147a1c4f93b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212356546-127006738.png)

```plain
抓不到https的数据包的时候，设置：
```

![1645069383166-75e9b921-b6e1-4d7b-8f2f-02cee0273679.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212359816-1815713425.png)

- #### 请求包-移动端&PC访问-自定义UA头

```plain
安卓访问：
GET / HTTP/1.1
Host: www.baidu.com
User-Agent: Mozilla/5.0 (Android 7.1.2; Tablet; rv:68.0) Gecko/68.0 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Cookie: BAIDUID=582046B331C25B9F4A11725B4C4D814D:FG=1; H_WISE_SIDS=107318_110085_114550_127969_132547_164870_179345_182234_184716_186635_186743_186840_187828_188740_189254_189325_189732_189755_190248_190473_190621_191067_191250_191368_191736_191810_192010_192206_192237_192351_192387_193037_193284_193348_193559_194085_194520_194583_195038_195108_195188_195342_195423_195552_195577_195679_195756_195915_196046_196051_196230_196428_196513_196834_196925_196940_197241_197288_197294_197337_197470_197471_197582_197710_197783_197810_197845_198076_198181_198189_198243_198260_198312_198419_198512_198579_198583_198649_198748_198897_198997_199023_199232_199302_199467_199566_199675_199840_199847_199877_199968; Hm_lvt_12423ecbc0e2ca965d84259063d35238=1638805266,1639040042; rsv_i=85a28ee9x%2FywqxcJFJBh4a3wljQbeaE0my5E8hL5ATMrcrDBUFBOOIz4x%2Fy%2FS%2FeqdoqROV8cBAKDFCZ6vO5MTqJLFC86iu4; BDSVRTM=345; plus_lsv=e9e1d7eaf5c62da9; plus_cv=1::m:7.94e+147
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
显示页面：
```

![1645069437681-896faf74-22dd-46dd-9d99-55c214967fde.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212402916-638513757.png)

```plain
pc访问：

GET / HTTP/1.1
Host: www.baidu.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Cookie: BAIDUID=8DCC4DBC473D921F92F028884ABD901C:FG=1; BIDUPSID=8DCC4DBC473D921F2292212FDC9EB8D6; PSTM=1638837611; BD_UPN=13314752; BDORZ=B490B5EBF6F3CD402E515D22BCDA1598; H_PS_PSSID=35637_34442_35104_31254_35626_34584_35491_35234_35644_35320_26350_35478_35561; BA_HECTOR=0hah8ka12g0405003b1gt4moe0q; baikeVisitId=caf11396-3737-45ea-bd2d-03a92b598836; delPer=0; BD_CK_SAM=1; PSINO=6; BD_HOME=1
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1

显示界面：
```

![1645069471377-7a9c0db5-f572-4656-b4c7-abb493cb37ba.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212406066-582226738.png)

```plain
两个访问的页面不一样，网站可以通过ua头来进行判断是手机访问，还是电脑访问。
电脑ua:
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
安卓模拟器ua：
User-Agent: Mozilla/5.0 (Android 7.1.2; Tablet; rv:68.0) Gecko/68.0 Firefox/68.0
修改au头后，完成测试：
```

![1645069504918-c5a7c777-67bb-46e8-b4fa-84936fd5b30b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212408619-117930313.png)

- #### 返回包-网站文件目录扫描-返回状态码

```plain
返回信息：

返回状态码：200，300,400,403,404,500
3XX跳转 处理过程，判断可有可无 
5XX内部错误，服务器问题，文件判断可有可无

复制图片地址：http://xiaodi8.com/zb_users/upload/2019/12/201912151576406028214564.jpg
抓包请求：找一个不存在的地址，返回404
```

![1645069534277-798d2554-6e3a-498d-84e1-7ff6ce555949.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212411165-1709700275.png)

```plain
一个存在地址，返回：
```

![1645069560691-3c310afb-553b-4a08-81c8-42ce7bfec3aa.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212413582-1686219710.png)

```plain
可以根据对方放回的数据包的值来判断文件是否存在（状态码）
存在文件夹返回：
```

![1645069584086-d8fc9f9c-8027-4bdc-acbc-9c4db41cc9ce.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212416759-1278239245.png)

```plain
不存在文件夹返回：
```

![1645069603561-b8406a63-59d3-444f-8156-6134774927e8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212419330-1514448529.png)

```plain
目录扫描可以用到burpsuite，进行简单的扫描。
```

- #### 数据包-WAF文件目录扫描-代理服务器

```plain
• 代理服务器原理：
没设置代理
浏览器--服务器
设置代理
浏览器--代理--服务器
127.0.0.1 8888
burpsuite监听127.0.0.1 8888
burpsuite拦截后选择性的进行丢弃或者放行

购买代理：www.kuaidaili.com
```

![1645069641992-37293215-52e4-4cdb-bb5d-ce8b437710f4.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909212422708-128173126.png)
